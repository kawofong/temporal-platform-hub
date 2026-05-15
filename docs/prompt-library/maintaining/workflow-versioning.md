---
id: workflow-versioning
title: Workflow Versioning
sidebar_label: 🏷️ Workflow Versioning
description: Add versioning to safely deploy breaking changes without disrupting running workflows.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
I need to make a breaking change to a Temporal workflow that has running executions. Help me implement versioning safely in [Python/Go/TypeScript/Java].

Current workflow code:
[Paste current implementation]

Proposed change:
[Describe what you want to change — e.g., "add a new activity between StepA and StepB" or 
"rename activity FooActivity to BarActivity"]

Requirements:
- Use GetVersion / workflow.GetVersion to gate the new behavior
- Ensure existing in-flight executions continue on the old code path
- Ensure new executions use the updated code path
- Add a comment indicating when the old version branch can be removed
- Explain what version numbers to use and why
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Ask the model to explain the replay scenario for both old and new executions to validate correctness.
- Include your deployment strategy (blue/green, rolling) — it affects how to handle the version cutover.

## Related prompts

- [Replay from History](../testing/replay-from-history.md)
- [Deprecate a Workflow](./deprecate-workflow.md)
