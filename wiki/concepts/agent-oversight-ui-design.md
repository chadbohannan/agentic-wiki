# Agent Oversight UI Design

An oversight interface makes an agent's state legible and controllable. It uses trace views, plan previews, permission prompts, undo and a stop control, and its design has to counter over-reliance rather than just collect clicks. This page covers exam objective 10.1, focusing on what an oversight surface must contain. The protocols and streaming that carry it are covered separately.

The protocols, such as AG-UI, are in [Human-Agent Interaction UIs](human-agent-interaction-uis.md). The traces the interface displays come from [Agent Observability and Tracing](agent-observability-tracing.md), and the gates it presents are the mechanisms in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md).

## Requirements from Article 14

EU AI Act Article 14 gives a usable list of requirements, even outside the EU. Overseers of high-risk systems should be able to **[web, artificialintelligenceact.eu reproduction of the text]**:
- understand the system's capacities and limits, and monitor for anomalies;
- stay aware of automation bias;
- interpret outputs correctly;
- decide to disregard, override or reverse an output;
- intervene or stop the system through a "stop" button or similar procedure that brings it to a safe state.

These map to interface features **[inferred]**:
- a trace or timeline view of what the agent did and why;
- a plan preview, editable before execution;
- a diff or dry-run preview before edits;
- per-action permission prompts with a scope, such as "allow once" or "always for this repo";
- undo for reversible steps, and explicit marking of irreversible ones;
- a visible setting for the level of autonomy;
- a global stop.

## Examples

Several products and protocols already implement parts of this:
- **Claude Code** shows each pending action in a permission prompt. It offers "don't ask again" scoped to a repository or command, or until the session ends, and provides a read-only plan mode **[web]**.
- **MCP clients** are told to show tool inputs before invocation and to indicate clearly when a tool is running. For elicitation they MUST show which server is asking, and in URL mode they MUST show the full URL before opening it **[web]**.
- **The NeMo Agent Toolkit** shipped an API server, a UI, and observability exporters for LangSmith, Phoenix, Weave, Langfuse and OpenTelemetry **[web, feature list]**.
- **AG-UI** standardises the event stream that carries state and interrupts between agent and front end. Its human-in-the-loop page returned 404 when checked **[web]**.

## Trade-offs

More prompts mean more safety on paper but approval fatigue in practice. Claude Code's auto mode replaces the human with a classifier for routine actions, trading human attention for model-based risk **[web for the mode; trade-off inferred]**. Showing a rationale next to a recommendation can raise trust without raising correctness, as [Agent Transparency and Explainability](agent-transparency-and-traceability.md) discusses. Undo is only as good as the tool: emails and payments cannot be rolled back, so gate them before execution, not after **[inferred]**.

## Interview Q&A

**What should an oversight interface show?** The current plan, tool calls with their inputs and outputs, state and cost. It needs controls to approve, edit, reject, undo and stop **[inferred, web]**.

**How do autonomy levels work in practice?** As graded modes: prompting every time, auto-accepting edits, plan-only, classifier-approved, and full bypass. Full bypass is advised only in sandboxes **[web]**.

**Why is undo not enough?** Some side effects are irreversible. Gate those before execution, and make reversible ones cheap to roll back **[inferred]**.

**Which Article 14 capabilities map to the interface?** Monitoring and understanding, interpreting outputs, overriding, a stop button, and awareness of automation bias **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA Agent Intelligence Toolkit (developer page)](https://developer.nvidia.com/agent-intelligence-toolkit) |
| 2026-10-28 | [EU AI Act Article 14: human oversight](https://artificialintelligenceact.eu/article/14/) |
| 2026-10-28 | [Claude Code permissions](https://code.claude.com/docs/en/permissions) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-28 | [MCP specification 2026-07-28: Elicitation](https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation) |
| 2026-10-28 | [AG-UI introduction](https://docs.ag-ui.com/introduction) |
