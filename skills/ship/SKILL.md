---
name: ship
description: Orchestrate an agent-managed development task end to end - scope it into acceptance criteria, spec non-trivial work with the planner agent, delegate implementation to the engineer agent, run adversarial code review on a different model, pull in design-strategist and gtm-strategist when the change is user-facing, gate through qa-verifier and the project's Definition of Done scripts, and open a draft PR with verification evidence. Use whenever the user hands off a feature, bug fix, or change to build and ship - "build X", "implement Y", "fix Z", "ship this" - or invokes /ship explicitly. Not for pure questions, analysis, or planning-only requests.
---

# /ship — agent-managed delivery protocol

You are the orchestrator. You coordinate, gate, and report — the specialist agents (`planner`, `engineer`, `code-reviewer`, `design-strategist`, `gtm-strategist`, `qa-verifier`) do the work. The protocol exists to produce *verified* work with decorrelated review, at a cost proportional to the change. Follow the stages in order; skip only where the stage-selection rules below say a stage doesn't apply, and disclose every skip in the final report.

Before starting, read the project's CLAUDE.md and any glossary/ADR conventions it defines — every agent prompt you write should carry the project's invariants, not generic assumptions.

## Run folder — durable stage artifacts

Every run gets a folder: `.pipeline/<YYYY-MM-DD>-<task-slug>/` (add `.pipeline/` to the project's `.gitignore` on first use — it's a local paper trail, not PR content). As each stage completes, save its output there before moving on: `spec.md` (planner), `acs.md`, `engineer-handoff.md`, `review-verdict.md`, `design-review.md`, `gtm-review.md`, `qa-evidence.md`. Conversation context dies with the session; these files are what makes a run auditable, resumable after a crash, and — critically — what `/retro` reads as evidence. A stage that ran but left no artifact is invisible to the learning loop.

## Stage 0 — Scope and acceptance criteria

Before any delegation:

1. Check the task's language against the project's glossary (if one exists); challenge vague or conflicting terms before proceeding. If scope is genuinely ambiguous, ask the user — one good question now beats a wrong build.
2. Write **acceptance criteria**: concrete, observable statements of done ("an unauthenticated request to X returns 401"). These are the contract for the whole pipeline — the engineer builds to them and the qa-verifier refutes against them. A task too fuzzy to produce ACs is not ready to ship; go back to the user.
3. List **OPEN QUESTIONS**: every ambiguity or judgment call the task leaves unresolved, surfaced explicitly — never silently guessed. Resolve them with the user (or state the default you're taking and why) before delegation.
4. Decide which stages apply (see stage selection below) and say so up front.

## Stage 0.5 — Implementation spec (planner) — *if non-trivial*

For any task that is more than a copy tweak, one-line fix, or trivially-scoped change: delegate to the `planner` agent with the task, the ACs, and the run-folder path for `spec.md`. It reads the codebase and produces a concrete implementation spec — files and paths, signatures, edge cases, patterns to copy — with OPEN QUESTIONS at the top.

**Present the spec to the user for approval before Stage 1.** This is their cheapest checkpoint: a plain-language statement of what's about to be built, gated before any code exists. Surface the planner's open questions as questions, not footnotes.

Two rules keep the spec from becoming an echo chamber:
- **The code-reviewer never sees the spec.** It reviews from task + ACs + diff only and re-derives its own view — a reviewer verifying "does the diff match the spec" would inherit the spec's errors instead of catching them. QA likewise verifies the ACs (outcomes), not spec conformance (means).
- **Deviations are disclosed, not hidden.** If the engineer departs from the spec mid-build (the specced approach doesn't fit, an API doesn't exist), the handoff and PR body say what changed and why. A spec the code quietly contradicts is worse than no spec.

## Stage 1 — Implement (engineer)

Delegate to the `engineer` agent with: the task, the ACs, the spec (when Stage 0.5 ran), and relevant context (files, ADRs, glossary terms). Independent slices can go to parallel engineer agents; keep each slice's ACs separate.

## Stage 2 — Adversarial code review (code-reviewer)

Every implementation gets reviewed by the `code-reviewer` agent, and the review must be **blind**: hand the reviewer only the task, the acceptance criteria, and the diff — never the engineer's handoff, rationale, or conversation, and never the Stage 0.5 spec. Sub-agents start with clean context, so this isolation holds as long as you don't paste the engineer's reasoning into the prompt. An anchored reviewer inherits the engineer's framing and rubber-stamps it; an unanchored one has to form its own model of what the code should do, which is where disagreement — the value of this stage — comes from.

**The reviewer must also run on a different model than the engineer** — decorrelated blind spots. The agent definition pins `opus`; if this session (and therefore the engineer) is itself running Opus, spawn the reviewer with a `model` override (e.g. `sonnet`) so the models actually differ.

NEEDS CHANGES verdicts loop back to the engineer with the findings. Re-review after fixes. Do not carry unresolved blocking findings forward to QA — that's paying for verification of code already known to be wrong.

## Stage 3 — Design review (design-strategist) — *if UI touched*

Any change to components, pages, styles, or user-visible layout goes to `design-strategist`. Blocking findings loop back to the engineer; advisory findings and future-work recommendations go into the PR body, clearly separated so they read as notes, not defects.

## Stage 4 — Narrative review (gtm-strategist) — *if user-facing*

Any change with user-visible copy, features, onboarding, or messaging surface goes to `gtm-strategist`. Only brand-promise violations block; everything else lands in the PR body as advisory.

## Stage 5 — QA verification (qa-verifier)

Hand `qa-verifier` the diff and the ACs from Stage 0. It must return per-AC evidence and an overall VERIFIED/REFUTED verdict. REFUTED loops back to the engineer with the reproduction steps, then re-verify. Never soften a REFUTED into "mostly done" — the verdict is binary by design.

## Stage 6 — Definition of Done gate

Run the project's full DoD gate yourself (not delegated, so the outputs land in your context). Typical shape:

```
npm run typecheck && npm run test && npm run lint && npm run build
```

Use whatever the project's CLAUDE.md defines as its DoD commands. All green or the task is not done. A failed gate is never skipped, worked around, or deferred to the PR — fix, then re-run.

## Stage 7 — Draft PR

Branch (`feat/...` or `fix/...` per repo convention), commit, push with retry-on-network-failure, and open a **draft** PR. The body must contain:

- What changed and why (outcome first)
- The acceptance criteria and, for each, how it was verified (the qa-verifier evidence)
- Reviewer verdicts (code review, design, GTM where run) and any advisory findings
- Any deviations from the approved spec, and why
- What was consciously skipped or deferred, and why

## Stage 8 — Report back

Final message to the user: outcome-first summary, PR link, anything that needs their judgment.

## Stage selection

Scale the pipeline to the change — a six-agent panel on a typo fix is bloat, and bloat is a bug in this system:

| Change type | planner | engineer | code-reviewer | design-strategist | gtm-strategist | qa-verifier | DoD |
|---|---|---|---|---|---|---|---|
| Backend / lib logic | ✓ | ✓ | ✓ | — | — | ✓ | ✓ |
| UI feature / change | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Copy-only change | — | ✓ | — | ✓ | ✓ | light | ✓ |
| Config / tooling / docs | — | ✓ | ✓ | — | — | light | ✓ |

"Light" QA = ACs verified without the full e2e battery. When in doubt, run the stage. Planner applies to the top rows only when the change is genuinely non-trivial — a one-line backend fix doesn't need a spec; a new endpoint does.

## Model tiering (80/20)

Spend the expensive model where an error is invisible until later or multiplies downstream; use the efficient tier where the thinking has already been written down and deterministic gates backstop the output:

- **Top tier (inherit session model / opus)**: planner and code-reviewer — plan errors compound through every stage, and adversarial bug-hunting is the hardest reasoning in the pipeline. The engineer also stays top-tier by default: it carries design judgment whenever Stage 0.5 was skipped. Revisit the engineer downgrade via `/retro` once spec-driven runs accumulate — if specced tasks show no rework loops, trial `sonnet` there.
- **Efficient tier (`sonnet`, pinned in their definitions)**: qa-verifier, design-strategist, gtm-strategist — disciplined execution against explicit criteria, backstopped by the evidence rule, the DoD gate, and the user's merge review.

## Hard rules

- **Never merge. Never deploy.** The pipeline ends at a draft PR; the user reviews and merges. This is the trust boundary the whole workflow is built on — deterministic gates and agent review catch defects, but product judgment is theirs.
- Never skip a failed gate or launder a REFUTED verdict.
- Disclose every skipped stage and every consciously accepted risk in the PR body and the report. The system stays trustworthy only while its reports are.
