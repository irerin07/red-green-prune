# Red Green Prune

**Test-first development without redundant tests.**

[한국어](README.ko.md)

Coding agents often implement first, add tests afterward, and still call it
TDD. They may also add several tests that protect the same behavior.

Red Green Prune keeps the workflow focused:

```text
ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
```

> Observe a relevant test failure before editing production, implement only
> behavior covered by that failing scope and the request, and keep only tests
> with distinct protective value.

## Before / after

You ask for case-insensitive username lookup.

Without Red Green Prune, an agent may implement the feature first and add
several equivalent tests afterward.

With Red Green Prune:

```text
ALIGN     Read the lookup path and nearest tests.
RED       Extend the nearest test and observe the missing behavior fail.
GREEN     Make the comparison case-insensitive and rerun the test.
REFACTOR  Change nothing unless the current work needs it.
PRUNE     Keep distinct protection; merge or omit equivalent new tests.
```

A test scope may contain multiple assertions or cases when they jointly
describe one observable contract. Red Green Prune does not require a separate
RED for every matcher.

## Modular skills

The workflow is split so agents load only the responsibility needed:

| Skill | Responsibility | Use it for |
| --- | --- | --- |
| `red-green-prune` | Thin entry point and routing | The complete workflow |
| `test-first-cycle` | Test scope fails before production | Features, fixes, APIs, calculations, validation |
| `minimal-change` | Keep production within the failing scope and request | GREEN, REFACTOR, non-behavior changes |
| `test-prune` | Duplicate-test evidence and deletion safety | Overlapping new tests or explicit test cleanup |

A normal behavior change uses `test-first-cycle` and `minimal-change`.
`test-prune` is loaded only when tests may overlap or cleanup is requested.

## How it works

### ALIGN and RED

Read the affected code and nearest tests. Select or write the smallest test
scope expressing the requested behavior, run it before editing production, and
confirm it fails because that behavior is missing.

Setup, syntax, flaky, unrelated, and pre-existing failures are not RED. If a
valid RED cannot be observed, the agent states the limitation instead of
claiming TDD.

### GREEN and REFACTOR

Implement only behavior asserted by the failing scope and requested by the
user. All assertions in one observable contract may be satisfied in the same
GREEN. Run the focused and relevant existing tests.

Refactor only while tests are green and only when the current change needs it.
Do not add unrequested dependencies, configuration, layers, persistence,
endpoints, extension points, or unrelated cleanup.

### PRUNE

Prevent redundant tests introduced during the current task. Keep tests that
protect distinct behavior, boundaries, equivalence classes, regressions,
security rules, contracts, or failure modes.

## Prune safely

Red Green Prune prevents new test bloat. It does not autonomously clean
historical suites.

During ordinary feature or bug work, the agent must not delete, disable, or
weaken a pre-existing test. Suspected historical duplicates are reported
separately. Removing them requires an explicit cleanup request; high-risk tests
also require evidence and approval.

## Install

### Codex

```bash
codex plugin marketplace add irerin07/red-green-prune
codex plugin add red-green-prune@red-green-prune
```

Start a new Codex session, then invoke the workflow:

```text
$red-green-prune implement case-insensitive username lookup
```

Codex may also activate a focused skill automatically from the task.

### Claude Code

Send these as separate prompts:

```text
/plugin marketplace add irerin07/red-green-prune
```

```text
/plugin install red-green-prune@red-green-prune
```

Start a new session, then invoke the workflow:

```text
/red-green-prune:red-green-prune implement case-insensitive username lookup
```

Or invoke one responsibility directly:

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

After updating, start a new session so Claude Code discovers the latest skills.

## What it does not do

- It does not require a separate RED for every matcher or assertion.
- It does not require RED for documentation, formatting, generated files, or
  behavior-preserving mechanical changes.
- It does not create a test framework in an established repository unless
  requested; a new project receives the minimum conventional harness.
- It does not delete historical coverage during ordinary implementation.
- It does not implement behavior outside the failing test scope and request.
- It does not claim TDD when RED was not observed before production.

The skill definitions are:

- [`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md)
- [`skills/test-first-cycle/SKILL.md`](skills/test-first-cycle/SKILL.md)
- [`skills/minimal-change/SKILL.md`](skills/minimal-change/SKILL.md)
- [`skills/test-prune/SKILL.md`](skills/test-prune/SKILL.md)

## Status

The rules are developed from recorded field experiments. See
[`experiments/`](experiments/) for observations and resulting changes. These
are evidence notes, not statistically significant benchmark claims.

## License

MIT
