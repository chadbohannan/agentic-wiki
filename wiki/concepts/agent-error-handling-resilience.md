# Agent Error Handling and Resilience

Resilient agents apply the standard distributed-systems toolkit (retry with backoff and jitter, timeouts, circuit breakers, fallbacks and graceful degradation) to LLM calls and tool calls, then add controls for failures that are specific to agents. This page covers exam objective 2.4.

Tool failures surface through the interface in [Tool Calling and MCP](tool-calling-and-mcp.md), malformed model output is reduced by [Structured Outputs](structured-outputs.md), and mid-stream failures complicate retries as described in [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md). Availability practices for the serving layer are in [Inference Cost and Availability](inference-cost-and-availability.md), and diagnosing these failures is covered in [Agent Failure Diagnosis](agent-failure-diagnosis.md).

## Retries

The study guide's Azure Architecture Center pages give the core rules. Retry only transient faults such as 429, 5xx and timeouts, and never 400, 401, 403 or 404. Strategies range from cancel through a single immediate retry to delayed incremental or exponential backoff; add jitter so many clients do not retry in lockstep, honour `Retry-After`, set per-call timeouts first, and keep total retry latency inside the SLO **[sourced]**. Avoid nested retries, where three layers of three attempts make nine, and cap aggregate retries per dependency (the Azure example is 60 per minute) **[sourced]**. In agents the loop is itself a retry layer, so step caps and replan caps belong to the same budget **[inferred]**; a vendor blog recommends a retry budget per trace **[web, vendor blog]**.

Idempotency decides what can be retried. Reads and idempotent writes (protected by idempotency keys) are safe, and non-idempotent actions such as payments or sent messages must not be retried automatically **[sourced]**. Classifying agent tools this way comes from vendor blogs **[web]**.

## Circuit breakers and degradation

A circuit breaker is a closed, open and half-open state machine that fails fast to protect a failing dependency and gives it time to recover, with half-open admitting limited trial traffic. Use one breaker per independent resource, and when it is open the caller can return cached or default values. It differs from retry because retry assumes the next attempt may succeed, whereas a breaker assumes failure is likely persistent **[sourced]**. Fallbacks include an alternate provider or model, a slower backup index, cached data with a timestamp, simpler functionality, queuing for later, or an honest message to the user **[sourced; agent examples from vendor blogs such as Portkey's]**.

## Failures specific to agents

Four classes need different responses **[inferred]**: malformed model output (parse errors), wrong tool or arguments (model errors, fixed by feeding the error back), tool or API outages (infrastructure errors, handled by retries and breakers) and runaway loops (needing step and cost caps). MCP formalises the model-fixable class by separating `isError: true` tool results from protocol errors **[web]**. In the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) the ReAct agent has parse-retry and tool-call-retry limits and a `max_tool_calls` cap **[web]**. No NVIDIA-specific retry or breaker feature turned up in the listed sources, which is a gap. Operationally, log transient faults as warnings, count retries, alert on a rising retry rate and emit breaker state changes, and test with fault injection and mocks that return real error types **[sourced]**.

## Interview Q&A

**Retry versus circuit breaker?** Retry assumes the next attempt may succeed; a breaker stops attempts when failure is likely persistent and probes through half-open. Use them together **[sourced]**.

**Why add jitter?** To prevent synchronised retries from many clients causing a retry storm **[sourced]**.

**Which tool calls must not be auto-retried?** Non-idempotent, side-effecting ones unless protected by idempotency keys **[sourced]**.

**The model returns invalid JSON for a tool call; what do you do?** A bounded retry with the parse error fed back (one retry by default in the toolkit), or constrained decoding to prevent it, then escalate or fall back after the cap **[web, inferred]**.

**What does graceful degradation look like when the search tool is down?** Switch to a backup index or cached results with a staleness note, or answer from model knowledge with an explicit caveat, rather than failing or looping **[sourced pattern, inferred application]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Transient Fault Handling (Azure Architecture Center)](https://learn.microsoft.com/en-us/azure/architecture/best-practices/transient-faults) |
| 2026-10-27 | [Circuit Breaker Pattern (Azure Architecture Center)](https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker) |
| 2026-10-27 | [Retry Pattern (Azure Architecture Center)](https://learn.microsoft.com/en-us/azure/architecture/patterns/retry) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [Resilience patterns for agents and LLM gateways (Gravitee)](https://gravitee.io/corpus/gen-1335/run-wild-run-free/resilience-patterns-for-run-wild-run-free-fallbacks-retries-and-circuit-breakers-for-ai-agents-and-llm-gateways.html) |
| 2026-10-27 | [Retries, fallbacks and circuit breakers in LLM apps (Portkey)](https://portkey.ai/blog/retries-fallbacks-and-circuit-breakers-in-llm-apps/) |
