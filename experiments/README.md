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
| [006](006-shipping-quote-api.md) | Shipping quote API | “Happy path” batched independent rules; test-after coverage was relabeled as guards | [PR #13](https://github.com/irerin07/red-green-prune/pull/13) |
| [007](007-hotel-quote-api.md) | Hotel quote API | Guard and validation behavior improved, but explicit happy-path batching persisted | Evidence for restructuring RED, not another rule |
| [008](008-payroll-calculation-api.md) | Payroll calculation API | Calculation and validation cycles separated; contract-falsifying assertions remained incomplete | Evidence only; no skill change |
| [009](009-commission-calculation-api.md) | Commission calculation API | Discriminating examples improved tests; a task-new boundary case was relabeled as a guard | [PR #16](https://github.com/irerin07/red-green-prune/pull/16) |
| [010](010-rebate-calculation-api.md) | Rebate calculation API | Scope composition and self-derived defects improved; late production mutation was reported as RED | [PR #17](https://github.com/irerin07/red-green-prune/pull/17) |
| [011](011-royalty-calculation-api.md) | Royalty calculation API | Mutation honesty improved, but the pre-edit gate was skipped and placeholders implemented later rules early | [PR #18](https://github.com/irerin07/red-green-prune/pull/18), corrected by [PR #19](https://github.com/irerin07/red-green-prune/pull/19) |
| [012](012-membership-evaluation-api.md) | Membership evaluation API | Seven focused cycles passed; the five-field checkpoint was repeatedly abbreviated without harming behavior | [PR #20](https://github.com/irerin07/red-green-prune/pull/20) |
| [013](013-loan-eligibility-api.md) | Loan eligibility API | RED sequencing improved, but independent rules were batched and task-new tests were relabeled as guards | Evidence only; no skill change ([PR #21](https://github.com/irerin07/red-green-prune/pull/21)) |
| [014](014-application-decisions-api.md) | Application decisions API | Cycle separation, boundary scoping, and RED reporting passed in a focused scenario | Evidence only; no skill change ([PR #22](https://github.com/irerin07/red-green-prune/pull/22)) |
| [015](015-program-eligibility-api.md) | Program eligibility API | Slightly higher complexity revived decision batching and deliberate placeholder scaffolding | Responsibility split in [PR #24](https://github.com/irerin07/red-green-prune/pull/24); evidence recorded by [PR #23](https://github.com/irerin07/red-green-prune/pull/23) |
| [016](016-delivery-eligibility-api.md) | Delivery eligibility API | Focused routing worked and prevented batching/placeholders; the final validation cycle lacked captured pre-production RED evidence | Evidence only; no skill change ([PR #25](https://github.com/irerin07/red-green-prune/pull/25)) |
| [017](017-benefit-eligibility-api.md) | Benefit eligibility API | Routing, decision separation, and late validation improved; status and eligibility were still batched, with incomplete UUID/echo sensitivity | Evidence in [PR #26](https://github.com/irerin07/red-green-prune/pull/26); executable cycle gate in [PR #27](https://github.com/irerin07/red-green-prune/pull/27) |
| [018](018-training-eligibility-api.md) | Training eligibility API | Authorization gate separated 404/status from business rules and improved sensitivity; reporting remained verbose | [PR #28](https://github.com/irerin07/red-green-prune/pull/28) |

## Recording policy

Record the request, environment, observed evidence, expected behavior, and any
resulting rule. Keep raw transcripts outside the repository unless they are
needed to verify a disputed observation. Do not change the skill from a
hypothetical failure alone; reproduce or observe the behavior first.
