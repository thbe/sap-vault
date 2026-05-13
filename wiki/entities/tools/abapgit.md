---
title: "abapGit"
type: entity
entity_kind: tool
sap_object_name: "abapGit"
tags: [tool, abapgit, git, transport, abap]
sap_release: ["NetWeaver 7.40+", "S/4HANA", "BTP ABAP (gCTS preferred for cloud)"]
status: stub
sources:
  - "[[wiki/sources/heinemann-ewm-coding-standards]]"
  - "[[wiki/sources/opencode-abap-context-library]]"
related:
  - "[[wiki/concepts/abapgit-serialization]]"
  - "[[wiki/patterns/multi-site-repo-architecture]]"
  - "[[wiki/topics/abap-quality-gates]]"
created: 2026-05-13
updated: 2026-05-13
---

# abapGit

> Open-source ABAP-to-Git bridge. Serialises repository objects (classes, programs, DDIC, etc.) into a Git-friendly format and back. Enables PR-based workflow, branching, and cross-system distribution outside the transport layer.

## Status

**Stub — pointer page.** Serialization mechanics (FULL vs PREFIX, `.abapgit.xml`, deserialization phases, AFF JSON) live on [[wiki/concepts/abapgit-serialization]]; multi-repo monorepo layout lives on [[wiki/patterns/multi-site-repo-architecture]]; gate flows live on [[wiki/topics/abap-quality-gates]].

To expand beyond stub: install/online vs standalone, online vs offline repos, branching strategy with transports, package-to-repo mapping, dependency handling, background mode (`ZABAPGIT_BACKGROUND`), abapGit ↔ gCTS coexistence on BTP.

## Related transactions / programs

- Program `ZABAPGIT` (or `ZABAPGIT_STANDALONE`) — main UI.
- Program `ZABAPGIT_BACKGROUND` — automated push/pull.

## Sources

- [[wiki/sources/heinemann-ewm-coding-standards]]
- [[wiki/sources/opencode-abap-context-library]]
