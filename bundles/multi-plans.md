# Multi-repo: Plans

You are running the Night Shift **Plans** bundle across **all target repositories** cloned into this session.

## Parse the per-repo allowlist first

Before discovering repos, scan **your own invocation prompt** for a `<night-shift-config>…</night-shift-config>` block and parse the `repos:` map out of it. See `bundles/_multi-runner.md` → **Per-repo task allowlist** for the exact contract, parsing rules, and fallback behavior. Summary:

- If the block is absent or malformed, treat as "no allowlist supplied" and log `allowlist: none (running all tasks)` in the final summary.
- Otherwise, for each repo key, the value is a list of task ids from `manifest.yml` that are allowed for that repo. Unknown ids → warn and ignore. A repo absent from the map → all tasks allowed.

For the **plans** bundle, the only relevant task id is `build-planned-features`. A repo whose allowlist does not include `build-planned-features` must be recorded as `not-selected` and dispatched to no subagents — skip the plan-file discovery entirely for it.

## Discover repos
List sibling directories at the top of your working tree. For each candidate, confirm it is a git repository via `git rev-parse --show-toplevel`.

## Per-work-item loop — isolated subagent per (repo, app, plan)

`build-planned-features` is `scope: app` in `manifest.yml`, so a repo with an `apps:` block fans out to one work-item per app. **On top of that, plans fan out further: one subagent per plan file.** Every pending plan gets its own agent and its own PR — no plan is ever skipped because another plan ran first.

For each discovered target repo, in directory-name order:

1. From the main wrapper, briefly `cd` into the repo to:
   - Look up the repo in the parsed allowlist. If `build-planned-features` is **not** in the repo's allowed list, record `not-selected` and continue (no dispatch for this repo).
   - Run `git status --porcelain` — if dirty, record `dirty-skip` and continue.
   - Check opt-out signals. Record `opted-out` and continue if any of: `.nightshift-skip` exists at the repo root, or `CLAUDE.md` / `AGENTS.md` / `README.md` contains the line `Night Shift: skip`.
   - Parse `## Night Shift Config` in `CLAUDE.md`. If it contains an `apps:` block, build one app-scope per `apps[]` entry (each with its own `app_path` + merged `scoped_config`). Otherwise build a single app-scope with `app_path = —`.
   - **For each app-scope, list plan files.** Resolve `PLANS_DIR`: `<app_path>/<plans dir>` when scoped, else `<plans dir>` (default `docs`). List `$PLANS_DIR/*-PLAN.md`. Skip plans whose front matter / heading marks them **deferred**, **blocked**, or **on hold**. Each surviving plan file becomes its own work-item `{repo, app_path, scoped_config, plan_file}`.
   - If an app-scope has zero plan files, emit one work-item with `plan_file = —` so it can report `silent` in the summary.
   - Capture the absolute repo path. `cd` back to the parent.
2. For each work-item from this repo, dispatch a `Task` subagent with this prompt (substitute `{REPO_PATH}`, `{APP_PATH}` — literal `—` when repo-wide, `{SCOPED_CONFIG}` as inline JSON / YAML, `{PLAN_FILE}` — literal `—` when no plans):

   ```
   Your working directory is {REPO_PATH}. cd into it now.
   App scope: {APP_PATH}          # "—" means repo-wide, single-app mode
   Plan file: {PLAN_FILE}         # "—" means no plans to process; exit silent
   Allowed tasks: [build-planned-features]   # this subagent runs this one task only
   Scoped config: {SCOPED_CONFIG}  # resolved test/build/plans dir/key pages

   If PLAN_FILE is "—", return `silent | PR: — | no plan files` and stop.

   Otherwise, fetch
   https://raw.githubusercontent.com/perandre/night-shift/main/tasks/build-planned-features.md
   and execute it against THIS ONE PLAN FILE ONLY. Do not scan for other plans; the
   dispatcher has already fanned out one subagent per plan. Implement the next
   pending phase of PLAN_FILE and open one PR for it.

   When APP_PATH is not "—":
   - Branch name must include the app slug:
         nightshift/plan-<app-slug>-<plan-slug>-phase-<N>-YYYY-MM-DD
     where <app-slug> is the last segment of APP_PATH (e.g. "web" for "apps/web").
   - PR title must name the app:
         nightshift/plan: <app_path> — <plan-name> phase <N>

   CLAUDE.md is optional. Honor `## Night Shift Config` if present, otherwise apply
   the defaults from
   https://raw.githubusercontent.com/perandre/night-shift/main/bundles/_multi-runner.md.

   At the end of your run, append ONE LINE to docs/NIGHTSHIFT-HISTORY.md (create the
   file if missing) under the `## Runs` heading at the top of the runs list. Format:
       - YYYY-MM-DD plans  <app_path or —>  <plan-slug>  <ok|silent|failed>  <terse note, max 80 chars>
   Then commit + push the history file (alongside other commits or as its own commit).

   Return EXACTLY ONE LINE to me in this format:
       <ok|silent|failed> | PR: <url or —> | <plan-slug> — <terse note, max 60 chars>
   ```
3. Capture only the one-line result. Do not echo subagent work into your own context.
4. Move on to the next work-item. **Never stop early** — every plan must get its own dispatch attempt, even if earlier plans failed.

If a subagent dispatch itself fails, record `failed | PR: — | dispatch error: <reason>`.

## Final report
Print this summary table and stop. The summary table is the primary artifact — it appears in the trigger dashboard and is how the user reviews the run. **Do not** write the summary to any external repo or location; the per-repo `docs/NIGHTSHIFT-HISTORY.md` files in each target repo are the only persisted history.

```
Night Shift plans — multi-repo summary

| Repo | App | Plan | Status | PR | Notes |
|------|-----|------|--------|----|-------|
| ...  | <app_path or —> | <plan-slug or —> | ok / silent / not-selected / opted-out / dirty-skip / failed | <url or —> | <terse> |
```

One row per (repo, app, plan). `App` is `—` for single-app repos. `Plan` is `—` when the app-scope had no plan files (the row will be `silent`). A repo excluded from the allowlist produces one row with `App = —`, `Plan = —`, `Status = not-selected`.

Include any `allowlist: …` or `allowlist warning: …` lines from the parsing step as bullet points beneath the table so the user sees them on the trigger dashboard.
