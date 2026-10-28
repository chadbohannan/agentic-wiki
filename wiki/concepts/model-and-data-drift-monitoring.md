# Model and Data Drift Monitoring

Drift monitoring detects when production inputs, outputs or quality move away from what the system was built and validated on, and decides when to retrain or adapt. The classical machine-learning framing the study guide cites needs translating for LLM agents, where ground truth is usually missing.

## The classical framing

NVIDIA's 2023 blog on monitoring models in production separates three kinds of drift **[sourced]**:
- **Data drift:** the input feature distributions shift away from the training data.
- **Model drift:** predictive power declines as the world changes.
- **Prediction drift:** the distribution of predictions shifts. This serves as a proxy when ground truth arrives late.

The blog recommends statistical tests plus continued performance tracking. It advises starting to monitor in development, and having a response plan that runs from alert to diagnosis. It names Evidently AI, SageMaker Model Monitor, and Prometheus with Grafana as tools.

## Translating to LLM agents

Classical feature drift becomes several kinds of drift for an agent:
- **Input drift:** a change in the mix of topics, languages or lengths, or in the embeddings of user queries.
- **Retrieval drift:** corpus changes, a stale index or falling relevance of the hits.
- **Tool and API drift:** changes to schemas or behaviour.
- **Model drift:** updates on the provider's side.

Ground truth is usually missing, so monitor proxies: online evaluator scores, user feedback, refusal and guardrail rates, tool-error rates, and the distributions of output length and embeddings. These sit on the same dashboards as the metrics in [LLM Reliability Metrics and SLOs](llm-reliability-metrics-slos.md).

The study guide also cites LangChain's time-weighted retriever. It scores each item by similarity plus (1 − decay rate)^hours since the item was last accessed, favouring fresh items **[sourced]**. That addresses stale knowledge, but it is a retrieval technique, not a monitor, and its connection to this objective is tangential.

## Trade-offs

Over-sensitive drift tests cause alert fatigue. Drift is not the same as harm. And retraining on drifted data needs fresh labels or judge scores, which is what the loop in [Feedback Loops and the Data Flywheel](feedback-loops-and-data-flywheel.md), and NVIDIA's [Data Flywheel Blueprint](../components/nvidia-data-flywheel-blueprint.md), are meant to supply.

## Interview Q&A

**Data drift, model drift, prediction drift?** A change in the input distribution; a decline in predictive performance; a change in the output distribution, used as a proxy when labels lag **[sourced]**.

**How do you monitor an LLM app without labels?** Through proxies: online judge scores, feedback, tool-error and refusal rates, and shifts in output and embedding distributions.

**What does a time-weighted retriever do, and is it monitoring?** It combines similarity with an exponentially decaying recency term to favour fresh items. It is a retrieval technique, not a monitor **[sourced]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [A guide to monitoring machine learning models in production (NVIDIA blog)](https://developer.nvidia.com/blog/a-guide-to-monitoring-machine-learning-models-in-production) |
| 2026-10-28 | [Monitoring ML models in production: data quality and integrity (Towards Data Science)](https://towardsdatascience.com/monitoring-machine-learning-models-in-production-how-to-track-data-quality-and-integrity-391435c8a299/) |
| 2026-10-28 | [Time-weighted retriever (LangChain JS)](https://js.langchain.com/docs/integrations/retrievers/time-weighted-retriever/) |
