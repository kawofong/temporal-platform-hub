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
A Temporal workflow is stuck and not making progress. Help me diagnose and resolve it.

Workflow details:
- Workflow type: [workflow type name]
- Workflow ID: [workflow ID]
- Run ID: [run ID]
- Last event in history: [paste the last few events from the UI or tctl/temporal CLI]
- Time since last progress: [duration]
- Worker status: [running / not running / unknown]

Symptoms:
[Describe what you observe — e.g., "stuck in RUNNING state, no pending activities, no timers"]

Please:
1. Identify the most likely root causes given the history
2. Provide a step-by-step diagnosis checklist
3. Suggest remediation options ranked by risk (prefer non-destructive first)
4. Warn about any risks with each option
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Paste the actual event history if you can — it dramatically improves diagnosis quality.
- Always try `temporal workflow describe` and check pending activities before escalating.

## Related prompts

- [Analyze Workflow History](./analyze-history.md)
- [Monitor Worker Health](./monitor-worker-health.md)
