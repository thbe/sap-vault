---
title: "SAP Clean ABAP styleguide"
type: source
tags: [clean-abap, styleguide, abap, governance, primary-source]
sap_release: ["NW 7.40+", "S/4HANA", "BTP ABAP"]
status: stable
sources: []
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[concepts/abap-naming-conventions]]"
  - "[[topics/abap-formatting]]"
  - "[[topics/abap-unit-testing]]"
  - "[[patterns/clean-abap-idioms]]"
  - "[[patterns/test-doubles-and-dependency-injection]]"
  - "[[entities/tools/code-pal-for-abap]]"
  - "[[entities/tools/abaplint]]"
created: 2026-05-13
updated: 2026-05-13
---

# SAP Clean ABAP styleguide

> The official SAP-published Clean ABAP guide — adoption of Robert C. Martin's *Clean Code* for ABAP. The de-facto baseline for new development on S/4HANA and BTP ABAP. Used by SAP's own in-house S/4HANA development teams.

## Bibliographic info

- **Author**: SAP SE (community-maintained on GitHub)
- **URL (canonical)**: <https://github.com/SAP/styleguides/blob/main/clean-abap/CleanABAP.md>
- **Local copy**: `raw/articles/clean-abap-styleguide.md` (5148 lines, 192 KB)
- **Format**: Markdown
- **Fetched**: 2026-05-13
- **License**: Apache 2.0 (per repo)

## TL;DR

Clean ABAP is **the** authoritative style guide for modern ABAP. It systematizes ~150 rules across naming, language constructs, classes, methods, error handling, comments, formatting, and testing. The guidance is opinionated, version-aware (notes which constructs require which NetWeaver release), and explicitly anti-Hungarian-notation. It is the primary source that should anchor every secondary "ABAP standards" document in the wiki.

## Key claims (with section anchors)

### Meta / governance
- **Adoption strategy** (`#how-to-get-started-with-clean-code`, `#how-to-refactor-legacy-code`): four-step plan — get the team aboard, boy-scout rule, build clean islands, talk about it. Don't commit all guidelines at once; start with an undisputed subset.
- **Mind the legacy / mind the performance** (`#mind-the-legacy`, `#mind-the-performance`): Clean ABAP is not religion. Validate against the oldest release you must support; don't optimize prematurely; profile before discarding rules. "We're not suggesting ignoring performance, but rather focus more on clean and well structured code during initial development, and use the profiler to identify critical areas to optimize."
- **Tools for automatic checking** (`#how-to-check-automatically`): code pal for ABAP (SAP), abapOpenChecks (community), abaplint (open-source ABAP parser, runs without SAP system, integrates with abapGit / GitHub Actions).
- **Relation to other guides** (`#how-to-relate-to-other-guides`): respects but is more precise than DSAG ABAP recommendations and SAP's ABAP Programming Guidelines.

### Naming
- **Forbid Hungarian notation and prefixes** (`#avoid-encodings-esp-hungarian-notation-and-prefixes`): no `lv_`, `lt_`, `lo_`, `ls_`, `iv_`, `rv_`, `ev_`, `cv_`. **Unambiguous and primary-source.** Resolves the long-standing tension with secondary sources (Heinemann, opencode libraries) which prescribe Hungarian — those are project-local deviations.
- **Use snake_case** (`#use-snake_case`); avoid abbreviations (`#avoid-abbreviations`); same abbreviations everywhere; nouns for classes, verbs for methods (`#use-nouns-for-classes-and-verbs-for-methods`); avoid noise words ("data", "info", "object").
- **Pattern names only if you mean them** (`#use-pattern-names-only-if-you-mean-them`): don't call a class `*_factory` unless it's actually a factory.

### Language
- **Prefer functional to procedural** (`#prefer-functional-to-procedural-language-constructs`): inline `DATA(x) =`, `VALUE #( FOR … )`, `to_upper(…)`, `+=` over `MOVE`, `TRANSLATE`, `ADD`, `LOOP/INSERT`.
- **Avoid obsolete language elements** (`#avoid-obsolete-language-elements`): use `@`-escaped host variables in Open SQL; consult NW documentation's stable "obsolete" section.
- **Don't use field symbols for dynamic data access** (`#do-not-use-field-symbols-for-dynamic-data-access`): from ABAP Platform 2021, `dref->*` replaces `ASSIGN dref->* TO <fs>`.

### Constants
- **Prefer ENUM to constants interfaces** (`#prefer-enum-to-constants-interfaces`): native `TYPES: BEGIN OF ENUM …` (NW 7.51+) replaces "constants interfaces" anti-pattern.
- **Constants need descriptive names** (`#constants-also-need-descriptive-names`): `status_inactive` not `c_01`.
- **Group constants** when ENUM unavailable (`#if-you-dont-use-enum-or-enumeration-patterns-group-your-constants`).

### Variables, Tables, Strings
- **Prefer inline to up-front declarations** (`#prefer-inline-to-up-front-declarations`).
- **Use the right table type** (`#use-the-right-table-type`); **avoid DEFAULT KEY** (`#avoid-default-key`); **prefer `INSERT INTO TABLE` to `APPEND TO`** (`#prefer-insert-into-table-to-append-to`); **prefer `LINE_EXISTS` to `READ TABLE`** (`#prefer-line_exists-to-read-table-or-loop-at`).
- **String templates with `|…|`** (`#use--to-assemble-text`); backticks for literals (`#use--to-define-literals`).

### Booleans, Conditions, Ifs
- `ABAP_BOOL`, `ABAP_TRUE`/`ABAP_FALSE`, `XSDBOOL` (chapter `#booleans`).
- **Make conditions positive** (`#try-to-make-conditions-positive`); **`IS NOT` over `NOT IS`** (`#prefer-is-not-to-not-is`).
- **`CASE` over `ELSE IF`** (`#prefer-case-to-else-if-for-multiple-alternative-conditions`); **keep nesting depth low** (`#keep-the-nesting-depth-low`).

### Classes
- **`FINAL` if not designed for inheritance** (`#final-if-not-designed-for-inheritance`).
- **Members `PRIVATE` by default, `PROTECTED` only if needed** (`#members-private-by-default-protected-only-if-needed`).
- **Prefer composition to inheritance** (`#prefer-composition-to-inheritance`).
- **Prefer `NEW` to `CREATE OBJECT`** (`#prefer-new-to-create-object`).
- **Multiple static creation methods over optional parameters** (`#prefer-multiple-static-creation-methods-to-optional-parameters`).

### Methods
- **`RETURN`, `EXPORT`, or `CHANGE` exactly one parameter** (`#return-export-or-change-exactly-one-parameter`).
- **Prefer `RETURNING` to `EXPORTING`** (`#prefer-returning-to-exporting`); even for large tables (`#returning-large-tables-is-usually-okay`).
- **Aim for fewer than three importing parameters** (`#aim-for-few-importing-parameters-at-best-less-than-three`).
- **Split methods instead of adding optional parameters** (`#split-methods-instead-of-adding-optional-parameters`).
- **Split methods instead of boolean input** (`#split-method-instead-of-boolean-input-parameter`): a `boolean` flag means two methods.
- **Do one thing, do it well, do it only** (`#do-one-thing-do-it-well-do-it-only`); **keep methods small** (`#keep-methods-small`); **descend one level of abstraction** (`#descend-one-level-of-abstraction`).
- **Fail fast** (`#fail-fast`); **`CHECK` only at method start** (`#check-vs-return`, `#avoid-check-in-other-positions`).

### Error handling
- **Prefer exceptions to return codes** (`#prefer-exceptions-to-return-codes`); **don't let failures slip through** (`#dont-let-failures-slip-through`).
- **`CX_STATIC_CHECK` for manageable** (`#throw-cx_static_check-for-manageable-exceptions`); **`CX_NO_CHECK` for unrecoverable** (`#throw-cx_no_check-for-usually-unrecoverable-situations`); **`CX_DYNAMIC_CHECK` for avoidable** (`#consider-cx_dynamic_check-for-avoidable-exceptions`).
- **Prefer `RAISE EXCEPTION NEW`** (`#prefer-raise-exception-new-to-raise-exception-type`) — NW 7.52+.
- **Wrap foreign exceptions** (`#wrap-foreign-exceptions-instead-of-letting-them-invade-your-code`).

### Comments
- **Express yourself in code, not comments** (`#express-yourself-in-code-not-in-comments`).
- **Comments explain why, not what** (`#write-comments-to-explain-the-why-not-the-what`).
- **Comment with `"`, not `*`** (`#comment-with--not-with-`).
- **No manual versioning** (`#dont-do-manual-versioning`); **no method-signature/end-of comments** (`#dont-add-method-signature-and-end-of-comments`); **delete commented-out code** (`#delete-code-instead-of-commenting-it`).
- **ABAP Doc only for public APIs** (`#abap-doc-only-for-public-apis`).

### Formatting (full chapter, currently undocumented elsewhere in vault)
- **No more than one statement per line** (`#no-more-than-one-statement-per-line`); **reasonable line length** (`#stick-to-a-reasonable-line-length`); **single blank line to separate** (`#add-a-single-blank-line-to-separate-things-but-not-more`).
- **Align assignments to the same object, but not different ones** (`#align-assignments-to-the-same-object-but-not-to-different-ones`).
- **Close brackets at line end** (`#close-brackets-at-line-end`).
- **Single-parameter calls on one line** (`#keep-single-parameter-calls-on-one-line`); **break and indent for multi-parameter** (`#line-break-multiple-parameters`, `#align-parameters`).
- **Use the ABAP Formatter** (`#use-the-abap-formatter-before-activating`); **team's settings, not personal** (`#use-your-teams-abap-formatter-settings`).
- **Don't align type clauses** (`#dont-align-type-clauses`); **don't chain assignments** (`#dont-chain-assignments`).

### Testing (full chapter)
- **Write testable code** (`#write-testable-code`); **enable others to mock you** (`#enable-others-to-mock-you`).
- **Test publics, not private internals** (`#test-publics-not-private-internals`).
- **Don't obsess about coverage** (`#dont-obsess-about-coverage`).
- **Put tests in local classes** (`#put-tests-in-local-classes`); **default test class name = `ltc_*`** (`#call-local-test-classes-by-their-purpose`).
- **CUT for "code under test"** (`#name-the-code-under-test-meaningfully-or-default-to-cut`).
- **Test against interfaces** (`#test-against-interfaces-not-implementations`).
- **Use dependency inversion to inject test doubles** (`#use-dependency-inversion-to-inject-test-doubles`); **`CL_ABAP_TESTDOUBLE`** (`#consider-to-use-the-tool-abap-test-double`); **test seams as temporary workaround** (`#use-test-seams-as-temporary-workaround`); **`LOCAL FRIENDS` for DI constructor access** (`#use-local-friends-to-access-the-dependency-inverting-constructor`).
- **Don't sub-class to mock** (`#dont-sub-class-to-mock-methods`); **don't add features only for tests** (`#dont-add-features-to-production-code-that-are-only-intended-for-use-during-automated-testing`); **don't build test frameworks** (`#dont-build-test-frameworks`).
- **Given-when-then** (`#use-given-when-then`); **"When" is exactly one call** (`#when-is-exactly-one-call`).
- **Few focused assertions** (`#few-focused-assertions`); **assert content, not quantity** (`#assert-content-not-quantity`); **`FAIL` for expected exceptions** (`#use-fail-to-check-for-expected-exceptions`).

## Open questions / things to verify

- **Last-update cadence**: the styleguide is community-maintained on `main` branch; for stability we anchor to "fetched 2026-05-13". A re-fetch every 6-12 months is recommended to pick up new rules.
- **Sub-section files** (`sub-sections/AvoidEncodings.md`, `sub-sections/FunctionGroupsVsClasses.md`, `sub-sections/Enumerations.md`) referenced from the main file are NOT included in the local copy. Fetch separately if deeper detail needed on any specific topic.
- **Cheat-Sheet** (`cheat-sheet/CheatSheet.md`) — print-optimized version. Not yet ingested.

## Reusable code samples extracted

The styleguide contains short illustrative code samples throughout. Reusable patterns extracted to dedicated wiki pages:

- ENUM type declaration → `[[snippets/abap-modern-syntax-cheatsheet]]` (existing) and `[[concepts/clean-abap]]` (new ENUM section).
- Given-when-then test method shape → `[[topics/abap-unit-testing]]` (already covered, refined this ingest).
- Test seams + LOCAL FRIENDS → `[[patterns/test-doubles-and-dependency-injection]]` (cross-link added this ingest).
- Idiomatic small rules (`LINE_EXISTS`, `XSDBOOL`, `IS NOT`, `dref->*`) → `[[patterns/clean-abap-idioms]]` (new this ingest).

## Related

- [[concepts/clean-abap]] — primary consumer; now anchored to this source.
- [[concepts/abap-exception-handling]] — CX_* parent-selection rationale anchored here.
- [[concepts/abap-naming-conventions]] — Hungarian-notation tension formally resolved by this source.
- [[topics/abap-formatting]] — formatting chapter materialized as new page.
- [[topics/abap-unit-testing]] — testing chapter rules cross-referenced.
- [[patterns/clean-abap-idioms]] — small-rule cheatsheet sourced from this guide.
- [[entities/tools/code-pal-for-abap]] — automated check tool referenced here.
- [[entities/tools/abaplint]] — open-source check tool referenced here.

## Sources backing this summary

- Self (primary source).
