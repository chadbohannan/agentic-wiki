# Agent Failure Diagnosis

Finding the root cause of an agent failure means combining traces, logs, a failure taxonomy and replay. The failure can sit anywhere from the inference engine (CUDA errors, tensor shapes, NCCL) up to orchestration and coordination between agents. Diagnosis starts from the traces in [Agent Observability and Tracing](agent-observability-tracing.md).

## A top-down method

Work in three steps:
1. **Classify the failure from the trace.** Typical classes are a provider or rate-limit error, a timeout, invalid tool arguments, the wrong tool, a loop, a bad plan, a retrieval miss, a hallucination or a guardrail block. The first two are infrastructure faults that the retries and circuit breakers in [Agent Error Handling and Resilience](agent-error-handling-resilience.md) should absorb; the rest are model or design faults that retrying will not fix.
2. **Localise it to a span.**
3. **Replay the same inputs** and compare with a good run.

Anomaly detection means alerting on changes in the rate of each class (error percentage, latency, a drop in run count, falling feedback scores), not only on hard failures. LangSmith's alerts work this way **[web]**.

Function-calling failures can be checked by rule: wrong function name, missing required parameters, wrong types or enum values, or hallucinated parameters. They can also be checked semantically, for whether a parameter is grounded in the context and whether units were converted correctly, as IBM's agent-evaluation explainer describes. For rate limits and provider errors, LangSmith's guidance is to reduce concurrency, rate-limit on the client side, and back off exponentially with jitter **[sourced]**.

## A failure taxonomy for multi-agent systems

Systems built on the topologies in [Multi-Agent Orchestration](multi-agent-orchestration.md) add failures between agents. The MAST study (Cemri et al., arXiv 2503.13657) annotated more than 1,600 traces from seven multi-agent frameworks. It proposes 14 failure modes in three categories: system design issues, misalignment between agents, and task verification. Its LLM-as-judge annotation pipeline agreed closely with human experts (κ = 0.88 on a 150-trace set) **[web]**. The practical reading is that many failures are problems of specification and verification rather than of model capability.

## Replay

ATIF trajectories from [NeMo Relay](../components/nemo-relay.md), and LangSmith threads and trajectories, are records that can be replayed. Arize Phoenix can replay LLM calls and rerun a dataset against different app versions. Relay's rejection marks and metadata make calls blocked by a guardrail visible afterwards **[web]**.

## Engine-level failures

TensorRT-LLM's troubleshooting page, which the study guide cites, gives a checklist **[sourced]**:
- Rebuild cleanly after updates.
- Set `CUDA_LAUNCH_BLOCKING=1` to locate plugin errors.
- Check input shapes against the profiles the engine was built with.
- Set `TLLM_LOG_LEVEL=TRACE` to see engine inputs, outputs and optimisation profiles.
- Pass `--shm-size=1g --ulimit memlock=-1` so NCCL works inside containers.
- Shrink batch and length limits when an engine build runs out of memory.
- Run under Slurm with `mpirun -n 1`.

This describes the older engine-build workflow rather than today's PyTorch backend, and it is build-and-debug guidance only loosely tied to agent diagnosis.

## Interview Q&A

**An agent worked yesterday and fails today with no deploy. Where do you look?** Filter traces by agent version, environment and error type. Then check for provider-side model changes, rate limits, changes to tool or API schemas, and changes to retrieval data. Replay a failing input against the previous pinned model.

**What are the MAST categories?** System design issues, misalignment between agents, and task verification, with 14 modes in total **[web]**.

**How do you tell a wrong tool choice from wrong tool arguments?** Use span-level checks: first the function name against the schema, then parameter validity and grounding, before judging the final answer **[sourced]**.

**A TensorRT-LLM engine errors on a valid-looking request. What are your first steps?** Compare the input's rank, length and device with the build-time profiles, and raise `TLLM_LOG_LEVEL` to TRACE. For plugin crashes, set `CUDA_LAUNCH_BLOCKING=1` **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [What is AI agent evaluation? (IBM)](https://www.ibm.com/think/topics/ai-agent-evaluation) |
| 2026-10-28 | [LangSmith evaluation: rate limiting](https://docs.smith.langchain.com/evaluation/how_to_guides/rate_limiting) |
| 2026-10-28 | [TensorRT-LLM troubleshooting](https://nvidia.github.io/TensorRT-LLM/reference/troubleshooting.html) |
| 2026-10-28 | [LangSmith alerts](https://docs.langchain.com/langsmith/alerts) |
| 2026-10-28 | [Why do multi-agent LLM systems fail? (MAST, arXiv 2503.13657)](https://arxiv.org/abs/2503.13657) |
| 2026-10-28 | [Arize Phoenix docs](https://arize.com/docs/phoenix) |
| 2026-10-28 | [NeMo Relay architecture](https://docs.nvidia.com/nemo/relay/about-nemo-relay/architecture) |
