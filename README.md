# Red Green Prune

**TDD without test bloat or speculative design.**

Red Green Prune guides coding agents through a disciplined cycle:

```text
ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
```

It requires a meaningful failing test before implementation, keeps production
changes minimal, and prevents redundant tests introduced during the current
task. Historical tests are never deleted during ordinary feature or bug work.

## Install

Copy `skills/red-green-prune` into your agent's skills directory. For Codex:

```text
~/.codex/skills/red-green-prune/
```

Then invoke it explicitly:

```text
$red-green-prune implement this behavior
```

Codex may also activate it automatically for TDD, test-first feature work, bug
fixes, excessive tests, or over-engineered implementations.

## What it enforces

- Clarify only decisions that materially affect behavior or contracts.
- Observe RED before changing production code.
- Write the smallest GREEN implementation.
- Refactor only proven problems while tests remain green.
- Prevent new test duplication without deleting historical coverage.
- Report the commands and results that prove RED and GREEN.

The complete workflow is defined in
[`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md).

## License

MIT
