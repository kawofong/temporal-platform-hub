---
id: signals-queries
title: Signals & Queries
sidebar_label: 📡 Signals & Queries
description: Add signal handlers and query handlers to an existing workflow definition.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Add signal and query handlers to the following Temporal workflow in [Python/Go/TypeScript/Java].

[Reference the workflow class to update, e.g., src/workflows/order_workflow.py — OrderWorkflow]

Signals to add:
[e.g., "cancel — allows external systems to request graceful cancellation"]
[e.g., "updatePriority(level: string) — updates processing priority mid-execution"]

Queries to add:
[e.g., "getStatus — returns current workflow state and progress percentage"]
[e.g., "getPendingItems — returns list of items not yet processed"]

Requirements:
- Signal handlers must be idempotent
- Queries must be read-only and not block
- Use appropriate state variables to track signal-driven state changes
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Using the Temporal Developer Skill

Activate the **[Temporal Developer Skill](https://temporal.io/blog/introducing-temporal-developer-skill)** in your AI coding assistant before running this prompt. With the skill active, your assistant knows the threading and determinism constraints that apply to signal and query handlers across SDKs, so it won't accidentally generate handlers that mutate state unsafely or block a query.

## Tips

- Signals are async — if you need synchronous interaction, consider using an Update instead.
- Ask the model to explain the state machine that the signals create to validate correctness.

## Related prompts

- [Design a New Workflow](./design-workflow.md)
