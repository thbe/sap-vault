---
title: "UI5 CSS Guidelines"
type: topic
tags: [ui5, fiori, css, bem, theming, styling]
sap_release: ["UI5 1.x freestyle", "S/4HANA FES embedded"]
status: draft
sources:
  - "[[sources/opencode-fiori-context-library]]"
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[concepts/sap-fiori-overview]]"
  - "[[topics/ui5-coding-standards]]"
created: 2026-05-11
updated: 2026-05-11
---

# UI5 CSS Guidelines

> Mandatory CSS conventions for SAP Fiori/UI5 apps. Designed to keep custom styling upgrade-safe and theme-compatible.

## Location & registration

- **File**: `webapp/css/style.css` (single project stylesheet, additional files only with justification).
- **Manifest registration** in `manifest.json`:

```json
{
  "sap.ui5": {
    "resources": {
      "css": [
        { "uri": "css/style.css" }
      ]
    }
  }
}
```

## Naming convention — BEM

Use **Block-Element-Modifier**:

- **Classes use hyphens**: `.profile-header`, `.profile-header__avatar`, `.profile-header--large`.
- **IDs use underscores**: `#user_image`.

```css
/* Block */
.profile-header { padding: 1rem; }

/* Element */
.profile-header__avatar { width: 48px; }

/* Modifier */
.profile-header--large { padding: 2rem; }
```

## Override safety — never touch SAP classes globally

> [!warning] Critical
> Overriding standard SAP classes globally affects **every** instance of that control in the app and silently breaks on UI5 upgrades when the underlying DOM changes.

```css
/* BAD — affects every sap.m.Button in the app */
.sapMBtn { border: 1px solid red; }
```

**Correct pattern**: add a project-specific class to the specific control, then scope the override under it.

```javascript
// JS — tag the control
this.byId("submitBtn").addStyleClass("y-submit-button");
```

```css
/* CSS — scope the override */
.y-submit-button .sapMBtnInner {
  border: 1px solid red;
}
```

## Theme parameters — don't hard-code colors

Use SAP UI Theming Framework parameters so the app respects theme switches (Quartz Light / Dark / High Contrast / Belize).

```css
/* BAD — breaks dark theme */
.y-status-error { color: #BB0000; }

/* GOOD — uses semantic theme parameter */
.y-status-error { color: var(--sapNegativeColor); }
```

Common theme parameters:

| Parameter                 | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| `--sapBrandColor`         | Brand accent                              |
| `--sapNegativeColor`      | Error / negative status                   |
| `--sapPositiveColor`      | Success / positive status                 |
| `--sapCriticalColor`      | Warning / critical status                 |
| `--sapNeutralColor`       | Informational                             |
| `--sapBackgroundColor`    | App background                            |
| `--sapTextColor`          | Default text                              |

For LESS-time access (in custom themes) use `@sapBrandColor` etc.

## Best practices

- **No inline styles**: `<Button style="color:red">` is prohibited. Use a class.
- **Shorthand**: `margin: 10px 5px;` not `margin-top: 10px; margin-right: 5px; ...`.
- **Mobile-first**: write base rules, then add `@media` queries for larger viewports.
- **Avoid `!important`** unless overriding a UI5 inline style — and document why.

## Anti-pattern reference

| Anti-pattern                            | Why it fails                                                | Replacement                                              |
| --------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------- |
| Global `.sapMBtn { ... }` override      | Affects all buttons; breaks on upgrade                      | Custom class on the specific control + nested selector   |
| Inline `style="color:red"` on a control | Bypasses theme; can't be overridden by stylesheet           | Add CSS class via `addStyleClass()`                      |
| Hard-coded hex colors                   | Doesn't follow theme; ugly in dark mode                     | Theme parameter (`--sapNegativeColor`)                   |
| `#myButton { ... }`                     | UI5 mangles IDs at runtime                                  | Class-based selector                                     |
| `!important` everywhere                 | Specificity arms race                                       | Increase selector specificity properly                   |

## Sources

- [[sources/opencode-fiori-context-library]] — `fiori-guidelines.md` §5 (CSS Implementation).
- [[sources/sap-development-standard-approach-abap-fiori-v1]] — Heinemann §5.10 (CSS).
