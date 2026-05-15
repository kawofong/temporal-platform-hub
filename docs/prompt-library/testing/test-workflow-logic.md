---
id: test-workflow-logic
title: Test Workflow Logic
sidebar_label: 🧩 Test Workflow Logic
description: Use the Temporal test environment to assert workflow state transitions and outcomes.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Write workflow-level tests for the following Temporal workflow in [Python/Go/TypeScript/Java].

[Paste your workflow code]

Testing framework: [pytest / JUnit / Jest / Go testing]

Tests to write:
1. Happy path — workflow completes successfully with expected outputs
2. Signal handling — send a signal mid-execution and assert the workflow responds correctly
3. Query responses — assert queries return accurate state at key points in execution
4. Cancellation — cancel the workflow and assert cleanup behavior
5. [Add any domain-specific scenarios]

Use the Temporal SDK's test environment (TestWorkflowEnvironment / WorkflowTestSuite) and 
mock all activity implementations.
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- The test environment uses simulated time — call `testEnv.sleep()` instead of real sleeps in tests.
- Mock activities return immediately by default, which is usually what you want for logic tests.

## Related prompts

- [Unit Test Activities](./unit-test-activities.md)
- [Simulate Failures](./simulate-failures.md)
