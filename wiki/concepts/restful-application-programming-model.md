---
title: "RESTful Application Programming Model (RAP)"
type: concept
tags: [rap, abap, s4hana, btp-abap, cds, odata]
sap_release: ["S/4HANA 1909+", "BTP ABAP"]
status: stable
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/cds-view-entities]]"
  - "[[concepts/behavior-definition]]"
  - "[[concepts/eml]]"
  - "[[concepts/mvc-in-abap]]"
  - "[[concepts/odata-service]]"
  - "[[concepts/sap-fiori-overview]]"
  - "[[topics/abap-unit-testing]]"
created: 2026-05-11
updated: 2026-05-13
---

# RESTful Application Programming Model (RAP)

> SAP's modern programming model for building OData-based business services on S/4HANA and BTP ABAP. CDS view entities form the data model, behavior definitions describe the controller, behavior implementations provide the logic in ABAP, and service bindings expose OData V4 endpoints consumed by Fiori Elements or any OData client.

## Overview

RAP replaces the legacy SEGW/Gateway stack ([[concepts/odata-service]] / [[entities/tools/segw]]) for new development on S/4HANA and is the **only** supported model on ABAP Cloud (BTP ABAP / Steampunk). It enforces a strict layered architecture and a declarative-first style: most CRUDQ behavior is generated from annotations and BDL syntax; ABAP code is only written for non-trivial logic (actions, determinations, validations, custom queries).

## Key ideas

- **Layered stack** (top-down):
  1. **Data model** — CDS view entities with annotations.
  2. **Behavior definition** (BDL, `.bdef`) — declares which fields are read/write, what actions/determinations/validations exist, locking strategy.
  3. **Behavior implementation** — ABAP class implementing `lhc_*` (local handler classes) generated from the BDL.
  4. **Projection layer** — CDS projection view + projection BDL exposing only what the service binding needs.
  5. **Service definition** — declares which projections form the service.
  6. **Service binding** — exposes the service as OData V2 / V4 UI / V4 API.
- **Implementation types**:
  - **Managed** — RAP runtime handles persistence (DB writes, locking, key generation). Use for greenfield entities backed by Z tables.
  - **Unmanaged** — Developer implements persistence (CREATE/UPDATE/DELETE handlers). Use to wrap existing BAPIs / legacy logic.
  - **Managed with additional save** — Managed persistence + custom save augmentation.
- **Draft enablement**: Edit-on-the-fly via temporary draft instances; enables Fiori draft pattern with one annotation in BDL.
- **Strict syntax modes**: `strict ( 2 )` is the current mandatory level — catches misuse early and is required for new development.
- **EML (Entity Manipulation Language)**: ABAP-internal API to read/modify RAP entities programmatically — see [[concepts/eml]].

## How it works

```
┌──────────────────────────────────────────────────────────────┐
│  Service Binding   (OData V4 UI)         ──→  Fiori Elements │
├──────────────────────────────────────────────────────────────┤
│  Service Definition                                          │
├──────────────────────────────────────────────────────────────┤
│  Projection BDL + Projection CDS  (exposed shape)            │
├──────────────────────────────────────────────────────────────┤
│  Behavior Definition (BDL)  +  Behavior Implementation (ABAP)│
├──────────────────────────────────────────────────────────────┤
│  Root + child CDS view entities  (semantic data model)       │
├──────────────────────────────────────────────────────────────┤
│  Z database tables  /  legacy BAPI                           │
└──────────────────────────────────────────────────────────────┘
```

Every layer is a transportable repository object (CDS = DDLS, BDL = BDEF, projection = DDLX/BDEF, service definition = SRVD, service binding = SRVB, behavior implementation = CLAS).

## Service-binding workflow

End-to-end recipe taking a CDS root entity to a callable OData endpoint. Order matters — each step depends on the previous being activated.

### 1. Build the data model
1. Root CDS view entity (`define root view entity ...`) over the persistent table or an existing CDS composition tree.
2. Child CDS view entities if the root is a composition.
3. Activate. ADT marks them with the *RAP root* / *RAP child* badge once a BDEF references them.

### 2. Define behavior
1. Create the **Behavior Definition** (`yotc_i_order.bdef`) with header `managed | unmanaged | abstract implementation in class zbp_<root> unique;` and `strict ( 2 );`.
2. Declare CRUDQ operations, fields, locking, authorisation, ETag, determinations, validations, actions.
3. Activate the BDEF — ADT auto-generates the **behavior implementation class skeleton** (`zbp_<root>`) and its local handler classes (`lhc_<entity>`) on first activation.
4. Implement the `lhc_*` handler methods for any non-managed concerns (validations, determinations, actions, custom save for `managed with additional save`, full CUD for `unmanaged`).

### 3. Add the projection layer
1. Create a **CDS Projection View** (`yotc_c_order.ddls`, `define root view entity ... as projection on YOTC_I_Order { ... }`) — exposes only the fields and associations the service consumer should see, applies UI annotations (`@UI.lineItem`, `@UI.identification`, etc.) here rather than on the root.
2. Create a **Projection Behavior Definition** (`yotc_c_order.bdef`, header `projection;`) — selects which BDEF operations to expose, optionally adds `use draft;` to enable draft handling.
3. Activate both.

### 4. Define the service
1. Create a **Service Definition** (`yotc_o4_order.srvd`):
   ```cds
   @EndUserText.label: 'Sales Order Service'
   define service YOTC_O4_Order {
     expose YOTC_C_Order as Order;
   }
   ```
2. Activate. The service is now defined but has no protocol binding yet.

### 5. Bind the service
1. Create a **Service Binding** (`yotc_o4_order_v4ui.srvb`) referencing the service definition. ADT prompts for:
   - **Binding Type**: `OData V4 - UI` (Fiori Elements V4) | `OData V4 - API` (programmatic / partner) | `OData V2 - UI` (Fiori Elements V2 / RAP-V2 fallback) | `InA - UI Service` (analytical).
   - **Category**: usually inferred from binding type.
2. Activate. Activation triggers metadata generation; errors here usually mean missing UI annotations or mismatched cardinality on the projection.
3. Click **Publish** (BTP ABAP) / *Local Service Endpoint* opens automatically (on-prem) — the binding now has a runnable URL.

### 6. Test the binding
- ADT *Service Binding Editor* shows the metadata document, entity sets, and a *Preview* button:
  - **OData V4 UI** → opens Fiori Elements preview with auto-generated List Report / Object Page.
  - **OData V4/V2 API** → opens browser-based service explorer (`$metadata`, GET on entity sets, `$batch` form).
- For programmatic clients use the URL shown in the binding header (`/sap/opu/odata4/sap/<srvname>/srvd_a2x/sap/<srvname>/0001/`).
- For BTP ABAP, additionally publish via a **Communication Scenario** (`SAP Communication Management` app) and create a **Communication Arrangement** so external clients can authenticate.

### 7. Wire up to consumption
- **Fiori Elements V4 app** → reference the published binding URL in the manifest's `dataSources` and instantiate a List Report / Object Page floorplan; UI annotations on the projection drive the layout (no UI5 controller code for the standard flows).
- **Custom UI5 app** → `sap.ui.model.odata.v4.ODataModel` pointing at the binding URL.
- **Server-to-server** → standard OData V4 client (e.g., SAP Cloud SDK, plain HTTP).

### Activation dependency graph

```
DB table / CDS source
       ↓
Root + child CDS view entities  (DDLS)
       ↓
Behavior Definition  (BDEF)              ──→  Behavior Implementation  (CLAS, lhc_*)
       ↓
Projection CDS view  (DDLS)
       ↓
Projection BDL  (BDEF, "projection;")
       ↓
Service Definition  (SRVD)
       ↓
Service Binding  (SRVB)  ──→  OData $metadata  ──→  consumer
```

A change to a lower layer requires re-activation upward. ADT's *Activate Inactive Objects* dialog usually orders this correctly; manual ordering matters when activating across packages.

> [!tip] Common activation errors
> - *"Behavior implementation class does not exist"* — activate the BDEF once to auto-generate the skeleton, then re-activate.
> - *"Field X is exposed in projection but not in root behavior"* — add `field ( ... ) X;` to the root BDEF.
> - *"Annotation @UI.lineItem requires position"* — UI annotations belong on the projection view, not the root.
> - *"Service binding cannot be published"* on BTP ABAP — check the linked communication scenario release status.

## Code example — minimal managed scenario

```cds
// Root CDS view entity — yotc_i_order
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Sales Order Root'
define root view entity YOTC_I_Order
  as select from yotc_a_order
{
  key order_id              as OrderId,
      customer_id           as CustomerId,
      total_amount          as TotalAmount,
      @Semantics.amount.currencyCode: 'Currency'
      currency              as Currency,
      created_at            as CreatedAt,
      created_by            as CreatedBy
}
```

```cds
// Behavior definition — yotc_i_order.bdef
managed implementation in class zbp_yotc_i_order unique;
strict ( 2 );

define behavior for YOTC_I_Order alias Order
persistent table yotc_a_order
lock master
authorization master ( instance )
etag master CreatedAt
{
  field ( numbering : managed, readonly ) OrderId;
  field ( mandatory ) CustomerId, TotalAmount, Currency;

  create;
  update;
  delete;

  determination calculateTotal on save { create; update; }
  validation validateCustomer  on save { create; field CustomerId; }
}
```

See [[concepts/behavior-definition]] for the BDL details and [[concepts/cds-view-entities]] for the data-model side.

## Release & version notes

> [!info] Available since
> RAP managed scenario: S/4HANA **1909**. Unmanaged + draft: **2020**. `strict ( 2 )`: **2208 / 2022**. EML in production-quality form: **2022**. Some BDL syntax (e.g. `numbering : managed`) requires 2022+.

> [!warning] ABAP Cloud
> On BTP ABAP / ABAP Cloud, RAP is the only supported programming model. Classic Gateway, function modules with screens, and direct DB modifying statements outside a managed save sequence are blocked.

## Related

- [[concepts/cds-view-entities]] — the data-model layer.
- [[concepts/behavior-definition]] — BDL syntax and managed/unmanaged choice.
- [[concepts/eml]] — programmatic access to RAP entities.
- [[concepts/mvc-in-abap]] — RAP is MVC made explicit at the language level.
- [[topics/abap-unit-testing]] — `CDS_TEST_ENVIRONMENT` and EML-friendly test doubles.

## Anti-patterns / pitfalls

- Writing directly to the persistence table in a managed scenario — breaks locking, ETag, draft.
- Bypassing the projection layer to expose root entities directly — leaks internal structure.
- Skipping `strict ( 2 )` "to silence warnings" — defers real errors to runtime.
- Embedding heavy business logic in determinations that run on every save — move long-running flows to async actions.
- Mixing managed and unmanaged behavior on the same compositional tree — pick one per root.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-rap-reference.md`, `architecture-deep.md`, `modern-abap-patterns.md`.
- *(future)* SAP RAP development guide — to be ingested.
