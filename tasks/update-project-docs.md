# Project Docs

Discover and refresh all project documentation files that are not covered by other docs-bundle tasks (changelog, user manual, ADRs, suggestions).

## Read project config first
Read `CLAUDE.md` for **Night Shift Config**: doc language, push protocol. If the dispatcher passed `allowed_tasks` and `update-project-docs` is not in it, exit silently.

**Scoping.** This task is `scope: repo` in `manifest.yml`. Project-level docs describe the whole repo, so even in a monorepo with `apps:` configured, this task runs **once per repo**, not once per app. Ignore any `app_path` passed by the multi-runner.

## Steps

### 1. Discover docs
Scan the repo root and `docs/` directory (one level deep) for Markdown files that look like project documentation. Common examples:

- README.md, CONTRIBUTING.md
- CONCEPT.md, TECH.md, ARCHITECTURE.md
- MODELS.md, ECOSYSTEM.md, GLOSSARY.md
- PRODUCTION-READINESS.md, DEPLOYMENT.md
- Any other .md file whose content describes the project, its tech stack, data model, architecture, concepts, ecosystem, or operational aspects

**Exclude** files managed by other night-shift tasks or not meant for refresh:
- CHANGELOG.md, docs/CHANGELOG.md (managed by `update-changelog`)
- docs/USER-MANUAL.md (managed by `update-user-guide`)
- docs/adr/*.md (managed by `document-decisions`)
- docs/SUGGESTIONS.md (managed by `suggest-improvements`)
- CLAUDE.md, LICENSE*, .github/**

Also exclude any `*-PLAN.md` files — those are roadmap documents managed by the plans bundle.

### 2. Check each doc for staleness
For each discovered doc:

1. Read the file to understand what it documents.
2. Check whether the **code it describes** has changed since the doc's last update:
   - Use `git log -1 --format=%ci -- <doc-path>` to find when the doc was last touched.
   - Identify the source files the doc covers (routes, models, configs, etc.).
   - Use `git log --since=<doc-last-update> --oneline -- <relevant-paths>` to see if anything changed.
3. If the source code hasn't changed, skip the doc.

### 3. Update stale docs
For each stale doc:

1. Re-read the current source code the doc covers.
2. Update sections that are out of date: new features, renamed concepts, changed architecture, removed components, new dependencies.
3. Preserve the doc's existing structure, tone, and language. Do not rewrite sections that are still accurate.
4. If a doc references version numbers, dates, or metrics, update them only if you can verify the new values from the code or git history.

**Do not create new docs.** Only update files that already exist. If the project has no docs beyond what other tasks manage, exit silently.

**Cap at 3 doc files per run** to keep diffs reviewable.

## Commit
```
git add <updated-doc-paths>
git commit -m "nightshift(docs): refresh <doc-1>, <doc-2>, ..."
```
Push using the project's push protocol.

## Idempotency
- Skip docs whose source code has not changed since the doc's last update.
- Never remove sections — only update or add.
- If nothing is stale, exit silently.
