# Ground Network — Project State
_Last verified 2026-07-10, directly from the live `main` branch and deployed site — not from chat memory._

## Confirmed from the live repo (github.com/coreydevdfw/ground-network)
- Repo renamed from `labor-network` to `ground-network`. GitHub auto-redirects the old URL.
- Live site (coreydevdfw.github.io/ground-network/) renders titled "Ground Network."
- ⚠️ Not yet reconciled: the `main` branch `index.html` still says "Labor Network" in `<title>`, meta tags, and the canonical URL — these haven't been updated to match the new name/repo.
- Repo root files: `index.html`, `manifest.json`, `favicon.png`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `logo-mark.svg` (new), `og-image.png`, `dart-stations.json`, `dcta-stations.json` (new — Denton County Transit Authority, a second transit agency added alongside DART).
- No `worker.js` in the repo — Cloudflare Worker proxy source location unknown, needs confirming.
- No service worker registered, no `beforeinstallprompt` handling — the current phone install (Add to Home Screen) was done manually through the browser, not via an automatic install prompt.

## Backend / integrations (confirmed via direct code read)
- Firebase Auth + Firestore — collections: `employers`, `jobPostings`, `applications`.
- Employer peer-verification/vouch system is live: `window.vouchForEmployer()`, a "Vouch" button on job cards (visible only to a logged-in employer viewing another employer's direct posting), a public `verified` badge once an employer has 2+ vouches, `employer_vouched` GA4 event fires on use.
- Adzuna + Jooble routed through a Cloudflare Worker proxy at `labor-network-proxy.coreydevdfw.workers.dev` (URL name unchanged despite the rebrand — worth confirming it's still correct/live).
- Mapbox GL JS v3.3.0 — map, 3D pins, nav mode, live traffic gauge driven by route congestion.
- GA4 (G-LTCHQGLGCY) — 14 custom events tracked across seeker and employer funnels.
- Applied-jobs tracking via localStorage, shown inline as "Applied ✓" on job cards — no aggregate "My Applications" view yet.
- Adzuna free tier: 25 calls/min, 250/day, 1,000/week, 2,500/month. Jooble limits undocumented publicly — check account confirmation email.

## Open questions for Corey
1. Where does `worker.js` actually live (Cloudflare dashboard directly? a separate repo?)?
2. Is "Ground Network" the final name going forward, or mid-rebrand? Should `index.html`'s title/meta/canonical be updated to match the new repo name?
3. What else changed "since the other day" that wouldn't show up just from reading the current file — decisions made, things tried and reverted, known bugs not yet in code comments?

## Why this file exists
Chat memory (`memory.md` in the Ground Network project) is auto-generated between claude.ai conversations and has been lagging real progress — it still describes the app as "Labor Network," has no record of the rebrand, the new transit dataset, or the vouch system. That file is also read-only to Claude in this session, so it can't be corrected here.

This file is the fix: commit it to the repo root, update it at the end of any session with real changes, and treat it — not chat memory — as the source of truth. Because it's committed and versioned, it can be pulled fresh via the same raw-GitHub method used for `index.html`, from any device, regardless of which app or session you're chatting in.
