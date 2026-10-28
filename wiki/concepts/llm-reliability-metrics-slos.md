# LLM Reliability Metrics and SLOs

An agent dashboard needs metrics at three layers: serving (latency, throughput and errors), agent behaviour (task success, tool-call validity, steps and cost), and quality (feedback, evaluator scores and drift). Service-level objectives (SLOs) are then set per tier. Agent-level and quality metrics are aggregated from the traces in [Agent Observability and Tracing](agent-observability-tracing.md); serving metrics also come from load tests.

## Serving metrics

NVIDIA's NIM benchmarking guide defines the serving metrics, the same ones measured under load in [LLM Inference Profiling](llm-inference-profiling.md) **[web]**:
- **Time to first token (TTFT):** from sending the query to the first non-empty token. It includes queueing, prefill and network time.
- **End-to-end latency (E2E):** TTFT plus generation time.
- **Inter-token latency (ITL, or time per output token):** (E2E − TTFT) / (output tokens − 1).
- **Tokens per second for the system:** rises with concurrency until saturation.
- **Tokens per second per user:** falls as concurrency rises.
- **Requests per second.**

The guide warns that tools define these differently, so compare only like with like.

OpenTelemetry has matching metrics, all with Development status **[web]**:
- **Server side:** `gen_ai.server.time_to_first_token`, `gen_ai.server.time_per_output_token` and `gen_ai.server.request.duration`.
- **Client side:** `gen_ai.client.operation.duration`.
- **Agent level:** it also lists agent metrics such as `gen_ai.invoke_agent.duration`.

GPU utilisation, memory and power come from DCGM through Prometheus and Grafana. NVIDIA's model-monitoring blog notes that Triton exports latency and utilisation in Prometheus format **[sourced]**.

## Agent behaviour

IBM's agent-evaluation explainer lists the behavioural metrics **[sourced]**:
- task success or completion rate, and error rate;
- steps or tool calls per task;
- tool-call validity: a wrong function name, missing parameters, wrong types, hallucinated parameters or ungrounded values;
- retries, loops and timeouts;
- token and compute cost, and latency.

Measure cost per successful task rather than per request, because that is the figure that exposes retry storms.

## Quality

The same IBM source adds user feedback scores, online evaluator scores, adherence to safety policy, and the success rate of prompt-injection attempts **[sourced]**. Data and prediction drift complete the picture, as described in [Model and Data Drift Monitoring](model-and-data-drift-monitoring.md).

## Dashboards, alerts and SLOs

LangSmith's alerts work on run count, cost, errors (count or percentage), feedback score and latency, over 5- or 15-minute windows. Each threshold can be previewed against history. Error-budget design comes from Google's SRE practice, where the error budget is "100% minus the SLO" and an error budget policy says what happens when it runs out, such as prioritising reliability work or freezing releases **[web]**. Applied to LLM services by analogy:
- percentile targets, such as p95 TTFT;
- separate tiers for interactive and batch traffic;
- regression thresholds for quality metrics, enforced before release by the gates in [Agent Release Management](agent-release-management.md);
- alerting on how fast the error budget is burning, rather than on single spikes.

The trade-offs **[web]**:
- Higher concurrency raises system throughput but lowers per-user tokens per second.
- Routing to cheaper models lowers cost but may cut task success.
- Averages hide tails, so use percentiles and slice by route and tool.

## Interview Q&A

**Define TTFT and ITL, and say what drives each.** TTFT is the time to the first non-empty token: queueing plus prefill plus network, so it depends on prompt length and load. ITL is the average gap between later tokens, set by decode speed, which is bound by memory bandwidth **[web]**.

**Which metrics matter for an agent beyond model latency?** Task success, tool-call validity, steps per task, retry and error rates, cost per successful task, feedback, and policy adherence **[sourced]**.

**Why do per-user and system tokens per second diverge?** Raising concurrency increases total throughput until compute saturates, while each user's share drops **[web]**.

**What alert would catch a silent quality regression?** One on average online-evaluator or feedback score over a window, or on drift in the output distribution, not only on errors and latency **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [A guide to monitoring machine learning models in production (NVIDIA blog)](https://developer.nvidia.com/blog/a-guide-to-monitoring-machine-learning-models-in-production) |
| 2026-10-28 | [NIM LLM benchmarking metrics](https://docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html) |
| 2026-10-28 | [OpenTelemetry GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai) |
| 2026-10-28 | [LangSmith alerts](https://docs.langchain.com/langsmith/alerts) |
| 2026-10-28 | [Implementing SLOs (Google SRE Workbook)](https://sre.google/workbook/implementing-slos/) |
