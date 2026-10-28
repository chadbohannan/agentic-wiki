# Agent Profiling and Result Analysis

Profiling records where an agent spends its time and tokens, and result analysis reads those profiles together with evaluation scores to decide what to change. This page covers exam objectives 3.2, 3.4 and 3.5: the NeMo Agent Toolkit's profiler, a procedure for analysing results, and how to compare agent configurations fairly.

The analysis chooses among the levers in [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md). Its scores come from [Agent Evaluation Pipelines](agent-evaluation-pipelines.md). Profiling at the serving layer, below the agent, is covered in [LLM Inference Profiling](llm-inference-profiling.md).

## The NeMo Agent Toolkit profiler

The profiler is a separate package, `nvidia-nat-profiler`. It runs through `nat eval` with a `profiler` section in the config, according to the toolkit's docs source (`develop` branch). It instruments LLM and tool calls through two mechanisms:
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

The docs' worked example compares several LLMs on an email-phishing workflow. It plots prompt against completion tokens, workflow runtime including the worst case, and total tokens. It uses nested-stack Gantt charts to dissect a slow model, and shows accuracy, relevance and groundedness side by side **[sourced]**. The sizing calculator uses profiler data to estimate GPU-cluster size from observed latency and throughput. The docs warn that results measured against public hosted LLMs may be inaccurate, so profile an isolated, self-hosted model **[sourced, intro only]**.

## Analysis procedure

A workable procedure, consistent with the sources but synthesised here, has seven steps **[inferred]**:

1. Slice scores by task type, tool, difficulty and failure category.
2. Read the worst-scoring transcripts and the judge's reasoning.
3. Classify each failure's cause: retrieval, tool selection or arguments, prompt, model capability, looping or stopping, or the environment.
4. Measure cost per successful task rather than per call. A Switchyard guide recommends this metric, according to a secondary snippet.
5. Pick a lever: a prompt or tool-description fix, a model swap, routing, caching or test-time compute.
6. Re-run with repetitions, and test significance, for example with NeMo Evaluator's McNemar-based `nel compare` **[web]**.
7. Gate the release on the result.

To compare agents across tasks and datasets (objective 3.2), run every candidate configuration on identical datasets with identical graders, and keep the judge model fixed. Report means with variance and pass^k, on one table of accuracy, p95 latency, tokens and cost **[inferred]**. The toolkit's API index has a `multi_eval_runner` module, but its behaviour was not read.

## Interview Q&A

**The profiler shows one LLM call dominating latency. What next?** Open the nested bottleneck chart. Then shorten that call's prompt or output, use prefix caching, switch to a smaller or routed model, parallelise independent tool calls, and re-check accuracy **[sourced, inferred]**.

**What is prompt-prefix analysis for?** Finding repeated prefixes to pre-warm in the KV cache **[sourced]**.

**How do you compare two agent versions fairly?** Use the same dataset, graders and judge, run several repetitions, and report variance and significance **[inferred, web]**.

**What does the sizing calculator need?** Profiler data from an isolated, GPU-hosted LLM rather than a shared public endpoint **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit profiler guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/profiler.md) |
| 2026-10-28 | [NeMo Agent Toolkit sizing calculator (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/sizing-calc.md) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
