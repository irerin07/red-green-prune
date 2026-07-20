---
name: red-green-prune
description: >
  Enforce test-first development without test bloat. Clarify material
  requirements, observe a meaningful RED, implement the minimum GREEN change,
  refactor only proven problems, and prevent redundant tests in the current
  change without deleting historical coverage. Use for TDD, test-first
  features, bug fixes, behavior changes, excessive tests, over-engineered
  implementations, and annotations without observable value. Do not require
  RED for documentation, formatting, generated files, mechanical renames, or
  deletion of proven-dead code.
---

# Red Green Prune

For observable behavior changes, follow:

`ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE`

Do not claim TDD unless RED was observed before the production change.

Apply this litmus throughout: if removing an addition changes no agreed
behavior, contract, required check, or distinct regression signal, omit it.

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

Do not write the full test matrix before the first GREEN. Add one meaningful
behavior at a time. Choose the smallest independently falsifiable rule or
meaningful equivalence class for a cycle. Do not use an endpoint, happy path,
calculation, or group of assertions as one behavior when any part could fail
while the others pass.

Do not batch independently failing rules into one RED. If each rule could fail
while the others pass, choose one, reach GREEN, then repeat. Shared
implementation does not merge them into one behavior. An implementation that
satisfies only the selected behavior is not throwaway merely because a later
cycle will extend or replace it.

Do not add a new passing test during RED unless it protects agreed behavior
that the upcoming GREEN change could otherwise regress. Report it as a guard,
not as RED. A passing test added after implementation is a guard only when a
later production change could regress that behavior. Otherwise it is
test-after: test the behavior before implementation and observe RED.

If RED cannot be observed, state why; never fabricate it.

## GREEN

Write the smallest production change satisfying the failing test and the
agreed behavior selected for the current cycle. Preserve existing contracts.

Reuse repository code, standard-library functionality, native platform
behavior, and installed dependencies before adding infrastructure. Do not add
speculative abstractions, extension points, configuration, dependencies,
fallbacks, annotations, or unrelated cleanup.

An annotation is justified only when runtime, framework behavior, static
analysis, serialization, validation, persistence, dependency injection, an
external contract, or an explicit repository rule requires it. Neighboring
code is evidence to investigate, not a reason to copy boilerplate.

Run the focused test and confirm GREEN.

## REFACTOR

Refactor only while tests are green and only to remove duplication of the same
policy, correct a misleading name, simplify control flow, or reuse an existing
pattern with functional value.

Similar shape is not sufficient. Extract behavior only when it has the same
meaning and reason to change. Keep the refactor within the current behavior
unless broader cleanup was requested. Run the focused test afterward.

## PRUNE

Prevent test bloat within the current change. Do not delete, disable, or weaken
a pre-existing test during ordinary feature or bug work. Report suspected
historical duplicates separately.

Keep a test when it uniquely protects a behavior, meaningful equivalence
class, boundary, regression, security rule, data-loss risk, contract, or
execution path.

Merge or remove tests introduced during this task only when all are true:

- both tests protect the same requirement and meaningful path;
- the same production defect would make both fail;
- the observed RED shows the surviving test detects that defect;
- removal loses no distinct diagnostic value.

Use the observed RED plus a structural comparison of setup, public path,
assertion, and failure reason as evidence. Green-after-deletion alone is not
proof. Keep the test whose failure was observed as RED; if another test should
survive instead, run that test against the pre-GREEN behavior and observe its
failure before deleting the original. If equivalent sensitivity is unclear,
keep both tests.

Parameterization removes test-code duplication, not necessarily behavioral
duplication. Retain cases representing different equivalence classes or
failure modes.

Prune pre-existing tests only when the user explicitly requests test cleanup.
For security, authorization, money, persistence, destructive behavior,
data-loss prevention, or historical regression tests, present evidence and
obtain approval before deletion.

After pruning tests introduced during this task, run the surviving focused
test and the relevant test scope.

## Worked example

During the current task, the agent adds two tests for this requirement:
username lookup must be case-insensitive.

Before:

```python
def test_finds_username_with_uppercase_query(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("ALICE").username == "alice"


def test_finds_username_with_mixed_case_query(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("AlIcE").username == "alice"
```

Both tests use the same setup, public path, normalization behavior, result,
and failure reason. Keep one:

```python
def test_username_lookup_is_case_insensitive(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("ALICE").username == "alice"
```

The RED already observed before GREEN shows that the surviving test detects
the missing normalization. No additional mutation ceremony is needed for two
tests introduced together with identical paths and failure reasons.

Do not prune this test:

```python
def test_username_lookup_does_not_expose_another_tenant(repo):
    ...
```

It calls the same method but protects a different authorization regression.
Likewise, keep an additional Unicode case-folding case if it distinguishes
`casefold()` from ASCII-only normalization.

## Exceptions

Skip RED for documentation, formatting, generated-file synchronization,
behavior-preserving mechanical renames, deletion of proven-dead code, and
metadata synchronization with a deterministic validator. Make the smallest
change and run the smallest relevant verification; do not invent a behavioral
test merely to satisfy the workflow.

If no runnable test environment exists in an established repository, do not
introduce one unless requested. Perform the strongest available check and state
the limitation. For a new project, establish the minimum runnable test
environment during ALIGN before the first RED. Ask the user when the choice
materially determines the project stack; otherwise prefer the ecosystem's
standard or already selected test tooling. Run the empty test harness once
before the first RED to confirm the environment itself works. Establishing the
environment is not GREEN: do not implement requested behavior while setting it
up.

Do not treat a flaky failure as RED. Separate unrelated pre-existing failures
from the requested work.

## Report

Stop when the agreed behavior and relevant checks pass. Report only applicable
evidence:

- material assumption or clarification;
- RED command and expected failure;
- GREEN command and passing result;
- tests pruned and the evidence supporting the decision;
- anything not verified.

Do not implement optional improvements after the requirement is satisfied.
