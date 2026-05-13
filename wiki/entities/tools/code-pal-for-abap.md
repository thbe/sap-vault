---
title: "code pal for ABAP"
type: entity
entity_kind: tool
sap_object_name: "code-pal-for-abap"
tags: [tool, atc, sci, clean-abap, quality, open-source]
sap_release: ["S/4HANA", "BTP ABAP", "NW 7.50+"]
status: stub
sources:
  - "[[sources/clean-abap-styleguide]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[entities/tools/atc]]"
  - "[[entities/tools/code-inspector]]"
  - "[[entities/tools/abaplint]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-13
updated: 2026-05-13
---

# code pal for ABAP

> SAP-published open-source Code Inspector check collection that automates a comprehensive subset of [[concepts/clean-abap|Clean ABAP]] rules. Plugs into [[entities/tools/code-inspector|SCI]] and [[entities/tools/atc|ATC]].

## Status

Stub — pointer page. Substantive Clean ABAP enforcement guidance lives on [[concepts/clean-abap]] and [[topics/abap-quality-gates]]. To expand beyond stub: install + variant authoring walkthrough, check-by-check Clean ABAP coverage matrix, false-positive triage patterns, comparison with abapOpenChecks and built-in SCI checks.

## Repository

- **GitHub**: <https://github.com/SAP/code-pal-for-abap>
- **License**: Apache 2.0
- **Maintainer**: SAP

## How it integrates

1. Install via abapGit into your ABAP system.
2. Activate in a custom SCI variant (e.g. `Y_CLEAN_ABAP`).
3. Wire the variant into your ATC default or per-package run-time variant.
4. Pre-transport ATC catches Clean ABAP violations alongside built-in SAP checks.

## Related

- [[concepts/clean-abap]] — the rules code pal automates.
- [[entities/tools/atc]] — the engine code pal plugs into.
- [[entities/tools/code-inspector]] — variant authoring container.
- [[entities/tools/abaplint]] — sister open-source checker, runs *outside* SAP on serialized abapGit code.
- [[entities/tools/abapgit]] — install vehicle.
- [[topics/abap-quality-gates]] — pipeline the variant feeds.

## Sources

- [[sources/clean-abap-styleguide]] — `#how-to-check-automatically` lists code pal as the canonical Clean ABAP check tool.
