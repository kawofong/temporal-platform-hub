---
id: regression-testing
title: Regression Testing
sidebar_label: 🔬 Regression Testing
description: Build a regression suite to catch behavioral regressions across workflow versions.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Build a regression test suite for the following Temporal workflow in [Python/Go/TypeScript/Java].

[Paste your workflow code]

Known historical issues or edge cases to cover:
[List any past bugs or tricky scenarios, e.g.:
- "Workflow hung when the payment service returned a 429 after 3 retries"
- "Compensation didn't run when the third activity timed out"]

Requirements:
1. Create a test case for each known issue to prevent regressions
2. Create parameterized tests for key input variations
3. Ensure the suite runs in CI without external dependencies (use test environment + mocks)
4. Add assertions on workflow outputs, not just completion status
5. Generate a test coverage summary showing which workflow branches are exercised
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Seed regression tests directly from past incidents — if it broke once, test it explicitly.
- Parameterized tests are especially useful for input validation edge cases.

## Related prompts

- [Test Workflow Logic](./test-workflow-logic.md)
- [Simulate Failures](./simulate-failures.md)
