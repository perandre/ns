# ADR 0002 — Opt-out default for repo participation

## Context

The first version of the multi-repo wrappers required each project to have a `## Night Shift Config` section in its `CLAUDE.md` to participate. Projects without that section were skipped. The reasoning was conservative — don't touch repos that haven't been "onboarded".

In practice this had two problems:

1. **Some projects don't have a `CLAUDE.md` file at all** and there's no good reason they should.
2. **Onboarding friction.** Adding a project meant editing its `CLAUDE.md`, committing, pushing, then editing the trigger sources. Two-step setup discouraged adoption.

## Decision

Flip the default: **all cloned repos participate unless they explicitly opt out.** A project is skipped only if:

- A file `.nightshift-skip` exists at the repo root, OR
- A line `Night Shift: skip` appears in `CLAUDE.md`, `AGENTS.md`, or `README.md`.

Without `CLAUDE.md`, the wrapper falls back to autodetected defaults: test/build commands inferred from the lockfile, push protocol = `git push origin <branch>`, doc language matched from existing docs, key pages from heuristic route detection.

If a project does have a `## Night Shift Config` section, those settings override the defaults.

## Consequences

- **Pro:** Adding a project to Night Shift is one step (edit trigger sources). No per-repo prep required.
- **Pro:** Greenfield projects without `CLAUDE.md` get value immediately.
- **Pro:** Pause is explicit and visible (a file you can `git blame`).
- **Con:** Slightly higher risk that an unsuitable project gets touched. Mitigated by per-task self-skip logic — every task has internal "exit silently if nothing applies" rules so unsuitable projects produce no commits.
- **Con:** Easy to forget that a new repo added to `sources[]` will get changes on the next run. Acceptable trade — speed over caution.
