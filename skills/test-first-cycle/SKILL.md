---
name: test-first-cycle
description: >
  Enforce honest test-first cycles for observable behavior changes. Use for
  features, bug fixes, APIs, calculations, validation, and other production
  behavior that must follow ALIGN -> RED -> GREEN. Separate independently
  falsifiable rules, observe real RED before production, and classify passing
  scope members, guards, and missed RED without manufacturing failures.
---

# Test First Cycle

Repeat `ALIGN -> RED -> GREEN` for one independently falsifiable rule at a
time. Do not claim TDD unless RED was observed before the production change.

## ALIGN

Before editing:

1. Read the affected production path, callers, and nearest tests.
2. Find the repository's test command and conventions.
3. State the observable success condition.
4. Resolve repository facts yourself.
5. Ask only when plausible answers would change public behavior, data,
   security, compatibility, persistence, or meaningful UX.

Use a conventional, reversible default for minor ambiguity and report it. Do
not ask the user about implementation details discoverable from the repository.

## RED

Use the smallest test change that exposes the missing behavior:

1. Run the nearest existing test. If it already fails for the expected
   behavioral reason, use that failure as RED and add no test.
2. Otherwise, add one case or assertion to the nearest existing test when it
   preserves that test's behavioral purpose.
3. Add one focused test only when no existing test can express the requirement
   clearly.

Run it before changing production code. Confirm it fails because the requested
behavior is missing at the expected boundary. For code that does not yet exist,
a missing-symbol failure, including the resulting compilation or load failure
at that boundary, is valid RED. Setup, syntax, and unrelated failures are not.

A passing existing test is not RED. If relevant tests already pass, determine
whether the behavior exists or the new requirement is not yet expressed.

For a bug, reproduce the failure through the closest stable public boundary.
Use a lower-level test only when the public test would be slow, unreliable, or
unable to isolate the regression.

Do not write the full test matrix before the first GREEN. Add the smallest
independently falsifiable rule or meaningful equivalence class at a time. An
endpoint, happy path, calculation, or assertion group is not one rule when any
part could fail while others pass. Shared implementation and later extension
do not justify batching.

For each cycle, state the rule and smallest distinguishing scope, run it, then
record before editing production:
`RED observed — rule: ...; scope: ...; failure: ...`.
Continue only if the scope distinguishes correct behavior from a plausible
nearby defect and fails because the rule is missing. Split independently
failing rules and repeat after GREEN. Use multiple cases only when they jointly
distinguish one boundary or meaningful equivalence rule. The executed command
is evidence; do not restate it.

When an absent endpoint, route, class, or symbol blocks the scope, count RED
only for the selected assertion whose nearby defect the scope distinguishes.
Assertions blocked behind that absence are not RED evidence for their rules.

Do not implement, revert, or mutate production code to manufacture RED. If the
behavior was already implemented, report the missed RED. Do not run a mutation
merely to compensate unless the user requested it or the risk independently
justifies it; any later mutation is sensitivity evidence, not TDD evidence.

Do not add a new passing test during RED unless it belongs to the current
rule's test scope or protects agreed behavior that the upcoming GREEN change
could otherwise regress. Report the former as a passing scope member, not as a
separate RED or guard; report the latter as a guard.

A test protecting behavior introduced by the current task is not a guard. Its
scope must have observed RED before implementation. If that behavior was
implemented without an observed RED for its scope, report the missed RED
instead of relabeling the test as a guard.

If RED cannot be observed, state why; never fabricate it.

## GREEN transition

After a valid RED, load and follow `minimal-change` for the production edit.
Run the focused scope and confirm GREEN before selecting the next independently
falsifiable rule.

If the production edit unavoidably satisfies a later rule, report missed RED
for that later scope. Do not manufacture a failure or claim it was test-first.

## Test environment

If no runnable test environment exists in an established repository, do not
introduce one unless requested. Perform the strongest available check and state
the limitation.

For a new project, establish the minimum runnable test environment during
ALIGN before the first RED. Ask the user when the choice materially determines
the project stack; otherwise prefer the ecosystem's standard or already
selected test tooling. Run the empty test harness once before the first RED to
confirm the environment itself works. Establishing the environment is not
GREEN: do not implement requested behavior while setting it up.

Do not treat a flaky failure as RED. Separate unrelated pre-existing failures
from the requested work.

## Report

Report only applicable cycle evidence:

- material assumption or clarification;
- RED command and expected failure;
- GREEN command and passing result;
- guards or passing scope members that were added;
- missed RED or anything not verified.
