# Epic assignment — author one shared-workflow ticket

Use this mode only when an epic owner has already created and pushed an
`epic/<slug>` branch, scaffolded one shared spec workflow there, and planned the
ticket in `specs/desired_program_model/ticket_plan.yaml`. `git-issue` authors the
work order; it does not create those workflow-wide artifacts.

Each GitHub issue maps to exactly one stable spec ticket. Keep the ordinary
Summary, References, Discovery notes, Worktree & branch, Spec workflow, and
Regression & close-out sections, then add this bounded assignment after the
completed Summary section and before References. Its presence selects epic mode
and overrides every ordinary instruction to branch from, open a PR against,
merge to, or close against the default branch.

## Assignment block

Render every field; do not leave placeholders in a dispatched issue.

````markdown
<!-- git-epic-workflow:assignment:start -->
## Epic execution — REQUIRED

```yaml
version: 1
epic:
  id: "<epic-id>"
  workflow: "<unique-workflow-name>"
  branch: "epic/<slug>"
  base_sha: "<commit-reachable-from-origin-epic>"
  plan_commit: "<commit-containing-canonical-plan>"
  schedule_revision: 1
  default_branch: "<default-branch>"
ticket:
  spec_id: "<stable-ticket-id>"
  feature_branch: "feature/<issue-number>-<slug>"
  worktree: "../wt-<issue-number>-<slug>"
  pr_base: "epic/<slug>"
  depends_on: []
  blocks: []
  wave: 1
  promotion_order: 10
  promotion_predecessor: null
  conflict_keys:
    production: []
    tla: []
    adapters: []
    test_graph: []
    workflow: []
validation:
  tlc: "<exact command or N/A: reason>"
  spec_unit: "<exact command>"
  repository_unit: "<exact command or N/A: reason>"
  graphs: ["<affected-repository-graph>"]
  spec_graph: "<repository spec-conformance graph or N/A: reason>"
  toolchain_spec_workflow: "N/A unless this repository is tla-spec-dev"
  evidence_root: "<ticket-results-path>"
review:
  mode: "external"
  ticket_agent_stops_after: "pr_open"
```

This issue belongs to an existing shared spec workflow. The epic assignment
overrides ordinary instructions to branch from or target the default branch.

- Start the worktree from the latest `origin/epic/<slug>` after all
  `depends_on` PRs are merged.
- Run `tla-spec-dev --spec-root specs open ticket <stable-ticket-id>`; never
  scaffold another workflow.
- Before close, wait for `promotion_predecessor`, reconcile the latest epic tip,
  and rerun the validation matrix.
- Mark and close only this spec ticket with every evidence path. Never run the
  whole-workflow close script and never use `--accept-new`.
- Push the sealed ticket branch and open its PR with base `epic/<slug>` and
  `Refs #<issue-number>`. Stop for external review; do not merge to the default
  branch or close the GitHub issue.
<!-- git-epic-workflow:assignment:end -->
````

## Validate before dispatch

- `epic.branch` exists remotely, `base_sha` and `plan_commit` are reachable
  from it, and
  `ticket.pr_base` exactly equals `epic.branch`.
- `epic.workflow` names the existing shared workflow, and `ticket.spec_id`
  identifies exactly one entry in its canonical ticket plan. The GitHub issue
  URL is recorded on that same entry.
- `schedule_revision`, `depends_on`, `blocks`, `wave`, conflict keys,
  `promotion_order`, `promotion_predecessor`, the validation matrix, and the
  evidence root exactly match the canonical plan entry. The assignment does not
  silently revise workflow-wide planning data.
- `validation.tlc`, `validation.spec_unit`, and `validation.repository_unit` are
  exact runnable commands; an inapplicable command is `N/A: <reason>`, never an
  empty string. `validation.graphs` contains exact repository graph names and
  `validation.spec_graph` names the repository's spec-conformance graph or an
  explicit N/A reason. `specWorkflow` is the tla-spec-dev toolchain's own graph
  and appears only when that repository is the target. The evidence root is a
  ticket-specific destination for reports and close evidence.
- Review remains external and the stop point remains `pr_open`.
- The body contains exactly one start marker and one end marker.

For a new issue, create the ordinary work order first, capture its number and
URL, write the URL into the canonical ticket plan, then render the final branch,
worktree, and assignment fields and update the body. For an existing issue,
preserve its body and replace only the bounded assignment. Never append a second
copy when resuming an epic.
