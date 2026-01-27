---
sidebar_position: 8
id: shared-responsibility
title: Shared Responsibility Model
sidebar_label: 🤝 Shared Responsibility
description: Defining team responsibilities for building and managing Temporal applications between Platform and Application teams.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](./intro.md).
:::

:::note
Tailor this matrix to clarify ownership boundaries so developers know who to contact.
:::

At ABC Financial, the ownership of Temporal applications is shared between the **Temporal Platform Team** (who manages Temporal Cloud infrastructure) and **Application Teams** (who build and run Temporal Workflows).

*Key: ✅= responsible, ❌= not responsible, 🤝🏼= shared responsibility*

### Identity Access Management (IAM)

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Temporal Cloud access ([go/temporal-request](http://go/temporal-request)) | ✅ | ❌ |
| [SAML](https://docs.temporal.io/cloud/saml) and [SCIM](https://docs.temporal.io/cloud/scim) configurations | ✅ | ❌ |
| Temporal Cloud [user groups](https://docs.temporal.io/cloud/user-groups) | ✅ | ❌ |
| User principal provisioning and de-provisioning | ✅ | ❌ |
| [User principal role](https://docs.temporal.io/cloud/users) assignment | ✅ | ❌ |
| [API key](https://docs.temporal.io/cloud/api-keys) provisioning | ✅ | ❌ |

### Network Connectivity

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| [Private Connectivity](https://docs.temporal.io/cloud/connectivity) to Temporal Cloud | ✅ | ❌ |
| Firewall rules to Temporal Cloud | ✅ | ❌ |

### Data Security

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Data compliance policy | ✅ | ❌ |
| [Data Converter](https://docs.temporal.io/evaluate/development-production-features/data-encryption) implementation | ✅ | ❌ |
| [Data Converter](https://docs.temporal.io/evaluate/development-production-features/data-encryption) usage | ❌ | ✅ |
| [Codec Server](https://docs.temporal.io/production-deployment/data-encryption) hosting | ✅ | ❌ |
| [Codec Server](https://docs.temporal.io/production-deployment/data-encryption) configuration (per Namespace) | ❌ | ✅ |

### Infrastructure

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Temporal Cloud Namespace provisioning ([go/temporal-namespace](http://go/temporal-namespace)) | ✅ | ❌ |
| [Temporal Cloud metrics](https://docs.temporal.io/production-deployment/cloud/metrics/reference) | ✅ | ❌ |
| Temporal Cloud [Namespace rate limits](https://docs.temporal.io/cloud/limits#namespace-level) | ❌ | ✅ |
| Temporal Cloud [Namespace Capacity](https://docs.temporal.io/cloud/capacity-modes) | ❌ | ✅ |
| [Temporal Cloud audit logs](https://docs.temporal.io/cloud/audit-logs) | ✅ | ❌ |

### Governance

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Temporal Platform Hub | ✅ | ❌ |
| [Temporal developer guide](#) | ✅ | ❌ |

### Development

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Workflow development | ❌ | ✅ |
| Automated tests (i.e. unit, integration, [replay](https://docs.temporal.io/develop/java/testing-suite#replay)) | ❌ | ✅ |
| Workflow versioning | ❌ | ✅ |

### Worker

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Worker identity authentication policy | ✅ | ❌ |
| Worker identity auth implementation | ❌ | ✅ |
| Worker identity auth rotation | ✅ | ❌ |
| Worker infrastructure health (e.g. Kubernetes health) | ✅ | ❌ |
| Worker deployment health | ❌ | ✅ |
| Worker configurations (i.e. Task Queue, Execution Slots) | 🤝🏼 (defaults) | 🤝🏼 (customization) |
| Worker auto-scaling framework (i.e. KEDA) | ✅ | ❌ |
| Worker auto-scaling configuration | ❌ | ✅ |

### Temporal Application Deployment

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Build pipeline for Worker | ✅ | ❌ |
| Artifact management | ✅ | ❌ |
| Workflow versioning management (e.g. [Worker Versioning](https://docs.temporal.io/production-deployment/worker-deployments/worker-versioning)) policy | ✅ | ❌ |
| Worker build (i.e. Workflow and Worker Definition) | ❌ | ✅ |
| Worker build release (i.e. control which build to release and when) | ✅ | ❌ |

### Observability

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Observability platform (e.g. Datadog, Dynatrace) | ✅ | ❌ |
| [Temporal SDK metrics](https://docs.temporal.io/references/sdk-metrics) collection | ✅ | ❌ |
| [Temporal SDK metrics](https://docs.temporal.io/references/sdk-metrics) configuration | ❌ | ✅ |
| Temporal custom metrics emission | ❌ | ✅ |
| [Temporal Cloud metrics](https://docs.temporal.io/cloud/metrics/openmetrics) collection | ✅ | ❌ |
| Monitoring dashboard ([go/temporal-dashboard](http://go/temporal-dashboard)) | ✅ | ❌ |
| Temporal Cloud platform alerts | ✅ | ❌ |
| Temporal Workflow alerts | ❌ | ✅ |

### Operation

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Support coordination with Temporal (the company) | ✅ | ❌ |
| Load testing | ❌ | ✅ |
| Incident response | 🤝🏼 (platform incident) | 🤝🏼 (application incident) |

### Cost

| Responsibility | Platform Team | Application Team |
| :---- | ----- | ----- |
| Temporal Cloud platform cost | ✅ | ❌ |
| Temporal Cloud Namespace cost | ❌ | ✅ |

## Decision framework

When in doubt, ask yourself:

* **Does the issue affect multiple teams or namespaces?** → Platform Team
* **Is it business logic or application-specific?** → Application Team
* **Does it require Temporal Cloud `Admin` access?** → Platform Team
