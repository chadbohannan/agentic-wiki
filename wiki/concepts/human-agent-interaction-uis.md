# Human-Agent Interaction UIs

Agent interfaces are the layer through which humans observe, steer and approve agent work. This page covers exam objective 1.1: the protocol that standardises the agent-to-frontend link, and what NVIDIA's reference designs provide.

The AG-UI documentation describes AG-UI as an open, lightweight, event-based protocol for a bi-directional link between a frontend and any agentic backend, carrying agent state, UI intents and user interactions over HTTP and WebSockets and streaming intermediate work across multi-turn sessions. It lists first-party support in Microsoft Agent Framework, Google ADK, AWS Strands and Bedrock AgentCore, LlamaIndex and others, partnerships with LangChain and CrewAI, and in-progress OpenAI Agents SDK support. It also separates AG-UI from A2UI, a generative-UI specification for delivering widgets, and says the two work well together **[web, read in full]**. AG-UI's place in the stack alongside MCP and A2A is described in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md), and the streaming and tool-progress events that make these interfaces responsive are covered in [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md).

NVIDIA's customer-service blueprint shows a conventional arrangement: a chat interface with an optional digital human, and an operations pipeline UI for administrators who review conversations and sentiment **[sourced]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ships an API server and UI for launching and inspecting workflows **[web]**.

The design concerns that follow are streaming partial output, exposing tool calls and traces, approval gates for human-in-the-loop control, and transparency **[inferred]**. They overlap with section 10 and are developed in [Agent Oversight UI Design](agent-oversight-ui-design.md).

## Interview Q&A

**What does AG-UI standardise?** The streaming events exchanged between an agent backend and its frontend **[web]**.

**Which UI features support oversight?** Trace visibility, approval gates, and interrupt and resume **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [AG-UI introduction](https://docs.ag-ui.com/introduction) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
