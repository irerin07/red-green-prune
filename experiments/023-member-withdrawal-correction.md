# Experiment 023: Member Withdrawal and Evaluation Correction

## Request

After updating from Experiment 022, invoke `/red-green-prune:red-green-prune`
without a task and provide a member-withdrawal API request in the follow-up.

The existing Spring Boot project contained user sign-up, read, and update
behavior; service-level Mockito tests; and thin controllers without dedicated
HTTP tests.

This run tested the three clarifications introduced after Experiment 022:
follow-up routing, production scaffolding before RED, and the prohibition on
adding a public adapter from a service-level scope.

## Observed behavior

### Follow-up routing passed

After the task arrived, the entry skill loaded both required sibling skills
before inspecting or editing files:

- `test-first-cycle`;
- `minimal-change`.

The follow-up routing clarification worked.

### Production scaffolding stayed behind RED

The agent wrote two service tests before editing production:

- an existing user is deleted;
- a missing user produces `IllegalArgumentException("User not found")`.

The focused run failed because `UserService.withdraw` did not exist. The agent
accepted the expected missing-symbol failure as RED and only then added the
method. No DTO, method stub, throwing placeholder, route, or adapter preceded
that RED.

The production-scaffolding clarification worked.

### Thin-controller testing was over-evaluated

After the service scope passed, the agent added a thin `DELETE /users/{id}`
adapter returning HTTP 204 and explicitly reported that the controller was not
tested. This contradicted the Experiment 022 sentence stating that a service
scope cannot authorize an unasserted public adapter.

The initial review classified this as an original-goal failure. The user then
clarified that thin controllers should not require tests by default. On that
priority, the classification was too strict.

A controller test would have distinct protective value for method, route,
binding, delegation, and status, but distinct value does not make every test
mandatory. Requiring one for every thin adapter conflicts with the project's
test-economy goal. The agent implemented the requested adapter, kept business
behavior under focused service tests, and disclosed the unverified HTTP wiring.
That is an acceptable cost-risk choice when the adapter contains no material
mapping, validation, authorization, error translation, or response logic.

Remove the blanket public-adapter prohibition rather than adding a more detailed
controller-testing rule. Existing reporting already requires anything not
verified to be stated.

### Destructive ALIGN failed

The withdrawal request did not settle deletion semantics. The agent inferred
hard deletion from the current schema and called soft deletion unrequested
persistence. It did not ask about hard deletion, soft deletion, anonymization,
retention, recovery, or related-data handling.

Absence of a soft-delete field does not authorize irreversible data deletion.
This choice materially affects persistence and data-loss behavior, so it should
be resolved during ALIGN unless an existing contract already settles it.

## Resulting change

- Keep the Experiment 022 follow-up-routing clarification.
- Keep the production-scaffolding clarification; the run demonstrated compliant
  missing-symbol RED behavior.
- Remove the blanket sentence prohibiting thin public adapters after a
  service-level scope.
- Add one ALIGN sentence requiring destructive deletion, retention,
  anonymization, and recovery semantics to be resolved before implementation.

Do not add a controller-testing decision tree. Let risk, existing conventions,
and the requested contract determine whether thin wiring warrants a separate
test, and report it when left unverified.

## Result

Experiment 023 passes follow-up routing, production-scaffolding discipline,
service-level test-first behavior, minimal change, and test economy. It fails
ALIGN by choosing irreversible hard deletion without an established contract.
It also corrects an over-strict evaluation rule introduced after Experiment
022.
