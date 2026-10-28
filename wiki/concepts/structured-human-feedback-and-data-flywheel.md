# Structured Human Feedback and the Data Flywheel

Structured feedback turns human reactions (ratings, corrections, preferences and annotations) into versioned data that feeds evaluation and training. NVIDIA packages this as a "data flywheel" built on NeMo microservices. This page covers exam objective 10.2: the kinds of feedback, NVIDIA's flywheel in its glossary form and in the blueprint, and the risks of closing the loop.

The page extends [Agent Feedback Loops](agent-feedback-loops.md), which covers explicit and implicit signals and linking feedback to traces. Collected feedback calibrates [LLM-as-Judge](llm-as-judge.md) and feeds the retraining in the [NVIDIA Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md). The in-context and RL counterparts are in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md).

## Kinds of feedback

Ordered roughly by how much information each item carries, from least to most **[inferred]**:
1. thumbs up or down;
2. 1–5 ratings;
3. reason codes;
4. free-text comments;
5. direct corrections, where the user edits the answer;
6. pairwise preferences between A and B, usable for DPO or RLHF;
7. expert annotation against a rubric.

Corrections and pairwise preferences are the most directly trainable. Thumbs are cheap, but sparse and skewed. OneReach notes that human corrections can feed back through methods such as RLHF, but that inconsistency between reviewers complicates audit **[sourced, vendor]**.

## NVIDIA's flywheel

NVIDIA's glossary defines the data flywheel as a self-improving loop of six steps **[sourced]**:
1. Data processing.
2. Model customisation, through domain-adaptive pretraining, LoRA or SFT.
3. Evaluation, looping back through steps 1–3 until the result is satisfactory.
4. Guardrails.
5. Deployment with RAG.
6. Enterprise data refinement, in which inference logs plus human and AI feedback return to step 1.

It names NeMo Curator, Customizer (LoRA and DPO) and Evaluator as the tools. The glossary gives no concrete human-annotation workflow, so "human feedback" there is a label rather than a mechanism.

The Data Flywheel Blueprint is more specific and more cautious, according to its README, read 2026-10-28 **[web]**:
- It ingests production prompt and response logs from Elasticsearch. Each record needs a timestamp, a `workload_id`, a `client_id`, the request and the response.
- It builds evaluation and training splits.
- It runs base, in-context-learning and LoRA experiments for each candidate NIM, and scores them with an LLM judge.
- "Humans make the final promotion decisions."

Its aim is to replace a large model with a smaller one of similar quality. The README has carried a deprecation notice since April 2026, and names no successor. In the blueprint, then, the feedback is mostly log replay judged by a machine, with the human at the promotion gate. It is distillation for cost rather than learning from user preferences, and the human-in-the-loop sits at the decision to promote a model **[inferred]**.

## Risks

The risks of closing the loop **[inferred]**:
- Feedback loops can entrench bias, since vocal users dominate the feedback.
- Training against a proxy judge metric invites reward hacking.
- Logged conversations raise privacy and consent issues.
- Feedback can be poisoned.

Gate any retraining behind regression evals, as described in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md). Current NVIDIA evaluation tooling is in [NeMo Evaluator and Helix Evaluation](../components/nemo-evaluator-and-helix-eval.md).

## Interview Q&A

**Name the steps of NVIDIA's data flywheel.** Data processing, customisation, evaluation, guardrails, deployment with RAG, and enterprise data refinement, where logs and human and AI feedback return to the start **[sourced]**.

**What does the Data Flywheel Blueprint do?** It replays production logs against candidate NIMs, runs base, in-context and LoRA experiments, scores them with an LLM judge, and leaves promotion to humans. The goal is a cheaper model of similar quality. It has been deprecated since April 2026 **[web]**.

**Which feedback is most useful for training?** Corrections and pairwise preferences, rather than thumbs, which are sparse and biased **[inferred]**.

**How do you stop a flywheel degrading quality?** Eval gates, held-out sets, human sign-off on promotion, and monitoring for drift and bias **[web, inferred]**.

**Which tools does NeMo provide for it?** Curator, Customizer (LoRA and DPO), Evaluator and Guardrails, with NIM for deployment **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Data flywheel (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/data-flywheel/) |
| 2026-10-28 | [Human-in-the-loop agentic AI systems (OneReach)](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
