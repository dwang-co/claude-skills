# UX Psychology Review Rules

## Purpose
Run this review when designing a new flow (generative) or auditing a built one (evaluative). The goal is to find where behavioral friction is losing users who would otherwise succeed, and where a well-evidenced principle would remove that friction — while explicitly ruling out uses that cross from "designed to help the user" into "designed to trick them."

This file is the shared evidence base for two invocation points:
- **Standalone**: `/ux-psychology` — run on demand for a full audit or design pass
- **Chained**: a short pass inside `design-brief.md` (generative, before mockups) and `design-qa.md` (evaluative, after the UI is built)

Every principle below was checked against its original primary source through an adversarial research process (multiple independent verifiers, each trying to refute the claim) before being included here. Where evidence is weak, contested, or was not independently re-verified, that is stated explicitly — do not upgrade a caveated claim to "settled science" when applying it. A UX principle whose real-world effect size is near zero is not a good reason to add friction or complexity to a design.

Priority order: blockers must be resolved before marking done. Critical findings must be resolved before merge. Warnings are flagged and triaged.

Invoke explicitly: during `design-brief.md` generation, during `design-qa.md` review, or standalone via `/ux-psychology`.

---

## The Ethical Line: Legitimate Use vs. Dark Pattern

Nearly every principle in this file can be used two ways: to help a user get what they actually want with less friction, or to manipulate them into something they wouldn't choose with full information and an unhurried moment. The mechanism is often identical — what differs is truthfulness, reversibility, and whether the friction is symmetric.

**The three-question test.** Before applying any principle below, ask:

1. **Is it true?** A default, a scarcity indicator, a social-proof number, a streak — all must reflect a real state of the world. A countdown that resets on refresh, an "only 2 left" that doesn't track real inventory, or a testimonial count that's fabricated fails this test regardless of which principle it's dressed up as.
2. **Is it symmetric?** The path that benefits the business (subscribing, sharing data, upgrading) should not be structurally easier than the path that benefits the user (canceling, opting out, downgrading). This is now a literal legal standard, not just an ethics guideline — see the Legal Guardrail section below.
3. **Would the user thank you for it, or feel tricked, if they noticed exactly what you did?** A default that saves a user five clicks on a choice they'd have made anyway passes. A default that quietly opts them into data sharing they'd have declined if asked directly does not.

Every principle entry below includes a legitimate example and a dark-pattern example specifically so the distance between them is visible. When a design applies a principle in a way that fails any of the three questions, that is a **Blocker**, not a Warning — regardless of how effective it is.

---

## Legal Guardrail (Regulatory Backstop, Not the Ceiling)

Compliance with the law is the floor for this review, not the goal — but it is useful because it tells you where "persuasive" has been formally ruled to become "manipulative." Treat every item below as a hard constraint.

**FTC — "Bringing Dark Patterns to Light" (Sept. 2022 staff report).** The FTC's own framework names four dominant categories of dark pattern, all of which map directly onto principles in this file when misused: (1) disguised ads / mislabeled content, (2) obstructed cancellation or subscription traps (violates reciprocity/reversibility), (3) manipulated choice architecture — non-neutral defaults, confirmshaming copy, hidden costs revealed late (violates the default-effect and contrast-effect legitimate use), (4) exploited privacy via pre-checked consent boxes or buried settings (violates default-effect legitimate use directly). *Sourced from consistent secondary legal analysis of the report rather than a directly quoted primary excerpt — the framework itself is well-corroborated.*

**CCPA (California).** Effect-based, not intent-based — you cannot defend a dark pattern by saying you didn't mean to trick anyone.
- A "dark pattern" is statutorily defined by its **substantial effect** of subverting or impairing user autonomy, decision-making, or choice — intent is irrelevant. *(Moderate confidence — this specific statutory-definition claim had a split verification vote; treat as directionally correct but don't cite it as airtight in a legal dispute.)*
- Consent obtained through an interface that qualifies as a dark pattern is **legally void**, even if the user clicked "I agree."
- **Symmetry in choice is a codified requirement** (11 CCR §7004(a)(2)): the privacy-protective path cannot be longer or harder than the less-protective path. If "Reject All" takes more clicks than "Accept All," that is a Blocker, not a design preference.

**EU DSA (Digital Services Act), Article 25.** Prohibits dark patterns on online platforms — interfaces that deceive, manipulate, or materially distort a user's ability to make free, informed decisions. Note the carve-out: practices already covered by the GDPR or the Unfair Commercial Practices Directive are judged under those frameworks instead, not the DSA.

**EU AI Act, Articles 5(1)(a)–(b).** Functions as a de facto dark-pattern ban for any AI-driven interface, without using the term: prohibits subliminal, manipulative, or deceptive techniques, and prohibits exploiting vulnerabilities related to age, disability, or economic situation, when either causes significant harm. Directly relevant to any AI-personalized nudge, recommendation, or pricing feature.

If a finding in this review would also trip one of the above, escalate it to **Blocker** automatically, regardless of its severity under the psychology-only classification below.

---

## Severity Classification

**Blocker — must fix before marking done:**
- A default or pre-selected option benefits the business at the user's measurable expense (pre-checked paid add-on, opt-out data sharing, auto-renewal with no reminder)
- A scarcity, urgency, or social-proof signal that isn't true (fake countdown, inventory number that doesn't reflect reality, fabricated testimonial count)
- Confirmshaming copy on a decline/cancel option ("No thanks, I don't want to save money")
- Asymmetric friction: the path away from a choice (cancel, unsubscribe, opt out, downgrade) requires more steps, more time, or more cognitive effort than the path toward it
- A core user action has no viable trigger at a moment of adequate motivation and ability (Fogg B=MAP) — the user wants to act, is able to act, and the UI gives them no way to
- Anything that would independently trip the Legal Guardrail section above

**Critical — must fix before merge:**
- A multi-step flow (3+ steps) with no goal-gradient signal (progress indicator, step count, "almost there") where the friction of not having one is real and measurable
- A principle is applied with no plausible user benefit — it exists solely to move a business metric, even if it doesn't cross into a statutory dark pattern
- A field has an obvious, safe, inferable default (e.g., country from IP, most common answer from other users, last-used value) and forces the user to choose anyway
- A flow structurally denies the user any opportunity for ownership/investment (IKEA effect) before asking them to pay or commit, where competitors in the category succeed by building that investment first

**Warning — flag before done, fix before next deploy:**
- A principle is applied but weakly — e.g., contrast effect used only on price and not on the value it's attached to; an anchor with no real reference point behind it
- Copy invokes loss aversion truthfully but in a tone that reads as manipulative pressure rather than honest information
- A design leans on **Zeigarnik effect** or **choice overload** as if they were settled, reliable science — both have weak or null modern replication evidence (see entries below). Recommend hedging the design rationale or reaching for a better-evidenced principle instead.
- A legitimate default, streak, or progress mechanic exists but its "why" isn't visible to the user — transparency isn't required by the ethical test above, but it strengthens trust and is cheap to add

---

## Principle Reference

Each entry: mechanism, primary citation, confidence level, a legitimate application, a dark-pattern misuse to explicitly rule out, and a one-line trigger for when to reach for it.

### 1. Smart Defaults (Default Effect)
**Mechanism:** Presenting an option as the default produces a large, causal shift in what people end up with — largely independent of whether they'd have chosen it deliberately. Three sub-mechanisms drive it: **implicit endorsement** (users read the default as the choice architect's recommendation), **effort asymmetry** (accepting the default takes zero effort; changing it takes some), and **status-quo/loss framing** (once something is the baseline, deviating from it feels like giving something up).
**Citation:** Johnson & Goldstein, "Do Defaults Save Lives?" and the fuller working paper (2003/2004) — controlled online experiment (n=161) found opt-out framing produced statistically indistinguishable consent rates from a no-default neutral condition, both far above opt-in framing (82% vs. 79% vs. 42%).
**Confidence:** High — primary source, unanimous verification.
**Sub-mechanisms named separately, for reference:**
- *Loss aversion* — losses loom larger than equivalent gains. Canonical citation: Kahneman & Tversky, "Prospect Theory" (1979), *Econometrica* 47(2). Foundational and extremely well-replicated in the broader literature; not independently re-verified as a standalone claim in this file's research pass, but this is one of the most robust findings in behavioral economics.
- *Status quo bias* — people disproportionately stick with the current state even when switching costs are trivial. Canonical citation: Samuelson & Zeckhauser, "Status Quo Bias in Decision Making" (1988), *Journal of Risk and Uncertainty*. Same confidence caveat as above.
**Legitimate example:** Defaulting a new resume section order to the arrangement that performs best for most users in that role, while leaving it one click to reorder.
**Dark-pattern example:** Defaulting to "share my data with partners" = on, buried under a settings menu the user has no reason to visit.
**When to invoke:** Any choice where most users would pick the same option anyway, or where no-decision is itself harming the user (analysis paralysis on a low-stakes field).

### 2. Goal-Gradient Effect
**Mechanism:** Effort and frequency of action increase as subjective distance to a goal shrinks. Traces to Hull's 1932 animal-learning studies, resurrected for human consumer behavior.
**Citation:** Kivetz, Urminsky & Zheng (2006), *Journal of Marketing Research* 43(1).
**Confidence:** High for the core effect. **Caveat:** two frequently-repeated specifics from the secondary literature — a café loyalty-card field experiment, and the claim that "illusory/endowed progress" (progress the user didn't earn) accelerates completion just as well as real progress — were checked directly against the primary source and did **not** hold up under adversarial verification. Do not cite either of those specifics; cite only the core proximity effect.
**Legitimate example:** A visible "3 of 5 sections tailored" indicator that reflects real, user-driven progress.
**Dark-pattern example:** A progress bar seeded at 60% before the user has done anything, to manufacture false momentum.
**When to invoke:** Any flow with 3+ discrete steps toward a concrete, user-valued outcome.

### 3. Endowment Effect
**Mechanism:** People demand more to give something up (willingness-to-accept) than they'd pay to acquire the same thing (willingness-to-pay) — ownership itself inflates perceived value.
**Citation:** Kahneman, Knetsch & Thaler (1990), *Journal of Political Economy* 98(6) — the coffee-mug experimental markets study.
**Confidence:** High for the core WTA > WTP gap. **Caveat:** this is a live, unresolved academic debate, not settled fact — Plott & Zeiler (2005) argued the gap is a procedural/elicitation artifact rather than a real endowment effect; Fehr, Hakimov & Kübler (2015) failed to replicate that elimination and reaffirmed the original gap. Present it as "well-supported but contested in mechanism," not as uncontroversial. A separate claim that the effect "persists despite market experience/learning" was explicitly refuted in this file's research pass — don't cite that specific framing.
**Legitimate example:** Letting a user build a master resume once, then showing how much of it carries into every future tailored version — making the accumulated asset visible.
**Dark-pattern example:** Threatening deletion of a user's saved work to pressure an upgrade, when deletion isn't actually necessary.
**When to invoke:** Any point where a user has already invested real content, time, or configuration, and you want to reflect that investment back to them honestly.

### 4. IKEA Effect
**Mechanism:** Self-assembly or self-authorship increases valuation of the result — but only when the task is completed successfully. The effect dissipates if the creation is left unfinished or destroyed.
**Citation:** Norton, Mochon & Ariely (2011/2012), Harvard Business School WP 11-091 / *Journal of Consumer Psychology* — IKEA box-building and origami experiments.
**Confidence:** High — primary source, unanimous verification, and a 2026 meta-analysis treats the completion-boundary condition as established.
**Legitimate example:** Letting the user actively confirm/edit each AI-suggested change (rather than one-click "accept all"), so the final tailored resume feels authored, not generated.
**Dark-pattern example:** Forcing a lengthy setup wizard whose only real purpose is to manufacture sunk-cost attachment before revealing the price.
**When to invoke:** Any moment where letting the user do a small amount of real work themselves would increase their attachment to the outcome — and where that work is genuinely useful, not manufactured busywork.

### 5. Reciprocity
**Mechanism:** People feel obligated to return a favor or concession. The best-evidenced version in this file is the **door-in-the-face** effect: an extreme initial request that gets refused, followed by a smaller one, roughly doubles compliance with the smaller request — driven by a felt concession, not by simple perceptual contrast (the research explicitly rules out contrast as the mechanism here).
**Citation:** Cialdini et al. (1975), *Journal of Personality and Social Psychology* 31(2).
**Confidence:** High for the core effect. **Caveat:** a specific control condition (two-requester or equivalent-request control eliminating the effect) was checked and did not hold up — don't cite that detail.
**Legitimate example:** Giving away a genuinely useful free tool (e.g., a resume health check) before ever asking for an account.
**Dark-pattern example:** A fake "we did you a favor" framing ("we already unlocked this for you!") to create obligation for something the user never asked for and doesn't need.
**When to invoke:** Early in a relationship with a new user, before any ask — and only where the "favor" is real and unconditional.

### 6. Contrast Effect (Compromise / Decoy Effect)
**Mechanism:** The same option is judged more or less attractive depending on what it's compared against — an intermediate ("compromise") option looks more attractive with an extreme option nearby, and adding a third, asymmetrically-dominated ("decoy") option can shift preference toward the option it resembles.
**Citation:** Simon & Tversky (1992) on extremeness aversion and background contrast effects; Huber, Payne & Puto (1982), *Journal of Consumer Research* 9(1), on asymmetric dominance; replicated with concrete numbers by Ariely & Hayout (preferences shifted from 40%/60% to 56%/8%/36% after adding a decoy).
**Confidence:** High for the empirical direction of the effect (decoys and background comparisons do shift preference toward the similar/dominant option). **Caveat:** the precise theoretical mechanism (a specific "similarity hypothesis" framing tested in the primary source) did not hold up in this file's verification pass — cite the empirical shift, not a specific claimed mechanism for why it happens.
**Legitimate example:** A pricing page where the middle tier is genuinely the best value for most users, and the layout makes that comparison easy rather than obscuring it.
**Dark-pattern example:** A "decoy" tier priced to make an inferior option look artificially good by comparison, when it isn't actually a reasonable choice for anyone.
**When to invoke:** Any page presenting 2+ comparable options (pricing, plan tiers, template choices) where relative framing, not just absolute value, will shape the decision.

### 7. BJ Fogg's Behavior Model (B = MAP)
**Mechanism:** A behavior occurs only when Motivation, Ability, and a Prompt/Trigger converge at the same moment — missing any one of the three, the behavior doesn't happen, however strong the others are. Triggers are not interchangeable: a **spark** boosts motivation for a low-ability moment, a **facilitator** increases ability/ease for a high-motivation moment, and a **signal** just reminds the user when both motivation and ability are already high.
**Citation:** Fogg, "A Behavior Model for Persuasive Design" (2009), Persuasive Technology conference proceedings (the Captology paper).
**Confidence:** High — primary source, unanimous verification. Known limitation (not a refutation): critics note the model oversimplifies motivation and underweights habit/environment as ongoing forces, not a one-time trigger event.
**Legitimate example:** Sending a reminder (signal) to finish a tailored resume only when the user has already shown motivation (opened the job posting) and the remaining task is genuinely quick (high ability).
**Dark-pattern example:** A push notification (signal) sent at a moment of low motivation and low ability, engineered purely to interrupt and re-engage regardless of relevance.
**When to invoke:** Diagnosing *why* a desired action isn't happening — is it missing motivation, missing ability (too hard/slow), or missing a trigger at the right moment? Fix the actual missing ingredient, not the other two.

### 8. Anchoring
**Mechanism:** An initial number, even an arbitrary one, systematically biases subsequent numeric judgments and estimates in its direction.
**Citation:** Garcia-Marques et al. (2023), *Brain and Behavior* — a controlled psychophysical proportion-estimation study. Pair with the canonical Tversky & Kahneman (1974) "wheel of fortune" anchoring study for standard UX framing; that classic study is the field's standard citation but was not independently re-verified in this file's research pass.
**Confidence:** Medium — the directly verified source is a narrower psychophysical study, not a general real-world pricing/donation-anchoring demonstration. A more specific "dual-mechanism" (bias + reduced sensitivity) account from the same paper was checked and refuted — don't cite that detail.
**Legitimate example:** Showing the original list price next to a genuine discount, so the user has an honest reference point for the value of the deal.
**Dark-pattern example:** A fabricated "was $199" struck through a price that was never actually $199.
**When to invoke:** Anywhere a user needs to judge whether a number (a price, a time estimate, a score) is good or bad, and a truthful reference point would help them judge it correctly.

### 9. Hick's Law
**Mechanism:** Decision time increases with the number and complexity of available choices, following a roughly logarithmic relationship to the amount of information (in bits) those choices represent.
**Citation:** Hick (1952), *Quarterly Journal of Experimental Psychology* — the original reaction-time/information-theory paper.
**Confidence:** High — primary source, unanimous verification.
**Legitimate example:** Narrowing a template picker to 3 curated options with a clear "see more" escape hatch, instead of showing all 20 upfront.
**Dark-pattern example:** Deliberately overloading a cancellation flow with irrelevant options and reasons-for-leaving surveys to slow the user down before they can actually cancel.
**When to invoke:** Any screen where the number of choices presented at once might itself be the friction, independent of how good any individual choice is.

### 10. Zeigarnik Effect — use with caution
**Mechanism (as commonly cited):** Unfinished or interrupted tasks are remembered better and create a felt tension toward completion.
**Citation:** Zeigarnik (1927) — the original study did find a large memory advantage for interrupted tasks (in the original samples, 90–110% better recall), and explicitly ruled out "interruption shock" as an alternative explanation.
**Confidence: Low as a reliable, generalizable UX lever.** A 2025 meta-analysis of 38 published studies found **no overall memory advantage** for unfinished tasks (weighted ratio 0.99 — essentially equal recall), and the effect is strongly moderated by mood/condition: a slight positive effect under relaxed conditions (1.07), no effect under neutral conditions (0.96), and a *reversed* effect under achievement-pressure conditions (0.88). Across the 8 studies with usable effect sizes, the average was small (d = 0.15).
**What this means for design:** The famous "progress bars and unfinished checklists nag at people" story is much weaker than its reputation. Don't present it in a design rationale as settled science. If you want a progress mechanic, lean on the goal-gradient effect (Principle 2) instead — that one has solid, unrefuted support — rather than justifying it via Zeigarnik.
**Legitimate example (if used at all):** A gently visible "resume 90% tailored" state the user can return to, framed as helpful continuity rather than as manufactured psychological pressure.
**Dark-pattern example:** An artificial "unfinished" badge designed purely to create anxiety and pull the user back in, unrelated to any real incomplete task.
**When to invoke:** Rarely, and only as a mild secondary justification alongside a better-evidenced principle — not as the primary rationale for a design decision.

### 11. Choice Overload / "Paradox of Choice" — use with caution
**Mechanism (as commonly cited):** More options reduce the likelihood of choosing at all, or reduce satisfaction with the choice made (popularized by the Iyengar & Lepper 2000 jam-tasting study).
**Confidence: Low as a general, reliable effect.** A meta-analysis across 63 experimental conditions from 50 published and unpublished studies (N = 5,036) found a **mean effect size of virtually zero** and not statistically significant — with considerable variance between individual studies. This is genuinely contested: a 2025 economics paper (Dean, Ravindran & Stoye) argues the null result is a statistical-power artifact of underpowered test designs, and reports a more powerful test that does find choice-overload effects in new data.
**What this means for design:** Do not treat "reduce the number of options" as a universal conversion lever backed by settled science — the evidence is genuinely split. If a specific screen seems to suffer from too many options, that's a legitimate hypothesis worth testing locally (and Hick's Law, Principle 9, gives you a solid decision-time mechanism to reason from), but don't cite "choice overload" as if the underlying phenomenon itself were proven.
**Legitimate example:** Testing a curated 3-template default view against a full 20-template gallery for a specific user segment, and measuring the actual outcome rather than assuming the smaller set wins.
**Dark-pattern example:** Artificially hiding cheaper or more suitable options behind extra clicks, justified internally as "reducing choice overload," when the real purpose is steering users toward a specific paid option.
**When to invoke:** As a hypothesis to test, not a rule to apply — and only alongside real usage data.

### 12. Peak-End Rule
**Mechanism:** People judge an experience largely by how it felt at its most intense moment (peak) and at its end, not by the average of the whole experience or its duration.
**Citation:** Redelmeier & Kahneman (1996) colonoscopy-duration studies; Fredrickson & Kahneman (1993) "duration neglect" work.
**Confidence:** Well-established in the literature — this is one of the more frequently replicated findings in the experienced-utility research program. **Not independently re-verified in this file's research pass** (source access was rate-limited before verification completed) — treat as a solid textbook citation rather than a freshly adversarially-checked claim here.
**Legitimate example:** Making sure the moment right after a successful resume tailoring (the "peak") and the final screen of a session (the "end") both feel genuinely good, even if a step in the middle was tedious.
**Dark-pattern example:** Engineering an artificially delightful cancellation confirmation screen specifically to leave a good final impression that papers over a frustrating cancellation process itself.
**When to invoke:** Designing the emotional arc of a multi-step flow — identify what the peak moment and the end moment currently feel like, independent of the average experience.

### 13. Social Proof
**Mechanism:** People use others' behavior as a signal for their own correct behavior, especially under uncertainty. Descriptive norms (what most people actually do) tend to outperform injunctive norms (what people should do) at changing behavior.
**Citation:** Goldstein, Cialdini & Griskevicius (2008), *Journal of Consumer Research* — hotel towel-reuse field experiments comparing message framings.
**Confidence:** Well-established, frequently replicated in field settings. **Not independently re-verified in this file's research pass** — treat as a solid textbook citation rather than freshly adversarially-checked here.
**Legitimate example:** "87% of PMs in your industry lead with quantified impact in their first bullet" — a true, specific, relevant statistic shown at the moment it's actionable.
**Dark-pattern example:** A fabricated "12 people are viewing this plan right now" counter with no real underlying data.
**When to invoke:** A decision point where the user is uncertain what "normal" or "good" looks like, and a true, specific comparison would resolve that uncertainty.

### 14. Scarcity / Urgency
**Mechanism:** Perceived value increases when something is, or appears to be, limited in availability or time.
**Citation:** Worchel, Lee & Adewole (1975) — the cookie-jar scarcity study is the standard citation in this literature.
**Confidence:** The underlying psychological effect is well-established, but **this file's research pass could not access a fully verifiable primary source this session** (the sources found were flagged low-quality on fetch) — treat this citation as the standard textbook reference rather than a freshly re-verified one, and hold scarcity claims to an especially high truthfulness bar in practice (see the ethical test above) since this is the single most commonly abused principle in dark-pattern design.
**Legitimate example:** A genuine limited-time launch discount with an honest, fixed end date that doesn't move.
**Dark-pattern example:** A countdown timer that resets when the page is reloaded, or an "only 2 left" label on a digital product with unlimited supply.
**When to invoke:** Only when the scarcity or time limit is real — never manufactured to create pressure alone.

---

## Step 0: Scope Check

Before reviewing, identify:
1. **Mode** — is this a generative pass (designing something not yet built, e.g. inside `design-brief.md`) or an evaluative pass (auditing something already built, e.g. inside `design-qa.md` or standalone via `/ux-psychology`)?
2. **What flow or screen is in scope?** Name the specific user journey — a full audit of "the whole app" is too broad to produce sharp findings; scope to a flow (onboarding, upgrade, a specific screen).
3. **What is the user trying to accomplish here, and what does the business want them to do?** When these two answers diverge, that divergence is exactly where dark-pattern risk concentrates — flag it explicitly before going further.

## Step 1: Run the Principle Pass

For the flow in scope, go through the 14 principles above and, for each one, answer:
- **Is this principle currently applied, missing, or misapplied?**
- If missing: would applying it (per its legitimate example) measurably reduce real friction, or would it be decoration with no user benefit? Only flag it if the former.
- If applied: does it pass the three-question ethical test? If not, classify per the Legal Guardrail and Severity Classification above.

Do not force all 14 principles onto every flow — most flows will only implicate 3–5 of them. A finding is only worth reporting if it's specific to this flow, not a generic restatement of the principle description.

## Step 2: Classify and Report

Every finding gets exactly one severity: **Blocker**, **Critical**, or **Warning**, using the definitions above. State which principle it relates to, the specific location (screen/component), and — for a generative pass — the specific recommended change; for an evaluative pass — the specific fix.

## Step 3: Sign-off

Output this summary:

**Mode**: [generative / evaluative]
**Flow(s) reviewed**: [name]
**Principles applied or considered**: [list only the ones actually implicated]

**Blockers**: [none / list — principle, location, why]
**Critical findings**: [none / list — principle, location, why]
**Warnings**: [none / list — principle, location, why]

**Ethical check**: [pass / list anything that failed the three-question test]
**Legal guardrail check**: [pass / list anything that would trip CCPA, DSA, or EU AI Act as described above]

Mark done only when all blockers are resolved. A design that converts better because it's confusing, asymmetric, or dishonest is not a win — it's a liability with a delay on it.

---

## Guiding Principles

- Every principle in this file is a tool for reducing real friction a real user feels — not a menu of tricks to increase a metric. If you can't name the user benefit, don't ship it.
- Evidence quality varies a lot across this list. Say so. A design rationale that leans on Zeigarnik or choice overload as if they were bulletproof is building on sand — prefer the high-confidence principles (defaults, goal-gradient, endowment, IKEA, reciprocity, contrast, Fogg's model, Hick's Law) when you have a choice of which lever to pull.
- The three-question ethical test and the legal guardrail are not a separate compliance chore bolted onto the "real" review — they are the review. A principle applied dishonestly or asymmetrically is a UX bug, not a growth win.
- When in doubt about whether something is manipulative, ask: would this survive the user finding out exactly how it works? If not, it's a Blocker.
