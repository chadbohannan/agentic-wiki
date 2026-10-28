# Kubernetes GPU Inference Deployment

Kubernetes is the default platform for running LLM and agent services. It provides scheduling, service discovery, rollouts and health-based restarts, and needs GPU-aware plumbing and LLM-specific autoscaling and load balancing on top. This page covers exam objectives 4.1, 4.4 and 4.5.

NIM containers are managed by the [NVIDIA NIM Operator](../components/nim-operator.md). GPU health and utilisation metrics come from [DCGM GPU Telemetry](../components/dcgm-gpu-telemetry.md). Cluster-level LLM orchestration above the engines is [NVIDIA Dynamo](../components/nvidia-dynamo.md), and model-aware routing is explained in [Disaggregated Serving and KV-Aware Routing](disaggregated-serving-and-kv-routing.md).

## The GPU layer

NVIDIA's Kubernetes glossary explains that containers package a service with its dependencies. Kubernetes then adds scheduling, service discovery, load balancing, rollouts and rollbacks, health-based restarts and secrets, and exposes GPUs through device plug-ins. NVIDIA's GPU Operator, device plug-in and DCGM provide the GPU layer. Multi-Instance GPU (MIG) can split an A100 into up to seven instances for finer-grained packing **[sourced]**. The glossary mentions only the A100, but later data-centre GPUs support MIG too **[inferred]**.

## Autoscaling on the right signal

NVIDIA's October 2024 reference pattern (Triton with TensorRT-LLM on Kubernetes) has these parts **[sourced]**:
- one pod per GPU, or per tensor- × pipeline-parallel group of GPUs for large models;
- a ClusterIP Service;
- a PodMonitor scraping Triton's metrics on port 8002;
- a Prometheus adapter that feeds a custom metric to a HorizontalPodAutoscaler (HPA);
- an L7 load balancer.

The HPA scaled on the ratio of queue time to compute time (target 1000m, 1–4 replicas) rather than on CPU, because GPU utilisation is a poor proxy for LLM saturation.

According to the Kubernetes documentation (read 2026-10-28), the HPA computes `desiredReplicas = ceil(currentReplicas × currentMetric / desiredMetric)` every 15 seconds by default. It accepts resource, per-pod custom, object and external metrics. It ignores changes within a 10% tolerance, and it waits out a 300-second stabilisation window before scaling down **[web]**. The same NVIDIA blog shows the consequences: scale-down took minutes, and cold start is dominated by building the engine and downloading weights. It mitigates cold start by caching engines on the node, and the NIM Operator does the same with its model cache **[sourced]**.

## Load balancing

Generic round-robin or least-request balancing ignores two facts: LLM requests vary enormously in cost, and KV-cache contents and LoRA adapters make backends non-interchangeable. Model-aware pickers, such as the Gateway API Inference Extension's endpoint picker and Dynamo's KV router, use queue depth, KV utilisation and adapter locality instead **[web, CNCF blog; web, Dynamo docs]**. For agents, session affinity helps cache reuse and keeps a multi-call loop on one model version during a rollout **[inferred; canary guidance from a snippet]**.

## An agent stack on Kubernetes

NVIDIA's guide to deploying the AI-Q blueprint on Oracle Cloud (June 2026) shows a typical layout **[web]**:
- a FastAPI backend and a Next.js frontend;
- in-cluster PostgreSQL holding the job store and checkpoints;
- a public load balancer;
- a Helm install into a namespace, with Terraform for the cloud infrastructure and Vault for keys.

By its own account it lacks TLS, high availability, autoscaling and monitoring. That makes it a useful checklist of the gap between a demo and production **[web]**. The [AI-Q Blueprint](../components/ai-q-blueprint.md) page covers the blueprint itself.

## Interview Q&A

**Why not autoscale an LLM on CPU or GPU utilisation?** GPU utilisation saturates early and CPU is irrelevant. NVIDIA scales on the ratio of queue time to compute time, on request concurrency, or on KV-cache usage **[sourced]**.

**Why is scale-up slow for LLMs, and how do you speed it up?** Weights must be downloaded and engines built and loaded. Use model caches (NIMCache), reuse engines on nodes, and keep warm minimum replicas **[sourced, inferred]**.

**What does a model-aware load balancer see that L7 round-robin does not?** Queue depth, KV-cache pressure, adapter locality and request criticality **[web]**.

**What is missing from the AI-Q-on-OCI example for production?** TLS, high availability, autoscaling and monitoring **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Kubernetes glossary (NVIDIA)](https://www.nvidia.com/en-us/glossary/kubernetes/) |
| 2026-10-28 | [Scaling LLMs with NVIDIA Triton and TensorRT-LLM using Kubernetes (NVIDIA blog)](https://developer.nvidia.com/blog/scaling-llms-with-nvidia-triton-and-nvidia-tensorrt-llm-using-kubernetes/) |
| 2026-10-28 | [Horizontal Pod Autoscaling (Kubernetes docs)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) |
| 2026-10-28 | [Deep dive into the Gateway API Inference Extension (CNCF)](https://www.cncf.io/blog/2025/04/21/deep-dive-into-the-gateway-api-inference-extension) |
| 2026-10-28 | [Deploy AI-Q on Oracle Cloud Infrastructure (NVIDIA blog)](https://developer.nvidia.com/blog/deploy-a-production-ready-nvidia-ai-q-blueprint-on-oracle-cloud-infrastructure/) |
