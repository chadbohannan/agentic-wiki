# Multimodal Agents

Multimodal agents use vision, audio and document-capable models, and multimodal retrieval, inside the agent loop. This page covers exam objective 2.2 and is one of the thinner areas of the wiki: the study guide's only source for it is a video that could not be read.

Images and audio travel through the same tool interface as text, and MCP defines image and audio content types (base64 data plus a MIME type) in tool results, as described in [Tool Calling and MCP](tool-calling-and-mcp.md) **[web]**. [NVIDIA NIM](../components/nvidia-nim.md) is the serving layer for the vision-language models discussed here.

## Approaches to multimodal RAG

There are three common approaches: convert non-text to text (captioning with a VLM, chart-to-table with DePlot-style models, OCR or transcription) and use a text embedder; embed images and text jointly in a shared space; or pass retrieved images straight to a VLM at generation time **[inferred; a Zilliz blog is the lead, but its body could not be read]**. Text conversion is cheap and debuggable but loses layout and visual detail, while image handling costs more tokens and latency per image or video frame and needs modality-appropriate evaluation **[inferred]**.

## NVIDIA's stack

The study guide's video (building multimodal RAG with LlamaIndex, NIM and Milvus) was unreadable, but secondary snippets suggest an example stack of LlamaIndex orchestration, NV Embed text embeddings, Milvus, NeVA 22B for images and DePlot for charts **[web, secondary snippet; unconfirmed]**. Its companion notebook in NVIDIA's GenerativeAIExamples repository has version drift (pymilvus API changes), per a forum thread **[web]**.

NIM for VLMs serves vision-language models behind an OpenAI-compatible API with function calling and structured generation, and its `tool_choice` includes `required` **[web, NIM docs summary]**. Nemotron Nano 12B v2 VL is an open 12B hybrid Transformer-Mamba VLM for document intelligence and video understanding, with a 128K context, an October 28, 2025 release date per OpenRouter and Efficient Video Sampling for long video. NIM VLM documentation now lists it among past releases, with newer models current **[web, secondary plus docs summary; moderate confidence]**.

Audio and speech (ASR, TTS, the NeMo voice agent) are not covered by any listed source. A NeMo Voice Agent release-notes page exists, but it was not read, so this is an open gap tracked in `TODO.md`.

## Interview Q&A

**Two ways to handle images in RAG?** Summarise them to text and embed the text, or embed images multimodally and hand retrieved images to a VLM **[inferred]**.

**Why use NIM for a VLM in an agent?** It is a packaged, optimised, OpenAI-compatible endpoint that also supports tool calls and structured output, so the VLM fits the same agent loop **[web, inferred]**.

**What goes wrong with multimodal RAG demos in production?** Dependency and API drift, token and latency cost of images, and chart or table extraction quality **[web, inferred]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Building Multimodal AI RAG With LlamaIndex, NVIDIA NIM, and Milvus (video)](https://www.youtube.com/watch?v=NaT5Eo97_I0) |
| 2026-10-27 | [Multimodal RAG: expanding beyond text (Zilliz)](https://zilliz.com/blog/multimodal-rag-expanding-beyond-text-for-smarter-ai) |
| 2026-10-27 | [Nemotron Nano 12B v2 VL (OpenRouter)](https://openrouter.ai/models/nvidia/nemotron-nano-12b-v2-vl) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
