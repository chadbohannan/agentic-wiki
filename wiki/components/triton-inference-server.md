# Triton Inference Server (Dynamo-Triton)

Triton is NVIDIA's general-purpose, multi-framework inference server. It provides dynamic and sequence batching, pluggable backends and model ensembles. Since March 2025 NVIDIA has branded it Dynamo-Triton **[web]**.

## Naming

NVIDIA's developer page now calls the product "Dynamo-Triton Open-Source Software, formerly NVIDIA Triton Inference Server" **[web]**. Secondary sources date the rename to Dynamo's launch on 2025-03-18 and describe Dynamo as complementing rather than replacing Triton **[web, snippet]**. [NVIDIA Dynamo](nvidia-dynamo.md) is a separate platform for large-scale LLM serving, with disaggregated serving and KV-cache offload; how the two fit together with NIM is mapped in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md). The user-guide pages the study guide cites still say "Triton Inference Server" **[sourced]**.

## Backends

Triton serves models from many frameworks through backends: TensorRT, ONNX Runtime, TensorFlow, PyTorch, OpenVINO, Python, DALI, FIL, [TensorRT-LLM](tensorrt-llm.md) and vLLM **[sourced]**. A backend is a shared library that implements Triton's C backend API, in which `TRITONBACKEND_ModelInstanceExecute` is the only mandatory function. Decoupled backends can send several responses per request, which is how LLM token streaming fits the model **[sourced]**. Custom logic goes into a new backend library or the Python backend **[sourced]**.

## Batching

According to the batcher guide, the dynamic batcher groups requests for stateless models, the same batching lever explained for LLMs in [LLM Inference Optimization](../concepts/llm-inference-optimization.md). To tune it **[sourced]**:
1. Choose a maximum batch size and enable the default batcher.
2. Measure with Performance Analyzer.
3. Add `max_queue_delay_microseconds`, or a larger batch, to trade latency for throughput.

`preferred_batch_size` is rarely needed outside TensorRT models with several optimisation profiles. The sequence batcher routes all requests of a stateful sequence to one model instance. With `iterative_sequence` it supports in-flight batching **[sourced]**. Ensembles chain pre-processing, model and post-processing steps into one request pipeline. `instance_group` runs several copies of a model so that requests to it are handled simultaneously; by default Triton creates one instance per GPU, and `KIND_CPU` places instances on the CPU **[web]**. The study guide's Triton optimisation page covers the latency side, which is summarised in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md) **[sourced]**. For LLM benchmarking, Performance Analyzer and GenAI-Perf have been superseded by AIPerf, as [LLM Inference Profiling](../concepts/llm-inference-profiling.md) explains **[web]**.

## When to use it

Triton is the general server for mixed workloads: vision, speech, embeddings and ensembles of several models. For LLMs it hosts TensorRT-LLM or vLLM backends. NIM packages an LLM-specific stack behind an OpenAI-compatible API, and Dynamo targets LLM serving across a cluster, as their own pages describe in detail **[web]**.

## Interview Q&A

**Dynamic or sequence batcher?** Dynamic for stateless models, combining requests into batches. Sequence for stateful models, pinning each sequence to one instance **[sourced]**.

**What does `max_queue_delay_microseconds` do, and what does it cost?** It holds requests briefly to form bigger batches, raising throughput at the cost of added latency **[sourced]**.

**How do you add a new framework or custom logic?** Implement a backend against the C API, or use the Python backend **[sourced]**.

**Is Triton deprecated in favour of Dynamo?** No. It was renamed Dynamo-Triton and continues, and Dynamo is a complementary LLM-serving platform **[web, snippet]**.

**How do you find the best configuration?** Measure across batch sizes and concurrency levels, with Performance Analyzer and Model Analyzer, or with AIPerf for LLMs **[sourced, web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Triton backend README](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/backend/README.html) |
| 2026-10-28 | [Triton batchers](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html) |
| 2026-10-28 | [Optimization - NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/optimization.html) |
| 2026-10-28 | [Dynamo-Triton (NVIDIA Developer)](https://developer.nvidia.com/dynamo-triton) |
| 2026-10-28 | [Triton model configuration](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/model_configuration.html) |
