# Experiment 016: Delivery Eligibility API

## Request

Build `POST /delivery-eligibility-evaluations` in a fresh Spring Boot project.
The response had independent requirements for HTTP 201, generated UUID,
signature-consent passthrough, an eligibility predicate combining an inclusive
distance boundary with membership, a fee derived from eligibility, and
positive-distance validation.

The request prohibited persistence, dependencies, configuration, services,
additional endpoints, placeholder response values, and speculative extension
mechanisms. This was the first field run after the responsibility split in
PR #24 and tested whether the thin entry point selected the focused skills
without losing cycle integrity.

## Environment

- Claude Code with Red Green Prune plugin `vc09f3673c6db`
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`
- Reported duration: 10 minutes 3 seconds
- Model, token count, and cost were not captured

The agent ran the empty harness before the first RED and inspected the Spring
Boot 4 test packages before writing the first test.

## Observed behavior

### Focused routing succeeded

The thin entry point loaded exactly:

```text
red-green-prune:test-first-cycle
red-green-prune:minimal-change
```

Both loaded successfully. `test-prune` was not loaded because duplicate-test
cleanup was not in scope. The agent still performed a short final overlap
review without importing the detailed pruning policy. This is the intended
selective-loading behavior.

### Cycles 1 through 5 were separated

The agent separately drove:

1. HTTP 201;
2. signature passthrough;
3. generated UUID;
4. the distance-or-member eligibility rule;
5. the eligibility-to-fee mapping and scale-two JSON output.

Multi-case scopes were used only where the cases jointly distinguished one
rule: opposite boolean values distinguished passthrough from a constant, three
eligibility cases pinned the inclusive boundary and membership branch, and two
fee cases pinned the result mapping. Production responses grew only as each
selected rule required them. No placeholder response values were introduced.

The UUID test parsed two generated values and confirmed they differed, which
distinguished valid generation from a constant UUID. The final refactor
consolidated duplicate request construction while the full suite remained
green.

### Cycle 6 lacks captured RED evidence

For non-positive distance validation, the transcript shows the test being
added and then immediately shows production edits adding `@Positive` and
`@Valid`. It does not show a test execution between those actions. A test run
appears only after the production edits.

The final report claims this RED:

```text
Status expected:<400> but was:<201>
```

That claim is not supported by the captured execution order. If the interface
omitted a tool event, the run is unverifiable from the artifact; otherwise the
agent reported a RED it had not observed. Under either interpretation, Cycle 6
cannot count as confirmed test-first evidence.

This does not identify missing policy. `test-first-cycle` already requires the
scope to run and fail before production editing.

### Explicit HALF_UP wording was reinterpreted

The request required monetary values with scale 2 using `HALF_UP`. Because the
only fee values were exact `BigDecimal` literals (`0.00` and `15.00`), the
agent omitted `setScale(2, HALF_UP)` as a no-op and tested raw JSON scale
instead.

The output requirement was satisfied, but the named implementation mechanism
was not used. This is an interpretation tension rather than evidence for a new
rule: `minimal-change` already preserves agreed contracts. A future run should
honor an explicit mechanism or resolve the conflict during ALIGN instead of
redefining it only in the final report.

### Cost remains unproven

The run took 10 minutes 3 seconds. It included environment verification,
classpath inspection, six RED/GREEN cycles, a refactor, and a full-suite run.
No token or cost data was captured, and earlier scenarios are not sufficiently
matched for a valid time comparison. The experiment therefore validates
selective loading but not a token, cost, or time improvement.

## Expected behavior

- Load `test-first-cycle` and `minimal-change` for an ordinary behavior change.
- Do not load `test-prune` without overlap or cleanup scope.
- Run and capture each selected test scope before its production edit.
- Treat an unsupported RED claim as missed or unverifiable, never as confirmed
  TDD evidence.
- Keep independently falsifiable rules in separate cycles and use multiple
  cases only when they jointly distinguish one rule.
- Honor explicit implementation constraints or resolve their conflict during
  ALIGN.
- Capture token, cost, and comparable timing data before making efficiency
  claims.

## Result

The responsibility split passes its first routing test: the entry point loaded
only the two required focused skills, and their combined behavior prevented the
batching and placeholder scaffolding seen in Experiment 015.

The run is still a partial process failure because Cycle 6 has no captured
pre-production RED execution. No SKILL.md change is made: the required ordering
is already explicit. Record the failure and repeat the split workflow in a new
scenario to determine whether it recurs.
