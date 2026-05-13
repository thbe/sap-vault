---
title: "abaplint"
type: entity
entity_kind: tool
sap_object_name: "abaplint"
tags: [tool, lint, abap, open-source, ci, abapgit]
sap_release: ["any (works on serialized abapGit code without SAP system)"]
status: stub
sources:
  - "[[sources/clean-abap-styleguide]]"
related:
  - "[[entities/tools/abapgit]]"
  - "[[concepts/abapgit-serialization]]"
  - "[[concepts/clean-abap]]"
  - "[[entities/tools/code-pal-for-abap]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-13
updated: 2026-05-13
---

# abaplint

> Open-source TypeScript reimplementation of the ABAP parser. Runs **without an SAP system** against [[entities/tools/abapgit|abapGit]]-serialized code. Enables true CI-on-GitHub-Actions, formatting checks, and a portable subset of Clean ABAP enforcement.

## Status

Stub — pointer page. To expand beyond stub: ruleset authoring (`abaplint.json`), GitHub Actions integration recipe, comparison with code pal for ABAP (in-system) and abapOpenChecks, formatter usage, editor integrations (VS Code language server).

## Repository

- **GitHub**: <https://github.com/abaplint/abaplint>
- **Docs**: <https://rules.abaplint.org>
- **License**: MIT

## What makes it different

| Property              | abaplint                              | code pal / abapOpenChecks (SCI)         |
| --------------------- | ------------------------------------- | --------------------------------------- |
| Where it runs         | Anywhere (Node.js)                    | Inside SAP system only                  |
| Input                 | abapGit-serialized files              | Live ABAP repository                    |
| Editor integration    | VS Code, others (LSP)                 | ADT only                                |
| CI integration        | GitHub Actions / Jenkins / any CI     | Requires SAP system in pipeline (gCTS)  |
| Formatting            | Yes (built-in formatter)              | No                                      |
| Coverage of Clean ABAP | Subset (formatting + structural)     | Comprehensive (SAP-authored)            |

Best used **in addition to** SCI/ATC, not as a replacement — its independence from a SAP system enables fast pre-commit feedback that ATC cannot.

## Related

- [[entities/tools/abapgit]] — abaplint operates on abapGit's file output.
- [[concepts/abapgit-serialization]] — the file format abaplint parses.
- [[concepts/clean-abap]] — the rule set abaplint partially enforces.
- [[entities/tools/code-pal-for-abap]] — in-system sister tool.
- [[topics/abap-quality-gates]] — pipeline integration target.

## Sources

- [[sources/clean-abap-styleguide]] — `#how-to-check-automatically` describes abaplint as "an open source reimplementation of the ABAP parser. It works without a SAP system and is meant to be used on code serialized using abapGit."
