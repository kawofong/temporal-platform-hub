---
id: analyze-history
title: Analyze Workflow History
sidebar_label: 📜 Analyze Workflow History
description: Interpret a workflow event history to identify failures, bottlenecks, or unexpected behavior.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Analyze the following Temporal workflow event history and provide a summary of what happened.

[Paste the workflow history JSON or the output of `temporal workflow show`]

Please provide:
1. A plain-language summary of the workflow execution from start to finish
2. Identification of any failures, retries, or unexpected events
3. Timeline of key milestones (activity starts/completions, signals received, etc.)
4. Any bottlenecks — activities with unusually long scheduled-to-start or start-to-close times
5. Recommendations for improving reliability or performance based on what you observe
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Export history via `temporal workflow show --workflow-id <id> --output json` for the best results.
- Ask follow-up questions about specific events to drill into failures.

## Related prompts

- [Debug a Stuck Workflow](./debug-stuck-workflow.md)
- [Replay from History](../testing/replay-from-history.md)
