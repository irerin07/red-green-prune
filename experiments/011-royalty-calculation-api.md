# Experiment 011: Royalty Calculation API

## Request

Build a new Spring Boot `POST /royalty-calculations` endpoint with:

- UUID id and HTTP 201;
- base royalty equal to gross revenue multiplied by royalty rate and divided by
  100;
- a 40.00 milestone bonus at or above 3000.00, otherwise 0.00;
- a non-negative manual bonus passed through to the response;
- total royalty equal to base royalty plus milestone bonus plus manual bonus;
- separate exact HTTP 400 contracts for non-positive gross revenue and royalty
  rate, and negative manual bonus;
- BigDecimal values with 2-decimal HALF_UP rounding;
- no persistence, dependency, configuration, additional endpoint, or
  speculative extension mechanism.

The request supplied no discriminating rounding example, no pair of values
around the threshold, and no UUID or exact-response assertion technique. It
prescribed no cycle count, mutation check, pruning procedure, command, or report
format.

## Environment

- Claude Code with Red Green Prune after PR #17
- Fresh Spring Boot 4.1.0 project using Java 21
- Spring Web MVC, validation, and the generated `contextLoads` test only
- No controller, error handler, or earlier experiment implementation to imitate

The exact model version, token usage, and wall-clock timing were not captured,
so this record does not compare model cost or execution speed.

## Observed behavior

The agent ran the empty test harness successfully before the first API test. Its
first test used the wrong Spring Boot 4 `WebMvcTest` package. It correctly
rejected that setup failure as RED, fixed the import, and accepted only the
resulting missing-controller failure as behavioral RED.

PR #17 improved evidence honesty but did not enforce its pre-edit gate.

The first cycle explicitly selected “the happy path” and combined HTTP 201, id
presence, base-royalty calculation, response fields, zero milestone bonus, zero
manual bonus, and total royalty. These rules can fail independently. The agent
did not record the required selected rule, nearby defect, distinguishing scope,
or observed failure checkpoint before editing production code.

The missing controller symbol prevented the request from reaching any response
assertion. The agent nevertheless treated that one absence as RED evidence for
all assertions in the test. In particular, `jsonPath("$.id").exists()` did not
distinguish a UUID from an arbitrary string, and the broad example did not
strictly verify the complete response shape.

Cycle 1 GREEN also introduced correct-looking `0.00` placeholders for
milestone bonus and manual bonus and implemented HALF_UP rounding. The agent
described the placeholders as a way to preserve later REDs, but they already
satisfied parts of later selected rules:

- the below-threshold example returned the specified zero bonus;
- the zero manual-bonus example returned the specified passthrough value;
- HALF_UP was selected before a rounding-sensitive scope ran.

Later cycles separately observed RED for the at-threshold bonus, a non-zero
manual bonus, and each validation contract. Those cycles were focused and the
validation assertions checked the exact field count and message.

For rounding, the agent independently selected
`100.00 * 12.345 / 100 = 12.345 -> 12.35`, which distinguishes HALF_UP from
HALF_EVEN. Because HALF_UP had already been implemented, the test passed on its
first run. The agent explicitly reported a missed pre-implementation RED,
temporarily changed production to HALF_EVEN, observed the test fail, and
restored HALF_UP.

This classification is a genuine PR #17 success: the agent did not relabel the
late mutation as TDD evidence. The mutation still added two avoidable production
edits and test runs solely to compensate for the missed RED.

The reported final suite passed with seven endpoint tests plus the generated
context test. Production code remained small and added no speculative
abstraction, persistence, dependency, configuration, or unrelated endpoint.

## Expected behavior

Before each production edit, emit a concise cycle checkpoint naming:

1. one selected rule;
2. one plausible nearby defect;
3. the smallest distinguishing test scope and command;
4. the observed failure caused by the missing selected rule.

When an absent endpoint, route, class, or symbol blocks the scope, count RED
only for the selected assertion that distinguishes its named nearby defect.
Assertions that were not reached do not become RED evidence merely because the
same request stopped earlier.

Scaffolding for the selected rule must not hard-code specified outputs,
constants, branches, annotations, formats, or policies for later rules. Leave
later behavior incomplete until its own gate.

If a rule was implemented before its scope observed RED, report the missed RED.
Do not run a production mutation merely to compensate unless the user requested
it or the risk independently justifies the additional check. A later mutation
remains sensitivity evidence, not TDD evidence.

## Result

PR #17 successfully prevented evidence laundering: the agent reported the
rounding miss honestly and classified its mutation as sensitivity evidence.

It did not make the gate operational. The agent skipped the checkpoint, reused
one missing-symbol failure for multiple blocked rules, and used placeholders to
implement future behavior before its gate.

This experiment therefore supports a compact execution-level follow-up:
require the checkpoint in a fixed format, limit absent-boundary RED to the
selected reachable assertion, reject future-rule placeholders, and avoid
unrequested mutation performed only to compensate for a missed RED.
