# Deciding and instructing the spec workflow

Some issues change **state-machine behavior**; those must carry a spec workflow
so the TLA+ specs, generated Python doubles, test-graph cases, and adapter
conformance tests move together. This is driven by the **spec-double-compiler**
skill (installed from `github:haydenrear/tla-spec-dev`), which provides the
`tla-spec-dev` CLI. This reference is about what the *issue* must say — the
mechanics live in that skill.

## Epic assignment override

An epic owner scaffolds the shared workflow exactly once on the epic branch and
plans all stable ticket IDs before dispatch. An epic issue therefore always
marks the spec workflow **REQUIRED**, maps to exactly one existing plan entry,
and embeds that ID in `ticket.spec_id`; the ticket agent runs:

```bash
tla-spec-dev --spec-root specs open ticket <stable-ticket-id>
```

It must not scaffold another workflow, create an ad hoc ticket, edit sibling
plan statuses, or run workflow-wide close/promotion. It updates the assigned
ticket's TLA+ model, spec-unit adapters, generated cases, Test Graph adapters
and graphs, and then closes only that ticket with the evidence declared in the
assignment. See `references/epic-assignment.md`.

## Decide: REQUIRED or NOT

Mark the Spec workflow section **REQUIRED** when the change:

- alters what an external caller can observe → `External.tla`, or
- adds/changes internal state, variables, actions, or invariants →
  `Internal.tla`, or
- changes any behavior currently covered by generated spec doubles / spec unit
  tests.

Mark **NOT REQUIRED** only for changes with no state-machine surface (docs,
pure refactors with identical behavior, build tweaks). State the reason in one
line so the implementer can challenge it.

When unsure, choose REQUIRED. A spec pass that turns out to be a no-op is cheap;
a behavior change that skipped the spec is a silent regression.

## What the issue must specify when REQUIRED

Fill the template's Spec workflow block with concrete, discovered edits:

- **Internal.tla** — the vars/actions/invariants to add or change.
- **External.tla** — the observable interface/behavior to add or change.
- **Test graph** — the spec-graph nodes/cases to add or update so the new
  behavior is exercised (see `references/regression-close.md` and the
  test-graph skill).
- **Unit-test adapters** — the adapter conformance tests that must be added or
  updated so the real implementation matches the generated double.

## The instruction to embed

> On feature-branch creation, open the spec workflow with **spec-double-compiler
> + tla-spec-dev**: edit `Internal.tla`/`External.tla` as above, regenerate the
> spec doubles, add/update the test-graph spec cases and adapter conformance
> tests, and open the in-repo spec ticket that this issue closes out.

The spec ticket opened here is what the close-out step attaches reports to and
closes via `tla-spec-dev` — see `references/regression-close.md`. Cross-link the
ticket back to the GitHub issue (`references/github-gh.md`).

That instruction is for an ordinary issue. In epic mode, replace "open the spec
workflow" with "open the assigned ticket in the existing shared workflow" and
retain the assignment's one-ticket-only boundary.
