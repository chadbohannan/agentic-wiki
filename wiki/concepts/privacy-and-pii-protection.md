# Privacy and PII Protection

Personal data enters an agent through user messages, retrieved documents, tool results and memory. It leaves through outputs, logs and calls to third-party tools. Protection means detecting, masking and minimising personal data at each of those points.

## Detection

The [NeMo Guardrails](../components/nemo-guardrails.md) rail-types documentation places PII detection in input, retrieval and output rails, which covers the model boundary **[web]**. The detectors in the Guardrails ecosystem include Microsoft Presidio and NVIDIA's GLiNER-PII **[sourced, web]**.

GLiNER-PII's model card (v1.0, released 2025-10-28) gives these details **[web]**:
- **What it is:** a non-generative detector built on GLiNER large-v2.1, with about 570M parameters. It returns entity spans with confidence scores.
- **Coverage:** more than 55 categories of personal and health information.
- **Training data:** about 100,000 synthetic records from NVIDIA's Nemotron-PII dataset, spanning more than 50 industries.
- **Language:** English only.
- **Benchmarks:** strict F1 of 0.87 on Nemotron-PII, 0.70 on Argilla PII and 0.64 on AI4Privacy.
- **Thresholds:** evaluated at 0.3, while the usage example uses 0.5.
- **Licence:** the NVIDIA Open Model License.

The card recommends validation and human review for high-stakes use. The Guardrails integration docs show an example configuration that blocks inputs and outputs containing PII rather than masking them **[web, snippet]**. NVIDIA's 2024 LangChain tutorial, which the study guide cites, has a "sensitive or personal information" self-check that blocks such input and shows no masking **[sourced]**.

## Design principles

Four principles follow from where personal data enters and leaves an agent:
- **Minimise data.** Do not retrieve or store what the task does not need, and redact at ingestion, in the pipeline described in [Document Ingestion and ETL](document-ingestion-etl.md), so personal data never reaches the index.
- **Mask or tokenise** before sending data to external model APIs.
- **Enforce each user's access control at retrieval time**, so the agent cannot fetch what the user cannot see.
- **Scrub memory and logs.** Stored memories in [Agent Memory](agent-memory.md) persist across sessions, and logs pull against privacy for the reasons in [Audit Trails and Traceability](audit-trails-and-traceability.md).
- **Choose between blocking and masking** by use case.

The trade-offs are recall against false positives (set by the threshold), the added latency of each rail, and the detector's coverage of languages and formats. GLiNER-PII is English-only and scores noticeably lower on out-of-domain benchmarks.

GDPR's territorial scope (Article 3) is wider than the EU's borders but narrower than "any EU resident's data": it covers processing in the context of an EU establishment, wherever the processing happens, and processing by a non-EU controller or processor when it offers goods or services to people in the EU or monitors their behaviour there **[web]**. An agent product sold into the EU, or one that tracks EU users' behaviour, is therefore in scope even if it runs elsewhere. The [EU AI Act](eu-ai-act.md), as amended by the 2026 digital omnibus, adds a legal basis for processing special-category data to detect and correct algorithmic bias **[web]**.

## Interview Q&A

**Where should PII controls sit in an agent pipeline?** In input, retrieval and output rails, and also in logs and memory **[web]**.

**Block or mask?** Block when any PII is disallowed. Mask when the task needs the rest of the text.

**What is data minimisation for RAG?** Index and retrieve only what is needed, enforce access control at query time, and expire data.

**What is NVIDIA's PII tool?** GLiNER-PII: an English-only span detector covering more than 55 categories, trained on synthetic data, used through Guardrails or directly. Tune its threshold on your own data **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Building safer LLM apps with LangChain templates and NeMo Guardrails (NVIDIA blog)](https://developer.nvidia.com/blog/building-safer-llm-apps-with-langchain-templates-and-nvidia-nemo-guardrails/) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [GLiNER-PII model card (Hugging Face)](https://huggingface.co/nvidia/gliner-pii) |
| 2026-10-28 | [GDPR Article 3: territorial scope](https://gdpr-info.eu/art-3-gdpr/) |
