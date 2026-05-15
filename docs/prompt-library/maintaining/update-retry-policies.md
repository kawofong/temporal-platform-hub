---
id: update-retry-policies
title: Update Retry Policies
sidebar_label: 🔁 Update Retry Policies
description: Review and tune retry policies across activities based on failure rate observations.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Review and improve the retry policies in the following Temporal workflow code in [Python/Go/TypeScript/Java].

[Paste your workflow and activity code]

Observed failure data (optional but helpful):
[e.g., "ChargePayment fails ~5% of the time with transient 503s, occasionally with non-retryable 402s"]

Requirements:
- Ensure transient errors (network, 5xx) are retried with appropriate backoff
- Ensure non-retryable errors (validation, auth, 4xx) fail immediately
- Set maximum retry limits that balance reliability with acceptable latency
- Avoid retry storms — use exponential backoff with jitter
- Flag any activities where the current policy could cause runaway retries
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Include actual error types and HTTP status codes your services return for the most accurate output.
- Ask for a table summarizing old vs. new policy values for easy review.

## Related prompts

- [Error Handling & Retries](../building/error-handling.md)
- [Analyze Workflow History](./analyze-history.md)
