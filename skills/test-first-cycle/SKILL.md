---
name: test-first-cycle
description: >
  Enforce honest test-first cycles for features, bug fixes, APIs,
  calculations, validation, and other observable behavior changes. Observe a
  real RED, authorize only the GREEN it proves, and report guards or missed RED
  honestly without manufacturing failures.
---

# Test First Cycle

Repeat `ALIGN -> RED -> GREEN` for one independently falsifiable rule at a
time. Do not claim TDD unless RED preceded its production change.

## ALIGN

Read the affected production path, callers, and nearest tests. Find the test
command and conventions, state observable success, and resolve repository facts
yourself. Ask only when plausible answers materially change public behavior,
data, security, compatibility, persistence, or meaningful UX. Use and report a
conventional reversible default for minor ambiguity.

For a new project, establish only the minimum conventional runnable test
harness. Ask when its choice materially determines the stack; otherwise use
already selected or ecosystem-standard tooling. Run the empty harness once.
Setup is not GREEN: do not implement requested behavior.

In an established repository without runnable tests, do not add a framework
unless requested. Perform the strongest available check and state the limit.

## RED

Select one rule that could fail while other requested rules pass. Shared
implementation, one endpoint, one happy path, or one test method does not merge
independent rules.

Prefer an existing test already failing for the expected behavioral reason.
Otherwise add one case or assertion to the nearest suitable test; add one
focused test only when neither option expresses the rule clearly.

For a bug, use the closest stable public boundary unless it is slow,
unreliable, or cannot isolate the regression.

Choose the smallest scope that distinguishes correct behavior from a plausible
nearby defect. Use multiple cases only when they jointly distinguish one rule.
For generated or mirrored values, reject plausible constants. Do not write the
full test matrix before the first GREEN.

Run before editing production. Setup, syntax, flaky, unrelated, and
pre-existing failures are not RED. A missing symbol, route, class, or endpoint,
including its compilation or load failure, proves only that boundary.
Assertions blocked behind it prove nothing.

Before GREEN, record exactly:

```text
RED: <command> -> <first relevant failure>
AUTHORIZED GREEN: <only production behavior that failure proves>
```

`AUTHORIZED GREEN` is the smallest change capable of resolving the first
relevant failure. It cannot include behavior the run did not reach. A failed matcher
authorizes only what makes it pass; blocked assertions require later RED. Do not implement anything outside the authorization.

A passing test is not RED. A passing case may belong to the current rule or
protect pre-existing behavior that GREEN could regress; report the latter as a
guard. Task-new behavior is never a guard. Report missed RED if it was
implemented without prior RED.

Never implement, revert, or mutate production to create failure. Use later
mutation only when requested or independently justified by risk; it is
sensitivity evidence, not TDD evidence. If RED cannot be observed, state why.
Never fabricate it.

## GREEN transition

Load `minimal-change`. Implement only `AUTHORIZED GREEN`, run the focused
scope, and confirm GREEN before selecting the next rule. If the minimum change
unavoidably satisfies another rule, report missed RED for it.

## Report

Emit only `RED`, `AUTHORIZED GREEN`, and `GREEN: <command> -> <result>`
until verification. Then emit:

```text
VERIFIED: ...
ASSUMPTIONS: ...
GUARDS/MISSED RED: ...
NOT VERIFIED: ...
```

Omit empty lines; no tables, file lists, or replay.
