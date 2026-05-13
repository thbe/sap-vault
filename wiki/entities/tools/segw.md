---
title: "SEGW (SAP Gateway Service Builder)"
type: entity
entity_kind: tool
sap_object_name: "SEGW"
tags: [tool, gateway, odata, legacy]
sap_release: ["NetWeaver 7.40+", "S/4HANA (still available, not preferred for new dev)"]
status: stub
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-fiori-context-library]]"
related:
  - "[[concepts/odata-service]]"
  - "[[concepts/restful-application-programming-model]]"
created: 2026-05-11
updated: 2026-05-13
---

# SEGW (SAP Gateway Service Builder)

> Legacy ABAP transaction for designing OData V2 services: define entity types, entity sets, associations, and generate `MPC`/`DPC` runtime classes. Method logic implemented in `*_DPC_EXT` (extension class).

## Status

**Stub — pointer page.** This entry exists so SEGW resolves as an entity from other pages. The substantive workflow content lives on [[concepts/odata-service]] (DPC_EXT methods, `$filter`/`$top`/`$skip` handling, testing). Marked legacy because **RAP service bindings are the preferred model on modern S/4HANA / BTP ABAP**.

To expand beyond stub: MPC vs DPC class structure, generated artefact lifecycle (regeneration vs activation), `*_DPC_EXT` redefinition mechanics, common transactions cross-reference.

## Key points (from existing source)

- **Workflow** (§5.14):
  1. Create project in SEGW following naming standards.
  2. Register endpoint in Gateway.
  3. Generate runtime artefacts.
  4. Define DDIC structures (reuse, don't include).
  5. Import structures as Entity/EntitySet; adjust field names; select keys.
  6. Build associations one direction; create both navigation properties.
  7. Generate service (warning: takes the existing service down briefly).
  8. Implement methods in `DPC_EXT`: read keys/filters, handle paging/sorting, call business layer, return object or exception.
  9. Test via `/IWFND/GW_CLIENT`.

> [!warning] Legacy
> SEGW is in maintenance mode. New OData services on S/4 should be built with **RAP service bindings** ([[concepts/restful-application-programming-model]]). SEGW remains for maintenance of existing services and for cases where RAP is not viable.

## Open topics

- Migration path SEGW → RAP.
- Mixed landscapes (some services SEGW, some RAP) — coexistence guidance.

## Related transactions

- TCODE `SEGW` — Service Builder.
- TCODE `/IWFND/MAINT_SERVICE` — Activate & maintain on the FES.
- TCODE `/IWFND/GW_CLIENT` — Test client.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §5.14.
- [[sources/opencode-fiori-context-library]] — `fiori-guidelines.md` §3 (DPC_EXT method list, `$filter`/`$top`/`$skip`, `/IWFND/GW_CLIENT` testing, HTTP status codes).
