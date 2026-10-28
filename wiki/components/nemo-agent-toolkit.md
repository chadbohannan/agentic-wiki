# NeMo Agent Toolkit (and its deprecation)

NVIDIA's NeMo Agent Toolkit is a framework-agnostic toolkit for building, connecting, evaluating and profiling agent workflows. It has been renamed twice and, as of October 2026, is deprecated **[web]**. The exam, written in September 2025, still treats it as NVIDIA's agent framework, so it is worth knowing both what it was and what replaced each piece; the [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md) synthesis tracks that lineage across all NVIDIA products.

## Status

NVIDIA's documentation and migration guide carry the notice "NVIDIA NeMo Agent Toolkit has been deprecated as of October 2026 and will reach end of life (EOL) by end of year 2026." The documentation at `docs.nvidia.com/aiqtoolkit/latest/` was showing v1.9 **[web]**. The migration guide names three successors **[web]**:
- [NeMo Relay](nemo-relay.md) for observability, middleware and governance;
- [NeMo Helix](nemo-helix.md), a unified CLI for evaluation, optimisation and hardening, which at first used toolkit workflows as its agent format;
- NeMo Fabric, a consistent interface for running agent harnesses and custom agents.

Two other 2026 items are not established as related. A Futurum summary of GTC 2026 describes a separate "NVIDIA Agent Toolkit" stack (NemoClaw runtime, the AI-Q blueprint, Nemotron), and a third-party article mentions NeMo Microservices sunsetting on October 1, 2026 **[web, snippet]**. Both come from secondary sources, and neither is asserted as a successor here.

## Naming and capabilities

The product started as the Agent Intelligence Toolkit (AIQ, with an `aiqtoolkit` package and `aiq` module and CLI) and was renamed NeMo Agent Toolkit on 2025-06-11, version 1.2, with core technology and roadmap stated unchanged. The package and CLI became `nvidia-nat` and `nat`, and v1.5.0 removed the `aiq` aliases **[web, snippet]**.

Its navigation lists YAML workflows; functions and function groups; LLM, embedder, retriever, memory and object-store components; MCP and A2A support; integration with LangGraph and other frameworks; observability; an API server and UI; and, under improving workflows, evaluation, a profiler, an optimizer, a sizing calculator, test-time compute and fine-tuning. Its agent types are ReAct, Reasoning, ReWOO, Router, Tool Calling, Parallel and Sequential executors **[sourced]**, which are concrete implementations of the designs in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md). The ReAct agent's retry and tool-call limits are given in [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md), the memory providers in [Agent Memory](../concepts/agent-memory.md), and the way its evaluator and optimiser drive design-time fixes in [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md). It can wrap a workflow in [NeMo Guardrails](nemo-guardrails.md), and NVIDIA's [AI-Q Blueprint](ai-q-blueprint.md) is the largest public application built on it.

For an interview, know the concepts (YAML workflows, functions as tools, agent types, evaluation and profiling) and be able to say what replaced each piece.

## Interview Q&A

**What was the Agent Intelligence Toolkit renamed to?** NeMo Agent Toolkit in v1.2; the package and CLI became `nvidia-nat` and `nat`, and the old aliases were removed by 1.5 **[web]**.

**What agent types does it ship?** ReAct, Reasoning, ReWOO, Router, Tool Calling, and Parallel and Sequential executors **[sourced]**.

**How do you cap runaway behaviour in its ReAct agent?** With `max_tool_calls`, `tool_call_max_retries` and `parse_agent_response_max_retries` **[web]**.

**What is its status?** Deprecated in October 2026 with end of life by the end of 2026, replaced by Relay, Helix and Fabric **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-27 | [NeMo Agent Toolkit migration guide](https://docs.nvidia.com/aiqtoolkit/latest/resources/migration-guide.html) |
| 2026-10-28 | [NVIDIA AIQToolkit (GitHub)](https://github.com/NVIDIA/AIQToolkit) |
| 2026-10-28 | [NVIDIA NeMo Agent Toolkit (developer page)](https://developer.nvidia.com/nemo-agent-toolkit) |
| 2026-10-28 | [NVIDIA AIQ Toolkit docs](https://docs.nvidia.com/aiqtoolkit/index.html) |
| 2026-10-28 | [NeMo Helix release 0.1.0 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/release-0-1-0) |
| 2026-10-27 | [NeMo Agent Toolkit 1.5 migration guide](https://docs.nvidia.com/nemo/agent-toolkit/1.5/resources/migration-guide.html) |
| 2026-10-27 | [Agent Intelligence Toolkit is now the NeMo Agent Toolkit (NVIDIA forum)](https://forums.developer.nvidia.com/t/nvidia-agent-intelligence-toolkit-is-now-the-nvidia-nemo-agent-toolkit/335881) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [Futurum: GTC 2026 autonomous agent infrastructure](https://www.futurumgroup.com/insights/at-gtc-2026-nvidia-stakes-its-claim-on-autonomous-agent-infrastructure/) |
