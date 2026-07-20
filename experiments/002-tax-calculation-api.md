# Experiment 002: Tax Calculation API

## Request

In the same Java Spring Boot project:

> Build an API that accepts an item name, unit price, quantity, and tax rate in
> the request body. Generate and return a tax calculation containing an id,
> subtotal, tax amount, and final total. Reject invalid input.

The user clarified that `taxRate` is a percent number, so `8` means 8%.

## Environment

- Claude Code with the Red Green Prune skill
- Existing Spring Boot project with MockMvc tests
- Existing quote and user endpoints
- Existing `{ "errors": { field: message } }` validation contract

The exact model version and wall-clock timing were not captured, so this record
does not compare model performance or execution speed.

## Observed behavior

ALIGN correctly asked how to interpret `taxRate`, a decision that changes the
computed public result.

The first cycle observed HTTP 404 before adding the endpoint and then reached
GREEN.

The second cycle introduced four independently failing validation rules at
once:

- blank item name;
- non-positive unit price;
- non-positive quantity;
- negative tax rate.

All four returned HTTP 201 before validation and were then implemented together
in one GREEN change.

During the same RED phase, the agent added a `taxRate = 0` boundary test. It
passed immediately. The test protected the agreed behavior that zero tax is
allowed, but it was not RED evidence.

The completed project reported six tax tests and eighteen tests in the full
suite, all passing.

## Expected behavior

Run a separate RED/GREEN cycle for each independently failing validation rule.
A newly added passing characterization test may guard behavior that the
upcoming GREEN change could regress, but it must not be reported as RED.

## Result

This repeated the batching found in
[Experiment 001](001-quote-api.md) and supplied direct evidence for both PR #8
rules:

> Do not batch independently failing rules into one RED. If each rule could fail
> while the others pass, choose one, reach GREEN, then repeat.

> Do not add a new passing test during RED unless it protects agreed behavior
> that the upcoming GREEN change could otherwise regress. Report it as a guard,
> not as RED.

Related change:
[PR #8 — Separate independently failing RED cases](https://github.com/irerin07/red-green-prune/pull/8).
