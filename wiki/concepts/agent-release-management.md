# Agent Release Management

Release management for agents applies the discipline of software delivery to a system whose behaviour depends on prompts, models and tools together, and whose outputs vary from run to run. It has three parts: versioning every artifact so a deployed combination can be named and rebuilt, gating each change on an evaluation against the previous version, and re-running those comparisons after release because behaviour can move without a deploy. No study-guide source covers agent CI/CD directly, so the pattern here is assembled from practitioner posts, vendor documentation and NVIDIA's tooling.

## Versioned artifacts

Prompts, model pins, tool schemas, guardrail policies and configuration should all be versioned artifacts that can be promoted and reversed. LangSmith's prompt management shows the mechanics **[web]**:
- **Commits:** every save is a commit.
- **Tags:** a tag points to one commit and can be moved. Promotion sets the reserved `staging` and `production` tags, and promoting to production does not remove a prompt from staging.
- **Rollback:** each environment keeps an ordered history, so rolling back re-points the tag to an earlier commit.
- **Pulling:** code pulls `name:production` to follow the tag, or `name:<commit-hash>` to pin a version immutably.
- **Controls:** one webhook per workspace fires on each commit, for CI/CD or GitHub sync, and owner-only mode restricts who can move tags.

Arize Phoenix also versions prompts **[web]**. Following a tag gives instant rollout and rollback, but behaviour can change without review; pinning a hash is reproducible, but every change needs a deploy **[web]**.

Models need the same treatment: pin provider model snapshots, NIM container tags and adapter versions, and keep the previous version deployed for rollback. A classic failure is mismatched versions across prompt, model and tools. The remedy is a manifest of all three, plus guardrail configuration, recorded in the metadata of every trace; LangSmith's metadata and tags support this **[web]**. Without the manifest, [Agent Failure Diagnosis](agent-failure-diagnosis.md) cannot tell which release caused a failure, and a regression run cannot say what it compared.

## The release pipeline

Several practitioner posts describe a shared pipeline, with illustrative thresholds **[web, snippet]**:
- **Offline gate.** Run a golden dataset against both the candidate and the baseline with the same graders (rules, LLM-as-judge and trajectory metrics), reusing the harness in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), and block merges on regression: a hard gate for safety, a softer one for quality. Arize Phoenix can group traces into datasets and rerun them against different app versions **[web]**.
- **Contract tests.** Test tools against recorded tool traces, so CI is deterministic.
- **Shadow.** Run the candidate on sampled live inputs and discard its outputs, at extra LLM cost.
- **Canary.** Expose 5–10% of traffic to the candidate, with promotion criteria written in advance, using the rollout machinery in [Kubernetes GPU Inference Deployment](kubernetes-gpu-inference-deployment.md).
- **Session pinning.** Keep each agent session on one version during a rollout.

Comparing two versions of a non-deterministic agent needs more care than comparing two builds of a service. In practice that means comparing per slice rather than only in aggregate, so an average cannot hide a regression on one tool path; repeating trials and reporting confidence intervals, using the paired statistics described in [Agent Profiling and Result Analysis](agent-profiling-and-result-analysis.md); and controlling evaluation cost and 429 errors with concurrency limits, a rate limiter and backoff, as LangSmith's guide advises **[sourced]**. Drift in the judge model and benchmark contamination can also fake a regression or hide one, as discussed in [LLM-as-Judge](llm-as-judge.md) and [Agent Benchmarks](agent-benchmarks.md).

## After release

Regression checks continue after rollout: online evaluators that score sampled production traces, user feedback scores, and scheduled re-runs of the offline suite. The scheduled re-runs matter because a hosted model can change underneath an unchanged name. Chen, Zaharia and Zou measured this on GPT-4, whose accuracy at identifying prime numbers fell from 84% in March 2023 to 51% in June 2023 **[web]**. IBM's explainer lists A/B testing, human review, simulation and benchmarks as evaluation methods, measured against success criteria defined in advance **[sourced]**. An alert on the eval, feedback or error SLOs in [LLM Reliability Metrics and SLOs](llm-reliability-metrics-slos.md) is the natural rollback trigger, re-pointing the production tag to the previous version.

Some loops propose changes automatically and gate them with evaluations. The NeMo Agent Toolkit's optimiser did this for hyperparameters and prompts, and retraining can be triggered by drift or by flywheel results, as in the [NVIDIA Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md) **[sourced, web]**. That blueprint insists on a human approval gate before promotion **[web]**, and the same rule is sensible for any automated proposal.

Governance wraps the whole pipeline: a registry of models and prompts with owners, change approval, an audit trail of deployed versions, versioned guardrail policies and tested rollback.

## In NVIDIA's stack

The NeMo Agent Toolkit's `nat eval` and profiler were the CI hooks for the gate, writing per-sample outputs and evaluator scores for quality, latency and reliability **[sourced]**. The toolkit is deprecated, and these duties pass to [NeMo Helix](../components/nemo-helix.md) for agent evaluation and optimisation, and to [NeMo Evaluator](../components/nemo-evaluator.md), whose `nel gate` turns benchmark thresholds into an explicit GO, NO-GO or INCONCLUSIVE release decision **[web]**. The Data Flywheel Blueprint benchmarks base, in-context-learning and LoRA-customised candidate models against data derived from production, scoring them with an LLM judge between 0 and 1 **[web]**. NVIDIA's AI Factory white paper has subsections titled "AgentOps", "GitOps Controller", "Artifact Repository", "Observability" and "Deployment Strategies", but the published page gives little more than these headings **[sourced]**. For the model tier, the [NIM Operator](../components/nim-operator.md) gives declarative, version-pinned rolling upgrades of NIM services **[web]**.

## Interview Q&A

**What is different about CI for agents compared with ordinary services?** Outputs are non-deterministic, behaviour is set by prompt, model and tools together, and an upstream model can drift silently. So gate on eval scores over trajectories, not on unit tests alone, and keep re-running after release.

**Shadow, canary or A/B?** Shadow runs the candidate silently on live inputs at extra LLM cost. A canary exposes a small share of users to it. A/B splits traffic to compare outcome metrics statistically **[web]**.

**How do you show that v2 is no worse than v1?** The same dataset and judges, repeated trials, per-slice comparison and confidence intervals on paired differences, with thresholds set in advance from a baseline distribution **[web]**.

**Tag or commit-hash pinning?** Tags move, giving fast rollout and rollback but risking silent change. Hashes are immutable, so they are reproducible but slower to change **[web]**.

**What do you record to make rollback safe?** A manifest of the prompt commit, model and adapter versions, tool schemas and guardrail configuration, stored in trace metadata, so a multi-agent system can be rolled back as a unit.

**Why re-run benchmarks when nothing changed in your code?** Provider model updates, changes to tools or APIs, and data drift all cause regressions without a deploy; GPT-4's shift between its March and June 2023 versions is the documented case **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit evaluation docs (1.8)](https://docs.nvidia.com/nemo/agent-toolkit/1.8/improve-workflows/evaluate.html) |
| 2026-10-28 | [AI Factory white paper](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/index.html) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
| 2026-10-28 | [LangSmith: manage prompts](https://docs.langchain.com/langsmith/manage-prompts) |
| 2026-10-28 | [LangSmith evaluation: rate limiting](https://docs.smith.langchain.com/evaluation/how_to_guides/rate_limiting) |
| 2026-10-28 | [Arize Phoenix docs](https://arize.com/docs/phoenix) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
| 2026-10-28 | [How is ChatGPT's behavior changing over time? (Chen, Zaharia, Zou; arXiv 2307.09009)](https://arxiv.org/abs/2307.09009) |
