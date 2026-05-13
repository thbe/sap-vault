---
title: "ABAP Naming Conventions"
type: concept
tags: [naming-conventions, governance, clean-abap, abap]
sap_release: ["S/4HANA 2023 FPS04", "S/4HANA (project standard)", "ECC (legacy reference)"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[topics/abap-quality-gates]]"
  - "[[patterns/multi-site-repo-architecture]]"
  - "[[entities/tools/atc]]"
created: 2026-05-11
updated: 2026-05-13
---

# ABAP Naming Conventions

> Project-wide rules for naming repository objects and internal elements so the codebase stays readable, searchable, and navigable.

## Overview

SAP reserves the `Y*` and `Z*` namespaces for the customer. The Gebr. Heinemann S/4 standard splits them by scope:

- **`Y*` = global / common** development (used across markets).
- **`Z*` = local market** development (single-country / single-org-unit).

All names are English. Naming is enforced via SCI variant `Y_NAMING_CONVENTIONS` and ATC checks before transport.

> [!warning] Contradiction — Y/Z scope (resolved in favor of EWM)
> [[sources/sap-development-standard-approach-abap-fiori-v1|v1 PDF]] defines `Y* = global / common, Z* = local / single-market`.
> [[sources/heinemann-ewm-coding-standards|EWM project (2025)]] scopes `Y*` much more narrowly: only **shared, cross-site** objects (package `YEWM`, common framework `YBC`/`YCA`, exception classes `YCX_*`). Site-specific code keeps `Z*` (e.g. `ZALM` for Alicante, `ZERL` for Erlenbach) — no rename churn during migration.
> **Active project standard for new EWM work**: follow the EWM scoping. Only promote a `Z*` object to `Y*` when ≥2 sites need it. See [[patterns/multi-site-repo-architecture]].

> [!info] Scope
> This page documents the Heinemann S/4-upgrade convention from [[sources/sap-development-standard-approach-abap-fiori-v1]]. Other SAP customers use different conventions; SAP itself only mandates the `Y/Z` prefix.

## Key ideas

- **Pattern**: `[PACKAGE]_[TYPE]_[Description]` for most repository objects.
- **Package prefix encodes business stream**: `YS00` (common), `YBPM`, `YOTC` (Order-to-Cash), `YPTP` (Procure-to-Pay), `YRTR`, etc.
- **Type prefix encodes object kind**: `CL_` class, `IF_` interface, `DB_` table, `V_` view, `ST_` structure, `TT_` table type, `DT_` data element, `DO_` domain, `SH_` search help, `ES_/EI_/EC_/BD_` enhancement spots/implementations/composite/BAdI, `SS_` smartform, `PDF_` interactive form, `WD_` Web Dynpro component, `WS_` web service.
- **Internal-element prefixes** distinguish scope and kind: `cl_`/`lcl_` (global/local class), `it_`/`lit_` (internal table), `wa_`/`lwa_` (work area), `o_`/`lo_` (object reference), `tp_`/`ltp_` (typed variable / "type prefix"), `co_`/`lco_` (constant), `ra_`/`lra_` (range table).
- **Parameter prefixes** combine direction + type: `i`/`e`/`c`/`r` for importing/exporting/changing/returning, `xx` placeholder filled with the type prefix → e.g. `iit_materials` (importing internal table), `ewa_matmaster` (exporting work area), `rtp_found` (returning typed parameter).
- **No naming for prohibited objects**: Logical Databases, SAP Queries, Report Writer, CATT Procedures, Match Codes — these are not allowed.

## How it works

### Repository objects (selected examples)

| Object             | Type | Format                          | Example                       |
| ------------------ | ---- | ------------------------------- | ----------------------------- |
| Common dev class   | DEVC | `Y[APPL]00`                     | `YS00`                        |
| Local dev class    | DEVC | `Z[MARKET]`                     | `ZNLD`                        |
| Class              | CLAS | `[DEV]_CL_[DESCR]`              | `YOTC_CL_ORDER_FACTORY`       |
| Interface          | INTF | `[DEV]_IF_[DESCR]`              | `YOTC_IF_ORDER_HANDLER`       |
| Exception class    | CLAS | `YCX_[DEV]_[DESCR]`             | `YCX_OTC_ORDER_INVALID`       |
| Executable report  | PROG | `[DEV]_R[APPL]_[DESCR]`         | `YOTC_RV_ORDER_REPRINT`       |
| Function group     | FUGR | `[DEV]_[DESCR]`                 | `YOTC_ORDER_UTILS`            |
| Function module    | FUNC | `Y_[DEV]_FM_[VERB]_[DESCR]`     | `Y_OTC_FM_CREATE_ORDER`       |
| Enhancement spot   | ENHS | `[DEV]_ES_[DESCR]`              | `YOTC_ES_ORDER_SAVE`          |
| BAdI implementation| SXCI | `[DEV]_BD_[DESCR]`              | `YOTC_BD_ORDER_PRICING`       |
| Database table     | TABL | `[DEV]_DB_[DESCR]`              | `YS00_DB_ENH_C`               |
| Database view      | VIEW | `[DEV]_V_[DESCR]`               | `YS00_V_ENH_CFG`              |
| Structure          | TABL | `[DEV]_ST_[DESCR]`              | `YOTC_ST_ORDER_HEADER`        |
| Table type         | TTYP | `[DEV]_TT_[DESCR]`              | `YOTC_TT_ORDER_ITEMS`         |
| Data element       | DTEL | `[DEV]_DT_[DESCR]`              | `YS00_DT_EXTID`               |
| Data domain        | DOMA | `[DEV]_DO_[DESCR]`              | `YS00_DO_ACTFLG`              |
| Lock object        | ENQU | `E[DEV]_[DESCR]`                | `EYOTC_ORDER`                 |
| Smartform          | SSFO | `[DEV]_SS_[DESCR]`              | `YS00_SS_PRINTDELIVERY`       |
| Interactive form   | SFPF | `[DEV]_PDF_[DESCR]`             | `YS00_PDF_INVOICE`            |

Full table: see [[sources/sap-development-standard-approach-abap-fiori-v1]] §6.1.

### Internal elements (selected)

| Element           | Global  | Local    | Example                                  |
| ----------------- | ------- | -------- | ---------------------------------------- |
| Class             | `cl_`   | `lcl_`   | `cl_material`                            |
| Interface         | `if_`   | `lif_`   | `if_create_material`                     |
| Type              | `ty_`   | `lty_`   | `ty_material`                            |
| Table type        | `ty_t_` | `lty_t_` | `ty_t_materials`                         |
| Internal table    | `it_`   | `lit_`   | `it_materials`                           |
| Work area         | `wa_`   | `lwa_`   | `wa_material`                            |
| Object reference  | `o_`    | `lo_`    | `o_alv` TYPE REF TO `cl_salv_table`      |
| Variable (typed)  | `tp_`   | `ltp_`   | `tp_quantity TYPE menge`                 |
| Constant          | `co_`   | `lco_`   | `co_standard_order TYPE auart VALUE 'TA'`|
| Range             | `ra_`   | `lra_`   | `ra_matnr TYPE RANGE OF matnr`           |
| Importing param   | `ixx_`  | n/a      | `iit_matnr` / `iwa_mara`                 |
| Exporting param   | `exx_`  | n/a      | `ewa_matmaster`                          |
| Changing param    | `cxx_`  | n/a      | `ctp_quantity TYPE menge`                |
| Returning param   | `rxx_`  | n/a      | `rtp_found TYPE abap_bool`               |

Full table: see [[sources/sap-development-standard-approach-abap-fiori-v1]] §6.2.

## Code example

```abap
"! YOTC_CL_ORDER_FACTORY — common (Y) class in OTC package
CLASS yotc_cl_order_factory DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS create_order
      IMPORTING iwa_header   TYPE yotc_st_order_header
                iit_items    TYPE yotc_tt_order_items
      EXPORTING ewa_result   TYPE yotc_st_order_result
      RETURNING VALUE(rtp_success) TYPE abap_bool
      RAISING   ycx_otc_order_invalid.

    CONSTANTS co_default_order_type TYPE auart VALUE 'TA'.
ENDCLASS.
```

## Release & version notes

> [!info] Applicability
> Applies to the Gebr. Heinemann S/4 upgrade project. SAP itself only mandates the `Y/Z` prefix; everything else is a project convention.

> [!warning] Legacy reference
> Module pools (`SAPM…`), subroutine pools (`SAPF…`), Web Dynpro (`WD_`/`WAP_`), and BSP (`BSP_`) entries in the convention are largely legacy on S/4. Prefer Fiori/UI5 + RAP for new UI work.

## Related

- [[concepts/clean-abap]]
- [[concepts/abap-exception-handling]] — `YCX_*` exception class hierarchy.
- [[patterns/multi-site-repo-architecture]] — how `YEWM` / `ZALM` / `ZERL` / `YBC` / `YCA` packages relate.
- [[topics/abap-quality-gates]]
- [[patterns/table-driven-enhancement-framework]] — uses these conventions throughout.
- [[entities/tools/atc]] — enforcement.

## EWM project specifics

The Heinemann EWM migration ([[sources/heinemann-ewm-coding-standards]]) adds:

- **Site-package prefixes**: `YEWM*` (shared EWM), `ZALM*` (Alicante), `ZERL*` (Erlenbach), `YBC*` / `YCA*` (cross-application framework).
- **Transaction codes**: `Y*` for shared (e.g. `YEWM_*`), `Z*` for site-specific (e.g. `ZALM_*`). Avoid renaming existing live transaction codes — operator muscle memory matters.
- **Exception classes**: always `YCX_<area>_<descr>` regardless of site (exceptions cross site boundaries via `YBC`/`YCA`). See [[concepts/abap-exception-handling]].
- **Includes**: keep legacy `ZALM*INC*` style includes only when consolidation cost outweighs benefit; new code prefers small, well-named ABAP classes over INCLUDE-based modularization.
- **No abbreviations** in object descriptions: prefer `YEWM_CL_PUTAWAY_STRATEGY` over `YEWM_CL_PUT_STRAT`.

## Anti-patterns / pitfalls

- Defaulting to `$TMP` "to fix later" — promoted objects then drag wrong names into transport history.
- Using the wrong prefix (`Z*` for global, `Y*` for local) — inverts the project's market/global split.
- Shortened English that loses meaning (`YOTC_CL_OH` instead of `YOTC_CL_ORDER_HANDLER`).
- Mixing parameter direction prefixes (`l_matnr` for an importing param) — defeats the SCI check.

## Object-name format strings (opencode reference)

The opencode ABAP context library codifies the same convention as concrete format strings, useful for code generators and consistency checks:

| Object        | Format string                  | Example                          |
| ------------- | ------------------------------ | -------------------------------- |
| Class         | `[DEV]_CL_[DESCR]`             | `YOTC_CL_ORDER_FACTORY`          |
| Interface     | `[DEV]_IF_[DESCR]`             | `YOTC_IF_ORDER_HANDLER`          |
| Exception cls | `YCX_[DEV]_[DESCR]`            | `YCX_OTC_ORDER_INVALID`          |
| Function group| `[DEV]_[DESCR]`                | `YOTC_ORDER_UTILS`               |
| Function module| `Y_[DEV]_FM_[VERB]_[DESCR]`   | `Y_OTC_FM_CREATE_ORDER`          |
| Structure     | `[DEV]_ST_[DESCR]`             | `YOTC_ST_ORDER_HEADER`           |
| Data element  | `[DEV]_DT_[DESCR]`             | `YS00_DT_EXTID`                  |
| OData entity  | `[DEV][NUM]_[VERB]_[DESCR]`    | `IF001_GET_SALES_ORDER`          |

> [!note] Hungarian-notation tension
> [[sources/opencode-abap-context-library]] is internally split — `clean-abap-essentials.md` forbids Hungarian prefixes (per the official SAP Clean ABAP styleguide), while `abap-standards.md` and `quick-ref.md` prescribe `lv_`/`lt_`/`lo_`/`ls_`. Same pragmatic-vs-purist tension already present between this project's standards and SAP's. **Active rule**: follow the project standard (Hungarian prefixes are mandatory on Heinemann codebases); reserve the pure Clean ABAP variant for greenfield BTP ABAP repos with no legacy.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 4, Appendices 6.1–6.2.
- [[sources/heinemann-ewm-coding-standards]] — `01-naming-conventions.md` (full file).
- [[sources/opencode-abap-context-library]] — `abap-standards.md` (object format strings, OData entity naming), `quick-ref.md` (Hungarian-prefix reference), `clean-abap-essentials.md` (counter-position).
