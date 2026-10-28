# Stateful Orchestration

Stateful orchestration runs an agent as a graph whose shared state is checkpointed as it executes, which allows loops, branching, pausing for a human and resuming after a failure. LangGraph is the reference implementation here because NVIDIA's own blueprints build on it.

## Why a graph rather than a chain

A prompt chain is fixed control flow: a linear sequence with no loops. A stateful graph adds shared state updated by nodes, conditional edges, cycles and resumption, which is what a design needs once it moves from the workflow end of the spectrum in [Agent Architecture Patterns](agent-architecture-patterns.md) towards the agent end. NVIDIA's customer-service blueprint uses LangGraph for its agent graph **[sourced]**, and the AI Factory white paper describes a graph control plane, file-system workspaces and long-running stateful workflows whose traces can be replayed **[sourced]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) is framework-agnostic and wraps LangChain and LangGraph among others, with trajectory tracing through profiling and [NeMo Relay](../components/nemo-relay.md) **[sourced]**.

## Persistence in LangGraph

According to LangGraph's persistence documentation, persistence has two systems, matching the short- and long-term split in [Agent Memory](agent-memory.md). Checkpointers save a thread's graph state as checkpoints, attached at compile time and selected by a `thread_id` in the run configuration; they serve conversation continuity, time travel, fault tolerance and the pause-for-approval interrupts in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md). The Store is separate, holds application-defined cross-thread data such as preferences and facts, and is read and written by nodes directly. `InMemorySaver` loses everything on restart and is for experiments only, `SqliteSaver` suits local development, and `PostgresSaver` is the production option; thread identifiers stored in Postgres should stay under 255 characters, and checkpoints accumulate over long conversations so a pruning or retention policy is needed. Subgraphs keep their own checkpoint namespaces, so shared data may need to go through the Store **[web]**.

Durable execution means resuming from the last good step rather than rerunning, which matters for long tool-heavy runs. The durable-execution page confirms the checkpointer and `thread_id` requirement. Secondary descriptions also mention "exit", "async" and "sync" durability modes and wrapping side effects in tasks, which the durable-execution page itself does not describe, so treat them as unconfirmed **[web, snippet]**. Idempotent tools remain good practice regardless, for the retry reasons in [Agent Error Handling and Resilience](agent-error-handling-resilience.md).

The costs are state-schema design, checkpoint overhead, state growth and migration, serialisation of non-JSON objects, concurrency on shared threads, replay determinism and retention of personal data.

## Interview Q&A

**Why a stateful graph over a chain?** For branching, loops, resumability and pause-and-resume for human approval.

**Checkpointer versus store?** The checkpointer holds per-thread short-term execution state; the store holds cross-thread long-term memory **[web]**.

**How do you make an agent resumable in production?** A persistent checkpointer such as Postgres, a `thread_id` per conversation, idempotent tools and a checkpoint retention policy **[web]**.

**Why persist state at all?** To pause for approval, recover from a crash, replay for debugging and keep multi-turn continuity **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) |
| 2026-10-28 | [LangGraph durable execution](https://docs.langchain.com/oss/python/durable-execution) |
| 2026-10-27 | [Agentic AI in the Factory (NVIDIA AI Factory white paper)](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-27 | [NVIDIA NeMo (product page)](https://www.nvidia.com/en-us/ai-data-science/products/nemo/) |
