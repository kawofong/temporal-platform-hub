---
id: error-handling
title: Error Handling & Retries
sidebar_label: 🛡️ Error Handling & Retries
description: Add robust error handling, custom retry logic, and compensation patterns.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Review the following Temporal workflow and activity code and improve its error handling in [Python/Go/TypeScript/Java].

[Paste your code here]

Improvements needed:
- Identify activities with missing or overly broad retry policies and suggest better defaults
- Add ApplicationError wrapping with appropriate retryable flags
- Implement compensation logic for any activities that need rollback on failure
- Ensure non-retryable errors (e.g., validation errors, auth failures) fail fast
- Add structured logging at key failure points
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Using the Temporal Developer Skill

Activate the **[Temporal Developer Skill](https://temporal.io/blog/introducing-temporal-developer-skill)** in your AI coding assistant before running this prompt. With the skill active, your assistant already knows how `ApplicationError`, `retryable` flags, and non-retryable error lists work across SDKs — so it will produce correct retry policy recommendations without you needing to explain Temporal's error model.

## Tips

- Paste the actual code rather than describing it — the model will catch specific issues you might miss.
- Ask it to list the failure modes it identified before generating the fix.

## Related prompts

- [Implement Activities](./implement-activities.md)
- [Saga / Compensation](./saga-pattern.md)
