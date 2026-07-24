# Experiment 020: Training Eligibility API Second Matched Run

## Request

Repeat the same `POST /training-eligibility-evaluations` request used in
Experiments 018 and 019 in another fresh Spring Boot project. The API required
HTTP 201, a generated UUID, eligibility from a module threshold and assessment,
a derived decision, notification-consent passthrough, and an exact HTTP 400
validation response.

This run tested the matcher-level authorization wording introduced after
Experiment 019. It also provided a third cost sample for the same request.

## Environment

- Claude Code with Red Green Prune plugin `v2662f652b765`
- Claude Opus 4.8
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `.\gradlew.bat test`

Captured session metrics:

| Metric | Value |
| --- | ---: |
| Context used | 74.4k / 1m tokens |
| Session cost | $2.73 |
| API duration | 7 minutes 13 seconds |
| Wall duration | 40 minutes 55 seconds |
| Model output | 30.1k tokens |
| Cache read | 2.8m tokens |
| Cache write | 56.4k tokens |
| Code change | 205 lines added, 11 removed |

The reported “72% minimal-change” and “79% red-green-prune” figures were
24-hour local usage characteristics, not this session's cost allocation.

## Observed behavior

### Test-first behavior was preserved

The entry point loaded `test-first-cycle` and `minimal-change`, but not
`test-prune`. The existing harness ran before new behavior work.

An obsolete Spring Boot 3 `@WebMvcTest` import caused compilation failure. The
agent correctly classified it as setup failure, found the Spring Boot 4
package, fixed the test, and reran before claiming RED.

Production changes followed observed failing tests for:

- the endpoint and HTTP 201 response;
- eligibility with qualifying and non-qualifying cases;
- both decision values;
- both notification-consent values;
- generated, non-constant UUID values;
- negative-input validation with an exact error response.

No persistence, dependency, configuration, service, extra endpoint,
placeholder value, or speculative extension mechanism was added. The resulting
tests covered distinct public behavior and plausible constant implementations.

### Matcher authorization failed again

The validation test asserted HTTP 400 followed by the exact JSON body.
Production returned 201, so execution stopped at the status matcher. The agent
still authorized and implemented `@Valid`, `@Min(0)`, and the exact-body
exception handler in one GREEN.

Under the matcher-level policy from Experiment 019, this was a failure because
the body assertion had not executed. The more general wording did not change
the behavior.

The agent's explanation exposed the underlying disagreement:

```text
This is the validation-contract rule (status + exact body together).
```

That interpretation is reasonable under ordinary TDD. The test existed before
production, represented one invalid-response contract, failed for missing
validation behavior, and went green after only that contract was implemented.
Classic RED/GREEN does not require every ordered assertion to fail separately.

Experiments 018–020 therefore show that the unstable element was not test-first
behavior. It was the repository's stricter evaluation rule requiring
matcher-by-matcher authorization.

### Reporting remained variable

The final response used the four requested report headings but remained
multi-line, added an extra requirement paragraph, and listed files. Output was
lower than Experiment 018 but materially higher than Experiment 019.

Two post-template runs provide a weak efficiency signal, not stable compliance.
Adding further reporting prose would repeat the same instruction loop, so the
reset removes the per-cycle output protocol and asks for RED evidence and final
verification once.

## Three-run comparison

| Metric | Experiment 018 | Experiment 019 | Experiment 020 |
| --- | ---: | ---: | ---: |
| Session cost | $2.89 | $2.35 | $2.73 |
| Model output | 32.7k | 24.1k | 30.1k |
| Context used | 78.6k | 71.6k | 74.4k |
| API duration | 7m 52s | 5m 43s | 7m 13s |
| Wall duration | 16m 47s | 33m 36s | 40m 55s |
| Lines added | 202 | 224 | 205 |

The two post-template runs averaged about 12% lower cost, 17% lower output,
7% lower context, and 18% lower API duration than Experiment 018. Their average
wall duration was about 122% higher. Three stochastic executions are
insufficient to attribute these changes to one wording revision.

The repeated setup discovery and different test implementations also contributed
variance. Future evaluation should compare several runs and change policy only
after the same original-goal failure repeats.

## Revised expected behavior

Evaluate Red Green Prune against its original objectives:

- run a focused test scope before editing production;
- confirm it fails because requested behavior is missing;
- implement only behavior asserted by that scope and requested by the user;
- allow multiple assertions or cases when they jointly describe one observable
  contract;
- reject setup, unrelated, flaky, and pre-existing failures as RED;
- avoid unrequested production code and redundant tests;
- verify focused and relevant existing tests.

Do not require each matcher or assertion to produce a separate RED. Do not
change the skill after one stochastic deviation from a non-core reporting
preference.

## Result

Experiment 020 is a pass under the original test-first and test-economy goals.
It is a failure only under the matcher-level proof system added during later
iterations. Repeating that policy did not stabilize behavior and conflicted
with token, time, and cycle-efficiency goals.

The evidence motivates a subtractive reset from matcher authorization to
failing test scopes in
[PR #30](https://github.com/irerin07/red-green-prune/pull/30).
