# Agent Protocols: MCP, A2A, AG-UI

Three open protocols divide the integration problem: MCP connects agents to tools and data, A2A connects agents to other agents, and AG-UI connects agents to users. This page covers exam objective 1.3 and states protocol versions as checked on 2026-10-28.

The AG-UI documentation presents the three as a stack, crediting MCP to Anthropic and A2A to Google **[web]**. MCP is the protocol side of the interface described in [Tool Calling and MCP](tool-calling-and-mcp.md), and A2A extends the topologies in [Multi-Agent Orchestration](multi-agent-orchestration.md) across organisational boundaries. AG-UI is the user-facing leg, discussed in [Human-Agent Interaction UIs](human-agent-interaction-uis.md), and MCP's security considerations are in [Agent Security Threats and Controls](agent-security-threats-and-controls.md).

## MCP

MCP is an open protocol for exposing tools, resources and prompts to agents. Its specification uses date-string revisions. The versioning page on modelcontextprotocol.io lists **2026-07-28** as the current revision, and it describes a stateless model: each request declares its protocol version in `_meta` (and in an `MCP-Protocol-Version` header on Streamable HTTP), a server can reject unsupported versions with an error that lists the ones it supports, and a `server/discover` call returns versions and capabilities. Revisions up to `2025-11-25` used an initialization handshake and remain supported for backward compatibility **[web, read in full]**. This resolves an earlier uncertainty: an O'Reilly article had cited a 2026-07-28 revision, and the specification itself confirms it. The same article reports that governance moved to the Agentic AI Foundation, a Linux Foundation directed fund, in December 2025 **[web, secondary]**. The `2025-11-25` revision added experimental Tasks and URL elicitation **[web, search snippet]**.

## A2A

The A2A specification's latest released version is **1.0.0**, after 0.3.0, 0.2.6 and 0.1.0, and versions are identified by major.minor only **[web, read in full]**. Its core objects are the Agent Card (a JSON identity document with skills, endpoint and authentication), Task (an identifier plus a lifecycle), Message (a role plus Parts), Part (text, file or data) and Artifact (a task output), with context grouping and extensions on top. It defines JSON-RPC, gRPC and HTTP+JSON bindings plus a route for custom ones. The migration appendix lists two breaking changes: removal of the `kind` discriminator and relocation of the extended Agent Card field. Long-running work is tracked by polling, streaming or webhook push **[web]**. The Linux Foundation has hosted A2A since June 2025 and an April 2026 press release cites more than 150 supporting organisations; one secondary source dates v1.0 to March 2026 **[web, snippet]**.

## In NVIDIA's stack

The NeMo Agent Toolkit is both an MCP client and server (including FastMCP) and an A2A client and server (`nvidia-nat-a2a`, `nat a2a serve`), with authentication for each **[web]**; see [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) for the deprecation status. NVIDIA's AI Factory white paper says agents use MCP to discover tools and that A2A and MCP traffic should be handled at gateways, with interim microservices until standard gateway APIs exist **[sourced]**. NVIDIA's multi-agent glossary adds that federated multi-agent systems collaborate across organisations through shared protocols **[sourced]**.

A useful rule of thumb is to use MCP when an agent calls a deterministic tool and A2A when it delegates to an opaque peer agent that has its own reasoning and long-running tasks **[inferred]**.

## Interview Q&A

**MCP versus A2A?** MCP is vertical (agent to tool), A2A is horizontal (agent to agent) **[web]**.

**What is an Agent Card?** A discovery document describing an agent's capabilities, endpoint and authentication **[web]**.

**How does A2A handle long tasks?** Through a task lifecycle that clients follow by polling, streaming or push webhooks **[web]**.

**How does the NeMo Agent Toolkit use them?** It acts as client and server for both **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [MCP specification versioning](https://modelcontextprotocol.io/specification/versioning) |
| 2026-10-28 | [A2A Protocol specification](https://a2a-protocol.org/latest/specification/) |
| 2026-10-27 | [AG-UI introduction](https://docs.ag-ui.com/introduction) |
| 2026-10-27 | [O'Reilly Radar on MCP governance](https://www.oreilly.com/radar/?p=19654) |
| 2026-10-27 | [NeMo Agent Toolkit A2A integration](https://docs.nvidia.com/nemo/agent-toolkit/1.7/components/integrations/a2a.html) |
| 2026-10-27 | [Agentic AI in the Factory (NVIDIA AI Factory white paper)](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html) |
| 2026-10-27 | [What Are Multi-Agent Systems? (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/multi-agent-systems/) |
