# NVIDIA NIM Operator

The NIM Operator is a Kubernetes operator for deploying and managing NIM microservices, including LLM, retrieval, speech and biology models, through custom resources. It handles model caching, autoscaling and rolling upgrades. It touches exam objectives 4.1 and 4.4.

The operator deploys the containers described in [NVIDIA NIM](nvidia-nim.md), and is one of the building blocks of [Kubernetes GPU Inference Deployment](../concepts/kubernetes-gpu-inference-deployment.md).

## Custom resources

The operator's documentation lists four custom resources **[web, docs landing page]**:
- **NIMCache** downloads models from NGC onto shared network storage, so all replicas reuse one copy. It also supports air-gapped installs.
- **NIMService** runs one NIM as a Deployment, backed by a NIMCache or a PVC. It can create an HPA, an ingress and a ServiceMonitor.
- **NIMPipeline** groups NIMServices, for example the embedding, reranking and LLM services of a RAG pipeline.
- **NIMBuild** pre-builds optimised TensorRT-LLM engines for each GPU profile, to cut startup time.

The launch blog (2024-09-30) listed only NIMService, NIMPipeline and NIMCache, so NIMBuild came later. The docs page shows no version number.

## Autoscaling and upgrades

According to the launch blog, autoscaling uses a standard HPA with minimum and maximum replicas, scaling behaviour and stabilisation windows. It can be driven by several kinds of metric **[web]**:
- per-pod resource or custom metrics, such as GPU memory;
- object metrics, such as maximum requests or KV-cache use;
- external metrics.

Upgrades are made by changing the NIM version in the resource, which the operator applies as a rolling update **[web]**.

Later operator docs add four constraints, seen only in search snippets **[web, snippet]**:
- CPU and memory metrics are not useful for NIM.
- Metric names come from the backend and differ between vLLM and SGLang, so check `/v1/metrics`.
- Autoscaling cannot be combined with a fixed `spec.replicas` or with multi-node deployments.
- A KServe integration exists.

## Interview Q&A

**What does NIMCache buy you?** Faster scale-out and cold start, shared model storage, and air-gapped operation **[web]**.

**NIMService versus NIMPipeline?** A single microservice versus a managed group, such as embedding, reranker and LLM **[web]**.

**How do you roll out a new NIM version?** Change the version in the resource, and the operator performs a rolling update **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NIM Operator docs](https://docs.nvidia.com/nim-operator/latest/index.html) |
| 2026-10-28 | [Managing AI inference pipelines on Kubernetes with NVIDIA NIM Operator (NVIDIA blog)](https://developer.nvidia.com/blog/managing-ai-inference-pipelines-on-kubernetes-with-nvidia-nim-operator) |
| 2026-10-28 | [NIM Operator KServe integration (3.0.0)](https://docs.nvidia.com/nim-operator/3.0.0/kserve.html) |
