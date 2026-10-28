# AI-Q Blueprint

AI-Q is NVIDIA's open reference blueprint for a long-horizon research agent. An intent router sends each query to a shallow research agent or a deep one, and the whole system is packaged as containers with Docker Compose and Helm deployment paths. It touches exam objective 4.1 as the cleanest NVIDIA example of a multi-agent system packaged for deployment.

Its routed shallow-and-deep hierarchy is an instance of the patterns in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md). It is built on the [NeMo Agent Toolkit](nemo-agent-toolkit.md), whose deprecation makes AI-Q's migration status a live question. Its PostgreSQL checkpoints are the persistence described in [Stateful Orchestration](../concepts/stateful-orchestration.md), and it is deployed with the practices in [Kubernetes GPU Inference Deployment](../concepts/kubernetes-gpu-inference-deployment.md).

## Architecture

NVIDIA's guide to deploying AI-Q on Oracle Cloud (2026-06-26) describes it as an open-source blueprint built on LangChain Deep Agents and the NeMo Agent Toolkit. An intent router sends each query to one of two agents **[web]**:
- a shallow research agent that is bounded, fast and cites its sources;
- a deep agent that plans and dispatches researcher sub-agents.

The deployment has a FastAPI backend, a Next.js frontend, and PostgreSQL for the job store and checkpoints.

A GitHub repository reached while looking for AI-Q is titled "Deep Researcher Agent Blueprint", and its description matches AI-Q **[web; the repository's relationship to the AI-Q name is unconfirmed]**:
- a LangGraph state machine, with concurrent researcher workers and a dedicated writer;
- CLI, web UI, async REST and MCP-server interfaces;
- YAML profiles in `configs/`;
- NeMo Agent Toolkit 1.8.0;
- deployment by local script, Docker Compose or Helm.

By default inference goes to the NVIDIA API Catalog, so no local GPU is needed. Self-hosting the models is a separate sizing exercise. NVIDIA's AI-Q documentation site lists versions from 1.2.1 to 2.2.1, the newest. It includes a Deep Research Bench evaluation, and its 2.0.0 benchmark page names FreshQA, Deep Research Bench and DeepSearchQA **[web, search summary]**. NVIDIA's enterprise reference architecture installs it into `rag` and `aiq` namespaces, with the GPU Operator as a prerequisite **[web, snippet]**.

## Status

The repository page pinned the NeMo Agent Toolkit at 1.8.0. The toolkit was deprecated in October 2026, with NeMo Relay, Helix and Fabric as successors, so check whether AI-Q has migrated before recommending its toolkit-based configuration **[inferred]**. NVIDIA's sibling LLM Router blueprint is now deprecated in favour of NeMo Switchyard, as [Inference Cost and Availability](../concepts/inference-cost-and-availability.md) notes, which shows how quickly these blueprints turn over **[web]**.

## Interview Q&A

**How is AI-Q structured?** An intent router in front of shallow and deep research agents. The deep agent has a planner, parallel researchers and a writer, with a FastAPI backend, a Next.js UI and PostgreSQL for state **[web]**.

**What would you add before calling the OCI deployment production-ready?** TLS, replicas and high availability for the backend and PostgreSQL, autoscaling, monitoring and tracing, and secret rotation. The guide itself lists the first four as missing **[web, inferred]**.

**Does it need local GPUs?** Not by default, because inference goes to the NVIDIA API Catalog. Self-hosted NIMs would need GPUs **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Deploy AI-Q on Oracle Cloud Infrastructure (NVIDIA blog)](https://developer.nvidia.com/blog/deploy-a-production-ready-nvidia-ai-q-blueprint-on-oracle-cloud-infrastructure/) |
| 2026-10-28 | [Deep Researcher Agent Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/deep-researcher-agent) |
| 2026-10-28 | [AI-Q Blueprint docs (2.2.1)](https://docs.nvidia.com/aiq-blueprint/2.2.1/genindex.html) |
| 2026-10-28 | [AI-Q enterprise reference architecture](https://docs.nvidia.com/enterprise-reference-architectures/ai-q-research-agent-blueprint/latest/system-configuration.html) |
| 2026-10-28 | [LLM Router blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/llm-router) |
