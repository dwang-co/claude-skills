---
description: Run the architecture review before starting any new feature or structural change. Checks branch, produces the architecture brief, classifies risk, and blocks or approves implementation. Invoke before any non-trivial task.
disable-model-invocation: false
allowed-tools: Bash Read Edit Write
---

# Architecture Review

You are running the architecture review gate. Follow every step in order. Do not skip steps because a change "seems small."

## Step 1: Load the Rules

Read the full architecture review rules:

~/.claude/rules/architecture.md

## Step 2: Execute the Review

Follow the rules exactly as written:

1. **Pre-flight Check** (Step 0 in the rules) — Verify branch and uncommitted work state before anything else. Stop if on main.

2. **Architecture Brief** (Step 1) — Produce the brief in the exact format specified: files created, modified, deleted; database changes; API surface changes; user flows affected; what stays the same.

3. **Risk Classification** (Step 2) — Classify as SAFE / CAUTION / HIGH RISK / DESTRUCTIVE using the criteria in the rules. Default to the more cautious tier when unclear.

4. **Protected File Check** (Step 3) — Check every file in scope against the protected categories: database, auth, root config, environment, git. If any protected file is touched, escalate the risk tier.

5. **UX and Testability Check** (Step 4) — Answer all five questions. Flag any yes answers in the brief.

6. **Scope Lock** — End with the correct sign-off phrase based on risk tier:
   - SAFE: "SAFE to proceed — starting implementation."
   - CAUTION (flag-only): "CAUTION: [risk]. Proceeding unless you redirect."
   - CAUTION (acknowledgment): "CAUTION: [risk]. Waiting for acknowledgment before continuing."
   - HIGH RISK: "BLOCKED: [risk]. Type 'confirmed' to proceed."
   - DESTRUCTIVE: "HARD STOP: [description]. This cannot be undone. Type 'I confirm' to proceed."

## During Implementation

If scope grows beyond the brief (Step 5 in the rules), stop and output:
"SCOPE CHANGE DETECTED: [what changed vs. the original brief]. Re-running architecture review before continuing."

Then re-run this review before proceeding.
