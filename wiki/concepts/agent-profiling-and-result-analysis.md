# Agent Profiling and Result Analysis

Profiling records where an agent spends its time and tokens, and result analysis reads those profiles together with evaluation scores to decide what to change. It works at the level of agent steps (LLM calls, tool calls, waiting); profiling the serving engine underneath is a separate discipline, covered in [LLM Inference Profiling](llm-inference-profiling.md).

## The NeMo Agent Toolkit profiler

The profiler is a separate package, `nvidia-nat-profiler` **[web]**. It runs through `nat eval` with a `profiler` section in the config, so it profiles the same runs that [Agent Evaluation Pipelines](agent-evaluation-pipelines.md) score. It instruments LLM and tool calls through two mechanisms:
- callbacks for LangChain and LangGraph, LlamaIndex, CrewAI, Google ADK and Semantic Kernel;
- a `@track_function` decorator, which emits start, chunk and end span events.

It records tokens, timings and inputs and outputs **[sourced]**. Its analysis options are:
- **Forecasts** of token uniqueness and of workflow runtime, using linear and random-forest models.
- **LLM metrics.**
- **Prompt-prefix analysis**, which finds common prefixes worth pre-populating in the KV cache.
- **Bottleneck analysis**, with simple or nested stacks and Gantt charts.
- **Concurrency-spike detection.**
- **An optional prediction trie** of LLM call patterns.

Its outputs **[sourced]**:
- the raw traces;
- `inference_optimization.json`, with 90%, 95% and 99% confidence intervals for latency, throughput and runtime;
- `standardized_data_all.csv`, with prompt and completion tokens and the framework for each call;
- analysis reports.

The docs' worked example compares several LLMs on an email-phishing workflow. It plots prompt against completion tokens, workflow runtime including the worst case, and total tokens. It uses nested-stack Gantt charts to dissect a slow model, and shows accuracy, relevance and groundedness side by side **[sourced]**. The sizing calculator uses profiler data to estimate GPU-cluster size from observed latency and throughput. The docs warn that results measured against public hosted LLMs may be inaccurate, so profile an isolated, self-hosted model **[sourced]**.

## Analysis procedure

A workable procedure has seven steps:

1. Slice scores by task type, tool, difficulty and failure category.
2. Read the worst-scoring transcripts and the judge's reasoning.
3. Classify each failure's cause: retrieval, tool selection or arguments, prompt, model capability, looping or stopping, or the environment.
4. Measure cost per successful task rather than per call; a Switchyard guide recommends this metric **[web, snippet]**.
5. Pick a lever from [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md): a prompt or tool-description fix, a model swap, routing, caching or test-time compute.
6. Re-run with repetitions, and test significance, for example with NeMo Evaluator's McNemar-based `nel compare` **[web]**.
7. Gate the release on the result.

To compare agents across tasks and datasets, run every candidate configuration on identical datasets with identical graders, and keep the judge model fixed. Report means with variance and pass^k, on one table of accuracy, p95 latency, tokens and cost. Miller's "Adding Error Bars to Evals" (Anthropic, arXiv 2411.00640) gives the statistics: report standard errors, cluster them when questions come in related groups, reduce variance by sampling several answers per question, compare two models on per-question paired differences rather than on their summary scores, and use power analysis to check that the eval can detect the difference that matters **[web]**.

## Interview Q&A

**The profiler shows one LLM call dominating latency. What next?** Open the nested bottleneck chart. Then shorten that call's prompt or output, use prefix caching, switch to a smaller or routed model, parallelise independent tool calls, and re-check accuracy **[sourced]**.

**What is prompt-prefix analysis for?** Finding repeated prefixes to pre-warm in the KV cache **[sourced]**.

**How do you compare two agent versions fairly?** Use the same dataset, graders and judge, run several repetitions, and test the per-question paired differences with standard errors rather than comparing two headline scores **[web]**.

**What does the sizing calculator need?** Profiler data from an isolated, GPU-hosted LLM rather than a shared public endpoint **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit profiler guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/profiler.md) |
| 2026-10-28 | [NeMo Agent Toolkit sizing calculator (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/sizing-calc.md) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
| 2026-10-28 | [Adding Error Bars to Evals (Miller, arXiv 2411.00640)](https://arxiv.org/abs/2411.00640) |
