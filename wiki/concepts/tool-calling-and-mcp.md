# Tool Calling and MCP

Tool calling is how an agent acts on the world: the model is given tool definitions, returns calls with arguments, and the host executes them and feeds results back. It is also the agent's largest attack surface, because every tool is a capability that injected instructions can borrow; the threats and controls are in [Agent Security Threats and Controls](agent-security-threats-and-controls.md).

## Function-calling mechanics

The mechanics are the same across providers: the request carries `tools` (a name, a description and JSON-schema parameters) and optionally `tool_choice`, the model returns one or more tool calls whose arguments should conform to the declared schema (the guarantee discussed in [Structured Outputs](structured-outputs.md)), the application executes them and returns the results, and the loop repeats, which is the [ReAct](react-reasoning-action.md) loop in its provider-native form. In the NIM 1.15 docs, `tool_choice` is valid only alongside `tools` and accepts `none`, `auto` or a named function, and tool calling is enabled automatically in model-specific containers for families including Llama 3.1 to 3.3, Mistral, Qwen2.5, GPT-OSS, DeepSeek-V3.x, Nemotron 3 Nano and the Llama Nemotron models with detailed thinking off. In NIM 2.x the operator enables it at startup with an engine-specific parser, as the [NVIDIA NIM](../components/nvidia-nim.md) page describes; on vLLM a request with `"tool_choice": "auto"` fails unless both `--enable-auto-tool-choice` and `--tool-call-parser` were set. NIM never talks to MCP servers: the client fetches MCP tool definitions, converts them to the OpenAI `tools` format and runs the calls against the MCP server **[web]**.

## Designing tools

A tool is an interface for a model, and designing it well probably matters as much as the prompt. Anthropic's "Building Effective Agents" asks for as much effort on the agent-computer interface as on a human-computer one, and its advice brings tool definitions under the same craft as the instructions in [Context Engineering](context-engineering.md): clear descriptions with usage examples and boundaries between similar tools, formats the model has seen often, and arguments that make mistakes hard. Its example is a coding agent that kept mishandling relative paths until the tool required absolute ones, after which the model "used this method flawlessly" **[web]**.

Anthropic's later guide to writing tools (September 2025) calls a tool "a contract between deterministic systems and non-deterministic agents" and turns that into specific rules **[web]**:

- **Build tools for tasks, not endpoints.** A tool that wraps an API call one-to-one usually returns too much and needs chaining. The guide prefers `schedule_event` to separate `list_users`, `list_events` and `create_event` tools, and `search_contacts` to `list_contacts`.
- **Namespace related tools** by service and resource, such as `asana_search` and `asana_projects_search`. The guide notes that prefix and suffix naming perform differently and the better choice varies by model.
- **Return meaningful context.** Prefer names and types to UUIDs and MIME types; resolving opaque identifiers to readable names or small indexes reduced hallucinated identifiers. A `response_format` option lets the agent ask for concise or detailed output; in the guide's Slack example the concise form used 72 tokens against 206.
- **Be token-efficient.** Paginate, filter and truncate with sensible defaults; Claude Code caps a tool response at 25,000 tokens by default. When truncating, say so and suggest a narrower query.
- **Write actionable errors**, not opaque codes or tracebacks, so the model can correct the call.
- **Describe the tool as to a new hire**, making implicit conventions explicit, and use unambiguous parameter names such as `user_id` rather than `user`.

The guide reports that small description changes had large effects. Refined tool descriptions helped Claude Sonnet 3.5 reach state of the art on SWE-bench Verified, and a fix to a search tool's description stopped the model appending "2025" to every query **[web]**. Its method is evaluation-driven: build a prototype, write realistic tasks that need many tool calls with verifiable outcomes, run them in an agent loop, track tool calls, tokens and errors, and refine against a held-out set **[web]**. That is the same discipline as [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), applied to the tool layer. The context-engineering guidance adds that a bloated or overlapping tool set is a common failure, and that if a human cannot say which tool applies, the agent probably cannot either **[web]**. Large tool sets also cost context on every step, which is why [Context Engineering](context-engineering.md) covers selecting or masking tools rather than offering all of them.

## MCP

The tools section of the MCP specification (current revision 2026-07-28) defines `tools/list`, which is paginated and now carries a cache time-to-live, and `tools/call`. A tool has a `name`, optional `title`, a `description`, an `inputSchema` (JSON Schema, 2020-12 by default) and an optional `outputSchema`, and its `annotations` must be treated as untrusted unless the server is trusted. Results can contain text, image, audio, resource links or embedded resources, plus `structuredContent`; when an output schema is declared, servers must return conforming structured content and clients should validate it **[web]**. The specification states that `structuredContent` is unrelated to LLM schema-constrained generation. A human should always be able to deny tool invocations, servers must validate inputs, enforce access control, rate-limit and sanitise outputs, and clients should confirm sensitive operations, apply timeouts and log usage for audit. Versioning, transports and governance of the protocol as a whole are in [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md).

Two parts of this revision differ from most earlier material. The protocol no longer has a connection-level session, so a stateful tool should return an explicit handle (a basket identifier, for instance) that the model carries into later calls, and the specification offers non-normative guidance on handle authorisation, opacity, lifetime and expiry errors. A server can also answer `tools/call` with an input-required result that asks the client for elicitation and retries with the answers. The earlier `2025-11-25` text listed an `execution.taskSupport` field, which does not appear in the 2026-07-28 tools page **[web]**.

MCP separates two error channels. Protocol errors (unknown tool, malformed request) come back as JSON-RPC errors that a model is unlikely to fix, while tool execution errors come back with `isError: true` and actionable text, and clients should pass those to the model so it can adjust and retry **[web]**. Retrying infrastructure failures, as opposed to model-fixable ones, follows the backoff and circuit-breaker rules in [Agent Error Handling and Resilience](agent-error-handling-resilience.md), and policy on what a call may do can be enforced by the execution rails in [NeMo Guardrails](../components/nemo-guardrails.md).

## In NVIDIA's stack

In the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md), tools are functions or function groups registered by plugins and referenced by name in YAML, and the toolkit acts as both MCP client and server **[sourced]**. Its migration guides record the MCP CLI moving to `nat mcp ...` with streamable-http at `/mcp` as the default transport (v1.3), `mcp_tool_wrapper` being superseded by `mcp_client`, and the function-group separator changing from `.` to `__` (v1.4) **[web]**. The 1.3 ReAct-agent documentation lists reliability settings: `retry_agent_response_parsing_errors` (default true), `parse_agent_response_max_retries` (1), `tool_call_max_retries` (1) and `max_tool_calls` (15), and defaults may differ by version **[web, snippet]**. See [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) for its deprecation.

## Interview Q&A

**What does `tool_choice` do?** It controls whether the model must, may or must not call tools: none, auto, or a forced named function, with `required` on some APIs **[web]**.

**How should a tool report failure?** Return an actionable error in the result (`isError: true` with a clear message) instead of a protocol error, so the model can correct itself **[web]**.

**Why keep the tool set small?** Overlapping tools make selection ambiguous and bloat context **[web]**.

**What makes a good tool for an agent?** One that does a whole task rather than wrapping one endpoint, returns readable, token-efficient output, fails with actionable messages, and has a description written as for a new hire **[web]**.

**How do you stop a ReAct agent looping on bad calls?** Cap tool calls and retries (`max_tool_calls`, `tool_call_max_retries`, `parse_agent_response_max_retries` in the toolkit), and use timeouts and step budgets **[web]**.

**MCP versus plain function calling?** Function calling is the model-side interface; MCP standardises how tools are discovered and invoked across servers and clients, so one tool works across frameworks.

**How does an MCP tool keep state without sessions?** It returns an opaque handle from a creation tool and takes it as an argument afterwards **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-27 | [MCP specification 2025-11-25: Tools](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) |
| 2026-10-28 | [NIM for LLMs 1.15: function calling](https://docs.nvidia.com/nim/large-language-models/1.15.0/function-calling.html) |
| 2026-10-28 | [NIM for LLM and VLM: tool calling and MCP integration](https://docs.nvidia.com/nim/large-language-models/latest/advanced-use-cases/tool-calling-and-mcp.html) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [NeMo Agent Toolkit migration guides](https://docs.nvidia.com/nemo/agent-toolkit/1.4/resources/migration-guide.html) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-28 | [Anthropic: Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) |
| 2026-10-27 | [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
