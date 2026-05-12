---
title: "Flower-Box Header"
type: snippet
language: abap
runnable: false
tags: [documentation, header, comments, governance, abap]
sap_release: ["S/4HANA (Heinemann project)", "ECC (compatible)"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[topics/abap-quality-gates]]"
  - "[[snippets/abap-doc-comments]]"
created: 2026-05-11
updated: 2026-05-11
---

# Flower-Box Header

> Standardised comment block placed at the top of every custom program, class method, enhancement spot, and any other place custom code is inserted. Plus a "Modification Log" block that grows with each change.

## Context

The Heinemann standard requires this header for **traceability and audit**. It links code to the WRICEF/RICEFW object, the project phase, the responsible developer, and all transport requests involved. Auditors and reviewers grep for the markers; modifications are bracketed with `Start of change <key>` / `End of change <key>` so diffs read like a journal.

## Code

### Initial header (new code)

```abap
*&--------------------------------------------------------------------------------------------*
*& Market           : <Market name or All>                                                    *
*& Author           : <developer name>                                                        *
*& Created On       : <creation date>                                                         *
*& Object ID        : <RICEFW object id>                                                      *
*& Project/Release  : S4 / Phase (e.g. S1U1, S1U2, S1U3, K1U1, P3W2)                          *
*& Functionality    : <Explain the functionality in brief>                                    *
*& Transport        : <List all transport request numbers used to achieve the functionality>  *
*&--------------------------------------------------------------------------------------------*
```

### Modification-log block (added below the header on first change)

```abap
*&--------------------------------------------------------------------------------------------*
*& Modification Log                                                                           *
*&--------------------------------------------------------------------------------------------*

*&--------------------------------------------------------------------------------------------*
*& Changed By       : <developer name / userid>                                               *
*& Changed On       : <change date>                                                           *
*& Object ID        : <RICEFW object id / CR Number>                                          *
*& Project/Release  : S4 / Phase (e.g. S1U1, S1U2, S1U3, K1U1, P3W2)                          *
*& New Functionality: <Explain the changes in brief>                                          *
*& Transport        : <List all the transport request numbers used>                           *
*& Search Key       : User/TR/Date                                                            *
*&--------------------------------------------------------------------------------------------*
```

### In-line change markers

```abap
* Start of change - <Search Key>
DATA(lv_new) = some_new_logic( ).
* End of change - <Search Key>
```

## Explanation

- **Header is mandatory** — programs, classes, methods, enhancement implementations, BAdI implementations, includes containing custom code. Anywhere a developer inserts their own logic.
- **Search key** = `User/TR/Date` (e.g. `TBENDLER/D01K901234/2026-05-11`). Pick something searchable; it must match the `Start of change` / `End of change` brackets.
- **Modification log grows downward** — never overwrite a previous entry.
- **Deletions**: if previously transported code is no longer needed, **delete it** rather than commenting it out. Source control (transport history + abapGit) holds the past.
- For class methods, place the header inside the implementation block at the top of the method.

## Variations

- **abapGit / Git-based projects**: redundant with commit metadata, but keep for in-system review tools (ADT, SE38, SE80) that don't show Git context.
- **RAP / CDS**: apply the same idea using ABAP doc comments (`"!`) and CDS annotations (`@EndUserText.label`, etc.) — the flower-box format is mainly for procedural / class code. See [[snippets/abap-doc-comments]] for the modern complement.
- **EWM project variant**: the `Market:` field is mandatory and uses values `All` / `EWM` / `ALM` (Alicante) / `ERL` (Erlenbach) — aligned with the [[patterns/multi-site-repo-architecture|multi-site package layout]]. Auditors filter by this field during reviews.
- **Short includes**: a one-line variant naming Object ID + Transport is acceptable if the parent already carries the full header.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §3.7.9.
- [[sources/heinemann-ewm-coding-standards]] — confirms continued use; `Market:` field values per site.
