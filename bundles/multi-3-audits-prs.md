# Multi-repo Bundle 3 — Audits & PRs

You are running the Night Shift **Audits & PRs** bundle across **all repositories** cloned into this session.

## Discover repos
List sibling directories from your starting working directory. For each candidate, run `git rev-parse --show-toplevel` to confirm it's a git repository. Build the list of valid repos before starting the loop.

## Per-repo loop
For each repo, in directory-name order:

1. `cd` into the repo.
2. `git status --porcelain` — if the tree is dirty, log `dirty-skip` and continue with the next repo.
3. Look for a `## Night Shift Config` section in `CLAUDE.md` (fallback: `AGENTS.md`). If neither exists, log `no-config-skip` and continue.
4. Fetch and execute the per-repo bundle:
   **https://raw.githubusercontent.com/perandre/night-shift/v3/bundles/3-audits-prs.md**
   That bundle creates one branch + one PR per audit task and continues internally on per-task exits.
5. Catch any unrecoverable error. Record `failed` for this repo and a one-line reason. Continue with the next repo.
6. Before moving on: `git checkout <default-branch>` to leave the repo in a clean state. Then `cd` back to the parent directory.

## Final report
After processing all repos, print a single summary table including the URLs of any PRs opened. No prose after the table.

```
Night Shift bundle 3 (Audits & PRs) — multi-repo summary

| Repo | Status | PRs opened | Notes |
|------|--------|-----------|-------|
| ...  | ok / skipped / dirty-skip / no-config-skip / failed | <urls or —> | <terse> |
```
