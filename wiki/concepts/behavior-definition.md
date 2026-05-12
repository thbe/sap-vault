---
title: "Behavior Definition (BDL)"
type: concept
tags: [rap, bdl, behavior, abap, s4hana, btp-abap]
sap_release: ["S/4HANA 1909+", "BTP ABAP"]
status: draft
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/restful-application-programming-model]]"
  - "[[concepts/cds-view-entities]]"
  - "[[concepts/eml]]"
created: 2026-05-11
updated: 2026-05-11
---

# Behavior Definition (BDL)

> The controller layer of a RAP business object. A `.bdef` source declares what operations the object supports, how persistence works, which fields are locked / mandatory / read-only, and where business logic hooks in (determinations, validations, side effects, actions).

## Overview

The BDL (Behavior Definition Language) source sits beside the CDS root view entity. Two artefact types:

- **Base BDEF** (`define behavior for ZR_Order …`) — implementation-side. Declares persistence model, fields, business logic hooks.
- **Projection BDEF** (`projection; define behavior for ZP_Order …`) — exposure-side. Subset of base behaviors made visible to a specific service.

Implementation lives in **behavior handler classes** (for determinations, validations, actions) and **saver classes** (for unmanaged save and `additional save`).

## Key ideas

- **Persistence flavour**:
  - **`managed`** — RAP runtime persists to a target table you declare; you write zero CRUD code.
  - **`managed with additional save`** — RAP persists, but your saver runs alongside (e.g. write to an audit log).
  - **`unmanaged`** — you write all CRUD in a saver class; used when wrapping legacy logic or non-DB persistence.
- **`strict( 2 )`** activates the **latest validation rules** (post-S/4 2022). Anything older is grandfathered. Always declare `strict(2)` on new BDEFs.
- **`with draft;`** enables draft handling — RAP persists half-finished edits to a parallel draft table so Fiori "save as draft" works.
- **Field characteristics**:
  - `readonly` — never settable.
  - `mandatory` — must be set on create.
  - `readonly:update` — settable on create, not on update.
  - `numbering : managed` — RAP assigns the key.
  - `numbering : unmanaged` — you assign keys in a determination.
- **Standard operations**: `create`, `update`, `delete`, plus `lock master` (or `lock dependent by _Parent` for children).
- **Determinations** — run *after* fields change. `determine on save`, `determine on modify { field a; field b; }`. Used to derive values.
- **Validations** — run *before* save. Reject the change if business rules violated. `validate on save { create; update; field a; }`.
- **Side effects** — UI metadata. Tell Fiori Elements "when field X changes, refetch fields Y and Z".
- **Actions** — non-CRUD operations exposed on the entity. `action ( factory true ) Approve result [1] $self;`.
- **Authorization**: `authorization master ( global, instance )` declares which authorization handler methods apply.

## Skeleton — managed root BDEF

```abap
managed implementation in class zbp_r_order unique;
strict ( 2 );
with draft;

define behavior for ZR_Order alias Order
persistent table yotc_order_h
draft table yotc_order_d
lock master
total etag LastChangedAt
authorization master ( instance )
etag master LastChangedAt
{
  // numbering & key
  field ( numbering : managed, readonly ) OrderId;

  // immutable after create
  field ( readonly : update ) OrderType, CurrencyCode;

  // mandatory on create
  field ( mandatory ) CustomerId;

  // operations
  create;
  update;
  delete;

  // association behaviour
  association _Items { create; with draft; }

  // logic hooks
  determination calculateTotal on modify { field _Items; create; }
  validation customerExists      on save  { field CustomerId; create; }
  validation netAmountPositive   on save  { field NetAmount; create; update; }

  // side effects (UI hints)
  side effects {
    field CustomerId affects field CustomerName;
    field _Items     affects field NetAmount;
  }

  // action
  action ( features : instance ) Approve result [1] $self;
}
```

## Skeleton — projection BDEF

```abap
projection;
strict ( 2 );

define behavior for ZP_Order alias Order
{
  use create;
  use update;
  use delete;

  use action Approve;

  use association _Items { create; }
}
```

Only operations explicitly listed via `use` are exposed on the projection.

## Handler class skeleton

```abap
CLASS zbp_r_order DEFINITION
  PUBLIC ABSTRACT FINAL FOR BEHAVIOR OF ZR_Order.
ENDCLASS.

CLASS lhc_order DEFINITION INHERITING FROM cl_abap_behavior_handler.
  PRIVATE SECTION.
    METHODS:
      calculateTotal FOR DETERMINE ON MODIFY
        IMPORTING keys FOR Order~calculateTotal,

      customerExists FOR VALIDATE ON SAVE
        IMPORTING keys FOR Order~customerExists,

      Approve FOR MODIFY
        IMPORTING keys FOR ACTION Order~Approve RESULT result.
ENDCLASS.
```

## Determination vs Validation vs Side Effect

| Mechanism      | Runs when                  | Purpose                                            | Can fail save? |
| -------------- | -------------------------- | -------------------------------------------------- | -------------- |
| Determination  | After change, before save  | Derive fields, propagate values                    | No (use validation if it might) |
| Validation     | On save                    | Reject invalid state, report messages              | Yes            |
| Side Effect    | UI metadata                | Tell Fiori what to refetch when field changes      | N/A (no ABAP code) |

## Release & version notes

> [!info] strict levels
> `strict ( 0 )` accepts anything (legacy). `strict ( 1 )` enforces post-1909 rules. `strict ( 2 )` enforces post-2022 rules. New code: always `strict ( 2 )`.

> [!warning] Available since
> Draft requires **S/4HANA 2020** or BTP ABAP. Managed-with-additional-save: S/4HANA 1909+.

## Related

- [[concepts/cds-view-entities]] — the data layer this BDEF binds to.
- [[concepts/eml]] — programmatic API to trigger behavior.
- [[concepts/restful-application-programming-model]]

## Anti-patterns / pitfalls

- **Forgetting `strict ( 2 )`** — old rules apply, latest checks silently skipped.
- **Putting business logic in `managed implementation` handlers when you really meant `unmanaged`** — fights the framework.
- **Validations that derive values** — that's a determination. Validations only inspect, never write.
- **No `lock master`** — concurrent edits collide silently in the runtime.
- **Side effects without backing logic** — Fiori asks for fields that haven't been computed; users see stale data.
- **Exposing every operation on the projection** — defeats the whole point of having a separate projection BDEF. Use `use` selectively.
- **Action without `result $self`** — Fiori refresh is broken; user sees the pre-action state.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-rap-reference.md` (BDL grammar, strict levels, field characteristics, determinations/validations/side-effects, projection BDEFs).
