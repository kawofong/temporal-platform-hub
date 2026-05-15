---
id: monitor-worker-health
title: Monitor Worker Health
sidebar_label: 📊 Monitor Worker Health
description: Generate queries and dashboards to monitor worker saturation, task latency, and errors.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Help me set up monitoring for Temporal workers in [Python/Go/TypeScript/Java].

Environment:
- Metrics backend: [Prometheus / Datadog / CloudWatch / other]
- Deployment: [Kubernetes / ECS / bare metal]
- Namespace: [your namespace]
- Task queues: [list your task queues]

Generate:
1. Key metrics to alert on (worker saturation, task schedule-to-start latency, activity error rate)
2. Recommended alert thresholds with justification
3. A dashboard layout with the most important panels
4. Runbook steps for each alert condition
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Temporal emits SDK metrics by default — make sure your worker is configured to export them.
- `temporal_task_schedule_to_start_latency` is the single best indicator of worker undersaturation.

## Related prompts

- [Worker Configuration](../building/worker-config.md)
- [Debug a Stuck Workflow](./debug-stuck-workflow.md)
