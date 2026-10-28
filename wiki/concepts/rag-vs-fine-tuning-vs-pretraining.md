# RAG vs Fine-Tuning vs Continued Pretraining

There are three ways to give a model domain knowledge. Retrieval injects it at inference time, while fine-tuning and continued pretraining change the weights. The evidence says retrieval is more reliable for facts, and fine-tuning is better for style, format and behaviour. Hybrids such as RAFT combine the two. The choice is the parametric versus non-parametric split that also organises [Agent Memory](agent-memory.md). The study guide's own sources were unavailable, so the page rests on papers.

## Evidence

Ovadia et al. (arXiv 2312.05934) compared unsupervised fine-tuning with RAG on knowledge-intensive tasks. Fine-tuning gave some gain, but RAG, built as in [RAG Retrieval Pipelines](rag-retrieval-pipelines.md), consistently outperformed it, for knowledge the model had already seen and for new knowledge. Models struggled to learn new facts through unsupervised fine-tuning, although exposing them to many paraphrases of a fact may help **[web]**.

RAFT (arXiv 2403.10131) fine-tunes a model for open-book question answering in a domain. During training each question comes with retrieved documents, some relevant and some distractors. The model learns to ignore the unhelpful ones, cite from the helpful ones and answer with chain-of-thought. The authors report gains on PubMed, HotpotQA and Gorilla **[web]**.

## Decision guide

Read together, the evidence above gives a rule of thumb for each option:
- **Retrieval** fits when knowledge changes often, or needs citations, access control or deletion. Retrieved documents can be removed, but facts in weights cannot.
- **Fine-tuning**, whether SFT, LoRA or preference tuning, fits when you need a consistent format, tone or tool-use behaviour, or lower latency and cost on a narrow task. Training on agent rewards, rather than on knowledge, is covered in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md).
- **Continued pretraining** fits when you have a large unlabelled in-domain corpus and the model lacks the domain's vocabulary. It is expensive, and you still need retrieval for facts.

The practical default is retrieval first, then fine-tuning the retriever, the embedding model or the generator where failure analysis points. Fine-tuning the embedding model on synthetic query–passage pairs is a cheap, high-yield step. NV-Embed, for example, was trained with synthetic data and hard negatives **[web]**.

## Interview Q&A

**A customer has 10,000 internal documents updated weekly. RAG or fine-tuning?** RAG, for freshness, citations and access control. Possibly also fine-tune the embedder.

**What does RAFT add?** It trains the generator to cope with noisy retrieved context, ignoring distractors and citing the relevant passages **[web]**.

**When is continued pretraining justified?** When there is a large corpus in a specialised language and the training budget to match. It is not a fix for facts that change quickly.

**Why can't fine-tuning guarantee that a fact is recalled?** Facts are learned weakly from unsupervised fine-tuning, and they cannot be updated or deleted precisely once they are in the weights **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Course: Adding New Knowledge to LLMs (NVIDIA DLI)](https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+C-FX-26+V1) |
| 2026-10-28 | [Fine-tuning or retrieval? (arXiv 2312.05934)](https://arxiv.org/abs/2312.05934) |
| 2026-10-28 | [RAFT (arXiv 2403.10131)](https://arxiv.org/abs/2403.10131) |
| 2026-10-28 | [NV-Embed (arXiv 2405.17428)](https://arxiv.org/abs/2405.17428) |
