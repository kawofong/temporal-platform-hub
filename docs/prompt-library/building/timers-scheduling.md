---
id: timers-scheduling
title: Timers & Scheduling
sidebar_label: ⏱️ Timers & Scheduling
description: Implement durable sleeps, deadlines, and scheduled workflow patterns.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Add timer and scheduling logic to the following Temporal workflow in [Python/Go/TypeScript/Java].

[Paste your existing workflow code or describe the scheduling requirement]

Requirements:
[Choose what applies:]
- Add a durable sleep of [duration] between steps
- Implement a deadline: if step X hasn't completed within [duration], take fallback action
- Send a reminder notification if the workflow hasn't progressed after [duration]
- Schedule this workflow to run on a cron schedule: [cron expression]

Use Temporal's workflow.sleep / workflow.timer (not time.sleep or OS timers) to ensure durability.
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Never use language-native sleep functions inside workflows — they break determinism.
- For cron workflows, clarify whether missed runs should backfill or skip.

## Related prompts

- [Design a New Workflow](./design-workflow.md)
- [Test Workflow Logic](../testing/test-workflow-logic.md)
