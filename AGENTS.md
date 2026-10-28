# Agentic AI Interview Wiki — Schema & Operating Rules

This is a persistent, LLM-maintained knowledge base on the state of the art in agentic AI, organized around the NVIDIA-Certified Professional: Agentic AI (NCP-AAI) exam blueprint. The LLM writes and maintains all wiki content. The human curates sources, directs investigation, and asks questions.

## Purpose

The primary purpose is to build a high-quality, current snapshot of the discipline as the study guide's references describe it, extended with web research where the references are thin or dated. The secondary purpose is interview preparation: a reader should be able to explain each concept, its trade-offs, and its NVIDIA-specific implementation. Exam weights are a guide to emphasis, not a goal.

## Directory Layout

```
agentic-cert/
├── AGENTS.md          # This file — schema and operating rules (CLAUDE.md imports it)
├── notes.md           # The user's own notes (exam blueprint table); treat as a source, do not edit
├── TODO.md            # Actionable gaps and follow-ups collected during ingests
├── sources.md         # Ingest queue: every study-guide link, grouped by exam section, with a status
├── sources/           # Raw, immutable inputs (the LLM adds files here only when the user directs a download, and never modifies them)
├── staging/           # Per-section findings written by ingest subagents; working material, not wiki content
└── wiki/              # LLM-generated and LLM-maintained markdown pages
    ├── index.md       # Content catalog — the routing layer
    ├── log.md         # Chronological record of all operations
    ├── concepts/      # Techniques, patterns, protocols, and failure modes
    ├── components/    # Specific products, libraries, and services (e.g. NeMo Agent Toolkit, NIM, Triton)
    └── syntheses/     # Cross-cutting analyses, comparisons, and exam-blueprint maps
```

Add other categories (e.g. `runbooks/`, `questions/`) only when pages that do not fit above start to accumulate.

## Page Conventions

Wiki pages are prose documents with no YAML frontmatter. The directory determines the category and the first heading is the title.

Internal links use relative paths and are embedded in context: every link appears inside a sentence that explains why the relationship matters. A link is a claim about a connection, and the surrounding prose is the evidence. There are no "See also" lists, including the prose kind: a paragraph near the top that only lists related pages ("X is covered in [A], Y in [B]") is a See-also list and is not allowed. Place each link at the point in the body where the relationship does work. If a relationship cannot be explained in a sentence, it is not meaningful enough to link.

Pages are written one per concept, not one per exam objective. When two objectives touch the same concept, they share a page; the mapping from objectives to pages lives in `syntheses/exam-blueprint.md` and as IDs in `index.md`, so pages do not open with "This page covers exam objective X".

Each page ends with a table of references that corroborates `log.md`: one row per source with the date of its most recent impact on the page.

```markdown
# Page Title

Opening paragraph establishing what this page covers and why it matters.

Body paragraphs with contextual links woven in.

| Ingest Date | Source |
| ----------- | ------ |
| 2026-10-27 | [study-guide.pdf](../../sources/nvidia-study-guide/study-guide.pdf) |
| 2026-10-27 | [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) |
```

Only files in `sources/` and properly formed URLs are written as links. Paths outside the project do not port between checkouts.

### Provenance

Attribution is inline, in natural language, so the reader sees what claim a source supports: "According to NVIDIA's AI Factory white paper...". In addition, tags mark where outside evidence enters, because this wiki mixes study-guide references with added web research:

- **[sourced]** — stated by a source in `sources.md`.
- **[web]** — from a source found during gap-filling; cite the URL.

Untagged prose is the wiki's own reasoning: trade-offs, recommendations and synthesis. That is the default voice of an LLM-written wiki, so it carries no tag. The consequence is a writing rule: a sentence that reads as a fact (a version, a number, a behaviour, a date, an absence such as "NVIDIA has no X") must either carry a tag or be worded as judgment ("in practice", "probably", "the safer reading is"). Where reasoning builds on a source, put the tag on the sourced sentence and let the inference follow in words ("which suggests...").

A tag ends the sentence it supports. When consecutive sentences rest on the same evidence, one tag at the end of the run covers them, and the prose should make the run clear, for example by opening with the attribution. Tags may be combined (`[sourced, web]`), and `snippet` may be appended (`[web, snippet]`) when a claim rests only on a search snippet or summary. No other words go inside a tag. Caveats that change how far a claim can be trusted (a vendor's own benchmark, a single secondary source, conflicting reports) belong in the sentence, where the reader sees them. Notes about the research process (what was fetched, what returned 403, what was read in full) belong in `log.md`, `TODO.md` or `sources.md`, never on a page.

Say so when a claim rests on a single source, a search snippet, or a vendor's own benchmark. Treat Medium posts and vendor blogs as leads and ground claims in primary sources (docs, papers, specs) where possible.

### Naming Conventions

- Concepts: `{concept-name}.md`
- Components: `{component-name}.md`
- Syntheses: `{analysis-topic}.md`

Filenames are lowercase, hyphenated and descriptive.

### Writing Style

Every sentence should earn its place. A short, precise page is better than a long, vague one. Prefer prose to bullet fragments; use a list only for genuinely parallel items.

Be explicit about scope and time. This field moves fast and the study guide is from September 2025: state product names and versions as of the date checked, and say whether a behavior is spec-defined or specific to one vendor or implementation. The reference table's ingest date is the page's "as of" date, so headings and routine prose carry no date stamps; a page states a product's version once, near the top. Product status, deprecations and successors are recorded in `syntheses/nvidia-agentic-stack.md`; other pages mention a deprecation only where it changes the advice on that page, and link there rather than restating dates.

Each concept page ends, before the reference table, with an **Interview Q&A** section: a few likely questions with concise answers (four or more on component pages). When a concept has an NVIDIA implementation, it is described in a section headed `## In NVIDIA's stack`, placed immediately before Interview Q&A, so the interview angle is always in the same place.

When new information contradicts existing wiki content, including the study guide, do not silently overwrite. Note the contradiction, cite both sources, and flag it to the user. Prefer the newer or more authoritative source in the page text, and record the disagreement.

## The Index

`wiki/index.md` is the routing layer and the first thing read when answering a query. Each entry is a link with a one-line description, grouped by the seven parts of an agent defined in `syntheses/agent-anatomy.md` (model, context, tools, loop and harness, memory and state, environment, control and observation), plus syntheses and governance; the description should say enough to triage without opening the page. Exam objective IDs (e.g. `1.4`) may be added in parentheses so the index can be scanned by blueprint section. The index is updated on every ingest. Do not list source material in it.

## The Log

`wiki/log.md` is a chronological, append-only record. Each entry is one line:

```
[YYYY-MM-DD] operation | [Subject](../sources/path) — brief description
```

For ingests the subject is a relative link to the source (or the source set's directory). Other operations use plain text subjects. Operations: `ingest`, `query`, `lint`, `enrich`, `update`.

## Operations

### Ingest

When a source, or a batch of sources from `sources.md`, is to be added:

1. **Read** the source fully; fetch it into `sources/{source-name}/` if the user wants a local copy.
2. **Discuss** key takeaways with the user, briefly: what matters, what is surprising, what contradicts existing pages or the study guide.
3. **Create or update** pages: a concept or component page for each substantive topic, with contextual cross-links on every page touched, including existing pages that should now link to the new one. Update the reference table of each.
4. **Update `index.md`** for every page touched.
5. **Append to `log.md`.**
6. **Update `sources.md`** with the source's status (`ingested`, `thin`, `failed`) and a note when the source is wrong, dead or superseded.
7. **Report** what was created or updated, what connections were found, and what gaps remain.

For large batches, ingest subagents may research one exam section each and write findings to `staging/sec-<n>.md` without touching the wiki. The main session then merges staging into pages: deduplicating overlaps, resolving conflicts, verifying claims that matter before stating them as fact, and prefixing source IDs with the section number so references stay unambiguous.

### Query

1. Read `index.md` to find relevant pages, then read them.
2. Search the web if the question needs more current or primary grounding.
3. Synthesize an answer with citations to wiki pages and original sources.
4. If the answer is substantial and reusable, offer to file it as a page.
5. Log the query.

### Lint

When asked for a health check, or periodically as the wiki grows:

1. Check for **contradictions** between pages.
2. Find **orphan pages**, meaning pages no other page links to (links from `index.md` do not count).
3. Identify **mentioned-but-missing** entities that deserve a page (e.g. NIM, Triton, TensorRT-LLM, NeMo Guardrails).
4. Flag **stale pages**: product names, versions, deprecations and protocol specs that have likely moved. Re-check these against primary sources.
5. Flag claims that rest only on **search snippets** or vendor self-reported numbers, and verify or soften them.
6. Flag **untagged factual claims**: sentences that read as facts but carry no tag. Source them or reword them as judgment.
7. Verify **link integrity**: internal links resolve, and `sources/` links exist.
8. Verify that every `ingested` row in `sources.md` is cited by at least one page's reference table.
9. Suggest new questions or sources that would fill gaps.
10. Fix what can be fixed, report the rest, and log the pass.

### Enrich

When a page would benefit from deeper context: search for primary sources (papers, specs, vendor docs, repositories), integrate findings as prose with citations, upgrade snippet-level claims to read-in-full ones, and log the enrichment. Ground component pages for NVIDIA open-source projects (NeMo Agent Toolkit, NeMo Guardrails, TensorRT-LLM) in their repositories' docs and example configurations.
