# Experiment 024: Post Deletion Contract Scope

## Request

In the same real Spring Boot project used for Experiment 023, invoke
`/red-green-prune:red-green-prune`, then request a post-creation API and follow
with a post-deletion API request.

The project already contained a member-withdrawal implementation using hard
deletion. The newly added `Post` resource had no deletion contract.

The deletion run used plugin version `v2b3ad7eadeda`, the merge commit for
PR #33. It tested whether the new destructive-ALIGN rule would distinguish an
existing contract from a merely similar deletion pattern.

## Observed behavior

### Test-first sequencing passed

For post deletion, the agent added two focused service cases before editing
production:

- an existing post is deleted;
- a missing post produces `IllegalArgumentException("Post not found")`.

The focused run failed because `PostService.delete` did not exist. The agent
accepted that expected missing-symbol failure as RED, then added the service
method and thin `DELETE /posts/{id}` adapter. The focused and full test suites
passed.

The two cases jointly described the requested deletion contract and did not
expand into a test matrix. No production scaffold preceded RED.

### Minimal change and test economy passed

The implementation reused the existing repository and controller patterns. It
added no persistence mechanism, dependency, configuration, service layer,
recovery mechanism, or controller test. The unverified thin HTTP wiring was
reported explicitly.

### Destructive ALIGN still failed

The agent treated `UserService.withdraw` as a repository-wide deletion
contract and inferred hard deletion for `Post` without asking. That inference
was not justified:

- the existing contract applied to `Users`, not `Post`;
- absence of soft-delete fields on `Post` described the current schema, not the
  required retention or recovery behavior;
- different resources in one repository may legitimately have different
  lifecycle, audit, legal, and recovery requirements.

The phrase “unless an existing contract settles them” allowed a contract for
one resource to be generalized to another. The destructive choice remained
materially ambiguous and should have been resolved during ALIGN.

## Resulting change

Narrow the exception to destructive ALIGN:

- an existing contract must apply to the same resource; or
- an explicit repository-wide policy must settle the choice.

State directly that a deletion pattern for another resource is not sufficient.
Do not add a deletion decision tree or require controller tests.

## Result

Experiment 024 passes follow-up routing, missing-symbol RED, focused-scope TDD,
minimal change, test economy, and honest reporting. It fails destructive ALIGN
because a resource-specific hard-delete pattern was incorrectly promoted to a
repository-wide contract.
