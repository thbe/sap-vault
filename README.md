# SAP ABAP / Fiori Wiki

A personal, LLM-maintained knowledge base for **ABAP and SAP Fiori/UI5 development**, focused on **S/4HANA + RAP + BTP ABAP**.

## How to use

- **Drop sources** into `raw/` (articles, PDFs, code samples, screenshots).
- **Talk to the LLM agent** — say "ingest this" pointing at a file, or ask any SAP/Fiori question.
- **Browse the wiki** in Obsidian — start at [[_Index]] or open the graph view.
- The LLM owns `wiki/`, `_Index.md`, and `log.md`. The human owns `raw/`.

The full operating contract lives in [AGENTS.md](AGENTS.md). Read that first.

## Layout

| Directory | Owner | Purpose |
|---|---|---|
| `raw/` | Human | Immutable source material |
| `wiki/` | LLM | Generated, interlinked knowledge pages |
| `templates/` | Human | Reference page templates |
| `_Index.md` | LLM | Catalog of all wiki pages |
| `log.md` | LLM | Chronological activity log |
| `AGENTS.md` | Both | Schema and operating manual |

## Conventions in 30 seconds

- Obsidian wikilinks: `[[concepts/rap|RAP]]`
- Frontmatter on every page (title, type, tags, sap_release, status, sources, related)
- Versions matter — every concept/entity/pattern declares which SAP releases it applies to
- Modern S/4HANA + RAP + Clean ABAP is the default; legacy is secondary reference
- Code blocks always tagged: `abap`, `cds`, `xml`, `js`, `sql`
