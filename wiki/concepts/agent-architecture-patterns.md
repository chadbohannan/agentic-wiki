# Agent Architecture Patterns

An LLM agent is a loop: perceive, reason, act through tools, observe, and repeat until a stop condition is met. The architectural choices are where on the workflow-to-agent spectrum to sit, how much to plan before acting, and how to keep the design adaptable as models and tools change. Each of those choices moves control flow between the model and the harness that runs it, one of the trade-offs between the parts of an agent laid out in [Anatomy of an Agent](../syntheses/agent-anatomy.md).

## Workflows versus agents

Anthropic's "Building Effective Agents" draws the most useful line in the field: workflows are systems where LLMs and tools are orchestrated through predefined code paths, while agents are systems where the LLM directs its own process and tool use **[web]**. Its advice is to find the simplest solution possible and add complexity only when needed, because agents trade cost and latency for flexibility and can compound errors, and because framework abstractions can obscure prompts and responses and make debugging harder.

Anthropic names five workflow patterns. Prompt chaining splits a task into sequential calls and trades latency for accuracy. Routing classifies an input and sends it to a specialised path, for example easy questions to a cheaper model; since late 2026 that classification step can be done by a [decision model](decision-models.md) that returns calibrated probabilities instead of text. Parallelization either splits independent subtasks (sectioning) or runs the same task several times and votes. Orchestrator-workers lets a central LLM decompose a task dynamically and delegate to workers, the seed of the supervisor topology in [Multi-Agent Orchestration](multi-agent-orchestration.md), and evaluator-optimizer runs a generate-and-critique loop **[web]**. A good interview answer treats these as the vocabulary for deciding where on the workflow-to-agent spectrum a given problem sits. The fixed patterns, often called static chains, are predictable and debuggable but brittle against unanticipated inputs, and their programmatic gates are far easier to write when each step returns schema-checked data, as in [Structured Outputs](structured-outputs.md). Dynamic flows, which choose the next step, template or model at run time, handle open-ended tasks at higher cost and with more loop risk **[web]**.

## Reactive, deliberative and hybrid designs

The classical taxonomy distinguishes reactive agents (tight perception-action coupling, little internal state, fast), deliberative agents (an explicit world model and plan, slower) and hybrids that put a fast reactive layer under a deliberative planner, with BDI as the classical reference. A 2026 arXiv paper maps this taxonomy onto LLM systems **[web, snippet]**. The hard design problem is the rule for escalating from reactive to deliberative handling, and the available guidance on it is practitioner blogs.

A single tool-calling or [ReAct](react-reasoning-action.md) step loop is closest to reactive. Plan-then-execute designs such as ReWOO, with a structured plan, workers and a writer, are deliberative, and the planning techniques behind them are in [LLM Agent Planning](llm-agent-planning.md). The [AI-Q blueprint](../components/ai-q-blueprint.md) is a hybrid, since its intent classifier makes one LLM call that answers greetings and capability questions directly and sends research queries down a shallow or a deep path, as the AI-Q documentation describes **[web]**.

## Adaptability and scale

NVIDIA's AI Factory white paper frames the shift as moving from static pipelines to autonomous, stateful workflows, with graph control planes of the kind described in [Stateful Orchestration](stateful-orchestration.md) and dynamic routing of model, depth and data source per request **[sourced]**. For adaptability it recommends treating agents as versioned services with rollback, using dynamic routing to limit expensive frontier-model calls while still permitting multi-hop depth, and deploying on Kubernetes with optional schedulers (KAI, Kueue, Volcano). It also names a capability-versus-isolation trade-off: more capable agents need looser sandbox limits. NVIDIA's multi-agent glossary adds that agents should be addable or replaceable without rebuilding the system **[sourced]**.

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ships built-in agent types that line up with these patterns: ReAct, Reasoning, ReWOO, Router, Tool Calling and Responses API agents, plus Parallel and Sequential executors **[web]**. The toolkit is now deprecated, so the pattern vocabulary is more durable than the product.

## Interview Q&A

**Workflow or agent?** Use a workflow when the steps are known: it is more predictable, cheaper and easier to test. Use an agent when the path or step count cannot be predicted, and accept the cost and error compounding **[web]**.

**What is a hybrid architecture?** A fast reactive path for routine requests plus a deliberative planning layer for complex ones, joined by an explicit escalation rule.

**How does an NVIDIA blueprint reflect this?** AI-Q's intent classifier, a single LLM call, labels each query as meta or research and picks a shallow or deep path for research, so a reactive front end routes into a deliberative planner only when the query needs one **[web]**.

**How do you scale an agent system?** Keep services stateless behind routers or gateways, externalise state, route per request to the right model, and autoscale on Kubernetes.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) |
| 2026-10-27 | [Agentic AI in the Factory (NVIDIA AI Factory white paper)](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html) |
| 2026-10-27 | [What Are Multi-Agent Systems? (NVIDIA glossary)](https://www.nvidia.com/en-us/glossary/multi-agent-systems/) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
| 2026-10-28 | [AI-Q intent classifier docs](https://docs.nvidia.com/aiq-blueprint/latest/architecture/agents/intent-classifier.html) |
| 2026-10-27 | [arXiv 2602.10479](https://arxiv.org/pdf/2602.10479) |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
