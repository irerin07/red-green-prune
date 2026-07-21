# Experiment 007: Hotel Quote API

## Request

Build a new Spring Boot `POST /hotel-quotes` endpoint with:

- UUID id and HTTP 201;
- room total equal to nightly rate multiplied by nights;
- breakfast fee equal to 12.00 multiplied by guests and nights when enabled,
  otherwise 0.00;
- final total equal to room total plus breakfast fee;
- separate exact HTTP 400 contracts for non-positive nightly rate, nights, and
  guests;
- BigDecimal money with 2-decimal HALF_UP rounding;
- no new persistence, dependency, configuration, or speculative mechanism.

The prompt specified functional behavior only. It did not prescribe cycle count,
guard handling, pruning, test commands, or reporting.

## Environment

- Claude Code with Red Green Prune at main commit `6f349b2fc3f2`, including
  PR #13
- Existing Spring Boot project after the Shipping Quote implementation
- Existing MockMvc and validation error conventions

The transcript reported 3 minutes 41 seconds. The exact model version was not
captured, and this single observation is not a benchmark.

## Observed behavior

The agent improved in several areas:

- breakfast=false received its own RED/GREEN cycle;
- nightlyRate, nights, and guests validation each received separate cycles;
- no post-implementation test was relabeled as a guard;
- no speculative architecture or dependency was added;
- focused tests and the full suite passed.

However, the first cycle was explicitly named “the happy path” and combined
independently falsifiable behavior:

- HTTP 201;
- id presence;
- room-total calculation;
- breakfast=true calculation;
- final-total calculation.

This directly contradicted the PR #13 rule that an endpoint, happy path,
calculation, or assertion group is not one behavior when parts can fail
independently. The final report nevertheless claimed “one behavior per cycle.”

Read-only inspection of the resulting test also found coverage gaps:

- `jsonPath("$.id").exists()` does not prove the id is a UUID;
- validation tests assert one error field but do not strictly prove the exact
  response body;
- all monetary examples divide evenly, so the tests do not distinguish HALF_UP
  from another rounding mode.

The production implementation itself uses UUID and HALF_UP and appears
functionally correct. The finding concerns observable test evidence.

## Expected behavior

Select the first cycle below the “happy path” label. Room-total calculation,
breakfast pricing, final-total calculation, and UUID format can fail
independently and should not be introduced behind one RED.

Verify public contracts with assertions capable of detecting their violation:
UUID shape for the id, strict JSON when the response must be exact, and a
rounding example that distinguishes HALF_UP.

## Result

PR #13 partially worked: guard relabeling stopped, breakfast=false was isolated,
and validation batching disappeared. Happy-path batching still occurred despite
an explicit rule naming that escape.

No immediate skill change followed. Because the rule already states the desired
behavior, adding another equivalent sentence would increase context without
resolving the compliance problem. If the same failure repeats in another
independent run, prefer restructuring RED into a short pre-run gate over adding
more policy prose.
