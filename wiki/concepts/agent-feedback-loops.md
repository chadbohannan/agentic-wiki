# Agent Feedback Loops

A feedback loop captures signals about an agent's output, from users and from automated graders, and routes them into evaluation and improvement. This page covers exam objectives 3.3 and 3.5: the kinds of feedback, how to structure them, and their limits.

Feedback complements the automated scores in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md) and calibrates [LLM-as-Judge](llm-as-judge.md). It drives the retraining described in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md). Collecting structured feedback from human reviewers and NVIDIA's data flywheel are covered in [Structured Human Feedback and the Data Flywheel](structured-human-feedback-and-data-flywheel.md).

## Kinds of feedback

Microsoft's production guide distinguishes two kinds of user feedback. Explicit feedback is ratings and comments. Implicit signals include rephrased or repeated questions and retry clicks. Both sit alongside automated judge scores. Trace spans can carry attributes such as `user_id` and `model_version`, so each piece of feedback ties to a specific version **[sourced]**.

Machine feedback works the same way. The NeMo Agent Toolkit's optimiser can feed evaluators' per-item reasoning back into its prompt search. The toolkit calls this "oracle feedback": it steers prompt mutation using the reasoning behind the worst-scoring items **[sourced, toolkit optimiser docs]**.

## Making feedback usable

Feedback is only useful if it is structured. Give it a fixed schema (a thumb or score, a reason code, free text and a trace ID) and link it to the trace. Then triage it into failure categories and add the failures to the offline evaluation set **[inferred]**. NVIDIA frames the whole cycle as a "data flywheel" in which human and AI feedback refine the models, though the study guide's source for this is a marketing page **[sourced]**. Two study-guide sources add colour. Confluent describes a "critical thinker" model that improves through feedback on agent outputs **[sourced]**. CapTech stresses employee feedback and co-pilot modes during adoption, citing UPS's ORION routing system **[sourced, consultancy claim]**.

Each signal has limits. Explicit feedback is sparse and skewed towards extremes, implicit signals are noisy, and logged conversations fall under privacy rules **[inferred]**.

## Interview Q&A

**Explicit versus implicit feedback?** Ratings and comments, versus behavioural signals such as rephrasing and retries **[sourced]**.

**How do you turn feedback into improvement?** Link it to traces, categorise it, add the failures to the offline eval set, fix prompts, tools or models, and re-run **[inferred, sourced]**.

**What is a data flywheel?** A loop in which usage generates feedback that improves the agent, which in turn attracts more usage **[sourced]**.

**What is wrong with relying on ratings alone?** They are sparse and skewed, so combine them with implicit and judge signals **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [AI Agents for Beginners: production (Microsoft)](https://microsoft.github.io/ai-agents-for-beginners/10-ai-agents-production/) |
| 2026-10-28 | [Powering the Next Generation of AI Agents (NVIDIA)](https://www.nvidia.com/en-us/ai/) |
| 2026-10-28 | [Agentic AI: top 5 challenges (Confluent)](https://www.confluent.io/blog/agentic-ai-the-top-5-challenges-and-how-to-overcome-them/) |
| 2026-10-28 | [5 common pitfalls in agentic AI adoption (CapTech)](https://www.captechconsulting.com/articles/navigating-the-challenges-5-common-pitfalls-in-agentic-ai-adoption) |
| 2026-10-28 | [NeMo Agent Toolkit optimizer guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/optimizer.md) |
