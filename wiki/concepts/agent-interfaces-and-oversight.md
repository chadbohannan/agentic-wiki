# Agent Interfaces and Oversight UI

An agent interface is the layer through which people observe, steer and approve agent work. A good one makes the agent's state legible and controllable through trace views, plan previews, permission prompts, undo and a stop control, and it has to counter over-reliance rather than just collect clicks. The pause-and-resume mechanics behind its approval buttons are in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md); this page is about the surface.

## The protocol layer

AG-UI standardises the link between a frontend and an agent backend. Its documentation describes an open, event-based, bi-directional protocol that carries agent state, UI intents and user interactions over HTTP and WebSockets and streams intermediate work across multi-turn sessions. It lists first-party support in Microsoft Agent Framework, Google ADK, AWS Strands and Bedrock AgentCore, LlamaIndex and others, partnerships with LangChain and CrewAI, and in-progress OpenAI Agents SDK support, and it distinguishes itself from A2UI, a generative-UI specification for delivering widgets, which it says pairs well with it **[web]**. AG-UI is the user-facing leg of the three-protocol stack in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md), and the token and tool-progress events it carries are what [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md) tunes for responsiveness.

NVIDIA's customer-service blueprint shows a conventional arrangement: a chat interface with an optional digital human, and an operations UI where administrators review conversations and sentiment **[sourced]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) shipped an API server and a UI for launching and inspecting workflows **[web]**. In the 1.1 quick start the server exposes `/generate` and `/chat` endpoints, each with a streaming variant, and the docs recommend `/chat/stream` so the UI can show intermediate steps as the agent works **[sourced]**.

## What oversight requires

EU AI Act Article 14 gives a usable checklist, even outside the EU. Overseers of high-risk systems should be able to **[web]**:
- understand the system's capacities and limits, and monitor for anomalies;
- stay aware of automation bias;
- interpret outputs correctly;
- decide to disregard, override or reverse an output;
- intervene or stop the system through a "stop" button or similar procedure that brings it to a safe state.

These map to interface features:
- a trace or timeline view of what the agent did, drawn from [Agent Observability and Tracing](agent-observability-tracing.md);
- a plan preview, editable before execution;
- a diff or dry-run preview before edits;
- per-action permission prompts with a scope, such as "allow once" or "always for this repo";
- undo for reversible steps, and explicit marking of irreversible ones;
- a visible setting for the level of autonomy;
- a global stop.

The human behind the interface also needs the training, time and authority that [Human Oversight and Accountability](human-oversight-and-accountability.md) describes; a stop button nobody is empowered to press is decoration.

## Examples

Several products and protocols already implement parts of this:
- **Claude Code** shows each pending action in a permission prompt, offers "don't ask again" scoped to a repository or command or until the session ends, and provides a read-only plan mode **[web]**.
- **MCP clients** are told to show tool inputs before invocation and to indicate when a tool is running. For elicitation they must show which server is asking, and in URL mode the full URL before opening it **[web]**.
- **AG-UI** carries state and interrupts between agent and frontend **[web]**.

## Trade-offs

More prompts mean more safety on paper and approval fatigue in practice. Claude Code's auto mode replaces the human with a classifier for routine actions, trading human attention for model-based risk. Showing a rationale next to a recommendation can raise trust without raising correctness, because reasoning text is not a faithful explanation, as [Agent Transparency and Explainability](agent-transparency-and-explainability.md) shows. Undo is only as good as the tool: emails and payments cannot be rolled back, so gate them before execution, not after.

## Interview Q&A

**What does AG-UI standardise?** The streaming events exchanged between an agent backend and its frontend: state, tool progress, interrupts and user input **[web]**.

**What should an oversight interface show?** The current plan, tool calls with their inputs and outputs, state and cost, with controls to approve, edit, reject, undo and stop.

**How do autonomy levels work in practice?** As graded modes: prompting every time, auto-accepting edits, plan-only, classifier-approved, and full bypass, the last advised only in sandboxes **[web]**.

**Why is undo not enough?** Some side effects are irreversible. Gate those before execution, and make reversible ones cheap to roll back.

**Which Article 14 capabilities map to the interface?** Monitoring and understanding, interpreting outputs, overriding, a stop button, and awareness of automation bias **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [AG-UI introduction](https://docs.ag-ui.com/introduction) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-28 | [Launching the Agent Toolkit API Server and UI (1.1)](https://docs.nvidia.com/nemo/agent-toolkit/1.1/quick-start/launching-ui.html) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
| 2026-10-28 | [NVIDIA Agent Intelligence Toolkit (developer page)](https://developer.nvidia.com/agent-intelligence-toolkit) |
| 2026-10-28 | [EU AI Act Article 14: human oversight](https://artificialintelligenceact.eu/article/14/) |
| 2026-10-28 | [Claude Code permissions](https://code.claude.com/docs/en/permissions) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-28 | [MCP specification 2026-07-28: Elicitation](https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation) |
