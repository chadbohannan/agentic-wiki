# Bias and Toxicity Mitigation

Toxicity is the generation of harmful content. Bias is systematically different treatment, or different quality of service, across groups. Both are measured and reduced across the whole lifecycle, not only at run time.

## Across the lifecycle

Mitigation is layered:
1. Curate and balance the data.
2. Align the model.
3. Classify inputs and outputs at run time, with classifiers wired in as rails through [NeMo Guardrails](../components/nemo-guardrails.md).
4. Evaluate with red-team and fairness tests inside [Agent Evaluation Pipelines](agent-evaluation-pipelines.md), using paired or adversarial sets generated as in [Synthetic Data for Evaluation](synthetic-data-for-evaluation.md).
5. Monitor in production.

NVIDIA's run-time classifiers, described in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md), come in two kinds; the zero-shot claim below is NVIDIA's own **[web]**:
- **Llama-3.1-Nemotron-Safety-Guard-8B-v3** labels both prompts and responses as safe or unsafe, across 23 categories in 9 languages, and claims to generalise zero-shot. It was trained on CultureGuard, a synthetic multilingual dataset.
- **Nemotron-Content-Safety-Reasoning-4B** enforces custom policies written by the user, with reasoning switched on or off.

For evaluation, garak runs families of probes against a target, including toxicity probes such as `realtoxicityprompts` and `lmrc` risk cards, and its detectors score the responses. IEEE 7003 is titled "Algorithmic Bias Considerations", and IEEE 7000 gives a process for addressing ethical concerns during system design **[web]**. The study guide cites IEEE's Ethically Aligned Design as background **[sourced]**. No bias benchmark (BBQ, for example) was read for this page, so no figures from one are given.

## Limits

Classifiers encode a particular taxonomy and culture. Thresholds trade false positives, meaning over-refusal, against misses. Judge models carry their own bias (see [LLM-as-Judge](llm-as-judge.md)), and fairness metrics can conflict with each other. For agents, bias can show up in decisions about tool use and in the quality of service, not only in text.

## Interview Q&A

**How do you reduce toxicity without wrecking helpfulness?** Use layered classifiers with tuned thresholds and custom policies, and measure over-refusal alongside violations.

**What does Nemotron Safety Guard output?** A safe or unsafe verdict for both the prompt and the response, with the violated categories from its 23-category taxonomy **[web]**.

**How do you measure bias in an agent?** Build counterfactual or paired test sets across attributes and compare gaps in outcomes and quality. Include tool-use decisions, not just text.

**Which tool probes for these failures?** garak, with toxicity and risk-card probe families **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Ethically Aligned Design (IEEE)](https://ethicsinaction.ieee.org/) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
| 2026-10-28 | [IEEE standards](https://standards.ieee.org/) |
