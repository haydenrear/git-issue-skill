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

## One command, and where it is

A worktree needs two things that must happen in one step: the checkout, and the
worktree's own Skill Manager home. `wt new` does both. It handles a plain repo
and an integration repo identically — it detects which it is standing in — so
there is exactly one instruction to embed, whatever the target repository is.

**Spell it resolved.** The script is shipped by the `git-integration-repo` unit,
and an installed unit's files live at `$SKILL_MANAGER_HOME/skills/<unit>/`. Write
that path out; never write `<git-integration-repo-skill>/…` or any other
placeholder the implementer has to resolve. That placeholder is the measured
cause of two field failures: one agent ran the wrong script, another concluded it
had to write its own (git-issue#4).

```bash
WT="${SKILL_MANAGER_HOME:-$HOME/.skill-manager}/skills/git-integration-repo/scripts/wt"
```

`SKILL_MANAGER_HOME` is exported by the launch shims every agent starts through;
the `:-` fallback is what makes the same line work from a bare shell.

## Naming

- Branch: `feature/<issue-number>-<short-slug>` (e.g. `feature/142-retry-budget`),
  which is what `wt new <issue-number>-<slug>` creates.
- Worktree dir: **`wt` chooses it** — `<parent>/<repo-name>-<ticket>`, placed
  beside the outermost enclosing integration repo so a nested repo's worktree
  never lands in a parent's `constituents/`. It is *not* `../wt-<ticket>`.
  The issue must tell the implementer to read the path off the command's output
  rather than assume one; an issue that says `cd ../wt-<slug>` sends it to a
  directory that does not exist.

## The instruction to embed in the issue

```bash
WT="${SKILL_MANAGER_HOME:-$HOME/.skill-manager}/skills/git-integration-repo/scripts/wt"

# from anywhere inside the repo
"$WT" new <issue-number>-<slug>
# -> created worktree /path/to/<repo>-<issue-number>-<slug>
cd /path/to/<repo>-<issue-number>-<slug>     # the path it just printed
```

A successful run prints that one line and nothing on stderr. To branch from
something other than the current HEAD, pass it: `"$WT" new <ticket> origin/main`.

**The one refusal worth pre-empting.** On the first ticket in a repository that
has never been given a home, `wt new` exits **3**:

```
error creating worktree: no Skill Manager home could be created for this worktree
  (usually: /path/to/repo has no project home yet)
fix: /path/to/home/skills/git-integration-repo/scripts/bootstrap-home.sh --root /path/to/repo
log: /tmp/wt-XXXXXX-run.log
```

The `fix:` line is already absolute and already resolved — run it verbatim, then
re-run the same `wt new`. It is a one-time step **per repository**, not per
worktree, and it is why the issue should say "if it exits 3, run the fix line it
prints" instead of leaving the implementer to interpret a refusal.

Everything else `wt` can tell you is on demand and costs nothing until asked:
`"$WT" info <ticket>` prints WORKTREE / BRANCH / LAUNCH / IF-EXIT-8 / CLOSE (and
PROPAGATE, only in an integration repo) for a worktree that already exists.

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
Create the worktree AND its own Skill Manager home with ONE command, from the
repo root. Same command for a plain repo and an integration repo:
`WT="${SKILL_MANAGER_HOME:-$HOME/.skill-manager}/skills/git-integration-repo/scripts/wt"`
`"$WT" new <issue-number>-<slug>`

It prints `created worktree <path>` — cd to that path (it is
`<parent>/<repo>-<issue-number>-<slug>`, not `../wt-...`).
If it exits 3 with "no project home yet", run the absolute `fix:` line it printed
(one time for this repository), then re-run the same `wt new`.
Do not substitute `git worktree add` — that leaves the worktree with no home, and
an agent launched in it writes the operator's global `~/.skill-manager`.

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
WT="${SKILL_MANAGER_HOME:-$HOME/.skill-manager}/skills/git-integration-repo/scripts/wt"
"$WT" close <issue-number>-<slug>
```

One command, both repo shapes, and it runs the gate and the removal **in that
order**, refusing the removal on a non-zero verdict. Prefer it to spelling the
two steps out: an issue body gets pasted verbatim, and two commands on separate
lines run the removal whatever the gate returned — which is the exact loss the
gate exists to prevent. (If you do spell them out, the `&&` is load-bearing:
`skill-manager home close-out --home <worktree>/.skill-manager --into
<repo-root>/.skill-manager && git worktree remove <worktree>`.)

`wt close` resolves the ticket by searching where ticket worktrees live, so it
works from a checkout other than the one that opened the worktree — but it must
be run from inside **some** git repository; from a non-repo directory it exits 1
with `not inside a git repository`. A successful close prints one line naming the
worktree, the branch that outlives it, and the home tier its skill work reached.

**Exit 1** names each blocking unit and the literal command that clears it —
`skill-manager unit publish <unit>` to reach the unit's own repository, or
`skill-manager home sync --from … --to … --merge` to lift it into the project home
so the teardown does not take it. **Exit 2** means the `--home` path is not a home
at all (usually the worktree directory was passed instead of its
`.skill-manager`), and **exit 9** means the destination home is frozen so nothing
was attempted; neither prints blockers, because neither assessed anything.
`"$WT" close <ticket>` wraps all of this — it is the same gate followed by the
removal, in an integration repo and a plain one alike. Put it in the issue's
close-out checklist; the implementer will not invent it.

For an epic ticket, external review owns merge and issue close. Do not remove
the worktree merely because the ticket agent opened its PR — but do run the gate
and record its verdict in the PR body, because the epic finalizer is the one who
will remove the worktree and cannot see inside its home.
