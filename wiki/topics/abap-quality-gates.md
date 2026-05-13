---
title: "ABAP Quality Gates"
type: topic
tags: [quality, atc, code-inspector, governance, abap]
sap_release: ["S/4HANA", "BTP ABAP", "NetWeaver 7.40+"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/heinemann-ewm-coding-standards]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/abap-naming-conventions]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-performance]]"
  - "[[topics/abap-security-checklist]]"
  - "[[patterns/dynamic-code-whitelist]]"
  - "[[patterns/form-to-method-refactor]]"
created: 2026-05-11
updated: 2026-05-13
---

# ABAP Quality Gates

> The concrete checks and tools every piece of custom ABAP must pass before being released into a transport.

## Why this matters

Standards on paper achieve nothing unless enforced. The Heinemann standard ties quality to a small set of mandatory tooling steps so reviewers and auditors can verify compliance objectively rather than by taste.

## Core concepts

- [[concepts/clean-abap]]
- [[concepts/abap-naming-conventions]]

## Practical guidance

### Pre-transport checklist

1. **Functional and technical design** complete and reviewed (no transports without spec).
2. **Authorization concept** in place — every custom transaction has an authorization check.
3. **Extended Program Check** clean (`SLIN`).
4. **Code Inspector** clean against the project SCI variant (e.g. `Y_NAMING_CONVENTIONS`).
5. **ATC** clean against the project check variant.
6. **Naming conventions** verified (Code Inspector covers most; reviewer confirms package + transport).
7. **Flower-box header** present on every program/method/enhancement (see [[snippets/flower-box-header]]).
8. **Modification log** appended for any change to existing code.
9. **Transport description** follows project format; transport check passed.

### What ATC / SCI catches

- Naming convention violations (variable prefixes, repository naming).
- Performance smells: `SELECT *`, nested SELECT, missing FAE empty-check.
- Use of obsolete language constructs.
- Missing `WHERE` clauses, unbounded reads.
- Hardcoded literals where constants are required.

### What it doesn't catch

- Architectural smell (god class, layering violations).
- Missing tests (unless ABAP Unit coverage is a configured check).
- Bad business logic — design review still needed.

## Patterns & anti-patterns

- Skipping ATC "because it's a small change" — small changes are exactly when regressions hide.
- Suppressing ATC findings without a documented exemption.
- Running checks only at transport release (too late) — run continuously in ADT.

## Tools

- [[entities/tools/atc]] — ABAP Test Cockpit.
- *(future)* `entities/tools/code-inspector` — `SCI`/`SCII`.
- *(future)* `entities/tools/slin` — Extended Program Check.
- *(future)* `entities/tools/abap-unit` — testing harness.

## Related topics

- [[topics/abap-performance]]
- [[topics/abap-security-checklist]] — the security-specific gate set.

## EWM project gates (S/4HANA 2023 FPS04 migration)

The Heinemann EWM migration ([[sources/heinemann-ewm-coding-standards]]) layers concrete per-phase gates on top of the generic checklist:

### Per-phase ATC variants

| Phase             | ATC variant                              | Focus                                              |
| ----------------- | ---------------------------------------- | -------------------------------------------------- |
| Inventory scan    | `YEWM_ATC_BASELINE`                      | Snapshot — nothing failed, just counted.           |
| Active migration  | `YEWM_ATC_MIGRATION`                     | New errors blocked; existing baseline grandfathered.|
| Pre-cutover       | `YEWM_ATC_RELEASE`                       | Full strictness; no exemptions without sign-off.   |

### Mandatory checks

- **ABAP Unit**: any class touched in a transport must have ≥1 unit test added/updated; new classes ship with tests. Coverage tracked but not yet a hard %-gate.
- **Security scan**: full [[topics/abap-security-checklist]] sweep before release; dynamic-code whitelist ([[patterns/dynamic-code-whitelist]]) reviewed.
- **FORM tracking**: each refactored FORM ([[patterns/form-to-method-refactor]]) logged so reviewers can target the diff.
- **Performance baseline**: RF dialog round-trip ≤2 s; new selects benchmarked against ST05 baseline. See [[topics/abap-performance]].
- **ABAP doc comments**: all public methods of new classes carry `"!` doc headers ([[snippets/abap-doc-comments]]).

### Exemption process

- Suppressions only via pseudo-comment `"#EC` with linked Jira ticket and reviewer initials in code comment.
- Exemption reviewed quarterly; expired exemptions trigger ATC failure on next run.

## Code review specifics

The opencode context library codifies concrete review-flow expectations on top of the static-check gates above.

### PR / transport sizing

- **Transport-sized PRs**: one transport = one PR. Don't bundle unrelated work.
- **Target <400 lines diff**; hard ceiling 800. Larger PRs slip review depth and lengthen turnaround.
- **<1 day turnaround** for review. Stale PRs accumulate merge conflicts and lose context.
- **Reviewer should be able to run the change locally** — no environment-specific assumptions in the diff.

### Review checklist

1. **Tests**: any class touched ships ≥1 ABAP Unit; new classes ship with tests + ≥80 % coverage on the touched method. See [[topics/abap-unit-testing]].
2. **Naming**: prefixes match the [[concepts/abap-naming-conventions]] table.
3. **Exceptions**: class-based, correct `CX_*` parent. No `EXCEPTIONS OTHERS = 0`. See [[concepts/abap-exception-handling]].
4. **Performance**: no `SELECT *`, no nested SELECT, FAE driver checked, table types chosen deliberately. See [[topics/abap-performance]].
5. **Documentation**: public methods carry `"!` ABAP doc; non-trivial logic has a `"` line comment explaining *why*.
6. **No commented-out code** — Git is the history.

### Modern flows (abapGit ↔ GitHub)

| Flow            | Direction                  | Use case                                         |
| --------------- | -------------------------- | ------------------------------------------------ |
| **One-way push**| ABAP → GitHub              | DEV system serializes; GitHub is the source of truth for review (read-only mirror). |
| **Two-way sync**| ABAP ↔ GitHub              | Branches developed externally pulled back into DEV via abapGit. Requires strict pull/activation discipline. |

Two-way sync only works with a stable [[concepts/abapgit-serialization]] (PREFIX folder logic, AFF for new object types, no encoding drift).

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 2 (Quality), §4.6 (Governance).
- [[sources/heinemann-ewm-coding-standards]] — `03-code-quality-performance.md` §"Quality gates" and §"ATC variants".
- [[sources/opencode-abap-context-library]] — `abap-code-review-guide.md` (PR sizing, review checklist, abapGit ↔ GitHub flows), `abapgit-integration.md`.
