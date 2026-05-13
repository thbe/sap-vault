# Wiki Index

_Last updated: 2026-05-13 · 51 pages_

> Catalog of every page in `wiki/`. Maintained by the LLM. Grouped by category.
> Format: `- [[path/to/page|Title]] — one-line summary [status]`

## Concepts
- [[wiki/concepts/abap-exception-handling|ABAP Exception Handling]] — Class-based `YCX_*` hierarchy, TRY/CATCH templates, CX_*_CHECK semantics, `RAISE EXCEPTION NEW` [stable]
- [[wiki/concepts/abapgit-serialization|abapGit Serialization]] — FULL vs PREFIX, `.abapgit.xml`, deserialization phases, encoding rules, AFF JSON [stable]
- [[wiki/concepts/abap-naming-conventions|ABAP Naming Conventions]] — Y/Z split, package + type prefixes; object format strings; Hungarian-notation tension [stable]
- [[wiki/concepts/behavior-definition|Behavior Definition (BDL)]] — RAP controller layer: managed/unmanaged, strict(2), draft, determinations/validations/side-effects [stable]
- [[wiki/concepts/cds-view-entities|CDS View Entities]] — RAP data-model layer: root/projection views, annotations, layered modelling [stable]
- [[wiki/concepts/clean-abap|Clean ABAP]] — Modern, OO, readable ABAP style baseline; FINAL by default, CX hierarchy, Hungarian tension [stable]
- [[wiki/concepts/eml|EML (Entity Manipulation Language)]] — ABAP-internal API for RAP: READ/MODIFY/EXECUTE/COMMIT, %cid, IN LOCAL MODE, failed/reported/mapped [stable]
- [[wiki/concepts/mvc-in-abap|MVC in ABAP]] — Model/View/Controller separation in custom ABAP development; multi-site BAdI controller split [stable]
- [[wiki/concepts/odata-service|OData Service]] — SEGW (legacy) vs RAP service binding (modern); DPC_EXT methods, `$filter`/`$top`/`$skip`, status codes [stable]
- [[wiki/concepts/restful-application-programming-model|RESTful Application Programming Model (RAP)]] — Modern programming model: CDS + BDL + service binding for S/4 + BTP ABAP [stable]
- [[wiki/concepts/sap-fiori-overview|SAP Fiori Overview]] — UI5 + FES landscape, deployment options, decision tree, project structure, security baseline [stable]

## Entities — ABAP
_(empty)_

## Entities — CDS
_(empty)_

## Entities — Fiori
_(empty)_

## Entities — Tools
- [[wiki/entities/tools/abap-unit|ABAP Unit]] — Built-in xUnit-style ABAP test framework; pointer to [[wiki/topics/abap-unit-testing]] [stub]
- [[wiki/entities/tools/abapgit|abapGit]] — Open-source ABAP↔Git bridge for PR-based workflow outside transports [stub]
- [[wiki/entities/tools/abaplint|abaplint]] — Open-source TypeScript ABAP parser; runs without SAP system on abapGit-serialized code [stub]
- [[wiki/entities/tools/atc|ATC (ABAP Test Cockpit)]] — Mandatory pre-transport static-check tool [stub]
- [[wiki/entities/tools/code-inspector|Code Inspector (SCI)]] — Static-check engine and variant authoring under ATC [stub]
- [[wiki/entities/tools/code-pal-for-abap|code pal for ABAP]] — SAP-published open-source SCI check collection automating Clean ABAP rules [stub]
- [[wiki/entities/tools/code-vulnerability-analyzer|Code Vulnerability Analyzer (CVA)]] — Licensed security-focused SCI checks via ATC [stub]
- [[wiki/entities/tools/sat|SAT (ABAP Runtime Trace)]] — Modern statement-level runtime trace; SE30 successor [stub]
- [[wiki/entities/tools/segw|SEGW (SAP Gateway Service Builder)]] — Legacy OData V2 service builder [stub]
- [[wiki/entities/tools/slin|SLIN (Extended Program Check)]] — Deep cross-program syntax/semantics check beyond standard syntax check [stub]
- [[wiki/entities/tools/st05|ST05 (Performance Trace)]] — SQL/RFC/Enqueue/Buffer trace; primary tool for SQL anti-pattern hunting [stub]

## Patterns
- [[wiki/patterns/brf-plus-ui-guard|BRF+ UI Guard]] — Custom UI mode that strips destructive actions from BRF+ Workbench in PRD [stable]
- [[wiki/patterns/brfplus-application-exit|BRF+ Application Exit]] — `CL_FDT_APPLICATION_SETTINGS` subclass injecting custom UI behavior into BRF+ Workbench [stub]
- [[wiki/patterns/clean-abap-idioms|Clean ABAP Idioms]] — Cheatsheet of small Clean ABAP rules: tables, strings, booleans, conditions, control flow [stable]
- [[wiki/patterns/dynamic-code-whitelist|Dynamic Code Whitelist]] — Allow-list remediation for `SUBMIT` / `CALL TRANSACTION` / dynamic generation [stable]
- [[wiki/patterns/form-to-method-refactor|FORM-to-Method Refactor]] — Mechanical recipe + prioritisation for converting FORM subroutines to class methods [stable]
- [[wiki/patterns/multi-site-repo-architecture|Multi-Site Repo Architecture]] — Three-tier YEWM ← ZALM/ZERL + YBC/YCA package layering with import order [stable]
- [[wiki/patterns/oo-design-patterns-in-abap|OO Design Patterns in ABAP]] — Singleton, Factory, Strategy, Observer, Facade, Template Method — ABAP-idiomatic notes [stable]
- [[wiki/patterns/parallel-cursor|Parallel Cursor]] — Replacement for O(n×m) nested-loop anti-pattern using sorted/hashed tables [stub]
- [[wiki/patterns/table-driven-enhancement-framework|Table-Driven Enhancement Framework]] — Config-table-driven BAdI/exit dispatcher (YS00 framework) [stable]
- [[wiki/patterns/test-doubles-and-dependency-injection|Test Doubles & Dependency Injection]] — Constructor/setter/back-door injection; `CL_ABAP_TESTDOUBLE`; CDS test environment [stable]

## Snippets
- [[wiki/snippets/abap-doc-comments|ABAP Doc Comments]] — `"!` syntax with shorttext/parameter/raising/link tags for self-documenting class APIs [stable]
- [[wiki/snippets/abap-modern-syntax-cheatsheet|ABAP Modern Syntax Cheatsheet]] — Prefer/over table for inline declarations, constructor expressions, table expressions, with release gates [stable]
- [[wiki/snippets/flower-box-header|Flower-Box Header]] — Mandatory comment header + modification-log block for custom ABAP [stable]
- [[wiki/snippets/ui5-controller-template|UI5 Controller Template]] — Canonical `sap.ui.define` controller skeleton with onInit, private helper, event handler patterns [stable]

## Topics
- [[wiki/topics/abap-formatting|ABAP Formatting]] — Layout, indentation, alignment, line-break rules; ABAP Formatter usage [stable]
- [[wiki/topics/abap-performance|ABAP Performance]] — DB + internal-table rules; SQL anti-pattern table; modern idioms (`lines( )`, table-expression DEFAULT) [stable]
- [[wiki/topics/abap-quality-gates|ABAP Quality Gates]] — Pre-transport checklist; per-phase ATC variants; PR sizing; abapGit↔GitHub flows [stable]
- [[wiki/topics/abap-security-checklist|ABAP Security Checklist]] — 8 security anti-patterns with severity, remediation, scan playbook [stable]
- [[wiki/topics/abap-unit-testing|ABAP Unit Testing]] — Test class skeleton, RISK LEVEL/DURATION, fixtures, Given-When-Then, CDS test environment [stable]
- [[wiki/topics/odata-security|OData Security]] — Auth, CSRF, CORS, AUTHORITY-CHECK split by on-prem FES vs BTP ABAP [stub]
- [[wiki/topics/odata-v4-vs-v2|OData V4 vs V2]] — Cheat sheet driving SEGW-vs-RAP and Fiori Elements V2-vs-V4 decisions [stub]
- [[wiki/topics/sap-forms-strategy|SAP Forms Strategy]] — BRF+ Adobe → XML → Custom Adobe → Smart → SAPScript decision hierarchy [stable]
- [[wiki/topics/ui5-coding-standards|UI5 Coding Standards]] — JS critical rules, anti-patterns, module pattern, formatting, Hungarian-notation note [stable]
- [[wiki/topics/ui5-css-guidelines|UI5 CSS Guidelines]] — BEM naming, theme parameters (`--sapNegativeColor`), override safety, manifest registration [stable]

## Sources
- [[wiki/sources/clean-abap-styleguide|SAP Clean ABAP styleguide]] — Official SAP-published Clean ABAP guide; ~150 rules across naming, language, classes, methods, errors, comments, formatting, testing [stable]
- [[wiki/sources/heinemann-ewm-coding-standards|Heinemann EWM Coding Standards]] — S/4HANA 2023 FPS04 EWM migration project standards (1,451 lines across 3 docs) [stable]
- [[wiki/sources/opencode-abap-context-library|opencode ABAP Context Library]] — Personal LLM context library, 12 files (~80 KB) covering modern ABAP, RAP, testing, abapGit, code review [stable]
- [[wiki/sources/opencode-fiori-context-library|opencode Fiori Context Library]] — Personal LLM context library, 2 files (~9 KB) covering UI5 JS standards, CSS/BEM, SEGW workflow, deployment, security [stable]
- [[wiki/sources/sap-development-standard-approach-abap-fiori-v1|SAP Development Standards & Approach — ABAP and Fiori v1]] — Heinemann S/4 upgrade project standard, T. Bendler 2023-03-31 [stable]
