# Document Ingestion and ETL

Enterprise RAG needs an extract-transform-load layer. It parses PDFs, Office files, images and audio into structured chunks rich in metadata, embeds them, writes them to a vector store and keeps them fresh. This page covers exam objectives 6.3 and 6.4: extraction, freshness, data-quality checks and synthetic data, with NVIDIA's tooling for each.

Parsing and chunking decisions made here cap the quality of [RAG Retrieval Pipelines](rag-retrieval-pipelines.md), and the embeddings land in the indexes of [Vector Databases and ANN Tuning](vector-databases-and-ann-tuning.md). Handling personal data at ingestion is covered in [Privacy and PII Protection](privacy-and-pii-protection.md), and multimodal content in [Multimodal Agents](multimodal-agents.md).

Retrieval can only return what ingestion preserved. Poor extraction of tables or charts caps answer quality whatever model sits downstream **[inferred]**.

## Extraction

NVIDIA's current documentation describes the NeMo Retriever Library. It extracts text, tables, charts, infographics and transcripts from documents and media, and runs either as a Python library or as a Kubernetes deployment, with inference on NIM microservices or local Nemotron models **[web]**. Its inputs:
- **Documents:** PDF, DOCX, PPTX, HTML (converted to Markdown), TXT, Markdown and JSON.
- **Images:** PNG, JPEG, BMP, TIFF and SVG.
- **Audio:** MP3 and WAV.
- **Video:** MP4, MOV, MKV and AVI.

The pipeline works in stages **[web]**:
1. Split each document into pages.
2. Classify regions of each page as text, table, chart or infographic.
3. Extract each region into a standard schema, using OCR where needed. PDFs use pdfium by default, or Nemotron Parse.
4. Chunk, filter and embed the results, and offload the images.

Output can go to LanceDB, and the library deploys with Docker, Helm or OpenShift. NVIDIA AI Enterprise support does not cover the library, its image or its chart **[web]**.

Names are in transition. The RAG Blueprint's release notes still call the component "NV-Ingest" (26.3.0 in Blueprint 2.6.0) and referred to "NeMo Retriever Library 26.01" in 2.4.0. The OCR NIM was renamed from `nemoretriever-ocr-v1` to `nemotron-ocr-v1` **[web]**. Nemotron-Parse-2.0, a 0.9B image-to-text model, does the layout-aware parsing **[web]**.

The Blueprint adds its own ingestion features **[web]**:
- audio ingestion, and continuous ingestion from object storage;
- OCR configuration, and enhanced PDF extraction through Nemotron Parse;
- custom metadata and a collection-level data catalogue;
- OpenTelemetry ingestion metrics and a `/status` endpoint, added in 2.4.0.

Multimodal embedding places page images, charts and tables in the same space as text. The Blueprint's 2.6.0 default embedder is `llama-nemotron-embed-vl-1b-v2` **[web]**.

## Freshness

There are three common patterns for keeping an index fresh:
- scheduled re-crawls with content hashing;
- event-driven re-ingestion;
- change data capture (CDC) from operational databases.

Debezium is log-based CDC. It reads the database's transaction log (for example, MySQL's binlog), emits one change event per row to Kafka topics, and lets consumers resume where they left off after downtime **[web, Debezium tutorial]**. Wiring CDC into a vector index means upserting and deleting embeddings keyed by document ID. No NVIDIA source fetched describes this **[inferred]**. The Blueprint provides continuous ingestion from object storage **[web]**. Changing the embedding model or schema means a full re-index: even switching the Blueprint from dense to hybrid search required a new collection and re-uploading the documents **[web]**.

## Data quality (6.4)

NeMo Curator (v1.4.0, also numbered 26.09) is aimed at training data. Its text pipeline offers **[web, Curator docs]**:
- **Deduplication:** exact (MD5), fuzzy (MinHash with locality-sensitive hashing) and semantic (embeddings with clustering).
- **Filtering:** heuristic filters, classifier filters and GPU-accelerated distributed classification.
- **Judging:** an LLM-judge workflow with rubrics.
- **Language identification:** fastText, covering 176 languages.

It also curates image, video and audio data. Applying it to RAG corpora is a reasonable extension **[inferred]**. PII redaction was not found in the Curator pages that were read, so do not assert that Curator removes PII without checking. RAG-specific checks, synthesised **[inferred]**:
- deduplicate near-identical chunks, strip boilerplate and filter by language;
- check that metadata is complete and that access-control lists propagate;
- watch the distribution of chunk lengths;
- run golden-query retrieval regression tests after each ingestion change.

## Synthetic data

NeMo Data Designer orchestrates calls to LLM endpoints (NVIDIA, OpenAI or vLLM) and adds batching, parallelism and validation. Users configure sampler and LLM columns, preview the output, and then generate at scale **[web, thin fetch]**. For a RAG team it can produce evaluation question sets and query–passage pairs for fine-tuning the retriever. NV-Embed's training used synthetic data in this way **[web, inferred]**. Search results also show Data Designer recipes for text-to-SQL data **[web, snippet]**. The risks are circularity and the generator's own biases **[inferred]**.

## Interview Q&A

**Why not just use PyPDF?** Tables, charts and scans lose their structure. Layout-aware extraction (page-element detection, OCR, Nemotron Parse) preserves it **[web]**.

**How do you keep the index fresh?** Incremental refresh by content hash, continuous object-store ingestion, CDC for databases, and upsert or delete by document ID. Do a full re-index when the embedding model changes **[web, inferred]**.

**Exact versus fuzzy versus semantic deduplication?** MD5 finds identical documents, MinHash with LSH finds near-duplicates, and embedding clustering finds paraphrases **[web]**.

**What can synthetic data do for a RAG team?** Generate evaluation questions and retriever fine-tuning pairs, with validation. Watch for circularity and the generator's biases **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Retriever Library extraction overview](https://docs.nvidia.com/nemo/retriever/latest/extraction/overview/) |
| 2026-10-28 | [RAG Blueprint docs](https://docs.nvidia.com/rag/latest/) |
| 2026-10-28 | [RAG Blueprint release notes](https://docs.nvidia.com/rag/latest/release-notes.html) |
| 2026-10-28 | [RAG Blueprint: hybrid search](https://docs.nvidia.com/rag/latest/hybrid_search.html) |
| 2026-10-28 | [Nemotron RAG collection (Hugging Face)](https://huggingface.co/collections/nvidia/nemotron-rag) |
| 2026-10-28 | [Debezium tutorial](https://debezium.io/documentation/reference/stable/kc-tutorial.html) |
| 2026-10-28 | [NeMo Curator: process text data](https://docs.nvidia.com/nemo/curator/latest/curate-text/process-data/index.html) |
| 2026-10-28 | [NeMo Data Designer](https://docs.nvidia.com/nemo/datadesigner/) |
| 2026-10-28 | [NV-Embed (arXiv 2405.17428)](https://arxiv.org/abs/2405.17428) |
