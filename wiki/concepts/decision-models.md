# Decision Models

A decision model answers typed questions about an input by returning a probability for each allowed answer, and generates no text. The category appeared in September and October 2026, after the study guide was written. It targets the many small decisions inside an agent, such as routing, classification, escalation and guardrail checks, where a full generative LLM call is slow and its output has to be parsed. TypeSafe AI coined the term "System One" model for it, and its Jev model is the reference API that later entrants copy. No exam objective names decision models. They are relevant to routing and decision-making (1.x, 5.x), cost tuning (3.x) and guardrails (9.x). Everything on this page dates from 2026-09-15 onwards and rests largely on vendor posts and press coverage, so treat its numbers as provisional.

Decision models are an alternative to the schema-constrained decoding in [Structured Outputs](structured-outputs.md). Constrained decoding still generates tokens and restricts the sampler. A decision model scores a fixed set of answers in one pass, so it cannot produce anything outside the schema. Its main use is the routing step of [Agent Architecture Patterns](agent-architecture-patterns.md) and the model-routing lever in [Tuning Agents for Accuracy, Latency and Cost](agent-tuning-accuracy-latency-cost.md).

## The interface

Jev, Cloudflare's Clef and the Stanford CLM all share one request shape, which third-party guides and a Rust client describe as follows. The application sends a `state`, the input such as a ticket or a page, and a set of named questions, each with instructions and criteria. There are three question types **[web, Cloudflare changelog and blog; Jev details from third-party guides, since TypeSafe's own API reference was not found]**:

- **`choice`** picks one of up to 255 options and returns the winner, a probability per option, and a confidence.
- **`score`** rates the state on an ordered rubric of 2 to 10 levels, returning a probability-weighted score and a probability per level.
- **`noul`** is a yes/no question, returning the probability of yes.

Questions are evaluated in parallel against the same state, and no question sees another's answer. Clef accepts up to 64 questions per request **[web]**. Guides advise splitting a judgment that weighs several factors into separate literal questions. They also advise pinning a model version (rather than an alias such as `jev-latest`) once production thresholds depend on the model's behaviour **[web, third-party guides]**. The output is a calibrated probability, not an answer: the application chooses thresholds and decides what to do when confidence is low, which suits escalation to a human as described in [Human-in-the-Loop Patterns](human-in-the-loop-patterns.md) **[inferred]**.

## How they work

Cloudflare's launch post (2026-10-01) describes Clef's internals in the most detail. Clef runs a single prefill-only pass through a frozen Qwen backbone. A routing head with two-stage attention then scores every allowed option of every question at once, and a softmax turns the scores into probabilities. The head is trained with rank-256 LoRA adapters on synthetic data that shuffles field order, prompts and schemas. The losses are label-smoothed cross-entropy plus a Brier loss for calibration, followed by an RL stage called Reinforcement Learning for Calibrated Decisions (RLCD) that gives partial credit when a score lands on the level next to the right one **[web, vendor blog]**. Calibration means that answers given a probability of 0.8 should be right about 80% of the time. TypeSafe names RLCD for Jev but publishes no architecture, and its claim that Jev "is not an LLM" is disputed (see Security below) **[web]**.

Strands Decider removes the language-model head from Qwen3.5-2B and adds a small "pointer head" with a rank-16 LoRA adapter, so it physically cannot generate text **[web, secondary]**. CLM-8B (Contrastive Language Models) takes a different, retrieval-like approach. A frozen Qwen3-8B encoder has two projection heads, one for states and one for actions, trained with a bidirectional InfoNCE loss, and the action closest to the state in embedding space wins. Action embeddings can be cached, so each request only encodes the state, which makes CLM fast when there are many candidates. Because it ranks only the candidates it is given, it will pick one even when all of them are wrong **[web, Hugging Face model card read]**.

## The models (as of 2026-10-28)

| Model | Vendor | Released | Weights | Base and size | Context and input |
| ----- | ------ | -------- | ------- | ------------- | ----------------- |
| Jev | TypeSafe AI | 2026-09-15 | Closed | Undisclosed | 32K, text |
| CLM-8B | Stanford researchers (press says with NVIDIA) | ~2026-09-23/25 | Apache 2.0 | Qwen3-8B, frozen, plus about 20M-parameter heads | Text |
| Clef / Clef-flash | Cloudflare | 2026-10-01 | Apache 2.0 | Qwen3.8 27B / Qwen3.5 9B, as named in the post | 64K, text plus up to four images |
| Strands Decider 2B | AWS Strands Labs | 2026-10-01 | Apache 2.0, with data and scripts | Qwen3.5 2B | Text, local only |

Sources for the table are the vendor posts, the CLM model card and press coverage. Smaller entrants named in comparison articles are Perplexity's pplx-decider-v1-27b, Bespoke Labs' Bespoke-Nimble-9B-v2, Together AI's Tev1-4B, Kev 9B and Laya. OpenAI's Decisions API is in limited preview **[web, secondary; not individually checked]**. Jev is hosted through TypeSafe, OpenRouter and other aggregators at about $0.042 per million input tokens, with output free at launch **[web, aggregator listings]**. Clef runs on Workers AI as `@cf/cloudflare/clef` and `@cf/cloudflare/clef-flash`, and the Cloudflare changelog says an existing Jev integration can switch by changing the endpoint and model **[web, vendor docs]**.

The NVIDIA connection is CLM-8B. Press coverage (VentureBeat, 2026-09-25) calls it a Stanford and NVIDIA model, and its pre-training used about 60M Nemotron question-answer pairs. The model card lists seven authors, including Christopher Ré, Marco Pavone and Azalia Mirhoseini, but names no affiliations, so NVIDIA's formal role is unconfirmed **[web]**. NVIDIA has not shipped a decision model as a NIM or product **[inferred from no search result]**.

## Claims and disagreements

All benchmark numbers come from the vendors themselves, and their latency figures disagree:

| Source | Jev | Clef | Clef-flash | Strands Decider |
| ------ | --- | ---- | ---------- | --------------- |
| TypeSafe | 70–500 ms end to end | | | |
| Cloudflare, median | 524 ms | 209 ms | 38.8 ms | |
| Independent test from Italy | | 524–726 ms | 191–205 ms | |
| CLM team, T-Rex game benchmark | 149.8 ms (CLM: 16.5 ms) | | | |
| Strands reports, on an RTX 3090 | | | | 106–115 ms |

The measurements are not comparable: Jev's are hosted round trips, while the open models were measured on the vendors' own GPUs **[web]**.

On accuracy, each vendor leads on its own benchmark board. Cloudflare reports Clef ahead on BANKING77 and CLINC150 intent classification, Jev ahead on When2Call and BRIGHT, and Clef far behind Jev on reasoning (GPQA Diamond 48.0 against 78.3) **[web, vendor blog and a third-party write-up]**. CLM-8B is roughly level with Jev zero-shot and trails it on BFCL v4 (95.2% against 99.2%). As a fine-tuned verifier ranking coding-agent candidates, CLM reports 81.6% on DeepSWE and 87.6% on Terminal-Bench 2.1 **[web, model card and VentureBeat]**. Strands Decider's JevBench score is reported as both 72% and 0.762 **[web, conflicting secondary sources]**. TypeSafe's "20–200x faster, 40–400x cheaper than LLMs" claim has not been independently reproduced **[web]**.

## Security

According to InfoWorld's report (2026-10-15), HiddenLayer found Jev open to adversarial text, prompt injection and TokenBreak-style character perturbations. These attacks usually fail against larger LLMs but worked on Jev out of the box. In a simulated checkout, an attacker-controlled ad steered a browser agent away from the user's purchase despite instructions to treat page text as untrusted. HiddenLayer's conclusion was that "under the hood, these are still language models at heart", and TypeSafe did not respond **[web, single security vendor via press]**. The general lesson is that a decision model's guarantee is the shape of its output, not its correctness. When its output drives an action, it needs the same untrusted-input controls as any LLM step, which are covered in [Agent Security Threats and Controls](agent-security-threats-and-controls.md) **[inferred]**.

## When to use one

Decision models fit high-volume, bounded decisions where the answer set is known and latency or cost matters:
- intent routing;
- ticket triage;
- moderation and trust-and-safety labels;
- "should this escalate" gates;
- picking the next action from a fixed menu;
- verifying candidates produced by a larger model.

They do not fit open-ended reasoning, planning, maths or anything that must produce text, and their accuracy falls off on reasoning-heavy questions. Vague criteria give probabilities near 0.5 **[web, vendor limitations plus the Flavio Copes test; inferred synthesis]**. The natural architecture is a hybrid: a generative LLM plans and writes, and a decision model handles the frequent, cheap branch points around it, which is the same idea as the model cascades and routers on the tuning page **[inferred]**.

## Interview Q&A

**What is a decision model, and how does it differ from structured output on an LLM?** It returns a calibrated probability for each answer in a declared schema, in one forward pass, with no text generated. Constrained decoding still samples tokens, just from a restricted set **[web, inferred]**.

**Where would you use one in an agent?** At routing, classification, escalation and guardrail checkpoints, and as a verifier ranking candidates. Keep a generative model for planning and writing **[inferred]**.

**What do you do with the probability?** Set thresholds per decision, send low-confidence cases to a human or a larger model, and monitor calibration over time **[inferred]**.

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
