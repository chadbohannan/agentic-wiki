# NeMo Evaluator and Helix Evaluation

NVIDIA's evaluation and optimisation tooling changed generations in 2026, and this page maps what exists as of 2026-10-28. The September 2025 study guide assumes the NeMo Agent Toolkit's `nat eval` and the NeMo Evaluator microservice. Today the toolkit is deprecated and its evaluation role passes to NeMo Helix and a NeMo Evaluator framework. The page touches exam objectives 3.1, 3.2 and 3.4. For an interview, know both generations **[inferred]**.

These tools run the pipelines described in [Agent Evaluation Pipelines](../concepts/agent-evaluation-pipelines.md), and they take over from the deprecated [NeMo Agent Toolkit](nemo-agent-toolkit.md). Its optimiser and test-time compute features are described in [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md), and its profiler in [Agent Profiling and Result Analysis](../concepts/agent-profiling-and-result-analysis.md).

## The products

**NeMo Agent Toolkit evaluation** covers `nat eval`, `nat optimize`, the profiler and the sizing calculator. The toolkit's GitHub README marks it deprecated as of October 2026, with end of life at the end of 2026, and names NeMo Helix as the successor for evaluation and optimisation **[sourced]**.

**NeMo Helix** is an open-source control plane for improving and hardening agents. It has a REST API, a Python SDK and a CLI, which is the primary interface, plus an alpha UI called NeMo Studio. Its features fall into three groups **[web, Helix v0.8 home page]**:
- **Evaluation:** LLM-judge, deterministic, agentic and RAG benchmarks, with regression suites backed by Harbor.
- **Optimisation:** skill, prompt and hyperparameter tuning, plus routing through Switchyard.
- **Hardening:** guardrails, garak red-teaming and an anonymiser.

It runs legacy LangGraph agents built on the toolkit alongside its own `agent.yaml` agents. Its first public release, 0.1.0, is dated 2026-05-20, and the docs list 0.8.0 as the newest version, so the product is young and moving fast **[web, release notes via search summary]**.

**NeMo Evaluator** (nightly docs, read 2026-10-28) is described as an LLM evaluation framework with 17 built-in benchmarks **[web, landing page]**:
- **Benchmarks** include MMLU, MMLU-Pro, MATH-500, GPQA, GSM8K, HumanEval, HealthBench and XSTest. It can also resolve tasks from NeMo Skills, lm-eval and VLMEvalKit.
- **Solvers** are chosen per benchmark: `simple`, `harbor`, `tool_calling`, `gym_delegation` and `openclaw`.
- **Backends** are local, Docker and SLURM, with automatic model deployment.

Its `nel compare` command diagnoses what changed between two runs, using McNemar's exact test, paired flip analysis and confidence intervals. `nel gate` turns benchmark thresholds into a GO, NO-GO or INCONCLUSIVE release decision **[web]**. NGC also packages τ²-bench as an Evaluator container **[web, snippet]**. The landing page does not mention Helix or the older microservice, so how the three relate is not established.

**NeMo Switchyard** routes each agent step to a suitable model. It has a `libsy` SDK and a gateway compatible with the OpenAI, Anthropic and Responses APIs, and it is described on the tuning page **[web, NVIDIA blog]**. **Data Designer** generates synthetic data and is described in [Synthetic Data for Evaluation](../concepts/synthetic-data-for-evaluation.md). RAG metrics come from Ragas through the toolkit, as [RAG Evaluation and Ragas](../concepts/rag-evaluation-ragas.md) explains **[sourced]**.

## Interview Q&A

**What replaces the NeMo Agent Toolkit's evaluation and optimiser?** NeMo Helix, a CLI-first control plane for evaluation, optimisation and hardening. NeMo Relay takes over observability, and NeMo Fabric provides the agent-harness interface **[sourced]**.

**What do `nel compare` and `nel gate` do?** `compare` tests whether two runs differ significantly, using McNemar's test on paired items. `gate` turns thresholds into a GO, NO-GO or INCONCLUSIVE release decision **[web]**.

**What is Switchyard?** An open-source router that sends each agent step to an appropriate model to cut cost **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA NeMo Agent Toolkit (GitHub)](https://github.com/NVIDIA/NeMo-Agent-Toolkit) |
| 2026-10-28 | [NeMo Helix home (v0.8)](https://docs.nvidia.com/nemo-helix/v0.8/documentation/home) |
| 2026-10-28 | [NeMo Helix release 0.1.0 notes](https://docs.nvidia.com/nemo-helix/v0.7.3/documentation/reference/release-notes/release-0-1-0.md) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
| 2026-10-28 | [Route AI Agent Workloads Across Models with NVIDIA NeMo Switchyard](https://developer.nvidia.com/blog/route-ai-agent-workloads-across-models-with-nvidia-nemo-switchyard) |
