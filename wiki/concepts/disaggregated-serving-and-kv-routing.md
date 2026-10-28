# Disaggregated Serving and KV-Aware Routing

LLM inference has two phases with different bottlenecks. Prefill processes the whole prompt in parallel and is compute-bound. Decode generates one token at a time and is limited by memory bandwidth. Disaggregated serving runs the two phases on separate GPU pools, and KV-aware routing sends each request to the worker that already holds most of its prompt in cache. Agent traffic benefits more than most, because every step resends a long shared prefix.

## Disaggregation

When both phases share GPUs, a long prefill stalls decodes already in progress, which shows up as inter-token latency spikes. The design document of [NVIDIA Dynamo](../components/nvidia-dynamo.md), NVIDIA's implementation (v1.0.0), runs the phases on separate engines, so each can use its own tensor-parallel size and hardware. Its example gives decode a larger tensor-parallel size than prefill **[sourced]**.

A request moves through the system in three steps **[sourced]**:
1. A PrefillRouter picks a prefill worker, either by KV overlap or by load.
2. The prefill worker computes the KV cache and returns transfer metadata.
3. The request is routed to a decode worker, which pulls the cache over NIXL (directly GPU to GPU, over NVLink or InfiniBand, without blocking) and generates the tokens.

Engines differ here. SGLang can start decode while prefill is still running, whereas vLLM and [TensorRT-LLM](../components/tensorrt-llm.md) prefill synchronously and decode waits. The ratio of prefill to decode workers ("xPyD") can be changed at runtime: new workers register and publish their KV capacity, and departing workers drain. The document gives no performance numbers and does not cover deciding per request whether to disaggregate.

## KV-aware routing

Dynamo's router guide (v1.0.0) gives the routing cost as `cost = overlap_score_weight × prefill_blocks + decode_blocks`, where prefill blocks are the uncached part of the prompt and decode blocks measure load from active sequences. The router picks the lowest-cost worker **[web]**. Three settings control it **[web]**:
- **`--router-kv-overlap-score-weight`** (default 1.0) trades TTFT against ITL. Higher favours cache reuse and lower TTFT, lower spreads decode load for better ITL, and 0 ignores the cache entirely. It can be overridden per request through `nvext.agent_hints`.
- **`--router-temperature`** (default 0) is deterministic at 0. Above zero it samples workers with a softmax over cost, which spreads load.
- **`--no-router-kv-events`** makes the router predict cache state from its own routing decisions instead of engine events, expiring entries after a TTL (`--router-ttl-secs`, default 120 s).

Kubernetes has the same idea in the Gateway API Inference Extension, part of the model-aware load balancing in [Kubernetes GPU Inference Deployment](kubernetes-gpu-inference-deployment.md). Its endpoint picker reads queue depth and KV-cache utilisation per pod, and accounts for request criticality and LoRA-adapter affinity. In the version a CNCF blog described (April 2025), the defaults were:
- **Critical requests** went to pods with fewer than 50 queued requests and the right adapter loaded.
- **Sheddable requests** needed KV utilisation under 80% and fewer than 5 queued requests, or they were dropped.

Those thresholds and the API (`InferenceModel`, `InferencePool`) have since changed, and an `InferenceObjective` resource now exists **[web, snippet]**.

## Trade-offs and agent relevance

Disaggregation adds a KV transfer and a more complex failure domain, so its place among the other cost levers is a judgement weighed in [Inference Cost and Availability](inference-cost-and-availability.md). It wins when prompts are long and decode latency targets are tight, and loses for short prompts or slow interconnects. Prefix-aware routing can create hot spots, which is why it has the temperature and load terms, and it depends on accurate cache-state events. Multi-step agents share long prefixes across steps and across parallel sub-agents, so cache-aware routing and KV offload turn repeated prefill into cache hits. Per-step TTFT, multiplied by the number of steps, dominates how fast an agent feels.

## Interview Q&A

**Why separate prefill and decode?** They have different bottlenecks (compute versus memory bandwidth). Separate pools allow different parallelism and stop long prefills blocking decodes **[sourced]**.

**What is transferred between the pools, and how?** The KV cache, GPU to GPU via NIXL over NVLink or InfiniBand **[sourced]**.

**When would disaggregation not help?** With short prompts, a slow interconnect or low load, where transfer and orchestration overhead exceed the interference removed.

**How does KV-aware routing differ from least-connections?** It weighs the uncached part of each prompt against worker load, so it prefers the worker that can reuse the most cached prefix rather than simply the least busy one **[web]**.

**Which signals would you autoscale on?** TTFT and ITL target attainment, queue depth and KV-cache utilisation, not CPU **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Dynamo disaggregated serving design (v1.0.0)](https://docs.nvidia.com/dynamo/v1.0.0/design-docs/disaggregated-serving) |
| 2026-10-28 | [Dynamo KV cache-aware routing guide (v1.0.0)](https://docs.nvidia.com/dynamo/v1.0.0/user-guides/kv-cache-aware-routing) |
| 2026-10-28 | [Deep dive into the Gateway API Inference Extension (CNCF)](https://www.cncf.io/blog/2025/04/21/deep-dive-into-the-gateway-api-inference-extension) |
