# NeMo Relay

NVIDIA NeMo Relay is a runtime for observing and controlling agent runs. Its Rust core sits behind Python and Node.js bindings, and it works through a scope stack, middleware (intercepts and guardrails), plugins, and an event stream that can be exported to ATOF, ATIF, OpenTelemetry and OpenInference. The September 2025 study guide predates it, but the migration guide of the deprecated [NeMo Agent Toolkit](nemo-agent-toolkit.md) names Relay as the successor for its observability, middleware and governance functions **[web]**, so it is where NVIDIA's answer to [Agent Observability and Tracing](../concepts/agent-observability-tracing.md) now lives.

## Architecture

NVIDIA's documentation says Relay "helps you observe and control what happens inside agent runs without rewriting the agent stack you already have" **[web]**. Its architecture page describes these parts **[web]**:
- **A shared Rust core** behind language bindings. Python and Node.js are primary, and Go and raw C FFI are experimental.
- **A scope stack** that gives runs parent–child structure and isolates concurrent requests.
- **Middleware registries** for tool calls and LLM calls.
- **A plugin system** for adaptive, observability and guardrail components, installed from configuration.
- **A canonical, parent-linked event stream.**
- **An asynchronous subscriber dispatcher**, so that publishing events never delays the real call.

As of 2026-10-28 the latest documentation is v0.11.0. Relay is pre-1.0, so its interfaces may change **[web]**.

## Event formats

Relay has four outputs **[web]**:
- **ATOF** (Agent Trajectory Observability Format) is the raw, canonical event stream as JSONL, and the recommended starting point.
- **ATIF** (Agent Trajectory Interchange Format) trajectories are generated from completed runs, for replay, analysis and evaluation.
- **OpenTelemetry (OTLP)** and **OpenInference** exporters feed standard backends.

The documentation does not name Phoenix or Langfuse, but OTLP and OpenInference output is the compatible path to both.

## Middleware and governance

The middleware concepts page distinguishes kinds of middleware by whether they change the real call **[web]**:
- **Intercepts** change the real execution path. There are request intercepts, execution intercepts (for retries, timing and routing) and stream-execution intercepts.
- **Guardrails** do not alter the call. Conditional-execution guardrails decide whether work may run at all, based on policy, budget or context. Sanitize guardrails rewrite only the observability copy of the data.
- **Metadata injectors** add flat, insert-only labels to events.

The execution order is:
1. conditional guardrails;
2. request intercepts;
3. request sanitisers, which affect only the start-event payload;
4. execution intercepts;
5. the real callback;
6. response sanitisers.

Failure behaviour differs by kind. Guardrails and request and execution intercepts fail closed: the callback does not run. A failing sanitiser clears the affected observability fields. Metadata injectors fail open. Middleware can be registered globally, locally to a scope (removed when the scope closes), or by a plugin. Standard credential headers are stripped from the event copy before sanitisers run, but custom header names need a sanitiser of your own. A guardrail that rejects a call emits a rejection mark and skips the start and end events **[web]**.

The documentation's navigation lists further features **[web, snippet]**:
- a plugin for [NeMo Guardrails](nemo-guardrails.md), so policy rails and Relay's own guardrails can run in one middleware chain; on the Guardrails side, a hooks server not yet in a release answers Relay's pre-tool-call check with allow or deny;
- an Adaptive plugin with an Adaptive Cache Governor;
- integrations for LangChain, LangGraph and Deep Agents;
- CLI integration for coding agents.

## Why it matters

Relay provides a framework-independent trace and event source at the runtime level, including calls that were rejected. Sanitisers minimise the personal data that reaches telemetry, and guardrails enforce budgets and policies while leaving the marks an [audit trail](../concepts/audit-trails-and-traceability.md) needs.

## Interview Q&A

**Intercept versus guardrail in Relay?** An intercept changes the real request or execution: it rewrites, retries, routes or replaces. A guardrail either gates whether the call runs, or rewrites only the observability copy **[web]**.

**What happens when a Relay guardrail or intercept throws?** It fails closed and the real callback does not run. A failing sanitiser blanks the affected telemetry fields. Only metadata injectors fail open **[web]**.

**How do you keep personal data out of traces while still sending it to the model?** Use sanitisers. They alter only the event payload sent to exporters, and the provider receives the original request **[web]**.

**ATOF versus ATIF?** ATOF is the raw, canonical event stream Relay captures. ATIF is a trajectory format derived from completed runs, for replay, analysis and evaluation **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Relay docs](https://docs.nvidia.com/nemo/relay/latest/) |
| 2026-10-28 | [NeMo Relay architecture](https://docs.nvidia.com/nemo/relay/about-nemo-relay/architecture) |
| 2026-10-28 | [NeMo Relay middleware concepts](https://docs.nvidia.com/nemo/relay/about-nemo-relay/concepts/middleware) |
| 2026-10-28 | [NeMo Agent Toolkit migration guide](https://docs.nvidia.com/aiqtoolkit/latest/resources/migration-guide.html) |
| 2026-10-28 | [NeMo Guardrails (develop): NeMo Relay hooks](https://github.com/NVIDIA-NeMo/Guardrails/blob/develop/docs/run-rails/using-hooks-server/nemo-relay.mdx) |
