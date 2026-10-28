# Multi-Agent Orchestration

Multi-agent systems split work across several LLM agents arranged as supervisor, hierarchical, decentralised or federated topologies. Whether to use several agents at all is the workflow-versus-agent decision in [Agent Architecture Patterns](agent-architecture-patterns.md); the answer is often no.

## Topologies and execution

NVIDIA's multi-agent glossary lists centralised (supervisor), decentralised, federated and hierarchical topologies with a trade-off for each, and describes execution as sequential, parallel or ask-for-help, with per-agent instructions, tools, memory and permissions and an optional model router, which it calls a "system of models" **[sourced]**. Anthropic's orchestrator-workers pattern is the supervisor idea at the LLM level, and Anthropic's own advice is to prefer the simplest solution **[web]**. The federated case, where agents belong to different organisations, is the natural fit for the A2A protocol in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md).

NVIDIA's examples show the range. Its customer-service blueprint uses a single LangGraph planning agent plus retrieval **[sourced]**, whereas the [AI-Q Blueprint](../components/ai-q-blueprint.md) uses a planner, concurrent researcher workers and a writer **[web, snippet]**. The study guide also cites a financial-modelling example in which a judge agent coordinates specialist agents with a human in the loop, but the arXiv link it gives for that example (2502.05439) resolves to a financial-services model-risk paper, so the claim rests on the guide's description alone **[sourced]**.

Parallelism and specialisation are the benefits; coordination overhead, token cost, error propagation and harder debugging are the costs. Anthropic puts a number on the token cost from its research system: agents use about 4× the tokens of a chat interaction and multi-agent systems about 15×, so the task's value has to justify the spend **[web]**. Agents that share a task also need shared state, which graph runtimes such as LangGraph provide as checkpointed thread state and subgraphs, described in [Stateful Orchestration](stateful-orchestration.md).

## Frameworks

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) is framework-agnostic, with plugins for ADK, Agno, AutoGen, CrewAI, LangChain/LangGraph, LlamaIndex, Semantic Kernel and Strands **[web]**. Beyond NVIDIA, two 2026 comparison blogs report that AutoGen is in maintenance mode with Microsoft Agent Framework (reported at 1.0 in April 2026) as its successor, that CrewAI uses role-based crews run sequentially or hierarchically, that the OpenAI Agents SDK coordinates through handoffs, and that LangGraph uses subgraphs and supervisor graphs **[web, snippet]**. These are blog claims; check versions against vendor documentation before relying on them.

## Interview Q&A

**Centralised versus decentralised?** Central control with a possible bottleneck, against flexibility with more coordination complexity **[sourced]**.

**When should you avoid multi-agent designs?** When one agent with tools suffices, and when the work needs every agent to share the same context or has many dependencies between agents. Anthropic names those domains as a poor fit and notes that most coding tasks parallelise less than research does **[web]**.

**How do agents hand off?** Through handoffs in the OpenAI SDK, graph edges in LangGraph, or A2A tasks across systems **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [What Are Multi-Agent Systems? (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/multi-agent-systems/) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-27 | [arXiv 2502.05439 (mismatched reference)](https://arxiv.org/abs/2502.05439) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) |
| 2026-10-27 | [Best multi-agent frameworks 2026 (futureagi)](https://futureagi.com/blog/best-multi-agent-frameworks-2026) |
| 2026-10-27 | [Best multi-agent frameworks 2026 (gurusup)](https://gurusup.com/blog/best-multi-agent-frameworks-2026) |
| 2026-10-28 | [How we built our multi-agent research system (Anthropic)](https://www.anthropic.com/engineering/multi-agent-research-system) |
