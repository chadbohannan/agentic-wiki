# NeMo Evaluator

NeMo Evaluator is NVIDIA's benchmark-evaluation framework, and the name has covered three things since the September 2025 study guide: the NeMo Evaluator microservice the guide assumes, an Evaluator plugin inside [NeMo Helix](nemo-helix.md) that the v0.8 Helix docs renamed Helix Evals, and an open-source framework with the `nel` CLI, described below. The evaluation pipeline all of them implement is described tool-neutrally in [Agent Evaluation Pipelines](../concepts/agent-evaluation-pipelines.md), and their place in the product lineage is in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md).

## Before Evaluator: the toolkit's evaluation

The study guide's evaluation tooling is the [NeMo Agent Toolkit](nemo-agent-toolkit.md): `nat eval`, `nat optimize`, the profiler and the sizing calculator. [Agent Profiling and Result Analysis](../concepts/agent-profiling-and-result-analysis.md) explains how to read the profiler's output, and [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md) covers the optimiser and test-time compute. The toolkit is now deprecated, and its README names NeMo Helix as the successor for evaluation and optimisation. RAG metrics in that generation came from Ragas through the toolkit, as [RAG Evaluation and Ragas](../concepts/rag-evaluation-ragas.md) explains **[sourced]**.

## The framework

The nightly docs describe NeMo Evaluator as an LLM evaluation framework with 17 built-in benchmarks **[web]**:
- **Benchmarks** include MMLU, MMLU-Pro, MATH-500, GPQA, GSM8K, HumanEval, HealthBench and XSTest. It can also resolve tasks from NeMo Skills, lm-eval and VLMEvalKit.
- **Solvers** are chosen per benchmark: `simple`, `harbor`, `tool_calling`, `gym_delegation` and `openclaw`.
- **Backends** are local, Docker and SLURM, with automatic model deployment.

NGC also packages τ²-bench as an Evaluator container **[web, snippet]**.

## Comparing runs and gating releases

`nel compare` diagnoses what changed between two runs. It uses McNemar's exact test on paired items, paired flip analysis and confidence intervals, so it can say whether a drop is statistically meaningful rather than noise. `nel gate` turns benchmark thresholds into a GO, NO-GO or INCONCLUSIVE release decision, which is the CI gate described in [Agent Release Management](../concepts/agent-release-management.md) **[web]**. The INCONCLUSIVE outcome matters: a gate that only answers pass or fail forces a decision when the sample is too small to support one.

## Relation to Helix

Helix's evaluation service implements the same pipeline as a platform service with agent runners (Fabric, Gym, Harbor), and the Helix docs call it the Evaluator plugin renamed **[web]**. The framework's landing page does not mention Helix or the older microservice, so whether `nel` and Helix Evals share code is not established. For an interview, the safe framing is that `nel` is the library for benchmarking models and Helix Evals is the platform service for evaluating agents in place.

## Interview Q&A

**What replaces the NeMo Agent Toolkit's evaluation and optimiser?** [NeMo Helix](nemo-helix.md), a CLI-first control plane for evaluation, optimisation and hardening. NeMo Relay takes over observability, and NeMo Fabric provides the agent-harness interface **[sourced]**.

**What do `nel compare` and `nel gate` do?** `compare` tests whether two runs differ significantly, using McNemar's test on paired items. `gate` turns thresholds into a GO, NO-GO or INCONCLUSIVE release decision **[web]**.

**Why McNemar's test rather than comparing two accuracy numbers?** Both runs score the same items, so the informative data are the items that flipped between them; McNemar's test uses exactly those discordant pairs **[web]**.

**Why have an INCONCLUSIVE gate outcome?** A pass-or-fail gate forces a release decision even when the sample is too small to tell the versions apart. INCONCLUSIVE says to collect more data rather than ship or block on noise.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA NeMo Agent Toolkit (GitHub)](https://github.com/NVIDIA/NeMo-Agent-Toolkit) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
| 2026-10-28 | [NeMo Helix: upgrading from NeMo Evaluator](https://docs.nvidia.com/nemo-helix/v0.8/documentation/evaluate-models/upgrading-from-evaluator) |
