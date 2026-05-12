---
title: "ABAP Unit Testing"
type: topic
tags: [testing, abap-unit, quality, clean-abap, abap]
sap_release: ["S/4HANA", "BTP ABAP", "NW 7.40+"]
status: draft
sources:
  - "[[sources/opencode-abap-context-library]]"
related:
  - "[[concepts/clean-abap]]"
  - "[[topics/abap-quality-gates]]"
  - "[[patterns/test-doubles-and-dependency-injection]]"
created: 2026-05-11
updated: 2026-05-11
---

# ABAP Unit Testing

> ABAP Unit is the in-language test framework: test classes are flagged `FOR TESTING`, methods carry `RISK LEVEL` and `DURATION` attributes, assertions go through `CL_ABAP_UNIT_ASSERT`. The whole infrastructure runs in ATC and Eclipse ADT.

## Why this matters

Without tests, every refactor is a gamble and every regression is a customer ticket. ABAP Unit is the cheapest safety net available — it runs in-system, requires no external tooling, and integrates with the [[topics/abap-quality-gates]] pipeline.

## Core philosophy

- **Test publics, not internals.** Needing to test a private method usually means the design has a hidden public API trying to escape.
- **Don't chase coverage.** 100 % coverage with weak assertions catches nothing. Focus on meaningful behaviour.
- **Don't test framework code.** Trust SAP standard libraries; test *your* logic.
- **Tests are documentation.** A reader should understand expected behaviour from the test code.
- **Make tests fast and isolated.** Unit tests hit no DB, no RFC, no file system. Use test doubles. See [[patterns/test-doubles-and-dependency-injection]].

## Test class skeleton

```abap
CLASS ltc_order_calculator DEFINITION
  FOR TESTING
  RISK LEVEL HARMLESS
  DURATION SHORT
  FINAL.

  PRIVATE SECTION.
    DATA mo_cut TYPE REF TO zcl_order_calculator.   " cut = class under test

    METHODS:
      class_setup    FOR TESTING CLASS-METHODS,    " runs once before any test
      setup,                                        " runs before each test
      teardown,                                     " runs after each test

      "! given a 3-line order, when calculating total, then sum is correct
      total_sums_all_lines        FOR TESTING,
      total_rejects_negative_qty  FOR TESTING RAISING cx_static_check.

ENDCLASS.

CLASS ltc_order_calculator IMPLEMENTATION.

  METHOD setup.
    mo_cut = NEW #( ).
  ENDMETHOD.

  METHOD total_sums_all_lines.
    " Given
    DATA(items) = VALUE zcl_order_calculator=>tt_item(
      ( product = 'A' qty = 2 price = '10.00' )
      ( product = 'B' qty = 1 price = '20.00' )
      ( product = 'C' qty = 4 price = '5.00' ) ).

    " When
    DATA(total) = mo_cut->calculate_total( items ).

    " Then
    cl_abap_unit_assert=>assert_equals(
      act = total
      exp = CONV decfloat34( '60.00' )
      msg = '2*10 + 1*20 + 4*5 = 60' ).
  ENDMETHOD.

  METHOD total_rejects_negative_qty.
    " Given
    DATA(items) = VALUE zcl_order_calculator=>tt_item(
      ( product = 'A' qty = -1 price = '10.00' ) ).

    " When / Then
    TRY.
        mo_cut->calculate_total( items ).
        cl_abap_unit_assert=>fail( 'Expected exception not raised' ).
      CATCH zcx_order_invalid INTO DATA(lx).
        cl_abap_unit_assert=>assert_equals(
          act = lx->if_t100_message~t100key-msgno
          exp = '042' ).
    ENDTRY.
  ENDMETHOD.

ENDCLASS.
```

## Test class attributes

| Attribute     | Values                                           | Meaning                                            |
| ------------- | ------------------------------------------------ | -------------------------------------------------- |
| `RISK LEVEL`  | `HARMLESS` / `DANGEROUS` / `CRITICAL`            | Whether test may modify persistent data — gates execution in PRD-like systems. |
| `DURATION`    | `SHORT` / `MEDIUM` / `LONG`                      | Expected run time — long tests skipped in fast CI lanes. |

Use **`RISK LEVEL HARMLESS DURATION SHORT`** by default for unit tests. Anything else needs justification.

## Fixture methods

| Method            | When it runs                       |
| ----------------- | ---------------------------------- |
| `class_setup`     | Once before any test in the class  |
| `setup`           | Before each test method            |
| `teardown`        | After each test method             |
| `class_teardown`  | Once after all tests in the class  |

Use `setup` to instantiate a fresh CUT (class-under-test) per test — keeps tests isolated.

## Given-When-Then

```abap
METHOD descriptive_name.
  " Given — arrange preconditions
  DATA(input) = …

  " When — exercise the system under test
  DATA(result) = mo_cut->do_thing( input ).

  " Then — assert
  cl_abap_unit_assert=>assert_equals( act = result exp = … ).
ENDMETHOD.
```

Rules:

- **One "When" per test.** If you need two, it's two tests.
- **Few focused assertions** — when a test fails, you should know exactly what broke.
- **Descriptive names**: `test_rejects_negative_quantity`, not `test_3` or `test_calculate`.

## Common assertions

| Method                                              | Use                                                    |
| --------------------------------------------------- | ------------------------------------------------------ |
| `assert_equals( act = exp = msg = )`                | Most common — equality of values or structures         |
| `assert_differs( act = exp = )`                     | Inequality                                             |
| `assert_initial( act = )` / `assert_not_initial( )` | Empty / non-empty                                      |
| `assert_true( act = )` / `assert_false( act = )`    | Boolean                                                |
| `assert_bound( act = )` / `assert_not_bound( )`     | Object reference bound / unbound                       |
| `assert_text_matches( act = pattern = )`            | Regex match                                            |
| `assert_table_contains( line = table = )`           | Table membership                                       |
| `assert_number_between( … )`                        | Numeric range                                          |
| `fail( msg = )`                                     | Explicit fail — use in the unreachable branch of TRY   |

## Testing EML / RAP code

Use the dedicated test environments:

- **`CL_CDS_TEST_ENVIRONMENT`** — creates test doubles for a CDS view's underlying data sources; you provide rows in-test, EML reads them.
- **`CL_OSQL_TEST_ENVIRONMENT`** — same idea for raw `SELECT` from DB tables.
- **`CL_ABAP_TESTDOUBLE`** — generic interface-double generator.

Combined with EML's **`IN LOCAL MODE`** (bypasses authorization), this enables fast, in-memory, isolated RAP tests. See [[concepts/eml]] and [[patterns/test-doubles-and-dependency-injection]].

## Anti-patterns

- ❌ **One test method that does setup + act + assert + act + assert** — split into separate tests.
- ❌ **Copying production code into the test "to verify"** — you're testing your copy, not the original. Test behaviour, not implementation.
- ❌ **Real DB / RFC / files in unit tests** — that's an integration test; ABAP Unit can run them but they're slow and brittle.
- ❌ **Giant helper methods that hide the test logic** — keep tests self-contained and readable.
- ❌ **Conditional logic in tests** (`IF` / `CASE`) — tests should be linear; branches indicate two tests merged.
- ❌ **Writing tests after the fact "for coverage"** — write tests that catch real bugs, not tests that exercise lines.
- ❌ **`assert_equals` on a giant structure** — assertion failure dumps the whole thing, hard to read. Assert specific fields.
- ❌ **Empty `setup` / `teardown`** — delete them.

## Integration into quality gates

See [[topics/abap-quality-gates]]:

- Every transport touching a class triggers an ABAP Unit run on that class.
- New classes ship with tests (≥ 1 test method).
- Test failures block transport release.

## Related

- [[concepts/clean-abap]]
- [[patterns/test-doubles-and-dependency-injection]]
- [[topics/abap-quality-gates]]
- [[concepts/eml]] — testing RAP code with `IN LOCAL MODE`.

## Sources

- [[sources/opencode-abap-context-library]] — `abap-testing-patterns.md` (test class skeleton, attributes, fixtures, assertions, RAP testing), `clean-abap-testing.md` (philosophy, anti-patterns).
