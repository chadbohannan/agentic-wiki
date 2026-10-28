# NVIDIA NIM

NVIDIA NIM packages a model, an optimised inference backend and an OpenAI-compatible API into a pre-built container. NIM chooses how to run the model on the GPUs it finds. Teams get a standard, hardware-aware endpoint instead of hand-building a serving stack.

## What a NIM is

A NIM container is pulled from NGC with an API key. It packages a model, a backend and an HTTP API. The 2.1.4 API reference lists an OpenAI-compatible `/v1/chat/completions` with streaming and tool calling, `/v1/completions`, the OpenAI Responses API at `/v1/responses`, and an Anthropic-compatible `/v1/messages`, beside NIM's own health, metadata, manifest and Prometheus metrics endpoints. In 2.x the engine in the image, vLLM or SGLang, decides which request features exist, so the schema-constrained output in [Structured Outputs](../concepts/structured-outputs.md) uses the engine's parameters rather than the NIM-specific fields of the 1.x line, and client code written for 1.x may need changes **[web]**.

Tool calling, the mechanism in [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md), is switched on at startup with a parser that matches the model: vLLM images take `--enable-auto-tool-choice --tool-call-parser <parser>`, SGLang images take only `--tool-call-parser`, and parser names differ between the engines. NIM does not connect to MCP servers itself; the client converts MCP tool definitions to the OpenAI `tools` format and executes the calls the model returns **[web]**. Because the API is OpenAI-compatible, the token and tool-progress streaming in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md) works with standard clients.

## Status

The NIM for LLM and VLM release notes show version 2.1.4 as current, with the page last updated 2026-10-23. They describe NIM LLM as built on the open-source vLLM and SGLang engines **[web]**:
- **Engines:** release 2.1.4 moved to vLLM 0.30.0, SGLang 0.5.20 and Mooncake 0.3.13 for KV-cache transfer.
- **Model-free containers:** alongside the model-specific NIMs, `nvidia/vllm-model-free-nim` and `nvidia/sglang-model-free-nim` load any compatible checkpoint named by `NIM_MODEL_PATH`.
- **Throughput:** NVIDIA claims optimisations for 28 NIMs of "up to 2.1x relative to the upstream open-source engine", with gains depending on the NIM, workload and GPU.

[TensorRT-LLM](tensorrt-llm.md) remains a profile backend (`trtllm`) in the 2.0.13 profile documentation **[web]**. This contradicts the older picture of NIM as TensorRT-LLM engines that a September 2025 exam may assume: TensorRT-LLM is now one backend among three, one of several such shifts tracked in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md).

## Model profiles

A profile is a pre-validated configuration named `<backend>-<precision>-tp<N>-pp1[-lora]` **[web]**:
- **Backends:** vllm, sglang or trtllm.
- **Precisions:** bf16, fp8, mxfp4 or nvfp4.

At startup NIM picks exactly one profile:
- `NIM_MODEL_PROFILE=default` picks the first compatible profile by backend priority.
- An explicit profile ID or description pins one.
- Otherwise a memory-aware selector estimates VRAM use and discards profiles that do not fit.

Settings are applied in this order: container arguments, then `NIM_PASSTHROUGH_ARGS`, then NIM environment variables, then `runtime_config.json`. `list-model-profiles --gpus=all` shows the options. Profile IDs can change between image versions **[web]**.

## Deployment

On Kubernetes, the [NVIDIA NIM Operator](nim-operator.md) manages NIMs through its NIMCache, NIMService, NIMPipeline and NIMBuild resources **[web]**. Run:ai can also deploy NIMs through Knative **[sourced]**:
- it pulls from `nvcr.io` with NGC credentials (`$oauthtoken` as the user);
- it autoscales between minimum and maximum replicas, including to zero;
- it does not support air-gapped installs, which need the Operator.

## Models and blueprints

NIM serves NVIDIA's Nemotron models and third-party models, including the vision-language models used in [Multimodal Agents](../concepts/multimodal-agents.md) **[web, snippet]**. Images, audio and video reach those models as OpenAI-style content parts, with per-model limits on images and video frames, as detailed on that page **[web]**. The Llama Nemotron item below comes from Apidog, a weak secondary source **[sourced]**, and the other two from NVIDIA's NIM documentation **[web]**:
- **Llama Nemotron:** Nano 8B, Super 49B and Ultra 253B, with a reasoning toggle and 128K context.
- **Nemotron 3:** a hybrid Mamba-Transformer mixture of experts in Nano, Super and Ultra sizes, with up to 1M context. The NIM docs list `nemotron-3-super-120b-a12b` and a guide for Nemotron 3 Ultra 550B-A55B.
- **Third-party models:** gpt-oss, Gemma 4, Qwen 3.x, GLM and Mistral Medium.

In NVIDIA's agent blueprints, NIM is the optimised serving layer beneath NeMo Retriever and the agent toolkit. The code-generation example serves DeepSeek-R1, Qwen2.5-Coder and Llama 3.3 70B through NIM **[sourced]**.

## Trade-offs

According to NVIDIA's NIM FAQ, production use (serving real end users or supporting business transactions) requires an NVIDIA AI Enterprise licence. The free Developer Program covers prototyping, research, development and testing on up to 16 GPUs, and a free 90-day evaluation licence includes commercial use **[web]**. Because images are pulled from NGC, air-gapped sites need mirroring or the Operator's NIMCache.

## Interview Q&A

**What is a NIM, and why use one?** A container holding a model, an optimised backend and an OpenAI-compatible API, configured per GPU through profiles. It speeds deployment and standardises the interface **[sourced]**.

**How does NIM decide how to run a model on your GPUs?** Through model profiles (backend, precision, tensor and pipeline parallelism, LoRA), chosen by `NIM_MODEL_PROFILE` or by automatic memory-aware selection **[web]**.

**How do you run NIM on Kubernetes at scale?** With the NIM Operator (NIMCache, NIMService, and an HPA on DCGM or NIM metrics, plus NIMBuild to cut startup), or with Run:ai on Knative **[web]**.

**Is NIM still TensorRT-LLM underneath?** TensorRT-LLM is one of three backends. The 2.x line is built on vLLM and SGLang and adds model-free containers **[web]**.

**Why can NIM drop into an agent framework?** It serves the OpenAI Chat Completions and Responses APIs and an Anthropic-compatible Messages API, with tool calling once a parser is configured, so OpenAI and Anthropic clients work unchanged **[web]**.

**How do you get JSON that matches a schema from NIM?** It depends on the version: an `nvext` field in 1.1, a top-level `guided_json` in later 1.x, and the engine's own structured-output parameters in 2.x **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA NIM general FAQ](https://docs.api.nvidia.com/nim/docs/product) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
| 2026-10-28 | [NIM model profiles and selection (2.0.13)](https://docs.nvidia.com/nim/large-language-models/2.0.13/deployment/model-profiles-and-selection.html) |
| 2026-10-28 | [NIM Operator docs](https://docs.nvidia.com/nim-operator/latest/index.html) |
| 2026-10-28 | [NIM for LLM and VLM: API reference](https://docs.nvidia.com/nim/large-language-models/latest/reference/api-reference.html) |
| 2026-10-28 | [NIM for LLM and VLM: tool calling and MCP integration](https://docs.nvidia.com/nim/large-language-models/latest/advanced-use-cases/tool-calling-and-mcp.html) |
| 2026-10-28 | [NIM for LLMs 1.15: structured generation](https://docs.nvidia.com/nim/large-language-models/1.15.0/structured-generation.html) |
| 2026-10-28 | [Run:ai: deploy inference workloads with NVIDIA NIM](https://docs.run.ai/v2.20/Researcher/workloads/inference/nim-inference/) |
| 2026-10-28 | [Using the Llama Nemotron API (Apidog)](https://apidog.com/blog/use-nvidia-llama-nemotron-api/) |
| 2026-10-28 | [AI Agents Blueprint (NVIDIA blog)](https://blogs.nvidia.com/blog/ai-agents-blueprint/) |
| 2026-10-28 | [Improve AI code generation using NVIDIA Agent Intelligence Toolkit (NVIDIA blog)](https://developer.nvidia.com/blog/improve-ai-code-generation-using-nvidia-agent-intelligence-toolkit/) |
