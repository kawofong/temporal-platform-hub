---
sidebar_position: 12
id: faqs
title: Frequently Asked Questions
sidebar_label: ❓ FAQs
description: Common questions and answers about using Temporal at your organization.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](./intro.md).
:::

:::note
Capture common questions from developers to reduce repeated support requests.
:::

## When should I use Temporal?

There are many reasons why you should use Temporal. Use the [Temporal Decision Framework](./decision-framework.md) to help you decide.

---

## AI Engineering

### Can I call OpenAI (or any LLM API) from inside a Workflow?

No. Workflow code must be deterministic — it is replayed from history whenever a Worker recovers from a failure. Calling an LLM inside a Workflow would re-execute the API call on replay, producing a different result and causing a non-determinism error.

Always wrap LLM calls in an **Activity**. The Activity result is recorded once in Workflow history and replayed from there on subsequent replays. The Workflow code itself sees the same result every time.

```python
# ❌ Never do this
@workflow.defn
class MyWorkflow:
    @workflow.run
    async def run(self, task: str) -> str:
        return openai.chat.completions.create(...)  # non-deterministic I/O in a Workflow

# ✅ Do this instead
@activity.defn
async def call_llm(task: str) -> str:
    return openai.chat.completions.create(...)  # I/O belongs in an Activity
```

See [Prompt versioning and replay safety](./ai-engineering/patterns.md#prompt-versioning-and-replay-safety) for a full example.

### How do I handle streaming LLM responses?

Use `activity.heartbeat()` inside the Activity to keep Temporal informed that the Activity is still alive while the stream is in progress. Set a `heartbeat_timeout` on the Activity schedule to control how long Temporal waits between heartbeats before declaring the Activity failed:

```python
@activity.defn
async def call_llm_streaming(messages: list[dict]) -> str:
    chunks = []
    async with openai_client.chat.completions.stream(...) as stream:
        async for chunk in stream:
            chunks.append(chunk.choices[0].delta.content or "")
            activity.heartbeat(f"Streamed {len(chunks)} chunks")  # keep-alive
    return "".join(chunks)

# In your Workflow:
result = await workflow.execute_activity(
    call_llm_streaming,
    messages,
    schedule_to_close_timeout=timedelta(minutes=10),
    heartbeat_timeout=timedelta(seconds=30),  # fail if silent for 30s
)
```

### What happens if my agent runs for days or hundreds of steps?

Temporal Workflow histories have a [size limit](https://docs.temporal.io/cloud/limits#workflow-execution-event-history-limits). An agent that accumulates hundreds of Activity results will eventually approach this limit. Use `continue_as_new` to atomically complete the current Workflow execution and start a new one, carrying the agent's state (messages, step count) into the next execution:

```python
# When the history is getting large, prune it and continue
if state.step % 100 == 0 and state.step > 0:
    workflow.continue_as_new(state)  # new execution, same agent state
```

See [Agent loop with continue-as-new](./ai-engineering/patterns.md#agent-loop-with-continue-as-new) for the full pattern.

### Where should my prompts live — in the Workflow or in an Activity?

Always in the **Activity**. Prompts are part of the non-deterministic I/O that defines a side effect (the LLM call), so they belong alongside the LLM call in Activity code. This has two benefits:

1. **Replay safety**: the Workflow records the Activity *result* in history, not the prompt. Replaying the Workflow never re-calls the LLM.
2. **Safe versioning**: you can deploy a new prompt in an Activity without affecting in-flight Workflows. In-flight executions continue to replay from their recorded history; only new executions pick up the updated prompt.

### How do I pass API keys to LLM Activities?

Fetch credentials from a secrets manager *inside the Activity function*, not via Workflow arguments. Workflow and Activity arguments are serialized as Temporal payloads and stored in Workflow history — even with payload encryption, minimize the credential material flowing through Temporal's data plane.

```python
@activity.defn
async def call_llm(messages: list[dict]) -> str:
    # Fetch a fresh key from AWS Secrets Manager on each execution
    secret = boto3.client("secretsmanager").get_secret_value(
        SecretId="prod/ai/openai-api-key"
    )
    api_key = json.loads(secret["SecretString"])["api_key"]
    client = AsyncOpenAI(api_key=api_key)
    ...
```

See [Credential management for AI provider APIs](./ai-engineering/security.md#credential-management-for-ai-provider-apis) for the full guidance.
