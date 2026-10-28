# Inference Cost and Availability

Serving cost and availability pull against each other. Precision, batching, routing and caching lower the cost per token, while headroom, redundancy and graceful degradation keep the service up. This page covers exam objective 4.5.

Batching and quantisation are explained conceptually in [LLM Inference Optimization](llm-inference-optimization.md). Disaggregation and KV routing are in [Disaggregated Serving and KV-Aware Routing](disaggregated-serving-and-kv-routing.md). The accuracy side of routing to cheaper models is in [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md), and the agent-level fallback behaviour is in [Agent Error Handling and Resilience](agent-error-handling-resilience.md).

## Precision

NVIDIA's DGX Cloud benchmarking post (March 2025) reports that FP8 beats BF16 on Hopper in both tokens per second and cost efficiency. Transformer Engine applies FP8 selectively per layer **[sourced, vendor]**. NVIDIA's NVFP4 blog describes the Blackwell 4-bit format **[web, vendor claims]**:
- **Encoding:** E2M1 values with an FP8 scale per 16-value micro-block plus an FP32 scale per tensor, about 4.5 bits per value.
- **Size:** roughly 3.5x smaller than FP16 and 1.8x smaller than FP8.
- **Accuracy:** at most about 1% loss when DeepSeek-R1-0528 was post-training-quantised from FP8 (MMLU-Pro 85 against 84, AIME24 91 against 89). That is one model in one study.
- **Energy:** the 25x and 50x efficiency figures compare against H100, not against FP8 on the same chip.
- **Tools:** TensorRT Model Optimizer and LLM Compressor produce the quantised models, which deploy on TensorRT-LLM and vLLM.

Re-run your own agent evals after quantising, because tool calling and long-horizon accuracy can degrade differently from benchmark scores **[inferred, echoed by an NVIDIA playbook snippet]**.

## Engine features and caching

In-flight batching and paged attention raise utilisation, as described on [TensorRT-LLM](../components/tensorrt-llm.md) **[sourced]**. Triton's dynamic batching trades queueing delay for throughput **[sourced]**. Speculative decoding helps mainly at low batch sizes **[web, snippet]**. Dynamo's KV block manager and prefix reuse cut recomputation **[web]**.

## Routing across models

Routing sends easy requests to small models. NVIDIA's LLM Router blueprint has two methods **[web, repository README read 2026-10-28]**:
- an intent classifier (Qwen 1.7B) whose intents map to models;
- a neural router trained on CLIP embeddings of text and images.

Its v2 is built on the NeMo Agent Toolkit with a FastAPI endpoint, and returns the chosen model's name rather than proxying the call. The README now marks the blueprint deprecated in favour of NeMo Switchyard, whose reported results (74% cost reduction for about 6 points of accuracy, vendor-reported) are on the tuning page **[web]**. [Decision models](decision-models.md) offer another way to make the routing call cheaply **[web]**.

## Scaling policy and capacity

Scale on queue, KV-cache or latency-target signals rather than CPU, and keep minimum replicas to avoid cold starts. Scale-down is slow by design, as [Kubernetes GPU Inference Deployment](kubernetes-gpu-inference-deployment.md) explains **[sourced]**. Adding GPUs trades time against cost non-linearly. In NVIDIA's DGX Cloud example, training Llama 3 70B on one trillion tokens took 97% less time (from 115.4 to 3.8 days) for 2.6% more cost, thanks to near-linear scaling. That is a training workload, and vendor-reported **[sourced]**.

Spot or preemptible GPUs are cheaper, by up to 70–90% according to a practitioner source. But a preemption can kill a whole model-parallel pipeline. SpotServe (arXiv 2311.15566) keeps its control plane on on-demand capacity and handles re-parallelisation and migration within the preemption grace period **[web, snippet]**.

A defensible pattern has five parts **[inferred]**:
- an on-demand floor sized for the service-level target;
- spot capacity for bursts, batch work and evaluation jobs;
- replicas spread across availability zones, with PodDisruptionBudgets;
- graceful drain, since Dynamo workers drain active requests and deregister, and can migrate in-flight requests;
- fallback to a smaller model or a hosted API when capacity is lost.

## Availability mechanisms

The available mechanisms are:
- health probes, and Dynamo's canary health checks;
- Kubernetes rollouts and rollbacks;
- queue-based load shedding of low-criticality traffic, through the Gateway API Inference Extension's sheddable requests;
- DCGM health policies that act on GPU errors.

They are described on their respective pages **[sourced, web]**.

## Interview Q&A

**FP8 or FP4: how do you decide?** Hardware support first, since NVFP4 needs Blackwell. Then weigh accuracy measured on your own tasks against the memory and throughput gain. Start at FP8 and move to NVFP4 only with eval evidence **[inferred, sourced]**.

**What is the cheapest way to cut agent cost without changing the model?** Prefix and KV caching, and routing easy steps to small models. Both exploit the repetitive structure of agent loops **[inferred]**.

**Would you use spot GPUs for inference?** Only behind a stable on-demand floor, with graceful drain and fast rerouting. Model-parallel pipelines make preemption expensive **[web, inferred]**.

**How do you trade availability against cost?** Headroom and redundancy cost money, so derive minimum replicas and zone spread from the SLO, and shed low-criticality traffic first **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Measure and improve AI workload performance with DGX Cloud Benchmarking (NVIDIA blog)](https://developer.nvidia.com/blog/measure-and-improve-ai-workload-performance-with-nvidia-dgx-cloud-benchmarking/) |
| 2026-10-28 | [Scaling LLMs with NVIDIA Triton and TensorRT-LLM using Kubernetes (NVIDIA blog)](https://developer.nvidia.com/blog/scaling-llms-with-nvidia-triton-and-nvidia-tensorrt-llm-using-kubernetes/) |
| 2026-10-28 | [Introducing NVFP4 (NVIDIA blog)](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/) |
| 2026-10-28 | [LLM Router blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/llm-router) |
| 2026-10-28 | [SpotServe (arXiv 2311.15566)](https://arxiv.org/abs/2311.15566) |
| 2026-10-28 | [Dynamo (GitHub)](https://github.com/ai-dynamo/dynamo) |
