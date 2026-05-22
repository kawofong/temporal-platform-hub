---
sidebar_position: 3
id: ai-patterns
title: AI Engineering Patterns
sidebar_label: 🧩 AI Patterns
description: Five production-tested Temporal patterns for LLM Activities, human-in-the-loop approval, prompt versioning, parallel tool dispatch, and long-running agent loops.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

These patterns complement the [core Temporal Patterns](../patterns.md) page and are specific to AI engineering workloads. Each pattern follows the same structure: what it does, why to use it, and a Python code sample.

For additional examples, see the [Temporal AI Cookbook](https://docs.temporal.io/ai-cookbook).

---

## LLM Activity with structured retries

|  |  |
| :---- | :---- |
| **What it does** | Wraps an LLM API call in a Temporal Activity with error-class-aware retry behavior: `next_retry_delay` for rate limits (respecting `Retry-After`), non-retryable errors for content policy violations, and `max_retries=0` on the LLM client so Temporal owns all retry logic. |
| **Why use it** | Default exponential backoff is wrong for LLM APIs. Rate limit errors (HTTP 429) include a `Retry-After` header that tells you exactly how long to wait. Content policy errors (HTTP 400) will never succeed on retry — retrying them wastes quota and delays surfacing the failure. Wrapping the LLM call in an Activity makes each call durable and observable in the Workflow history. |

```python
from datetime import timedelta
from temporalio import activity, workflow
from temporalio.common import RetryPolicy
from temporalio.exceptions import ApplicationError

@activity.defn
async def call_llm(system_prompt: str, messages: list[dict]) -> str:
    from openai import AsyncOpenAI, RateLimitError, BadRequestError

    # Disable client-level retries — let Temporal own all retry logic
    client = AsyncOpenAI(max_retries=0)

    try:
        activity.heartbeat("Calling LLM...")
        completion = await client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "system", "content": system_prompt}] + messages,
        )
        return completion.choices[0].message.content

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
        # Content policy / invalid request — retrying won't help
        raise ApplicationError(
            str(e), type="ContentPolicyError", non_retryable=True
        )


# In your Workflow — all retries (rate limits and server errors) are handled by Temporal
result = await workflow.execute_activity(
    call_llm,
    args=[system_prompt, messages],
    start_to_close_timeout=timedelta(minutes=5),
    retry_policy=RetryPolicy(
        initial_interval=timedelta(seconds=2),
        backoff_coefficient=2.0,
        maximum_interval=timedelta(minutes=2),
        maximum_attempts=10,
        non_retryable_error_types=["ContentPolicyError"],
    ),
)
```

---

## Human-in-the-loop approval gate

|  |  |
| :---- | :---- |
| **What it does** | Pauses a Workflow at a decision point and waits for a human to send a Signal (or Update) with an approval or rejection decision. Falls back to a configurable default action if no response arrives within the timeout window. |
| **Why use it** | AI agents that take real-world actions (sending emails, executing trades, modifying records) need human oversight before high-risk steps. Temporal's Signal/Update primitives implement this natively, without polling tables, webhook endpoints, or custom state machines. The immutable Workflow history records who approved what, and when — satisfying audit requirements. |

```python
from dataclasses import dataclass
from datetime import timedelta
from typing import Optional
from temporalio import workflow
from temporalio.exceptions import ApplicationError

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
        """Called externally by a reviewer via CLI, UI, or webhook handler."""
        self._approval = decision

    @workflow.run
    async def run(self, task: str) -> str:
        proposed_action = await workflow.execute_activity(
            plan_action,
            task,
            start_to_close_timeout=timedelta(minutes=2),
        )

        # Notify reviewers with the Workflow ID so they can send the signal back
        await workflow.execute_activity(
            notify_reviewers,
            args=[proposed_action, workflow.info().workflow_id],
            start_to_close_timeout=timedelta(minutes=1),
        )

        # Wait up to 24 hours; returns False if the timeout expires first
        approved_in_time = await workflow.wait_condition(
            lambda: self._approval is not None,
            timeout=timedelta(hours=24),
        )

        if not approved_in_time:
            return "Action cancelled: approval request timed out after 24 hours"

        if not self._approval.approved:
            return f"Action rejected by {self._approval.reviewer}: {self._approval.reason}"

        return await workflow.execute_activity(
            execute_action,
            proposed_action,
            start_to_close_timeout=timedelta(minutes=5),
        )
```

:::tip
Use [Workflow Update](https://docs.temporal.io/develop/python/message-passing#updates) instead of Signal when the reviewer is actively waiting for a response (e.g., a chatbot UI). Updates return a value synchronously to the caller; Signals are fire-and-forget.
:::

---

## Prompt versioning and replay safety

|  |  |
| :---- | :---- |
| **What it does** | Keeps prompt templates inside Activity functions (never in Workflow functions), ensuring that Workflow replay always reproduces the same result from history rather than re-calling the LLM with a potentially changed prompt. |
| **Why use it** | Temporal replays Workflow code from history during failure recovery. Any non-deterministic call inside a Workflow — including an LLM call — will produce a different result on replay and cause a non-determinism error. Placing prompts and LLM calls in Activities means the result is recorded once in history and replayed from there. This also makes prompt changes safe: deploying a new prompt version in an Activity does not affect in-flight Workflows — only new executions pick up the change. |

```python
from datetime import timedelta
from temporalio import activity, workflow

# ❌ Wrong: LLM call directly in the Workflow breaks determinism and replay
@workflow.defn
class BadAgentWorkflow:
    @workflow.run
    async def run(self, task: str) -> str:
        import openai
        # This is non-deterministic I/O inside a Workflow — never do this
        response = openai.chat.completions.create(  # BUG
            model="gpt-4o",
            messages=[{"role": "user", "content": task}],
        )
        return response.choices[0].message.content


# ✅ Correct: LLM call and prompt live in an Activity
SYSTEM_PROMPT_V2 = """
You are a financial analysis assistant at ABC Financial.
Answer questions based only on the provided context.
If you are unsure, say so — do not speculate.
"""

@activity.defn
async def call_llm_with_prompt(task: str, context: str) -> str:
    import openai
    # Disable client-level retries — let Temporal own all retry logic
    client = openai.AsyncOpenAI(max_retries=0)
    response = await client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT_V2},
            {"role": "user", "content": f"Context:\n{context}\n\nTask: {task}"},
        ],
    )
    return response.choices[0].message.content


# In a real project, activities live in a separate file (e.g., activities.py).
# Import them with imports_passed_through to avoid sandbox reloads on every Workflow task:
with workflow.unsafe.imports_passed_through():
    from activities import retrieve_context, call_llm_with_prompt

@workflow.defn
class GoodAgentWorkflow:
    @workflow.run
    async def run(self, task: str) -> str:
        context = await workflow.execute_activity(
            retrieve_context, task,
            start_to_close_timeout=timedelta(minutes=1),
        )
        # The prompt lives in the Activity — safe for replay, safe to version
        return await workflow.execute_activity(
            call_llm_with_prompt,
            args=[task, context],
            start_to_close_timeout=timedelta(minutes=5),
        )
```

---

## Parallel tool dispatch

|  |  |
| :---- | :---- |
| **What it does** | Fans out multiple independent tool calls from the agent loop in a single Workflow step using `asyncio.gather`, then collects all results before proceeding. |
| **Why use it** | LLMs frequently return multiple tool calls in a single response (e.g., search three sources simultaneously). Dispatching them sequentially adds the latency of each tool call in series. Parallel dispatch with `asyncio.gather` runs them concurrently and completes in the time of the slowest single call, not the sum of all calls. Each tool is still a durable Activity — if any single tool fails, only that Activity is retried, not the others. |

```python
import asyncio
from datetime import timedelta
from temporalio import workflow

with workflow.unsafe.imports_passed_through():
    from activities import call_llm, execute_tool

@workflow.defn
class AgentWorkflow:
    @workflow.run
    async def run(self, task: str) -> str:
        messages = [{"role": "user", "content": task}]

        # Step 1: Ask the LLM what tools to call
        llm_response = await workflow.execute_activity(
            call_llm,
            args=[SYSTEM_PROMPT, messages],
            start_to_close_timeout=timedelta(minutes=5),
        )

        # Step 2: Dispatch all tool calls concurrently
        # asyncio.gather schedules all Activities at once and waits for all to finish
        tool_results = await asyncio.gather(*[
            workflow.execute_activity(
                execute_tool,
                args=[tool_call.name, tool_call.arguments],
                start_to_close_timeout=timedelta(minutes=2),
            )
            for tool_call in llm_response.tool_calls
        ])

        # Step 3: Add all tool results to the message history
        messages.append({"role": "assistant", "content": llm_response.content,
                         "tool_calls": llm_response.tool_calls})
        for tool_call, result in zip(llm_response.tool_calls, tool_results):
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result,
            })

        # Step 4: Synthesize the final answer
        return await workflow.execute_activity(
            call_llm,
            args=[SYSTEM_PROMPT, messages],
            start_to_close_timeout=timedelta(minutes=5),
        )
```

:::note
`asyncio.gather` is safe inside Temporal Workflows. The Temporal Python SDK is built on `asyncio` and handles concurrent Activity dispatch correctly. Each call to `workflow.execute_activity` returns a coroutine that Temporal schedules as a separate Activity Task.

To handle partial failures gracefully (so one failing tool doesn't cancel the others), use `asyncio.gather(*tasks, return_exceptions=True)` and filter exceptions from the results before building the next LLM message.

**Tool Activity idempotency**: Temporal Activities have at-least-once execution semantics — an Activity that fails mid-execution may run again on retry. Make write-side tool Activities idempotent: use an idempotency key (e.g., the Workflow ID + step counter) for operations like sending emails, executing trades, or modifying records. Read-side tools (search, lookup) are naturally idempotent.
:::

---

## Agent loop with continue-as-new

|  |  |
| :---- | :---- |
| **What it does** | Runs an agent loop across many steps by atomically completing the current Workflow execution and starting a new one — carrying over the agent's state — when the event history approaches the size limit. |
| **Why use it** | Temporal Workflow histories have a [size limit](https://docs.temporal.io/cloud/limits#workflow-execution-event-history-limits). An agent that runs for hundreds of steps accumulates a large history (each Activity input and output is an event). Without `continue_as_new`, the Workflow will eventually hit the limit and fail. `continue_as_new` prunes the history while preserving the agent's logical state (messages, step count), enabling agents that run for days or thousands of steps. |

```python
import asyncio
from dataclasses import dataclass, field
from datetime import timedelta
from temporalio import workflow
from temporalio.exceptions import ApplicationError

with workflow.unsafe.imports_passed_through():
    from activities import call_llm, execute_tool

@dataclass
class AgentState:
    task: str
    messages: list[dict] = field(default_factory=list)
    step: int = 0
    final_answer: str | None = None

# Hard-coded step guard — belt-and-suspenders alongside the SDK-provided signal below.
# Keep this well below the Temporal history limit.
STEPS_PER_EXECUTION = 100

@workflow.defn
class LongRunningAgentWorkflow:
    @workflow.run
    async def run(self, state: AgentState) -> str | None:
        while True:
            # Prefer the SDK signal over the step counter: is_continue_as_new_suggested()
            # returns True when event history approaches the server-side warn threshold
            # (~10k events / 10 MB). The step counter fires first on shorter agents and
            # acts as a belt-and-suspenders guard if the SDK signal hasn't fired yet.
            if workflow.info().is_continue_as_new_suggested or (
                state.step > 0 and state.step % STEPS_PER_EXECUTION == 0
            ):
                # Pass the current agent state to the next execution.
                # The new execution starts fresh history but picks up from state.step.
                workflow.continue_as_new(state)

            llm_response = await workflow.execute_activity(
                call_llm,
                args=[SYSTEM_PROMPT, state.messages],
                start_to_close_timeout=timedelta(minutes=5),
            )

            state.messages.append({"role": "assistant", "content": llm_response.content})
            state.step += 1

            if llm_response.is_final:
                return llm_response.content

            if llm_response.tool_calls:
                tool_results = await asyncio.gather(*[
                    workflow.execute_activity(
                        execute_tool,
                        args=[tc.name, tc.arguments],
                        start_to_close_timeout=timedelta(minutes=2),
                    )
                    for tc in llm_response.tool_calls
                ])
                for tc, result in zip(llm_response.tool_calls, tool_results):
                    state.messages.append({
                        "role": "tool",
                        "tool_call_id": tc.id,
                        "content": result,
                    })
```

:::tip
When passing `state.messages` across `continue_as_new` calls, trim the message history to a fixed window (e.g., last 20 messages) to keep the Workflow input size manageable. The full unabridged history remains in the earlier Workflow execution records if needed for audit purposes.

The same trimming applies **within** a single execution: `state.messages` is passed as an Activity argument on every `call_llm` invocation. Temporal has a **2 MB payload limit** per argument — a long conversation can approach this limit before `continue_as_new` fires. Trim the active context window before each LLM call; the full history is always recoverable from prior execution records.
:::

---

## Additional resources

- [Temporal AI Cookbook](https://docs.temporal.io/ai-cookbook) — Step-by-step solutions covering the patterns above with runnable code
- [temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent) — Full reference implementation combining orchestrator Workflow, tool Activities, and HITL approval
- [Human-in-the-Loop AI Agent](https://docs.temporal.io/ai-cookbook/human-in-the-loop) — AI Cookbook recipe for HITL patterns
- [Retry Policy from HTTP Responses](https://docs.temporal.io/ai-cookbook/http-retry-enhancement-python) — AI Cookbook recipe for reading `Retry-After` headers
