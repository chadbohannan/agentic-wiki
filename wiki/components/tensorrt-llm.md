# TensorRT-LLM

TensorRT-LLM is NVIDIA's open-source library for accelerating LLM inference on NVIDIA GPUs. It has moved from a compile-an-engine workflow to a PyTorch-based runtime with a high-level Python LLM API that scales from one GPU to many nodes **[web]**. It reaches users in three ways: as the `trtllm` backend in [NVIDIA NIM](nvidia-nim.md) profiles, as a backend of [Triton Inference Server (Dynamo-Triton)](triton-inference-server.md), or as one of the engines that [NVIDIA Dynamo](nvidia-dynamo.md) orchestrates across a cluster.

## Features

The current overview documentation lists these features **[web]**:
- in-flight batching;
- paged attention, with a paged KV cache that supports block reuse;
- FP8 on H100 and later, and FP4 on B200;
- speculative decoding (EAGLE, multi-token prediction and n-gram);
- tensor, pipeline and expert parallelism;
- chunked prefill;
- LoRA and guided decoding;
- disaggregated serving, in beta, which splits context (prefill) and generation (decode) across GPUs.

The GitHub README adds context parallelism, "wide expert parallelism" for mixture-of-experts models, and custom attention, GEMM and MoE kernels. It also links quantised checkpoints such as a DeepSeek FP4 model. The version badge read 1.4.0rc2 on 2026-10-28 **[sourced]**. Quantisation recipes come from TensorRT Model Optimizer, and NVFP4 inference ships in both TensorRT-LLM and vLLM **[web]**.

NVIDIA's 2023 inference-optimisation blog explains why each feature matters, a story told engine-neutrally in [LLM Inference Optimization](../concepts/llm-inference-optimization.md) **[sourced]**:
- **Batching** raises decode utilisation, because prefill is compute-bound and decode is memory-bound.
- **In-flight batching** removes static batching's wait for the longest sequence.
- **Paging the KV cache** avoids over-reserving contiguous memory.
- **Quantisation** shrinks weights and KV, raising throughput.
- **Speculative decoding** spends spare compute to cut sequential steps.
- **Parallelism** fits large models across GPUs.

Classic TensorRT guidance still applies underneath: measure a baseline first, then use fusion, precision, CUDA graphs, multiple streams and timing caches. The TensorRT line itself is now on version 11 **[sourced]**.

## Speculative decoding in practice

The speculative-decoding documentation (1.0.0) adds three caveats **[web, snippet]**:
- The draft and target models need identical tokenizers, or acceptance collapses.
- EAGLE-3 is supported in a modified form with a single draft sequence rather than a tree.
- Speculation cannot be switched off dynamically, so the gains appear mainly at low batch sizes.

Widely repeated speedup figures, such as 3.6x for speculative decoding and 40k tokens/s on B200, trace to an unofficial mirror rather than NVIDIA documentation, so do not quote them.

## Workflow change

Before 2025, TensorRT-LLM meant building an engine with `trtllm-build` for a fixed tensor- × pipeline-parallel GPU layout. NVIDIA's October 2024 Kubernetes blog still shows that flow **[sourced]**. The current documentation centres on the PyTorch backend, with `trtllm-serve` for serving and `trtllm-bench` for the load tests described in [LLM Inference Profiling](../concepts/llm-inference-profiling.md) **[sourced, web]**. Triton's backend list still includes both TensorRT-LLM and vLLM **[sourced]**.

## Trade-offs

Aggressive FP8 or FP4 quantisation needs accuracy validation on your own tasks. Speculative-decoding gains depend on the acceptance rate and cost extra memory for the draft model. Larger batches raise throughput but also each request's TTFT and inter-token latency.

## Interview Q&A

**What is in-flight batching?** Continuous batching: finished sequences leave and new requests join at every iteration, instead of the whole batch waiting for its slowest member **[sourced]**.

**Why page the KV cache?** KV memory grows with batch size and sequence length. Fixed-size blocks remove fragmentation and over-reservation, and allow prefix blocks to be reused **[sourced, web]**.

**Which precisions run on which GPUs?** FP8 on H100 and later, FP4 on Blackwell (B200) **[web]**.

**When does speculative decoding help?** In latency-bound, low-batch serving. A cheap drafter proposes tokens and the target verifies them in parallel. At high batch sizes it competes for compute **[sourced, web]**.

**TensorRT-LLM versus vLLM?** Both are engines. TensorRT-LLM is NVIDIA-tuned with NVFP4 and custom kernels, and Dynamo can orchestrate either **[sourced]**.

**Which parallelism for which situation?** Tensor parallelism within a node for latency and fit, pipeline parallelism across nodes, and expert parallelism for mixture-of-experts models, each trading communication against memory **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NVIDIA TensorRT-LLM (GitHub)](https://github.com/NVIDIA/TensorRT-LLM) |
| 2026-10-28 | [TensorRT-LLM overview](https://nvidia.github.io/TensorRT-LLM/latest/overview.html) |
| 2026-10-28 | [TensorRT-LLM performance analysis](https://nvidia.github.io/TensorRT-LLM/performance/perf-analysis.html) |
| 2026-10-28 | [TensorRT-LLM speculative decoding (1.0.0)](https://nvidia.github.io/TensorRT-LLM/1.0.0/features/speculative-decoding.html) |
| 2026-10-28 | [Scaling LLMs with NVIDIA Triton and TensorRT-LLM using Kubernetes (NVIDIA blog)](https://developer.nvidia.com/blog/scaling-llms-with-nvidia-triton-and-nvidia-tensorrt-llm-using-kubernetes/) |
| 2026-10-28 | [Mastering LLM techniques: inference optimization (NVIDIA blog)](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/) |
| 2026-10-28 | [TensorRT best practices](https://docs.nvidia.com/deeplearning/tensorrt/latest/performance/best-practices.html) |
| 2026-10-28 | [Triton backend README](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/backend/README.html) |
| 2026-10-28 | [Introducing NVFP4 (NVIDIA blog)](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/) |
