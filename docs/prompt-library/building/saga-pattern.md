---
id: saga-pattern
title: Saga / Compensation
sidebar_label: 🔄 Saga / Compensation
description: Implement the saga pattern with compensating transactions for distributed consistency.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Implement the saga pattern with compensating transactions in the following Temporal workflow in [Python/Go/TypeScript/Java].

[Paste your existing workflow code or describe the steps]

Steps and their compensations:
[e.g.,
- ReserveInventory → compensation: ReleaseInventory
- ChargePayment → compensation: RefundPayment
- CreateShipment → compensation: CancelShipment]

Requirements:
- Execute compensations in reverse order on any failure
- Each compensation should be idempotent and have its own retry policy
- Log which compensations ran and their outcomes
- Surface a meaningful error to the caller indicating which step failed
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Compensation activities should be non-retryable only if they genuinely cannot succeed — most should retry.
- Ask the model to walk through the failure scenario for each step to validate the compensation logic.

## Related prompts

- [Error Handling & Retries](./error-handling.md)
- [Implement Activities](./implement-activities.md)
