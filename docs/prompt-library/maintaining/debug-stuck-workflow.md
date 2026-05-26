---
id: debug-stuck-workflow
title: Debug a Stuck Workflow
sidebar_label: 🔍 Debug a Stuck Workflow
description: Diagnose why a workflow is blocked and generate a plan to unblock it safely.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
/temporal-developer A Temporal workflow is stuck and not making progress. Help me diagnose and resolve it. See the attached Workflow Event History in JSON. 

Please:
1. Identify the most likely root causes given the history
2. Provide a step-by-step diagnosis checklist
3. Suggest remediation options ranked by risk (prefer non-destructive first)
4. Warn about any risks with each option
```

## Example output

```
# Diagnosis: `TRANSFER-IWC-855`

## What the history actually shows

Workflow type **`AccountTransferWorkflowRecoverableFailure`** on task queue **`MoneyTransfer`**, Java SDK worker `92526@Kas-MacBook-Pro.local`. It executed cleanly through event 23 — `Validate` activity :white_check_mark:, 1s timer :white_check_mark:, `Withdraw` activity :white_check_mark:, 3s timer :white_check_mark: — then on event 26 the workflow task failed during replay:Failure handling event 25 of type 'EVENT_TYPE_WORKFLOW_TASK_STARTED' during execution.
{WorkflowTaskStartedEventId=25, CurrentStartedEventId=25}
cause: WorkflowTask: failure executing SCHEDULED->WORKFLOW_TASK_STARTED,
       transition history is [CREATED->WORKFLOW_TASK_SCHEDULED]
applicationFailureInfo.type: io.temporal.internal.statemachines.InternalWorkflowTaskException

The stack is entirely inside `io.temporal.internal.statemachines.WorkflowStateMachines` / `ReplayWorkflowRunTaskHandler` — no user code on the stack. That is the signature of a **non-determinism / replay mismatch** (Temporal rule **TMPRL1100**). The state machine drove the history up to "schedule activity / timer" but couldn't reconcile what the workflow code did next with what's recorded in history.

## 1. Most likely root causes (ranked)

The workflow type name `AccountTransferWorkflowRecoverableFailure` is itself a strong hint — this looks like the standard Temporal "money transfer" demo's deliberately-broken variant designed to demonstrate recoverable failures. So in priority order:

1. **Non-determinism from a code change after event 23.** The workflow committed Validate → sleep(1s) → Withdraw → sleep(3s). When it woke from the second timer, the next command it produced (most likely `Deposit`) didn't match what replay expected, *or* the code has since been modified between attempts. This is by far the most common cause given the stack trace.
2. **A workflow-code bug thrown on the path between event 23 and the next command.** Could be an unhandled exception in workflow logic (e.g., a deliberate `throw new RuntimeException(...)` in the demo to simulate a "recoverable failure" before `Deposit`). Workflow task failures from user exceptions show up identically in history but normally include user frames in the stack — these don't, which pushes this below #1.
3. **Worker has stale/wrong workflow code registered.** A different worker build was deployed that no longer matches the recorded history.
4. **Sandbox / SDK internal bug.** Lowest probability — the stack is purely internal, but the SDK's own state machine throwing is usually still triggered by a user-side mismatch.

Workflow task failures don't fail the workflow — Temporal will keep retrying the workflow task with exponential backoff forever. That's *why it appears stuck*: it's in RUNNING status, looping on the same failing task, making no forward progress.

## 2. Step-by-step diagnosis checklist

Run these in order; stop when you have the answer.

1. **Confirm current status.**
   `temporal workflow describe --workflow-id TRANSFER-IWC-855`
   Expect: status RUNNING, pending workflow task with attempt count > 1 and climbing.

2. **Check worker logs around `15:35:22Z`** on host `Kas-MacBook-Pro.local`. The SDK logs the full non-determinism diff to the worker — that diff is the single most valuable artifact. Look for `NonDeterministicException`, `Command type X doesn't match event type Y`, or your own application stack frames.

3. **Inspect the workflow code.** Open `AccountTransferWorkflowRecoverableFailure` and look at what runs after the second sleep (`workflow.sleep(Duration.ofSeconds(3))` between Withdraw and Deposit). Check whether:
   - the next activity call's name, args, or order changed,
   - a new activity / timer / signal was inserted before it,
   - `Workflow.newRandom()` / `Workflow.currentTimeMillis()` is being used (vs. `java.util.Random` / `System.currentTimeMillis()` — those are non-deterministic),
   - any `if` branch reads from a non-deterministic source.

4. **Check git history / deployment for changes after `2026-05-21T15:35:18Z`** (when event 23's timer was started). Anything merged or hot-swapped on the worker between then and now is the prime suspect.

5. **Run a replay test locally** against this history JSON. In Java, `WorkflowReplayer.replayWorkflowExecution(historyJson, AccountTransferWorkflowRecoverableFailure.class)`. This reproduces the exact failure offline with full stack into user code, without touching production.

6. **Verify worker is on the right task queue and registers the right workflow class.** `MoneyTransfer` queue, workflow type `AccountTransferWorkflowRecoverableFailure`. A typo here would prevent task pickup entirely — not your symptom, but worth a 5-second check.
```

## Tips

- Paste the actual event history if you can — it dramatically improves diagnosis quality.
- Always try `temporal workflow describe` and check pending activities before escalating.

