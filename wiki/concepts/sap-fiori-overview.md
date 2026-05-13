---
title: "SAP Fiori Overview"
type: concept
tags: [fiori, ui5, frontend, s4hana]
sap_release: ["S/4HANA (FES embedded or hub)", "BTP"]
status: stable
sources:
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[sources/opencode-fiori-context-library]]"
related:
  - "[[concepts/odata-service]]"
  - "[[concepts/restful-application-programming-model]]"
  - "[[topics/ui5-coding-standards]]"
  - "[[topics/ui5-css-guidelines]]"
  - "[[snippets/ui5-controller-template]]"
created: 2026-05-11
updated: 2026-05-13
---

# SAP Fiori Overview

> SAP's design system + technology stack for modern business apps. UI built with SAPUI5, served by an ABAP Front-End Server (FES) — embedded in S/4HANA or run as a standalone hub — consuming OData services from the back end.

## Architecture & landscape

```
[ Browser ]
     │ HTTPS
     ▼
[ SAP Web Dispatcher ]   ← load balancing, request routing
     │
     ▼
[ ABAP Front-End Server (FES) ]   ← UI5 runtime, theming, Launchpad
     │ OData (HTTP)
     ▼
[ SAP Gateway ]   ← OData service registration, $filter/$top/$skip handling
     │
     ▼
[ ABAP Back-End Server (BES) ]   ← business logic, persistence
```

### Deployment options

| Option           | When                                                     | Notes                                                |
| ---------------- | -------------------------------------------------------- | ---------------------------------------------------- |
| **Embedded FES** | Default for S/4HANA single back end                      | FES + BES on the same instance — fewer moving parts  |
| **Hub FES**      | Multi-back-end consolidation (e.g., S/4 + ECC + custom)  | One UI5 runtime, multiple OData destinations         |
| **BTP**          | New apps independent of on-prem ABAP                     | UI5 served from BTP HTML5 Apps; OData via Connectivity |

## Development environment

- **Primary IDE**: **SAP Business Application Studio (BAS)**.
- **Legacy**: SAP Web IDE — deprecated by SAP; do not start new projects there.
- **Mock-ups / prototyping**: **SAP Build** for screen design before development.
- **Theme**: standard SAP Fiori themes (Quartz Light/Dark, Horizon). Custom theming requires explicit approval.

## App-selection decision tree

Evaluate **in this order** before building anything custom:

1. **Standard Fiori app exists?** → Use it.
2. **Standard app extensible?** → Create a Fiori extension.
3. **Full custom UI required?** → Develop a custom UI5 app.
4. **Fiori not viable?** → SAP GUI for HTML (WebGUI) — last resort.

## Project structure (custom UI5 app)

```
webapp/
├── controller/      # JS controllers (PascalCase, .controller.js)
├── view/            # XML views (PascalCase, .view.xml)
├── i18n/            # Translation property files
├── css/             # Custom stylesheet (style.css)
├── model/           # Local JSON models
├── Component.js
├── manifest.json
└── index.html
```

- **Views**: XML mandatory (`*.view.xml`). JS views only for rare dynamic cases.
- **Controllers**: separate folder, filename matches view (`CustomerProfile.controller.js`).

## Naming & namespace

- **Project namespace**: project convention, e.g., `y.fiori.app.<module>`.
- **Reserved**: namespaces **must not** start with `sap` or `new`.
- **View files**: PascalCase (`CustomerProfile.view.xml`).
- **Controllers**: match the view (`CustomerProfile.controller.js`).

> [!note] `Use sap.* namespace only` is misleading
> The `quick-ref.md` of [[sources/opencode-fiori-context-library]] contains the line "Use sap.* namespace only" — this refers to **module imports** via `sap.ui.define(["sap/m/Button"])`, not the **app namespace**, which must NOT start with `sap`. Reserved root namespace.

## Coding standards

Detailed in dedicated topic pages:

- [[topics/ui5-coding-standards]] — JavaScript critical rules, anti-patterns, module pattern.
- [[topics/ui5-css-guidelines]] — BEM naming, theme parameters, override safety, manifest registration.
- [[snippets/ui5-controller-template]] — canonical controller skeleton.

## Pre-requisites for app rollout

Per Heinemann standard ([[sources/sap-development-standard-approach-abap-fiori-v1]] §5.3):

- OData service activated in `/IWFND/MAINT_SERVICE`.
- Catalog assigned to PFCG role.
- Group assigned to catalog.
- End-user authorisation maintained.

## Security baseline

- **HTTPS only** — all communication.
- **XSS** — output encoding handled by standard UI5 controls (`sap.m.Text` etc.). Custom HTML via `sap.ui.core.HTML` requires manual sanitisation.
- **CORS** — respect Same-Origin policy; configure FES side rather than disabling browser checks.
- **Input validation** — both client (UX) and server (security boundary).

## Open topics

- **Fiori Elements** (List Report, Object Page) vs freestyle UI5.
- **TypeScript-based UI5** development with `ui5-tooling`.
- **App deployment** to FES vs BTP (HTML5 Apps).
- **OData V4** + RAP service bindings vs SEGW V2 ([[concepts/odata-service]]).
- **Theming + branding** workflow.

## Sources

- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Ch 5 (architecture, decision tree, project structure, security).
- [[sources/opencode-fiori-context-library]] — `fiori-guidelines.md` §1–2, §6 (deployment, project layout, security).
