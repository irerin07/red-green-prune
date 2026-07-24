# Experiment 018: Training Eligibility API

## Request

Build `POST /training-eligibility-evaluations` in a fresh Spring Boot
project. A valid response required HTTP 201, a generated UUID, an eligibility
predicate combining a module threshold with an assessment result, a decision
derived from eligibility, and notification-consent passthrough. Negative module
counts had to return an exact HTTP 400 error body.

The request prohibited persistence, dependencies, configuration, services,
additional endpoints, placeholder response values, and speculative extension
mechanisms. This run directly tested the executable authorization gate added
after Experiment 017: an absent endpoint had to authorize only the route and
status, leaving all response behavior for later reachable RED cycles.

## Environment

- Claude Code with Red Green Prune plugin `vf77f5970ee32`
- Claude Opus 4.8
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation support already installed
- Only the generated `contextLoads` test existed
- Test command: `.\gradlew.bat test`

Captured session metrics:

| Metric | Value |
| --- | ---: |
| Context used | 78.6k / 1m tokens |
| Session cost | $2.89 |
| API duration | 7 minutes 52 seconds |
| Wall duration | 16 minutes 47 seconds |
| Model output | 32.7k tokens |
| Cache read | 2.9m tokens |
| Cache write | 60.6k tokens |
| Code change | 202 lines added, 13 removed |

Claude Code reported about 9.4k tokens of installed skill metadata. The
separate “64% minimal-change” and “71% red-green-prune” figures were 24-hour
local usage characteristics, not a per-session cost breakdown.

## Observed behavior

### Focused routing remained correct

The entry point loaded exactly `test-first-cycle` and `minimal-change`. It
did not load `test-prune`. The installed-skill list in `/context all`
contains metadata for all available skills; it does not show that all skill
bodies were loaded.

The agent ran the existing harness before the first behavior test. An outdated
Spring Boot 3 `AutoConfigureMockMvc` import caused compilation failure. The
agent correctly classified it as setup failure, found the Spring Boot 4
package, fixed it, and reran the test without claiming RED.

### The authorization gate corrected Experiment 017

The first behavior test reached HTTP 404 and expected 201. Before production
editing, the agent emitted:

```text
RED: ... endpoint missing -> 404, expected 201
AUTHORIZED GREEN: create POST /training-eligibility-evaluations handler that
returns HTTP 201 (no body required)
```

The GREEN created only a mapped endpoint returning an empty 201 response. It
did not implement eligibility, response fields, validation, or placeholders.
This is the exact separation missing from Experiment 017, where the same 404
was followed by both endpoint creation and eligibility implementation.

Later cycles observed reachable failures before implementing:

- the three-case eligibility scope;
- both decision values;
- both notification-consent values;
- generated identifiers;
- negative-input status;
- the exact validation error body.

Each production edit stayed inside its recorded `AUTHORIZED GREEN`.

### Test sensitivity improved

The eligibility scope used three cases: qualifying input, insufficient modules,
and failed assessment. Together they reject constant results and incomplete
AND implementations.

Notification passthrough tested both `true` and `false`, rejecting either
constant. The UUID test parsed two identifiers and required them to differ,
rejecting a fixed valid UUID. These repair the two sensitivity regressions
observed in Experiment 017.

### Blocked validation assertions were separated correctly

The negative-input test first asserted only HTTP 400 and observed 201. The
authorized GREEN added `@Valid` and `@Min(0)`. Only after status became 400
did the agent add the strict response-body assertion, observe the missing body,
and add the controller-scoped validation handler.

This split follows the same gate principle as the initial 404: an assertion
that execution has not reached cannot authorize production behavior.

### Reporting remained verbose

The functional workflow passed, but output discipline did not. During work the
agent emitted substantial cycle narration beyond the required `RED`,
`AUTHORIZED GREEN`, and `GREEN` lines. The final response replayed all
cycles in a large table, listed files and implementation details, and repeated
verification already present in the transcript. Character rendering also
corrupted the table.

The existing Report section already requested compact output, so another
prohibition would repeat policy that failed. The resulting change replaces the
prose request with a bounded, executable final-output template and explicitly
disallows tables, file inventories, and cycle replays.

## Efficiency comparison

Compared with Experiment 017:

| Metric | Experiment 017 | Experiment 018 | Change |
| --- | ---: | ---: | ---: |
| Session cost | $1.98 | $2.89 | +46% |
| Model output | 23.1k | 32.7k | +42% |
| Context used | 67.9k | 78.6k | +16% |
| API duration | 5m 17s | 7m 52s | +49% |
| Wall duration | 32m 33s | 16m 47s | -48% |
| Lines added | 138 | 202 | +46% |

The run improved correctness and safety but used more output tokens, API time,
and money. This pair is not a controlled benchmark: the prompts and required
implementations differ, and Experiment 018 added stronger sensitivity tests and
an exact error handler. The comparison therefore identifies a cost signal, not
causation. The verbose report is one directly observed source of avoidable
output.

## Expected behavior

- Preserve the authorization gate unchanged.
- Keep loading only `test-first-cycle` and `minimal-change` for ordinary
  behavior work.
- Continue authorizing only route/status after an absent-endpoint RED.
- Continue rejecting constant implementations for generated and mirrored
  values.
- Keep blocked assertions in later cycles.
- Emit no cycle narration beyond the three required evidence lines.
- Finish with bounded verification, assumptions, guards or missed RED, and
  remaining limits; omit empty optional fields and do not replay cycles.

## Result

Experiment 018 is a functional pass and a reporting-efficiency failure. The
authorization gate corrected the central Experiment 017 defect without
placeholders or new policy. Test sensitivity and late validation sequencing
also passed. Reporting remained materially more verbose than instructed, while
token and cost measurements worsened in this unmatched comparison.

The evidence and bounded reporting format are recorded in
[PR #28](https://github.com/irerin07/red-green-prune/pull/28).
