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
  <a class="prompt-card" href="./learning/compare-to-alternatives">
    <span class="prompt-card-icon">⚖️</span>
    <p class="prompt-card-title">Compare to Alternatives</p>
    <p class="prompt-card-desc">Understand how Temporal compares to queues, cron jobs, or other orchestration tools.</p>
  </a>
  <a class="prompt-card" href="./learning/explain-temporal-concepts">
    <span class="prompt-card-icon">📚</span>
    <p class="prompt-card-title">Explain Temporal Concepts</p>
    <p class="prompt-card-desc">Get plain-language explanations of Temporal's core concepts tailored to your background.</p>
  </a>
  <a class="prompt-card" href="./learning/explain-workflow-history">
    <span class="prompt-card-icon">🧠</span>
    <p class="prompt-card-title">Explain Workflow History</p>
    <p class="prompt-card-desc">Demystify the event sourcing model and how Temporal replays workflow executions.</p>
  </a>
  <a class="prompt-card" href="./learning/visualize-workflow">
    <span class="prompt-card-icon">🗺️</span>
    <p class="prompt-card-title">Visualize a Workflow</p>
    <p class="prompt-card-desc">Generate a sequence diagram or flowchart of a workflow from code or a description.</p>
  </a>
</div>

## Build

Use these prompts when designing and implementing new Temporal workflows and activities.

<div class="prompt-grid">
  <a class="prompt-card" href="./building/child-workflows">
    <span class="prompt-card-icon">🌿</span>
    <p class="prompt-card-title">Child Workflows</p>
    <p class="prompt-card-desc">Break a large workflow into parent/child relationships with proper cancellation propagation.</p>
  </a>
  <a class="prompt-card" href="./building/design-workflow">
    <span class="prompt-card-icon">🧱</span>
    <p class="prompt-card-title">Design a New Workflow</p>
    <p class="prompt-card-desc">Scaffold a Temporal workflow from a plain-language description of the business process.</p>
  </a>
  <a class="prompt-card" href="./building/error-handling">
    <span class="prompt-card-icon">🛡️</span>
    <p class="prompt-card-title">Error Handling & Retries</p>
    <p class="prompt-card-desc">Add robust error handling, custom retry logic, and compensation patterns.</p>
  </a>
  <a class="prompt-card" href="./building/implement-activities">
    <span class="prompt-card-icon">⚡</span>
    <p class="prompt-card-title">Implement Activities</p>
    <p class="prompt-card-desc">Generate activity stubs with correct retry policies, timeouts, and heartbeating.</p>
  </a>
  <a class="prompt-card" href="./building/saga-pattern">
    <span class="prompt-card-icon">🔄</span>
    <p class="prompt-card-title">Saga / Compensation</p>
    <p class="prompt-card-desc">Implement the saga pattern with compensating transactions for distributed consistency.</p>
  </a>
  <a class="prompt-card" href="./building/signals-queries">
    <span class="prompt-card-icon">📡</span>
    <p class="prompt-card-title">Signals & Queries</p>
    <p class="prompt-card-desc">Add signal handlers and query handlers to an existing workflow definition.</p>
  </a>
  <a class="prompt-card" href="./building/timers-scheduling">
    <span class="prompt-card-icon">⏱️</span>
    <p class="prompt-card-title">Timers & Scheduling</p>
    <p class="prompt-card-desc">Implement durable sleeps, deadlines, and scheduled workflow patterns.</p>
  </a>
  <a class="prompt-card" href="./building/worker-config">
    <span class="prompt-card-icon">⚙️</span>
    <p class="prompt-card-title">Worker Configuration</p>
    <p class="prompt-card-desc">Configure workers with appropriate task queue settings, concurrency, and pollers.</p>
  </a>
</div>

## Maintain

Use these prompts when operating, updating, or optimizing workflows already running in production.

<div class="prompt-grid">
  <a class="prompt-card" href="./maintaining/analyze-history">
    <span class="prompt-card-icon">📜</span>
    <p class="prompt-card-title">Analyze Workflow History</p>
    <p class="prompt-card-desc">Interpret a workflow event history to identify failures, bottlenecks, or unexpected behavior.</p>
  </a>
  <a class="prompt-card" href="./maintaining/debug-stuck-workflow">
    <span class="prompt-card-icon">🔍</span>
    <p class="prompt-card-title">Debug a Stuck Workflow</p>
    <p class="prompt-card-desc">Diagnose why a workflow is blocked and generate a plan to unblock it safely.</p>
  </a>
  <a class="prompt-card" href="./maintaining/deprecate-workflow">
    <span class="prompt-card-icon">🗑️</span>
    <p class="prompt-card-title">Deprecate a Workflow</p>
    <p class="prompt-card-desc">Drain and retire a workflow version while keeping in-flight executions safe.</p>
  </a>
  <a class="prompt-card" href="./maintaining/monitor-worker-health">
    <span class="prompt-card-icon">📊</span>
    <p class="prompt-card-title">Monitor Worker Health</p>
    <p class="prompt-card-desc">Generate queries and dashboards to monitor worker saturation, task latency, and errors.</p>
  </a>
  <a class="prompt-card" href="./maintaining/namespace-changes">
    <span class="prompt-card-icon">🗂️</span>
    <p class="prompt-card-title">Namespace Changes</p>
    <p class="prompt-card-desc">Safely migrate workflows or update namespace configuration with minimal downtime.</p>
  </a>
  <a class="prompt-card" href="./maintaining/optimize-performance">
    <span class="prompt-card-icon">🚀</span>
    <p class="prompt-card-title">Optimize Performance</p>
    <p class="prompt-card-desc">Identify and resolve workflow performance issues including latency and throughput bottlenecks.</p>
  </a>
  <a class="prompt-card" href="./maintaining/update-retry-policies">
    <span class="prompt-card-icon">🔁</span>
    <p class="prompt-card-title">Update Retry Policies</p>
    <p class="prompt-card-desc">Review and tune retry policies across activities based on failure rate observations.</p>
  </a>
  <a class="prompt-card" href="./maintaining/workflow-versioning">
    <span class="prompt-card-icon">🏷️</span>
    <p class="prompt-card-title">Workflow Versioning</p>
    <p class="prompt-card-desc">Add versioning to safely deploy breaking changes without disrupting running workflows.</p>
  </a>
</div>

## Test

Use these prompts when writing, improving, or running tests for Temporal workflows and activities.

<div class="prompt-grid">
  <a class="prompt-card" href="./testing/integration-testing">
    <span class="prompt-card-icon">🔗</span>
    <p class="prompt-card-title">Integration Testing</p>
    <p class="prompt-card-desc">Write end-to-end tests that run against a real Temporal server in CI.</p>
  </a>
  <a class="prompt-card" href="./testing/load-testing">
    <span class="prompt-card-icon">📈</span>
    <p class="prompt-card-title">Load Testing</p>
    <p class="prompt-card-desc">Generate a load test plan to validate worker capacity and workflow throughput targets.</p>
  </a>
  <a class="prompt-card" href="./testing/mock-external-services">
    <span class="prompt-card-icon">🤖</span>
    <p class="prompt-card-title">Mock External Services</p>
    <p class="prompt-card-desc">Replace external API calls and database interactions with test doubles.</p>
  </a>
  <a class="prompt-card" href="./testing/regression-testing">
    <span class="prompt-card-icon">🔬</span>
    <p class="prompt-card-title">Regression Testing</p>
    <p class="prompt-card-desc">Build a regression suite to catch behavioral regressions across workflow versions.</p>
  </a>
  <a class="prompt-card" href="./testing/replay-from-history">
    <span class="prompt-card-icon">⏪</span>
    <p class="prompt-card-title">Replay from History</p>
    <p class="prompt-card-desc">Replay a production workflow history to detect non-determinism after a code change.</p>
  </a>
  <a class="prompt-card" href="./testing/simulate-failures">
    <span class="prompt-card-icon">💥</span>
    <p class="prompt-card-title">Simulate Failures</p>
    <p class="prompt-card-desc">Inject activity failures and timeouts to verify retry and compensation behavior.</p>
  </a>
  <a class="prompt-card" href="./testing/test-workflow-logic">
    <span class="prompt-card-icon">🧩</span>
    <p class="prompt-card-title">Test Workflow Logic</p>
    <p class="prompt-card-desc">Use the Temporal test environment to assert workflow state transitions and outcomes.</p>
  </a>
  <a class="prompt-card" href="./testing/unit-test-activities">
    <span class="prompt-card-icon">🧪</span>
    <p class="prompt-card-title">Unit Test Activities</p>
    <p class="prompt-card-desc">Write unit tests for activity implementations with mocked dependencies.</p>
  </a>
</div>
