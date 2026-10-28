# NVIDIA NIM

NVIDIA NIM packages a model, an optimised inference backend and an OpenAI-compatible API into a pre-built container. NIM chooses how to run the model on the GPUs it finds. Teams get a standard, hardware-aware endpoint instead of hand-building a serving stack. NIM touches exam objectives 7.2 and 7.5, and through its API it also touches 2.3 and 2.5.

NIM serves models through engines such as [TensorRT-LLM](tensorrt-llm.md), vLLM and SGLang. It exposes the structured generation and function calling described in [Structured Outputs](../concepts/structured-outputs.md) and [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md), and it streams as described in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md). It is deployed at scale with the [NVIDIA NIM Operator](nim-operator.md). Its vision-language models are covered in [Multimodal Agents](../concepts/multimodal-agents.md).

## What a NIM is

A NIM container is pulled from NGC with an API key. It packages a model, a backend and an HTTP API **[inferred from the Run:ai page and release notes]**. The LLM API is OpenAI-compatible, covering chat completions, `tools` and `tool_choice`, streaming, and guided decoding through `nvext`. The NIM structured-generation and function-calling pages return no text to the fetch tool, so these details rest on search summaries **[web, snippet-level]**.

## Current state (checked 2026-10-28)

The NIM for LLM and VLM release notes show version 2.1.4 as current, with the page last updated 2026-10-23. They describe NIM LLM as built on the open-source vLLM and SGLang engines **[web]**:
- **Engines:** release 2.1.4 moved to vLLM 0.30.0, SGLang 0.5.20 and Mooncake 0.3.13 for KV-cache transfer.
- **Model-free containers:** alongside the model-specific NIMs, `nvidia/vllm-model-free-nim` and `nvidia/sglang-model-free-nim` load any compatible checkpoint named by `NIM_MODEL_PATH`.
- **Throughput:** NVIDIA claims optimisations for 28 NIMs of "up to 2.1x relative to the upstream open-source engine", with gains depending on the NIM, workload and GPU.

TensorRT-LLM remains a profile backend (`trtllm`) in the 2.0.13 profile documentation **[web]**. This contradicts the older picture of NIM as TensorRT-LLM engines that a September 2025 exam may assume: TensorRT-LLM is now one backend among three.

## Model profiles

A profile is a pre-validated configuration named `<backend>-<precision>-tp<N>-pp1[-lora]` **[web, 2.0.13 docs]**:
- **Backends:** vllm, sglang or trtllm.
- **Precisions:** bf16, fp8, mxfp4 or nvfp4.

At startup NIM picks exactly one profile:
- `NIM_MODEL_PROFILE=default` picks the first compatible profile by backend priority.
- An explicit profile ID or description pins one.
- Otherwise a memory-aware selector estimates VRAM use and discards profiles that do not fit.

Settings are applied in this order: container arguments, then `NIM_PASSTHROUGH_ARGS`, then NIM environment variables, then `runtime_config.json`. `list-model-profiles --gpus=all` shows the options. Profile IDs can change between image versions.

## Deployment

On Kubernetes, the NIM Operator manages NIMs through its NIMCache, NIMService, NIMPipeline and NIMBuild resources **[web]**. Run:ai can also deploy NIMs through Knative **[sourced]**:
- it pulls from `nvcr.io` with NGC credentials (`$oauthtoken` as the user);
- it autoscales between minimum and maximum replicas, including to zero;
- it does not support air-gapped installs, which need the Operator.

## Models and blueprints

NIM serves NVIDIA's Nemotron models and third-party models **[web; Llama Nemotron details from Apidog, a weak source; Nemotron 3 partly snippet-level]**:
- **Llama Nemotron:** Nano 8B, Super 49B and Ultra 253B, with a reasoning toggle and 128K context.
- **Nemotron 3:** a hybrid Mamba-Transformer mixture of experts in Nano, Super and Ultra sizes, with up to 1M context. The NIM docs list `nemotron-3-super-120b-a12b` and a guide for Nemotron 3 Ultra 550B-A55B.
- **Third-party models:** gpt-oss, Gemma 4, Qwen 3.x, GLM and Mistral Medium.

In NVIDIA's agent blueprints, NIM is the optimised serving layer beneath NeMo Retriever and the agent toolkit. The code-generation example serves DeepSeek-R1, Qwen2.5-Coder and Llama 3.3 70B through NIM **[sourced]**.

## Trade-offs

Production use of NIM is licensed under NVIDIA AI Enterprise, with free development access through the Developer Program. Images are gated behind NGC, so air-gapped sites need mirroring or NIMCache **[inferred for licensing]**.

## Interview Q&A

**What is a NIM, and why use one?** A container holding a model, an optimised backend and an OpenAI-compatible API, configured per GPU through profiles. It speeds deployment and standardises the interface **[sourced, inferred]**.

**How does NIM decide how to run a model on your GPUs?** Through model profiles (backend, precision, tensor and pipeline parallelism, LoRA), chosen by `NIM_MODEL_PROFILE` or by automatic memory-aware selection **[web]**.

**How do you run NIM on Kubernetes at scale?** With the NIM Operator (NIMCache, NIMService, and an HPA on DCGM or NIM metrics, plus NIMBuild to cut startup), or with Run:ai on Knative **[web]**.

**Is NIM still TensorRT-LLM underneath?** TensorRT-LLM is one of three backends. The 2.x line is built on vLLM and SGLang and adds model-free containers **[web]**.

**Why can NIM drop into an agent framework?** Its API is OpenAI-compatible, including function calling and structured generation **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
| 2026-10-28 | [NIM model profiles and selection (2.0.13)](https://docs.nvidia.com/nim/large-language-models/2.0.13/deployment/model-profiles-and-selection.html) |
| 2026-10-28 | [NIM Operator docs](https://docs.nvidia.com/nim-operator/latest/index.html) |
| 2026-10-28 | [Run:ai: deploy inference workloads with NVIDIA NIM](https://docs.run.ai/v2.20/Researcher/workloads/inference/nim-inference/) |
| 2026-10-28 | [Using the Llama Nemotron API (Apidog)](https://apidog.com/blog/use-nvidia-llama-nemotron-api/) |
| 2026-10-28 | [AI Agents Blueprint (NVIDIA blog)](https://blogs.nvidia.com/blog/ai-agents-blueprint/) |
| 2026-10-28 | [Improve AI code generation using NVIDIA Agent Intelligence Toolkit (NVIDIA blog)](https://developer.nvidia.com/blog/improve-ai-code-generation-using-nvidia-agent-intelligence-toolkit/) |
