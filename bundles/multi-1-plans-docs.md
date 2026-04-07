# Multi-repo Bundle 1 — Plans & Docs

You are running the Night Shift **Plans & Docs** bundle across **all repositories** cloned into this session.

## Discover repos
List sibling directories from your starting working directory. For each candidate, run `git rev-parse --show-toplevel` to confirm it's a git repository. Build the list of valid repos before starting the loop.

## Per-repo loop — isolated subagent per repo

**Context isolation requirement:** the main wrapper must NOT process bundle work itself. For each repo, dispatch a subagent via the `Task` tool. The main wrapper only stores the subagent's one-line result, never its intermediate output.

For each discovered repo, in directory-name order:

1. From the main wrapper, briefly `cd` into the repo to:
   - Run `git status --porcelain` — if dirty, record `dirty-skip` and continue to the next repo.
   - Check opt-out signals. Record `opted-out` and continue if any of: `.nightshift-skip` exists at the repo root, or `CLAUDE.md` / `AGENTS.md` / `README.md` contains the line `Night Shift: skip`.
   - Capture the absolute repo path. `cd` back to the parent.
2. Dispatch a `Task` subagent with this prompt (substitute `{REPO_PATH}` with the absolute path):

   ```
   Your working directory is {REPO_PATH}. cd into it now.

   Fetch https://raw.githubusercontent.com/perandre/night-shift/v4/bundles/1-plans-docs.md
   and execute it against this repository.

   CLAUDE.md is optional. If it has a `## Night Shift Config` section, follow those overrides.
   Otherwise apply the defaults from
   https://raw.githubusercontent.com/perandre/night-shift/v4/bundles/_multi-runner.md
   (autodetect test/build commands, plain `git push origin <branch>`, etc).

   When you are done, return EXACTLY ONE LINE in this format:
   <ok|failed> | <terse note, max 80 chars>
   ```
3. Capture only that one-line result. Do **not** echo or summarize the subagent's work into your own context.
4. Move on to the next repo. The previous subagent's context is discarded.

If a subagent dispatch itself fails (not the inner work — the dispatch), record `failed | dispatch error: <reason>` and continue.

## Final report
After processing all repos, print a single summary table and stop. No prose after the table.

```
Night Shift bundle 1 (Plans & Docs) — multi-repo summary

| Repo | Status | Notes |
|------|--------|-------|
| ...  | ok / skipped / dirty-skip / opted-out / failed | <terse> |
```
