---
name: test-first-cycle
description: >
  Enforce test-first development for features, bug fixes, APIs, calculations,
  validation, and other observable behavior changes. Require a focused test
  scope to fail because behavior is missing before editing production code.
---

# Test First Cycle

## ALIGN

Read the affected production path and nearest tests. Find the test command and
state the observable result. Resolve repository facts yourself. Ask only when
plausible answers materially change public behavior, data, security,
compatibility, persistence, or meaningful UX.

For destructive behavior, ask before choosing hard deletion, soft deletion,
retention, anonymization, or recovery semantics unless an existing contract
for the same resource or an explicit repository-wide policy settles them. A
deletion pattern for another resource does not settle the choice.

For a new project, establish only the minimum conventional runnable test
harness and run it once before the first RED. Setup is not GREEN: do not
implement requested behavior while creating the harness.

In an established repository without runnable tests, do not add a framework
unless requested. Perform the strongest available check and state the limit.

## RED

Select or write the smallest test scope that expresses the requested behavior.
Prefer an existing failing test, then a case or assertion in the nearest test,
and create a focused test only when needed.

Run the scope before editing production. Confirm it fails because the behavior
is missing. Setup, syntax, flaky, unrelated, and pre-existing failures are not
RED. A missing symbol or load failure is valid only when the expected
production boundary does not exist. Production DTOs, methods, stubs, routes,
and adapters are production code; do not add them first just to make RED
compile.

A scope may contain multiple assertions or cases when they jointly describe one
observable contract. Use extra cases only to distinguish a meaningful boundary
or plausible constant implementation; do not create a full matrix by default.

If no valid RED can be observed, state why instead of claiming TDD.

## GREEN

Load `minimal-change`. Implement only behavior asserted by the failing scope
and requested by the user. It is valid to satisfy all assertions in that scope
in one GREEN. Run the focused scope and confirm it passes.

Do not claim test-first for behavior implemented before its test scope was
observed failing.

## REFACTOR AND REPORT

Refactor only while tests are green and only when the current change needs it.
Run the relevant tests afterward.

Report the RED command and reason, final verification, material assumptions,
and anything not verified once. Do not replay every cycle.
