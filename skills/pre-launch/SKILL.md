---
description: Run the complete pre-launch review before any production deploy. Chains security, legal, SEO, and performance reviews in sequence. All blockers across all four reviews must be resolved before launch. Beta counts as launch.
disable-model-invocation: false
allowed-tools: Bash Read
---

# Pre-Launch Review

You are running the pre-launch gate. Before doing anything else, answer one question:

**Is this a first launch or a subsequent deploy?**

- **First launch** — the product is going public for the first time, or entering any form of beta or public testing: run all four reviews in full.
- **Subsequent deploy** — an update to an already-public product: run Reviews 1 and 4 always. Skip Reviews 2 and 3 unless the criteria below are met:
  - Skip **Legal (Review 2)** unless: new data collection was added, AI features were added or changed, payment or subscription logic was changed, or terms/privacy policy need updating.
  - Skip **SEO (Review 3)** unless: new pages were added, URL routes were changed, metadata was modified, or `robots.txt` / `sitemap.xml` logic was touched.
  - When skipping a review, state the reason explicitly in the composite sign-off.

A single blocker in any review that is run stops the deploy.

## Review 1: Security

Read the full security rules:

~/.claude/rules/security.md

Run Step 7 (Deploy Checklist) in full. Also run any steps triggered by changes in this deploy:
- Step 1 (Secrets and Environment) — always run
- Step 2 (Auth) — if auth was touched
- Step 3 (Authorization and RLS) — if data access was touched
- Step 4 (Input Validation) — if any new external input was added
- Step 5 (API Surface) — if any routes were added or modified
- Step 6 (Dependencies) — if any packages were added or updated

Produce the security sign-off from Step 8 before moving to Review 2.

---

## Review 2: Legal

Read the full legal rules:

~/.claude/rules/legal.md

Run Step 8 (Pre-Launch Legal Checklist) in full. Also verify:
- Step 2 (GDPR Lawful Basis) — if EU/UK users are possible
- Step 4 (User Rights) — deletion mechanism functional
- Step 5 (AI-Specific) — if AI features are included
- Step 6 (Breach Response) — confirm a process exists

Produce the legal sign-off from Step 9 before moving to Review 3.

---

## Review 3: SEO

Read the full SEO rules:

~/.claude/rules/seo.md

Run the pre-launch SEO gate in full.

Produce the SEO sign-off before moving to Review 4.

---

## Review 4: Performance

Read the full performance rules:

~/.claude/rules/performance.md

Run the deploy performance checklist in full.

Produce the performance sign-off before the composite output.

---

## Composite Pre-Launch Sign-off

After all reviews are complete, output this summary:

**Deploy type**: [first launch / subsequent deploy]
**Deploy target**: [staging / production]

| Review | Status | Blockers | Critical | Warnings |
|---|---|---|---|---|
| Security | [pass / issues] | [count] | [count] | [count] |
| Legal | [pass / skipped — reason / issues] | [count or —] | [count or —] | [count or —] |
| SEO | [pass / skipped — reason / issues] | [count or —] | [count or —] | [count or —] |
| Performance | [pass / issues] | [count] | [count] | [count] |

**Overall status**: [CLEAR TO LAUNCH / BLOCKED — list all blockers across all reviews run]

**All blockers**: [none / full list — must resolve before launch]
**All critical findings**: [none / full list — must resolve before merge]
**All warnings**: [none / list — schedule before next release]

Do not launch until every blocker across all reviews that were run is resolved. A blocker in any single review blocks the entire deploy.
