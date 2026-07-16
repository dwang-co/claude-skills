# Security Review Rules

## Purpose
Run this review before any deploy or any change touching auth, sessions, API routes, database access, secrets, AI features, or user data. The goal is to catch vulnerabilities before they reach production — not after.

Priority order: blockers must be resolved before deploy. Critical findings must be resolved before merge. Warnings must be flagged and triaged.

Invoke explicitly: before any deploy, and before merging any auth, AI, or data-handling change.

---

## Severity Classification

**Blocker — stop deploy, fix immediately:**
- Secret, API key, or credential exposed in code, logs, or client bundle
- Authentication bypass possible — any route or resource accessible without valid session
- Missing or broken Row Level Security on any Supabase table containing user data
- SQL injection, XSS, or command injection vulnerability
- Unvalidated user input passed directly to a database query, shell command, or rendered as HTML
- CORS configured to allow all origins (`*`) on routes that accept credentials
- User A can read or mutate User B's data (broken object-level authorization)
- User input passed directly into an AI prompt without sanitization (prompt injection)
- Server Action missing an auth check inside the action itself

**Critical — must fix before merge:**
- Session token stored in localStorage (use httpOnly cookies)
- Sensitive data (PII, tokens, keys) logged to console or error tracking
- API route missing authentication check on a protected resource
- Missing Zod validation on any external boundary: API route, form submission, webhook payload
- Password or token transmitted over HTTP (not HTTPS) in any environment
- Third-party dependency with a known high-severity CVE introduced in this change
- Rate limiting absent on auth endpoints (login, signup, password reset, OTP)
- No rate limiting or spend cap on AI API endpoints
- Webhook payload accepted without timestamp validation (replay attack risk)
- API route passes user-supplied fields directly to a database write without whitelisting allowed fields

**Warning — flag before done, fix before next deploy:**
- Error messages returning internal system detail to the client (stack traces, DB errors, file paths)
- Missing CSRF protection on state-mutating form actions
- Overly permissive Supabase RLS policy (e.g., `USING (true)` on a table with user data)
- Supabase Storage bucket with a public policy on user-specific content
- OAuth redirect URI not strictly validated
- Dependency version pinning absent — floating `*` or `^` on a security-sensitive package
- Missing `Content-Security-Policy`, `X-Frame-Options`, or `Referrer-Policy` headers in production config
- Admin or internal routes not separated from user-facing routes at the middleware level
- AI prompt includes more user data than the specific request requires (over-provisioned context)

---

## Step 0: Pre-flight Check

Before beginning the review, establish scope:

1. **What changed?** Run `git diff main...HEAD --name-only` and list all files modified in this branch.
2. **Is any protected surface touched?** Protected surfaces: auth flow, middleware, Supabase client config, API routes, Server Actions, AI prompt construction, environment variables, RLS policies, session handling, any file that reads or writes user data.
3. **Is this a deploy review or a change review?** Deploy reviews cover the full surface. Change reviews focus on the delta.

Do not skip Step 0. A review without known scope is not a review.

---

## Step 1: Secrets and Environment Audit

Check every file in the diff and the full project for secret exposure.

**Verify:**
- [ ] No secrets, tokens, API keys, or credentials are hardcoded in any source file
- [ ] `.env` and `.env.local` are in `.gitignore` — confirm with `git check-ignore -v .env`
- [ ] `.env.example` exists and contains only placeholder values (no real credentials)
- [ ] No secret values appear in `console.log`, error tracking calls, or analytics events
- [ ] No secret values are passed to the client bundle via `NEXT_PUBLIC_*` unless they are intentionally public (e.g., a publishable Supabase anon key — but never the service role key or the Anthropic API key)
- [ ] All environment variables used in the code are documented in `.env.example`

If any secret is found in code or git history: **blocker — stop and resolve before proceeding.**

For git history exposure: the secret must be rotated immediately, not just removed from HEAD. Removing from HEAD does not remove it from history.

---

## Step 2: Authentication and Session Review

For any change touching auth, sessions, middleware, protected routes, or Server Actions:

**Authentication:**
- [ ] Every protected route has an authentication check — server-side, not client-side only
- [ ] Middleware (`middleware.ts`) enforces auth on all routes under protected paths
- [ ] Auth checks use the Supabase server client (not the browser client) for server components and API routes
- [ ] No route relies solely on a client-side redirect to protect server-accessible data

**Server Actions:**
- [ ] Every Server Action that accesses protected data verifies the session inside the action itself — not just in the component that calls it
- [ ] Server Actions are not treated as protected by middleware matchers — they are HTTP endpoints callable by anyone who knows the action ID
- [ ] Verify: if the Server Action were called directly via HTTP (bypassing the UI entirely), would it still enforce auth?

**Sessions:**
- [ ] Session tokens are stored in httpOnly cookies — not localStorage or sessionStorage
- [ ] Session expiry and refresh behavior is tested: expired sessions redirect to login, not to an error page
- [ ] Logout invalidates the server-side session, not just the client cookie

**Auth endpoints (login, signup, OTP, password reset):**
- [ ] Rate limiting is in place — confirm the mechanism (Supabase built-in, middleware, or external service)
- [ ] Error messages do not reveal whether an email is registered (user enumeration risk)
- [ ] Password reset tokens are single-use and expire within a short window (15–60 minutes)
- [ ] OAuth callback URLs are on an allowlist — not dynamically constructed from user input

If this change adds a new auth flow or modifies an existing one: walk through the full flow manually and confirm each check above before proceeding.

---

## Step 3: Authorization and Data Access Review

For any change that reads or writes user data:

**Row Level Security:**
- [ ] RLS is enabled on every Supabase table that stores user data — run `SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public';` to verify
- [ ] Every table has explicit RLS policies — no table relies on a missing policy defaulting to deny
- [ ] RLS policies are scoped to `auth.uid()` — not to a role or a static value
- [ ] No policy uses `USING (true)` on a table with user data without documented justification
- [ ] Service role key is never used in client-side code — it bypasses RLS entirely
- [ ] Database functions using `SECURITY DEFINER` are explicitly reviewed — they run with elevated privileges and bypass RLS

**Supabase Storage:**
- [ ] If the app stores files, every Storage bucket has an explicit access policy — no bucket is left with a default public policy on user-specific content
- [ ] Storage policies are scoped to the authenticated user's ID, not just authenticated status
- [ ] Bucket policies are verified against the same checklist as table RLS — a correctly configured table does not protect files stored in a misconfigured bucket

**API-level authorization:**
- [ ] Every API route that returns user data filters by the authenticated user's ID — not just by a parameter the client passes
- [ ] No route accepts a user ID as a parameter and trusts it without verifying it matches the session
- [ ] Admin or privileged operations check for a role claim on the session, not just the presence of a session

---

## Step 4: Input Validation Review

For any change that accepts data from external sources (users, webhooks, third-party APIs):

**Validation coverage:**
- [ ] Every API route handler validates its request body with Zod before processing
- [ ] Every form submission is validated server-side with Zod — not just client-side
- [ ] Webhook payloads are validated: schema check + signature verification + timestamp check before processing
- [ ] Query parameters and URL path params used in database queries are validated and sanitized

**Mass assignment prevention:**
- [ ] API routes that write to the database explicitly whitelist the fields they accept — they do not pass the raw request body to an insert or update
- [ ] Privileged fields (`role`, `is_admin`, `subscription_plan`, `subscription_tier`) cannot be set by users through any API route, regardless of what they include in the request body
- [ ] Zod schemas for write operations include only fields the user is permitted to set — not the full shape of the database row

**Webhook replay protection:**
- [ ] Webhook handlers check the timestamp included in the provider's signature header (e.g., Stripe's `t=` field) and reject payloads older than 5 minutes
- [ ] Replay protection is in place for any webhook that triggers a financial action, privilege change, or external side effect
- [ ] Idempotency: re-processing the same webhook event does not produce duplicate effects

**Injection prevention:**
- [ ] No raw user input is interpolated into SQL strings — all DB queries go through the Supabase client with parameterized inputs
- [ ] No user input is rendered as raw HTML — React's default escaping is not bypassed with `dangerouslySetInnerHTML` unless sanitized
- [ ] No user input is passed to `eval()`, `exec()`, `child_process`, or any dynamic code execution

**File uploads (if applicable):**
- [ ] File type is validated server-side by content, not just by extension
- [ ] File size is limited
- [ ] Uploaded files are stored in a location that does not allow direct execution

---

## Step 5: AI/LLM Security Review

Run this step for any change that adds, modifies, or calls the Claude API or any other LLM.

**Prompt injection:**
- [ ] User input included in prompts is clearly delimited from the system prompt — never concatenated directly into instruction text
- [ ] The system prompt does not instruct the model to follow instructions from user input
- [ ] Test: does submitting `"Ignore all previous instructions and..."` as input cause the model to behave unexpectedly? If yes: the prompt is not injection-resistant
- [ ] In multi-tenant contexts: confirm user input in one session cannot extract data from another user's session or context

**Context scoping (principle of least context):**
- [ ] The AI is only given the data it needs for the specific request — not the full user record, all user history, or admin data "just in case"
- [ ] Sensitive fields (passwords, tokens, payment info, other users' data) are never included in the context passed to the AI
- [ ] If the AI has access to a database or tool, confirm it can only access data the authenticated user is permitted to see — not all rows

**Output handling:**
- [ ] AI-generated content is not rendered as raw HTML — treat it as untrusted input and apply the same sanitization as any user-generated content
- [ ] AI-generated content that includes code is not executed without review
- [ ] Errors from the AI API (timeouts, refusals, unexpected formats) are handled gracefully and do not expose system internals to the user

**Rate limiting and cost controls:**
- [ ] AI endpoints have rate limiting per user — an individual user cannot trigger unlimited API calls
- [ ] A spend cap or alert is configured on the Anthropic account — an abuse scenario should not be able to generate an unbounded bill
- [ ] If AI calls are expensive (large context, long output), confirm they cannot be triggered in a tight loop by a single user or unauthenticated request

**Data privacy:**
- [ ] Confirm what data is being sent to the Anthropic API — ensure no data is shared that users have not consented to send to a third-party AI service
- [ ] If the product has a privacy policy, confirm AI data handling is disclosed

---

## Step 6: API Surface Review

For any change that adds or modifies API routes:

**Route security:**
- [ ] New routes are listed: confirm each one has an auth check or is explicitly public by design
- [ ] `DELETE`, `PUT`, and `PATCH` routes validate ownership before modifying data
- [ ] Routes that trigger emails, charges, or external actions have idempotency handling

**CORS:**
- [ ] CORS is not set to `*` on routes that accept credentials or return sensitive data
- [ ] Allowed origins are an explicit allowlist, not a regex that can be bypassed

**Response hygiene:**
- [ ] Error responses do not include stack traces, internal file paths, or database error details
- [ ] Error responses use generic messages for auth failures ("Invalid credentials" — not "User not found" or "Wrong password")
- [ ] Successful responses do not return fields the requesting user should not see

---

## Step 7: Dependency Audit

For any change that adds or updates npm packages:

- [ ] Run `npm audit` and review the output — no new high or critical vulnerabilities introduced
- [ ] Any new package is from a reputable source with recent maintenance activity
- [ ] No package is installed that requests unusual permissions (filesystem access, network access to unexpected hosts)
- [ ] Dependency versions are pinned at the minor or patch level in `package.json` for security-sensitive packages

If `npm audit` reports high or critical CVEs: research each one. A CVE in a dev-only dependency that never runs in production is different from one in a runtime dependency.

---

## Step 8: Deploy Checklist

Run this step only on deploy reviews, not on per-change reviews.

**Environment:**
- [ ] Production environment variables are set and do not contain placeholder or development values
- [ ] Supabase service role key is set server-side only — not in the client environment
- [ ] Anthropic API key is set server-side only — never in a `NEXT_PUBLIC_` variable
- [ ] `NEXT_PUBLIC_` variables contain no secrets
- [ ] No debug mode, verbose logging, or development tooling is enabled in the production build

**Headers:**
- [ ] `Content-Security-Policy` header is set and restricts script sources to known origins
- [ ] `X-Frame-Options: DENY` or `SAMEORIGIN` is set to prevent clickjacking
- [ ] `Referrer-Policy: strict-origin-when-cross-origin` is set
- [ ] `X-Content-Type-Options: nosniff` is set

**Supabase:**
- [ ] RLS is enabled and verified on every table — run the query from Step 3 against the production schema
- [ ] Storage bucket policies verified against production buckets
- [ ] No migration is being run for the first time in production without having been tested locally
- [ ] Production database is not accessible from the public internet except through Supabase's API layer

**Vercel:**
- [ ] Environment variables are set under the correct environment scope (Production / Preview / Development)
- [ ] Preview deployments do not have access to production secrets — they use a separate Supabase project or branch

---

## Step 9: Security Sign-off

Before marking done or proceeding with deploy, output this summary:

**Review type**: [deploy / pre-merge / auth change / AI change / data-handling change]
**Scope**: [list files and surfaces reviewed]

**Secrets and environment**: [pass / blockers found — list]
**Authentication, sessions, and Server Actions**: [pass / issues — classify]
**Authorization and RLS**: [pass / issues — classify]
**Input validation**: [pass / issues — classify]
**AI/LLM security**: [pass / not applicable — explain / issues — classify]
**API surface**: [pass / issues — classify]
**Dependencies**: [pass / CVEs found — list severity]
**Deploy checklist**: [pass / not applicable — explain / issues — list]

**Blockers**: [none / list — must resolve before deploy]
**Critical findings**: [none / list — must resolve before merge]
**Warnings**: [none / list — flag for triage]

Do not deploy and do not merge until all blockers are resolved. Do not mark a critical finding as "low priority" without documenting the specific reason and getting explicit acknowledgment.

---

## Guiding Principles

- Security is not a phase. It is a property of every decision. Catch it here, not in production.
- When in doubt, default to more restrictive. It is easier to loosen a policy than to recover from a breach.
- Never dismiss a finding because "this is just an MVP" or "we'll fix it later." Security debt compounds faster than technical debt.
- A deploy that exposes user data is not a deploy. It is an incident.
