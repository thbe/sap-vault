---
title: "UI5 Coding Standards"
type: topic
tags: [ui5, fiori, javascript, coding-standards, anti-patterns]
sap_release: ["UI5 1.x freestyle", "S/4HANA FES embedded"]
status: stable
sources:
  - "[[sources/opencode-fiori-context-library]]"
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[concepts/sap-fiori-overview]]"
  - "[[topics/ui5-css-guidelines]]"
  - "[[snippets/ui5-controller-template]]"
created: 2026-05-11
updated: 2026-05-13
---

# UI5 Coding Standards

> Mandatory JavaScript rules for SAP Fiori/UI5 (classic freestyle). Enforced for upgrade compatibility, performance, and security.

## Critical rules (PROHIBITED)

| # | Rule                                                       | Why                                                          |
| - | ---------------------------------------------------------- | ------------------------------------------------------------ |
| 1 | No global variables                                        | Pollutes namespace; breaks composition                       |
| 2 | No access to `_underscore` members                         | Internal API; changes silently on UI5 upgrade                |
| 3 | No `console.log()` in production                           | Performance + log noise                                      |
| 4 | No jQuery DOM manipulation on UI5 controls                 | Bypasses control lifecycle; breaks rendering                 |
| 5 | No `setTimeout` for async                                  | Race conditions; not cancelable; not awaitable               |
| 6 | No global control lookup (`sap.ui.getCore().byId(...)`)    | Cross-view coupling; brittle                                 |
| 7 | No inline `style="..."` attributes                         | Bypasses theming; impossible to override cleanly             |
| 8 | No global SAP CSS overrides (`.sapMBtn { ... }`)           | Affects every control of that class app-wide                 |

## DO / DON'T table

| Concern              | DON'T                                              | DO                                                                   |
| -------------------- | -------------------------------------------------- | -------------------------------------------------------------------- |
| Equality             | `==`, `!=`                                         | `===`, `!==`                                                         |
| Variable declaration | `var x = ...`                                      | `let` / `const`                                                      |
| Module loading       | Global `sap.m.Button` access                       | `sap.ui.define(['sap/m/Button'], function (Button) { ... })`         |
| Control by ID        | `jQuery("#myButton")`                              | `this.byId("myButton")` or `this.getView().byId("myButton")`         |
| Cross-view access    | `sap.ui.getCore().byId(...)`                       | Component model + event bus                                          |
| Async                | `setTimeout(..., 0)`                               | `Promise`, `attachEventOnce`                                         |
| State                | `var myGlobal = ...`                               | Component/controller instance, JSON model on the component           |
| Logging              | `console.log(...)` in production                   | `sap/base/Log` with severity; remove all debug logs before deploy    |

## Module skeleton (mandatory shape)

```javascript
sap.ui.define([
  "sap/ui/core/mvc/Controller",
  "sap/m/MessageToast"
], function (Controller, MessageToast) {
  "use strict";

  return Controller.extend("y.fiori.app.controller.Main", {
    onInit: function () {
      this._initializeModel();
    },

    _initializeModel: function () {
      const oModel = this.getOwnerComponent().getModel();
      this.getView().setModel(oModel);
    },

    onButtonPress: function (oEvent) {
      const sText = this.byId("input").getValue();
      MessageToast.show("Hello " + sText);
    }
  });
});
```

See full template at [[snippets/ui5-controller-template]].

## Naming

- **Controllers**: `<View>.controller.js` matching `<View>.view.xml` (PascalCase).
- **Namespace**: project-defined (e.g., `y.fiori.app.*`); **must not** start with `sap` or `new`.
- **Hungarian notation** for JS variables: `sText` (string), `oModel` (object), `aItems` (array), `bFlag` (boolean), `iCount` (integer), `fnHandler` (function). UI5 community standard for classic freestyle.

> [!note] Hungarian notation
> The UI5 community has historically standardised on Hungarian-prefixed JS variable names (`sText`, `oModel`, `aItems`). Modern TypeScript-based UI5 development trends away from this since types are explicit. For classic freestyle UI5 1.x as covered by [[sources/opencode-fiori-context-library]] and [[sources/sap-development-standard-approach-abap-fiori-v1]], keep the prefixes — same pragmatic-vs-purist tension noted for ABAP on [[concepts/clean-abap]].

## Formatting

- **Strict equality**: `===`, `!==`.
- **Semicolons**: mandatory.
- **Indent**: tabs or 4 spaces, **consistent** within a project.
- **`"use strict"`** at the top of every module factory.

## Anti-patterns explained

### Global state

```javascript
// BAD — pollutes window/global
var myGlobal = "test";

// GOOD — store on component model
this.getOwnerComponent().getModel("local").setProperty("/myValue", "test");
```

### Private member access

```javascript
// BAD — _oScroller may disappear in the next UI5 release
oList._oScroller.scrollTo(0, 100);

// GOOD — use the public API
oList.scrollToIndex(0);
```

### DOM manipulation

```javascript
// BAD — bypasses control lifecycle, will be re-rendered away
jQuery(this.byId("title").getDomRef()).text("Hello");

// GOOD — control API
this.byId("title").setText("Hello");
```

### Cross-view lookup

```javascript
// BAD — brittle, breaks if view IDs change or component is loaded twice
sap.ui.getCore().byId("__component0---view---btn").setText("Hello");

// GOOD — view-relative
this.byId("btn").setText("Hello");
```

### `setTimeout` for async

```javascript
// BAD — race condition, no error path
setTimeout(function () { doNextThing(); }, 100);

// GOOD — Promise-based
loadData().then(doNextThing).catch(handleError);

// GOOD — wait for an event
this.byId("table").attachEventOnce("updateFinished", doNextThing);
```

## Source self-contradictions to be aware of

> [!note] `jQuery.sap.byId()` is deprecated
> The `quick-ref.md` of [[sources/opencode-fiori-context-library]] mentions `jQuery.sap.byId()` for DOM access. This API is **deprecated since UI5 1.58 and removed in newer versions**. The companion `fiori-guidelines.md` correctly prescribes `this.byId()`. Use `this.byId()`.

> [!note] `Use sap.* namespace only`
> The same `quick-ref.md` says "Use sap.* namespace only" — this is misleading. App namespaces **must not** start with `sap` (reserved by SAP). The advice likely refers to `sap.ui.define` for module imports, which is correct. See [[concepts/sap-fiori-overview]] for project namespace conventions.

## Sources

- [[sources/opencode-fiori-context-library]] — `fiori-guidelines.md` §4 (JS standards), `quick-ref.md` (DO/DON'T list, controller template).
- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Heinemann §5.5–5.13 (project structure, JS, formatting, naming, security).
