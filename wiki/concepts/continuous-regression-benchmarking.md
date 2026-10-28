# Continuous Regression Benchmarking

Continuous regression benchmarking re-runs a fixed benchmark, together with online comparisons, on every change and on a schedule. The goal is to catch regressions against the previous version before and after rollout. This page covers exam objective 8.3. Apart from IBM's explainer and NVIDIA's flywheel, its practices come from blog-grade sources and synthesis, and are labelled so.

Regression suites reuse the harness in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), gate releases in [Agent MLOps and CI/CD](agent-mlops-cicd.md), and depend on the version tracking in [Prompt and Model Versioning and Rollback](prompt-and-model-versioning-rollback.md).

## Three layers

**Offline.** Run a golden dataset against both the candidate and the baseline with the same graders (rules, LLM-as-judge and trajectory metrics), and gate merges on thresholds **[inferred]**. Arize Phoenix supports this directly: it can group traces into datasets and rerun them against different app versions **[web]**.

**Online, before release.** Shadow mode runs the candidate on live inputs and discards its outputs. A canary exposes a small share of users to it **[inferred, blog-grade]**.

**After release.** Use online evaluators that score sampled production traces, feedback scores, and scheduled re-runs, because hosted model versions can change underneath you **[inferred]**. IBM's explainer lists A/B testing, human review, simulation and benchmarks as evaluation methods, measured against success criteria defined in advance **[sourced]**.

## Mechanics

The practical rules, synthesised from the pages linked here **[inferred]**:
- Record version metadata on every run (agent version, prompt commit, model pin). LangSmith's metadata and tags support this **[web]**.
- Compare per slice, not just in aggregate.
- Handle non-determinism with repeated trials and confidence intervals.
- Control evaluation cost and 429 errors with concurrency limits, a rate limiter and backoff, as LangSmith's guide advises **[sourced]**.
- Watch for drift in the judge model and for benchmark contamination, as discussed in [LLM-as-Judge](llm-as-judge.md) and [Agent Benchmarks](agent-benchmarks.md).

On the NVIDIA side, the Data Flywheel Blueprint benchmarks base, in-context-learning and LoRA-customised candidate models against data derived from production, scoring them with an LLM judge between 0 and 1 **[web, blueprint README]**. NeMo Evaluator's `nel gate` turns benchmark thresholds into a release decision, as described in [NeMo Evaluator and Helix Evaluation](../components/nemo-evaluator-and-helix-eval.md) **[web]**.

## Interview Q&A

**Shadow, canary or A/B?** Shadow runs the candidate silently on live inputs. A canary exposes a small share of users to it. A/B splits traffic to compare outcome metrics statistically **[inferred]**.

**How do you show that v2 is no worse than v1 for a non-deterministic agent?** Use the same dataset and judges, repeated trials, per-slice comparison and confidence intervals, with thresholds set in advance **[inferred]**.

**Why re-run benchmarks when nothing changed in your code?** Provider model updates, changes to tools or APIs, and data drift all cause regressions without a deploy **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [LangSmith evaluation: rate limiting](https://docs.smith.langchain.com/evaluation/how_to_guides/rate_limiting) |
| 2026-10-28 | [Arize Phoenix docs](https://arize.com/docs/phoenix) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
