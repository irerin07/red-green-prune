---
name: red-green-prune
description: >
  Orchestrate the complete Red Green Prune workflow for test-first development
  without test bloat or speculative design. Use when the user explicitly asks
  for Red Green Prune or the complete ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
  workflow. Route each responsibility to the focused sibling skill instead of
  loading unrelated policy.
---

# Red Green Prune

For observable behavior changes, follow:

`ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE`

Do not claim TDD unless RED was observed before the production change.

## Route by responsibility

Load and follow only the sibling skills needed for the task:

- `test-first-cycle` for features, bug fixes, APIs, calculations, validation,
  or any other observable production behavior change. It owns ALIGN, honest
  RED evidence, and one-rule RED-to-GREEN cycles.
- `minimal-change` for the production edit and any refactor. It owns the
  smallest GREEN change and rejects speculative code, infrastructure,
  annotations, and cleanup.
- `test-prune` when the current task adds tests that may overlap, or when the
  user explicitly requests duplicate-test review, parameterization, cleanup,
  or deletion. It owns pruning evidence and historical-test safety.

A normal behavior change needs `test-first-cycle` and `minimal-change`. Do not
load `test-prune` merely because tests exist; load it when overlap or cleanup is
actually in scope.

For direct use in Claude Code, the focused skills are available as:

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

## Non-behavior changes

Documentation, formatting, generated-file synchronization,
behavior-preserving mechanical renames, deletion of proven-dead code, and
metadata synchronization with a deterministic validator do not require RED.
Use `minimal-change` and the smallest relevant verification.

## Report

Stop when the agreed behavior and relevant checks pass. Combine only the
applicable evidence reported by the loaded skills. Do not implement optional
improvements after the requirement is satisfied.
