---
title: "OO Design Patterns in ABAP"
type: pattern
tags: [design-patterns, oo, clean-abap, abap]
sap_release: ["NW 7.40+", "S/4HANA", "BTP ABAP"]
status: stable
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[patterns/test-doubles-and-dependency-injection]]"
  - "[[concepts/abap-exception-handling]]"
created: 2026-05-11
updated: 2026-05-13
---

# OO Design Patterns in ABAP

> The "Gang of Four"-style patterns most commonly applied in modern ABAP — with ABAP-idiomatic implementation notes and the pitfalls specific to the language (no constructor overloading, no static interfaces, package visibility quirks).

## Why a single page

These are well-known patterns covered in countless OO texts. The value-add here is the **ABAP-idiomatic implementation** and the **decision context** (when to apply which) — not yet another GoF re-explanation. Each section is a sketch; expand into its own page only if a pattern accumulates project-specific guidance.

## Singleton

**Intent**: exactly one instance of a class, globally accessible.

```abap
CLASS zcl_config_registry DEFINITION PUBLIC FINAL CREATE PRIVATE.
  PUBLIC SECTION.
    CLASS-METHODS get_instance
      RETURNING VALUE(ro_instance) TYPE REF TO zcl_config_registry.

    METHODS get_value
      IMPORTING iv_key          TYPE string
      RETURNING VALUE(rv_value) TYPE string.

  PRIVATE SECTION.
    CLASS-DATA so_instance TYPE REF TO zcl_config_registry.
    METHODS constructor.
    DATA mt_config TYPE … .
ENDCLASS.

CLASS zcl_config_registry IMPLEMENTATION.
  METHOD get_instance.
    IF so_instance IS NOT BOUND.
      so_instance = NEW #( ).
    ENDIF.
    ro_instance = so_instance.
  ENDMETHOD.
ENDCLASS.
```

> [!warning] Test pain
> Singletons make tests stateful — the instance survives between tests. If you must use one, expose a `class-method reset( )` for `setup`. Better: inject the instance via [[patterns/test-doubles-and-dependency-injection]].

**When to use**: cross-cutting registries (config, log buffer) where dependency injection is impractical.
**When NOT to use**: anywhere you can pass the dependency explicitly.

## Factory

**Intent**: encapsulate construction logic so callers don't `NEW` concrete classes directly.

```abap
CLASS zcl_order_factory DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    CLASS-METHODS create_for_market
      IMPORTING iv_market   TYPE bukrs
      RETURNING VALUE(ro_order) TYPE REF TO zif_order.
ENDCLASS.

CLASS zcl_order_factory IMPLEMENTATION.
  METHOD create_for_market.
    ro_order = SWITCH #( iv_market
      WHEN 'DE01' THEN NEW zcl_order_de( )
      WHEN 'US01' THEN NEW zcl_order_us( )
      ELSE             NEW zcl_order_default( ) ).
  ENDMETHOD.
ENDCLASS.
```

ABAP-idiomatic notes:

- **`CREATE PRIVATE` + factory class-method** is the canonical form — prevents direct `NEW` by callers.
- Combine with **registry** when the factory's logic is data-driven (table-resolved class names) — same shape as [[patterns/table-driven-enhancement-framework]].

**When to use**: multiple variants of the same interface; construction needs context (config lookup, environment detection); want a single chokepoint for instantiation.
**When NOT to use**: only one implementation will ever exist — `NEW zcl_order( )` is fine.

## Strategy

**Intent**: encapsulate interchangeable algorithms behind a common interface.

```abap
INTERFACE zif_pricing_strategy PUBLIC.
  METHODS price
    IMPORTING is_item        TYPE zst_item
    RETURNING VALUE(rv_total) TYPE wertv8.
ENDINTERFACE.

CLASS zcl_pricing_standard DEFINITION PUBLIC FINAL.
  PUBLIC SECTION. INTERFACES zif_pricing_strategy. ENDCLASS.

CLASS zcl_pricing_promo    DEFINITION PUBLIC FINAL.
  PUBLIC SECTION. INTERFACES zif_pricing_strategy. ENDCLASS.

" Caller — chooses strategy at runtime
DATA(strategy) = COND #( WHEN is_promo_active( )
                         THEN CAST zif_pricing_strategy( NEW zcl_pricing_promo( ) )
                         ELSE CAST zif_pricing_strategy( NEW zcl_pricing_standard( ) ) ).
DATA(total) = strategy->price( ls_item ).
```

**When to use**: multiple "ways to do the same thing"; want to swap algorithm without changing caller; testability via [[patterns/test-doubles-and-dependency-injection]].
**When NOT to use**: only one algorithm — premature abstraction.

## Observer

**Intent**: subject notifies registered observers when its state changes.

```abap
INTERFACE zif_order_observer PUBLIC.
  METHODS on_order_changed
    IMPORTING is_order TYPE zst_order.
ENDINTERFACE.

CLASS zcl_order_subject DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS:
      attach IMPORTING io_observer TYPE REF TO zif_order_observer,
      detach IMPORTING io_observer TYPE REF TO zif_order_observer,
      notify IMPORTING is_order    TYPE zst_order.

  PRIVATE SECTION.
    DATA mt_observers TYPE STANDARD TABLE OF REF TO zif_order_observer.
ENDCLASS.

CLASS zcl_order_subject IMPLEMENTATION.
  METHOD attach. APPEND io_observer TO mt_observers. ENDMETHOD.
  METHOD detach. DELETE mt_observers WHERE table_line = io_observer. ENDMETHOD.
  METHOD notify.
    LOOP AT mt_observers INTO DATA(lo).
      lo->on_order_changed( is_order ).
    ENDLOOP.
  ENDMETHOD.
ENDCLASS.
```

ABAP-idiomatic notes:

- **ABAP events** (`EVENTS`, `RAISE EVENT`, `SET HANDLER`) are the language-native version of this pattern. Use them for in-process broadcast.
- Observers swallow exceptions — wrap each `lo->on_order_changed( … )` in TRY/CATCH or one bad observer breaks the rest.

**When to use**: decouple a domain object from N consumers that react to its changes (logging, cache invalidation, UI refresh).
**When NOT to use**: only one consumer — direct call is simpler.

## Facade

**Intent**: simple unified interface in front of a complex subsystem.

```abap
CLASS zcl_order_facade DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS create_and_release
      IMPORTING iwa_order      TYPE zst_order
      RETURNING VALUE(rv_id)   TYPE vbeln_va
      RAISING   zcx_order_invalid.
ENDCLASS.

CLASS zcl_order_facade IMPLEMENTATION.
  METHOD create_and_release.
    rv_id = mo_factory->create( iwa_order ).
    mo_pricing->calculate( rv_id ).
    mo_credit->check( rv_id ).
    mo_release->release( rv_id ).
  ENDMETHOD.
ENDCLASS.
```

**When to use**: simplify the "happy path" through a multi-step subsystem; provide a stable boundary for outside callers (RFC, OData, Fiori).
**When NOT to use**: facade just delegates one-to-one to a single class — that's a wrapper, not a facade.

## Template Method

**Intent**: base class defines the algorithm skeleton; subclasses fill in the steps.

```abap
CLASS zcl_export_base DEFINITION PUBLIC ABSTRACT.
  PUBLIC SECTION.
    METHODS run FINAL.

  PROTECTED SECTION.
    METHODS:
      fetch_data   ABSTRACT,
      transform    ABSTRACT,
      write_output ABSTRACT.
ENDCLASS.

CLASS zcl_export_base IMPLEMENTATION.
  METHOD run.
    fetch_data( ).
    transform( ).
    write_output( ).
  ENDMETHOD.
ENDCLASS.

CLASS zcl_export_csv DEFINITION INHERITING FROM zcl_export_base FINAL.
  PROTECTED SECTION.
    METHODS:
      fetch_data REDEFINITION,
      transform REDEFINITION,
      write_output REDEFINITION.
ENDCLASS.
```

ABAP-idiomatic notes:

- **Mark the template method `FINAL`** — subclasses must not override the orchestration; they only fill in steps.
- **Mark hook methods `ABSTRACT` or `PROTECTED`** — never `PUBLIC` if they're algorithm-internal.
- Often loses out to **Strategy** in modern designs — composition over inheritance.

**When to use**: a well-defined invariant algorithm with clear variation points.
**When NOT to use**: variation is more than 50 % of the algorithm — use Strategy or a higher-level Pipeline pattern.

## Comparison

| Pattern        | Decoupling axis            | Most common in ABAP for…                       |
| -------------- | -------------------------- | ---------------------------------------------- |
| Singleton      | Lifecycle                  | Config / log registries; tolerated, not loved  |
| Factory        | Construction               | Multi-variant business objects; market-specific code |
| Strategy       | Algorithm                  | Pricing, validation, routing                   |
| Observer       | Notification               | UI refresh, audit, cache invalidation          |
| Facade         | Subsystem boundary         | RFC / OData entry points                       |
| Template Method| Algorithm + variation      | Reports / exports / batch jobs                 |

## Anti-patterns

- **Singleton-with-state used as global mutable variable** — defeats encapsulation; tests can't isolate.
- **Factory that always returns the same subclass** — useless indirection.
- **Strategy where each "strategy" is one method on a class** — just pass a function reference or use polymorphism on the domain object directly.
- **Observer that lets handler exceptions kill the notify loop** — wrap each observer call.
- **Facade that exposes every method of every subsystem method** — that's not a facade, that's an aggregator with no value-add.
- **Template Method with public hook methods** — subclasses ignore the algorithm and call hooks directly.

## Related

- [[concepts/clean-abap]] — patterns implement Clean ABAP's "small classes, clear responsibilities".
- [[patterns/test-doubles-and-dependency-injection]] — DI is the substrate that makes Strategy, Factory, Observer testable.
- [[concepts/abap-exception-handling]] — patterns and exception flow interact (Observer notify must catch).
- [[patterns/table-driven-enhancement-framework]] — a data-driven Factory + Strategy combination.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-testing-patterns.md` (pattern listing).
- *(future)* SAP-samples/abap-cheat-sheets — concrete idiomatic snippets.
- *(future)* GoF + Clean Code references for non-ABAP background.
