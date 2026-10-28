# NeMo Guardrails

NeMo Guardrails is NVIDIA's open-source (Apache 2.0) Python toolkit, with a companion microservice, for wrapping an LLM application in programmable rails. Rails check inputs, dialogue, retrieved context, tool execution and outputs, and are configured in YAML plus the Colang language. It touches exam objective 7.1 and objectives 9.2 and 9.4.

The rails wrap a model served by [NVIDIA NIM](nvidia-nim.md), and enforce policy on the tool calls described in [Tool Calling and MCP](../concepts/tool-calling-and-mcp.md). Their place in a defence-in-depth design is set out in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md). The NVIDIA classifier models they call are covered in [NVIDIA Safety Models and garak](nvidia-safety-models-and-garak.md), and the threats they address are in [Agent Security Threats and Controls](../concepts/agent-security-threats-and-controls.md).

## What it is

Guardrails adds a policy layer between the user, the LLM, retrieved context and tools, rather than relying on the model's own alignment **[inferred]**. According to NVIDIA's developer page, it blocks, alters or validates unsafe, off-topic, malicious or policy-violating inputs and responses. It can be added "before and after LLM calls without changing the application's LLM" **[sourced]**.

The GitHub README (read 2026-10-28) gives the latest release as v0.24.1, supporting Python 3.10–3.13 and installed with `pip install nemoguardrails` **[web]**. A separate Guardrails microservice is a production container that uses the same configuration model, so configurations carry over **[sourced]**. The README now names an `IORails` runtime engine alongside `LLMRails`, and the repository contains a `nemo-guardrails-relay-plugin` Rust crate. Neither is explained there, but the plugin presumably connects Guardrails to [NeMo Relay](nemo-relay.md) **[web; connection inferred]**.

## Rail types

There are five stages **[sourced, web rail-types doc]**:
- **Input rails** run before the LLM: content safety, jailbreak detection, topic control and PII masking.
- **Retrieval rails** filter and validate RAG chunks, so only trusted context reaches the model.
- **Dialog rails** steer a multi-turn flow using canonical forms, for topic control and guided conversation.
- **Execution rails** check tool calls, their arguments and their results. This is the hook for agent security.
- **Output rails** evaluate and post-process the response: safety, PII removal and filtering.

Input and output rails are the most common. Jailbreak protection applies only to input, while content safety and PII apply to both. The README's telemetry section also mentions tool-call and tool-result rails, which suggests the categories are growing **[web]**.

## Configuration and Colang

A configuration is a directory. It holds a `config.yml` (models, which rails to use, prompts, tracing and runtime settings), Colang flow files (`.co`), and optional Python for custom actions **[sourced for YAML, Colang and Python; exact filenames inferred]**. The core objects are `RailsConfig` and `LLMRails`, and `generate` takes messages in the OpenAI Chat Completions style **[sourced]**.

Colang is the language for flows, guardrail logic and event-driven behaviour. Colang 1.0 is the default. Colang 2.0 is selected with `colang_version: "2.x"`. It was alpha in 0.8 and beta from 0.9, and it is a rewrite with `flow` and `await` syntax and an event-driven model. A `nemoguardrails convert` tool migrates 1.0 files, but the result needs manual review **[sourced; web, syntax details from a snippet]**. Colang 1.0 matches user intents to canonical forms and flows, while 2.0 is a more general language of parallel flows and events **[inferred]**.

## Integration

There are three integration paths **[sourced, web]**:
- **The Python SDK.**
- **`nemoguardrails server`**, which serves `POST /v1/chat/completions` (taking a `config_id` and `messages`) in a Chat Completions-like format, so guardrails can sit in front of an existing client or a NIM.
- **LangChain and LangGraph**, an opt-in integration set with `NEMOGUARDRAILS_LLM_FRAMEWORK=langchain` or `set_default_framework("langchain")`. LangChain tools can be registered as custom actions.

For tool-using agents, execution rails validate tool inputs and outputs, and NVIDIA recommends keeping authentication credentials away from the LLM **[sourced]**. Content safety can come from an LLM self-check or from NVIDIA's classifier models, and third-party integrations include ActiveFence, Cisco AI Defense, Presidio, GLiNER-PII and LlamaGuard **[sourced, web]**.

## Trade-offs

Each rail that calls an LLM adds a model call and its latency, and output rails need buffering when the response is streamed **[inferred]**. The README warns that the built-in rails may not fit every production case. Telemetry is on by default: it sends an anonymous usage event when rails are instantiated, plus periodic heartbeats. Opt out with `NEMO_GUARDRAILS_NO_USAGE_STATS=1` or `DO_NOT_TRACK=1` set before start-up **[web]**. NVIDIA Build endpoints are for evaluation only **[sourced]**.

## Interview Q&A

**Name the five rail types and where each runs.** Input (before the LLM), retrieval (on RAG chunks), dialog (steering the flow), execution (around tool and action calls), and output (before the user sees the reply) **[sourced]**.

**How do you guard an agent's tool use?** Execution rails validate tool-call arguments and results. Keep credentials out of the LLM's context and trace every action. Combine this with input rails against injection **[sourced]**.

**How do you put guardrails in front of an existing OpenAI-style client?** Run `nemoguardrails server` and point the client at its `/v1/chat/completions` endpoint **[sourced, web]**.

**Colang 1.0 or 2.0?** 1.0 is the default. 2.0 is opt-in through `colang_version`, uses `flow` and `await` syntax, and has a conversion tool whose output needs review **[sourced, web]**.

**What do rails cost?** Added latency and tokens for LLM-based checks. Use small dedicated safety models and put cheap heuristic rails first **[inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Guardrails (GitHub)](https://github.com/NVIDIA/NeMo-Guardrails) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
