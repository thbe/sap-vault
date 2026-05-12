---
title: "MVC in ABAP"
type: concept
tags: [mvc, design, abap, architecture]
sap_release: ["S/4HANA", "ECC"]
status: draft
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/restful-application-programming-model]]"
  - "[[patterns/multi-site-repo-architecture]]"
created: 2026-05-11
updated: 2026-05-11
---

# MVC in ABAP

> Separation of concerns into Model (data + business logic), View (presentation), and Controller (flow / coordination) — applied to ABAP custom development.

## Overview

ABAP's classical reporting style mixes DB access, business logic, and screen output in a single program. MVC discipline splits these so each layer can be tested, replaced, or reused independently. The Heinemann S/4 standard mandates MVC for new development.

## Key ideas

- **Model** = persistence + business rules. Implemented as a class (or set of classes) with no UI knowledge. Owns SELECTs, validations, calculations.
- **View** = whatever renders to a user: ALV grid, classical Dynpro screen, Web Dynpro view, Fiori UI5 (consuming OData). Should hold no business logic.
- **Controller** = orchestrates: receives user input, calls Model, hands result to View. In ABAP this is often the report/program itself, or a Web Dynpro/RAP controller.

## How it works

Typical mapping in S/4-era custom development:

| Layer       | Classical ABAP            | Modern S/4 (RAP)                     |
| ----------- | ------------------------- | ------------------------------------ |
| Model       | `*_CL_…` business class   | CDS view + behavior definition (BDEF)|
| View        | ALV / Dynpro / SAP GUI    | Fiori Elements + UI5 freestyle       |
| Controller  | Report / WD app           | RAP service binding + Fiori app      |

> [!info] Modern angle
> [[concepts/restful-application-programming-model|RAP]] is essentially MVC at the framework level: CDS = Model, BDEF = controller behavior, OData/Fiori = View. Custom MVC inside an ABAP class is mostly relevant for non-RAP custom code (background jobs, interface handlers, classical reports surviving on S/4).

## Code example

```abap
" Model
CLASS yotc_cl_order_model DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS get_open_orders
      IMPORTING ira_kunnr  TYPE RANGE OF kunnr
      RETURNING VALUE(rit_orders) TYPE yotc_tt_order_items.
ENDCLASS.

" View (ALV)
CLASS yotc_cl_order_view DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS display IMPORTING iit_orders TYPE yotc_tt_order_items.
ENDCLASS.

" Controller (the report)
REPORT yotc_rv_open_orders.
DATA: o_model TYPE REF TO yotc_cl_order_model,
      o_view  TYPE REF TO yotc_cl_order_view.
START-OF-SELECTION.
  CREATE OBJECT: o_model, o_view.
  o_view->display( iit_orders = o_model->get_open_orders( ra_kunnr ) ).
```

## Release & version notes

> [!info] Applicability
> MVC is a discipline, not a feature — works on any release. The standard recommends it for all new dev regardless of front-end choice.

## Related

- [[concepts/clean-abap]]
- [[concepts/restful-application-programming-model]]
- [[concepts/sap-fiori-overview]]
- [[patterns/multi-site-repo-architecture]] — site-specific BAdI implementations as the controller-layer split between shared and per-site behavior.

## EWM project specifics

In the Heinemann EWM landscape ([[sources/heinemann-ewm-coding-standards]]), the MVC split also runs along the **multi-site axis**:

- **Model** = shared business logic in `YEWM_*` classes; site-specific overrides via BAdI implementations in `ZALM_*` / `ZERL_*`.
- **View** = SAPUI5 / Fiori Elements consuming RAP services; classical RF dialog screens for warehouse-floor scanners (legacy but still primary UI for goods movement).
- **Controller** = RAP service for new apps; for RF screens, the controller is still the dialog flow logic — kept thin, delegating to a `YEWM_CL_*` model class.

See [[patterns/multi-site-repo-architecture]] for how the package layering enforces this split.

## Anti-patterns / pitfalls

- "Mega class" that knows about both DB tables and ALV columns — same problem as the procedural report.
- View calling DB directly (e.g. `SELECT` in a screen PBO module).
- Controller doing business logic that should live in Model.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §2.2, §2.3.
- [[sources/heinemann-ewm-coding-standards]] — `02-coding-guidelines.md` §"Site-specific extension points".
