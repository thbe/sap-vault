---
title: "ABAP Unit"
type: entity
entity_kind: tool
sap_object_name: "ABAP Unit"
tags: [tool, testing, abap-unit, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[wiki/sources/heinemann-ewm-coding-standards]]"
  - "[[wiki/sources/opencode-abap-context-library]]"
related:
  - "[[wiki/topics/abap-unit-testing]]"
  - "[[wiki/patterns/test-doubles-and-dependency-injection]]"
  - "[[wiki/entities/tools/atc]]"
created: 2026-05-13
updated: 2026-05-13
---

# ABAP Unit

> SAP's built-in xUnit-style test framework. Test classes marked `FOR TESTING` with `RISK LEVEL` and `DURATION`; executed via ADT, SE80, or transport release gates.

## Status

**Stub — pointer page.** Substantive workflow (skeleton, Given-When-Then, fixtures, CDS test environment) lives on [[wiki/topics/abap-unit-testing]]; doubles and DI live on [[wiki/patterns/test-doubles-and-dependency-injection]]. This entity exists so "ABAP Unit" resolves as a distinct tool.

To expand beyond stub: test runner mechanics, coverage measurement (`SCOV`), test-relations (`tested by`), code-coverage thresholds, parallel execution, gateway to ATC, BTP ABAP test classes.

## Related transactions / shortcuts

- ADT *Run As → ABAP Unit Test* (`Ctrl+Shift+F10`).
- TCODE `SCOV` — coverage analyzer.
- TCODE `SAUNIT_CLIENT_SETUP` — admin.

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]
- [[wiki/sources/heinemann-ewm-coding-standards]]
- [[wiki/sources/opencode-abap-context-library]]
