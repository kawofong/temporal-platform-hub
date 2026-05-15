---
id: visualize-workflow
title: Visualize a Workflow
sidebar_label: 🗺️ Visualize a Workflow
description: Generate a sequence diagram or flowchart of a workflow from code or a description.
---

:::note
Replace the placeholder prompt below with the version your team has validated. Add example output, tips, and any org-specific context.
:::

## Prompt

```
Generate a visual diagram of the following Temporal workflow.

[Paste your workflow code OR describe the process in plain language, e.g.:
"When an order is placed, reserve inventory, then charge payment. If payment succeeds, 
create a shipment and send a confirmation email. If payment fails, release the inventory."]

Please produce:
1. A sequence diagram in Mermaid format showing the interactions between the Workflow, 
   Activities, and external systems
2. A happy-path flowchart showing the step-by-step execution
3. An error-path flowchart showing what happens when each activity fails

Label each step with the activity name and note any retry or compensation behavior.
```

## Example output

> Add an example Mermaid diagram your team has validated.

## Tips

- Mermaid diagrams render natively in GitHub, Notion, and many wikis — paste the output directly.
- Ask for a "swimlane" variant if you want to show which system (Temporal, your service, external API) owns each step.
- Paste the Mermaid output into [mermaid.live](https://mermaid.live) to preview it instantly.

## Related prompts

- [Explain Temporal Concepts](./explain-temporal-concepts.md)
- [Explain Workflow History](./explain-workflow-history.md)
- [Design a New Workflow](../building/design-workflow.md)
