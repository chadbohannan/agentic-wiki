# RAG Evaluation and Ragas

Evaluating retrieval-augmented generation means scoring retrieval and generation separately, so that a bad answer can be traced to the retriever or to the generator. This page covers exam objectives 3.1 and 3.2. It defines Ragas's core metrics and how the NeMo Agent Toolkit uses them.

The metrics evaluate the pipelines built in [RAG Retrieval Pipelines](rag-retrieval-pipelines.md) and run inside the harness of [Agent Evaluation Pipelines](agent-evaluation-pipelines.md). Most are computed by an LLM judge, so the caveats in [LLM-as-Judge](llm-as-judge.md) apply.

## Dimensions and metrics

RAG quality has three independent parts **[web, arXiv 2309.15217 abstract]**:
- Did retrieval return relevant, focused context?
- Is the answer faithful to that context?
- Is the answer correct and relevant to the question?

The Ragas paper proposes reference-free metrics for these dimensions, so evaluation does not depend on human-written ground truth. The Ragas documentation (read 2026-10-28) defines the core metrics as follows **[web]**:
- **Faithfulness** = (claims in the response supported by the retrieved context) ÷ (total claims in the response).
- **Context recall** (LLM variant) = (claims in the reference answer supported by the retrieved context) ÷ (total claims in the reference). The reference answer stands in for annotated reference contexts. A non-LLM variant counts the reference contexts that were retrieved.
- **Context precision** averages precision@k over the ranked chunks, counting only the relevant ones. A relevant chunk ranked below an irrelevant one lowers the score.

The documentation lists further metric families by name: Context Entities Recall, Noise Sensitivity and Response Relevancy; the NVIDIA metrics Answer Accuracy, Context Relevance and Response Groundedness; agent metrics; lexical metrics such as BLEU, ROUGE and exact match; and rubric metrics such as AspectCritic **[web, names only]**.

The NeMo Agent Toolkit's evaluation guide recommends three NVIDIA metrics **[sourced]**:
- `AnswerAccuracy` compares the answer with the ground truth.
- `ContextRelevance` compares the retrieved context with the question.
- `ResponseGroundedness` compares the answer with the retrieved context.

It also allows `FactualCorrectness` with keyword arguments **[sourced]**.

## Diagnosis

The point of separate metrics is attribution **[inferred]**:
- **Low context relevance or recall** points at the retriever, chunking or embeddings.
- **Good context but low faithfulness or groundedness** points at the generator's prompt or at hallucination.
- **Faithful but inaccurate answers** suggest the corpus itself is wrong or stale.

The study guide's DLI course for this objective, "Evaluating RAG and Semantic Search Systems", could not be read, so its syllabus is not represented here.

## Interview Q&A

**How do you separate retriever failure from generator failure?** Score context relevance and recall separately from faithfulness, groundedness and answer accuracy **[inferred, sourced metric names]**.

**Define faithfulness and context recall.** Faithfulness is the share of the answer's claims that the retrieved context supports. Context recall is the share of the reference answer's claims that the retrieved context supports **[web]**.

**Why reference-free metrics?** Ground-truth labels are expensive, and judged metrics scale **[web]**.

**Which three Ragas metrics does the NeMo Agent Toolkit recommend for RAG?** `AnswerAccuracy`, `ContextRelevance` and `ResponseGroundedness` **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Course: Evaluating RAG and Semantic Search Systems (NVIDIA DLI)](https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+S-FX-32+V1) |
| 2026-10-28 | [NeMo Agent Toolkit evaluate guide (docs source, develop)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/develop/docs/source/improve-workflows/evaluate.md) |
| 2026-10-28 | [Ragas (arXiv 2309.15217)](https://arxiv.org/abs/2309.15217) |
| 2026-10-28 | [Ragas available metrics](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/) |
| 2026-10-28 | [Ragas: Faithfulness](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/faithfulness/) |
| 2026-10-28 | [Ragas: Context Recall](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/context_recall/) |
| 2026-10-28 | [Ragas: Context Precision](https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/context_precision/) |
