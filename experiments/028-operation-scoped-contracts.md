# Experiment 028: Operation-scoped explicit contracts

## Context

- Scenario: add `CommentService.get(id)` returning comment id and content
- Repository state: comments support active, absent, and soft-deleted states;
  update hides soft-deleted comments
- Plugin version: not captured in the transcript
- Attribution signal: the run used PR 37-specific language about a task-spanning
  choice, different assertions, and existing tests or documentation

## Observed behavior

The agent loaded `test-first-cycle` and `minimal-change` before editing. During
ALIGN it found that read visibility for soft-deleted comments was unresolved,
asked the user, and waited. The user chose to treat deleted comments as not
found.

The RED scope then covered active, absent, and deleted comments. It observed a
missing-symbol failure for the requested `CommentResponse` and
`CommentService.get` boundary before production changes. GREEN returned the
requested id and content, filtered deleted comments, and left the controller
untouched. The full suite passed.

Lifecycle-state discovery, direct state observation, scoped implementation, and
test-first ordering therefore passed.

## Failure

The request did not specify the absent-comment result. Plausible contracts
included an exception, an empty optional, or a null result, and they required
different assertions.

The agent did not ask. It asserted
`IllegalArgumentException("Comment not found")`, implemented that result, and
later reported that update and delete tests established the resource's existing
not-found contract.

Those tests covered the same resource but different operations. They did not
directly settle read behavior. The phrase "for this resource" in the explicit
contract definition allowed the agent to promote another operation's tests into
the read contract, bypassing the per-RED decision gate. Reporting the choice as
an assumption after implementation violated the gate's timing rule as well.

## Resulting change

Scope explicit contracts to both the resource and operation:

> An explicit contract is an existing test or user-facing documentation that
> directly settles the choice for this resource and operation, or an explicit
> repository-wide policy.

No new phase, question category, test matrix, or mutation requirement is added.
The change closes only the observed cross-operation inference path.
