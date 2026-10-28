# NeMo Guardrails

NeMo Guardrails is NVIDIA's open-source (Apache 2.0) Python toolkit, with a companion microservice, for putting a policy layer between the user, the LLM, retrieved context and tools instead of relying on the model's own alignment. According to NVIDIA's developer page, it blocks, alters or validates unsafe, off-topic, malicious or policy-violating inputs and responses, and can be added "before and after LLM calls without changing the application's LLM" **[sourced]**. It is one layer, not the whole of a defence-in-depth design; where it sits among the others is set out in [Layered Safety and Escalation](../concepts/layered-safety-and-escalation.md).

The current release is v0.24.1 (2026-09-16), for Python 3.10–3.13 (`pip install nemoguardrails`), and the repository has moved from `NVIDIA/NeMo-Guardrails` to `NVIDIA-NeMo/Guardrails` **[web]**. The Guardrails microservice is a production container using the same configuration model, so configurations carry over **[sourced]**. Version details below refer to this release.

## Rail types

The configuration reference lists five categories under the `rails` key of `config.yml`, each named by when it triggers **[sourced, web]**:
- **Input rails** run when user input arrives: content safety, jailbreak detection, topic control and PII masking.
- **Retrieval rails** run after RAG retrieval, so only trusted chunks reach the model.
- **Dialog rails** run after the user message has been mapped to a canonical form, steering a multi-turn flow.
- **Execution rails** run before and after a custom action, the Python functions that Colang flows call.
- **Output rails** run on the generated response: safety, fact and hallucination checks, and PII removal.

The model's own tool calls have separate `tool_call` and `tool_result` rail sections, and the built-in `tool call validation` and `tool result validation` flows run only on the IORails engine described below. NVIDIA's older rail-types overview describes execution rails as checking tool calls, but the engine reference separates the two: execution rails govern custom actions, and tool rails govern the model's own calls **[web]**. Jailbreak protection applies only to input, while content safety and PII apply to both directions **[sourced]**.

## Configuration

A configuration is a `config` folder. Only `config.yml` is required: it names the models, the active rails and their settings, general instructions and sample conversations. Optional parts are Colang flows (`.co` files, loaded recursively from anywhere in the folder, conventionally `rails/`), `prompts.yml`, custom actions decorated with `@action()` in `actions.py` or an `actions/` package, a `config.py` run once at start-up to register custom LLM or embedding providers, and a `kb/` folder of Markdown documents for the built-in retrieval rail **[web]**. The SDK loads the folder with `RailsConfig.from_path` and runs it with `LLMRails`, whose `generate` takes messages in the OpenAI Chat Completions style **[sourced, web]**.

A typical `config.yml`, adapted from NVIDIA's parallel-rails example, uses NemoGuard classifiers served as NIMs, the models described in [NVIDIA Safety Models and garak](nvidia-safety-models-and-garak.md) **[web]**:

```yaml
models:
  - type: main
    engine: nim
    model: meta/llama-3.1-70b-instruct
  - type: content_safety
    engine: nim
    model: nvidia/llama-3.1-nemoguard-8b-content-safety
rails:
  input:
    parallel: True
    flows:
      - content safety check input $model=content_safety
      - jailbreak detection model
  output:
    flows:
      - content safety check output $model=content_safety
```

A flow names the model it uses with `$model=`, and a flow that names a model type the configuration does not declare is rejected. Per-rail settings, such as the PII entity types to mask or jailbreak perplexity thresholds, go under `rails.config` **[web]**.

## Colang

Colang is an event-driven language, interpreted by a Python runtime, that models an interaction as a stream of events: user utterances, LLM responses, actions and their results, and triggered rails **[web]**. Colang 1.0 is still the default. Colang 2.0 was alpha in 0.8 and has been beta since 0.9, and is selected with `colang_version: "2.x"` **[web]**.

Colang 1.0 defines canonical forms by example utterances and chains them into flows; the dialog rails then match a new message to the nearest canonical form, which is why the NIM tutorial pairs the main model with an embedding NIM **[sourced, web]**:

```
define user express greeting
  "Hello"
  "Hi"

define flow greeting
  user express greeting
  bot express greeting
```

Colang 2.0 is a rewrite of language and runtime: an explicit `main` flow, flows activated explicitly and run in parallel, a standard library brought in with `import`, Python-like `if` and `await`, and a generation operator `...` that asks the LLM for a value. Its input rail is a flow named `input rails` that takes `$input_text` and can `abort` **[web]**:

```
import guardrails

flow input rails $input_text
  $input_safe = await check user utterance $input_text
  if not $input_safe
    bot say "I'm sorry, I can't respond to that."
    abort
```

The docs list a limitation that matters when choosing: the built-in guardrails library is "not yet fully usable" from Colang 2.0 **[web]**. `nemoguardrails convert` migrates files between versions, and its output needs review **[sourced, web, snippet]**. Colang 1.0 suits configurations built from the catalogue's input and output rails, while 2.0 suits event-driven dialogue such as multimodal or avatar interactions.

## Engines: LLMRails and IORails

Release 0.21.0 (2026-03-12) introduced a second engine, IORails, which gained streaming output rails, OpenTelemetry, tool calling and most of the built-in catalogue over the next three releases. Both engines read the same `RailsConfig`, but they trade generality for speed **[web]**:
- **LLMRails** runs the full Colang 1.0 and 2.x runtime: dialog, retrieval and execution rails, embeddings, conversation state, LangChain models and the event API.
- **IORails** runs input, output and tool rails without Colang. It compiles each flow from the rail's manifest and calls the rail action directly, adding parallel rails, admission control through a work queue, OpenTelemetry token and duration metrics, and optional speculative generation. It is stateless and accepts Colang 1.0 configurations only.

The recommended entry point is a `Guardrails` facade, which routes to IORails when the configuration uses only `input`, `output`, `config`, `tool_call` and `tool_result` sections, every flow compiles, and no custom LLM is passed; otherwise it falls back to LLMRails and logs why. `require_iorails=True` raises instead of falling back, and `IORails.unsupported_reason(config, llm)` explains the decision. IORails can run 59 of the 78 surfaces in the built-in catalogue; most of the rest read Colang conversation state such as retrieved chunks **[web]**.

## Latency

Each rail that calls a model adds a call and its latency, and output rails need buffering when the response is streamed. Guardrails offers three ways to cut that cost **[web]**:
- **Parallel rails.** `parallel: True` under `rails.input` or `rails.output` runs independent rails concurrently, and the first to block short-circuits the rest. The docs warn that input rails which modify the message can race, so those should run sequentially; on IORails, any rewriting rail forces sequential execution.
- **Speculative generation.** `speculative_generation: True` under `rails.input` (IORails only, non-streaming, labelled experimental) starts the main LLM call at the same time as the input rails. A safe request hides the input-rail latency; an unsafe one cancels or discards the generation, so the cost is the tokens wasted on blocked requests. Output rails still run on every response.
- **Streaming output rails.** `streaming` with `chunk_size` and `context_size` checks the response in chunks as it streams.

Speculative generation is the same bet as speculative execution elsewhere: it pays when unsafe requests are rare, and NVIDIA's own illustration assumes 2% **[web]**.

## Integration

There are four integration paths **[sourced, web]**:
- **The Python SDK**, through the `Guardrails` facade or `LLMRails` directly.
- **`nemoguardrails server`**, which serves a Chat Completions-like `POST /v1/chat/completions` taking a `config_id`, so guardrails can sit in front of an existing client or a model served by [NVIDIA NIM](nvidia-nim.md). It runs on LLMRails unless `NEMO_GUARDRAILS_IORAILS_ENGINE=1` routes supported configurations to IORails, a path the docs call early release.
- **LangChain and LangGraph**, an opt-in integration on LLMRails only (`NEMOGUARDRAILS_LLM_FRAMEWORK=langchain`), in which LangChain tools can be registered as custom actions.
- **Agent hooks**, which are unreleased: on the `develop` branch, a `nemoguardrails hooks-server` checks a coding agent's tool calls against a tool policy and per-tool rails. Its `/v1/hooks/nemo-relay` endpoint receives a tool's name and arguments from a [NeMo Relay](nemo-relay.md) plugin and answers `allow` or `deny` with a reason.

The Relay integration shows a fail-open trap. The docs tell plugin authors to map any timeout or error to a failure, never to an allow. Even so, what happens to a failed check depends on the Relay integration: the Claude Code and Codex integrations block the call from Relay 0.10.1, while the pi extension lets it run by default **[web]**. A guardrail is only as strong as its failure mode, which is the point of fail-closed design in [Agent Security Threats and Controls](../concepts/agent-security-threats-and-controls.md).

For tool-using agents, NVIDIA recommends keeping authentication credentials away from the LLM **[sourced]**. Content safety can come from an LLM self-check or from NVIDIA's classifiers, and third-party integrations include ActiveFence, Cisco AI Defense, Presidio, GLiNER-PII and LlamaGuard **[sourced, web]**.

## Trade-offs

The README warns that the built-in rails may not fit every production case. Reasoning traces bypass the output rails on both engines, so output rails check the final answer, not the model's reasoning. Telemetry is on by default (an anonymous usage event at start-up plus heartbeats) and is disabled with `NEMO_GUARDRAILS_NO_USAGE_STATS=1` or `DO_NOT_TRACK=1` **[web]**.

## Interview Q&A

**Name the rail types and when each runs.** Input (on user input), retrieval (on RAG chunks), dialog (after the canonical form is computed), execution (around custom actions) and output (on the response). Tool-call and tool-result rails check the model's own tool calls **[sourced, web]**.

**What goes in a Guardrails configuration?** A `config` folder with a required `config.yml` (models and active rails), plus optional Colang `.co` flows, `prompts.yml`, `actions.py`, `config.py` and a `kb/` folder **[web]**.

**LLMRails or IORails?** LLMRails runs the full Colang runtime, needed for dialog, retrieval and custom-action rails. IORails is a faster, stateless engine for input, output and tool rails, with parallelism, admission control and metrics. The `Guardrails` facade picks IORails when the configuration allows **[web]**.

**How do you keep guardrails from adding too much latency?** Run independent rails in parallel, use small classifier NIMs instead of LLM self-checks, stream output rails in chunks, and, on IORails, speculatively generate while the input rails run **[web]**.

**How do you guard an agent's tool use?** Validate tool calls and results with tool rails, keep credentials out of the LLM's context, trace every action, and make the check fail closed. Combine this with input rails against injection **[sourced, web]**.

**Colang 1.0 or 2.0?** 1.0 is the default and the only one IORails accepts. 2.0 is beta, opt-in through `colang_version`, and better suited to event-driven multimodal dialogue, but the built-in rails library is not yet fully usable from it **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NeMo Guardrails (NVIDIA Developer)](https://developer.nvidia.com/nemo-guardrails) |
| 2026-10-28 | [NeMo Guardrails (GitHub)](https://github.com/NVIDIA-NeMo/Guardrails) |
| 2026-10-28 | [NeMo Guardrails releases (GitHub)](https://github.com/NVIDIA-NeMo/Guardrails/releases) |
| 2026-10-28 | [NeMo Guardrails rail types](https://docs.nvidia.com/nemo/guardrails/latest/about/rail-types.html) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: configuration overview](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/configure-rails/overview.mdx) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: guardrails configuration](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/configure-rails/yaml-schema/guardrails-configuration.mdx) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: engine feature support](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/reference/engine-feature-support.mdx) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: Colang guide](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/configure-rails/colang/index.mdx) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: Colang 1.0 core concepts](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/configure-rails/colang/colang-1/tutorials/2-core-colang-concepts/README.mdx) |
| 2026-10-28 | [NeMo Guardrails v0.24.1: Colang 2.0 input rails](https://github.com/NVIDIA-NeMo/Guardrails/blob/v0.24.1/docs/configure-rails/colang/colang-2/getting-started/input-rails.mdx) |
| 2026-10-28 | [NeMo Guardrails (develop): NeMo Relay hooks](https://github.com/NVIDIA-NeMo/Guardrails/blob/develop/docs/run-rails/using-hooks-server/nemo-relay.mdx) |
| 2026-10-28 | [Securing Generative AI Deployments With NVIDIA NIM and NeMo Guardrails](https://developer.nvidia.com/blog/securing-generative-ai-deployments-with-nvidia-nim-and-nvidia-nemo-guardrails) |
| 2026-10-28 | [NIM for LLMs release notes](https://docs.nvidia.com/nim/large-language-models/latest/about-nim-llm/release-notes.html) |
