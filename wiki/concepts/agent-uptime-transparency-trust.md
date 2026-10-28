# Agent Uptime, Transparency and Trust

Keeping an agent available, and earning the trust of users and auditors, takes SLOs and alerting, graceful degradation, audit trails, disclosure of AI use and its limits, human oversight, and telemetry that respects privacy. This page covers exam objective 8.5. It is the thinnest objective in the study guide's sources, so most of the page is synthesis that links to fuller treatments elsewhere.

Uptime depends on the capacity and failover practices in [Inference Cost and Availability](inference-cost-and-availability.md) and the fallbacks in [Agent Error Handling and Resilience](agent-error-handling-resilience.md). Transparency to users is developed in [Agent Transparency and Explainability](agent-transparency-and-traceability.md) and [Human Oversight and Accountability](human-oversight-and-accountability.md), and the regulatory side in [EU AI Act](eu-ai-act.md).

## Uptime

The elements, mostly synthesised **[web, LangSmith alerts; otherwise inferred]**:
- SLOs and error budgets, set as described in [LLM Reliability Metrics and SLOs](llm-reliability-metrics-slos.md);
- health checks, retries with backoff and circuit breakers;
- fallback models and graceful degradation;
- failover across regions or providers;
- alerts that actually reach on-call staff. LangSmith routes alerts to PagerDuty and Slack.

## Transparency and audit

IBM's agent-evaluation explainer says evaluation should cover trustworthiness, policy compliance and bias. It measures policy-adherence rate, fairness scores and customer satisfaction **[sourced]**. Traces and event streams double as audit evidence. In [NeMo Relay](../components/nemo-relay.md), several features support this **[web]**:
- metadata injectors label every event for audit;
- rejection marks record blocked actions;
- sanitisers minimise sensitive data in telemetry;
- standard credential headers are stripped from the event copy.

The Data Flywheel Blueprint's README insists on human oversight, and places production security (authentication, authorisation, monitoring and trust boundaries) on the deployer **[web]**. Transparency to users is designed in the interface, as [Human-Agent Interaction UIs](human-agent-interaction-uis.md) describes: disclosing that the user is talking to an AI, citing sources, and explaining tool actions **[inferred]**. The record-keeping duties are in [Audit Trails and Traceability](audit-trails-and-traceability.md).

No primary source was read on public status pages, model cards for deployed agents, or incident post-mortem practice, so those topics are not covered.

## Interview Q&A

**How do traces support trust and compliance?** They give a replayable, attributable record of actions and tool calls. With sanitisation and metadata, they provide audit evidence without exposing personal data **[inferred, web]**.

**How do you keep an LLM agent up?** SLOs with alerting, retries with backoff and circuit breakers, fallback models, graceful degradation, and capacity-aware scaling **[inferred; sourced for backoff]**.

**What does fail-closed mean for governance middleware?** If a policy guardrail or intercept errors, the action is blocked rather than allowed **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [LangSmith alerts](https://docs.langchain.com/langsmith/alerts) |
| 2026-10-28 | [NeMo Relay middleware concepts](https://docs.nvidia.com/nemo/relay/about-nemo-relay/concepts/middleware) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
