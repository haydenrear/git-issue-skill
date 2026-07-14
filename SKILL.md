---
name: git-issue
description: >-
  Use when creating a tracker issue that an agent will pick up and implement —
  especially GitHub issues via the `gh` CLI. Drives a discovery-first workflow:
  scan the repo before writing, embed a References section as the implementer's
  discovery starting point, tell the implementer to create a worktree and
  feature branch, optionally bind the issue to one ticket in an owner-created
  epic/* shared spec workflow, decide whether a TLA+ spec workflow is needed
  (Internal.tla / External.tla changes, test-graph and unit-test adapter updates
  opened with spec-double-compiler + tla-spec-dev), and spell out the regression
  test graphs, spec/unit tests, spec-ticket close-out, and commit/push that finish
  the work. Trigger on "file an issue", "create a ticket", "open a GitHub issue",
  scheduling an issue in an epic branch, or planning work that another agent will
  implement.
skill-imports:
  - unit: spec-double-compiler
    path: SKILL.md
    reason: When an issue needs a spec workflow, it drives Internal.tla/External.tla spec doubles and spec unit tests via the tla-spec-dev CLI.
  - unit: test-graph
    path: SKILL.md
    reason: The issue's regression section names test_graph graphs to run for regression, including tla-spec-dev spec-graph integrations.
  - unit: deploy-helm
    path: SKILL.md
    reason: Issues that touch deployable surfaces reference deploy-helm environments for validation.
  - unit: skill-manager
    path: references/workflows.md
    reason: This skill is installed and synced as a skill-manager unit.
---

# git-issue

Create issues that another agent can pick up and implement end-to-end without
re-discovering the whole repository. An issue produced by this skill is a
**work order**: it carries the discovery you already did, points the implementer
at a worktree/branch, decides up front whether the change needs a TLA+ spec
workflow, and lists exactly which tests close it out.

An issue may instead be one scheduled slice of an existing epic. In that mode,
keep the ordinary work-order sections and add the marker-delimited assignment
from `references/epic-assignment.md`. That assignment takes precedence over
ordinary default-branch worktree, PR-target, and issue-close instructions.

The core workflow is **tracker-agnostic**. GitHub via the `gh` CLI is the
default and only wired adapter; see `references/github-gh.md`. To target another
tracker, keep every step below and swap only the create/comment/close commands.

## The five moves

1. **Discover before you write.** Never open an issue from the title alone.
   Scan the repo to locate the code, specs, tests, and docs the change will
   touch, and capture what you find. See `references/discovery.md`.
2. **Write the issue with a References section.** The issue body embeds the
   discovery output as a *starting point* for the implementer — concrete files,
   symbols, specs, and docs, each as `path:line` where possible. Create it with
   `gh issue create`. See `references/github-gh.md` and the template below.
3. **Instruct a worktree + feature branch.** The issue tells the implementer to
   create a dedicated git worktree and feature branch named for the issue before
   touching anything. An epic assignment instead names the exact worktree and
   branch created from the epic branch. See `references/worktree-branch.md` and
   `references/epic-assignment.md`.
4. **Decide the spec workflow.** During discovery, decide whether the change
   alters observable/internal state-machine behavior. If yes, the issue names
   the Internal.tla / External.tla edits, the test-graph and unit-test adapter
   updates, and tells the implementer to open the spec workflow with
   **spec-double-compiler + tla-spec-dev** at branch-creation time. For an epic,
   the owner scaffolds one shared workflow and each issue opens exactly its one
   planned ticket; the ticket agent never scaffolds again. See
   `references/spec-workflow.md`.
5. **Spell out close-out.** The issue lists which test graphs run for regression
   (including tla-spec-dev spec-graph integrations), tells the implementer to
   attach those reports to the spec ticket that closes in the repo, close that
   ticket via spec-double-compiler + tla-spec-dev, run spec unit tests and unit
   tests, then commit and push to the feature branch. An epic work order supplies
   exact validation commands and evidence paths, targets its PR at the epic
   branch, and stops for external review. See `references/regression-close.md`.

## Before you create the issue

Run discovery (`references/discovery.md`) and answer these, because they change
what the issue body must contain:

- **What does this touch?** The file/symbol list becomes the References section.
- **Does it change state-machine behavior?** Decides the spec-workflow section
  (move 4). Err toward yes whenever the change alters what an external caller can
  observe or an internal invariant.
- **What could it regress?** The affected test graphs become the close-out
  checklist (move 5).
- **Ordinary issue or epic assignment?** Epic mode is valid only after the epic
  owner has created and pushed the epic branch, scaffolded the shared workflow
  once, and planned the issue's one stable spec ticket. Collect the assignment
  fields in `references/epic-assignment.md`; do not invent or scaffold a second
  workflow while authoring the issue.

## Issue body template

Fill every section. Omit a section only when it is genuinely N/A, and say so
explicitly rather than deleting it — the implementer relies on the shape.

```markdown
## Summary
<one paragraph: the change and why it matters>

## References  <!-- discovery starting point; not exhaustive -->
- `path/to/file.ext:LINE` — <why it's relevant>
- `path/to/Spec.tla` — <state machine this touches, if any>
- <doc / ADR / prior issue / PR links>

## Discovery notes
<what you learned scanning the repo: entry points, invariants, adjacent code,
open questions the implementer should resolve first>

## Worktree & branch
Create a dedicated worktree and feature branch before editing:
`git worktree add ../wt-<issue-slug> -b feature/<issue-number>-<slug>`
(see references/worktree-branch.md)

## Spec workflow — REQUIRED | NOT REQUIRED
<!-- If REQUIRED, keep this block; if NOT REQUIRED, state why in one line. -->
On feature-branch creation, open the spec workflow with spec-double-compiler +
tla-spec-dev. Expected changes:
- **Internal.tla**: <state/vars/actions to add or change>
- **External.tla**: <observable behavior / interface to add or change>
- **Test graph**: <spec-graph nodes/cases to add or update>
- **Unit-test adapters**: <adapter conformance tests to add or update>

## Regression & close-out
Run these to close the issue:
- **Test graphs**: <named graphs>, including the tla-spec-dev spec-graph integration graph
- Attach the test-graph reports to the spec ticket that closes out in the repo
- Close the spec ticket via spec-double-compiler + tla-spec-dev
- Run spec unit tests and unit tests
- Commit and push to `feature/<issue-number>-<slug>`
```

For epic mode, retain every standard section above and insert the rendered
marker-delimited block from `references/epic-assignment.md` after the completed
Summary section and before `## References`. The block is the machine-readable
override; do not merely describe the epic in prose.

## Operating order

1. Confirm `gh` is authenticated and the repo is right
   (`references/github-gh.md`).
2. Discover (`references/discovery.md`) → collect References + regression scope.
3. Decide the spec workflow (`references/spec-workflow.md`). If the epic owner
   supplied a planned ticket, select epic mode and validate its assignment
   (`references/epic-assignment.md`).
4. Render the template, create the issue with `gh issue create`, capture the
   issue number/URL. For a new epic issue, use that number to finalize the
   feature branch and worktree fields, then edit the body with the complete
   assignment. For an existing issue, preserve its body and replace only the
   bounded assignment region.
5. For an ordinary required spec workflow, note that the implementer opens the
   in-repo ticket on branch creation. For an epic, verify the issue maps to one
   existing planned ticket and instruct the implementer to run only `open ticket
   <id>` against the already-scaffolded shared workflow.

## Boundaries

- This skill **creates and structures** issues; it does not implement them. The
  implementing agent follows the worktree/spec/close-out instructions the issue
  carries.
- It does not run test graphs or the spec workflow itself — it names them so the
  implementer runs them. The mechanics live in the referenced skills
  (test-graph, spec-double-compiler, deploy-helm).
- It does not close issues automatically; close-out is the implementer's final
  commit/push plus ticket close.
- It does not create an epic branch or scaffold a shared epic workflow. The epic
  owner supplies those artifacts and the canonical ticket plan before this
  skill writes an epic assignment.
- An epic ticket PR uses `Refs #<issue>`, targets the declared epic branch, and
  stops for external review. It does not close the GitHub issue or merge into
  either the epic or default branch.
