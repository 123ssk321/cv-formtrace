# Implementation and tracking protocol

The current deliverables are plans and issue backlogs. Implementation begins when the user activates a project. One active project receives the 16-hour weekly budget; all other plans remain queued. Relative months begin at activation, not repository creation. A public planning repository does not count as a functional release.

## Model roles

GPT-6 Astra with high reasoning effort designs the architecture and execution plan. GPT-6 Luna with medium reasoning effort maintains GitHub tracking. GPT-6 Luna or GPT-5.6 with medium reasoning effort can implement bounded issues once implementation is requested. Escalate changes to the product scope, dataset rights, evaluation protocol, architectural interfaces, or budget to an Astra design review; routine coding choices stay with the implementer.

## Starting an issue

1. Read README.md, docs/EXECUTION_PLAN.md, and planning/backlog.json in the selected repository. Check the linked GitHub Project for the current status and any newer issue discussion.
2. Select one task whose dependencies are complete. For an epic, create the bounded child tasks described in its body before coding. Assign hours to children by redistributing the epic estimate; do not count both totals.
3. Record the active task in GitHub Projects. The portfolio-wide work-in-progress limit is one implementation issue. Branch from the current default branch and preserve unrelated work.
4. Follow the issue's steps and output contract. Public datasets with existing annotations are permitted when their licenses fit. Generate new synthetic labels automatically. Manual data collection, labeling, or per-example curation is outside scope. Inspecting failures and obtaining workflow feedback are permitted, but do not silently turn them into a labeling dependency.

## Completing an issue

1. Run the meaningful validations listed in its acceptance criteria and record commands, versions, inputs, and observed results. Distinguish synthetic, public real-data, and customer-workflow evidence.
2. Open a pull request when a code change is ready for review. Link the issue and include evidence. Creating an issue or writing a plan is not completion of its future implementation work.
3. Update the Project status only to reflect actual progress. Close an issue after its acceptance criteria and review requirements pass. Report blockers with the missing dependency and next feasible action; leave dependent issues queued.
4. Update provenance and licensing records when dependencies, model weights, or datasets change. A non-commercial benchmark needs explicit permission before use in commercial development or evaluation.
5. At each milestone, record used hours, results, remaining risk, and the continue/reduce/stop decision. Re-estimate downstream work against the remaining budget.

## Tracking state

Use GitHub issues and Projects for live execution status; keep stable task IDs and technical scope in planning/backlog.json. Capture accepted scope changes in both with a linked decision record. Use gh to inspect issues, milestones, and project items before creating anything, preventing duplicates. Credentials belong in the platform's secret store, never in issue bodies or files. Plans and publication do not authorize paid infrastructure, purchases, or messages to external people.

Recurring tracking is not configured by this setup. On a future tracking run, reconcile GitHub state, update only evidence-supported progress, and report a concise next-action list.
