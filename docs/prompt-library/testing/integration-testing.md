---
id: integration-testing
title: Integration Testing
sidebar_label: 🔗 Integration Testing
description: Write end-to-end tests that run against a real Temporal server in CI.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Write integration tests for the following Temporal workflow in [Python/Go/TypeScript/Java] that run against a real Temporal server.

[Paste your workflow code]

CI environment:
- Temporal server: [Temporal CLI dev server / Docker Compose / Temporal Cloud staging namespace]
- Test framework: [pytest / JUnit / Jest / Go testing]

Requirements:
1. Start a real worker and execute the workflow end-to-end
2. Use real activity implementations (or lightweight stubs that call test doubles for external APIs)
3. Assert on the final workflow result and any side effects (database state, events emitted, etc.)
4. Include setup/teardown to clean up workflow executions between test runs
5. Generate a Docker Compose or CI config snippet to spin up Temporal server for the test run
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- `temporal server start-dev` is the fastest way to get a local Temporal server for CI — no Docker needed.
- Keep integration tests focused on the contract between your workflow and external systems.

## Related prompts

- [Test Workflow Logic](./test-workflow-logic.md)
- [Mock External Services](./mock-external-services.md)
