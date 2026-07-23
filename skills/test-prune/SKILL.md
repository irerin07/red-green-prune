---
name: test-prune
description: >
  Prevent or remove redundant tests without weakening distinct or historical
  coverage. Use when a task adds tests that may overlap, a suite appears
  bloated, tests seem duplicate, parameterization or merging is proposed, or
  the user requests test cleanup or deletion. Require evidence before pruning
  and protect pre-existing high-risk tests.
---

# Test Prune

Prevent test bloat within the current change. Do not delete, disable, or weaken
a pre-existing test during ordinary feature or bug work. Report suspected
historical duplicates separately.

Keep a test when it uniquely protects a behavior, meaningful equivalence
class, boundary, regression, security rule, data-loss risk, contract, or
execution path.

## Prune tests from the current task

Merge or remove tests introduced during this task only when all are true:

- both tests protect the same requirement and meaningful path;
- the same production defect would make both fail;
- the observed RED shows the surviving test detects that defect;
- removal loses no distinct diagnostic value.

Use the observed RED plus a structural comparison of setup, public path,
assertion, and failure reason as evidence. Green-after-deletion alone is not
proof. Keep the test whose failure was observed as RED; if another test should
survive instead, run that test against the pre-GREEN behavior and observe its
failure before deleting the original. If equivalent sensitivity is unclear,
keep both tests.

Parameterization removes test-code duplication, not necessarily behavioral
duplication. Retain cases representing different equivalence classes or
failure modes.

After pruning tests introduced during this task, run the surviving focused
test and the relevant test scope.

## Historical tests

Prune pre-existing tests only when the user explicitly requests test cleanup.
For security, authorization, money, persistence, destructive behavior,
data-loss prevention, or historical regression tests, present evidence and
obtain approval before deletion.

## Worked example

During the current task, the agent adds two tests for this requirement:
username lookup must be case-insensitive.

Before:

```python
def test_finds_username_with_uppercase_query(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("ALICE").username == "alice"


def test_finds_username_with_mixed_case_query(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("AlIcE").username == "alice"
```

Both tests use the same setup, public path, normalization behavior, result,
and failure reason. Keep one:

```python
def test_username_lookup_is_case_insensitive(repo):
    repo.add(User(username="alice"))
    assert repo.find_by_username("ALICE").username == "alice"
```

The RED already observed before GREEN shows that the surviving test detects
the missing normalization. No additional mutation ceremony is needed for two
tests introduced together with identical paths and failure reasons.

Do not prune this test:

```python
def test_username_lookup_does_not_expose_another_tenant(repo):
    ...
```

It calls the same method but protects a different authorization regression.
Likewise, keep an additional Unicode case-folding case if it distinguishes
`casefold()` from ASCII-only normalization.

## Report

Report tests pruned, the evidence supporting each decision, the verification
run afterward, suspected historical duplicates left untouched, and anything
not verified.
