# Experiment 012: Membership Evaluation API

## Request

Build `POST /membership-evaluations` in a fresh Spring Boot project. The API
must return a UUID, a trimmed display name, an adult flag with an age-18
boundary, and a newsletter status. It must also return exact validation errors
for a blank name and negative age.

The request deliberately specified outcomes without prescribing test cases for
whitespace, both sides of the age boundary, both newsletter values, UUID
validation, or exact-response matching. This tested whether the skill could
derive distinguishing scopes without supplied examples.

## Environment

- Claude Code with Red Green Prune after PR #19
- Fresh Spring Boot 4.1.0 / Java 21 project
- Web MVC and validation starters already installed
- Only the generated `contextLoads` test existed
- Test command: `./gradlew test`
- Model, token count, and cost were not captured

The agent ran the empty harness before the first RED. It initially used the
Spring Boot 3 `@WebMvcTest` import, correctly classified the resulting failure
as setup-related, found the Spring Boot 4 package, and retried before accepting
RED.

## Observed behavior

The agent completed seven separate cycles:

1. valid request returns 201;
2. surrounding name whitespace is removed;
3. ages 18 and 17 jointly distinguish the adult boundary;
4. newsletter `true` and `false` jointly distinguish the status mapping;
5. the id matches a UUID pattern;
6. blank name returns the exact required error body;
7. negative age returns the exact required error body.

Each production change followed an observed behavioral failure. The agent did
not batch independently failing rules, mutate production to manufacture RED,
or use placeholders that preimplemented later rules. The age and newsletter
pairs were correctly treated as multi-case scopes for one rule. The final full
suite passed with eight tests, including `contextLoads`. No speculative
infrastructure, redundant tests, or prune candidates were added.

The five-field RED checkpoint was not followed consistently:

- cycle 1 stated rule, nearby defect, scope, command, and an expected outcome;
- cycle 2 omitted the command;
- cycles 3 through 5 omitted an explicit nearby-defect label and command;
- cycles 6 and 7 omitted the command.

Despite those omissions, the test cases themselves distinguished the relevant
nearby defects: 18 versus 17, true versus false, a UUID regex, and strict JSON.
The executed command was already visible in the tool record. Repeating it in
the checkpoint added reporting cost without changing behavior.

The checkpoint also labeled a predicted result as `observed (expected)` before
the test ran. The actual failure was confirmed afterward, so the workflow was
honest, but the template blurred planning and observation.

The cheap-deferral branch of the GREEN rule was exercised successfully. The
branch for unavoidable early satisfaction of a later rule was not exercised.

## Expected behavior

For each cycle, the agent should:

1. select one rule and its smallest distinguishing scope;
2. run that scope;
3. record the actual failure before editing production.

The scope must still distinguish correct behavior from a plausible nearby
defect. The nearby defect need not be repeated as a mandatory report field
when the scope makes it concrete, and the test command need not be narrated
when the execution record already supplies it.

## Result

The core cycle, scope, guard, and minimal-GREEN rules passed this experiment.
The failure was in the reporting interface: the five-field checkpoint was
longer than the agent needed and placed `observed` before observation.

The resulting change replaces it with a three-field post-run checkpoint:
`rule`, `scope`, and actual `failure`. The command remains execution evidence,
and nearby-defect discrimination remains a semantic requirement rather than a
mandatory narrative field. No structural split or unrelated rule change is
included. This produced
[PR #20 — Simplify the RED evidence checkpoint](https://github.com/irerin07/red-green-prune/pull/20).