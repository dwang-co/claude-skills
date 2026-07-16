# Architecture Review Rules

## Purpose
Run this review before starting any new feature or making structural changes. The goal is to catch irreversible mistakes early, protect core files, surface UX regressions, and lock scope before any code is written.

Invoke explicitly: referenced during initial planning before any non-trivial feature starts.

---

## Step 0: Pre-flight Check

Before producing the architecture brief, verify two things:

1. **Branch**: Are we on a feature branch (`feat/*` or `fix/*`), not `main`? If on `main`: stop. Create the correct branch first before proceeding.
2. **Uncommitted work**: Is there uncommitted work in the working directory that could be lost or confused with the new change? If yes: surface it and ask whether to commit, stash, or continue before proceeding.

Do not advance to Step 1 until both conditions are clear.

---

## Step 1: Produce the Architecture Brief

Before writing any code, output a brief in this exact format:

**What this changes**
- Files being CREATED: [list, or "none"]
- Files being MODIFIED: [list, or "none"]
- Files being DELETED or MOVED: [list, or "none" — if any, flag immediately]
- Database changes: [schema additions, migrations, RLS rules — or "none"]
- API surface changes: [new routes, modified routes, removed routes — or "none"]
- User flows affected: [list every user-facing flow this touches — auth, onboarding, primary actions — or "none"]

**Supabase column verification** (required when any Supabase query is added or modified):
Before writing any `.from('table').select(...)`, `.insert(...)`, `.update(...)`, or `.eq('column', ...)` call, grep `supabase/migrations/*.sql` and confirm every column name used in code exactly matches what's in the migration. Mismatches cause silent runtime failures — type checking does not catch these.
- List each table and the columns you will reference: [table → columns]
- Confirm each column exists in the migration SQL: [verified / ❌ mismatch — fix first]

**What stays the same**
Explicitly name the specific features, pages, and files that are NOT being touched by this change. This is a scope lock — if implementation requires touching something listed here, stop and re-run this review.

**Risk tier**: [SAFE / CAUTION / HIGH RISK / DESTRUCTIVE]
**Reversible**: [Yes / No / Partially — one sentence explaining why]

End the brief with one of:
- "SAFE to proceed — starting implementation."
- "CAUTION: [specific risk]. Waiting for acknowledgment before continuing." *(for acknowledgment-required items)*
- "CAUTION: [specific risk]. Proceeding unless you redirect." *(for flag-only items)*
- "BLOCKED: [specific risk and what could go wrong]. Type 'confirmed' to proceed."
- "HARD STOP: [description of what will be permanently lost]. This cannot be undone. Type 'I confirm' to proceed."

---

## Step 2: Classify Risk

### SAFE — proceed without approval
- Creating new files: components, pages, utilities, types, hooks
- Adding new npm packages (state what it does, why it's needed, alternatives considered)
- Styling changes: Tailwind classes, layout adjustments, visual tweaks
- Adding new Supabase tables (RLS must be enabled — flag if it isn't)
- Adding new API routes that don't modify existing behavior
- Writing or updating tests

### CAUTION — two sub-tiers

**Requires acknowledgment** — flag the risk, then wait before proceeding:
- Changing the props interface, routing behavior, or data-fetching pattern of an existing component
- Changing the response shape of an existing API route
- Moving or renaming any file where an importer is in a protected category or critical user flow (see Step 3 — check the import chain, not just the count)
- Adding or changing environment variable keys in `.env.example`

**Flag and proceed** — state the risk, then continue unless redirected:
- Adding columns to an existing Supabase table
- Adding new optional props to existing components without changing existing ones
- General styling or layout changes to existing components that don't remove functionality

### HIGH RISK — block and require explicit approval
Output: "BLOCKED: [explanation of what could go wrong and why it's hard to reverse]. Type 'confirmed' to proceed."

Triggers:
- Deleting any file
- Modifying existing Supabase table schemas beyond adding columns
- Any database migration that touches existing rows or constraints
- Changing authentication flow, session handling, or `middleware.ts`
- Modifying root layout (`app/layout.tsx`) or `next.config.js`
- Moving or renaming files where any importer is in a protected category or is part of a critical user flow
- Removing or changing existing user-facing features or flows
- Changing Supabase RLS policies on existing tables

### DESTRUCTIVE — hard stop, always require explicit approval
Output: "HARD STOP: [description of what will be permanently lost]. This cannot be undone. Type 'I confirm' to proceed."

Triggers:
- Dropping database tables or columns
- Deleting migration files
- Removing RLS from any Supabase table
- Any `rm -rf` or bulk file deletion command
- Resetting or overwriting git history
- Deleting files in protected categories (see Step 3)

---

## Step 3: Protected File Categories

Before modifying or deleting any file, check if it belongs to a protected category. If it does, classify as HIGH RISK or DESTRUCTIVE regardless of how minor the change appears.

**Database**
`supabase/migrations/**`, `supabase/schema.sql`, any seed files

**Auth**
`middleware.ts`, files in `lib/auth/`, `utils/auth/`, Supabase client config files

**Root config**
`app/layout.tsx`, `next.config.js`, `package.json`, `tsconfig.json`, `tailwind.config.ts`

**Environment and secrets**
`.env`, `.env.local`, `.env.example`, any file containing API keys, tokens, or secrets

**Git**
`.gitignore`, `.gitmodules`

When touching a protected file, always state explicitly:
"This is a protected file. Here is exactly what I am changing and why: [explanation]."

---

## Step 4: UX and Testability Check

Before finalizing the architecture brief, answer these five questions:

1. Does this change affect a user-facing flow? (auth, onboarding, primary product action)
2. Does this remove or degrade loading, error, or empty states?
3. Does this affect the mobile layout at 375px?
4. Does this remove functionality the user currently has?
5. Does this design make the feature testable? If no, flag what makes it structurally hard to test.

If yes to questions 1–4: flag it explicitly in the brief.
If it removes existing functionality: escalate to HIGH RISK minimum, regardless of initial classification.
If the answer to question 5 is no: flag the testability concern in the brief before proceeding.

---

## Step 5: Scope Creep Check (During Implementation)

The architecture brief is a scope lock. If during implementation the actual change list grows beyond what was stated in the brief, stop and re-run this review before continuing.

Trigger a scope check when any of the following occur:
- A file not listed in the brief needs to be modified
- A protected file is unexpectedly affected
- A user flow not listed in "user flows affected" turns out to be impacted
- The total files being created or modified exceeds the brief by 3 or more

When triggered, output:
"SCOPE CHANGE DETECTED: [what changed vs. the original brief]. Re-running architecture review before continuing."

---

## Step 6: Recovery Protocol

If a HIGH RISK or DESTRUCTIVE action fails or produces unexpected results mid-execution:

1. Stop immediately. Do not take further action to "fix" it.
2. Run `git status` and report the current state of the working directory.
3. Do not run additional commands until the state is fully understood.
4. Report: what was attempted, what happened, what state the system is in now.
5. Wait for explicit instruction before proceeding.

Compounding a failed irreversible action with more actions is the primary cause of unrecoverable data loss.

---

## Guiding Principles

- Default to the more cautious risk tier when classification is unclear.
- Never skip this review because a change "seems small." Small changes to protected files or core flows can have large blast radius.
