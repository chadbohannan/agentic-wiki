# Agent Benchmarks

Public agent benchmarks supply shared tasks for comparing models and scaffolds. Each measures a different layer: code changes, policy-bound dialogue, open-ended assistance, interactive environments or function calling. This page covers exam objectives 3.1 and 3.2: what the main benchmarks measure and where they mislead.

Benchmarks supply tasks for the harnesses in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), and several depend on the model-based grading discussed in [LLM-as-Judge](llm-as-judge.md). The newer "decision model" category reports its own benchmarks, such as BFCL and the Jev Decision Index, which are discussed on [Decision Models](decision-models.md).

## The main benchmarks

**SWE-bench** (arXiv 2310.06770) has 2,294 real GitHub issues from 12 Python repositories. The model must edit the codebase so the issue's tests pass. At publication the best system, Claude 2, resolved about 1.96% **[web, abstract]**. The subset now used most is **SWE-bench Verified**, released by OpenAI with the SWE-bench authors in August 2024. It is 500 samples that 93 professional developers judged well specified and fairly tested, out of 1,699 they reviewed. It supersedes the original test set and SWE-bench Lite **[web, OpenAI announcement via search summary; the page itself returns 403]**. Anthropic's evals post names SWE-bench Verified and Terminal-Bench as the backbone of coding-agent evaluation **[web]**.

**τ-bench** (arXiv 2406.12045) pairs an agent with a user simulated by an LLM, domain API tools and a policy document. Success is judged by comparing the final database state with an annotated goal. The benchmark introduced the pass^k reliability metric **[web, abstract]**.

**GAIA** (arXiv 2311.12983) has 466 real-world questions that need reasoning, multimodality, web browsing and tool use. At publication humans scored 92% and GPT-4 with plugins 15%. The answers to 300 questions are withheld for the leaderboard **[web, abstract]**.

**AgentBench** (arXiv 2308.03688, ICLR 2024) spans eight interactive environments. It found weak long-term reasoning, decision-making and instruction following to be the main obstacles **[web, abstract]**.

**BFCL**, the Berkeley Function Calling Leaderboard, has evolved through four versions:
- **V1** scored single-turn calls by comparing their syntax trees with expected calls.
- **V2** added enterprise and community-contributed functions.
- **V3** added multi-turn conversations.
- **V4** evaluates agentic behaviour holistically.

It reports overall accuracy as the unweighted mean of its sub-categories, alongside cost and latency. It separates models with native function calling from models prompted to imitate it **[web, leaderboard page; version descriptions partly from snippets]**. Anthropic's post also cites WebArena and OSWorld for computer-use agents **[web]**.

## Limits

A benchmark score belongs to a model and its scaffold together, and contamination and saturation erode its value over time. Simulated users are imperfect proxies. A 2026 study (arXiv 2601.17087) found that τ-bench retail success rates shifted by up to 9 points depending on which LLM played the user. The simulated users underestimated performance on hard tasks, overestimated it on moderate ones, and gave worse results for speakers of African American Vernacular English and Indian English **[web, abstract]**. Use public leaderboards as a first filter, then build domain evals from your own failures **[inferred, consistent with Anthropic]**. Match the benchmark to the kind of agent **[web, Anthropic; mapping inferred]**:

| Agent type | Benchmark |
| ---------- | --------- |
| Coding | SWE-bench Verified, Terminal-Bench |
| Conversational, policy-bound | τ-bench |
| Research assistants | GAIA-style grounded questions |
| Computer use | WebArena, OSWorld |
| Tool calling | BFCL |

## Interview Q&A

**What does τ-bench add over function-calling benchmarks?** Multi-turn dialogue with a simulated user, domain policies, grading on final database state, and pass^k reliability **[web]**.

**What is SWE-bench Verified, and why does it exist?** A 500-task subset of SWE-bench that human developers checked for clear issue descriptions and fair tests. Many original tasks were underspecified or had tests that rejected valid fixes **[web]**.

**What does BFCL measure?** Function-calling accuracy, from single-turn calls through multi-turn and agentic use, with cost and latency reported alongside **[web]**.

**Why not rely only on public benchmarks?** Scores depend on the scaffold, suffer contamination and saturation, don't match your domain or policies, and vary with the simulated user **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [SWE-bench (arXiv 2310.06770)](https://arxiv.org/abs/2310.06770) |
| 2026-10-28 | [Introducing SWE-bench Verified (OpenAI)](https://openai.com/index/introducing-swe-bench-verified/) |
| 2026-10-28 | [τ-bench (arXiv 2406.12045)](https://arxiv.org/abs/2406.12045) |
| 2026-10-28 | [GAIA (arXiv 2311.12983)](https://arxiv.org/abs/2311.12983) |
| 2026-10-28 | [AgentBench (arXiv 2308.03688)](https://arxiv.org/abs/2308.03688) |
| 2026-10-28 | [Berkeley Function Calling Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html) |
| 2026-10-28 | [Simulated-user variance in τ-bench (arXiv 2601.17087)](https://arxiv.org/abs/2601.17087) |
| 2026-10-28 | [Demystifying Evals for AI Agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) |
