---
title: "OData Service"
type: concept
tags: [odata, gateway, rap, fiori, segw, service]
sap_release: ["S/4HANA", "ECC + Gateway hub", "BTP ABAP"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-fiori-context-library]]"
related:
  - "[[concepts/restful-application-programming-model]]"
  - "[[concepts/sap-fiori-overview]]"
  - "[[entities/tools/segw]]"
created: 2026-05-11
updated: 2026-05-13
---

# OData Service

> HTTP-based REST-ish protocol used to expose ABAP business data to Fiori/UI5 (and other) front ends. Two creation paths on SAP: legacy SEGW (OData V2) and modern RAP service bindings (OData V2 or V4).

## Two paths

| Path                       | Protocol     | Tool / Artifact                          | When                                              |
| -------------------------- | ------------ | ---------------------------------------- | ------------------------------------------------- |
| **SEGW (legacy)**          | OData V2     | TCODE `SEGW`, `*_DPC_EXT` class          | Maintenance of existing services; ECC; non-RAP   |
| **RAP service binding**    | OData V2/V4  | CDS view + BDL + service definition      | New S/4 + BTP ABAP development (preferred)        |

## SEGW workflow (legacy)

### Development steps

1. **Create project** in SEGW following naming standards.
2. **Import DDIC structures** to create Entity Types and Entity Sets (reuse, don't duplicate).
3. **Generate runtime objects** — produces `MPC` (Model Provider) and `DPC` (Data Provider) classes plus their `_EXT` extension classes.
4. **Implement methods** in the `*_DPC_EXT` extension class — never edit the generated parent.
5. **Register & activate** the service in `/IWFND/MAINT_SERVICE` on the FES.
6. **Test** with `/IWFND/GW_CLIENT` before UI integration.

### Methods to redefine in `*_DPC_EXT`

| Method                | Purpose                          | HTTP                       |
| --------------------- | -------------------------------- | -------------------------- |
| `<Entity>_GET_ENTITY`     | Read single                      | `GET /Entity(key)`         |
| `<Entity>_GET_ENTITYSET`  | Read list (filter/page/sort)     | `GET /EntitySet?$filter=…` |
| `<Entity>_CREATE_ENTITY`  | Create                           | `POST /EntitySet`          |
| `<Entity>_UPDATE_ENTITY`  | Update                           | `PUT/PATCH /Entity(key)`   |
| `<Entity>_DELETE_ENTITY`  | Delete                           | `DELETE /Entity(key)`      |

### Mandatory query-option handling

- **`$filter`** — always implement in `GET_ENTITYSET`. Parse `IT_FILTER_SELECT_OPTIONS` into a `WHERE` clause.
- **`$top` / `$skip`** — always implement for paging on large datasets. Use `IS_PAGING-TOP` / `IS_PAGING-SKIP`.
- **`$orderby`** — implement when client-side sorting is insufficient.
- **`$expand`** — for navigation properties; implement `*_GET_EXPANDED_ENTITY(SET)` if performance requires it.
- **`$select`** — usually handled by Gateway; only implement if column projection materially reduces DB cost.

### Field naming — policy

> [!info] Resolved policy — split by audience
> Both conventions in our sources are valid; the policy is to **split by service audience**:
>
> - **Internal services** (consumed only by our own Fiori apps / internal back-ends) → keep ABAP upper-case names (`MATNR`, `LIFNR`) so DDIC structures map automatically and `DPC_EXT` stays glue-only. Source: [[sources/opencode-fiori-context-library]] (`fiori-guidelines.md` §3.2).
> - **Partner-facing or platform services** (external consumers, published catalogues, cross-org integration) → use **English semantic UpperCamelCase** (`MaterialNumber`, `SupplierId`, `FirstName`) to decouple the OData contract from DDIC and present a clean external API. Source: [[sources/sap-development-standard-approach-abap-fiori-v1]] §5.4.
>
> Decide audience at service-design time and document it on the service registration. Don't mix conventions inside one service.

### Other SEGW rules

- **Reuse DDIC structures**, don't include them — supports complex-type mapping cleanly.
- **Optional field removal** at the GW layer is safe.
- **Select key fields explicitly** in `GET_ENTITYSET`.
- **Build associations one direction**; create both navigation properties on each entity.
- **Delegate business calls** to the back-end logic layer (BAPI / class method) — `DPC_EXT` is glue code only.

### Testing — `/IWFND/GW_CLIENT`

Verify before UI integration:

| Status | Meaning                          |
| ------ | -------------------------------- |
| `200`  | OK — read succeeded              |
| `201`  | Created                          |
| `204`  | No Content (delete / void update)|
| `400`  | Bad request (filter parse, etc.) |
| `404`  | Not found                        |
| `500`  | Server error                     |

> [!warning] Legacy reference
> SEGW is in maintenance mode. New OData services on S/4 should be built with **RAP service bindings**; see [[concepts/restful-application-programming-model]]. SEGW remains for maintenance of existing services and where RAP isn't viable.

## RAP service binding (modern)

CDS view + Behavior Definition + Service Definition + Service Binding produce an OData service without DPC/MPC plumbing. See [[concepts/restful-application-programming-model]], [[concepts/cds-view-entities]], [[concepts/behavior-definition]] for the mechanics.

## Documented future work

The following are out of scope for this page and tracked as future ingest targets — page is stable as-is, these will land in dedicated pages or expansions when sources are ingested:

- Detailed RAP service-binding workflow (V2 vs V4) — *(future)* extension to [[concepts/restful-application-programming-model]].
- OData V4 trade-offs vs V2 (capabilities, draft handling, batch) — *(future)* dedicated topic page.
- Authentication / CORS / CSRF on BTP vs on-prem — *(future)* dedicated security topic.
- `$batch` request handling in DPC_EXT — *(future)* expansion to the SEGW workflow above.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §5.4, §5.14.
- [[sources/opencode-fiori-context-library]] — `fiori-guidelines.md` §3 (OData & Gateway development).
