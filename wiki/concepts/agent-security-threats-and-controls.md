# Agent Security Threats and Controls

An LLM agent turns text into actions. The security question therefore shifts from "can the model be tricked into saying something bad" to "what can a tricked model do". The controls that actually limit damage are structural (least privilege, sandboxing, gating actions in code); classifiers help, but only as one layer.

## Threat catalogues

The OWASP Top 10 for LLM Applications 2025 lists **[web]**:
- LLM01 prompt injection;
- LLM02 sensitive information disclosure;
- LLM03 supply chain;
- LLM04 data and model poisoning;
- LLM05 improper output handling;
- LLM06 excessive agency;
- LLM07 system prompt leakage;
- LLM08 vector and embedding weaknesses;
- LLM09 misinformation;
- LLM10 unbounded consumption.

OWASP's separate Top 10 for Agentic Applications for 2026 was released on 2025-12-09. It extends that list for agents. The release date is OWASP's; the entry names come from third-party summaries whose wording varies slightly **[web]**:
- ASI01 agent goal hijack;
- ASI02 tool misuse and exploitation;
- ASI03 identity and privilege abuse;
- ASI04 agentic supply-chain vulnerabilities;
- ASI05 unexpected code execution;
- ASI06 memory and context poisoning, which targets the long-term stores in [Agent Memory](agent-memory.md);
- ASI07 insecure inter-agent communication;
- ASI08 cascading failures;
- ASI09 exploitation of human trust in the agent;
- ASI10 rogue agents.

## How agents get attacked

Direct injection comes from the user. Indirect injection arrives in content the agent retrieves, such as web pages, emails, tickets, documents and tool results. Greshake et al. (arXiv 2302.12173) demonstrated this against real LLM-integrated applications: such applications "blur the line between data and instructions", so an attacker can plant prompts in data likely to be retrieved, and processing them can act like arbitrary code execution **[web]**. That is why RAG and the tools in [Tool Calling and MCP](tool-calling-and-mcp.md) widen the attack surface.

Simon Willison's "lethal trifecta" (2025-06-16) says that an agent combining three things can be steered into leaking data:
- access to private data;
- exposure to untrusted content;
- a way to communicate externally.

He notes that guardrail products cannot reliably prevent this. His advice is to avoid combining all three, or to constrain the agent so that untrusted input it has read cannot trigger consequential actions **[web]**. Tool poisoning hides instructions in a tool's description or schema. MCPTox (arXiv 2508.14925) measured it on 45 live MCP servers and 353 real tools across 20 agents: the most vulnerable, o1-mini, was compromised in 72.8% of cases, no model refused more than 3% of attacks, and more capable models were often more susceptible because the attack exploits their better instruction-following **[web]**.

Bounded outputs are no defence. HiddenLayer reported in October 2026 that TypeSafe's Jev [decision model](decision-models.md), whose outputs are restricted to typed choices, was still swayed by prompt injection and TokenBreak-style perturbations, including an injected ad that redirected a browser agent. Constraining the output space does not make the input trustworthy. The finding is a single security vendor's, reported by InfoWorld **[web]**.

## Controls, from most to least reliable

1. **Reduce capability.** Grant least privilege and minimal scope; for databases this is the case for parameterised tools over free-form SQL made in [Structured Data Access and Text-to-SQL](structured-data-access-text-to-sql.md). The MCP security best practices recommend progressive least-privilege scopes: start minimal, step up incrementally, and log each elevation with a correlation ID. They warn that omnibus scopes widen the blast radius and blur the audit trail. OWASP's "excessive agency" describes agents given more autonomy or permissions than they need **[web]**.
2. **Sandbox.** The same MCP guidance recommends running local servers (see [Agent Protocols: MCP, A2A, AG-UI](agent-protocols-mcp-a2a.md)) in sandboxes with minimal default privileges, and showing the exact start-up command before running it. It prefers the stdio transport for local servers, and routes server-side clients through egress proxies that block private IP ranges against server-side request forgery **[web]**.
3. **Gate actions in code, not in the prompt.** Validate tool arguments, use allowlists, and require approval for consequential actions. [NeMo Guardrails](../components/nemo-guardrails.md) is NVIDIA's hook for this: tool-call and tool-result rails validate the model's tool calls and their results, and execution rails wrap custom actions **[web]**.
4. **Use proper identity.** MCP servers must not accept tokens that were not issued to them (no token passthrough), which also protects the [audit trail](audit-trails-and-traceability.md) **[web]**.
5. **Add probabilistic detection as one layer, never the only one.** This means jailbreak and injection classifiers and content-safety models, such as those in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md).

On the NVIDIA side, the Guardrails docs map jailbreak protection to input rails and agent security to execution rails. NemoGuard JailbreakDetect is a NIM classifier, and garak is NVIDIA's open-source scanner for probing an endpoint before release **[web]**.

Detection layers add latency and false positives. Structural controls, such as removing a leg of the trifecta or removing write tools, cost capability. Human approval costs throughput and suffers from approval fatigue: Anthropic reports that Claude Code users accepted 93% of manual permission prompts, and built auto mode's classifier because people stop paying close attention to what they approve **[web]**.

## Interview Q&A

**What is indirect prompt injection, and why do agents make it worse?** Malicious instructions arrive in data the agent reads, such as a page, an email or a tool result, rather than from the user. Agents have tools, so the injected text can trigger actions or exfiltration.

**What is the lethal trifecta, and how do you design around it?** Private data, untrusted content and external communication in one agent. Remove at least one leg, or block consequential actions once untrusted input has been read **[web]**.

**Why is a guardrail classifier not a sufficient defence?** It is probabilistic and can be bypassed, and no one knows how to prevent injection with full reliability. Pair it with least privilege, sandboxing and gating actions in code **[web]**.

**Name three MCP-specific attacks and a mitigation for each.** A confused deputy via an OAuth proxy: per-client consent and exact redirect-URI matching. Token passthrough: reject tokens not issued to the server. Compromise of a local server: consent that shows the exact command, plus a sandbox **[web]**.

**What does ASI01 in OWASP's agentic list mean?** Agent goal hijack: an attacker redirects the agent's objective, usually through text it reads **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/llm-top-10/) |
| 2026-10-28 | [OWASP Top 10 for Agentic Applications (Cycode summary)](https://cycode.com/blog/owasp-top-10-agentic-applications/) |
| 2026-10-28 | [The lethal trifecta (Simon Willison)](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) |
| 2026-10-28 | [MCP security best practices](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: engine feature support](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/reference/engine-feature-support.mdx) |
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
| 2026-10-28 | [HiddenLayer on Jev (InfoWorld)](https://www.infoworld.com/article/4236103/typesafes-jev-even-more-susceptible-to-attack-than-llms-says-security-vendor.html) |
| 2026-10-28 | [Not what you've signed up for: Indirect Prompt Injection (Greshake et al., arXiv 2302.12173)](https://arxiv.org/abs/2302.12173) |
| 2026-10-28 | [Claude Code auto mode (Anthropic Engineering, 2026-03-25)](https://www.anthropic.com/engineering/claude-code-auto-mode) |
| 2026-10-28 | [MCPTox: A Benchmark for Tool Poisoning Attack on Real-World MCP Servers (arXiv 2508.14925)](https://arxiv.org/abs/2508.14925) |
