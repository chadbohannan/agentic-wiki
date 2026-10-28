# Multimodal Agents

A multimodal agent takes images, audio, video or page images as input, reasons over them inside the loop, and may retrieve them. Most of the work is engineering the input: decoding the media, sampling it down to what a model can afford, and then either converting it to text or passing it to a model that reads the modality natively. The concerns are formats, token budgets, latency and GPU placement, and they decide what the agent can afford to look at. Documents and page images bound for a RAG index go through the extraction pipeline in [Document Ingestion and ETL](document-ingestion-etl.md). The study guide's only source here is a video, so most of the page rests on NVIDIA documentation and web research.

## Three ways in

Each modality can enter the agent by one of three routes:
- **Convert to text:** speech recognition for audio, or a VLM writing captions for images and video.
- **Pass natively:** hand the media to a model that accepts it.
- **Embed:** store the media as vectors for later search.

Conversion is cheap to store and easy to debug, and every downstream component, including guardrails and judges, already handles text. It loses tone, layout and anything the captioner did not mention. Native input keeps that detail, but every request pays for it again in tokens. Pipelines that face users usually convert speech to text, because the language model reasons faster over text. Pipelines that watch video usually caption it once and reason over the captions.

Retrieval follows the same three routes. Multimodal RAG can convert non-text to text (captioning with a VLM, chart-to-table with DePlot-style models, OCR or transcription) and use a text embedder, as the extraction stage of ingestion does; embed images and text jointly in a shared space; or pass retrieved images straight to a VLM at generation time. Text conversion is cheap and debuggable but loses layout and visual detail, while image handling costs more tokens and latency per image or video frame and needs modality-appropriate evaluation.

Once inside the loop, media travels through the same tool interface as text: MCP defines image and audio content types (base64 data plus a MIME type) in tool results, as described in [Tool Calling and MCP](tool-calling-and-mcp.md) **[web]**.

## Token budgets and sampling

Media gets expensive because each image or frame becomes many input tokens. NVIDIA's NIM documentation for Qwen3.6 says each 32×32 patch of an image (1,024 pixels) maps to one multimodal token. Video uses 32×32×2 patches, where the 2 is two consecutive frames, so 2,048 pixels per token, and is sampled at 2 frames per second by default **[web]**. By that rule a 1024×1024 image costs about 1,024 tokens. A one-minute clip at 448×448 and 2 fps costs about 12,000 tokens: 120 frames, paired into 60 steps of 196 patches each. NIM's Cosmos Reason1 page says that model performs best at 16K multimodal tokens or fewer **[web]**. Frame rate and resolution are therefore the main cost controls. A long video has to be cut into chunks, captioned chunk by chunk, and the captions summarised.

Chunking trades detail against cost. Short chunks catch brief events but multiply VLM calls. Overlap between chunks keeps an event from being split across a boundary at the cost of some duplicate captions. For audio, the main choice is streaming against offline recognition. Streaming returns partial transcripts while the user is still talking and suits conversation. Offline models see the whole recording, which allows features such as word timestamps and better punctuation.

Decoding is also work. A CPU that decodes many video streams can plausibly become the bottleneck before the GPU does, which is the case for decoding on the GPU, as NVIDIA's video pipeline below does.

## In NVIDIA's stack

### Images, audio and video in NIM

NIM for LLM and VLM takes media through OpenAI-compatible Chat Completions content parts **[web]**:
- **Images:** an `image_url` part, holding a remote URL or a base64 `data:` URL.
- **Audio:** an `audio_url` part, which is an NVIDIA extension to the OpenAI schema, or OpenAI's `input_audio` part.
- **Video:** a `video_url` part. Video is supported only on the vLLM backend.

A remote URL must be reachable from the container, and inline base64 avoids the fetch at request time **[web]**. Base64 is probably the safer default for agents. A URL that came from a user or a tool makes the inference server fetch an address on someone else's behalf, which is the server-side request forgery pattern discussed in [Agent Security Threats and Controls](agent-security-threats-and-controls.md).

The per-model NIM for VLMs pages (1.7.0, the latest listed in its release notes) set the following limits for Qwen3.6 **[web]**:
- at most five images per request by default, raised through `NIM_MAX_IMAGES_PER_PROMPT`;
- video turned off by default, enabled with `NIM_MAX_VIDEOS_PER_PROMPT=1`;
- frame sampling through `media_io_kwargs` with `fps` or `num_frames`, where a value above what the video actually has returns HTTP 400.

Defaults vary by model: Kimi-K2.6 allows four images, and Cosmos Reason1 samples at 4 fps and accepts only one of `fps` or `num_frames` **[web]**. A pipeline should therefore read the page for the model it deploys rather than assume one limit. The same release adds Nemotron 3 Nano Omni, a model that takes audio as well as vision **[web]**.

### Vision-language models in the loop

NIM for LLM and VLM serves vision-language models behind an OpenAI-compatible API, with tool calling on both its vLLM and SGLang backends **[web]**. Support varies by model: the VLM release notes warn that Ministral 3 14B may still attempt tool calls when `tool_choice=none` and tools are present **[web]**. Nemotron Nano 12B v2 VL is an open 12B hybrid Transformer-Mamba VLM for document intelligence and video understanding, with a 128K context, an October 28, 2025 release date per OpenRouter and Efficient Video Sampling for long video. NIM VLM documentation now lists it among past releases, with newer models current **[web, snippet]**.

Secondary descriptions of the study guide's video (building multimodal RAG with LlamaIndex, NIM and Milvus) suggest an example stack of LlamaIndex orchestration, NV Embed text embeddings, Milvus, NeVA 22B for images and DePlot for charts **[web, snippet]**. Its companion notebook in NVIDIA's GenerativeAIExamples repository has version drift (pymilvus API changes), per a forum thread **[web]**.

### Speech NIM

Speech NIM packages Nemotron speech models as GPU containers for recognition (ASR), synthesis (TTS) and translation (NMT). The older Riva ASR and TTS NIM documentation now points to it, and its clients still use the Riva Python client. ASR and TTS expose HTTP REST, gRPC and a Realtime WebSocket API, while NMT is gRPC only and covers 36 languages **[web]**. The ASR support matrix for 26.10.0 splits models by mode **[web]**:
- **Streaming:** Nemotron ASR Streaming, with 40 locales, automatic language detection and eight-speaker diarization, on Blackwell and DGX Spark only.
- **Streaming and offline:** the Parakeet CTC and RNNT models.
- **Offline only:** Parakeet TDT (with word timestamps), Canary and Whisper Large v3, the last two of which also translate speech.

The matrix requires a GPU of compute capability 8.0 or higher with 16 GB of memory, and individual profiles range from about 3 GB to about 50 GB **[web]**.

### Voice agents

NVIDIA's Nemotron Voice Agent blueprint chains ASR, an LLM and TTS on the open-source Pipecat framework. Its README describes the following **[web]**:
- **Turn-taking:** voice activity detection and end-of-utterance logic decide when the agent speaks, and the agent handles interruptions.
- **Clients:** WebRTC for the browser, plus an OpenAI Realtime-compatible WebSocket.
- **Default models:** Nemotron ASR Streaming, Nemotron 3.5 Lightning 30B A3B and Magpie TTS.
- **Omni variant:** a single Nemotron Omni model replaces the ASR and LLM stages.
- **Hardware:** profiles run from one workstation GPU to a server layout of about 80 GB.

Version 2.3.0 (October 2026) added streaming-input LLM prefill to lower time to first token. The README claims sub-second end-to-end latency without giving measurements **[web]**. An NVIDIA FAQ, which labels itself AI-generated and possibly in error, attributes most of that budget to the LLM, at 400–600 ms to first token, with ASR adding 80–160 ms and TTS 78–110 ms to first byte **[web]**. These are NVIDIA's own and loosely sourced figures. If they hold even roughly, they show where tuning pays: the LLM's first token, not the speech models, and the streaming techniques in [Streaming and Real-Time Agent UX](streaming-and-realtime-agent-ux.md) apply directly.

### Video Search and Summarization

The Video Search and Summarization (VSS) blueprint is NVIDIA's reference pipeline for long and live video. Its architecture changed between 2.x and 3.x. In 2.4.1, each chunk was captioned by a VLM, and its audio could be transcribed with Riva ASR. Context-aware RAG then indexed the captions and transcripts in both a vector database and a graph database, and an LLM aggregated them into summaries **[web, snippet]**.

In 3.3.0 the blueprint is a set of microservices beneath a video agent **[web]**:
- **RT-VLM:** captions video and flags incidents.
- **RT-CV:** a DeepStream detection and tracking pipeline using RT-DETR, Grounding DINO and Sparse4D.
- **RT-Embedding:** creates Cosmos-Embed1 embeddings for search.

RT-VLM takes files and live RTSP streams. It decodes H.264, H.265, AV1, VP8 and VP9, using hardware AV1 decoding by default and a DeepStream pipeline for RTSP that shares one decode among requests for the same stream **[web]**. Video is split by `chunk_duration` and `chunk_overlap_duration` (the docs' example is 60 s chunks with 10 s overlap), and a per-chunk setting samples either frames per second or a fixed frame count **[web]**. Captions stream back as timestamped chunk responses and are published to Kafka. An alert is produced by asking the VLM a yes/no anomaly question about each chunk **[web]**. That alert is an LLM judgment that needs evaluating like any classifier, and a small [decision model](decision-models.md) is one alternative for that yes/no step. Supported VLMs include Cosmos Reason 1 to 3, Qwen3-VL and Nemotron 3 Nano Omni, or any OpenAI-compatible endpoint **[web]**.

## Interview Q&A

**Two ways to handle images in RAG?** Summarise them to text and embed the text, or embed images multimodally and hand retrieved images to a VLM.

**Why use NIM for a VLM in an agent?** It is a packaged, optimised, OpenAI-compatible endpoint that also supports tool calls and structured output, so the VLM fits the same agent loop **[web]**.

**What goes wrong with multimodal RAG demos in production?** Dependency and API drift, token and latency cost of images, and chart or table extraction quality **[web]**.

**How would you feed an hour of video to an agent?** Do not send it whole. Decode on the GPU, cut it into overlapping chunks, sample a few frames per second at modest resolution, caption each chunk with a VLM, and transcribe the audio. Index the captions and transcripts with timestamps so the agent searches text and returns to the frames only when it needs them. VSS follows this design, with chunk duration, overlap and frame sampling as its settings **[web]**.

**What drives the cost of an image or video request?** Pixels. NIM's Qwen3.6 page maps each 32×32 patch to one token, and each 32×32 patch across two frames for video, so resolution and frame rate set the token count **[web]**.

**Cascaded voice pipeline or an omni model?** A cascade (ASR, LLM, TTS) lets you swap and evaluate each stage, add guardrails on the text, and read a transcript for audit. An omni model removes one hop and keeps tone of voice. NVIDIA's voice blueprint ships both, with the cascade as its default **[web]**.

**Streaming or offline ASR?** Streaming for live conversation, where partial transcripts and end-of-utterance detection set responsiveness. Offline for recordings, where whole-file models give timestamps, punctuation and translation. In Speech NIM, Nemotron ASR Streaming is streaming-only and Canary, Whisper and Parakeet TDT are offline-only **[web]**.

**Where is the latency in a voice agent?** Mostly in the LLM's time to first token, about 400–600 ms by an AI-generated NVIDIA FAQ, against well under 200 ms each for ASR and TTS **[web]**. Use a smaller or faster model for the spoken turn, and prefill while the user is still speaking.

**Any security concern with media URLs?** NIM fetches a remote media URL from inside the container **[web]**. If agents or users can supply that URL, it becomes a request-forgery path, so send base64 or restrict where media can come from.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [NIM for LLM and VLM: image, audio and video input](https://docs.nvidia.com/nim/large-language-models/latest/advanced-use-cases/multimodal-input.html) |
| 2026-10-28 | [NIM for VLMs 1.7.0: Qwen3.6 API](https://docs.nvidia.com/nim/vision-language-models/1.7.0/examples/qwen3.6/api.html) |
| 2026-10-28 | [NIM for VLMs 1.4.1: Cosmos Reason1 API](https://docs.nvidia.com/nim/vision-language-models/1.4.1/examples/cosmos-reason1/api.html) |
| 2026-10-28 | [NIM for VLMs 1.7.0: Kimi-K2.6 API](https://docs.nvidia.com/nim/vision-language-models/1.7.0/examples/kimi-k2.6/api.html) |
| 2026-10-28 | [NIM for VLMs release notes](https://docs.nvidia.com/nim/vision-language-models/1.7.0/release-notes.html) |
| 2026-10-28 | [Speech NIM 26.10.0 documentation](https://docs.nvidia.com/nim/speech/26.10.0/index.html) |
| 2026-10-28 | [Speech NIM 26.10.0: ASR support matrix](https://docs.nvidia.com/nim/speech/26.10.0/reference/support-matrix/asr.html) |
| 2026-10-28 | [Riva ASR NIM (migration notice)](https://docs.nvidia.com/nim/riva/asr/latest/index.html) |
| 2026-10-28 | [Nemotron Voice Agent blueprint (GitHub)](https://github.com/NVIDIA-AI-Blueprints/nemotron-voice-agent) |
| 2026-10-28 | [NVIDIA FAQ: voice agent latency targets](https://perspectives.nvidia.com/nemotron-speech/task/faq/what-end-to-end-latency-should-i-target-for-a-conversational-voice-agent-to-feel/) |
| 2026-10-28 | [VSS 3.3.0 architecture](https://docs.nvidia.com/vss/latest/content/architecture.html) |
| 2026-10-28 | [VSS 3.3.0: Real-Time VLM](https://docs.nvidia.com/vss/3.3.0/system-components/vision-microservices/real-time-vlm.html) |
| 2026-10-28 | [VSS 2.4.1 architecture](https://docs.nvidia.com/vss/2.4.1/content/architecture.html) |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Building Multimodal AI RAG With LlamaIndex, NVIDIA NIM, and Milvus (video)](https://www.youtube.com/watch?v=NaT5Eo97_I0) |
| 2026-10-27 | [Multimodal RAG: expanding beyond text (Zilliz)](https://zilliz.com/blog/multimodal-rag-expanding-beyond-text-for-smarter-ai) |
| 2026-10-27 | [Nemotron Nano 12B v2 VL (OpenRouter)](https://openrouter.ai/models/nvidia/nemotron-nano-12b-v2-vl) |
| 2026-10-28 | [MCP specification 2026-07-28: Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools) |
| 2026-10-28 | [NIM for LLM and VLM: tool calling and MCP integration](https://docs.nvidia.com/nim/large-language-models/latest/advanced-use-cases/tool-calling-and-mcp.html) |
| 2026-10-28 | [NIM for VLMs release notes](https://docs.nvidia.com/nim/vision-language-models/1.7.0/release-notes.html) |
