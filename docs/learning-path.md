---
sidebar_position: 5
id: learning-path
title: Learning Paths
sidebar_label: 📚 Learning Paths
description: Structured learning paths from foundational concepts to advanced patterns, tailored for Software Developers, AI Developers, and Platform Engineers.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](./intro.md).
:::

:::note
Customize learning paths for your developers to learn Temporal based on their skills and personas.
:::

This guide provides a structured learning path from foundational concepts to advanced patterns, tailored specifically for Software Developers, AI Developers, and Platform Engineers.

Temporal offers free, self-paced training courses that provide a solid grounding in the platform. Developers can sign up for free for these courses using their work emails at [learn.temporal.io](http://learn.temporal.io).

## Foundation

1. [Temporal 101: Introducing the Temporal Platform](https://learn.temporal.io/courses/temporal_101/)
   1. Learn the fundamentals of Temporal, including Workflows, Activities, and the core value proposition of Durable Execution.
2. [Temporal 102: Exploring Durable Execution](https://learn.temporal.io/courses/temporal_102/)
   1. You will acquire skills necessary to use Temporal throughout the development lifecycle by learning how to test, debug, and deploy applications.

## Intermediate

1. [Securing Application Data](https://learn.temporal.io/courses/appdatasec/)
   1. Provides general guidance and example applications for addressing user management, encryption standards, and key rotation.
2. [Interacting with Workflows](https://learn.temporal.io/courses/interacting_with_workflows/)
   1. Learn how to interact with Workflows using Signal, Update, and Query.
3. [Crafting an Error Handling Strategy](https://learn.temporal.io/courses/errstrat/)
   1. You will explore the nature of different types of failures and investigate the support that Temporal provides for addressing them.

## Advanced

The Advanced learning paths are tailored to 3 distinct user personas:

1. [Platform Engineers](#platform-engineer)
2. [Software Developers](#software-developers)
3. [AI Developers](#ai-developers)

### Platform Engineer {#platform-engineer}

1. [Introduction to Temporal Cloud](https://learn.temporal.io/courses/intro_to_temporal_cloud/)
   1. Learn the role of Temporal Cloud, how to log into and navigate its Web UI, and how to perform tasks that new Temporal Cloud users may do in preparation for using this service.
2.  [Best practices | Temporal Platform Documentation](https://docs.temporal.io/best-practices)
   1. Learn the foundational principles and best practices for using Temporal Cloud.

### Software Developers {#software-developers}

1. [Versioning Workflows](https://learn.temporal.io/courses/versioning/)
   1. In this course, you will learn how to safely evolve your Temporal application code in production.
2. [Worker Versioning](https://learn.temporal.io/courses/worker_versioning/)
   1. You will learn the benefits of Worker Versioning and evaluate tradeoffs of various versioning approaches.

### AI Developers {#ai-developers}

The AI Developer track is structured differently from the Software Developer track. Rather than starting with Sagas and payment processing, AI engineers should start with the agent loop pattern and work backward to understand *why* Temporal's determinism rules matter for their workloads.

1. **Start here**: [Building Durable AI Applications with Temporal](https://learn.temporal.io/tutorials/ai/building-durable-ai-applications/)
   1. Build your first durable AI application. Understand how LLM calls map to Activities, why prompts must not live in Workflow code, and how Temporal's retry primitives differ from what LangChain or similar frameworks provide.
2. **Core pattern**: [AI Engineering → Reference Architecture](./ai-engineering/reference-architecture.md)
   1. Understand the orchestrator Workflow pattern, correct LLM retry strategy (respecting `Retry-After` headers), parallel tool dispatch with `asyncio.gather`, and the human-in-the-loop Signal/Update gate.
3. **Production patterns**: [AI Engineering → AI Patterns](./ai-engineering/patterns.md)
   1. Five patterns covering LLM Activities with structured retries, human-in-the-loop approval, prompt versioning and replay safety, parallel tool dispatch, and agent loops with `continue_as_new` for history pruning.
4. **Long-running agents**: [Continue as New](https://docs.temporal.io/develop/python/continue-as-new)
   1. Understand how `continue_as_new` prunes Workflow history for agents that run for hundreds of steps, and how to carry agent state across executions.
5. **Security**: [AI Engineering → Security & Governance](./ai-engineering/security.md)
   1. Payload encryption for LLM I/O, credential management for third-party AI APIs, namespace isolation for sensitive AI workloads, and audit trail requirements for model governance and the EU AI Act.
6. **MCP tools**: [Building Durable MCP Tool with Temporal](https://learn.temporal.io/tutorials/ai/building-mcp-tools-with-temporal/)
   1. Learn how to build long-running Model Context Protocol (MCP) tools backed by Temporal Activities, and how to run MCP tool calls under the identity of the requesting user.

## What's next

* Enhance your coding agent with [Temporal Skills and MCP tools](https://docs.temporal.io/with-ai) for expert-level Temporal guidance as you develop.
* Check whether [Temporal is the right technology for your use case](./decision-framework.md).
