# NVIDIA Safety Models and garak

NVIDIA's model-based safety toolchain sits around NeMo Guardrails. It includes classifier models for content safety, topic control and jailbreak detection, a PII detector, and the garak vulnerability scanner. The classifiers run as rails inside [NeMo Guardrails](nemo-guardrails.md) and are served like any other model through [NVIDIA NIM](nvidia-nim.md), so they form the model-based layer of the design in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md). Model names in this area change quickly, so cite the documentation page rather than memory.

## The models

The Guardrails documentation named these models on 2026-10-28 **[web]**:
- Llama 3.1 NemoGuard 8B Content Safety;
- the NemoGuard Jailbreak Detection NIM;
- the NemoGuard Topic Control NIM;
- GLiNER-PII.

The tutorials index adds the Nemotron Content Safety NIM, Nemotron-Content-Safety-Reasoning-4B, and a multimodal safety tutorial that uses a vision LLM as judge. The NIM release notes list `nemotron-3.5-content-safety` **[web]**.

The model cards give these details **[web]**:
- **Safety-Guard-8B-v3** is Llama-3.1-8B-Instruct with a LoRA adapter. It covers 23 safety categories in 9 languages, under the NVIDIA Open Model License plus the Llama 3.1 licence.
- **Content-Safety-Reasoning-4B** is built on Gemma-3-4B-it. It supports custom policies, has a reasoning toggle and a 128K context, and is licensed under the NVIDIA Open Model License plus Gemma terms.

Both licences stack on top of the base model's, which is the general pattern for derived models described in [Model and Dataset Licensing](../concepts/model-and-dataset-licensing.md). The topic-control and jailbreak NIMs deployed on one GPU each, with image tags 1.10.1 in the NeMo Platform docs **[web, snippet]**.

Each model maps to a rail stage **[web]**:
- **Content safety:** input and output.
- **Jailbreak detection:** input.
- **Topic control:** input and dialog.
- **GLiNER-PII:** input, retrieval and output.

A reasoning safety model helps with custom or novel policies, at the cost of latency. With reasoning off it classifies quickly **[web]**.

## garak

garak is an open-source (Apache 2.0) scanner for LLM vulnerabilities, according to its GitHub README. It probes for hallucination, data leakage, prompt injection, misinformation, toxicity and jailbreaks **[web]**. Its toxicity probes supply the red-team measurement step in [Bias and Toxicity Mitigation](../concepts/bias-and-toxicity-mitigation.md). It has five kinds of component:
- **Probes** generate interactions with the model.
- **Detectors** decide whether a response shows a failure.
- **Generators** connect to the target.
- **Harnesses** structure a run. The default, `probewise`, runs each probe with its recommended detectors.
- **Evaluators** report the results.

Targets include Hugging Face, OpenAI, AWS Bedrock, LiteLLM, generic REST endpoints and NIM (`--target_type nim`, with a `NIM_API_KEY`). Example probe families are `encoding` (injection through text encodings), `dan` (DAN-style jailbreaks), `promptinject`, `malwaregen`, `packagehallucination`, `xss` and `realtoxicityprompts`. Each run writes a JSONL report of every attempt and a hit log of the attempts that found a vulnerability **[web]**. Run it before release and in CI against the actual endpoint.

## Interview Q&A

**Which NVIDIA model supports which rail?** Content safety: input and output. Jailbreak detection: input. Topic control: input and dialog. GLiNER-PII: input, retrieval and output **[web]**.

**Why use a reasoning safety model?** Custom or novel policies benefit from explicit reasoning, at higher latency. Turning reasoning off gives fast classification **[web]**.

**What is garak, and when do you run it?** An open-source scanner that fires families of attack probes at an endpoint and detects failures. Run it before release and in CI **[web]**.

**Does a safety NIM remove the need for least privilege?** No. Classifiers are probabilistic, so structural controls still bound the damage.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
