# LLM Reliability Metrics and SLOs

An agent dashboard needs metrics at three layers: serving (latency, throughput and errors), agent behaviour (task success, tool-call validity, steps and cost), and quality (feedback, evaluator scores and drift). Service-level objectives (SLOs) are then set per tier. This page covers exam objective 8.1.

The metrics come from the tracing in [Agent Observability and Tracing](agent-observability-tracing.md) and the load tests in [LLM Inference Profiling](llm-inference-profiling.md). They set the gates in [Continuous Regression Benchmarking](continuous-regression-benchmarking.md). Quality drift is treated in [Model and Data Drift Monitoring](model-and-data-drift-monitoring.md), and the availability practices built on these targets are in [Agent Uptime, Transparency and Trust](agent-uptime-transparency-trust.md).

## Serving metrics

NVIDIA's NIM benchmarking guide defines the serving metrics **[web]**:
- **Time to first token (TTFT):** from sending the query to the first non-empty token. It includes queueing, prefill and network time.
- **End-to-end latency (E2E):** TTFT plus generation time.
- **Inter-token latency (ITL, or time per output token):** (E2E − TTFT) / (output tokens − 1).
- **Tokens per second for the system:** rises with concurrency until saturation.
- **Tokens per second per user:** falls as concurrency rises.
- **Requests per second.**

The guide warns that tools define these differently, so compare only like with like.

OpenTelemetry has matching metrics, all with Development status **[web, first 100k of 136k characters read]**:
- **Server side:** `gen_ai.server.time_to_first_token`, `gen_ai.server.time_per_output_token` and `gen_ai.server.request.duration`.
- **Client side:** `gen_ai.client.operation.duration`.
- **Agent level:** its table of contents also lists agent metrics such as `gen_ai.invoke_agent.duration`, but their definitions were not read.

GPU utilisation, memory and power come from DCGM through Prometheus and Grafana. NVIDIA's model-monitoring blog notes that Triton exports latency and utilisation in Prometheus format **[sourced]**.

## Agent behaviour

IBM's agent-evaluation explainer lists the behavioural metrics **[sourced]**:
- task success or completion rate, and error rate;
- steps or tool calls per task;
- tool-call validity: a wrong function name, missing parameters, wrong types, hallucinated parameters or ungrounded values;
- retries, loops and timeouts;
- token and compute cost, and latency.

Measure cost per successful task rather than per request, because that is the figure that exposes retry storms **[inferred]**.

## Quality

The same IBM source adds user feedback scores, online evaluator scores, adherence to safety policy, and the success rate of prompt-injection attempts **[sourced]**. Data and prediction drift complete the picture.

## Dashboards, alerts and SLOs

LangSmith's alerts work on run count, cost, errors (count or percentage), feedback score and latency, over 5- or 15-minute windows. Each threshold can be previewed against history **[web]**. Error-budget SLO design is standard site-reliability practice applied here by analogy, since no LLM-specific SLO source was read **[inferred]**:
- percentile targets, such as p95 TTFT;
- separate tiers for interactive and batch traffic;
- alerting on how fast the error budget is burning, rather than on single spikes.

The trade-offs **[web, inferred]**:
- Higher concurrency raises system throughput but lowers per-user tokens per second.
- Routing to cheaper models lowers cost but may cut task success.
- Averages hide tails, so use percentiles and slice by route and tool.

## Interview Q&A

**Define TTFT and ITL, and say what drives each.** TTFT is the time to the first non-empty token: queueing plus prefill plus network, so it depends on prompt length and load. ITL is the average gap between later tokens, set by decode speed, which is bound by memory bandwidth **[web, inferred]**.

**Which metrics matter for an agent beyond model latency?** Task success, tool-call validity, steps per task, retry and error rates, cost per successful task, feedback, and policy adherence **[sourced, inferred]**.

**Why do per-user and system tokens per second diverge?** Raising concurrency increases total throughput until compute saturates, while each user's share drops **[web]**.

**What alert would catch a silent quality regression?** One on average online-evaluator or feedback score over a window, or on drift in the output distribution, not only on errors and latency **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [A guide to monitoring machine learning models in production (NVIDIA blog)](https://developer.nvidia.com/blog/a-guide-to-monitoring-machine-learning-models-in-production) |
| 2026-10-28 | [NIM LLM benchmarking metrics](https://docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html) |
| 2026-10-28 | [OpenTelemetry GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai) |
| 2026-10-28 | [LangSmith alerts](https://docs.langchain.com/langsmith/alerts) |
