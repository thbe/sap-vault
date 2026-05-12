---
title: "Dynamic Code Execution Whitelist"
type: pattern
tags: [security, dynamic-code, abap, atc]
sap_release: ["S/4HANA", "BTP ABAP", "ECC"]
status: draft
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[topics/abap-security-checklist]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# Dynamic Code Execution Whitelist

> When `SUBMIT <var>` or `CALL TRANSACTION <var>` cannot be eliminated, gate the dynamic call with a whitelist constant of allowed targets. Anything else raises an exception.

## Context

Dynamic code execution is a perennial ATC security finding: `SUBMIT <variable>`, `CALL TRANSACTION <variable>`, `CALL METHOD (variable_class)=>method`, and `GENERATE SUBROUTINE POOL` all let attacker-controlled (or careless-developer-controlled) data steer execution. Without input validation, this is a privilege-escalation vector.

The Heinemann EWM standard sets the policy:

| Pattern                                            | Policy                                | ATC Finding |
| -------------------------------------------------- | ------------------------------------- | ----------- |
| `SUBMIT <variable>`                                | Prohibited unless whitelist-validated | Error       |
| `CALL TRANSACTION <variable>`                      | Prohibited unless whitelist-validated | Error       |
| `GENERATE SUBROUTINE POOL`                         | Prohibited                            | Error       |
| `CALL METHOD (variable_class_name)=>method`        | Permitted with caution; document why  | Warning     |
| `ASSIGN COMPONENT <variable> OF STRUCTURE`         | Permitted                             | Info        |

## Pattern

### Anti-pattern (don't ship this)

```abap
" BAD — unvalidated dynamic execution
DATA: lv_tcode TYPE tcode.
lv_tcode = io_request->get_tcode( ).   " untrusted input
CALL TRANSACTION lv_tcode.
```

### Pattern (whitelist remediation)

```abap
CONSTANTS: BEGIN OF co_allowed_tcodes,
  monitor TYPE tcode VALUE 'ZEWM_MON',
  reprint TYPE tcode VALUE 'ZEWM_RP',
  unpack  TYPE tcode VALUE 'ZEWM_UP',
END OF co_allowed_tcodes.

DATA: lt_allowed TYPE STANDARD TABLE OF tcode.
lt_allowed = VALUE #(
  ( co_allowed_tcodes-monitor )
  ( co_allowed_tcodes-reprint )
  ( co_allowed_tcodes-unpack ) ).

IF line_exists( lt_allowed[ table_line = lv_tcode ] ).
  CALL TRANSACTION lv_tcode.
ELSE.
  RAISE EXCEPTION TYPE ycx_ewm_general_exception
    MESSAGE e001(yewm_global) WITH lv_tcode.
ENDIF.
```

### Same pattern for `SUBMIT`

```abap
CONSTANTS: BEGIN OF co_allowed_reports,
  inventory  TYPE syrepid VALUE 'ZEWM_INV_LIST',
  loadlist   TYPE syrepid VALUE 'ZEWM_LOAD_LIST',
END OF co_allowed_reports.

IF lv_report = co_allowed_reports-inventory
   OR lv_report = co_allowed_reports-loadlist.
  SUBMIT (lv_report) AND RETURN.
ELSE.
  RAISE EXCEPTION TYPE ycx_ewm_general_exception
    MESSAGE e002(yewm_global) WITH lv_report.
ENDIF.
```

## Key decisions

- **Constants, not table-driven** for high-trust whitelists — auditors can grep the source.
- **Table-driven** (e.g. `YS00_DB_ENH_C` rows) is acceptable for lower-trust scenarios when business needs to extend the list without a transport, but the table must be **client-independent + change-document logged**.
- **Always raise an exception** on whitelist miss — never silent skip, never default-execute.
- **Log the attempted value** in the exception text or audit log so security review can spot probing patterns.

## When to use

- Cockpit / launcher transactions that select a target program from configuration.
- Multi-variant report wrappers that switch the actual report by user choice.
- Workflow event handlers that dispatch to one of N processing methods.

## When NOT to use

- If the dynamic call has a fixed set of < 5 alternatives, **replace with a `CASE` statement** calling each statically. Cleaner, ATC-clean, no whitelist needed.
- If the call target is determined by a class hierarchy, use **polymorphic dispatch** (`CALL METHOD lo_handler->process( )`) — no dynamic name involved.

## Acceptance criteria

- ATC variant must include the dynamic-execution checks at **Error** level.
- Phase-gate scan: zero unmitigated dynamic `SUBMIT` or `CALL TRANSACTION` findings.
- Code review: every `(<var>)` parameterized call must show a whitelist guard within the same method (or document the upstream guard).

## Related

- [[topics/abap-security-checklist]] — full security scan list (BREAK-POINT, sy-uname gates, commented AUTHORITY-CHECK, ASSERT 1=2, etc.).
- [[concepts/abap-exception-handling]] — exception class hierarchy used by the raise.
- [[topics/abap-quality-gates]] — ATC variant configuration.

## Anti-patterns / pitfalls

- **Whitelist as `IF lv_x = 'A' OR lv_x = 'B' OR …` strung across 20 lines** — readable for 3 entries, unreadable at 11 (real ERL finding M-05). Use a constants structure or table.
- **Silent fallback** (`IF allowed. CALL TRANSACTION lv_tcode. ENDIF.`) — caller never learns the call was skipped.
- **Whitelist in a Z-table that's open for end-user maintenance** without change documents — defeats the purpose.
- **Logging only on success** — attacker probes vanish from the audit trail.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §2.3 (Dynamic Code Execution — Cross-Repo Policy); §2.1 findings M-01, M-02, M-05.
