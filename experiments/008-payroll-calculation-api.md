# Experiment 008: Payroll Calculation API

## Request

Build a new Spring Boot `POST /payroll-calculations` endpoint with:

- UUID id and HTTP 201;
- regular pay for up to 40 hours;
- overtime pay above 40 hours at 1.5 times the hourly rate;
- bonus pay equal to the submitted non-negative bonus;
- gross pay equal to regular pay plus overtime pay plus bonus pay;
- separate exact HTTP 400 contracts for non-positive hourly rate and hours
  worked, and negative bonus;
- BigDecimal inputs and money with 2-decimal HALF_UP rounding;
- no new persistence, dependency, configuration, endpoint, or speculative
  mechanism.

The prompt specified functional behavior only. It did not prescribe cycle count,
guard handling, pruning, test commands, or reporting.

## Environment

- Claude Code with Red Green Prune at main commit `6f349b2fc3f2`, including
  PR #13
- Existing Spring Boot project after the Hotel Quote implementation
- Existing MockMvc and validation error conventions

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

The agent used seven RED/GREEN cycles:

1. regular pay capped at 40 hours;
2. overtime pay above 40 hours;
3. bonus pay passthrough;
4. gross pay calculation;
5. non-positive hourly-rate validation;
6. non-positive hours-worked validation;
7. negative-bonus validation.

Cycles 2 through 4 used deliberate `0.00` placeholders. Each selected
calculation therefore produced an observed value-mismatch RED before its
production implementation. Each validation rule also received its own observed
RED. No post-implementation test was relabeled as a guard, no speculative
architecture was added, and the reported full suite passed with 40 tests across
7 classes.

This was a substantial improvement over Experiments 006 and 007: the agent
explicitly applied the independently-falsifiable-rule criterion and did not
group the calculation or validation rules under one happy-path cycle.

The first cycle still combined HTTP 201, id presence, and regular-pay
calculation behind one missing-endpoint 404. Read-only inspection of the
resulting tests also found contract-evidence gaps:

- `jsonPath("$.id").exists()` does not prove the id is a UUID;
- no exact-40-hours case proves overtime is zero at the threshold;
- the negative-bonus test cannot distinguish `@PositiveOrZero` from
  `@Positive`, because no `bonus = 0` success case exists;
- validation tests assert one error field but do not strictly prove the exact
  response body;
- all monetary examples divide evenly, so the tests do not distinguish HALF_UP
  from another rounding mode.

The production implementation uses UUID, BigDecimal, HALF_UP,
`@PositiveOrZero`, and the expected formulas. The finding concerns observable
test evidence rather than an observed functional defect.

## Expected behavior

Continue selecting independently falsifiable rules as separate RED/GREEN
cycles. For every specified public contract, use an assertion and input capable
of distinguishing the required behavior from a plausible incorrect
implementation.

For this request, that includes UUID shape, the exact 40-hour and zero-bonus
boundaries, strict JSON where the body must be exact, and a rounding example
that distinguishes HALF_UP.

## Result

PR #13 materially improved cycle scoping. Calculation and validation batching
did not repeat, and guard relabeling remained absent. A narrow first-cycle scope
issue remained around endpoint scaffolding, and the larger remaining weakness
was contract falsifiability: several specified properties were implemented but
not proven by discriminating tests.

No skill change follows from this run. The existing cycle rule worked for the
main calculations, while the evidence gaps identify a separate candidate for
future evaluation. If the same contract-falsification pattern repeats, evaluate
a short operational RED gate rather than adding overlapping policy prose.
