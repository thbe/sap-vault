# Wiki Activity Log

> Append-only chronological record. Newest entries at the bottom.
> Each entry: `## [YYYY-MM-DD] {ingest|query|lint|refactor|meta} | {short title}`
> Quick recall: `grep "^## \[" log.md | tail -20`

---

## [2026-05-11] meta | Initial scaffolding

- Created vault skeleton: `raw/`, `wiki/`, `templates/`, `index.md`, `log.md`, `README.md`, `AGENTS.md`.
- Wrote operating contract in `AGENTS.md` (schema, page conventions, ingest/query/lint workflows).
- Added page templates in `templates/` for concept, entity, pattern, snippet, source, topic.
- Domain: ABAP + Fiori/UI5, focused on S/4HANA + RAP + BTP ABAP. Legacy as secondary reference.
- Workflow defaults: per-source supervision, frontmatter required, release tagging required, Clean ABAP by default.
- Pages created: 0
- Pages updated: 0
- Next: human drops first source into `raw/` to begin.

---

## [2026-05-11] ingest | SAP Development Standards & Approach (Heinemann v0.3, 2023)

- Source PDF parsed via liteparse (`~/.npm-global/bin/lit`) → `/tmp/sap-std.txt`, 86 pages, 3705 lines.
- Discussed 6-bullet takeaways with human; filing plan approved as-is.
- Source page created with full TL;DR, key claims by chapter, open questions (RAP not covered, Hungarian UI5 dated, Web IDE deprecated, MV45AFZZ ECC-era, Y_NAMING_CONVENTIONS variant origin to verify).
- 3 concept drafts (`abap-naming-conventions` with full repository + internal-element tables; `clean-abap`; `mvc-in-abap`).
- 2 topic drafts (`abap-performance`, `abap-quality-gates`).
- 2 pattern pages (`table-driven-enhancement-framework` with full table schemas + dispatcher code; `brf-plus-ui-guard` with subclass skeleton).
- 1 stable snippet (`flower-box-header`).
- 5 stubs flagged for future expansion (`restful-application-programming-model`, `odata-service`, `sap-fiori-overview`, `entities/tools/atc`, `entities/tools/segw`).
- `index.md` rebuilt from empty → 14 pages.
- Pages created: [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]], [[wiki/concepts/abap-naming-conventions]], [[wiki/concepts/clean-abap]], [[wiki/concepts/mvc-in-abap]], [[wiki/concepts/odata-service]], [[wiki/concepts/restful-application-programming-model]], [[wiki/concepts/sap-fiori-overview]], [[wiki/entities/tools/atc]], [[wiki/entities/tools/segw]], [[wiki/patterns/brf-plus-ui-guard]], [[wiki/patterns/table-driven-enhancement-framework]], [[wiki/snippets/flower-box-header]], [[wiki/topics/abap-performance]], [[wiki/topics/abap-quality-gates]]
- Pages updated: [[_Index.md]]
- Contradictions: none (first source).
- Follow-ups: ingest official SAP Clean ABAP styleguide; ingest RAP development guide to fill stubs; verify SCI variant `Y_NAMING_CONVENTIONS` origin; revisit Fiori chapter against modern UI5 + TS practice.

---

## [2026-05-11] ingest | Heinemann EWM Coding Standards (S/4HANA 2023 FPS04 migration)

- Source: `/home/bendlert/Code/git/thbe-gh/EWM/` (private submodule monorepo, 58 abapGit repos). Read `AGENTS.md`, `README.md`, and `docs/migration/standards/{01-naming-conventions,02-coding-guidelines,03-code-quality-performance}.md` (1,451 lines combined).
- Discussed 6-bullet takeaways with human; 17-page filing plan approved.
- `raw/` is gitignored — customer-internal source stays private; only curated wiki published.
- Source page captures TL;DR, claims grouped by document, 6 open questions.
- 2 new concept drafts: `abapgit-serialization` (FULL vs PREFIX, `.abapgit.xml`, DEVCLASS rules), `abap-exception-handling` (`YCX_*` hierarchy, TRY/CATCH templates, RESUMABLE, FM-to-exception translation).
- 3 new pattern drafts: `multi-site-repo-architecture` (three-tier YEWM ← ZALM/ZERL + YBC/YCA, decision tree, import order), `dynamic-code-whitelist` (SUBMIT/CALL TRANSACTION remediation), `form-to-method-refactor` (mechanical recipe + prioritisation).
- 2 new topic drafts: `abap-security-checklist` (8 patterns with severity + scan playbook), `sap-forms-strategy` (BRF+ Adobe → XML → Custom Adobe → Smart → SAPScript hierarchy).
- 1 new stable snippet: `abap-doc-comments` (`"!` syntax with shorttext/parameter/raising/link tags).
- Updates to existing pages: `abap-naming-conventions` (EWM Y/Z scoping; **contradiction callout** v1-vs-EWM resolved in favor of EWM), `clean-abap` (no FORM / no dynamic code / class-based exceptions), `mvc-in-abap` (multi-site BAdI controller split), `abap-quality-gates` (per-phase ATC variants, ABAP Unit + security + perf gates, exemption process), `abap-performance` (SQL anti-pattern table, RF dialog <2s SLA, `Y/ZCL_EWM_PERF_MEASURE` infra), `flower-box-header` (EWM `Market:` field values).
- `index.md` rebuilt: 14 → 23 pages.
- Pages created: [[wiki/sources/heinemann-ewm-coding-standards]], [[wiki/concepts/abapgit-serialization]], [[wiki/concepts/abap-exception-handling]], [[wiki/patterns/multi-site-repo-architecture]], [[wiki/patterns/dynamic-code-whitelist]], [[wiki/patterns/form-to-method-refactor]], [[wiki/topics/abap-security-checklist]], [[wiki/topics/sap-forms-strategy]], [[wiki/snippets/abap-doc-comments]]
- Pages updated: [[wiki/concepts/abap-naming-conventions]], [[wiki/concepts/clean-abap]], [[wiki/concepts/mvc-in-abap]], [[wiki/topics/abap-quality-gates]], [[wiki/topics/abap-performance]], [[wiki/snippets/flower-box-header]], [[_Index.md]]
- Contradictions: **1 resolved** — Y/Z namespace scope (v1 says Y=global/Z=local-market; EWM scopes Y to shared/cross-site only and keeps Z* for site-specific to avoid rename churn). Inline `> [!warning] Contradiction` callout on `abap-naming-conventions`; EWM treated as current project standard.
- Follow-ups: confirm whether `YCL_EWM_PERF_MEASURE` is project-wide or EWM-only; verify ATC variant names; ingest SAP Clean ABAP styleguide to cross-reference; expand `abapgit-serialization` once SAP abapGit docs ingested; check if site-specific BAdI pattern deserves its own page.

---

## [2026-05-11] ingest | opencode ABAP Context Library (12 files, personal LLM context)

- Source: `~/Code/git/thbe/dotfiles/opencode/.config/opencode/context/languages/abap/` — 12 markdown files (~80 KB), the user's personal opencode LLM context library covering modern ABAP, RAP, abapGit, testing, code review.
- All 12 files fully read; 6-bullet takeaways discussed; filing plan approved (7 new + 7 expansions + 3 light additions).
- Source page captures TL;DR, key claims grouped by domain (modern ABAP syntax / RAP stack / testing / abapGit AFF / clean ABAP / code review), open questions, no cross-source contradictions; documents the internal Hungarian-notation tension within this source.
- 7 new pages: `snippets/abap-modern-syntax-cheatsheet` (prefer/over table with release gates), `concepts/cds-view-entities` (root/projection, annotations, layered modelling), `concepts/behavior-definition` (BDL: managed/unmanaged, strict(2), draft, determinations/validations/side-effects), `concepts/eml` (READ/MODIFY/EXECUTE/COMMIT, %cid, IN LOCAL MODE, failed/reported/mapped), `topics/abap-unit-testing` (test class skeleton, RISK LEVEL/DURATION, G-W-T, CDS test env), `patterns/test-doubles-and-dependency-injection` (constructor/setter/back-door, `CL_ABAP_TESTDOUBLE`), `patterns/oo-design-patterns-in-abap` (Singleton/Factory/Strategy/Observer/Facade/Template Method).
- 7 expansions: `concepts/restful-application-programming-model` (**promoted stub → draft** with full overview + BDL example + layer diagram), `concepts/abapgit-serialization` (encoding rules table, 4-phase deserialization, `#ns#` vs `(ns)` namespace encoding, AFF JSON), `concepts/clean-abap` (FINAL by default + CREATE PRIVATE factory, CX hierarchy table, Hungarian-notation note), `concepts/abap-exception-handling` (CX_*_CHECK semantics table, `RAISE EXCEPTION NEW`), `concepts/abap-naming-conventions` (object format strings + OData entity examples, Hungarian-notation note), `topics/abap-performance` (`lines( )` over `DESCRIBE`, table-expression `DEFAULT`), `topics/abap-quality-gates` (PR sizing <400 lines, review checklist, abapGit↔GitHub one-way/two-way flows).
- 3 light additions (second source backref + Sources block): `patterns/table-driven-enhancement-framework`, `patterns/brf-plus-ui-guard`, `topics/sap-forms-strategy`.
- `index.md` rebuilt: 23 → 30 pages.
- Pages created: [[wiki/sources/opencode-abap-context-library]], [[wiki/snippets/abap-modern-syntax-cheatsheet]], [[wiki/concepts/cds-view-entities]], [[wiki/concepts/behavior-definition]], [[wiki/concepts/eml]], [[wiki/topics/abap-unit-testing]], [[wiki/patterns/test-doubles-and-dependency-injection]], [[wiki/patterns/oo-design-patterns-in-abap]]
- Pages updated: [[wiki/concepts/restful-application-programming-model]] (stub → draft), [[wiki/concepts/abapgit-serialization]], [[wiki/concepts/clean-abap]], [[wiki/concepts/abap-exception-handling]], [[wiki/concepts/abap-naming-conventions]], [[wiki/topics/abap-performance]], [[wiki/topics/abap-quality-gates]], [[wiki/patterns/table-driven-enhancement-framework]], [[wiki/patterns/brf-plus-ui-guard]], [[wiki/topics/sap-forms-strategy]], [[_Index.md]]
- Contradictions: none cross-source new. **Internal tension** within this source: `clean-abap-essentials.md` forbids Hungarian prefixes while `abap-standards.md`/`quick-ref.md` prescribe them — same pragmatic-vs-purist split already noted on `concepts/abap-naming-conventions`. Documented as `> [!note]` (not `> [!warning] Contradiction`) since resolution is unchanged: follow the active project standard.
- Follow-ups: ingest official SAP Clean ABAP styleguide; ingest SAP RAP development guide; flesh out remaining stubs (`odata-service`, `sap-fiori-overview`, `entities/tools/atc`, `entities/tools/segw`); consider snippet pages for the BDL/EML examples currently embedded in concept pages.

---

## [2026-05-11] ingest | opencode Fiori Context Library (2 files, personal LLM context)

- Source: `~/Code/git/thbe/dotfiles/opencode/.config/opencode/context/languages/fiori/` — 2 markdown files (~9 KB), the user's personal opencode LLM context library for classic freestyle SAP Fiori/UI5 development.
- Both files fully read; 6-bullet takeaways discussed (smaller/narrower than ABAP library; heavy overlap with existing Fiori stubs; concrete UI5 JS + CSS + SEGW detail new to wiki; 2 internal source self-contradictions to flag; 1 cross-source contradiction on OData property naming); filing plan approved (3 new + 2 expansions + 1 light addition).
- Source page captures TL;DR, key claims grouped by domain (architecture / project structure / OData-SEGW / JS critical rules / CSS / security), open questions including dated `jQuery.sap.byId()` and misleading "use sap.* namespace" advice, and one **new cross-source contradiction** on OData property naming (auto-mapping `MATNR` per this source vs semantic `FirstName` per Heinemann standard).
- 3 new pages: `topics/ui5-coding-standards` (8-row PROHIBITED rules table, DO/DON'T table, module skeleton, Hungarian-notation note, anti-pattern walkthroughs, source self-contradiction notes), `topics/ui5-css-guidelines` (BEM, theme parameters table with `--sapNegativeColor` etc., override-safety pattern, anti-pattern reference table), `snippets/ui5-controller-template` (canonical `sap.ui.define` skeleton with onInit/onExit, private helpers, event handlers, common extensions).
- 2 expansions: `concepts/sap-fiori-overview` (**promoted stub → draft** with full landscape diagram, 3-row deployment table, decision tree, project structure, namespace rules with `sap.*` clarification note), `concepts/odata-service` (**promoted stub → draft** with SEGW vs RAP comparison table, full DPC_EXT method table with HTTP verbs, mandatory query-option handling, status-code table, **`> [!warning] Contradiction`** callout on field-naming policy).
- 1 light addition: `entities/tools/segw` — added second source backref, **promoted stub → draft**, redirected detail to `concepts/odata-service`.
- `index.md` rebuilt: 30 → 34 pages.
- Pages created: [[wiki/sources/opencode-fiori-context-library]], [[wiki/topics/ui5-coding-standards]], [[wiki/topics/ui5-css-guidelines]], [[wiki/snippets/ui5-controller-template]]
- Pages updated: [[wiki/concepts/sap-fiori-overview]] (stub → draft), [[wiki/concepts/odata-service]] (stub → draft, +contradiction callout), [[wiki/entities/tools/segw]] (stub → draft), [[_Index.md]]
- Contradictions: **1 new cross-source** — OData property naming convention (auto-map ABAP names vs semantic English names). Both defensible; resolution depends on whether the service is internal-only or partner-facing. Documented as `> [!warning] Contradiction` on `concepts/odata-service`. **2 internal** to this source: `jQuery.sap.byId()` (deprecated; guidelines doc has correct `this.byId()`) and "Use sap.* namespace only" (misleading; guidelines doc correctly says don't start namespace with `sap`). Both flagged as `> [!note]` callouts on the relevant topic/concept pages.
- Follow-ups: confirm OData property-naming policy (auto-map vs semantic) and resolve the contradiction; add Fiori Elements coverage (List Report / Object Page) — major gap for modern S/4 development; add TypeScript-based UI5 development guidance; ingest official SAP Fiori design guidelines for theming + branding; modernise Web IDE references throughout once a current SAP source is ingested.
