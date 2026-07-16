# Testing Rules

## Purpose
Run this review after each task, before marking anything done. The goal is to catch regressions, ensure new logic has test coverage, and verify the feature works correctly in the browser before shipping.

Invoke explicitly: after implementation is complete, before marking a task done.

---

## Step 0: Supabase Schema Consistency Check

Before writing any tests or running DoD commands, if this task added or modified any Supabase query:

Run: `grep -n "column_name_you_used" supabase/migrations/*.sql` for every column name your code references.

**TypeScript does not catch Supabase column name mismatches.** A query referencing a non-existent column compiles and passes type checks — it only fails at runtime with a Supabase error. The three most common failure patterns:

1. **Wrong column name**: `session_date` when the schema says `practiced_at`
2. **Missing column**: querying `total_sessions` before it's been added to the schema via a migration
3. **Type enum mismatch**: code uses `'solid'` when schema `CHECK` constraint says `'okay'`

For each Supabase table touched in this task, confirm:
- [ ] Every `.select('col')` column exists in the migration SQL for that table
- [ ] Every `.eq('col', val)` column exists in the migration SQL
- [ ] Every `.insert({ col: val })` key is a real column name
- [ ] Every `.update({ col: val })` key is a real column name
- [ ] Any string value compared against a `CHECK` constraint matches the exact allowed values in the migration

If any mismatch is found: fix the code (or add a migration) before proceeding. Do not move to testing until this is clean.

---

## Step 1: Test Writing Requirement

Before running any checks, ask: was any new logic written in this task?

**New logic requires unit tests. This is not optional.**

New logic includes:
- Any new function, utility, or data transformation
- Any new custom hook
- Any new API route handler or server action
- Any new conditional branching or business rule

Does NOT require new tests:
- Pure styling or Tailwind class changes
- Static text or copy changes
- Configuration file changes with no logic

If existing logic was modified: update the corresponding tests. If the modified code has no existing tests: write them before marking done. Do not leave modified untested code behind.

---

## Step 1: Write Tests

For each piece of new or modified logic, tests must cover:

1. **Happy path** — the expected input produces the expected output
2. **Error case** — at least one failure scenario is handled correctly
3. **Edge cases** — empty input, null values, boundary conditions relevant to the logic

Test file location: co-located with the file under test or in `__tests__/` following the existing project convention.

Tests must be written and passing before moving to Step 2. Do not defer tests to a follow-up task.

**Unit tests vs. integration tests**

Some logic cannot be meaningfully tested in isolation. In a Next.js + Supabase project, the following warrant integration tests rather than (or in addition to) unit tests:
- API route handlers and server actions that query the database
- Authentication flows involving middleware, sessions, or Supabase Auth
- Data transformations that depend on real database response shapes

Integration tests use Vitest with a Supabase dev project. If a dev project is not configured, flag this as a gap before marking done — do not skip and claim unit tests are sufficient.

---

## Step 2: Run DoD Commands in Order

Run these in sequence. Fix all failures before proceeding to the next command. Do not skip ahead.

```
npm run typecheck    # zero TypeScript errors
npm run test         # all tests pass
npm run lint         # zero warnings
npm run build        # production build succeeds
```

**Hard rules:**
- Never fix a failing test by deleting, commenting out, or skipping it. Fix the underlying code.
- Never use `--no-verify` or suppress lint warnings with inline ignores unless the warning is a known false positive — and if so, explain why in a comment.
- If a test fails then passes on re-run, do not accept it as passing. A test that fails intermittently is a flaky test. Isolate the non-determinism (timing, shared state, random data), fix the root cause, and confirm it passes consistently before proceeding.
- If `npm run build` fails but tests pass: the task is not done. The build failure must be resolved.

---

## Step 3: Regression Check

This is a unit-level regression check. Visual regressions and multi-step flow regressions are caught in Step 4, not here.

After the full test suite passes, explicitly verify:

1. Were any tests passing before this task that are now failing? If yes: this is a regression. Fix the root cause before proceeding — do not assume the test was wrong.
2. Were any tests deleted or skipped during this task? If yes: explain why and confirm it was intentional.
3. Does the test count match expectations? A task that adds new logic should add new tests — a net-zero or negative test count change is a signal something was missed.

If a regression is found: stop, identify the root cause, fix it, and re-run from Step 2.

---

## Step 4: Smoke Test

Start the dev server and confirm the change works end to end. This is a correctness check — not a design or UX review. For any UI task, invoke design-qa.md separately for the full visual, hierarchy, accessibility, and interaction review.

- [ ] Dev server starts without errors
- [ ] The primary user flow for this change works end to end
- [ ] No console errors or failed network requests during the flow
- [ ] Use the Supabase dev project — do not smoke test against production

If the dev server cannot be started, report the error. Do not claim success.

If a bug is found: stop, document what action was taken and what was expected, fix it, then re-run from the beginning of this step.

---

## Step 5: Three-State Check

For every new user-facing interaction added in this task, confirm all three states exist:

- [ ] **Loading** — a visible indicator while the action is in progress
- [ ] **Error** — a human-readable message that tells the user what went wrong and what to do next
- [ ] **Empty** — a handled state when there is no data to show

If any state is missing: it must be added before marking done. These are not optional UI polish — they are required per project standards.

---

## Step 6: Coverage Summary

Before marking the task done, output this summary:

**Tests written this task**: [list new test files or test cases added]
**Tests updated this task**: [list modified test cases, or "none"]
**Untested modified files**: [list any files changed without test coverage — flag if any]
**DoD commands**: [all passing / list any failures]
**UI verified at 375px**: [yes / not applicable — explain if not applicable]
**UI verified at 1280px**: [yes / not applicable — explain if not applicable]
**Three states verified (loading / error / empty) for all new interactions**: [yes / not applicable — explain]
**Regressions found**: [none / describe if any were found and how they were resolved]

Mark done only when every line above is green or explicitly accounted for.

---

## Guiding Principles

- A task is not done because the code is written. It is done when the tests pass, the UI works, and nothing that worked before is now broken.
- Never skip tests to ship faster. Untested code is a deferred regression.
- If something cannot be tested (a third-party integration, an environment-specific behavior), say so explicitly — do not silently skip and claim coverage.
