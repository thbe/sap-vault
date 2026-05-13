---
title: "BRF+ Application Exit"
type: pattern
tags: [pattern, brfplus, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[wiki/patterns/brf-plus-ui-guard]]"
created: 2026-05-13
updated: 2026-05-13
---

# BRF+ Application Exit

> Subclass of `CL_FDT_APPLICATION_SETTINGS` (or release-equivalent superclass) registered on a BRF+ application to inject custom UI behavior — including suppressing actions, restricting object types, and gating publish in productive systems.

## Status

**Stub.** Referenced as the implementation prerequisite for [[wiki/patterns/brf-plus-ui-guard]] (production action stripping). Method signatures vary by release; expansion requires a current BRF+ Workbench reference or system-side inspection.

To expand beyond stub:
- Exact superclass per release (`CL_FDT_APPLICATION_SETTINGS` vs newer alternatives).
- Method catalogue with redefinition contracts (`GET_ALLOWED_OBJECT_TYPES`, `GET_AVAILABLE_ACTIONS`, etc.).
- Registration flow (which BRF+ application property points to the exit class).
- Transport implications (exit class vs BRF+ object).
- Worked example tied to [[wiki/patterns/brf-plus-ui-guard]] PRD-mode action filter.

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]
