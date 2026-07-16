# Legal Review Rules

## Purpose
Run this review before launch and before any feature that collects, stores, processes, or shares user data. The goal is to catch liability exposure, missing disclosures, and compliance gaps before real users are affected.

This review does not replace a lawyer. It catches the obvious, high-frequency gaps that solo builders routinely miss. When a finding is marked BLOCKED, consult legal counsel before proceeding.

Priority order: blockers must be resolved before launch. Critical findings must be resolved before the feature goes live. Warnings must be flagged and scheduled.

Invoke explicitly: before any product launch, before adding any new data collection, and before any feature that uses AI to process user-submitted content.

---

## Severity Classification

**Blocker — do not launch, fix immediately:**
- No privacy policy exists and the product collects any personal data (email = personal data)
- No terms of service exists and users are creating accounts or paying
- Personal data is being collected or processed with no identified lawful basis under GDPR (for any product accessible to EU/UK users)
- Personal data is being transferred to a country without adequate data protection and no legal basis exists for the transfer
- User data is being sold or shared with third parties in ways not disclosed to the user
- A data breach has occurred and affected users have not been notified within the required window (72 hours under GDPR, varies by state under US law)
- Payment card data (full card numbers, CVV) is being stored — PCI DSS violation regardless of scale
- Product may be used by children under 13 with no minimum age statement in the ToS and no mechanism to prevent or handle underage signups (COPPA — FTC fines are per-violation per-day regardless of company size)

**Critical — must resolve before the feature goes live:**
- Privacy policy exists but does not disclose a data type that is actually being collected
- No cookie consent mechanism exists and the product sets non-essential cookies for users in the EU or UK
- AI is processing user-submitted content and no disclosure exists in the privacy policy
- Transactional emails are being sent without a physical mailing address in the footer (CAN-SPAM violation)
- Marketing emails are being sent without an unsubscribe link (CAN-SPAM violation, criminal exposure in Canada under CASL)
- User data is being shared with a third-party service (analytics, error tracking, AI) not listed in the privacy policy
- No mechanism exists for users to request deletion of their account and data (required under GDPR and CCPA)
- Users can upload content and no DMCA takedown policy exists and no DMCA agent has been registered

**Warning — flag before launch, schedule before next release:**
- Privacy policy or terms of service has not been reviewed since a significant feature change
- Data retention period is not defined — data is kept indefinitely with no deletion schedule
- No Data Processing Agreement (DPA) with vendors who process EU user data on your behalf
- Subscription or billing terms are unclear on renewal, cancellation, and refund conditions
- No process defined for handling a Subject Access Request (SAR) — a user asking what data you hold on them
- Product has significant users in US states beyond California that now have comprehensive privacy laws (Virginia, Colorado, Connecticut, Texas, Florida, Oregon, Montana — check user geography)

---

## Step 0: Scope Check

Before beginning, answer these five questions:

1. **Who are your users?** US only, EU/UK included, global, or unknown? EU/UK users trigger GDPR. California users trigger CCPA. Canada users trigger CASL for email. Six additional US states now have comprehensive privacy laws beyond California — if you have material users in Virginia, Colorado, Connecticut, Texas, Florida, or Oregon, flag for a state-by-state check.
2. **What personal data does the product collect?** At minimum: email address at signup. Also check: name, payment info, location, usage behavior, IP address, device identifiers, user-generated content.
3. **Does the product use AI to process user-submitted content?** If yes: Anthropic's usage policies, the EU AI Act (for EU users), and your privacy policy must all address this.
4. **Is the product monetized?** Free tools have lighter obligations than products that charge users.
5. **Can users upload or submit content that could include third-party copyrighted material?** If yes: DMCA safe harbor applies.

Document the answers. Every finding below is interpreted in light of this scope.

---

## Step 1: Data Collection Audit

Before checking documents, map what the product actually collects. Documents can only be assessed against reality.

**Identify every data type collected:**
- [ ] Account data: email, name, password (hashed), profile fields
- [ ] Payment data: handled by Stripe or similar (confirm you never touch raw card data)
- [ ] Usage data: pages visited, features used, actions taken, timestamps
- [ ] Device and network data: IP address, browser, OS, referrer URL
- [ ] User-generated content: text inputs, uploads, prompts sent to AI
- [ ] Third-party data: anything passed in from OAuth providers (Google, GitHub)

**For each data type, answer:**
- Why is it collected? (business reason)
- Where is it stored? (Supabase table, third-party service)
- Who can access it? (your team, vendors, is it subject to RLS)
- How long is it kept?
- Is it shared with any third party?

If any data type cannot be answered on all five dimensions: flag it. Unknown data handling is a compliance gap.

**Third-party services that receive user data — check each one:**
- Supabase (database host): review their DPA at supabase.com/privacy
- Vercel (hosting, logs): review their DPA; server logs may contain IP addresses
- Anthropic (AI API): if user content is sent to the API, review Anthropic's usage policy — confirm whether inputs are used for model training and disclose this to users
- Analytics (Vercel Analytics, PostHog, Mixpanel, etc.): disclose in privacy policy
- Error tracking (Sentry, etc.): confirm PII is not captured in error payloads; disclose in privacy policy
- Email (Resend, SendGrid, etc.): user email addresses are shared with this vendor; disclose in privacy policy

---

## Step 2: GDPR Lawful Basis

Run this step for any product accessible to EU or UK users. Skip only if you have verified your product is blocked from EU/UK access by geography.

Under GDPR, disclosing what you collect is not sufficient. You must have a documented legal basis for processing each category of personal data. Processing without a lawful basis is a GDPR violation regardless of how accurate your privacy policy is.

**For each data type identified in Step 1, identify and document the lawful basis:**

| Lawful Basis | When it applies |
|---|---|
| **Contract** | Data is necessary to deliver the service the user signed up for. Email for account creation qualifies. |
| **Consent** | User has freely given, specific, informed, and unambiguous opt-in. Required for marketing emails, non-essential cookies, and any processing beyond what's needed to deliver the service. |
| **Legitimate interests** | Processing serves a genuine business interest that does not override user rights. Analytics, security logging, and fraud prevention typically qualify — but you must conduct and document a balancing test. |
| **Legal obligation** | Processing is required by law (tax records, court orders). |
| **Vital interests** | Life-or-death situations. Rarely applicable to SaaS. |

**Checklist:**
- [ ] Every category of personal data has an identified lawful basis documented in or alongside the privacy policy
- [ ] Consent-based processing has a valid consent record — not assumed, not bundled into ToS acceptance, not pre-ticked
- [ ] Legitimate interests processing has a documented balancing test (brief written record that the business interest outweighs user rights impact)
- [ ] Processing for analytics or behavioral tracking is not relying on "contract" as its basis — this is a common invalid claim; analytics typically requires consent or a documented legitimate interests test
- [ ] If consent is withdrawn, the processing stops and the data is deleted or anonymized within a reasonable window
- [ ] Users are informed of their lawful basis in the privacy policy for each processing category

**Common mistakes to check:**
- Using "contract" as the basis for analytics, marketing, or profiling — these do not qualify
- Using ToS acceptance as a proxy for GDPR consent — these are legally separate acts
- Claiming legitimate interests without a balancing test on file

---

## Step 3: Legal Documents Check

**Privacy Policy — required if you collect any personal data:**
- [ ] Privacy policy exists and is publicly accessible without logging in
- [ ] It is linked in the footer of every page and at the point of data collection (signup form)
- [ ] It lists every category of data collected (must match the audit in Step 1)
- [ ] It lists every third-party service that receives user data
- [ ] It states the purpose and lawful basis for each data type collected (GDPR)
- [ ] It states how long data is retained (or the criteria used to determine retention)
- [ ] It describes user rights: access, correction, deletion, portability, restriction, objection
- [ ] It states how to contact you with a data request or complaint (email address is sufficient)
- [ ] It includes an effective date and is updated when data practices change
- [ ] If AI processes user content: it discloses this explicitly, including which provider and whether content may be used for training

**Terms of Service — required if users create accounts or pay:**
- [ ] Terms of service exists and is publicly accessible
- [ ] It is linked at signup and in the footer
- [ ] Users must affirmatively accept it (checkbox or "by continuing you agree" at signup) — not assumed by using the product
- [ ] It defines what the product is and what it is not (scope of service)
- [ ] It includes an acceptable use policy — what users may not do
- [ ] It limits your liability (limitation of liability clause)
- [ ] It states governing law and jurisdiction
- [ ] If subscription: renewal terms, cancellation policy, and refund policy are stated clearly
- [ ] It includes IP ownership: who owns user-generated content, and a license grant allowing you to store and display it
- [ ] It includes a warranty disclaimer — product is provided "as is" with no implied warranties
- [ ] It includes an indemnification clause — user covers you for damages caused by their misuse of the product
- [ ] It includes a dispute resolution clause — arbitration or small claims, governing jurisdiction, class action waiver
- [ ] It states how terms will be updated — notice method (email, in-app), and that continued use after notice constitutes acceptance
- [ ] It states when and why you can terminate or suspend an account, and what happens to user data on termination
- [ ] It includes a force majeure clause covering outages and failures outside your control
- [ ] It includes a severability clause — if one provision is invalid, the rest of the terms survive

**Cookie Notice — required for EU/UK users if non-essential cookies are set:**
- [ ] If the product sets analytics, tracking, or advertising cookies: a cookie consent banner is present for EU/UK users
- [ ] The banner does not pre-check non-essential cookies — consent must be opt-in, not opt-out
- [ ] A link to the cookie policy or privacy policy is present in the banner
- [ ] Essential cookies (session, auth) do not require consent — do not ask for it separately

**DMCA Safe Harbor — required if users can upload or submit content:**
- [ ] A DMCA agent has been registered with the US Copyright Office (one-time $6 filing at copyright.gov/dmca-agent) — without this, the safe harbor does not apply
- [ ] A DMCA takedown policy is published in the Terms of Service or at a dedicated `/dmca` URL
- [ ] The policy states: how to submit a valid takedown notice, what information is required, and what happens when one is received
- [ ] A process exists internally to respond to valid takedown notices promptly (within 2 weeks at most)
- [ ] A counter-notice process exists for users who believe their content was wrongly removed

---

## Step 4: User Rights and Consent

**Account deletion and data erasure:**
- [ ] A mechanism exists for users to delete their own account
- [ ] Deleting an account triggers deletion (or anonymization) of personal data — not just a soft delete that keeps the data
- [ ] Deletion cascades to Supabase tables holding user data (verify the RLS and cascade rules)
- [ ] A process exists for responding to manual deletion requests sent by email within 30 days

**Data access and portability:**
- [ ] A process exists for responding to a user who asks "what data do you have on me?"
- [ ] If technically feasible: users can export their own data

**Marketing email consent:**
- [ ] Users have explicitly opted in to marketing emails — not pre-checked, not assumed from account creation
- [ ] Transactional emails (receipts, password resets, account notifications) do not require separate opt-in, but must not contain marketing content beyond the transaction
- [ ] Every marketing email contains: sender name and physical address, clear subject line, one-click unsubscribe link
- [ ] Unsubscribe requests are processed within 10 business days (CAN-SPAM requirement)

---

## Step 5: AI-Specific Compliance

Run this step whenever the product sends user-submitted content to an AI API (Anthropic, OpenAI, or any other).

**Anthropic API usage:**
- [ ] Review Anthropic's current usage policies — confirm whether inputs are used for model training by default and whether this can be opted out
- [ ] If user content (prompts, documents, messages) is sent to the API: the privacy policy discloses this
- [ ] If sensitive user data (health information, financial data, personal communications) is sent to the AI: assess whether this creates additional obligations (HIPAA if health data, financial regulations if financial data)
- [ ] Users are not misled about whether they are interacting with AI — if the product presents an AI persona, it must not deny being AI when sincerely asked (required under EU AI Act and Anthropic's usage policy)

**EU AI Act — for products accessible to EU users:**
- [ ] The product does not use AI for any prohibited practice: social scoring, subliminal manipulation, real-time biometric surveillance in public spaces, or exploitation of vulnerable groups
- [ ] Users are clearly informed when they are interacting with AI-generated content or an AI system
- [ ] If the product makes automated decisions that significantly affect users (credit, hiring, access to services): users have a right to human review — flag for legal counsel if applicable
- [ ] If the product could be classified as a high-risk AI system under the EU AI Act (healthcare, employment, education, law enforcement contexts): flag immediately for legal counsel before launch

**AI-generated content:**
- [ ] If the product generates content that users could interpret as factual: include a disclaimer about AI accuracy
- [ ] If the product generates content in the user's voice (ghostwriting, drafts): the terms address IP ownership — note that AI-generated content may not be independently copyrightable under current case law
- [ ] If user content is used to train or fine-tune any model: this requires explicit, informed consent separate from the general ToS

---

## Step 6: Breach Response Process

A data breach is a security incident that results in unauthorized access to, disclosure of, or loss of personal data. This step defines what to do — not just when to notify, but the full response sequence.

**Immediate triage (within hours of discovery):**
- [ ] Stop the active breach if possible — revoke credentials, disable endpoints, isolate affected systems
- [ ] Preserve evidence — do not delete logs or overwrite affected systems before documenting the state
- [ ] Identify the scope: what data was affected, how many users, what time window, which systems
- [ ] Determine whether personal data was actually accessed or only potentially exposed — this affects notification obligations

**Notification obligations — assess within 24 hours:**

| Jurisdiction | Who to notify | Deadline | When required |
|---|---|---|---|
| EU/UK (GDPR) | Lead supervisory authority in the EU member state where you're established, or the ICO for UK | 72 hours from discovery | When personal data is affected and the breach is likely to result in risk to individuals |
| EU/UK (GDPR) | Affected users | Without undue delay | When breach is likely to result in high risk to individuals (identity theft, financial loss, discrimination) |
| US — all states | Affected users | Varies by state (30–90 days typical; some states: without unreasonable delay) | When sensitive personal data is compromised |
| California (CCPA) | California AG if 500+ California residents affected | Expeditiously | As above |

- [ ] Identify which jurisdictions are triggered based on affected users' locations
- [ ] Draft notification to affected users: what happened, what data was involved, what you are doing, what users should do, contact information for questions
- [ ] If GDPR applies: prepare regulatory notification with incident timeline, data categories affected, estimated number of individuals, likely consequences, and measures taken
- [ ] Do not speculate in notifications — only disclose what is confirmed

**After containment:**
- [ ] Document the full incident timeline in writing — what happened, when it was discovered, what was done
- [ ] Conduct a root cause analysis — what vulnerability or process failure caused the breach
- [ ] Implement remediation — fix the root cause, not just the symptom
- [ ] Review and update security practices based on findings

---

## Step 7: Payment and Subscription Compliance

Run this step if the product charges users.

**Billing transparency:**
- [ ] Price is clearly displayed before payment is captured — no hidden fees revealed at checkout
- [ ] Recurring billing (subscription) is clearly labeled as recurring, not one-time
- [ ] The billing cycle (monthly/annual) and renewal date are communicated at signup and in receipts
- [ ] Cancellation instructions are easy to find — not buried or requiring a support ticket

**Refunds:**
- [ ] Refund policy is stated in the terms of service
- [ ] Policy complies with the app store rules if distributed through an app store (Apple/Google handle refunds directly for digital goods purchased through their stores)

**PCI DSS:**
- [ ] Raw card data (full card number, CVV, expiry) is never stored or logged — Stripe or a certified processor handles this entirely
- [ ] Confirm with `grep -r "card_number\|cvv\|cvc\|cardNumber" .` that no card fields are being captured directly

---

## Step 8: Pre-Launch Legal Checklist

Run this step before any public launch (beta counts as public). Pre-launch email collection (waitlists) also triggers CAN-SPAM and GDPR — do not wait until launch to comply.

- [ ] Privacy policy is live at a stable URL (e.g., `/privacy`)
- [ ] Terms of service is live at a stable URL (e.g., `/terms`)
- [ ] Both documents are linked in the footer of every page
- [ ] Both documents are linked at the point of account creation
- [ ] Cookie consent is implemented if applicable (EU/UK users + non-essential cookies)
- [ ] Data deletion mechanism is functional and tested
- [ ] No third-party service receives user data that is not disclosed in the privacy policy
- [ ] A contact email exists for legal/privacy inquiries and is listed in the privacy policy
- [ ] If using AI to process user content: disclosed in privacy policy
- [ ] If users can upload content: DMCA agent registered and takedown policy published
- [ ] GDPR lawful basis documented for each data category (Step 2 complete)
- [ ] Minimum age stated in ToS; underage signup handling defined
- [ ] If collecting emails pre-launch (waitlist): CAN-SPAM and GDPR consent requirements already met

**Before handling real payment data:**
- [ ] Stripe (or equivalent) is the only system that touches card data
- [ ] Webhook signatures are verified before processing Stripe events

---

## Step 9: Legal Sign-off

Before marking done or proceeding with launch, output this summary:

**Review scope**: [pre-launch / new data collection / AI feature / payment feature]
**User geography**: [US only / EU/UK included / global / unknown — list states with material user bases]
**Data collected**: [list categories identified in Step 1]
**AI processing**: [yes — provider and disclosure status / no]
**Monetized**: [yes / no]
**User-generated content**: [yes — DMCA status / no]

**Privacy policy**: [live and complete / missing / incomplete — list gaps]
**Terms of service**: [live and complete / missing / incomplete — list gaps]
**Cookie consent**: [in place / not applicable / missing]
**GDPR lawful basis**: [documented for all data categories / not applicable / gaps — list]
**DMCA safe harbor**: [agent registered and policy published / not applicable / missing]
**User deletion mechanism**: [functional / missing]
**Marketing email compliance**: [pass / issues — list]
**AI disclosure and EU AI Act**: [pass / not applicable / issues — list]
**Breach response process**: [defined / not yet defined]
**Payment compliance**: [pass / not applicable / issues — list]
**Pre-launch checklist**: [all pass / items outstanding — list]

**Blockers**: [none / list — do not launch until resolved]
**Critical findings**: [none / list — resolve before feature goes live]
**Warnings**: [none / list — schedule before next release]

Do not launch until all blockers are resolved. Do not treat a blocker as a warning because the product is small or early — liability does not scale with company size.

---

## Guiding Principles

- Collecting an email address means you are handling personal data. The obligations start there, not at scale.
- A privacy policy that does not match what the product actually does is worse than no privacy policy — it is a false statement to your users.
- GDPR applies if any EU or UK resident uses your product, regardless of where your company is incorporated. Disclosure alone is not compliance — lawful basis is required.
- US privacy law is no longer just California. Check your user geography against the expanding state-level landscape before each launch.
- "We'll fix it after launch" is how legal debt becomes legal liability. The earlier a gap is caught, the cheaper it is.
- This review does not replace a lawyer. It catches the obvious gaps. If a finding is BLOCKED, get qualified legal advice before proceeding.
