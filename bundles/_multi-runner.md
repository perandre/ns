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

## The loop — one isolated subagent per repo

**Critical:** Each repo must be processed in its own subagent (via the `Task` tool) so the main wrapper's context window does not accumulate state across repos. The main wrapper only stores a single one-line result per repo.

For each discovered repo, in directory-name order:

1. From the main wrapper, `cd` into the repo just long enough to:
   - Run `git status --porcelain` — if dirty, record `dirty-skip` and skip to the next repo.
   - Check for opt-out signals. Record `opted-out` and skip if **any** of these are true:
     - A file `.nightshift-skip` exists at the repo root.
     - `CLAUDE.md`, `AGENTS.md`, or `README.md` contains a line `Night Shift: skip`.
   - `cd` back to the parent directory.
2. Otherwise, dispatch a **subagent** via the `Task` tool with a self-contained prompt that:
   - Tells the subagent its working directory (the repo's absolute path).
   - Gives it the URL of the inner bundle to fetch and execute.
   - Instructs it to perform all of the inner bundle's work (read CLAUDE.md if present, run tasks, commit, push).
   - Asks it to return **one single line** summarizing what it did, in the format: `<status> | <terse note>` where status ∈ {`ok`, `failed`}.
3. Capture only that one-line result. **Do not** read or echo any of the subagent's intermediate work into the main wrapper's context.
4. Move on to the next repo. The previous repo's details are now isolated to the finished subagent and do not pollute the next iteration.

If a subagent throws an unrecoverable error, record `failed | <one-line reason>` and continue. Never abort the multi-repo run.

## Defaults when no config exists

If `CLAUDE.md` is missing or has no `## Night Shift Config` section, fall back to:

| Setting | Default |
|---|---|
| Test command | First of: `npm test`, `pnpm test`, `yarn test`, `bun test`, `cargo test`, `pytest`, `go test ./...` — based on lockfile / project files. If none detect, tasks needing tests self-skip. |
| Build command | First of: `npm run build`, `pnpm build`, `yarn build`, `bun run build`, `cargo build`, `go build ./...`. If none, tasks needing build self-skip. |
| Push protocol | `git push origin <branch>` |
| Default branch | Read from `git symbolic-ref refs/remotes/origin/HEAD` |
| Doc language | Match existing docs in `docs/` or `README.md`; fall back to English |
| Key pages | Heuristic: top-level routes in the framework's pages/app directory |
| Task subset | All tasks; each one self-skips when not applicable |

A project with explicit Night Shift Config in `CLAUDE.md` always overrides these defaults.

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
