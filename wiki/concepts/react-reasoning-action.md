# ReAct: Reasoning and Acting

ReAct interleaves reasoning traces with tool actions, and it is the foundation of most tool-calling agents. It corresponds to exam objective 1.2 and is the basic loop behind the reactive end of [Agent Architecture Patterns](agent-architecture-patterns.md).

According to the original paper (Yao et al., arXiv 2210.03629, October 2022), the model generates reasoning traces and task-specific actions in an interleaved fashion: reasoning lets it plan, track and revise, while actions query outside sources **[web]**. The paper reports that on HotpotQA and FEVER, grounding through a Wikipedia API reduces the hallucination and error propagation seen with chain-of-thought alone, and that on ALFWorld and WebShop it beats imitation and reinforcement-learning baselines by 34% and 10% absolute success using only one or two in-context examples **[web]**. The ICLR 2023 venue comes from a secondary summary.

The loop is conventionally written as Thought, Action, Observation, repeated until a Final Answer **[inferred, standard description]**. Its costs are the flip side of its adaptivity: every step is a sequential LLM call, so latency and tokens grow with the step count, a misleading observation can propagate through later steps, and the loop needs a maximum-iteration limit. The plan-ahead alternative, ReWOO, plans the whole tool graph up front and uses fewer calls at the price of adaptivity; [LLM Agent Planning](llm-agent-planning.md) compares them. Provider-native tool calling and reasoning models have largely replaced hand-parsed text ReAct, but the thought-act-observe loop remains the mental model, and the interface it depends on is described in [Tool Calling and MCP](tool-calling-and-mcp.md) **[inferred]**.

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) offers ReAct, ReWOO, Tool Calling and Reasoning agents as configurable workflow types **[web]**.

## Interview Q&A

**What does ReAct add over chain-of-thought?** Grounding through actions and observations, which reduces hallucination **[web]**.

**ReAct versus ReWOO?** ReWOO plans tool calls up front, which is cheaper and less adaptive; ReAct adapts at every step. The toolkit lists both **[inferred; web]**.

**What are the failure modes?** Loops, tool errors and growing context. Mitigate with step caps, retries and summarisation **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) |
| 2026-10-27 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
