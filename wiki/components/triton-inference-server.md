# Triton Inference Server (Dynamo-Triton)

Triton is NVIDIA's general-purpose, multi-framework inference server. It provides dynamic and sequence batching, pluggable backends and model ensembles. Since March 2025 NVIDIA has branded it Dynamo-Triton. It touches exam objective 7.4, and the study guide's Triton pages also appear under objective 2.5.

For LLMs, Triton hosts engines such as [TensorRT-LLM](tensorrt-llm.md). Its batching and instance settings are serving-layer levers in [LLM Inference Optimization](../concepts/llm-inference-optimization.md). Its latency trade-offs for streaming are discussed in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md).

## Naming

NVIDIA's developer page now calls the product "Dynamo-Triton Open-Source Software, formerly NVIDIA Triton Inference Server" **[web]**. Search snippets date the rename to 2025-03-18 and say Dynamo does not replace Triton **[web, snippet]**. [NVIDIA Dynamo](nvidia-dynamo.md) is a separate platform for large-scale LLM serving, with disaggregated serving and KV-cache offload. The user-guide pages the study guide cites still say "Triton Inference Server" **[sourced]**.

## Backends

Triton serves models from many frameworks through backends: TensorRT, ONNX Runtime, TensorFlow, PyTorch, OpenVINO, Python, DALI, FIL, TensorRT-LLM and vLLM **[sourced]**. A backend is a shared library that implements Triton's C backend API, in which `TRITONBACKEND_ModelInstanceExecute` is the only mandatory function. Decoupled backends can send several responses per request, which is how LLM token streaming fits the model **[sourced]**. Custom logic goes into a new backend library or the Python backend **[sourced, inferred]**.

## Batching

According to the batcher guide, the dynamic batcher groups requests for stateless models. To tune it **[sourced]**:
1. Choose a maximum batch size and enable the default batcher.
2. Measure with Performance Analyzer.
3. Add `max_queue_delay_microseconds`, or a larger batch, to trade latency for throughput.

`preferred_batch_size` is rarely needed outside TensorRT models with several optimisation profiles. The sequence batcher routes all requests of a stateful sequence to one model instance. With `iterative_sequence` it supports in-flight batching **[sourced]**. Ensembles chain pre-processing, model and post-processing steps into one request pipeline. `instance_group` runs several copies of a model concurrently **[inferred; the ensemble page was not read]**. The study guide's Triton optimisation page covers the latency side, which is summarised on the streaming page **[sourced]**. For LLM benchmarking, Performance Analyzer and GenAI-Perf have been superseded by AIPerf, as [LLM Inference Profiling](../concepts/llm-inference-profiling.md) explains **[web]**.

## When to use it

Triton is the general server for mixed workloads: vision, speech, embeddings and ensembles of several models. For LLMs it hosts TensorRT-LLM or vLLM backends. NIM packages an LLM-specific stack behind an OpenAI-compatible API, and Dynamo targets LLM serving across a cluster **[inferred]**.

## Interview Q&A

**Dynamic or sequence batcher?** Dynamic for stateless models, combining requests into batches. Sequence for stateful models, pinning each sequence to one instance **[sourced]**.

**What does `max_queue_delay_microseconds` do, and what does it cost?** It holds requests briefly to form bigger batches, raising throughput at the cost of added latency **[sourced]**.

**How do you add a new framework or custom logic?** Implement a backend against the C API, or use the Python backend **[sourced, inferred]**.

**Is Triton deprecated in favour of Dynamo?** No. It was renamed Dynamo-Triton and continues, and Dynamo is a complementary LLM-serving platform **[web, snippet for the FAQ wording]**.

**How do you find the best configuration?** Measure across batch sizes and concurrency levels, with Performance Analyzer and Model Analyzer, or with AIPerf for LLMs **[sourced, web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Triton backend README](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/backend/README.html) |
| 2026-10-28 | [Triton batchers](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html) |
| 2026-10-28 | [Optimization - NVIDIA Triton Inference Server](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/optimization.html) |
| 2026-10-28 | [Dynamo-Triton (NVIDIA Developer)](https://developer.nvidia.com/dynamo-triton) |
