# Audit Trails and Traceability

An audit trail answers four questions: who or what did what, with which inputs, under whose authority, and why it was allowed. This page covers exam objective 9.1. It sets out what to log for an agent, why propagating identity matters, and how regulation turns logging into a legal duty.

Audit records are built on the traces described in [Agent Observability and Tracing](agent-observability-tracing.md). They serve the explainability goals in [Agent Transparency and Explainability](agent-transparency-and-traceability.md) and the regulatory duties in [EU AI Act](eu-ai-act.md). Logs hold the data that [Privacy and PII Protection](privacy-and-pii-protection.md) aims to protect, so the two pages pull against each other.

## What to record

For an agent, the audit record should hold the following, all correlated by a trace ID **[inferred]**:
- the identities of the user and the agent;
- the versions of the model, prompt and policy;
- the context that was retrieved;
- every tool call, with its arguments and results;
- the verdicts of guardrails;
- human approvals.

Versioned prompts and models, as described in [Prompt and Model Versioning and Rollback](prompt-and-model-versioning-rollback.md), are what make a logged trace reproducible.

The MCP security best practices make two concrete points **[web]**:
- **Token passthrough breaks accountability.** Downstream logs show a different identity from the MCP server that forwarded the call.
- **Scope elevations should be logged** with correlation IDs.

NeMo Guardrails has a tracing section in its YAML configuration, but its options were not read **[web]**. In [NeMo Relay](../components/nemo-relay.md), metadata injectors label every event and rejection marks record blocked calls, so blocked actions are visible in the trail as well as completed ones **[web]**.

## The regulatory duty

Under Article 12 of the EU AI Act, high-risk AI systems "shall technically allow for the automatic recording of events (logs) over the lifetime of the system". The logs must support three things: identifying situations that may present a risk or a substantial modification, post-market monitoring, and monitoring of the system's operation by deployers **[web, AI Act text via artificialintelligenceact.eu]**. Article 19 requires providers to keep these logs for a period suited to the system's purpose, and for at least six months unless other EU or national law says otherwise **[web]**.

## Design points

The practical design points, synthesised **[inferred]**:
- Redact personal data before writing it to logs.
- Use tamper-evident, access-controlled storage.
- Set retention from the applicable regime, with six months as the AI Act minimum for high-risk systems.
- Weigh sampled against full logging, for cost and privacy.
- Keep logs usable for evaluation and the data flywheel.

## Interview Q&A

**What belongs in an agent's audit log?** Identities, versions, retrieved context, tool calls with their arguments and results, guardrail decisions, approvals, and a correlation ID **[inferred]**.

**Why does token passthrough hurt auditing?** Downstream logs attribute actions to the wrong identity, and the server cannot tell its clients apart **[web]**.

**Where do audit and privacy conflict?** Logs hold the sensitive data you are trying to protect. Redact, minimise, restrict access and set retention **[inferred]**.

**Which regulation mandates logging for AI?** The EU AI Act, for high-risk systems. Article 12 requires automatic logging over the system's lifetime, and Article 19 requires keeping logs for at least six months **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [MCP security best practices](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Relay middleware concepts](https://docs.nvidia.com/nemo/relay/about-nemo-relay/concepts/middleware) |
| 2026-10-28 | [EU AI Act Article 12: record-keeping](https://artificialintelligenceact.eu/article/12/) |
| 2026-10-28 | [EU AI Act Article 19: automatically generated logs](https://artificialintelligenceact.eu/article/19/) |
