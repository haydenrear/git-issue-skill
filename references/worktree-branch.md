# Worktree & feature branch

The issue instructs the implementer to work in a **dedicated git worktree on a
feature branch** — never on the default branch, never in the primary checkout.
This keeps the change isolated, lets the spec workflow (which generates files)
run without polluting the main tree, and makes the branch↔issue link explicit.

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

## Ordering with the spec workflow

Branch/worktree creation is the trigger point for the spec workflow. If the
issue's Spec workflow section is REQUIRED, the implementer opens
spec-double-compiler + tla-spec-dev **immediately after** creating the branch,
so the generated spec doubles and manifests are committed on the feature branch
from the start. See `references/spec-workflow.md`.

## Cleanup

After the branch merges and the issue closes, the implementer removes the
worktree:

```bash
git worktree remove ../wt-<issue-number>-<slug>
```
