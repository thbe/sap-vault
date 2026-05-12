---
title: "abapGit Serialization"
type: concept
tags: [abapgit, git, serialization, packaging, abap]
sap_release: ["S/4HANA 2023 FPS04", "abapGit ≥ 1.130"]
status: draft
sources:
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/abap-naming-conventions]]"
  - "[[patterns/multi-site-repo-architecture]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-11
updated: 2026-05-11
---

# abapGit Serialization

> How abapGit converts ABAP repository objects into a Git-trackable on-disk layout — and the rules a Heinemann project follows so the layout stays clean across repos and SAP systems.

## Overview

abapGit serializes every SAP object into one or more files under `/src/`. Each object becomes a paired set: `.<obj>.abap` (source) + `.<obj>.xml` (metadata). Configuration lives in a root `.abapgit.xml` file. Folder layout is controlled by the **folder logic** setting — `FULL` (deep, packages encoded in folder names) or `PREFIX` (shallow, root-package prefix stripped).

> [!info] Applies to
> S/4HANA on-premise via abapGit ≥ 1.130. The Heinemann EWM migration explicitly standardises on PREFIX. BTP ABAP uses gCTS instead, with similar concepts.

## Key ideas

- **Object pair invariant** — every `.abap` source file has a paired `.xml` metadata file. Never create one without the other.
- **`#` in filenames** is SAP namespace encoding (e.g. `#igz#bas_hu.clas.abap`). The `#` replaces `/` from `/IGZ/` namespace. Do not "fix" these.
- **PREFIX folder logic** is the project standard — produces shorter, more readable paths than FULL.
- **Master language** uniformly `E` (English) across all repos.
- **Lowercase filenames**, **UTF-8** encoding, **LF** line endings.
- **No system-specific data** in serialized XML — strip usernames, system IDs, timestamps that don't belong to the object's authored history.
- **Deserialization order matters** — domains before data elements before tables before classes; abapGit handles most of this but cross-package dependencies need awareness.

## How it works

### `.abapgit.xml` (repository root)

```xml
<?xml version="1.0" encoding="utf-8"?>
<asx:abap xmlns:asx="http://www.sap.com/abapxml" version="1.0">
 <asx:values>
  <DATA>
   <MASTER_LANGUAGE>E</MASTER_LANGUAGE>
   <STARTING_FOLDER>/src/</STARTING_FOLDER>
   <FOLDER_LOGIC>PREFIX</FOLDER_LOGIC>
   <IGNORE>
    <item>/.gitignore</item>
    <item>/LICENSE</item>
    <item>/README.md</item>
    <item>/docs/</item>
    <item>/package.json</item>
   </IGNORE>
  </DATA>
 </asx:values>
</asx:abap>
```

Identical in structure across all sibling repos — only the root SAP package (derived externally) differs.

### `package.devc.xml` (per package)

```xml
<DEVCLASS>YEWM_CO</DEVCLASS>     <!-- common layer -->
<DEVCLASS>ZALM_CO</DEVCLASS>     <!-- Allermöhe site -->
<DEVCLASS>ZERL_CO</DEVCLASS>     <!-- Erlensee site -->
```

`CTEXT` (description) reviewed and standardised in English at migration time.

### Object XML — DEVCLASS update

Individual object XML files (`.clas.xml`, `.tabl.xml`, `.fugr.xml`, `.intf.xml`, `.ssfo.xml`, etc.) contain a `DEVCLASS` field that **must** be updated when the object is reassigned to a new package. The object **name** field stays unchanged unless an explicit rename is in scope.

| XML Element  | Action on package change                     |
| ------------ | -------------------------------------------- |
| `<DEVCLASS>` | Change to new target package                 |
| `<CNAM>`     | Leave unchanged (original author)            |
| `<CDAT>`     | Leave unchanged (original creation date)     |
| Object name  | Leave unchanged (rename is a separate step)  |

## FULL → PREFIX folder logic

| Property     | FULL                                  | PREFIX                                  |
| ------------ | ------------------------------------- | --------------------------------------- |
| Folder name  | Full package name (e.g., `zewmco/`)   | Package name minus root prefix          |
| Root folder  | `/src/zewm/`                          | `/src/`                                 |
| Nesting      | Deep (`/src/zewm/zewmco/zewmcosub/`)  | Shallow (`/src/co/cosub/`)              |

Conversion formula:

```
Source package:  ZEWM<suffix>
Target package:  <ROOT>_<SUFFIX>     (ROOT = YEWM | ZALM | ZERL)
Target folder:   /src/<suffix>/

Source package:  ZEWM<parent><child>
Target package:  <ROOT>_<PARENT>_<CHILD>
Target folder:   /src/<parent>/<child>/
```

Concrete example:

```
Source (FULL):                          Target (PREFIX):
/src/zewm/zewmco/zcl_ewm_co.clas    →  /src/co/zcl_ewm_co.clas.abap
/src/zewm/zewmrf/zewmrfmon/         →  /src/rf/mon/
/src/zewm/zewmrf/zewmrfmon/dlg/     →  /src/rf/mon/dlg/
```

## Code example — minimal repo layout

```
YEWM/
├── .abapgit.xml                          # FOLDER_LOGIC=PREFIX
├── README.md
├── src/
│   ├── package.devc.xml                  # DEVCLASS=YEWM (structure pkg)
│   ├── co/
│   │   ├── package.devc.xml              # DEVCLASS=YEWM_CO
│   │   ├── ycl_ewm_co_stock.clas.abap
│   │   └── ycl_ewm_co_stock.clas.xml
│   └── rf/
│       ├── package.devc.xml              # DEVCLASS=YEWM_RF
│       └── mon/
│           ├── package.devc.xml          # DEVCLASS=YEWM_RF_MON
│           └── ycl_ewm_rf_monitor.clas.abap
└── docs/
```

## Release & version notes

> [!info] Available since
> Folder-logic switching requires abapGit **≥ 1.130**. Older versions only support FULL.

> [!warning] BTP ABAP
> abapGit on BTP is read-only / development-only. Production deployment uses gCTS. The serialization concepts apply but the workflow differs.

## Migration cleanup checklist (FULL → PREFIX)

When converting an existing repo:

1. Verify abapGit ≥ 1.130 in source + target systems.
2. Remap folder paths per the conversion formula above.
3. Update `<DEVCLASS>` in every object XML to the new target package name.
4. Generate fresh `package.devc.xml` for each new package node.
5. Verify UTF-8 + LF encoding (most tools handle automatically).
6. Strip system-specific data (e.g. development-system IDs that crept in via manual edits).
7. Deserialize on a clean target system — abapGit will refuse if order constraints are violated.

## Related

- [[concepts/abap-naming-conventions]] — package + object names that drive the folder layout.
- [[patterns/multi-site-repo-architecture]] — three-tier repo split this serialization layout supports.
- [[topics/abap-quality-gates]] — serialization cleanup as a Phase 2 quality gate.

## Anti-patterns / pitfalls

- Manually editing `.abapgit.xml` mid-flight without understanding deserialization implications.
- Renaming `#igz#…` files to "fix" the encoding — breaks SAP namespace mapping.
- Mixing folder logics across sibling repos — defeats the architectural symmetry.
- Committing `.abap` without `.xml` (or vice versa) — abapGit refuses to import.
- Leaving system-specific timestamps/usernames in serialized XML — pollutes Git history with noise diffs on every re-export.

## File encoding & whitespace rules

abapGit is strict about on-disk encoding. Deviations cause spurious diffs on every export.

| File type            | Encoding       | Line endings | Indent                |
| -------------------- | -------------- | ------------ | --------------------- |
| `.abap` source       | UTF-8 (no BOM) | LF           | 2 spaces              |
| `.xml` metadata      | UTF-8 **+ BOM**| LF           | 1 space               |
| `.json` (AFF)        | UTF-8 (no BOM) | LF           | 2 spaces              |
| `.devc.xml` package  | UTF-8 + BOM    | LF           | 1 space               |

- **Final newline** mandatory on every file.
- **No trailing whitespace** on any line.
- **Tabs forbidden** anywhere in serialized output.

## Deserialization phases

abapGit imports objects in four ordered phases. Cross-phase dependencies that violate this order cause activation errors:

1. **EARLY** — domains, data elements, message classes (anything with no ABAP dependencies).
2. **DDIC** — tables, views, lock objects, search helps, table types, structures.
3. **ABAP** — classes, interfaces, function groups, programs, includes.
4. **LATE** — enhancement implementations, transport-bound objects, view clusters.

Within a phase, alphabetical/dependency order applies. Violations typically surface as "type not found" or "object inactive" during pull.

## Namespace encoding in filenames

| On disk                         | SAP technical name      | Notes                                              |
| ------------------------------- | ----------------------- | -------------------------------------------------- |
| `#igz#bas_hu.clas.abap`         | `/IGZ/BAS_HU`           | `#ns#` form — preferred, filesystem-safe everywhere |
| `(igz)bas_hu.clas.abap`         | `/IGZ/BAS_HU`           | `(ns)` form — older, fragile on case-insensitive FS |

The `#ns#` encoding is the modern default; older repos may still contain `(ns)` filenames. Do not "fix" either — abapGit relies on the exact pattern to map back to `/NS/` namespaces.

## ABAP File Format (AFF — JSON-based)

abapGit ≥ 1.140 and SAP-native gCTS use the **ABAP File Format**, a JSON-based replacement for the XML metadata files. AFF separates header metadata from object payload and is forward-compatible with newer object types (CDS, BDL, service definitions).

```json
{
  "formatVersion": "1",
  "header": {
    "description": "Sales order factory",
    "originalLanguage": "E"
  },
  "class": {
    "category": "00",
    "exposure": "2",
    "final": true
  }
}
```

> [!info] AFF availability
> AFF is the default for new object types (CDS view entities, behavior definitions, service definitions/bindings). Classic objects retain XML metadata for backward compatibility. Mixed repos are normal.

## Sources

- [[sources/heinemann-ewm-coding-standards]] — `01-naming-conventions.md` §3, §7, §8; `02-coding-guidelines.md` G8.
- [[sources/opencode-abap-context-library]] — `abapgit-integration.md` (encoding rules, deserialization phases, namespace encoding, AFF JSON).
- [abapGit official docs](https://docs.abapgit.org/) *(future ingest)*.
