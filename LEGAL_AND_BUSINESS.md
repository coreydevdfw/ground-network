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
- **Placement fee mechanics** — decided 2026-07-11, see dedicated section below. Still open: exact flat fee dollar amount (leaning $250-500 to start) and final Stripe/payment integration work.

## Placement fee mechanics (decided 2026-07-11)
Design finalized in conversation; not yet built. Captured here so implementation can happen in a later session without re-deriving the design.

- **Fee structure:** Flat fee per hire (not a percentage of pay, not tiered by role — flat fits the hourly/warehouse-logistics center of gravity of current job categories). Amount not finalized; leaning toward the $250-$500 range to start. Must be implemented as a configurable value, not hardcoded, so it can be adjusted without a code deploy.
- **Trigger / anti-fraud mechanism:** Dual confirmation. Employer marks an applicant "Hired" (existing dropdown) — this alone does NOT trigger the fee, it moves the application into a pending-confirmation state. The seeker is then asked (in-app / email) "Did you get this job at [Employer]?" The fee is only triggered once the seeker independently confirms. A seeker "no" or a non-response past some cutoff means no fee is charged. An employer-says-hired / seeker-says-no mismatch is a manual review flag (at current volume: a notification to Corey, not an automated dispute system).
  - Still open: exact non-response timeout/cutoff before a "no fee" default kicks in.
- **Guarantee/refund period:** None. No clawback or refund logic needed — the fee is due once dual confirmation completes, full stop.
- **Collection mechanism:** Hybrid. Default path is invoicing (net-15 or net-30) after confirmation — no card required just to sign up or post a job, keeping employer-side friction low. Once confirmed, the employer is also offered an optional "pay now by card" path for those who'd rather not wait on an invoice. Requires Stripe (or equivalent) integration before either path is live. Card-on-file is NOT a gate on posting jobs — only real money movement is gated on having Stripe + a formed business entity in place.
- **Dependency on entity formation:** Payment processors generally require a real business entity + EIN for anything beyond minimal test volume — this is the point where the LLC-timing decision (still open, see above) becomes concretely necessary rather than just a liability-shield nice-to-have.
- **Multi-state note:** This structure (charging the employer, not the seeker) is what currently keeps Ground Network on the permissive side of Texas's employment-agency licensing rule. Re-verify this per-state before turning on real fee collection in any market beyond Texas.

## Why this file exists
Same rationale as PROJECT_STATE.md: chat memory across sessions is unreliable and has previously lagged real progress. Legal and business decisions are high-stakes enough that they need their own durable, committed, versioned record — separate from the engineering/technical log in PROJECT_STATE.md — so any future session (or an actual attorney/accountant brought in later) can see exactly what's been decided, what's been deferred on purpose, and what's still genuinely open.
