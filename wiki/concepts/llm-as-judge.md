# LLM-as-Judge

An LLM judge scores another model's output against a rubric, a reference answer or a competing answer. It is the only scalable way to grade open-ended output, but it brings biases of its own. It is one of the three grader families in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), and most metrics in [RAG Evaluation and Ragas](rag-evaluation-ragas.md) are judge-computed, so the calibration needs described here apply to them too.

## What judges can and cannot do

Zheng et al. (arXiv 2306.05685) introduced MT-Bench and Chatbot Arena. Using them, they showed that strong judges such as GPT-4 agree with human preferences more than 80% of the time, the same level as agreement between humans. They also named the judge's biases: position bias, verbosity bias, self-enhancement bias, and limited ability to grade maths and reasoning. A later survey (arXiv 2411.15594) organises the field around consistency, bias mitigation and measuring reliability **[web]**.

Each mitigation follows from a bias. Zheng et al. tested the first three themselves **[web]**:
- Call the judge twice with the answers swapped, and count a win only if the same answer wins both times, otherwise a tie.
- Add a few example judgments: three examples raised GPT-4's position consistency from 65.0% to 77.5%.
- Grade against a reference answer or rubric rather than open preference: a reference-guided prompt cut GPT-4's failure rate on maths questions from 70% to 15%.
- Use a different model family for the judge than for the generator, against self-enhancement bias.
- Calibrate against a human-labelled sample, such as the rated items collected in [Feedback Loops and the Data Flywheel](feedback-loops-and-data-flywheel.md), and track agreement over time.

Anthropic's evals guidance adds two more: give the judge an "Unknown" option, and calibrate model graders against human graders **[web]**. The judge is itself a model whose version changes scores, so pin it as you would any dependency. Its biases can also be demographic. Ye et al. (arXiv 2410.02736) catalogue twelve judge biases, including a "diversity" bias in which judgments shift when an answer is attributed to an identity group, with most models most sensitive to female and refugee status **[web]**. That matters when judges score fairness, as [Bias and Toxicity Mitigation](bias-and-toxicity-mitigation.md) discusses.

## Trade-offs

Every judged item costs one or more model calls, so judges add cost and latency to every evaluation run. Their scores vary between runs, carry the biases above, and can drift with the generator they grade. Code graders are cheap and deterministic but brittle. Use judges for what code cannot check, and code for everything else.

## In NVIDIA's stack

The NeMo Agent Toolkit's evaluation guide wires judges in several ways **[sourced]**:
- **Ragas "NVIDIA metrics":** `AnswerAccuracy`, `ContextRelevance` and `ResponseGroundedness` are judge metrics that return a score from 0 to 1. The guide recommends `max_tokens: 8` for their judges, since only a number is returned. Their prompts cannot be configured. The guide's judge leaderboard ranks Llama-3.3-Nemotron-Super-49B first, followed by Mixtral models.
- **`tunable_rag_evaluator`:** a configurable judge prompt that by default scores coverage, correctness and relevance with adjustable weights. Its `answer` field describes the expected answer rather than giving a ground-truth string.
- **`trajectory`:** judges the intermediate steps with 1024 `max_tokens`.
- **`langsmith_judge`:** wraps the openevals library.

## Interview Q&A

**Name the known judge biases.** Position, verbosity, self-enhancement, and weak grading of hard reasoning **[web]**.

**How would you make a judge trustworthy?** Calibrate it against a human-labelled sample, track agreement, randomise answer order, use rubrics or references, and pin the judge model **[web]**.

**Why does the NeMo Agent Toolkit set `max_tokens: 8` for its Ragas judges?** The metric returns only a score, so short output is enough and is cheaper **[sourced]**.

**When would you use the tunable RAG evaluator?** When you need a custom rubric, or only have descriptions of expected answers rather than exact references **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit evaluate guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/evaluate.md) |
| 2026-10-28 | [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena (arXiv 2306.05685)](https://arxiv.org/abs/2306.05685) |
| 2026-10-28 | [A survey on LLM-as-a-judge (arXiv 2411.15594)](https://arxiv.org/abs/2411.15594) |
| 2026-10-28 | [Demystifying Evals for AI Agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) |
| 2026-10-28 | [Justice or Prejudice? Quantifying Biases in LLM-as-a-Judge (Ye et al., arXiv 2410.02736)](https://arxiv.org/abs/2410.02736) |
