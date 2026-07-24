# Experiment 019: Training Eligibility API Matched Run

## Request

Repeat Experiment 018's `POST /training-eligibility-evaluations` request in
another fresh Spring Boot project. The API required HTTP 201, a generated UUID,
eligibility from a module threshold and assessment result, a derived decision,
notification-consent passthrough, and an exact HTTP 400 validation response.

This matched run kept the prompt, model, framework, Java version, dependencies,
and initial project shape aligned with Experiment 018. Its primary purpose was
to test whether the bounded Report template reduced output while preserving the
authorization gate.

## Environment

- Claude Code with Red Green Prune plugin `v77be43446420`
- Claude Opus 4.8
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `.\gradlew.bat test`

Captured session metrics:

| Metric | Value |
| --- | ---: |
| Context used | 71.6k / 1m tokens |
| Session cost | $2.35 |
| API duration | 5 minutes 43 seconds |
| Wall duration | 33 minutes 36 seconds |
| Model output | 24.1k tokens |
| Cache read | 2.4m tokens |
| Cache write | 53.6k tokens |
| Code change | 224 lines added, 26 removed |

The reported “69% minimal-change” and “75% red-green-prune” figures were
24-hour local usage characteristics, not this session's cost allocation.

## Observed behavior

### Focused routing still worked

The entry point loaded `test-first-cycle` and `minimal-change`, but not
`test-prune`. The agent ran the existing harness first. An obsolete Spring
Boot 3 `@WebMvcTest` import caused a compilation failure; the agent correctly
treated it as setup failure, found the Spring Boot 4 package, and reran without
claiming RED.

A later attempt to use Jackson's `ObjectMapper` also failed because that class
was not available on this test classpath. The agent switched to regex extraction
for UUID comparison. This tooling detour contributed output and makes the two
runs matched prompts rather than deterministic executions.

### Authorization passed through cycle 5

The first test observed 404 and authorized only an endpoint returning 201. The
GREEN did not add a response body or business behavior.

Separate reachable RED cycles then drove:

- generated UUID output, including different values across two requests;
- the eligibility predicate with qualifying and non-qualifying cases;
- both decision values;
- both notification-consent values.

No placeholder response fields were introduced. These cycles retained the main
authorization and sensitivity improvements from Experiment 018.

### Validation batched a blocked assertion

The final test asserted HTTP 400 and then an exact JSON error body. Production
returned HTTP 201, so MockMvc stopped at the status matcher. The body assertion
was not reached.

The recorded authorization nevertheless included both:

```text
validate completedModules >= 0
on violation return 400 with the exact error body
```

GREEN added `@Valid`, `@Min(0)`, and the validation exception handler
together. Only the status behavior had been proven by the first relevant
failure; the exact body required another RED after status became 400.

Experiment 018 correctly separated the same requirement into status and body
cycles. The matched rerun therefore shows that the gate's 404-specific example
did not reliably generalize to other earlier matcher failures, despite the
broader blocked-assertion policy already being present.

The resulting change replaces the 404 example rather than adding policy:

```text
A failed matcher authorizes only what makes it pass; blocked assertions require
later RED.
```

This applies the same reachability rule to status, body, field, calculation, and
other ordered assertions.

### Reporting improved but was not fully bounded

The final answer used the required `VERIFIED`, `ASSUMPTIONS`,
`GUARDS/MISSED RED`, and `NOT VERIFIED` labels and did not generate another
large table. This coincided with lower model output, session cost, context use,
and API duration.

Compliance was partial. The fields remained multi-line and detailed, an extra
requirements paragraph followed `NOT VERIFIED`, and cycle narration during
work exceeded the three evidence lines. Because the format showed a measurable
improvement in its first forward test, the Report section remains unchanged
until another run establishes whether the remaining verbosity is stable.

## Efficiency comparison

| Metric | Experiment 018 | Experiment 019 | Change |
| --- | ---: | ---: | ---: |
| Session cost | $2.89 | $2.35 | -19% |
| Model output | 32.7k | 24.1k | -26% |
| Context used | 78.6k | 71.6k | -9% |
| API duration | 7m 52s | 5m 43s | -27% |
| Wall duration | 16m 47s | 33m 36s | +100% |
| Lines added | 202 | 224 | +11% |

This is the closest comparison so far because the user prompt and environment
were intentionally matched. It is still one stochastic rerun: test
implementation differed, the Jackson detour added work, and wall duration moved
opposite the API metrics. The measurements support a reporting-efficiency
signal, not a causal performance claim.

## Expected behavior

- Preserve the successful route-only first GREEN.
- For every ordered assertion, authorize only the first failed matcher.
- Run again after that matcher passes before authorizing blocked assertions.
- Preserve constant-sensitive UUID and passthrough tests.
- Keep the Report template unchanged for the next forward test.
- Compare performance across multiple matched runs before claiming improvement.

## Result

Experiment 019 is a partial pass. Reporting became shorter and cheaper on the
measured API dimensions, and authorization remained focused through five
cycles. The validation cycle implemented a blocked error-body contract from a
status-only RED, proving that matcher reachability needed to be stated
generally rather than through a 404 example.

The evidence and one-for-one wording replacement are recorded in
[PR #29](https://github.com/irerin07/red-green-prune/pull/29).
