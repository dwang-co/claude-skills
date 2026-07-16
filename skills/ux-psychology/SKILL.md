---
description: Run a UX-psychology pass on a flow or screen — either designing a new one (generative) or auditing a built one (evaluative) — against 14 evidence-based behavioral principles (smart defaults, goal-gradient effect, endowment effect, IKEA effect, reciprocity, contrast effect, BJ Fogg's Behavior Model, anchoring, Hick's Law, and more), each rated by confidence and paired with an explicit legitimate-use vs. dark-pattern distinction. Use whenever the user wants to find friction, spot missed behavioral-design opportunities, or check that a persuasive design choice hasn't crossed into manipulation. Also runs automatically as part of design-brief and design-qa.
disable-model-invocation: false
allowed-tools: Bash Read Grep Glob
---

# UX Psychology Review

You are running a behavioral-design pass — either helping design a new flow or auditing a built one against a set of evidence-based psychology principles. This is not a menu of persuasion tricks to increase a metric; it's a way of finding real friction a real user feels, and ruling out designs that would only "work" by being dishonest or asymmetric.

## Step 1: Load the Rules

Read the full principle reference, severity classification, and legal guardrail:

~/.claude/rules/ux-psychology.md

## Step 2: Determine Mode and Scope

Ask yourself (don't ask the user unless genuinely ambiguous):

1. **Generative or evaluative?** Is there a flow being designed that doesn't exist yet (generative — apply principles to the spec/plan), or a built UI to review (evaluative — apply principles to the actual screens/code)?
2. **What specific flow or screen is in scope?** Scope to a named journey (onboarding, upgrade, a specific screen) — not "the whole app." If the request is that broad, ask which flow matters most right now, or run each flow as a separate pass.
3. **What is the user trying to accomplish, and what does the business want them to do?** Name both. Where they diverge is where dark-pattern risk concentrates.

## Step 3: Run the Principle Pass

Follow Step 1 (Run the Principle Pass) from `~/.claude/rules/ux-psychology.md` exactly as written: go through the 14 principles, but only report findings that are specific to this flow and would produce a real, measurable reduction in friction — not a generic restatement of a principle's description. Most flows only implicate 3–5 of the 14; forcing all of them is a sign the pass is being done mechanically rather than with judgment.

For every principle currently applied (intentionally or not), run the three-question ethical test from the rules file and check it against the Legal Guardrail section. Anything that fails escalates to **Blocker** automatically.

## Step 4: Classify Findings

Every finding gets exactly one severity — **Blocker**, **Critical**, or **Warning** — per the definitions in the rules file. State the principle, the specific location, and the specific recommended change (generative) or fix (evaluative).

## Step 5: Sign-off

Produce the sign-off exactly as specified in `~/.claude/rules/ux-psychology.md` Step 3:

**Mode**: [generative / evaluative]
**Flow(s) reviewed**: [name]
**Principles applied or considered**: [list]

**Blockers**: [none / list]
**Critical findings**: [none / list]
**Warnings**: [none / list]

**Ethical check**: [pass / list failures]
**Legal guardrail check**: [pass / list failures]

This is an informational/advisory pass, not a hard gate on its own — but any Blocker found here should be treated with the same seriousness as a security or accessibility blocker, and resolved before the flow ships.
