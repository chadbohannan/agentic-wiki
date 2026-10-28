# LLM Inference Profiling

Profiling an LLM service happens at two layers. Load generation measures latency and throughput as a user sees them. GPU and system traces then show where the time goes. This page covers exam objective 4.3: the metric definitions, the tools (AIPerf, which replaces GenAI-Perf and Perf Analyzer for LLMs; Nsight Systems; DCGM), and how agent traffic changes the method.

Profiling measures engines such as [TensorRT-LLM](../components/tensorrt-llm.md) and clusters managed by [NVIDIA Dynamo](../components/nvidia-dynamo.md), with GPU metrics from [DCGM GPU Telemetry](../components/dcgm-gpu-telemetry.md). The findings feed [Agent Profiling and Result Analysis](agent-profiling-and-result-analysis.md) at the agent level and [Inference Cost and Availability](inference-cost-and-availability.md).

## Metrics

NVIDIA's NIM benchmarking guide defines the standard metrics **[sourced]**:
- **Time to first token (TTFT):** from the query to the first non-empty token. It includes queueing, prefill and network time.
- **Inter-token latency (ITL, also called time per output token):** the average gap between output tokens, excluding TTFT. ITL = (E2E − TTFT) / (output tokens − 1).
- **End-to-end latency (E2E):** TTFT plus generation time.
- **Total tokens per second:** rises with concurrency until compute saturates, and can then fall.
- **Tokens per second per user:** output length divided by E2E. It falls as concurrency rises.
- **Requests per second.**

Tools define these slightly differently, so compare only like with like.

## Load generation: AIPerf

NVIDIA's blog of 2026-09-18 names AIPerf as the "designated successor" to GenAI-Perf **[web]**:
- **Why it replaced GenAI-Perf:** GenAI-Perf was built on Triton's Perf Analyzer and ran as a single process, which becomes limited by Python's GIL at high concurrency or request rates.
- **Architecture:** AIPerf is a ground-up rewrite. Worker processes generate load and separate record-processor services handle results, coordinated over ZMQ.
- **Load modes:** concurrency and request-rate modes with ramping and sweeps; constant, Poisson and gamma arrival patterns; and replay of traces in the Mooncake, Baseten and WEKA formats.
- **GPU metrics:** collected through DCGM or pynvml, alongside the latency results.

AIPerf's migration guide notes one difference for reasoning models. AIPerf's TTFT counts the first token of any type, so compare GenAI-Perf's TTFT with AIPerf's time to first output token **[web, snippet]**. The study guide's Triton material uses Perf Analyzer, which this supersedes for LLM workloads, as [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md) notes. For agents, a single request is the wrong unit: real load is bursty, multi-step sessions, so replay traces or model concurrency at session level **[inferred]**.

## Finding the bottleneck

Nsight Systems gives a combined CPU and GPU timeline. It covers CUDA, cuBLAS, cuDNN and TensorRT activity; NVLink, PCIe and DRAM traffic; Tensor Core use; a multi-node view; and NVTX ranges **[sourced]**. TensorRT-LLM's performance-analysis guide explains how to bound a capture to a window of iterations, using `TLLM_PROFILE_START_STOP` with `nsys -c cudaProfilerApi`. It also explains how to add NVTX, GIL and garbage-collection markers, and optionally the PyTorch profiler **[sourced]**. For continuous fleet monitoring, DCGM's profiling fields and health checks are described on its own page. Its profiling conflicts with Nsight, so pause it while tracing **[sourced]**.

## Interview Q&A

**TTFT versus ITL: what drives each?** TTFT is queueing plus prefill plus network time. ITL is decode speed, which is bound by memory bandwidth **[sourced]**.

**Why did NVIDIA replace GenAI-Perf?** Its single-process client became the bottleneck at high concurrency because of the GIL, which skewed results **[web]**.

**GPU utilisation is 90% but throughput is poor. Why?** Utilisation only says a kernel was resident. Check SM, tensor and DRAM activity, batch size, and CPU-side stalls in the timeline **[inferred]**.

**How would you load-test an agent backend?** Replay recorded multi-step session traces with realistic arrival patterns, rather than firing independent single requests **[inferred, web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NIM LLM benchmarking metrics](https://docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html) |
| 2026-10-28 | [Benchmarking LLM inference at scale with AIPerf (NVIDIA blog)](https://developer.nvidia.com/blog/benchmarking-llm-inference-at-scale-with-aiperf/) |
| 2026-10-28 | [Migrating from GenAI-Perf to AIPerf](https://docs.nvidia.com/aiperf/getting-started/migrating-from-gen-ai-perf) |
| 2026-10-28 | [NVIDIA Nsight Systems](https://developer.nvidia.com/nsight-systems) |
| 2026-10-28 | [TensorRT-LLM performance analysis](https://nvidia.github.io/TensorRT-LLM/performance/perf-analysis.html) |
| 2026-10-28 | [DCGM feature overview](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html) |
