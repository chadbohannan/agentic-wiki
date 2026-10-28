# ReAct and the Agent Loop

ReAct interleaves reasoning with tool actions, and the loop it named is the core of almost every agent built today: the model thinks, calls a tool, reads the result, and repeats until it can answer. Everything else in an agent, from context management to guardrails, is arranged around this loop, so knowing exactly how it runs, how it stops and how it fails is the foundation for the rest of the design, as [Anatomy of an Agent](../syntheses/agent-anatomy.md) sets out. It sits at the reactive end of the spectrum in [Agent Architecture Patterns](agent-architecture-patterns.md).

## The paper

According to the original paper (Yao et al., arXiv 2210.03629, ICLR 2023), the model generates reasoning traces and task-specific actions in an interleaved fashion: reasoning lets it plan, track and revise, while actions query outside sources. The paper reports that on HotpotQA and FEVER, grounding through a Wikipedia API reduces the hallucination and error propagation seen with chain-of-thought alone, and that on ALFWorld and WebShop it beats imitation and reinforcement-learning baselines by 34% and 10% absolute success using only one or two in-context examples **[web]**. The contribution was the combination. Chain-of-thought alone reasons over stale internal knowledge, and acting alone has no place to plan or to notice that a result is wrong; interleaving the two lets each observation correct the next thought.

## The loop as built today

The paper's format was text: the model wrote `Thought:`, `Action:` and `Action Input:` lines, and the application parsed them, ran the tool and appended an `Observation:` line. Modern agents run the same loop through provider-native tool calling. The request carries tool schemas, the model returns structured tool calls (one or more per turn), the application executes them and appends the results as messages, and the loop ends when the model replies without a tool call, the mechanics laid out in [Tool Calling and MCP](tool-calling-and-mcp.md) **[web]**. Reasoning models now think between tool calls inside the model rather than in a visible `Thought:` line, but the cycle of decide, act and observe is unchanged.

Anthropic's "Building Effective Agents" describes agents in exactly these terms: they are "typically just LLMs using tools based on environmental feedback in a loop". It stresses that the agent must get "ground truth" from the environment at each step to judge its progress, that it can pause for human feedback at checkpoints or blockers, and that stopping conditions such as a maximum number of iterations keep it under control **[web]**. The ground-truth point is the important one. A loop whose observations are real (a test run, a query result, a compiler error) corrects itself; a loop whose observations are the model's own claims about progress does not.

The software that runs the loop is now usually called the harness. Anthropic describes its Claude Agent SDK as a general-purpose agent harness that runs the model with tools, context management and compaction **[web]**. A minimal harness is a few dozen lines of code. A production harness also manages the context window, enforces budgets and permissions, executes tools in a sandbox, retries failures, streams progress and records a trace, and most of the engineering effort in an agent goes there rather than into the loop itself.

## Stopping and budgets

A loop that relies on the model to decide when it is done needs outside limits as well. The usual set is a cap on steps or tool calls, a token or cost budget, a wall-clock timeout, and a check for repetition, such as the same tool called with the same arguments several times running. When a limit trips, the agent should stop with a clear partial result or escalate to a person rather than fail silently; the escalation patterns are in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md). The loop is also a retry layer in its own right, since a model that sees a tool error will often try again, so step caps and the per-call retry policies in [Agent Error Handling and Resilience](agent-error-handling-resilience.md) belong to one budget rather than multiplying.

## Costs and failure modes

ReAct's costs are the flip side of its adaptivity. Every step is a sequential model call, so latency and tokens grow with the step count, and because each call resends the whole history, input tokens grow faster than the step count does. The failure modes follow from the same structure:

- **Looping.** The agent repeats an action that keeps failing, or alternates between two. Step caps and repetition checks catch it; better error messages from tools prevent it.
- **Error propagation.** A wrong or misleading observation is accepted and built on for many steps, the "context poisoning" described in [Context Engineering](context-engineering.md).
- **Context growth.** Long tool outputs fill the window and degrade recall, which is why compaction and tool-result clearing exist.
- **Premature stopping.** The agent declares success without checking, which Anthropic observed in its long-running coding agents marking features done after only unit tests or `curl` checks **[web]**.
- **Goal drift.** Over many steps the original task fades from attention. Manus counters it by having the agent rewrite a to-do file so the plan sits at the end of the context **[web]**.

## Variants

The main alternative is to plan before acting. ReWOO plans the whole tool graph up front, runs it, and writes an answer, which uses fewer model calls at the price of adaptivity; plan-and-execute designs add a replanning step when results surprise the planner. [LLM Agent Planning](llm-agent-planning.md) compares them. Reflection loops such as Reflexion wrap ReAct in an outer cycle of trying, critiquing and retrying, covered in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md). For long or open-ended tasks the trend is towards a plain ReAct loop over a capable model with good tools and careful context management, rather than elaborate control structures, which matches Anthropic's advice to start with the simplest design that works **[web]**.

## In NVIDIA's stack

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) offers ReAct, ReWOO, Tool Calling and Reasoning agents as configurable workflow types **[web]**. Its ReAct agent's documentation lists settings for parse failures alongside tool limits: `retry_agent_response_parsing_errors` (default true), `parse_agent_response_max_retries` (1), `tool_call_max_retries` (1) and `max_tool_calls` (15) in version 1.3 **[web, snippet]**. The parse settings suggest it reads the model's text in something like the paper's format. The Tool Calling agent, by contrast, "does not reason between steps" and chooses tools from their schemas, and it requires a model that supports tool or function calling **[web]**. That probably means native tool calls, which need a model and NIM configured for them, as [NVIDIA NIM](../components/nvidia-nim.md) describes. The toolkit is deprecated, so these setting names are best treated as an example of the controls any harness needs rather than as a lasting API.

## Interview Q&A

**What does ReAct add over chain-of-thought?** Grounding. Actions fetch real observations that correct the reasoning, which reduced hallucination and error propagation in the paper's experiments **[web]**.

**How does a modern agent implement ReAct?** Through native tool calling: the model returns structured tool calls, the harness executes them and appends results, and the loop ends when the model answers without a tool call.

**How do you make sure the loop terminates?** Step or tool-call caps, token and time budgets, and repetition detection, with a defined behaviour (partial answer or escalation) when a limit trips.

**ReAct versus ReWOO?** ReWOO plans tool calls up front, which is cheaper and less adaptive; ReAct decides at every step. Choose ReWOO when the tool sequence is predictable.

**What are the main failure modes?** Loops, propagated errors, context growth, premature completion and goal drift. Mitigate with budgets, actionable tool errors, compaction, verification against the environment and recited plans.

**Why do agent costs grow faster than the number of steps?** Each step resends the full history, so input tokens grow roughly with the square of the step count unless the context is compacted or cached.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
| 2026-10-28 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-28 | [NeMo Agent Toolkit Tool Calling agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/tool-calling-agent.html) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-28 | [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| 2026-10-28 | [Manus: Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) |
