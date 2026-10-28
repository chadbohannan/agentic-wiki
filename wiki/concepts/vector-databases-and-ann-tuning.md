# Vector Databases and ANN Tuning

Vector stores serve approximate nearest-neighbour (ANN) search, trading a little recall for a large gain in speed over exact search. The index family and its parameters set the balance between recall, latency and memory, and metadata filtering interacts badly with ANN unless it is handled deliberately. This page covers exam objective 6.2.

These indexes are the retrieval store of [RAG Retrieval Pipelines](rag-retrieval-pipelines.md). They are filled by [Document Ingestion and ETL](document-ingestion-etl.md), and they also back the long-term memory in [Agent Memory (Short- and Long-Term)](agent-memory.md).

## Index families

**HNSW** (Malkov and Yashunin, arXiv 1603.09320) builds a multi-layer proximity graph, much like a skip list **[web, abstract]**. Each element's top layer is drawn from an exponentially decaying distribution, and a search descends from sparse, long-range upper layers, giving logarithmic scaling.

pgvector's HNSW has three main parameters **[web, pgvector README]**:
- `m`, with a default of 16.
- `ef_construction`, with a default of 64. Higher values give better recall but slower builds.
- `hnsw.ef_search`, set at query time, with a default of 40.

Compared with IVFFlat, HNSW has a better speed–recall trade-off, but it builds more slowly and uses more memory. It can be created on an empty table.

**IVF** partitions the vectors into `lists` (cells around centroids), and a query probes only the nearest `probes` cells. pgvector's guidance for IVFFlat **[web]**:
- Start `lists` at rows ÷ 1000 for up to 1M rows, and at √rows beyond that.
- `ivfflat.probes` defaults to 1. A suggested starting point is √lists.
- Build the index after loading the data, because the centroids are learned from it.

IVF builds faster and uses less memory than HNSW, at a worse recall trade-off.

**Compression.** Product quantisation (IVF-PQ) compresses vectors into short codes to save memory. It costs recall, which is usually recovered by re-ranking candidates with the full vectors **[inferred; PQ from general knowledge]**. pgvector offers `halfvec` (half precision, indexable up to 4,000 dimensions) and binary quantisation, which fetches candidates by Hamming distance and then re-ranks them with the original vectors **[web]**. NVIDIA's embedding NIM supports configurable output dimensions, which also shrinks storage. Whether it uses Matryoshka-style training is not stated on that page **[web]**.

## Tuning

A tuning recipe, synthesised from the parameters above **[inferred]**:
1. Build a ground-truth set by brute-force search.
2. Sweep `ef_search` or `probes`, plotting recall@k against p95 latency.
3. Choose the smallest setting that meets the recall target.
4. Raise `ef_construction` or `m` only if recall plateaus too low.
5. Re-tune after the data grows or the embedding model changes.

## Filtering

pgvector applies filters after the approximate index scan, so a selective filter can return fewer than k rows **[web, pgvector README]**. Its mitigations are:
- a B-tree index for very selective filters;
- partial indexes when a filter has few distinct values;
- partitioning when it has many, for example per tenant;
- iterative scans (`hnsw.iterative_scan` in strict or relaxed order), bounded by `hnsw.max_scan_tuples`, default 20,000.

Milvus supports metadata-filtered ANN and range search **[web]**. The RAG Blueprint supports custom metadata, and from 2.6.0 it can generate filter expressions dynamically for Elasticsearch **[web, release notes]**.

## Milvus and GPU indexing

Milvus is Apache-2.0 licensed, separates compute from storage, and runs stateless components on Kubernetes. It has Lite, Standalone and Distributed modes **[web, Milvus overview]**:
- **Indexes:** HNSW, IVF, FLAT, SCANN and DiskANN, with quantised variants, memory mapping, and GPU indexing including CAGRA.
- **Search:** hybrid multi-vector search and BM25 full-text search.

Its "30–70% faster than FAISS and HNSWLib" claim is the vendor's own.

CAGRA (arXiv 2308.15136, ICDE 2024) is a graph ANN index designed for parallel hardware. Its authors report construction 2.2–27x faster than HNSW, large-batch throughput 33–77x higher at 90–95% recall, and single-query search 3.4–53x faster at 95% recall **[web, authors' benchmarks]**. NVIDIA's cuVS page claims 21x faster index builds, 12.5x lower build cost, 29x search throughput and 11x lower latency. The page is internally inconsistent, also stating 34x **[web, vendor]**. It lists integrations with FAISS, Milvus, Lucene, Kinetica, Solr, OpenSearch, Weaviate, DataStax and Elastic, and notes that GPU-built indexes can be converted for CPU search. It does not mention pgvector. In the RAG Blueprint 2.6.0, GPU-accelerated Elasticsearch requires enterprise access and is off by default, so GPU indexing is optional in NVIDIA's own reference design **[web]**.

## Choosing a store

pgvector suits modest data that already lives in Postgres, where transactions, joins and running one system matter. A dedicated store such as Milvus or Elasticsearch suits large scale, hybrid search and GPU indexing. GPU index builds matter most when very large corpora are re-indexed often **[inferred]**.

## Interview Q&A

**HNSW versus IVF?** HNSW has better recall and latency but more memory and slower builds, and needs no training. IVF is cheaper to build and store, needs data to train its centroids, and its recall depends on the number of probes **[web]**.

**A filtered query returns too few results. Why?** The filter is applied after the ANN scan. Use iterative scans, partial indexes or partitions, or a store with filter-aware search **[web]**.

**How do you tune recall against latency?** Measure recall against brute force and sweep `ef_search` or `probes`. Consider quantisation with re-ranking to save memory **[web, inferred]**.

**Why use a GPU for vector search?** For much faster index builds and high-throughput batch search with CAGRA and cuVS. Indexes built on GPU can then serve from CPU. The speedups are author- and vendor-reported **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [HNSW (arXiv 1603.09320)](https://arxiv.org/abs/1603.09320) |
| 2026-10-28 | [pgvector (GitHub)](https://github.com/pgvector/pgvector) |
| 2026-10-28 | [Milvus overview](https://milvus.io/docs/overview.md) |
| 2026-10-28 | [CAGRA (arXiv 2308.15136)](https://arxiv.org/abs/2308.15136) |
| 2026-10-28 | [NVIDIA cuVS](https://developer.nvidia.com/cuvs) |
| 2026-10-28 | [NeMo Retriever text embedding NIM](https://docs.nvidia.com/nim/nemo-retriever/text-embedding/latest/overview.html) |
| 2026-10-28 | [RAG Blueprint release notes](https://docs.nvidia.com/rag/latest/release-notes.html) |
