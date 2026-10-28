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

Internal links use relative paths and are embedded in context: every link appears inside a sentence that explains why the relationship matters. A link is a claim about a connection, and the surrounding prose is the evidence. There are no "See also" lists. If a relationship cannot be explained in a sentence, it is not meaningful enough to link.

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

Attribution is inline, in natural language, so the reader sees what claim a source supports: "According to NVIDIA's AI Factory white paper...". In addition, mark the epistemic status of substantive claims, because this wiki mixes study-guide references, added web research and synthesis:

- **[sourced]** — stated by a source in `sources.md`.
- **[web]** — from a source found during gap-filling; cite the URL.
- **[inferred]** — the LLM's synthesis, not stated by any source.

Say so when a claim rests on a single source, a search snippet, or a vendor's own benchmark. Treat Medium posts and vendor blogs as leads and ground claims in primary sources (docs, papers, specs) where possible.

### Naming Conventions

- Concepts: `{concept-name}.md`
- Components: `{component-name}.md`
- Syntheses: `{analysis-topic}.md`

Filenames are lowercase, hyphenated and descriptive.

### Writing Style

Every sentence should earn its place. A short, precise page is better than a long, vague one. Prefer prose to bullet fragments; use a list only for genuinely parallel items.

Be explicit about scope and time. This field moves fast and the study guide is from September 2025: state product names and versions as of the date checked, and say whether a behavior is spec-defined or specific to one vendor or implementation.

Each concept page ends, before the reference table, with an **Interview Q&A** section: a few likely questions with concise answers.

When new information contradicts existing wiki content, including the study guide, do not silently overwrite. Note the contradiction, cite both sources, and flag it to the user. Prefer the newer or more authoritative source in the page text, and record the disagreement.

## The Index

`wiki/index.md` is the routing layer and the first thing read when answering a query. Each entry is a link with a one-line description, grouped by category; the description should say enough to triage without opening the page. Exam objective IDs (e.g. `1.4`) may be added in parentheses so the index can be scanned by blueprint section. The index is updated on every ingest. Do not list source material in it.

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
6. Verify **link integrity**: internal links resolve, and `sources/` links exist.
7. Verify that every `ingested` row in `sources.md` is cited by at least one page's reference table.
8. Suggest new questions or sources that would fill gaps.
9. Fix what can be fixed, report the rest, and log the pass.

### Enrich

When a page would benefit from deeper context: search for primary sources (papers, specs, vendor docs, repositories), integrate findings as prose with citations, upgrade snippet-level claims to read-in-full ones, and log the enrichment. If NVIDIA repositories (NeMo Agent Toolkit, NeMo Guardrails, TensorRT-LLM) are indexed in `code-rag`, use it to ground component pages in actual code and configuration; check with `list_repos` first.
