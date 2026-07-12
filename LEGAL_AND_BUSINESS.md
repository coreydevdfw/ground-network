# Ground Network — Legal & Business State
_Companion to PROJECT_STATE.md. This file is the durable record of legal/business decisions, open questions, and compliance-relevant backlog — committed and versioned so it survives across sessions, not dependent on chat memory._

## Current legal posture (as of 2026-07-11)
- **Entity:** None formed yet. Operating as an individual. No business name, EIN, or liability shield in place.
- **Legal docs on the live site:** None. No Privacy Policy, no Terms of Service, no acceptable-use/EEO policy, no age requirement stated anywhere.
- **Placement fee:** Advertised in the "Post a job" UI ("Free to post. You only pay a placement fee if you hire someone through the app.") but **not implemented** — no payment processing, no invoicing, no Stripe/payment processor integration. Hire status is a self-reported dropdown only (Contacted / Interviewing / Hired / Not a fit) with zero enforcement.
- **Employer verification:** Peer-vouch system only (2 vouches from other employers = "verified" badge). No EIN, business license, or business-email-domain check.
- **Applicant PII collected:** Firestore `applications` collection stores `applicantName`, `applicantEmail`, `applicantPhone` for every direct-posting application, visible to the employer. No privacy policy currently discloses this.
- **Geolocation:** Custom "Share your location for a tighter radius" prompt exists in the UI, but no privacy disclosure backs it.
- **Regulatory research done:** Texas Workforce Commission guidance (public, not a legal opinion) indicates Texas does not require an employment-agency license as long as job seekers are never charged — Ground Network's model charges the employer, not the seeker, which is the permissive side of that rule. This has NOT been confirmed directly with TWC or an attorney, and other states may regulate fee-charging agencies regardless of which side pays — must be re-checked before charging fees in any market beyond Texas.

## Parked / backlog — legal-adjacent features (not yet built, intentionally deferred)
These were discussed 2026-07-11 and explicitly parked for later rather than built immediately:
1. **Privacy Policy** — needs to cover applicant PII (name/email/phone), geolocation use, Firebase/Firestore/GA4 data handling, and Texas Data Privacy and Security Act applicability (thresholds not yet checked).
2. **Job scam / fraud reporting** — no mechanism currently exists for a job seeker to report a fraudulent or suspicious posting. The only existing "flag" concept in the code is unrelated (stale-listing detection based on post age, not abuse reporting).
3. **Account & data deletion — both sides:**
   - Employer side: delete employer account + associated `employers` doc + owned `jobPostings`.
   - Job seeker side: delete applicant PII from the `applications` collection (name/email/phone) tied to a specific application, and/or full account deletion if seeker accounts exist beyond localStorage-tracked "applied" state.
   - This is baseline expectation under most current state privacy laws and should be built as a real, working data-deletion path, not just a policy statement.

## Open decisions in progress
- **Business entity formation** (sole prop vs. LLC) — leaning toward forming an LLC before real revenue starts rather than after, but not yet decided/filed.
- **Placement fee mechanics** — actively being hashed out as of 2026-07-11 (see below / next session update for outcome). Key open sub-decisions: fee structure (flat vs. % of pay vs. tiered), hire-confirmation/anti-fraud mechanism (employer self-report only vs. dual employer+seeker confirmation), guarantee/refund period (none vs. 30/90-day), and payment collection approach (Stripe invoicing vs. card-on-file auto-charge).

## Why this file exists
Same rationale as PROJECT_STATE.md: chat memory across sessions is unreliable and has previously lagged real progress. Legal and business decisions are high-stakes enough that they need their own durable, committed, versioned record — separate from the engineering/technical log in PROJECT_STATE.md — so any future session (or an actual attorney/accountant brought in later) can see exactly what's been decided, what's been deferred on purpose, and what's still genuinely open.
