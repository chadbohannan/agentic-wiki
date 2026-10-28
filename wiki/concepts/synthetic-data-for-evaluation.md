# Synthetic Data for Evaluation

When real labelled data is scarce, teams generate test cases with LLMs and simulate users for multi-turn agents. This page covers exam objectives 3.1 and 3.3: when synthetic data helps, where it misleads, and NVIDIA's tooling for it.

Synthetic test sets feed [Agent Evaluation Pipelines](agent-evaluation-pipelines.md) and [RAG Evaluation and Ragas](rag-evaluation-ragas.md), and their quality is usually checked with the methods in [LLM-as-Judge](llm-as-judge.md). The best-known simulated-user benchmark, τ-bench, is described in [Agent Benchmarks](agent-benchmarks.md).

## Uses and risks

Synthetic data is good for bootstrapping coverage and probing edge cases. It should not replace real failures as the core of a suite: Anthropic recommends starting from 20–50 tasks drawn from real failures **[web]**. Simulated users are unreliable proxies. A 2026 study (arXiv 2601.17087) found τ-bench retail success rates swung by up to 9 points depending on which LLM played the user, with systematic errors by task difficulty and by the user's dialect **[web, abstract]**. Validate synthetic sets against production logs, and report which simulator and generator were used, since scores depend on both **[inferred]**.

## NVIDIA's tooling

NeMo Data Designer is an open-source library for configuring and running synthetic-data pipelines. It also appears as a plugin in NeMo Helix. A pipeline is declared as a schema of columns with dependencies, constraints, seed data, processors, profilers and inference settings. Data Designer then orchestrates the LLM calls, batching and validation against the specification **[web, Helix synthetic-data page; thin]**. Nemotron's synthetic-data documentation lists outputs such as chat SFT data, tool-calling SFT data and DPO preference pairs **[web, snippet]**. Its column types, judge columns and use for evaluation datasets specifically were not confirmed.

No NVIDIA product named "User Sim" was found. NVIDIA does document a voice-agent evaluation harness that pairs a simulated-user bot with the agent under test **[web, snippet]**. The study guide's reference to a user simulator remains unresolved.

## Interview Q&A

**When is synthetic evaluation data appropriate?** To bootstrap coverage and edge cases. Validate it against real logs, and keep real failures at the core of the suite **[web, inferred]**.

**What is the main risk of LLM-simulated users?** They behave differently from humans, so scores depend on the simulator and can miss dialect and demographic effects **[web]**.

**What is NeMo Data Designer?** A library for declaring and running validated synthetic-data pipelines: columns, dependencies, constraints and LLM calls **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Demystifying Evals for AI Agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) |
| 2026-10-28 | [Simulated-user variance in τ-bench (arXiv 2601.17087)](https://arxiv.org/abs/2601.17087) |
| 2026-10-28 | [NeMo Helix: design synthetic data](https://docs.nvidia.com/nemo-helix/v0.8.0/documentation/design-synthetic-data.md) |
| 2026-10-28 | [NVIDIA voice-agent evaluation quickstart](https://docs.nvidia.com/nemo/labs-voice-agent/evaluate-voice-agents/run-evaluations/quickstart) |
