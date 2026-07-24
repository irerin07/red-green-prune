# Experiment 021: Training Eligibility Test-Scope Reset

## Request

Repeat the same `POST /training-eligibility-evaluations` request used in
Experiments 018–020 in a fresh Spring Boot project. The API required HTTP 201,
a generated UUID, eligibility from a module threshold and assessment, a derived
decision, notification-consent passthrough, and an exact HTTP 400 validation
response.

This run tested the test-scope reset merged in
[PR #30](https://github.com/irerin07/red-green-prune/pull/30). The reset removed
matcher-level GREEN authorization and restored the original requirement: run a
focused test scope before production, observe failure because requested behavior
is missing, and implement only the requested behavior asserted by that scope.

## Environment

- Claude Code with Red Green Prune plugin `v4a6c17931307`
- Claude Opus 4.8
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`

Captured session metrics:

| Metric | Value |
| --- | ---: |
| Context used | 52.3k / 1m tokens |
| Session cost | $0.92 |
| API duration | 2 minutes 36 seconds |
| Wall duration | 11 minutes 15 seconds |
| Model output | 11.4k tokens |
| Cache read | 579.8k tokens |
| Cache write | 34.3k tokens |
| Code change | 136 lines added, 1 removed |

The reported “92% minimal-change” and “98% red-green-prune” figures were
24-hour local usage characteristics, not this session's cost allocation.

## Observed behavior

### Test-first behavior passed

The entry point loaded `test-first-cycle` and `minimal-change`. The existing
harness ran and passed before behavior work.

An obsolete Spring Boot 3 `AutoConfigureMockMvc` import caused a compilation
failure. The agent correctly classified it as setup failure, found the Spring
Boot 4 package, fixed the test, and reran before claiming RED.

The agent wrote five API tests before editing production. After the setup fix,
the focused class failed because the endpoint was absent. Production was then
implemented in one GREEN and the focused class passed. The full suite also
passed, including the pre-existing `contextLoads` test.

The tests protected distinct requested behavior:

- HTTP 201, UUID-shaped id, qualifying eligibility, decision, and the false
  notification-consent path;
- the module threshold;
- the assessment half of the eligibility conjunction;
- the true notification-consent path;
- negative-module validation with the exact HTTP 400 body.

No persistence, dependency, configuration, service, extra endpoint, placeholder
response value, or speculative extension mechanism was added. The production
change was one controller with request and response records plus the
controller-local validation handler needed by the specified error response.

### One broad scope replaced matcher ceremony

All five tests were selected and run together. Because the endpoint was absent,
they failed at the HTTP boundary before their deeper assertions could execute.
The agent then implemented the complete requested endpoint in one GREEN.

This is broader than a micro-cycle for each rule, but it satisfies the reset's
original-goal criteria: the tests existed before production, the selected scope
failed for missing requested behavior, production implemented only behavior
asserted by that scope and requested by the user, and the resulting tests were
neither redundant nor speculative.

Treating the blocked body assertions as unauthorized would recreate the
matcher-level proof system rejected by Experiments 018–020. The run therefore
does not justify restoring per-assertion or per-matcher authorization.

### Efficiency improved materially

| Metric | Experiment 020 | Experiment 021 | Change |
| --- | ---: | ---: | ---: |
| Session cost | $2.73 | $0.92 | -66% |
| Model output | 30.1k | 11.4k | -62% |
| Context used | 74.4k | 52.3k | -30% |
| API duration | 7m 13s | 2m 36s | -64% |
| Wall duration | 40m 55s | 11m 15s | -73% |
| Lines added | 205 | 136 | -34% |

A single stochastic run cannot attribute every improvement to the wording
change. However, the run removed the repeated matcher-authorization ceremony
while preserving the original test-first, test-economy, and minimal-change
outcomes.

## Result

Experiment 021 passes the original Red Green Prune objectives:

- a runnable baseline was confirmed;
- a focused test scope was observed failing before production changed;
- setup failure was not mislabeled as RED;
- only requested and asserted production behavior was implemented;
- tests were distinct and economical;
- focused and full verification passed;
- no speculative production structure was added.

No skill change results from this run. Keep the skill frozen and use additional
field runs to determine whether broad scopes cause an original-goal failure,
rather than treating micro-cycle preference as a failure by itself.
