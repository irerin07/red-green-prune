# Experiment 002: Quote API

## Request

In an existing Java Spring Boot project:

> Build an API that accepts a product name, unit price, quantity, and an optional
> discount code in the request body. Generate and return a quote containing an
> id, subtotal, discount amount, and final total. The discount code SAVE10 gives
> a 10% discount. Reject invalid input.

The user clarified that an unknown discount code must return HTTP 400 with an
`errors.discountCode` entry.

## Environment

- Claude Code with the Red Green Prune skill
- Existing Spring Boot project with MockMvc tests
- Existing `{ "errors": { field: message } }` validation contract

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

ALIGN correctly identified the public ambiguity for unknown discount codes and
asked before implementation.

During validation work, however, the agent introduced tests for three rules and
implemented them in one RED/GREEN cycle:

- blank product name;
- non-positive unit price;
- non-positive quantity.

Each rule could fail while the others passed. A RED was observed, but the cycle
did not isolate one independently failing behavior at a time.

The completed project reported six quote tests and twelve tests in the full
suite, all passing.

## Expected behavior

Choose one independently failing validation rule, observe RED, implement the
minimum GREEN change, then repeat for the next rule.

## Result

Together with the repeated observation in
[Experiment 003](003-tax-calculation-api.md), this produced the PR #8 rule:

> Do not batch independently failing rules into one RED. If each rule could fail
> while the others pass, choose one, reach GREEN, then repeat.

Related change:
[PR #8 — Separate independently failing RED cases](https://github.com/irerin07/red-green-prune/pull/8).
