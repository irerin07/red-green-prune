# Experiment 027: Unresolved behavior decisions

## Context

- Base evidence: real-project user, post, and comment lifecycle work
- Trigger: review of the tests expected after creating, reading, updating, and
  deleting a stateful resource

## Observed behavior

Across the field runs, agents improved RED sequencing and state observation but
still selected unspecified public behavior before the user had decided it.
Examples included whether deleting a missing resource should fail or succeed
idempotently and whether a soft-deleted resource should remain available to
other operations.

Writing a focused test did not solve this problem. Either plausible outcome
could be encoded test-first, so the test could silently convert the agent's
assumption into the contract. Reporting the assumption after implementation
also arrived too late for the user to choose.

The existing workflow created two structural gaps:

- ALIGN was task-level but did not define the task-wide decisions it should
  collect;
- the workflow was written as one linear cycle even though agents executed
  multiple behavior scopes.

Repeated implementation patterns also risked being treated as contracts even
when no test, user-facing documentation, or repository-wide policy established
them.

## Analysis

A full test list should not be designed during ALIGN. Tests remain products of
the RED/GREEN cycles. Decision discovery instead needs two levels:

- ALIGN identifies choices that span the task and batches unresolved questions;
- a gate before each RED identifies preconditions that change the selected
  behavior's expected outcome.

An explicit contract must directly settle the choice. A test or user-facing
documentation applicable to the resource can do so, as can an explicit
repository-wide policy. Repeated code shape alone cannot.

When a decision appears after ALIGN, the affected behavior must pause and the
question must be raised immediately. Work may continue only when it is clearly
independent; uncertainty stops progress rather than authorizing an assumption.

## Resulting changes

- Represent behavior work as
  `ALIGN -> (RED -> GREEN -> REFACTOR)* -> PRUNE`.
- Collect task-wide unresolved choices during ALIGN.
- Define explicit contracts once in ALIGN for later RED gates.
- Before each RED, identify outcome-changing preconditions and ask when the
  request or an explicit contract does not settle different assertions.
- Prevent tests and final reports from laundering an agent-selected requirement.
