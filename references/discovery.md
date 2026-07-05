# Discovery — scan before you write

An issue is only useful if it points the implementer at the right code. Do this
scan first; its output fills the `## References`, `## Discovery notes`, the
spec-workflow decision, and the regression scope.

## What to find

1. **Entry points and owners.** Where does the behavior live now? Grep for the
   feature's nouns/verbs, the public API surface, the config that toggles it.
2. **State machine surface.** Search for `*.tla` specs and the generated spec
   doubles. If the change alters observable or internal state-machine behavior,
   note the specific spec (`Internal.tla` / `External.tla`) — this decides the
   spec-workflow section. See `references/spec-workflow.md`.
3. **Tests and graphs.** Locate the `test_graph/` project, the graphs in
   `build.gradle.kts`, the unit tests, and any adapter conformance tests. What
   graph would catch a regression in this area? That list becomes close-out.
4. **Docs / prior art.** ADRs, READMEs, prior issues/PRs on the same surface.

## How to record it

For each relevant location capture `path:line` and a one-line reason. That pair
is exactly what the `## References` bullets need. Keep it a *starting point*,
not an exhaustive map — the implementer discovers the rest from these anchors.

Useful sweeps (adjust tools to the repo):

```bash
# state-machine specs and their doubles
git ls-files '*.tla' 'spec_double*/**' 2>/dev/null
# the test-graph project and composed graphs
git ls-files 'test_graph/**' 'build.gradle.kts'
# the feature's code by keyword
git grep -n "<feature-keyword>"
```

## Decisions to lock before writing

- **References list** — the files/symbols/specs above.
- **Spec workflow: REQUIRED or NOT** — REQUIRED if any `Internal.tla` /
  `External.tla` behavior or invariant changes. When unsure, mark REQUIRED; a
  no-op spec pass is cheap, a missed one is not.
- **Regression scope** — the named test graphs (including the tla-spec-dev
  spec-graph integration graph) that must pass to close the issue.

Carry these three into the issue body template in `SKILL.md`.
