# Agent Observability and Tracing

Agent observability records each run as a trace of nested spans (agent, LLM call, tool call, retrieval) with metadata, so its behaviour can be monitored, debugged and evaluated. A conventional service trace shows latency and errors. An agent trace must also show decisions: which tool was chosen with what arguments, what the model saw, and why the loop continued. Traces are the raw material for three other jobs: root-cause analysis in [Agent Failure Diagnosis](agent-failure-diagnosis.md), the dashboards in [LLM Reliability Metrics and SLOs](llm-reliability-metrics-slos.md), and the compliance record in [Audit Trails and Traceability](audit-trails-and-traceability.md). Below the agent, GPU-level signals come from a separate source, [DCGM GPU Telemetry](../components/dcgm-gpu-telemetry.md).

## The data model

LangSmith's observability concepts give a representative model **[web]**:
- **Run:** the unit of tracing, similar to an OpenTelemetry span.
- **Trace:** runs grouped under one trace ID.
- **Thread:** the traces of a multi-turn conversation, grouped by a `thread_id` metadata key. A thread's "trajectory" is a flat view of its messages.
- **Feedback scores** attach to runs, and metadata and tags (agent version, environment) support filtering.

A trace is capped at 25,000 runs, and the hosted service keeps data for 180 days. LangChain apps turn on tracing with `LANGSMITH_TRACING=true` and an API key **[web]**. The study guide's LangChain tracing source describes the same idea: a trace records the steps of a run, from inputs through the calls made to the outputs **[sourced]**.

## Standards

OpenTelemetry's GenAI semantic conventions now live in their own repository. They define spans for creating and invoking agents and for invoking workflows **[web]**:
- **`invoke_agent`** is a CLIENT span for remote agents such as hosted assistants, and an INTERNAL span for in-process frameworks.
- **`invoke_workflow`** covers multi-agent or graph orchestration.
- **Operation values** include `plan` and `execute_tool`.

The key attributes are `gen_ai.operation.name`, `gen_ai.provider.name`, the agent's id, name and version, `gen_ai.conversation.id`, token counts under `gen_ai.usage.*`, and `gen_ai.response.finish_reasons`. Message content, system instructions and tool definitions are opt-in, because they can contain personal data, and `_ref` variants can point to external storage instead. Almost all `gen_ai.*` attributes have Development status, meaning they are unstable and may be renamed. `error.type` and the `server.*` attributes are Stable **[web]**.

OpenTelemetry's 2025 blog on agent observability says that a convention for agent applications existed, based on Google's agent whitepaper, while one for frameworks was still under discussion. It contrasts two ways of instrumenting: telemetry built into the framework (CrewAI, for example), and external instrumentation libraries (Traceloop, Langtrace, OpenTelemetry's instrumentation-genai) **[web]**.

## Ways to collect

There are three places to instrument **[sourced]**:
- **SDK or auto-instrumentation inside the app.**
- **A gateway or proxy in the call path.** The study guide's Portkey source describes this: one trace-ID header ties together a request's LLM calls, and the gateway logs cost, timing and payloads and can add caching and retries. It is easy to add, but it sees only LLM calls, not internal tool or planning steps.
- **Runtime middleware**, such as [NeMo Relay](../components/nemo-relay.md), which sees tool and planning steps as well as LLM calls, including calls a guardrail blocked.

The trade-offs are sampling against completeness, the cost of storing prompts, personal data in content attributes, and vendor lock-in against standard formats.

## Backends

- **LangSmith** is hosted, with a self-hosting option. It alerts on run count, cost, error count or percentage, average feedback score and average latency, over 5- or 15-minute windows, and sends alerts to Slack, PagerDuty, Dynatrace or a webhook **[web]**.
- **Langfuse** is open source and self-hostable, with traces of nested observations and scores **[web]**.
- **Arize Phoenix** is built on OpenTelemetry and OpenInference and accepts OTLP. It offers LLM, code and human evaluators, prompt versioning, and datasets and experiments for comparing app versions **[web]**.

IBM frames agent evaluation as sitting inside "a broader formal observability framework" **[sourced]**. On the NVIDIA side, the NeMo Agent Toolkit shipped observability exporters and a profiler. With the toolkit deprecated, its migration guide names NeMo Relay as the successor for observability and middleware **[web]**.

## Interview Q&A

**What should an agent trace contain beyond a normal request trace?** Nested spans for each LLM and tool call, with inputs and outputs, token counts and finish reasons. Add agent and version metadata, a conversation or thread ID, and feedback or eval scores attached to runs, so the decisions are visible and not just the timings **[web]**.

**Why use the OpenTelemetry GenAI conventions instead of a vendor SDK?** Telemetry becomes portable across frameworks and backends. But the conventions are still in Development status, so attribute names can change, and content capture is opt-in because of personal data **[web]**.

**Gateway or in-app instrumentation?** A gateway needs almost no code change, groups LLM calls by trace ID and logs cost, but it cannot see non-LLM steps. In-app or runtime instrumentation sees the whole agent loop **[sourced]**.

**Thread versus trace in LangSmith?** A trace is one operation, such as one turn. A thread groups the traces of a multi-turn conversation by `thread_id` **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [Log, trace and monitor with Portkey (LangChain)](https://python.langchain.com/docs/integrations/providers/portkey/logging_tracing_portkey/) |
| 2026-10-28 | [LangChain tracing concepts](https://python.langchain.com/docs/concepts/tracing/) |
| 2026-10-28 | [LangSmith observability concepts](https://docs.langchain.com/langsmith/observability-concepts) |
| 2026-10-28 | [LangSmith alerts](https://docs.langchain.com/langsmith/alerts) |
| 2026-10-28 | [OpenTelemetry GenAI semantic conventions](https://github.com/open-telemetry/semantic-conventions-genai) |
| 2026-10-28 | [AI agent observability (OpenTelemetry blog, 2025)](https://opentelemetry.io/blog/2025/ai-agent-observability/) |
| 2026-10-28 | [Arize Phoenix docs](https://arize.com/docs/phoenix) |
| 2026-10-28 | [NeMo Agent Toolkit migration guide](https://docs.nvidia.com/aiqtoolkit/latest/resources/migration-guide.html) |
