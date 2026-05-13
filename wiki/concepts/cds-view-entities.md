---
title: "CDS View Entities"
type: concept
tags: [cds, rap, data-model, abap, s4hana, btp-abap]
sap_release: ["S/4HANA 1909+", "BTP ABAP", "NW 7.55+ (view entities)"]
status: stable
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/restful-application-programming-model]]"
  - "[[concepts/behavior-definition]]"
  - "[[concepts/eml]]"
  - "[[concepts/odata-service]]"
created: 2026-05-11
updated: 2026-05-13
---

# CDS View Entities

> The modern CDS modelling artefact — `DEFINE VIEW ENTITY` — that replaces the legacy `DEFINE VIEW` syntax and underpins RAP business objects on S/4HANA and BTP ABAP.

## Overview

A CDS view entity is the data-model layer of RAP. It's defined in DDL source, lives outside the ABAP Dictionary (no underlying DDIC SQL view), supports richer expressions than the old `DEFINE VIEW`, and is the only form going forward.

> [!info] Available since
> View entities require **NW 7.55 / S/4HANA 2020** and later. On older systems use `DEFINE VIEW` — but on S/4HANA 2023+ new development should always use `DEFINE VIEW ENTITY`.

## Key ideas

- **`DEFINE VIEW ENTITY`** — modern form. No paired SQL view, no client field in definition (handled automatically).
- **Root vs child** — `@ObjectModel.compositionRoot: true` marks the root; children link via `composition [*] of`.
- **Associations** are read-time joins exposed as path expressions (`_BusinessPartner.Name`); **compositions** are tighter parent/child links that drive RAP behavior cascades (delete child when parent deleted, draft propagation, etc.).
- **Casts and expressions**: `cast(…)`, `case … when …`, arithmetic, string operations all available; richer than `DEFINE VIEW`.
- **Semantics annotations** (`@Semantics.amount.currencyCode`, `@Semantics.quantity.unitOfMeasure`) tell consumers how to format values.
- **Aggregation views** with `GROUP BY` for analytical or KPI layers (`@Analytics.dataCategory: #CUBE`).
- **Provider contract** — `@AbapCatalog.viewEnhancementCategory` controls how the view can be extended downstream.

## Layered modelling pattern

The canonical RAP layering for an interface-stable, OData-exposed business object:

```
ZI_Order            <- "interface" view: 1:1 over DB tables, stable identifiers
ZR_Order            <- "root" view: adds compositions to children, root annotation
ZP_Order            <- "projection" / consumption: subset of fields, UI annotations
```

Behavior + service definition + service binding sit on top of `ZP_Order`.

## Code example — root view entity with composition + association

```abap
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Sales Order — Root View'
@ObjectModel.semanticKey: ['OrderId']
define root view entity ZR_Order
  as select from yotc_order_h as Header
  composition [0..*] of ZR_OrderItem  as _Items
  association [1..1] to I_BusinessPartner as _Customer
                       on $projection.CustomerId = _Customer.BusinessPartner
{
  key Header.order_id      as OrderId,
      Header.order_type    as OrderType,
      Header.customer_id   as CustomerId,

      @Semantics.amount.currencyCode: 'CurrencyCode'
      Header.net_amount    as NetAmount,
      Header.currency_code as CurrencyCode,

      @Semantics.quantity.unitOfMeasure: 'Unit'
      Header.weight        as Weight,
      Header.weight_unit   as Unit,

      // associations exposed for path access in projections / Fiori
      _Items,
      _Customer
}
```

## Code example — projection view

```abap
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Sales Order — Projection'
@Metadata.allowExtensions: true
@Search.searchable: true
define root view entity ZP_Order
  provider contract transactional_query
  as projection on ZR_Order
{
  key OrderId,
      @Search.defaultSearchElement: true
      OrderType,
      _Customer.LastName as CustomerName,   // path expression via association
      NetAmount,
      CurrencyCode,

      /* exposed composition */
      _Items : redirected to composition child ZP_OrderItem
}
```

## Common annotations

| Annotation                                       | Purpose                                              |
| ------------------------------------------------ | ---------------------------------------------------- |
| `@AccessControl.authorizationCheck`              | Whether DCL applies (`#CHECK`, `#NOT_REQUIRED`, `#NOT_ALLOWED`) |
| `@EndUserText.label`                             | Default label for UI                                 |
| `@ObjectModel.semanticKey`                       | Logical key (drives Fiori URL params)                |
| `@ObjectModel.compositionRoot`                   | Marks root of composition tree (alt. to `root` kw)   |
| `@Semantics.amount.currencyCode`                 | Amount + currency pair                               |
| `@Semantics.quantity.unitOfMeasure`              | Quantity + UoM pair                                  |
| `@UI.lineItem`, `@UI.facet`, `@UI.headerInfo`    | Fiori Elements layout                                |
| `@Search.searchable` / `defaultSearchElement`    | Full-text search behavior                            |
| `@Metadata.allowExtensions`                      | Allows append-extensions of the view                 |
| `@Analytics.dataCategory`                        | Marks cube/dimension/fact for analytics              |

## How it works

1. Activate the CDS source → SAP generates the underlying database view (or expression in HANA SQL when on HANA).
2. Associations resolve lazily at read time — `SELECT … FROM ZP_Order` does *not* join `_Customer` unless a field from it is selected.
3. Compositions are part of the object identity — RAP traverses them for save/draft/lock.
4. Annotations are read by:
   - **RAP runtime** (e.g. `@ObjectModel.*`).
   - **OData service generator** (e.g. `@UI.*`, `@Search.*`).
   - **DCL** (e.g. `@AccessControl.*`).

## Release & version notes

> [!info] DEFINE VIEW → DEFINE VIEW ENTITY
> `DEFINE VIEW` is legacy. New objects use `DEFINE VIEW ENTITY`. Existing legacy views can be migrated; they coexist but lack expressivity (no aggregation in projection, no provider contracts, no virtual elements).

> [!warning] Legacy reference
> On ECC, only DDIC views (SE11) exist — no CDS at all. Migration to S/4 is the trigger to introduce CDS.

## Related

- [[concepts/behavior-definition]] — the controller layer on top of a CDS root view entity.
- [[concepts/eml]] — read/modify these entities programmatically.
- [[concepts/restful-application-programming-model]] — the overall picture.
- [[concepts/odata-service]] — the runtime exposure.

## Anti-patterns / pitfalls

- Building a root view entity *without* `@ObjectModel.semanticKey` — Fiori URLs become opaque GUIDs.
- Using **associations** when you mean **compositions** — breaks delete cascade and draft propagation.
- Putting business logic in CDS expressions ("calc field that does seven CASEs") — push complex logic to ABAP, keep CDS readable.
- `select *` in a projection — be explicit; projections are the public API contract of the service.
- Forgetting `@Metadata.allowExtensions: true` on a projection meant to be extensible — downstream teams can't append fields.
- Mixing analytical (`#CUBE`) and transactional behavior in one view — split into separate views.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-rap-reference.md` (CDS view entities, annotations, layered modelling).
