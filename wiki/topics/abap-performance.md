---
title: "ABAP Performance"
type: topic
tags: [performance, abap, database, internal-tables]
sap_release: ["S/4HANA", "ECC", "BTP ABAP"]
status: draft
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[topics/abap-quality-gates]]"
  - "[[patterns/multi-site-repo-architecture]]"
created: 2026-05-11
updated: 2026-05-11
---

# ABAP Performance

> Cross-cutting topic — the rules of thumb that separate fast custom ABAP from the kind that takes the system down on month-end.

## Why this matters

Performance pitfalls in ABAP are repeatable and well-known. Most production incidents from custom code trace back to a small set of patterns: full-table scans, nested SELECTs, unbounded internal-table reads, FOR ALL ENTRIES on an empty driver. The Heinemann standard codifies these into mandatory rules; this page consolidates them.

## Core concepts

- [[concepts/clean-abap]] — readable code is also auditable code.
- *(future)* [[concepts/cds-views]] — push computation to the DB on HANA.
- *(future)* [[concepts/amdp]] — when CDS isn't expressive enough.

## Practical guidance

### Database access

- **No `SELECT *`** — list the fields you need.
- **Use indexed fields in WHERE** — check `SE11` for available indexes; consider secondary indexes only when justified.
- **Limit the result set** with `UP TO n ROWS` when only a sample is needed.
- **Limit traffic** with aggregations (`SUM`, `COUNT`, `GROUP BY`) instead of pulling rows and computing in ABAP.
- **Avoid nested `SELECT`** — use joins or `FOR ALL ENTRIES`.
- **`FOR ALL ENTRIES`**: always check the driver table is not empty first.

  ```abap
  IF lit_keys IS NOT INITIAL.
    SELECT … FROM … FOR ALL ENTRIES IN lit_keys
      WHERE matnr = lit_keys-matnr.
  ENDIF.
  ```

  An empty driver means *no WHERE restriction* → full table scan.

- **Archive aged data** — large legacy tables benefit from SARA / DART before tuning queries.

### Internal-table processing

- Pick the right kind:
  - `STANDARD` when sequential access dominates.
  - `SORTED` when you read by a known prefix of the key.
  - `HASHED` for unique-key point lookups.
- **`READ TABLE … BINARY SEARCH`** on STANDARD tables only after explicit `SORT`.
- **Avoid `LOOP … READ TABLE …`** — use a parallel cursor or change the table type.
- **Type explicitly** — implicit typing (e.g. `DATA lt LIKE itab`) hides intent and disables optimizer hints.

### Readability & modularization

- Short methods/FORMs — long routines hide quadratic loops.
- Reuse via classes/FMs; don't copy-paste SELECTs across reports.
- Multilingual support via text symbols, not concatenated strings.

### Coding hygiene

- Use **constants** (`CONSTANTS co_…`) instead of magic literals.
- Avoid **obsolete and non-released function modules** — check `SE37` release status.
- Don't modify standard SAP — enhance via [[patterns/table-driven-enhancement-framework]] or BAdIs.
- Class-based exceptions; no `MESSAGE 'foo' TYPE 'E'` from deep helpers.
- BAPIs over BDC; BDC only when no API exists.
- Document async interfaces explicitly.

## Patterns & anti-patterns

- [[patterns/table-driven-enhancement-framework]] — controlled custom hooks.
- *(future)* `patterns/parallel-cursor` — keep two iterators in step instead of nested loops.
- *(future)* Anti-pattern: `for-all-entries-without-check`.

## Tools

- [[entities/tools/atc]] — performance-relevant checks (SQL injection, expensive selects).
- *(future)* `entities/tools/sat` — Runtime Analysis (transaction `SAT`).
- *(future)* `entities/tools/st05` — Performance Trace.

## Related topics

- [[topics/abap-quality-gates]]

## EWM project specifics

The Heinemann EWM migration ([[sources/heinemann-ewm-coding-standards]]) adds these concrete rules on top of the generic guidance:

### SQL anti-pattern quick table

| Anti-pattern                                  | Fix                                                                | Severity |
| --------------------------------------------- | ------------------------------------------------------------------ | -------- |
| `SELECT *`                                    | List required fields                                               | High     |
| Nested `SELECT` (loop inside loop)            | Join, FAE, or sorted-table parallel cursor                         | High     |
| `FOR ALL ENTRIES` without `IS NOT INITIAL`    | Guard with empty-check                                             | Critical |
| `SELECT … INTO TABLE … ORDER BY` then re-sort | Pick one — sort on DB or on app                                    | Medium   |
| `SELECT SINGLE` without full PK               | Use `UP TO 1 ROWS` or rethink the key                              | Medium   |
| `LOOP AT itab … READ TABLE itab2`             | `SORTED` / `HASHED` table or parallel cursor                       | High     |
| Implicit client handling on cross-client data | Specify client explicitly                                          | Medium   |
| Missing `WHERE` on EWM doc tables (`/SCWM/*`) | EWM tables are huge — unrestricted reads kill the warehouse server | Critical |

### RF dialog SLA

- **Round-trip target: ≤ 2 seconds** end-to-end for any RF scanner interaction (operators standing at a putaway location won't wait longer; degrades pick rate).
- Includes DB time, app time, network. Measure with the project performance harness (see below).
- If exceeded, profile with ST05 + SAT; consider preloading reference data on RF session start.

### Performance measurement infrastructure

- **`YCL_EWM_PERF_MEASURE`** (shared) / **`ZCL_EWM_PERF_MEASURE`** (site-specific) — wrapper class for instrumented timing.
- Usage: bracket suspect logic with `start( 'label' )` / `stop( 'label' )`; results land in a custom logging table that feeds a Fiori monitoring tile.
- Site variant `ZCL_EWM_PERF_MEASURE` may extend `YCL_EWM_PERF_MEASURE` with site-specific thresholds (Alicante's network vs Erlenbach's differ).

### Other EWM-specific notes

- **`/SCWM/*` standard tables**: always restrict by warehouse number (`LGNUM`) — first index field, mandatory in `WHERE`.
- **Mass-update via BAPI**: prefer `/SCWM/API_*` bulk APIs over per-document calls; commit in chunks of 100–500.
- **Background processing**: long-running site-specific jobs in `ZALM_*` / `ZERL_*` packages, scheduled per site, never globally.

## Modern ABAP idioms with a perf angle

### Prefer `lines( itab )` over `DESCRIBE TABLE`

```abap
" Preferred — built-in, no extra DATA declaration
IF lines( lt_orders ) > 100.
  ...
ENDIF.

" Older
DATA lv_count TYPE i.
DESCRIBE TABLE lt_orders LINES lv_count.
IF lv_count > 100. ... ENDIF.
```

### Table-expression `DEFAULT` instead of `READ TABLE` + IF

```abap
" Preferred — single expression, no SY-SUBRC dance
DATA(ls_order) = VALUE #( lt_orders[ order_id = lv_id ]
                          DEFAULT ls_empty ).

" Older
READ TABLE lt_orders INTO ls_order WITH KEY order_id = lv_id.
IF sy-subrc <> 0.
  ls_order = ls_empty.
ENDIF.
```

Both compile to equivalent bytecode; the modern form is shorter, type-checked, and free of `SY-SUBRC` plumbing.

> [!info] Available since
> `lines( )` built-in: NetWeaver **7.40**. Table expressions + `DEFAULT`: NetWeaver **7.40 SP08**.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 3 in full.
- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §"Performance rules" and §"RF dialog SLA".
- [[sources/opencode-abap-context-library]] — `modern-abap-patterns.md` (`lines( )`, table-expression `DEFAULT`), `quick-ref.md` (modern idiom table).
