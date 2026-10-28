# Streaming and Real-Time Agent UX

Streaming tokens, tool-call progress and intermediate steps keeps conversational agents responsive while long multi-tool runs are in flight. This page covers exam objective 2.5: what to stream, how frameworks expose it, and how streaming interacts with serving latency, validation and failure.

The interfaces that consume these streams are described in [Human-Agent Interaction UIs](human-agent-interaction-uis.md), and retrying a failed stream without duplicating output is a problem treated in [Agent Error Handling and Resilience](agent-error-handling-resilience.md). The serving-side tuning behind time-to-first-token is in [LLM Inference Optimization](llm-inference-optimization.md).

## What to stream and how

A streaming agent can emit tokens, reasoning deltas, tool-call argument chunks, tool lifecycle events and state updates. LangGraph's streaming documentation (read 2026-10-28) lists the stream modes `values` (full state after each step), `updates` (per-node state changes), `messages` (LLM token and metadata pairs), `custom` (arbitrary data from `get_stream_writer`), `checkpoints`, `tasks` and `debug`. There is no dedicated tool-call mode, so tools report progress through `custom`. The page recommends a newer typed-projection API introduced in LangGraph v1.2 for new applications **[web]**. An earlier version of this page cited a `stream_events(version="v3")` API with `message.text`, `message.reasoning` and `message.tool_calls` projections, which came from a search summary; the current docs mention `stream_events(..., version="v3")` only in passing, so the projection names are unverified **[web, flagged]**. Deployed LangGraph servers stream over Server-Sent Events by default, with WebSocket optional **[web, search summary]**.

On the NVIDIA side, NIM exposes OpenAI-compatible streaming, and its function-calling documentation has a section on tool calls in streamed responses, but only the page outline was readable **[web, snippet-level]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) includes an API server and UI with intermediate-step visibility, again from its navigation listing only **[sourced, unread]**.

## Latency in the serving layer

NVIDIA's Triton optimisation guide explains that dynamic batching raises throughput but can add queueing delay, that minimum latency means concurrency 1 with no batcher, that model instances and TensorRT improve utilisation, and that Perf Analyzer measures all of it **[sourced]**. Triton has since been renamed NVIDIA Dynamo-Triton (March 2025), see [Triton Inference Server](../components/triton-inference-server.md), and for LLM workloads Perf Analyzer and GenAI-Perf are being superseded by AIPerf, as [LLM Inference Profiling](llm-inference-profiling.md) explains **[web]**. For LLM streaming, the user-visible metrics are time to first token and inter-token latency **[inferred]**.

## Interaction with other concerns

Streaming must coexist with tool-execution pauses and approval gates: the MCP specification says a human should always be able to deny tool invocations **[web]**. Mid-stream errors cannot be retried transparently without duplicating output, so retry before the first token or buffer, and show partial progress. Streaming also lowers perceived latency at the cost of complicating validation (a structured output is incomplete until the end), guardrails (output rails may need buffering) and retries **[inferred]**.

## Interview Q&A

**Why stream agent output?** For perceived latency, visibility into long tool runs, and the ability to interrupt **[inferred]**.

**Which transports?** SSE for one-way token and event streams, WebSockets when interaction is bidirectional **[web, inferred]**.

**How does dynamic batching affect streaming latency?** It improves throughput but can add queue delay for individual requests, so tune against latency targets **[sourced]**.

**What makes streamed structured output hard to validate?** The JSON is incomplete until finished, so use a partial parser or validate on completion **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Optimization - NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/optimization.html) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-28 | [LangGraph streaming](https://docs.langchain.com/oss/python/langgraph/streaming) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
