---
id: child-workflows
title: Child Workflows
sidebar_label: 🌿 Child Workflows
description: Break a large workflow into parent/child relationships with proper cancellation propagation.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Refactor the following Temporal workflow in [Python/Go/TypeScript/Java] to use child workflows.

[Paste your existing workflow code]

Refactoring goals:
- Identify logical sub-processes that would benefit from independent execution history
- Extract them as child workflows with appropriate parent-close policies
- Ensure cancellation propagates correctly from parent to children
- Preserve error handling — child workflow failures should bubble up appropriately
- Add comments explaining why each sub-process was made a child workflow
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Consider `ABANDON` parent-close policy for child workflows that should complete independently.
- Child workflows have their own history limits — use them for long-running sub-processes.

## Related prompts

- [Design a New Workflow](./design-workflow.md)
- [Error Handling & Retries](./error-handling.md)
