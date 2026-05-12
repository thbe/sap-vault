---
title: "FORM → Method Refactor"
type: pattern
tags: [refactoring, clean-abap, form, oo, technical-debt, abap]
sap_release: ["S/4HANA", "BTP ABAP (mandatory)", "NetWeaver 7.40+"]
status: draft
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-performance]]"
created: 2026-05-11
updated: 2026-05-11
---

# FORM → Method Refactor

> Mechanical pattern for retiring `FORM`/`PERFORM` subroutines in favour of class methods. The refactor is a prerequisite for ABAP Unit testability, ABAP Cloud compatibility, and clean OOP encapsulation.

## Context

`FORM`/`PERFORM` is **obsolete** in modern ABAP:

- **ATC level "Error"** in ABAP for Cloud Development — code with FORMs cannot be deployed to Steampunk.
- **ATC level "Warning"** in Standard ABAP — flagged but not blocking on-premise.
- Prevents ABAP Unit testing — FORMs cannot be invoked from test classes cleanly.
- Breaks OO encapsulation — implicit access to function-group globals.

The Heinemann EWM scan found **8,229 FORM definitions** across 1,328 source files (5,370 ERL + 2,859 ALM, ~12% of files). Refactoring is multi-year work — this pattern keeps each refactor mechanical and reviewable.

## Pattern

### Before — FORM in a function-group include

```abap
*&---------------------------------------------------------------------*
*& Form check_authority
*&---------------------------------------------------------------------*
FORM check_authority USING iv_object TYPE tobj_name.

  AUTHORITY-CHECK OBJECT iv_object
                  ID 'ACTVT' FIELD '03'.
  IF sy-subrc <> 0.
    MESSAGE e001(yewm_global).
  ENDIF.

ENDFORM.

" Caller:
PERFORM check_authority USING 'S_TCODE'.
```

### After — Static method on a utility class

```abap
CLASS ycl_ewm_authority DEFINITION
  PUBLIC
  FINAL
  CREATE PRIVATE.

  PUBLIC SECTION.
    CLASS-METHODS check
      IMPORTING iv_object TYPE tobj_name
      RAISING   ycx_ewm_authority_error.
ENDCLASS.

CLASS ycl_ewm_authority IMPLEMENTATION.
  METHOD check.

    AUTHORITY-CHECK OBJECT iv_object
                    ID 'ACTVT' FIELD '03'.
    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE ycx_ewm_authority_error.
    ENDIF.

  ENDMETHOD.
ENDCLASS.

" Caller:
TRY.
    ycl_ewm_authority=>check( iv_object = 'S_TCODE' ).
  CATCH ycx_ewm_authority_error.
    " handle / re-raise
ENDTRY.
```

## Mechanical refactor steps

For each FORM:

1. **Identify the right home class.** A new utility class per concern, or an existing class that already owns the area.
2. **Convert USING parameters → IMPORTING / EXPORTING / CHANGING / RETURNING** with explicit types.
3. **Replace `MESSAGE … TYPE 'E'` with `RAISE EXCEPTION TYPE ycx_…`** — see [[concepts/abap-exception-handling]].
4. **Remove implicit access to function-group globals** — pass them as IMPORTING parameters or store on the class.
5. **Add ABAP Doc** (`"!`) on the public method — see [[snippets/abap-doc-comments]].
6. **Add a test method** to the class's local test include (`ltcl_*`) — at minimum one happy-path case. Heinemann EWM mandates this for all new classes.
7. **Update every `PERFORM` site** to a static method call.
8. **Remove the FORM** from the include after all callers are migrated.
9. **Run ATC + ABAP Unit + Extended Program Check** before transport.

## Prioritisation strategy

Don't refactor everything at once. The Heinemann EWM phasing:

| Priority | Targets                                                            | Phase |
| -------- | ------------------------------------------------------------------ | ----- |
| HIGH     | Performance-critical packages (`zewmrt`, `zewmmf`, `zewmgi`, RF dialogs) | 6     |
| HIGH     | FORMs called from newly-created OOP code                           | 5     |
| MEDIUM   | Monitoring, action-handling, retrieval                              | 6     |
| LOW      | Config packages (rarely modified)                                  | post-migration |
| SKIP     | Migration tooling (`zewmmig`, `zewmmi`) — being archived            | n/a   |

**Rule of thumb**: refactor on touch. Any FORM you modify for a defect or enhancement gets converted to a method as part of the change. This keeps the debt curve flat without requiring a dedicated cleanup project.

## Acceptance criteria

After refactor of a HIGH-priority package:

- Zero `FORM` / `PERFORM` definitions remaining.
- ATC clean against project variant.
- ABAP Unit coverage > 0% (every refactored method has at least one test).
- Performance baseline preserved (< 5% regression vs pre-refactor measurement) — see [[topics/abap-performance]] §RF-dialog SLA.

## When NOT to refactor

- **One-shot programs** scheduled for archival — wasted effort.
- **Customer-modified SAP standards** (e.g. enhancement-implementation FORMs) — refactor lives outside the enhancement; convert the *implementation* to a method call into a custom class.
- **Subroutine pools generated at runtime** (`GENERATE SUBROUTINE POOL`) — those are a separate, harder problem; eliminate the dynamic generation first.

## Related

- [[concepts/clean-abap]] — broader OO baseline this pattern enforces.
- [[concepts/abap-exception-handling]] — replaces `MESSAGE … TYPE 'E'` from FORMs.
- [[topics/abap-performance]] — refactored methods can be hot-spotted via SAT in ways FORMs cannot.
- [[snippets/abap-doc-comments]] — required on the new public method.

## Anti-patterns / pitfalls

- **Wrapping a FORM in a method without changing internals** — the FORM still exists; ATC still flags it. Move the *body* into the method and delete the FORM.
- **Keeping `MESSAGE … TYPE 'E'` inside the new method** — same coupling problem; raise an exception instead.
- **Passing the entire global state as one giant structure** — refactor opportunity to break dependencies, not preserve them.
- **Refactoring without a test** — undetected regressions in long-tail FORMs.
- **Mass-converting an entire package in one transport** — un-reviewable. Batch by class or sub-area.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §3 (Technical Debt — FORM Routines).
