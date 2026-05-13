---
title: "BRF+ UI Guard"
type: pattern
tags: [brfplus, security, governance, custom-framework, abap]
sap_release: ["S/4HANA (S4CORE 104+)"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/abap-naming-conventions]]"
created: 2026-05-11
updated: 2026-05-13
---

# BRF+ UI Guard

> A custom UI-mode subclass of `CL_FDT_WD_UI_SIMPLE_MODE` that hides risky toolbar actions (Delete, Mark/Unmark Obsolete, Search My Object, etc.) so power users can maintain BRF+ decision tables in production without being able to alter table structure or metadata.

## Problem

To make BRF+ decision tables editable in TST/PRD (so business can maintain mappings without a transport) you have to set up an Application Exit class implementing `IF_FDT_APPLICATION_SETTINGS` and override `GV_AUTHORITY_CHECK` and `GV_GET_CHANGEABILITY` in its constructor. Once enabled, **the standard BRF+ Workbench UI exposes destructive actions**: add/remove fields, mark obsolete, change properties. This is unsafe in PRD.

## Solution

Wrap the BRF+ Workbench in a custom UI mode that strips the dangerous actions:

1. **Custom class `YOTC_CL_BRF_UI_CONTROL`** inherits from `CL_FDT_WD_UI_SIMPLE_MODE`.
2. Redefine `IF_FDT_WD_UI_MODE~GET_CONFIGURATION` to remove undesired controls via:
   - `IF_FDT_WD_REPOSITORY_CONFIG`
   - `IF_FDT_WD_OBJM_CONFIGURATION`
   - `IF_FDT_WD_UI_MODE`
3. **Custom report** (modelled on `FDT_WD_START_CATALOG_BROWSER`) that launches the BRF+ browser with this UI mode injected.
4. **Custom transaction code** wrapping that report — given to power users instead of the standard `BRF+`/`BRFPLUS` TCODE.
5. (Optional) Embed the custom TCODE in a Fiori app — requires the report to expose at least one PARAMETER or SELECT-OPTION.

## Implementation

```abap
"! YOTC_CL_BRF_UI_CONTROL — restricts BRF+ Workbench UI in PRD/PRE-PRD.
CLASS yotc_cl_brf_ui_control DEFINITION
  PUBLIC INHERITING FROM cl_fdt_wd_ui_simple_mode FINAL.

  PUBLIC SECTION.
    METHODS if_fdt_wd_ui_mode~get_configuration REDEFINITION.

ENDCLASS.

CLASS yotc_cl_brf_ui_control IMPLEMENTATION.
  METHOD if_fdt_wd_ui_mode~get_configuration.
    " Call super
    super->if_fdt_wd_ui_mode~get_configuration(
      IMPORTING eo_repository_config = DATA(lo_repo)
                eo_objm_configuration = DATA(lo_objm) ).

    " Strip risky actions
    lo_repo->set_search_my_objects_enabled( abap_false ).
    lo_objm->set_delete_button_enabled( abap_false ).
    lo_objm->set_mark_obsolete_enabled( abap_false ).
    lo_objm->set_unmark_obsolete_enabled( abap_false ).
    " ... add further restrictions as required

    eo_repository_config = lo_repo.
    eo_objm_configuration = lo_objm.
  ENDMETHOD.
ENDCLASS.
```

> [!note] Method signatures
> The exact setter names depend on the SAP release. Inspect the parent class and interface in your system; the source PDF lists them generically as "Search My Object Option, Delete Toolbar button, Unmark Obsolete, Mark Obsolete etc."

## When to use

- Production / pre-production BRF+ decision tables maintained by business users.
- Audit / SoD requirements: data maintenance allowed, structural change forbidden.
- BRF+ used as a "configurable code" mechanism (not transport-driven).

## When NOT to use

- DEV systems — full BRF+ access is intended there.
- Pure transport-driven BRF+ where no editing happens in PRD.
- Single-power-user scenarios where SU24 / role-based authorization on `BRFPLUS` already suffices.

## Related patterns

- *(future)* `patterns/brfplus-application-exit` — the prerequisite that makes objects editable in non-DEV.
- [[patterns/table-driven-enhancement-framework]] — same governance philosophy: lock down what the standard exposes too liberally.

## Anti-patterns

- Granting end users the standard BRF+ TCODE in PRD with the Application Exit enabled — unrestricted destructive actions.
- Putting the custom report on a role without a corresponding training / approval workflow for the maintainable decision tables.
- Forgetting to add a parameter to the report when surfacing it via Fiori — the launch will fail.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — §6.4 in full; §6.3.3 for the BRF+ Application Exit prerequisite.
- [[sources/opencode-abap-context-library]] — `architecture-deep.md` (BRF+ UI guard pattern corroborated).
