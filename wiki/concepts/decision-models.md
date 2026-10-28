# Decision Models

A decision model answers typed questions about an input by returning a probability for each allowed answer, and generates no text. It targets the many small decisions inside an agent, such as routing, classification, escalation and guardrail checks, where a full generative call is slow and its output has to be parsed. The category appeared in September and October 2026, after the study guide, and no exam objective names it. TypeSafe AI coined the term "System One" model, and its Jev model set the API that later entrants copy. Nearly everything known about these models comes from vendor posts and press coverage, so their numbers are provisional.

Decision models are an alternative to the schema-constrained decoding in [Structured Outputs](structured-outputs.md). Constrained decoding still generates tokens from a restricted sampler; a decision model scores a fixed set of answers in one pass and so cannot produce anything outside the schema. Its main use is the routing step of the workflow patterns in [Agent Architecture Patterns](agent-architecture-patterns.md), where it replaces a generative classifier call.

## The interface

Jev, Cloudflare's Clef and the Stanford CLM share one request shape, described by third-party guides and a Rust client, since TypeSafe publishes no API reference of its own. The application sends a `state` (the input, such as a ticket or a page) and a set of named questions, each with instructions and criteria, of three types **[web]**:

- **`choice`** picks one of up to 255 options and returns the winner, a probability per option, and a confidence.
- **`score`** rates the state on an ordered rubric of 2 to 10 levels, returning a probability-weighted score and a probability per level.
- **`noul`** is a yes/no question, returning the probability of yes.

Questions are evaluated in parallel against the same state and cannot see each other's answers; Clef accepts up to 64 per request. Guides advise splitting a judgment that weighs several factors into separate literal questions, and pinning a model version rather than an alias such as `jev-latest` once thresholds depend on it **[web]**. The output is a calibrated probability, not an answer: the application sets thresholds and decides what to do when confidence is low, which suits escalation as described in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md).

## How they work

Cloudflare's launch post (2026-10-01) gives the most detail. Clef runs one prefill-only pass through a frozen Qwen backbone, then a routing head scores every option of every question at once and a softmax turns the scores into probabilities. The head is trained with LoRA adapters on synthetic data, using cross-entropy plus a Brier loss for calibration (answers given 0.8 should be right about 80% of the time) and a final RL stage, Reinforcement Learning for Calibrated Decisions (RLCD) **[web]**. TypeSafe names RLCD for Jev but publishes no architecture, and its claim that Jev "is not an LLM" is disputed by the security findings below.

Two open models take other routes. Strands Decider replaces Qwen3.5-2B's language-model head with a small pointer head, so it physically cannot generate text, according to secondary coverage. CLM-8B (Contrastive Language Models) is retrieval-like: a frozen Qwen3-8B encoder with separate projection heads for states and actions, trained contrastively, picks the action nearest the state in embedding space. Action embeddings can be cached, which makes it fast over many candidates, but it ranks only what it is given and will pick one even when all are wrong **[web]**.

## The models

| Model | Vendor | Released | Weights | Base and size | Context and input |
| ----- | ------ | -------- | ------- | ------------- | ----------------- |
| Jev | TypeSafe AI | 2026-09-15 | Closed | Undisclosed | 32K, text |
| CLM-8B | Stanford researchers (press says with NVIDIA) | ~2026-09-23/25 | Apache 2.0 | Qwen3-8B, frozen, plus about 20M-parameter heads | Text |
| Clef / Clef-flash | Cloudflare | 2026-10-01 | Apache 2.0 | Qwen3.8 27B / Qwen3.5 9B, as named in the post | 64K, text plus up to four images |
| Strands Decider 2B | AWS Strands Labs | 2026-10-01 | Apache 2.0, with data and scripts | Qwen3.5 2B | Text, local only |

Jev is hosted by TypeSafe and aggregators such as OpenRouter at about $0.042 per million input tokens. Clef runs on Workers AI, and Cloudflare's changelog says a Jev integration can switch by changing the endpoint and model. Comparison articles name further entrants (Perplexity, Bespoke Labs, Together AI) and an OpenAI Decisions API in limited preview, none checked individually **[web]**.

The NVIDIA connection is CLM-8B. VentureBeat calls it a Stanford and NVIDIA model, and its pre-training used about 60M Nemotron question-answer pairs, but the model card names no affiliations, so NVIDIA's formal role is unconfirmed **[web]**. No NVIDIA decision model has appeared as a NIM or announced product, so for now the NVIDIA option is to self-host an open-weight model.

## Claims and disagreements

Every benchmark comes from a vendor, and the latency figures are not comparable: Jev's are hosted round trips, while the open models were timed on their vendors' own GPUs. Cloudflare reports a median of 524 ms for Jev, 209 ms for Clef and 39 ms for Clef-flash; an independent test measured Clef at 524–726 ms; the CLM team reports 16.5 ms for CLM against 150 ms for Jev on its own benchmark; Strands reports 106–115 ms on an RTX 3090 **[web]**.

On accuracy each vendor leads on its own board. Cloudflare reports Clef ahead on intent classification (BANKING77, CLINC150) but far behind Jev on reasoning (GPQA Diamond 48.0 against 78.3), and CLM-8B is roughly level with Jev zero-shot while trailing it on BFCL v4. TypeSafe's "20–200x faster, 40–400x cheaper than LLMs" has not been independently reproduced **[web]**.

## Security

According to InfoWorld's report (2026-10-15), HiddenLayer found Jev open to adversarial text, prompt injection and TokenBreak-style character perturbations. These attacks usually fail against larger LLMs but worked on Jev out of the box. In a simulated checkout, an attacker-controlled ad steered a browser agent away from the user's purchase despite instructions to treat page text as untrusted. HiddenLayer's conclusion was that "under the hood, these are still language models at heart", and TypeSafe did not respond **[web]**. The finding comes from a single security vendor, reported through the press. The general lesson is that a decision model's guarantee is the shape of its output, not its correctness. When its output drives an action, it needs the same untrusted-input controls as any LLM step, which are covered in [Agent Security Threats and Controls](agent-security-threats-and-controls.md).

## When to use one

Decision models fit high-volume, bounded decisions where the answer set is known and latency or cost matters: intent routing, ticket triage, moderation labels, escalation gates, picking the next action from a fixed menu, and verifying candidates produced by a larger model. They do not fit open-ended reasoning, planning or anything that must produce text, and vague criteria give probabilities near 0.5 **[web]**. The natural architecture is a hybrid in which a generative LLM plans and writes and a decision model handles the frequent, cheap branch points, the same idea as the model cascades and routers in [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md).

## Interview Q&A

**What is a decision model, and how does it differ from structured output on an LLM?** It returns a calibrated probability for each answer in a declared schema, in one forward pass, with no text generated. Constrained decoding still samples tokens, just from a restricted set **[web]**.

**Where would you use one in an agent?** At routing, classification, escalation and guardrail checkpoints, and as a verifier ranking candidates. Keep a generative model for planning and writing.

**What do you do with the probability?** Set thresholds per decision, send low-confidence cases to a human or a larger model, and monitor calibration over time.

**Are decision models safer because they cannot hallucinate free text?** No. Their output space is bounded, but HiddenLayer showed injection and perturbation attacks still change Jev's decisions, so treat their inputs as untrusted **[web]**.

**Is there an NVIDIA option?** Not as a product. CLM-8B is an open Stanford model reported as an NVIDIA collaboration and trained partly on Nemotron data, and the open-weight models can be self-hosted on NVIDIA GPUs (Cloudflare tested on an H200) **[web]**.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-28 | [Cloudflare: Introducing Clef](https://blog.cloudflare.com/clef-decision-models) |
| 2026-10-28 | [Cloudflare changelog: Clef on Workers AI](https://developers.cloudflare.com/changelog/post/2026-10-01-clef-workers-ai/) |
| 2026-10-28 | [A deep dive into Clef (Flavio Copes)](https://flaviocopes.com/clef/) |
| 2026-10-28 | [TypeSafe AI launches Jev (LetsDataScience)](https://letsdatascience.com/news/typesafe-ai-launches-jev-decision-model-889a38c0) |
| 2026-10-28 | [Jev 1.13 (OpenRouter)](https://openrouter.ai/models/typesafe/jev-1.13) |
| 2026-10-28 | [typesafe-jev Rust client (docs.rs)](https://docs.rs/typesafe-jev) |
| 2026-10-28 | [HiddenLayer on Jev (InfoWorld)](https://www.infoworld.com/article/4236103/typesafes-jev-even-more-susceptible-to-attack-than-llms-says-security-vendor.html) |
| 2026-10-28 | [CLM-v0.1-8B model card (Hugging Face)](https://huggingface.co/Contrastive-LM/CLM-v0.1-8B) |
| 2026-10-28 | [Stanford and NVIDIA's CLM-8B (VentureBeat)](https://venturebeat.com/ai/stanford-and-nvidias-open-clm-8b-caches-reusable-agent-actions-and-runs-up-to-9x-faster-than-jev-in-tests) |
| 2026-10-28 | [Strands Decider 2B tested (eesel)](https://www.eesel.ai/blog/strands-decider) |
| 2026-10-28 | [Decision models: Jev vs Clef vs Strands Decider (eesel)](https://www.eesel.ai/blog/decision-models) |
