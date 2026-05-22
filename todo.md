To Add:
1. A new "🤖 AI Engineering" top-level section — not just a page, a section with sub-pages. The Decision Framework already lists AI agents, RAG, and ML inference as good Temporal use cases, but then... nothing. That's a huge gap. This section should cover:

Reference architecture — the most impactful addition. Show concretely how Temporal fits into an agentic stack: where the orchestrator workflow lives, how tool calls map to activities, how you handle LLM retries differently from regular retries (exponential backoff makes no sense when a 429 from OpenAI has a Retry-After header), and where the human-in-the-loop gate sits. A diagram would do more work than five pages of prose.
use: 
- https://temporal.io/blog/using-multi-agent-architectures-with-temporal
- https://temporal.io/blog/build-resilient-agentic-ai-with-temporal
- https://github.com/temporal-community/temporal-ai-agent

AI-specific patterns (additions to the existing Patterns page, or a sub-page):

LLM Activity with structured retries — model the LLM call as an activity, heartbeat for long streaming responses, use non-retryable errors for content policy failures vs. retryable for rate limits
Human-in-the-loop approval — signals/updates for reviewer decisions, timeout fallbacks
Prompt versioning and replay safety — the determinism rules matter differently when your "side effect" is an LLM call; explain why prompts belong in activities, not workflows
Parallel tool dispatch — fanning out tool calls from an agent loop with asyncio.gather or equivalent
Agent loop with continue-as-new — long-running agents that need history pruning
use the Temporal AI cookbook for this: https://docs.temporal.io/ai-cookbook


2. Data security and governance — you specifically mentioned this. I'd make it a sub-page of the AI section rather than a standalone page, because the audience is the same. Key content:

Payload encryption for LLM I/O: prompts and completions often contain sensitive data (PII, proprietary context). The existing Encryption pattern is generic; AI engineers need a specific call-out that prompt text is payload and should go through the custom Data Converter. use: https://temporal.io/blog/how-to-protect-sensitive-data-in-a-temporal-application

OAuth vs. service-level auth: you raised this directly. The distinction worth making is: use short-lived OAuth tokens (via activity-level credential refresh) for calls to external AI APIs that support it, rather than long-lived API keys baked into worker config. This is especially important because AI workloads often call third-party providers (OpenAI, Anthropic, etc.) that an enterprise security team scrutinizes more than internal services. For MCP tool calls, you often want this to be run under the authority of the  user
Namespace isolation for AI workloads: AI workflows that process sensitive data (healthcare, finance) should be in dedicated namespaces, not shared with general-purpose engineering. The existing Architecture page has namespace conventions but doesn't say why isolation matters for AI specifically. Mention nexus (https://docs.temporal.io/nexus) as a way to support namespace isolation but provide bridges between teams/namespaces
Audit trail framing: the existing Decision Framework mentions audit trail as a benefit, but AI engineers care about this for a specific reason — model governance and regulatory compliance (EU AI Act, etc.). A sentence or two connecting Temporal's immutable history to "you now have a complete record of every prompt, tool call, and decision your agent made" would resonate strongly.



3. Updates to existing pages (lighter lift, high value):

Decision Framework → AI/ML section: expand the three bullet points into actual depth. Right now it says "AI agents: why Temporal is perfect: long-running autonomous execution..." but doesn't explain how. Link to the new AI patterns.
Learning Paths: add an "AI Engineer" track distinct from the "Software Engineer" track. The existing path assumes you care about Sagas and payment processing; AI engineers should start with the agent loop pattern and work backward to understand why determinism matters.
FAQs: add AI-specific questions — "Can I call OpenAI from inside a Workflow?" (no, activity), "How do I handle streaming LLM responses?" (heartbeat + async completion), "What happens if my agent runs for days?" (continue-as-new).
