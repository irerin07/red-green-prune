# Experiment 015: Program Eligibility API

## Request

Build `POST /program-eligibility-evaluations` in a fresh Spring Boot project.
The response had independent requirements for HTTP 201, UUID generation, an
eligibility predicate combining an inclusive age boundary with residency,
decision-string mapping, and contact-consent passthrough.

The request excluded validation, arithmetic, persistence, and infrastructure.
Compared with Experiment 014, it added only one predicate condition. This tested
whether cycle separation remained reliable when the behavior became slightly
more complex.

## Environment

- Claude Code with Red Green Prune after PR #20
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC test support already installed
- Only the generated `contextLoads` test existed
- Test command: `.\gradlew.bat test`
- Model, token count, cost, and duration were not captured

The agent ran the empty harness before the first RED. It initially used the old
Spring Boot package for `@WebMvcTest`, correctly rejected the compilation
failure as setup-related, corrected the import, and reran the test.

## Observed behavior

The agent recorded actual `rule`, `scope`, and `failure` after each run and
before each production edit. It separated HTTP status, UUID generation, contact
consent, and the individual eligibility equivalence classes. It added no
service, dependency, validation, persistence, configuration, extra endpoint, or
speculative extension mechanism. The full suite passed.

However, two existing rules were violated.

### Manufactured future RED

During the HTTP-status GREEN, the agent explicitly said it would return default
or unproven field values so later rules would fail, then introduced the complete
response structure with:

```java
new EligibilityResponse(null, false, null, false)
```

The selected cycle required only HTTP 201. An empty response could have
satisfied it without creating response fields or placeholder values. The
production structure was therefore added to manufacture later failures rather
than because the current GREEN required it.

### Eligibility and decision batching

The qualifying, underage, and non-resident tests each asserted both `eligible`
and `decision`. In the qualifying cycle, `$.eligible` failed first, so the
following decision assertion was blocked and did not supply RED evidence for
decision mapping. The GREEN still implemented both:

```java
boolean eligible = ...;
String decision = eligible ? "ELIGIBLE" : "NOT_ELIGIBLE";
```

The final report justified the missing decision cycle by saying decision was
covered by the eligibility cases and that a separate test would add bloat. This
confused final coverage with test-first order. A separate cycle did not require
a new test method: after eligibility became green, a decision assertion could
have been added to an existing test, observed failing, and then implemented.

The final tests covered both decision strings, so the resulting regression
coverage was adequate. The violation was the lack of observed decision RED
before production implementation.

The eligibility cases themselves were valid distinct equivalence classes:
qualifying adult resident, underage resident, and adult non-resident. Handling
them in separate cycles was acceptable because the age and resident clauses
could fail independently.

No task-new test was labeled as a guard in this run. Guard relabeling did not
recur, but the decision rule was implemented without its own RED instead.

## Expected behavior

- Return an empty 201 response for the status cycle; do not create placeholder
  fields to arrange future failures.
- Drive the eligibility predicate without decision mapping.
- After eligibility is green, add or expose a decision assertion, observe its
  failure, and only then implement decision mapping.
- Reuse an existing test method if that avoids test-count growth; cycle
  separation concerns evidence order, not the number of test methods.
- Continue to handle the age and residency clauses as independently falsifiable
  equivalence classes.

## Result

Experiment 015 is a process failure despite correct final behavior and tests.
The batching observed in Experiment 013 recurred after only a modest increase
from the clean Experiment 014 scenario. Placeholder construction also returned
with an explicit intent to make later tests fail.

No SKILL.md sentence is added: both behaviors are already prohibited. The
014/015 contrast supports an attention and responsibility-boundary problem
rather than missing policy. After this evidence is merged, the next change
should mechanically split cycle integrity, minimal implementation, and test
pruning into independently triggered skills with a thin entry point. This evidence was recorded in
[PR #23 — Record program eligibility experiment](https://github.com/irerin07/red-green-prune/pull/23).
