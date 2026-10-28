# LLM Agent Planning

Planning is how an agent turns a goal into a sequence of actions and revises that sequence as results come in. The central design choice is how much to plan before acting.

## Taxonomy

Huang et al.'s survey (arXiv 2402.02716, February 2024), which the study guide cites, sorts LLM-agent planning into five families **[sourced]**:
- **Task decomposition:** split the goal into sub-goals, using the techniques in [Chain-of-Thought and Task Decomposition](chain-of-thought-and-task-decomposition.md).
- **Plan selection:** generate several candidate plans and pick one.
- **External module:** hand planning to a formal planner.
- **Reflection:** revise the plan from feedback, as described in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md).
- **Memory:** retrieve past experience to inform the plan.

The categories overlap in practice, since most working agents combine decomposition, reflection and memory.

## Interleaved versus up-front planning

[ReAct](react-reasoning-action.md) interleaves thought, action and observation. It adapts to what each tool returns, but it resends a growing context at every step. ReWOO (Xu et al., arXiv 2305.18323) separates reasoning from observations. A planner lays out the whole tool-call graph in advance, with placeholders for evidence; workers execute the calls; and a solver composes the answer. The paper reports 5x token efficiency and a 4% accuracy gain on HotpotQA, and robustness when tools fail **[web]**. The cost is that a static plan cannot react to a surprising observation, so ReWOO suits tasks whose structure is predictable.

Plan-and-execute with replanning sits between the two. In LangChain's description (2024), a planner writes a multi-step plan, executors carry out each step with tools, and the agent is then called with a re-planning prompt to either answer or write a follow-up plan. LangChain claims speed and cost gains over ReAct because the large model is not consulted after every action, while noting that tool calls remain serial **[web]**. Plan selection applies the search ideas from ToT and from test-time compute work. It samples candidate plans and scores them with a verifier or a process reward model.

## External planners

LLM+P (Liu et al., arXiv 2304.11477) works in three steps. The LLM translates a natural-language problem into PDDL, a classical planner searches for a solution, and the result is translated back into natural language **[web]**. The LLM handles the language and the solver handles the search, which pays off when constraints are formal and correctness matters, as in scheduling and logistics.

## Failure modes and NVIDIA's implementation

Long-horizon plans fail in several ways: errors compound across steps, agents invoke tools that do not exist, and loops never terminate. The mitigations are step limits, per-step validation, reflection and replanning, checkpointed state, and human approval for risky steps. The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ships these patterns as agent types, so an architect chooses among them through configuration: ReAct, Reasoning, ReWOO, Router, Tool Calling, and Parallel and Sequential executors **[sourced]**. Its ReAct agent caps runaway behaviour with tool-call and retry limits **[web]**.

## Interview Q&A

**Name the planning families in the Huang survey.** Task decomposition, plan selection, external module, reflection and memory **[sourced]**.

**ReAct versus ReWOO?** ReAct adapts at every step but is token-heavy. ReWOO plans once, so it uses about 5x fewer tokens and its failures are easier to trace, but its plan is rigid **[web]**.

**How do you keep long-horizon plans reliable?** Decompose, validate each step, replan on failure, cap iterations, checkpoint state, and require human approval for risky actions.

**When would you use an external planner?** When constraints are formal and the plan must be provably valid. The LLM translates to PDDL and the solver searches **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Understanding the Planning of LLM Agents: A Survey (arXiv 2402.02716)](https://arxiv.org/abs/2402.02716) |
| 2026-10-28 | [ReWOO (arXiv 2305.18323)](https://arxiv.org/abs/2305.18323) |
| 2026-10-28 | [LLM+P (arXiv 2304.11477)](https://arxiv.org/abs/2304.11477) |
| 2026-10-28 | [Scaling test-time compute (arXiv 2408.03314)](https://arxiv.org/abs/2408.03314) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-28 | [Plan-and-Execute Agents (LangChain blog)](https://www.langchain.com/blog/planning-agents) |
