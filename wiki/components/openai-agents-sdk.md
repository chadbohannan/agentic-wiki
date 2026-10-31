# OpenAI Agents SDK

The OpenAI Agents SDK is a lightweight, Python-first framework for building agents from a few primitives: agents, handoffs, guardrails, sessions and tracing. Its documentation describes it as "a production-ready upgrade" of OpenAI's earlier experimental project, Swarm **[web]**. As of 2026-10-28 the latest release is 0.24.0 under the MIT licence, and it needs Python 3.10 or later, so it is still pre-1.0 and its interfaces may move **[web]**. It is the smallest of the three frameworks in the wiki, which makes it a good reference for what an agent harness minimally contains.

## Primitives

An `Agent` is an LLM with instructions and tools. Its key fields are `instructions` (a string, or a function of the run context that returns one), `tools`, `handoffs`, `model` and `model_settings`, `output_type` for structured output, `input_guardrails` and `output_guardrails`, `hooks` for lifecycle callbacks, and `tool_use_behavior` **[web]**. Setting `output_type` to a Pydantic model, dataclass, `TypedDict` or list switches the model to structured outputs, the mechanism covered in [Structured Outputs](../concepts/structured-outputs.md). A context object passed to the runner is available to every agent, tool and handoff through `RunContextWrapper.context`, which is the place for application state and dependencies, kept separate from anything the model sees **[web]**.

Function tools are Python functions turned into tools with generated schemas and Pydantic validation, and the SDK also supports MCP servers and hosted tools alongside them **[web]**. The design rules for what a good tool looks like are in [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md). `tool_use_behavior` decides what happens after a tool returns: the default `run_llm_again` sends results back to the model, `stop_on_first_tool` makes the first tool's output the final answer, and `StopAtTools` ends the run when a named tool is called. By default `tool_choice` resets to `auto` after a tool call, which the documentation says prevents infinite loops **[web]**.

## The run loop

`Runner.run`, `run_sync` and `run_streamed` execute the loop, which the documentation states plainly **[web]**:
1. Call the model for the current agent with the current input.
2. If the output is final, meaning text of the desired type with no tool calls, return it.
3. If the output is a handoff, switch the current agent and input and loop.
4. If the output contains tool calls, run them, append the results and loop.

A `max_turns` limit raises `MaxTurnsExceeded` when exceeded, and `max_turns=None` removes it. `error_handlers` for `max_turns`, `model_refusal` and `invalid_final_output` return a controlled final output instead of raising. The exception hierarchy separates model timeouts, tool timeouts, malformed model output and guardrail tripwires, which lets a caller apply different recovery rules in the manner of [Agent Error Handling and Resilience](../concepts/agent-error-handling-resilience.md) **[web]**. This is the loop described in [ReAct and the Agent Loop](../concepts/react-reasoning-action.md), packaged: the turn budget is built in, while the pages read describe no token or cost budget.

## Two ways to delegate

The SDK offers two multi-agent primitives, and the choice between them is the main design decision **[web]**.

- **Agents as tools.** A manager agent calls specialists through `agent.as_tool(...)` and keeps control of the conversation. The documentation recommends this when one agent should own the final answer, combine several specialists' outputs or enforce shared guardrails.
- **Handoffs.** Each handoff is presented to the model as a tool, named by default `transfer_to_<agent_name>`, and the receiving agent takes over the rest of the turn with the conversation history. Options include an `input_type` schema for model-supplied metadata, an `on_handoff` callback, an `is_enabled` flag and an `input_filter` that transforms the history the next agent receives. Nested handoff history is an opt-in beta that compacts earlier turns into a summary block; the documentation notes it changes representation only and does not redact sensitive data.

The documentation also separates LLM-driven orchestration, where the model plans and delegates, from code-driven orchestration, where structured outputs route between agents, agents are chained, a task agent loops with an evaluator, or independent agents run in parallel with `asyncio.gather` **[web]**. Code-driven flows give more predictable speed and cost, and the two can be mixed. These correspond to the workflow-versus-agent split in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md), and the topologies in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md): a triage agent with handoffs is a decentralised hand-off design, an agents-as-tools manager is a supervisor.

## MCP and streaming

MCP servers attach in four ways: a hosted MCP tool that OpenAI's Responses API calls on the model's behalf, streamable HTTP, stdio for local subprocesses, and HTTP with server-sent events, which the documentation marks deprecated **[web]**. Tool lists can be filtered with static allow and block lists or a dynamic callable, `require_approval` can be set per server or tool, and tool-list caching is opt-in. Hosted tools do not support client-side tool guardrails because the Responses API executes them, and the documentation warns that connecting to untrusted servers exposes data and actions, which is the risk in [Agent Security Threats and Controls](../concepts/agent-security-threats-and-controls.md) **[web]**. `Runner.run_streamed` yields raw Responses API events (including text deltas), completed run items such as tool calls, and a notice when a handoff changes the current agent **[web]**.

## Guardrails

Input guardrails check the first agent's input and output guardrails check the last agent's final output, each returning a result whose `tripwire_triggered` flag raises an exception **[web]**. An input guardrail runs in parallel with the agent by default for the lowest latency, at the cost that the agent may already have spent tokens or run tools before the tripwire cancels it; setting `run_in_parallel=False` makes it blocking, so nothing runs until it passes. Tool guardrails attach to function tools and can allow a call, replace its output or raise a tripwire. The documented limits are the important part: input guardrails do not run on later agents in a handoff chain, output guardrails do not run on earlier agents, and tool guardrails do not cover handoffs, hosted tools or built-in execution tools such as the shell and computer tools **[web]**. A designer who puts a safety check only on the entry agent has therefore not protected the specialists it hands off to. The layering strategy these checks belong to is in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md).

Pausing for approval uses `needs_approval` on a tool and a serialisable run state, described in [Human-in-the-Loop Patterns](../concepts/human-in-the-loop-patterns.md).

## Sessions and state

A session gives client-side conversation memory: before each run the runner loads the session's history, and afterwards it stores the new items **[web]**. Backends include SQLite, Redis, SQLAlchemy databases, MongoDB, Dapr, a server-side OpenAI Conversations session, a compaction wrapper and an encryption wrapper. `session_input_callback` lets a caller prune or reorder history before it reaches the model, and `SessionSettings(limit=N)` caps how much is fetched. The alternatives are manual `result.to_input_list()` history or server-managed `conversation_id` and `previous_response_id`, which cannot be combined with a session in the same run. Two documented warnings carry over to any design: a session ID selects history but does not authenticate the user, and the SQLite session does not detect tampering with stored rows **[web]**. Sessions are conversation memory in the sense of [Agent Memory](../concepts/agent-memory.md); the pages read describe no checkpointed graph state of the kind in [LangGraph](langgraph.md), so resuming mid-run appears to depend on the serialised run state used for approvals. The overview also lists sandbox agents, which run specialists "inside real isolated workspaces" with manifest-defined files and resumable sessions **[web]**; that is the SDK's answer to the environment part of [Anatomy of an Agent](../syntheses/agent-anatomy.md), which the wiki otherwise covers only through [Agent Security Threats and Controls](../concepts/agent-security-threats-and-controls.md).

## Tracing and models

Tracing is on by default. A trace wraps each runner call and contains spans for agents, model generations, function calls, guardrails and handoffs, plus speech spans for voice pipelines **[web]**. The default exporter sends to OpenAI's backend, `add_trace_processor` adds another destination, and `set_trace_processors` replaces the defaults, which is the way to redact before export. By default the traces capture model and tool inputs and outputs, a setting turned off with `trace_include_sensitive_data` or its environment variable. The documentation states that tracing is unavailable to organisations on a Zero Data Retention policy, and lists dozens of external integrations, including Langfuse, Arize Phoenix, MLflow, Datadog and LangSmith, which it says are maintained by their own vendors and are not endorsed by OpenAI **[web]**. The span model is comparable to the one in [Agent Observability and Tracing](../concepts/agent-observability-tracing.md), though the SDK uses its own trace format rather than the OpenTelemetry GenAI conventions.

The SDK uses the Responses API for OpenAI models, and a chat-completions model class is the usual route to other providers **[web]**. Third-party adapters for LiteLLM and any-llm are beta and best-effort, and the documentation lists provider gaps: no Responses API, weaker structured-output support, and Responses-only fields silently dropped unless `strict_feature_validation` is set. Running a non-OpenAI model therefore needs tracing disabled or redirected, and testing for these gaps.

## Trade-offs

The SDK has the fewest concepts of the three frameworks, and most of its behaviour is visible in a short script. The cost is that anything beyond conversation memory and a turn limit is the developer's job: the pages read describe no checkpointed state and no graph structure to visualise, and some defaults favour convenience over caution, such as tracing capturing model and tool inputs and outputs unless switched off. In practice it suits request-scoped agents with a clear triage-and-specialist structure, and it is a weaker fit where runs last hours, must survive restarts, or need durable approval queues.

## In NVIDIA's stack

Nothing in the sources the wiki has read says that NVIDIA's blueprints are built on this SDK. Its relevance to the NVIDIA material is that it is a peer framework to the ones the [NeMo Agent Toolkit](nemo-agent-toolkit.md) wraps, and [NVIDIA NIM](nvidia-nim.md) exposes OpenAI-compatible APIs, which is the interface the SDK's chat-completions path targets **[web]**. Whether a given NIM supports the Responses API, or only chat completions, should be checked against the NIM release in use before pairing it with the SDK; the wiki has not verified it.

## Interview Q&A

**Handoff or agent-as-tool?** A handoff transfers the rest of the turn to a specialist, which then answers the user directly. An agent-as-tool leaves the manager in control and returns the specialist's output as a tool result. Use the second when one agent must own the final answer or apply shared guardrails **[web]**.

**What are the agent loop's termination conditions?** A final output of the expected type with no tool calls, or an error such as `MaxTurnsExceeded`. `tool_use_behavior` can also end the run at a named tool **[web]**.

**Where do input and output guardrails run, and what is the catch?** Input guardrails run on the first agent and output guardrails on the last. Neither runs on the intermediate agents of a handoff chain, so a checked entry point does not protect the specialists **[web]**.

**Why would you set `run_in_parallel=False` on an input guardrail?** The default runs the check alongside the agent for lower latency, but the agent may already have consumed tokens or run tools. Blocking mode stops anything from starting until the check passes **[web]**.

**How does the SDK give an agent memory, and what does it not give?** Sessions store and reload conversation items from a chosen backend. They do not authenticate users, do not checkpoint intermediate graph state, and cannot be combined with server-managed conversation IDs in one run **[web]**.

**What happens to traces for a ZDR organisation?** Tracing is unavailable, so a team in that position has to disable the default exporter and supply its own processor if it wants traces **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) |
| 2026-10-28 | [OpenAI Agents SDK: agents](https://openai.github.io/openai-agents-python/agents/) |
| 2026-10-28 | [OpenAI Agents SDK: running agents](https://openai.github.io/openai-agents-python/running_agents/) |
| 2026-10-28 | [OpenAI Agents SDK: handoffs](https://openai.github.io/openai-agents-python/handoffs/) |
| 2026-10-28 | [OpenAI Agents SDK: orchestrating multiple agents](https://openai.github.io/openai-agents-python/multi_agent/) |
| 2026-10-28 | [OpenAI Agents SDK: guardrails](https://openai.github.io/openai-agents-python/guardrails/) |
| 2026-10-28 | [OpenAI Agents SDK: sessions](https://openai.github.io/openai-agents-python/sessions/) |
| 2026-10-28 | [OpenAI Agents SDK: tracing](https://openai.github.io/openai-agents-python/tracing/) |
| 2026-10-28 | [OpenAI Agents SDK: MCP](https://openai.github.io/openai-agents-python/mcp/) |
| 2026-10-28 | [OpenAI Agents SDK: streaming](https://openai.github.io/openai-agents-python/streaming/) |
| 2026-10-28 | [OpenAI Agents SDK: models](https://openai.github.io/openai-agents-python/models/) |
| 2026-10-28 | [openai-agents on PyPI](https://pypi.org/project/openai-agents/) |
