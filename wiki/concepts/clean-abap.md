---
title: "Clean ABAP"
type: concept
tags: [clean-abap, abap, quality, oo]
sap_release: ["S/4HANA 2020+", "BTP ABAP", "NetWeaver 7.50+ (subset)"]
status: draft
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[concepts/mvc-in-abap]]"
  - "[[patterns/form-to-method-refactor]]"
  - "[[patterns/dynamic-code-whitelist]]"
  - "[[topics/abap-performance]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# Clean ABAP

> A set of style and design guidelines (originally published by SAP at <https://github.com/SAP/styleguides>) for writing readable, testable, modern ABAP — in spirit aligned with Robert Martin's "Clean Code".

## Overview

Clean ABAP favours object-oriented, modular, well-named, side-effect-light code over the procedural / report-driven style typical of older ECC code. It is the de-facto baseline for new development on S/4HANA and BTP ABAP (Steampunk).

The Heinemann S/4 standard ([[sources/sap-development-standard-approach-abap-fiori-v1]]) leans toward Clean ABAP without naming it explicitly: it mandates OO over procedural, MVC for new dev, class-based exceptions, modularization, and clear naming.

## Key ideas

- **Object-oriented by default**: classes + interfaces; procedural code only for legacy maintenance.
- **MVC separation**: data access, business logic, and presentation in distinct modules. See [[concepts/mvc-in-abap]].
- **Class-based exceptions** (`CX_*`) for error handling — never message types `E`/`A` from deep code; never `SY-SUBRC` propagation.
- **Small, single-purpose methods**; the standard recommends modularization of "complex logic" into FORMs/methods so each unit fits on one screen.
- **Constants over magic numbers/strings** (`CONSTANTS co_… VALUE …`).
- **Avoid obsolete language**: no `TABLES` work-areas, no header lines, no `OCCURS`, no obsolete FMs.
- **Don't modify standard SAP** — use enhancement spots / BAdIs / append structures. See [[patterns/table-driven-enhancement-framework]].
- **Readability over micro-optimization** — but DB and internal-table optimization remain mandatory; see [[topics/abap-performance]].
- **Multilingual support**: text symbols + OTR, never hard-coded UI strings.
- **No `FORM` / `PERFORM` in new code** — convert to private methods of a class. Existing FORMs are tolerated only in unchanged legacy includes; touching one triggers refactor. See [[patterns/form-to-method-refactor]].
- **No dynamic `SUBMIT` / `CALL TRANSACTION` / `GENERATE SUBROUTINE POOL`** without an explicit allow-list and security review. See [[patterns/dynamic-code-whitelist]].
- **Class-based exceptions everywhere**: `YCX_*` hierarchy, `RAISING` declared, `TRY/CATCH` at the right layer (never empty `CATCH`). See [[concepts/abap-exception-handling]].

## How it works

Clean ABAP is enforced through a combination of:

1. **ATC (ABAP Test Cockpit)** with appropriate check variants.
2. **Code Inspector** SCI variants (project-specific, e.g. `Y_NAMING_CONVENTIONS`).
3. **Code reviews** referencing the project standard.
4. **CI gates** in modern setups (abapGit + ATC pipeline; gCTS on BTP).

## Code example

```abap
" Clean-ish: class-based, named, exception-driven
CLASS yotc_cl_order_factory DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    CONSTANTS co_default_order_type TYPE auart VALUE 'TA'.

    METHODS create
      IMPORTING iwa_header TYPE yotc_st_order_header
      RETURNING VALUE(rtp_order_id) TYPE vbeln_va
      RAISING   ycx_otc_order_invalid.
ENDCLASS.

" Anti-pattern (legacy):
" REPORT zorder.
" TABLES vbak.
" SELECT * FROM vbak.
"   PERFORM check.
"   IF sy-subrc <> 0. MESSAGE 'Bad' TYPE 'E'. ENDIF.
" ENDSELECT.
```

## Release & version notes

> [!info] Available since
> Clean ABAP guidelines are version-agnostic in spirit, but several preferred constructs (constructor expressions, inline declarations, `FOR` iteration, `REDUCE`) require **NW 7.40 SP08+**. RAP-specific guidance assumes **S/4HANA 1909+ or BTP ABAP**.

## Related

- [[concepts/abap-naming-conventions]]
- [[concepts/mvc-in-abap]]
- [[concepts/restful-application-programming-model]]
- [[topics/abap-performance]]
- [[topics/abap-quality-gates]]

## Anti-patterns / pitfalls

- Procedural reports with embedded SQL, screen IO, and business logic mixed.
- `SY-SUBRC` plumbing instead of class-based exceptions.
- Modifying SAP standard objects directly instead of using enhancements.
- Header lines on internal tables; `TABLES` declarations.
- Hardcoded English messages — fails multilingual requirements.
- Dropping Clean ABAP rules under "performance" pretext when actual hot path is elsewhere.

## Additional Clean ABAP rules

### Class design defaults

- **`FINAL` by default** — every class is `FINAL` unless explicit polymorphism is needed. Prevents accidental inheritance chains.
- **`CREATE PRIVATE` + factory method** for classes with non-trivial construction. The factory returns an interface reference; the class is invisible to consumers.
- **One public interface per class** when the class is consumed externally — keeps the contract narrow and test-doubleable. See [[patterns/test-doubles-and-dependency-injection]].
- **Constructor injection** for dependencies; no direct global access (`SY-*`, transparent tables) inside business logic.

```abap
CLASS yotc_cl_order_factory DEFINITION PUBLIC FINAL CREATE PRIVATE.
  PUBLIC SECTION.
    INTERFACES yotc_if_order_factory.
    CLASS-METHODS create
      RETURNING VALUE(ro_factory) TYPE REF TO yotc_if_order_factory.
ENDCLASS.
```

### Exception class hierarchy at a glance

Clean ABAP picks the exception parent based on contract semantics — see [[concepts/abap-exception-handling]] for the full treatment.

| Parent             | Semantics                                                  | When to use                              |
| ------------------ | ---------------------------------------------------------- | ---------------------------------------- |
| `CX_STATIC_CHECK`  | Compiler enforces `RAISING` clause on every caller         | **Default** — business / domain errors   |
| `CX_DYNAMIC_CHECK` | Compiler does not enforce; runtime panics if uncaught      | Programmer errors detected at runtime    |
| `CX_NO_CHECK`      | Never declared in `RAISING`; can be `RESUMABLE`            | System errors, RF-dialog resumable flows |

> [!warning] Hungarian notation tension
> The official SAP Clean ABAP styleguide **forbids** Hungarian prefixes (`lv_`, `lt_`, `lo_`, `ls_`). The Heinemann project standards and `quick-ref.md` in [[sources/opencode-abap-context-library]] **prescribe** them. Same tension is captured in [[concepts/abap-naming-conventions]]. Resolution: follow the project standard for the codebase you are working in; do not retrofit either way mid-flight.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §2.3, §2.4, §2.5, §3.5, §3.7.
- [[sources/heinemann-ewm-coding-standards]] — `02-coding-guidelines.md` (FORMs, exceptions, dynamic code), `03-code-quality-performance.md` (modularization, ABAP doc).
- [[sources/opencode-abap-context-library]] — `clean-abap-essentials.md` (FINAL/CREATE PRIVATE, exception hierarchy, Hungarian-notation stance), `modern-abap-patterns.md`.
- *(future)* SAP Clean ABAP styleguide on GitHub — to be ingested.
