---
sidebar_position: 4
id: ai-security
title: AI Security & Governance
sidebar_label: 🔐 Security & Governance
description: Payload encryption for LLM I/O, credential management for AI providers, namespace isolation for sensitive AI workloads, and audit trail for model governance at ABC Financial.
toc_max_heading_level: 3
---

:::info
This page is part of the [Temporal Platform Hub](../intro.md).
:::

AI workloads introduce data security and governance concerns that don't appear in traditional Temporal applications. Prompts and completions often contain PII, proprietary business context, or regulated financial data. The third-party AI providers that process this data (OpenAI, Anthropic, etc.) are subject to heightened enterprise security scrutiny. This page covers how ABC Financial addresses these concerns.

For additional background, see [How to protect sensitive data in a Temporal application](https://temporal.io/blog/how-to-protect-sensitive-data-in-a-temporal-application).

---

## Payload encryption for LLM I/O

Every input to an Activity (the prompt) and every output from an Activity (the completion) is serialized as a Temporal payload and stored in Workflow history. Without encryption, these payloads are readable by anyone with access to the Temporal Cloud UI, the Temporal CLI, or the namespace's audit logs.

**At ABC Financial, all AI namespaces use a custom Data Converter** that encrypts payloads end-to-end before they leave the application process. The Temporal service stores only ciphertext — it never sees plaintext prompt content or model responses.

```python
# worker_setup.py
from temporalio.client import Client
from temporalio.converter import DataConverter
from my_org.encryption import EncryptionCodec  # your custom codec

async def create_client() -> Client:
    return await Client.connect(
        "your-namespace.tmprl.cloud:7233",
        namespace="finance-ai-prd",
        data_converter=DataConverter(
            payload_codec=EncryptionCodec(),  # encrypts all payloads in/out
        ),
    )
```

The Codec Server (deployed separately) decrypts payloads on demand for authorized users viewing the Web UI. This means:

- **Temporal Server**: sees only ciphertext
- **Temporal Cloud UI**: shows plaintext to users who authenticate through the Codec Server
- **Unauthorized users**: see encrypted blobs even if they have namespace access

See the [Encryption pattern](../patterns.md#encryption) for the Data Converter implementation and [Architecture](../architecture.md) for ABC Financial's Codec Server deployment.

:::warning
Without payload encryption, every prompt you send to an LLM — including any PII or proprietary context injected into that prompt — is stored in plaintext in Workflow history. Enable encryption before using AI namespaces with customer data.
:::

---

## Credential management for AI provider APIs

### The wrong way: long-lived API keys in Worker config

Long-lived API keys (e.g., `OPENAI_API_KEY`) baked into Worker environment variables are the most common credential anti-pattern in AI workloads:

- Keys are often over-scoped (full account access rather than per-project)
- Rotation is disruptive (requires Worker restart or redeployment)
- Keys can be leaked through logs, crash dumps, or over-broad IAM access
- They cannot be scoped to the identity of the *user* who initiated the request

### The right way: short-lived credentials fetched per Activity

Fetch credentials inside the Activity from a secrets manager, and prefer short-lived tokens where the provider supports them:

```python
import boto3
import json
from temporalio import activity

@activity.defn
async def call_llm(messages: list[dict]) -> str:
    # Fetch a fresh API key from AWS Secrets Manager on each Activity execution.
    # The key is never stored in Workflow history (it's fetched inside the activity,
    # not passed as an Activity argument).
    secret = boto3.client("secretsmanager").get_secret_value(
        SecretId="prod/ai/openai-api-key"
    )
    api_key = json.loads(secret["SecretString"])["api_key"]

    from openai import AsyncOpenAI
    client = AsyncOpenAI(api_key=api_key)
    response = await client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
    )
    return response.choices[0].message.content
```

:::caution
Never pass API keys as Workflow or Activity *arguments*. Arguments are serialized as Temporal payloads and stored in Workflow history. Even with payload encryption, minimize how much credential material flows through Temporal's data plane.
:::

### MCP tool calls under user authority

When AI agents invoke [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) tools that act on behalf of a specific user — accessing their calendar, email, or internal systems — the tool call should run under that user's identity, not a shared service account.

Implement this by passing a user-scoped OAuth token (fetched from your identity provider) into the Activity:

```python
from dataclasses import dataclass
from temporalio import activity, workflow

@dataclass
class MCPToolCall:
    tool_name: str
    arguments: dict
    user_id: str  # The user on whose behalf the agent is acting

@activity.defn
async def execute_mcp_tool(tool_call: MCPToolCall) -> str:
    # Exchange the user ID for a short-lived, scoped OAuth token
    token = await fetch_user_token(
        user_id=tool_call.user_id,
        scopes=["calendar.read", "email.send"],
    )
    return await call_mcp_server(
        tool=tool_call.tool_name,
        args=tool_call.arguments,
        auth_token=token,
    )
```

This ensures that:
- The action is traceable to the specific user in enterprise audit logs
- The token scope is limited to what the tool actually needs
- Tokens expire automatically — no long-lived credentials accumulate

---

## Namespace isolation for AI workloads

At ABC Financial, AI workloads that process sensitive data run in **dedicated namespaces** rather than shared general-purpose namespaces. This provides:

- **Separate encryption keys**: AI namespace encryption material is isolated from other workloads
- **Independent access controls**: only AI team members and the Codec Server can access AI namespace data
- **Separate audit logs**: AI Workflow histories are auditable independently of unrelated workloads
- **Blast radius containment**: a misconfigured AI workflow cannot affect payments or other critical domains

### Recommended namespace structure for AI

| Namespace | Purpose |
| :---- | :---- |
| `finance-ai-dev` | AI development and experimentation |
| `finance-ai-stg` | AI staging — validated against staging LLM endpoints |
| `finance-ai-prd` | Production AI workloads with customer data |
| `finance-ai-research` | Long-running research agents — isolated from production |

AI namespaces should have all the same guardrails as other production namespaces (deletion protection, private connectivity, payload encryption, audit logs). See [Architecture](../architecture.md) for the full environment feature matrix.

### Cross-namespace coordination with Nexus

When AI agents need to invoke capabilities owned by other teams — for example, an AI agent in `finance-ai-prd` that needs to trigger a payment workflow in `finance-payments-prd` — use [Temporal Nexus](https://docs.temporal.io/nexus) rather than direct cross-namespace calls.

Nexus lets the payments team expose a clean service contract (a Nexus Operation) that the AI team can call without needing access to the payments namespace:

```
finance-ai-prd                         finance-payments-prd
──────────────────────                 ──────────────────────
AgentWorkflow                          PaymentWorkflow
  │                                      │
  │── Nexus Operation ─────────────────► │
  │   (payments-service/initiate)        │
  │                                      │
  │◄─ Nexus Result ─────────────────────  │
```

Benefits:
- The AI team never has credentials or access to the payments namespace
- The payments team can change their Workflow internals without the AI team needing to update anything
- Each Nexus call is a durable, observable operation with its own retry and timeout controls

See [Temporal Nexus documentation](https://docs.temporal.io/nexus) for setup instructions.

---

## Audit trail for model governance

Temporal's immutable Workflow history is a built-in audit trail for every decision an AI agent makes. Each event in the history records:

- The exact inputs to every LLM call (the prompt, messages, model name)
- The exact output from every LLM call (the completion text)
- Every tool invocation (name, arguments, result)
- Every human approval decision (who approved, timestamp, reason)
- Every retry and failure (with error type and message)

This matters specifically for AI workloads because:

**Model governance**: when a model produces an unexpected or harmful output, you can look up the exact prompt that produced it, the model version used, and the full message history leading to that decision.

**Regulatory compliance**: the [EU AI Act](https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai) (Articles 12 and 17) requires high-risk AI systems to maintain logs sufficient to trace the system's decisions. Temporal's history satisfies this requirement without custom logging infrastructure.

**Incident response**: when an agent takes an unintended action, the Workflow history is the first place to look. You can replay the exact execution that caused the incident in the Temporal Web UI or via the CLI.

### Enabling long-term history retention for AI namespaces

For regulatory compliance, configure [Workflow History Export](https://docs.temporal.io/cloud/export) on AI production namespaces to export closed Workflow histories to long-term storage (S3, GCS) before they expire:

```
Temporal Cloud AI Namespace
  │
  └─► Workflow History Export
        │
        └─► S3 bucket (finance-temporal-audit-logs)
              └─► Retention: 7 years (configurable per compliance requirement)
```

Contact the Temporal Platform team to enable History Export on your AI namespace.
