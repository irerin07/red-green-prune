# Experiment 009: Commission Calculation API

## Request

Build a new Spring Boot `POST /commission-calculations` endpoint with:

- UUID id and HTTP 201;
- base commission equal to sales amount multiplied by commission rate and
  divided by 100;
- a 50.00 threshold bonus at or above 1000.00, otherwise 0.00;
- a non-negative adjustment passed through to the response;
- total commission equal to base commission plus threshold bonus plus
  adjustment;
- separate exact HTTP 400 contracts for non-positive sales amount and
  commission rate, and negative adjustment;
- BigDecimal values with 2-decimal HALF_UP rounding;
- no new persistence, dependency, configuration, endpoint, or speculative
  mechanism.

The request supplied concrete discriminating examples for HALF_UP
(`10.005 -> 10.01`) and both sides of the threshold
(`999.99 -> 0.00`, `1000.00 -> 50.00`). It specified no cycle count, guard
classification, pruning procedure, test command, or reporting format.

## Environment

- Claude Code with Red Green Prune at main commit `6f349b2fc3f2`, including
  PR #13
- Existing Spring Boot project after the Payroll Calculation implementation
- Existing MockMvc and validation error conventions

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

The agent used seven RED/GREEN cycles for base commission, threshold bonus,
adjustment passthrough, total commission, and three validation rules. It used
`0.00` placeholders to preserve later value-mismatch REDs.

Several behaviors improved over Experiment 008:

- the HALF_UP example produced a discriminating `10.01` assertion;
- adjustment passthrough used `5.00` rather than the already-correct
  `0.00` placeholder;
- the at-threshold case received its own observed RED;
- calculation and validation rules remained in separate cycles;
- no speculative architecture, dependency, or unrelated change was added;
- the reported full suite passed with 47 tests across 8 classes.

After implementing the threshold branch, the agent added the
`999.99 -> 0.00` case as a passing test and labeled it a guard. That
classification contradicted the skill: the case protected behavior introduced
by the current task, not pre-existing agreed behavior. It should have been
reported as a missed RED under the test-level model.

The output inspection also found repeated contract-evidence gaps:

- `jsonPath("$.id").exists()` still did not prove UUID format;
- validation tests still asserted one error path instead of the exact response
  body;
- the first cycle combined HTTP 201, id presence, and base-commission
  calculation behind one 404;
- adjustment zero was accepted by a valid request, but its response value was
  not asserted.

The production implementation used UUID, BigDecimal, HALF_UP,
`@PositiveOrZero`, and the specified formulas. The finding concerns workflow
classification and observable test evidence.

## Expected behavior

Treat the selected rule's test scope, rather than each individual case, as the
unit of RED evidence. Multiple cases may belong to one scope when they jointly
distinguish the same boundary or meaningful equivalence rule.

For the threshold rule, run the `999.99` and `1000.00` cases in one focused
scope before GREEN. The scope is valid RED because the at-threshold case fails;
the already-passing below-threshold case is a scope member, not a separate RED
or guard.

Before GREEN, also confirm that the selected scope distinguishes a correct
implementation from plausible nearby defects such as a wrong comparator, UUID
format, rounding mode, or response shape.

## Review follow-up

Post-run review identified that the below-threshold case was behavior introduced
by the current task and therefore could not be classified as a guard. It also
identified that multiple cases may jointly define one rule, making the RED
scope—not each individual test—the correct unit of evidence.

The run validates the need for scope-level classification. It does not yet show
that an agent will derive plausible nearby defects without concrete examples in
the request; that remains the next forward-test target.

Review also found that changing the evidence unit affects two adjacent guard
rules. The skill therefore updates the pre-GREEN gate, passing-test
classification, and missed-RED wording together rather than adding an isolated
policy sentence.

## Result

Guard relabeling repeated after Experiment 006, while UUID and exact-response
evidence gaps repeated after Experiments 007 and 008. The failures were therefore
observed rather than hypothetical.

This experiment produced
[PR #16 — Validate RED at the test-scope level](https://github.com/irerin07/red-green-prune/pull/16).
The change introduces a pre-GREEN falsifiability gate and makes scope members,
guards, and missed REDs mutually exclusive.

The gate's scope mechanics are evidence-backed. Its ability to make an agent
derive plausible nearby defects without prompt-provided examples remains
unverified and is reserved for the next experiment.
