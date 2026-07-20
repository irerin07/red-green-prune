# Experiment 005: VIP25 Skill-Driven Evaluation

## Request

Extend the same `POST /quotes` endpoint with `VIP25`:

- apply a 25% discount when subtotal is at least 200.00;
- return an exact `errors.discountCode` response below 200.00;
- preserve SAVE10, VIP20, absent-code, and unknown-code behavior;
- keep BigDecimal 2-decimal HALF_UP rounding;
- add no persistence, dependencies, endpoints, or speculative mechanisms.

Unlike [Experiment 004](004-vip20-guided.md), the prompt did not instruct the
agent how to structure RED/GREEN cycles, guards, test commands, pruning, or its
report. Those behaviors had to come from Red Green Prune.

## Environment

- Claude Code with the Red Green Prune version produced by PR #8
- Existing Spring Boot project after the VIP20 change
- Existing data-driven `DiscountCode` enum and MockMvc quote tests

The exact model version was not captured. The transcript reported 3 minutes
7 seconds, but this single observation is not a benchmark.

## Observed behavior

The agent independently inspected existing behavior, added no duplicate guards,
ran focused tests, ran the full suite, and avoided new machinery.

It also explicitly identified two genuine failures:

- eligible VIP25 returned HTTP 400 because the code was unknown;
- below-minimum VIP25 returned the unknown-code message instead of the required
  minimum-subtotal message.

Despite calling them distinct failures, the agent wrote both tests before
GREEN and enabled both behaviors with one enum entry. It justified batching
with two arguments:

> both behaviors are enabled by the same one-line entry

> an artificial throwaway intermediate

It also invoked the global omission litmus to argue that an intermediate
implementation satisfying only the first selected behavior should be skipped.

The final implementation was functionally correct and reported ten quote tests
and twenty-two tests in the full suite, all passing. The failure was procedural:
two independently failing behaviors were combined into one RED/GREEN cycle.

## Expected behavior

Shared implementation does not make independently failing behaviors one
behavior. The expected progression was:

1. observe the eligible VIP25 failure and add VIP25 with no minimum;
2. observe the below-minimum failure and set its minimum to 200.00.

The intermediate implementation is the minimum GREEN for the first selected
behavior, not throwaway code. The final production code is the same; only the
observed cycle evidence differs.

## Result

This skill-driven run exposed an escape hatch between RED and GREEN:

- RED required independently failing behaviors to be separated;
- GREEN referred ambiguously to all agreed behavior;
- the agent used shared implementation and efficiency arguments to override
  per-cycle scoping.

The observation produced
[PR #11 — Close the shared-implementation RED escape](https://github.com/irerin07/red-green-prune/pull/11).
