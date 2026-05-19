---
id: worker-config
title: Worker Configuration
sidebar_label: ⚙️ Worker Configuration
description: Configure workers with appropriate task queue settings, concurrency, and pollers.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Generate a production-ready Temporal worker configuration in [Python/Go/TypeScript/Java].

Context:
- Namespace: [your namespace]
- Task queue: [your task queue name]
- Workflows registered: [list workflow types]
- Activities registered: [list activity types]
- Expected throughput: [approximate workflow starts per second]
- Activity characteristics: [e.g., "mostly I/O bound HTTP calls, p99 ~2s"]

Requirements:
- Set max concurrent workflow tasks, activity tasks, and local activity executions
- Configure poller counts appropriate for the expected load
- Add graceful shutdown handling
- Include sticky execution settings if appropriate
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Using the Temporal Developer Skill

Activate the **[Temporal Developer Skill](https://temporal.io/blog/introducing-temporal-developer-skill)** in your AI coding assistant before running this prompt. With the skill active, your assistant understands Temporal's concurrency model and poller mechanics, so it will recommend concurrency settings that match your activity characteristics (I/O-bound vs. CPU-bound) without generic boilerplate explanations.

## Tips

- I/O-bound activities benefit from higher concurrency; CPU-bound activities should match core count.
- Include your organization's standard namespace and connection config in the prompt for accurate output.

## Related prompts

- [Monitor Worker Health](../maintaining/monitor-worker-health.md)
- [Load Testing](../testing/load-testing.md)
