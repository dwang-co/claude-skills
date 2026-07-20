---
name: retro
description: Run a retrospective on a /ship session, PR review feedback, or a stretch of agent-assisted work, and turn the lessons into proposed improvements to the agent definitions (.claude/agents/), skills (.claude/skills/), and CLAUDE.md. Proposals only - every change is a diff presented for the user's approval, one at a time; nothing is ever self-applied. Includes a mandatory consolidation pass so the context stays lean. Use when the user says "retro", "what did we learn", "improve the agents/workflow", or after a PR lands with substantial human corrections.
---

# /retro — the learning loop

This is how the agent system gets smarter: not by changing the model, but by compounding better context. That only works if the context stays *good* — a retro that only ever adds rules produces a bloated, contradictory instruction set that makes every future session worse. Your output is proposals; the user is the curator; and pruning is as much your job as capturing.

## Step 1 — Gather evidence

Review whichever of these exist for the period under retro:

- **Run folders** (`.pipeline/<date>-<slug>/`): the primary durable record. Read each run's spec, handoffs, verdicts, and QA evidence — compare what the planner specced against what shipped (deviations = spec quality signal), what the reviewer caught against what the engineer handed off (foreseeable-defect signal), and whether specced runs looped less than unspecced ones (the data that decides the engineer model downgrade).
- The session itself, if still available: where did an agent loop back? What did code-reviewer or qa-verifier catch that engineer should have foreseen? What did a stage miss that a later stage (or the user) caught?
- PR review feedback: what did the user correct at review? Human corrections are the highest-signal input here — each one is a defect the whole agent pipeline missed.
- Rework and friction: stages that ran but added nothing, questions the orchestrator had to ask that a better prompt would have pre-answered, wrong assumptions traced to a missing or misleading instruction.

A lesson must trace to a *specific observed failure or friction*, not a hypothetical. "This might help someday" is how bloat starts.

## Step 2 — Draft proposals

For each lesson, draft a concrete diff to the file that owns the behavior: an agent definition in `.claude/agents/`, a skill in `.claude/skills/`, or CLAUDE.md. Prefer editing an existing rule over adding a new one. Write rules as *why + what*, not bare imperatives — future agents follow rules better when the reason travels with them.

Guard against overfitting: one incident is an anecdote, not a rule. If the lesson generalizes ("QA needs ACs before it can verify anything"), propose it. If it's specific to one odd task, propose nothing — or at most a note in the PR, not a permanent instruction.

Proposals aren't limited to edits. When the evidence shows a *recurring workflow with no owner* — the same multi-step sequence reinvented across sessions, or a capability gap no existing agent covers — propose creating a **new skill** or a **new agent definition**. New capabilities go through the same one-at-a-time approval gate as edits, and the bar is higher: a new skill is a permanent context cost on every future session, so it must be justified by observed repetition, not anticipated usefulness.

## Step 3 — The consolidation pass (mandatory)

Every retro also audits the existing context for bloat. Every token in these files is spent on every future session — each one must make the system better, not worse. Look for:

- **Stale rules**: instructions about code, decisions, or phases that no longer exist.
- **Redundancy**: the same rule stated in two files, or a rule that merely restates what CLAUDE.md already says. One owner per rule; others reference it.
- **Dead weight**: rules added by past retros that have never observably changed behavior. Provenance tallies (Step 5) are the evidence.
- **Contradictions**: rules that pull in different directions — surface the conflict for the user to resolve rather than letting agents silently pick one.

Propose deletions and merges with the same seriousness as additions. A retro that proposes only additions should be treated as incomplete — say explicitly what you looked at and found clean.

## Step 4 — Present for approval

Present each proposal **one at a time** via AskUserQuestion: the diff, the observed failure it traces to, and expected effect. The user approves, edits, or rejects. **Never apply a change that wasn't approved** — an unsupervised self-editing loop is exactly the failure mode this design rejects. Rejected proposals are recorded (Step 5) so future retros don't re-propose them.

## Step 5 — Provenance

Each touched file keeps a short `## Provenance` footer: one line per retro-originated rule — date added, one-phrase trigger. On later retros, update the line if the rule earned its keep or propose deletion if it never did. This tally is the data behind consolidation and promotion decisions. Also note rejected proposals here (one line) to prevent re-proposal.

## Step 6 — Promotion path (project → global)

When a rule or agent has proven out across multiple sessions and is not specific to one project, propose promoting it to the user's global Claude config (`~/.claude/` or their public skills repo) so it applies across all projects. Promote what's earning its keep, drop what's going unused. Note in the project copy that it was promoted, so the two don't drift silently.
