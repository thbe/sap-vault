---
title: "ATC (ABAP Test Cockpit)"
type: entity
entity_kind: tool
sap_object_name: "ATC"
tags: [tool, quality, atc, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP"]
status: stub
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[topics/abap-quality-gates]]"
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/clean-abap]]"
created: 2026-05-11
updated: 2026-05-11
---

# ATC (ABAP Test Cockpit)

> SAP's central tool for static code checks on ABAP. Runs Code Inspector check variants on objects, packages, or transport requests, with results managed via worklists and exemptions.

## Status

**Stub.** Heinemann standard mandates ATC use but doesn't document the tool itself; expand from SAP Help.

## Key points (from existing source)

- **Mandatory** before transport release ([[topics/abap-quality-gates]]).
- Runs project-specific SCI variants — e.g. `Y_NAMING_CONVENTIONS` enforces the prefix conventions in [[concepts/abap-naming-conventions]].
- Complements (not replaces) the Extended Program Check (`SLIN`).

## Open topics

- ATC vs SCI vs Code Inspector terminology.
- Exemption workflow and approval governance.
- ATC as remote service (central system checking child systems).
- Pseudo-comments to suppress findings (and when not to).
- Integration with abapGit + CI pipelines.

## Related transactions / classes

- TCODE `ATC` — main entry.
- TCODE `SCI` / `SCII` — Code Inspector standalone.
- TCODE `SLIN` — Extended Program Check.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §2.1.5–2.1.6.
