# NVIDIA Data Flywheel Blueprint

The Data Flywheel Blueprint is a reference pipeline built on NeMo microservices. It mines production prompt and response logs to find, fine-tune and evaluate smaller models that can match a larger incumbent. It is NVIDIA's reference implementation of the loop in [Feedback Loops and the Data Flywheel](../concepts/feedback-loops-and-data-flywheel.md), narrowed to one goal: distilling to a cheaper model, which is one of the cost levers in [Tuning Agents for Accuracy, Latency and Cost](../concepts/agent-tuning-accuracy-latency-cost.md). Its README marks it deprecated as of April 2026, after the September 2025 study guide was written.

## Status

The README says: "This project is deprecated. It is no longer actively maintained, and new production use is not recommended." It names no successor and keeps the code for reference only **[web]**. An NGC Helm chart (newest 0.3.1, dated 2025-10-17) carries no deprecation flag, and some related blueprints on build.nvidia.com are tagged for deprecation **[web, snippet]**. Treat the README as authoritative. The pieces it wired together (Evaluator, Customizer, Data Designer) live on in the evaluation layer mapped in [NVIDIA Agentic Stack](../syntheses/nvidia-agentic-stack.md).

## How it works

According to the README, the pipeline works in five stages **[web]**:
1. Log production traffic tagged by `workload_id`, which identifies the task, such as one agent node or route. Rerunning on fresh logs is also how the pipeline would respond to the shifts described in [Model and Data Drift Monitoring](../concepts/model-and-data-drift-monitoring.md).
2. Group the logs by task, de-duplicate them, and build evaluation and fine-tuning datasets with class-aware stratified splits.
3. Launch LoRA fine-tuning jobs.
4. Run LLM-as-judge evaluations, comparing results across NIMs.
5. Have a human review the results and decide what to promote.

For each candidate NIM it runs three experiments: the base model replaying raw prompts, in-context learning with few-shot examples drawn from traffic, and a LoRA-customised model. It uses four NeMo services: Datastore for datasets, Customizer for fine-tuning, Evaluator for judging, and a Deployment Manager that starts and stops NIMs. Fine-tuning defaults to LoRA SFT with 2 epochs, batch size 16, learning rate 1e-4, adapter dimension 32 and dropout 0.1. A self-hosted judge needs six H100 or A100 GPUs, and a remote judge two. The stack runs on FastAPI, MongoDB, Redis with Celery, and Elasticsearch, and deploys with Docker Compose or Helm, according to the README **[web]**.

NVIDIA's reported results come from internal cases and are vendor-reported **[web]**:
- **HR chatbot:** a fine-tuned Llama 3.2 1B handled tool calling at about 98% of the 70B model's accuracy, for up to 98.6% lower inference cost.
- **Model swap without fine-tuning:** Qwen 2.5 32B Coder matched Llama 3.1 70B, cutting cost and time to first token by more than half.

## Caveats the authors state

The README is candid about its limits **[web]**:
- Production traffic flows into fine-tuning without PII removal. Redaction appears only on the roadmap.
- The production model's own responses are the only reference, with no hand labelling.
- Runs are serialised, one at a time.
- The Flywheel is "a flashlight, not an autopilot": promotion stays a human decision, and production security is the deployer's responsibility.

## Interview Q&A

**What does the Data Flywheel Blueprint automate?** Mining logged production traffic, building datasets, fine-tuning smaller candidate models, and judging them against the incumbent, so cheaper models can replace expensive ones **[web]**.

**What are the risks of training on production logs?** Personal data leaking into weights, label noise from using model output as the reference, and feedback loops that reinforce errors. That is why the README keeps humans responsible for promotion **[web]**.

**Which NeMo microservices does it use?** Datastore, Customizer and Evaluator, plus a deployment manager for NIMs **[web]**.

**Is it current?** No. Its README has marked it deprecated and reference-only since April 2026, with no named successor **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Data Flywheel Blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/data-flywheel) |
