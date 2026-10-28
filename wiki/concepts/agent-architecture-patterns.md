# Agent Architecture Patterns

An LLM agent is a loop: perceive, reason, act through tools, observe, and repeat until a stop condition is met. This page covers the main architectural choices for building one (workflow versus agent, reactive versus deliberative, and how to keep the design adaptable) and ties them to NVIDIA's reference designs. It corresponds to exam objectives 1.2, 1.6 and 1.8.

The reactive loop is made concrete in [ReAct: Reasoning and Acting](react-reasoning-action.md), and the plan-first variants it contrasts with are treated in [LLM Agent Planning](llm-agent-planning.md). Once a design needs more than one agent, [Multi-Agent Orchestration](multi-agent-orchestration.md) covers the topologies, and durable state across steps is the subject of [Stateful Orchestration](stateful-orchestration.md).

## Workflows versus agents

Anthropic's "Building Effective Agents" draws the most useful line in the field: workflows are systems where LLMs and tools are orchestrated through predefined code paths, while agents are systems where the LLM directs its own process and tool use **[web]**. Its advice is to find the simplest solution possible and add complexity only when needed, because agents trade cost and latency for flexibility and can compound errors, and because framework abstractions can obscure prompts and responses and make debugging harder. Read in full on 2026-10-28.

Anthropic names five workflow patterns. Prompt chaining splits a task into sequential calls and trades latency for accuracy. Routing classifies an input and sends it to a specialised path, for example easy questions to a cheaper model; since late 2026 that classification step can be done by a [decision model](decision-models.md) that returns calibrated probabilities instead of text. Parallelization either splits independent subtasks (sectioning) or runs the same task several times and votes. Orchestrator-workers lets a central LLM decompose a task dynamically and delegate to workers, and evaluator-optimizer runs a generate-and-critique loop **[web]**. A good interview answer treats these as the vocabulary for deciding where on the workflow-to-agent spectrum a given problem sits.

## Reactive, deliberative and hybrid designs

The classical taxonomy distinguishes reactive agents (tight perception-action coupling, little internal state, fast), deliberative agents (an explicit world model and plan, slower) and hybrids that put a fast reactive layer under a deliberative planner, with BDI as the classical reference. A 2026 arXiv paper maps this taxonomy onto LLM systems **[web]**, but only its search snippet was read, so treat the mapping as a lead. The hard design problem is the rule for escalating from reactive to deliberative handling, and the sources found for that are practitioner blogs and weak.

Mapping the taxonomy onto current tooling is our own synthesis **[inferred]**. A single tool-calling or ReAct step loop is closest to reactive. Plan-then-execute designs such as ReWOO, with a structured plan, workers and a writer, are deliberative. The [AI-Q blueprint](../components/ai-q-blueprint.md) is a hybrid, since its intent classifier sends a query down a shallow or a deep research path, as the AI-Q documentation describes **[web, snippet]**.

## Adaptability and scale

NVIDIA's AI Factory white paper frames the shift as moving from static pipelines to autonomous, stateful workflows, with graph control planes and dynamic routing of model, depth and data source per request **[sourced]**. For adaptability (objective 1.8) it recommends treating agents as versioned services with rollback, using dynamic routing to limit expensive frontier-model calls while still permitting multi-hop depth, and deploying on Kubernetes with optional schedulers (KAI, Kueue, Volcano). It also names a capability-versus-isolation trade-off: more capable agents need looser sandbox limits **[sourced]**. NVIDIA's multi-agent glossary adds that agents should be addable or replaceable without rebuilding the system **[sourced]**.

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ships built-in agent types that line up with these patterns: ReAct, Reasoning, ReWOO, Router, Tool Calling and Responses API agents, plus Parallel and Sequential executors **[web]**. The toolkit is deprecated as of October 2026, so the pattern vocabulary is more durable than the product.

## Interview Q&A

**Workflow or agent?** Use a workflow when the steps are known: it is more predictable, cheaper and easier to test. Use an agent when the path or step count cannot be predicted, and accept the cost and error compounding **[web]**.

**What is a hybrid architecture?** A fast reactive path for routine requests plus a deliberative planning layer for complex ones, joined by an explicit escalation rule **[inferred]**.

**How does an NVIDIA blueprint reflect this?** AI-Q classifies intent and then follows a shallow or deep research path **[web, snippet]**.

**How do you scale an agent system?** Keep services stateless behind routers or gateways, externalise state, route per request to the right model, and autoscale on Kubernetes **[inferred, grounded in the AI Factory paper]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) |
| 2026-10-27 | [Agentic AI in the Factory (NVIDIA AI Factory white paper)](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html) |
| 2026-10-27 | [What Are Multi-Agent Systems? (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/multi-agent-systems/) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
| 2026-10-27 | [AI-Q Blueprint documentation](https://docs.nvidia.com/aiq-blueprint/2.1.0/index.html) |
| 2026-10-27 | [arXiv 2602.10479](https://arxiv.org/pdf/2602.10479) |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
