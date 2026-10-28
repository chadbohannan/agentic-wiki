# Refining Agent Decision-Making

Refining decision-making means measuring and improving how an agent chooses actions and tools, using trajectory evaluation, LLM judges, evaluator-optimizer loops, and iteration on tools and prompts. This page covers exam objective 2.6.

Trajectory-level findings feed the reflection mechanisms in [Reflection and Feedback Adaptation](reflection-and-feedback-adaptation.md). The toolkit's evaluation and profiling features are in [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md), and tool-description fixes are covered in [Tool Calling and MCP](tool-calling-and-mcp.md). Building the harness is the subject of [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), and the judging methods are in [LLM-as-Judge](llm-as-judge.md).

## Evaluate the trajectory

Judging only the final answer misses wrong tool choices, needless loops and poor error recovery. The TRAJECT-Bench paper (arXiv 2510.04550, October 2025) grades each step of an agent's tool-use trajectory on tool selection, argument correctness and dependency or order satisfaction, as well as final-answer accuracy. It reports that models often confuse similar tools, sometimes pick tools without considering their parameters, and degrade as tool counts and trajectory length grow, with the main bottleneck appearing when tasks move from short to medium-length trajectories **[web, abstract read in full]**. Because several different paths can be correct (three tool calls or twelve), prefer rule-based checks of required tools and outcomes to strict path matching **[web, vendor blogs; inferred]**.

For ambiguous cases an LLM judge helps, using rubrics, ground-truth examples and calibration against human labels. One vendor suggests 75 to 90 percent agreement with humans before trusting a judge, which is unverified **[web, vendor blog]**.

## Levers for improvement

Anthropic's guidance supplies most of the practical levers. Improve tool descriptions and signatures from observed errors, start prompts minimal and add guidance per failure, use an evaluator-optimizer loop when criteria are clear, check ground truth from the environment at each step, add human checkpoints, and cap iterations with stopping conditions **[web]**. Feeding tool execution errors back to the model enables self-correction, as the MCP specification describes **[web]**. More refinement raises accuracy at the cost of latency and tokens, and more autonomy raises compounding-error risk, so Anthropic advises sandboxed testing with guardrails **[web]**.

The NeMo Agent Toolkit's navigation lists evaluation, profiling and performance monitoring, an optimizer, a sizing calculator, test-time compute and fine-tuning under improving workflows, but the details of each were not read **[sourced, gap]**. Their successors after the toolkit's deprecation are covered in [NeMo Evaluator and Helix Evaluation](../components/nemo-evaluator-and-helix-eval.md).

## Interview Q&A

**How do you evaluate whether an agent picks the right tool?** Compare the chosen tools with a task-defined required set, validate arguments, measure call success rate, and use a judge only for ambiguous cases **[web, vendor glossary; inferred]**.

**What is an evaluator-optimizer loop, and when do you use it?** One LLM generates and another critiques against clear criteria, iterating; use it when criteria are explicit and iteration measurably helps **[web]**.

**The agent confuses two similar tools; what do you do?** Rewrite the descriptions to delineate boundaries, merge or remove overlapping tools, add examples, and rerun the eval set **[web]**.

**Why not exact trajectory matching in tests?** Multiple valid paths exist and agents are non-deterministic **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [TRAJECT-Bench (arXiv 2510.04550)](https://arxiv.org/abs/2510.04550) |
| 2026-10-27 | [How to use LLM-as-judge for agent evaluation (Label Studio)](https://labelstud.io/learningcenter/how-to-use-llm-as-judge-for-agent-evaluation) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-27 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-27 | [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
