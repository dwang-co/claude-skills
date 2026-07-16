---
description: Run the design QA review before marking any UI task done. Verifies responsiveness at 375px and 1280px, visual hierarchy, contrast, interaction states, form accessibility, brand coherence against the approved design, and behavioral friction (ux-psychology.md) against dark-pattern misuse.
disable-model-invocation: false
allowed-tools: Bash Read
---

# Design QA Review

You are running the design QA gate. A UI task is not done because it looks right at a glance. Run every step in order. Fix all experience-breaking issues before marking done.

## Step 1: Load the Rules

Read the full design QA rules:

~/.claude/rules/design-qa.md

## Step 2: Execute the Review

Follow all eight steps as written:

1. **Design Reference Check** (Step 0) — Locate the approved design artifact. If none exists, flag it and state: "No approved design artifact found. Proceeding with experience, quality, and coherence checks only."

2. **Responsiveness Check** (Step 1) — Verify at 375px, 1280px, and 768px. Fix any experience-breaking issue before continuing. Do not proceed with broken layout.

3. **Hierarchy and Intent** (Step 2) — Apply the two-second test. Every screen must have exactly one primary intent. Flag if the eye cannot identify what to do within two seconds.

4. **Design Quality** (Step 3) — Typography (16px minimum body, 1.4–1.6 line height), color contrast (WCAG AA: 4.5:1 normal text, 3:1 large), reduction (remove anything that doesn't serve primary intent), spatial consistency.

5. **UX Clarity and Error Prevention** (Step 4) — Orientation, error prevention on destructive actions, feedback after every action, no dead ends.

6. **Fidelity Check** (Step 5) — Compare against approved design. Classify every deviation as intentional or unintentional.

7. **Brand Coherence** (Step 6) — Copy tone, interaction patterns, and visual language consistent with the rest of the product.

8. **Interaction States and Forms** (Step 7) — All three states (loading, error, empty) present for every new interaction. All form requirements met: visible labels, inline validation, keyboard-submittable, focus returns to first error on submission failure.

## Step 2.5: Behavioral Friction Audit

Read the full UX psychology rules:

~/.claude/rules/ux-psychology.md

Run Step 1 ("Run the Principle Pass") from that file against the flow just reviewed — evaluative mode, since this UI is already built. Only report findings specific to this flow, not a generic restatement of a principle. Any finding that fails the three-question ethical test or trips the Legal Guardrail section escalates to Blocker automatically, feeding into the same severity tiers used elsewhere in this review.

## Step 3: Output Sign-off

Produce the design sign-off from Step 8 of the rules:

**Design artifact referenced**: [name or "none — quality and coherence checks only"]
**Project design system applied**: [yes / no / not yet defined]

**Responsiveness**
- 375px: [pass / issues]
- 1280px: [pass / issues]
- 768px: [pass / issues noted]

**Hierarchy and intent**: [pass / issues]
**Design quality**: [pass / issues]
**UX clarity and error prevention**: [pass / issues]
**Fidelity**: [pass / deviations — intentional or unintentional]
**Brand coherence**: [pass / issues]
**Interaction states (loading / error / empty)**: [all present / missing — list]
**Forms and keyboard**: [pass / issues]

**Behavioral friction (ux-psychology.md)**
- Blockers: [none / list]
- Critical findings: [none / list]
- Warnings: [none / list]
- Ethical/legal guardrail check: [pass / list failures]

**Experience-breaking issues resolved**: [yes / list outstanding]
**Experience-degrading issues flagged**: [none / list for follow-up]
**Polish noted for later**: [none / list]

Mark done only when all experience-breaking issues are resolved.
