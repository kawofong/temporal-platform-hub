---
id: optimize-performance
title: Optimize Performance
sidebar_label: 🚀 Optimize Performance
description: Identify and resolve workflow performance issues including latency and throughput bottlenecks.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Review the following Temporal workflow for performance issues and suggest optimizations in [Python/Go/TypeScript/Java].

[Paste your workflow and activity code]

Observed symptoms:
[e.g., "end-to-end workflow latency is 45s, we expect ~10s" or "throughput caps at 50 workflows/min"]

Current worker config:
[Paste worker configuration]

Please:
1. Identify sequential activities that could run in parallel
2. Flag unnecessarily large payloads being passed through workflow state
3. Recommend worker concurrency settings based on the activity characteristics
4. Suggest any caching opportunities at the activity level
5. Identify history bloat risks (e.g., large loops without continue-as-new)
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Parallelizing activities is often the highest-leverage change — ask specifically about that first.
- Workflows with history over ~10k events should use continue-as-new.

## Related prompts

- [Worker Configuration](../building/worker-config.md)
- [Load Testing](../testing/load-testing.md)
