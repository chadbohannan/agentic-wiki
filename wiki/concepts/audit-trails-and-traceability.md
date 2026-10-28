# Audit Trails and Traceability

An audit trail answers four questions about any agent action: who or what did it, with which inputs, under whose authority, and why it was allowed. Traceability is the property that lets you reconstruct that answer after the fact, and it is the one layer of agent transparency that can actually be verified; the harder questions of why a model chose what it did are the subject of [Agent Transparency and Explainability](agent-transparency-and-explainability.md).

## What to record

An audit record is a trace from [Agent Observability and Tracing](agent-observability-tracing.md) kept for accountability rather than debugging. For an agent it should hold the following, all correlated by a trace ID:
- the identities of the user and the agent;
- the versions of the model, prompt and policy;
- the context that was retrieved;
- every tool call, with its arguments and results;
- the verdicts of guardrails, including blocked actions;
- human approvals and overrides.

The versions matter because a logged trace is only reproducible if the deployed combination can be rebuilt, which is the versioning discipline set out in [Agent Release Management](agent-release-management.md). Graph runtimes add replayable state on top: LangGraph's checkpoints let an auditor step through a run, as [Stateful Orchestration](stateful-orchestration.md) describes. The MCP tools specification says clients should log tool usage for audit **[web]**, and OneReach, a vendor, recommends logging every AI-driven decision for later analysis and regulatory reporting **[sourced]**.

Identity is the part most often lost. The MCP security best practices warn that token passthrough breaks accountability, because downstream logs then show a different identity from the MCP server that forwarded the call, and they ask for scope elevations to be logged with correlation IDs **[web]**.

## The regulatory duty

Under Article 12 of the EU AI Act, high-risk AI systems "shall technically allow for the automatic recording of events (logs) over the lifetime of the system". The logs must support identifying situations that may present a risk or a substantial modification, post-market monitoring, and monitoring by deployers. Article 19 requires providers to keep these logs for a period suited to the system's purpose, and for at least six months unless other EU or national law says otherwise **[web]**. The wider obligations are on the [EU AI Act](eu-ai-act.md) page.

## Design points

Logs hold exactly the data that [Privacy and PII Protection](privacy-and-pii-protection.md) tries to keep out of circulation, so audit and privacy pull against each other. The usual resolution:
- Redact personal data before writing it to logs.
- Use tamper-evident, access-controlled storage.
- Set retention from the applicable regime, with six months as the AI Act minimum for high-risk systems.
- Weigh sampled against full logging, for cost and privacy.
- Keep logs usable for evaluation and the [data flywheel](feedback-loops-and-data-flywheel.md).

## In NVIDIA's stack

The NeMo Agent Toolkit's exporters (Phoenix, Langfuse, LangSmith, Weave, OpenTelemetry) were its tracing route before its deprecation; [NeMo Relay](../components/nemo-relay.md) now fills that role and adds features aimed at audit **[web]**. Metadata injectors label every event, rejection marks record blocked calls so that refused actions appear in the trail as well as completed ones, sanitisers minimise sensitive data in telemetry, and standard credential headers are stripped from the event copy. Relay's governance middleware is fail-closed: if a policy intercept errors, the action is blocked rather than allowed. NeMo Guardrails also has a tracing section in its YAML configuration **[web]**.

## Interview Q&A

**What belongs in an agent's audit log?** Identities, model, prompt and policy versions, retrieved context, tool calls with arguments and results, guardrail decisions, approvals, timestamps and a correlation ID.

**Traceability versus explainability?** Traceability reconstructs what happened from logs, traces, versions and approvals, and can be verified. Explainability tries to say why, and for LLMs is only partly achievable.

**Why does token passthrough hurt auditing?** Downstream logs attribute actions to the wrong identity, and the server cannot tell its clients apart **[web]**.

**How do traces support compliance without leaking data?** Sanitise and label events at capture, so the trail is attributable and replayable without exposing personal data **[web]**.

**Which regulation mandates logging for AI?** The EU AI Act, for high-risk systems: Article 12 requires automatic logging over the system's lifetime, and Article 19 retention of at least six months **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Human-in-the-loop agentic AI systems (OneReach)](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-28 | [MCP security best practices](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Relay middleware concepts](https://docs.nvidia.com/nemo/relay/about-nemo-relay/concepts/middleware) |
| 2026-10-28 | [EU AI Act Article 12: record-keeping](https://artificialintelligenceact.eu/article/12/) |
| 2026-10-28 | [EU AI Act Article 19: automatically generated logs](https://artificialintelligenceact.eu/article/19/) |
