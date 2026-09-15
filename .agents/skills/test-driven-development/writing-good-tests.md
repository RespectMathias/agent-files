# Writing Good Tests

Load this reference when: writing or changing tests, adding mocks, or
adding cleanup/helper methods for tests.

## Overview

A test exists to catch a specific break:

```txt
1. Every test names the break it catches
2. Every test exercises the real thing
```

Strict TDD produces both naturally: write the test first, watch it fail
against real code, and mock only when the real dependency proves slow or
external.

## Principle 1: Name the Break

Before writing the test body, answer: what production change should make
this test fail, and is that change a bug or a decision? It should catch a
wrong branch, missing side effect, wrong argument, boundary case, or broken
contract.

Derive expectations independently. Prefer literals and hand-checked
fixtures, especially table-driven tests with literal `want` values. An
expectation computed by the code under test or its helpers can pass no
matter what that code does:

```typescript
// Bad. Mirror assertion uses same builder on both sides, so it is always true
const expected = buildSearchQuery({ tag: "urgent" });
expect(buildSearchQuery({ tag: "urgent" })).toBe(expected);

// Good. Hand-derived literal
expect(buildSearchQuery({ tag: "urgent" })).toBe('tag:"urgent"');
```

No change detectors. If only intentional decisions can fail a test, such
as changing a constant, exact wording, or private structure, it fires on
redesign and sleeps through bugs. Test behavior depending on the decision:
not `expect(MAX_RETRIES).toBe(5)` but "a failing call is retried 5 times and
the 6th attempt never happens."

Behavior, not text. Asserting that a script, skill, or config contains an
exact line proves only that the source contains that line. Run scripts
against controlled inputs and assert outputs, side effects, or exit codes.
Test agent instructions through consuming-agent behavior
(superpowers:writing-skills). Human prose earns no test.

Your code, not the framework. Test the contract your code makes at its
boundaries: route registered, query emitted, payload produced. Upstream
mechanics are their maintainers' tests. Asserting that a router invokes its
registered handler tests the framework, not your code. If upstream behavior
surprised you, write one narrow characterization test naming the assumption.

The same boundary applies inside your code: constructors, getters, constants,
and trivial forwarding earn tests only when they validate, normalize,
default, derive, enforce, or cause side effects. Otherwise test the first
consumer-visible result that depends on them.

### Gate Function 1/2

```txt
BEFORE writing the test body:
  Name the production change that would make this test fail.

  Cannot name one:
    Redesign around an observable behavior

  "The source text changed":
    Run the artifact and assert its effects

  Only intentional decisions:
    Change detector; test the behavior that depends on the decision

  Confirm the expected value is derived without the code under test.
  IF it reuses the code's logic or helpers:
    Replace it with a literal or hand-checked fixture
```

## Principle 2: Exercise the Real Thing

The mock earns no assertions. An assertion that passes when the mock is
present and fails when it is absent says nothing about the component.
Assert real component behavior. If the mock is what you are checking,
unmock it or delete the assertion.

```typescript
// Bad. Mock existence
expect(screen.getByTestId("sidebar-mock")).toBeInTheDocument();

// Good. Real behavior
expect(screen.getByRole("navigation")).toBeInTheDocument();
```

Human partner correction: "Are we testing the behavior of a mock?"

Mock at the right level. Learn every side effect of the real method before
replacing it. Keep what the test depends on real and mock the slow or
external operation below it. When unsure, run against the real implementation
first.

```typescript
// Bad. The mock swallows the config write that duplicate detection reads
vi.mock("ToolCatalog", () => ({
  discoverAndCacheTools: vi.fn().mockResolvedValue(undefined),
}));

// Good. Mock only the slow server startup; the config write stays real
vi.mock("MCPServerManager");
```

Make doubles specific. When arguments, call counts, or ordering are part of
the contract, assert them. A fake that accepts anything verifies nothing.
Give success, error, and malformed branches distinct fixtures or spies so
the wrong branch cannot satisfy the expectation.

Mirror real data completely. Mock the complete documented structure, not
just fields the test reads. Partial mocks can hide integration failures when
downstream code reads an omitted field.

Production classes carry production methods only. Test-only cleanup lives in
test utilities, never as a `destroy()` on the production class. Ask: is this
method called only from tests? Does this class own the resource lifecycle?
Wrong answers: test utility.

Prefer real components over complex mocks. Switch to an integration test when
mock setup outgrows test logic, mocks miss real methods, or tests break when
the mock changes.

Human partner question: "Do we need to be using a mock here?"

### Gate Function 2/2

```txt
BEFORE adding a mock or test helper:
  List the real method's side effects.
  Keep the ones the test depends on real.
  Mock the slow/external level below them.

  Mock responses mirror the complete real structure.

  A method only tests call lives in test utilities, not production.

  About to assert on the mock itself?
    Unmock it or delete the assertion.
```

## Tests Ship With the Implementation

"Complete" means the TDD cycle: failing test, minimal implementation,
refactor. Ship only tests the behavior needs. Trivial code and human prose
earn none. Process-only tests cost maintenance forever.

## The Mutation Check

Before finishing, mentally mutate the production code. At least one test
should fail for each realistic mutation:

- Wrong constant or argument
- Wrong branch handler
- Missing state change or side effect
- Empty or default return
- Missing validation for zero, empty, nil, unauthorized, or malformed input

A mutation nothing catches marks behavior as unprotected or the test as
tautological.

## Quick Reference

| When you...                        | Do                                                          |
| ---------------------------------- | ----------------------------------------------------------- |
| Write any test                     | Name the break it catches, a bug, not a decision            |
| Build an expected value            | Derive it by hand; never with the code under test           |
| Test a script or document          | Run it / pressure-test its consumer; never grep its text    |
| Reach for a dependency test        | Test your boundary contract, not their documented mechanics |
| Want to assert on a mocked element | Test the real component, or unmock it                       |
| Are about to mock a method         | Learn its side effects; mock the slow/external level        |
| Build a mock response              | Mirror the real structure completely                        |
| Need cleanup only tests use        | Put it in test utilities                                    |
| Watch mock setup balloon           | Switch to an integration test with real components          |
| Finish a test file                 | Run the mutation check                                      |

## Warning Signs

- Setup and assertion share the same object, guaranteeing equality
- The test can fail only through a panic, crash, or missing selector
- The test fails on every intentional change, never on accidental breakage
- Expected values are hidden behind loops, builders, or helpers
- The test greps source text, or asserts a removed symbol stays removed
- The test would still matter if only the framework remained
- The test exists for coverage, checking no side effect or outcome
- An assertion checks a `*-mock` test ID, or fails if you remove the mock
- A method is called only from test files
- Mock setup is more than half the test, or you can't explain why the mock is needed
- Mocking "just to be safe"
