---
id: unit-test-activities
title: Unit Test Activities
sidebar_label: 🧪 Unit Test Activities
description: Write unit tests for activity implementations with mocked dependencies.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Write unit tests for the following Temporal activities in [Python/Go/TypeScript/Java].

[Paste your activity code]

Testing framework: [pytest / JUnit / Jest / Go testing]

For each activity, write tests that cover:
- Happy path with valid inputs
- Error handling — transient errors that should be retried
- Error handling — non-retryable errors that should fail immediately
- Edge cases: empty inputs, boundary values, missing optional fields

Use mocks/stubs for all external dependencies (HTTP clients, database connections, etc.).
Do not use the Temporal test environment for activity unit tests — test the function directly.
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Activity functions are plain functions — test them directly, no Temporal harness needed.
- Ask the model to add test names that read as human sentences for better failure messages.

## Related prompts

- [Mock External Services](./mock-external-services.md)
- [Test Workflow Logic](./test-workflow-logic.md)
