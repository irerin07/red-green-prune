# Experiment 001: User API

## Request

In a newly scaffolded Java Spring Boot project:

> Build an API that accepts a name, email, phone number, and date of birth in
> `yyyy/MM/dd` format, then generates user data.

The user clarified that invalid input must return HTTP 400 with
`{ "errors": { field: message } }`, and that the response must contain a
generated id plus the submitted fields.

## Environment

- Claude Code with the Red Green Prune skill
- Spring Boot 4.1.0, Java 21, Gradle, and MockMvc
- Existing empty test harness with one `contextLoads` test
- No persistence dependency

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

The agent ran the empty harness before the first behavioral RED. The first test
attempt used an outdated `AutoConfigureMockMvc` import and failed to compile.
The agent correctly rejected that setup failure as RED, fixed the import, then
observed the expected HTTP 404 at the public boundary.

The agent used separate RED/GREEN cycles for the happy path, invalid email, and
a missing required field.

One cycle added two date cases before GREEN:

- wrong format: `1990-05-15`;
- valid format but impossible date: `1990/02/30`.

These cases detect different defects: a format-only check can reject the first
while accepting the second. They therefore deserved distinct coverage, but
because either rule could fail while the other passed, the current skill would
run them as separate RED/GREEN cycles.

The missing-name RED was followed by adding `@NotBlank` to all four fields at
once. That implemented the agreed required-fields policy, but it also showed
why “one behavior at a time” needed a more operational boundary when field
rules can fail independently.

The completed project reported five user-controller tests and six tests in the
full suite, all passing. No live `bootRun` or curl smoke test was performed.

## Expected behavior

Reject setup or compilation mistakes as RED evidence, establish a behavioral
failure at the expected boundary, and isolate independently failing rules into
separate RED/GREEN cycles.

Keep the two date tests because they protect distinct failure modes; test
retention and cycle granularity are separate decisions.

## Result

This run provided the baseline evidence that the agent could distinguish a
setup failure from behavioral RED and complete a real test-first API flow. It
also exposed the cycle-granularity ambiguity later repeated more clearly in
[Experiment 002](002-quote-api.md) and
[Experiment 003](003-tax-calculation-api.md).

The repeated behavior produced:
[PR #8 — Separate independently failing RED cases](https://github.com/irerin07/red-green-prune/pull/8).
