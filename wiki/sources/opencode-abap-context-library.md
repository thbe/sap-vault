---
title: "opencode ABAP Context Library (personal LLM context)"
type: source
tags: [abap, rap, clean-abap, abapgit, testing, syntax-reference, llm-context]
source_kind: code
author: "Thorsten Bendler (personal curation; underlying material from SAP-samples/abap-cheat-sheets, SAP Clean ABAP styleguide, abapGit docs, internal YS00 standards)"
publication_date: 2026-05-11
url: "git@github.com:thbe/dotfiles — opencode/.config/opencode/context/languages/abap/"
raw_path: "private (gitignored — lives in dotfiles repo, not in raw/)"
sap_release_discussed: ["S/4HANA 2023+", "BTP ABAP 2402+", "NetWeaver 7.40+ (subset)"]
status: stable
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/restful-application-programming-model]]"
  - "[[concepts/abapgit-serialization]]"
  - "[[topics/abap-unit-testing]]"
  - "[[snippets/abap-modern-syntax-cheatsheet]]"
created: 2026-05-11
updated: 2026-05-11
---

# opencode ABAP Context Library (personal LLM context)

## TL;DR

A 12-file, ~80 KB curated reference library used as on-demand LLM context for ABAP development sessions in the [opencode](https://github.com/sst/opencode) agentic editor. Distilled from official SAP material (SAP-samples/abap-cheat-sheets, Clean ABAP styleguide, abapGit AFF docs) plus the author's internal YS00 standards. Each file declares its own token budget and is loaded by task type (development, code generation, RAP, testing, architecture, abapGit). The library complements the previously-ingested project standards by adding modern ABAP **syntax**, **RAP**, and **ABAP Unit** depth that the standards documents skip.

## Bibliographic info

- **Author**: Thorsten Bendler — personal curation.
- **Underlying sources**: SAP-samples/abap-cheat-sheets (Apache-2.0), SAP Clean ABAP styleguide (CC-BY-4.0), abapGit/AFF documentation (MIT), internal Heinemann YS00 standards.
- **Date snapshot**: 2026-05-11 (the version read for this ingest).
- **Format**: 12 Markdown files in a dotfiles repo, loaded by opencode based on a `quick-ref.md` context-loading table.
- **Raw path**: outside the vault — `dotfiles/opencode/.config/opencode/context/languages/abap/`. Not copied into `raw/` (would duplicate the dotfiles repo).

## The 12 files

| File                          | Bytes | Subject                                                                                       |
| ----------------------------- | ----- | --------------------------------------------------------------------------------------------- |
| `quick-ref.md`                | 3.4 K | Naming quick-check, DO/DON'T list, class template, context-loading table.                     |
| `abap-standards.md`           | 8.8 K | Mandatory project standards (ATC, MVC, OOP, BAPI > FM > BDC, performance rules).              |
| `abap-syntax-reference.md`    | 13.8K | Constructor expressions (VALUE/CORRESPONDING/NEW/CONV/COND/SWITCH/FILTER/FOR/REDUCE/LET), string templates, ABAP SQL. |
| `abap-rap-reference.md`       | 8.7 K | CDS view entities, BDL (managed/unmanaged, strict(2), draft, projection), EML.                |
| `abapgit-integration.md`      | 9.4 K | File naming, encoding (UTF-8+BOM/LF/indent), folder logic, deserialization phases, AFF.       |
| `abap-testing-patterns.md`    | 6.7 K | ABAP Unit, Given-When-Then, fixtures, DI, CL_ABAP_TESTDOUBLE, OSQL/CDS test envs, OO patterns. |
| `architecture-deep.md`        | 6.9 K | YS00 enhancement framework dispatcher, Form strategy hierarchy, BRF+ UI guard.                |
| `clean-abap-essentials.md`    | 2.8 K | Methods, classes, naming, CX_*_CHECK hierarchy, core principles.                              |
| `clean-abap-testing.md`       | 2.5 K | Test philosophy, G-W-T, dependency inversion, test doubles, anti-patterns.                    |
| `modern-abap-patterns.md`     | 2.3 K | "Prefer modern over legacy" table — inline declarations, FINAL, string templates, etc.        |
| `abap-formatting.md`          | 1.8 K | 120-char lines, 2-space indent, multi-line method call style, Pretty Printer settings.        |
| `abap-code-review-guide.md`   | 2.2 K | Review process (<400 lines ideal, <1 day turnaround), workflow with abapGit, feedback prefixes. |

## Key claims

### Modern ABAP language

- **Inline declarations + `FINAL(...)`** are the default — `DATA` only when reassignment is intended (`modern-abap-patterns.md`).
- **String templates `|text { var }|`** replace `CONCATENATE` (`abap-syntax-reference.md`).
- **Table expressions** `itab[ key = x ]` raise `CX_SY_ITAB_LINE_NOT_FOUND` — use `VALUE #( itab[...] DEFAULT ... )` to avoid TRY (`modern-abap-patterns.md`).
- **`REDUCE`, `FILTER`, `FOR ... IN itab`, `COND #()`, `SWITCH #()`** replace LOOP-with-accumulator and IF/CASE chains for assignment (`abap-syntax-reference.md`).
- **Predicative method calls**: `IF is_valid( )` is preferred over `IF is_valid( ) = abap_true` (`modern-abap-patterns.md`).
- **`lines( itab )` / `strlen( text )`** replace `DESCRIBE` (`modern-abap-patterns.md`).
- **ABAP SQL CTE** (`WITH +cte AS ( ... )`) and window functions (`OVER (PARTITION BY ...)`) supported on HANA (`abap-syntax-reference.md`).
- **MODIFY ENTITIES upsert**: `MODIFY` with `INSERT … OR UPDATE` semantics where available (`abap-syntax-reference.md`).

### RAP

- **CDS view entities (`DEFINE VIEW ENTITY`)** are the post-2020 replacement for `DEFINE VIEW`. Annotations drive UI metadata, semantics, and behavior projection (`abap-rap-reference.md`).
- **Behavior Definitions** declare `strict(2)` to enforce the latest validation rules; `managed` (RAP runtime handles persistence) vs `unmanaged` (caller handles) vs `additional save` (mixed).
- **Determinations** run after data change to derive values; **validations** run before save to reject invalid states; **side effects** annotate UI dependencies so Fiori Elements knows what to refetch.
- **Draft-enabled** business objects need `with draft;` plus draft table; allow save-as-draft flow in Fiori.
- **EML** is the ABAP API: `MODIFY ENTITY`, `READ ENTITY`, `EXECUTE ACTION`, `CREATE BY \_assoc`, `COMMIT ENTITIES`, `IN LOCAL MODE` for tests.
- **Projection BDEFs** (`projection;`) expose only a subset of base behavior to a specific service.

### Testing & DI

- **Given-When-Then** is the standard test structure; **one "When" per test**; descriptive method names (`test_rejects_negative_quantity`) over numbered tests (`clean-abap-testing.md`).
- **Test publics, not internals** — needing to test privates signals a design problem.
- **Three DI techniques**: constructor injection (preferred), back-door (production code never aware), setter (last resort).
- **Test seams** in production code via `"BEGIN OF TEST` macros — used for legacy code that can't be cleanly refactored.
- **CL_ABAP_TESTDOUBLE** for interface stubs; **CL_OSQL_TEST_ENVIRONMENT** for SQL-on-DB doubles; **CL_CDS_TEST_ENVIRONMENT** for CDS-view tests with provided test doubles.
- **Risk Level + Duration** attributes on test classes (`RISK LEVEL HARMLESS DURATION SHORT`) govern when ATC runs them.

### abapGit & AFF

- **File pair invariant**: `<name>.<type>.abap` + `<name>.<type>.xml` (or `.json` for AFF objects).
- **Encoding**: UTF-8 + BOM for XML, UTF-8 (no BOM) for `.abap`, LF line endings, final newline. ABAP indent 2 spaces, XML indent 1 space.
- **Folder logic**: `PREFIX` (recommended), `FULL`, or `MIXED`. PREFIX strips the root package prefix so paths are shorter.
- **Deserialization phases**: EARLY (packages) → DDIC (domains, data elements, tables) → ABAP (classes, function groups) → LATE (transports, ATC variants).
- **Namespace encoding**: `/IGZ/CL_FOO` becomes `#igz#cl_foo.clas.abap` on disk; appears as `(IGZ)CL_FOO` in some SAP UIs.
- **AFF (ABAP File Format)** is the SAP-standard successor for some object types — JSON-based, schema-versioned, language-agnostic.

### Standards & enhancement architecture

- **MVC mandatory** for new development.
- **Method preference order**: Method on class > Function Module > BAPI > BDC.
- **Enhancement preference order**: User Exit (legacy) → BAdI → Enhancement Spot → Implicit/Explicit enhancement.
- **YS00 enhancement framework** (`YS00_DB_ENH_C` control + `YS00_DB_ENH_V` value + `YS00_CL_COMMON_ENHC_FRWRK` dispatcher with `GET_FLEX_FIELD_NAME` / `GET_EXITS`) is the project standard for multi-market BAdI dispatch — same framework documented from a different angle than [[sources/sap-development-standard-approach-abap-fiori-v1]].
- **Form strategy hierarchy**: BRF+ Adobe → XML → Custom Adobe → Smart Forms → SAPScript (banned for new dev).
- **BRF+ UI guard via `CL_FDT_WD_UI_SIMPLE_MODE` subclass** — same pattern as [[patterns/brf-plus-ui-guard]].

### Code review

- **Transport-sized PRs** — one logical change per review; <400 lines ideal, <1000 max.
- **<1 business day turnaround**; reviews lose context when stale.
- **Comment prefixes**: `[blocking]`, `[suggestion]`, `[question]`.
- **abapGit→GitHub workflows**: one-way (review-only) or two-way (full Git flow).

## Code samples worth keeping

Extracted as dedicated snippet pages:

- [[snippets/abap-modern-syntax-cheatsheet]] — the prefer/over table.
- [[snippets/abap-doc-comments]] *(pre-existing — same `"!` syntax referenced here)*.
- [[snippets/flower-box-header]] *(pre-existing)*.

Sub-page snippets embedded in concept/pattern pages:

- ABAP Unit class template → [[topics/abap-unit-testing]].
- CL_ABAP_TESTDOUBLE example → [[patterns/test-doubles-and-dependency-injection]].
- CDS view entity with annotations → [[concepts/cds-view-entities]].
- Behavior definition with determinations/validations → [[concepts/behavior-definition]].
- EML modify/read/action calls → [[concepts/eml]].

## Pages this source touches

**New pages created from this source**:

- [[snippets/abap-modern-syntax-cheatsheet]]
- [[concepts/cds-view-entities]]
- [[concepts/behavior-definition]]
- [[concepts/eml]]
- [[topics/abap-unit-testing]]
- [[patterns/test-doubles-and-dependency-injection]]
- [[patterns/oo-design-patterns-in-abap]]

**Existing pages expanded**:

- [[concepts/restful-application-programming-model]] — promoted from stub.
- [[concepts/abapgit-serialization]] — encoding rules, deserialization phases, namespace encoding, AFF.
- [[concepts/clean-abap]] — Hungarian-notation tension, CX_*_CHECK hierarchy, FINAL by default.
- [[concepts/abap-exception-handling]] — CX_STATIC_CHECK vs DYNAMIC vs NO_CHECK semantics, `RAISE EXCEPTION NEW`.
- [[concepts/abap-naming-conventions]] — object format strings, OData entity examples.
- [[topics/abap-performance]] — `lines( itab )` over `DESCRIBE`, table-expression `DEFAULT`.
- [[topics/abap-quality-gates]] — review workflow specifics (transport-sized PRs, abapGit→GitHub).

**Existing pages confirmed (second source backref added)**:

- [[patterns/table-driven-enhancement-framework]] — YS00 framework re-confirmed.
- [[patterns/brf-plus-ui-guard]] — `CL_FDT_WD_UI_SIMPLE_MODE` pattern re-confirmed.
- [[topics/sap-forms-strategy]] — Forms hierarchy re-confirmed.

## Open questions / to verify

- **AFF current state**: which object types are AFF-default vs still XML-default in S/4HANA 2023+? `abapgit-integration.md` mentions AFF JSON but doesn't enumerate covered types.
- **`strict(2)`** in BDL — confirm that's the current strictest level on S/4 2023 (could be `strict(3)` by now).
- **CDS view entity vs DDL `@AbapCatalog.viewEnhancementCategory`** — interaction with extensibility not covered.
- **Singleton/Factory snippets** — the testing-patterns file lists OO patterns by name; concrete ABAP idiomatic implementations should be cross-checked against SAP-samples once those are ingested directly.
- **Test seam macro syntax** — `clean-abap-testing.md` references it abstractly; need an authoritative example.

## Contradictions with other sources

### Internal contradiction (within this source)

- **Hungarian notation**: `clean-abap-essentials.md` says **no `lv_/lt_/lo_/ls_` prefixes**; `abap-standards.md` and `quick-ref.md` from the same library prescribe exactly those prefixes for internal variables. Resolution: this mirrors a pragmatic project-vs-purist tension also present in [[sources/sap-development-standard-approach-abap-fiori-v1]] and [[sources/heinemann-ewm-coding-standards]] — the project standards win in practice. Documented on [[concepts/clean-abap]] and [[concepts/abap-naming-conventions]] as an explicit note rather than a `> [!warning] Contradiction` callout.

### Cross-source

- **None new.** This source corroborates the Y/Z scoping, MVC, OOP > FM, FAE empty-check, no SELECT-in-loop, no header lines, ATC enforcement claims from the previous two sources. The Y/Z scope contradiction already resolved in favor of [[sources/heinemann-ewm-coding-standards]] stands.
