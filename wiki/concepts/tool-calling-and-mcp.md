# Tool Calling and MCP

Tool calling is how an agent acts on the world: the model is given tool definitions, returns calls with arguments, and the host executes them and feeds results back. This page covers exam objective 2.3: function-calling mechanics, tool design, MCP as the standard integration protocol, and the reliability settings that stop a tool loop going wrong.

MCP's wider role is expanded in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md), and typed tool arguments depend on [Structured Outputs](structured-outputs.md). The retry and circuit-breaker policies applied to tool failures are in [Agent Error Handling and Resilience](agent-error-handling-resilience.md), and the loop that decides when to call tools is [ReAct: Reasoning and Acting](react-reasoning-action.md). [NeMo Guardrails](../components/nemo-guardrails.md) can enforce safety rules on calls, and injection and permission risks are covered in [Agent Security Threats and Controls](agent-security-threats-and-controls.md).

## Function-calling mechanics

The mechanics are the same across providers: the request carries `tools` (a name, a description and JSON-schema parameters) and optionally `tool_choice`, the model returns one or more tool calls, the application executes them and returns the results, and the loop repeats **[web]**. NVIDIA's NIM documentation says `tool_choice` is valid only alongside `tools`, accepts `none`, `auto` or a named function, and that `required` appears in older LLM docs and VLM NIM docs from 1.6; it also documents streamed and parallel tool calls and says tool calling is auto-enabled for model families including Llama 3.1 to 3.3, Mistral, GPT-OSS, Nemotron 3 Nano and the Llama Nemotron models with reasoning off. These NIM details come from search summaries because fetching the page returned only an outline, and the supported list varies by NIM version, so check the version in use **[web, snippet-level]**. The [NVIDIA NIM](../components/nvidia-nim.md) page covers the serving side.

## Designing tools

Anthropic advises treating the tool interface like a prompt: clear descriptions with usage examples and boundaries between similar tools, formats the model has seen often, and arguments designed so mistakes are hard (requiring absolute paths removed a class of errors in its coding agent). Its context-engineering guidance adds a small, non-overlapping tool set with token-efficient outputs **[web]**. The oft-quoted heuristic that if a human cannot say which tool applies the agent cannot either comes from secondary summaries of that guidance.

## MCP

The tools section of the MCP specification (current revision 2026-07-28, read in full) defines `tools/list`, which is paginated and now carries a cache time-to-live, and `tools/call`. A tool has a `name`, optional `title`, a `description`, an `inputSchema` (JSON Schema, 2020-12 by default) and an optional `outputSchema`, and its `annotations` must be treated as untrusted unless the server is trusted. Results can contain text, image, audio, resource links or embedded resources, plus `structuredContent`; when an output schema is declared, servers must return conforming structured content and clients should validate it **[web]**. The specification states that `structuredContent` is unrelated to LLM schema-constrained generation. A human should always be able to deny tool invocations, servers must validate inputs, enforce access control, rate-limit and sanitise outputs, and clients should confirm sensitive operations, apply timeouts and log usage for audit.

Two parts of this revision differ from what earlier material, including the pages this wiki first drew on, describes. The protocol no longer has a connection-level session, so a stateful tool should return an explicit handle (a basket identifier, for instance) that the model carries into later calls, and the specification offers non-normative guidance on handle authorisation, opacity, lifetime and expiry errors. A server can also answer `tools/call` with an input-required result that asks the client for elicitation and retries with the answers. The earlier `2025-11-25` text listed an `execution.taskSupport` field, which does not appear in the 2026-07-28 tools page **[web]**.

MCP separates two error channels. Protocol errors (unknown tool, malformed request) come back as JSON-RPC errors that a model is unlikely to fix, while tool execution errors come back with `isError: true` and actionable text, and clients should pass those to the model so it can adjust and retry **[web]**.

## In the NeMo Agent Toolkit

In the toolkit, tools are functions or function groups registered by plugins and referenced by name in YAML, and it acts as both MCP client and server and supports A2A **[sourced]**. Its migration guides record the MCP CLI moving to `nat mcp ...` with streamable-http at `/mcp` as the default transport (v1.3), `mcp_tool_wrapper` being superseded by `mcp_client`, and the function-group separator changing from `.` to `__` (v1.4) **[web]**. The 1.3 ReAct-agent documentation lists reliability settings: `retry_agent_response_parsing_errors` (default true), `parse_agent_response_max_retries` (1), `tool_call_max_retries` (1) and `max_tool_calls` (15), read from a search summary, so defaults may differ by version **[web, snippet-level]**. See [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) for its deprecation.

## Interview Q&A

**What does `tool_choice` do?** It controls whether the model must, may or must not call tools: none, auto, or a forced named function, with `required` on some APIs **[web]**.

**How should a tool report failure?** Return an actionable error in the result (`isError: true` with a clear message) instead of a protocol error, so the model can correct itself **[web]**.

**Why keep the tool set small?** Overlapping tools make selection ambiguous and bloat context **[web]**.

**How do you stop a ReAct agent looping on bad calls?** Cap tool calls and retries (`max_tool_calls`, `tool_call_max_retries`, `parse_agent_response_max_retries` in the toolkit), and use timeouts and step budgets **[web]**.

**MCP versus plain function calling?** Function calling is the model-side interface; MCP standardises how tools are discovered and invoked across servers and clients, so one tool works across frameworks **[inferred]**.

**How does an MCP tool keep state without sessions?** It returns an opaque handle from a creation tool and takes it as an argument afterwards **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-27 | [MCP specification 2025-11-25: Tools](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) |
| 2026-10-27 | [NIM for LLMs: function calling](https://docs.nvidia.com/nim/large-language-models/latest/function-calling.html) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [NeMo Agent Toolkit migration guides](https://docs.nvidia.com/nemo/agent-toolkit/1.4/resources/migration-guide.html) |
| 2026-10-27 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-27 | [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
