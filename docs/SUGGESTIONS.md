# Suggestions

Improvement ideas for the Night Shift project itself. Maintained by the `suggest-improvements` task.

## DEV.md is referenced but doesn't exist

- **Why:** README.md and earlier conversations talk about a `DEV.md` for plumbing details (versioning, raw URLs, environment IDs). The file was never created. Anyone clicking through expecting it will hit a 404.
- **Effort:** S
- **Files:** create `DEV.md` (or remove the references). Suggest removing references — the README is short enough that plumbing details can live there or in `HOW-TO.md`.

## `bundles/all.md` doesn't write to the central runs log

- **Why:** The multi-* wrappers write to `runs/YYYY-MM.md` in the night-shift repo via the cloned `night-shift` source. `bundles/all.md` runs in a single repo (no multi-repo wrapper, no night-shift clone) so the run log entry never happens. The history is only in the target repo's `docs/NIGHTSHIFT-HISTORY.md`.
- **Effort:** S
- **Files:** `bundles/all.md` — add a note that the central run log is multi-* only, OR add an instruction to clone/push to the central log if the user has access. The note is the safer default.

## Manifest parsing is implicit

- **Why:** `manifest.yml` is the declared source of truth for tasks and bundles, but the bundle prompts (`bundles/<name>.md`) hardcode their task list as URLs. Editing the manifest without also editing the bundle file means the manifest and reality drift apart silently.
- **Effort:** M
- **Files:** Either (a) make bundle prompts fetch and parse the manifest at runtime (more dynamic, slightly more agent work), or (b) add a note to `HOW-TO.md` reminding maintainers to edit both the manifest and the bundle file. Option (b) is the cheaper start.
