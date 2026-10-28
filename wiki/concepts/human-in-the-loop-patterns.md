# Human-in-the-Loop Patterns

Human-in-the-loop (HITL) for agents means inserting a pause in which a person approves, edits, answers or rejects before the agent proceeds. It is implemented as approval gates, interrupts with resume, elicitation of missing input, and escalation based on risk or confidence. The screens that present these pauses are designed in [Agent Interfaces and Oversight UI](agent-interfaces-and-oversight.md); this page covers the mechanisms behind them and their costs.

## Two meanings

HITL in the classic machine-learning sense means annotators labelling data and experts validating outputs. HolisticAI's post, which the study guide cites, describes this sense (annotation, validation and continuous feedback) and mentions neither human-on-the-loop nor human-in-command **[sourced]**. Runtime HITL for agents is different: the agent is mid-task and a human decision gates a specific action. OneReach, a vendor, describes this kind of oversight as tiered, in three ways **[sourced]**:
- routine work is autonomous and high-stakes work is routed to a person;
- autonomy adapts with risk or confidence;
- escalation happens by automatic routing or by real-time handoff, on triggers like those in [Layered Safety and Escalation](layered-safety-and-escalation.md).

The usual taxonomy comes from the EU High-Level Expert Group's Ethics Guidelines for Trustworthy AI (April 2019). Human-in-the-loop is the capability for human intervention in every decision cycle, which the guidelines note is often neither possible nor desirable; human-on-the-loop is involvement in the design cycle and monitoring of the system in operation; human-in-command is oversight of the system's overall activity, including its wider impact, and the decision whether, when and how to use it or override it **[web, snippet]**. Engineering usage narrows these to per-action approval, monitoring with the power to intervene, and setting the boundaries with a kill switch.

## Mechanisms

**LangGraph interrupts.** `interrupt(value)` suspends the graph, saves its state through the checkpointer described in [Stateful Orchestration](stateful-orchestration.md), and surfaces the value to the caller. The graph waits indefinitely until it is resumed with `Command(resume=value)` on the same `thread_id`, and the resume value becomes the return value of `interrupt()`. It needs a checkpointer, a thread ID and a JSON-serialisable payload. Common patterns are routing on approve or reject, reviewing and editing state, and calling `interrupt()` inside a tool so that a reviewer can approve, edit the arguments or cancel. There are caveats **[web]**:
- On resume the node restarts from its beginning, so any side effects before the interrupt must be idempotent.
- Do not wrap `interrupt()` in a bare try/except.
- Keep the order of interrupts stable.
- Static breakpoints are for debugging, not HITL.

**OpenAI Agents SDK.** A tool sets `needs_approval`, either to True or to an async function that decides for each call from its arguments and context. An unapproved call pauses the run, and `result.interruptions` lists the pending approvals. `state.approve()` or `reject()` records the decision, with sticky always-approve and always-reject options and a rejection message. `Runner.run(agent, state)` then resumes, and the state serialises to JSON. The documentation warns to authenticate reviewers on the server and not to trust tool arguments supplied by the client. It also says that "approval permits a tool call to execute; it does not guarantee exactly-once execution" **[web]**.

**MCP tool confirmation.** The tools specification says there SHOULD always be a human in the loop able to deny tool invocations. Clients SHOULD show which tools are exposed, indicate when they are invoked, show inputs before calling the server, and log usage for audit. Tool annotations are untrusted unless the server is trusted **[web]**. These are SHOULD-level recommendations: the protocol does not enforce an approval interface.

**MCP elicitation** lets a server ask the user for missing input. In the current specification (2026-07-28), part of the protocol whose tool side is described in [Tool Calling and MCP](tool-calling-and-mcp.md), it is stateless: the server returns an `InputRequiredResult` containing an `elicitation/create` request, and the client retries the original call with the user's response **[web]**. There are two modes:
- **Form mode** collects structured data against a restricted, flat JSON Schema: strings, numbers, booleans, and single- or multi-select enums. Servers MUST NOT use form mode to ask for passwords, API keys, tokens or payment credentials.
- **URL mode**, introduced in 2025-11-25, sends the user to an external URL for sensitive interactions, such as third-party OAuth or payment, so the data never passes through the client or the LLM. Clients MUST show the full URL and get consent before opening it.

Responses are `accept` (with content in form mode), `decline` or `cancel`. Clients MUST show which server is asking and offer clear decline and cancel options **[web]**. Elicitation is a server asking the user for input. Approval is the client asking the user to authorise an action. They are separate mechanisms.

**Graded autonomy in a product.** Claude Code's permission system evaluates deny rules before ask rules before allow rules, and a deny in any settings scope beats an allow elsewhere. Its modes run from `default` (prompt on first use) through `acceptEdits`, `plan` (read-only exploration), `auto` (a background classifier reviews actions) and `dontAsk`, to `bypassPermissions`, which is recommended only in isolated containers or VMs **[web]**. This is one product's design, not a standard.

**NVIDIA.** The NeMo Agent Toolkit's documentation lists "Interactive Workflows" and a HITL middleware module. The toolkit is deprecated, so do not build new work on it **[web]**. NeMo Guardrails' execution and dialog rails are an automated complement to a human gate, not a substitute for one.

## Design trade-offs

The costs **[sourced]**:
- approval fatigue, as users click through frequent prompts;
- latency and abandoned runs, because a paused run needs durable state and a timeout policy;
- inconsistency and bias among reviewers;
- the cost of human review at scale.

Triage rules reduce the load: gate by the risk of the action (irreversible, externally visible, spending money or touching personal data), by model confidence or disagreement, and by novelty. LLM confidence is not reliably calibrated: Xiong et al. (ICLR 2024) found that models asked to verbalise their confidence tend to be overconfident, and that sampling several answers and measuring their consistency helps **[web]**. Validate any confidence signal against labelled outcomes before using it to route decisions.

## Interview Q&A

**How do you implement an approval gate in LangGraph?** Call `interrupt()` in a node or tool with the proposed action. The checkpointer persists the state, and the client resumes with `Command(resume=decision)` on the same thread ID. Make side effects before the interrupt idempotent, because the node replays **[web]**.

**Approval versus elicitation in MCP?** Approval is a client-side confirmation before a tool call, a SHOULD in the spec. Elicitation is a server's request for user input, returned as an `InputRequiredResult`. Its form mode must not ask for secrets, and its URL mode handles sensitive flows out of band **[web]**.

**Which actions should be gated?** Irreversible or externally visible actions, costly ones, and ones touching sensitive data, plus low-confidence or ambiguous cases. Let low-risk reads run on their own.

**What goes wrong with gates?** Approval fatigue, duplicated side effects on replay, stale approvals, trusting arguments supplied by the client, and inconsistent reviewers **[sourced, web]**.

**HITL, HOTL or human-in-command?** In the EU expert group's terms: intervention in every decision cycle, design-time involvement plus monitoring in operation, or oversight of the whole system including whether and how to use it. In agent engineering these become per-action approval, monitoring with the power to intervene, and setting boundaries with a kill switch **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [Human-in-the-loop AI (HolisticAI)](https://www.holisticai.com/blog/human-in-the-loop-ai) |
| 2026-10-28 | [Human-in-the-loop agentic AI systems (OneReach)](https://onereach.ai/blog/human-in-the-loop-agentic-ai-systems/) |
| 2026-10-28 | [AI agents with human-in-the-loop (Cobus Greyling)](https://cobusgreyling.medium.com/ai-agents-with-human-in-the-loop-f910d0c0384b) |
| 2026-10-28 | [NVIDIA Agent Intelligence Toolkit (developer page)](https://developer.nvidia.com/agent-intelligence-toolkit) |
| 2026-10-28 | [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) |
| 2026-10-28 | [OpenAI Agents SDK: human in the loop](https://openai.github.io/openai-agents-python/human_in_the_loop/) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-28 | [MCP specification 2026-07-28: Elicitation](https://modelcontextprotocol.io/specification/2026-07-28/client/elicitation) |
| 2026-10-28 | [Claude Code permissions](https://code.claude.com/docs/en/permissions) |
| 2026-10-28 | [Ethics Guidelines for Trustworthy AI (EU HLEG, 2019)](https://digital-strategy.ec.europa.eu/en/library/ethics-guidelines-trustworthy-ai) |
| 2026-10-28 | [Can LLMs Express Their Uncertainty? (Xiong et al., arXiv 2306.13063)](https://arxiv.org/abs/2306.13063) |
