# Chain-of-Thought and Task Decomposition

Chain-of-thought (CoT) prompting gets a model to write out intermediate reasoning steps, and task decomposition breaks a goal into smaller sub-problems. Both improve accuracy on multi-step problems, and both cost extra tokens and latency. This page covers exam objective 5.2. It traces the line from prompted reasoning, through search over reasoning paths, to reasoning models trained with reinforcement learning and the test-time compute they spend.

Planners build their plans out of these decomposition techniques, as [LLM Agent Planning](llm-agent-planning.md) explains. Interleaving reasoning with tool calls gives [ReAct: Reasoning and Acting](react-reasoning-action.md). How to write the prompts themselves is covered in [Prompt Engineering and Chains](prompt-engineering-and-chains.md).

## Prompted reasoning

Wei et al. (2022, arXiv 2201.11903) introduced CoT prompting. Few-shot exemplars containing worked reasoning lead the model to reason the same way. With eight exemplars, PaLM 540B set the GSM8K state of the art at the time **[web]**. Few-shot CoT is a form of in-context learning: the model adapts from demonstrations in the prompt with no weight updates, an ability the GPT-3 paper (arXiv 2005.14165) brought to prominence **[web]**.

Self-consistency (Wang et al., arXiv 2203.11171) samples several reasoning paths and takes a majority vote on the final answer. Its abstract reports gains over plain CoT of +17.9% on GSM8K, +11.0% on SVAMP, +12.2% on AQuA, +6.4% on StrategyQA and +3.9% on ARC-challenge, at N times the token cost **[web, abstract read]**.

Least-to-most prompting (Zhou et al., arXiv 2205.10625) is decomposition made explicit. It splits a problem into simpler sub-problems and solves them in order, with each step using the earlier answers. That lets it solve problems harder than its exemplars. On SCAN, code-davinci-002 reached 99% or more on every split with 14 exemplars, against 16% for CoT **[web, abstract read]**. The planning survey the study guide cites treats task decomposition as one of the five families of agent planning **[sourced]**.

## Searching over thoughts

Tree of Thoughts (Yao et al., arXiv 2305.10601) generalises CoT into a search. The model proposes several candidate "thoughts", evaluates them, and looks ahead or backtracks. On Game of 24, GPT-4 with CoT solved 4% of tasks and ToT solved 74% **[web, abstract read]**.

Graph of Thoughts (Besta et al., arXiv 2308.09687) models thoughts as the vertices of an arbitrary graph, with dependencies as edges. This allows thoughts to be merged and refined in feedback loops. The authors report sorting quality 62% better than ToT at more than 31% lower cost **[web, abstract read; authors' own benchmark]**.

## Reasoning models and test-time compute

Reasoning models are trained with reinforcement learning, and they produce long reasoning traces without needing exemplars. DeepSeek-R1's abstract (arXiv 2501.12948) says reasoning can be developed through pure RL, without human-labelled reasoning trajectories **[web, abstract read]**. Further details come from the paper body and were not re-read in this pass **[web, not re-verified]**:
- R1-Zero was trained by RL alone.
- R1 added cold-start data and multiple stages.
- The RL algorithm was GRPO.

NVIDIA's Llama-Nemotron family (arXiv 2505.00949) comes in Nano 8B, Super 49B and Ultra 253B sizes. It is described as the first open model family with a dynamic reasoning toggle. It was built through these stages **[web, abstract read]**:
1. Neural architecture search on Llama 3.
2. Distillation and continued pretraining.
3. SFT plus large-scale RL.

Two further details about the toggle come only from search snippets:
- It is a system prompt, "detailed thinking on/off".
- NIM's reasoning-model page says Super 49B v1.5 reasons by default and is switched off with `/no_think`.

That NIM page returns no text to the fetch tool, so both details remain unconfirmed, and whether Nemotron 3 keeps a toggle has not been checked **[web, snippet-level]**. NVIDIA's blog recipe trains a small reasoning model in about 30 hours on one H100. It uses NeMo Curator plus LoRA SFT on the Llama Nemotron post-training dataset, and notes that SFT followed by RL gives the best results **[web, vendor blog]**.

Snell et al. (arXiv 2408.03314) show that allocating inference compute adaptively per prompt can be more than 4x more efficient than best-of-N. In a FLOPs-matched comparison it can beat a 14x larger model, on problems where the small model already has a non-trivial success rate **[web, abstract read]**. This is the argument for routing: spend reasoning tokens where difficulty warrants them **[inferred]**.

## Trade-offs

Every technique here trades tokens and latency for accuracy. A visible chain of thought is not guaranteed to be faithful to how the model actually reached its answer. Reasoning models can overthink simple requests. Toggling reasoning per request, or routing by difficulty, is the main cost lever **[inferred]**.

## Interview Q&A

**CoT versus ToT versus GoT?** CoT is one linear chain. ToT searches a tree of thoughts with evaluation and backtracking. GoT allows arbitrary graphs, so thoughts can be merged and refined in loops **[web]**.

**How does self-consistency work, and what does it cost?** Sample N reasoning chains and majority-vote the answer, at N times the tokens **[web]**.

**What is least-to-most prompting?** Decompose into simpler sub-problems, then solve them in sequence, each building on earlier answers **[web]**.

**What is test-time compute scaling?** Spending more inference compute per query, through sampling, search or longer reasoning. It is most efficient when allocated adaptively by difficulty **[web]**.

**How do you control reasoning cost with Nemotron?** Turn reasoning off per request with the system-prompt or `/no_think` toggle, and route easy queries to non-reasoning mode **[web, snippet-level; inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Understanding the Planning of LLM Agents: A Survey (arXiv 2402.02716)](https://arxiv.org/abs/2402.02716) |
| 2026-10-27 | [Chain-of-thought prompting (arXiv 2201.11903)](https://arxiv.org/abs/2201.11903) |
| 2026-10-27 | [Language Models are Few-Shot Learners (arXiv 2005.14165)](https://arxiv.org/abs/2005.14165) |
| 2026-10-28 | [Self-consistency (arXiv 2203.11171)](https://arxiv.org/abs/2203.11171) |
| 2026-10-28 | [Least-to-most prompting (arXiv 2205.10625)](https://arxiv.org/abs/2205.10625) |
| 2026-10-28 | [Tree of Thoughts (arXiv 2305.10601)](https://arxiv.org/abs/2305.10601) |
| 2026-10-28 | [Graph of Thoughts (arXiv 2308.09687)](https://arxiv.org/abs/2308.09687) |
| 2026-10-28 | [DeepSeek-R1 (arXiv 2501.12948)](https://arxiv.org/abs/2501.12948) |
| 2026-10-28 | [Llama-Nemotron (arXiv 2505.00949)](https://arxiv.org/abs/2505.00949) |
| 2026-10-28 | [Scaling test-time compute (arXiv 2408.03314)](https://arxiv.org/abs/2408.03314) |
| 2026-10-27 | [NIM reasoning models](https://docs.nvidia.com/nim/large-language-models/latest/reasoning-model.html) |
| 2026-10-27 | [Train a reasoning-capable LLM in one weekend with NeMo (NVIDIA blog)](https://developer.nvidia.com/blog/train-a-reasoning-capable-llm-in-one-weekend-with-nvidia-nemo/) |
