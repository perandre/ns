# Night Shift

![Night Shift](night-shift.png)

While you sleep, an AI agent runs maintenance jobs across your repositories. You wake up to commits and PRs, and a one-line summary in each project's history file.

## Get started

Open Claude Code in any project and paste this:

```
Set up Night Shift on my account. It's a scheduled-job framework (https://github.com/perandre/night-shift) that creates three nightly triggers running maintenance tasks across my repos. Read SETUP.md from that repo as the configuration reference, then walk me through interactively: ask which GitHub repos to include, confirm the schedule, and confirm with me before creating each scheduled trigger.
```

Claude will read the runbook, ask you which repositories to manage, confirm the schedule, and create three nightly scheduled jobs in your account. Two minutes, no manual config.

Want to try one bundle locally before scheduling anything? Paste this in any project instead. It runs all four bundles against the current repo and asks you to confirm before each commit:

```
Run the Night Shift bundles (https://github.com/perandre/night-shift) against this repo as a one-shot test. Read bundles/all.md from that repo for the bundle order and rules, then walk me through it — ask before committing or pushing anything.
```

## What you'll find in your repo tomorrow morning

- **Planned features partly built** — picks the next phase from a `docs/*-PLAN.md` file and opens a PR
- **Updated docs** — changelog entries, user guide pages, decision records
- **New test coverage** — fills coverage gaps following your existing test patterns
- **Fixed accessibility issues** — WCAG AA violations on key pages
- **Translated UI strings** — moves hardcoded text into your i18n system
- **Audit PRs** — security, bug, SEO, and performance issues, each as its own PR

Each affected repo gets a one-line entry in `docs/NIGHTSHIFT-HISTORY.md`.

## The bundles

| Bundle | What it does | Mode |
|---|---|---|
| **plans** | Implements the next phase of a planning document | One PR per plan |
| **docs** | Updates changelog, user guide, decision records, suggestions | Direct to main |
| **code-fixes** | Adds tests, fixes accessibility, completes translations | Direct to main |
| **audits** | Finds security / bug / SEO / performance issues | One PR per area |

`manifest.yml` is the single source of truth for what tasks exist, what they do, what bundle they belong to, and what order they run in. Edit one file to add, rename, reorder, or move tasks.

## Stopping Night Shift on a project

Add **either**:
- An empty file `.nightshift-skip` at the repo root
- A line `Night Shift: skip` in `CLAUDE.md`, `AGENTS.md`, or `README.md`

The wrapper reports `opted-out` for that repo and moves on.

## Customising per project

Add a `## Night Shift Config` section to the project's `CLAUDE.md`. All fields are optional — anything unset uses sensible defaults autodetected from your `package.json` and repo layout.

```markdown
## Night Shift Config
- Tasks: build-planned-features, update-changelog, add-tests, find-security-issues
- Doc language: Norwegian (nb)
- Test command: npm test
- Build command: npm run build
- Push: git push mirror main && git push origin main
- Key pages: /dashboard, /surveys, /people
```

## How to add a project, add a task, or run something now

See **[HOW-TO.md](HOW-TO.md)** — five copy-paste recipes covering the common operations.
