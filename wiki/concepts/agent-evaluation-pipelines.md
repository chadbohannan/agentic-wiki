# Agent Evaluation Pipelines

An agent evaluation pipeline runs an agent on curated tasks, scores what it did and how it did it, and gates releases on the result. This page covers exam objectives 3.1, 3.2 and 3.5. It explains the vocabulary, the grader families, trajectory and outcome scoring, how to handle non-determinism, and NVIDIA's tooling.

Pipelines score the loops described in [Refining Agent Decision-Making](refining-agent-decision-making.md). The graders they use are compared in [LLM-as-Judge](llm-as-judge.md), public task suites are covered in [Agent Benchmarks](agent-benchmarks.md), and retrieval metrics are in [RAG Evaluation and Ragas](rag-evaluation-ragas.md). Test data often comes from [Synthetic Data for Evaluation](synthetic-data-for-evaluation.md), and results feed [Agent Profiling and Result Analysis](agent-profiling-and-result-analysis.md). NVIDIA's current tooling is [NeMo Evaluator and Helix Evaluation](../components/nemo-evaluator-and-helix-eval.md). Evaluation gates feed the release process in [Agent MLOps and CI/CD](agent-mlops-cicd.md), and the production counterpart is [Continuous Regression Benchmarking](continuous-regression-benchmarking.md).

## Vocabulary

The NeMo Agent Toolkit's evaluation guide defines evaluation as running a workflow on curated test data and scoring it with quantitative metrics such as accuracy, reliability and latency **[sourced]**. Anthropic's "Demystifying Evals for AI Agents" (January 2026, read in full) gives the clearest vocabulary **[web]**:
- **Task:** one test, with inputs and success criteria.
- **Trial:** one attempt at a task. Run several, because outputs vary.
- **Grader:** scores one aspect of a trial.
- **Transcript (or trace):** the full record of tool calls and reasoning.
- **Outcome:** the final state of the environment. Whether the reservation exists matters more than the booking agent's claim that it made one.
- **Harness:** runs the tasks and aggregates the results.

The same post stresses that evaluating an agent means evaluating the model and its scaffold together **[web]**.

## Graders

Anthropic describes three grader families **[web]**:
- **Code-based graders** (string match, unit tests, state and tool-call checks) are fast, cheap and reproducible, but brittle when there is more than one valid answer.
- **Model-based graders** (rubrics, pairwise comparison, several judges) handle open-ended output, but they are non-deterministic and costlier, and need calibration against humans.
- **Human graders** are the gold standard but slow, and are used mainly to calibrate model graders.

Anthropic also advises grading outcomes rather than rigid step sequences, giving LLM judges an "Unknown" option, and allowing partial credit **[web]**.

Trajectory evaluation scores the path as well as the answer: tool selection, argument correctness, recovery from errors and loop count. The toolkit's `trajectory` evaluator gives a judge LLM the workflow's intermediate steps and the available tools, and returns a score from 0 to 1. The guide recommends 1024 `max_tokens` for the judge **[sourced]**. Ragas also lists agent metrics: Topic Adherence, Tool Call Accuracy, Tool Call F1 and Agent Goal Accuracy **[web, names only]**.

## Non-determinism

Agents vary from run to run, so a single trial says little. pass@k is the probability that at least one of k trials succeeds, which suits tasks where one good answer is enough. pass^k is the probability that all k succeed, which is what a customer-facing agent needs. The two are equal at k = 1 and diverge as k grows: a 75% per-trial success rate gives about 42% pass^3 **[web, Anthropic]**. pass^k comes from τ-bench (arXiv 2406.12045), which found GPT-4o-class agents under 50% success on its tasks and below 25% pass^8 in its retail domain **[web, abstract]**. The toolkit supports repetition with `nat eval --reps=N`, for averages and variance **[sourced]**.

## Building and running the suite

Anthropic separates two kinds of eval. Capability evals start with low pass rates and drive improvement. Regression evals should sit near 100% and catch backsliding. A capability eval that saturates graduates into the regression suite. Anthropic recommends starting with 20–50 tasks drawn from real failures, and testing both that a behaviour occurs when it should and that it doesn't when it shouldn't. A 0% pass rate across many trials usually means the task itself is broken **[web]**. Microsoft's production guide describes offline evaluation (curated sets in CI/CD) and online evaluation (live traffic, A/B and shadow tests) as a loop, in which failures found online are added to the offline set **[sourced]**.

The NeMo Agent Toolkit's evaluator, read from its `develop`-branch docs source (v1.9 era), works as follows **[sourced]**:
- **Configuration:** `nat eval --config_file …` reads an `eval` section with a dataset, named evaluators (each with a `_type`), and a `max_concurrency` setting (default 8).
- **Datasets:** json, jsonl, csv, xls, parquet or a custom format. Each entry has an id, a question and an answer.
- **Built-in evaluators:** `ragas`, `trajectory`, `tunable_rag_evaluator`, `langsmith`, `langsmith_custom` and `langsmith_judge`. Custom evaluators and dataset loaders are plugins.
- **Outputs:** `workflow_output.json`, a per-evaluator output file, and a console summary with each evaluator's average score, workflow p95 runtime and LLM p95 latency.
- **Operational flags:** `--resume` checkpoints progress, `--skip_workflow` scores answers generated earlier, and `--skip_completed_entries` and `--override` adjust a run.
- **Integrations:** Weave and LangSmith can visualise the results.

The toolkit is deprecated. Its evaluation role passes to NeMo Helix and NeMo Evaluator, which have their own page **[web]**.

LLM judges make evaluation runs slow and expensive. The toolkit's docs warn of 429 rate-limit errors and suggest lowering concurrency or hosting a NIM locally **[sourced]**. Trajectory judging is especially call-heavy **[inferred]**.

## Interview Q&A

**Why evaluate trajectories and not just final answers?** Final-answer scoring misses wrong tool choices, wasted loops and poor recovery, and it can reward lucky answers. Score tool selection, arguments and step count, and check the outcome in the environment **[web, sourced]**.

**pass@k versus pass^k?** pass@k is the chance of at least one success in k tries. pass^k is the chance that all k succeed, so use it for agents where consistency matters **[web]**.

**Capability versus regression evals?** Capability evals measure what the agent can newly do and start with low pass rates. Regression evals guard tasks it already solves and should sit near 100% **[web]**.

**How do you run an evaluation in the NeMo Agent Toolkit?** Run `nat eval` with a config holding the dataset and evaluators. It produces the workflow output, per-evaluator scores and a summary with p95 latency, and `--reps` measures variance **[sourced]**.

**How do you keep an eval suite from going stale?** Add production failures to it, read transcripts, watch for saturation, and maintain it like a unit-test suite **[web, sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit evaluate guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/evaluate.md) |
| 2026-10-28 | [AI Agents for Beginners: production (Microsoft)](https://microsoft.github.io/ai-agents-for-beginners/10-ai-agents-production/) |
| 2026-10-28 | [Demystifying Evals for AI Agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) |
| 2026-10-28 | [τ-bench (arXiv 2406.12045)](https://arxiv.org/abs/2406.12045) |
| 2026-10-28 | [Ragas available metrics](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/) |
