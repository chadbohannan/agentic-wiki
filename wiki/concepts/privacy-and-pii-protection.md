# Privacy and PII Protection

Personal data enters an agent through user messages, retrieved documents, tool results and memory. It leaves through outputs, logs and calls to third-party tools. This page covers exam objective 9.2: detecting, masking and minimising personal data at each of those points, with NVIDIA's GLiNER-PII model and NeMo Guardrails rails.

PII is handled at ingestion in [Document Ingestion and ETL](document-ingestion-etl.md), and at the model boundary through the rails in [NeMo Guardrails](../components/nemo-guardrails.md). Stored memories need the same care, as [Agent Memory (Short- and Long-Term)](agent-memory.md) notes. The tension with logging is discussed in [Audit Trails and Traceability](audit-trails-and-traceability.md), and the regulatory background is in [EU AI Act](eu-ai-act.md).

## Detection

The NeMo Guardrails rail-types documentation places PII detection in input, retrieval and output rails **[web]**. The detectors in the Guardrails ecosystem include Microsoft Presidio and NVIDIA's GLiNER-PII **[sourced, web]**.

GLiNER-PII's model card (v1.0, released 2025-10-28) gives these details **[web, model card read]**:
- **What it is:** a non-generative detector built on GLiNER large-v2.1, with about 570M parameters. It returns entity spans with confidence scores.
- **Coverage:** more than 55 categories of personal and health information.
- **Training data:** about 100,000 synthetic records from NVIDIA's Nemotron-PII dataset, spanning more than 50 industries.
- **Language:** English only.
- **Benchmarks:** strict F1 of 0.87 on Nemotron-PII, 0.70 on Argilla PII and 0.64 on AI4Privacy.
- **Thresholds:** evaluated at 0.3, while the usage example uses 0.5.
- **Licence:** the NVIDIA Open Model License.

The card recommends validation and human review for high-stakes use. Snippets of the Guardrails integration docs describe an example configuration that blocks inputs and outputs containing PII rather than masking them **[web, snippet]**. NVIDIA's 2024 LangChain tutorial, which the study guide cites, has a "sensitive or personal information" self-check that blocks such input and shows no masking **[sourced]**.

## Design principles

These principles are synthesised **[inferred]**:
- **Minimise data.** Do not retrieve or store what the task does not need.
- **Mask or tokenise** before sending data to external model APIs.
- **Enforce each user's access control at retrieval time**, so the agent cannot fetch what the user cannot see.
- **Scrub memory and logs.**
- **Choose between blocking and masking** by use case.

The trade-offs are recall against false positives (set by the threshold), the added latency of each rail, and the detector's coverage of languages and formats. GLiNER-PII is English-only and scores noticeably lower on out-of-domain benchmarks.

GDPR applies whenever personal data of people in the EU is processed **[inferred, background]**. The EU AI Act, as amended by the 2026 digital omnibus, adds a legal basis for processing special-category data to detect and correct algorithmic bias **[web; one source describes it as proposed, law-firm sources as adopted]**.

## Interview Q&A

**Where should PII controls sit in an agent pipeline?** In input, retrieval and output rails, and also in logs and memory **[web]**.

**Block or mask?** Block when any PII is disallowed. Mask when the task needs the rest of the text **[inferred]**.

**What is data minimisation for RAG?** Index and retrieve only what is needed, enforce access control at query time, and expire data **[inferred]**.

**What is NVIDIA's PII tool?** GLiNER-PII: an English-only span detector covering more than 55 categories, trained on synthetic data, used through Guardrails or directly. Tune its threshold on your own data **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Building safer LLM apps with LangChain templates and NeMo Guardrails (NVIDIA blog)](https://developer.nvidia.com/blog/building-safer-llm-apps-with-langchain-templates-and-nvidia-nemo-guardrails/) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [GLiNER-PII model card (Hugging Face)](https://huggingface.co/nvidia/gliner-pii) |
