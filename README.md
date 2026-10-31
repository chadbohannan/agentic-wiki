# Agentic AI Interview Wiki

A knowledge base on agentic AI, organized around the NVIDIA-Certified Professional: Agentic AI (NCP-AAI) exam blueprint, and written and maintained by an LLM. It exists to get you ready to explain each concept, its trade-offs and its NVIDIA implementation in an interview or on the exam.

## How to use it

**Start with the gap interview.** The intended way to consume this wiki is through the `gap-interview` skill, not by reading the pages in order. Open Claude Code in this directory and run:

```
/gap-interview
```

The interviewer asks one scenario question at a time, using a running example (a "blork" support agent), and you answer in your own words. After each answer it tells you what you got right, what you missed, and fills the gap briefly. It rates each answer Solid, Partial or Gap and records the result, so the interview picks up where it stopped the next time you run it. Ask for a status at any point to see your gap list.

The output is a prioritized list of what you don't yet know, each item pointing at the wiki page to read. Those pages are your textbook for the gaps the interview finds. You can also ask the interviewer a side question; it answers briefly and returns to the pending question.

When you want to fix a gap rather than just find it, use the `teaching` skill, which walks the same pages as a paced curriculum.

## What's here

| Path | Purpose |
| ---- | ------- |
| `.claude/skills/gap-interview/` | The interview protocol and rating scale. This is the front door. |
| `.claude/skills/teaching/` | The instructor skill and its curriculum, for closing the gaps the interview finds. |
| `interview-log.md` | Your progress: results, gap list, strengths, next question. Local and gitignored; it is created on first use. |
| `wiki/` | The pages themselves: `concepts/`, `components/` (products and libraries), `syntheses/` (the agent anatomy, the NVIDIA stack, the exam blueprint), with `index.md` as the catalog. |
| `sources/`, `sources.md` | Raw inputs and the ingest queue. |
| `TODO.md` | Claims and areas that are not yet verified. |

## Reading the pages

Claims are tagged **[sourced]** (stated by a study-guide source) or **[web]** (found during gap-filling). Untagged prose is the wiki's own judgment. Product status in this field moves quickly, so treat `wiki/syntheses/nvidia-agentic-stack.md` as the place to check what is current, and check `TODO.md` before relying on an area it flags.

## Maintaining the wiki

The wiki's schema, page conventions and operating rules (ingest, query, lint, enrich) are in `AGENTS.md`. They apply to whoever edits wiki content, and the interview and teaching skills deliberately do not edit pages.
