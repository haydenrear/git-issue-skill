# Worktree & feature branch

The issue instructs the implementer to work in a **dedicated git worktree on a
feature branch** — never on the default branch, never in the primary checkout.
This keeps the change isolated, lets the spec workflow (which generates files)
run without polluting the main tree, and makes the branch↔issue link explicit.

## Epic assignment override

Before embedding the ordinary command below, check for the
`git-epic-workflow:assignment:start` marker. When present, the assignment in
`references/epic-assignment.md` wins: use its exact feature branch and worktree,
wait until every `depends_on` PR is merged into the declared remote epic branch,
fetch, and create the worktree from the latest `origin/epic/<slug>`. Never
substitute `origin/<default-branch>` in epic mode. The ticket PR also targets the
declared epic branch rather than the default branch.

## Naming

- Branch: `feature/<issue-number>-<short-slug>` (e.g. `feature/142-retry-budget`).
- Worktree dir: a sibling of the repo, `../wt-<issue-number>-<slug>`, so tooling
  and IDEs don't index it as nested.

## The instruction to embed in the issue

```bash
# from the repo root, branching off the up-to-date default branch
git fetch origin
git worktree add ../wt-<issue-number>-<slug> -b feature/<issue-number>-<slug> origin/main
cd ../wt-<issue-number>-<slug>
```

Adjust `origin/main` to the repo's default branch.

## What the issue must say about the worktree's Skill Manager home

A worktree is not only a checkout — it carries its own Skill Manager home at
`<worktree>/.skill-manager`, a **real copy** (not a symlink) of the repository's
`<repo>/.skill-manager`, which is itself a copy of the operator's
`~/.skill-manager`. Copies, so that two tickets in two worktrees cannot silently
overwrite each other's skills.

That home is gitignored. Every consequence follows from that one fact, and an
issue that omits them hands the implementer a way to lose work with no trace:

1. **The home has to exist before anything mutating runs.** `install`, `sync`,
   `bind`, `upgrade` and `project resolve` write into whatever
   `SKILL_MANAGER_HOME` names, and before the local home exists that is the
   operator's global home. Say which command creates it.
2. **A skill edit made inside the home is in no diff.** Not in `git status`, not
   in the PR, not in an integration fan-out. It must be published to the unit's
   own repository, or it is deleted with the worktree.
3. **Teardown is gated.** `git worktree remove` deletes the home without asking.
   The issue must name the gate, not leave it to be remembered.

State it concretely rather than in the abstract, e.g.:

```markdown
## Worktree & branch
Create a dedicated worktree and feature branch before editing:
`git worktree add ../wt-<issue-number>-<slug> -b feature/<issue-number>-<slug> origin/main`

The worktree carries its own Skill Manager home. Bootstrap it immediately after
creation and before any `install`/`sync`/`bind`/`project resolve`
(an integration repo's `new-change.sh` does this for you):
`<git-integration-repo-skill>/scripts/bootstrap-home.sh --root ../wt-<issue-number>-<slug>`

Launch agents through `<worktree>/.skill-manager/bin/launch/{claude,codex,gemini}`.
If you improve a skill while working this ticket, that edit lives in the
gitignored home and is in no diff — publish it with
`skill-manager unit publish <unit> --ticket <issue-number>` before teardown.
(see references/worktree-branch.md)
```

If the repository is not onboarded to per-checkout homes, say **that** instead of
saying nothing — "this repo has no project home; agents run the operator's global
`~/.skill-manager`, so do not install or sync anything" is a complete and useful
instruction. Silence reads as "there is nothing to know here".

## Ordering with the spec workflow

Branch/worktree creation is the trigger point for the spec workflow. If the
issue's Spec workflow section is REQUIRED, the implementer opens
spec-double-compiler + tla-spec-dev **immediately after** creating the branch,
so the generated spec doubles and manifests are committed on the feature branch
from the start. See `references/spec-workflow.md`.

## Cleanup

After the branch merges and the issue closes, the implementer removes the
worktree — but **the home is checked first**, because the removal is what destroys
it:

```bash
skill-manager home close-out --home ../wt-<issue-number>-<slug>/.skill-manager \
                             --into <repo-root>/.skill-manager \
  && git worktree remove ../wt-<issue-number>-<slug>
```

Keep the `&&` when you embed this. An issue body gets pasted verbatim, and two
commands on separate lines run the removal whatever the gate returned — which is
the exact loss the gate exists to prevent.

**Exit 1** names each blocking unit and the literal command that clears it —
`skill-manager unit publish <unit>` to reach the unit's own repository, or
`skill-manager home sync --from … --to … --merge` to lift it into the project home
so the teardown does not take it. **Exit 2** means the `--home` path is not a home
at all (usually the worktree directory was passed instead of its
`.skill-manager`), and **exit 9** means the destination home is frozen so nothing
was attempted; neither prints blockers, because neither assessed anything. In an
integration repo,
`<git-integration-repo-skill>/scripts/close-change.sh <ticket>` runs the gate and
the removal in that order and refuses on a non-zero verdict. Put whichever of
these applies in the issue's close-out checklist; the implementer will not invent
it.

For an epic ticket, external review owns merge and issue close. Do not remove
the worktree merely because the ticket agent opened its PR — but do run the gate
and record its verdict in the PR body, because the epic finalizer is the one who
will remove the worktree and cannot see inside its home.
