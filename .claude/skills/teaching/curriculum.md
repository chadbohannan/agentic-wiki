# Curriculum

Goal: explain the core ideas of agentic AI, their trade-offs, and NVIDIA's implementations well enough to answer interview and NCP-AAI questions. Spine: the seven parts in `wiki/syntheses/agent-anatomy.md`.

## Current

Not started. First lesson: item 1.

## Up next

1. What an agent is: a model in a loop choosing tools (`syntheses/agent-anatomy.md`, opening). Workflow vs agent (`concepts/agent-architecture-patterns.md`).
2. The loop: call the model, run tool calls, append results, stop (`concepts/react-reasoning-action.md`). Stopping budgets and failure modes.
3. Tools: function calling, why tool design matters (`concepts/tool-calling-and-mcp.md`).
4. Context: the window as an attention budget (`concepts/context-engineering.md`).
5. Model: choosing, routing, and serving cost; why agents care about prefix caching (`concepts/agent-tuning-accuracy-latency-cost.md`, `concepts/llm-inference-optimization.md`).
6. Memory vs run state (`concepts/agent-memory.md`, `concepts/stateful-orchestration.md`).
7. Retrieval: RAG pipelines, and RAG vs fine-tuning (`concepts/rag-retrieval-pipelines.md`, `concepts/rag-vs-fine-tuning-vs-pretraining.md`).
8. Planning and reasoning (`concepts/llm-agent-planning.md`, `concepts/chain-of-thought-and-task-decomposition.md`).
9. Multi-agent orchestration and protocols: MCP, A2A, AG-UI (`concepts/multi-agent-orchestration.md`, `concepts/agent-protocols-mcp-a2a.md`).
9b. The three frameworks, one at a time, each against the same question ("who owns the loop, the state and the stopping rule?"): LangGraph (`components/langgraph.md`), OpenAI Agents SDK (`components/openai-agents-sdk.md`), CrewAI (`components/crewai.md`). Then compare them.
10. Environment and security: sandboxing, prompt injection, the lethal trifecta (`concepts/agent-security-threats-and-controls.md`). Note: the wiki calls this its thinnest part.
11. Guardrails and human oversight (`concepts/layered-safety-and-escalation.md`, `concepts/human-in-the-loop-patterns.md`, `components/nemo-guardrails.md`).
12. Observation: traces, metrics, failure diagnosis (`concepts/agent-observability-tracing.md`, `concepts/agent-failure-diagnosis.md`).
13. Evaluation: graders, trajectories, pass@k, LLM-as-judge (`concepts/agent-evaluation-pipelines.md`, `concepts/llm-as-judge.md`).
14. The NVIDIA stack: one request through the layers, and product status today (`syntheses/nvidia-agentic-stack.md`).
15. Deployment: NIM, Triton, Dynamo, Kubernetes (`components/nvidia-nim.md`, `components/nvidia-dynamo.md`, `concepts/kubernetes-gpu-inference-deployment.md`).
16. Governance: EU AI Act, risk frameworks (`concepts/eu-ai-act.md`, `concepts/ai-risk-frameworks-and-standards.md`).
17. Mock interview round: mixed questions drawn from the Interview Q&A sections. Use `syntheses/exam-blueprint.md` to find weak sections.

## Learned

(none yet)

## Notes on the student

- Carried over from a similar course: go slowly, one idea at a time, under 150 words. Go smaller when confused.
- Unknown yet: prior background in LLMs, Python, and NVIDIA tooling. Ask one calibrating question in the first lesson, then record the answer here.
- Parked tangents and wiki gaps found while teaching go here.
