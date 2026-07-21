# Experiment 006: Shipping Quote API

## Request

Build a new Spring Boot `POST /shipping-quotes` endpoint with:

- UUID id and HTTP 201 for valid requests;
- a 5.00 base fee through 1.00 kg;
- 2.00 for each started kilogram above 1.00 kg;
- a 10.00 express fee when requested, otherwise 0.00;
- final fee equal to base fee plus express fee;
- exact HTTP 400 error for non-positive weight;
- BigDecimal money with 2-decimal HALF_UP rounding;
- no new persistence, dependency, configuration, or speculative mechanism.

The prompt specified functional behavior only. It did not prescribe cycle count,
guard handling, pruning, test commands, or reporting.

## Environment

- Claude Code with Red Green Prune after PR #11
- Existing Spring Boot project with user, quote, and tax APIs
- Existing MockMvc and validation error conventions

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

The agent used two RED/GREEN cycles:

1. one “happy-path calculation” cycle for endpoint creation, UUID, overweight
   base-fee calculation, express=true pricing, total calculation, and HTTP 201;
2. one cycle for non-positive weight validation.

After both production changes, it added passing boundary cases for the started-
kilogram rule and express=false, then labeled them guards. No later production
change was pending.

It also added a `0.50 -> 5.00` row and removed it during PRUNE using hypothetical
mutation reasoning. Neither the removed row nor its survivor had observed RED
for that defect, so the existing PRUNE evidence condition was not satisfied.

The implementation was functionally correct, reused existing validation and
error handling, added no speculative architecture, and left the full suite
green.

## Expected behavior

The base-fee rule, started-kilogram boundary behavior, express pricing, and
validation can fail independently. Cycle selection should therefore follow the
smallest independently falsifiable rule or meaningful equivalence class rather
than the endpoint or “happy path.”

Tests for behavior introduced by the task should observe RED before
implementation. They should not be added afterward and relabeled as guards.

## Result

This skill-driven run exposed two classification escapes:

- grouping independently falsifiable rules under a “happy path” name;
- relabeling test-after coverage for newly introduced behavior as guards.

It produced
[PR #13 — Close happy-path batching and guard relabeling escapes](https://github.com/irerin07/red-green-prune/pull/13).

No new PRUNE wording was added because the existing rule already required
observed RED evidence.
