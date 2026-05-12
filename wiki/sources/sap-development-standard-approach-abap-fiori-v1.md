---
title: "SAP Development Standards & Approach — ABAP and Fiori v1"
type: source
tags: [naming-conventions, performance, quality, fiori, ui5, gateway, brfplus, enhancement-framework, clean-abap, internal-doc]
source_kind: pdf
author: "Thomas Bendler"
publication_date: 2023-03-31
url: ""
raw_path: "raw/pdfs/SAP - Development Standard Approach for ABAP and Fiori v1.pdf"
sap_release_discussed: ["S/4HANA (project: Gebr. Heinemann S/4 Upgrade)", "ECC (legacy reference)"]
status: stable
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/clean-abap]]"
  - "[[concepts/mvc-in-abap]]"
  - "[[concepts/sap-fiori-overview]]"
  - "[[concepts/odata-service]]"
  - "[[topics/abap-performance]]"
  - "[[topics/abap-quality-gates]]"
  - "[[patterns/table-driven-enhancement-framework]]"
  - "[[patterns/brf-plus-ui-guard]]"
  - "[[snippets/flower-box-header]]"
  - "[[entities/tools/atc]]"
  - "[[entities/tools/segw]]"
created: 2026-05-11
updated: 2026-05-11
---

# SAP Development Standards & Approach — ABAP and Fiori v1

## TL;DR

Internal development-standards document for the **Gebr. Heinemann S/4HANA upgrade project**, authored by Thomas Bendler (v0.3, 31.03.2023). Defines opinionated naming conventions, ABAP quality and performance rules, Fiori/UI5 development guidelines, and two custom architectural frameworks (table-driven enhancement framework and BRF+ UI guard). The doc straddles modern S/4 and legacy ECC tech: solid Clean-ABAP-leaning principles sit alongside SEGW/Gateway, Web IDE, and classical Hungarian-notation UI5. Treat as a **transitional standard**, not a pure modern RAP playbook.

## Bibliographic info

- **Author**: Thomas Bendler
- **Date**: Initial 13.01.2023, v0.3 31.03.2023
- **Project**: Gebr. Heinemann — S/4 Upgrade Project
- **Format**: PDF, 86 pages
- **Raw file**: `raw/pdfs/SAP - Development Standard Approach for ABAP and Fiori v1.pdf`
- **Extracted text**: `/tmp/sap-std.txt` (3705 lines, parsed via liteparse)

## Key claims

### Quality & design (Ch 2)
- Mandatory ATC checks and SAP Code Inspector before any transport (§2.1.5–2.1.6).
- Custom transactions and table maintenance must follow naming + authorization rules (§2.1.3–2.1.4).
- MVC separation is required for new development; OO programming preferred over procedural (§2.3–2.4).

### Performance (Ch 3)
- DB access: no `SELECT *`, limit traffic/range/result set, use indexes, archive aged data (§3.3).
- For-all-entries: **always check driver table is non-empty first** (otherwise full scan).
- Internal tables: use SORTED/HASHED with appropriate keys; `READ … BINARY SEARCH` on STANDARD; avoid nested loops without keyed reads (§3.4.2).
- Class-based exceptions for error handling; modularize with FORMs/methods (§3.5).
- Prefer BAPIs over BDC; document calls, especially asynchronous interfaces (§3.7.8).
- **Flower-box header comment block mandatory** at top of every program/method/enhancement spot (§3.7.9). Modification log added below header on changes; "Start of change/End of change" markers around modifications.

### Naming conventions (Ch 4 + Appendix 6.1, 6.2)
- `Y*` namespace = global/common; `Z*` namespace = local market.
- Package taxonomy: `YS00` (common), `YBPM`, `YOTC`, `YPTP`, `YRTR` (per business stream).
- Pattern: `[PACKAGE]_[TYPE]_[Description]` — e.g. `YS00_SS_PRINTDELIVERY` (common smartform), `ZNL_SS_…` (local NL).
- Repository-object table (Appendix 6.1) defines prefixes per object type: `CL_` classes, `IF_` interfaces, `DB_` tables, `V_` views, `ST_` structures, `TT_` table types, `DT_` data elements, `DO_` domains, `SH_` search helps, `ES_/EI_/EC_/BD_` enhancements, `IDOC_/IEXT_` IDocs, `SS_` smartforms, `PDF_/PDFIF_` interactive forms, `WD_/WAP_/BSP_/WS_` web objects, etc.
- Internal-element prefixes (Appendix 6.2): `cl_`/`lcl_`, `it_`/`lit_`, `wa_`/`lwa_`, `o_`/`lo_`, `tp_`/`ltp_`, `co_`/`lco_`, `ra_`/`lra_`, parameter prefixes `ixx_`/`exx_`/`cxx_`/`rxx_` where xx encodes the data-type prefix. SCI variant `Y_NAMING_CONVENTIONS` enforces these.
- Transport descriptions follow project conventions (§4.6.3); transport check enforced (§4.6.4).

### Fiori / UI5 (Ch 5)
- Embedded FES on S/4 is the target landscape; SAP Web Dispatcher in front; ABAP back-end server hosts business logic (§5.3).
- OData service naming standards defined (§5.4) — non-SAP English field names (e.g. `FirstName` not `VNAME`).
- UI5 project structure, JS guidelines, code formatting (§5.5–5.8).
- **Naming conventions use classical Hungarian notation** (`sId`, `oRef`, `aItems`, `bFlag`, `iCount`) — pre-modern UI5 style (§5.9).
- CSS guidance leans BEM-ish; uses standard SAP CSS classes where possible (§5.12).
- UI5 security: XSS prevention via output encoding (§5.13).
- **Gateway development uses SEGW** (legacy Gateway Service Builder), DPC_EXT class for method implementation (§5.14).

### Strategies (Appendix 6.3, 6.4)
- **Enhancement framework** (§6.3.1): control table `YS00_DB_ENH_C` + value table `YS00_DB_ENH_V` + dispatcher class `YS00_CL_COMMON_ENHC_FRWRK` (methods `GET_FLEX_FIELD_NAME`, `GET_EXITS`). Maintained via view cluster + TCODE `YS00_ENH_CNTRL`. Enhancements toggleable via active flag, sequenced via `SEQUENCE_NO`, filtered by 5 flex fields with specific/generic search.
- **Form strategy** (§6.3.2): Adobe Forms preferred, BRF+-driven output management (Note 2248229). Common global class `YS00_CL_CUSTOMER_FORM_OBJECT`. So10 text constants in tables `YS00_DB_BC_01/02`. Form fonts: Lucida Sans Unicode 9pt.
- **BRF+ strategy** (§6.3.3): decision tables maintained Specific→Generic; Application Exit class implementing `IF_FDT_APPLICATION_SETTINGS` to enable editing in TST/PRD by overriding `GV_AUTHORITY_CHECK` and `GV_GET_CHANGEABILITY`.
- **BRF+ UI guard** (§6.4): custom class `YOTC_CL_BRF_UI_CONTROL` inheriting from `CL_FDT_WD_UI_SIMPLE_MODE`, redefining `IF_FDT_WD_UI_MODE~GET_CONFIGURATION` to strip risky toolbar buttons (Delete, Mark Obsolete, Search My Object) — wrapped by custom report and TCODE. Software component: `S4CORE 104`.

## Code samples worth keeping

- [[snippets/flower-box-header]] — extracted from §3.7.9.
- *(future)* enhancement-framework dispatcher signature — see [[patterns/table-driven-enhancement-framework]].
- *(future)* BRF+ application-exit class skeleton.

## Pages this source touches

- [[concepts/abap-naming-conventions]]
- [[concepts/clean-abap]]
- [[concepts/mvc-in-abap]]
- [[concepts/sap-fiori-overview]]
- [[concepts/odata-service]]
- [[topics/abap-performance]]
- [[topics/abap-quality-gates]]
- [[patterns/table-driven-enhancement-framework]]
- [[patterns/brf-plus-ui-guard]]
- [[snippets/flower-box-header]]
- [[entities/tools/atc]]
- [[entities/tools/segw]]

## Open questions / to verify

- Source uses SEGW (legacy) for OData — for new RAP-based S/4 development, projection views + service bindings replace this. Confirm which path is in scope per project.
- UI5 Hungarian notation (`sId`, `oRef`) is pre-TypeScript convention; modern UI5 + TS practice differs. Flag where applicable.
- `MV45AFZZ` user-exit reference (§3.7.5) is ECC-era; on S/4 prefer BAdIs / enhancement spots.
- Web IDE is deprecated → SAP Business Application Studio (BAS) is the current IDE; pages should note this.
- Confirm SAP-release applicability of the SCI variant `Y_NAMING_CONVENTIONS` — is it shipped or project-specific?
- Form font/style choices (Lucida Sans Unicode 9pt) are project-specific brand choices, not universal — note as such.

## Contradictions with other sources

- _(none yet — first source ingested)_
