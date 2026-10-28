# NVIDIA Agentic Stack

NVIDIA's agent software is a stack of products, each owning one layer of a request's path, and in the thirteen months between the September 2025 study guide and October 2026 a large share of it was renamed, deprecated or replaced. This page follows one agent request down through the layers, then gives each product's status as of 2026-10-28, so the rest of the wiki can name a product without retelling its history. Every status below is sourced on the linked component or concept page; the layering itself is this wiki's reading of how the products fit together.

## One request, top to bottom

A user's message reaches the agent through a frontend, often over the AG-UI event protocol described in [Agent Interfaces and Oversight UI](../concepts/agent-interfaces-and-oversight.md). NVIDIA has no frontend product of its own beyond the UIs bundled with its blueprints.

The agent itself is a loop of LLM and tool calls. The exam's framework for building that loop is the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md), which wraps LangGraph, CrewAI and others rather than replacing them. Its replacements split the toolkit's jobs three ways: [NeMo Relay](../components/nemo-relay.md) observes and governs each run through middleware, NeMo Fabric is the interface for running agent harnesses, and [NeMo Helix](../components/nemo-helix.md) takes over evaluation and optimisation. Every model call and tool call then passes through policy: [NeMo Guardrails](../components/nemo-guardrails.md) applies input, dialog, retrieval, execution and output rails, calling the classifiers in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md).

Before a model is called, NeMo Switchyard may route the step to a cheaper or stronger model, the lever described in [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md). The call lands on a [NIM](../components/nvidia-nim.md), a container with an OpenAI-compatible API that picks an engine profile for the GPUs it finds. Inside it, the tokens are computed by vLLM, SGLang or [TensorRT-LLM](../components/tensorrt-llm.md). At cluster scale, [NVIDIA Dynamo](../components/nvidia-dynamo.md) sits above those engines, splitting prefill from decode and routing by KV-cache overlap. Non-LLM models, such as vision, speech and embedding ensembles, are still served by [Triton](../components/triton-inference-server.md), now branded Dynamo-Triton.

Under the serving layer is Kubernetes: the GPU Operator exposes the hardware, the [NIM Operator](../components/nim-operator.md) caches models and autoscales NIMs, and [DCGM](../components/dcgm-gpu-telemetry.md) reports GPU health and utilisation. Retrieval runs beside the model path. The NVIDIA RAG Blueprint ingests documents through NV-Ingest (also called the NeMo Retriever Library), embeds them with Nemotron embedding NIMs, and stores them in Milvus or Elasticsearch, using GPU indexes from cuVS, as [RAG Retrieval Pipelines](../concepts/rag-retrieval-pipelines.md) and [Vector Databases and ANN Tuning](../concepts/vector-databases-and-ann-tuning.md) describe.

Finally, the improvement loop runs offline. Evaluation runs on Helix or the [NeMo Evaluator](../components/nemo-evaluator.md) framework. Test data comes from Data Designer, curated data from NeMo Curator, fine-tuning from NeMo Customizer, and reinforcement learning from NeMo RL with NeMo Gym environments; Helix now bundles Data Designer and Customizer beside its evaluation service. The [Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md) wired several of these together, until it was deprecated.

## Status

| Product | Layer | Status | Lineage |
| ------- | ----- | ------ | ------- |
| [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) | Agent framework, eval, profiling | Deprecated October 2026; end of life end of 2026; docs at v1.9 | Was Agent Intelligence Toolkit (AIQ) until v1.2 (2025-06-11); succeeded by Relay, Helix and Fabric |
| [NeMo Relay](../components/nemo-relay.md) | Observability, middleware, governance | Active, pre-1.0 (v0.11.0) | Successor to the toolkit's observability |
| [NeMo Helix](../components/nemo-helix.md) | Evaluation, optimisation, hardening, fine-tuning | Active, pre-1.0 (0.8.1, 2026-10-23; 0.9.0 announced for 2026-11-03) | Successor to `nat eval` and `nat optimize`; absorbs the Evaluator plugin as Helix Evals |
| NeMo Fabric | Agent-harness interface | Named in the migration guide; not yet covered here | Successor to the toolkit's runtime |
| [NeMo Evaluator](../components/nemo-evaluator.md) | Benchmark evaluation | Active (`nel compare`, `nel gate`); relation to Helix Evals undocumented | Older microservice of the same name; Helix's Evaluator plugin renamed Evals |
| NeMo Switchyard | Model routing | Active (blog 2026-08-11) | Replaces the deprecated LLM Router blueprint |
| [NeMo Guardrails](../components/nemo-guardrails.md) | Policy rails | Active (v0.24.1), Apache 2.0 | Repository moved to `NVIDIA-NeMo/Guardrails`; `IORails` engine since 0.21; Relay hooks unreleased |
| [Safety models and garak](../components/nvidia-safety-models-and-garak.md) | Classifiers, red-teaming | Active; model names change often | Current docs list both NemoGuard and Nemotron content-safety names |
| [NIM](../components/nvidia-nim.md) | Model serving | Active (2.1.4) | Moved from TensorRT-LLM-first to vLLM and SGLang, with TensorRT-LLM as one backend among three |
| [TensorRT-LLM](../components/tensorrt-llm.md) | Inference engine | Active (1.4.0rc2) | Moved from `trtllm-build` engines to a PyTorch runtime |
| [Dynamo](../components/nvidia-dynamo.md) | Cluster-scale serving | Active (1.6.1) | Announced at GTC 2025-03-18 |
| [Triton](../components/triton-inference-server.md) | General inference server | Active | Renamed Dynamo-Triton on 2025-03-18 |
| [NIM Operator](../components/nim-operator.md) | Kubernetes management | Active | Added NIMBuild after launch |
| [AI-Q Blueprint](../components/ai-q-blueprint.md) | Reference research agent | Active (2.2.1); pinned to toolkit 1.8.0 | Migration off the toolkit unconfirmed |
| [Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md) | Reference improvement loop | Deprecated April 2026; no successor named | None |
| RAG Blueprint | Reference RAG pipeline | Active (2.6.0, 2026-05-30) | Ingestion component renamed between NV-Ingest and NeMo Retriever Library |

A third-party report that NeMo Microservices sunset on 2026-10-01, and a GTC 2026 "NVIDIA Agent Toolkit" stack, are unconfirmed and not counted here **[web, snippet]**.

## What this means for the exam

The exam was written against the older half of each lineage: the toolkit, Triton by its old name, TensorRT-LLM as the NIM engine, and the Data Flywheel Blueprint as the reference loop. Expect questions in that vocabulary, and answer them on their own terms; the [NCP-AAI Exam Blueprint](exam-blueprint.md) lists the other places where the guide's references have gone stale. In an interview, answer with the current picture and name the transition, because knowing that the toolkit's jobs split three ways, or that NIM now runs on vLLM and SGLang, shows the candidate has used the stack rather than memorised the guide.

The pattern behind the churn is also worth stating. Read across the table, NVIDIA is moving from monolithic toolkits and one-off blueprints to narrow, composable services (one each for observability, evaluation, routing and harnesses) and from proprietary-first engines to upstream open-source engines with NVIDIA optimisations on top.

## Interview Q&A

**What replaced the NeMo Agent Toolkit?** Three products: NeMo Relay for observability and governance, NeMo Helix for evaluation and optimisation, and NeMo Fabric for running agent harnesses **[web]**.

**Is Triton dead?** No. It was renamed Dynamo-Triton in March 2025 and remains the general multi-framework server. Dynamo is a separate layer for cluster-scale LLM serving **[web]**.

**What engine does a NIM use?** One of vLLM, SGLang or TensorRT-LLM, chosen by profile for the GPUs present. Current release notes describe NIM LLM as built on vLLM and SGLang **[web]**.

**Where do Dynamo and NIM differ?** NIM packages one model as a hardware-aware container. Dynamo coordinates many engine workers across nodes, adding disaggregated serving, KV-aware routing and SLA-driven autoscaling **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit migration guide](https://docs.nvidia.com/aiqtoolkit/latest/resources/migration-guide.html) |
| 2026-10-28 | [NeMo Relay docs](https://docs.nvidia.com/nemo/relay/latest/) |
| 2026-10-28 | [NeMo Helix home (v0.8)](https://docs.nvidia.com/nemo-helix/v0.8/documentation/home) |
| 2026-10-28 | [NeMo Helix release 0.8.1 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/release-0-8-1) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
| 2026-10-28 | [Route AI Agent Workloads Across Models with NVIDIA NeMo Switchyard](https://developer.nvidia.com/blog/route-ai-agent-workloads-across-models-with-nvidia-nemo-switchyard) |
| 2026-10-28 | [LLM Router blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/llm-router) |
| 2026-10-28 | [NeMo Guardrails (GitHub)](https://github.com/NVIDIA-NeMo/Guardrails) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
| 2026-10-28 | [NVIDIA TensorRT-LLM (GitHub)](https://github.com/NVIDIA/TensorRT-LLM) |
| 2026-10-28 | [Dynamo docs](https://docs.nvidia.com/dynamo/latest/index.html) |
| 2026-10-28 | [Dynamo-Triton (NVIDIA Developer)](https://developer.nvidia.com/dynamo-triton) |
| 2026-10-28 | [NIM Operator docs](https://docs.nvidia.com/nim-operator/latest/index.html) |
| 2026-10-28 | [AI-Q Blueprint docs (2.2.1)](https://docs.nvidia.com/aiq-blueprint/2.2.1/genindex.html) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
| 2026-10-28 | [RAG Blueprint release notes](https://docs.nvidia.com/rag/latest/release-notes.html) |
| 2026-10-28 | [NeMo RL Documentation](https://docs.nvidia.com/nemo/rl/latest/index.html) |
