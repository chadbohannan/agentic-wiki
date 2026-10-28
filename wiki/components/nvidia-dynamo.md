# NVIDIA Dynamo

NVIDIA Dynamo is an open-source, datacenter-scale orchestration layer for LLM inference. It is not an inference engine: engines such as [TensorRT-LLM](tensorrt-llm.md), vLLM and SGLang do the token computation, and Dynamo coordinates GPUs across nodes around them. It is NVIDIA's implementation of the techniques explained in [Disaggregated Serving and KV-Aware Routing](../concepts/disaggregated-serving-and-kv-routing.md).

## Status and history

NVIDIA announced Dynamo at GTC on 2025-03-18 as the platform for LLM-scale serving. Triton Inference Server was folded in and renamed NVIDIA Dynamo-Triton, as noted on the [Triton Inference Server](triton-inference-server.md) page **[sourced]**. Dynamo sits above NIM and the engines rather than replacing them, the layering laid out in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md). Dynamo is written in Rust for performance, with Python for extensibility **[web]**. Its documentation showed v1.6.1 as the latest release on 2026-10-28, with versions back to 0.7.0. The README's quick-start containers are tagged 1.6.0, and a news item dated 03/15 announced Dynamo 1.0 **[web]**.

## Components

As of 2026-10-28 the README lists these components **[web]**:
- **Frontend:** an OpenAI-compatible HTTP entry point.
- **KV-aware router:** routes by worker load and KV-cache overlap, to avoid redundant prefill.
- **Planner:** an SLA-driven autoscaler that profiles workloads and right-sizes the prefill and decode GPU pools against time to first token (TTFT) and inter-token latency (ITL), the metrics a user feels in [Streaming and Real-Time Agent UX](../concepts/streaming-and-realtime-agent-ux.md).
- **KV Block Manager (KVBM):** offloads KV cache from GPU to CPU memory, SSD and remote storage.
- **NIXL:** a point-to-point transfer library with one API over NVLink, InfiniBand, RoCE and Ethernet.
- **ModelExpress:** streams model weights GPU to GPU over NIXL, to speed up cold starts.
- **Grove:** a Kubernetes operator for topology-aware gang scheduling, aimed mainly at NVL72 systems, which extends the scheduling model in [Kubernetes GPU Inference Deployment](../concepts/kubernetes-gpu-inference-deployment.md).
- **AISimulate:** predicts serving behaviour and searches deployment configurations offline.
- **Fault tolerance:** canary health checks and migration of requests already in flight.

NVIDIA's launch blog describes how the router tracks where KV blocks live in a radix structure. It also describes how the planner decides between aggregated and disaggregated serving and moves GPUs between the prefill and decode pools **[sourced]**.

## Vendor claims

All of these figures are vendor-reported, mostly on Blackwell hardware, and best-case, and secondary write-ups quote them inconsistently **[sourced]**:
- **Up to 30x more requests** for DeepSeek-R1, on GB200 NVL72 with TensorRT-LLM, FP4 precision and 32K/8K input/output lengths. It is labelled "projected, subject to change".
- **More than 2x throughput** for Llama 70B on Hopper, with vLLM and FP8.
- **2x faster TTFT** from KV-aware routing on Qwen3-Coder 480B, in a Baseten benchmark.
- **"80% fewer SLA breaches at 5% lower TCO"** for the planner, from an Alibaba talk that gives no baseline.

Never quote a bare multiplier in an interview without the hardware and the baseline.

## When it pays off

Dynamo earns its operational cost for large models, long contexts, bursty or prefix-heavy multi-turn traffic, and multi-node GPU fleets. For a small model on one node it adds complexity for little gain. One secondary source notes that KV routing gains little on short, unique, single-turn traffic **[web, snippet]**. Agents are a strong case, because every agent step resends a growing shared prefix (system prompt, tool schemas, history) that cache-aware routing turns into cache hits.

## Interview Q&A

**What does Dynamo do that Triton with a Kubernetes HPA does not?** It coordinates at cluster level. An HPA scales whole-model replicas on a coarse signal. Dynamo splits prefill from decode, routes by KV-cache locality, tiers KV memory, and scales the prefill and decode pools separately against TTFT and ITL targets **[sourced]**.

**Does Dynamo replace TensorRT-LLM or vLLM?** No. It orchestrates them, and they remain the execution layer **[web]**.

**What happened to Triton?** It was folded into the Dynamo platform and renamed NVIDIA Dynamo-Triton in March 2025 **[sourced]**.

**How would you treat a "30x" claim?** As a projection specific to the hardware, model, precision and sequence length (GB200 NVL72, FP4, 32K/8K), not a general speedup **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Scaling LLMs with NVIDIA Triton and TensorRT-LLM using Kubernetes (NVIDIA blog)](https://developer.nvidia.com/blog/scaling-llms-with-nvidia-triton-and-nvidia-tensorrt-llm-using-kubernetes/) |
| 2026-10-28 | [Dynamo (GitHub)](https://github.com/ai-dynamo/dynamo) |
| 2026-10-28 | [Dynamo docs](https://docs.nvidia.com/dynamo/latest/index.html) |
| 2026-10-28 | [Introducing NVIDIA Dynamo (NVIDIA blog)](https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/) |
