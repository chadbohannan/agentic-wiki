# Tuning Agents for Accuracy, Latency and Cost

Tuning an agent is a multi-objective problem. Every lever trades accuracy against latency and cost, so the honest result is a Pareto front rather than a single number. Which lever to pull should follow from the measurements in [Agent Profiling and Result Analysis](agent-profiling-and-result-analysis.md), not from habit.

## Fixing decision quality

The cheapest levers are not parameters but the agent's own interfaces, and they come from reading failed trajectories in the [evaluation suite](agent-evaluation-pipelines.md). This is the design-time counterpart of the run-time self-correction in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md): a developer reads the transcript and changes the agent, rather than the agent critiquing itself. Anthropic's guidance supplies most of the practical moves: improve tool descriptions and signatures from observed errors, following the tool-design rules in [Tool Calling and MCP](tool-calling-and-mcp.md); start prompts minimal and add guidance per failure; use an evaluator-optimizer loop when criteria are clear; check ground truth from the environment at each step; add human checkpoints; and cap iterations with stopping conditions. Feeding tool execution errors back to the model enables self-correction, as the MCP specification describes. More refinement raises accuracy at the cost of latency and tokens, and more autonomy raises compounding-error risk, so Anthropic advises sandboxed testing with guardrails **[web]**.

## Sampling parameters

The NeMo Agent Toolkit's optimiser guide names the knobs: hyperparameters (`temperature`, `top_p`, `max_tokens`), prompts, model choices and processing parameters **[sourced]**. It sets temperature 0.0 by default for its OpenAI and NIM providers. Its default search space is temperature 0.1–0.8 in steps of 0.2, `top_p` 0.5–1.0, and NIM `max_tokens` 128–2176 **[sourced]**. Low temperature helps tool-call determinism and stable evaluations. Higher temperature helps diverse generation and best-of-N sampling.

## Automated optimisation

The toolkit's optimiser, `nat optimize`, is installed with the `nvidia-nat[config-optimizer]` extra **[sourced]**.
- **Search space:** configuration fields are marked as `OptimizableField` with a `SearchSpace`, which can be continuous (low, high, step, log), categorical, or a prompt.
- **Numeric and categorical parameters** are searched with Optuna: TPE by default for a single objective, NSGA-II for several, with grid search optional.
- **Prompts** are evolved by a custom genetic algorithm. It uses tournament or roulette selection, elitism, crossover, LLM-driven mutation and a diversity penalty, and it can take "oracle feedback" from the worst-scoring items.
- **Trials:** each runs the workflow `reps_per_param_set` times (default 3) through weighted evaluators, each set to maximise or minimise. For example, latency is minimised with weight 0.2 and correctness maximised with weight 0.8. A `target` score stops the search early.
- **Outputs:** `best_params.json`, the Optuna trial data, Pareto and parallel-coordinate plots, `optimized_prompts.json` and the genetic algorithm's prompt history.
- **Starting values from the docs:** 20–50 trials with 3–5 repetitions each (10–20 when outputs are very stochastic). For the genetic algorithm: population 10–16, 5–8 generations, crossover 0.7, mutation 0.2, elitism 1–2, tournament size 3.

Its successor is the optimisation service in [NeMo Helix](../components/nemo-helix.md), which in 0.8 runs named strategies (prompt-master, Switchyard routing, Analyst, Experimentalist) and combines numeric search with prompt optimisation; NVIDIA calls these strategies a research preview that still needs developer review **[web]**.

Outside NVIDIA, DSPy (arXiv 2310.03714) treats LM pipelines as declarative modules and compiles them against a metric, learning the demonstrations and instructions that the prompt-writing practice in [Context Engineering](context-engineering.md) otherwise leaves to a person. Its authors report compiled programs beating standard few-shot prompting, and small models rivalling expert-written prompt chains **[web]**. It solves the same problem as the toolkit's prompt optimiser by different means.

## Model routing and cascades

Routing sends easy steps to cheap models and hard steps to strong ones. RouteLLM (arXiv 2406.18665) trains routers on preference data to choose between a strong and a weak model. It cut cost by more than 2x in some settings without loss of quality **[web]**. NVIDIA's NeMo Switchyard (blog, 2026-08-11) routes each agent step among models, through a `libsy` SDK or a gateway compatible with the OpenAI, Anthropic and Responses APIs. It offers four routers: LLM-classifier, stage and escalation routers that need no tuning, and a tunable prefill router. NVIDIA reports two results, both its own measurements **[web]**:
- On a LangChain benchmark of 145 tasks: 74% lower cost than using the frontier model alone, about 6 points less accuracy, with about 7% of calls going to the frontier model.
- On a Cognition benchmark: 50.6% at $3.11 mean cost, within 2.8 points of the frontier model at about 28% lower cost.

An independent write-up noted that the accuracy gap is within run-to-run variance **[web, snippet]**.

For bounded routing and classification steps, a [decision model](decision-models.md) such as Jev or Clef replaces a generative call with a single scoring pass. Their vendors claim large latency and cost savings that have not been independently reproduced **[web]**. Microsoft's production guide lists the same cost levers: small models for simple tasks, routing by complexity, response caching, and verifying with evals first **[sourced]**.

## Distillation and quantisation

Distillation trains a smaller student model on a larger teacher's outputs. Quantisation lowers numeric precision, for example to FP8 or INT4, to cut memory and latency. Both usually cost a little accuracy, which must be re-measured with the evaluation suite. On the NVIDIA side, [TensorRT-LLM](../components/tensorrt-llm.md) and NIM serve FP8 and FP4 models, and the serving-side levers beyond precision (batching, KV caching, speculative decoding) are in [LLM Inference Optimization](llm-inference-optimization.md) and, framed as cost per token, in [Inference Cost and Availability](inference-cost-and-availability.md). The toolkit also documented fine-tuning with NeMo Customizer (DPO) and OpenPipe ART **[sourced]**.

## Test-time compute

Spending more inference on harder queries, by sampling candidates, verifying them or revising them, extends the reasoning techniques in [Chain-of-Thought and Task Decomposition](chain-of-thought-and-task-decomposition.md) and raises accuracy at the cost of latency and tokens, which grow with the number of samples. Snell et al. (arXiv 2408.03314) found that allocating compute by prompt difficulty was more than 4x more efficient than best-of-N. With that allocation, a small model could beat a 14x larger one in FLOPs-matched settings, on problems where the small model already succeeds sometimes **[web]**. The toolkit's experimental `nat.experimental.test_time_compute` package builds this from four kinds of pipeline stage **[sourced]**:
- **Search:** for example, multi-LLM planning.
- **Editing:** for example, iterative plan refinement.
- **Scoring:** for example, LLM-based plan scoring.
- **Selection:** for example, best-of-N or a threshold.

These stages apply to planning, tool-use and agent-execution pipelines.

## Interview Q&A

**How would you cut agent cost by 50% without much accuracy loss?** Route easy steps to a small model through a cascade or escalation router, cache prefixes and trim tokens. Then verify with the eval suite and report the Pareto front **[sourced, web]**.

**The agent confuses two similar tools; what do you do?** Rewrite the descriptions to delineate boundaries, merge or remove overlapping tools, add examples, and rerun the eval set **[web]**.

**What is an evaluator-optimizer loop, and when do you use it?** One LLM generates and another critiques against clear criteria, iterating; use it when criteria are explicit and iteration measurably helps **[web]**.

**What does the NeMo Agent Toolkit's optimiser tune, and how?** Numeric and categorical parameters with Optuna and prompts with a genetic algorithm, scored against weighted multi-objective eval metrics over repeated runs **[sourced]**.

**What is the test-time compute trade-off?** More samples and verification raise accuracy on hard prompts at the cost of latency and tokens, so allocate it by difficulty **[web]**.

**Why repeat runs for each parameter set?** LLM outputs are noisy, and averaging stabilises the objective **[sourced]**.

**What is the risk in quantising or distilling?** Silent accuracy regressions on edge cases, so re-run the regression evals.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit optimizer guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/optimizer.md) |
| 2026-10-28 | [NeMo Agent Toolkit test-time compute (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/test-time-compute.md) |
| 2026-10-28 | [AI Agents for Beginners: production (Microsoft)](https://microsoft.github.io/ai-agents-for-beginners/10-ai-agents-production/) |
| 2026-10-28 | [Powering the Next Generation of AI Agents (NVIDIA)](https://www.nvidia.com/en-us/ai/) |
| 2026-10-28 | [NeMo Helix home (v0.8)](https://docs.nvidia.com/nemo-helix/v0.8/documentation/home) |
| 2026-10-28 | [DSPy (arXiv 2310.03714)](https://arxiv.org/abs/2310.03714) |
| 2026-10-28 | [RouteLLM (arXiv 2406.18665)](https://arxiv.org/abs/2406.18665) |
| 2026-10-28 | [NeMo Helix release 0.8.0 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/current-release) |
| 2026-10-28 | [Route AI Agent Workloads Across Models with NVIDIA NeMo Switchyard](https://developer.nvidia.com/blog/route-ai-agent-workloads-across-models-with-nvidia-nemo-switchyard) |
| 2026-10-28 | [Scaling test-time compute (arXiv 2408.03314)](https://arxiv.org/abs/2408.03314) |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
