# Agent Memory (Short- and Long-Term)

LLMs are stateless, so an agent's memory is whatever the system chooses to put back into the context window: a bounded, session-scoped buffer for short-term memory and external stores for long-term memory. This page covers exam objectives 1.4 and 5.1: the taxonomy, representative systems, the vector versus graph trade-off, and NVIDIA's implementations.

Graph-structured long-term memory builds on [Knowledge Graphs and GraphRAG](knowledge-graphs-graphrag.md). The loop that turns experience into stored lessons is covered in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md), and thread-scoped short-term state is persisted by the mechanisms in [Stateful Orchestration](stateful-orchestration.md). The stores behind long-term memory are detailed in [Vector Databases and ANN Tuning](vector-databases-and-ann-tuning.md), and privacy handling for stored memories is in [Privacy and PII Protection](privacy-and-pii-protection.md).

## Taxonomy

IBM's explainer states that LLMs cannot remember on their own and that short-term memory is overwritten and does not persist **[sourced]**. Short-term or working memory is a rolling buffer or the context window; in LangGraph it is thread-scoped state saved by a checkpointer keyed on `thread_id` **[sourced; web]**. Long-term memory is split, borrowing from cognitive science through the CoALA framework (Sumers, Yao, Narasimhan and Griffiths, arXiv 2309.02427, TMLR 2024), into episodic memory (specific past events and trajectories), semantic memory (facts and preferences) and procedural memory (skills, action sequences and instructions) **[sourced via IBM; web, CoALA from a search snippet]**.

A survey the study guide cites offers an alternative taxonomy: object (personal or system) by form (parametric or non-parametric) by time (short or long), with construction, management, retrieval and usage as the operations, and parametric memory meaning weights or KV cache **[sourced]**. Note that the guide's arXiv ID for that survey (2504.15965v1) is a memory survey even though the guide labels it a planning survey. A 2026 survey (arXiv 2603.07670) formalises memory as a write-manage-read loop with five mechanism families: context-resident compression, retrieval-augmented stores, reflective self-improvement, hierarchical virtual context and policy-learned management, and highlights write-path filtering, contradiction handling, latency budgets and governance as practical issues **[web, search snippet]**.

## Representative systems

MemGPT, now Letta, pages information between the context window and external storage in the manner of an operating system (Packer et al., arXiv 2310.08560, October 2023; the "OS-style paging" wording is a paraphrase) **[web, snippet]**. Mem0 extracts, consolidates and retrieves facts, with a graph variant; its paper reports on LOCOMO a 26% relative LLM-judge gain over OpenAI's memory, about 91% lower p95 latency and over 90% token savings against full context, and the graph variant gained only about 2% over the base, all the authors' own numbers (arXiv 2504.19413) **[web, vendor self-reported; not re-verified in this pass]**. Reflexion stores verbal reflections in an episodic buffer (arXiv 2303.11366), and Voyager keeps verified code as a procedural skill library (arXiv 2305.16291), both detailed in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md) **[web, abstracts read]**. Zep's temporal graph is covered in [Knowledge Graphs and GraphRAG](knowledge-graphs-graphrag.md).

Vector memory gives simple semantic recall but is weak on multi-hop and temporal relations; graph memory handles those better at a higher build cost **[inferred]**.

## Long context versus retrieval

Bigger windows give working memory but not persistence across sessions, selective retrieval, deletion or access control, and their cost grows with length **[web, search snippet of 2603.07670]**. A Chroma study reports "context rot", where accuracy degrades with input length well before the limit **[web, search snippet only; unverified]**. Practitioners converge on a hybrid of retrieving candidates and reasoning in a long window, with just-in-time retrieval and compaction **[web, practitioner blogs; weak]**. NVIDIA's Jamba 1.5 post shows the architecture side of cheap long context with a 256K Mamba-Transformer hybrid **[sourced]**.

## NVIDIA's implementations

NVIDIA's customer-service blueprint embeds queries and responses for within-conversation retrieval as short-term memory, and stores an end-of-conversation summary plus sentiment in a structured database as long-term memory **[sourced]**. The AI Factory white paper adds persistent virtual workspaces that hold datasets, code and intermediate artefacts across planning cycles **[sourced]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) has a memory module with provider plugins for Mem0, Redis and Zep (MemMachine experimental in 1.6): developers implement a `MemoryEditor` with search, add and remove, select the backend in configuration, and from 1.4 can use an automatic memory wrapper that stores messages and injects retrieved context before each agent call **[web, toolkit docs]**. Relay captures trajectories across tools, models and memory **[sourced]**. The main trade-offs are write quality, staleness and contradiction, retrieval latency, privacy of personal data in memory, and the cost of LLM-based extraction **[inferred]**.

## Interview Q&A

**Short-term versus long-term?** Short-term is context or thread state; long-term persists across sessions **[web]**.

**How do you decide what to store?** Summarise at session end, extract facts, and score importance and recency **[inferred, partly from the blueprint]**.

**Episodic, semantic, procedural?** Logged past events, distilled facts and preferences, and learned skills or instructions **[sourced]**.

**How do you implement both in LangGraph?** A checkpointer per `thread_id` for short-term, a Store for cross-thread long-term, with Postgres rather than `InMemorySaver` in production **[web]**.

**Why not just use a 1M-token context?** Cost and latency, degraded recall on long inputs, no persistence across sessions and no selective forgetting or access control, so use retrieval alongside long context **[web, inferred]**.

**Vector versus graph memory?** Vector for fuzzy semantic recall, graph for relational, multi-hop or temporal queries, with hybrids common **[inferred]**.

**How does the NeMo Agent Toolkit handle memory?** Pluggable `MemoryEditor` providers selected in YAML, with an optional automatic memory wrapper **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [AI Agent Memory (IBM)](https://www.ibm.com/think/topics/ai-agent-memory) |
| 2026-10-27 | [Memory survey (arXiv 2504.15965v1)](https://arxiv.org/html/2504.15965v1) |
| 2026-10-27 | [Jamba 1.5 hybrid architecture (NVIDIA blog)](https://developer.nvidia.com/blog/jamba-1-5-llms-leverage-hybrid-architecture-to-deliver-superior-reasoning-and-long-context-handling) |
| 2026-10-27 | [NVIDIA NeMo (product page)](https://www.nvidia.com/en-us/ai-data-science/products/nemo/) |
| 2026-10-27 | [Agentic AI in the Factory (NVIDIA AI Factory white paper)](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html) |
| 2026-10-27 | [Three Building Blocks for AI Virtual Assistants for Customer Service (NVIDIA)](https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/) |
| 2026-10-28 | [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) |
| 2026-10-27 | [CoALA (arXiv 2309.02427)](https://arxiv.org/pdf/2309.02427) |
| 2026-10-27 | [A survey of agent memory (arXiv 2603.07670)](https://arxiv.org/abs/2603.07670) |
| 2026-10-27 | [MemGPT (arXiv 2310.08560)](https://arxiv.org/abs/2310.08560) |
| 2026-10-27 | [Mem0 (arXiv 2504.19413)](https://arxiv.org/abs/2504.19413) |
| 2026-10-27 | [Reflexion (arXiv 2303.11366)](https://arxiv.org/abs/2303.11366) |
| 2026-10-27 | [Voyager (arXiv 2305.16291)](https://arxiv.org/pdf/2305.16291) |
| 2026-10-27 | [NeMo Agent Toolkit memory docs (1.6)](https://docs.nvidia.com/nemo/agent-toolkit/1.6/build-workflows/memory.html) |
