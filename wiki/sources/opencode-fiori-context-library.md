---
title: "opencode Fiori Context Library (personal LLM context)"
type: source
tags: [fiori, ui5, javascript, css, odata, segw, llm-context]
source_kind: code
author: "Thorsten Bendler (personal curation; underlying material from SAP Fiori design guidelines, Heinemann project standards, classic UI5 best-practice docs)"
publication_date: 2026-05-11
url: "git@github.com:thbe/dotfiles — opencode/.config/opencode/context/languages/fiori/"
raw_path: "private (gitignored — lives in dotfiles repo, not in raw/)"
sap_release_discussed: ["S/4HANA (FES embedded)", "UI5 1.x classic freestyle"]
status: stable
related:
  - "[[concepts/sap-fiori-overview]]"
  - "[[concepts/odata-service]]"
  - "[[topics/ui5-coding-standards]]"
  - "[[topics/ui5-css-guidelines]]"
  - "[[snippets/ui5-controller-template]]"
created: 2026-05-11
updated: 2026-05-11
---

# opencode Fiori Context Library (personal LLM context)

## TL;DR

A 2-file, ~9 KB personal LLM context library covering **classic freestyle SAP Fiori/UI5 development**: JavaScript coding standards, CSS/BEM conventions, SEGW OData V2 workflow, deployment topology, app-selection decision tree, and security basics. The companion to the previously-ingested [[sources/opencode-abap-context-library]] for the front-end side. Smaller and narrower than the ABAP library — does not cover RAP service bindings, Fiori Elements, TypeScript-based UI5, or modern `ui5-tooling`. Modernises and adds concrete coding-level detail to the Heinemann standard's Fiori chapter ([[sources/sap-development-standard-approach-abap-fiori-v1]]).

## Bibliographic info

- **Author**: Thorsten Bendler — personal curation.
- **Underlying sources**: SAP Fiori design guidelines, internal Heinemann project standards, classic UI5 best-practice references.
- **Date snapshot**: 2026-05-11 (the version read for this ingest).
- **Format**: 2 Markdown files in a dotfiles repo, loaded by opencode based on a `quick-ref.md` context-loading table.
- **Raw path**: outside the vault — `dotfiles/opencode/.config/opencode/context/languages/fiori/`. Not copied into `raw/` (would duplicate the dotfiles repo).

## The 2 files

| File                   | Bytes | Subject                                                                                |
| ---------------------- | ----- | -------------------------------------------------------------------------------------- |
| `quick-ref.md`         | 1.6 K | DO/DON'T list, controller template, context-loading table.                             |
| `fiori-guidelines.md`  | 7.0 K | Architecture, project structure, OData/SEGW workflow, JS rules, CSS/BEM, security.     |

## Key claims

### Architecture & deployment

- **Embedded FES**: ABAP Front-End Server and Back-End Server on the same instance (typical S/4 deployment).
- **Landscape**: SAP Web Dispatcher → SAP Gateway (OData) → ABAP Server (business logic).
- **IDE**: SAP Web IDE (deprecated) **or** SAP Business Application Studio (BAS, current).
- **Prototyping**: SAP Build for screen mock-ups before development.
- **App-selection decision tree** (in order): standard Fiori app → extend standard → custom UI5 → SAP GUI for HTML (last resort).

### Project structure & naming

- **XML views mandatory** (`*.view.xml`); JS views only in rare dynamic cases.
- **Folder layout**: `webapp/{controller,view,i18n,css,model}/`.
- **View names**: PascalCase (`CustomerProfile.view.xml`).
- **Namespaces**: project convention (e.g., `y.fiori.app.*`); **must not** start with `sap` or `new`.
- **Controllers** in their own folder; filename matches view (`CustomerProfile.controller.js`).

### OData / SEGW

- **SEGW workflow**: project → import DDIC structures → generate runtime → implement in `*_DPC_EXT`:
  - `GET_ENTITY` (read single), `GET_ENTITYSET` (read list), `CREATE_ENTITY`, `UPDATE_ENTITY`, `DELETE_ENTITY`.
- **Field naming**: keep ABAP upper-case names (`MATNR`) for automatic backend mapping. (Note: this contradicts the Heinemann standard's "use semantic English property names like `FirstName`" — see contradictions below.)
- **`$filter` always implemented** in `GET_ENTITYSET`; **`$top`/`$skip`** for paging.
- **Test** with `/IWFND/GW_CLIENT` before UI integration; verify status codes (200/201/204).

### JavaScript / UI5 critical rules

- **No global variables** — store in `sap.ui.getCore().getModel()` or component/controller instance.
- **No private member access** — never touch `oControl._oScroller`-style internals; they change on upgrade.
- **No `console.log`** in production code.
- **No DOM manipulation via jQuery** on UI5 controls — use the control API (`setText()`, `setVisible()`).
- **Strict equality**: `===`, `!==` only.
- **No `setTimeout`** for async — use Promises / `attachEventOnce`.
- **View-relative ID access**: `this.byId("x")`, never `jQuery("#x")` (IDs are mangled).
- **`sap.ui.define`** for module dependencies; no global namespace access.
- **`let`/`const`** preferred over `var` (per `quick-ref.md`).
- **Hungarian notation** for JS variables (`sText`, `oModel`, `aItems`) per `quick-ref.md`.

### CSS

- **File**: `webapp/css/style.css`, registered in `manifest.json` under `resources.css`.
- **BEM naming**: classes use hyphens (`.profile-header`, `.profile-header--large`); IDs use underscores.
- **Never override SAP classes globally** — `.sapMBtn { ... }` affects every button. Always namespace via custom class on the specific control: `myButton.addStyleClass("myCustomButton")` then target `.myCustomButton .sapMBtnInner`.
- **No inline `style="..."`** attributes.
- **Shorthand properties** preferred (`margin: 10px 5px;`).
- **Theme parameters** (`@sapUiBrand`) over hard-coded hex colors.

### Security

- **XSS**: standard UI5 controls (`sap.m.Text` etc.) handle output encoding automatically.
- **`sap.ui.core.HTML`**: dangerous — sanitise content before setting.
- **All user input** validated and sanitised before sending to backend.
- **HTTPS only**; respect CORS / Same-Origin.

## Code samples worth keeping

Extracted as a dedicated snippet:

- [[snippets/ui5-controller-template]] — `sap.ui.define` controller skeleton with `onInit`, private helper, and event handler patterns from `quick-ref.md`.

Inline patterns kept in the topic pages:

- DO/DON'T comparisons in [[topics/ui5-coding-standards]].
- BEM CSS examples + global-override anti-pattern in [[topics/ui5-css-guidelines]].

## Pages this source touches

**New pages created from this source**:

- [[topics/ui5-coding-standards]]
- [[topics/ui5-css-guidelines]]
- [[snippets/ui5-controller-template]]

**Existing pages expanded**:

- [[concepts/sap-fiori-overview]] — promoted from stub; added UI5 coding rules summary, decision tree, deployment topology.
- [[concepts/odata-service]] — added SEGW DPC_EXT method workflow, `$filter`/`$top`/`$skip`, GW_CLIENT testing, status codes.

**Existing pages confirmed (second source backref added)**:

- [[entities/tools/segw]] — DPC_EXT method list + GW_CLIENT testing reconfirmed.

## Open questions / to verify

- **OData property naming standard** — this source says "use ABAP upper-case names like `MATNR`" but the Heinemann standard (§5.4) prescribes "semantic English names like `FirstName`". Reconcile against current SAP / project policy. Documented as a `> [!warning] Contradiction` callout on [[concepts/odata-service]].
- **Hungarian notation in UI5 JS** — community-standard for UI5 1.x freestyle, but TypeScript-based UI5 development de-emphasises it. Worth re-evaluating once a TypeScript-UI5 source is ingested.
- **`jQuery.sap.byId()`** — mentioned in `quick-ref.md` as DOM access; **deprecated since UI5 1.58, removed in newer versions**. The `fiori-guidelines.md` correctly prescribes `this.byId()`. Source self-contradicts; flagged as a `> [!note]` on [[topics/ui5-coding-standards]].
- **`Use sap.* namespace only`** in `quick-ref.md` — misleading; `fiori-guidelines.md` correctly says **don't start namespace with `sap`**. Source self-contradicts; flagged on [[concepts/sap-fiori-overview]].
- **Web IDE references** — Web IDE is sunset; modernise to BAS + `ui5-tooling`.
- **Fiori Elements** (List Report, Object Page) — not covered. Major gap for modern S/4 development.
- **TypeScript-based UI5** — not covered.

## Contradictions with other sources

### Internal contradictions (within this source)

- **`jQuery.sap.byId()` (quick-ref) vs `this.byId()` (guidelines)** — quick-ref advice is dated; the guidelines doc is correct. Flagged as a `> [!note]` on [[topics/ui5-coding-standards]]; resolution: use `this.byId()`.
- **"Use sap.* namespace only" (quick-ref) vs "Don't start namespace with sap" (guidelines)** — quick-ref is misleading. Flagged on [[concepts/sap-fiori-overview]]; resolution: use project namespace (e.g., `y.fiori.*`); use `sap.ui.define` for module imports.

### Cross-source

- **OData property naming** — this source ("`MATNR` upper-case for auto-mapping") vs [[sources/sap-development-standard-approach-abap-fiori-v1]] §5.4 ("English semantic names like `FirstName`, UpperCamelCase"). Both are defensible:
  - Auto-mapping is faster to develop and avoids hand-coded conversion in DPC.
  - Semantic English names are more consumer-friendly and decouple the OData contract from DDIC.
  - **Resolution pending** — likely depends on whether the service is internal-only or also consumed by partners. Documented as `> [!warning] Contradiction` callout on [[concepts/odata-service]] until project policy is confirmed.
- **Hungarian notation** — same pragmatic-vs-purist tension already documented on [[concepts/abap-naming-conventions]] and [[concepts/clean-abap]] for the ABAP side. UI5 community standard is to keep Hungarian for JS variables; flagged as a `> [!note]` on [[topics/ui5-coding-standards]] rather than a contradiction.
