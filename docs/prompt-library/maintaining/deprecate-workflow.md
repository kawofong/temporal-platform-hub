---
id: deprecate-workflow
title: Deprecate a Workflow
sidebar_label: 🗑️ Deprecate a Workflow
description: Drain and retire a workflow version while keeping in-flight executions safe.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Help me safely deprecate the following Temporal workflow type in [Python/Go/TypeScript/Java].

Workflow to deprecate: [workflow type name]
Replacement workflow (if any): [new workflow type name or "none"]
Current in-flight executions: [approximate count or "unknown"]
Desired timeline: [e.g., "drain within 2 weeks"]

Please provide:
1. A deprecation checklist — steps to stop new executions from being started
2. How to monitor drain progress (query for remaining running executions)
3. Whether any in-flight executions need manual intervention
4. Code changes needed to remove the workflow safely after drain completes
5. Communication template to notify teams that depend on this workflow
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Use `temporal workflow list --query 'WorkflowType="OldWorkflowName" AND ExecutionStatus="Running"'` to track drain.
- Keep the old worker code deployed until all in-flight executions complete.

## Related prompts

- [Workflow Versioning](./workflow-versioning.md)
- [Namespace Changes](./namespace-changes.md)
