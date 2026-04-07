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

~~Manifest parsing is implicit~~ **Fixed.** Bundle prompts now fetch and parse `manifest.yml` at runtime to resolve their task list. Editing the manifest is enough — the bundle file does not need updating.
