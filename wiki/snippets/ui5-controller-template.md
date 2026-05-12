---
title: "UI5 Controller Template"
type: snippet
tags: [ui5, fiori, controller, template, sap-ui-define]
sap_release: ["UI5 1.x"]
status: stable
language: js
runnable: true
sources:
  - "[[sources/opencode-fiori-context-library]]"
related:
  - "[[topics/ui5-coding-standards]]"
  - "[[concepts/sap-fiori-overview]]"
created: 2026-05-11
updated: 2026-05-11
---

# UI5 Controller Template

> Canonical `sap.ui.define`-based controller skeleton with `onInit`, private helper, and event handler patterns. Follows the project's UI5 coding standards.

## Skeleton

```javascript
sap.ui.define([
  "sap/ui/core/mvc/Controller",
  "sap/m/MessageToast",
  "sap/ui/model/json/JSONModel"
], function (Controller, MessageToast, JSONModel) {
  "use strict";

  return Controller.extend("y.fiori.app.controller.Main", {

    /* ------------------------------------------------------------------ */
    /*  Lifecycle                                                          */
    /* ------------------------------------------------------------------ */

    onInit: function () {
      this._initializeViewModel();
    },

    onExit: function () {
      // Clean up listeners, intervals, etc.
    },

    /* ------------------------------------------------------------------ */
    /*  Private helpers (underscore prefix)                                */
    /* ------------------------------------------------------------------ */

    _initializeViewModel: function () {
      const oViewModel = new JSONModel({
        busy: false,
        title: ""
      });
      this.getView().setModel(oViewModel, "view");
    },

    /* ------------------------------------------------------------------ */
    /*  Event handlers (on<EventName>)                                     */
    /* ------------------------------------------------------------------ */

    onButtonPress: function (oEvent) {
      const sText = this.byId("input").getValue();
      MessageToast.show("Hello " + sText);
    },

    onSelectionChange: function (oEvent) {
      const oItem = oEvent.getParameter("listItem");
      const oContext = oItem.getBindingContext();
      // ... navigate / load detail / etc.
    }
  });
});
```

## Conventions used

- **`sap.ui.define`** for module dependencies — never global access.
- **`"use strict"`** at the top of the factory.
- **PascalCase** controller name matching its view file.
- **Project namespace** (`y.fiori.app.*`) — never starts with `sap` or `new`.
- **`onInit` / `onExit`** lifecycle hooks defined first.
- **Private helpers** prefixed with `_` — internal to the controller.
- **Event handlers** named `on<EventName>` (e.g., `onButtonPress`, `onSelectionChange`).
- **`this.byId(...)`** for view-relative control access — never `jQuery("#id")` or `sap.ui.getCore().byId(...)`.
- **Hungarian notation**: `sText`, `oItem`, `oContext`, `oViewModel`.
- **`const`** for non-reassigned bindings; **`let`** otherwise; **never `var`**.

## Common extensions

### Component model access

```javascript
const oModel = this.getOwnerComponent().getModel("myService");
```

### Router navigation

```javascript
const oRouter = sap.ui.core.UIComponent.getRouterFor(this);
oRouter.navTo("Detail", { id: "123" });
```

### i18n text resolution

```javascript
const sText = this.getView()
  .getModel("i18n")
  .getResourceBundle()
  .getText("welcomeMessage", [sUserName]);
```

## Sources

- [[sources/opencode-fiori-context-library]] — `quick-ref.md` controller template, `fiori-guidelines.md` §4.
