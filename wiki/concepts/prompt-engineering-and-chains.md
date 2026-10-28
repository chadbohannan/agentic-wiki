# Prompt Engineering and Chains

Prompt engineering covers designing prompts (zero-shot, few-shot, chain-of-thought, p-tuning) and composing them into static or dynamic chains, and it is now usually framed as the wider discipline of context engineering. This page covers exam objective 2.1.

The chains and routers here are the workflow patterns compared in [Agent Architecture Patterns](agent-architecture-patterns.md). Reasoning-eliciting prompts such as chain-of-thought are developed in [Chain-of-Thought and Task Decomposition](chain-of-thought-and-task-decomposition.md), and schema-constrained output is covered in [Structured Outputs](structured-outputs.md).

## Prompting techniques

NVIDIA's 2023 introduction to prompt engineering and p-tuning defines zero-shot prompting as an instruction with no examples, few-shot prompting as adding worked examples so the model adapts in context, and chain-of-thought as eliciting stepwise reasoning, either with examples or a cue phrase **[sourced]**. P-tuning trains a small encoder that produces "virtual tokens" prepended to the prompt, which are cached in a lookup table after training. It is far cheaper than full fine-tuning and leaves the base model frozen, at the cost of non-human-readable, per-task artefacts and a need for training data **[sourced]**. Because that article is from 2023, current practice leans on in-context prompting, LoRA and other parameter-efficient methods, and reasoning models trained with reinforcement learning more than on p-tuning **[inferred]**.

## Chains, static and dynamic

Anthropic's "Building Effective Agents" describes prompt chaining as splitting a task into sequential calls, each consuming the previous output, with optional programmatic gates between steps; it trades latency for accuracy because each call is an easier task. Its dynamic patterns choose the next step at run time: routing classifies an input and dispatches it, parallelization runs sectioned or voting calls, orchestrator-workers lets an LLM decompose the task, and evaluator-optimizer loops generate and critique against clear criteria. Fully agentic flows cost more and risk compounding errors, and Anthropic advises the simplest workable design, often a single call with retrieval and examples **[web, read in full]**. Static chains are predictable and debuggable but brittle against unanticipated inputs, while dynamic flows handle open-ended tasks at higher cost and with more loop risk **[inferred]**.

The [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) exposes router, sequential-executor, parallel-executor, ReAct, ReWOO and tool-calling agent types configured in YAML, the toolkit's realisation of static versus dynamic chains, according to its documentation navigation **[sourced]**. Whether the exam expects toolkit-specific vocabulary beyond that is unknown.

## Context engineering

Anthropic's context-engineering guidance (dated September 2025 by a search snippet) frames prompt engineering as one part of managing everything the model sees: system prompt, tools, history and retrieved data. It recommends the smallest set of high-signal tokens because recall degrades as the window fills ("context rot"), system prompts pitched at the right altitude that start minimal and add guidance from observed failures, a few diverse canonical examples over long edge-case rule lists, just-in-time retrieval, compaction, structured note-taking and sub-agents with clean contexts **[web, snippet-level; the page itself should be read in full]**.

## Interview Q&A

**Prompt chaining versus a router?** Chaining is a fixed sequence in which step N feeds step N+1, with optional validation gates; a router classifies the input once and dispatches to a specialised prompt or model **[web]**.

**When would you pick p-tuning over few-shot?** When examples consume the token budget or fail to generalise and labelled data exists; today you would also weigh LoRA or a stronger model **[sourced, inferred]**.

**What is a dynamic chain?** One whose next step, template or model is selected at run time from state or model output rather than hard-coded **[web, inferred phrasing]**.

**How do you stop long-running agent prompts degrading?** Treat context as scarce: a minimal system prompt, compaction or clearing of stale tool results, external notes, sub-agents and just-in-time retrieval **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [An Introduction to LLMs: Prompt Engineering and P-Tuning (NVIDIA)](https://developer.nvidia.com/blog/an-introduction-to-large-language-models-prompt-engineering-and-p-tuning/) |
| 2026-10-27 | [NVIDIA Agent Intelligence Toolkit overview](https://docs.nvidia.com/aiqtoolkit/latest/index.html) |
| 2026-10-28 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
| 2026-10-27 | [Anthropic: Effective context engineering for AI agents](https://anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
