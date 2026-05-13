---
title: "SLIN (Extended Program Check)"
type: entity
entity_kind: tool
sap_object_name: "SLIN"
tags: [tool, quality, slin, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[wiki/entities/tools/atc]]"
  - "[[wiki/entities/tools/code-inspector]]"
  - "[[wiki/topics/abap-quality-gates]]"
created: 2026-05-13
updated: 2026-05-13
---

# SLIN (Extended Program Check)

> Deep cross-program syntax/semantics check beyond the standard ABAP syntax check. Catches issues a normal compile misses (unused variables, problematic interface mismatches, dynamic call risks).

## Status

**Stub — pointer page.** Mentioned in quality-gate flows alongside [[wiki/entities/tools/atc|ATC]] and [[wiki/entities/tools/code-inspector|SCI]]. Substantive gate semantics live on [[wiki/topics/abap-quality-gates]].

To expand beyond stub: SLIN check categories, when SLIN catches what ATC/SCI miss, integration with transport release, performance considerations on large programs, BTP ABAP availability.

## Related transactions

- TCODE `SLIN` — Extended Program Check (ad-hoc).
- Menu path from SE38: *Program → Check → Extended Program Check*.

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]
