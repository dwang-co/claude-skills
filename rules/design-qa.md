# Design QA Rules

## Purpose
Run this review before marking any UI task done. The goal is to verify the built interface matches the approved design, communicates clearly, respects the user's attention, and feels like it belongs to the same product as everything else built before it.

Priority order: experience-breaking issues must be fixed before done. Experience-degrading issues must be flagged. Polish is noted for later.

Invoke explicitly: after implementation, before marking a UI task done.

---

## Severity Classification

**Experience-breaking — must fix before marking done:**
- Layout broken or content overflowing at 375px or 1280px
- Interactive element (button, link, input) that cannot be activated
- Form input with no visible label
- Missing loading, error, or empty state on a user-facing interaction
- Error message that is a raw code, stack trace, or developer-facing string
- Focus ring removed or keyboard navigation broken
- No clear visual hierarchy — a user cannot identify what to do within two seconds of seeing the screen
- Two or more competing primary actions with equal visual weight
- Text or UI component with color contrast below WCAG AA (4.5:1 for normal text, 3:1 for large text and UI components)
- User can reach a dead-end state with no recovery path

**Experience-degrading — flag and note before marking done:**
- Spacing or sizing that deviates from the approved design but does not break layout
- Spacing that is internally inconsistent — no discernible rhythm or grid
- Component that works but does not match the design system pattern
- Missing hover states on desktop interactive elements
- Copy tone that is inconsistent with the voice used elsewhere in the product
- Interaction pattern that contradicts how the same action is handled on another screen
- Secondary action that is unclear but not invisible
- Typography that matches the spec but fails readability (body text below 16px, line height below 1.4)

**Polish — note for a future pass:**
- Micro-interactions or transitions that could be improved
- Minor visual inconsistency that does not affect usability or coherence

---

## Step 0: Design Reference Check

Before reviewing anything, locate the approved design artifact for this task.

Ask: is there an approved design — a Figma file, Claude Design output, mockup, or written spec — for what was just built?

- **Yes**: reference it explicitly in this review. All fidelity judgments are made against it.
- **No**: flag it before proceeding. Output: "No approved design artifact found for this task. Fidelity cannot be assessed. Proceeding with experience, quality, and coherence checks only — confirm this is intentional."

If a project-level design system exists (color palette, typography scale, spacing system, component patterns), note it here. All fidelity and coherence checks apply against it as well as the task artifact.

---

## Step 1: Responsiveness Check

Open the built UI and verify at all three breakpoints. Fix any experience-breaking issue before continuing.

**At 375px (mobile):**
- Does all content fit within the viewport without horizontal scroll?
- Do all text elements render without overflow or truncation that cuts off meaning?
- Are touch targets (buttons, links, inputs) at least 44px tall and spaced far enough apart to tap accurately?
- Does the layout stack correctly — no side-by-side elements that collapse into each other?
- Are images and media constrained to the viewport width?
- Is the primary action visible without scrolling, or clearly reachable with minimal scroll?

**At 1280px (desktop):**
- Does the layout use the available width appropriately — no content pinched to a narrow column?
- Do all interactive elements have visible hover states? Missing hover states on desktop are experience-degrading.
- Does anything that was stacked on mobile now display in its intended desktop layout?

**At 768px (spot check):**
- Does the layout transition cleanly between mobile and desktop, with no broken midpoint?

Flag every breakpoint failure as experience-breaking. Do not proceed until all are resolved.

---

## Step 2: Hierarchy and Intent Check

This is the most fundamental design quality check. Both Apple and Pentagram start here. A screen can be responsive, correct, and pixel-perfect against a spec and still fail if the hierarchy is unclear or the intent is split.

**The two-second test:**
Look at the screen fresh. Within two seconds, can you identify:
1. What this screen is for?
2. What the user is being asked to do?

If the answer to either question is no, the hierarchy has failed. This is experience-breaking.

**Single primary intent:**
Every screen should have exactly one primary intent — one thing it is asking the user to do. Identify it. Then verify:
- Is the primary action the most visually prominent element on the screen?
- Is every other element subordinate to it in visual weight?
- If two actions appear equally prominent, one of them needs to be subordinated or the design needs to be reconsidered.

**Visual flow:**
- Does the eye move naturally from the most important element to the second most important, then to supporting content?
- Are elements ordered by importance, not by the convenience of the layout?

**Gestalt grouping:**
- Do elements that belong together look like they belong together — through proximity, alignment, or shared visual treatment?
- Are unrelated elements visually separated?
- A group of related inputs that looks like a list of unrelated items is a grouping failure.

---

## Step 3: Design Quality Check

This step evaluates whether the design itself meets a quality standard — independent of whether it matches the spec. A design that faithfully reproduces a poor-quality spec is still a poor-quality build.

**Typography:**
- Is body text at least 16px on mobile? (Apple uses 17px as its baseline; 16px is the acceptable minimum.)
- Is line height for body text between 1.4 and 1.6? Text that is too tight or too loose both hurt readability.
- Is there meaningful weight contrast between heading and body? A heading and body text that are the same weight and size create no hierarchy.
- Is letter spacing appropriate — not artificially tight on body text, not excessively loose on headings?

**Color and contrast:**
- Does all normal-sized text (below 18px regular or 14px bold) meet WCAG AA contrast: 4.5:1 against its background?
- Do large text elements and UI components (borders, icons used for meaning) meet 3:1 contrast?
- Is color being used to communicate meaning — or only for decoration? If color is the only signal that distinguishes an element (e.g., a red error state with no other indicator), it fails for users who cannot perceive color.

**Reduction:**
- Is there anything on this screen that does not serve the primary intent?
- Does the screen ask the user to process more than one concept at a time?
- Is any complexity hidden until the user needs it (progressive disclosure), or is everything visible at once?
- Apply Pentagram's test: if you removed an element and the screen got better, the element should not be there.

**Spatial consistency:**
- Does the spacing follow a consistent rhythm — not arbitrary, not mixing 12px, 13px, 15px, 17px margins?
- Are like elements given the same spacing treatment throughout the screen?
- Spatial inconsistency does not need to match a specific grid system — it needs to be internally rhythmic. If no grid is defined, the spacing should at minimum follow a consistent multiplier (8px, 16px, 24px, 32px).

---

## Step 4: UX Clarity and Error Prevention Check

This step tests whether a user who has never seen this feature can use it correctly — and whether the UI reduces the chance of them making a mistake.

**Orientation:**
- Is it immediately clear what this screen or feature is for?
- Is the primary action visually distinct and obviously actionable?
- Is there any element that looks interactive but is not, or looks static but is interactive?
- Does any modal, sheet, dialog, or overlay auto-open on page load without explicit user action? Auto-triggering overlays interrupt users before they signal intent — this is experience-breaking unless there is a documented, intentional exception (e.g., a required onboarding step). Default state for all overlays is closed.

**Error prevention (Apple's priority over error correction):**
- Does the UI prevent users from reaching invalid states?
- Are destructive actions (delete, remove, overwrite) confirmed before executing?
- Are inputs validated inline, before submission, so errors are caught early?
- Are disabled states explained — not just grayed out, but accompanied by a reason why?
- Are smart defaults set to reduce the chance of user error?

**Feedback and flow:**
- After every action, does the user receive immediate feedback? (loading state, success confirmation, or error)
- If an action succeeds, is the outcome visible — does something change on screen to confirm it worked?
- If an action fails, does the error message tell the user what went wrong and what to do next in plain language?

**Dead ends:**
- Can a user get into a state with no clear path forward?
- Is the empty state informative — does it tell the user why it's empty and what to do?

Flag dead-end states and missing error prevention on destructive actions as experience-breaking. Feedback gaps without any confirmation are experience-breaking. Missing inline validation is experience-degrading.

---

## Step 5: Fidelity Check

Compare the built UI against the approved design artifact from Step 0. If no design artifact exists, skip this step and note it in the sign-off.

**Visual fidelity:**
- Do spacing, sizing, and layout match the approved design?
- Do colors, typography, and iconography match?
- Are the correct components used — not substitutes that look similar but differ in behavior?

**Deviation classification:**
For every deviation from the approved design, classify it:
- **Intentional** — a deliberate improvement identified during the quality checks above. State why the deviation produces a better result than the spec.
- **Unintentional** — a mistake or oversight. Fix before marking done.

If a project design system exists:
- Does this screen use the correct tokens (colors, spacing, type styles) from the system?
- Does it introduce any one-off values that should instead reference the system?

---

## Step 6: Brand Coherence Check

This step checks project-level coherence — whether this screen feels like it came from the same product as everything else built before it. This is not a brand identity check (that requires a mature design system). It is a check that the product does not feel like it was built by different teams at different times.

**Copy tone:**
- Do the labels, CTAs, error messages, empty states, and confirmation text on this screen sound like the same voice as the rest of the product?
- Is the register consistent — not formal in one place, casual in another?
- Read a CTA from this screen next to a CTA from another screen. Do they feel like they were written by the same person?

**Interaction patterns:**
- Do similar actions on this screen behave the same way they do elsewhere in the product?
- If a confirmation dialog is used for deletion in one place, is it used here too?
- If a bottom sheet is used for a certain category of action elsewhere, is the same pattern used here?
- Inconsistent patterns force users to re-learn the product mid-use.

**Visual language:**
- Do components on this screen — cards, buttons, headings, inputs — look like they belong to the same family as their equivalents on other screens?
- Is any element introducing a new visual pattern that has no precedent in the product and no clear reason to exist?

Copy tone inconsistency and interaction pattern contradictions are experience-degrading. A screen that introduces a new visual pattern without justification should be flagged and the deviation classified as intentional or unintentional.

---

## Step 7: Interaction States and Forms

Check every user-facing interaction added in this task against the following required standards.

**Interaction states — all three required:**
- [ ] **Loading**: a visible indicator appears while an async action is in progress. The UI does not freeze silently.
- [ ] **Error**: a human-readable message appears that tells the user what went wrong and what to do next. Not a status code. Not a generic "something went wrong."
- [ ] **Empty**: when there is no data to show, the empty state renders correctly and tells the user why and what to do.

**Forms — all required:**
- [ ] Every input has a visible label above or beside it — not placeholder text used as a label
- [ ] Required fields are clearly marked
- [ ] Inline validation errors appear at the field level, not only at the top of the form
- [ ] The form can be submitted using the keyboard alone (Tab to navigate, Enter to submit)
- [ ] On submission error, focus returns to the first field with an error

**Focus and keyboard:**
- [ ] Focus rings are visible on all interactive elements — not removed or hidden
- [ ] Tab order follows the logical reading order of the page
- [ ] No interactive functionality requires a mouse — every action is reachable by keyboard

Any missing state or form requirement is experience-breaking. Fix before marking done.

---

## Step 8: Design Sign-off

Before marking the task done, output this summary:

**Design artifact referenced**: [name or description, or "none — quality and coherence checks only"]
**Project design system applied**: [yes / no / not yet defined]

**Responsiveness**
- 375px: [pass / issues — classify]
- 1280px: [pass / issues — classify]
- 768px: [pass / issues noted]

**Hierarchy and intent**: [pass / issues — classify as breaking or degrading]
**Design quality** (typography / contrast / reduction / spatial): [pass / issues — classify]
**UX clarity and error prevention**: [pass / issues — classify]
**Fidelity**: [pass / deviations — classify as intentional or unintentional]
**Brand coherence** (copy / patterns / visual language): [pass / issues — classify]
**Interaction states (loading / error / empty)**: [all present / missing — list which]
**Forms and keyboard**: [pass / issues — list which requirements are unmet]

**Experience-breaking issues resolved**: [yes / list any outstanding]
**Experience-degrading issues flagged**: [none / list for follow-up]
**Polish noted for later**: [none / list]

Mark done only when all experience-breaking issues are resolved and accounted for.

---

## Guiding Principles

- A UI task is not done because it looks right at a glance. It is done when a first-time user can orient, act, and recover — without instruction.
- Every screen has one primary intent. If you cannot name it in five words, the design is not ready.
- Match the approved design. Deviate intentionally and document why. Never deviate by accident.
- Brand coherence is built one screen at a time. Every inconsistency you accept becomes the new standard.
