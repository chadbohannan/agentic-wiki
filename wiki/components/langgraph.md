# LangGraph

LangGraph is an open-source orchestration framework and runtime for stateful agents, built by LangChain Inc. Its documentation calls it "a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents" and says it mixes "deterministic, hand-coded steps with LLM-driven agentic steps in the same graph" **[web]**. As of 2026-10-28 the latest release is 1.2.18 under the MIT licence, and it needs Python 3.10 or later **[web]**. It is named in several of NVIDIA's own agent examples, so it is where the wiki's abstract ideas about [stateful orchestration](../concepts/stateful-orchestration.md) get concrete.

## Where it sits

LangGraph is the lowest of three layers from the same vendor. LangChain provides integrations and higher-level agent abstractions such as `create_agent`, which are built on LangGraph, and Deep Agents is a batteries-included harness built on LangChain agents, with context compression, a virtual file system and subagents **[web]**. LangGraph itself works without LangChain, and its documentation says it does not abstract prompts or architecture **[web]**. That puts it at the "write the loop yourself" end of the spectrum, the opposite pole from the role-playing abstractions in [CrewAI](crewai.md) and the lightweight primitives of the [OpenAI Agents SDK](openai-agents-sdk.md).

## The Graph API

A program is a `StateGraph` over a state schema, plus nodes and edges **[web]**.

- **State** is a `TypedDict` by default, a dataclass when defaults are needed, or a Pydantic model when validation is worth the cost. Each key can carry a reducer, written `Annotated[type, reducer]`, that says how an update merges with the current value; with no reducer an update replaces the value. `operator.add` appends to a list, and `add_messages` appends messages and updates existing ones by ID. `MessagesState` is a prebuilt state with one `messages` key using that reducer. Reducers matter because parallel nodes can write the same key in the same step.
- **Nodes** are ordinary functions that take the state and return a partial update. `START` and `END` are virtual nodes marking entry and exit.
- **Edges** are static (`add_edge`) or conditional (`add_conditional_edges`, where a router function returns the next node). If a node has several outgoing static edges, all destinations run in the next step.
- **`Send(node, arg)`** returned from a router fans out to a node with its own payload, which suits map-reduce when the number of branches is unknown in advance.
- **`Command(update=..., goto=...)`** returned from a node combines a state update with routing, and `Command(resume=...)` continues a paused run. The documentation warns against mixing static edges with `Command.goto` from the same node, since both paths then run.
- **`compile()`** validates the graph and attaches runtime options such as a checkpointer.

Execution proceeds in discrete super-steps: nodes that run in parallel share a step, and the run ends when no node is active and no messages are pending. A `recursion_limit` in the run configuration bounds the number of steps and raises `GraphRecursionError` when exceeded; the documentation gives 1000 as the default in recent versions, and `RemainingSteps` lets a node exit gracefully before the limit **[web]**. Because the limit counts steps rather than cost, it bounds a runaway loop but is not a token budget, so the budgets described in [ReAct and the Agent Loop](../concepts/react-reasoning-action.md) still have to be added in code.

The documentation shows each of Anthropic's workflow patterns in graph form: prompt chaining and evaluator-optimizer with conditional edges, parallelization with fan-out edges, routing with a structured-output classifier feeding a router, orchestrator-worker with `Send`, and the agent loop with `MessagesState`, bound tools and a prebuilt `ToolNode` **[web]**. That mapping is why the framework is a natural way to implement the patterns in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md): the choice between workflow and agent becomes a choice of how much routing is hard-coded edges and how much is a model call inside a router.

## The Functional API

The same runtime has a second front end for code that is already written as ordinary Python. `@entrypoint` turns a function into a workflow, and `@task` wraps a unit of work such as an API call and returns a future **[web]**. There is no declared state schema, because state is scoped to each function, and the graph cannot be visualised because it is built at run time. On resume the entrypoint replays from its start, loading finished task results from the checkpoint instead of recomputing them. Non-deterministic work and side effects therefore belong inside tasks, and a task that had started but not finished may run again, so its side effects need to be idempotent **[web]**. The documentation's advice is to choose the Functional API for minimal restructuring of existing code and the Graph API for an explicit, visualisable structure, and notes that the two can be combined **[web]**.

## Persistence, pausing and streaming

Compiling with a checkpointer saves a checkpoint after each super-step, keyed by a `thread_id`, which gives fault tolerance, time travel and resumable pauses; the Store holds data shared across threads. The savers, their production caveats and the split between short-term and long-term memory are in [Stateful Orchestration](../concepts/stateful-orchestration.md) and [Agent Memory](../concepts/agent-memory.md). The `interrupt()` and `Command(resume=...)` approval gate, with its ordering and idempotency rules, is described in [Human-in-the-Loop Patterns](../concepts/human-in-the-loop-patterns.md), and the stream modes are in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md).

Subgraphs, a compiled graph used as a node in another, are the framework's unit of composition for multi-agent designs **[web]**. A subgraph can be added directly as a node when it shares state keys with its parent, or called from inside a wrapper node when it needs private state and a different schema. The subgraph's `checkpointer` setting then chooses its memory: the default gives each call fresh state while inheriting the parent's checkpointer, so interrupts still work; `True` accumulates state across calls on one thread; `False` disables checkpointing, interrupts and durable execution. Per-thread subgraphs do not support parallel calls to the same subgraph, because the calls write to the same namespace **[web]**. The documentation recommends the per-invocation default for most subagents and reserves per-thread for those that need multi-turn memory, such as a research assistant. How subgraphs fit among the other topologies is in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md).

## Observability and deployment

LangGraph's observability page covers tracing to LangSmith, switched on with environment variables such as `LANGSMITH_TRACING` and `LANGSMITH_API_KEY`, with selective tracing through a `tracing_context`, tags and metadata on the run configuration, and an anonymiser for redacting patterns such as social security numbers **[web]**. That page does not discuss other backends or OpenTelemetry, so the vendor-neutral route is the one in [Agent Observability and Tracing](../concepts/agent-observability-tracing.md) rather than anything LangGraph-specific. For deployment, a `langgraph.json` file names the dependencies, the graphs (each mapped to a compiled graph or a function that builds one) and environment variables; the application-structure page names LangSmith Deployment, a managed hosting platform, as the target and does not describe self-hosting **[web]**.

## Trade-offs

Control is the benefit and the cost. Because the developer owns the state schema, the routing and the loop, every behaviour is visible and checkpointable, and a run can be inspected, replayed or paused at any step. The price is that nothing is free: a simple tool-calling agent needs a state type, a model node, a tool node and a router, where a higher-level framework needs a role and a goal. Two operational rules follow from how resumption works. A node may re-run from its start after an interrupt or retry, so side effects before a pause must be idempotent, and everything stored in a checkpoint must be serialisable. In practice a team that needs durable pauses, approval gates and replayable runs gets more from LangGraph than from the role-based frameworks, and a team that only needs a short tool loop may be better served by [the OpenAI Agents SDK](openai-agents-sdk.md).

## In NVIDIA's stack

LangGraph appears throughout NVIDIA's reference material, as a wrapped framework, as the implementation behind blueprints and as an integration target.

- The [NeMo Agent Toolkit](nemo-agent-toolkit.md) wraps LangGraph, and its profiler has callbacks for LangChain and LangGraph **[web]**. NVIDIA's customer-service blueprint builds its agent graph on LangGraph **[sourced]**.
- The RAG Blueprint's off-by-default agentic mode is a LangGraph plan-and-execute pipeline, described in [RAG Retrieval Pipelines](../concepts/rag-retrieval-pipelines.md), and the repository that matches [AI-Q](ai-q-blueprint.md) is described as a LangGraph state machine with concurrent researcher workers and a writer **[web]**.
- [NeMo Relay](nemo-relay.md) lists LangGraph among its integrations, and [NeMo Guardrails](nemo-guardrails.md) has an opt-in LangChain and LangGraph integration on its `LLMRails` engine only **[web]**.
- [NeMo Helix](nemo-helix.md) began by running toolkit LangGraph agents and by 0.8 kept them as a legacy path beside its own `agent.yaml` format **[web]**.

The toolkit's deprecation, recorded in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md), does not deprecate LangGraph, which is a third-party project. What changes is the wrapper around it, so a LangGraph agent is the most portable thing to carry across the NVIDIA transitions.

When the choice is between LangGraph and the other two frameworks, the deciding considerations are collected in [Choosing an Agent Framework](../syntheses/choosing-an-agent-framework.md).

## Interview Q&A

**What is LangGraph, and what is it not?** A low-level runtime that models an agent as a graph over shared, checkpointed state. It does not supply prompts, a ready-made agent architecture or a model abstraction; LangChain's `create_agent` and Deep Agents add those above it **[web]**.

**Why do some state keys need reducers?** When parallel nodes update the same key in one super-step, the framework needs a rule to merge the updates. A key without a reducer is overwritten, so `operator.add` or a custom function is needed to combine lists **[web]**.

**Compare `add_conditional_edges`, `Send` and `Command`.** A conditional edge routes to a node chosen by a router function. `Send` fans out to a node with its own payload, used when the branch count is dynamic. `Command` lets a node update state and choose the next node in one return value **[web]**.

**Graph API or Functional API?** The Graph API for an explicit structure that can be visualised and that checkpoints after every step; the Functional API to add persistence and approval pauses to existing Python control flow with little restructuring **[web]**.

**What is the difference between the three subgraph checkpointer settings?** `None` gives fresh state per call while still allowing interrupts, `True` keeps state across calls on a thread, and `False` turns off checkpointing and durable execution **[web]**.

**What goes wrong if a node does something irreversible before `interrupt()`?** The node restarts from its beginning on resume, so the action runs again. Make it idempotent or move it after the interrupt **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview) |
| 2026-10-28 | [LangGraph Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) |
| 2026-10-28 | [LangGraph workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents) |
| 2026-10-28 | [LangGraph functional API](https://docs.langchain.com/oss/python/langgraph/functional-api) |
| 2026-10-28 | [LangGraph subgraphs](https://docs.langchain.com/oss/python/langgraph/use-subgraphs) |
| 2026-10-28 | [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) |
| 2026-10-28 | [LangGraph observability](https://docs.langchain.com/oss/python/langgraph/observability) |
| 2026-10-28 | [LangGraph application structure](https://docs.langchain.com/oss/python/langgraph/application-structure) |
| 2026-10-28 | [LangGraph on PyPI](https://pypi.org/project/langgraph/) |
