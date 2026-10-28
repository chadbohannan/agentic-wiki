# Context Engineering

Context engineering is the work of deciding which tokens an agent's model sees at each step. Those tokens include the system prompt, tool definitions, examples, conversation history, retrieved documents and the results of earlier tool calls. Prompt engineering is the part of this concerned with writing instructions; context engineering is the larger job, because in an agent most of the window is not written by the developer at all but accumulates as the loop runs. It is probably the single most important skill in building agents today, since the model is fixed and the context is the main thing the builder controls on every call.

## Why context is a budget

Anthropic's context-engineering guidance (September 2025) defines the task as curating "the optimal set of tokens" during inference, and treats context as a finite resource with diminishing returns: each token draws on an "attention budget". It gives two reasons. Attention relates every token to every other, so relationships grow with the square of length, and models see fewer long sequences in training, so they have less practice with long-range dependencies. The guidance describes the decline as gradual rather than a cliff **[web]**.

Two studies give that decline a shape. Liu et al.'s "Lost in the Middle" (TACL, 2023) found that on multi-document question answering and key-value retrieval, accuracy was highest when the relevant information sat at the beginning or end of the input and dropped significantly when it sat in the middle, even for long-context models **[web]**. Chroma's "Context Rot" report (July 2025) tested 18 models and found that reliability falls as input grows even on simple tasks, that a single topically related distractor hurts, and that on LongMemEval a focused prompt of about 300 tokens beat the full prompt of about 113K tokens holding the same answer **[web]**. Chroma also found that models did better on haystacks of shuffled sentences than on coherent essays, a result it reports across all tested models without a settled explanation **[web]**. A larger window therefore raises the ceiling without removing the need to curate, a point [Agent Memory](agent-memory.md) makes when it weighs long context against retrieval.

## What goes into the window

Anthropic's guidance gives the principle as "the smallest set of high-signal tokens" that makes the desired outcome likely, and applies it to each part of the context **[web]**:

- **System prompt.** Pitch it at the right altitude, between brittle hard-coded if-else rules and vague guidance that assumes shared context. Start minimal on the best available model and add instructions only to fix observed failures. Minimal does not mean short.
- **Tools.** Keep the set small, self-contained and non-overlapping, with token-efficient outputs; the guidance warns that if a human cannot say which tool applies, the agent probably cannot either. The design rules are in [Tool Calling and MCP](tool-calling-and-mcp.md).
- **Examples.** Use a few diverse, canonical examples rather than a long list of edge-case rules, which is the few-shot practice described below.
- **Data.** Prefer just-in-time retrieval, in which the agent holds lightweight references (paths, queries, links) and loads content with tools when it needs it.

The last of these is the biggest shift from the RAG era. Anthropic describes Claude Code as a hybrid: project instruction files are loaded up front, and the rest is found at run time with glob, grep, `head` and `tail`, so large files never enter the window whole. The guidance names the cost as speed, since run-time exploration is slower than pre-retrieval and needs good tools to avoid wasting context, and suggests the hybrid may suit less dynamic domains such as law and finance **[web]**. Embedding-based pre-retrieval, as in [RAG Retrieval Pipelines](rag-retrieval-pipelines.md), remains the better choice when the corpus is large, unstructured and not navigable by name.

## Writing the prompt

Prompt engineering is the older and narrower craft inside this one. NVIDIA's 2023 introduction to prompt engineering and p-tuning defines zero-shot prompting as an instruction with no examples, few-shot prompting as adding worked examples so the model adapts in context, and chain-of-thought as eliciting stepwise reasoning, either with examples or a cue phrase, a family developed in [Chain-of-Thought and Task Decomposition](chain-of-thought-and-task-decomposition.md) **[sourced]**. P-tuning trains a small encoder that produces "virtual tokens" prepended to the prompt, which are cached in a lookup table after training. It is far cheaper than full fine-tuning and leaves the base model frozen, at the cost of non-human-readable, per-task artefacts and a need for training data **[sourced]**. Because that article is from 2023, current practice leans on in-context prompting, LoRA and other parameter-efficient methods, and reasoning models trained with reinforcement learning more than on p-tuning. Writing prompts by hand is also no longer the only option: optimisers such as DSPy learn instructions and demonstrations against a metric, as described in [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md).

Composing prompts into sequences of calls (prompt chaining, routing, parallel and evaluator-optimizer flows) is a question of control flow rather than of context, and belongs to the workflow patterns in [Agent Architecture Patterns](agent-architecture-patterns.md).

## Four operations

LangChain's survey of the field (July 2025) groups the techniques into four operations **[web]**:

- **Write** information outside the window, to a scratchpad, a file or long-term memory, so it can be recovered later.
- **Select** what to bring back in: state fields, retrieved memories, documents, or a subset of tool descriptions chosen by retrieval.
- **Compress** what is already there, by summarising or trimming.
- **Isolate** context across sub-agents, sandboxes or separate state fields so that no single window carries everything.

The same post relays Drew Breunig's names for what goes wrong when context is not managed: poisoning (a hallucination enters the context and is treated as fact), distraction (the context outweighs what the model learned in training), confusion (superfluous content steers the answer) and clash (parts of the context contradict each other) **[web]**. These are useful vocabulary in [Agent Failure Diagnosis](agent-failure-diagnosis.md), because a trace often shows the bad token entering the window several steps before the bad action.

## Long-horizon tasks

Tasks that outrun one window need explicit techniques. Anthropic's guidance names three **[web]**:

- **Compaction.** Summarise the history as it nears the limit and continue from the summary. Claude Code keeps architectural decisions, unresolved bugs and implementation details, discards redundant tool output, and carries on with the summary plus the five most recently used files. The guidance advises tuning the compaction prompt for recall first and precision second, and calls clearing old tool results the lightest form of compaction.
- **Structured note-taking.** The agent writes notes to a file or memory store outside the window and reads them back after a reset. Anthropic's example is Claude playing Pokémon, which kept maps, progress and strategy notes across thousands of steps.
- **Sub-agents.** A lead agent delegates focused work to sub-agents with clean windows. Each may use tens of thousands of tokens exploring but returns a summary of 1,000 to 2,000 tokens. This is the context argument for the topologies in [Multi-Agent Orchestration](multi-agent-orchestration.md).

The guidance matches each to a task shape: compaction for long back-and-forth, notes for iterative work with milestones, sub-agents for research that benefits from parallel exploration **[web]**. Anthropic's later post on long-running agents adds that compaction "doesn't always pass perfectly clear instructions to the next agent", and that its coding agents worked better when each session left structured artefacts for the next: a JSON feature list with pass flags, a progress file, git commits, and an `init.sh` that starts the environment **[web]**. The lesson is that durable state should live in files the agent and its successors can read, not only in a summary.

## Designing for the KV cache

Manus's account of building its agent (July 2025) argues that KV-cache hit rate is "the single most important metric" for a production agent. Its agent averages about 100 input tokens per output token, because context grows each step while outputs are short tool calls, and on the Claude Sonnet pricing it quotes cached input cost a tenth of uncached **[web]**. The post draws several rules from this **[web]**:

- Keep the prompt prefix stable, since a one-token change, such as a timestamp at the top of the system prompt, invalidates the cache from that point.
- Make the context append-only, and serialise deterministically, since unstable JSON key order silently breaks the cache.
- Mask tools rather than removing them. Tool definitions sit near the front of the context, so adding or removing them mid-task invalidates the cache and leaves references to tools that no longer exist. Manus constrains decoding to allowed tools instead, using name prefixes such as `browser_` and `shell_` to select groups.
- Treat the file system as context. Drop a page's content but keep its URL, or a document's text but keep its path, so compression is restorable.
- Recite the plan. Manus rewrites a `todo.md` as it works, which pushes the plan to the end of the window where attention is strongest; its typical task takes about 50 tool calls.
- Keep failures in context, so the model sees the error and shifts away from repeating it.
- Vary the formatting of repeated actions, because a uniform history makes the model imitate its own pattern; the post's example is an agent falling into a rhythm while reviewing 20 résumés.

These rules partly conflict with the compaction advice above, since compaction rewrites history and breaks the cache. The practical reconciliation is to compact rarely and at clear boundaries, and to keep everything before the compaction point stable. Manus is one vendor writing about its own system, so its numbers are best read as an informed account rather than a measured benchmark.

## In NVIDIA's stack

The cache argument carries straight into NVIDIA's serving layer. [Disaggregated Serving and KV-Aware Routing](disaggregated-serving-and-kv-routing.md) describes how [NVIDIA Dynamo](../components/nvidia-dynamo.md) routes each request to the worker already holding most of its prompt in cache, which is exactly what an append-only agent context needs to turn each step's prefill into a cache hit **[web]**. Manus's own advice for self-hosted serving is to enable prefix caching in the engine and route a session's requests consistently, which is the same idea **[web]**. On the framework side, the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) offers an automatic memory wrapper that retrieves stored context before each agent call, a select operation in the terms above, with the provider plugins described in [Agent Memory](agent-memory.md) **[web]**. NVIDIA's study material treats this area mainly as prompt engineering and memory, and the term "context engineering" does not appear in the study guide **[sourced]**.

## Interview Q&A

**How is context engineering different from prompt engineering?** Prompt engineering writes the instructions. Context engineering manages everything in the window on every step, most of which (tool results, history, retrieved data) the agent accumulates rather than the developer writes **[web]**.

**When would you pick p-tuning over few-shot?** When examples consume the token budget or fail to generalise and labelled data exists; today you would also weigh LoRA or a stronger model **[sourced]**.

**Why not just use a model with a million-token window?** Recall degrades with length and with position, distractors hurt, and every step re-pays for the whole window. Chroma found a focused 300-token prompt beat the full 113K-token version **[web]**.

**What is compaction, and what does it cost?** Summarising history to continue in a fresh window. It loses detail and breaks the prompt cache, so tune it for recall and do it at clear boundaries, keeping durable state in files **[web]**.

**Why mask tools instead of removing them?** Removing tools changes the prefix and invalidates the KV cache, and leaves references in history to tools that no longer exist **[web]**.

**Name four ways context goes wrong.** Poisoning, distraction, confusion and clash **[web]**.

**When would you choose just-in-time retrieval over pre-retrieval?** When the data is navigable by name or structure (code, file trees, APIs) and the agent has good search tools; pre-retrieval wins on large unstructured corpora and when latency matters.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [An Introduction to LLMs: Prompt Engineering and P-Tuning (NVIDIA)](https://developer.nvidia.com/blog/an-introduction-to-large-language-models-prompt-engineering-and-p-tuning/) |
| 2026-10-28 | [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) |
| 2026-10-28 | [Anthropic: Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) |
| 2026-10-28 | [Manus: Context Engineering for AI Agents](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) |
| 2026-10-28 | [LangChain: Context Engineering for Agents](https://www.langchain.com/blog/context-engineering-for-agents) |
| 2026-10-28 | [Chroma: Context Rot](https://www.trychroma.com/research/context-rot) |
| 2026-10-28 | [Lost in the Middle (arXiv 2307.03172)](https://arxiv.org/abs/2307.03172) |
| 2026-10-28 | [NeMo Agent Toolkit documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html) |
