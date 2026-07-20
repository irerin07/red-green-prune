# Red Green Prune

**TDD without test bloat or speculative design.**

[한국어](README.ko.md)

Coding agents often implement first, add tests afterward, and still call it
TDD. They over-specify edge cases, repeat equivalent tests, introduce
abstractions before they are needed, and copy annotations because neighboring
code has them.

Red Green Prune gives the agent a stricter, smaller loop:

```text
ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
```

Understand the behavior. Observe a real failure. Make the smallest change that
passes. Improve only what the change proves necessary. Prevent new test bloat.

## Before / after

You ask for case-insensitive username lookup.

Without Red Green Prune, an agent may implement the feature immediately, add
several tests afterward, introduce a `UsernameNormalizer` abstraction, and
copy annotations from adjacent classes for consistency.

With Red Green Prune:

```text
ALIGN     Read the lookup path and existing tests; resolve semantics from the repo.
RED       Extend the nearest test and observe the expected failure.
GREEN     Make the existing comparison case-insensitive.
REFACTOR  Nothing proven necessary, so change nothing.
PRUNE     Keep the test that demonstrated RED; add no equivalent cases.
```

The result is actual test-first development with less production code and
fewer tests, not a larger process wrapped around the same implementation.

## How it works

### ALIGN

Read the affected code, callers, and nearest tests before editing. Ask only
when different answers would materially change public behavior, data,
security, compatibility, persistence, or meaningful UX. Resolve facts that are
already in the repository without asking the user.

### RED

Prefer the smallest test change: use an existing relevant failure, extend the
nearest test, or add one focused test. Run it before production changes and
confirm it fails because the requested behavior is missing.

### GREEN

Write the smallest production change satisfying the failing test, agreed
behavior, and existing contracts. Avoid speculative abstractions,
configuration, dependencies, fallbacks, annotations, and unrelated cleanup.

### REFACTOR

Refactor only while tests are green and only when the current change exposes a
real design problem. Similar-looking code is not automatically shared code.

### PRUNE

Prevent redundant tests introduced during the current task. Keep distinct
behaviors, boundaries, equivalence classes, regressions, security rules, and
failure modes.

## Prune, safely

Red Green Prune prevents new test bloat. It does not autonomously clean
historical test suites.

During ordinary feature or bug work, the agent must not delete, disable, or
weaken a pre-existing test. Suspected historical duplicates are reported
separately. Removing them requires an explicit cleanup request; high-risk tests
also require evidence and approval.

This restriction is deliberate. An extra historical test costs maintenance
time. A wrongly deleted regression test can cost a production incident.

## Install

### Codex

```bash
codex plugin marketplace add irerin07/red-green-prune
codex plugin add red-green-prune@red-green-prune
```

Start a new Codex session, then invoke the skill explicitly:

```text
$red-green-prune implement case-insensitive username lookup
```

Codex may also activate it automatically for TDD, test-first feature work, bug
fixes, excessive tests, or over-engineered implementations.

### Claude Code

Send these as two separate prompts:

```text
/plugin marketplace add irerin07/red-green-prune
```

```text
/plugin install red-green-prune@red-green-prune
```

Start a new session, then invoke the skill:

```text
/red-green-prune implement case-insensitive username lookup
```

## What it does not do

- It does not require RED for documentation, formatting, generated files,
  mechanical renames, or deletion of proven-dead code.
- It does not create a test framework when the repository has none.
- It does not delete historical coverage during ordinary implementation work.
- It does not implement optional improvements after the requirement is met.
- It does not claim TDD when the RED step was not observed.

The complete workflow is defined in
[`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md).

## Status

This is the initial release. The intended outcomes are better RED-before-GREEN
compliance, less speculative production code, and fewer redundant tests
introduced per task without reducing correctness or safety.

Benchmarks for LOC, tokens, cost, time, and safety are planned. No performance
claims are made yet.

## License

MIT
