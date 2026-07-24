---
name: test-first-cycle
description: >
  Enforce honest test-first cycles for observable behavior changes. Use for
  features, bug fixes, APIs, calculations, validation, and other production
  behavior that must follow ALIGN -> RED -> GREEN. Observe a real failure,
  authorize only the GREEN it proves, and report guards or missed RED honestly.
---

# Test First Cycle

Repeat `ALIGN -> RED -> GREEN` for one independently falsifiable rule at a
time. Do not claim TDD unless RED preceded its production change.

## ALIGN

Before editing:

1. Read the affected production path, callers, and nearest tests.
2. Find the repository's test command and conventions.
3. State the observable success condition.
4. Resolve repository facts yourself.
5. Ask only when plausible answers materially change public behavior, data,
   security, compatibility, persistence, or meaningful UX.

Use a conventional, reversible default for minor ambiguity and report it. Do
not ask about implementation details discoverable from the repository.

For a new project, establish only the minimum conventional runnable test
harness before the first RED. Ask when the choice materially determines the
stack; otherwise use already selected or ecosystem-standard tooling. Run the
empty harness once. Setup is not GREEN: do not implement requested behavior.

If an established repository has no runnable test environment, do not add one
unless requested. Perform the strongest available check and state the limit.

## RED

Select one rule that could fail while other requested rules pass. Shared
implementation, one endpoint, one happy path, or one test method does not make
independent rules a single rule.

Prefer, in order:

1. an existing test already failing for the expected behavioral reason;
2. one case or assertion added to the nearest suitable test;
3. one focused new test.

For a bug, reproduce the failure through the closest stable public boundary.
Use a lower-level test only when the public test would be slow, unreliable, or
unable to isolate the regression.

Choose the smallest scope that distinguishes correct behavior from a plausible
nearby defect. Use multiple cases only when they jointly distinguish one rule.
For generated or mirrored values, reject plausible constant implementations.
Do not write the full test matrix before the first GREEN.

Run the scope before editing production. Setup, syntax, flaky, unrelated, and
pre-existing failures are not RED. A missing symbol, route, class, or endpoint,
including its compilation or load failure, is valid only for the boundary it
proves. Assertions blocked behind it prove nothing.

Before GREEN, record exactly:

```text
RED: <command> -> <first relevant failure>
AUTHORIZED GREEN: <only production behavior that failure proves>
```

`AUTHORIZED GREEN` must be the smallest change capable of resolving the first
relevant failure. It cannot include behavior the run did not reach. A 404
authorizes only the route/status boundary; blocked response fields,
calculations, and rules require later RED cycles.

If a requested production change is not in `AUTHORIZED GREEN`, do not implement
it in this cycle. Reach GREEN, then expose the next rule.

A passing test is not RED. A passing case may belong to the current rule's
scope or protect pre-existing agreed behavior that GREEN could regress. Report
the latter as a guard. Task-new behavior is never a guard; if implemented
without prior RED, report missed RED.

Never implement, revert, or mutate production merely to create failure. Do not
use mutation to compensate unless the user requests it or independent risk
justifies it; any later mutation is sensitivity evidence, not TDD evidence.

If RED cannot be observed, state why. Never fabricate it.

## GREEN transition

Load and follow `minimal-change`. Implement only `AUTHORIZED GREEN`, run the
focused scope, and confirm GREEN before selecting the next rule. If the minimum
change unavoidably satisfies another rule, report missed RED for that rule.

## Report

During work, emit only the two RED authorization lines and the GREEN result per
cycle. Finish with one compact cycle summary, material assumptions, guards or
missed RED, and anything not verified. Do not repeat the transcript.
