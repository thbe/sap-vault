---
title: "ABAP Security Checklist"
type: topic
tags: [security, atc, audit, governance, abap]
sap_release: ["S/4HANA", "ECC", "BTP ABAP"]
status: stable
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[topics/abap-quality-gates]]"
  - "[[patterns/dynamic-code-whitelist]]"
  - "[[concepts/abap-exception-handling]]"
created: 2026-05-11
updated: 2026-05-13
---

# ABAP Security Checklist

> Concrete patterns to grep for, ATC checks to enable, and remediation steps for the recurring security findings in custom ABAP. Drawn from a real audit of 17 ERL findings + ALM scan plan.

## Why this matters

Custom ABAP carries a stable set of security smells: developer breakpoints in production, username gates, commented-out authority checks, dynamic execution, abend-as-error-handling. They are **easy to grep for** and **expensive to fix in the wrong order** (a breakpoint behind a commented authority check is a privilege-escalation chain, not two separate issues). This page lists the patterns and the remediation in one place.

## Severity scale

| Severity | Examples                                              | Gate impact                |
| -------- | ----------------------------------------------------- | -------------------------- |
| HIGH     | Username gate, breakpoint in BAdI, disabled auth check | Block migration / release  |
| MEDIUM   | Dynamic SUBMIT/CALL TRANSACTION, hardcoded user list, `ASSERT 1 = 2` | Fix before phase exit |
| LOW      | Unused variables, unreachable code                    | ATC remediation            |
| INFO     | Missing documentation                                  | Phase 6 cleanup            |

## Patterns to scan for

### 1. Developer breakpoints

```abap
BREAK-POINT.
break <username>.       " e.g. break schmidt
break-point id 'X'.
```

| Risk     | Stops the application server in PRD; if a user matches `break <user>`, that user freezes the entire dialog session. |
| -------- | ------------------------------------------------------------------------------------------------------------------- |
| Find     | `rg -wn 'BREAK-POINT\|^\s*break\s'` across the source set.                                                           |
| Fix      | Delete. If you need conditional debugging, use the ABAP debugger's session breakpoints.                              |
| Severity | HIGH (in BAdI implementations / production paths)                                                                   |

### 2. Username gates

```abap
CHECK sy-uname EQ 'SCHMIDA'.
IF sy-uname EQ 'X' OR sy-uname EQ 'Y' OR sy-uname EQ 'Z'.
```

| Risk     | Logic gated on a single named user — the user leaves, the function dies. Worse: anyone learning the username acquires the access. |
| -------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Find     | `rg -n "sy-uname\s+(EQ\|=)\s+'"`                                                                                                  |
| Fix      | Replace with role-based `AUTHORITY-CHECK OBJECT 'S_TCODE'` (or appropriate auth object) + SU24 maintenance.                      |
| Severity | HIGH                                                                                                                              |

### 3. Disabled / commented authority checks

```abap
*  AUTHORITY-CHECK OBJECT 'Z_EWM_FOO'
*    ID 'ACTVT' FIELD '03'.
" or:
" temporarily disabled — TODO: re-enable
```

| Risk     | The check exists in design, is missing in execution. Auditors miss it (looks like a comment); attackers don't (function runs). |
| -------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Find     | `rg -n '^\s*\*\s*AUTHORITY-CHECK\|^\s*\"\s*AUTHORITY-CHECK'`                                                                  |
| Fix      | Uncomment, restore, add SU24 entry, transport.                                                                                |
| Severity | HIGH                                                                                                                          |

### 4. Dynamic code execution

```abap
SUBMIT (lv_report) AND RETURN.
CALL TRANSACTION lv_tcode.
GENERATE SUBROUTINE POOL lt_source NAME lv_name.
```

| Risk     | Untrusted input determines what code executes. Privilege-escalation vector. |
| -------- | --------------------------------------------------------------------------- |
| Find     | ATC variant with the dynamic-call checks enabled.                           |
| Fix      | Whitelist remediation pattern — see [[patterns/dynamic-code-whitelist]].    |
| Severity | MEDIUM (with whitelist) → HIGH (without)                                    |

### 5. `ASSERT 1 = 2` (intentional abend)

```abap
ASSERT 1 = 2.   " "this can never happen"
```

| Risk     | Application crashes mid-LUW; no rollback, no user message, no log entry. Breaks audit trail. |
| -------- | -------------------------------------------------------------------------------------------- |
| Find     | `rg -n 'ASSERT\s+1\s*=\s*2'`                                                                  |
| Fix      | Replace with `RAISE EXCEPTION TYPE ycx_ewm_general_exception` carrying a real message.       |
| Severity | MEDIUM                                                                                       |

### 6. `EXCEPTIONS OTHERS = 0`

```abap
CALL FUNCTION 'EWM_…'
  EXCEPTIONS OTHERS = 0.
```

| Risk     | Every error from the called FM is silently mapped to "success". Defects look like working code. |
| -------- | ----------------------------------------------------------------------------------------------- |
| Find     | `rg -n 'OTHERS\s*=\s*0'`                                                                         |
| Fix      | Map every named exception to a real `sy-subrc` value and translate to a class-based exception.   |
| Severity | HIGH (for critical FMs), MEDIUM (for read-only FMs)                                              |

### 7. RFC calls without exception handling

```abap
CALL FUNCTION 'RFC_…' DESTINATION 'X'
  EXPORTING …
  IMPORTING ….
" no EXCEPTIONS clause, no TRY/CATCH
```

| Risk     | Network failure or RFC timeout dumps the entire dialog step. |
| -------- | ------------------------------------------------------------ |
| Find     | Manual review around `DESTINATION` / `IN BACKGROUND TASK`.    |
| Fix      | Wrap in `TRY/CATCH cx_root` (or specific RFC exceptions).    |
| Severity | HIGH                                                         |

### 8. Test programs and demo classes in production paths

```abap
zewm_test_alert.prog
zcl_ewmgwl_demo.clas
zsceblo_log_example.prog
```

| Risk     | Test code with debug statements transports to PRD and drifts unmaintained. |
| -------- | -------------------------------------------------------------------------- |
| Find     | Object-name scan for `_test`, `_demo`, `_example`, `_old`.                  |
| Fix      | Delete before migration (Phase 0). Source control retains history.          |
| Severity | LOW (existence) → HIGH (if they contain breakpoints)                        |

## Pre-migration scan playbook

Before any Phase 1 (classification) starts, run this scan against both source repos:

```bash
# Quick high-severity grep over /src
rg -n 'BREAK-POINT|^\s*break\s|sy-uname\s+EQ\s+'\''|^\s*\*\s*AUTHORITY-CHECK|ASSERT\s+1\s*=\s*2|OTHERS\s*=\s*0' src/

# ATC with security check variant
# Transaction SCI / ATC, variant including S_OBJECT_AUTHORITY_CHECK + DYNAMIC_CHECK + PERFORMANCE_DB
```

Document every finding in a tracker table:

| # | Severity | Category | Object | Finding | Fix | Target Repo | Phase |
| - | -------- | -------- | ------ | ------- | --- | ----------- | ----- |
| H-01 | HIGH | Credentials | `zewm_del_loc_bp.prog` | `CHECK sy-uname EQ 'X'` + `BREAK-POINT` | DELETE entire object | — | Phase 0 |

(Real example from ERL audit — the entire program was a one-user backdoor, deletion was the fix.)

## Acceptance criteria per phase

| Phase | Criterion                                                       |
| ----- | --------------------------------------------------------------- |
| 0     | Delete-list cleared (test/debug objects removed)                 |
| 1     | Source-system security scan completed; findings documented       |
| 3     | ALM HIGH/MEDIUM findings fixed in target repo                    |
| 4     | ERL HIGH/MEDIUM findings fixed in target repo                    |
| 5     | Cross-repo ATC + Extended Program Check + authority validation   |
| ALL   | Zero HIGH or MEDIUM security findings in any target repo         |

## Tools

- [[entities/tools/atc]] — security check variants.
- *(future)* `entities/tools/slin` — Extended Program Check; flags many of these patterns.
- *(future)* `entities/tools/abapgit` — for the grep-the-source approach (works on serialized files outside the SAP system).
- *(future)* `entities/tools/code-vulnerability-analyzer` — SAP CVA, when licensed.

## Related

- [[topics/abap-quality-gates]] — security findings as a gate criterion.
- [[patterns/dynamic-code-whitelist]] — remediation for §4.
- [[concepts/abap-exception-handling]] — remediation for §5, §6, §7.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §2 (Security Findings) including the full ERL 17-finding table.
