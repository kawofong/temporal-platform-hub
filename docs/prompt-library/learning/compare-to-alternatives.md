---
id: compare-to-alternatives
title: Compare to Alternatives
sidebar_label: ⚖️ Compare to Alternatives
description: Understand how Temporal compares to queues, cron jobs, or other orchestration tools.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context.
:::

## Prompt

```
Help me understand when to use Temporal versus the alternatives for the following use case.

Use case:
[Describe what you're trying to build, e.g., "process customer orders with payment, 
inventory, and fulfillment steps" or "run a nightly data pipeline across 3 services"]

Alternatives we're considering:
[List what you're comparing, e.g.:
- Message queues (SQS, Kafka, RabbitMQ)
- Cron jobs / scheduled tasks
- AWS Step Functions / Azure Durable Functions
- Homegrown orchestration (polling loops, state tables)
- Airflow / Prefect / Dagster]

For each alternative:
1. How would you implement this use case with it?
2. What breaks down or gets complicated as the use case scales?
3. Where does Temporal have a clear advantage?
4. Where might the alternative actually be the better fit?

Conclude with a recommendation and the key trade-off.
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Be specific about your use case — generic comparisons are less useful than scenario-specific ones.
- Ask it to frame the answer for a specific audience (e.g., "explain this to a VP of Engineering making a build-vs-buy decision").

## Related prompts

- [Explain Temporal Concepts](./explain-temporal-concepts.md)
- [Design a New Workflow](../building/design-workflow.md)
