# FormTrace

Status: planning only. No application, benchmark results, deployment, or product validation is implemented or claimed by these files.

FormTrace is a narrow intake tool for single-page, Portuguese-language EUR invoices: extract eight header/total fields, show their source evidence, route uncertainty to review, and export an auditable JSON/CSV record.

Public repository: [123ssk321/cv-formtrace](https://github.com/123ssk321/cv-formtrace), confirmed public by the parent. Public planning from day one does not count as the first functional public result. This planning task creates no remote repository, issues, or deployment.

## Start here

- [Execution plan](docs/EXECUTION_PLAN.md): scope, contracts, data rights, milestones, metrics, release and maintenance gates.
- [Importable backlog](planning/backlog.json): 32 bounded tasks, dependencies, acceptance criteria, implementation/testing instructions and output paths. Intended for later import using `gh` by Luna; do not treat import as implementation.

## Time and budget

No project is activated by this plan. When FormTrace is activated, it is the only active project. All other projects remain parked until explicitly activated; the 16 hours/week budget is global, not per project.

| Allocation | Hours |
| --- | ---: |
| 32 tasks, each 6-12 hours | 336 |
| Explicit contingency reserve | 80 |
| Total at 16 hours/week | 416 / 26 weeks |

Reserve is 19.2% of the total. First functional public result: month 4, target week 16 (220 task hours + 36 reserve). Validation, private deployment hardening and maintenance finish by week 26, approximately month 6 (116 task hours + 44 reserve). Months are relative calendar months from activation; week numbers govern scheduling.

## Boundaries

Use procedural synthetic documents with exact automatically generated labels by default. Existing public annotations are allowed if their licenses cover the intended commercial development and use. No new manual annotation, data collection campaign, or manual dataset cleanup is required or permitted by this plan. A noncommercial dataset is not a commercial-development shortcut, including for evaluation or tuning.

The public demo processes bundled/generated fictional samples; the local package supports bounded file upload. User interviews are optional, and nobody is to be contacted as part of this plan. Review corrections are product actions and never automatically become training labels.

Default local OCR: Tesseract 5 with `por.traineddata` from the official `tessdata_fast` repository, separately pinned and license-checked. One alternate page-segmentation retry is allowed; unresolved cases go to review. The execution plan gives the exact adapter behavior and primary sources.

Conservative implementation agents follow one ready backlog issue at a time after explicit activation. Luna imports the backlog later; the parent supplies `AGENTS.md`. Escalate proposed scope, architecture, OCR-engine or budget changes to GPT-6 Astra at high effort with a concrete decision brief; do not silently redesign the project.

