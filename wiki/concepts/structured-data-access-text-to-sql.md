# Structured Data Access and Text-to-SQL

Agents reach structured data in two ways: by generating queries (text-to-SQL), or by calling typed tools that wrap fixed queries. They then combine the results with unstructured retrieval. This page covers exam objective 6.5: the text-to-SQL pattern, how well it works, the safeguards it needs, and how to keep answers fresh.

A database query is exposed to an agent as a tool, so the interface in [Tool Calling and MCP](tool-calling-and-mcp.md) applies. Generated queries should satisfy the shape constraints in [Structured Outputs](structured-outputs.md). Unstructured data goes through [RAG Retrieval Pipelines](rag-retrieval-pipelines.md), and relational questions over documents can use [Knowledge Graphs and GraphRAG](knowledge-graphs-graphrag.md).

## The pattern

Retrieve schema context (table definitions plus documentation of tables and columns) by embedding search, give it to an LLM to write SQL, execute the query, and return the results. NVIDIA's Vanna blog post implements this. It embeds table definitions from `sqlite_master`, together with hand-written documentation, using `llama-3.2-nv-embedqa-1b-v2`, and stores them in Milvus. It retrieves the top two matches and generates SQL with Llama 3.1 70B through NIM. The post admits that quality depends on the hand-written documentation, and it reports no accuracy or latency figures **[web, weak source]**.

## How well it works

The BIRD benchmark (arXiv 2305.03111) tests text-to-SQL on large, realistic databases. Its human baseline is about 93% execution accuracy. Press reports from 2026 put the best submitted systems in the low 80s: Huawei's DataGallery at 82.39% overall on 2026-09-09, and Google's Gemini-SQL2 at 80.04% on the single-model track **[web, press coverage, not checked against the official leaderboard]**. One analysis found that about half of BIRD's financial-domain items contain errors, which limits how much the top scores mean **[web, secondary]**. Even the best systems get roughly one query in five wrong on realistic schemas, so the safeguards below are necessary, not optional **[inferred]**.

## Safeguards

The usual safeguards, synthesised from practice rather than taken from one source **[inferred]**:
- read-only credentials, allow-listed schemas, query timeouts and row limits;
- parsing, validating or dry-running the SQL before executing it;
- pruning the schema for wide catalogues;
- few-shot examples that encode business definitions;
- a repair loop on errors;
- human approval for any write.

The alternative to free-form SQL is a set of parameterised tools exposed through function calling or MCP, which trade flexibility for safety and auditability. NVIDIA's RAG Blueprint exposes an MCP server and an OpenAI-compatible search endpoint from version 2.4.0 **[web, release notes]**.

## Mixing structured and unstructured data

Route each question to the right store:
- lookups and aggregations to SQL;
- narrative questions to vector search;
- some questions to both.

Knowledge graphs offer a middle path **[inferred]**. For freshness, live queries against the system of record are fresh by construction. Derived indexes need CDC and streaming, as described in [Document Ingestion and ETL](document-ingestion-etl.md). Caches need TTLs, and vector indexes lag by the pipeline's latency **[inferred]**. No NVIDIA blueprint for structured-data RAG was found.

## Interview Q&A

**How do you stop a text-to-SQL agent damaging data?** A read-only role, query validation, limits, a sandbox, and human approval for writes **[inferred]**.

**Why embed the schema?** Large catalogues do not fit in the prompt, so retrieve the relevant tables, columns and documentation **[web]**.

**Fixed tools or generated SQL?** Fixed tools are safer and easier to audit. Generated SQL covers the long tail of questions **[inferred]**.

**How accurate is text-to-SQL today?** On BIRD, the best systems reach about 80–82% execution accuracy against a human baseline of about 93% (press-reported, 2026) **[web]**.

**How do you guarantee fresh answers?** Query the live source, or stream changes into the index through CDC and monitor the lag **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Accelerating text-to-SQL inference on Vanna with NVIDIA NIM (NVIDIA blog)](https://developer.nvidia.com/blog/accelerating-text-to-sql-inference-on-vanna-with-nvidia-nim-for-faster-analytics) |
| 2026-10-28 | [BIRD (arXiv 2305.03111)](https://arxiv.org/abs/2305.03111) |
| 2026-10-28 | [Gemini-SQL2 tops BIRD (AI Weekly)](https://aiweekly.co/alerts/googles-gemini-sql2-tops-bird-text-to-sql-at-8004) |
| 2026-10-28 | [Huawei DataGallery tops BIRD (Technology Magazine)](https://technologymagazine.com/news/huawei-datagallery-tops-benchmark-for-enterprise-text2sql-ai) |
| 2026-10-28 | [BIRD and the real-database gap (Beancount research log)](https://beancount.io/bean-labs/research-logs/2026/06/06/bird-benchmark-text-to-sql-real-database-gap) |
| 2026-10-28 | [RAG Blueprint release notes](https://docs.nvidia.com/rag/latest/release-notes.html) |
