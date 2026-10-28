# Anatomy of an Agent

An LLM agent is a model running in a loop, choosing tools to call and reading their results until a task is done. That one sentence hides seven parts, and almost every design question in agent engineering is a question about one of them or about where a responsibility sits between them. This page names the parts, says what each is for and how it fails, and points to the page that treats it in depth. It is the conceptual spine of the wiki: the other concept pages hang off one of the parts below.

## The seven parts

Anthropic's "Building Effective Agents" calls the basic building block an "augmented LLM": a model given retrieval, tools and memory. It describes agents as "LLMs using tools based on environmental feedback in a loop" **[web]**. Unpacking that description gives seven parts.

1. **The model** chooses what to do next.
2. **The context** is everything the model sees when it chooses.
3. **The tools** are how its choices act on the world.
4. **The loop**, run by a harness, carries results back and decides when to stop.
5. **Memory and state** persist beyond a single context window.
6. **The environment** is where actions land, and what limits them.
7. **Control and observation** are the budgets, policies, people and traces that keep the system safe and fixable.

### Model

The model is the policy: given the context, it emits either a tool call or an answer. Its capability sets the ceiling for everything else, and it is also the largest share of cost and latency, because it runs once per step. The design choices are which model to use for which step, whether to use a reasoning model that thinks before each call, and how to serve it fast enough that a twenty-step task still feels responsive. [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md) covers routing steps to cheaper or stronger models, and [LLM Inference Optimization](../concepts/llm-inference-optimization.md) covers making each call cheaper. Because every step resends a long shared prefix, serving with prefix caching and cache-aware routing matters more for agents than for chat, as [Disaggregated Serving and KV-Aware Routing](../concepts/disaggregated-serving-and-kv-routing.md) explains.

### Context

The context is the model's entire view of the world at the moment it decides: the system prompt, tool definitions, examples, history, retrieved data and tool results. The model is fixed at run time, so context is the main lever the builder controls on every step, and [Context Engineering](../concepts/context-engineering.md) is the discipline of spending it well. It fails gradually: recall falls as the window fills and as relevant facts drift into the middle, and a single wrong observation can poison every later step. Writing the instructions, the older craft of prompt engineering, is one part of that discipline, and making the model's output machine-checkable is covered in [Structured Outputs](../concepts/structured-outputs.md).

### Tools

Tools are the agent's actuators and its senses: a search tool is how it sees, and a write tool is how it acts. A tool is an interface designed for a model, not a human or a program, and the SWE-agent paper made the case that this interface ("agent-computer interface") is worth as much design effort as the model's prompt. Its custom interface for editing and navigating code took a language-model agent to 12.5% pass@1 on SWE-bench, which its authors reported as state of the art at the time **[web]**. The mechanics, design rules and the MCP standard are in [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md), and the protocols that connect agents to tools, to each other and to user interfaces are in [Agent Protocols: MCP, A2A, AG-UI](../concepts/agent-protocols-mcp-a2a.md). Tools are also the attack surface: every tool is a capability that an injected instruction can borrow.

### Loop and harness

The loop is the control flow: call the model, execute its tool calls, append the results, repeat until it stops. [ReAct and the Agent Loop](../concepts/react-reasoning-action.md) describes the loop and how it fails. The harness is the code that runs it, and it carries most of the engineering: context management, budgets, retries, permissions, streaming and tracing. How much control flow lives in the harness rather than in the model is the workflow-versus-agent decision in [Agent Architecture Patterns](../concepts/agent-architecture-patterns.md). At one end, code fixes the steps and the model fills them in; at the other, the model chooses every step. Planning up front, as in [LLM Agent Planning](../concepts/llm-agent-planning.md), and splitting work across several loops, as in [Multi-Agent Orchestration](../concepts/multi-agent-orchestration.md), are both ways of adding structure to the loop.

### Memory and state

The model is stateless, so whatever the agent should remember must be stored outside the window and selected back in. Two different things go by the name. Run state is where the loop has got to: its messages, plan and pending approvals, checkpointed so a run can pause and resume, as in [Stateful Orchestration](../concepts/stateful-orchestration.md). Memory is what the agent learns across runs: facts, preferences, past episodes and skills, as in [Agent Memory](../concepts/agent-memory.md). Retrieval over documents is a third kind of store, described in [RAG Retrieval Pipelines](../concepts/rag-retrieval-pipelines.md). Increasingly, the simplest store is the file system itself: Anthropic's long-running coding agents hand off work through a progress file, a feature list and git history **[web]**.

### Environment

The environment is where tool calls take effect: a shell, a browser, a database, a set of APIs. It is the source of the "ground truth" that keeps the loop honest, and it is also where damage happens, so it needs boundaries. Anthropic's sandboxing for Claude Code is a worked example: file-system isolation limits what the agent can read and change, network isolation limits where it can send data, and both are needed, since without the second a compromised agent can exfiltrate keys and without the first it can escape the sandbox. Anthropic reports that the sandbox cut permission prompts by 84% in internal use, because actions inside the boundary no longer need approval **[web]**. The threats a sandbox defends against are in [Agent Security Threats and Controls](../concepts/agent-security-threats-and-controls.md).

### Control and observation

Control decides what the agent may do. It includes step and cost budgets, guardrails on inputs, outputs and tool calls, and points where a person must approve, in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md) and [Human-in-the-Loop Patterns](../concepts/human-in-the-loop-patterns.md). Observation makes the run visible afterwards: a trace of every model call and tool call, described in [Agent Observability and Tracing](../concepts/agent-observability-tracing.md). Evaluation turns those traces into scores against a test set, as in [Agent Evaluation Pipelines](../concepts/agent-evaluation-pipelines.md), and that is what lets the other six parts be improved without guessing.

## Where a responsibility should live

Most design decisions move a responsibility from one part to another, and the parts trade off in predictable ways.

- **Prompt, tool or code.** A rule can be an instruction in the prompt, a constraint in a tool's arguments, or a check in the harness. Anthropic's example is a coding agent that kept getting relative paths wrong until the tool was changed to require absolute paths, after which the model "used this method flawlessly" **[web]**. Moving a rule from prompt to tool to code makes it more reliable and less flexible. Safety-critical rules belong in code.
- **Context or memory.** Keeping information in the window is simple and makes it immediately visible to the model, but expensive and subject to rot. Moving it to a store adds a retrieval step that can miss.
- **Model or harness.** Letting the model decide the next step handles surprises; fixing the step in code is cheaper and testable. The right split is usually the least autonomy that solves the problem.
- **One loop or several.** Sub-agents keep each window clean and allow parallel work, at a cost in tokens and coordination.

A useful habit when diagnosing a failure is to ask which part it belongs to: the model lacked the capability, the context lacked the information or held the wrong information, the tool was ambiguous or returned an unhelpful error, the loop did not stop, the environment allowed something it should not have, or no trace exists to tell. [Agent Failure Diagnosis](../concepts/agent-failure-diagnosis.md) gives a method along these lines.

## In NVIDIA's stack

NVIDIA's products map onto the parts unevenly, and [NVIDIA Agentic Stack](nvidia-agentic-stack.md) follows one request through them in detail. The model is served by [NIM](../components/nvidia-nim.md), with [TensorRT-LLM](../components/tensorrt-llm.md) or another engine inside and [Dynamo](../components/nvidia-dynamo.md) above for cluster-scale routing **[web]**. The loop, tools and memory were the job of the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md), which wraps LangGraph and other frameworks and acts as an MCP client and server; it is deprecated, and its runtime role passes to NeMo Fabric **[sourced, web]**. Control is [NeMo Guardrails](../components/nemo-guardrails.md) with the classifiers in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md), observation is [NeMo Relay](../components/nemo-relay.md), and evaluation and optimisation are [NeMo Helix](../components/nemo-helix.md) **[web]**. The parts NVIDIA's documentation says least about are context and environment: it has no product dedicated to context management, and sandboxing appears only as one of Helix's plugin services **[web]**. That gap is worth naming in an interview, since those two parts are where much of current agent engineering happens.

## Interview Q&A

**What are the components of an LLM agent?** A model, its context, tools, a loop run by a harness, memory and run state, the environment where actions land, and controls and traces around them.

**Where should a business rule go: the prompt, a tool or code?** As far towards code as it can go while the task stays possible. Prompts are flexible and unreliable, tool argument constraints are firmer, and checks in the harness are deterministic.

**What does the harness do that the model does not?** It runs the loop and owns everything outside the model's choice: context management, budgets, retries, permissions, sandboxing, streaming and tracing.

**Why do agents need sandboxes when they already have guardrails?** Guardrails are probabilistic filters on content. A sandbox is a hard boundary on what any action can reach, so it still holds when a filter is bypassed **[web]**.

**How would you start diagnosing a failing agent?** Read the trace and assign the failure to a part: missing capability, bad context, a poor tool, a loop that did not stop, an over-permissive environment, or missing observability.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-28 | [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| 2026-10-28 | [Anthropic: Making Claude Code more secure and autonomous with sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing) |
| 2026-10-28 | [SWE-agent: Agent-Computer Interfaces (arXiv 2405.15793)](https://arxiv.org/abs/2405.15793) |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
