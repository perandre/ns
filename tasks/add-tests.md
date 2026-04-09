# Tests

Find coverage gaps and add tests following the project's existing patterns. **One PR with up to 10 new tests.**

## Read project config first
Read `CLAUDE.md` for **Night Shift Config**: test command, build command, default branch, push protocol. If the dispatcher passed `allowed_tasks` and `add-tests` is not in it, exit silently.

**Scoping.** If the dispatching multi-runner passes an `app_path` (non-empty, not `—`), operate inside that app only:
- Only walk files under `<app_path>` when hunting for coverage gaps.
- Use the app-scoped test and build commands from the scoped config.
- Branch: `nightshift/tests-<app-slug>-YYYY-MM-DD`.
- PR title: `nightshift/tests: <app_path> — add coverage for <N> units`.

Without an `app_path` (single-app repo), behave as before: walk the whole repo, use the top-level test/build commands.

## Steps
1. Check for an existing open night-shift tests PR for this app (or repo when unscoped):
   ```
   gh pr list --search "nightshift/tests in:title" --state open
   ```
   If one exists for the same app, exit silently — do not stack PRs.
2. Run the scoped test command once to confirm a green baseline. If it fails, **exit immediately** — do not try to fix unrelated breakage tonight.
3. Inspect the existing test layout **under `<app_path>`** (or repo-wide when unscoped). Mimic the project's chosen framework, file location convention, and assertion style exactly.
4. Identify **up to 10** untested or under-tested units (module / route / component / utility) **inside `<app_path>`** where tests would be high-value and low-risk to write. Prefer pure logic over UI / network code. Stop at 10 even if more gaps exist.
5. Write tests for each identified unit. Follow project conventions for mocks, fixtures, and naming. Do not touch files outside `<app_path>` when scoped.
6. Run the scoped **test suite** and the scoped **build command** after each test file. If a test is flaky or fails, revert that test file and continue with the next unit. Do not commit failing tests.
7. Collect all passing tests in one branch (include app slug when scoped):
   ```
   # scoped:
   git checkout -b nightshift/tests-<app-slug>-YYYY-MM-DD
   # unscoped:
   git checkout -b nightshift/tests-YYYY-MM-DD
   ```
8. Push and open the PR (prefix title with `<app_path> — ` when scoped):
   ```
   gh pr create --title "nightshift/tests: <app_path> — add coverage for <N> units" \
     --body "$(cat <<'EOF'
   ## Summary
   Found coverage gaps and added tests for <N> units.

   ## Tests added
   - <bullet per test file, naming the unit under test>

   ## Verification
   - All tests pass locally
   EOF
   )"
   ```

## Idempotency
- One sweep PR open at a time.
- Do not modify production code in this task. If a test reveals a bug, leave a note in `docs/SUGGESTIONS.md` and stop.
- If no meaningful coverage gaps remain on the configured key pages, exit silently.
