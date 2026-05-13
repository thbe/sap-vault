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

---

## [2026-05-13] meta | Promote 7 drafts to stable (quick-win batch)

- Policy locked in with human: multi-source corroboration preferred; single-source acceptable for internal governance topics (Heinemann EWM, opencode libraries, dev standard v1). `*(future)*` wikilinks acceptable as documented future work — not promotion blockers.
- Audited all 26 draft pages against the stable bar (`entities/tools/atc.md`, sources, snippets). Identified 7 quick wins: contradictions resolved, sources cited, no missing prereqs.
- Fixed broken wikilink in `mvc-in-abap.md`: `concepts/cds-views` → `concepts/cds-view-entities` (also added `behavior-definition` link).
- Pages created: none.
- Pages updated (status: draft → stable): [[wiki/concepts/mvc-in-abap]], [[wiki/concepts/abap-naming-conventions]], [[wiki/concepts/abap-exception-handling]], [[wiki/concepts/sap-fiori-overview]], [[wiki/topics/ui5-coding-standards]], [[wiki/topics/ui5-css-guidelines]], [[wiki/topics/sap-forms-strategy]], [[_Index.md]].
- Contradictions: none new.
- Follow-ups: small-effort batch (~1-2 hr) — promote remaining drafts that just need `*(future)*` policy applied: `clean-abap`, `abapgit-serialization`, `restful-application-programming-model`, `cds-view-entities`, `behavior-definition`, `eml`, `abap-performance`, `abap-quality-gates`, `abap-security-checklist`, `abap-unit-testing`, `test-doubles-and-dependency-injection`, `oo-design-patterns-in-abap`, `multi-site-repo-architecture`, `table-driven-enhancement-framework`, `dynamic-code-whitelist`, `form-to-method-refactor`. Real work remaining: `concepts/odata-service` (resolve naming contradiction + ingest V4/RAP source), `patterns/brf-plus-ui-guard` (create `patterns/brfplus-application-exit` stub prereq), `entities/tools/segw` (expand or demote to stub).

---

## [2026-05-13] meta | Promote 16 drafts to stable (small-effort batch)

- Policy applied (locked at m0006): multi-source preferred; single-source acceptable for internal governance topics; `*(future)*` wikilinks acceptable as documented future work.
- All 16 pages reviewed: meet stable bar (multi-source backing or scoped single-source, runnable examples, resolved/no contradictions, no broken links).
- Pages promoted draft → stable (status: stable, updated: 2026-05-13):
  - Concepts: [[wiki/concepts/clean-abap]], [[wiki/concepts/abapgit-serialization]], [[wiki/concepts/restful-application-programming-model]], [[wiki/concepts/cds-view-entities]], [[wiki/concepts/behavior-definition]], [[wiki/concepts/eml]]
  - Topics: [[wiki/topics/abap-performance]], [[wiki/topics/abap-quality-gates]], [[wiki/topics/abap-security-checklist]], [[wiki/topics/abap-unit-testing]]
  - Patterns: [[wiki/patterns/test-doubles-and-dependency-injection]], [[wiki/patterns/oo-design-patterns-in-abap]], [[wiki/patterns/multi-site-repo-architecture]], [[wiki/patterns/table-driven-enhancement-framework]], [[wiki/patterns/dynamic-code-whitelist]], [[wiki/patterns/form-to-method-refactor]]
- index.md: 16 `[draft]` tags flipped to `[stable]`.
- Remaining drafts (3): `concepts/odata-service` (naming contradiction + V4/RAP source ingest), `patterns/brf-plus-ui-guard` (needs `patterns/brfplus-application-exit` stub prereq), `entities/tools/segw` (expand or demote to stub).
- Follow-ups: ingest SAP Clean ABAP styleguide (referenced as future in clean-abap.md); ingest abapGit official docs (referenced in abapgit-serialization.md); ingest SAP RAP development guide (referenced in restful-application-programming-model.md); create stub entity pages for `code-inspector`, `slin`, `abap-unit`, `sat`, `st05`, `code-vulnerability-analyzer`, `abapgit`, `parallel-cursor` pattern.

---

## [2026-05-13] meta | Resolve final 3 drafts (odata-service, brf-plus-ui-guard, segw)

- User decisions (m0031): odata-service → split-by-audience policy + future-work conversion + promote; brf-plus-ui-guard → promote (acceptable per *(future)* policy); segw → demote to stub.
- [[wiki/concepts/odata-service]] draft → stable: replaced "Contradiction" warning callout with resolved split-by-audience policy info callout (internal = ABAP UPPER auto-map; partner-facing = UpperCamelCase semantic). Renamed "Open topics" → "Documented future work" with explicit future-ingest pointers (RAP V4 workflow, V4 vs V2, auth/CORS/CSRF, `$batch`).
- [[wiki/patterns/brf-plus-ui-guard]] draft → stable: no body changes. `*(future)* patterns/brfplus-application-exit` reference and release-dependent method-signature note retained as documented open work per policy.
- [[wiki/entities/tools/segw]] draft → stub: rewrote Status section to declare "stub — pointer page" with explicit expansion path (MPC vs DPC structure, generated artefact lifecycle, *_DPC_EXT redefinition mechanics).
- index.md: 2 [draft] → [stable], 1 [draft] → [stub].
- All 26 original drafts now resolved: 23 → stable, 1 → stub, 0 remaining draft.
- Follow-ups: dedicated patterns/brfplus-application-exit page; OData V4 vs V2 topic page; OData security topic (auth/CORS/CSRF); RAP service-binding expansion on RAP concept page; SEGW stub expansion when needed.

---

## [2026-05-13] meta | Create outstanding stub pages (tools, patterns, topics)

- Executed LLM-doable follow-ups from prior draft-resolution batch (m0034 "Do all outstanding"). Source-ingestion follow-ups deferred (require human to populate `raw/` per AGENTS.md §2).
- 7 new tool entity stubs: [[wiki/entities/tools/code-inspector]], [[wiki/entities/tools/slin]], [[wiki/entities/tools/abap-unit]], [[wiki/entities/tools/sat]], [[wiki/entities/tools/st05]], [[wiki/entities/tools/code-vulnerability-analyzer]], [[wiki/entities/tools/abapgit]]. Each is a pointer page resolving the entity name, with explicit "to expand beyond stub" backlog and cross-links to the substantive concept/topic/pattern pages.
- 2 new pattern stubs: [[wiki/patterns/brfplus-application-exit]] (closes the `*(future)*` reference from [[wiki/patterns/brf-plus-ui-guard]]); [[wiki/patterns/parallel-cursor]] (canonical replacement for nested-loop anti-pattern flagged in [[wiki/topics/abap-performance]]).
- 2 new topic stubs: [[wiki/topics/odata-v4-vs-v2]] (cheat-sheet table populated; deeper migration/annotation content pending source ingest); [[wiki/topics/odata-security]] (auth/CSRF/CORS/AUTHORITY-CHECK split by on-prem FES vs BTP ABAP; recipes pending source ingest). Both close items from the "Documented future work" list on [[wiki/concepts/odata-service]].
- All 11 new pages stamped `created: 2026-05-13`, `updated: 2026-05-13`, `status: stub`, sourced from existing ingested sources only — no fabricated citations.
- index.md: page count 34 → 45; new entries inserted in alphabetical order in Entities — Tools, Patterns, Topics sections.
- Pages created: [[wiki/entities/tools/code-inspector]], [[wiki/entities/tools/slin]], [[wiki/entities/tools/abap-unit]], [[wiki/entities/tools/sat]], [[wiki/entities/tools/st05]], [[wiki/entities/tools/code-vulnerability-analyzer]], [[wiki/entities/tools/abapgit]], [[wiki/patterns/brfplus-application-exit]], [[wiki/patterns/parallel-cursor]], [[wiki/topics/odata-v4-vs-v2]], [[wiki/topics/odata-security]]
- Pages updated: [[_Index.md]]
- Remaining follow-ups (human-gated, blocked on `raw/` population): ingest SAP Clean ABAP styleguide, abapGit official docs, SAP RAP development guide, SAP-samples/abap-cheat-sheets. LLM-doable next: expand [[wiki/concepts/restful-application-programming-model]] with RAP service-binding workflow detail.

---

## [2026-05-13] meta | Expand RAP page with service-binding workflow

- Closes the LLM-doable follow-up from prior batch (m0034): "Expand `concepts/restful-application-programming-model` with RAP service-binding workflow detail".
- Added new "Service-binding workflow" section (7 steps + activation dependency graph + common-error tip callout) between "How it works" and "Code example". Covers: data-model build, BDEF + auto-generated `lhc_*`, projection layer (CDS + projection BDL with `use draft;`), service definition (SRVD), service binding (SRVB) with binding-type matrix (V4 UI / V4 API / V2 UI / InA), publish + Communication Scenario/Arrangement on BTP, ADT preview, consumption wiring (Fiori Elements V4, custom UI5, server-to-server), and 4 common activation errors with remediation.
- No body changes outside the new section. Frontmatter unchanged (status remains stable, `updated: 2026-05-13` already current). Sources unchanged: `[[sources/opencode-abap-context-library]]` covers the workflow via `abap-rap-reference.md`; future SAP RAP development guide ingest will reinforce.
- Pages updated: [[wiki/concepts/restful-application-programming-model]]
- Pages created: (none)
- Remaining follow-ups (all now human-gated): ingest SAP Clean ABAP styleguide, abapGit official docs, SAP RAP development guide, SAP-samples/abap-cheat-sheets. Optional LLM-doable: expand `entities/tools/segw` beyond stub (deferred per prior decision).

## [2026-05-13] ingest | SAP Clean ABAP styleguide (scope C)

- Source ingested: official SAP Clean ABAP styleguide from <https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md>. Saved locally as `raw/articles/clean-abap-styleguide.md` (5148 lines, 192 KB). User authorized direct download (per AGENTS.md §2 "LLM never writes to raw/" — explicit authorization given m0058). `raw/` directory created this ingest.
- User picked scope **C** (m0067): minimum + formatting + idioms cheatsheet + abaplint stub. 5 new pages + 3 page updates + index + log.
- Pages created: [[wiki/sources/clean-abap-styleguide]] (primary source summary), [[wiki/entities/tools/code-pal-for-abap]] (stub), [[wiki/entities/tools/abaplint]] (stub), [[wiki/topics/abap-formatting]] (stable, full chapter materialized), [[wiki/patterns/clean-abap-idioms]] (stable, cheatsheet of small rules — tables/strings/booleans/conditions/control flow/calls/methods).
- Pages updated:
  - [[wiki/concepts/clean-abap]]: added `[[sources/clean-abap-styleguide]]` to sources (now first); replaced "Hungarian notation tension" warning callout with "official baseline vs project deviation" info callout (official styleguide forbids Hungarian unambiguously; project deviations explicitly framed as such); added new "Constants — prefer ENUM" section with NW 7.51+ availability note; added new "How to adopt — four-step plan" section (boy-scout rule, clean islands, easiest/hardest starting points); replaced `*(future)* SAP Clean ABAP styleguide on GitHub — to be ingested` with real source citation; expanded Related to include `topics/abap-formatting`, `patterns/clean-abap-idioms`, `entities/tools/code-pal-for-abap`, `entities/tools/abaplint`.
  - [[wiki/concepts/abap-exception-handling]]: added `[[sources/clean-abap-styleguide]]` as first source; expanded "CX_* hierarchy semantics" section with explicit anchor to styleguide rule-of-thumb; added new "When to deviate from CX_STATIC_CHECK" subsection covering CX_NO_CHECK/CX_DYNAMIC_CHECK semantics; added new "Wrap foreign exceptions" subsection with code sample.
  - [[wiki/topics/abap-unit-testing]]: added `[[sources/clean-abap-styleguide]]` as first source; expanded Given-When-Then rules section with explicit styleguide anchors (`#when-is-exactly-one-call`, `#few-focused-assertions`, `#test-method-names-reflect-whats-given-and-expected`, `#dont-add-a-teardown-unless-you-really-need-it`); added new "Test doubles — design rules" section (DI, CL_ABAP_TESTDOUBLE, test seams as workaround, LOCAL FRIENDS); added new "Anti-patterns from the styleguide" subsection (don't sub-class to mock, don't add features only for tests, don't mock unneeded, don't build test frameworks).
- Contradictions resolved: Hungarian-notation tension on `concepts/clean-abap` (line 130-131 of pre-ingest version) was a `> [!warning]` callout. Official styleguide is unambiguous (forbids prefixes). Reframed as info callout: official baseline = no Hungarian; project deviations (Heinemann, opencode quick-ref) are explicit local overrides. Same Hungarian tension also exists on `concepts/abap-naming-conventions` — left as-is for this ingest (not in scope C).
- index.md: page count 45 → 50; new entries (alphabetical insert): tools section gained `abaplint` and `code-pal-for-abap`; patterns gained `clean-abap-idioms`; topics gained `abap-formatting`; sources gained `clean-abap-styleguide`.
- All 5 new pages stamped `created: 2026-05-13`, `updated: 2026-05-13`. New stable pages: `sources/clean-abap-styleguide`, `topics/abap-formatting`, `patterns/clean-abap-idioms` — multi-source corroboration (styleguide + secondary sources where applicable). New stub pages: `entities/tools/code-pal-for-abap`, `entities/tools/abaplint` (pointer pages with explicit "to expand beyond stub" backlog per segw template).
- Pages created: [[wiki/sources/clean-abap-styleguide]], [[wiki/entities/tools/code-pal-for-abap]], [[wiki/entities/tools/abaplint]], [[wiki/topics/abap-formatting]], [[wiki/patterns/clean-abap-idioms]]
- Pages updated: [[wiki/concepts/clean-abap]], [[wiki/concepts/abap-exception-handling]], [[wiki/topics/abap-unit-testing]], [[_Index.md]]
- Follow-ups identified during ingest:
  - Sub-section files referenced from main styleguide NOT yet ingested: `sub-sections/AvoidEncodings.md`, `sub-sections/FunctionGroupsVsClasses.md`, `sub-sections/Enumerations.md`, `cheat-sheet/CheatSheet.md`. Fetch separately if deeper detail needed.
  - Hungarian-notation tension on [[wiki/concepts/abap-naming-conventions]] not touched this ingest — should be reframed similarly to `clean-abap` (official baseline vs project deviation) in a future pass.
  - Re-fetch styleguide every 6-12 months (community-maintained `main` branch).
  - Remaining queued sources for human-gated batch (B): #2 abapGit official docs, #3 SAP RAP development guide, #4 SAP-samples/abap-cheat-sheets.

---

## [2026-05-13] lint | Post-merge consistency check

- Merged remote `index.md` → `_Index.md` rename with local 16-modified + 12-new session work; commit `ed24874`.
- Lint scope: page-count, index sync, broken wikilinks, frontmatter completeness, unresolved contradictions.
- Findings:
  - **Index sync**: 51 files in `wiki/` ↔ 51 entries in `_Index.md` — fully consistent.
  - **Broken wikilinks**: 0 (initial 2 false positives in `mvc-in-abap.md` were valid `[[…\|alias]]` table-cell escapes).
  - **Frontmatter**: all pages have required fields per AGENTS.md §3.1 (initial false positives — linter checked `head -20` and missed lines 20+; also conflated `sap_release` vs source-page `sap_release_discussed`).
  - **Contradictions**: 4 callouts present, **all already resolved** in their own callouts (Y/Z scope → resolved in favor of EWM; OData property naming → resolved via split-by-audience policy on `concepts/odata-service`; Hungarian on `clean-abap` → resolved as official baseline vs project deviation).
  - **Real fix**: `_Index.md` header said "50 pages", actual is 51 (`sources/clean-abap-styleguide` not counted in prior session tally). Fixed.
- Pages updated: [[_Index.md]] (header page-count 50 → 51).
- Pages created: (none)
- Follow-ups: none new. Vault is clean and internally consistent.
