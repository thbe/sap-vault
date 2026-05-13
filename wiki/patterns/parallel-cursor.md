---
title: "Parallel Cursor"
type: pattern
tags: [pattern, performance, abap, internal-table]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP"]
status: stub
sources:
  - "[[wiki/sources/heinemann-ewm-coding-standards]]"
related:
  - "[[wiki/topics/abap-performance]]"
created: 2026-05-13
updated: 2026-05-13
---

# Parallel Cursor

> Replacement for the classic O(n×m) `LOOP AT itab1 ... LOOP AT itab2 WHERE ...` anti-pattern. Uses `READ TABLE ... BINARY SEARCH` (sorted) or sorted/hashed table types to position a cursor in the inner table once per outer row, then walks forward only as long as the join key matches.

## Status

**Stub.** Anti-pattern is documented in [[wiki/topics/abap-performance]] (SQL/internal-table anti-pattern table). This page exists so the canonical replacement pattern resolves as a named entity.

To expand beyond stub:
- Canonical code template (sorted table + `READ ... BINARY SEARCH ... TRANSPORTING NO FIELDS` + `LOOP FROM sy-tabix WHERE`).
- Sorted-table variant (declarative cursor via `SORTED` table type — cleaner; preferred in Clean ABAP).
- Hashed-table variant (when only point lookup needed; not a parallel cursor proper).
- Complexity proof (O((n+m) log m) → O(n+m) with sorted/hashed types).
- Modern alternative: `FOR ... IN GROUP` / `LOOP AT GROUP` (release-gated; see [[wiki/snippets/abap-modern-syntax-cheatsheet]]).
- Worked refactor: nested-loop → parallel cursor → grouping expression.

## Sources

- [[wiki/sources/heinemann-ewm-coding-standards]]
