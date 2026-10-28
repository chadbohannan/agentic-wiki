# Agent MLOps and CI/CD

CI/CD for agents applies the release discipline of software to a system whose behaviour depends on prompts, models and tools together, and whose outputs vary from run to run. This page covers exam objective 4.2. No study-guide source covers agent CI/CD directly, so most of the page is synthesis from practitioner posts and NVIDIA's tooling, and is labelled accordingly.

The gates rely on the offline checks in [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), rollouts run on the infrastructure in [Kubernetes GPU Inference Deployment](kubernetes-gpu-inference-deployment.md), and versioning and rollback are detailed in [Prompt and Model Versioning and Rollback](prompt-and-model-versioning-rollback.md). The scheduled re-runs that catch drift once an agent is live are [Continuous Regression Benchmarking](continuous-regression-benchmarking.md).

## The pattern

Several practitioner posts describe a shared pattern. They were seen only in search snippets, and their thresholds are illustrative **[web, snippet-level]**:
- Treat prompts, tool schemas and model pins as versioned artifacts.
- Run a golden-set evaluation in CI and block merges on regression: a hard gate for safety, a softer one for quality.
- Add tool contract tests with recorded tool traces, so CI is deterministic.
- Run the candidate in shadow mode on sampled production traffic.
- Canary 5–10% of traffic, with promotion criteria written in advance.
- Keep each agent session on one version during a rollout.
- Re-run evals on a schedule, because a provider can change a model and cause a regression without any code change.

Governance wraps this **[inferred]**:
- a registry of models and prompts with owners;
- change approval;
- an audit trail of deployed versions;
- versioned guardrail policies;
- tested rollback.

## NVIDIA's pieces

The NeMo Agent Toolkit's `nat eval` and profiler are the CI hooks for the gate. They write per-sample outputs and evaluator scores for quality, latency and reliability **[sourced]**. The toolkit is deprecated, and these duties pass to [NeMo Evaluator and Helix Evaluation](../components/nemo-evaluator-and-helix-eval.md), whose `nel gate` produces an explicit GO, NO-GO or INCONCLUSIVE release decision **[web]**. NVIDIA's AI Factory white paper has subsections titled "AgentOps", "GitOps Controller", "Artifact Repository", "Observability" and "Deployment Strategies", but only its outline could be read **[sourced, table of contents only]**. The [NIM Operator](../components/nim-operator.md) gives declarative, version-pinned rolling upgrades for the model tier **[web]**.

## Interview Q&A

**What is different about CI for agents compared with ordinary services?** Outputs are non-deterministic. Behaviour is set by prompt, model and tools together, and an upstream model can drift silently. So gate on eval scores over trajectories, not on unit tests alone **[inferred]**.

**Shadow versus canary?** Shadow runs the candidate on live inputs and discards its outputs, at extra LLM cost. Canary exposes a small slice of real users to the candidate **[web]**.

**How do you set gate thresholds?** From a baseline distribution on a representative golden set, with per-slice checks (for example by tool path), so an aggregate score cannot hide a regression **[web, inferred]**.

**How do you version a multi-agent system?** Version each prompt, tool schema and model pin, plus a manifest for the composition, and roll back the manifest as a unit **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Agent Toolkit evaluation docs (1.8)](https://docs.nvidia.com/nemo/agent-toolkit/1.8/improve-workflows/evaluate.html) |
| 2026-10-28 | [AI Factory white paper](https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/index.html) |
| 2026-10-28 | [NeMo Evaluator docs (nightly)](https://docs.nvidia.com/nemo/evaluator/latest/index.html) |
