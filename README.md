# Claude Code Skills

A set of custom [Claude Code](https://claude.com/claude-code) skills I use to run software projects end-to-end with AI — from architecture review through security, legal, SEO, performance, and UX sign-off — without a technical background.

Each skill is a checklist-driven review that Claude runs automatically at a specific point in the dev workflow, so nothing ships without the same rigor a team of specialist reviewers would apply.

## What's here

| Skill | Runs when | Purpose |
|---|---|---|
| `arch-review` | Before any non-trivial feature or structural change | Classifies risk (SAFE / CAUTION / HIGH RISK / DESTRUCTIVE), protects critical files, locks scope before code is written |
| `design-brief` | Before any UI design work begins | Generates a structured design spec (character, color, typography, screens, motion) with a behavioral-design pass baked in |
| `design-qa` | Before marking any UI task done | Checks responsiveness, hierarchy, contrast, interaction states, accessibility, and brand coherence against the approved design |
| `feature-done` | Before every commit | Runs the full Definition of Done: test coverage, regression check, security spot-check, design QA for UI work |
| `pre-launch` | Before every production deploy | Chains security, legal, SEO, and performance review; blockers must clear before launch |
| `ux-psychology` | Inside `design-brief` / `design-qa`, or standalone | Audits a flow against 14 evidence-based behavioral principles (defaults, goal-gradient, endowment, anchoring, etc.), each with a legitimate-use vs. dark-pattern distinction |

`rules/` holds the underlying review checklists (architecture, security, testing, legal, SEO, performance, UX psychology, design QA) that the skills above load and apply.

## Why this exists

I'm a non-technical PM building real products with Claude Code. These skills encode the review process a competent engineering team would run by default — so "vibe coding" still ships with security, legal, and UX guardrails instead of skipping them because no one's there to ask.

## Using these skills

Drop `skills/*` into `~/.claude/skills/` and `rules/*` into `~/.claude/rules/` (or the project-local `.claude/` equivalent). Claude Code will pick them up automatically and invoke them at the trigger points described in each `SKILL.md`.
