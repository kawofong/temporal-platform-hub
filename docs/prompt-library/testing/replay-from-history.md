---
id: replay-from-history
title: Replay from History
sidebar_label: ⏪ Replay from History
description: Replay a production workflow history to detect non-determinism after a code change.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Help me set up workflow history replay testing for non-determinism detection in [Python/Go/TypeScript/Java].

Context:
- I have a workflow that has been running in production
- I want to validate that my code changes don't break replay of existing executions

Please generate:
1. A script to export workflow histories from Temporal (using the CLI or SDK)
2. A replay test harness that runs each exported history against the updated code
3. Clear error output when a non-determinism error is detected, showing which event caused the mismatch
4. Instructions for how to integrate this into CI so it runs on every PR
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Export histories from workflows that exercised different code paths — one happy-path history isn't enough.
- Run replay tests before deploying any workflow code change to catch non-determinism early.

## Related prompts

- [Analyze Workflow History](../maintaining/analyze-history.md)
- [Workflow Versioning](../maintaining/workflow-versioning.md)
