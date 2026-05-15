---
id: design-workflow
title: Design a New Workflow
sidebar_label: 🏗️ Design a New Workflow
description: Scaffold a Temporal workflow from a plain-language description of the business process.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context (namespace, SDK language, coding standards).
:::

## Prompt

```
You are an expert in Temporal workflow design. Given the following business process description, 
scaffold a Temporal workflow in [Python/Go/TypeScript/Java].

Business process:
[Describe the process here — e.g., "When a customer submits an order, reserve inventory, 
charge payment, and send a confirmation email. If payment fails, release the inventory."]

Requirements:
- Use meaningful workflow and activity names that reflect the domain
- Include appropriate timeout and retry policy defaults
- Add comments explaining non-obvious design decisions
- Flag any steps that should be separate child workflows
```

## Example output

> Add an example of what good output looks like for your team's use cases.

## Tips

- Be specific about the SDK language — output quality varies significantly between languages.
- Include your organization's namespace and task queue naming conventions in the prompt.
- Ask for a sequence diagram alongside the code to validate the happy path before implementation.

## Related prompts

- [Implement Activities](./implement-activities.md)
- [Error Handling & Retries](./error-handling.md)
- [Child Workflows](./child-workflows.md)
