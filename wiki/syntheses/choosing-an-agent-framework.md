# Choosing an Agent Framework

An interviewer who asks about [LangGraph](../components/langgraph.md), the [OpenAI Agents SDK](../components/openai-agents-sdk.md) and [CrewAI](../components/crewai.md) is rarely testing recall of their APIs. The question is usually a scenario, and the answer is a choice with a reason. This page organises the choice around the considerations that decide it, in the order a good answer raises them. The feature facts come from each framework's documentation as of 2026-10-28 and are tagged; the weighing of them is the wiki's own judgment and is untagged.

## First: is a framework needed at all?

Anthropic's guidance in "Building Effective Agents" is to find the simplest solution possible, because framework abstractions can obscure prompts and responses and invite unnecessary complexity **[web]**, a point made in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md). A fixed pipeline of a few model calls may need nothing more than the provider API and ordinary code. A framework earns its place when the run needs something the developer would otherwise build by hand: a loop with limits, pausing and resuming, delegation among agents, or tracing. The rest of this page assumes that threshold is crossed.

## The considerations

**Does a run have to survive a restart or wait for a person?** This is the strongest discriminator. LangGraph checkpoints state after every step under a thread ID, so a run can pause at `interrupt()` for as long as needed and resume on another process **[web]**. CrewAI Flows can persist state with `@persist` and resume under the same ID, at method rather than step granularity **[web]**. The Agents SDK stores conversation items in sessions and pauses for approval through a serialisable run state, but the pages read describe no per-step checkpoint of an in-flight run **[web]**. A workload with long-running tasks, approval queues or recovery requirements points to LangGraph; a request-scoped assistant that answers within one HTTP call does not need it. The mechanisms are in [Stateful Orchestration](../concepts/stateful-orchestration.md) and [Human-in-the-Loop Patterns](../concepts/human-in-the-loop-patterns.md).

**Who decides the control flow, the developer or the model?** LangGraph makes the developer draw nodes and edges, and the model's autonomy lives inside nodes and routers. The Agents SDK gives the model the loop and lets it pick tools and handoffs, with code-driven chaining available when predictability matters **[web]**. CrewAI splits the two: a Flow is deterministic code and a Crew is model-driven collaboration **[web]**. If auditors or on-call engineers must be able to say exactly which path a run took, explicit graphs and Flows are easier to defend than a free-running loop. If the task space is open-ended and the cost of a wrong step is low, the SDK's thin loop is faster to build and to change.

**What shape is the multi-agent problem?** The three map onto different topologies in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md). A triage agent that passes the user to a specialist is the SDK's handoff, and a manager that consults specialists and keeps the answer is its agent-as-tool **[web]**. Parallel workers with a combining step are a LangGraph `Send` fan-out with a reducer. A fixed team doing research, drafting and review is a sequential crew, and a manager-led team is a hierarchical crew, whose manager's review logic the documentation does not describe **[web]**. The mistake to avoid is picking a multi-agent framework for a problem a single agent with good tools would solve, since each extra agent adds model calls, cost and failure modes.

**Where must safety checks sit, and what do they cover?** The SDK's input guardrails run only on the first agent and its output guardrails only on the last, and tool guardrails do not cover handoffs or hosted tools **[web]**, so a design with handoffs needs checks in the specialists as well. CrewAI guardrails attach per task and retry the agent on failure **[web]**. In LangGraph a check is a node or a router, which makes placement fully explicit and also entirely the developer's responsibility. The layering logic is in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md); the practical point is that each framework puts the default check in a different place, and the answer to "where is the unguarded gap" differs accordingly.

**How tied is the choice to a model provider?** The SDK is built around OpenAI's Responses API, and other providers go through a chat-completions model class or best-effort LiteLLM and any-llm adapters, with documented gaps in structured output and Responses-only fields **[web]**. CrewAI has native SDKs for OpenAI, Anthropic, Google Gemini, Azure and AWS Bedrock, and reaches other providers, including NVIDIA NIM, through LiteLLM **[web]**. LangGraph does not abstract models itself; model integrations come from LangChain or direct client code **[web]**. A team that must run on self-hosted open models, or on [NIM](../components/nvidia-nim.md), should check how well that path is supported before committing; the SDK's non-OpenAI path is the least first-class of the three.

**Where do traces go, and who can see them?** The SDK traces by default to OpenAI's backend, includes model and tool inputs and outputs unless told not to, and is unavailable to organisations on Zero Data Retention; replacing the processors redirects it **[web]**. LangGraph's documented tracing is LangSmith, with a self-hosted LangSmith mentioned and an anonymiser for redaction **[web]**. The CrewAI observability page lists third-party integrations such as Langfuse, Arize Phoenix, MLflow and OpenLIT and describes no built-in tracing **[web]**. For regulated or air-gapped work, the answer should be about controlling the export path, using the OpenTelemetry-based approach in [Agent Observability and Tracing](../concepts/agent-observability-tracing.md) and the audit needs in [Audit Trails and Traceability](../concepts/audit-trails-and-traceability.md).

**How mature is the interface?** LangGraph is at 1.2.x and CrewAI at 1.15.x, while the Agents SDK is at 0.24, so the SDK's interfaces are more likely to move **[web]**. Version numbers are weak evidence of stability on their own, but they are a fair prompt to pin versions and budget for upgrades.

**What does the team know, and how fast must it ship?** CrewAI's role, goal and backstory vocabulary lets a team stand up a working multi-step pipeline quickly and explain it to non-engineers. The SDK is the shortest path for a team that already uses OpenAI. LangGraph asks for more design up front, which pays back when the run must be debugged, replayed or audited. Prototype speed and production control pull in opposite directions, and the answer depends on which phase the project is in.

## Scenarios

| Scenario | Likely choice | Reason |
| -------- | ------------- | ------ |
| Back-office agent that drafts a payment and waits days for approval | LangGraph | Checkpointed pause and resume with a durable store |
| Customer-support bot with a triage agent and billing and technical specialists | Agents SDK | Handoffs fit the shape; guardrails must be repeated on specialists |
| Weekly research, draft and review pipeline for a content team | CrewAI | A sequential crew inside a Flow matches fixed specialist steps |
| Regulated workflow that must replay any run step by step | LangGraph | Explicit graph and per-step checkpoints |
| Prototype to validate an idea this week, OpenAI models only | Agents SDK | Fewest concepts and built-in tracing |
| Self-hosted open models behind NIM | LangGraph or CrewAI | Both reach NIM; the SDK's non-OpenAI path is best-effort |

These are defaults to argue from, not rules. Mixing is legitimate: the documentation shows a LangGraph subgraph used as a subagent and a CrewAI Crew called from inside a Flow, and the SDK's agents can be wrapped as tools by a larger system **[web]**.

## In NVIDIA's stack

For the exam the relevant fact is that the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) is framework-agnostic and wraps all three, while NVIDIA's own blueprints lean on LangGraph, as described in [NVIDIA Agentic Stack](nvidia-agentic-stack.md). With the toolkit deprecated, the framework choice is the part of an NVIDIA agent that stays with the developer, and LangGraph is the one with the most documented NVIDIA integrations. A good interview answer says that the framework is a choice among peers and that NVIDIA's contribution sits below it, in serving, guardrails and observability.

## Interview Q&A

**Which framework would you pick for a long-running workflow with human approval, and why?** LangGraph, because it checkpoints state at every step under a thread ID and `interrupt()` waits indefinitely. The caveat is that the node restarts on resume, so side effects before the interrupt must be idempotent **[web]**.

**When is the OpenAI Agents SDK the wrong choice?** When runs must survive restarts with mid-run state, when the team needs first-class support for non-OpenAI or self-hosted models, or when traces must stay out of OpenAI's backend and the team does not want to replace the default exporter. It is also pre-1.0 **[web]**.

**What is the risk of putting guardrails only on the first agent in a handoff design?** In the SDK, input guardrails do not run on later agents, so a specialist reached by handoff is unchecked unless it carries its own **[web]**.

**Why might a team prefer CrewAI's Flows over a bare Crew?** A Flow gives deterministic, stateful control with persistence and human feedback, and calls a Crew only for steps that need collaboration; the documentation recommends starting production applications that way **[web]**.

**A client says they need the same agent to run on OpenAI today and on an open model next quarter. What do you check?** How each framework reaches the second provider: the SDK needs a chat-completions class or beta adapter, CrewAI uses LiteLLM for providers without a native SDK, and structured-output and tool-calling support varies by model. Test the exact model before committing **[web]**.

**How would you justify not using a framework?** A fixed chain of calls does not need one, and Anthropic's advice is to start with direct API use because abstractions can hide prompts and responses **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [CrewAI LLMs](https://docs.crewai.com/en/concepts/llms) |
| 2026-10-28 | [CrewAI observability overview](https://docs.crewai.com/en/observability/overview) |
| 2026-10-28 | [OpenAI Agents SDK: MCP](https://openai.github.io/openai-agents-python/mcp/) |
| 2026-10-28 | [OpenAI Agents SDK: streaming](https://openai.github.io/openai-agents-python/streaming/) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
