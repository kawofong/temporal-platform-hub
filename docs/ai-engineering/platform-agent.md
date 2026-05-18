---
sidebar_position: 5
id: ai-platform-agent
title: AI Platform Agent
sidebar_label: 🏭 Platform Agent
description: Build a single reusable Temporal Workflow that every team can invoke with their own system prompt, context, and tools — so the platform team builds the infrastructure once and product teams never hand-roll retry logic, HITL plumbing, or continue-as-new again.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

Many AI Engineering implementations are scattered, with each team building their own agent Workflows. That works for a single team but doesn't scale: every team ends up hand-rolling the same retry policy, the same human-in-the-loop plumbing, the same `continue_as_new` logic, and the same heartbeating boilerplate. The platform agent pattern centralises all of that once and exposes a single invocation interface — **send in a system prompt, some context, and a list of tools, and get back a durable AI agent.**

At ABC Financial, the platform team owns and deploys one `PlatformAgentWorkflow`. Each product team runs their own Temporal Worker, registers their own tool Activities on it, and starts a Workflow execution with their own prompt, context, and tool list. **The Workflow is the shared piece — Workers are not.** Teams inherit retries, HITL gates, audit history, and long-run safety without writing a single line of Workflow code.

---

## Why a platform agent

| Without a platform agent | With a platform agent |
| :---- | :---- |
| Every team writes their own agent Workflow | One Workflow, maintained by the AI Eng platform team |
| Every team has to figure out guardrails  | Centralised, consistently correct guardrails with hooks for custom needs |
| Retry policy for LLM calls duplicated across teams | Centralised, consistently correct retry strategy |
| `continue_as_new` logic re-implemented or forgotten | Automatic history pruning every N steps |
| No standard audit trail across teams | Every execution has a structured Workflow ID and Search Attributes |
| Prompt + tool changes deployed independently per team | Versioning and governance in one place |

---

## Platform interface design

The Workflow input is the only contract product teams need to understand.

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class ToolDefinition:
    """Describes a tool the LLM may call.

    ``name`` must exactly match the Temporal Activity name registered on the Worker.
    ``task_queue`` must match the task queue that Worker is listening on.
    ``parameters`` is a JSON Schema object passed to the LLM in the tool-calling spec.
    """
    name: str                        # e.g. "finops__cost_lookup"
    description: str                 # shown to the LLM
    parameters: dict[str, Any]       # JSON Schema: {"type": "object", "properties": {...}}
    task_queue: str = "ai-platform"  # task queue for this tool's Worker; override per team

@dataclass
class PlatformAgentInput:
    system_prompt: str               # defines the agent's behaviour and persona
    task: str                        # the user's request
    context: str                     # RAG results, structured data, or any relevant text
    tools: list[ToolDefinition]      # activities the agent may invoke
    max_steps: int = 50              # hard cap before forcing completion
    hitl_required: bool = False      # pause and wait for human approval before each tool execution
```

### Workflow ID convention

Use a structured Workflow ID so executions are queryable by team and purpose:

```
{team}-{task-slug}-{uuid4}
```

Examples:
- `finops-monthly-variance-3f8a1c`
- `risk-trade-analysis-b2e490`
- `compliance-doc-review-9d31fa`

This makes it straightforward to filter Workflow history by team in the Temporal UI or via `tctl` / the SDK client.

---

## The PlatformAgentWorkflow

```python
import asyncio
import uuid
from dataclasses import dataclass, field
from datetime import timedelta
from typing import Any, Optional

from temporalio import workflow
from temporalio.common import RetryPolicy
from temporalio.exceptions import ApplicationError

with workflow.unsafe.imports_passed_through():
    from activities import call_llm_with_tools

# Number of agent steps before calling continue_as_new to prune event history.
# Keep this well below the Temporal history size limit.
STEPS_PER_EXECUTION = 80


@dataclass
class _AgentState:
    """Internal state carried across continue_as_new boundaries."""
    input: PlatformAgentInput
    messages: list[dict] = field(default_factory=list)
    step: int = 0


@workflow.defn
class PlatformAgentWorkflow:

    def __init__(self) -> None:
        self._approval: Optional[bool] = None
        self._approval_reason: Optional[str] = None
        self._reviewer: Optional[str] = None

    # ── Human-in-the-loop signal ──────────────────────────────────────────────

    @workflow.signal
    def submit_approval(self, approved: bool, reviewer: str, reason: str = "") -> None:
        """Sent by a human reviewer via CLI, UI, or webhook handler."""
        self._approval = approved
        self._reviewer = reviewer
        self._approval_reason = reason

    # ── Main loop ─────────────────────────────────────────────────────────────

    @workflow.run
    async def run(self, state: _AgentState) -> str:
        inp = state.input

        # Seed the message history on the first execution only
        if state.step == 0:
            state.messages = [
                {"role": "system", "content": inp.system_prompt},
                {"role": "user", "content": f"Context:\n{inp.context}\n\nTask: {inp.task}"},
            ]

        # Build the LLM tool-calling schema and a name→definition lookup map
        tool_def_map = {td.name: td for td in inp.tools}
        lm_tools = [
            {
                "type": "function",
                "function": {
                    "name": td.name,
                    "description": td.description,
                    "parameters": td.parameters,
                },
            }
            for td in inp.tools
        ]

        while state.step < inp.max_steps:

            # Prune history periodically by continuing as new.
            # The new execution receives the current state and resumes seamlessly.
            if state.step > 0 and state.step % STEPS_PER_EXECUTION == 0:
                workflow.continue_as_new(state)

            # ── Step 1: Ask the LLM what to do next ──────────────────────────
            llm_response = await workflow.execute_activity(
                call_llm_with_tools,
                args=[state.messages, lm_tools],
                start_to_close_timeout=timedelta(minutes=5),
                retry_policy=RetryPolicy(
                    initial_interval=timedelta(seconds=2),
                    backoff_coefficient=2.0,
                    maximum_interval=timedelta(minutes=2),
                    maximum_attempts=10,
                    non_retryable_error_types=["ContentPolicyError"],
                ),
            )

            state.messages.append({
                "role": "assistant",
                "content": llm_response["content"],
                "tool_calls": llm_response.get("tool_calls"),
            })
            state.step += 1

            # ── Step 2: Return if the LLM has a final answer ──────────────────
            if not llm_response.get("tool_calls"):
                return llm_response["content"]

            # ── Step 3: Optional HITL gate before tool execution ──────────────
            if inp.hitl_required:
                self._approval = None
                await workflow.execute_activity(
                    "notify_hitl_reviewers",
                    args=[llm_response["tool_calls"], workflow.info().workflow_id],
                    start_to_close_timeout=timedelta(minutes=1),
                )
                approved_in_time = await workflow.wait_condition(
                    lambda: self._approval is not None,
                    timeout=timedelta(hours=24),
                )
                if not approved_in_time or not self._approval:
                    reason = self._approval_reason or "timed out after 24 hours"
                    return f"Agent stopped: tool execution rejected by {self._reviewer or 'timeout'} — {reason}"

            # ── Step 4: Dispatch tool calls concurrently by Activity name ─────
            # Each tool Activity is dispatched to the task queue declared in its
            # ToolDefinition — so each team can run their own Worker.
            tool_results = await asyncio.gather(*[
                workflow.execute_activity(
                    tc["function"]["name"],
                    args=[tc["function"]["arguments"]],
                    task_queue=tool_def_map[tc["function"]["name"]].task_queue,
                    start_to_close_timeout=timedelta(minutes=2),
                    retry_policy=RetryPolicy(maximum_attempts=3),
                )
                for tc in llm_response["tool_calls"]
            ])

            for tc, result in zip(llm_response["tool_calls"], tool_results):
                state.messages.append({
                    "role": "tool",
                    "tool_call_id": tc["id"],
                    "content": result,
                })

        raise ApplicationError(
            f"Agent exceeded {inp.max_steps} steps without producing a final answer",
            non_retryable=True,
        )
```

The Workflow dispatches tool calls by Activity name via `workflow.execute_activity(tc["function"]["name"], ...)`. The name in each `ToolDefinition` must exactly match the Activity name registered on the Worker — this is the only coupling point between the Workflow and the tools teams provide.

---

## Tool Activity registration

### Naming convention

Use `{team}__{tool_name}` (double underscore) to prevent naming collisions across teams:

| Team | Tool | Activity name |
| :---- | :---- | :---- |
| FinOps | Cost lookup | `finops__cost_lookup` |
| Risk | Trade history | `risk__trade_history` |
| Compliance | Policy search | `compliance__policy_search` |

### Implementing a tool Activity

Each team implements their tools as standard Temporal Activities and registers them with the platform Worker:

```python
# finops/tools.py
from temporalio import activity

@activity.defn(name="finops__cost_lookup")
async def cost_lookup(arguments: dict) -> str:
    """Look up cloud cost data for the given team and time range."""
    team = arguments["team"]
    period = arguments.get("period", "last_30_days")
    # ... query internal cost database ...
    return f"FinOps cost for {team} over {period}: $12,450"


@activity.defn(name="finops__budget_alert")
async def budget_alert(arguments: dict) -> str:
    """Check whether a team has exceeded its budget threshold."""
    team = arguments["team"]
    # ... check budget system ...
    return f"{team} is at 87% of monthly budget"
```

### Worker deployment

The `PlatformAgentWorkflow` is the shared piece — Workers are don't have to be. The platform team deploys a Worker that runs the Workflow and the LLM Activity. Each product team deploys their own Worker that registers their own tool Activities and the main Workflow.

**Platform Worker** — owned by the platform team:

```python
# platform/worker.py
from temporalio.client import Client
from temporalio.worker import Worker
from activities import call_llm_with_tools, notify_hitl_reviewers

async def main():
    client = await Client.connect("your-namespace.tmprl.cloud:7233")
    worker = Worker(
        client,
        task_queue="ai-platform",
        workflows=[PlatformAgentWorkflow],
        activities=[call_llm_with_tools, notify_hitl_reviewers],
    )
    await worker.run()
```

**Team Worker** — owned and deployed by each product team, registering only their own Activities:

```python
# finops/worker.py
from temporalio.client import Client
from temporalio.worker import Worker
from finops.tools import cost_lookup, budget_alert
from platform_agent import PlatformAgentWorkflow  # shared Workflow definition

async def main():
    client = await Client.connect("your-namespace.tmprl.cloud:7233")
    worker = Worker(
        client,
        task_queue="ai-platform-finops",   # team-specific task queue
        # PlatformAgentWorkflow is registered here so this Worker can execute
        # Workflow tasks for agent runs started on this task queue.
        workflows=[PlatformAgentWorkflow],
        activities=[cost_lookup, budget_alert],
    )
    await worker.run()
```

Teams set `task_queue` on each `ToolDefinition` to match the queue their Worker is listening on. The Workflow dispatches each tool call to the right Worker automatically.

---

## Calling the platform agent

Any team starts a `PlatformAgentWorkflow` execution by constructing a `PlatformAgentInput` with their own prompt, context, and tools. No Temporal infrastructure knowledge is required beyond this:

```python
import uuid
from temporalio.client import Client

async def run_finops_agent(task: str, rag_context: str) -> str:
    client = await Client.connect("your-namespace.tmprl.cloud:7233")

    result = await client.execute_workflow(
        PlatformAgentWorkflow.run,
        _AgentState(input=PlatformAgentInput(
            system_prompt=(
                "You are a FinOps assistant at ABC Financial. "
                "Answer questions about cloud costs using only the provided context. "
                "Always cite the data source in your answer."
            ),
            task=task,
            context=rag_context,
            tools=[
                ToolDefinition(
                    name="finops__cost_lookup",
                    description="Look up cloud cost data for a team and time range",
                    parameters={
                        "type": "object",
                        "properties": {
                            "team": {"type": "string", "description": "Team name"},
                            "period": {
                                "type": "string",
                                "enum": ["last_7_days", "last_30_days", "last_quarter"],
                            },
                        },
                        "required": ["team"],
                    },
                    task_queue="ai-platform-finops",
                ),
                ToolDefinition(
                    name="finops__budget_alert",
                    description="Check whether a team has exceeded its budget threshold",
                    parameters={
                        "type": "object",
                        "properties": {
                            "team": {"type": "string"},
                        },
                        "required": ["team"],
                    },
                    task_queue="ai-platform-finops",
                ),
            ],
            max_steps=20,
            hitl_required=False,
        )),
        id=f"finops-{task[:20].replace(' ', '-')}-{uuid.uuid4().hex[:6]}",
        task_queue="ai-platform",
    )
    return result
```

---

## Team isolation and governance

### Per-team task queues

Each team runs their tool Activities on a dedicated task queue, so one team's slow or noisy Activities cannot affect another team's agent:

```
ai-platform          ← PlatformAgentWorkflow + call_llm_with_tools
ai-platform-finops   ← FinOps tool Activities only
ai-platform-risk     ← Risk tool Activities only
```

The Workflow dispatches to the correct queue via the `task_queue` field on each `ToolDefinition` — see [Worker deployment](#worker-deployment) above.

### Namespace isolation

For workloads that process regulated or sensitive data, run the platform agent in a dedicated namespace with its own encryption key. See [Security & Governance](./security.md) for payload encryption setup and namespace configuration.

### Cross-namespace calls with Nexus

When teams operate in separate Temporal Namespaces, standard `workflow.execute_activity` dispatch only reaches Workers in the same Namespace. [Temporal Nexus](https://docs.temporal.io/nexus) bridges this gap: it provides a durable, peer-to-peer RPC layer that routes Operation calls from a caller Namespace to a handler Namespace through a registered Nexus Endpoint — with built-in retries, circuit breaking, and observability.

Two patterns arise in the platform agent context.

#### Invoking the platform agent from a team Namespace

The most common pattern: a team's Workflows live in their own Namespace and invoke the platform agent in the platform Namespace through a Nexus Endpoint, without direct access to the platform Namespace.

**Handler side** — the platform team wraps `PlatformAgentWorkflow` in a Nexus Service:

```python
# platform/nexus_service.py
import uuid
import nexusrpc
import nexusrpc.handler
from temporalio import nexus
from platform_agent import PlatformAgentWorkflow, _AgentState

@nexusrpc.service
class PlatformAgentNexusService:
    run_agent: nexusrpc.Operation[_AgentState, str]

@nexusrpc.handler.service_handler(service=PlatformAgentNexusService)
class PlatformAgentNexusServiceHandler:
    @nexus.workflow_run_operation
    async def run_agent(
        self,
        ctx: nexus.WorkflowRunOperationContext,
        state: _AgentState,
    ) -> nexus.WorkflowHandle[str]:
        return await ctx.start_workflow(
            PlatformAgentWorkflow.run,
            state,
            id=f"agent-{uuid.uuid4().hex[:8]}",
        )
```

Register the Nexus handler on the platform Worker alongside the Workflow:

```python
worker = Worker(
    client,
    task_queue="ai-platform",
    workflows=[PlatformAgentWorkflow],
    activities=[call_llm_with_tools, notify_hitl_reviewers],
    nexus_service_handlers=[PlatformAgentNexusServiceHandler()],
)
```

**Caller side** — a team Workflow in a different Namespace calls the platform agent via the Endpoint. The team only imports the shared service contract, not any platform internals:

```python
# finops/workflows.py — runs in the finops namespace
from datetime import timedelta
from temporalio import workflow
from platform_agent.nexus_service import (
    PlatformAgentNexusService,
    _AgentState, PlatformAgentInput, ToolDefinition,
)

PLATFORM_NEXUS_ENDPOINT = "ai-platform"

@workflow.defn
class FinOpsReportWorkflow:
    @workflow.run
    async def run(self, task: str, context: str) -> str:
        nexus_client = workflow.create_nexus_client(
            service=PlatformAgentNexusService,
            endpoint=PLATFORM_NEXUS_ENDPOINT,
        )
        return await nexus_client.execute_operation(
            PlatformAgentNexusService.run_agent,
            _AgentState(input=PlatformAgentInput(
                system_prompt="You are a FinOps assistant at ABC Financial...",
                task=task,
                context=context,
                tools=[
                    ToolDefinition(
                        name="finops__cost_lookup",
                        description="Look up cloud cost data for a team and time range",
                        parameters={"type": "object", "properties": {"team": {"type": "string"}}, "required": ["team"]},
                        task_queue="ai-platform-finops",
                    ),
                ],
            )),
            schedule_to_close_timeout=timedelta(hours=1),
        )
```

Create the Nexus Endpoint once to wire the caller Namespace to the platform Namespace and task queue:

```bash
temporal operator nexus endpoint create \
  --name ai-platform \
  --target-namespace ai-platform-namespace \
  --target-task-queue ai-platform
```

#### Team tools as Nexus Operations

Conversely, if the `PlatformAgentWorkflow` runs in the platform Namespace but a team's tool Activities run in a separate Namespace, the team wraps their tools as Nexus Operations. The platform Workflow calls them via `workflow.create_nexus_client()` using a shared service contract that the team publishes alongside their Nexus handler. This follows the same handler/caller pattern above — the team is the handler, the platform agent is the caller.

See the [Nexus Python feature guide](https://docs.temporal.io/develop/python/nexus/feature-guide) for complete handler and caller implementation examples.

### Search Attributes for audit queries

Add Search Attributes to every `PlatformAgentWorkflow` execution so you can filter the Temporal UI and run audit queries across teams:

```python
from temporalio.common import SearchAttributeKey, TypedSearchAttributes

TEAM_ATTR = SearchAttributeKey.for_keyword("Team")
MODEL_ATTR = SearchAttributeKey.for_keyword("AIModel")
CLASSIFICATION_ATTR = SearchAttributeKey.for_keyword("DataClassification")

result = await client.execute_workflow(
    PlatformAgentWorkflow.run,
    ...,
    search_attributes=TypedSearchAttributes([
        (TEAM_ATTR, "finops"),
        (MODEL_ATTR, "gpt-4o"),
        (CLASSIFICATION_ATTR, "internal"),
    ]),
)
```

Query running or completed executions by team:

```bash
tctl workflow list --query 'Team="finops" AND ExecutionStatus="Running"'
```

This provides a full, queryable audit trail of which team ran which agent, with which model, against which data classification — all backed by immutable Workflow history.

---

## Additional resources

- [temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent) — Reference implementation the platform agent pattern is derived from
- [Reference Architecture](./reference-architecture.md) — Deep dive on the orchestrator Workflow, LLM retry strategy, and HITL gate
- [AI Patterns](./patterns.md) — Human-in-the-loop approval, continue-as-new, and parallel tool dispatch patterns referenced above
- [Security & Governance](./security.md) — Payload encryption, namespace isolation, and credential management for AI workloads
