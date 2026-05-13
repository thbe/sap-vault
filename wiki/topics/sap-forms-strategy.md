---
title: "SAP Forms Strategy"
type: topic
tags: [forms, adobe-forms, smart-forms, sapscript, brfplus, abap]
sap_release: ["S/4HANA", "ECC (legacy reference)"]
status: stable
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[patterns/brf-plus-ui-guard]]"
created: 2026-05-11
updated: 2026-05-13
---

# SAP Forms Strategy

> Which form technology to choose for new development, what to migrate, what to keep, and what to never start.

## Why this matters

SAP has accumulated five forms technologies over three decades. New developers default to whatever they last saw. The Heinemann EWM standard codifies a hard preference hierarchy so the codebase doesn't grow in five directions at once.

## Preference hierarchy

| Rank | Technology               | Use for                                             | Status                  |
| ---- | ------------------------ | --------------------------------------------------- | ----------------------- |
| 1    | **BRF+ Adobe Forms**     | Output where business needs configurable rules     | **Preferred**           |
| 2    | **Standard XML Forms**   | Standardised layouts (delivery notes, invoices)    | Preferred               |
| 3    | **Custom Adobe Forms (SFP)** | Free-form custom output                        | Acceptable for new dev  |
| 4    | **Smart Forms (SSF)**    | Existing legacy forms                              | Acceptable, no new dev  |
| 5    | **SAPScript**            | —                                                  | **Avoid** completely    |

## Decision tree

```
Need form output?
  ├── Will business want to vary layout/content via configuration?
  │     └── YES → BRF+ Adobe Forms (rank 1)
  │
  ├── Is there a standard layout (e.g. delivery, invoice)?
  │     └── YES → Standard XML Forms (rank 2)
  │
  ├── Custom output, no rule variability?
  │     └── YES → Custom Adobe Forms (SFP, rank 3)
  │
  └── Maintaining an existing Smart Form?
        └── YES → Keep it (rank 4) — do not extend, do not start new ones
```

**SAPScript**: never. Migration plans should mark any remaining SAPScript output for conversion to Adobe Forms before the source object is touched again.

## Migration policy (Heinemann EWM)

- **23 existing Smart Forms** (20 ERL + 3 ALM) migrate **as-is** — they work, no business need to change.
- **Future maintenance** of those Smart Forms: keep editing them in place. Don't extend significantly; if a major redesign is needed, that's the trigger to convert to Adobe.
- **New form output requirements** raised post-migration: must use Adobe Forms (SFP) or BRF+ Adobe Forms.
- **Optional Phase 6 work unit**: evaluate the 23 Smart Forms for Adobe Forms conversion. Low priority — these work fine, focus is on migration first.

## Practical guidance

### Choosing BRF+ Adobe vs Custom Adobe

Use BRF+ Adobe Forms when **at least one** of these is true:

- Business wants to change layout or content based on customer/region/product without an ABAP transport.
- The form has > 3 conditional branches (e.g. show different sections per delivery type).
- Multiple form variants exist that differ only by rule logic (consolidates them under one configurable form).

Use Custom Adobe Forms (SFP only, no BRF+) when:

- The form is purely static layout with a few data bindings.
- The complexity does not justify a BRF+ setup.

### When to migrate a Smart Form to Adobe

- Layout needs structural redesign (not just text or position tweaks).
- ABAP Cloud / Steampunk deployment is on the roadmap (Smart Forms are not supported on cloud).
- The form is in the HIGH-priority FORM-debt list — refactor at the same time.

### Productionising Adobe Forms

- Adobe LiveCycle Designer (or its successor) for layout.
- Form interface (`SFP` transaction) — define the data structure separately from the layout.
- BRF+ application + decision tables for rule-driven variants.
- Output via `cl_fp` API (modern) or BC-XOM / output management config.

## Anti-patterns

- **Mixing technologies in one form output** — e.g. SAPScript wrapper calling an Adobe sub-form. Pick one.
- **Hard-coded form names in ABAP** — use TVARVC or `YS00_DB_ENH_C` style configuration so business can switch variants without a transport.
- **Smart Form for new requirement "because we know how"** — short-term ease, long-term debt, blocks ABAP Cloud migration.
- **SAPScript for any new output** — ban.
- **Opening a Smart Form in ECC and "lifting and shifting" to S/4 without re-evaluation** — opportunity to right-size missed.

## Related

- [[patterns/brf-plus-ui-guard]] — BRF+ workbench safety pattern (production guard).
- [[concepts/clean-abap]] — same modernisation drive applied across all object types.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `02-coding-guidelines.md` G4 (Forms Strategy).
- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 3 (Forms section).
- [[sources/opencode-abap-context-library]] — `architecture-deep.md` (forms hierarchy corroborated).
