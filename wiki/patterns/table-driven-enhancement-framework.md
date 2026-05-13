---
title: "Table-Driven Enhancement Framework"
type: pattern
tags: [enhancement, badi, governance, custom-framework, abap]
sap_release: ["S/4HANA (Heinemann project)", "ECC (compatible)"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-13
---

# Table-Driven Enhancement Framework

> A custom dispatcher pattern: BAdI / customer-exit code that delegates to dynamically-resolved class methods listed in a configuration table — toggleable per market and per business identifier without code changes.

## Problem

In a multi-market S/4 implementation, the same SAP enhancement spot (BAdI / user-exit) must execute different code for different markets, business contexts, or WRICEFs. The naive approach modifies the BAdI implementation every time a new requirement lands → constant transports of the same hot object, merge conflicts between teams, and no easy way to switch behaviour off in a specific environment.

## Solution

Two custom tables drive a dispatcher class:

- **`YS00_DB_ENH_C` (Control)** — declares each `EXITID` (a logical enhancement identifier), the SAP exit name + method it hooks into, the search algorithm (specific vs generic), and up to 5 flex-field names that act as filter dimensions.
- **`YS00_DB_ENH_V` (Value)** — for each `EXITID`, lists the concrete filter values, the implementing class + method, an active flag, calling sequence number, and WRICEF reference.

A single BAdI implementation calls the dispatcher class **`YS00_CL_COMMON_ENHC_FRWRK`**, which:
1. Reads the configured filter fields for this exit (`GET_FLEX_FIELD_NAME`).
2. Resolves the matching active rows in the value table (`GET_EXITS`), specific-to-generic.
3. Calls each resolved class/method dynamically in sequence order.

Each implementing method shares the same signature as the BAdI/exit, so `CALL METHOD` works generically.

Maintenance is via view cluster, exposed by **TCODE `YS00_ENH_CNTRL`**.

## Implementation

### Tables (essential fields)

`YS00_DB_ENH_C` — control:

| Field            | Key | Type                | Meaning                          |
| ---------------- | --- | ------------------- | -------------------------------- |
| MANDT            | Y   | MANDT               | Client                           |
| EXITID           | Y   | YS00_DT_EXTID       | Enhancement identifier           |
| FILTER_FLD1..5   |     | YS00_DT_FILFLDn     | Names of filter fields           |
| SAP_EXITNAME     |     | YS00_DT_EXITNAME    | SAP Exit name                    |
| SAP_METHOD_NAME  |     | SEOCPDNAME          | Method name in SAP exit          |
| SEARCH_ALGO      |     | YS00_DT_SERALGO     | `S` specific / `G` generic       |

`YS00_DB_ENH_V` — value:

| Field           | Key | Type            | Meaning                                |
| --------------- | --- | --------------- | -------------------------------------- |
| MANDT           | Y   | MANDT           | Client                                 |
| EXITID          | Y   | YS00_DT_EXTID   | Enhancement identifier                 |
| FILTERVALUE1..5 | Y   | YS00_DT_VALn    | Filter field values                    |
| SEQUENCE_NO     |     | YS00_DT_SEQ     | Calling sequence (unique per EXITID)   |
| CLASS_NAME      |     | SEOCLSNAME      | Implementing class                     |
| METHOD_NAME     |     | SEOCPDNAME      | Implementing method                    |
| ACTIVE_FLAG     |     | YS00_DT_ACTFLG  | `X` = on                               |
| WRICEF          |     | YS00_DT_WRICEF  | Free reference to WRICEF               |

### Dispatcher class skeleton

```abap
"! YS00_CL_COMMON_ENHC_FRWRK — central enhancement dispatcher.
CLASS ys00_cl_common_enhc_frwrk DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.

    "! Resolve the configured flex-field names for an exit, return as a
    "! pre-populated table the caller fills with runtime values.
    METHODS get_flex_field_name
      IMPORTING iv_sap_exitname    TYPE ys00_dt_exitname
                iv_sap_method_name TYPE seocpdname
      CHANGING  ct_table           TYPE ys00_tt_extfield.

    "! Resolve the active class/method rows matching the populated
    "! flex-field values, sorted by SEQUENCE_NO.
    METHODS get_exits
      CHANGING  ct_table1 TYPE ys00_tt_extfield   " input: filter values
                ct_table2 TYPE ys00_tt_classn.    " output: class+method+seq

ENDCLASS.
```

### Calling pattern in a BAdI

```abap
METHOD if_some_badi~some_method.
  DATA: lt_filters TYPE ys00_tt_extfield,
        lt_calls   TYPE ys00_tt_classn,
        lo_handler TYPE REF TO object.

  " 1. Get configured filter field names for this exit
  ys00_cl_common_enhc_frwrk=>get_flex_field_name(
    EXPORTING iv_sap_exitname    = 'IF_SOME_BADI'
              iv_sap_method_name = 'SOME_METHOD'
    CHANGING  ct_table           = lt_filters ).

  " 2. Populate filter values from runtime context
  LOOP AT lt_filters ASSIGNING FIELD-SYMBOL(<f>).
    ASSIGN COMPONENT <f>-fieldtechname1 OF STRUCTURE is_context TO FIELD-SYMBOL(<v>).
    IF sy-subrc = 0. <f>-fieldconfigval1 = <v>. ENDIF.
    " ...repeat for fields 2..5
  ENDLOOP.

  " 3. Resolve active implementations
  ys00_cl_common_enhc_frwrk=>get_exits(
    CHANGING ct_table1 = lt_filters
             ct_table2 = lt_calls ).

  " 4. Call each in sequence
  LOOP AT lt_calls ASSIGNING FIELD-SYMBOL(<c>).
    CREATE OBJECT lo_handler TYPE (<c>-classname).
    CALL METHOD lo_handler->(<c>-methodname)
      EXPORTING is_context = is_context
      CHANGING  cs_result  = cs_result.
  ENDLOOP.
ENDMETHOD.
```

## When to use

- Multi-market / multi-org S/4 rollouts with shared SAP exits but divergent custom logic.
- Need to switch enhancement code on/off without transport (pre-prod testing, business-driven activation).
- Multiple WRICEFs hooking the same BAdI, owned by different teams.
- Sequence-sensitive enhancement chains.

## When NOT to use

- Single-tenant, single-team customizations — overhead is not justified.
- Standard SAP **already exposes a switching mechanism** (e.g. SFW switches, Output Management config) — use that first.
- Tight inner-loop code where dynamic `CALL METHOD` overhead matters.
- RAP-native scenarios — prefer behavior implementations and projection augmentations over BAdIs where possible.

## Related patterns

- *(future)* `patterns/badi-implementation-strategy`
- *(future)* `patterns/output-management-brfplus`

## Anti-patterns

- Letting the value table grow into a "soft transport" — anything in PRD that changes behaviour silently is an audit risk. Lock with workflow approval.
- Skipping `ACTIVE_FLAG` checks → orphan rows still execute.
- Not enforcing `SEQUENCE_NO` uniqueness per `EXITID` → non-deterministic execution order.
- Reusing the same `EXITID` across unrelated business contexts.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §6.3.1 in full.
- [[sources/opencode-abap-context-library]] — `architecture-deep.md` (enhancement framework pattern corroborated).
