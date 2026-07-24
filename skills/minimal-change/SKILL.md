---
name: minimal-change
description: >
  Keep production edits within the user's request and the current failing test
  scope. Use after a valid RED, or for non-behavior changes, to prevent
  unrequested code, dependencies, configuration, layers, persistence,
  endpoints, and cleanup.
---

# Minimal Change

## GREEN

When paired with `test-first-cycle`, implement only behavior asserted by the
failing test scope and requested by the user. All assertions in that scope may
be satisfied together.

Reuse existing project mechanisms. Preserve existing contracts. Do not add
unrequested dependencies, configuration, layers, persistence, endpoints,
extension points, fallbacks, or unrelated cleanup.

Run the focused test and relevant existing tests. Stop when they pass.

## REFACTOR

Refactor only while tests are green and only when needed to remove duplication
of the same policy, correct a misleading name, or simplify current control
flow. Similar shape alone does not justify extraction. Keep the refactor within
the requested change and rerun relevant tests.

## NON-BEHAVIOR CHANGES

Documentation, formatting, generated-file synchronization, and
behavior-preserving mechanical changes do not require RED. Make the smallest
change and run the smallest relevant verification.

Report verification, material assumptions, and anything not verified. Do not
add optional improvements.
