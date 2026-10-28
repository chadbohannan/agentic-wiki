# AI-Q Blueprint

AI-Q is NVIDIA's open reference blueprint for a long-horizon research agent. An intent router sends each query to a shallow research agent or a deep one, and the whole system is packaged as containers with Docker Compose and Helm deployment paths. It is the cleanest NVIDIA example of a multi-agent system packaged for deployment.

## Architecture

NVIDIA's guide to deploying AI-Q on Oracle Cloud (2026-06-26) describes it as an open-source blueprint built on LangChain Deep Agents and the [NeMo Agent Toolkit](nemo-agent-toolkit.md). An intent router sends each query to one of two agents, a routed supervisor hierarchy of the kind described in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md) **[web]**:
- a shallow research agent that is bounded, fast and cites its sources;
- a deep agent that plans and dispatches researcher sub-agents.

The AI-Q documentation describes the router as an intent classifier that is the entry node for every query. In one LLM call it labels the intent `meta` (greetings, chit-chat, capability questions), which it answers directly, or `research`, for which it returns a depth decision of `shallow` for quick lookups or `deep` for comprehensive research; deep queries pass through a clarifier agent first **[web]**. The routing is an LLM judgment with no fixed criteria, so its accuracy is worth evaluating like any other classifier.

The deployment has a FastAPI backend, a Next.js frontend, and PostgreSQL for the job store and the checkpoints that let a long research run survive restarts, the persistence model in [Stateful Orchestration](../concepts/stateful-orchestration.md).

A GitHub repository titled "Deep Researcher Agent Blueprint" matches AI-Q's description, though whether it is the same project under another name is unconfirmed **[web]**:
- a LangGraph state machine, with concurrent researcher workers and a dedicated writer;
- CLI, web UI, async REST and MCP-server interfaces;
- YAML profiles in `configs/`;
- NeMo Agent Toolkit 1.8.0;
- deployment by local script, Docker Compose or Helm.

By default inference goes to the NVIDIA API Catalog, so no local GPU is needed. Self-hosting the models is a separate sizing exercise. NVIDIA's AI-Q documentation site lists versions from 1.2.1 to 2.2.1, the newest. It includes a Deep Research Bench evaluation, and its 2.0.0 benchmark page names FreshQA, Deep Research Bench and DeepSearchQA. NVIDIA's enterprise reference architecture installs it into `rag` and `aiq` namespaces, with the GPU Operator as a prerequisite, so the Helm path inherits the scheduling and autoscaling concerns in [Kubernetes GPU Inference Deployment](../concepts/kubernetes-gpu-inference-deployment.md) **[web, snippet]**.

## Status

The repository page pinned the NeMo Agent Toolkit at 1.8.0. The toolkit has since been deprecated, so check whether AI-Q has migrated before recommending its toolkit-based configuration. NVIDIA's sibling LLM Router blueprint is now deprecated in favour of NeMo Switchyard, as [Inference Cost and Availability](../concepts/inference-cost-and-availability.md) notes, which shows how quickly these blueprints turn over **[web]**. Because the blueprint depends on so many moving products, [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md) is where their current status and successors are tracked.

## Interview Q&A

**How is AI-Q structured?** An intent router in front of shallow and deep research agents. The deep agent has a planner, parallel researchers and a writer, with a FastAPI backend, a Next.js UI and PostgreSQL for state **[web]**.

**What would you add before calling the OCI deployment production-ready?** TLS, replicas and high availability for the backend and PostgreSQL, autoscaling, monitoring and tracing, and secret rotation. The guide itself lists the first four as missing **[web]**.

**Does it need local GPUs?** Not by default, because inference goes to the NVIDIA API Catalog. Self-hosted NIMs would need GPUs **[web]**.

**Why does a research agent need a database for checkpoints?** A deep research run takes many steps and minutes, so a restart would otherwise lose all of it. Checkpointing graph state to PostgreSQL lets the run resume where it stopped, and the job store lets clients poll an asynchronous job **[web]**.

**What is the risk in adopting AI-Q today?** Its configuration is built on the NeMo Agent Toolkit, which is deprecated, and its sibling router blueprint has already been replaced. Treat it as a reference architecture to learn from, and check what it depends on before building on it.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Deploy AI-Q on Oracle Cloud Infrastructure (NVIDIA blog)](https://developer.nvidia.com/blog/deploy-a-production-ready-nvidia-ai-q-blueprint-on-oracle-cloud-infrastructure/) |
| 2026-10-28 | [Deep Researcher Agent Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/deep-researcher-agent) |
| 2026-10-28 | [AI-Q Blueprint docs (2.2.1)](https://docs.nvidia.com/aiq-blueprint/2.2.1/genindex.html) |
| 2026-10-28 | [AI-Q enterprise reference architecture](https://docs.nvidia.com/enterprise-reference-architectures/ai-q-research-agent-blueprint/latest/system-configuration.html) |
| 2026-10-28 | [LLM Router blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/llm-router) |
| 2026-10-28 | [AI-Q intent classifier docs](https://docs.nvidia.com/aiq-blueprint/latest/architecture/agents/intent-classifier.html) |
