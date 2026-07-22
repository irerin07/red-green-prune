# Experiment 010: Rebate Calculation API

## Request

Build a new Spring Boot `POST /rebate-calculations` endpoint with:

- UUID id and HTTP 201;
- base rebate equal to purchase amount multiplied by rebate rate and divided by
  100;
- a 30.00 volume bonus at or above 2000.00, otherwise 0.00;
- a non-negative manual credit passed through to the response;
- total rebate equal to base rebate plus volume bonus plus credit;
- separate exact HTTP 400 contracts for non-positive purchase amount and rebate
  rate, and negative manual credit;
- BigDecimal values with 2-decimal HALF_UP rounding;
- no persistence, unrelated endpoint, dependency, configuration, or speculative
  extension mechanism.

The request deliberately supplied no discriminating boundary or rounding
examples. It also prescribed no cycle count, test cases, guard classification,
mutation check, pruning procedure, command, or report format.

A prior run of the same request in the accumulated experiment project was
discarded as a contaminated pilot because earlier tests exposed equivalent
boundary and rounding examples.

## Environment

- Claude Code with Red Green Prune after PR #16
- Fresh Spring Boot 4.1.0 project using Java 17
- Spring Web MVC, validation, and the generated `contextLoads` test only
- No controller, error handler, or prior experiment test to imitate

The exact model version, token usage, and wall-clock timing were not captured,
so this record does not compare model cost or execution speed.

## Observed behavior

The agent first ran the existing harness successfully. Its initial test attempts
then encountered two Spring Boot 4 setup errors: a Jackson package mismatch and
a moved MockMvc autoconfiguration package. It rejected both as invalid RED,
corrected the test harness, and only then accepted the endpoint 404 as
behavioral RED.

Several behaviors validated the PR #16 design:

- the agent parsed the id and passed it to `UUID.fromString` instead of checking
  only for field presence;
- it independently selected `1999.99` and `2000.00` as two cases jointly
  defining the inclusive threshold scope;
- both threshold cases were run before the threshold GREEN;
- it selected a non-zero `15.50` manual credit;
- it selected `8.125 -> 8.13` to distinguish HALF_UP from nearby rounding
  modes;
- validation tests used strict whole-JSON comparison;
- no passing task-new boundary case was relabeled as a guard;
- eight endpoint tests and the existing context test passed;
- the final production code remained small and added no speculative
  infrastructure.

This clean run therefore provides positive evidence that the agent can derive
plausible nearby defects without prompt-provided examples.

Two process failures remained.

First, Cycle 1 explicitly combined HTTP 201, UUID format, and base-rebate
calculation behind one endpoint 404. These contracts can fail independently, so
the cycle still violated the one-selected-rule requirement.

Second, HALF_UP was implemented in Cycle 1 before its discriminating test. The
agent discovered the missing coverage during PRUNE, changed production from
HALF_UP to HALF_DOWN, observed the new test fail, and restored HALF_UP. This
proved test sensitivity but did not recover the missed pre-implementation RED.
The final report nevertheless listed it as an eighth RED/GREEN cycle and claimed
that every RED preceded its production change.

The mutation added avoidable production edits and test runs. Final LOC and test
count remained reasonable, but the run did not capture tokens or time needed to
quantify the overhead.

## Expected behavior

Complete the RED gate before any production edit for each cycle:

1. name one selected rule;
2. identify a plausible nearby defect;
3. build the smallest scope that distinguishes the correct implementation;
4. run the scope and observe at least one failure caused by the missing rule.

Do not implement and revert behavior, or mutate already-correct production code,
to manufacture RED. If a behavior was implemented before its scope failed,
report a missed RED. A later mutation may be reported as sensitivity evidence,
not TDD evidence.

For this request, UUID format, base-rebate calculation, and HTTP status should
not be claimed as one independently falsifiable rule merely because the absent
endpoint initially makes one request fail at 404.

## Result

PR #16 succeeded at scope composition and nearby-defect derivation. The
threshold scope, UUID check, strict JSON comparison, and self-selected rounding
case all improved over Experiments 007 through 009.

It did not reliably enforce the gate before production edits. The late rounding
test also exposed a new evidence-laundering path: implementation followed by
production mutation was reported as RED-before-GREEN.

This experiment produced
[PR #17 — Enforce the RED gate before production edits](https://github.com/irerin07/red-green-prune/pull/17).
The change moves the gate to an explicit pre-edit checkpoint and classifies
later mutation as sensitivity evidence rather than TDD evidence.
