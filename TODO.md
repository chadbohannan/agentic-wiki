# TODO

Actionable gaps collected from the ingest of sections 1-8 and 10 (2026-10-28). Items are grouped by what to do, with the likely first step. "Page" means a file under `wiki/`.

## Pending ingest work

- [x] Sections 1-10 ingested and merged (72 pages). `staging/sec-*.md` is kept as provenance.
- [x] Data Flywheel status checked against the README (2026-10-28): one notice, "deprecated ... no longer actively maintained", April 2026, no successor named. Both pages agree.
- [x] Relay-as-successor is now attributed to the toolkit's migration guide in `components/nemo-relay.md` (2026-10-28).
- [ ] Review the three overlapping trace/audit pages (`agent-observability-tracing`, `audit-trails-and-traceability`, `agent-transparency-and-traceability`) for duplication and consistent claims.

## Verify before relying on (claims that matter or are weakly supported)

- [ ] **EU AI Act.** Application dates are verified against the Commission's AI Act Service Desk timeline (as amended by the Digital Omnibus). Omnibus number (2026/1744), OJ date (2026-07-24) and entry into force (2026-07-27) now agree across several law-firm summaries (2026-10-28); EUR-Lex still unread, and sources disagree on the Article 50(2) transition scope. Article 14/50 wording was read from an unofficial reproduction, so quote EUR-Lex.
- [x] **Kubernetes HPA defaults** (15 s sync, 10% tolerance, 300 s scale-down window) confirmed on kubernetes.io (2026-10-28).
- [ ] **Dynamo performance claims** (30x, 7x, 2x, 80%) are vendor-reported. Record the stated hardware and model conditions or drop the numbers.
- [ ] **Snippet-only claims** flagged in pages: Nemotron 3 family, Helix release dates and `nemo agents optimize` subcommands, Triton-to-Dynamo-Triton FAQ, Colang 2 migration guide, RRF paper (formula and k=60 now confirmed via Elasticsearch docs), `decay_rate` default of 0.01 in the time-weighted retriever, Switchyard figures, SpotServe, AIPerf migration guide. (Dynamo router flags and AIPerf blog confirmed 2026-10-28.) Fetch each primary page or soften the claim.
- [ ] **AI-Q repository identity.** The guessed repo URL returned `deep-researcher-agent`; confirm the repo name and model list, and check whether AI-Q has migrated off the deprecated NeMo Agent Toolkit (AI-Q docs list 2.2.1 as newest; LLM Router is itself deprecated in favour of Switchyard, per its README) (`components/ai-q-blueprint.md`).
- [ ] **NeMo Evaluator vs Helix.** How the new NeMo Evaluator relates to Helix and the older Evaluator microservice is unknown (`components/nemo-evaluator-and-helix-eval.md`).
- [ ] **Agent protocol versions.** MCP (current 2026-07-28) and A2A (1.0.0) are now verified against their specs. Still open: the "AutoGen in maintenance mode / Microsoft Agent Framework 1.0" claim, which rests on two 2026 blog snippets (`concepts/multi-agent-orchestration.md`); check Microsoft's docs.
- [ ] **OpenTelemetry `gen_ai.*` conventions.** Read the unread tails of the agent-span and metrics specs (`execute_tool` attributes, agent metrics, token usage) before stating metric names beyond those already listed. Mostly "Development" status, so re-check for renames.
- [ ] **Course IDs.** The guide uses `C-FX-*` and the web page `S-FX-*`; all course pages are empty stubs. Decide whether to ask NVIDIA or check the DLI catalog in a browser.
- [ ] **Exam weights.** Guide and web page disagree for Deployment (5% vs 13%) and Run/Monitor/Maintain (7% vs 5%); the guide's total is 87%. Note which the user wants to treat as authoritative.

## Content gaps by area

### Architecture and development (sections 1, 2)

- [ ] No primary source for reactive/deliberative/hybrid agents applied to LLMs beyond one arXiv snippet (2602.10479); read it.
- [ ] A2A spec read only to ~100k of 256k characters (relation to MCP, security sections unread).
- [ ] GraphRAG, Zep/Graphiti and LangGraph persistence claims are snippet-grade; read the papers and docs. No NVIDIA-specific knowledge-graph source found.
- [ ] Audio/speech modality (Riva ASR/TTS, NeMo voice agent) is uncovered; objective 2.2 and 7.5.
- [ ] NeMo Agent Toolkit eval/profiler/optimizer internals and API server/streaming behavior (largely moot after deprecation; capture only what Relay/Helix/Fabric inherit).
- [ ] NIM structured-generation and function-calling pages gave outlines only; read versioned pages (e.g. `docs.nvidia.com/nim/large-language-models/<version>/...`).
- [ ] OpenAI and Anthropic native structured-output docs were read only through secondary summaries.

### Cognition, planning, memory (section 5)

- [ ] Primary page not retrieved for Plan-and-Solve. (LLM+P abstract read 2026-10-28.)
- [x] ToT, GoT, Reflexion, Self-Refine, Voyager, self-consistency, least-to-most, ReWOO, Llama-Nemotron and test-time-compute numbers checked against abstracts (2026-10-28). [ ] Still from snippets: MemGPT; DeepSeek-R1's GRPO/R1-Zero details (abstract no longer names them); ReWOO module names; the Nemotron "detailed thinking" and `/no_think` toggle (NIM reasoning-model page returns no text).
- [ ] Context-rot and long-context-vs-RAG cost claims rest on practitioner blogs (Chroma study); find the primary study.
- [ ] Nemotron 3 reasoning toggle unverified; NeMo RL docs read only ~100k of 653k characters.
- [ ] Replace the guide's mislabeled references with the right ones on the pages: GPT-3 paper (2005.14165) for in-context learning; arXiv 2402.02716 for planning.

### Evaluation and tuning (section 3)

- [x] Ragas faithfulness, context recall and context precision definitions read; SWE-bench Verified details confirmed (500 samples, 93 annotators) (2026-10-28). [ ] BFCL category list still partial.
- [ ] Quantization, distillation, DSPy-vs-optimizer comparison, temperature/sampling guidance are `[inferred]`; find primary sources.
- [ ] Not researched: online evaluation / A-B statistics, LLM-judge bias mitigation beyond abstracts, cost-per-task definitions, Langfuse/Weave detail.
- [ ] NeMo Data Designer column types and judge columns; "User Sim" product not found.

### Deployment (section 4)

- [ ] Dynamo router, planner, KVBM, Grove and Kubernetes (`DynamoGraphDeployment`) subpages unread; fetch `docs.nvidia.com/dynamo/latest/<page>.md`.
- [ ] AI Factory white paper "Deployment Strategies", "AgentOps", "GitOps Controller", "Observability" sections unread.
- [ ] NIM Operator HPA example with a KV-cache metric; GPU Operator/DRA, MIG vs time-slicing, KServe.
- [ ] Kubernetes spot/preemption drain mechanics (terminationGracePeriod, preStop, PodDisruptionBudget) are `[inferred]`.
- [ ] Nsight Systems page read ~100k of ~498k characters; DCGM page last ~8k unread (dcgm-exporter details).
- [ ] Objective 4.2 (MLOps/CI-CD/governance): no primary NVIDIA source; the page is blog-grade plus synthesis.

### Knowledge integration (section 6)

- [ ] NVIDIA primary source for text-to-SQL agents; try the Data Designer recipe `.../recipes/code-generation/nemotron-super-text-to-sql`.
- [ ] NeMo Curator PII redaction unconfirmed; Curator relevance to RAG corpora; Data Designer details.
- [ ] PQ/IVF-PQ and DiskANN papers, ColBERT/late interaction (Nemotron ColEmbed), embedding NIM `input_type` and Matryoshka support.
- [ ] Blueprint releases after 2.6.0; cuVS CAGRA-to-HNSW docs; Blueprint accuracy benchmarks.
- [ ] CDC-to-vector-index patterns and streaming freshness (Kafka/Flink); RAG security (ACL-aware retrieval, injection via documents).

### NVIDIA platform (section 7)

- [ ] Objective 7.5 (multimodal input pipelines) is the thinnest: Riva/speech NIMs, NIM VLM input limits, video preprocessing (VSS). Decide whether to split `multimodal-input-pipelines` out of `multimodal-agents`.
- [ ] NeMo Guardrails `config.yml` layout and rails syntax unverified (docs URLs 404); read the GitHub `docs/` tree; Colang 2.0 examples and execution order.
- [ ] NIM architecture overview page 404; NIM licensing/AI Enterprise terms; multi-LLM, LoRA and VLM specifics.
- [ ] Triton ensembles, `perf_analyzer`, `model_analyzer` claims are `[inferred]`; TensorRT-LLM release notes, NVFP4/FP8 KV details, NIM Operator and TensorRT-LLM version numbers.
- [ ] Current NeMo Curator best-practices page (only the 24.12 copy was read).
- [ ] Check whether NVIDIA repos (NeMo Guardrails, TensorRT-LLM) are indexed in `code-rag` (`list_repos`); if so, ground the component pages in code.

### Run, monitor, maintain (section 8)

- [ ] No primary source on LLM-specific SLOs/error budgets, canary/shadow/A-B for agents, public status and transparency practice, or incident post-mortems; those parts are `[inferred]`.
- [ ] NeMo Relay: license, install, migration path from the toolkit, Phoenix/Langfuse compatibility, ATOF/ATIF schemas. Version was v0.11.0 (pre-1.0).
- [ ] MAST failure taxonomy per-category counts; Langfuse/Phoenix licensing and self-hosting; LangSmith online evaluators page.

### Safety, ethics, compliance (section 9)

- [ ] EUR-Lex pages and the Council press release could not be read (empty body); read the adopted Regulation (EU) 2024/1689 and the Omnibus text directly.
- [ ] OWASP Agentic Top 10 names (ASI01-ASI10) now consistent across several third-party summaries (2026-10-28); the OWASP PDF itself is still unread.
- [x] GLiNER-PII model card read (2026-10-28). [ ] MCPTox 36.5% attack-success rate still unverified.
- [ ] FDA's January 2025 draft lifecycle guidance: still listed as draft in latest sources; the PCCP guidance is final (Dec 2024). Confirm on fda.gov. The guide's FDA link is dead; the page uses the AI-enabled device list instead.
- [ ] ISO/IEC 42001 now described from standards bodies and certifiers (ISO page still 403); NIST AI 600-1 risk list, and the NeMo Guardrails Security Guidelines page (guessed URL 404).
- [ ] No bias/toxicity benchmark sources read (e.g. BBQ, RealToxicityPrompts, ToxiGen); add them to `concepts/bias-and-toxicity-mitigation.md`.
- [ ] Ground Guardrails configuration examples in code (`code-rag` or the GitHub `docs/` tree).
- [ ] Medium "Responsible AI Revisited" (403) and LeewayHertz (vendor marketing, ~100k of 154k read): treat as leads only.

### Human-AI interaction (section 10)

- [ ] NeMo Agent Toolkit "Interactive Workflows"/HITL mechanism and API Server/UI page (guessed path 404); what Relay/Helix/Fabric offer for HITL.
- [ ] AG-UI's own human-in-the-loop/interrupt docs; LangGraph Agent Inbox; OpenAI Agents SDK guardrails; Anthropic Agent SDK `canUseTool`.
- [ ] Confidence-based routing/calibration and automation-bias literature (human-factors studies) are `[inferred]`.
- [ ] Model/agent/system cards (Mitchell et al.; A2A AgentCard), NIST AI RMF, preference-data tooling (DPO pairs, Label Studio/Argilla, NeMo Customizer DPO).

## Source retries

- [ ] Medium and Towards Data Science posts that returned 403 (Cobus Greyling; zbabar; mariaprokofieva; indrajit.kar; TDS RAG-vs-fine-tuning; TDS ML monitoring). Try a browser-capable fetch (Claude in Chrome) or archive copies, but treat them as leads either way.
- [ ] Oracle agentic-AI page (403) and Aporia (SSL failure; Coralogix acquired Aporia in Dec 2024 per search titles only).
- [ ] BytePlus MCP agent-memory page (dead redirect): search for a replacement or drop from the queue.
- [ ] Re-fetch `docs.nvidia.com/aiqtoolkit/...` Tutorials and FAQ equivalents on the new docs tree (the old URLs resolve to the overview page).

## Wiki quality

- [x] Sections 1 and 2 pages, `agent-memory`, `stateful-orchestration` and `components/nemo-agent-toolkit.md` converted to prose with inline attribution (2026-10-28). Section 5 pages converted 2026-10-28. Sections 3, 4, 6, 7, 8, 9 and 10 (51 pages) converted 2026-10-28; no script markers or staging refs remain.
- [ ] Lint pass: `[web]` claims resting on search snippets, stale product names, every `ingested` row in `sources.md` cited by a page, orphans.
- [x] All `1.S4`-style refs and `staging/` table rows replaced with descriptive source links (2026-10-28).
- [ ] Write a few `syntheses/` pages that tie the material together for interviews: architecture-to-production walkthrough, NVIDIA stack map (what each product does and what replaced what), "what changed since the study guide".
- [ ] Decide whether to keep `staging/` after merging; if kept, mention it in the log as provenance.

### Left over from the sections 1-2 cleanup

- [ ] **NIM structured-generation and function-calling docs** returned no text to WebFetch (JavaScript-rendered), so `concepts/structured-outputs.md` and `concepts/tool-calling-and-mcp.md` still rest on search summaries for NIM parameters and the `tool_choice` list. Read them in a browser.
- [ ] **LangGraph durability modes** ("exit", "async", "sync") and the tasks wrapper could not be confirmed on the persistence or durable-execution pages; read the checkpointers guide (`concepts/stateful-orchestration.md`).
- [ ] **LangGraph v1.2 typed-projection streaming API** and the `message.text`/`message.reasoning` names: read the event-streaming page (`concepts/streaming-and-realtime-agent-ux.md`).
- [ ] **Mem0, MemGPT, CoALA and the 2603.07670 survey** (Reflexion and Voyager abstracts read 2026-10-28) were cited in `concepts/agent-memory.md` from earlier snippets and were not re-read in this pass.
- [ ] **Anthropic context-engineering post** is dated from a search snippet; read it in full (`concepts/prompt-engineering-and-chains.md`).
- [ ] **arXiv 2602.10479** (reactive/deliberative mapping onto LLM agents) is snippet-only (`concepts/agent-architecture-patterns.md`).

### Decision models (added 2026-10-28)

- [ ] Find TypeSafe's own Jev API reference and model docs; field names and limits on `concepts/decision-models.md` come from third-party guides and the Clef docs.
- [ ] Confirm NVIDIA's role in CLM-8B (press says Stanford and NVIDIA; the model card names no affiliations) and whether CLM-35B shipped.
- [ ] Read the Strands Decider launch post and resolve its JevBench score (72% vs 0.762).
- [ ] Re-check the category in a month: benchmarks are all vendor-reported and the field is six weeks old.

### Left over from the sections 3-10 cleanup

- [ ] **AG-UI human-in-the-loop page** still 404; `agent-oversight-ui-design.md` says so.
- [ ] **NeMo Guardrails `IORails` and `nemo-guardrails-relay-plugin`** appear in the README but are unexplained; read the docs before describing them (`components/nemo-guardrails.md`).
- [ ] **BIRD text-to-SQL scores** (Gemini-SQL2 80.04%, DataGallery 82.39%) are press-reported; check the official leaderboard (`concepts/structured-data-access-text-to-sql.md`).
- [ ] **Helix current version**: docs list 0.8.0 as newest; release notes for 0.8 were not read.
- [ ] **Interview-level synthesis**: the three trace/audit pages were rewritten separately; still review them together for overlap (see the item under Pending ingest work).
