# LLM Inference Optimization

LLM serving is limited less by arithmetic than by memory: generating each token means rereading the weights and a growing KV cache. This page explains why that bottleneck arises and the techniques that address it: batching, paged KV memory, attention variants, quantisation, sparsity, distillation, speculative decoding, parallelism and disaggregation. Most of it follows NVIDIA's "Mastering LLM Techniques: Inference Optimization" blog (November 2023), which the study guide cites. It predates FP4 and Blackwell, so newer material is added where noted. On NVIDIA hardware nearly all of these techniques are implemented in one engine, [TensorRT-LLM](../components/tensorrt-llm.md), which is also one of the backends behind [NVIDIA NIM](../components/nvidia-nim.md).

## Why decode is memory-bound

Prefill processes all the input tokens together as a highly parallel matrix–matrix operation that saturates the GPU. Decode generates one token per step as a matrix–vector operation that leaves compute idle while the GPU waits on memory transfers **[sourced]**. Prefill therefore drives time to first token, and decode drives inter-token latency.

The KV cache per token is 2 × layers × hidden size × bytes per value, where the 2 covers keys and values. For Llama 2 7B in FP16 (32 layers, hidden size 4096) at batch size 1 and 4,096 tokens, that is about 2 GB, against about 14 GB for the weights. The article notes that this memory grows linearly with batch size and sequence length, which limits the throughput that can be served and makes long contexts hard **[sourced]**.

## Techniques

The blog's techniques fall into four groups **[sourced]**.

**Attention and KV memory**
- **Multi-query attention (MQA)** has all query heads share one key/value head. It reads and stores far less KV data, but may cost accuracy and needs training with MQA.
- **Grouped-query attention (GQA)** shares one key/value head per group of query heads, sitting between full multi-head attention and MQA. An existing model can be uptrained to it with a fraction of the original compute. Llama 2 70B uses GQA.
- **FlashAttention** computes exact attention in tiles to cut reads and writes to GPU memory. It is mathematically identical to standard attention.
- **PagedAttention** stores each request's KV cache in fixed-size blocks located through a block table. It allocates blocks as tokens are generated, instead of reserving, say, 2,048 tokens up front.

**Batching**
- **In-flight (continuous) batching** evicts finished sequences and admits new requests at every step. Static batching waits for the longest request.

**Model compression**
- **Quantisation** lowers the precision of weights and activations, and most models survive 8 bits or fewer. Weights are easy to quantise because they are fixed. Activations contain outliers that widen their range, which is why LLM.int8() keeps some at higher precision.
- **2:4 structured sparsity** zeroes two of every four values. GPUs accelerate the pattern in hardware, and it combines with quantisation.
- **Distillation** trains a smaller student on a teacher's outputs. DistilBERT is 40% smaller than BERT, retains 97% of its language understanding, and runs 60% faster.

**Speculative decoding and parallelism**
- **Speculative decoding** has a cheap drafter propose several tokens, which the main model verifies in parallel. Everything from the first mismatch onwards is discarded.
- **Pipeline parallelism** splits a model's layers across devices and suffers idle "bubbles" that micro-batching reduces.
- **Tensor parallelism** splits weight matrices and attention heads within a layer.
- **Sequence parallelism** splits LayerNorm and Dropout along the sequence dimension.
- **Data parallelism** is mainly a training technique.

## Newer layers

Since that blog, the following have arrived:
- **Low-precision formats:** FP8 on Hopper, and NVFP4 on Blackwell, which is described in [Inference Cost and Availability](inference-cost-and-availability.md) **[web]**.
- **Disaggregated serving**, which runs prefill and decode on separate GPU pools. It is in beta in TensorRT-LLM and central to [NVIDIA Dynamo](../components/nvidia-dynamo.md), as [Disaggregated Serving and KV-Aware Routing](disaggregated-serving-and-kv-routing.md) explains **[web]**.
- **Chunked prefill**, which interleaves pieces of a long prompt with ongoing decodes **[web]**.

At the engine level, NVIDIA's TensorRT best practices still apply: measure a baseline first, with the tools in [LLM Inference Profiling](llm-inference-profiling.md), then use layer fusion, reduced precision, CUDA graphs, multiple streams and timing caches. At the serving layer, the dynamic batcher in [Triton Inference Server (Dynamo-Triton)](../components/triton-inference-server.md) trades a short queueing delay for larger batches **[sourced]**.

The study guide also cites NVIDIA's training performance guide, but that is a separate topic. The guide is now titled for Megatron Bridge rather than NeMo Framework. It recommends a parallelism ladder of data, then tensor, then context, then pipeline parallelism. It reports a 1.2–1.5x gain from FP8 in training, and advises overlapping communication and recomputing activations. The study guide's data-curation best practices cover quality-scoring of training data **[sourced]**.

## Trade-offs

Every lever trades one metric for another:
- **Bigger batches** raise throughput but also each request's latency.
- **Quantisation** saves memory and bandwidth but needs accuracy checks on your own tasks.
- **Speculative decoding** helps latency at low load but wastes compute when acceptance is low.
- **More parallelism** fits bigger models at the cost of communication.

## Interview Q&A

**Why is decode memory-bound?** It produces one token at a time as a matrix–vector operation, rereading the weights and KV cache at every step with little compute reuse **[sourced]**.

**How big is the KV cache?** 2 × layers × hidden size × bytes per token. That is about 2 GB for Llama 2 7B in FP16 at 4,096 tokens **[sourced]**.

**How do you cut TTFT, and how do you raise throughput?** For TTFT: smaller batches, chunked prefill or disaggregation. For throughput: bigger batches, in-flight batching and quantisation.

**What does GQA buy you?** Fewer key/value heads shrink the KV cache and memory reads, with little accuracy loss **[sourced]**.

**What is the risk of speculative decoding?** A low acceptance rate wastes compute, so it needs a well-matched drafter.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Mastering LLM techniques: inference optimization (NVIDIA blog)](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/) |
| 2026-10-28 | [TensorRT best practices](https://docs.nvidia.com/deeplearning/tensorrt/latest/performance/best-practices.html) |
| 2026-10-28 | [Triton batchers](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html) |
| 2026-10-28 | [Performance tuning guide (Megatron Bridge)](https://docs.nvidia.com/nemo-framework/user-guide/latest/performance/performance-guide.html) |
| 2026-10-28 | [Data curation best practices (NeMo Framework)](https://docs.nvidia.com/nemo-framework/user-guide/latest/datacuration/bestpractices.html) |
| 2026-10-28 | [TensorRT-LLM overview](https://nvidia.github.io/TensorRT-LLM/latest/overview.html) |
| 2026-10-28 | [Dynamo-Triton (NVIDIA Developer)](https://developer.nvidia.com/dynamo-triton) |
