---
title: "ABAP Modern Syntax Cheatsheet"
type: snippet
tags: [abap, syntax, clean-abap, modern-abap, cheatsheet]
sap_release: ["NetWeaver 7.40 SP08+", "S/4HANA", "BTP ABAP"]
status: stable
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-performance]]"
language: abap
runnable: false
created: 2026-05-11
updated: 2026-05-11
---

# ABAP Modern Syntax Cheatsheet

> Prefer this over that — a quick lookup for replacing legacy ABAP idioms with modern equivalents. Every modern form requires **NetWeaver 7.40 SP08+** at minimum; constructor expressions need 7.40 SP08, table comprehensions need 7.40 SP05+, CDS-related forms require S/4HANA.

## Prefer → Over

| Modern (✅)                                       | Legacy (❌)                                                       | Why                                            |
| ------------------------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------- |
| `DATA(result) = method( ).`                       | `DATA result TYPE … . result = method( ).`                        | Inline declarations reduce noise               |
| `FINAL(value) = method( ).`                       | `DATA(value) = method( ). " never reassigned`                     | `FINAL` signals immutability                   |
| <code>\|text { var } more\|</code>                | `CONCATENATE 'text ' var ' more' INTO target.`                    | String templates: readable, type-safe          |
| `NEW zcl_class( )`                                | `CREATE OBJECT lo_obj TYPE zcl_class.`                            | Shorter, functional style                      |
| `RAISE EXCEPTION NEW zcx_err( msg = … ).`         | `RAISE EXCEPTION TYPE zcx_err EXPORTING msg = … .`                | Consistent with `NEW` pattern                  |
| `result = method( param = val ).`                 | `CALL METHOD obj->method EXPORTING param = val.`                  | Functional call style                          |
| `xsdbool( condition )`                            | `IF condition. rv = abap_true. ELSE. rv = abap_false. ENDIF.`     | Boolean expression helper                      |
| `CONV typename( value )`                          | `DATA temp TYPE typename. temp = value.`                          | Inline conversion                              |
| `COND #( WHEN x = 1 THEN 'one' ELSE 'other' )`    | `IF x = 1. r = 'one'. ELSE. r = 'other'. ENDIF.`                  | Conditional expressions for assignment         |
| `SWITCH #( v WHEN 1 THEN 'a' WHEN 2 THEN 'b' )`   | `CASE v. WHEN 1. r = 'a'. WHEN 2. r = 'b'. ENDCASE.`              | Switch expressions for assignment              |
| `VALUE #( itab[ key = x ] DEFAULT … )`            | `READ TABLE itab WITH KEY key = x INTO wa. IF sy-subrc = 0 …`     | Table expressions (with DEFAULT to avoid CX)   |
| `FILTER #( itab WHERE field = x )`                | `LOOP AT itab WHERE field = x. APPEND … TO new. ENDLOOP.`         | Filter expressions                             |
| `REDUCE #( INIT s = 0 FOR r IN itab NEXT s = s + r-v )` | `LOOP AT itab. s = s + r-v. ENDLOOP.`                        | Aggregation expression                         |
| `FOR row IN itab ( row-field )`                   | `LOOP AT itab. APPEND … TO new. ENDLOOP.`                         | Table comprehension                            |
| `ENUM` (typed enumeration)                        | `CONSTANTS: BEGIN OF c_status, draft TYPE … VALUE …, …, END OF.`  | Type-safe enumerations (NW 7.51+)              |
| `IF is_valid( )`                                  | `IF is_valid( ) = abap_true`                                      | Predicative method call — cleaner boolean check|
| `lines( itab )`                                   | `DESCRIBE TABLE itab LINES lv_n.`                                 | Built-in function                              |
| `strlen( text )`                                  | `DATA len … . len = STRLEN( text ).`                              | Built-in function                              |
| `CORRESPONDING #( source MAPPING t = s )`         | Field-by-field `MOVE` or `MOVE-CORRESPONDING`                     | Explicit mapping, no hidden conversions        |
| `VALUE itab_type( ( f = 1 ) ( f = 2 ) )`          | `APPEND VALUE #( f = 1 ) TO itab. APPEND VALUE #( f = 2 ) TO itab.`| Inline table literals                         |
| `LET … IN`                                        | Helper variable above the expression                              | Scoped binding inside an expression            |

## Key rules

- **Use inline declarations at first point of use**, not at method top — keeps declarations next to context.
- **`FINAL` over `DATA`** whenever the variable won't be reassigned — compiler enforces immutability.
- **Don't chain**: write `a = b. b = c.` — not `a = b = c.` Hides intent and breaks step-debugging.
- **Avoid `DESCRIBE`** — `lines( )`, `strlen( )`, `numofchar( )` are direct expressions.
- **Table expressions raise `CX_SY_ITAB_LINE_NOT_FOUND`** when no match — use `VALUE #( itab[ … ] DEFAULT … )` or wrap in TRY/CATCH.
- **Don't mix old and new syntax** in the same method — pick one style per method; mixing reads as half-finished refactor.

## Release gates

| Construct                                  | Minimum release          |
| ------------------------------------------ | ------------------------ |
| Inline declarations (`DATA(x)`)            | NW 7.40 SP02             |
| `FINAL( )` inline                          | NW 7.40 SP08 (full form) |
| String templates                           | NW 7.40                  |
| Constructor expressions (VALUE/NEW/CONV)   | NW 7.40 SP08             |
| Table expressions (`itab[ … ]`)            | NW 7.40 SP08             |
| `COND`, `SWITCH`, `FILTER`, `REDUCE`, `FOR`| NW 7.40 SP08             |
| `xsdbool( )`                               | NW 7.02                  |
| `LET … IN`                                 | NW 7.40 SP08             |
| Predicative method call                    | NW 7.40 SP08             |
| `MESH`, `CORRESPONDING` with MAPPING       | NW 7.40 SP08             |
| Typed `ENUM`                               | NW 7.51                  |
| ABAP SQL CTE (`WITH +cte AS …`)            | NW 7.51 (HANA)           |

> [!warning] Legacy systems
> Anything below NW 7.40 SP08 (rare on S/4 but common on older NetWeaver Java stacks or downstream-only systems) needs the legacy forms. Check `SY-ABAP_VERSION` if writing code that must compile in mixed landscapes.

## Anti-patterns

- Wrapping every old statement in a TRY/CATCH "to be safe" — use `DEFAULT` on table expressions instead of catching `CX_SY_ITAB_LINE_NOT_FOUND` everywhere.
- Inlining everything into one mega-expression — readable code beats clever code. If a `REDUCE` needs three nested `COND`s, extract a helper method.
- `DATA(x) = some_call( )` followed by reassignment — should have been `FINAL(x)` if not reassigned, plain `DATA x TYPE …` if reassigned with explicit type.

## Sources

- [[sources/opencode-abap-context-library]] — `modern-abap-patterns.md` (the prefer/over table) and `abap-syntax-reference.md` (release gates, construct details).
