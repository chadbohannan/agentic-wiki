# Prompt and Model Versioning and Rollback

Prompts, model pins, tool schemas and configuration should be versioned artifacts that can be promoted and reversed, with the deployed combination recorded on every trace. This page covers exam objective 8.4: the mechanics of prompt versioning, pinning models, and automated tuning with a human gate.

Versioned artifacts are what let [Continuous Regression Benchmarking](continuous-regression-benchmarking.md) compare releases, and what let the rollouts in [Agent MLOps and CI/CD](agent-mlops-cicd.md) be reversed. The recorded manifest is also what makes [Agent Failure Diagnosis](agent-failure-diagnosis.md) possible after a bad release.

## Prompt versioning

LangSmith's prompt management shows the mechanics **[web]**:
- **Commits:** every save is a commit.
- **Tags:** a tag points to one commit and can be moved. Promotion sets the reserved `staging` and `production` tags, and promoting to production does not remove a prompt from staging.
- **Rollback:** each environment keeps an ordered history, so rolling back re-points the tag to an earlier commit.
- **Pulling:** code pulls `name:production` to follow the tag, or `name:<commit-hash>` to pin a version immutably.
- **Controls:** one webhook per workspace fires on each commit, for CI/CD or GitHub sync, and owner-only mode restricts who can move tags.

Arize Phoenix also versions prompts **[web]**.

## Model pinning

Pin exact model versions: provider model snapshots, NIM container tags and adapter versions. Keep the previous version deployed for rollback **[inferred]**. The [NIM Operator](../components/nim-operator.md) gives declarative, version-pinned rolling upgrades of NIM services **[web]**.

Following a tag gives instant rollout and rollback, but behaviour can change without review. Pinning a hash is reproducible, but every change needs a deploy **[web, inferred]**. A classic failure is mismatched versions across prompt, model and tools. Record a manifest of all three, plus guardrail configuration, in trace metadata, so that diagnosis (objective 8.2) and comparison (8.3) are possible **[inferred]**.

## Automated tuning

Objective 8.4 also covers loops that propose changes automatically and gate them with evaluations. The NeMo Agent Toolkit's optimiser did this for hyperparameters and prompts. It is deprecated, and its successor features are in NeMo Helix, as [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md) describes **[sourced, web]**. Retraining can be triggered by drift or by flywheel results, as in the [NVIDIA Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md). Keep a human approval gate before promotion, as that blueprint itself insists **[web]**.

## Interview Q&A

**Tag or commit-hash pinning?** Tags move, giving fast rollout and rollback but risking silent change. Hashes are immutable, so they are reproducible but slower to change **[web]**.

**What do you record to make rollback safe?** A manifest of the prompt commit, the model and adapter versions, the tool schemas and the guardrail configuration, stored in trace metadata **[inferred]**.

**How is rollback triggered automatically?** An alert on the eval, feedback or error SLO after a canary re-points the production tag to the previous version **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [LangSmith: manage prompts](https://docs.langchain.com/langsmith/manage-prompts) |
| 2026-10-28 | [Arize Phoenix docs](https://arize.com/docs/phoenix) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
