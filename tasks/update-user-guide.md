# Documentation

Discover and refresh all project documentation files — user guides, READMEs, technical docs, concept docs, model descriptions, ecosystem overviews, production-readiness checklists, and any other Markdown docs in the repo.

## Read project config first
Read `CLAUDE.md` for **Night Shift Config**: doc language, key pages, push protocol. If the dispatcher passed `allowed_tasks` and `update-user-guide` is not in it, exit silently.

**Scoping.** If the dispatching multi-runner passes an `app_path` (non-empty, not `—`):
- Look for docs both at `<app_path>/` and at the repo root.
- For user-facing docs (user manual, README), prefer per-app versions at `<app_path>/docs/` or `<app_path>/`. Create a per-app user manual at `<app_path>/docs/USER-MANUAL.md` if missing and the app has UI routes.
- For repo-wide docs (CONCEPT, TECH, MODELS, ECOSYSTEM, PRODUCTION-READINESS, ARCHITECTURE, etc.), update them in place — don't duplicate into app directories.
- Scope UI route walks to `<app_path>`. Read `key pages` from the scoped config.
- Commit message names the app when changes are app-scoped.

Without an `app_path`, behave as a single-app repo.

## Steps

### 1. Discover documentation files
Scan the repo root, `docs/` directory, and (when scoped) `<app_path>/` and `<app_path>/docs/` for Markdown files that are project documentation. This includes but is not limited to:

- **User-facing:** USER-MANUAL.md, README.md, GETTING-STARTED.md, CONTRIBUTING.md
- **Technical:** TECH.md, ARCHITECTURE.md, MODELS.md, API.md
- **Conceptual:** CONCEPT.md, GLOSSARY.md, ECOSYSTEM.md
- **Operational:** PRODUCTION-READINESS.md, DEPLOYMENT.md, RUNBOOK.md

Discover whatever exists — don't limit to this list.

**Exclude** files managed by other night-shift tasks or not meant for refresh:
- CHANGELOG.md, docs/CHANGELOG.md (managed by `update-changelog`)
- docs/adr/*.md (managed by `document-decisions`)
- docs/SUGGESTIONS.md (managed by `suggest-improvements`)
- CLAUDE.md, LICENSE*, .github/**, *-PLAN.md

### 2. Check each doc for staleness
For each discovered doc:

1. Read the file to understand what it documents.
2. Identify the source files it covers (routes, models, configs, components, etc.).
3. Compare the doc's last update (`git log -1 --format=%ci -- <doc-path>`) against changes in those source files (`git log --since=<doc-last-update> --oneline -- <relevant-paths>`).
4. If the source code hasn't changed, skip the doc.

### 3. Update stale docs
For each stale doc:

1. Re-read the current source code the doc covers.
2. Update sections that are out of date: new features, renamed concepts, changed architecture, removed components, new dependencies, new routes.
3. Preserve the doc's existing structure, tone, and language. Do not rewrite sections that are still accurate.
4. Write in the configured **doc language**.

For **user manuals** specifically, also walk UI routes and key components to document:
- What the user sees
- What actions are available
- What happens on each action (in user terms, not implementation terms)

**Do not create new doc files** except for the user manual (`USER-MANUAL.md`) when the app has UI routes and no manual exists yet.

**Cap at 5 doc files per run** to keep diffs reviewable.

## Commit
```
# scoped:
git commit -m "nightshift(docs): <app_path> — refresh documentation"
# unscoped:
git commit -m "nightshift(docs): refresh documentation"
```
Push using the project's push protocol.

## Idempotency
- Skip docs whose source code has not changed since the doc's last update.
- Never remove sections — only update or add content.
- If nothing is stale, exit silently.
