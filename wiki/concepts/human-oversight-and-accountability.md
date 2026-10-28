# Human Oversight and Accountability

Meaningful oversight requires a trained, empowered human with the time, information and authority to override an agent, and it has to counter automation bias. Accountability assigns responsibility for an agent's actions to named people and processes, never to the model. This page covers exam objective 10.4.

Oversight is exercised through [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md) and the interfaces in [Agent Oversight UI Design](agent-oversight-ui-design.md). For high-risk systems in the EU it is a legal requirement under the [EU AI Act](eu-ai-act.md). The governance frameworks that organise it are in [AI Risk Frameworks and Standards](ai-risk-frameworks-and-standards.md).

## What the law requires

Article 14 of the EU AI Act sets design requirements for high-risk AI **[web, artificialintelligenceact.eu reproduction]**:
- appropriate human–machine interface tools so that people can effectively oversee the system while it is in use;
- oversight proportionate to the risk, autonomy and context.

Overseers must be able to:
- understand the system's capacities and limits;
- stay aware of automation bias ("automatically relying or over-relying on the output");
- interpret outputs correctly;
- disregard, override or reverse an output;
- stop the system with a "stop" button or similar procedure.

Paragraph 5 requires separate verification by at least two competent people for certain biometric identification uses, with exceptions. This is law for high-risk systems in the EU, not a general standard for agents. Under the 2026 Digital Omnibus it applies from 2027-12-02 for Annex III systems and from 2028-08-02 for Annex I systems **[web]**.

## Automation bias

Article 14 names automation bias explicitly, which makes it a likely interview topic. Mitigations **[inferred]**:
- show uncertainty and sources;
- for high-stakes decisions, have the reviewer form a judgment before seeing the AI's suggestion;
- audit a sample of approved items;
- measure override rates, since a near-zero rate may signal rubber-stamping rather than quality;
- rotate and train reviewers.

OneReach lists reviewer bias, skills gaps and inconsistency as challenges, and recommends empowered reviewers with override authority and dashboards **[sourced, vendor]**. HolisticAI lists the bottleneck, cost and dependency risks of human review **[sourced]**.

## Accountability

The practices **[inferred]**:
- Log who approved what and when, because human decisions are part of the audit trail described in [Audit Trails and Traceability](audit-trails-and-traceability.md).
- Define escalation paths and on-call ownership.
- Version prompts and models, as described in [Agent MLOps and CI/CD](agent-mlops-cicd.md).
- Keep a kill switch.

Automated oversight, such as guardrails, classifiers and LLM judges, scales, but it is itself a model. [NeMo Guardrails](../components/nemo-guardrails.md) enforces policy on inputs, outputs and tool execution, and should complement human review for high-stakes actions, not replace it **[inferred]**. Approval flows must also authenticate reviewers and must not accept replacement arguments supplied by the client, as the OpenAI Agents SDK documentation warns **[web]**.

## Interview Q&A

**What is automation bias, and how do you mitigate it?** Over-reliance on AI output. Mitigate it with calibrated uncertainty, independent judgment before the AI's suggestion is revealed, sampled audits, override-rate monitoring and training **[web for the definition; mitigations inferred]**.

**What must an overseer be able to do under Article 14?** Understand the system's limits, watch for automation bias, interpret outputs, override or reverse them, and stop the system **[web]**.

**Human review is a bottleneck. What do you do?** Tier by risk and confidence, automate low-risk cases with guardrails and sampled audits, and reserve humans for irreversible or ambiguous cases **[sourced for tiering; inferred]**.

**Who is accountable for an agent's action?** People and organisations (the provider, the deployer, the approver), not the model. The evidence is the log of approvals and versions **[inferred]**.

**Do guardrails replace human oversight?** No. They are automated and fallible, and work best as a first filter **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Human-in-the-loop AI (HolisticAI)](https://www.holisticai.com/blog/human-in-the-loop-ai) |
| 2026-10-28 | [Human-in-the-loop agentic AI systems (OneReach)](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/) |
| 2026-10-28 | [EU AI Act Article 14: human oversight](https://artificialintelligenceact.eu/article/14/) |
| 2026-10-28 | [AI Act Service Desk: implementation timeline](https://ai-act-service-desk.ec.europa.eu/en/ai-act/timeline/timeline-implementation-eu-ai-act) |
| 2026-10-28 | [OpenAI Agents SDK: human in the loop](https://openai.github.io/openai-agents-python/human_in_the_loop/) |
