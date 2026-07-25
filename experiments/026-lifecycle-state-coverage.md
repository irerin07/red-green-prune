# Experiment 026: Lifecycle-state coverage

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

## Analysis

The repeated failure was not a demand for exhaustive combinations. The missing
cases were existing lifecycle states that changed the requested outcome:

- active versus absent for update;
- active versus soft-deleted for comment update.

Reading only the nearest happy path or the requested method is insufficient for
stateful resources. ALIGN and RED need to consider the resource's already
implemented lifecycle, while remaining bounded to states that materially change
the requested result.

## Resulting change

Add one rule to `test-first-cycle`:

> For stateful resources, cover each existing lifecycle state that materially
> changes the requested outcome, such as absent, active, or deleted, without
> building a full state matrix.

This preserves focused tests while preventing a feature-local RED from ignoring
a known lifecycle state. No pruning, minimal-change, or mutation requirement is
added.
