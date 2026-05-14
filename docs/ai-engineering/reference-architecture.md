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

The Workflow is the durable brain of your agent. It holds the agent's message history, step count, and any pending decisions. It does not call the LLM or any external service directly — it schedules Activities to do that work and waits for their results.

```python
from dataclasses import dataclass, field
from datetime import timedelta
from typing import Optional
from temporalio import workflow
from temporalio.exceptions import ApplicationError

@dataclass
class AgentInput:
    task: str
    system_prompt: str

@dataclass
class AgentState:
    task: str
    system_prompt: str
    messages: list[dict] = field(default_factory=list)
    step: int = 0

@workflow.defn
class AgentWorkflow:
    MAX_STEPS = 50  # Hard cap before forcing completion

    @workflow.run
    async def run(self, input: AgentInput) -> str:
        state = AgentState(task=input.task, system_prompt=input.system_prompt)
        state.messages.append({"role": "user", "content": input.task})

        while state.step < self.MAX_STEPS:
            # Step 1: Ask the LLM what to do next
            response = await workflow.execute_activity(
                call_llm,
                args=[state.system_prompt, state.messages],
                schedule_to_close_timeout=timedelta(minutes=5),
            )

            state.messages.append({"role": "assistant", "content": response.content})
            state.step += 1

            # Step 2: If the LLM is done, return the final answer
            if response.is_final:
                return response.content

            # Step 3: Execute the tool calls the LLM requested
            if response.tool_calls:
                import asyncio
                tool_results = await asyncio.gather(*[
                    workflow.execute_activity(
                        execute_tool,
                        args=[tc.name, tc.arguments],
                        schedule_to_close_timeout=timedelta(minutes=2),
                    )
                    for tc in response.tool_calls
                ])
                for tc, result in zip(response.tool_calls, tool_results):
                    state.messages.append({
                        "role": "tool",
                        "tool_call_id": tc.id,
                        "content": result,
                    })

        raise ApplicationError(
            f"Agent exceeded {self.MAX_STEPS} steps without completing",
            non_retryable=True,
        )
```

## LLM retry strategy

The default exponential backoff retry policy is **not appropriate for LLM API calls**. Here's why:

- **HTTP 429 (rate limit)**: The API returns a `Retry-After` header telling you exactly how long to wait. Exponential backoff ignores this and may retry too early (wasting quota) or too late (adding unnecessary latency).
- **HTTP 400 / content policy violation**: This will never succeed on retry. Retrying burns quota and delays the error surface.
- **HTTP 500 / 503 (server error)**: These are genuinely transient and benefit from retried attempts, but with moderate backoff.

Structure your LLM Activity to handle each error class explicitly:

```python
import asyncio
from temporalio import activity
from temporalio.exceptions import ApplicationError

@activity.defn
async def call_llm(system_prompt: str, messages: list[dict]) -> LLMResponse:
    from openai import AsyncOpenAI, RateLimitError, BadRequestError

    client = AsyncOpenAI()

    # Inner retry loop for rate limits — handles Retry-After header directly
    for attempt in range(5):
        try:
            activity.heartbeat(f"LLM call attempt {attempt + 1}")
            completion = await client.chat.completions.create(
                model="gpt-4o",
                messages=[{"role": "system", "content": system_prompt}] + messages,
            )
            return parse_llm_response(completion)

        except RateLimitError as e:
            # Respect the Retry-After header if present
            retry_after = int(e.response.headers.get("retry-after", 10))
            activity.heartbeat(f"Rate limited — waiting {retry_after}s")
            await asyncio.sleep(retry_after)
            continue

        except BadRequestError as e:
            # Content policy, invalid request — do not retry
            raise ApplicationError(
                str(e),
                type="ContentPolicyError",
                non_retryable=True,
            )

    raise ApplicationError("LLM rate limit retries exhausted", non_retryable=False)
```

Configure the Activity's Temporal retry policy to cover transient server errors (500/503), while relying on the in-Activity loop for rate limits:

```python
from temporalio.common import RetryPolicy

llm_retry_policy = RetryPolicy(
    initial_interval=timedelta(seconds=2),
    backoff_coefficient=2.0,
    maximum_interval=timedelta(minutes=1),
    maximum_attempts=3,
    non_retryable_error_types=["ContentPolicyError"],
)

response = await workflow.execute_activity(
    call_llm,
    args=[system_prompt, messages],
    schedule_to_close_timeout=timedelta(minutes=10),
    retry_policy=llm_retry_policy,
)
```

## Heartbeat for streaming responses

For streaming LLM responses (e.g., when using `stream=True`), use `activity.heartbeat()` to prevent the Activity from timing out while the stream is in progress. This also allows the Activity to be cancelled cleanly mid-stream if the Workflow is cancelled.

```python
@activity.defn
async def call_llm_streaming(system_prompt: str, messages: list[dict]) -> str:
    from openai import AsyncOpenAI

    client = AsyncOpenAI()
    chunks = []

    async with client.chat.completions.stream(
        model="gpt-4o",
        messages=[{"role": "system", "content": system_prompt}] + messages,
    ) as stream:
        async for chunk in stream:
            delta = chunk.choices[0].delta.content or ""
            chunks.append(delta)
            # Heartbeat every chunk so Temporal knows we're alive
            activity.heartbeat(f"Streamed {len(chunks)} chunks")

    return "".join(chunks)
```

Set `heartbeat_timeout` in the Workflow to control how long Temporal waits between heartbeats before declaring the Activity failed:

```python
response = await workflow.execute_activity(
    call_llm_streaming,
    args=[system_prompt, messages],
    schedule_to_close_timeout=timedelta(minutes=10),
    heartbeat_timeout=timedelta(seconds=30),  # Fail if no heartbeat for 30s
)
```

## Human-in-the-loop gate

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
            schedule_to_close_timeout=timedelta(minutes=2),
        )

        # Notify reviewers (via activity that sends Slack/email)
        await workflow.execute_activity(
            notify_reviewers,
            args=[proposed_action, workflow.info().workflow_id],
            schedule_to_close_timeout=timedelta(minutes=1),
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
            schedule_to_close_timeout=timedelta(minutes=5),
        )
```

:::tip
For lower-latency interactive use cases (e.g., a chatbot where the user is actively waiting), use [Workflow Update](https://docs.temporal.io/develop/python/message-passing#updates) instead of a Signal. Updates can return a value synchronously to the caller, making them better suited for request-response patterns.
:::

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
