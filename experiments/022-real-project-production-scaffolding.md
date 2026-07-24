# Experiment 022: Real-Project Production Scaffolding

## Requests

Two short requests were made in an existing Spring Boot project:

1. create a sign-up API;
2. after explicitly invoking `/red-green-prune:red-green-prune`, create a
   member-information update API as the follow-up task.

The project already contained a `Users` entity, `UserRepository`,
`UserService`, `UserController`, and focused service characterization tests.

This experiment tested ordinary short requests in a real project rather than a
fully specified synthetic API prompt. It also tested routing when the entry
skill was invoked before the behavior task arrived in a follow-up.

## Observed behavior

### Production scaffolding preceded RED twice

For sign-up, the agent created `UserSignUpRequest` and an unimplemented
`UserService.signUp` method before writing and running the service test. For
update, it repeated the sequence with `UserUpdateRequest` and an unimplemented
`UserService.update` method.

Both stubs threw `UnsupportedOperationException("not implemented")`. The agent
then described the resulting test failures as RED.

The stated reason was to make the tests compile. This contradicts the existing
missing-boundary rule: an expected missing-symbol compile or load failure is a
valid RED. DTOs, methods, and deliberate stubs are production changes, so the
observed failures were created after production had already been edited.

### Untested HTTP boundaries were added during GREEN

The sign-up failing scope covered only `UserService.signUp`. After that scope
passed, the agent added `POST /users`, HTTP 201, request binding, and an id
response without an HTTP test.

The update failing scope covered only service-level entity mutation. After that
scope passed, the agent added `PUT /users/{id}`, request binding, HTTP 200, and
an empty response without an HTTP test.

In both cases the agent called the controller a “thin passthrough” and treated
the context-load test as endpoint-wiring verification. A context-load test does
not protect route, method, status, body, binding, or delegation. A service
scope therefore authorized production behavior it did not assert.

### Follow-up routing was skipped

The entry skill was explicitly invoked before the update task was supplied.
It asked which behavior to implement, received the task in a follow-up, and
then edited files without visibly loading `test-first-cycle` and
`minimal-change` as required by the entry route.

The follow-up sequence matters because a skill may be invoked before the user
provides the concrete task. Routing must occur after the task becomes known,
not only during the initial invocation.

### Minimality and test economy remained good

The production designs were otherwise small. The agent added no validation,
duplicate-email policy, persistence abstraction, additional service layer, or
speculative extension mechanism. Each task added one focused service test and
no redundant matrix.

The update test correctly distinguished both field mutations. The sign-up test
returned the generated id but did not inspect the entity passed to `save`, so
it would not detect omitted email or phone-number mapping.

## Original-goal failures

The repeated behavior violated two core rules already present in the skill:

- production was edited before a failing scope was observed;
- GREEN added public behavior not asserted by the failing scope.

The explicit entry invocation followed by skipped sibling loading also exposed
a routing gap for tasks delivered in a follow-up.

This is not a matcher-level proof preference. DTOs and stubs were concrete
production edits before RED, and the HTTP APIs had no preceding test scope at
all.

## Resulting change

Apply three narrow clarifications:

1. Route and load sibling skills after the task is known, including when the
   task arrives in a follow-up, and before any file edit.
2. State that production DTOs, methods, stubs, routes, and adapters must not be
   added merely to make RED compile.
3. State that a service-level failing scope does not authorize an unasserted
   public endpoint or adapter during GREEN.

Do not add another ALIGN rule. The current material-public-behavior criterion
already covers choices such as PUT versus PATCH and response shape; this run
does not show that more ALIGN prose would improve compliance.

## Result

Experiment 022 fails the original test-first and failing-scope boundaries while
passing minimal-change and test-economy goals. The repeated real-project
behavior supports the narrow routing, scaffolding, and public-boundary changes
above.
