---
title: "ABAP Exception Handling"
type: concept
tags: [exceptions, error-handling, clean-abap, oo, abap]
sap_release: ["S/4HANA", "BTP ABAP", "NetWeaver 7.40+"]
status: draft
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[topics/abap-security-checklist]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# ABAP Exception Handling

> Class-based exceptions (`CX_*` / project `YCX_*`) are the only acceptable error mechanism in modern ABAP. `SY-SUBRC` plumbing, `MESSAGE … TYPE 'E'` from deep code, and `EXCEPTIONS OTHERS = 0` are anti-patterns.

## Overview

Class-based exceptions decouple error detection from error reporting, allow rich context (text, parameters, previous exception chain), and integrate cleanly with TRY/CATCH/CLEANUP blocks. Modern ABAP standards (Clean ABAP, Heinemann EWM) treat them as mandatory for new code and a remediation target for legacy code on touch.

## Key ideas

- **Project-specific exception class hierarchy** rooted at a single base (e.g. `YCX_EWM_GENERAL_EXCEPTION`) inheriting from `CX_STATIC_CHECK` (or `CX_DYNAMIC_CHECK` / `CX_NO_CHECK` per use case).
- **Specialised subclasses by failure category**: authority, DB, network/RFC, configuration, business rule.
- **Resumable exceptions** (`CX_NO_CHECK` with `RESUMABLE`) for RF-dialog flows where the user can fix and retry without restarting the transaction.
- **Exception chaining** via `EXPORTING previous = lx_root` preserves the underlying cause across layers.
- **Catch-all** at module boundaries: `CATCH cx_root INTO DATA(lx_root)` then log + re-raise as the project's general exception.
- **Function-module exceptions** (`EXCEPTIONS not_found = 1, OTHERS = 2`) translate to class-based exceptions at the call site — never propagated as raw `sy-subrc`.
- **Banned**: `EXCEPTIONS OTHERS = 0` (silently swallows errors), `MESSAGE 'foo' TYPE 'E'` from deep helpers (couples logic to message classes), `MESSAGE … RAISE` in class methods (forces ABAP runtime to special class change).

## How it works

### Project hierarchy (Heinemann EWM example)

```
CX_STATIC_CHECK
└── YCX_EWM_GENERAL_EXCEPTION         (base)
    ├── YCX_EWM_AUTHORITY_ERROR       (authorization failures)
    ├── YCX_EWM_DB_ERROR              (database access failures)
    └── YCX_EWM_RF_DIALOG_ERROR       (RF dialog — resumable)
```

The base class typically defines:

- A constant per error condition (`co_not_found`, `co_invalid_input`, …).
- Linkage to a message class via `IF_T100_MESSAGE` for end-user text.
- Optional `previous` parameter for chaining.

### Class method — TRY/CATCH template

```abap
METHOD process_delivery.
  TRY.
      lo_dlv->process( iv_dlv_no ).

    CATCH ycx_ewm_db_error INTO DATA(lx_db).
      ycl_ewm_log=>error( lx_db->get_text( ) ).
      RAISE EXCEPTION TYPE ycx_ewm_general_exception
        EXPORTING previous = lx_db.

    CATCH cx_root INTO DATA(lx_root).
      "! Catch-all for unexpected errors at the module boundary
      ycl_ewm_log=>error( lx_root->get_text( ) ).
      RAISE EXCEPTION TYPE ycx_ewm_general_exception
        EXPORTING previous = lx_root.
  ENDTRY.
ENDMETHOD.
```

### Function-module call — translate to exception

```abap
CALL FUNCTION 'EWM_WAREHOUSE_ORDER_GET'
  EXPORTING  iv_warehouse = lv_lgnum
  IMPORTING  es_who       = ls_who
  EXCEPTIONS
    not_found    = 1
    no_authority = 2
    OTHERS       = 3.

CASE sy-subrc.
  WHEN 0.
    " continue
  WHEN 1.
    RAISE EXCEPTION TYPE ycx_ewm_general_exception
      MESSAGE e001(yewm_global) WITH lv_lgnum.
  WHEN 2.
    RAISE EXCEPTION TYPE ycx_ewm_authority_error.
  WHEN OTHERS.
    RAISE EXCEPTION TYPE ycx_ewm_general_exception
      MESSAGE e999(yewm_global).
ENDCASE.
```

### Resumable exception (RF dialog)

```abap
CLASS ycx_ewm_rf_dialog_error DEFINITION
  PUBLIC
  INHERITING FROM cx_no_check
  CREATE PUBLIC.
ENDCLASS.

" Raiser:
RAISE RESUMABLE EXCEPTION TYPE ycx_ewm_rf_dialog_error.

" Caller:
TRY.
    process_step( ).
  CATCH RESUMABLE ycx_ewm_rf_dialog_error.
    show_warning_to_user( ).
    RESUME.   " continue after the RAISE
ENDTRY.
```

## Release & version notes

> [!info] Available since
> Class-based exceptions: NetWeaver **6.10+**. `RESUMABLE` exceptions: NetWeaver **7.0+**. The `CX_ROOT` family is universally available.

> [!warning] BTP ABAP / ABAP Cloud
> `MESSAGE … TYPE 'E'` is restricted in ABAP for Cloud Development. Class-based exceptions are the only portable mechanism.

## Anti-patterns / pitfalls

- **`EXCEPTIONS OTHERS = 0`** — silently maps every unhandled FM exception to "success". Banned by Heinemann EWM standards for any critical FM call.
- **`SY-SUBRC` propagation** — bubbling `sy-subrc` up the call stack via OUT parameters defeats type safety and loses context.
- **`MESSAGE … TYPE 'E'` in class methods** — couples class to a message class and short-circuits TRY/CATCH; raise the class-based exception with `MESSAGE` ID instead.
- **Empty CATCH blocks** — silently swallowing exceptions. If you catch, you must log, re-raise, or document why neither is needed.
- **Catching `CX_ROOT` everywhere** — overly broad; catch the most specific class you can handle and let the rest propagate.
- **Forgetting `CLEANUP`** — open cursors, locked tables, or held resources leak when an exception bypasses normal flow.

## Related

- [[concepts/clean-abap]]
- [[topics/abap-security-checklist]] — disabled `AUTHORITY-CHECK` is a security finding; raising `YCX_EWM_AUTHORITY_ERROR` is the prescribed remediation.
- [[topics/abap-quality-gates]] — ATC enforces no `EXCEPTIONS OTHERS = 0` for critical FMs.

## CX_* hierarchy semantics

ABAP's three exception parents differ only in how the compiler and runtime treat them. Picking the wrong one weakens the contract.

| Parent             | Declared in `RAISING`? | Caller must catch? | Use for                                          |
| ------------------ | ---------------------- | ------------------ | ------------------------------------------------ |
| `CX_STATIC_CHECK`  | **Required**           | Yes (compiler enforced) | Recoverable business / domain errors             |
| `CX_DYNAMIC_CHECK` | Required               | Yes (runtime panic if not) | Programmer-detectable errors (precondition violations) |
| `CX_NO_CHECK`      | **Forbidden**          | No                 | System errors; **resumable** RF-dialog patterns  |

**Rule of thumb**: pick `CX_STATIC_CHECK` unless you have a specific reason. It pushes error handling into the type system and prevents silent omissions.

## Preferred `RAISE` syntax (NW 7.52+)

```abap
" Preferred — RAISE EXCEPTION NEW (concise, no extra DATA declaration)
RAISE EXCEPTION NEW ycx_otc_order_invalid(
  textid    = ycx_otc_order_invalid=>co_customer_blocked
  customer  = lv_customer ).

" Older but still valid
RAISE EXCEPTION TYPE ycx_otc_order_invalid
  EXPORTING textid   = ycx_otc_order_invalid=>co_customer_blocked
            customer = lv_customer.
```

> [!info] Available since
> `RAISE EXCEPTION NEW` syntax: NetWeaver **7.52**. Strongly preferred on all systems that support it.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §7 (Error Handling & Robustness).
- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 2 (Quality), §3.7 (OO).
- [[sources/opencode-abap-context-library]] — `clean-abap-essentials.md` (CX hierarchy decision rules), `modern-abap-patterns.md` (`RAISE EXCEPTION NEW`).
