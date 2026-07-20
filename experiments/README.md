# Field Experiments

These records connect changes to Red Green Prune with behavior observed during
real agent runs. They are evidence notes, not claims of statistically
significant benchmarks.

| Experiment | Scenario | Main observation | Resulting change |
| --- | --- | --- | --- |
| [001](001-user-api.md) | Spring Boot user API | Setup failure was correctly rejected as RED; cycle granularity remained ambiguous | Baseline evidence; later addressed by [PR #8](https://github.com/irerin07/red-green-prune/pull/8) |
| [002](002-quote-api.md) | Spring Boot quote API | Independent validation rules were batched into one RED/GREEN cycle | [PR #8](https://github.com/irerin07/red-green-prune/pull/8) |
| [003](003-tax-calculation-api.md) | Spring Boot tax API | Batching repeated; a passing boundary test was added during RED | [PR #8](https://github.com/irerin07/red-green-prune/pull/8) |
| [004](004-vip20-guided.md) | VIP20 quote extension | Explicit workflow instructions produced compliant separate cycles | Guided baseline; no skill change |
| [005](005-vip25-skill-driven.md) | VIP25 quote extension | Shared implementation and “throwaway” arguments overrode per-cycle scoping | [PR #11](https://github.com/irerin07/red-green-prune/pull/11) |

## Recording policy

Record the request, environment, observed evidence, expected behavior, and any
resulting rule. Keep raw transcripts outside the repository unless they are
needed to verify a disputed observation. Do not change the skill from a
hypothetical failure alone; reproduce or observe the behavior first.
