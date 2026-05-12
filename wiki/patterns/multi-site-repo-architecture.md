---
title: "Multi-Site Repo Architecture (Three-Tier)"
type: pattern
tags: [architecture, abapgit, repo-structure, multi-site, abap]
sap_release: ["S/4HANA 2023 FPS04", "abapGit ≥ 1.130"]
status: draft
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/abapgit-serialization]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# Multi-Site Repo Architecture (Three-Tier)

> Reusable repository topology for SAP customer code that must serve **multiple operational sites** with shared common logic and site-specific extensions, plus cross-system utility libraries deployed independently across all SAP systems.

## Context

Many large enterprises run the same SAP module (EWM, FI, MM, …) on the same system but with **per-site variations** — different physical processes, different integrations, different paper trails. Two failure modes recur:

1. **Single site repo, copy-pasted per site** → divergence, double maintenance, bug fixes lost.
2. **Single common repo, site-specific `IF site = 'XYZ'` branches everywhere** → entangled code, cross-site regressions on every change.

The three-tier pattern resolves this: a **common layer** for genuinely shared logic, **site-specific layers** for variants, and **cross-system utility repos** for code that has nothing to do with the business module at all.

## Architecture

```
                    YBC          YCA
              (admin tools)  (user tools)
                    ▲            ▲
                    │            │  USE ACCESS
                    │            │
ZALM_EWM  ──→    YEWM     ←──  ZERL_EWM
(site A)      (common)        (site B)
```

| Repo            | Role                                          | Namespace        |
| --------------- | --------------------------------------------- | ---------------- |
| `YEWM`          | Common business module (EWM)                  | `Y*` (shared)    |
| `ZALM_EWM`      | Site A (Allermöhe) — module extensions        | `Z*` (site)      |
| `ZERL_EWM`      | Site B (Erlensee) — module extensions         | `Z*` (site)      |
| `YBC`           | Admin/system utilities (basis, jobs, users)   | `Y*` (shared)    |
| `YCA`           | App utilities (XLSX, REST, string/XML/email)  | `Y*` (shared)    |

### Hard rules

- **Site repos never depend on each other.** ZALM_EWM and ZERL_EWM are siblings, not partners. A change to one cannot directly affect the other.
- **Site repos depend on common.** ZALM_EWM and ZERL_EWM both have a USE ACCESS reference to YEWM.
- **Common depends on cross-system utilities.** YEWM has USE ACCESS to YBC and YCA.
- **Cross-system utilities have no dependencies.** YBC and YCA are independent siblings, deployed across every SAP system on the landscape.
- **Y-prefix scope = shared/common.** Site-specific objects keep `Z*` — no rename churn.

### Import order

```
1. YBC + YCA          (parallel, independent)
2. YEWM               (depends on YBC, YCA)
3. ZALM_EWM           ┐ parallel,
   ZERL_EWM           ┘ both depend on YEWM
```

## Key decisions

- **Where does an object belong?** Decision tree:
  ```
  Is the object identical in both sites?
    ├── YES → YEWM (common)
    └── NO
         ├── Site A only? → ZALM_EWM
         ├── Site B only? → ZERL_EWM
         └── Diverged?
             ├── Common base extractable? → Base in YEWM, extensions in site repos
             └── No common base? → Fork to both site repos (highest maintenance cost)
  ```
- **Y-prefix renaming** applies only to objects entering YEWM. Site-repo objects keep their `Z*` names.
- **Package naming**: `<ROOT>_<SUFFIX>` (e.g., `YEWM_CO`, `ZALM_GI`, `ZERL_ACC`). Suffix re-used directly from source `ZEWM<SUFFIX>` packages where applicable.
- **Folder logic**: PREFIX uniformly across all repos. See [[concepts/abapgit-serialization]].
- **Master language**: E uniformly across all repos.
- **Transaction prefixes**: `Y_EWM_*` (common), `Y_ALM_*` (site A), `Y_ERL_*` (site B).

## When to use

✅ Same SAP module deployed to ≥ 2 operationally-distinct sites with shared logic.
✅ Distinct teams owning each site — clear repo boundaries reduce merge conflict risk.
✅ Long-term roadmap includes more sites — the pattern scales horizontally (`Z<SITE>_EWM`).
✅ Cross-system tooling exists that doesn't belong to any business module — extract to YBC/YCA.

## When NOT to use

❌ Single-site deployment — overhead without benefit.
❌ Sites are short-lived (e.g., temporary warehouses) — fold their code into the common layer.
❌ Logic is identical across sites with only customising differences — use TVARVC or `YS00_DB_ENH_C` table-driven config in the common layer instead. See [[patterns/table-driven-enhancement-framework]].

## Trade-offs

| Pro                                                           | Con                                                         |
| ------------------------------------------------------------- | ----------------------------------------------------------- |
| Clear ownership: site teams own site repos                    | Higher cognitive overhead (3+ repos to navigate)            |
| Bug fixes in common layer benefit all sites automatically     | Cross-repo refactors require coordinated transports         |
| No risk of site-A change breaking site B                      | Common-layer changes need broader regression testing        |
| Y/Z prefix lets reviewers see scope at a glance               | Phase 6 mandatory rename for objects moving Z→Y is non-trivial |
| Cross-system utilities deployable to every system in landscape | YBC/YCA versioning must stay backward-compatible            |

## Implementation checklist

1. **Inventory** every object → classify as common / site-specific / cross-system / dead code.
2. **Set up four+ Git repos** with identical `.abapgit.xml` template (PREFIX, `/src/`, `E`).
3. **Create root packages** (`YEWM`, `ZALM_EWM`, `ZERL_EWM`, `YBC`, `YCA`) as structure packages where appropriate.
4. **Configure USE ACCESS**: site repos → YEWM; YEWM → YBC, YCA. **No** site-to-site or YBC↔YCA links.
5. **Migrate in dependency order**: YBC + YCA → YEWM → site repos.
6. **Per-phase quality gate**: ATC + ABAP Unit + Extended Program Check on each repo before proceeding. See [[topics/abap-quality-gates]].
7. **Cross-reference update**: when a YEWM object is renamed Z→Y, update every site-repo reference.
8. **Document the topology** in each repo's README so newcomers understand the contract.

## Variations

- **Two-tier** (common + one site) — collapse to a single common repo if the site is the only consumer; revisit when a second site appears.
- **Module-tree common** — split YEWM further by sub-module (`YEWM_RF`, `YEWM_GR`) if YEWM grows beyond ~5,000 objects.
- **Per-system YBC** — when a system has truly system-specific basis tooling, fork YBC; rare.

## Related

- [[concepts/abap-naming-conventions]] — Y/Z scoping rules this pattern operationalises.
- [[concepts/abapgit-serialization]] — folder logic + DEVCLASS conventions.
- [[topics/abap-quality-gates]] — per-phase quality gates that protect cross-repo integrity.
- [[patterns/table-driven-enhancement-framework]] — alternative for site variants that are pure config differences.

## Anti-patterns / pitfalls

- **Site-to-site dependency** — even one breaks the model. Common code must absorb the shared logic.
- **God common repo** — pulling everything into YEWM "to be safe" recreates the entanglement problem.
- **Mixed folder logic** across sibling repos — defeats architectural symmetry.
- **Lazy classification** — calling something "common" without verifying both sites actually use the same logic.
- **Skipping cross-reference update** after a Z→Y rename — site repos compile, but ATC scans surface `Z*` ghosts.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `AGENTS.md` §Migration Architecture; `01-naming-conventions.md` §1, §2; `02-coding-guidelines.md` G1.
