# Agent Security Threats and Controls

An LLM agent turns text into actions. The security question therefore shifts from "can the model be tricked into saying something bad" to "what can a tricked model do". This page covers exam objective 9.1: the threats (direct and indirect prompt injection, tool poisoning, excessive agency, confused-deputy flaws in MCP) and the controls that actually limit damage (least privilege, sandboxing, gating actions in code, and breaking the "lethal trifecta").

Some of these controls are enforced by [NeMo Guardrails](../components/nemo-guardrails.md) and the models in [NVIDIA Safety Models and garak](../components/nvidia-safety-models-and-garak.md). Threats that come through tools and MCP servers relate to [Tool Calling and MCP](tool-calling-and-mcp.md) and [Agent Protocols: MCP and A2A](agent-protocols-mcp-a2a.md). Memory poisoning targets the stores in [Agent Memory (Short- and Long-Term)](agent-memory.md), and the record-keeping side is in [Audit Trails and Traceability](audit-trails-and-traceability.md).

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

OWASP's separate Top 10 for Agentic Applications for 2026 was released on 2025-12-09. It extends that list for agents **[web; release date from OWASP, names from consistent third-party summaries; wording varies slightly between sources]**:
- ASI01 agent goal hijack;
- ASI02 tool misuse and exploitation;
- ASI03 identity and privilege abuse;
- ASI04 agentic supply-chain vulnerabilities;
- ASI05 unexpected code execution;
- ASI06 memory and context poisoning;
- ASI07 insecure inter-agent communication;
- ASI08 cascading failures;
- ASI09 exploitation of human trust in the agent;
- ASI10 rogue agents.

## How agents get attacked

Direct injection comes from the user. Indirect injection arrives in content the agent retrieves, such as web pages, emails, tickets, documents and tool results. That is why RAG and MCP tools widen the attack surface **[inferred, consistent with OWASP and Willison]**.

Simon Willison's "lethal trifecta" (2025-06-16) says that an agent combining three things can be steered into leaking data:
- access to private data;
- exposure to untrusted content;
- a way to communicate externally.

He notes that guardrail products cannot reliably prevent this. His advice is to avoid combining all three, or to constrain the agent so that untrusted input it has read cannot trigger consequential actions **[web]**. Tool poisoning hides instructions in a tool's description or schema. A search snippet cites an "MCPTox" benchmark with 36.5% average attack success across 20 models, which was not verified **[web, snippet]**.

Bounded outputs are no defence. HiddenLayer reported in October 2026 that TypeSafe's Jev [decision model](decision-models.md), whose outputs are restricted to typed choices, was still swayed by prompt injection and TokenBreak-style perturbations, including an injected ad that redirected a browser agent. Constraining the output space does not make the input trustworthy **[web, one security vendor via InfoWorld]**.

## Controls, from most to least reliable

1. **Reduce capability.** Grant least privilege and minimal scope. The MCP security best practices recommend progressive least-privilege scopes: start minimal, step up incrementally, and log each elevation with a correlation ID. They warn that omnibus scopes widen the blast radius and blur the audit trail **[web]**. OWASP's "excessive agency" describes agents given more autonomy or permissions than they need **[web, entry teaser only]**.
2. **Sandbox.** The same MCP guidance recommends running local MCP servers in sandboxes with minimal default privileges, and showing the exact start-up command before running it. It prefers the stdio transport for local servers, and routes server-side clients through egress proxies that block private IP ranges against server-side request forgery **[web]**.
3. **Gate actions in code, not in the prompt.** Validate tool arguments, use allowlists, and require approval for consequential actions. NeMo Guardrails' execution rails are NVIDIA's hook for validating tool calls, arguments and results **[web]**.
4. **Use proper identity.** MCP servers must not accept tokens that were not issued to them (no token passthrough), which also protects the audit trail **[web]**.
5. **Add probabilistic detection as one layer, never the only one.** This means jailbreak and injection classifiers and content-safety models **[inferred]**.

On the NVIDIA side, the Guardrails docs map jailbreak protection to input rails and agent security to execution rails. NemoGuard JailbreakDetect is a NIM classifier, and garak is NVIDIA's open-source scanner for probing an endpoint before release **[web]**.

Detection layers add latency and false positives. Structural controls, such as removing a leg of the trifecta or removing write tools, cost capability. Human approval costs throughput and suffers from approval fatigue **[inferred]**.

## Interview Q&A

**What is indirect prompt injection, and why do agents make it worse?** Malicious instructions arrive in data the agent reads, such as a page, an email or a tool result, rather than from the user. Agents have tools, so the injected text can trigger actions or exfiltration **[inferred]**.

**What is the lethal trifecta, and how do you design around it?** Private data, untrusted content and external communication in one agent. Remove at least one leg, or block consequential actions once untrusted input has been read **[web]**.

**Why is a guardrail classifier not a sufficient defence?** It is probabilistic and can be bypassed, and no one knows how to prevent injection with full reliability. Pair it with least privilege, sandboxing and gating actions in code **[web, inferred]**.

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
| 2026-10-28 | [garak (GitHub)](https://github.com/NVIDIA/garak) |
| 2026-10-28 | [HiddenLayer on Jev (InfoWorld)](https://www.infoworld.com/article/4236103/typesafes-jev-even-more-susceptible-to-attack-than-llms-says-security-vendor.html) |
