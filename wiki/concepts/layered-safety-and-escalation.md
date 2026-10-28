# Layered Safety and Escalation

No single safety check is reliable, so agent safety is built in layers: filters before the LLM, dialogue constraints, validation of tool calls, output checks, monitoring, and escalation to a human. This page covers exam objectives 9.4 and 9.2, including how to measure whether an agent stays in scope.

The layers are implemented with [NeMo Guardrails](../components/nemo-guardrails.md) and the models in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md). Escalation to a person follows [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md), and the pause-and-resume mechanics are in [Stateful Orchestration](stateful-orchestration.md). Whether a failing check blocks or allows an action is the fail-closed or fail-open choice discussed in [Agent Error Handling and Resilience](agent-error-handling-resilience.md).

## The layers

NeMo Guardrails formalises five stages: input, retrieval, dialog, execution and output rails. Its runtime evaluates each message against the configured rails and returns either the approved response or a configured safe alternative **[web, Guardrails architecture docs]**. A typical stack, in order **[inferred]**:
1. Cheap deterministic filters: regular expressions, allowlists, and length and rate limits.
2. Classifier rails for jailbreaks, content safety, topic control and PII.
3. Constrained dialogue flows.
4. Execution rails and least-privilege tools.
5. Output rails for safety and groundedness.
6. Monitoring and incident response.

NVIDIA's 2024 tutorial, cited by the study guide, shows two model-based checks. One is the LLM self-check variant, a policy prompt that returns Yes or No. The other is a fact-checking rail that tests whether the answer is entailed by the retrieved evidence **[sourced]**. Ragas's Topic Adherence metric gives a measurable score for scope control. It computes precision, recall and F1 over on-topic answers and refusals **[sourced]**.

## Escalation

Define the triggers: low confidence, a tripped guardrail, a high-impact action, repeated denials, or a regulated domain. Route the case to a human with the full trace, and fail closed for consequential actions **[inferred]**.

## Trade-offs

Each layer adds latency and cost, since every model-based rail is an extra inference. False positives stack up and frustrate users. Guardrails that run on the same infrastructure as the model can become a single point of failure **[inferred]**. Output rails interact badly with streaming, as [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md) notes.

## Interview Q&A

**Why layer checks rather than rely on one strong classifier?** Failure modes differ and detectors are probabilistic, so defence in depth catches what any one layer misses **[inferred]**.

**List the NeMo Guardrails stages.** Input, retrieval, dialog, execution and output **[web]**.

**When should an agent escalate to a human?** When a guardrail trips, confidence is low, an action is irreversible or high-impact, or the domain is regulated **[inferred]**.

**How do you test guardrails?** Red-team with garak, keep regression suites of both attack and benign prompts, and use Topic Adherence for scope. Track the block rate and the false-positive rate **[web, inferred]**.

**What does layering cost?** Latency, GPU cost per rail, and false positives **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Building safer LLM apps with LangChain templates and NeMo Guardrails (NVIDIA blog)](https://developer.nvidia.com/blog/building-safer-llm-apps-with-langchain-templates-and-nvidia-nemo-guardrails/) |
| 2026-10-28 | [Ragas agent metrics](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/agents/) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
