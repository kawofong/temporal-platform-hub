---
sidebar_position: 13
id: prompt-library
title: Prompt Library
sidebar_label: 💬 Prompt Library
description: A centralized repository of recommended prompts for engineers and business users building, maintaining, and testing Temporal workflows with AI coding tools.
toc_max_heading_level: 2
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

:::note
Add, remove, or edit prompt cards to reflect the patterns your organization finds most valuable. Each card links to a detailed page with the full prompt text, example output, and usage tips.
:::

A curated collection of prompts for use with AI coding tools (Claude, Copilot, Cursor, etc.) when working with Temporal. Organized by workflow stage — pick a card to see the full prompt and guidance.

## Learn

Use these prompts to understand how Temporal works, explore core concepts, and visualize workflow execution.

<div class="prompt-grid-single-row">
  <a class="prompt-card" href="prompt-library/learning/explain-temporal-concepts">
    <span class="prompt-card-icon">📚</span>
    <p class="prompt-card-title">Explain Temporal Concepts</p>
    <p class="prompt-card-desc">Get plain-language explanations of Temporal's core concepts tailored to your background.</p>
  </a>
  <a class="prompt-card" href="prompt-library/learning/explain-workflow-history">
    <span class="prompt-card-icon">🧠</span>
    <p class="prompt-card-title">Explain Workflow History</p>
    <p class="prompt-card-desc">Demystify the event sourcing model and how Temporal replays workflow executions.</p>
  </a>
  <a class="prompt-card" href="prompt-library/learning/visualize-workflow">
    <span class="prompt-card-icon">🗺️</span>
    <p class="prompt-card-title">Visualize a Workflow</p>
    <p class="prompt-card-desc">Generate a sequence diagram or flowchart of a workflow from code or a description.</p>
  </a>
</div>

## Build

Use these prompts when designing and implementing new Temporal workflows and activities.

:::tip Use the Temporal Developer Skill
The **[Temporal Developer Skill](https://temporal.io/blog/introducing-temporal-developer-skill)** gives AI coding assistants deep, up-to-date knowledge of the Temporal SDK and best practices. Activate it in your AI tool before running any of the prompts below. With the skill active, your assistant understands Temporal-specific patterns (durable execution, determinism constraints, activity heartbeating, etc.) out of the box — so your prompts can focus on business logic instead of re-explaining the framework.

**Recommended workflow:** activate the skill once at the start of a session, then run the build prompts in sequence as you scaffold, implement, and wire up your workflow.
:::

<div class="prompt-grid">
  <a class="prompt-card" href="prompt-library/building/child-workflows">
    <span class="prompt-card-icon">🌿</span>
    <p class="prompt-card-title">Child Workflows</p>
    <p class="prompt-card-desc">Break a large workflow into parent/child relationships with proper cancellation propagation.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/design-workflow">
    <span class="prompt-card-icon">🧱</span>
    <p class="prompt-card-title">Design a New Workflow</p>
    <p class="prompt-card-desc">Scaffold a Temporal workflow from a plain-language description of the business process.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/error-handling">
    <span class="prompt-card-icon">🛡️</span>
    <p class="prompt-card-title">Error Handling & Retries</p>
    <p class="prompt-card-desc">Add robust error handling, custom retry logic, and compensation patterns.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/implement-activities">
    <span class="prompt-card-icon">⚡</span>
    <p class="prompt-card-title">Implement Activities</p>
    <p class="prompt-card-desc">Generate activity stubs with correct retry policies, timeouts, and heartbeating.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/signals-queries">
    <span class="prompt-card-icon">📡</span>
    <p class="prompt-card-title">Signals & Queries</p>
    <p class="prompt-card-desc">Add signal handlers and query handlers to an existing workflow definition.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/timers-scheduling">
    <span class="prompt-card-icon">⏱️</span>
    <p class="prompt-card-title">Timers & Scheduling</p>
    <p class="prompt-card-desc">Implement durable sleeps, deadlines, and scheduled workflow patterns.</p>
  </a>
  <a class="prompt-card" href="prompt-library/building/worker-config">
    <span class="prompt-card-icon">⚙️</span>
    <p class="prompt-card-title">Worker Configuration</p>
    <p class="prompt-card-desc">Configure workers with appropriate task queue settings, concurrency, and pollers.</p>
  </a>
</div>

## Maintain

Use these prompts when operating, updating, or optimizing workflows already running in production.

<div class="prompt-grid">
  <a class="prompt-card" href="prompt-library/maintaining/debug-stuck-workflow">
    <span class="prompt-card-icon">🔍</span>
    <p class="prompt-card-title">Debug a Stuck Workflow</p>
    <p class="prompt-card-desc">Diagnose why a workflow is blocked and generate a plan to unblock it safely.</p>
  </a>
</div>

## Test

Use these prompts when writing, improving, or running tests for Temporal workflows and activities.

<div class="prompt-grid">
  <a class="prompt-card" href="prompt-library/testing/replay-from-history">
    <span class="prompt-card-icon">⏪</span>
    <p class="prompt-card-title">Replay from History</p>
    <p class="prompt-card-desc">Replay a production workflow history to detect non-determinism after a code change.</p>
  </a>
</div>
