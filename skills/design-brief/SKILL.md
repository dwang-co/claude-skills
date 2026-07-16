---
description: Generate a structured design brief for use in Claude Design. Reads project context, runs a focused interview on what's missing, runs a behavioral-design pass (ux-psychology.md) on the priority screens, and produces a committed design spec covering character, color system, typography, screens, motion, anti-patterns, and open design questions. Run before any UI design work begins.
disable-model-invocation: false
allowed-tools: Bash Read Write
---

# Design Brief Generator

You are generating a design brief that will serve as the primary input to Claude Design for UI mockup generation. The brief must be specific enough that a designer has no ambiguity on committed decisions, and explicit about open questions so the design session is focused and productive.

---

## Step 1: Gather Project Context

Read these sources in order. Extract everything relevant to visual identity, product emotion, target users, and core experience:

1. Read the project-level `CLAUDE.md` in the current working directory — product goals, users, and tech stack
2. Check for any existing design files: `design-brief.md`, `brand.md`, or design-related notes in the root
3. Run `ls` to understand the project structure

Do not ask questions that can be answered from context already gathered.

---

## Step 2: Identify Gaps and Ask Once

After reading the context, identify what a complete brief requires that you cannot infer. The categories below are required — only ask about ones that are genuinely unknown:

1. **Product character** — Named persona or character? What archetype? North star test for design decisions?
2. **Emotional core** — What should the user feel when they open the app? What feeling must be avoided?
3. **Visual metaphor** — Does the product name or concept suggest a visual world? (e.g., "Forge" → fire/workshop/craftsmanship)
4. **Anti-patterns** — What 3–5 existing products should this NOT look like, and why?
5. **Color direction** — Dark or light first? Any existing brand colors or palette instinct?
6. **Priority screens** — Which 2–3 screens are most critical for the first design session?
7. **Open design questions** — What should be left for the designer to explore rather than locked in the brief?

Ask all missing questions in a single message. Do not ask one at a time. Wait for all answers before proceeding to Step 3.

---

## Step 3: Behavioral Design Pass

Read the full UX psychology rules:

~/.claude/rules/ux-psychology.md

For each priority screen identified in Step 2, run Step 1 ("Run the Principle Pass") from that file in **generative mode** — the screens don't exist yet, so this is about which of the 14 principles would remove real friction if designed in from the start (a smart default, a goal-gradient progress indicator, a decoy-free pricing layout, a Fogg-model trigger at the right moment), not a menu to apply everywhere. Most screens will only implicate 2–4 principles; skip the rest.

Fold the specific, concrete recommendations directly into the relevant screen's "Layout" and "Key component or non-obvious interaction" sections in Step 4 below — do not create a separate, disconnected psychology appendix. Any recommendation that would only work by being dishonest or asymmetric (per the three-question ethical test in the rules file) is rejected here, not carried into the brief.

---

## Step 4: Generate the Brief

Produce a complete design brief using this exact structure. Every section is required. "TBD" is not acceptable in committed-value sections — if something is truly unknown, it becomes an open design question.

---

### Brief Template

**[Project Name] — Design Brief**
*For use in Claude Design. Do not share externally.*

---

**Product in One Sentence**

[Single sentence: what the product is and what it does emotionally for the user — not a feature list]

---

**The Character: [Name or "Product Personality"]**

[Archetype in 2–3 words. Then: what this character embodies in concrete terms — not adjectives, but behaviors and relationships. What does this product do as if it were a person?]

In visual language:
- [Pair 1: what it is / what it is not]
- [Pair 2: what it is / what it is not]
- [Pair 3: what it is / what it is not]
- [Pair 4: what it is / what it is not]

**The north star test for every design decision:** [One question a designer can ask of any element to know if it belongs]

---

**Color System**

[2–3 sentences: where this palette comes from conceptually. The palette should feel inevitable given the product name and character.]

**Committed Values**

| Role | Hex | Usage |
|---|---|---|
| Base | `#______` | [Page backgrounds — describe the feeling, not just the function] |
| Surface | `#______` | [Cards, containers] |
| Surface elevated | `#______` | [Floating elements, modals] |
| Accent | `#______` | [Primary interactions — the brand color] |
| Text primary | `#______` | [All primary copy] |
| Text muted | `#______` | [Secondary info, timestamps, metadata] |
| [Additional role if needed] | `#______` | [description] |
| Destructive | `#______` | [Errors only — never used for warning or info] |

**[Accent Color] Usage Map — [N] Uses, Rigorously Defined**

[List every use of the accent color with name and opacity. This prevents proliferation. Every use must have a name and a specific opacity. 3–5 uses maximum.]

1. **[Use name]** — [opacity]%. [Description]
2. **[Use name]** — [opacity]%. [Description]
3. **[Use name]** — [opacity]%. [Description]

**What These Colors Communicate Together**

[1–2 sentences on the emotional effect of the palette as a system — what does the user feel when they see these colors together]

**Dark/Light:**

[State the decision clearly. If dark-first: "Light mode is not in scope." If light-first: state that. Do not hedge.]

---

**Typography**

[1–2 sentences on the font family choice and why it fits the product character. Reference Next.js defaults (Geist Sans) if no reason to deviate.]

**Type Scale — Committed Values**

| Level | Size | Weight | Line Height | Letter Spacing | Use |
|---|---|---|---|---|---|
| H1 | [px] | [weight] | [ratio] | [value or "default"] | [primary use] |
| H2 | [px] | [weight] | [ratio] | [value or "default"] | [primary use] |
| Body large | [px] | [weight] | [ratio] | [value or "default"] | [primary reading surface] |
| Body | [px] | [weight] | [ratio] | [value or "default"] | [secondary content] |
| Label | [px] | [weight] | [ratio] | [value or "default"] | [UI labels, metadata] |
| Caption | [px] | [weight] | [ratio] | [value or "default"] | [timestamps, supplementary] |

**Alignment rule:** [State the philosophy — what is left-aligned and why, when (if ever) centering is permitted]

---

**Spacing and Structure**

- **Base grid:** [value]px
- **Page margins:** [mobile]px on mobile, [tablet]px on tablet
- **Card radius:** [value]px — [rationale in a few words: "considered, not corporate" / "sharp, professional" / etc.]
- **Elevation:** [how depth is expressed — background color difference only / shadow / border / combination. Be specific.]
- **Touch targets:** [minimum]px minimum. [preferred]px preferred for primary actions.
- **Vertical rhythm:** [One sentence on the philosophy — tight or generous, and why it fits the product]

---

**[Character Name] Visual Language**

*This section specifies how the product's primary content surface renders — the element users look at most.*

**[Primary content type — e.g., message container, content card, etc.]**

[Exact spec: background, borders, padding dimensions, font level, color. Specific enough that a developer can implement it from this alone. Call out what is deliberately absent — no border, no shadow, etc. — and why.]

**[Thinking/loading state]**

[Exact spec: what renders, colors at what opacity, timing in ms, animation behavior. Name the animation — it should feel like the product character, not a generic loader.]

**[The most significant moment in the product UX — the "letter moment"]**

[Exact spec: layout, container treatment, colors, fade-in timing. State what makes this moment distinct from all other moments — and what that singularity communicates to the user.]

**[Modes or states, if applicable]**

[If the product adapts between states or modes: what changes visually and what stays identical. Be explicit about which visual properties are frozen across all modes.]

---

**Motion Principles**

[1–2 sentences on the role of motion in this product. Is it primarily functional (feedback, orientation)? Primarily emotional (habit-wiring, celebration)? Both?]

**General Transitions**
- Duration: [range]ms
- Easing: [specific function] — never linear (mechanical), never elastic (playful/childish) [adapt rationale to product character]
- [Primary content type]: [specific timing and fade/slide behavior]

**[Key animation — the most emotionally significant motion in the product]**

Triggered when: [specific user action]

- **Effect:** [describe the visual effect precisely]
- **Expand/reveal:** [timing and easing]
- **Fade:** [timing and easing]
- **Total duration:** [total]ms
- [Any exclusions: "No X. No Y. No Z."] [Rationale for restraint]
- **Reduced motion fallback:** [exact fallback behavior]

**Error States**

[How errors animate — or don't. Restrained or expressive. State the rationale in relation to the product character.]

---

**Screen [N]: [Screen Name]**

*Repeat this block for each priority screen (2–3 screens minimum).*

**Purpose:** [One sentence: what this screen accomplishes in the product]

**Emotional target:** "[Quoted sentence in the user's voice — what they should feel or think when they land on this screen]"

**Layout** *(priority order, top to bottom)*
1. [Most important element]
2. [Second]
3. [Third]
4. [etc.]

**[Key component or non-obvious interaction]**

[Exact spec for the element most likely to be designed incorrectly without explicit guidance. Cover: visual treatment, copy direction, what it must not look like, why.]

**States Matrix** *(include only if this screen has meaningfully different states)*

| State | [Visual dimension] | [Copy dimension] | [Interaction dimension] |
|---|---|---|---|
| [State name] | [value] | [value] | [value] |

**Rule:** [One rule that governs all states — what never changes regardless of state]

---

**Icon and Illustration Policy**

- **Library:** [specific library — Lucide unless there is a specific reason otherwise]
- **Style:** [outlined / filled]
- **Size:** [standard]px standard, [small]px small contexts
- **Stroke weight:** [value]px
- **Color:** [muted color] for utility icons, [accent color] for active/interactive states
- **Illustrations:** [Yes/no and a specific rationale — most products should say no to custom illustration and say explicitly why]

---

**What [Product Name] Must NOT Look Like**

| Do not reference | Why |
|---|---|
| [Product or genre] | [Specific reason this visual language is wrong for this product] |
| [Product or genre] | [Specific reason] |
| [Product or genre] | [Specific reason] |
| [Product or genre] | [Specific reason] |

*Minimum 4 entries. These protect the design from genre defaults and lazy borrowing.*

---

**[Character] Voice Reference — Five Scenarios**

*Every piece of copy in the design mockup must be calibrated against these examples. Do not use placeholder text.*

**[Scenario 1 — first interaction]:**
"[Example copy]"

**[Scenario 2 — recurring/habitual use]:**
"[Example copy]"

**[Scenario 3 — a setback or struggle state]:**
"[Example copy]"

**[Scenario 4 — a success or achievement state]:**
"[Example copy]"

**[Scenario 5 — an edge case or transition]:**
"[Example copy]"

---

**Design Decisions to Make in Claude Design**

*These are intentionally left open for the design session to explore. Each is a genuine question with no single right answer.*

1. [Question 1 — typically about a specific component treatment]
2. [Question 2 — typically about a layout or hierarchy choice]
3. [Question 3 — typically about a visual language tension identified in the brief]

---

**The Test**

Before marking any screen complete, answer three questions:
1. [Derived from the character/archetype — does this express it?]
2. [Derived from the emotional target — does this make the user feel the right thing?]
3. [Derived from the north star test stated in The Character section]

---

## Step 5: Save the Brief

Write the completed brief to `design-brief.md` in the current working directory (the project root).

Confirm: "Design brief saved to `design-brief.md`. Ready for Claude Design."

---

## Quality Bar

The brief passes only if all of these are true:

- Every color is a specific hex value — no "warm dark" or "amber-ish"
- Every typography value is in pixels and unitless ratios — no "medium" or "large"
- Every screen has an emotional target in the user's voice (a quoted sentence)
- The accent color usage map lists every use with name and opacity — no other uses permitted
- The anti-patterns section has at least 4 entries with specific rationale for each
- The open design questions are genuine questions — not deferred decisions disguised as questions
- A designer reading only this document could build a mockup with no follow-up questions on committed values
- The voice reference examples sound like a real person, not marketing copy — they could be cut and pasted directly into the product
