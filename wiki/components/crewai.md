# CrewAI

CrewAI is an open-source Python framework for multi-agent systems built around two ideas: Crews, teams of role-playing agents that work on tasks, and Flows, event-driven workflows that hold the application's state and control flow. Its documentation calls Flows "the backbone of your AI application" and Crews "the units of work within your Flow", and recommends starting production applications with a Flow that calls a Crew where a step needs autonomous collaboration **[web]**. As of 2026-10-28 the latest release on PyPI is 1.15.33, for Python 3.10 to 3.13 **[web]**. It is the most abstract of the three frameworks in the wiki: where [LangGraph](langgraph.md) asks the developer to define state and edges and the [OpenAI Agents SDK](openai-agents-sdk.md) to define instructions and handoffs, CrewAI asks for a role, a goal and a backstory.

## Agents, tasks and crews

An agent has a required `role`, `goal` and `backstory`, which are rendered into its prompt, plus tools and limits. The documented defaults include `max_iter` of 20, `allow_delegation` off, `respect_context_window` on, `cache` on and `max_retry_limit` of 2, and optional `max_rpm`, `max_execution_time` and `reasoning` settings **[web]**. The default model is taken from the `OPENAI_MODEL_NAME` environment variable and falls back to a GPT-4 model, so an unconfigured agent calls an OpenAI endpoint. An agent can also run alone with `agent.kickoff(...)`, which takes messages and an optional Pydantic `response_format` and returns the raw text, parsed object and usage metrics **[web]**.

A task has a `description`, an `expected_output`, and optionally an `agent`, `tools`, a `context` list of earlier tasks, `async_execution`, structured output through `output_pydantic` or `output_json`, an `output_file`, a `callback`, a `human_input` flag and guardrails **[web]**. `context` is the dependency mechanism: a task waits for the tasks it lists and receives their outputs, and without it, output is passed on to the next task. A task guardrail is either a function that takes the task output and returns a pass flag and a result, or a plain string that creates an LLM-judged guardrail using the agent's model. On failure the error returns to the agent, which retries until the guardrail passes or `guardrail_max_retries` (default 3) is reached **[web]**. That makes task guardrails an in-loop output check in the sense of [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md), applied per task rather than at the edge of the run.

A crew combines agents and tasks with a `process` **[web]**.

- **Sequential**, the default, runs tasks one after another.
- **Hierarchical** adds a manager that delegates tasks and validates results, and needs a `manager_llm` or a custom `manager_agent`. The documentation recommends strong models for the manager, and the page does not describe the manager's delegation tools or how validation works internally, so how reliably the manager reviews work is not documented.

Other crew settings include `planning`, in which a planner adds a plan to each task description before the run, a tool-result `cache` that is on by default, a crew-wide `max_rpm`, and step and task callbacks. Kickoff comes in synchronous, thread-wrapped async and native async (`akickoff`) forms, with `_for_each` variants over a list of inputs, and the result carries the raw, parsed and per-task outputs and token usage **[web]**. A replay command re-runs from a chosen task while keeping earlier context **[web]**. Compared with the five patterns in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md), a sequential crew is a prompt chain whose steps are agents, and a hierarchical crew is an orchestrator-worker design with an LLM-run orchestrator, the supervisor topology in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md).

## Flows

A Flow is a class whose methods are wired by decorators **[web]**. `@start()` marks entry points, `@listen(...)` runs a method when another completes and passes it that method's output, and `@router(...)` returns a label that downstream `@listen("label")` methods react to. `or_` and `and_` combine triggers, so a method can wait for any or all of several upstream steps. State is either an untyped dictionary or a Pydantic model given as the class parameter, with an `id` added automatically. `@persist` saves state, by default to SQLite, and a Flow resumes under the same ID or forks from a saved snapshot under a new one. `@human_feedback` pauses for a person, optionally using an LLM to map the response to one of several outcomes that trigger different listeners, and needs CrewAI 1.8.0 or later. A Flow's `kickoff` returns the output of the last method to finish, can stream, and aggregates token usage across the run.

Inside a listener the usual pattern is to build a Crew, kick it off with inputs from state, and write the result back to state. Hard-coded control stays in the Flow, and model-driven collaboration stays inside the Crew. That is the same separation [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md) recommends between a deterministic workflow and the agentic steps inside it, and it explains why the documentation steers production users to Flows first. A Flow's state and `@persist` play the role LangGraph's checkpointed state plays in [Stateful Orchestration](../concepts/stateful-orchestration.md), but at method granularity rather than per super-step, and the Flow pages read do not mention time travel.

## Tools and failures

Tools are `BaseTool` subclasses with an `args_schema`, or functions decorated with `@tool` whose docstring is what the agent reads to decide when to call them **[web]**. A tool can be sync or async, and a `cache_function` controls which results are cached. The built-in `crewai_tools` package groups tools for web search and scraping, retrieval over documents and databases, file access and code execution. MCP servers can supply tools, and a server error is recorded as a tool failure. A `tool_failure_policy` of `ignore`, `warn` (default) or `raise` can be set on the tool, task, agent or crew, with the most specific setting winning, and failures are visible on the result and as events **[web]**. A default of `warn` means a failing tool does not stop a run by itself, which is worth knowing before trusting a crew's output; the failure classes it must be combined with are in [Agent Error Handling and Resilience](../concepts/agent-error-handling-resilience.md).

## Models and observability

CrewAI has native SDK integrations for OpenAI, Anthropic, Google Gemini, Azure and AWS Bedrock, and reaches every other provider, including Ollama and NVIDIA NIM, through LiteLLM; model strings need a provider prefix such as `openai/gpt-4o`, and the documentation notes that structured-output support varies by provider and model **[web]**. A local NIM is addressed with the `openai/` prefix and a `base_url`. Streaming emits chunk events that carry the agent and task, which a custom listener can filter **[web]**. The observability page lists third-party integrations such as Langfuse, Arize Phoenix, MLflow, Opik and OpenLIT and does not describe a built-in tracing feature, so the trace backend is the developer's choice **[web]**.

## Memory

CrewAI's memory documentation describes a single `Memory` class that replaced separate short-term, long-term, entity and external types **[web]**. Records live in a hierarchical scope tree, `remember` and `recall` are the core calls, and an LLM infers scope, categories and importance when none are given. Recall ranks by a weighted sum of semantic similarity, recency decay and importance, with documented defaults of 0.5, 0.3 and 0.2 and a 30-day half-life. When saving, similar existing records are compared and an LLM chooses whether to keep, update, delete or insert, which is a form of the consolidation discussed in [Agent Memory](../concepts/agent-memory.md). Setting `memory=True` on a crew extracts facts after each task and recalls context before each one, and the default store is LanceDB on local disk with OpenAI embeddings unless configured otherwise. Because extraction and consolidation each cost model calls, and the model that decides what to keep is itself fallible, memory behaviour is worth evaluating rather than assuming. The crew-level `memory` setting is documented as storing short-term, long-term and entity memories, a wording that predates the unified class, so older descriptions of CrewAI memory may not match the current API **[web]**.

## Trade-offs

The role-based vocabulary makes a prototype quick to write and easy to explain to non-engineers, and Flows give production users explicit control. The costs are indirection and cost control. The role, goal and backstory strings are prompt text with an effect on behaviour that has to be tested, a hierarchical crew adds a manager's model calls to every delegation, and Anthropic's general warning in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md) that framework layers can obscure the underlying prompts and responses applies. In practice CrewAI suits well-understood pipelines of specialist steps, such as research, draft and review, and is less natural for open-ended single-agent tool loops or for runs that need replayable fine-grained checkpoints.

## In NVIDIA's stack

CrewAI reaches NVIDIA NIM through LiteLLM **[web]**, and the [NeMo Agent Toolkit](nemo-agent-toolkit.md) is framework-agnostic with a plugin for CrewAI, and its profiler has a CrewAI callback, so a CrewAI crew can be profiled and evaluated through the toolkit **[web]**. Both are recorded in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md) and [Agent Profiling and Result Analysis](../concepts/agent-profiling-and-result-analysis.md). Given the toolkit's deprecation in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md), the wiki has not verified whether [NeMo Relay](nemo-relay.md) or [NeMo Helix](nemo-helix.md) support CrewAI directly; the Relay documentation read lists LangChain, LangGraph and Deep Agents. No NVIDIA blueprint the wiki has read is built on CrewAI.

## Interview Q&A

**Crew or Flow?** A Crew is a team of agents that decides how to collaborate on a goal; a Flow is event-driven code that owns state and control flow and calls Crews for the steps that need agents. The documentation advises starting production applications with a Flow **[web]**.

**Sequential or hierarchical process?** Sequential runs the listed tasks in order. Hierarchical adds a manager agent that assigns and reviews work and needs a `manager_llm` or `manager_agent`; it costs extra model calls and its review logic is not documented in detail **[web]**.

**How do tasks pass information to each other?** Through the `context` list, which makes a task wait for and receive earlier tasks' outputs, or implicitly by relaying output to the next task in a sequential crew **[web]**.

**How do you enforce an output contract on a task?** `output_pydantic` or `output_json` for the shape, and a function or string guardrail for the content. A failed guardrail sends the error back to the agent, which retries up to `guardrail_max_retries` **[web]**.

**What does `@router` do in a Flow?** It returns a label, and listeners subscribed to that label run, which gives branching without hard-coding the next method **[web]**.

**What are the risks of the defaults?** Agents fall back to an OpenAI model if none is configured, delegation is off unless enabled, and `tool_failure_policy` defaults to `warn`, so a failing tool logs rather than halts **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [CrewAI introduction](https://docs.crewai.com/en/introduction) |
| 2026-10-28 | [CrewAI agents](https://docs.crewai.com/en/concepts/agents) |
| 2026-10-28 | [CrewAI tasks](https://docs.crewai.com/en/concepts/tasks) |
| 2026-10-28 | [CrewAI crews](https://docs.crewai.com/en/concepts/crews) |
| 2026-10-28 | [CrewAI flows](https://docs.crewai.com/en/concepts/flows) |
| 2026-10-28 | [CrewAI memory](https://docs.crewai.com/en/concepts/memory) |
| 2026-10-28 | [CrewAI tools](https://docs.crewai.com/en/concepts/tools) |
| 2026-10-28 | [CrewAI hierarchical process](https://docs.crewai.com/en/learn/hierarchical-process) |
| 2026-10-28 | [CrewAI LLMs](https://docs.crewai.com/en/concepts/llms) |
| 2026-10-28 | [CrewAI observability overview](https://docs.crewai.com/en/observability/overview) |
| 2026-10-28 | [crewai on PyPI](https://pypi.org/project/crewai/) |
