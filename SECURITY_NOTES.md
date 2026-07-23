# Ground Network — Security Notes
_Started 2026-07-22. Companion to PROJECT_STATE.md and LEGAL_AND_BUSINESS.md — same rationale: committed and versioned so security thinking survives across sessions instead of living only in chat memory that can lag or vanish. Findings below are grounded in a direct read of `firestore.rules`, `admin.html`, and `index.html`'s job-card rendering path, not general advice._

## Findings that need attention

1. **Job card rendering doesn't HTML-escape employer/aggregator-supplied text (XSS risk).** In `index.html`, job cards are built with direct template interpolation into `innerHTML` — e.g. `card.innerHTML = \`<div class="card-title">${job.title}</div>...\`` — with no escaping of `job.title`, `job.company`, or other fields. Those fields come from two untrusted sources: Adzuna/Jooble/direct-ATS aggregation (external, not controlled by us) and, more importantly, **the employer job-posting form itself** — any signed-up employer can type arbitrary text into title/company/description and it renders unescaped in every visitor's browser. This is a real stored-XSS path: a malicious or compromised employer account could post a job with `<img src=x onerror=...>` in the title and it would execute for every job seeker who views that listing.
 - Contrast: `admin.html` does this correctly — it has a proper `escapeHtml()` helper and uses it consistently on every rendered field (company name, email, job title, status). The fix pattern already exists in the codebase; it just needs to be applied to `index.html`'s job card / popup templates too.
 - Priority: high. This is the single most concrete, exploitable issue found today. Recommend a dedicated session to audit every `innerHTML =` assignment in `index.html` and route untrusted values through an `escapeHtml()`-equivalent (or switch to `textContent`/DOM APIs for the untrusted spans).

2. **`employers` collection is fully publicly readable, including email.** `firestore.rules` has `allow read: if true` on `/employers/{employerId}` with no field-level restriction, and no split between `get` and `list` — meaning anyone can run a single client-side query to dump the entire employers collection (company name, email, verified status, vouch list) with no auth. Worth a deliberate decision: is employer email meant to be public? If it only needs to be used internally, consider either (a) not storing email in a publicly-readable field, or (b) splitting employer docs into a public-safe subset (company name, verified badge) plus a private doc/subcollection for email, readable only by the owner and admin.

3. **`applications.create` has no auth requirement and no rate limiting.** This is intentional — job seekers apply without logging in — but it means anyone can script-spam fake applications against any real job posting at volume, with only the field-shape/status validation as a guard. Not a data-breach risk, but an abuse/spam-volume risk against employer inboxes. Worth pairing with **Firebase App Check** (blocks non-legitimate/scripted clients from hitting Firestore directly) and/or a lightweight honeypot field or submission-rate check in the Cloudflare Worker.

4. **No Content-Security-Policy.** `index.html`'s `<head>` has no CSP meta tag. Given the app relies on several inline `<script>` blocks, retrofitting a CSP isn't a quick fix (would need a nonce/hash strategy or a broad `'unsafe-inline'` that limits how much protection it actually buys), but it's worth scoping as a future defense-in-depth layer — especially important as a second line of defense given finding #1 above.

5. **Cloudflare Worker proxy isn't version-controlled.** Per `PROJECT_STATE.md`, `worker.js` lives only in the Cloudflare dashboard's Quick Edit editor, with a manually-resynced local reference copy. No git history, no diff review, no rollback path if a bad edit ships, and no visibility from this repo into whether it has its own rate limiting or abuse protection on the Adzuna/Jooble/direct-ATS routes it proxies. Worth a deliberate future step (`wrangler` + a `worker/` folder in this repo) once there's time to invest in it — flagged in `PROJECT_STATE.md` already as a "not yet done" item, restated here specifically as a security/auditability gap, not just a workflow inconvenience.

6. **Admin bootstrap is fully manual.** Adding an admin requires hand-editing the `/admins` collection in the Firebase console — appropriate at solo-operator scale, but worth revisiting if/when anyone else gets admin access (no offboarding checklist yet either).

## Already solid — confirmed, not just assumed

- **Firestore rules are well-scoped and were deliberately built from real production rules**, not a guessed draft (see the detailed provenance note at the bottom of `firestore.rules` itself, dated 2026-07-14). `verified` is server-derived from vouch count rather than client-settable; duplicate self-vouching is blocked; `applications` uses `get` (single known ID) rather than `list` for the no-login status-check link, which correctly prevents bulk enumeration of applicant PII even though individual links aren't authenticated.
- **Adzuna/Jooble API keys are server-side only**, behind the Cloudflare Worker proxy — never shipped in client code.
- **Firebase web API key exposure is expected and fine** — Firebase's own documented model puts access control in Firestore/Auth rules, not in hiding the config key. `admin.html` even has a comment saying so. Not a finding, just confirming this isn't accidentally on the list above.
- **`admin.html` is properly auth-gated** (Firebase Auth + `/admins` Firestore doc check) and consistently escapes rendered data.

## Open questions for Corey

1. Should employer email be publicly readable at all (finding #2), or was that always meant to be internal-only?
2. Worth scoping a dedicated "escape all the innerHTML in index.html" session soon, given finding #1 is a real exploitable path once employers are actually signing up and posting jobs?
3. Any appetite for Firebase App Check now, or park it until real traffic/abuse shows up?

## Why this file exists

Same rationale as `PROJECT_STATE.md` and `LEGAL_AND_BUSINESS.md`: security thinking is high-stakes enough that it shouldn't live only in a chat session that can be summarized, lag behind reality, or disappear. This file is the durable record — update it whenever a security-relevant decision gets made or a new finding turns up, and treat it as the source of truth for "what's been checked, what's still open" rather than relying on memory of a past conversation.
