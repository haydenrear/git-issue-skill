# Regression & close-out

The issue's final section is a checklist that tells the implementer exactly how
to prove the change is safe and finish it. Name these concretely during
discovery so the implementer runs them without guessing.

## Epic assignment override

When the issue contains an epic assignment, its validation matrix and evidence
root are mandatory and override generic examples below. Run the exact TLC,
spec-unit, repository-unit, and named graph commands (or retain the explicit
`N/A: <reason>`), store their reports below `validation.evidence_root`, and pass
those paths to `tla-spec-dev ... close ticket <stable-ticket-id>`. Close and
promote only the assigned ticket after reconciling the latest epic tip and
rerunning the full matrix; never use `--accept-new` or the whole-workflow close
script.

Use the repository-specific graph names in the assignment. `specWorkflow` is
the tla-spec-dev repository's own CLI-lifecycle graph and is not implicitly
required in other repositories.

```bash
tla-spec-dev --spec-root specs close ticket <stable-ticket-id> \
  --summary "<what landed>" \
  --result <evidence-path> \
  --result <another-evidence-path>
```

After commit and push, open a PR whose base is `ticket.pr_base` (the epic branch)
and whose body uses `Refs #<issue-number>`. Include the exact commands, evidence
paths, close-history path, and resulting commit SHA, then stop for external
review. Do not self-merge, target the default branch, or close the GitHub issue.

## 1. Run the regression test graphs

List the specific `test_graph` graphs that cover the changed surface (from
discovery). These run via the **test-graph** skill. Always include, when the
issue had a spec workflow, the **tla-spec-dev spec-graph integration graph** —
the graph that exercises the generated spec doubles against the real adapters,
so a spec/impl divergence fails loudly.

```bash
# in the test_graph project (see the test-graph skill for exact invocation)
# run the named graphs, e.g.:
./gradlew testGraph --graph <named-graph>
# and the spec-graph integration graph when a spec workflow ran
```

## 2. Attach reports to the spec ticket

The test-graph run produces aggregated reports. Attach those reports to the
**in-repo spec ticket** that this issue closes out (the ticket opened during the
spec workflow — `references/spec-workflow.md`). This ties the passing evidence to
the ticket that gets closed, not just to the PR.

## 3. Close the spec ticket via spec-double-compiler + tla-spec-dev

Close the in-repo spec ticket using **spec-double-compiler + tla-spec-dev** (the
`tla-spec-dev` CLI ticket-close path). This records that the spec, doubles,
graph cases, and adapters all landed and passed together.

## 4. Run spec unit tests and unit tests

Before committing, run both layers and require green:

- **Spec unit tests** — the generated/spec-double unit tests (spec-double-compiler).
- **Unit tests** — the repo's own unit test suite for the changed code.

## 5. Commit and push to the feature branch

Only after 1–4 pass:

```bash
git add -A
git commit -m "<issue-number>: <what changed> — specs, graph, adapters, tests"
git push -u origin feature/<issue-number>-<slug>
```

Then open the PR (`Closes #<issue-number>`) or close the issue with a summary of
the graphs run and reports attached (`references/github-gh.md`).

That final sentence applies only to an ordinary issue. Epic ticket PRs use
`Refs`, remain open for external review, and leave issue closing to epic
finalization.

## Checklist to embed in the issue

- [ ] Named test graphs pass (incl. tla-spec-dev spec-graph integration graph)
- [ ] Test-graph reports attached to the in-repo spec ticket
- [ ] Spec ticket closed via spec-double-compiler + tla-spec-dev
- [ ] Spec unit tests pass
- [ ] Unit tests pass
- [ ] Committed and pushed to `feature/<issue-number>-<slug>`
