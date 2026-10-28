# Wiki Index

Pages are grouped by the seven parts of an agent set out in [Anatomy of an Agent](syntheses/agent-anatomy.md): model, context, tools, loop and harness, memory and state, environment, and control and observation, followed by governance, which sits outside the agent. Concept and component pages share each group; components are NVIDIA products and live under `components/`. Exam objective IDs from the NCP-AAI study guide are in parentheses; [NCP-AAI Exam Blueprint](syntheses/exam-blueprint.md) maps objectives to pages and records the guide's errors, and `../TODO.md` lists open gaps.

## Syntheses
- [Anatomy of an Agent](syntheses/agent-anatomy.md) — The conceptual spine: the seven parts, how each fails, where a responsibility should live, and diagnosing a failure by part
- [NVIDIA Agentic Stack](syntheses/nvidia-agentic-stack.md) — One agent request traced through NVIDIA's layers, plus every product's status and lineage as of October 2026
- [NCP-AAI Exam Blueprint](syntheses/exam-blueprint.md) — The ten exam sections mapped to wiki pages, with weight discrepancies and broken study-guide references

## 1. Model

### Choosing and adapting the model
- [Chain-of-Thought and Task Decomposition](concepts/chain-of-thought-and-task-decomposition.md) (5.2) — CoT, self-consistency, least-to-most, ToT/GoT, reasoning models and test-time compute
- [Decision Models](concepts/decision-models.md) (post-guide) — Typed-probability models (Jev, Clef, CLM-8B) for routing and gating; vendor claims and security findings
- [RAG vs Fine-Tuning vs Continued Pretraining](concepts/rag-vs-fine-tuning-vs-pretraining.md) (6.1, 6.4) — Retrieval for facts, fine-tuning for behaviour, and RAFT as the hybrid

### Serving the model
- [LLM Inference Optimization](concepts/llm-inference-optimization.md) (7.4) — Why decode is memory-bound, and batching, paged KV, quantisation, speculative decoding and parallelism
- [LLM Inference Profiling](concepts/llm-inference-profiling.md) (4.3) — TTFT, ITL and throughput; AIPerf load tests; Nsight and DCGM traces
- [Disaggregated Serving and KV-Aware Routing](concepts/disaggregated-serving-and-kv-routing.md) (4.1, 4.5) — Splitting prefill from decode and routing by cache overlap, and why agents benefit
- [Kubernetes GPU Inference Deployment](concepts/kubernetes-gpu-inference-deployment.md) (4.1, 4.4, 4.5) — GPU scheduling, autoscaling on queue or KV signals, and model-aware load balancing
- [Inference Cost and Availability](concepts/inference-cost-and-availability.md) (4.5) — FP8/FP4, batching, routing and spot capacity against headroom, redundancy and fallback
- [NVIDIA NIM](components/nvidia-nim.md) (7.2, 7.5) — Model containers with OpenAI-compatible APIs that pick a vLLM, SGLang or TensorRT-LLM profile per GPU
- [TensorRT-LLM](components/tensorrt-llm.md) (4.3, 4.5, 7.4) — NVIDIA's inference engine: kernels, FP8/FP4, speculative decoding, now a PyTorch runtime
- [NVIDIA Dynamo](components/nvidia-dynamo.md) (4.1, 4.3, 4.4, 4.5) — Cluster orchestration above the engines: disaggregation, KV-aware routing, KV offload, SLA autoscaling
- [Triton Inference Server (Dynamo-Triton)](components/triton-inference-server.md) (7.4) — The general multi-framework server: backends, dynamic and sequence batching, ensembles
- [NVIDIA NIM Operator](components/nim-operator.md) (4.1, 4.4) — Kubernetes CRDs that cache, deploy, autoscale and upgrade NIMs
- [DCGM GPU Telemetry](components/dcgm-gpu-telemetry.md) (4.3, 4.4) — GPU health and utilisation metrics, exported to Prometheus for monitoring and autoscaling

## 2. Context

### Shaping the window
- [Context Engineering](concepts/context-engineering.md) (2.1, 1.4) — Context as an attention budget; prompting techniques and p-tuning; just-in-time retrieval, write/select/compress/isolate, compaction, notes, sub-agents and KV-cache-aware design
- [Structured Outputs](concepts/structured-outputs.md) (2.3, 2.4) — JSON mode versus schema-constrained decoding, NIM's guided decoding, and why valid shape is not truth
- [Multimodal Agents](concepts/multimodal-agents.md) (2.2, 7.5) — Getting images, audio and video into the loop: token budgets, multimodal RAG, NIM media limits and VLM tool calling, Speech NIM, the voice agent blueprint and VSS video chunking

### Retrieval
- [RAG Retrieval Pipelines](concepts/rag-retrieval-pipelines.md) (6.1, 1.7) — Chunking, hybrid search, reranking, query rewriting, agentic RAG, and the NVIDIA RAG Blueprint
- [Vector Databases and ANN Tuning](concepts/vector-databases-and-ann-tuning.md) (6.2) — HNSW, IVF and PQ trade-offs, tuning, filtered search, Milvus and GPU indexing with CAGRA
- [Document Ingestion and ETL](concepts/document-ingestion-etl.md) (6.3, 6.4) — Parsing documents into chunks with NV-Ingest, freshness through CDC, and data-quality checks
- [Knowledge Graphs and GraphRAG](concepts/knowledge-graphs-graphrag.md) (1.7) — GraphRAG for global, multi-hop questions, and temporal graphs as agent memory
- [Structured Data Access and Text-to-SQL](concepts/structured-data-access-text-to-sql.md) (6.5) — Text-to-SQL versus typed tools, BIRD accuracy, safeguards, and routing between stores

## 3. Tools
- [Tool Calling and MCP](concepts/tool-calling-and-mcp.md) (2.3) — Function-calling mechanics, Anthropic's tool-design rules, the MCP tools specification, and loop limits
- [Agent Protocols: MCP, A2A, AG-UI](concepts/agent-protocols-mcp-a2a.md) (1.3) — The three-protocol stack: agent-to-tool, agent-to-agent, agent-to-user; versions and governance

## 4. Loop and harness
- [Agent Architecture Patterns](concepts/agent-architecture-patterns.md) (1.2, 1.6, 1.8, 2.1) — Workflows versus agents, Anthropic's five workflow patterns as static and dynamic chains, and reactive, deliberative and hybrid designs
- [ReAct and the Agent Loop](concepts/react-reasoning-action.md) (1.2) — The paper, the loop as built with native tool calls, harnesses, stopping budgets, failure modes and variants
- [LLM Agent Planning](concepts/llm-agent-planning.md) (5.3) — Five planning families, interleaved versus up-front planning (ReAct vs ReWOO), and external PDDL planners
- [Reflection and Feedback Adaptation](concepts/reflection-and-feedback-adaptation.md) (5.5) — In-context self-critique (Self-Refine, Reflexion) versus weight-level RL, and why reflection needs external signals
- [Multi-Agent Orchestration](concepts/multi-agent-orchestration.md) (1.5) — Supervisor, hierarchical, decentralised and federated topologies, their costs, and the frameworks implementing them
- [Agent Error Handling and Resilience](concepts/agent-error-handling-resilience.md) (2.4) — Retries with backoff, idempotency, circuit breakers, fallbacks, and agent-specific failure classes
- [Streaming and Real-Time Agent UX](concepts/streaming-and-realtime-agent-ux.md) (2.5) — What to stream, LangGraph stream modes, and streaming's conflicts with retries and guardrails
- [NeMo Agent Toolkit (and its deprecation)](components/nemo-agent-toolkit.md) (2.3, 2.4, 2.6, 7.3) — The exam's agent framework: YAML workflows, agent types, eval and profiler; deprecated October 2026
- [AI-Q Blueprint](components/ai-q-blueprint.md) (4.1) — NVIDIA's research agent: intent router, shallow and deep agents, Compose and Helm deployment

## 5. Memory and state
- [Agent Memory (Short- and Long-Term)](concepts/agent-memory.md) (1.4, 5.1) — Context-window versus episodic, semantic and procedural memory; MemGPT, Mem0, Zep; NVIDIA's implementations
- [Stateful Orchestration](concepts/stateful-orchestration.md) (1.6, 5.4) — Graph runtimes with checkpointed state for loops, pauses and recovery, using LangGraph persistence

## 6. Environment
- [Agent Security Threats and Controls](concepts/agent-security-threats-and-controls.md) (9.1) — Prompt injection, tool poisoning and the lethal trifecta, with controls ranked by reliability

The environment is the wiki's thinnest part: sandboxing and execution environments have no page of their own yet, and are treated only inside the anatomy synthesis and the security page.

## 7. Control and observation

### Guardrails and safety
- [Layered Safety and Escalation](concepts/layered-safety-and-escalation.md) (9.4, 9.2) — Defence in depth from cheap filters to human escalation, failing closed, and topic adherence
- [Privacy and PII Protection](concepts/privacy-and-pii-protection.md) (9.2) — Where personal data enters and leaves an agent, GLiNER-PII, and minimisation principles
- [Bias and Toxicity Mitigation](concepts/bias-and-toxicity-mitigation.md) (9.3) — Lifecycle mitigation, NVIDIA safety classifiers, garak probes, and IEEE 7003
- [NeMo Guardrails](components/nemo-guardrails.md) (7.1, 9.2, 9.4) — Programmable rails configured in `config.yml` and Colang 1.0/2.0; LLMRails vs the faster IORails engine; parallel rails and speculative generation
- [NVIDIA Safety Models and garak](components/nvidia-safety-models-and-garak.md) (9.2, 9.3, 9.4) — Content-safety, topic, jailbreak and PII classifiers, and the garak vulnerability scanner

### Human oversight
- [Human-in-the-Loop Patterns](concepts/human-in-the-loop-patterns.md) (10.1) — Approval gates, interrupts with resume, MCP elicitation, and risk-based escalation as implemented today
- [Agent Interfaces and Oversight UI](concepts/agent-interfaces-and-oversight.md) (1.1, 10.1) — AG-UI, Article 14's checklist turned into interface features, and approval fatigue
- [Agent Transparency and Explainability](concepts/agent-transparency-and-explainability.md) (10.3, 8.5) — Why chain-of-thought is not a faithful explanation, and Article 50 disclosure duties
- [Human Oversight and Accountability](concepts/human-oversight-and-accountability.md) (10.4) — Meaningful oversight, automation bias, and assigning accountability to people

### Observation
- [Agent Observability and Tracing](concepts/agent-observability-tracing.md) (8.1, 8.2) — Runs as traces of spans, OpenTelemetry GenAI conventions, instrumentation points and backends
- [LLM Reliability Metrics and SLOs](concepts/llm-reliability-metrics-slos.md) (8.1, 8.5) — Serving, agent and quality metrics for dashboards, alerting, and per-tier SLOs
- [Agent Failure Diagnosis](concepts/agent-failure-diagnosis.md) (8.2) — A top-down method, the MAST multi-agent taxonomy, replay, and engine-level failures
- [Model and Data Drift Monitoring](concepts/model-and-data-drift-monitoring.md) (8.1, 8.4) — Classical drift types translated to LLM agents, where ground truth is usually missing
- [Audit Trails and Traceability](concepts/audit-trails-and-traceability.md) (9.1, 8.5) — What to log, identity propagation, Relay's audit features, and AI Act Articles 12 and 19
- [NeMo Relay](components/nemo-relay.md) (8.1, 8.2, 8.5) — The toolkit's observability successor: scope stack, fail-closed middleware, and ATOF/ATIF/OTLP export

### Evaluation and improvement
- [Agent Evaluation Pipelines](concepts/agent-evaluation-pipelines.md) (3.1, 3.2, 3.5, 2.6) — Tasks, trials and graders; trajectory scoring and tool-choice checks; pass@k vs pass^k; capability versus regression suites
- [LLM-as-Judge](concepts/llm-as-judge.md) (3.1, 3.5) — Judge agreement with humans, known biases, calibration, and the toolkit's judge metrics
- [RAG Evaluation and Ragas](concepts/rag-evaluation-ragas.md) (3.1, 3.2) — Faithfulness, context recall and precision, NVIDIA's Ragas metrics, and failure attribution
- [Agent Benchmarks](concepts/agent-benchmarks.md) (3.1, 3.2) — SWE-bench, τ-bench, GAIA, AgentBench and BFCL, what each measures, and their limits
- [Synthetic Data for Evaluation](concepts/synthetic-data-for-evaluation.md) (3.1, 3.3) — Generated test sets and simulated users, their biases, and NeMo Data Designer
- [Agent Profiling and Result Analysis](concepts/agent-profiling-and-result-analysis.md) (3.2, 3.4, 3.5) — The toolkit profiler's outputs, a seven-step procedure from scores to a decision, and error bars on evals
- [Tuning Agents for Accuracy, Latency and Cost](concepts/agent-tuning-accuracy-latency-cost.md) (3.4, 3.5, 2.6) — Fixing tool choice from trajectories, sampling, automated prompt optimisation, routing (Switchyard), distillation and test-time compute
- [Feedback Loops and the Data Flywheel](concepts/feedback-loops-and-data-flywheel.md) (3.3, 3.5, 10.2) — Kinds of human and machine feedback, making it trainable, NVIDIA's flywheel, and its risks
- [Agent Release Management](concepts/agent-release-management.md) (4.2, 8.3, 8.4) — Versioned prompts and model pins with a trace manifest, eval gates in CI, shadow and canary rollout, post-release regression re-runs and rollback
- [NeMo Evaluator](components/nemo-evaluator.md) (3.1, 3.2) — NVIDIA's benchmark framework and its `nel compare` significance test and `nel gate` release decision; lineage from the toolkit's `nat eval`
- [NeMo Helix](components/nemo-helix.md) (3.1, 3.4, 3.5, 9.x) — Self-hosted control plane succeeding the toolkit's eval and optimiser: agent runners, optimisation strategies, gateway with routing and rails, garak, fine-tuning; pre-1.0
- [NVIDIA Data Flywheel Blueprint](components/nvidia-data-flywheel-blueprint.md) (8.4) — Distilling to cheaper models from production logs; deprecated April 2026

## Governance and compliance
- [EU AI Act](concepts/eu-ai-act.md) (9.5) — Risk tiers, GPAI duties, and the application timeline as amended by the 2026 Digital Omnibus
- [AI Risk Frameworks and Standards](concepts/ai-risk-frameworks-and-standards.md) (9.5, 9.1) — NIST AI RMF and its GenAI profile, ISO/IEC 42001, and the IEEE 7000 series
- [FDA AI-Enabled Medical Devices](concepts/fda-ai-enabled-medical-devices.md) (9.5) — The FDA's device list, GMLP and PCCP principles, and draft lifecycle guidance status
- [Model and Dataset Licensing](concepts/model-and-dataset-licensing.md) (9.5) — NVIDIA Open Model, Llama and Gemma terms, Apache 2.0, and how derived licences stack
