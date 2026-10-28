# Knowledge Graphs and GraphRAG

Knowledge graphs give agents structured entities and relations, and GraphRAG uses them to answer global, multi-hop questions that vector retrieval handles poorly. This page covers exam objective 1.7.

The GraphRAG paper (Edge et al., "From Local to Global: A Graph RAG Approach to Query-Focused Summarization", arXiv 2404.16130, April 2024) treats corpus-wide questions as query-focused summarisation rather than retrieval. An LLM first extracts an entity knowledge graph from the documents and pregenerates summaries for communities of closely related entities. At query time each community summary yields a partial answer, and the partial answers are summarised into the final response. On global sensemaking questions over datasets of about one million tokens, the authors report substantially better comprehensiveness and diversity than a conventional RAG baseline **[web, abstract read in full]**. The baseline it beats is the vector retrieval covered in [RAG Retrieval Pipelines](rag-retrieval-pipelines.md).

For agents, the relevant variant is the temporal graph used as long-term memory, discussed in [Agent Memory (Short- and Long-Term)](agent-memory.md). The Zep paper (arXiv 2501.13956, January 2025) describes a temporal knowledge graph with episodic, semantic and community layers in which changed facts receive validity timestamps instead of being deleted. Its abstract reports 94.8% against MemGPT's 93.4% on the DMR benchmark and up to 18.5% higher accuracy with 90% lower latency on LongMemEval **[web, abstract read in full; vendor self-reported]**. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) integrates Zep Cloud as a memory backend **[web]**.

The trade-offs are our synthesis **[inferred]**. Indexing is expensive because it needs many LLM calls, quality depends on schema and entity resolution, and updates cost more than appending a vector. Use a graph for relational, multi-hop or aggregate questions and plain vector retrieval for lookups. The study guide's section 1 sources contained no NVIDIA-specific knowledge-graph material, which is an open gap.

## Interview Q&A

**When GraphRAG over vector RAG?** For global or thematic questions and multi-hop relational ones **[web]**.

**What does it cost?** Heavy index-time LLM extraction **[inferred]**.

**Why temporal graphs for memory?** Facts change over time, so the graph tracks when each was valid **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [From Local to Global: A Graph RAG Approach (arXiv 2404.16130)](https://arxiv.org/abs/2404.16130) |
| 2026-10-28 | [Zep: A Temporal Knowledge Graph Architecture for Agent Memory (arXiv 2501.13956)](https://arxiv.org/abs/2501.13956) |
