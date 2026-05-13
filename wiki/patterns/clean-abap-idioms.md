---
title: "Clean ABAP Idioms"
type: pattern
tags: [clean-abap, idioms, cheatsheet, abap, modern-syntax]
sap_release: ["NW 7.40+ (most)", "NW 7.51+ (ENUM)", "ABAP Platform 2021+ (dref->*)", "NW 7.52+ (RAISE EXCEPTION NEW)"]
status: stable
sources:
  - "[[sources/clean-abap-styleguide]]"
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[snippets/abap-modern-syntax-cheatsheet]]"
  - "[[concepts/abap-exception-handling]]"
  - "[[topics/abap-performance]]"
created: 2026-05-13
updated: 2026-05-13
---

# Clean ABAP Idioms

> Cheatsheet of small, single-purpose Clean ABAP rules — the "tactical" idioms (tables, strings, booleans, conditions, control flow) that aren't big enough for their own concept page but accumulate across thousands of lines. Each entry: anti-pattern → preferred form → rationale.

## How to use

Skim this page during code review. Each rule is small. Together they shape every method's surface texture. Coverage is enforced by [[entities/tools/code-pal-for-abap]] and a subset by [[entities/tools/abaplint]].

## Tables

### `LINE_EXISTS` over `READ TABLE` for membership

```abap
" preferred
IF line_exists( orders[ status = 'OPEN' ] ).
  ...
ENDIF.

" anti-pattern
READ TABLE orders TRANSPORTING NO FIELDS WITH KEY status = 'OPEN'.
IF sy-subrc = 0.
  ...
ENDIF.
```

**Why**: expression form, no `sy-subrc` plumbing, single statement.

### Table expression with `OPTIONAL` for "row may be absent"

```abap
" preferred
DATA(line) = VALUE #( orders[ id = 4711 ] OPTIONAL ).

" anti-pattern
READ TABLE orders INTO DATA(line) WITH KEY id = 4711.
IF sy-subrc <> 0.
  CLEAR line.
ENDIF.
```

If the row must exist, drop `OPTIONAL` — runtime exception is the correct behavior.

### `INSERT INTO TABLE` over `APPEND TO`

```abap
" preferred (works for any table type)
INSERT order INTO TABLE orders.

" anti-pattern (only standard tables; sortedness lost; semantics weak)
APPEND order TO orders.
```

`APPEND` is reserved for the rare case "I genuinely need positional append on a standard table".

### Avoid `DEFAULT KEY`

```abap
" preferred — explicit key
DATA orders TYPE STANDARD TABLE OF order_t WITH KEY id.

" anti-pattern — silent "all non-numeric fields" key
DATA orders TYPE STANDARD TABLE OF order_t WITH DEFAULT KEY.
```

`DEFAULT KEY` collects every non-numeric field as the key, leading to surprising performance characteristics and hidden coupling to the structure layout.

### Use the right table type

| Type      | When                                            |
| --------- | ----------------------------------------------- |
| `HASHED`  | Lookup by full unique key, no iteration order   |
| `SORTED`  | Lookup by partial key, ordered iteration        |
| `STANDARD` | Append-and-iterate, no key-based lookup        |

Wrong type = O(n) when you need O(1).

## Strings

### Backticks for literals

```abap
" preferred — `…` is type `string` (no trailing-blank trimming)
DATA(prefix) = `Order: `.

" anti-pattern — '…' is type `c` of computed length (trailing blanks trimmed)
DATA(prefix) = 'Order: '.
```

### `|…|` for assembly

```abap
" preferred
DATA(msg) = |Customer { customer } has { lines( orders ) } open orders|.

" anti-pattern
CONCATENATE 'Customer ' customer ' has ' lv_count_str ' open orders' INTO msg.
```

## Booleans

### Use `ABAP_BOOL`, not custom types

```abap
" preferred
DATA is_active TYPE abap_bool.

" anti-pattern
DATA is_active TYPE c LENGTH 1.
DATA is_active TYPE flag.
```

### Compare to `ABAP_TRUE` / `ABAP_FALSE`

```abap
" preferred
IF is_active = abap_true.
IF is_active = abap_false.

" anti-pattern (works but cryptic)
IF is_active = 'X'.
IF is_active IS INITIAL.
```

### `XSDBOOL` to assign

```abap
" preferred
DATA(is_open) = xsdbool( status = 'O' ).

" anti-pattern
IF status = 'O'.
  is_open = abap_true.
ELSE.
  is_open = abap_false.
ENDIF.
```

## Conditions

### Make conditions positive

```abap
" preferred
IF is_active = abap_true.
  do_something( ).
ELSE.
  do_other( ).
ENDIF.

" anti-pattern (forces double-mental-negation)
IF NOT is_inactive = abap_true.
  do_something( ).
ELSE.
  do_other( ).
ENDIF.
```

### `IS NOT` over `NOT IS`

```abap
" preferred
IF order IS NOT BOUND.
IF amount IS NOT INITIAL.

" anti-pattern
IF NOT order IS BOUND.
IF NOT amount IS INITIAL.
```

### Predicative method calls

```abap
" preferred
IF validator->is_valid( order ).

" anti-pattern
IF validator->is_valid( order ) = abap_true.
```

A method returning `ABAP_BOOL` reads naturally as a predicate.

## Ifs

### `CASE` over `ELSE IF` for multiple alternatives

```abap
" preferred
CASE status.
  WHEN 'O'. handle_open( ).
  WHEN 'C'. handle_closed( ).
  WHEN 'X'. handle_cancelled( ).
  WHEN OTHERS. raise_unknown( ).
ENDCASE.

" anti-pattern
IF status = 'O'.
  handle_open( ).
ELSEIF status = 'C'.
  handle_closed( ).
ELSEIF status = 'X'.
  handle_cancelled( ).
ELSE.
  raise_unknown( ).
ENDIF.
```

### Keep nesting depth low

```abap
" preferred — early return
METHOD process.
  IF order IS INITIAL.
    RAISE EXCEPTION NEW zcx_invalid_input( ).
  ENDIF.

  IF order-status = 'X'.
    RETURN.
  ENDIF.

  do_real_work( order ).
ENDMETHOD.

" anti-pattern (arrow code)
METHOD process.
  IF order IS NOT INITIAL.
    IF order-status <> 'X'.
      do_real_work( order ).
    ENDIF.
  ELSE.
    RAISE EXCEPTION NEW zcx_invalid_input( ).
  ENDIF.
ENDMETHOD.
```

## Control flow

### Fail fast

Validate inputs at method entry; raise immediately. Don't let bad data reach the middle of the method.

```abap
METHOD calculate.
  IF items IS INITIAL.
    RAISE EXCEPTION NEW zcx_no_items( ).
  ENDIF.
  IF rate <= 0.
    RAISE EXCEPTION NEW zcx_invalid_rate( ).
  ENDIF.

  result = ... " core logic, inputs are now trusted
ENDMETHOD.
```

### `CHECK` only at the very start

```abap
" preferred (or use IF-based fail-fast above)
METHOD process_active_only.
  CHECK order-active = abap_true.
  ...
ENDMETHOD.

" anti-pattern (mid-method CHECK is hard to spot)
METHOD process.
  do_setup( ).
  CHECK order-active = abap_true.
  do_more( ).
ENDMETHOD.
```

`CHECK` mid-method silently exits the enclosing block — surprising behavior. Use `RETURN` (explicit) or `IF … RETURN. ENDIF.` instead.

## Variables

### Inline declaration over up-front

```abap
" preferred
DATA(orders) = read_orders( ).
DATA(filtered) = filter_active( orders ).

" anti-pattern
DATA orders TYPE order_tab.
DATA filtered TYPE order_tab.
orders = read_orders( ).
filtered = filter_active( orders ).
```

### `dref->*` over field-symbol assign for dynamic data (ABAP Platform 2021+)

```abap
" preferred (modern)
result = dref->*.

" anti-pattern (legacy boilerplate)
ASSIGN dref->* TO FIELD-SYMBOL(<fs>).
result = <fs>.
```

## Constants

### `ENUM` over constants interfaces (NW 7.51+)

```abap
" preferred
TYPES: BEGIN OF ENUM message_severity,
         warning,
         error,
       END OF ENUM message_severity.

" anti-pattern
INTERFACE common_constants.
  CONSTANTS: warning TYPE symsgty VALUE 'W',
             error   TYPE symsgty VALUE 'E'.
ENDINTERFACE.
```

### Descriptive constant names

```abap
" preferred
CONSTANTS status_inactive TYPE mmsta VALUE '90'.

" anti-pattern (name = value, no semantic information)
CONSTANTS c_90 TYPE mmsta VALUE '90'.
```

## Calls

### Omit `EXPORTING` keyword

```abap
" preferred
result = calc( a = 1 b = 2 ).

" anti-pattern
result = calc( EXPORTING a = 1 b = 2 ).
```

### Omit single-parameter name

```abap
" preferred
result = parse( input ).

" anti-pattern
result = parse( i_input = input ).
```

### Omit `me->` for own attributes/methods

```abap
" preferred
DATA(result) = calculate( ).

" anti-pattern
DATA(result) = me->calculate( ).
```

`me->` is reserved for disambiguation when a local variable shadows an instance attribute.

## Methods

### `RETURN`, `EXPORT`, or `CHANGE` — exactly one parameter

A method should have a single output. Multiple outputs almost always indicates two methods coupled into one.

```abap
" preferred (single RETURNING)
METHODS calculate IMPORTING items TYPE tt_item
                  RETURNING VALUE(result) TYPE decfloat34
                  RAISING   zcx_invalid_input.

" anti-pattern (multiple outputs hide multiple responsibilities)
METHODS calculate IMPORTING items TYPE tt_item
                  EXPORTING result TYPE decfloat34
                            warnings TYPE tt_warning
                            log_id TYPE i.
```

### Split methods over boolean input parameter

```abap
" preferred
METHODS save_draft IMPORTING order TYPE zorder.
METHODS save_final IMPORTING order TYPE zorder.

" anti-pattern
METHODS save IMPORTING order TYPE zorder
                       is_draft TYPE abap_bool.
```

A boolean parameter that swings the method's behavior is two methods crammed into one.

## See also

- [[concepts/clean-abap]] — parent page covering full Clean ABAP scope.
- [[snippets/abap-modern-syntax-cheatsheet]] — companion cheatsheet for inline declarations, constructor expressions, table expressions.
- [[concepts/abap-exception-handling]] — `RAISE EXCEPTION NEW` and CX_* hierarchy.
- [[topics/abap-performance]] — when an idiom would conflict with measured performance, see this page first.

## Sources

- [[sources/clean-abap-styleguide]] — chapters Tables, Strings, Booleans, Conditions, Ifs, Variables, Constants, Methods (calls, parameter rules, body).
- [[sources/opencode-abap-context-library]] — `modern-abap-patterns.md` corroborates and adds release-gate detail.
