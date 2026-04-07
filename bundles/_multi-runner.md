# Multi-repo runner — shared protocol

This file documents the loop semantics used by `multi-1-*`, `multi-2-*`, and `multi-3-*`. It is not fetched by triggers directly — read it for reference.

## How the trigger lays out repos

When a trigger declares multiple `sources[]` entries with `git_repository`, the remote environment clones each one as a sibling directory at the top of the working tree. After cloning, the working directory looks like:

```
/workspace/        ← or wherever the runner places things
├── repo-a/        ← .git inside
├── repo-b/
└── repo-c/
```

Do not assume a fixed parent path. Discover dynamically:

```bash
# from the runner's starting directory:
ls -1 -d */ 2>/dev/null
# then for each candidate:
( cd "$dir" && git rev-parse --show-toplevel 2>/dev/null )
```

A directory is a repo if `git rev-parse --show-toplevel` succeeds inside it.

## The loop

For each discovered repo, in directory-name order:

1. `cd` into the repo.
2. `git status --porcelain` — confirm the tree is clean. If dirty, log `dirty-skip` and continue.
3. Look for `## Night Shift Config` in `CLAUDE.md` (or `AGENTS.md` as fallback). If neither file exists, log `no-config-skip` and continue. (Tasks still have safe defaults, but a project that hasn't been onboarded should be skipped to avoid surprise commits.)
4. Run the inner bundle (the `multi-N-*.md` file specifies which one). Treat its own "exit silently" rules as success.
5. Catch any uncaught failure from the inner bundle. Record it. **Do not abort the multi-repo run.**
6. `cd` back to the parent directory before starting the next repo.

## Final report

After all repos are processed, print one table:

```
Night Shift bundle <N> — multi-repo summary

| Repo                  | Status   | Notes                              |
|-----------------------|----------|------------------------------------|
| friskgarden-kartlegg. | ok       | 2 commits pushed                   |
| brain                 | skipped  | no Night Shift Config              |
| other-project         | failed   | test command exited 1 in task 05   |
```

Status values: `ok`, `skipped`, `failed`. Keep notes terse — this is for the morning review. No further prose after the table.
