---
id: namespace-changes
title: Namespace Changes
sidebar_label: 🗂️ Namespace Changes
description: Safely migrate workflows or update namespace configuration with minimal downtime.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Help me plan a safe namespace configuration change for Temporal.

Change type:
[Choose one:
- Migrate running workflows from namespace A to namespace B
- Update retention period on an existing namespace
- Update search attribute configuration
- Change namespace-level security policies]

Details:
- Current namespace: [name, region, retention]
- Desired state: [describe what you want]
- Number of active workflows: [approximate count]
- Acceptable downtime: [none / maintenance window available]

Please provide:
1. Step-by-step migration plan
2. Rollback plan
3. Validation steps to confirm success
4. Any Temporal Cloud-specific considerations
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Namespace migrations in Temporal Cloud require Temporal support involvement for large-scale moves.
- Always validate retention changes in a non-production namespace first.

## Related prompts

- [Deprecate a Workflow](./deprecate-workflow.md)
- [Monitor Worker Health](./monitor-worker-health.md)
