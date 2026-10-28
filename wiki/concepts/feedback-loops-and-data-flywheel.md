# Feedback Loops and the Data Flywheel

A feedback loop captures signals about an agent's output, from users, reviewers and automated graders, and turns them into versioned data that feeds evaluation and training. NVIDIA packages the whole cycle as a "data flywheel" on NeMo microservices. The loop is only as good as its structure: unstructured thumbs pile up, while feedback with a schema and a trace ID becomes test cases and training pairs.

## Kinds of feedback

Microsoft's production guide splits user feedback into explicit signals (ratings and comments) and implicit ones (rephrased or repeated questions, retry clicks), both sitting alongside automated judge scores **[sourced]**. Within explicit feedback, the forms differ in how much information each item carries, from least to most:
1. thumbs up or down;
2. 1–5 ratings;
3. reason codes;
4. free-text comments;
5. direct corrections, where the user edits the answer;
6. pairwise preferences between A and B, usable for DPO or RLHF;
7. expert annotation against a rubric.

Corrections and pairwise preferences are the most directly trainable. Thumbs are cheap but sparse and skewed towards extremes, implicit signals are noisy, and logged conversations fall under privacy rules. OneReach, a vendor, notes that human corrections can feed back through RLHF but that inconsistency between reviewers complicates audit **[sourced]**.

Machine feedback works the same way. The NeMo Agent Toolkit's optimiser could feed evaluators' per-item reasoning back into its prompt search, steering prompt mutation with the reasoning behind the worst-scoring items; the toolkit called this "oracle feedback" **[sourced]**. Human labels are also what keep an automated grader honest: a sample of rated items is how [LLM-as-Judge](llm-as-judge.md) gets calibrated.

## Making feedback usable

Give feedback a fixed schema (a thumb or score, a reason code, free text and a trace ID) and attach it to the trace. Microsoft's guide shows spans carrying attributes such as `user_id` and `model_version`, so each item ties to the exact version that produced it **[sourced]**. Then triage feedback into failure categories and add the failures to the offline set in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md). Two study-guide sources add colour: Confluent describes a "critical thinker" model that improves through feedback on agent outputs, and CapTech, a consultancy, stresses employee feedback and co-pilot modes during adoption, citing UPS's ORION routing system **[sourced]**.

## Risks

Closing the loop has costs:
- Feedback can entrench bias, since vocal users dominate it, so it needs the same curation and balancing as any training data in [Bias and Toxicity Mitigation](bias-and-toxicity-mitigation.md).
- Training against a proxy judge metric invites reward hacking.
- Logged conversations raise privacy and consent issues.
- Feedback can be poisoned.

The standard defence is to gate any retraining behind regression evals and human sign-off, using a release gate such as `nel gate` in [NeMo Evaluator](../components/nemo-evaluator.md), and to watch for the shifts described in [Model and Data Drift Monitoring](model-and-data-drift-monitoring.md).

## In NVIDIA's stack

NVIDIA's glossary defines the data flywheel as a six-step loop **[sourced]**:
1. Data processing.
2. Model customisation, through domain-adaptive pretraining, LoRA or SFT.
3. Evaluation, looping back through steps 1–3 until the result is satisfactory.
4. Guardrails.
5. Deployment with RAG.
6. Enterprise data refinement, in which inference logs plus human and AI feedback return to step 1.

It names NeMo Curator, Customizer (LoRA and DPO) and Evaluator as the tools. The glossary gives no concrete human-annotation workflow, so "human feedback" there is a label rather than a mechanism, and the study guide's other source for the flywheel is a marketing page.

The [NVIDIA Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md) is more specific and more cautious. It replays production logs against candidate NIMs, scores base, in-context and LoRA variants with an LLM judge, and states that "humans make the final promotion decisions" **[web]**. Its feedback is therefore mostly machine-judged log replay with a human at the promotion gate: distillation for cost rather than learning from user preferences. The blueprint has been deprecated since April 2026, and the in-context and reinforcement-learning ways an agent adapts to feedback are in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md).

## Interview Q&A

**Explicit versus implicit feedback?** Ratings and comments, versus behavioural signals such as rephrasing and retries **[sourced]**.

**Which feedback is most useful for training?** Corrections and pairwise preferences; thumbs are sparse and skewed.

**How do you turn feedback into improvement?** Link it to traces, categorise it, add the failures to the offline eval set, fix prompts, tools or models, and re-run.

**Name the steps of NVIDIA's data flywheel.** Data processing, customisation, evaluation, guardrails, deployment with RAG, and enterprise data refinement, where logs and feedback return to the start **[sourced]**.

**How do you stop a flywheel degrading quality?** Eval gates, held-out sets, human sign-off on promotion, and monitoring for drift and bias.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [AI Agents for Beginners: production (Microsoft)](https://microsoft.github.io/ai-agents-for-beginners/10-ai-agents-production/) |
| 2026-10-28 | [Powering the Next Generation of AI Agents (NVIDIA)](https://www.nvidia.com/en-us/ai/) |
| 2026-10-28 | [Data flywheel (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/data-flywheel/) |
| 2026-10-28 | [Agentic AI: top 5 challenges (Confluent)](https://www.confluent.io/blog/agentic-ai-the-top-5-challenges-and-how-to-overcome-them/) |
| 2026-10-28 | [5 common pitfalls in agentic AI adoption (CapTech)](https://www.captechconsulting.com/articles/navigating-the-challenges-5-common-pitfalls-in-agentic-ai-adoption) |
| 2026-10-28 | [Human-in-the-loop agentic AI systems (OneReach)](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/) |
| 2026-10-28 | [NeMo Agent Toolkit optimizer guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/optimizer.md) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
