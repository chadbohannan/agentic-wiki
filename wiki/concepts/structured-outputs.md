# Structured Outputs

Structured output forces an LLM's response to match a schema so downstream code and tools can trust it. The guarantee covers shape, never truth.

## Levels of control

Prompt-only instruction ("reply in JSON") is fragile. JSON mode (`response_format: json_object`) guarantees syntactically valid JSON of any shape, including an empty object. Schema- or grammar-constrained decoding restricts the sampler so output must conform to a JSON schema, a regular expression, a grammar or a list of choices **[web, snippet]**. There are two integration points: a final-response schema, and strict schemas for the tool arguments in [Tool Calling and MCP](tool-calling-and-mcp.md). Both descriptions come from secondary sources rather than provider documentation **[web]**.

## What the guarantee covers

When the output is a pick from a known set rather than a structured document, the [decision models](decision-models.md) that appeared in late 2026 go further: they score every allowed answer in one pass and return probabilities without generating any tokens **[web]**. Constrained decoding guarantees shape, not truth. Output can be schema-valid and still hallucinated. Schemas also have engine limits: OpenAI's structured outputs, for example, support recursive schemas but cap a schema at 5,000 object properties, 10 levels of nesting and 1,000 enum values, and the first request with a new schema pays extra latency while it is processed **[web]**. Structured output makes the parse-error retries in [Agent Error Handling and Resilience](agent-error-handling-resilience.md) rarer, but the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ReAct agent keeps parse-retry settings because ReAct's text format is free-form **[web, snippet]**.

MCP draws a separate line. A tool may declare an `outputSchema`; if so, the server must return conforming `structuredContent` and the client should validate it. The current specification (2026-07-28) states explicitly that this is unrelated to LLM schema-constrained generation: it validates tool results, not model generation **[web]**.

## In NVIDIA's stack

[NVIDIA NIM](../components/nvidia-nim.md) has always constrained output by JSON schema, regular expression, choice list or context-free grammar, but the request shape changed twice, so an answer or a client written against one version can fail on another. NIM 1.1 put `guided_json`, `guided_regex` and their siblings inside an `nvext` extension to the OpenAI request **[web]**. By 1.15 they were top-level extra-body fields (`guided_json`, `guided_regex`, `guided_choice`, and `guided_grammar` in EBNF). The 1.15 docs recommend `guided_json` over `response_format={"type": "json_object"}` because JSON mode permits any valid JSON, including an empty object. xgrammar is the default and fastest backend, a fall-back to outlines can slow the first request and fail to compile some regexes, and SGLang profiles do not support lm-format-enforcer **[web]**.

NIM 2.x hands requests to the engine in the image. vLLM removed the `guided_*` fields in 0.12.0 in favour of a `structured_outputs` object (`json`, `regex`, `choice`, `grammar`, `structural_tag`) or a `response_format` carrying a JSON schema, and its backends are now xgrammar and guidance, chosen automatically by default, with lm-format-enforcer deprecated from 0.33.0 **[web]**. NIM 2.1.4 ships vLLM 0.30.0, so on its vLLM images the 1.x `guided_json` field no longer applies **[web]**.

## Interview Q&A

**JSON mode versus schema-constrained decoding?** JSON mode guarantees only syntactic JSON, while a schema (`guided_json` in NIM 1.x, `structured_outputs.json` or a `json_schema` response format on vLLM) constrains decoding so keys, types and enums are enforced **[web]**.

**Does structured output eliminate hallucination?** No, only format errors; values still need semantic validation **[web]**.

**How do you request a schema from NIM?** In 1.1 through `nvext`, in later 1.x through a top-level `guided_json`, and in 2.x through the engine's own parameters, which on vLLM means `structured_outputs` or `response_format` **[web]**.

**Which constrained-decoding backends matter?** xgrammar is the fast default in both NIM 1.x and vLLM; outlines was NIM 1.x's slower fall-back, and current vLLM pairs xgrammar with guidance **[web]**.

**How does MCP's `outputSchema` differ?** It validates tool results from server to client and does not constrain the model **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-28 | [NIM for LLMs 1.15: structured generation](https://docs.nvidia.com/nim/large-language-models/1.15.0/structured-generation.html) |
| 2026-10-28 | [NIM for LLMs 1.1: structured generation](https://docs.nvidia.com/nim/large-language-models/1.1.0/structured-generation.html) |
| 2026-10-28 | [NIM for LLM and VLM: API reference](https://docs.nvidia.com/nim/large-language-models/latest/reference/api-reference.html) |
| 2026-10-28 | [vLLM: structured outputs](https://docs.vllm.ai/en/latest/features/structured_outputs/) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [Structured output LLM overview (morphllm)](https://www.morphllm.com/structured-output-llm) |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
| 2026-10-28 | [Structured Outputs (OpenAI API docs)](https://developers.openai.com/api/docs/guides/structured-outputs) |
