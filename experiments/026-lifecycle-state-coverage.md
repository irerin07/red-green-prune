# Experiment 026: Lifecycle-state coverage and unresolved choices

## Context

- Plugin version: `v09e351e92665`
- Environment: an existing Spring Boot application with service-level tests
- Scenarios: comment update after soft deletion, followed by a review of the
  existing user sign-up and update tests

## Observed behavior

The comment update run correctly observed the changed comment content, showing
that the state-change evidence rule from experiment 025 worked. It tested an
active comment and a missing comment, but did not inspect the resource's
existing soft-deleted state. Production therefore allowed a deleted comment to
be updated and reported that limitation only after implementation.

A focused follow-up request added a failing test for a deleted comment. The test
observed the missing guard, and GREEN required one repository-result filter.
The focused and full suites then passed with 19 tests.

A later review of `UserServiceTest` found the same class of omission in older
coverage: update tested an active user but not an absent user. The sign-up test
also asserted only the generated id, not the fields sent to persistence.

## Characterization repair

The repair left production code unchanged and:

- captured the saved user to assert email and phone number;
- added the absent-user update case;
- reported both additions as green-on-arrival characterization coverage rather
  than claiming new RED evidence;
- temporarily swapped persisted fields to confirm the new sign-up assertions
  detected the defect, then restored production;
- finished with seven user-service tests and 20 full-suite tests passing.

The temporary mutation supplied optional sensitivity evidence. It was not
test-first evidence and is not made mandatory by this experiment.

## Requirement-decision gap

Review across the resource lifecycle exposed a separate failure. Agents often
selected an unspecified outcome by writing its test, then reported the choice
as an assumption after implementation. For example, a request to delete a
missing user leaves at least two plausible public contracts: reject it or
succeed idempotently. A test for either outcome can be test-first while still
encoding an unauthorized design decision.

The existing ALIGN instruction said to ask when plausible answers materially
changed public behavior, but it did not operationally separate:

- task-wide choices, such as what deletion means for the resource;
- behavior-local preconditions, such as absent, already deleted, or partial
  input;
- executable or documented contracts from repeated implementation patterns.

It also presented the workflow as one linear RED/GREEN pass despite real tasks
requiring multiple focused cycles.

## Analysis

The repeated failure was not a demand for exhaustive combinations. The missing
cases were existing lifecycle states that changed the requested outcome:

- active versus absent for update;
- active versus soft-deleted for comment update.

Reading only the nearest happy path or the requested method is insufficient for
stateful resources. At the same time, predefining a full test list would recreate
the test-bloat problem. The bounded model is:

- ALIGN once for task-wide choices;
- a decision gate before each RED for the selected behavior;
- repeated RED, GREEN, and REFACTOR cycles;
- PRUNE once after the requested behavior is complete.

## Resulting changes

- Require coverage of existing lifecycle states that materially change the
  requested outcome without building a full state matrix.
- Define an explicit contract as a directly applicable existing test,
  user-facing documentation, or repository-wide policy; repeated code patterns
  alone are not contracts.
- Ask task-wide unresolved choices together before the first RED.
- Before each RED, surface unresolved behavior-local choices immediately and
  pause affected or ambiguously dependent work.
- Represent the workflow as
  `ALIGN -> (RED -> GREEN -> REFACTOR)* -> PRUNE`.

These changes prevent a test from silently choosing the requirement while
preserving focused scopes. They add no mandatory mutation testing and do not
expand pruning or minimal-change responsibilities.
