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

## Modular skills

The workflow is split by responsibility so agents load only the policy needed
for the current task:

| Skill | Responsibility | Use it for |
| --- | --- | --- |
| `red-green-prune` | Thin entry point and routing | The complete workflow |
| `test-first-cycle` | ALIGN, honest RED evidence, one-rule cycles | Features, fixes, APIs, calculations, validation |
| `minimal-change` | Minimum GREEN and restrained REFACTOR | Implementation, over-engineering, unnecessary annotations |
| `test-prune` | Duplicate-test evidence and deletion safety | Overlapping new tests or explicit test cleanup |

A normal behavior change uses `test-first-cycle` and `minimal-change`.
`test-prune` is loaded only when tests may overlap or cleanup is requested.
Moving the rules into independent skills reduces routine context without
hiding all of them behind references that would still be loaded together.

## How it works

### ALIGN and RED

Read the affected code, callers, and nearest tests before editing. Resolve
repository facts, select one independently falsifiable rule, and observe a
real failure before changing production code. Passing tests, guards, and
missed RED are reported honestly rather than relabeled as TDD evidence.

### GREEN and REFACTOR

Write the smallest production change satisfying the selected rule and existing
contracts. Avoid speculative abstractions, configuration, dependencies,
fallbacks, annotations, and unrelated cleanup. Refactor only while tests are
green and the current change proves a design problem exists.

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

Start a new Codex session, then invoke the complete workflow explicitly:

```text
$red-green-prune implement case-insensitive username lookup
```

Codex may also activate a focused skill automatically from the task.

### Claude Code

Send these as two separate prompts:

```text
/plugin marketplace add irerin07/red-green-prune
```

```text
/plugin install red-green-prune@red-green-prune
```

Start a new session, then invoke the complete workflow:

```text
/red-green-prune:red-green-prune implement case-insensitive username lookup
```

Or invoke one responsibility directly:

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

After updating the plugin, start a new session so Claude Code discovers the
latest skill files.

## What it does not do

- It does not require RED for documentation, formatting, generated files,
  mechanical renames, or deletion of proven-dead code.
- It does not create a test framework in an established repository unless
  requested; a new project receives the smallest conventional test harness.
- It does not delete historical coverage during ordinary implementation work.
- It does not implement optional improvements after the requirement is met.
- It does not claim TDD when the RED step was not observed.

The skill definitions are:

- [`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md)
- [`skills/test-first-cycle/SKILL.md`](skills/test-first-cycle/SKILL.md)
- [`skills/minimal-change/SKILL.md`](skills/minimal-change/SKILL.md)
- [`skills/test-prune/SKILL.md`](skills/test-prune/SKILL.md)

## Status

The rules are developed from recorded field experiments. See
[`experiments/`](experiments/) for observations and the changes they produced.
These are evidence notes, not statistically significant benchmark claims.

LOC, tokens, cost, time, and safety benchmarks are still planned. No
performance claims are made yet.

## License

MIT
