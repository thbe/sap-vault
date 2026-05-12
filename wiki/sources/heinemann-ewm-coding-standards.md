---
title: "Heinemann EWM Coding Standards (Migration Docs)"
type: source
tags: [ewm, abap, abapgit, migration, standards, heinemann]
sap_release: ["S/4HANA 2023 FPS04"]
status: stable
sources: []
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/clean-abap]]"
  - "[[concepts/abapgit-serialization]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-quality-gates]]"
  - "[[topics/abap-performance]]"
  - "[[topics/abap-security-checklist]]"
  - "[[topics/sap-forms-strategy]]"
  - "[[patterns/multi-site-repo-architecture]]"
  - "[[patterns/dynamic-code-whitelist]]"
  - "[[patterns/form-to-method-refactor]]"
created: 2026-05-11
updated: 2026-05-11
---

# Heinemann EWM Coding Standards (Migration Docs)

> Applied coding principles from the Heinemann EWM (Extended Warehouse Management) repository consolidation project. Concrete, in-flight standards governing the unification of two warehouse-site codebases (Erlensee + Allermöhe) onto a three-tier ABAP architecture.

## Bibliographic info

- **Repository**: `Heinemann-Group/EWM` (private monorepo, 58 abapGit submodules)
- **Local path**: `/home/bendlert/Code/git/thbe-gh/EWM`
- **Author**: Heinemann Group SAP team (T. Bendler et al.)
- **Format**: Markdown documentation in a Git monorepo
- **Date reviewed**: 2026-05-11
- **Target SAP system**: S/4HANA 2023 FPS04
- **Key files ingested**:
  - `AGENTS.md` (143 lines) — repo architecture & namespace policy
  - `README.md` (299 lines) — project structure, /IGZ/ package matrix
  - `docs/migration/standards/01-naming-conventions.md` (577 lines)
  - `docs/migration/standards/02-coding-guidelines.md` (277 lines)
  - `docs/migration/standards/03-code-quality-performance.md` (597 lines)

## TL;DR

A real-world, in-flight migration of ~21,000 ABAP objects across two warehouse sites onto a three-tier repository architecture (`YEWM` common + `ZALM_EWM`/`ZERL_EWM` site-specific + `YBC`/`YCA` cross-system). The standards are operationally-tested: per-phase ATC + ABAP Unit + security gates, hard pass criteria, concrete remediation patterns, and explicit anti-patterns from real findings (17 documented ERL security issues). Sharpens — and in places contradicts — the [[sources/sap-development-standard-approach-abap-fiori-v1|v1 Heinemann standards PDF]].

## Key claims (by document)

### AGENTS.md / README.md — Architecture

- Three-tier monorepo: `YEWM` (common EWM) ← `ZALM_EWM` / `ZERL_EWM` (site-specific). `YBC` (admin tools) + `YCA` (user tools/utilities) are independent cross-system siblings.
- Site repos **never depend on each other**. Import order: YBC + YCA → YEWM → site repos in parallel.
- `/IGZ/` (`IGZ_*` prefix) = third-party Best Practice framework — not modified, fit-gap analysed only.
- Same logical /IGZ/ module can exist under both ALM/ and ERL/ as **separate repos with separate codebases** (not forks).
- 11 /IGZ/ packages overlap, 24 ALM-only, 1 ERL-only — full matrix in README.

### 01-naming-conventions.md — Y/Z scoping

- **Y-prefix scope is narrow**: applies only to common/shared code in `YEWM`, `YBC`, `YCA`. Site-specific code in `ZALM_EWM` and `ZERL_EWM` **retains the Z-prefix** (no rename churn).
- Sub-package pattern: `<ROOT>_<SUFFIX>` (e.g., `YEWM_CO`, `ZALM_GI`, `ZERL_ACC`). Suffix re-used directly from source `ZEWM<SUFFIX>` packages.
- Folder-logic conversion: source `FULL` → target `PREFIX`. Strips root-package prefix from each folder name. Concrete formula and per-repo file-tree examples given.
- New transactions: `Y_EWM_*` (common), `Y_ALM_*` (Allermöhe), `Y_ERL_*` (Erlensee).
- Object naming for new common objects: `YCL_EWM_<area>_*`, `YIF_EWM_<area>_*`, `YEWM_T_*`, `YEWM_DE_*`, `YEWM_DO_*`. Site repos use `ZCL_ALM_*` / `ZCL_ERL_*`.
- `.abapgit.xml` template: `MASTER_LANGUAGE=E`, `STARTING_FOLDER=/src/`, `FOLDER_LOGIC=PREFIX`.
- Decision tree at §9: identical in both → YEWM common; ALM-only → ZALM_EWM; ERL-only → ZERL_EWM; diverged → extract base to YEWM + site extensions or fork.

### 02-coding-guidelines.md — Gap analysis vs. organization standards

Nine identified gaps (G1–G9), each with severity, current state, and resolution:

- **G1 — Y/Z naming (CRITICAL/HIGH)**: Y-prefix renaming applies to YEWM only; Phase 6 mandatory rename for YEWM objects, site repos only need cross-reference updates.
- **G2 — Enhancement dispatcher (CRITICAL/MEDIUM)**: Audit existing enhancement implementations for `YS00_CL_COMMON_ENHC_FRWRK` dispatcher compliance. Existing `ZCL_IM` delegation pattern accepted; mandate dispatcher for new enhancements.
- **G3 — Quality gates per phase (CRITICAL/HIGH)**: Per-phase ATC + Extended Program Check + authority-check validation. +3–4 weeks effort.
- **G4 — Forms strategy (IMPORTANT/LOW)**: BRF+ Adobe → XML Forms → Custom Adobe → Smart Forms (acceptable, no new) → SAPScript (avoid). 23 Smart Forms migrated as-is.
- **G5 — Constants pattern (IMPORTANT/LOW)**: `YS00_CL_CONSTANTS` or TVARVC preferred; existing `ZIF_EWM_*_CONST` interfaces accepted as functionally equivalent.
- **G6 — Header template (IMPORTANT/LOW)**: Flower-box header with `Market` field. Phase 6 batch update; mandatory for new objects.
- **G7 — Fiori/UI5 standards (IMPORTANT/MEDIUM)**: XML Views mandatory, PascalCase, SEGW for OData, no DOM manipulation, `===`, `sap.ui.define`, BEM CSS, ESLint scan in Phase 5.
- **G8 — abapGit serialization (MODERATE/LOW)**: UTF-8 + LF, no system-specific data in XML, lowercase filenames, deserialization order awareness.
- **G9 — ABAP Unit Tests (CRITICAL/HIGH/NEW)**: Test classes migrate with subjects. Zero test failures is hard gate. Local test includes (`ltcl_*`) verified in serialized XML. New classes require at minimum one test method. +4–5 weeks effort.

Revised migration timeline: **20–28 weeks** (was 18–26).

### 03-code-quality-performance.md — Applied standards

- **Security findings (ERL, 17 documented)**: 6 HIGH (username gates, breakpoints in BAdI implementations, disabled `AUTHORITY-CHECK`), 6 MEDIUM (dynamic SUBMIT/CALL TRANSACTION without whitelist, hardcoded user whitelists, `ASSERT 1 = 2` patterns), 5 LOW/INFO. ALM not yet audited — required in Phase 1.
- **Acceptance criterion**: Zero HIGH/MEDIUM security findings post-Phase 4.
- **Dynamic code execution policy**: `SUBMIT <var>` and `CALL TRANSACTION <var>` prohibited unless whitelist-validated. `GENERATE SUBROUTINE POOL` prohibited entirely. Concrete remediation pattern given.
- **FORM/PERFORM debt**: 8,229 FORM definitions across both repos (5,370 ERL + 2,859 ALM, ~12% of source files). ATC error in ABAP Cloud, warning in Standard ABAP. Phased refactor — HIGH-priority packages (`zewmrt`, `zewmmf`, `zewmgi`, RF dialogs) in Phase 6. Concrete BEFORE/AFTER refactor pattern.
- **Dead code policy**: ~2,500 files to delete or archive in Phase 0 — test/debug objects (e.g., `zewm_del_loc_bp`, `zewm_test`), migration tooling (`ZEWMMIG`, `ZEWMMI`), 53 BSP apps (all have UI5 replacements), `_OLD` superseded classes.
- **Code duplication**: Category A (identical, ~1,500–2,500 objects → YEWM single copy), B (>90% similar, ~500–800 → YEWM merged with deltas applied), C (same name, divergent logic, ~200–400 → fork or refactor with YEWM base — highest maintenance risk), D (unique → respective site repo).
- **Performance — RF dialog SLA**: < 2 seconds response time for all RF transactions. Hard requirement. `ZCL_EWM_PERF_MEASURE` infrastructure class in ERL → migrate to YEWM as `YCL_EWM_PERF_MEASURE`. Baseline before/after each phase; <5% regression tolerance.
- **DB anti-pattern table**: `SELECT *` in loops, `SELECT` inside `LOOP AT ITAB` (N+1), missing index usage, sync RFC in dialog, large ITAB without `HASHED`, `FORM` as event handler, unnecessary `COMMIT WORK` in loops, `MESSAGE ... RAISE` in class.
- **Exception handling policy**: `YCX_EWM_GENERAL_EXCEPTION` base + `YCX_EWM_AUTHORITY_ERROR`, `YCX_EWM_DB_ERROR`, `YCX_EWM_RF_DIALOG_ERROR` (resumable). Ban on `EXCEPTIONS OTHERS = 0` for critical FMs. Concrete TRY/CATCH templates.
- **Documentation standard**: ABAP Doc `"!` syntax with `<p class="shorttext">`, `@parameter`, `@raising` tags. ATC `SLIN_DESC` enforcement on public methods.
- **Per-phase quality gate checklist** (§9.3): per-phase boxes for delete-list, ATC, ABAP Unit, Extended Program Check, security findings, FORM refactoring, Y-prefix rename, exception handling, RF perf baseline, documentation.

## Code samples worth keeping

Extracted into dedicated snippets/patterns:

- Dynamic code whitelist pattern → [[patterns/dynamic-code-whitelist]]
- FORM → method refactor template → [[patterns/form-to-method-refactor]]
- Exception handling TRY/CATCH templates → [[concepts/abap-exception-handling]]
- ABAP Doc `"!` documentation block → [[snippets/abap-doc-comments]]
- Heinemann flower-box header (Market variant) → updated [[snippets/flower-box-header]]
- `.abapgit.xml` PREFIX template → [[concepts/abapgit-serialization]]

## Open questions / things to verify

1. **Status of `YS00_CL_COMMON_ENHC_FRWRK` in EWM context** — the v1 standards PDF mandates this dispatcher pattern; EWM accepts the simpler `ZCL_IM` delegation pattern as "compliant in spirit". When does the strict dispatcher actually apply?
2. **Y/Z scope contradiction with v1 PDF** — v1 said `Y*` = global / common dev, `Z*` = local market. EWM scopes Y much more narrowly to "shared/cross-site". Which applies for non-EWM new dev (e.g., FI/CO custom code)?
3. **`YCX_EWM_*` exception class hierarchy** — proposed in EWM standards but actual implementation status (in `YEWM` repo) not verified. Are these classes already created or planned?
4. **`ZCL_EWM_PERF_MEASURE` API** — referenced as the perf-baseline tool. Method signatures and usage pattern not yet documented in the wiki.
5. **ABAP Cloud (Steampunk) compatibility** — the standards target on-premise S/4HANA 2023 FPS04. How much of the FORM-debt and dynamic-code policy maps directly to BTP ABAP cloud?
6. **Phase 0 status** — which delete-list items have actually been removed from production at time of ingest?

## Ingest notes

- Source is **applied/operational** (real findings, real remediation plan). Wiki pages stay `status: draft` until verified against the actual `YEWM` repo contents.
- Customer-internal repository — kept private; only the wiki abstractions are published.
- Where this source contradicts [[sources/sap-development-standard-approach-abap-fiori-v1]], EWM is treated as authoritative (more recent, more concrete). Contradictions flagged inline on touched pages.
