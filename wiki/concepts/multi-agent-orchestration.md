# Multi-Agent Orchestration

Multi-agent systems split work across several LLM agents arranged as supervisor, hierarchical, decentralised or federated topologies. This page covers exam objective 1.5: the topologies, how agents execute and hand off, and the frameworks that implement them as of late 2026.

Agent-to-agent hand-offs across systems rely on the protocols in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md), and the shared state agents need is covered in [Stateful Orchestration](stateful-orchestration.md). Whether to use several agents at all is the workflow-versus-agent decision in [Agent Architecture Patterns](agent-architecture-patterns.md), and NVIDIA's [AI-Q Blueprint](../components/ai-q-blueprint.md) is a worked example.

## Topologies and execution

NVIDIA's multi-agent glossary lists centralised (supervisor), decentralised, federated and hierarchical topologies with a trade-off for each, and describes execution as sequential, parallel or ask-for-help, with per-agent instructions, tools, memory and permissions and an optional model router, which it calls a "system of models" **[sourced]**. Anthropic's orchestrator-workers pattern is the supervisor idea at the LLM level, and Anthropic's own advice is to prefer the simplest solution **[web]**.

NVIDIA's examples show the range. Its customer-service blueprint uses a single LangGraph planning agent plus retrieval **[sourced]**, whereas AI-Q uses a planner, concurrent researcher workers and a writer, according to the AI-Q documentation snippet **[web]**. The study guide also cites a financial-modelling example in which a judge agent coordinates specialist agents with a human in the loop, but the arXiv link it gives for that example (2502.05439) resolves to a financial-services model-risk paper, so the claim rests on the guide's description alone **[sourced, with a flagged reference error]**.

Parallelism and specialisation are the benefits; coordination overhead, token cost, error propagation and harder debugging are the costs **[inferred]**.

## Frameworks

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) is framework-agnostic, with plugins for ADK, Agno, AutoGen, CrewAI, LangChain/LangGraph, LlamaIndex, Semantic Kernel and Strands **[web]**. Beyond NVIDIA, two 2026 comparison blogs report that AutoGen is in maintenance mode with Microsoft Agent Framework (reported at 1.0 in April 2026) as its successor, that CrewAI uses role-based crews run sequentially or hierarchically, that the OpenAI Agents SDK coordinates through handoffs, and that LangGraph uses subgraphs and supervisor graphs **[web, search snippets from blogs; verify against vendor documentation before relying on versions]**.

## Interview Q&A

**Centralised versus decentralised?** Central control with a possible bottleneck, against flexibility with more coordination complexity **[sourced]**.

**When should you avoid multi-agent designs?** When one agent with tools suffices, since extra agents add cost and failure surface **[inferred, supported by Anthropic's simplicity advice]**.

**How do agents hand off?** Through handoffs in the OpenAI SDK, graph edges in LangGraph, or A2A tasks across systems **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [What Are Multi-Agent Systems? (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/multi-agent-systems/) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-27 | [arXiv 2502.05439 (mismatched reference)](https://arxiv.org/abs/2502.05439) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) |
| 2026-10-27 | [Best multi-agent frameworks 2026 (futureagi)](https://futureagi.com/blog/best-multi-agent-frameworks-2026) |
| 2026-10-27 | [Best multi-agent frameworks 2026 (gurusup)](https://gurusup.com/blog/best-multi-agent-frameworks-2026) |
