# Experiment 017: Benefit Eligibility API

## Request

Build `POST /benefit-eligibility-evaluations` in a fresh Spring Boot project.
The response required HTTP 201, generated UUID, an eligibility predicate
combining an inclusive service boundary with active employment, a decision
string derived from eligibility, and expedited-flag passthrough. Negative
service years had to return HTTP 400.

The request prohibited persistence, dependencies, configuration, services,
additional endpoints, placeholder response values, and speculative extension
mechanisms. This run tested whether the split workflow would preserve the
routing success from Experiment 016, separate eligibility from its derived
decision, and observe a real RED for late validation.

## Environment

- Claude Code with Red Green Prune plugin `vc09f3673c6db`
- Claude Opus 4.8
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`

Captured session metrics:

| Metric | Value |
| --- | ---: |
| Context used | 67.9k / 1m tokens |
| Session cost | $1.98 |
| API duration | 5 minutes 17 seconds |
| Wall duration | 32 minutes 33 seconds |
| Model output | 23.1k tokens |
| Cache read | 1.8m tokens |
| Cache write | 49.9k tokens |
| Code change | 138 lines added, 9 removed |

Claude Code reported about 9.4k tokens of installed skill metadata. The four
Red Green Prune descriptions accounted for approximately 460 tokens:
`test-first-cycle` 130, `test-prune` 120, `minimal-change` 120, and the entry
point 90.

The separate “42% minimal-change” and “47% red-green-prune” figures were
24-hour local usage characteristics, not a per-session cost breakdown. They
must not be attributed solely to this experiment.

## Observed behavior

### Focused routing succeeded again

The entry point loaded exactly:

```text
red-green-prune:test-first-cycle
red-green-prune:minimal-change
```

It did not load `test-prune`. The agent ran the empty harness first. An outdated
Spring Boot 3 `@WebMvcTest` import caused a compilation failure; the agent
correctly classified it as setup-related, located the Spring Boot 4 package,
fixed it, and reran the test.

### Decision and validation improved

The decision mapping received its own RED after eligibility production code
existed. Both `ELIGIBLE` and `NOT_ELIGIBLE` cases failed because the decision
field was absent, then the agent implemented the mapping. The batching from
Experiment 015 did not recur at this boundary.

The final validation cycle also showed a test execution before production
editing. The negative-years test returned 201 instead of 400, after which the
agent added `@Min(0)` and `@Valid`. The unsupported RED gap from Experiment 016
did not recur.

No placeholder response values, service, configuration, persistence,
dependency, handler, or extension mechanism was added.

### HTTP status and eligibility were batched

Cycle 1 was declared as:

```text
Eligibility AND-rule + 201
```

The test sent three eligibility cases, but every request first asserted HTTP
201. With no endpoint, all three stopped at that assertion and returned 404.
The captured RED therefore proved only that the endpoint did not return 201.
It supplied no RED evidence for the eligibility predicate behind the blocked
status assertion.

The GREEN nevertheless created the endpoint and implemented:

```java
eligible = yearsOfService >= 5 && activeEmployee
```

HTTP status and business calculation can fail independently and should have
been separate cycles. This directly violated existing `test-first-cycle`
instructions about independently falsifiable rules and assertions blocked by
an absent endpoint.

This is not missing policy. It shows that the split reduced some failures but
did not make the cycle gate reliable.

### Two tests did not fully distinguish nearby defects

The UUID test parsed one response value as a UUID. A constant valid UUID would
still pass, so it did not distinguish generation from a fixed value.

The expedited test asserted only the `true` case. An implementation that always
returned `true` would pass, so it did not fully prove input passthrough. In
Experiment 016, two UUID responses and both boolean directions covered these
nearby defects; that sensitivity regressed here.

### Efficiency is now measurable but not yet comparative

The session cost $1.98 and produced 23.1k output tokens for a 138-line change.
That is material overhead, but no matched monolithic run exists, so the data
does not prove the split improved or worsened cost.

Installed metadata grew because four descriptions are always present. The
intended saving comes from loading fewer skill bodies, not from metadata, and
`/context all` does not isolate body-token cost per invocation. A controlled
comparison needs the same prompt, project baseline, model, and clean session
under both the monolith and split versions.

## Expected behavior

- Load only `test-first-cycle` and `minimal-change` for ordinary behavior work.
- First drive only the endpoint/status rule when an absent endpoint blocks all
  response assertions.
- After status is green, run the eligibility scope, observe its own failure,
  and then implement the predicate.
- Keep decision mapping in a later cycle, as this run did.
- Run late validation tests before their production annotations, as this run
  did.
- Test generated identifiers against the plausible constant-value defect.
- Test passthrough values in both directions when either constant is plausible.
- Compare efficiency only with a matched baseline.

## Result

Experiment 017 is a partial process failure. Selective routing, placeholder
avoidance, decision separation, and late-validation RED all passed. The first
cycle still batched HTTP status with eligibility even though only the status
assertion produced RED evidence. UUID generation and expedited passthrough also
had incomplete defect sensitivity.

No SKILL.md change is made in this evidence PR. The violated rules already
exist. After recording this run, review whether `test-first-cycle` should be
compressed from explanatory policy into a shorter executable phase gate rather
than adding more prohibition text.
