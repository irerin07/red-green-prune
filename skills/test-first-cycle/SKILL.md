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
state the observable result. Resolve repository facts yourself.

Identify choices that span the whole task and would require different
assertions. For destructive behavior, this includes hard deletion, soft
deletion, retention, anonymization, or recovery semantics. Resolve them from
the request or an applicable explicit contract; a pattern for another resource
does not settle the choice. Ask unresolved choices together before the first
RED.

An explicit contract is an existing test or user-facing documentation that
directly settles the choice for this resource, or an explicit repository-wide
policy. A repeated implementation pattern alone is not a contract.

For a new project, establish only the minimum conventional runnable test
harness and run it once before the first RED. Setup is not GREEN: do not
implement requested behavior while creating the harness.

In an established repository without runnable tests, do not add a framework
unless requested. Perform the strongest available check and state the limit.

## RED

Before each RED, identify preconditions that could change the expected
outcome—for example, the target is absent, already in the target state, or the
request is partially specified.

If multiple plausible outcomes would require different assertions, resolve the
choice from the request or an applicable explicit contract. Ask only when
neither does.

If this gate finds an unresolved choice, raise it immediately and pause the
affected behavior. Continue only work that is clearly independent of it. If
independence is unclear, stop.

Do not settle it by asserting one of the candidates. Reporting the choice after
implementation is not a substitute for asking.

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

For stateful resources, cover each existing lifecycle state that materially
changes the requested outcome, such as absent, active, or deleted, without
building a full state matrix.

For state-changing behavior, make the failing scope observe the requested state
or the value sent to the persistence boundary. A status, generated identifier,
or mocked return value alone does not prove that requested fields or
relationships were applied.

If no valid RED can be observed, state why instead of claiming TDD.

## GREEN

Load `minimal-change`. Implement only behavior asserted by the failing scope
and requested by the user. It is valid to satisfy all assertions in that scope
in one GREEN. Run the focused scope and confirm it passes.

Do not claim test-first for behavior implemented before its test scope was
observed failing.

## REFACTOR AND REPORT

Refactor only while tests are green and only when the current change needs it.
Run the relevant tests afterward. If requested behavior remains, return to RED
for the next smallest scope.

Report the RED command and reason, final verification, material assumptions,
and anything not verified once after all cycles. Do not replay every cycle.
