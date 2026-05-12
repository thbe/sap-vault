---
title: "EML (Entity Manipulation Language)"
type: concept
tags: [rap, eml, abap, s4hana, btp-abap]
sap_release: ["S/4HANA 1909+", "BTP ABAP"]
status: draft
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/restful-application-programming-model]]"
  - "[[concepts/cds-view-entities]]"
  - "[[concepts/behavior-definition]]"
  - "[[topics/abap-unit-testing]]"
created: 2026-05-11
updated: 2026-05-11
---

# EML (Entity Manipulation Language)

> The ABAP-side API for reading, creating, modifying, deleting, and executing actions on RAP business objects — the way ABAP code talks to a behavior-defined entity without falling back to direct DB access.

## Overview

EML is the layer between handler/saver classes (or any consumer of a RAP BO) and the RAP runtime. Where ABAP SQL talks to DB tables, EML talks to RAP entities: it respects authorization, draft handling, validations, determinations, and the entire behavior model.

The five core statements:

| Statement                  | Purpose                              |
| -------------------------- | ------------------------------------ |
| `READ ENTITY[IES]`         | Fetch instance data (or feature ctrl)|
| `MODIFY ENTITY[IES]`       | Create / Update / Delete             |
| `MODIFY … EXECUTE`         | Run an action                        |
| `COMMIT ENTITIES`          | Persist changes (calls savers)       |
| `ROLLBACK ENTITIES`        | Discard the buffer                   |

## Key ideas

- **EML works against a transactional buffer**, not the DB. `MODIFY` queues changes; `COMMIT ENTITIES` runs validations + savers and writes.
- **`IN LOCAL MODE`** bypasses authorization — used in tests and in trusted dispatchers; never in user-facing flows.
- **`FAILED`, `REPORTED`, `MAPPED`** are the three response structures every modify returns. Inspect them — RAP doesn't raise exceptions for soft business failures.
- **Path expressions via compositions**: `CREATE BY \_Items` to insert children under a parent without separate key plumbing.
- **`%cid` (content ID)** is the client-assigned temp ID for new instances. Use it to link parent+children created in the same call.
- **`%control`** carries a structure of `if_abap_behv=>mk-on` flags indicating which fields are being modified.
- **Always check `failed`** after a modify — a successful return value doesn't mean the change took effect.

## Code example — read

```abap
READ ENTITIES OF ZR_Order
  ENTITY Order
    FIELDS ( OrderId OrderType NetAmount CurrencyCode )
    WITH VALUE #( ( OrderId = '0001000001' )
                  ( OrderId = '0001000002' ) )
  RESULT DATA(orders)
  FAILED DATA(failed)
  REPORTED DATA(reported).

LOOP AT orders INTO DATA(order).
  " ... use order-NetAmount, etc.
ENDLOOP.
```

## Code example — create with child by composition

```abap
MODIFY ENTITIES OF ZR_Order
  ENTITY Order
    CREATE FIELDS ( OrderType CustomerId CurrencyCode )
      WITH VALUE #( ( %cid       = 'NEW_ORDER_1'
                      OrderType  = 'TA'
                      CustomerId = 'C0001'
                      CurrencyCode = 'EUR' ) )

    CREATE BY \_Items
      FIELDS ( Product Quantity )
      WITH VALUE #( ( %cid_ref = 'NEW_ORDER_1'      " link to parent
                      %target  = VALUE #(
                        ( %cid     = 'ITEM_1'
                          Product  = 'P-100'
                          Quantity = 5 )
                        ( %cid     = 'ITEM_2'
                          Product  = 'P-200'
                          Quantity = 3 ) ) ) )

  MAPPED   DATA(mapped)
  FAILED   DATA(failed)
  REPORTED DATA(reported).

IF failed IS INITIAL.
  COMMIT ENTITIES
    RESPONSE OF ZR_Order
    FAILED DATA(commit_failed)
    REPORTED DATA(commit_reported).
ENDIF.
```

After commit, `mapped-order` holds the mapping `%cid` → real `OrderId`.

## Code example — execute action

```abap
MODIFY ENTITIES OF ZR_Order
  ENTITY Order
    EXECUTE Approve
    FROM VALUE #( ( OrderId = '0001000001'
                    %param  = VALUE #( comment = 'OK for shipment' ) ) )
  RESULT   DATA(result)
  FAILED   DATA(failed)
  REPORTED DATA(reported).

COMMIT ENTITIES.
```

## Code example — update with %control

```abap
MODIFY ENTITIES OF ZR_Order
  ENTITY Order
    UPDATE
      FIELDS ( NetAmount )       " also implicit via %control flag
      WITH VALUE #( ( OrderId   = '0001000001'
                      NetAmount = '12500.00'
                      %control  = VALUE #( NetAmount = if_abap_behv=>mk-on ) ) )
  FAILED DATA(failed).
```

## `IN LOCAL MODE` (tests / trusted callers)

```abap
MODIFY ENTITIES OF ZR_Order IN LOCAL MODE
  ENTITY Order
    CREATE FIELDS ( OrderType ) ...
```

`IN LOCAL MODE` bypasses authorization checks. Reserved for:

- Unit tests using `CL_CDS_TEST_ENVIRONMENT`.
- Trusted dispatcher code where authorization was checked at a higher layer.

> [!warning] Never in user-facing code
> Bypassing auth checks in production-reachable code is a security finding — flag as [[topics/abap-security-checklist]] critical.

## Failed / Reported / Mapped

| Structure  | Purpose                                                                            |
| ---------- | ---------------------------------------------------------------------------------- |
| `FAILED`   | Per-key failure indicators (`%fail-cause = if_abap_behv=>cause-…`).                |
| `REPORTED` | Messages (warnings, errors) keyed by entity instance — surface to user.            |
| `MAPPED`   | `%cid → key` mappings (and `%pid → physical id`) for newly-created instances.      |

Always inspect `failed` before issuing `COMMIT ENTITIES` — if non-initial, roll back or skip the commit.

## Release & version notes

> [!info] Available since
> EML on managed RAP: **S/4HANA 1909**. Draft-aware EML: **S/4HANA 2020+**. Released to BTP ABAP from start.

## Related

- [[concepts/cds-view-entities]] — what EML reads from.
- [[concepts/behavior-definition]] — what EML obeys.
- [[topics/abap-unit-testing]] — `CL_CDS_TEST_ENVIRONMENT` for testing EML flows.
- [[concepts/restful-application-programming-model]] — the umbrella concept.

## Anti-patterns / pitfalls

- **Forgetting to check `failed`** — RAP soft-fails silently; your test passes, production data is wrong.
- **`COMMIT ENTITIES` without checking `failed` first** — commits a partial state.
- **Direct DB write (`INSERT yotc_order_h`) bypassing EML** — skips validations, determinations, lock, draft propagation. Use unmanaged scenario if you need to.
- **`IN LOCAL MODE` in handler classes** — almost always wrong; handlers run under the caller's authorization context.
- **Calling EML inside a validation handler** — risk of reentrancy; use the `keys` table passed in instead.
- **Reading `%cid_ref` results before commit** — the real key isn't assigned yet. Use `MAPPED` from the response.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-rap-reference.md` (EML statements, structures, IN LOCAL MODE).
