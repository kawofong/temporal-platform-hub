---
id: simulate-failures
title: Simulate Failures
sidebar_label: 💥 Simulate Failures
description: Inject activity failures and timeouts to verify retry and compensation behavior.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Write failure injection tests for the following Temporal workflow in [Python/Go/TypeScript/Java].

[Paste your workflow code]

Failure scenarios to test:
1. Activity fails with a retryable error N times, then succeeds — assert workflow still completes
2. Activity exceeds max retries — assert workflow fails with the correct error type
3. Activity times out (schedule-to-close) — assert timeout is handled correctly
4. Compensation activities run in the correct order when a saga step fails
5. Worker restart mid-workflow — assert workflow resumes correctly (use test environment's skip-time)

Use the Temporal test environment to inject failures via activity mock implementations.
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Test both retryable and non-retryable error paths — they often have very different behavior.
- Use `testEnv.setTestTimeout` generously — failure path tests can involve many retries in simulated time.

## Related prompts

- [Test Workflow Logic](./test-workflow-logic.md)
- [Error Handling & Retries](../building/error-handling.md)
