---
title: "OData Security"
type: topic
tags: [odata, security, csrf, cors, auth, btp, fiori]
sap_release: ["S/4HANA (on-prem)", "BTP ABAP (Steampunk)"]
status: stub
sources:
  - "[[wiki/sources/sap-development-standard-approach-abap-fiori-v1]]"
  - "[[wiki/sources/opencode-fiori-context-library]]"
related:
  - "[[wiki/concepts/odata-service]]"
  - "[[wiki/concepts/restful-application-programming-model]]"
  - "[[wiki/topics/abap-security-checklist]]"
created: 2026-05-13
updated: 2026-05-13
---

# OData Security

> Authentication, CSRF protection, and CORS handling for OData services — split by deployment surface (on-prem S/4 via FES vs BTP ABAP via Steampunk + Cloud Connector / API Management).

## Status

**Stub.** Listed under "Documented future work" on [[wiki/concepts/odata-service]]. Existing sources document fragments (Heinemann v1 covers FES side; Fiori context library covers UI5 CSRF token flow); a consolidated dedicated source ingestion is pending.

## Dimensions to cover

### Authentication
- **On-prem (FES)**: SAML2 to SAP IdP, X.509, basic (legacy), Kerberos/SPNEGO for SSO.
- **BTP ABAP**: SAP IAS (Identity Authentication Service) → IPS, OAuth2 token flow, communication arrangements / users / scenarios.
- Service-user vs named-user implications for audit trail and authorisations (`AUTHORITY-CHECK`).

### CSRF
- V2 + V4 both require `X-CSRF-Token: Fetch` then echo on writes.
- UI5 handles automatically when using `ODataModel`; manual fetch required for hand-rolled clients.
- Token TTL, session pinning, stateless mode caveats.

### CORS
- Same-origin assumption inside Fiori Launchpad — usually no CORS needed.
- Cross-origin (e.g. external SPA hitting BTP ABAP): configure via SAP API Management / Cloud Connector / `ICF` allowed origins.
- Preflight (`OPTIONS`) and credentials.

### Authorisation
- Cross-link to [[wiki/topics/abap-security-checklist]] for `AUTHORITY-CHECK` patterns inside DPC_EXT and RAP determinations/validations.
- RAP **strict(2)** behavior implications — see [[wiki/concepts/behavior-definition]].

### Transport security
- TLS termination (SAP Web Dispatcher / FES / BTP ingress).
- HSTS, secure-cookie flags.

## To expand beyond stub

- Per-deployment configuration recipes (on-prem FES vs BTP ABAP).
- UI5 client-side patterns (CSRF refresh on 403, retry logic).
- API Management policy templates for OData fronting.
- Audit-log integration.

## Sources

- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1]] — FES-side patterns.
- [[wiki/sources/opencode-fiori-context-library]] — CSRF + UI5 model fragments.
