# Experiment 013: Loan Eligibility API

## Request

Build `POST /loan-eligibility-evaluations` in a fresh Spring Boot project. The
API had independent requirements for HTTP status, UUID generation, name
trimming, debt-to-income calculation, HALF_UP rounding, two eligibility
conditions, decision mapping, and four exact validation errors.

The request supplied no test examples for the rounding mode, eligibility
boundaries, UUID format, or exact JSON comparison. This tested whether the
agent would derive distinguishing scopes and use the simplified RED checkpoint
introduced by PR #20.

## Environment

- Claude Code with Red Green Prune after PR #20
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation starters already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`
- Reported duration: 10 minutes 38 seconds
- Model, token count, and cost were not captured

The agent ran the empty harness before the first RED. It initially used the old
Spring Boot package for `@AutoConfigureMockMvc`, correctly rejected the
compilation failure as setup-related, corrected the import, and reran the test.

## Observed behavior

### Improvements

The agent observed real failures before production edits and did not mutate
production to manufacture RED. It derived useful discriminating inputs without
being given examples:

- `0.125` distinguishes HALF_UP (`0.13`) from HALF_EVEN or HALF_DOWN (`0.12`);
- credit score `699` isolates the score condition;
- a 50% debt-to-income rate isolates the debt condition;
- score `700` with a 40% rate pins the inclusive boundaries;
- a UUID regular expression checks format rather than mere presence;
- strict JSON checks enforce the exact error bodies.

Validation rules were handled in separate cycles. The implementation stayed at
the HTTP boundary, added no service layer, persistence, dependency,
configuration, or speculative extension mechanism, and finished with 12 new
tests plus the existing `contextLoads` test passing.

The simplified checkpoint improved sequencing. Early cycles stated the rule and
scope, ran the test, and then recorded the actual failure before production
edits. Later validation cycles abbreviated the post-run record to statements
such as `RED observed — returned 201`; their surrounding text supplied the rule
and scope, but the three fields were not recorded consistently.

### Violations

Two independently falsifiable rules were batched in cycle 2. The selected scope
combined debt-to-income calculation with two-decimal HALF_UP rounding, and the
GREEN implementation chose HALF_UP before a rounding-sensitive RED. A
calculation cycle using an exactly divisible input could have deferred the
rounding rule without extra production structure.

Cycle 4a batched the eligibility result and decision mapping. `eligible` can be
correct while `decision` is wrong, so they required separate cycles. The agent
implemented both after the same RED.

Two task-new tests passed on first run and were reported as guards:

- the HALF_UP-sensitive `0.125 -> 0.13` case;
- the inclusive `700 / 40.00` eligibility boundary.

The report also called these missed RED, which was honest, but the guard label
conflicts with the existing rule that behavior introduced by the current task
is not a guard. Both missed REDs were avoidable through earlier scope selection.
They were not examples of unavoidable later-rule satisfaction.

The final report alternated between 12 and 13 tests. The likely meaning was 12
new tests and 13 total including `contextLoads`, but the count should have been
stated consistently.

## Expected behavior

- Select calculation with an exactly divisible input, observe RED, and implement
  only the calculation.
- Select a separate exact-half scope before choosing HALF_UP.
- Drive `eligible` before implementing `decision`.
- Drive `decision` through its own RED after eligibility is green.
- Use inclusive boundary cases in the selected eligibility scopes rather than
  adding them afterward.
- Report a task-new test that passes after implementation as missed RED, not as
  a guard.
- Record actual `rule`, `scope`, and `failure` after every run and before the
  production edit.

## Result

Experiment 013 is a partial pass. The smaller checkpoint improved the order and
honesty of RED reporting, and the agent derived strong tests without supplied
examples. However, batching and task-new guard relabeling recurred despite
already explicit rules.

No SKILL.md change follows from this experiment. Adding more wording would
duplicate existing policy and increase the attention cost. A smaller follow-up
experiment should isolate cycle separation and guard classification. If the
same violations recur, the evidence supports splitting responsibilities rather
than adding another rule.