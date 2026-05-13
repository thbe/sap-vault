# Wiki Index

_Last updated: 2026-05-11 · 34 pages_

> Catalog of every page in `wiki/`. Maintained by the LLM. Grouped by category.
> Format: `- [[path/to/page|Title]] — one-line summary [status]`

## Concepts
- [[wiki/concepts/abap-exception-handling|ABAP Exception Handling]] — Class-based `YCX_*` hierarchy, TRY/CATCH templates, CX_*_CHECK semantics, `RAISE EXCEPTION NEW` [draft]
- [[wiki/concepts/abapgit-serialization|abapGit Serialization]] — FULL vs PREFIX, `.abapgit.xml`, deserialization phases, encoding rules, AFF JSON [draft]
- [[wiki/concepts/abap-naming-conventions|ABAP Naming Conventions]] — Y/Z split, package + type prefixes; object format strings; Hungarian-notation tension [draft]
- [[wiki/concepts/behavior-definition|Behavior Definition (BDL)]] — RAP controller layer: managed/unmanaged, strict(2), draft, determinations/validations/side-effects [draft]
- [[wiki/concepts/cds-view-entities|CDS View Entities]] — RAP data-model layer: root/projection views, annotations, layered modelling [draft]
- [[wiki/concepts/clean-abap|Clean ABAP]] — Modern, OO, readable ABAP style baseline; FINAL by default, CX hierarchy, Hungarian tension [draft]
- [[wiki/concepts/eml|EML (Entity Manipulation Language)]] — ABAP-internal API for RAP: READ/MODIFY/EXECUTE/COMMIT, %cid, IN LOCAL MODE, failed/reported/mapped [draft]
- [[wiki/concepts/mvc-in-abap|MVC in ABAP]] — Model/View/Controller separation in custom ABAP development; multi-site BAdI controller split [draft]
- [[wiki/concepts/odata-service|OData Service]] — SEGW (legacy) vs RAP service binding (modern); DPC_EXT methods, `$filter`/`$top`/`$skip`, status codes [draft]
- [[wiki/concepts/restful-application-programming-model|RESTful Application Programming Model (RAP)]] — Modern programming model: CDS + BDL + service binding for S/4 + BTP ABAP [draft]
- [[wiki/concepts/sap-fiori-overview|SAP Fiori Overview]] — UI5 + FES landscape, deployment options, decision tree, project structure, security baseline [draft]

## Entities — ABAP
_(empty)_

## Entities — CDS
_(empty)_

## Entities — Fiori
_(empty)_

## Entities — Tools
- [[wiki/entities/tools/atc|ATC (ABAP Test Cockpit)]] — Mandatory pre-transport static-check tool [stub]
- [[wiki/entities/tools/segw|SEGW (SAP Gateway Service Builder)]] — Legacy OData V2 service builder [draft]

## Patterns
- [[wiki/patterns/brf-plus-ui-guard|BRF+ UI Guard]] — Custom UI mode that strips destructive actions from BRF+ Workbench in PRD [draft]
- [[wiki/patterns/dynamic-code-whitelist|Dynamic Code Whitelist]] — Allow-list remediation for `SUBMIT` / `CALL TRANSACTION` / dynamic generation [draft]
- [[wiki/patterns/form-to-method-refactor|FORM-to-Method Refactor]] — Mechanical recipe + prioritisation for converting FORM subroutines to class methods [draft]
- [[wiki/patterns/multi-site-repo-architecture|Multi-Site Repo Architecture]] — Three-tier YEWM ← ZALM/ZERL + YBC/YCA package layering with import order [draft]
- [[wiki/patterns/oo-design-patterns-in-abap|OO Design Patterns in ABAP]] — Singleton, Factory, Strategy, Observer, Facade, Template Method — ABAP-idiomatic notes [draft]
- [[wiki/patterns/table-driven-enhancement-framework|Table-Driven Enhancement Framework]] — Config-table-driven BAdI/exit dispatcher (YS00 framework) [draft]
- [[wiki/patterns/test-doubles-and-dependency-injection|Test Doubles & Dependency Injection]] — Constructor/setter/back-door injection; `CL_ABAP_TESTDOUBLE`; CDS test environment [draft]

## Snippets
- [[wiki/snippets/abap-doc-comments|ABAP Doc Comments]] — `"!` syntax with shorttext/parameter/raising/link tags for self-documenting class APIs [stable]
- [[wiki/snippets/abap-modern-syntax-cheatsheet|ABAP Modern Syntax Cheatsheet]] — Prefer/over table for inline declarations, constructor expressions, table expressions, with release gates [stable]
- [[wiki/snippets/flower-box-header|Flower-Box Header]] — Mandatory comment header + modification-log block for custom ABAP [stable]
- [[wiki/snippets/ui5-controller-template|UI5 Controller Template]] — Canonical `sap.ui.define` controller skeleton with onInit, private helper, event handler patterns [stable]

## Topics
- [[wiki/topics/abap-performance|ABAP Performance]] — DB + internal-table rules; SQL anti-pattern table; modern idioms (`lines( )`, table-expression DEFAULT) [draft]
- [[wiki/topics/abap-quality-gates|ABAP Quality Gates]] — Pre-transport checklist; per-phase ATC variants; PR sizing; abapGit↔GitHub flows [draft]
- [[wiki/topics/abap-security-checklist|ABAP Security Checklist]] — 8 security anti-patterns with severity, remediation, scan playbook [draft]
- [[wiki/topics/abap-unit-testing|ABAP Unit Testing]] — Test class skeleton, RISK LEVEL/DURATION, fixtures, Given-When-Then, CDS test environment [draft]
- [[wiki/topics/sap-forms-strategy|SAP Forms Strategy]] — BRF+ Adobe → XML → Custom Adobe → Smart → SAPScript decision hierarchy [draft]
- [[wiki/topics/ui5-coding-standards|UI5 Coding Standards]] — JS critical rules, anti-patterns, module pattern, formatting, Hungarian-notation note [draft]
- [[wiki/topics/ui5-css-guidelines|UI5 CSS Guidelines]] — BEM naming, theme parameters (`--sapNegativeColor`), override safety, manifest registration [draft]

## Sources
- [[wiki/sources/heinemann-ewm-coding-standards|Heinemann EWM Coding Standards]] — S/4HANA 2023 FPS04 EWM migration project standards (1,451 lines across 3 docs) [stable]
- [[wiki/sources/opencode-abap-context-library|opencode ABAP Context Library]] — Personal LLM context library, 12 files (~80 KB) covering modern ABAP, RAP, testing, abapGit, code review [stable]
- [[wiki/sources/opencode-fiori-context-library|opencode Fiori Context Library]] — Personal LLM context library, 2 files (~9 KB) covering UI5 JS standards, CSS/BEM, SEGW workflow, deployment, security [stable]
- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1|SAP Development Standards & Approach — ABAP and Fiori v1]] — Heinemann S/4 upgrade project standard, T. Bendler 2023-03-31 [stable]
