# Reflection and Feedback Adaptation

Agents adapt in two ways. In context, they critique and revise their own output, or store lessons for later attempts, without changing any weights. Offline, they are retrained on rewards and feedback. This page covers exam objective 5.5 and the trade-off between those two routes.

Reflection is one of the planning families in [LLM Agent Planning](llm-agent-planning.md). The lessons it stores are a form of the episodic memory in [Agent Memory (Short- and Long-Term)](agent-memory.md). The offline evaluation that decides what to retrain is in [Refining Agent Decision-Making](refining-agent-decision-making.md). Collecting the user signals that feed retraining is covered in [Agent Feedback Loops](agent-feedback-loops.md) and [Structured Human Feedback and the Data Flywheel](structured-human-feedback-and-data-flywheel.md).

## In-context adaptation

Self-Refine (Madaan et al., arXiv 2303.17651) has one model generate, critique and revise its own output. Across seven tasks, from dialogue to maths, its abstract reports roughly 20% absolute improvement on average **[web, abstract read]**. Reports that gains flatten after two or three iterations are secondary **[web, unverified]**.

Reflexion (Shinn et al., arXiv 2303.11366) adapts the agent "not by updating weights". It writes verbal reflections on task feedback into an episodic memory buffer, and later trials read them. The feedback can be scalar or free-form, and can come from external or simulated sources. The abstract reports 91% pass@1 on HumanEval against 80% for GPT-4 **[web, abstract read]**. An AlfWorld figure circulating in secondary sources (65% to 97%) is not in the abstract and is unverified.

Voyager (arXiv 2305.16291) combines this with procedural memory. Its iterative prompting folds in environment feedback, execution errors and self-verification, and verified behaviours go into a growing library of executable skills. The authors report 3.3x more unique items and up to 15.3x faster tech-tree milestones in Minecraft than the prior state of the art **[web, abstract read]**.

The main caveat is intrinsic self-correction. Huang et al., "Large Language Models Cannot Self-Correct Reasoning Yet" (ICLR 2024, arXiv 2310.01798), find that without external feedback LLMs struggle to correct their own reasoning, and performance can even fall after self-correction **[web, abstract read]**. Reflection therefore works best when it is anchored to an external signal: test results, tool errors, a verifier or a judge **[inferred from 2310.01798]**.

## Weight-level adaptation

Durable behaviour change means training. NeMo RL runs RL post-training, including GRPO, PPO, DPO, DAPO and reward-model training. It takes agentic rollouts from NeMo Gym environments and uses verifiable or custom rewards, according to its documentation and NVIDIA's NeMo product page **[sourced]**. DeepSeek-R1 showed that pure RL can elicit long reasoning, as [Chain-of-Thought and Task Decomposition](chain-of-thought-and-task-decomposition.md) describes **[web]**.

In production, the loop runs through several stages **[inferred from NVIDIA's NeMo product page]**:
1. Collect user and judge signals.
2. Curate them with NeMo Curator.
3. Evaluate with NeMo Evaluator.
4. Retrain.

This is the data flywheel.

## Trade-offs

In-context reflection is cheap, immediate and reversible. But it is bounded by the context window, its stored lessons can be noisy or wrong, and without external signals it can confirm its own errors. RL is durable, but it needs reward design and compute, and it risks reward hacking **[inferred]**.

## Interview Q&A

**Reflexion versus Self-Refine?** Self-Refine iterates on a single output through self-critique. Reflexion carries verbal lessons from feedback across episodes in an episodic memory **[web]**.

**Can an LLM reliably fix its own reasoning?** Not intrinsically, per Huang et al. (ICLR 2024). Anchor reflection in external feedback such as tests, tool errors or verifiers **[web]**.

**How can an agent adapt without retraining?** Reflection memory, updated few-shot examples, skill libraries and prompt changes **[web, inferred]**.

**When do you retrain with RL?** When you have verifiable rewards or preference data and need durable behaviour change, for example GRPO or DPO in NeMo RL **[sourced, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [NVIDIA NeMo (product page)](https://www.nvidia.com/en-us/ai-data-science/products/nemo/) |
| 2026-10-27 | [NeMo RL Documentation](https://docs.nvidia.com/nemo/rl/latest/index.html) |
| 2026-10-27 | [Understanding the Planning of LLM Agents: A Survey (arXiv 2402.02716)](https://arxiv.org/abs/2402.02716) |
| 2026-10-28 | [Self-Refine (arXiv 2303.17651)](https://arxiv.org/abs/2303.17651) |
| 2026-10-28 | [Reflexion (arXiv 2303.11366)](https://arxiv.org/abs/2303.11366) |
| 2026-10-28 | [Voyager (arXiv 2305.16291)](https://arxiv.org/abs/2305.16291) |
| 2026-10-28 | [Large Language Models Cannot Self-Correct Reasoning Yet (arXiv 2310.01798)](https://arxiv.org/abs/2310.01798) |
| 2026-10-28 | [DeepSeek-R1 (arXiv 2501.12948)](https://arxiv.org/abs/2501.12948) |
