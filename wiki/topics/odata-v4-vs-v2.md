---
title: "OData V4 vs V2"
type: topic
tags: [odata, v2, v4, rap, fiori]
sap_release: ["S/4HANA", "BTP ABAP"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[wiki/sources/opencode-fiori-context-library]]"
related:
  - "[[wiki/concepts/odata-service]]"
  - "[[wiki/concepts/restful-application-programming-model]]"
  - "[[wiki/entities/tools/segw]]"
created: 2026-05-13
updated: 2026-05-13
---

# OData V4 vs V2

> Cheat sheet for the version split that drives nearly every modern SAP service decision: **V2 = SEGW + Fiori Elements legacy + classic UI5**, **V4 = RAP service binding + Fiori Elements V4 + modern UI5/SAPUI5 metadata**.

## Status

**Stub.** Listed under "Documented future work" on [[wiki/concepts/odata-service]]. Filled enough to capture the high-level decision; detailed specification differences pending a dedicated source ingestion.

## At-a-glance comparison

| Aspect              | OData V2                                  | OData V4                                                |
| ------------------- | ----------------------------------------- | ------------------------------------------------------- |
| Year / standard     | 2010, SAP-extended                        | 2014, OASIS standard                                    |
| ABAP build path     | SEGW + `*_DPC_EXT`                        | RAP service binding (CDS + BDL)                         |
| Metadata format     | EDMX 1.0 (verbose XML)                    | EDMX 4.0 (slimmer; capabilities annotations standard)   |
| JSON shape          | `d.results` wrapper                       | Direct `value` array (cleaner)                          |
| Batch               | `$batch` multipart MIME                   | `$batch` JSON payload (also supported)                  |
| Draft-enabled       | Custom (sticky session)                   | First-class via RAP draft                               |
| Actions / Functions | Function imports                          | Bound/unbound actions and functions (richer semantics)  |
| Filter expressions  | Limited                                   | Full grammar (`any`/`all`, lambdas, arithmetic)         |
| Capability metadata | Custom annotations                        | Standardised vocabulary (`Capabilities.*`)              |

## To expand beyond stub

- Full annotation vocabulary differences (UI, Capabilities, Common, Measures).
- Migration patterns (V2 SEGW → V4 RAP; coexistence in one app).
- Fiori Elements V2 vs V4 floorplan deltas.
- Performance differences (`$expand` semantics, `$apply` aggregations).
- Tooling: `/IWFND/GW_CLIENT` (V2 only) vs ADT service-binding test UI (V4-friendly).
- Authentication / CSRF / CORS deltas → cross-link to forthcoming [[wiki/topics/odata-security]].

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]
- [[wiki/sources/opencode-fiori-context-library]]
