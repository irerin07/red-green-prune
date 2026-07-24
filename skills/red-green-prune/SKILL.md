---
name: red-green-prune
description: >
  Develop observable behavior test-first without growing redundant tests. Use
  when the user asks for Red Green Prune, TDD, RED -> GREEN -> REFACTOR, or
  focused test cleanup. Load only the sibling skills needed for the task.
---

# Red Green Prune

For behavior changes, follow:

`ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE`

The core rules are:

- observe a relevant test failure before editing production;
- implement only behavior covered by that failing test scope and the request;
- keep only tests with distinct protective value.

## Route

Load:

- `test-first-cycle` and `minimal-change` for behavior changes;
- `test-prune` when new tests may overlap or the user requests test review,
  parameterization, cleanup, or deletion;
- only `minimal-change` for non-behavior changes.

Focused skills are also available directly:

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

Stop when the requested behavior and relevant checks pass. Do not add optional
improvements or repeat evidence from sibling skills.
