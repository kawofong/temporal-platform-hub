---
sidebar_position: 2
id: ai-reference-architecture
title: AI Agent Reference Architecture
sidebar_label: 🏗️ Reference Architecture
description: How to structure a production agentic AI stack with Temporal — orchestrator Workflows, tool Activities, LLM retry strategy, and human-in-the-loop gates.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

This page covers the canonical way to structure a Temporal-backed agentic AI system. The patterns described here are derived from the [temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent) reference implementation and production deployments at ABC Financial.

## Core principle: Workflows orchestrate, Activities execute

The single most important rule in a Temporal AI stack is that **all non-deterministic I/O belongs in Activities, not Workflows**. This includes every LLM call, every tool invocation, every external API request, and every database read.

Workflow code is replayed from history during failure recovery. If you call an LLM directly inside a Workflow, the replay will attempt to call the LLM again — producing a different result, breaking determinism, and corrupting the Workflow's state. Wrapping the LLM call in an Activity solves this: Activities are side effects that are recorded once and replayed from history.

## Architecture overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Orchestrator Workflow                        │
│                                                                  │
│   ┌──────────────┐      ┌──────────────────────────────────┐    │
│   │  Agent State │      │          Agent Loop               │    │
│   │  (messages,  │◄────►│  plan → act → observe → repeat   │    │
│   │   step count)│      └──────────────┬───────────────────┘    │
│   └──────────────┘                     │                        │
│                                        │ execute_activity()     │
└────────────────────────────────────────┼────────────────────────┘
                                         │
              ┌──────────────────────────┼──────────────────────────┐
              │                          │                          │
              ▼                          ▼                          ▼
     ┌─────────────────┐      ┌─────────────────────┐    ┌─────────────────┐
     │   LLM Activity  │      │   Tool Activities   │    │  HITL Gate      │
     │                 │      │                     │    │                 │
     │ - Call LLM API  │      │ - Search / lookup   │    │ Signal or       │
     │ - Heartbeat on  │      │ - Database query    │    │ Update waits    │
     │   streaming     │      │ - External API call │    │ for human       │
     │ - Structured    │      │ - File I/O          │    │ decision with   │
     │   retry policy  │      │ - Parallel dispatch │    │ timeout fallback│
     └─────────────────┘      └─────────────────────┘    └─────────────────┘
```

## The Orchestrator Workflow

The Workflow is the durable brain of your agent. It holds the conversation history and any pending state. It never calls the LLM or any external service directly — it schedules Activities for that work and waits for their results.

Rather than accepting a single task string at start, the Workflow exposes a `send_message` **Update** that the client calls once per user turn. The Update handler delivers the message and suspends until the `run` loop finishes the turn. The `run` loop owns the full plan → act → observe logic — it waits for a turn to arrive, executes all Activity calls, sets the reply, then loops back to wait for the next turn.

Keeping the loop in `run` makes the execution path easier to trace: the Workflow history shows a clear sequence of turn-wait → activity calls → reply for each user message. Updates still give the client a synchronous reply without polling.

```python
import asyncio
from dataclasses import dataclass
from datetime import timedelta
from typing import Optional
from temporalio import workflow
from temporalio.exceptions import ApplicationError

# Activity modules must be imported with imports_passed_through to prevent
# the Python Workflow sandbox from reloading them on every Workflow task.
with workflow.unsafe.imports_passed_through():
    from activities import call_llm, execute_tool

@dataclass
class AgentInput:
    system_prompt: str  # Initial instructions; user messages arrive via Update

@workflow.defn
class AgentWorkflow:
    MAX_STEPS_PER_TURN = 20  # LLM + tool iterations allowed per user message

    def __init__(self) -> None:
        self._system_prompt: str = ""
        self._messages: list[dict] = []
        self._done: bool = False
        self._turn_ready: bool = False
        self._reply: Optional[str] = None

    # ── Update: one call per user turn ───────────────────────────────────────

    @workflow.update
    async def send_message(self, user_message: str) -> str:
        """Deliver a user message and block the caller until the agent replies.

        Appends the message and signals the run loop that a new turn is ready,
        then suspends until the loop finishes processing and sets the reply.
        """
        self._messages.append({"role": "user", "content": user_message})
        self._turn_ready = True
        await workflow.wait_condition(lambda: self._reply is not None)
        reply = self._reply
        self._reply = None
        return reply

    @send_message.validator
    def validate_send_message(self, user_message: str) -> None:
        """Reject invalid messages before they are accepted into event history.

        Validators run synchronously before the Update is recorded — rejected
        Updates cost no history events and return an error to the caller immediately.
        """
        if not user_message or not user_message.strip():
            raise ValueError("user_message must not be empty")
        if self._done:
            raise ValueError("cannot send a message to a session that has ended")

    # ── Signal: end the session ───────────────────────────────────────────────

    @workflow.signal
    def end_session(self) -> None:
        """Cleanly terminate the Workflow after the current turn completes."""
        self._done = True

    # ── Query: read conversation state ────────────────────────────────────────

    @workflow.query
    def get_messages(self) -> list[dict]:
        """Return the full conversation history — safe to call at any time."""
        return self._messages

    # ── Main loop ─────────────────────────────────────────────────────────────

    @workflow.run
    async def run(self, input: AgentInput) -> str:
        self._system_prompt = input.system_prompt

        while not self._done:
            # Block until a new user turn arrives or the session ends.
            # wait_condition is durable — no worker thread is consumed while waiting.
            await workflow.wait_condition(
                lambda: self._turn_ready or self._done
            )
            if self._done:
                break
            self._turn_ready = False

            # Agent loop: plan → act → observe, up to MAX_STEPS_PER_TURN iterations
            for _ in range(self.MAX_STEPS_PER_TURN):
                response = await workflow.execute_activity(
                    call_llm,
                    args=[self._system_prompt, self._messages],
                    start_to_close_timeout=timedelta(minutes=5),
                )
                self._messages.append({"role": "assistant", "content": response.content})

                if response.is_final:
                    # Signal the Update handler that the reply is ready
                    self._reply = response.content
                    break

                if response.tool_calls:
                    tool_results = await asyncio.gather(*[
                        workflow.execute_activity(
                            execute_tool,
                            args=[tc.name, tc.arguments],
                            start_to_close_timeout=timedelta(minutes=2),
                        )
                        for tc in response.tool_calls
                    ])
                    for tc, result in zip(response.tool_calls, tool_results):
                        self._messages.append({
                            "role": "tool",
                            "tool_call_id": tc.id,
                            "content": result,
                        })
            else:
                raise ApplicationError(
                    f"Agent exceeded {self.MAX_STEPS_PER_TURN} steps in a single turn",
                    non_retryable=True,
                )

        return "Session ended"
```

**Client usage**

```python
# Start the long-lived Workflow — no initial task string required
handle = await client.start_workflow(
    AgentWorkflow.run,
    AgentInput(system_prompt="You are a helpful assistant."),
    id=f"agent-{user_id}-{session_id}",  # meaningful per-session identifier — also acts as a uniqueness key
    task_queue="ai-agents-prd",
)

# Each user turn is a blocking Update — returns once the agent has replied
reply = await handle.execute_update(AgentWorkflow.send_message, "What is Temporal?")
print(reply)  # "Temporal is a durable execution platform..."

reply = await handle.execute_update(AgentWorkflow.send_message, "How does it handle failures?")
print(reply)

# End the session when the user is done
await handle.signal(AgentWorkflow.end_session)
```

:::note
**Payload size**: message histories passed as Activity arguments grow with every turn. Temporal has a **2 MB payload limit** per argument. In long sessions, trim `self._messages` to a rolling context window (e.g., the last 20 messages) before passing it to `call_llm`. The full history is preserved in the Workflow event record — only the active context window needs to be trimmed.

**Long sessions and continue-as-new**: this Workflow has no hard cap on total turns and no `continue_as_new`. For sessions with hundreds of turns the event history will grow unbounded. Add CAN at a `MAX_TURNS_BEFORE_CONTINUE` threshold — see the [Agent loop with continue-as-new](./patterns.md#agent-loop-with-continue-as-new) pattern for the recipe, including the `is_continue_as_new_suggested()` API.
:::

## LLM retry strategy

The default exponential backoff retry policy is **not appropriate for LLM API calls**. Here's why:

- **HTTP 429 (rate limit)**: The API returns a `Retry-After` header telling you exactly how long to wait. Exponential backoff ignores this and may retry too early (wasting quota) or too late (adding unnecessary latency).
- **HTTP 400 / content policy violation**: This will never succeed on retry. Retrying burns quota and delays the error surface.
- **HTTP 500 / 503 (server error)**: These are genuinely transient and benefit from retried attempts, but with moderate backoff.

Structure your LLM Activity to handle each error class explicitly:

```python
from datetime import timedelta
from temporalio import activity
from temporalio.exceptions import ApplicationError

@activity.defn
async def call_llm(system_prompt: str, messages: list[dict]) -> LLMResponse:
    from openai import AsyncOpenAI, RateLimitError, BadRequestError

    # Disable client-level retries — let Temporal own all retry logic
    client = AsyncOpenAI(max_retries=0)

    try:
        activity.heartbeat("Calling LLM...")
        completion = await client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "system", "content": system_prompt}] + messages,
        )
        return parse_llm_response(completion)

    except RateLimitError as e:
        # Pass the Retry-After value to Temporal as next_retry_delay.
        # Temporal releases the worker slot during the delay — never sleep in an Activity.
        retry_after_secs = int(e.response.headers.get("retry-after", 10))
        raise ApplicationError(
            f"Rate limited — retrying in {retry_after_secs}s",
            type="RateLimitError",
            next_retry_delay=timedelta(seconds=retry_after_secs),
        )

    except BadRequestError as e:
        # Content policy, invalid request — do not retry
        raise ApplicationError(
            str(e),
            type="ContentPolicyError",
            non_retryable=True,
        )
```

Configure the Activity's Temporal retry policy to handle all retries. Rate limit errors surface with the correct `next_retry_delay` (from the `Retry-After` header); server errors use the configured exponential backoff:

```python
from temporalio.common import RetryPolicy

llm_retry_policy = RetryPolicy(
    initial_interval=timedelta(seconds=2),
    backoff_coefficient=2.0,
    maximum_interval=timedelta(minutes=2),
    maximum_attempts=10,
    non_retryable_error_types=["ContentPolicyError"],
)

response = await workflow.execute_activity(
    call_llm,
    args=[system_prompt, messages],
    start_to_close_timeout=timedelta(minutes=5),
    retry_policy=llm_retry_policy,
)
```

## Heartbeat for streaming responses

For streaming LLM responses (e.g., when using `stream=True` as in this [example](https://github.com/dallastexas92/temporal-redis-pubsub)), use `activity.heartbeat()` to prevent the Activity from timing out while the stream is in progress. This also allows the Activity to be cancelled cleanly mid-stream if the Workflow is cancelled.

```python
@activity.defn
async def call_llm_streaming(system_prompt: str, messages: list[dict]) -> str:
    from openai import AsyncOpenAI

    # Disable client-level retries — let Temporal handle retries
    client = AsyncOpenAI(max_retries=0)
    chunks = []

    async with client.chat.completions.stream(
        model="gpt-4o",
        messages=[{"role": "system", "content": system_prompt}] + messages,
    ) as stream:
        async for chunk in stream:
            delta = chunk.choices[0].delta.content or ""
            chunks.append(delta)
            # Batch heartbeats — each heartbeat is a Temporal Action (counts toward cost)
            if len(chunks) % 10 == 0:
                activity.heartbeat(f"Streamed {len(chunks)} chunks")

    return "".join(chunks)
```

Set `heartbeat_timeout` in the Workflow to control how long Temporal waits between heartbeats before declaring the Activity failed:

```python
response = await workflow.execute_activity(
    call_llm_streaming,
    args=[system_prompt, messages],
    start_to_close_timeout=timedelta(minutes=10),
    heartbeat_timeout=timedelta(seconds=30),  # Fail if silent for 30s
)
```

## Human-in-the-loop gate

Temporal supports multiple distinct human-in-the-loop patterns, including both a **one-shot approval gate** (block until a reviewer signals yes/no) and a **multi-turn interactive loop** (block indefinitely waiting for messages from a user). Both use Workflow Signals and Updates and Durable waits. Which you reach for depends on whether a human is actively in a conversation or asynchronously reviewing a proposed action.

### One-shot approval gate

Some agent actions require human approval before execution — for example, sending an email, executing a trade, or deleting records. Implement this as a Signal-based pause with a timeout fallback:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class ApprovalDecision:
    approved: bool
    reviewer: str
    reason: Optional[str] = None

@workflow.defn
class AgentWorkflow:
    def __init__(self):
        self._approval: Optional[ApprovalDecision] = None

    @workflow.signal
    def submit_approval(self, decision: ApprovalDecision) -> None:
        self._approval = decision

    @workflow.run
    async def run(self, input: AgentInput) -> str:
        # ... agent runs, proposes a high-risk action ...

        proposed_action = await workflow.execute_activity(
            plan_action, input.task,
            start_to_close_timeout=timedelta(minutes=2),
        )

        # Notify reviewers (via activity that sends Slack/email)
        await workflow.execute_activity(
            notify_reviewers,
            args=[proposed_action, workflow.info().workflow_id],
            start_to_close_timeout=timedelta(minutes=1),
        )

        # Wait up to 24 hours for a human to signal approval
        self._approval = None
        approved_in_time = await workflow.wait_condition(
            lambda: self._approval is not None,
            timeout=timedelta(hours=24),
        )

        if not approved_in_time or not self._approval.approved:
            reason = self._approval.reason if self._approval else "timed out"
            return f"Action cancelled: {reason}"

        # Execute the approved action
        return await workflow.execute_activity(
            execute_action, proposed_action,
            start_to_close_timeout=timedelta(minutes=5),
        )
```

:::tip
For lower-latency interactive use cases (e.g., a chatbot where the user is actively waiting), use [Workflow Update](https://docs.temporal.io/develop/python/message-passing#updates) instead of a Signal. Updates can return a value synchronously to the caller, making them better suited for request-response patterns.
:::

### Multi-turn chat via Signals

For conversational agents — where a user sends messages, the agent responds, proposes a tool call, waits for the user to confirm, and then continues — the [temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent/blob/main/workflows/agent_goal_workflow.py) reference implementation uses three signals and a central `workflow.wait_condition` loop:

| Signal / Update | Purpose |
|---|---|
| `user_prompt(prompt)` | Signal — user sends a chat message; appended to a queue for ordered processing |
| `confirm()` | Signal — user approves a proposed tool action; sets a boolean the loop checks |
| `respond_to_action(approved, reason)` | **Update** — approve or reject a proposed action; returns a status string synchronously to the caller |
| `end_chat()` | Signal — user ends the session; sets a flag that causes the loop to return |

The Workflow blocks in a `while True` loop waiting for any of these signals, then branches on which flag changed:

```python
from collections import deque
from datetime import timedelta
from typing import Deque, List, Optional
from temporalio import workflow

with workflow.unsafe.imports_passed_through():
    from activities import call_llm, execute_tool

@workflow.defn
class AgentGoalWorkflow:
    def __init__(self) -> None:
        self.prompt_queue: Deque[str] = deque()
        self.confirmed: bool = False
        self.rejected: bool = False
        self.rejection_reason: str = ""
        self.chat_ended: bool = False
        self.conversation_history: List[dict] = []
        self._pending_tool: Optional[str] = None
        self._pending_args: dict = {}

    # ── Signals ──────────────────────────────────────────────────────────────

    @workflow.signal
    async def user_prompt(self, prompt: str) -> None:
        """Receive a user message. Appended to a queue so messages are
        processed in order even if multiple signals arrive back-to-back."""
        if not self.chat_ended:
            self.prompt_queue.append(prompt)

    @workflow.signal
    async def confirm(self) -> None:
        """User approves the proposed tool action."""
        self.confirmed = True

    @workflow.update
    async def respond_to_action(self, approved: bool, reason: str = "") -> str:
        """Approve or reject a proposed tool action.

        Returns a status string synchronously to the caller — prefer this over
        the bare `confirm` Signal when the client needs acknowledgment.
        """
        if approved:
            self.confirmed = True
            return "Action approved"
        self.rejected = True
        self.rejection_reason = reason
        return f"Action rejected: {reason}"

    @workflow.signal
    async def end_chat(self) -> None:
        """Cleanly terminates the agent loop."""
        self.chat_ended = True

    # ── Queries ───────────────────────────────────────────────────────────────

    @workflow.query
    def get_conversation_history(self) -> List[dict]:
        """UI polls this to render the chat thread without modifying state."""
        return self.conversation_history

    # ── Main loop ─────────────────────────────────────────────────────────────

    @workflow.run
    async def run(self, input: AgentInput) -> str:
        waiting_for_confirm = False

        while True:
            # Block until any signal or update arrives — no polling, no busy-waiting.
            # The Workflow sleeps durably; no worker thread is consumed.
            await workflow.wait_condition(
                lambda: bool(self.prompt_queue)
                        or self.chat_ended
                        or self.confirmed
                        or self.rejected
            )

            # Branch 1: end_chat signal received
            if self.chat_ended:
                return str(self.conversation_history)

            # Branch 2: confirm signal/update received — execute the pending tool
            if self.confirmed and waiting_for_confirm and self._pending_tool:
                self.confirmed = False
                waiting_for_confirm = False
                tool_result = await workflow.execute_activity(
                    execute_tool,
                    args=[self._pending_tool, self._pending_args],
                    start_to_close_timeout=timedelta(minutes=2),
                )
                self.conversation_history.append(
                    {"actor": "tool", "response": tool_result}
                )
                self._pending_tool = None
                continue

            # Branch 2b: rejected via respond_to_action Update — cancel pending tool
            if self.rejected and waiting_for_confirm:
                self.rejected = False
                waiting_for_confirm = False
                self.conversation_history.append(
                    {"actor": "system", "response": f"Action rejected: {self.rejection_reason}"}
                )
                self._pending_tool = None
                self.rejection_reason = ""
                continue

            # Branch 3: new user message — call LLM, decide next step
            if self.prompt_queue:
                prompt = self.prompt_queue.popleft()
                self.conversation_history.append({"actor": "user", "response": prompt})

                tool_data = await workflow.execute_activity(
                    call_llm,
                    args=[prompt, self.conversation_history],
                    start_to_close_timeout=timedelta(minutes=5),
                )
                self.conversation_history.append({"actor": "agent", "response": tool_data})

                if tool_data.get("next") == "confirm":
                    # LLM wants to call a tool — surface it to the user for approval
                    self._pending_tool = tool_data["tool"]
                    self._pending_args = tool_data.get("args", {})
                    waiting_for_confirm = True
```

Key points from the [temporal-ai-agent implementation](https://github.com/temporal-community/temporal-ai-agent/blob/main/workflows/agent_goal_workflow.py):

- **Queue over direct execution**: `user_prompt` appends to `prompt_queue` instead of triggering computation immediately, so back-to-back signals are always processed in order.
- **Zero-cost blocking**: `workflow.wait_condition` with no timeout blocks durably — the Workflow persists its state in Temporal without consuming a worker thread while waiting for the next message.
- **Four-way branch**: The loop's single `wait_condition` condition covers all signals and updates; the `if` chain inside determines what happened. This avoids nested `wait_condition` calls that are harder to reason about.
- **Update for proposal responses**: `respond_to_action` is a `@workflow.update` — after the agent proposes a tool call the client can call it to approve or reject and receive a synchronous acknowledgment. The bare `confirm` Signal is retained for simple cases that don't need a return value.
- **Queries never block**: `get_conversation_history` is a `@workflow.query` — the UI can call it at any time to render the current chat thread without interrupting the running loop.
- **Continue-as-new on long conversations**: after `MAX_TURNS_BEFORE_CONTINUE` iterations the workflow calls `continue_as_new`, passing the conversation summary and prompt queue, so event history never grows unbounded.
- **Signal rate limit**: a single Workflow Execution can sustain roughly ≤5 Signals/second. In a real-time chat this limit is never reached. If you are fanning `user_prompt` Signals programmatically (e.g., from a message queue consumer), stay under this threshold or batch messages into a single Signal payload.

## Tool call mapping

Every tool your agent can invoke — web search, database query, API call, file read — should be a separate Temporal Activity registered on the same Worker as the Orchestrator Workflow. This gives you:

- **Retry control per tool**: a flaky search API gets different retry settings than a local database read
- **Timeout isolation**: one slow tool doesn't block others
- **Observability**: each tool invocation is a named event in the Workflow history

```python
# Register all tools as Activities on the Worker
worker = Worker(
    client,
    task_queue="ai-agents-prd",
    workflows=[AgentWorkflow],
    activities=[
        call_llm,
        web_search,          # e.g., Tavily, Perplexity
        query_database,      # internal data sources
        call_external_api,   # third-party service calls
        send_notification,   # Slack, email
    ],
)
```

For further implementation details, see the [AI Patterns](./patterns.md) page.
