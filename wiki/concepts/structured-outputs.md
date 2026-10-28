# Structured Outputs

Structured output forces an LLM's response to match a schema so downstream code and tools can trust it. This page covers exam objectives 2.3 and 2.4: the levels of control from prompting to constrained decoding, the NVIDIA NIM implementation, and what the guarantee does and does not cover.

Constrained decoding is one defence against the parse failures discussed in [Agent Error Handling and Resilience](agent-error-handling-resilience.md), and tool-argument schemas are part of the function-calling flow in [Tool Calling and MCP](tool-calling-and-mcp.md). The serving implementation is covered in [NVIDIA NIM](../components/nvidia-nim.md).

## Levels of control

Prompt-only instruction ("reply in JSON") is fragile. JSON mode (`response_format: json_object`) guarantees syntactically valid JSON of any shape, including an empty object. Schema- or grammar-constrained decoding restricts the sampler so output must conform to a JSON schema, a regular expression, a grammar or a list of choices **[web, secondary descriptions plus the NIM docs summary]**. There are two integration points, a final-response schema and strict tool-argument schemas **[web, secondary; not checked against provider docs]**.

## NVIDIA NIM

According to a summary of the NIM for LLMs structured-generation documentation, NIM can constrain output by JSON schema, regex, context-free grammar or choice list. It recommends `guided_json` over generic JSON mode and the xgrammar backend as the fastest, warns that falling back to outlines can slow first inference and that some regexes fail to compile under outlines, and notes that lm-format-enforcer is not supported on SGLang profiles. The parameters travel in an `nvext` extension to the OpenAI schema in older docs, and the NIM VLM docs show `client.beta.chat.completions.parse` with a response model and `guided_regex` in `nvext` **[web, snippet-level]**. A direct fetch of the page on 2026-10-28 returned no usable text, so these details are moderately confident and the exact request shape varies by NIM version.

## What the guarantee covers

When the output is a pick from a known set rather than a structured document, the [decision models](decision-models.md) that appeared in late 2026 go further: they score every allowed answer in one pass and return probabilities without generating any tokens **[web]**. Constrained decoding guarantees shape, not truth. Output can be schema-valid and still hallucinated, and very large or recursive schemas can be rejected or slow to compile **[web, from a low-quality blog; treat as inferred]**. Structured output makes parse-error retries rarer, but the [NeMo Agent Toolkit](../components/nemo-agent-toolkit.md) ReAct agent keeps parse-retry settings because ReAct's text format is free-form **[web, snippet-level]**.

MCP draws a separate line. A tool may declare an `outputSchema`; if so, the server must return conforming `structuredContent` and the client should validate it. The current specification (2026-07-28) states explicitly that this is unrelated to LLM schema-constrained generation: it validates tool results, not model generation **[web, read in full]**.

## Interview Q&A

**JSON mode versus `guided_json`?** JSON mode guarantees only syntactic JSON, while `guided_json` constrains decoding to a supplied schema so keys, types and enums are enforced **[web]**.

**Does structured output eliminate hallucination?** No, only format errors; values still need semantic validation **[web, inferred]**.

**Which NIM backends matter?** xgrammar is fast and preferred, outlines is the fallback and can add first-request latency or fail to compile some regexes **[web, snippet-level]**.

**How does MCP's `outputSchema` differ?** It validates tool results from server to client and does not constrain the model **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [NIM for LLMs: structured generation](https://docs.nvidia.com/nim/large-language-models/latest/structured-generation.html) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-27 | [NeMo Agent Toolkit ReAct agent docs](https://docs.nvidia.com/nemo/agent-toolkit/1.3/workflows/about/react-agent.html) |
| 2026-10-27 | [Structured output LLM overview (morphllm)](https://www.morphllm.com/structured-output-llm) |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
