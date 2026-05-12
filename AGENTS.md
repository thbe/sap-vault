# AGENTS.md — Wiki Schema & Operating Manual

> This file is the operating contract between the human and the LLM agent maintaining this wiki.
> The human curates sources and asks questions. The LLM owns the `wiki/` directory and all its bookkeeping.
> Read this file at the start of every session.

---

## 1. Purpose

A professional knowledge base for **ABAP and SAP Fiori/UI5 development**, focused on **S/4HANA + RAP (RESTful Application Programming Model) + BTP ABAP (Steampunk)**, with legacy ABAP (ECC, classic Dynpro, BOPF, Gateway) treated as secondary reference.

Goals:
- Compound knowledge across articles, docs, OSS notes, books, code samples, and personal experience.
- Maintain a single source of truth on concepts (RAP, CDS, Clean ABAP, OData, UI5 patterns).
- Track release/version applicability so we never confuse 7.50 vs S/4 2023 vs Steampunk.
- Make code patterns and snippets discoverable and reusable.

---

## 2. Directory layout

```
raw/                  # IMMUTABLE sources. LLM reads but never modifies.
  articles/           # Web clips, blog posts (SAP Community, Medium, etc.)
  docs/               # Official SAP Help excerpts, release notes
  pdfs/               # Books, whitepapers, OSS notes (PDF)
  code/               # Raw code samples to analyze
  assets/             # Images, diagrams, screenshots

wiki/                 # LLM-OWNED. Generated, interlinked markdown.
  concepts/           # Big ideas: RAP, CDS, Clean ABAP, OData, BOPF, OO ABAP
  entities/
    abap/             # CL_*, IF_*, function modules, BAPIs, transactions
    cds/              # CDS views, behavior definitions, projections
    fiori/            # Fiori apps, UI5 controls, annotations, Elements
    tools/            # ADT, BAS, Fiori Tools, abapGit, transport tools
  patterns/           # Reusable design patterns + anti-patterns (named)
  snippets/           # Curated code snippets with explanation & context
  sources/            # One summary page per ingested source
  topics/             # Cross-cutting: performance, security, testing, migration

templates/            # Page templates (reference, not active wiki content)
index.md              # Catalog of all wiki pages (LLM-maintained)
log.md                # Append-only chronological activity log
README.md             # Brief orientation for humans
AGENTS.md             # This file
```

**Rules:**
- LLM **never** writes to `raw/`. Only the human adds sources there.
- LLM **fully owns** `wiki/`, `index.md`, `log.md`. Edits freely.
- `templates/` is reference material — copy from it, don't modify in flight.

---

## 3. Page conventions

### 3.1 Frontmatter (required on every wiki page)

```yaml
---
title: "Page Title"
type: concept | entity | pattern | snippet | source | topic
tags: [rap, cds, fiori, abap, ...]
sap_release: ["S/4HANA 2023", "BTP ABAP 2402"]   # All releases this applies to
status: stub | draft | stable | deprecated        # Page maturity
sources: ["[[sources/some-source]]", ...]         # Backing sources
related: ["[[concepts/rap]]", ...]                # Manually curated cross-refs
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

For `entity` pages add:
```yaml
entity_kind: class | interface | function_module | bapi | tcode | cds_view | bdef | fiori_app | ui5_control | annotation
sap_object_name: "CL_SALV_TABLE"   # Verbatim SAP technical name when applicable
```

For `snippet` pages add:
```yaml
language: abap | js | xml | cds | yaml
runnable: true | false
```

### 3.2 Filename convention

- `kebab-case.md`
- For SAP technical objects, use the lowercased technical name: `cl-salv-table.md`, `bapi-salesorder-create.md`, `va01.md`.
- Concepts use natural names: `restful-application-programming-model.md` (with shorter aliases noted in frontmatter if helpful).

### 3.3 Wikilinks

Always use Obsidian wikilinks: `[[concepts/rap|RAP]]`, `[[entities/abap/cl-salv-table]]`. Prefer the relative path form so links work even if titles change. Use the alias `|` for readable inline text.

### 3.4 Standard sections (concept pages)

```markdown
# {Title}

> One-sentence definition.

## Overview
What it is, why it exists, when to use it.

## Key ideas
Bullet points of essential concepts.

## How it works
Mechanics, lifecycle, components.

## Code example
Minimal runnable example with explanation.

## Release & version notes
What's available where; gotchas across releases.

## Related
- [[...]]

## Anti-patterns / pitfalls
Common mistakes.

## Sources
- [[sources/...]]
```

Adapt freely — these are starting points, not handcuffs.

### 3.5 Code blocks

Always tag the language: ` ```abap `, ` ```js `, ` ```xml `, ` ```cds `, ` ```sql `. Include a brief comment header noting the SAP release if behavior is version-specific.

### 3.6 Versioning notes

When a feature is release-specific, call it out explicitly:

> [!info] Available since
> S/4HANA 2022, BTP ABAP 2208. Not available in ECC.

Use Obsidian callouts (`> [!note]`, `> [!warning]`, `> [!tip]`, `> [!info]`) for emphasis.

---

## 4. Operations

### 4.1 INGEST — adding a new source

When the human points to a file in `raw/` (or pastes content) and says "ingest this":

1. **Read** the source fully. For PDFs/images use the appropriate skill (`liteparse` for unstructured docs, `analysis-vision` for diagrams/screenshots).
2. **Discuss** key takeaways with the human in 3-6 bullet points before writing anything. Confirm direction.
3. **Create** `wiki/sources/{slug}.md` — a summary page with:
   - Frontmatter (type: source)
   - Bibliographic info (author, date, URL/file path, format)
   - 1-paragraph TL;DR
   - Key claims (bulleted, each with section/page reference)
   - Code samples worth keeping (or links to extracted snippets)
   - Open questions / things to verify
4. **Identify** which existing wiki pages this source touches. List them.
5. **Update** those pages: add new info, refine claims, flag contradictions inline using:
   > [!warning] Contradiction
   > [[sources/old]] claims X; [[sources/new]] claims Y. Reason: ...
6. **Create** new entity/concept/pattern pages as needed (start as `status: stub` if thin).
7. **Add** new code samples to `wiki/snippets/` if reusable.
8. **Update** `index.md` (new pages added, summaries refreshed if needed).
9. **Append** entry to `log.md` (see §6).
10. **Report** to the human: pages created, pages updated, contradictions found, suggested follow-ups.

A typical ingest touches 5-15 wiki pages. That's the point.

### 4.2 QUERY — answering a question

1. Read `index.md` first to find candidate pages.
2. Read the relevant pages (and follow wikilinks one hop if needed).
3. Synthesize an answer with **inline wikilink citations** to the wiki pages used.
4. If the answer reveals a gap (page missing, contradiction, stale info) — note it and offer to fix.
5. **Offer to file the answer** as a new wiki page if it's substantive (analysis, comparison, decision). Don't auto-file — ask first.
6. Append a short entry to `log.md`.

### 4.3 LINT — periodic health check

When the human says "lint the wiki":

1. **Orphans**: pages with no inbound wikilinks.
2. **Stubs**: `status: stub` pages older than N days that should be expanded.
3. **Contradictions**: scan for `> [!warning] Contradiction` callouts and surface unresolved ones.
4. **Stale releases**: pages tagged with old releases that may need updating.
5. **Missing cross-refs**: if page A mentions concept B by name but doesn't link, suggest the link.
6. **Index drift**: pages that exist but aren't in `index.md`, or vice versa.
7. **Naming inconsistencies**: same SAP object referenced under different filenames.
8. **Suggested research**: gaps where a web search or new source would help.

Output a structured report. Don't auto-fix — propose changes and let the human prioritize.

### 4.4 REFACTOR — restructuring

When pages get long, split them. When categories get crowded, subdivide. Always update wikilinks across the wiki when renaming/moving (use grep first to find references).

---

## 5. index.md format

Grouped by category. Each entry: `- [[path/to/page|Title]] — one-line summary [status]`.

```markdown
# Wiki Index

_Last updated: YYYY-MM-DD · {N} pages_

## Concepts
- [[concepts/rap|RAP]] — RESTful Application Programming Model for ABAP [stable]
- ...

## Entities — ABAP
- ...

## Entities — CDS
- ...

## Entities — Fiori
- ...

## Entities — Tools
- ...

## Patterns
- ...

## Snippets
- ...

## Topics
- ...

## Sources
- ...
```

Keep it tight. The index is for fast scanning.

---

## 6. log.md format

Append-only. Every entry starts with the same prefix so it's grep-able:

```
## [YYYY-MM-DD] {kind} | {short title}

- Summary line.
- Pages created: [[...]], [[...]]
- Pages updated: [[...]], [[...]]
- Notes / follow-ups.
```

`{kind}` ∈ `ingest | query | lint | refactor | meta`.

Quick recall: `grep "^## \[" log.md | tail -20`.

---

## 7. Workflow defaults (current)

- **Per-source supervision**: discuss takeaways before writing. Don't batch-ingest unless the human says so.
- **Code is first-class**: prefer extracting reusable snippets to `wiki/snippets/` over leaving them buried in source summaries.
- **Release tagging**: every concept/entity/pattern page must specify `sap_release`. If unknown, write `["unknown"]` and flag for follow-up.
- **Modern bias**: when patterns differ between legacy and modern, lead with S/4HANA + RAP + Clean ABAP. Note legacy as "Legacy reference" subsection if relevant.
- **Clean ABAP by default**: code samples follow Clean ABAP guidelines unless the page is explicitly about legacy.

These are evolving — update this section as workflow preferences emerge.

---

## 8. Skills to leverage

- `obsidian-markdown` — wikilinks, callouts, frontmatter syntax.
- `obsidian-cli` — for vault operations if/when needed.
- `liteparse` — parsing PDFs and unstructured docs in `raw/pdfs/`.
- `analysis-vision` — for diagrams, screenshots, architecture pictures in `raw/assets/`.
- `obsidian-bases` — for building dynamic views over the wiki via `.base` files.

Load these on-demand via the `skill` tool, not preemptively.

### 8.1 External consumption: the `llm-wiki` skill

This vault is registered with the `llm-wiki` skill (in `~/.config/opencode/skills/llm-wiki/registry.md` as `sap-vault`). When an external agent invokes that skill on an SAP topic, the workflow is:

1. Skill resolves the vault by trying registered candidate paths in order.
2. Skill reads **this `AGENTS.md`** as the authoritative operating contract.
3. Skill reads `index.md` (see §5) to discover pages.
4. Skill reads relevant pages, follows wikilinks one hop, respects `status` and `sap_release` tags from frontmatter (see §3.1).
5. Skill synthesizes an answer with wikilink citations, flags gaps, and offers to file findings back — but **never edits this vault from outside**. Edits happen inside the vault per §4.

Sections that are **load-bearing for external agents**:
- §2 (directory layout) — so the skill knows where to look.
- §3 (page conventions, especially §3.1 frontmatter) — so the skill can interpret status/version/source metadata correctly.
- §5 (index.md format) — so the skill can navigate.

Changes to those three sections may break external consumption. Update the skill's `registry.md` triggers if domain coverage expands.

---

## 9. What the LLM should NOT do

- Don't invent SAP technical details. If unsure about a class name, table, or release availability — say so and mark `status: stub` or add a `> [!warning] Unverified` callout.
- Don't silently overwrite a page during ingest. Diff-style mental model: add, refine, flag — don't erase.
- Don't write to `raw/`.
- Don't skip the `log.md` entry.
- Don't create deeply nested directories beyond what's defined here without proposing the change first.
