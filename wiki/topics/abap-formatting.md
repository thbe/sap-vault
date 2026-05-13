---
title: "ABAP Formatting"
type: topic
tags: [formatting, layout, abap, clean-abap, readability]
sap_release: ["any"]
status: stable
sources:
  - "[[sources/clean-abap-styleguide]]"
  - "[[sources/sap-development-standard-approach-abap-fiori-v1]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[snippets/abap-modern-syntax-cheatsheet]]"
  - "[[snippets/flower-box-header]]"
  - "[[snippets/abap-doc-comments]]"
  - "[[topics/abap-quality-gates]]"
created: 2026-05-13
updated: 2026-05-13
---

# ABAP Formatting

> Layout, indentation, alignment, and line-break rules for ABAP source code. Distinct from naming and design — purely about *visual structure*. The official Clean ABAP styleguide dedicates a full chapter to it; this page distills the rules.

## Why this matters

Formatting rules carry near-zero cost (the ABAP Formatter applies most of them automatically) but disproportionate readability gain. Mixed formatting in a codebase signals "no standard agreed" and slows every reader; consistent formatting becomes invisible — exactly the goal.

Per Clean ABAP: *optimize for reading, not for writing.* Code is read 10× more than it is written.

## Core rules

### One statement per line

```abap
" good
DATA(name) = 'Alice'.
DATA(role) = 'Admin'.

" anti-pattern
DATA: name TYPE string, role TYPE string. name = 'Alice'. role = 'Admin'.
```

### Reasonable line length

The styleguide does not name a hard number; **120 columns** is a common project default. Anything beyond ~140 columns hurts side-by-side diffing. Pick one project-wide and configure the ABAP Formatter accordingly.

### Single blank line to separate

```abap
" good
DATA(orders) = read_orders( ).
DATA(filtered) = filter_active( orders ).

DATA(totals) = calculate_totals( filtered ).

" anti-pattern (excess blank lines)
DATA(orders) = read_orders( ).


DATA(filtered) = filter_active( orders ).



DATA(totals) = calculate_totals( filtered ).
```

Don't obsess: don't double-blank between every statement, don't single-blank inside tightly-coupled sequences.

## Alignment

### Align assignments to the same object — but not different ones

```abap
" good — same target, aligned
header-customer = '4711'.
header-currency = 'EUR'.
header-total    = 1000.

" good — different targets, NOT aligned
DATA(name) = 'Alice'.
DATA(roles) = read_roles( name ).
DATA(active_roles) = filter_active( roles ).
```

The visual signal of aligned `=` is "these belong together". Mis-applied across unrelated assignments, it suggests a relationship that doesn't exist.

### Don't align type clauses

```abap
" anti-pattern
DATA name        TYPE string.
DATA description TYPE string.

" good
DATA name TYPE string.
DATA description TYPE string.
```

Aligning `TYPE` adds noise without informational value. Same logic: a long member name shouldn't force every other declaration to add padding.

## Method calls

### Single-parameter calls on one line

```abap
" good
result = calculate_total( items ).

" anti-pattern (gratuitous break)
result = calculate_total(
  items ).
```

### Keep parameters behind the call

```abap
" good
result = calculate( items = orders
                    rate  = 0.19 ).

" anti-pattern (orphaned opener)
result = calculate(
                    items = orders
                    rate  = 0.19 ).
```

### Line-break for multiple parameters; align them

```abap
" good
result = process(
  source        = data_source
  filter        = active_only
  date_from     = '20260101'
  date_to       = '20261231' ).

" anti-pattern (inconsistent indentation)
result = process( source = data_source filter = active_only
  date_from = '20260101'
       date_to = '20261231' ).
```

### Close brackets at line end — not on a new line

```abap
" good
result = VALUE my_table(
  ( id = 1 name = 'A' )
  ( id = 2 name = 'B' ) ).

" anti-pattern
result = VALUE my_table(
  ( id = 1 name = 'A' )
  ( id = 2 name = 'B' )
).
```

The closing `)` belongs at the end of the previous line. The "C-style" dangling bracket is a JavaScript habit that doesn't belong in ABAP.

### Break the call to a new line if it gets too long

```abap
" good
DATA(result) =
  zcl_long_class_name=>create_factory_for_specific_use_case(
    type      = co_type_a
    parameter = lv_input ).
```

When the call won't fit on one line *with* its first parameter, push the entire call to the next line.

## Indentation

### Snap to tab; let the formatter handle width

ABAP convention is **2-space indentation** (not tabs). The ABAP Formatter enforces this. Do not mix.

### Indent inline declarations like method calls

```abap
" good
DATA(result) = process(
  source = orders
  filter = active ).
```

Inline `DATA(…) =` at the start counts as a normal assignment; the call's parameters indent relative to the `(`.

## Use the ABAP Formatter

Pretty Printer (now ABAP Formatter in ADT) handles the bulk of these rules mechanically. Run it before activation. Two non-negotiables per the styleguide:

1. **Use the formatter** — never hand-format what the tool can do for you.
2. **Use the team's settings** — not your personal preferences. Formatter config is a team artifact, checked into version control where possible (`.abaplint.json`, project SCI variant).

> [!tip] Per-team consistency
> If your team is on different formatter settings, you'll see thrash in every diff (someone re-formats on activation, the next person re-formats back). Lock the settings before adopting Clean ABAP enforcement.

## Anti-patterns / pitfalls

- ❌ **Aligning `TYPE` clauses** — adds noise.
- ❌ **Chaining declarations** (`DATA: a TYPE i, b TYPE i, c TYPE i.`) — masks add/remove diffs and forces colons/commas.
- ❌ **Chained assignments** (`a = b = c = 1.`) — hard to read, harder to debug.
- ❌ **Dangling closing brackets** on their own line.
- ❌ **Personal formatter settings** that re-thrash on every commit.
- ❌ **Tabs in source** — ABAP convention is spaces.
- ❌ **Multiple statements per line** — every Clean ABAP rule about size and readability rests on this.

## Related

- [[concepts/clean-abap]] — the parent governance page.
- [[snippets/abap-modern-syntax-cheatsheet]] — modern construct examples that follow these rules.
- [[snippets/flower-box-header]] — a project-specific exception that overrides "comment with `"`, not `*`" for legacy/modification headers.
- [[snippets/abap-doc-comments]] — `"!` ABAP Doc syntax (separate concern from formatting).
- [[topics/abap-quality-gates]] — formatter integration into pre-transport gates.

## Sources

- [[sources/clean-abap-styleguide]] — `#formatting` chapter (entire chapter is the source).
- [[sources/sap-development-standard-approach-abap-fiori-v1]] — confirms 2-space indent and Pretty Printer use as project standard.
