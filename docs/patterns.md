---
sidebar_position: 9
id: patterns
title: Temporal Patterns
sidebar_label: 🧩 Patterns
description: Common Temporal Workflow design patterns with code samples for Python and Java.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](./intro.md).
:::

:::note
Curate Temporal Workflow patterns relevant to your use cases so developers can quickly find solutions.
:::

## Parallel Activity

|  |  |
| :---- | :---- |
| **What it does** | Execute multiple Activities concurrently. |
| **Why use it** | Improve Workflow performance when Activities are independent and don't need sequential execution. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_parallel_activity.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloParallelActivity.java) |

## Custom Search Attributes

|  |  |
| :---- | :---- |
| **What it does** | Adds custom key-value metadata to Workflow executions. |
| **Why use it** | Enables advanced filtering, sorting, and visibility of Workflows in the Web UI and CLI based on business-specific data. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_search_attributes.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloSearchAttributes.java) |

## Child Workflow

|  |  |
| :---- | :---- |
| **What it does** | Spawns a new Workflow execution from within a parent Workflow. |
| **Why use it** | Partition work into smaller chunks, encapsulates Activities into observable components, and model business entities with different lifecycles. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_child_workflow.py), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/asyncchild) |

## Continue as new

|  |  |
| :---- | :---- |
| **What it does** | Atomically completes the current Workflow execution and starts a new one with the same Workflow ID. |
| **Why use it** | Prevents "Event History Limit Exceeded" errors and other [Workflow Execution limits](https://docs.temporal.io/cloud/limits#workflow-execution-event-history-limits) by clearing the history. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_continue_as_new.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloPeriodic.java) |

## Exception handling

|  |  |
| :---- | :---- |
| **What it does** | Implements logic to catch and respond to Activity or Workflow failures. |
| **Why use it** | Ensures system resilience by defining fallback logic, compensation transactions, or specific retry policies when errors occur. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_exception.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloException.java) |

## Cancellation

|  |  |
| :---- | :---- |
| **What it does** | Sends a request to gracefully terminate a running Workflow or specific scope. |
| **Why use it** | Stops unnecessary processing and cleans up resources when a result is no longer needed or a user explicitly stops the process. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_cancellation.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloCancellationScope.java) |

## Async Activity completion

|  |  |
| :---- | :---- |
| **What it does** | Enables the Activity Function to return without the Activity Execution completing. |
| **Why use it** | Essential for long-running external processes that can heartbeat and inform Temporal of its completion. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_async_activity_completion.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloAsyncActivityCompletion.java) |

## Local Activity

|  |  |
| :---- | :---- |
| **What it does** | Executes short-lived Activity logic within the same process as the Workflow Worker. |
| **Why use it** | Reduces latency and history size for short, high-throughput operations that do not require global durability guarantees. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/blob/main/hello/hello_local_activity.py), [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloLocalActivity.java) |

## Batch Processing (Sliding Window)

|  |  |
| :---- | :---- |
| **What it does** | Processes a large stream of items in controlled, concurrent chunks. |
| **Why use it** | Manages concurrency and throughput limits while efficiently processing high volumes of data without overwhelming downstream services. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/tree/main/batch_sliding_window), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/batch/slidingwindow) |

## Custom Metrics

|  |  |
| :---- | :---- |
| **What it does** | Emits application-specific telemetry (counters, gauges, timers) from Workflows and Activities. |
| **Why use it** | Provides observability into business-level KPIs and specific Workflow performance characteristics beyond default system metrics. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/tree/main/custom_metric), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/metrics) |

## Encryption

|  |  |
| :---- | :---- |
| **What it does** | Encrypts Workflow and Activity payloads client-side using a custom Data Converter. |
| **Why use it** | Ensures sensitive data remains secure and opaque to the Temporal Server, satisfying strict compliance and privacy requirements. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/tree/main/encryption), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/encryptedpayloads) |

## Polling

|  |  |
| :---- | :---- |
| **What it does** | Periodically checks the state of an external system from within an Activity. |
| **Why use it** | Provides reliable integration with external APIs or systems that do not provide webhooks or asynchronous event notifications. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/tree/main/polling), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/polling) |

## Worker routing

|  |  |
| :---- | :---- |
| **What it does** | Dynamically routes Activities to specific Task Queues monitored by designated Workers. |
| **Why use it** | Targets tasks to specific hosts or environments; required for file-system affinity, local caching strategies, or hardware-specific (e.g., GPU) operations. |
| **Code samples** | [Python](https://github.com/temporalio/samples-python/tree/main/worker_specific_task_queues), [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/fileprocessing) |

## Task Queue Priority and Fairness

|  |  |
| :---- | :---- |
| **What it does** | Priority controls execution order within a Task Queue. Fairness controls the ratio of tasks dispatched to your Workers. |
| **Why use it** | Separate low-priority and high-priority tasks on a shared Worker pool, ensure urgent tasks execute first, and prevent large tenants from starving smaller ones in multi-tenant systems. |
| **Code samples** | [Task Queue Priority and Fairness](https://docs.temporal.io/develop/task-queue-priority-fairness) |

## Saga

|  |  |
| :---- | :---- |
| **What it does** | Manages long-running, distributed transactions by executing a sequence of steps. If a step fails, it triggers "compensating actions" (undo operations) in reverse order to revert the changes made by previous steps. |
| **Why use it** | Ensures data consistency across microservices (e.g., booking a flight, hotel, and car) without locking resources for long periods. It handles partial failures gracefully by rolling back the system to a known consistent state. |
| **Code samples** | [Java](https://github.com/temporalio/samples-java/blob/main/core/src/main/java/io/temporal/samples/hello/HelloSaga.java) |

## Early Return

|  |  |
| :---- | :---- |
| **What it does** | Uses "Update with Start" to begin a Workflow execution and synchronously return a result to the client (e.g., validation success) while continuing to process longer-running tasks (e.g., database updates, external API calls) in the background. |
| **Why use it** | Drastically reduces end-user latency in interactive applications. Users receive immediate feedback (like an "Order Received" confirmation) without waiting for the entire process to complete. |
| **Code samples** | [Java](https://github.com/temporalio/samples-java/tree/main/core/src/main/java/io/temporal/samples/earlyreturn) |

## Standalone Activity

:::warning[Pre-Release]
Standalone Activity is in pre-release and not recommended for production use. See [Standalone Activity documentation](https://docs.temporal.io/standalone-activity) for the latest status.
:::

|  |  |
| :---- | :---- |
| **What it does** | Executes a top-level Activity directly from a Client, without wrapping it in a Workflow. |
| **Why use it** | Reduces cost and latency for single-step, reliable jobs that need at-least-once execution with retries and timeouts but do not require Workflow orchestration. |
| **Code samples** | [Python](https://docs.temporal.io/develop/python/standalone-activities) |

## Additional resources

* [Temporal Design Patterns](https://taonic.github.io/temporal-design-patterns/) — A curated catalog of design patterns by a Temporal Solution Architect, covering Entity Workflow, Updatable Timer, Signal with Start, Approval, and more.
* [Temporal Code Exchange](https://temporal.io/code-exchange) — Example Temporal applications across languages and use cases.
