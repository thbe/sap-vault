---
title: "Test Doubles and Dependency Injection"
type: pattern
tags: [testing, di, test-doubles, abap-unit, clean-abap, abap]
sap_release: ["S/4HANA", "BTP ABAP", "NW 7.40+"]
status: draft
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[topics/abap-unit-testing]]"
  - "[[concepts/clean-abap]]"
  - "[[concepts/abap-exception-handling]]"
created: 2026-05-11
updated: 2026-05-11
---

# Test Doubles and Dependency Injection

> Make ABAP classes testable in isolation by injecting their collaborators through interfaces, and replacing those collaborators in tests with `CL_ABAP_TESTDOUBLE` (for interfaces) or `CL_OSQL_TEST_ENVIRONMENT` / `CL_CDS_TEST_ENVIRONMENT` (for DB / CDS access).

## Problem

Production code commonly *creates* its dependencies inline:

```abap
" anti-pattern
METHOD process_order.
  DATA(repo) = NEW zcl_order_db_repository( ).   " hardcoded DB hit
  DATA(order) = repo->load( iv_order_id ).
  ...
ENDMETHOD.
```

Now you can't test `process_order` without a real DB. The class **and its dependencies** are entangled, and the test surface explodes.

## Solution — invert the dependency

1. Define an **interface** for the collaborator: `ZIF_ORDER_REPOSITORY`.
2. The class **declares the dependency** as a reference to the interface, not the concrete class.
3. The dependency is **injected** at construction time.

```abap
INTERFACE zif_order_repository PUBLIC.
  METHODS load
    IMPORTING iv_order_id    TYPE vbeln_va
    RETURNING VALUE(rs_order) TYPE zst_order
    RAISING   zcx_order_not_found.
ENDINTERFACE.

CLASS zcl_order_processor DEFINITION PUBLIC FINAL.
  PUBLIC SECTION.
    METHODS constructor
      IMPORTING io_repository TYPE REF TO zif_order_repository.

    METHODS process_order
      IMPORTING iv_order_id TYPE vbeln_va
      RAISING   zcx_order_invalid.

  PRIVATE SECTION.
    DATA mo_repository TYPE REF TO zif_order_repository.
ENDCLASS.

CLASS zcl_order_processor IMPLEMENTATION.
  METHOD constructor.
    mo_repository = io_repository.
  ENDMETHOD.

  METHOD process_order.
    DATA(order) = mo_repository->load( iv_order_id ).
    " ... pure business logic against `order`, no I/O.
  ENDMETHOD.
ENDCLASS.
```

Production wiring:

```abap
DATA(processor) = NEW zcl_order_processor(
  io_repository = NEW zcl_order_db_repository( ) ).
```

Test wiring uses a test double in place of the DB repository.

## Three injection techniques

| Technique               | When to use                                          | Pros                  | Cons                                            |
| ----------------------- | ---------------------------------------------------- | --------------------- | ----------------------------------------------- |
| **Constructor (preferred)** | Default — the class needs the dependency to function | Explicit, immutable, no test seam in production | Requires all callers to pass it (or a factory)  |
| **Setter**              | Optional collaborators, runtime swap                  | Flexible              | Class can exist in a partially-initialised state |
| **Back-door / test seam** | Legacy code you can't refactor safely                 | Doesn't touch production API | Production code has to know about tests indirectly |

> [!tip] Default to constructor injection
> Use setter only when the dependency truly is optional. Use back-door / test seams only when refactoring would be unsafe or out of scope.

## CL_ABAP_TESTDOUBLE — stubbing an interface

Generate a runtime double for any interface, set return values, exercise the CUT.

```abap
CLASS ltc_order_processor DEFINITION
  FOR TESTING RISK LEVEL HARMLESS DURATION SHORT FINAL.

  PRIVATE SECTION.
    DATA:
      mo_repo_double TYPE REF TO zif_order_repository,
      mo_cut         TYPE REF TO zcl_order_processor.

    METHODS:
      setup,
      process_order_loads_via_repo FOR TESTING RAISING cx_static_check.
ENDCLASS.

CLASS ltc_order_processor IMPLEMENTATION.

  METHOD setup.
    " Generate a double for the interface
    mo_repo_double ?= cl_abap_testdouble=>create( 'ZIF_ORDER_REPOSITORY' ).
    mo_cut = NEW #( io_repository = mo_repo_double ).
  ENDMETHOD.

  METHOD process_order_loads_via_repo.
    " Configure the double to return a canned order for one specific key
    DATA(expected_order) = VALUE zst_order(
      order_id = '0001000001' net_amount = '100.00' ).

    cl_abap_testdouble=>configure_call( mo_repo_double
      )->returning( expected_order ).

    mo_repo_double->load( CONV vbeln_va( '0001000001' ) ).

    " Exercise the CUT
    mo_cut->process_order( iv_order_id = '0001000001' ).

    " (optional) verify the double was called
    cl_abap_testdouble=>verify_expectations( mo_repo_double ).
  ENDMETHOD.

ENDCLASS.
```

Configuration verbs:

| Verb                 | Effect                                                    |
| -------------------- | --------------------------------------------------------- |
| `returning( v )`     | Next matching call returns `v`                            |
| `raise_exception( ex )` | Next matching call raises `ex`                          |
| `ignore_all_parameters( )` | Match call regardless of args                         |
| `set_parameter( … )` / `set_table_parameter( … )` | Set exporting/changing/result values |

> [!info] Stub > Mock
> Prefer **stubs** (return canned data) over **mocks** (assert that specific calls happened). Mocks couple tests to implementation details — they break when refactoring even if behaviour is unchanged.

## CL_OSQL_TEST_ENVIRONMENT — doubling raw `SELECT`

When the CUT contains direct ABAP SQL (`SELECT … FROM zorder_h`), `CL_OSQL_TEST_ENVIRONMENT` redirects those reads to test doubles.

```abap
DATA(env) = cl_osql_test_environment=>create(
  i_dependency_list = VALUE #( ( 'ZORDER_H' ) ) ).

env->insert_test_data( VALUE zorder_h_tab(
  ( order_id = '0001000001' net_amount = '100.00' )
  ( order_id = '0001000002' net_amount = '200.00' ) ) ).

" Now any SELECT FROM zorder_h in the CUT reads from the doubles, not the DB.

env->destroy( ).   " in teardown
```

## CL_CDS_TEST_ENVIRONMENT — doubling CDS views

Same idea, for CDS view entities — supplies test data for the underlying tables / views that a CDS reads from.

```abap
DATA(env) = cl_cds_test_environment=>create(
  i_for_entity   = 'ZR_ORDER'
  i_dependency_list = VALUE #( ( 'ZORDER_H' ) ( 'ZORDER_I' ) ) ).

env->insert_test_data( VALUE zorder_h_tab( … ) ).
env->insert_test_data( VALUE zorder_i_tab( … ) ).
```

Combine with EML `IN LOCAL MODE` (see [[concepts/eml]]) for in-memory RAP testing.

## When to mock vs not

| Mock when                                                  | Don't mock when                                               |
| ---------------------------------------------------------- | ------------------------------------------------------------- |
| The collaborator is external / slow (DB, RFC, file system) | The collaborator is a simple value object or pure function    |
| The collaborator has side effects you don't want in tests  | The collaborator is the system under test                     |
| Multiple behaviour paths depend on collaborator output     | A small fake (in-memory implementation) is just as easy       |

> [!warning] Don't mock the system under test
> If you find yourself doubling the class you're testing, the test proves nothing. Refactor the class until its real behaviour is testable.

## Anti-patterns

- **Friend declarations for test access** — couples test class to production internals. Use DI instead.
- **`CALL METHOD ('METHOD_NAME')` dynamic invocation in tests** — fragile; refactoring renames break tests silently.
- **Globals / SET PARAMETER ID for "config"** — pass it in via constructor.
- **Mocking everything** — over-specified tests fail on any refactor; integration suffers.
- **Mocks with verify-call-count assertions everywhere** — locks in implementation; behaviour-focused tests should care about return values, not call counts.
- **`CL_ABAP_TESTDOUBLE` on a class with no interface** — the double mechanism needs an interface. If the collaborator is a class, extract an interface first.

## Related

- [[topics/abap-unit-testing]] — the framework this pattern lives in.
- [[concepts/clean-abap]] — dependency inversion as a Clean ABAP rule.
- [[concepts/abap-exception-handling]] — test exception paths with `raise_exception( )` on the double.
- [[concepts/eml]] — `IN LOCAL MODE` for RAP tests.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-testing-patterns.md` (DI techniques, CL_ABAP_TESTDOUBLE, CL_OSQL_TEST_ENVIRONMENT, CL_CDS_TEST_ENVIRONMENT), `clean-abap-testing.md` (mock-vs-stub, anti-patterns).
