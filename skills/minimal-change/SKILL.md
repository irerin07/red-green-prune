---
name: minimal-change
description: >
  Keep implementation and refactoring no larger than proven necessary. Use
  when making a production change or reviewing over-engineering, speculative
  abstractions, dependencies, configuration, fallbacks, annotations, or
  cleanup without observable value. Apply after a valid RED when paired with
  test-first-cycle.
---

# Minimal Change

Apply this litmus throughout: if removing an addition changes no agreed
behavior, contract, required check, or distinct regression signal, omit it.

## Minimum GREEN

When paired with `test-first-cycle`, edit production only after valid RED. Write
the smallest production change satisfying the failing scope and the agreed
behavior selected for the current cycle. Preserve existing contracts.

Scaffolding is not RED evidence for later rules. Defer unselected behavior
unless doing so requires extra production structure. If the current GREEN
unavoidably satisfies a later rule, report missed RED for that later scope; do
not add placeholders, manufacture failure, or claim test-first.

Reuse repository code, standard-library functionality, native platform
behavior, and installed dependencies before adding infrastructure. Do not add
speculative abstractions, extension points, configuration, dependencies,
fallbacks, annotations, or unrelated cleanup.

An annotation is justified only when runtime, framework behavior, static
analysis, serialization, validation, persistence, dependency injection, an
external contract, or an explicit repository rule requires it. Neighboring
code is evidence to investigate, not a reason to copy boilerplate.

Run the focused check and confirm the change works.

## REFACTOR

Refactor only while tests are green and only to remove duplication of the same
policy, correct a misleading name, simplify control flow, or reuse an existing
pattern with functional value.

Similar shape is not sufficient. Extract behavior only when it has the same
meaning and reason to change. Keep the refactor within the current behavior
unless broader cleanup was requested. Run the focused check afterward.

## Non-behavior changes

Documentation, formatting, generated-file synchronization,
behavior-preserving mechanical renames, deletion of proven-dead code, and
metadata synchronization with a deterministic validator do not require RED.
Make the smallest change and run the smallest relevant verification; do not
invent a behavioral test merely to satisfy a workflow.

## Stop

Stop when the agreed behavior and relevant checks pass. Report material
defaults, verification performed, and anything not verified. Do not implement
optional improvements after the requirement is satisfied.
