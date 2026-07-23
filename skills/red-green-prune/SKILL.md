---
name: red-green-prune
description: >
  Route the complete Red Green Prune workflow. Use when the user explicitly
  asks for Red Green Prune or ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE.
  Delegate detailed policy to focused sibling skills so unrelated policy is
  not loaded.
---

# Red Green Prune

For observable behavior changes, follow:

`ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE`

Do not claim TDD unless RED was observed before the production change.

## Route

Load only the sibling skills needed:

- `test-first-cycle` for ALIGN, RED evidence, and rule-by-rule cycles;
- `minimal-change` for the production edit and REFACTOR;
- `test-prune` only when new tests may overlap or the user requests test
  review, parameterization, cleanup, or deletion.

A normal behavior change needs `test-first-cycle` and `minimal-change`.
Non-behavior changes need only `minimal-change` and relevant verification.

Claude Code also exposes the focused skills directly:

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

Stop when the agreed behavior and relevant checks pass. Combine only applicable
evidence from the loaded skills; do not add optional improvements.
