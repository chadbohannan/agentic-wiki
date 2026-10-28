# Model and Dataset Licensing

Licences decide which models an organisation may serve, fine-tune and redistribute. Licence terms stack: a model derived from another carries both sets of terms. This page covers the NVIDIA Open Model License, the Llama 3.1 Community License, Gemma terms and Apache 2.0, and how they combine in NVIDIA's own safety models. The questions arise whenever a team serves a model through [NVIDIA NIM](../components/nvidia-nim.md) or fine-tunes one, an option weighed in [RAG vs Fine-Tuning vs Continued Pretraining](rag-vs-fine-tuning-vs-pretraining.md).

## NVIDIA Open Model License

The NVIDIA Open Model License page (last modified 2025-10-24) gives these terms **[web]**:
- **Grant:** the licence is commercial, perpetual, worldwide and royalty-free.
- **Derivatives:** allowed, and owned by their creator, subject to NVIDIA's ownership of the base model.
- **Outputs:** NVIDIA claims no rights in them.
- **Termination:** rights end if you bypass or circumvent a safety guardrail without a substantially similar replacement, or if you sue alleging the model infringes intellectual property.
- **Redistribution:** requires a Notice file with attribution.

The guardrail-bypass clause is unusual. It ties the licence directly to safety engineering: removing a model's guardrails without a replacement voids your right to use it.

## Llama 3.1 Community License

The Llama 3.1 Community License (2024-07-23) sets these terms **[web]**:
- Services with more than 700 million monthly active users need a separate licence from Meta.
- Products must display "Built with Llama".
- Derivative model names must begin with "Llama".
- Meta's acceptable-use policy is incorporated.

## How terms stack

NVIDIA's safety models, described in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md), carry two licences each **[web]**:
- **Safety-Guard-8B-v3:** the NVIDIA Open Model License plus the Llama 3.1 Community License.
- **Content-Safety-Reasoning-4B:** the NVIDIA licence plus the Gemma Terms of Use and Prohibited Use Policy.

GLiNER-PII is under the NVIDIA Open Model License alone. Tools such as garak and NeMo Guardrails are Apache 2.0, a permissive licence with an explicit patent grant and attribution through a NOTICE file **[web]**.

"Open weights" is not the same as "open source" in the Open Source Initiative's sense. The licence field on a model card is a legal claim, to be checked for each version. Dataset licences and data provenance are separate checks.

## Interview Q&A

**Can I use models under the NVIDIA Open Model License commercially?** Yes, with attribution and a notice, provided you do not bypass the model's guardrails without an equivalent replacement **[web]**.

**What is unusual about that licence?** Bypassing safety guardrails terminates your rights **[web]**.

**What must I do if I fine-tune and release a Llama 3.1 model?** Begin its name with "Llama", include the licence, and display "Built with Llama" **[web]**.

**Apache 2.0 versus the Llama licence?** Apache 2.0 is permissive and grants patent rights. The Llama licence adds a user threshold, an acceptable-use policy and naming rules **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA Open Model License](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-license/) |
| 2026-10-28 | [Llama 3.1 Community License](https://dev.meta.ai/llama/llama3_1/license) |
| 2026-10-28 | [GLiNER-PII model card (Hugging Face)](https://huggingface.co/nvidia/gliner-pii) |
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
