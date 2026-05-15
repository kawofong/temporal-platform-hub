---
id: load-testing
title: Load Testing
sidebar_label: 📈 Load Testing
description: Generate a load test plan to validate worker capacity and workflow throughput targets.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Create a load test plan for the following Temporal workflow in [Python/Go/TypeScript/Java].

Workflow: [workflow type name]
Target throughput: [N workflow starts per second / minute]
Target p99 latency: [expected end-to-end completion time]
Test duration: [how long to run the load test]
Environment: [staging namespace and connection details]

Please generate:
1. A load test script using [omes / custom starter / your preferred tool] that ramps up to target throughput
2. Key metrics to capture during the test (throughput, latency percentiles, error rate, worker saturation)
3. Pass/fail criteria for the test
4. Worker scaling recommendations based on the target load
5. A post-test analysis checklist
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Always load test against a staging namespace — never production.
- Watch `temporal_task_schedule_to_start_latency` as the primary saturation signal during the test.

## Related prompts

- [Worker Configuration](../building/worker-config.md)
- [Monitor Worker Health](../maintaining/monitor-worker-health.md)
