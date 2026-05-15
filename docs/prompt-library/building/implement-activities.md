---
id: implement-activities
title: Implement Activities
sidebar_label: ⚡ Implement Activities
description: Generate activity stubs with correct retry policies, timeouts, and heartbeating.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
Generate Temporal activity implementations in [Python/Go/TypeScript/Java] for the following operations:

Activities needed:
[List each activity, e.g.:
- ReserveInventory(orderId, items) — calls inventory service HTTP API
- ChargePayment(orderId, amount) — calls Stripe API
- SendConfirmationEmail(orderId, email) — calls SendGrid API]

For each activity:
- Set schedule-to-close, start-to-close, and heartbeat timeouts appropriate for the operation
- Configure retry policy (max attempts, backoff, non-retryable errors)
- Add heartbeating for any activity expected to run longer than 10 seconds
- Handle and wrap external errors as ApplicationError with retryable=false where appropriate
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Specify which errors from external services should be non-retryable (e.g., 400 Bad Request vs 500).
- Include your organization's standard timeout values if you have them.
- Ask the model to explain its timeout choices so you can review them.

## Related prompts

- [Design a New Workflow](./design-workflow.md)
- [Error Handling & Retries](./error-handling.md)
