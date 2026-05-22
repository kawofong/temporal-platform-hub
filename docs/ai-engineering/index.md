---
sidebar_position: 1
id: ai-engineering-overview
title: AI Engineering with Temporal
sidebar_label: 🤖 Overview
description: Why Temporal is the right foundation for production agentic AI systems at ABC Financial, and what this section covers.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

Many agentic AI frameworks were designed for short-lived chains of tool calls. This works well for prototyping but shows cracks in production: a process crash loses all agent progress, rate limit backoff ignores `Retry-After` headers, and involving a human reviewer mid-flight requires custom polling tables and webhook plumbing. Temporal was built for exactly these scenarios.

At ABC Financial, engineering teams use Temporal as the durable backbone for AI workloads ranging from single-model inference pipelines to long-running autonomous agents that span hours or days.

## Why Temporal for AI engineering

| Challenge | What breaks without Temporal | What Temporal provides |
| :---- | :---- | :---- |
| **LLM API unreliability** | Process crash loses all agent progress | Durable execution resumes from the last successful step |
| **Rate limits (HTTP 429)** | Exponential backoff ignores `Retry-After` headers | Activity retries with customizable backoff and delay extraction |
| **Long-running agents** | Framework state lives in memory; process restart = start from zero | Workflow state is persisted durably to the Temporal service |
| **Human-in-the-loop** | Custom polling loops, DB flags, and webhook plumbing | Native Signal and Update primitives pause and resume Workflows |
| **Parallel tool calls** | Sequential tool execution adds unnecessary latency | `asyncio.gather` across Activities dispatches them concurrently |
| **Tool call durability** | A tool call that fails mid-run is lost; the agent must restart from scratch | Each tool call is an Activity; results are persisted so a recovered agent resumes after the last successful tool |
| **Audit and compliance** | No record of which prompts, tools, or decisions an agent made | Immutable Workflow history captures every Activity input and output |

## What this section covers

- [**Reference Architecture**](./reference-architecture.md) — How to structure an agentic stack with Temporal: where the orchestrator Workflow lives, how tool calls map to Activities, how to handle LLM retries correctly, and where the human-in-the-loop gate sits.
- [**AI Patterns**](./patterns.md) — Five production-tested patterns for LLM Activities, human-in-the-loop approval gates, prompt versioning, parallel tool dispatch, and long-running agent loops with history pruning.
- [**Agent Framework**](./agent-framework.md) — An internal shared agent framework: one reusable Workflow every team invokes with their own system prompt, context, and tools — retry logic, HITL gates, and audit history are built once and inherited by all teams.
- [**Security & Governance**](./security.md) — Payload encryption for LLM I/O, credential management for third-party AI providers, namespace isolation for sensitive AI workloads, and audit trail for model governance.

## Temporal AI use cases at ABC Financial

The following are representative AI workloads where ABC Financial engineering teams use Temporal as the durable backbone. They serve as a placeholder for your organization to document and showcase your own Temporal AI use cases.

| Use case | Pattern | Why Temporal |
| :---- | :---- | :---- |
| **Regulatory document review** | Long-running agent loop with HITL approval | Compliance reviewers approve or reject AI recommendations via Signal before the agent proceeds; the full decision trail is in Workflow history for audit |
| **Fraud detection pipeline** | ML inference Activity with structured retries | Model inference calls are Activities; rate-limit backoff respects `Retry-After`; failed inference steps resume from the last successful checkpoint rather than restarting the full pipeline |
| **Customer inquiry agent** | Shared agent framework with per-team tools | The platform team owns retry logic and HITL gates; the customer-service team registers their own CRM lookup and policy search Activities on a dedicated Worker |
| **FinOps cost analysis** | Parallel tool dispatch | Cloud cost lookups, budget checks, and forecast queries are dispatched concurrently via `asyncio.gather`; the agent returns a unified summary in one durable execution |
| **Loan application processing** | Human-in-the-loop approval gate | The AI pre-screens applications and surfaces a recommendation; a credit officer approves or rejects via Signal within a configurable SLA window before any downstream action is taken |
| **Model governance audit** | Immutable Workflow history as audit trail | Every prompt sent to the LLM and every tool call result is captured in Workflow history — providing a complete record for EU AI Act compliance and internal model governance review |
| **Customer support agent** | Shared agent framework with per-team tools | A durable conversational agent handles customer enquiries end-to-end — looking up account history, raising tickets, and escalating to a human representative if and when the agent cannot resolve the issue |
| **Financial advice agent** | Long-running agent loop with HITL approval | A personalised advisor agent analyses a customer's portfolio and spending patterns, drafts recommendations, and routes them through a licensed adviser approval gate before any advice is delivered |
| **Future credit analysis** | ML inference Activity with structured retries | Forward-looking credit scoring runs multiple model inference calls as Activities; each step is checkpointed so a partial run survives worker restarts without reprocessing upstream data |
| **Market engagement analysis** | Parallel tool dispatch | Sentiment analysis, news summarisation, and competitor-price lookups are dispatched concurrently across Activities; results are aggregated in the Workflow and published to downstream systems durably |

---

## Key resources

- [temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent) — Reference implementation of a production-ready Temporal AI agent
- [Temporal AI Cookbook](https://docs.temporal.io/ai-cookbook) — Step-by-step solutions for common AI engineering patterns with code
- [Build resilient Agentic AI with Temporal](https://temporal.io/blog/build-resilient-agentic-ai-with-temporal) — Overview of Temporal's differentiators for agentic workloads
- [Using Multi-Agent Architectures with Temporal](https://temporal.io/blog/using-multi-agent-architectures-with-temporal) — Deep dive on multi-agent coordination patterns
