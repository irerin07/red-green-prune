# Experiment 014: Application Decisions API

## Request

Build `POST /application-decisions` in a fresh Spring Boot project. The response
had independent requirements for HTTP 201, a generated UUID, eligibility at an
inclusive score-80 boundary, decision mapping, and passthrough of a manual
review flag.

The request intentionally excluded validation, arithmetic, persistence, and
error handling. This isolated two behaviors that failed in Experiment 013:
separating independently falsifiable rules and avoiding task-new guard
relabeling.

## Environment

- Claude Code with Red Green Prune after PR #20
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC test support already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`
- Model, token count, cost, and duration were not captured

The agent ran the empty harness before the first RED. It initially used the old
Spring Boot package for `@AutoConfigureMockMvc`, correctly rejected the
compilation failure as setup-related, corrected the import, and reran the test.

## Observed behavior

The agent completed five independent cycles:

1. a valid request returns HTTP 201;
2. `eligible` follows the inclusive score-80 boundary;
3. `decision` maps from eligibility;
4. `reviewRequired` mirrors `manualReview`;
5. `id` is a generated UUID.

Eligibility and decision were not batched. The eligibility scope included score
80 and 79 before GREEN, jointly distinguishing `>= 80` from `> 80`,
always-true, and always-false defects. Decision used APPROVED and REJECTED cases
in its own later scope. Manual review used true and false in one passthrough
scope. UUID verification checked both format and per-request uniqueness.

For every cycle, the agent stated the selected rule and scope, ran the test,
and then recorded the actual rule, scope, and failure before editing production.
The old command narration did not return.

No task-new test was added after its behavior had already been implemented. The
agent therefore neither created nor relabeled a passing-after-implementation
test as a guard. This is the preferred prevention path, but it does not directly
test classification when such a test already exists.

The implementation remained one small controller with nested request and
response records. It added no service, validation, dependency, persistence,
configuration, additional endpoint, or speculative extension mechanism. The
full suite passed with eight new tests plus `contextLoads`, for nine total.

The final PRUNE report incorrectly referred to seven new tests even though the
transcript and full-suite count show eight. This was a reporting arithmetic
error, not a test-selection or skill-policy failure.

## Expected behavior

- Form the 79/80 boundary scope before implementing eligibility.
- Implement eligibility without decision mapping.
- Observe a separate RED for APPROVED/REJECTED mapping.
- Treat true and false manual-review values as one meaningful passthrough scope.
- Record actual RED evidence before every production edit.
- Avoid adding task-new passing tests after implementation; if one occurs,
  report missed RED rather than calling it a guard.

The run matched these expectations except that the last classification branch
was avoided rather than directly exercised.

## Result

Experiment 014 is a clean behavioral pass for cycle separation, scope design,
minimal GREEN, RED reporting, and test pruning. The violations from Experiment
013 did not recur in this smaller scenario.

No SKILL.md change follows. This is the first clean stabilization pass after
Experiment 013. One more comparable no-change pass is required before treating
the relevant policy as stable enough for responsibility-based skill
separation. This evidence was recorded in
[PR #22 — Record application decisions experiment](https://github.com/irerin07/red-green-prune/pull/22).
