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

| `ship` | When handing off a feature, fix, or change to build end-to-end | Orchestrates the agent-managed delivery pipeline: acceptance criteria → implementation spec (approved by you) → engineer → blind adversarial code review on a different model → design + GTM review for user-facing work → adversarial QA against the criteria → Definition of Done gate → draft PR. Never merges or deploys |
| `retro` | After a `/ship` run or a PR with substantial human corrections | The learning loop: proposes evidence-traced edits to the agent definitions and skills, one at a time for approval, with a mandatory consolidation pass so the instruction set stays lean |

`agents/` holds the specialist agent definitions the `ship` pipeline delegates to:

| Agent | Role | Model tier |
|---|---|---|
| `planner` | Reads the codebase and turns a scoped task into an implementation spec — files, signatures, edge cases, OPEN QUESTIONS surfaced for human approval before any code is written | Top |
| `engineer` | Implements against acceptance criteria (and the spec, when one exists), writes tests alongside, discloses deviations and risks in a structured handoff | Top |
| `code-reviewer` | Adversarial review of the diff — runs blind (no engineer rationale, no spec) on a different model so blind spots don't overlap; read-only by construction | Top (`opus`) |
| `qa-verifier` | Tries to *refute* "it works": drives the real app, attacks edge cases, requires evidence for every claim; binary VERIFIED/REFUTED verdict | Efficient (`sonnet`) |
| `design-strategist` | Reviews UI changes at three levels: design-system conformance, brand/product strategy fit, and a user-journey walkthrough as the target user | Efficient (`sonnet`) |
| `gtm-strategist` | Reads each change as a message to the user: positioning fit, comprehension, copy that overclaims what the product enforces | Efficient (`sonnet`) |

The pipeline's design principles: specs are approved by a human before code exists, the reviewer never sees the spec (decorrelation over conformance), every stage writes a durable artifact to `.pipeline/` that `retro` later reads as evidence, expensive models are spent only where errors compound (planning, review), and the pipeline always ends at a draft PR — a human merges.

`rules/` holds the underlying review checklists (architecture, security, testing, legal, SEO, performance, UX psychology, design QA) that the skills above load and apply.

## Why this exists

I'm a non-technical PM building real products with Claude Code. These skills encode the review process a competent engineering team would run by default — so "vibe coding" still ships with security, legal, and UX guardrails instead of skipping them because no one's there to ask.

## Using these skills

Drop `skills/*` into `~/.claude/skills/`, `agents/*` into `~/.claude/agents/`, and `rules/*` into `~/.claude/rules/` (or the project-local `.claude/` equivalents). Claude Code will pick them up automatically and invoke them at the trigger points described in each `SKILL.md`.
