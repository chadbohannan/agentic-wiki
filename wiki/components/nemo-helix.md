# NeMo Helix

NeMo Helix is NVIDIA's open-source control plane for evaluating, optimising and hardening agents, and it is what the NeMo Agent Toolkit's migration guide names as the successor to `nat eval` and `nat optimize`. It matters for an interview because it is the current answer to "how would you improve an agent on NVIDIA's stack", even though the September 2025 exam predates it. Its first public release, 0.1.0, is dated 2026-05-20, and the current release is 0.8.1 (2026-10-23). Release notes for 0.9.0 are already published with a date of 2026-11-03, after this page was checked, so the 0.9 details below describe a release that is announced rather than shipped **[web]**. Where Helix sits beside Relay, Fabric, Switchyard and NIM is laid out in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md).

## What it is

The v0.8 documentation describes Helix as running on the user's own infrastructure, locally or on Kubernetes, and reachable through a REST API, a CLI, a Python SDK and NeMo Studio, an alpha web UI; the CLI is the primary interface. Its infrastructure (agent execution and sandboxing, files, secrets, authentication, model management and inference middleware) is built from plugins, each with a default implementation that can be swapped for the user's own. The architecture diagram lists its services as managed agents, optimisation, Guardrails, Evaluator, fine-tuning, garak, Data Designer, Anonymizer and Safe Synthesizer, with NVIDIA Build, OpenAI, Anthropic, Gemini and Ollama as model providers **[web]**.

Most of those services were separate NeMo microservices in 2025: Customizer, Evaluator, Guardrails and Data Designer. Helix therefore reads as the platform that absorbs that family under one CLI, which fits the unconfirmed report of a NeMo Microservices sunset noted on the [NeMo Agent Toolkit](nemo-agent-toolkit.md) page, though no NVIDIA source read so far says so.

## Agents and the candidate flow

Release 0.1.0 managed the whole agent lifecycle through `nemo agents` (create, deploy, invoke, evaluate, optimise, secure, undeploy), and it used toolkit workflows as the agent format, so LangGraph agents worked through the toolkit's wrapper and other frameworks needed one. By 0.8 Helix had its own `agent.yaml` format, kept the toolkit's LangGraph agents as a legacy path, and ran registered agents through NeMo Fabric harnesses, deploying them to Docker, Kubernetes or OpenShell. The 0.9 notes rename `nemo agents create` to `nemo agents register`; registering a name alone creates a trace-only agent that can be analysed but not run **[web]**.

Helix never edits a running agent in place. Optimisation and security changes are applied to a sibling agent, validated, and only then promoted **[web]**. This is the validate-then-promote discipline of [Agent Release Management](../concepts/agent-release-management.md) built into the tool.

## Inference gateway and virtual models

Every model call goes through an Inference Gateway, where providers are registered and exposed as virtual models. Middleware is attached to a virtual model: Switchyard routing between strong and weak models, and [NeMo Guardrails](nemo-guardrails.md) input and output rails **[web]**. Routing and rails can therefore be added or changed by pointing an agent at a different virtual model, without touching its code. In the 0.9 notes, Switchyard optimisation names its virtual models `<agent>-<digest>` by content, so an identical rerun reuses them and a changed routing configuration gets a new name **[web]**.

## Evaluation

Helix's evaluation service implements the pipeline described in [Agent Evaluation Pipelines](../concepts/agent-evaluation-pipelines.md). Release 0.1.0 offered LLM-as-judge, similarity, RAG and agentic metrics against agent or model targets. By 0.8 it could run a registered agent through Fabric, Gym or Harbor runners, filter stored tasks and tasksets, store artefacts in the Files service and serve a live metric endpoint **[web]**. Harbor task suites can serve as the regression suite that such a release gate re-runs.

The service was called the Evaluator plugin until the v0.8 docs renamed it NeMo Helix Evals. The CLI group moved from `nemo evaluator` to `nemo evals`, the API prefix from `/apis/evaluator/` to `/apis/evals/`, and the configuration block from `evaluator:` to `evals:` **[web]**. The upgrade guide warns that the old block is not an alias: a leftover `evaluator: {sandboxed_gym_default: true}` is silently ignored, so the new default execution mode applies, and an old flag that disabled evaluations falls back to enabled. It also says older jobs disappear from the new job lists, though they remain in the core Jobs service. The 0.9 notes disable custom Python metrics serialised with cloudpickle by default, because they run user code with service credentials **[web]**. The standalone [NeMo Evaluator](nemo-evaluator.md) framework, with its `nel` CLI, is documented separately, and neither set of docs explains how the two relate.

## Optimisation

Helix succeeds the toolkit's optimiser described in [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md). In 0.8, `nemo agents optimize list-strategies` describes the available strategies and `nemo agents optimize run-strategy --strategy <name>` submits one. The strategies include prompt-master, Switchyard routing, Analyst and Experimentalist, and Helix can combine numeric hyperparameter search with prompt optimisation. The release notes call these strategies research-preview agentic workflows whose "quality and autonomy still require developer review" **[web]**.

## Hardening, data and training

Helix bundles [NeMo Guardrails](nemo-guardrails.md) for content safety, jailbreak detection and PII redaction, and the red-teaming scanner described in [NVIDIA Safety Models and garak](nvidia-safety-models-and-garak.md). It adds Anonymizer for PII in training data, Data Designer for the synthetic data described in [Synthetic Data for Evaluation](../concepts/synthetic-data-for-evaluation.md), and Safe Synthesizer for differentially private synthetic data. Customizer runs fine-tuning, including DPO and GRPO; GRPO needs Kubernetes and Ray, and the 0.8 notes list as a known issue that GRPO's sandboxing "has not undergone security review". In 0.9, trace analysis is renamed NeMo Compass, producing insights with trace and span evidence, severity and confidence **[web]**.

## Maturity

Helix is pre-1.0, and minor releases break things. In 0.8, OIDC configuration changed so that CLIs from 0.7 or earlier cannot log in to a 0.8 server. In 0.9, agent registration and trace-analysis commands were renamed **[web]**. Pin the server, CLI and SDK versions together, and describe Helix in an interview as the named successor that is still young, not as a settled product.

## Interview Q&A

**What is NeMo Helix?** An open-source, self-hosted control plane for evaluating, optimising and hardening agents, with a CLI, SDK, REST API and the alpha Studio UI. It bundles evaluation, optimisation strategies, Guardrails, garak, Data Designer and fine-tuning behind one gateway **[web]**.

**How does it relate to the NeMo Agent Toolkit?** It succeeds the toolkit's evaluation and optimiser. It began by running toolkit workflows as its agent format and moved to its own `agent.yaml` and Fabric harnesses, keeping toolkit LangGraph agents as a legacy path **[web]**.

**What happened to NeMo Evaluator inside Helix?** The Evaluator plugin was renamed Helix Evals (`nemo evals`, `/apis/evals/`), and its configuration block is not aliased, so stale settings are silently dropped **[web]**.

**How does Helix change a running agent safely?** It applies optimisation or security changes to a sibling agent, validates the sibling and only then promotes it **[web]**.

**How would you add routing or guardrails to an agent without code changes?** Point the agent at a virtual model on Helix's Inference Gateway that carries Switchyard or Guardrails middleware **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [NeMo Helix home (v0.8)](https://docs.nvidia.com/nemo-helix/v0.8/documentation/home) |
| 2026-10-28 | [NeMo Helix release 0.1.0 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/release-0-1-0) |
| 2026-10-28 | [NeMo Helix release 0.8.0 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/current-release) |
| 2026-10-28 | [NeMo Helix release 0.8.1 notes](https://docs.nvidia.com/nemo-helix/v0.8/documentation/reference/release-notes/release-0-8-1) |
| 2026-10-28 | [NeMo Helix release 0.9.0 notes](https://docs.nvidia.com/nemo-helix/v0.9/documentation/reference/release-notes/release-0-9-0) |
| 2026-10-28 | [NeMo Helix: upgrading from NeMo Evaluator](https://docs.nvidia.com/nemo-helix/v0.8/documentation/evaluate-models/upgrading-from-evaluator) |
| 2026-10-28 | [NeMo Agent Toolkit migration guide](https://docs.nvidia.com/aiqtoolkit/latest/resources/migration-guide.html) |
