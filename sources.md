# Sources

Extracted from the NVIDIA-Certified Professional: Agentic AI Exam Study Guide (`sources/nvidia-study-guide/study-guide.pdf`, Sep 2025 edition). Status: `pending` | `ingested` | `thin` (fetched, no usable content) | `failed`.

Link titles are matched by position to the guide's reading lists. A link appears under the first section that cites it; later repeats are listed as "also cited".

**Weight discrepancy:** the guide gives Deployment and Scaling 5% and Run, Monitor, and Maintain 7%. `notes.md` (from the NVIDIA web page) gives 13% and 5%. The guide's weights sum to 87%, so one of them is likely stale. Confirm before relying on either.

**Course ID discrepancy:** the guide links courses as `C-FX-*` (e.g. Deploying RAG Pipelines is `C-FX-18`), while the certification web page links `S-FX-*` (self-paced) IDs.

## 1. Agent Architecture and Design (15%)

Objectives: 1.1 UIs for human-agent interaction · 1.2 reasoning/action frameworks (ReAct) · 1.3 agent-to-agent protocols · 1.4 short/long-term memory · 1.5 multi-agent orchestration · 1.6 logic trees, prompt chains, stateful orchestration · 1.7 knowledge graphs · 1.8 adaptability and scalability

| Status | Source | URL |
|---|---|---|
| thin | Course: Building Agentic AI Applications with LLMs — **page has only a stale 2023 workshop listing** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+C-FX-25+V1 |
| ingested | Agentic AI in the Factory (NVIDIA whitepaper) | https://docs.nvidia.com/ai-enterprise/planning-resource/ai-factory-white-paper/latest/agentic-ai-in-the-factory.html |
| failed | Building Autonomous AI with NVIDIA Agentic NeMo (Medium) — **fetch returned 403** | https://medium.com/@zbabar/building-autonomous-ai-with-nvidia-agentic-nemo-e2992ae58cea |
| ingested | Three Building Blocks for AI Virtual Assistants for Customer Service with an NVIDIA NIM Agent Blueprint | https://developer.nvidia.com/blog/three-building-blocks-for-creating-ai-virtual-assistants-for-customer-service-with-an-nvidia-nim-agent-blueprint/ |
| ingested | Agentic AI: Towards Autonomous Artificial Intelligence Agents (arXiv 2502.05439) — **LINK MISMATCH: resolves to a financial-services model-risk crews paper, not "Agentic AI: Towards Autonomous AI Agents"** | https://arxiv.org/abs/2502.05439 |
| failed | Catch Me If You Can: A Multi-Agent Framework for Financial Fraud Detection (Medium) — **fetch returned 403** | https://medium.com/@mariaprokofieva/catch-me-if-you-can-a-multi-agent-framework-for-financial-fraud-detection-03faa7caba7c |
| ingested | What Are Multi-Agent Systems? (NVIDIA glossary) | https://www.nvidia.com/en-us/glossary/multi-agent-systems/ |

## 2. Agent Development (15%)

Objectives: 2.1 prompts and dynamic chains · 2.2 multimodal models · 2.3 custom tools/APIs · 2.4 error handling (retry, graceful failure) · 2.5 conversation flows with streaming · 2.6 refine decision-making

| Status | Source | URL |
|---|---|---|
| thin | Course: Building RAG Agents With LLMs — **page has only a stale 2023 workshop listing** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+S-FX-15+V1 |
| ingested | Optimization — NVIDIA Triton Inference Server | https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/optimization.html |
| ingested | NVIDIA Agent Intelligence Toolkit Overview (1.1.0) — **now NeMo Agent Toolkit, DEPRECATED Oct 2026 (EOL end of 2026); successors NeMo Relay/Helix/Fabric** | https://docs.nvidia.com/aiqtoolkit/latest/index.html |
| ingested | An Introduction to LLMs: Prompt Engineering and P-Tuning (NVIDIA blog) | https://developer.nvidia.com/blog/an-introduction-to-large-language-models-prompt-engineering-and-p-tuning/ |
| thin | Building Multimodal AI RAG With LlamaIndex, NVIDIA NIM, and Milvus (video; link inferred by position) — **title only; no transcript** | https://www.youtube.com/watch?v=NaT5Eo97_I0 |
| failed | Design Considerations of Advanced Agentic AI for Real-World Applications (Medium) — **fetch returned 403** | https://medium.com/@official.indrajit.kar/design-considerations-of-advanced-agentic-ai-for-real-world-applications-c5ef317ba1e1 |
| ingested | Transient Fault Handling (Azure Architecture Center) | https://learn.microsoft.com/en-us/azure/architecture/best-practices/transient-faults |
| ingested | Circuit Breaker Pattern (Azure Architecture Center) | https://learn.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker |
| ingested | Retry Pattern (Azure Architecture Center) | https://learn.microsoft.com/en-us/azure/architecture/patterns/retry |

## 3. Evaluation and Tuning (13%)

Objectives: 3.1 evaluation pipelines and benchmarks · 3.2 compare across tasks/datasets · 3.3 structured user feedback · 3.4 tune parameters (accuracy vs latency) · 3.5 analyze results for targeted optimization

| Status | Source | URL |
|---|---|---|
| thin | Course: Evaluating RAG and Semantic Search Systems — **page has only a stale 2023 workshop listing** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+S-FX-32+V1 |
| ingested | Powering the Next Generation of AI Agents (NVIDIA AI) — **page now titled "AI Agents: Built to Reason, Plan, Act"** | https://www.nvidia.com/en-us/ai/ |
| ingested | NVIDIA Agent Intelligence Toolkit Tutorials — **MISMATCH: resolves to the NeMo Agent Toolkit overview page** | https://docs.nvidia.com/aiqtoolkit/latest/tutorials/index.html |
| ingested | NVIDIA Agent Intelligence Toolkit FAQ — **MISMATCH: resolves to the NeMo Agent Toolkit overview page** | https://docs.nvidia.com/aiqtoolkit/latest/resources/faq.html |
| ingested | Launching the Agent Toolkit API Server and UI (1.1) | https://docs.nvidia.com/nemo/agent-toolkit/1.1/quick-start/launching-ui.html |
| ingested | NVIDIA NeMo Agent Toolkit (GitHub) | https://github.com/NVIDIA/NeMo-Agent-Toolkit |
| failed | Agentic AI: The Next Big Thing in Artificial Intelligence (Oracle) — **fetch returned 403** | https://www.oracle.com/artificial-intelligence/agentic-ai/ |
| ingested | Agentic AI: The Top 5 Challenges and How to Overcome Them (Confluent) | https://www.confluent.io/blog/agentic-ai-the-top-5-challenges-and-how-to-overcome-them/ |
| ingested | AI Agents for Beginners — Production Patterns (Microsoft) | https://microsoft.github.io/ai-agents-for-beginners/10-ai-agents-production/ |
| ingested | Navigating the Challenges: 5 Common Pitfalls in Agentic AI Adoption (CapTech) | https://www.captechconsulting.com/articles/navigating-the-challenges-5-common-pitfalls-in-agentic-ai-adoption |

## 4. Deployment and Scaling (5% in guide; 13% in notes.md)

Objectives: 4.1 production multi-agent deployment · 4.2 MLOps CI/CD, monitoring, governance · 4.3 profile under distributed load · 4.4 Docker/Kubernetes with load balancing · 4.5 cost vs availability

| Status | Source | URL |
|---|---|---|
| thin | Course: Deploying RAG Pipelines for Production at Scale — **page is a generic stub; never names the course** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+C-FX-18+V1 |
| ingested | NVIDIA TensorRT-LLM (GitHub) | https://github.com/NVIDIA/TensorRT-LLM |
| ingested | Measure and Improve AI Workload Performance With DGX Cloud Benchmarking | https://developer.nvidia.com/blog/measure-and-improve-ai-workload-performance-with-nvidia-dgx-cloud-benchmarking/ |
| ingested | Kubernetes Glossary (NVIDIA) — **page titled "Kubernetes: An Overview of Container Orchestration"; Nsight Systems page only partly read** | https://www.nvidia.com/en-us/glossary/kubernetes/ |
| ingested | NVIDIA Nsight Systems | https://developer.nvidia.com/nsight-systems |
| ingested | Kube Prometheus for GPU Telemetry | https://docs.nvidia.com/datacenter/cloud-native/gpu-telemetry/latest/kube-prometheus.html |
| ingested | Scaling LLMs With NVIDIA Triton and TensorRT-LLM Using Kubernetes | https://developer.nvidia.com/blog/scaling-llms-with-nvidia-triton-and-nvidia-tensorrt-llm-using-kubernetes/ |
| ingested | TensorRT-LLM Performance Analysis | https://nvidia.github.io/TensorRT-LLM/performance/perf-analysis.html |

Also cited: Agentic AI in the Factory (section 1).

## 5. Cognition, Planning, and Memory (10%)

Objectives: 5.1 short/long-term memory · 5.2 reasoning (chain-of-thought, task decomposition) · 5.3 planning for multi-step decisions · 5.4 stateful orchestration · 5.5 adapt reasoning from feedback

| Status | Source | URL |
|---|---|---|
| thin | Course: Building RAG Agents With LLMs (alternate `C-FX-15` link) — **page has only a stale 2023 workshop listing** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+C-FX-15+V1 |
| ingested | NVIDIA NeMo | https://www.nvidia.com/en-us/ai-data-science/products/nemo/ |
| ingested | Large Language Models Are In-Context Learners (arXiv 2310.10501) — **LINK MISMATCH: resolves to the NeMo Guardrails paper, not "LLMs Are In-Context Learners" (likely GPT-3, 2005.14165)** | https://arxiv.org/abs/2310.10501 |
| ingested | NeMo RL Documentation | https://docs.nvidia.com/nemo/rl/latest/index.html |
| ingested | Jamba 1.5 LLMs Leverage Hybrid Architecture | https://developer.nvidia.com/blog/jamba-1-5-llms-leverage-hybrid-architecture-to-deliver-superior-reasoning-and-long-context-handling |
| ingested | Understanding the Planning of LLM Agents: A Survey (HTML, arXiv 2504.15965v1; link inferred by position) — **LINK MISMATCH: is a memory survey ("From Human Memory to AI Memory"); the planning survey is 2402.02716** | https://arxiv.org/html/2504.15965v1 |
| ingested | AI Agent Memory (IBM) | https://www.ibm.com/think/topics/ai-agent-memory |
| failed | MCP Agent Memory Types, Management, Implementation (BytePlus) — **URL now redirects to an unrelated ModelArk page** | https://www.byteplus.com/en/topic/542179?title=mcp-agent-memory-types-management-implementation |
| ingested | Understanding the Planning of LLM Agents: A Survey (arXiv 2402.02716) | https://arxiv.org/abs/2402.02716 |

## 6. Knowledge Integration and Data Handling (10%)

Objectives: 6.1 retrieval pipelines (RAG, hybrid) · 6.2 vector DB tuning · 6.3 ETL for enterprise data · 6.4 data quality and preprocessing · 6.5 real-time structured/unstructured access

| Status | Source | URL |
|---|---|---|
| thin | Course: Adding New Knowledge to LLMs — **page is a generic stub** | https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+C-FX-26+V1 |
| failed | How to Make Your LLM More Accurate with RAG and Fine-Tuning (Towards Data Science) — **fetch returned 403** | https://towardsdatascience.com/how-to-make-your-llm-more-accurate-with-rag-fine-tuning/ |

## 7. NVIDIA Platform Implementation (7%)

Objectives: 7.1 NeMo Guardrails · 7.2 NIM microservices · 7.3 NeMo Agent Toolkit · 7.4 TensorRT-LLM and Triton for latency · 7.5 multimodal input pipelines on NVIDIA hardware

| Status | Source | URL |
|---|---|---|
| ingested | Best Practices — NVIDIA TensorRT Documentation | https://docs.nvidia.com/deeplearning/tensorrt/latest/performance/best-practices.html |
| ingested | Batchers — Triton Inference Server | https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html |
| ingested | Triton Inference Server Backend | https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/backend/README.html |
| ingested | NeMo Guardrails (NVIDIA Developer) — **developer page redirects to the docs overview** | https://developer.nvidia.com/nemo-guardrails |
| ingested | NVIDIA NeMo Guardrails (GitHub) — **repo is now NVIDIA-NeMo/Guardrails** | https://github.com/NVIDIA/NeMo-Guardrails |
| ingested | Performance Tuning Guide — NeMo Framework — **MISMATCH: page is "Performance Tuning Guide — Megatron Bridge" (training side)** | https://docs.nvidia.com/nemo-framework/user-guide/latest/performance/performance-guide.html |
| ingested | Best Practices (data curation) — NeMo Framework — **/latest/ URL is 404; read the 24.12 copy** | https://docs.nvidia.com/nemo-framework/user-guide/latest/datacuration/bestpractices.html |
| ingested | NVIDIA NeMo Agent Toolkit (developer page) | https://developer.nvidia.com/nemo-agent-toolkit |
| ingested | NVIDIA AIQ Toolkit docs — **serves NeMo Agent Toolkit 1.9 with the deprecation banner** | https://docs.nvidia.com/aiqtoolkit/index.html |
| ingested | NVIDIA AIQToolkit (GitHub) | https://github.com/NVIDIA/AIQToolkit |
| ingested | Mastering LLM Techniques: Inference Optimization | https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/ |
| ingested | Deploy Inference Workloads With NVIDIA NIM (Run:ai) — **URL redirects to a landing page; current page found via run-ai-docs.nvidia.com** | https://docs.run.ai/v2.20/Researcher/workloads/inference/nim-inference/ |
| ingested | How to Use the NVIDIA Llama Nemotron API for Advanced AI Agents (Apidog) — **low reliability: placeholder endpoint, not the OpenAI-compatible NIM API** | https://apidog.com/blog/use-nvidia-llama-nemotron-api/ |
| ingested | How to Deploy Llama-3.1-Nemotron-70B-Instruct on a Cloud VM (DEV) — **low reliability: does not use NIM, vLLM or TensorRT-LLM** | https://dev.to/nodeshiftcloud/how-to-deploy-llama-31-nemotron-70b-instruct-on-a-virtual-machine-in-the-cloud-3777 |
| ingested | AI Agents Blueprint: Designing Foundation Models and Agents for the Next Wave of AI — **MISMATCH: redirects to "Chat With Your Enterprise Data Through Open-Source AI-Q NVIDIA Blueprint"** | https://blogs.nvidia.com/blog/ai-agents-blueprint/ |
| ingested | Improve AI Code Generation Using NVIDIA Agent Intelligence Toolkit — **retitled "NeMo Agent Toolkit"** | https://developer.nvidia.com/blog/improve-ai-code-generation-using-nvidia-agent-intelligence-toolkit/ |
| ingested | NVIDIA NeMo: A Scalable Generative AI Framework (GitHub) | https://github.com/NVIDIA/NeMo |

## 8. Run, Monitor, and Maintain (7% in guide; 5% in notes.md)

Objectives: 8.1 dashboards and reliability metrics · 8.2 logs/errors/anomalies for root cause · 8.3 benchmark against prior versions · 8.4 automated tuning, retraining, versioning · 8.5 uptime, transparency, trust

| Status | Source | URL |
|---|---|---|
| ingested | What Is AI Agent Evaluation? (IBM) — **loosely relevant: says little about continuous production monitoring** | https://www.ibm.com/think/topics/ai-agent-evaluation |
| ingested | Log, Trace, and Monitor (LangChain / Portkey) — **URL moved; content matches** | https://python.langchain.com/docs/integrations/providers/portkey/logging_tracing_portkey/ |
| ingested | Time-Weighted Retriever (LangChain JS) — **link redirects to a generic overview; real page found under docs.langchain.com/oss/javascript; loosely relevant (retrieval freshness, not monitoring)** | https://js.langchain.com/docs/integrations/retrievers/time-weighted-retriever/ |
| ingested | Troubleshooting (TensorRT-LLM) — **loosely relevant: build-time debugging for the legacy TensorRT-engine workflow** | https://nvidia.github.io/TensorRT-LLM/reference/troubleshooting.html |
| thin | LangChain Tracing Concepts — **redirects to a generic LangChain overview** | https://python.langchain.com/docs/concepts/tracing/ |
| thin | LangChain Structured Outputs Concepts — **redirects to a generic LangChain overview** | https://python.langchain.com/docs/concepts/structured_outputs/ |
| ingested | LangSmith Evaluation: Rate Limiting — **URL moved; content matches** | https://docs.smith.langchain.com/evaluation/how_to_guides/rate_limiting |
| ingested | A Guide to Monitoring Machine Learning Models in Production (NVIDIA blog) | https://developer.nvidia.com/blog/a-guide-to-monitoring-machine-learning-models-in-production |
| failed | Monitoring ML Models in Production: Track Data Quality and Integrity (Towards Data Science) — **fetch returned 403** | https://towardsdatascience.com/monitoring-machine-learning-models-in-production-how-to-track-data-quality-and-integrity-391435c8a299/ |

## 9. Safety, Ethics, and Compliance (5%)

Objectives: 9.1 security and audit trails · 9.2 compliance guardrails · 9.3 bias and toxicity · 9.4 layered safety (filters, escalation) · 9.5 licensing and regulatory compliance

| Status | Source | URL |
|---|---|---|
| ingested | Building Safer LLM Apps With LangChain Templates and NeMo Guardrails | https://developer.nvidia.com/blog/building-safer-llm-apps-with-langchain-templates-and-nvidia-nemo-guardrails/ |
| ingested | NVIDIA NeMo Guardrails (filtered listing) — **redirects to docs.nvidia.com** | https://developer.nvidia.com/nemo-guardrails?sortBy=developer_learning_library%2Fsort%2Ffeatured_in.nemo_guardrails%3Adesc%2Ctitle%3Aasc&hitsPerPage=12 |
| failed | AI and ML in Software as a Medical Device (FDA) — **404; read the FDA AI-enabled medical device list page instead** | https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-and-machine-learning-software-medical-device |
| failed | Proposal for a Regulation on AI (EU AI Act, 2021 proposal) — **empty body; this is the superseded 2021 proposal, the adopted text is Regulation (EU) 2024/1689** | https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:52021PC0206 |
| ingested | Ethically Aligned Design (IEEE) — **redirects to the IEEE Standards autonomous-systems initiative page** | https://ethicsinaction.ieee.org/ |
| ingested | Securing Generative AI Deployments With NVIDIA NIM and NeMo Guardrails | https://developer.nvidia.com/blog/securing-generative-ai-deployments-with-nvidia-nim-and-nvidia-nemo-guardrails/ |
| ingested | Metrics for Agentic AI (Ragas) — **page title is "Agent and Tool-Use Metrics"** | https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/agents/ |
| ingested | AI for Regulatory Compliance (LeewayHertz) — **vendor marketing, low reliability; first 100k of 154k characters read** | https://www.leewayhertz.com/ai-for-regulatory-compliance/#core-technologies |
| failed | Responsible AI Revisited (Medium) — **fetch returned 403** | https://medium.com/@adnanmasood/responsible-ai-revisited-critical-changes-and-updates-since-our-2023-playbook-0c1610d57f37 |

## 10. Human-AI Interaction and Oversight (5%)

Objectives: 10.1 user-in-the-loop UIs · 10.2 structured feedback loops · 10.3 transparency (explainability, traceability) · 10.4 human oversight and intervention

| Status | Source | URL |
|---|---|---|
| ingested | NVIDIA Agent Intelligence Toolkit (developer page) — **redirects to the deprecated NeMo Agent Toolkit docs** | https://developer.nvidia.com/agent-intelligence-toolkit |
| ingested | NVIDIA Data Flywheel Glossary | https://www.nvidia.com/en-us/glossary/data-flywheel/ |
| failed | AI Agents With Human-in-the-Loop (Medium) — **fetch returned 403** | https://cobusgreyling.medium.com/ai-agents-with-human-in-the-loop-f910d0c0384b |
| ingested | Human-in-the-Loop AI (HolisticAI) — **covers classic ML human review, not agent oversight** | https://www.holisticai.com/blog/human-in-the-loop-ai |
| ingested | Human-in-the-Loop Agentic AI Systems (OneReach.ai) | https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/ |
| failed | Aporia: AI Guardrails — **SSL handshake failure; no archived copy** | https://www.aporia.com/learn/ai-guardrails/ |
| ingested | Chain-of-Thought (CoT) Prompting (Codecademy) — **loosely related: a prompting tutorial that duplicates the chain-of-thought page** | https://www.codecademy.com/article/chain-of-thought-cot-prompting |

Also cited: Improve AI Code Generation Using NVIDIA Agent Intelligence Toolkit (section 7).

## Not a study source

- NVIDIA certification contact form: https://www.nvidia.com/en-us/learn/organizations/contact-us/
