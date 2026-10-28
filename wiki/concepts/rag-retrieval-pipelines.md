# RAG Retrieval Pipelines

Retrieval-augmented generation (RAG) retrieves external passages at query time and conditions the model's answer on them. In production, quality depends less on the generator than on the retrieval pipeline: chunking, hybrid dense and sparse search, fusion, reranking and query rewriting. Agentic variants add planning and self-checks on top. The study guide's own sources for this topic were unavailable, so everything here comes from wider research. Whether to retrieve at all, rather than fine-tune, is the question in [RAG vs Fine-Tuning vs Continued Pretraining](rag-vs-fine-tuning-vs-pretraining.md).

## Origin and stages

Lewis et al. (arXiv 2005.11401) introduced RAG. They paired a pretrained seq2seq generator ("parametric memory") with a dense vector index of Wikipedia reached through a neural retriever ("non-parametric memory") **[web]**. Today the term covers any pipeline that retrieves text, images or tables and injects them into an LLM's prompt. The stages are:
1. Ingest and parse the documents, the ETL layer in [Document Ingestion and ETL](document-ingestion-etl.md).
2. Chunk them.
3. Embed and index the chunks.
4. At query time, optionally rewrite or decompose the query.
5. Retrieve, using dense search over the indexes in [Vector Databases and ANN Tuning](vector-databases-and-ann-tuning.md), sparse search, or both.
6. Fuse and rerank the results.
7. Assemble the context and generate the answer.
8. Optionally check relevance and groundedness, and loop.

## Chunking

Chroma's chunking study used 472 synthetic queries over five corpora, scored on token-level recall, precision and intersection-over-union (IoU) **[web]**. It is a small synthetic set from a single vendor. Its findings:
- The choice of chunker shifts recall by up to about 9%.
- A recursive character splitter at about 200 tokens with no overlap was a consistently strong baseline.
- The OpenAI Assistants default of 800 tokens with 400 overlap scored low.
- Semantic and LLM-based chunkers gave the highest recall (91.9 and 91.3), at higher compute cost.
- Overlap helped recall with a small embedding model but hurt IoU, so "no overlap is best" is not universal.

Anthropic's Contextual Retrieval prepends a 50–100-token, chunk-specific context written by an LLM to each chunk before embedding and BM25 indexing. On Anthropic's datasets, top-20 retrieval failures fell by 35% with contextual embeddings alone, 49% with contextual BM25 added, and 67% with reranking added, figures Anthropic reports on its own data **[web]**. The same post suggests skipping RAG entirely for knowledge bases under about 200k tokens and putting them in the prompt instead. Long context is not free, though. "Lost in the Middle" (arXiv 2307.03172) found that models use information in the middle of long inputs worse than information at either end **[web]**.

## Dense, sparse and hybrid retrieval

Dense embeddings capture meaning but can miss exact tokens such as error codes. BM25 is lexical and catches them, so fusing the two ranked lists beats either alone **[web]**. Reciprocal Rank Fusion (Cormack et al., SIGIR 2009) scores each document as the sum of 1/(k + rank) across the lists, which avoids calibrating incomparable scores. Elasticsearch's documentation gives k = 60 as the default **[web]**.

NVIDIA's RAG Blueprint enables hybrid search with `APP_VECTORSTORE_SEARCHTYPE=hybrid` **[web]**:
- It uses RRF by default, with a weighted ranker as an alternative (dense and sparse weights default to 0.5).
- RRF in Elasticsearch requires a commercial licence.
- Milvus collections created for dense-only search must be recreated and re-ingested to use hybrid search.

Milvus supports learned sparse vectors (SPLADE, BGE-M3) natively, with sparse and dense vectors in one collection **[web]**.

## Reranking

A reranker rescores the top-N candidates jointly with the query. NVIDIA's NeMo Retriever reranking NIM documentation describes it as most useful in hybrid retrieval, where first-stage scores are on different scales. Its benchmark gives average recall@5 across five datasets of 0.5699 for dense retrieval alone, 0.7070 with reranking, and 0.7137 with dense, sparse and reranking. Reranking 500 passages on an H100 takes about 1,750 ms. These are NVIDIA's own benchmark figures. A vision-language reranker (`nvidia/llama-nemotron-rerank-vl-1b-v2`) scores text queries against text and image passages **[web]**. The trade-off is to retrieve wide and pay latency for precision. Anthropic retrieved about 150 candidates and kept 20 **[web]**.

## Query rewriting and decomposition

HyDE (arXiv 2212.10496) has an LLM write a hypothetical answer, embeds it, and retrieves real documents near it. Its authors report it competitive with fine-tuned retrievers in zero-shot settings **[web]**. The Blueprint's query decomposition is off by default **[web]**:
- An LLM splits a multi-hop question into sub-queries, rewriting them with the context gathered so far. Where the hops follow entity relations, [Knowledge Graphs and GraphRAG](knowledge-graphs-graphrag.md) is often the better tool, and where the answer sits in tables, [Structured Data Access and Text-to-SQL](structured-data-access-text-to-sql.md).
- It retrieves and ranks for each sub-query and generates follow-up questions.
- It stops at `MAX_RECURSION_DEPTH` (default 3) and synthesises an answer.
- It works on a single collection only, and it adds latency.

## Self-reflection and agentic RAG

Self-RAG (arXiv 2310.11511) trains a single model to decide when to retrieve and to critique both the passages and its own output, using special reflection tokens **[web]**. The Blueprint's self-reflection feature is an untrained version of the same idea **[web]**:
- A context-relevance check rewrites the query and retrieves again when its score is low, an online use of the metric that [RAG Evaluation and Ragas](rag-evaluation-ragas.md) applies offline.
- A groundedness check regenerates the answer when its score is low.
- Both are scored 0–2, with a default threshold of 1 and at most three loops.
- Responses do not stream during the groundedness check.

The Blueprint's Agentic RAG mode, also off by default, is a LangGraph plan-and-execute pipeline **[web]**:
1. An initial retrieval.
2. A planner that emits a scope-discovery plan, an answer plan or an empty plan.
3. Parallel mini-agent tasks that retrieve, answer or reformulate.
4. Synthesis, with optional verification.

It does not yet support guardrails, self-reflection, query decomposition or VLM inference, and its tasks have no dependency graph. The general agentic-RAG idea is that an agent decides whether and what to retrieve, treating retrieval as a tool. The Blueprint's agent actions are limited to retrieval and LLM calls **[web]**.

## In NVIDIA's stack

The RAG Blueprint documentation covers these capabilities **[web]**:
- **Deployment:** Docker, Helm and OpenShift, plus a library mode and a "Lite" mode (Milvus Lite with cloud APIs).
- **Retrieval:** hybrid search, multi-collection retrieval, query decomposition, multi-turn conversations, agentic RAG and self-reflection.
- **Multimodal and ingestion:** a multimodal retriever with a VLM embedder and reranker, audio ingestion, continuous ingestion from object storage, and custom metadata.
- **Integration:** NeMo Guardrails, an MCP server, and Milvus and Elasticsearch backends.

Release 2.6.0 (2026-05-30) changed several defaults **[web]**:
- Elasticsearch became the default vector database. Its GPU-accelerated support requires enterprise access and is off by default.
- SeaweedFS replaced MinIO for object storage.
- The default LLM became `nvidia/nemotron-3-super-120b-a12b` and the default embedder `nvidia/llama-nemotron-embed-vl-1b-v2`.
- Ingestion moved to NV-Ingest 26.3.0.

The embedding NIM (release 2.4) has these features **[web]**:
- 1B (efficiency) and 8B (quality) models behind one OpenAI-compatible API;
- inputs of up to 32K tokens and configurable output dimensions;
- an NVFP4 1B variant for Blackwell;
- embedding of text and of page images, tables and charts.

It descends from the research model NV-Embed (arXiv 2405.17428). NV-Embed is a decoder-only LLM with latent-attention pooling, trained by two-stage contrastive instruction tuning with hard negatives and synthetic data, and it topped MTEB in 2024. NVIDIA's retrieval models are now grouped on Hugging Face as "Nemotron RAG" **[web]**.

## Interview Q&A

**Why hybrid search?** Dense retrieval misses exact identifiers and jargon, and BM25 misses paraphrase. Fuse them with RRF, which needs no score calibration because it uses ranks, or with weighted scores **[web]**.

**Why add a reranker if you already have the top k?** The first stage is a cheap bi-encoder searching millions of chunks. The reranker scores query–passage pairs jointly on a short list, trading latency for precision (about 1.75 s for 500 passages on an H100, per NVIDIA) **[web]**.

**When would you use agentic RAG instead of plain RAG?** For ambiguous or multi-hop questions where one retrieval pass is not enough. It costs extra LLM calls and latency, which is why the Blueprint ships it off by default **[web]**.

**What chunk size?** There is no universal answer. Start with recursive splits of about 200 tokens and measure token-level recall and IoU on your own queries. Contextual enrichment and semantic chunking raise recall at extra compute cost **[web]**.

**RAG or long context?** Small corpora can go straight in the prompt (Anthropic suggests under 200k tokens). At scale, positional degradation in long contexts and the cost per query argue for retrieval **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [RAG (arXiv 2005.11401)](https://arxiv.org/abs/2005.11401) |
| 2026-10-28 | [Evaluating chunking strategies (Chroma)](https://www.trychroma.com/research/evaluating-chunking) |
| 2026-10-28 | [Contextual Retrieval (Anthropic)](https://www.anthropic.com/news/contextual-retrieval) |
| 2026-10-28 | [Lost in the Middle (arXiv 2307.03172)](https://arxiv.org/abs/2307.03172) |
| 2026-10-28 | [Reciprocal rank fusion (Elasticsearch docs)](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/reciprocal-rank-fusion) |
| 2026-10-28 | [RAG Blueprint: hybrid search](https://docs.nvidia.com/rag/latest/hybrid_search.html) |
| 2026-10-28 | [Milvus overview](https://milvus.io/docs/overview.md) |
| 2026-10-28 | [NeMo Retriever text reranking NIM](https://docs.nvidia.com/nim/nemo-retriever/text-reranking/latest/overview.html) |
| 2026-10-28 | [HyDE (arXiv 2212.10496)](https://arxiv.org/abs/2212.10496) |
| 2026-10-28 | [RAG Blueprint: query decomposition](https://docs.nvidia.com/rag/latest/query_decomposition.html) |
| 2026-10-28 | [Self-RAG (arXiv 2310.11511)](https://arxiv.org/abs/2310.11511) |
| 2026-10-28 | [RAG Blueprint: self-reflection](https://docs.nvidia.com/rag/latest/self-reflection.html) |
| 2026-10-28 | [RAG Blueprint: agentic RAG](https://docs.nvidia.com/rag/latest/agentic-rag.html) |
| 2026-10-28 | [RAG Blueprint docs](https://docs.nvidia.com/rag/latest/) |
| 2026-10-28 | [RAG Blueprint release notes](https://docs.nvidia.com/rag/latest/release-notes.html) |
| 2026-10-28 | [NeMo Retriever text embedding NIM](https://docs.nvidia.com/nim/nemo-retriever/text-embedding/latest/overview.html) |
| 2026-10-28 | [NV-Embed (arXiv 2405.17428)](https://arxiv.org/abs/2405.17428) |
| 2026-10-28 | [Nemotron RAG collection (Hugging Face)](https://huggingface.co/collections/nvidia/nemotron-rag) |
