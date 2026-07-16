---
description: Run the full Definition of Done sequence after completing a task. Chains testing review (test coverage, DoD commands, regression check, UI verification) and design QA for any UI changes. Nothing is done until this passes.
disable-model-invocation: false
allowed-tools: Bash Read
---

# Feature Done Review

You are running the Definition of Done sequence. A task is not done because the code is written. Run every step in order. Fix all failures before proceeding to the next.

## Part 1: Testing Review

Read the full testing rules:

~/.claude/rules/testing.md

Follow all six steps as written:

1. **Test Writing Requirement** (Step 0) — Was new logic written? If yes, tests are required before proceeding. Not optional.

2. **Write Tests** (Step 1) — Happy path, error case, and edge cases for all new or modified logic. Tests must be passing before moving on.

3. **DoD Commands** (Step 2) — Run in this exact order. Fix all failures before the next command:
   ```
   npm run typecheck
   npm run test
   npm run lint
   npm run build
   ```
   Never fix a failing test by deleting or skipping it. Never use --no-verify.

4. **Regression Check** (Step 3) — Were any previously passing tests broken? Were any tests deleted? Does the test count match expectations?

5. **UI Verification** (Step 4) — Start the dev server. Walk the golden path at 375px and 1280px. Trigger an error state. Verify empty state. Verify loading state. If a bug is found, stop and fix before continuing.

6. **Three-State Check** (Step 5) — For every new user-facing interaction: loading, error, and empty states must all exist.

## Part 2: Security Spot-Check

Read the relevant sections of:

~/.claude/rules/security.md

Check only what applies to this task — skip any item that wasn't touched:

- **Secrets** (always check) — No API keys, tokens, or credentials hardcoded in new or modified files. `.env` and `.env.local` are in `.gitignore`. No secrets in `NEXT_PUBLIC_*` variables.
- **Auth** (if new routes or Server Actions were added) — Every new protected route and Server Action has an auth check inside it. Server Actions are not protected by middleware alone — verify each one independently.
- **RLS** (if new Supabase tables were added) — RLS is enabled on every new table. Every new table has explicit policies scoped to `auth.uid()`.
- **Input validation** (if new API routes, forms, or webhook handlers were added) — Every new API route validates its request body with Zod. Every new form is validated server-side, not just client-side.
- **Mass assignment** (if new write operations were added) — Accepted fields are explicitly whitelisted. Privileged fields (`role`, `is_admin`, `subscription_tier`) cannot be set by users through any route.

If any blocker is found: stop. Do not proceed to design QA or commit. Fix the issue and re-run from Part 1.

Add a one-line security status to the sign-off:
**Security spot-check**: [pass / blocker found — describe / not applicable — explain]

## Part 3: Design QA (UI changes only)

If this task included any user-facing UI changes, continue to the design QA review. Skip if the task was purely backend, data, or configuration. When skipping, note it in the sign-off.

Read the full design QA rules:

~/.claude/rules/design-qa.md

Follow all eight steps as written, paying particular attention to:
- Responsiveness at 375px and 1280px (Step 1)
- Hierarchy and intent — the two-second test (Step 2)
- Interaction states for all new interactions (Step 7)
- Forms and keyboard navigation (Step 7)

## Output: Coverage Summary

End with the sign-off from testing.md Step 6, plus the two additions below:

**Tests written this task**: [list]
**Tests updated this task**: [list or "none"]
**Untested modified files**: [list or "none"]
**DoD commands**: [all passing / list failures]
**UI verified at 375px**: [yes / not applicable — explain]
**UI verified at 1280px**: [yes / not applicable — explain]
**Three states verified**: [yes / not applicable — explain]
**Regressions found**: [none / describe]
**Security spot-check**: [pass / blocker found — describe / not applicable — explain]

If UI changes were included, append the design sign-off from design-qa.md Step 8.

Mark done only when every line is green or explicitly accounted for. A blocker in the security spot-check is a hard stop — do not commit until resolved.

Mark done only when every line is green or explicitly accounted for.
