# Ground Network — Legal & Business State
_Companion to PROJECT_STATE.md. This file is the durable record of legal/business decisions, open questions, and compliance-relevant backlog — committed and versioned so it survives across sessions, not dependent on chat memory._

## Current legal posture (as of 2026-07-11)
- **Entity:** None formed yet. Operating as an individual. No business name, EIN, or liability shield in place.
- **Legal docs on the live site:** None. No Privacy Policy, no Terms of Service, no acceptable-use/EEO policy, no age requirement stated anywhere.
- **Placement fee:** Advertised in the "Post a job" UI ("Free to post. You only pay a placement fee if you hire someone through the app.") but **not implemented** — no payment processing, no invoicing, no Stripe/payment processor integration. Hire status is a self-reported dropdown only (Contacted / Interviewing / Hired / Not a fit) with zero enforcement.
- **Employer verification:** Peer-vouch system only (2 vouches from other employers = "verified" badge). No EIN, business license, or business-email-domain check.
- **Applicant PII collected:** Firestore `applications` collection stores `applicantName`, `applicantEmail`, `applicantPhone` for every direct-posting application, visible to the employer. No privacy policy currently discloses this.
- **Geolocation:** Custom "Share your location for a tighter radius" prompt exists in the UI, but no privacy disclosure backs it.
- **Regulatory research done (updated 2026-07-11):** Confirmed via multiple public sources that Texas has **no state-level licensing or registration process for employment agencies at all** — there is no board, no application, no case-by-case review of a business model like Ground Network's. TWC's Employment Service Guide is public policy guidance, not a determination authority, and TWC has no mechanism to issue a binding ruling on a specific business. This means there's no "meeting with TWC" to have about this — a general inquiry would likely just get pointed back to the same public guide already reviewed.
  - **Important distinction:** this is separate from standard TWC *employer* registration (unemployment insurance tax registration, required within 10 days of first paying $1,500+ in wages in a quarter or employing someone 20 weeks/year). That requirement applies to Ground Network itself once it has actual employees (not yet the case — Corey is solo), and separately applies to every employer using the platform for their own hiring (not something Ground Network needs to handle on their behalf). It has nothing to do with whether the placement-fee *business model* needs an employment-agency license — it doesn't, per the above.
  - The real path to genuine legal certainty on the placement-fee model is a **Texas business/employment attorney consultation**, not TWC — an attorney can issue a specific opinion TWC has no authority to give. This consultation likely also covers the still-open LLC formation decision in the same conversation. **Parked for later** (not urgent while the fee mechanism is unbuilt) — when ready, next step is drafting a one-page plain-English brief (what the platform does, who pays the fee, the tiered structure, the confirmation mechanic) to bring to that conversation.
  - Other states may still regulate fee-charging agencies differently (some license regardless of which side pays) — must be re-checked per state before charging fees anywhere beyond Texas.

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
- **Placement fee mechanics** — decided 2026-07-11, see dedicated section below, including finalized tiered fee amounts. Still open: final Stripe/payment integration work, and confirming the $60k+ tier's $2k/$3k split once real salaried postings exist.

## Placement fee mechanics (decided 2026-07-11)
Design finalized in conversation; not yet built. Captured here so implementation can happen in a later session without re-deriving the design.

- **Fee structure:** Flat fee per hire, tiered by role type — not a single number, not a percentage of pay. Three tiers, researched and decided 2026-07-11 against industry benchmarks (contingency recruiter fees run 15-30% of first-year salary; flat-fee platforms run $1,500-5,000 under $75k; SHRM's 2026 average cost-per-hire is $4,700-5,800, so all three tiers below sit well under what employers already typically spend to fill a role):
  1. **Hourly / blue-collar** (warehouse, logistics, food service, retail floor, etc.): **$250-500 flat**, working default $375. Fits the current job-category mix; no established market benchmark to undercut, this tier effectively defines the category.
  2. **Salaried, under $60k** (entry-level office/admin/professional): **$1,500 flat**. Comfortably below what a 20% contingency fee would charge on a comparable role (20% of $50k = $10,000), and far below the SHRM average.
  3. **Salaried, $60k+**: **flat fee, proposed $2,000 for $60k-$100k and $3,000 for $100k+** (both within the $1,500-3,000 range approved 2026-07-11) — a lightweight second breakpoint rather than one flat number across a very wide salary span. This specific $2k/$3k split is a reasonable default pending Corey's confirmation once real salaried postings exist to sanity-check against; percentage-based pricing (5-8% of salary) was considered and explicitly passed over in favor of staying flat/simple.
  All figures must be implemented as configurable values, not hardcoded, so they can be adjusted without a code deploy.
- **Trigger / anti-fraud mechanism:** Dual confirmation. Employer marks an applicant "Hired" (existing dropdown) — this alone does NOT trigger the fee, it moves the application into a pending-confirmation state. The seeker is then asked (in-app / email) "Did you get this job at [Employer]?" The fee is only triggered once the seeker independently confirms. A seeker "no" or a non-response past some cutoff means no fee is charged. An employer-says-hired / seeker-says-no mismatch is a manual review flag (at current volume: a notification to Corey, not an automated dispute system).
  - Still open: exact non-response timeout/cutoff before a "no fee" default kicks in.
- **Guarantee/refund period:** None. No clawback or refund logic needed — the fee is due once dual confirmation completes, full stop.
- **Collection mechanism:** Hybrid. Default path is invoicing (net-15 or net-30) after confirmation — no card required just to sign up or post a job, keeping employer-side friction low. Once confirmed, the employer is also offered an optional "pay now by card" path for those who'd rather not wait on an invoice. Requires Stripe (or equivalent) integration before either path is live. Card-on-file is NOT a gate on posting jobs — only real money movement is gated on having Stripe + a formed business entity in place.
- **Dependency on entity formation:** Payment processors generally require a real business entity + EIN for anything beyond minimal test volume — this is the point where the LLC-timing decision (still open, see above) becomes concretely necessary rather than just a liability-shield nice-to-have.
- **Multi-state note:** This structure (charging the employer, not the seeker) is what currently keeps Ground Network on the permissive side of Texas's employment-agency licensing rule. Re-verify this per-state before turning on real fee collection in any market beyond Texas.

### Revenue modeling (2026-07-11)
Rough gross-revenue scenarios at fee midpoints ($375 hourly / $1,500 salaried), haircut for the dual-confirmation mechanic (not every self-reported hire will get seeker-confirmed):
- Early DFW pilot (5 hourly + 1 salaried hire/mo): ~$20,000-36,000/yr at 50-90% confirmation rates.
- Grown-out DFW (20 hourly + 5 salaried/mo): ~$90,000-162,000/yr.
- Multi-city (60 hourly + 15 salaried/mo): ~$270,000-486,000/yr.
Figures are gross, before Stripe processing fees, collections losses, and operating costs. Sources: SHRM 2026 Benchmarking Report, plus staffing/recruiting industry fee surveys (frontlinesourcegroup.com, spectraforce.com, leonar.app, instawork.com, recruiterslineup.com, valuablerecruitment.com, isgpartners.com, pin.com) — see chat for full source list.

## Why this file exists
Same rationale as PROJECT_STATE.md: chat memory across sessions is unreliable and has previously lagged real progress. Legal and business decisions are high-stakes enough that they need their own durable, committed, versioned record — separate from the engineering/technical log in PROJECT_STATE.md — so any future session (or an actual attorney/accountant brought in later) can see exactly what's been decided, what's been deferred on purpose, and what's still genuinely open.
