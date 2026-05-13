---
title: "SAT (ABAP Runtime Trace / Tracer)"
type: entity
entity_kind: tool
sap_object_name: "SAT"
tags: [tool, performance, trace, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA"]
status: stub
sources:
  - "[[wiki/sources/heinemann-ewm-coding-standards]]"
related:
  - "[[wiki/topics/abap-performance]]"
  - "[[wiki/entities/tools/st05]]"
created: 2026-05-13
updated: 2026-05-13
---

# SAT (ABAP Runtime Trace)

> Modern successor to SE30. Records ABAP statement-level execution (call hierarchy, method/FM/perform timings, hit counts). Primary tool for finding ABAP-side hotspots; pair with [[wiki/entities/tools/st05|ST05]] for SQL-side hotspots.

## Status

**Stub — pointer page.** Substantive performance tuning recipe lives on [[wiki/topics/abap-performance]]. This entry exists so SAT resolves as a distinct entity.

To expand beyond stub: trace variants (Aggregated vs Per-call), filter by package/program/user, comparing two traces, gross vs net time, exporting traces, common pitfalls (trace overhead, sampling vs full).

## Related transactions

- TCODE `SAT` — runtime trace.
- TCODE `SE30` — legacy predecessor (still callable, redirects to SAT on modern systems).
- TCODE `ST05` — SQL/RFC/Buffer trace ([[wiki/entities/tools/st05]]).

## Sources

- [[wiki/sources/heinemann-ewm-coding-standards]]
