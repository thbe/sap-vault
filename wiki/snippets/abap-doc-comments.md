---
title: "ABAP Doc Comments"
type: snippet
language: abap
runnable: false
tags: [documentation, abap-doc, comments, clean-abap, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP"]
status: stable
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[snippets/flower-box-header]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# ABAP Doc Comments

> The `"!` ABAP Doc syntax — used by ADT, SE80, and ATC's `SLIN_DESC` check to render and verify class/method/parameter documentation.

## Context

The Heinemann EWM standard mandates ABAP Doc comments on **all public and protected methods**, plus class-level documentation on every class. Enforcement is via ATC `SLIN_DESC` (Extended Program Check description coverage). This is a different mechanism from the [[snippets/flower-box-header|flower-box header]] — the flower-box documents *who/when/transport*; ABAP Doc documents *what the API does*.

## Code

### Class header

```abap
"! <p class="shorttext">Warehouse order processing utility</p>
"! Provides validation, posting, and output triggers for warehouse orders.
"! All methods raise {@link YCX_EWM_GENERAL_EXCEPTION} on failure.
CLASS ycl_ewm_who_processor DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC.
```

### Method header

```abap
PUBLIC SECTION.
  "! <p class="shorttext">Process delivery confirmation</p>
  "! Validates the delivery, posts goods movement, and triggers output.
  "! Idempotent: calling twice with the same delivery is a no-op on the second call.
  "!
  "! @parameter iv_dlv_no    | Delivery number (10 chars)
  "! @parameter iv_warehouse | Warehouse number (4 chars)
  "! @parameter rs_result    | Posting result with material doc number and status
  "! @raising ycx_ewm_authority_error | User lacks S_EWM_LGN/03 authorization
  "! @raising ycx_ewm_general_exception | Delivery not found or posting failure
  METHODS process_delivery_confirmation
    IMPORTING iv_dlv_no    TYPE vbeln_va
              iv_warehouse TYPE lgnum
    RETURNING VALUE(rs_result) TYPE yewm_st_post_result
    RAISING   ycx_ewm_authority_error
              ycx_ewm_general_exception.
```

### Interface

```abap
"! <p class="shorttext">Warehouse-order handler contract</p>
"! Implementations are dispatched by {@link YCL_EWM_WHO_DISPATCHER}
"! based on warehouse process type. See SAP Note 2345678 for the
"! handler registration procedure.
INTERFACE yif_ewm_who_handler PUBLIC.
  "! Dispatched by warehouse process type. Implementations must be idempotent.
  "! @parameter iv_who | Warehouse order number
  METHODS handle IMPORTING iv_who TYPE /scwm/de_who.
ENDINTERFACE.
```

### Local class (test class)

```abap
"! @testing YCL_EWM_WHO_PROCESSOR
CLASS ltcl_who_processor DEFINITION FINAL FOR TESTING
  DURATION SHORT
  RISK LEVEL HARMLESS.

  PRIVATE SECTION.
    METHODS:
      "! Happy path: valid delivery posts and returns success.
      process_delivery_ok    FOR TESTING RAISING cx_static_check,
      "! Negative: missing delivery raises general exception.
      process_delivery_404   FOR TESTING RAISING cx_static_check.
ENDCLASS.
```

## Tag reference

| Tag                            | Purpose                                                     |
| ------------------------------ | ----------------------------------------------------------- |
| `<p class="shorttext">…</p>`   | Required on first line — appears in ADT outliner & SE80     |
| `@parameter <name> \| <text>`  | One per parameter — appears in tooltip                      |
| `@raising <class> \| <text>`   | One per raised exception class                              |
| `{@link OBJECT}`               | Cross-reference (auto-resolved by ADT)                      |
| `@testing <CLASS>`             | On a test class — links to the SUT                          |
| `@deprecated`                  | Marks API as deprecated                                     |

## Conventions

- **First line is the shorttext** — keep ≤ 60 characters; this is what shows in tooltips and outliners.
- **Body in plain English** — describe behaviour, contracts, side-effects, idempotency.
- **One `@parameter` per parameter** — order matches the signature.
- **One `@raising` per exception class** — including `RAISING cx_static_check` if used.
- **Cross-reference with `{@link …}`** — ADT renders these as clickable links.
- **No marketing prose** — describe what callers need to know to use the method correctly.

## Verification

- ATC variant: include `SLIN_DESC` (description check). Configure to require shorttext on every public/protected method.
- ADT: hover over a method call → tooltip shows the rendered doc. If empty, the doc is missing or malformed.
- SE80: navigation pane "Documentation" tab.

## Related

- [[snippets/flower-box-header]] — the *other* mandatory comment block (governance metadata, not API doc).
- [[concepts/clean-abap]] — broader context.
- [[topics/abap-quality-gates]] — ATC enforcement.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §8.2 (Method Documentation Standard).
