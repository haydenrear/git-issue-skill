# Deciding and instructing the spec workflow

Some issues change **state-machine behavior**; those must carry a spec workflow
so the TLA+ specs, generated Python doubles, test-graph cases, and adapter
conformance tests move together. This is driven by the **spec-double-compiler**
skill (installed from `github:haydenrear/tla-spec-dev`), which provides the
`tla-spec-dev` CLI. This reference is about what the *issue* must say — the
mechanics live in that skill.

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
