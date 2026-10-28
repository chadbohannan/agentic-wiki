# LLM-as-Judge

An LLM judge scores another model's output against a rubric, a reference answer or a competing answer. It is the only scalable way to grade open-ended output, but it brings biases of its own. This page covers exam objectives 3.1 and 3.5: what judges agree with, how they fail, how to calibrate them, and how NVIDIA's tooling wires them in.

Model-based graders are one of the grader families in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md). Most metrics in [RAG Evaluation and Ragas](rag-evaluation-ragas.md) are judge-computed, so they share the calibration needs described here. Human feedback that calibrates judges is covered in [Agent Feedback Loops](agent-feedback-loops.md), and judge bias is one of the risks in [Bias and Toxicity Mitigation](bias-and-toxicity-mitigation.md).

## What judges can and cannot do

Zheng et al. (arXiv 2306.05685) introduced MT-Bench and Chatbot Arena. Using them, they showed that strong judges such as GPT-4 agree with human preferences more than 80% of the time, the same level as agreement between humans. They also named the judge's biases: position bias, verbosity bias, self-enhancement bias, and limited ability to grade maths and reasoning **[web, abstract]**. A later survey (arXiv 2411.15594) organises the field around consistency, bias mitigation and measuring reliability **[web]**.

Each mitigation follows from a bias **[inferred, standard practice]**:
- Swap the order of answers and average the scores.
- Grade against a rubric or reference answer rather than open preference.
- Use a different model family for the judge than for the generator.
- Calibrate against a human-labelled sample and track agreement over time.

Anthropic's evals guidance adds two more: give the judge an "Unknown" option, and calibrate model graders against human graders **[web]**. The judge is itself a model whose version changes scores, so pin it as you would any dependency **[inferred]**.

## In NVIDIA's tooling

The NeMo Agent Toolkit's evaluation guide wires judges in several ways **[sourced]**:
- **Ragas "NVIDIA metrics":** `AnswerAccuracy`, `ContextRelevance` and `ResponseGroundedness` are judge metrics that return a score from 0 to 1. The guide recommends `max_tokens: 8` for their judges, since only a number is returned. Their prompts cannot be configured. The guide's judge leaderboard ranks Llama-3.3-Nemotron-Super-49B first, followed by Mixtral models.
- **`tunable_rag_evaluator`:** a configurable judge prompt that by default scores coverage, correctness and relevance with adjustable weights. Its `answer` field describes the expected answer rather than giving a ground-truth string.
- **`trajectory`:** judges the intermediate steps with 1024 `max_tokens`.
- **`langsmith_judge`:** wraps the openevals library.

## Trade-offs

Every judged item costs one or more model calls, so judges add cost and latency to every evaluation run. Their scores vary between runs, carry the biases above, and can drift with the generator they grade. Code graders are cheap and deterministic but brittle. Use judges for what code cannot check, and code for everything else **[inferred]**.

## Interview Q&A

**Name the known judge biases.** Position, verbosity, self-enhancement, and weak grading of hard reasoning **[web]**.

**How would you make a judge trustworthy?** Calibrate it against a human-labelled sample, track agreement, randomise answer order, use rubrics or references, and pin the judge model **[web, inferred]**.

**Why does the NeMo Agent Toolkit set `max_tokens: 8` for its Ragas judges?** The metric returns only a score, so short output is enough and is cheaper **[sourced]**.

**When would you use the tunable RAG evaluator?** When you need a custom rubric, or only have descriptions of expected answers rather than exact references **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit evaluate guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/evaluate.md) |
| 2026-10-28 | [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena (arXiv 2306.05685)](https://arxiv.org/abs/2306.05685) |
| 2026-10-28 | [A survey on LLM-as-a-judge (arXiv 2411.15594)](https://arxiv.org/abs/2411.15594) |
| 2026-10-28 | [Demystifying Evals for AI Agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) |
