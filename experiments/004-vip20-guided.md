# Experiment 004: VIP20 Guided Compliance

## Request

Extend the existing Spring Boot `POST /quotes` endpoint with `VIP20`:

- apply a 20% discount when subtotal is at least 100.00;
- return an exact `errors.discountCode` response below 100.00;
- preserve SAVE10, absent-code, and unknown-code behavior;
- keep BigDecimal 2-decimal HALF_UP rounding;
- add no persistence, dependencies, endpoints, or speculative mechanisms.

The prompt also prescribed the workflow: use separate RED/GREEN cycles for the
eligible and below-minimum behaviors, inspect existing coverage before adding
guards, run focused and full tests, and report cycle evidence.

## Environment

- Claude Code with Red Green Prune
- Existing Spring Boot project with user, quote, and tax APIs
- Existing data-driven `DiscountCode` enum and MockMvc quote tests

The exact model version was not captured. The transcript reported 5 minutes
41 seconds, but this single observation is not a benchmark.

## Observed behavior

The agent followed the prescribed workflow:

1. It found existing tests for SAVE10, absent code, and unknown code and added
   no duplicate guards.
2. Cycle A observed VIP20 rejected as unknown, then added the 20% code and
   reached GREEN.
3. Cycle B observed a below-minimum request returning 201, then added the
   minimum-subtotal rule and exact 400 response.
4. It ran the focused quote suite after each cycle and the full suite at the
   end.

The implementation reused the existing enum, added a minimum subtotal per code,
and mapped a small domain exception through the existing error contract. No new
dependency, endpoint, persistence, or configuration was added.

The final run reported eight quote tests and twenty tests in the full suite,
all passing.

## Expected behavior

This experiment tested whether the agent could comply with a fully specified
workflow and functional contract. Because the prompt explicitly required cycle
separation, guard inspection, and reporting, success cannot be attributed to
the skill alone.

## Result

The run passed as a guided compliance test and established the implementation
baseline for [Experiment 005](005-vip25-skill-driven.md). It produced no skill
change by itself.
