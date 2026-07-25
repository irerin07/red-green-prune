# Experiment 025: Comment State Change and Soft Deletion

## Request

In the real Spring Boot project used for Experiments 023 and 024, use plugin
version `v0ab06ecb0066` and perform two follow-up tasks:

1. create a comment API for a post;
2. create a comment deletion API.

The project already contained `Users` and `Post` resources with hard-deletion
patterns. The run tested ordinary state-change evidence and the resource-scoped
destructive ALIGN rule introduced by PR #34.

## Observed behavior

### Comment creation stayed test-first but did not prove the state change

The agent loaded `test-first-cycle` and `minimal-change`, wrote a focused
`CommentServiceTest` before production, and observed a valid missing-symbol RED
for the absent comment production boundary. It then added the entity, request
record, repository, service, and thin controller. Focused and full tests passed.

The success test was named as though it proved that content, post, and author
were persisted, but its only assertion was the generated identifier:

```java
Long id = commentService.create(10L, request);
assertThat(id).isEqualTo(100L);
```

The mocked repository returned that identifier for any `Comment`. The test
would therefore still pass if production ignored the requested content or
saved a comment without the selected post or author. The missing-post and
missing-author tests proved the lookups and errors, but did not prove that the
resolved relationships were applied to the saved entity.

The sequence was test-first, but the failing scope did not express the core
state change that GREEN implemented. A generated ID or mocked return value was
used as a proxy for persisted state.

The existing success test could cover the contract without adding another test:
capture the `Comment` passed to `CommentRepository.save` and assert its content,
post, and author before retaining the ID assertion.

### Resource-scoped destructive ALIGN passed

For comment deletion, the agent explicitly classified deletion as destructive
and stated that `PostService.delete` was a pattern for another resource, not a
contract for `Comment`. Before writing tests or production code, it asked the
user to choose deletion semantics. The user selected soft deletion.

The agent then wrote focused service cases, observed a valid missing-boundary
RED, and implemented `deletedAt`, a domain delete operation, a transactional
service method, and a thin `DELETE /comments/{id}` adapter. It verified that the
comment was marked, that physical repository deletion was not called, and that
a missing comment produced the agreed exception. The focused and full suites
passed.

PR #34's destructive ALIGN clarification worked as intended and needs no
further change.

### Additional minimal-change observation

The soft-delete test asserted both `comment.isDeleted()` and
`comment.getDeletedAt() != null`, while `isDeleted()` only returned
`deletedAt != null`. This duplicated one signal and caused a test-only
production convenience method to be added. The existing minimum-change rule
already prohibits that expansion; record it without adding another rule unless
it recurs.

## Resulting change

Add one narrow state-change evidence rule to `test-first-cycle`:

- make a state-changing scope observe the requested resulting state or the
  value sent to the persistence boundary;
- do not treat status, generated identifiers, or mocked return values alone as
  proof that requested fields or relationships were applied.

This strengthens an existing success test rather than requiring more tests or a
new sensitivity matrix.

## Result

Experiment 025 passes routing, production-scaffold discipline, missing-symbol
RED, destructive ALIGN, focused deletion TDD, and honest reporting. It fails
state-change evidence for comment creation because the test did not observe the
content or relationships that production persisted. It also records one minor
redundant-assertion/minimal-change issue without changing policy for it.
