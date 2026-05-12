---
title: "RESTful Application Programming Model (RAP)"
type: concept
tags: [rap, abap, s4hana, btp-abap, cds, odata]
sap_release: ["S/4HANA 1909+", "BTP ABAP"]
status: draft
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
updated: 2026-05-11
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
