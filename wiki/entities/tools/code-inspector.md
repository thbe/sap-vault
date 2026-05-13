---
title: "Code Inspector (SCI)"
type: entity
entity_kind: tool
sap_object_name: "SCI"
tags: [tool, quality, sci, atc, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP (limited)"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[wiki/sources/heinemann-ewm-coding-standards]]"
related:
  - "[[wiki/entities/tools/atc]]"
  - "[[wiki/topics/abap-quality-gates]]"
  - "[[wiki/concepts/abap-naming-conventions]]"
created: 2026-05-13
updated: 2026-05-13
---

# Code Inspector (SCI)

> Static code analysis engine underneath ATC. Defines reusable **check variants** (e.g. `Y_NAMING_CONVENTIONS`) that ATC executes against objects, packages, or transports.

## Status

**Stub — pointer page.** Substantive variant configuration and gate semantics live on [[wiki/entities/tools/atc|ATC]] and [[wiki/topics/abap-quality-gates]]. This entry exists so SCI/Code Inspector resolves as a distinct entity.

To expand beyond stub: variant authoring (TCODE `SCI`), check categories (security, performance, syntax, naming), priority levels, message exemption flow, project-level vs global variants, ATC ↔ SCI relationship, BTP ABAP limitations.

## Related transactions

- TCODE `SCI` — Code Inspector (variant maintenance, ad-hoc runs).
- TCODE `ATC` — orchestrator that consumes SCI variants ([[wiki/entities/tools/atc]]).

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]
- [[wiki/sources/heinemann-ewm-coding-standards]]
