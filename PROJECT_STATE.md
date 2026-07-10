# Ground Network — Project State
_Last verified 2026-07-10, directly from the live `main` branch and deployed site — not from chat memory._

## Confirmed from the live repo (github.com/coreydevdfw/ground-network)
- Repo renamed from `labor-network` to `ground-network`. GitHub auto-redirects the old URL.
- Live site (coreydevdfw.github.io/ground-network/) renders titled "Ground Network."
- Confirmed reconciled: `<title>`, meta tags, and canonical URL in `index.html` now say "Ground Network" — the rebrand is complete in code, not just the repo name.
- Repo root files: `index.html`, `manifest.json`, `favicon.png`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `logo-mark.svg` (new), `og-image.png`, `dart-stations.json`, `dcta-stations.json` (new — Denton County Transit Authority, a second transit agency added alongside DART).
- No `worker.js` in the repo — Cloudflare Worker proxy source location unknown, needs confirming.
- No service worker registered, no `beforeinstallprompt` handling — the current phone install (Add to Home Screen) was done manually through the browser, not via an automatic install prompt.

## Backend / integrations (confirmed via direct code read)
- Firebase Auth + Firestore — collections: `employers`, `jobPostings`, `applications`.
- Employer peer-verification/vouch system is live: `window.vouchForEmployer()`, a "Vouch" button on job cards (visible only to a logged-in employer viewing another employer's direct posting), a public `verified` badge once an employer has 2+ vouches, `employer_vouched` GA4 event fires on use.
- Adzuna + Jooble routed through a Cloudflare Worker proxy at `ground-network-proxy.coreydevdfw.workers.dev` (renamed from labor-network-proxy; confirmed live). Worker now caches successful upstream responses for 600s via the Cache API to conserve Adzuna's free-tier daily quota.
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


## Session update — 2026-07-10 (dynamic search location + Jooble radius fix)
- Root cause found for why job results were always centered on a hardcoded "Dallas, TX" string sent to both Adzuna (`where=`) and Jooble (`location`), even when the user's actual location (via geolocation or ZIP) was elsewhere in DFW/Greater North Dallas.
- Added `searchLocationText` (defaults to 'Dallas, TX') plus `updateSearchLocationText()`, which reverse-geocodes the current `CENTER` coordinates via Mapbox's Geocoding API to get the real place + state (e.g. "Plano, TX" — confirmed CENTER's actual coords resolve there, not "Dallas").
- `refetchJobsAroundCenter()` now awaits `updateSearchLocationText()` before fetching, so both Adzuna and Jooble queries use the accurate current-location text.
- Fixed a real bug: Jooble's `radius` param was being sent as `"25"` (miles, invalid) — Jooble's API only accepts `0/4/8/16/26/40/80` (km). Added `joobleRadiusKm(miles)` to snap to the nearest valid step and wired it into the Jooble fetch body.
- Verified empirically before shipping: tested Adzuna's `/us/` endpoint directly with `lat`/`long` and `latitude`/`longitude` params — both returned HTTP 400. Adzuna does not support coordinate-based search on this endpoint; `where=` (text) is the only working location mechanism, hence the reverse-geocode approach instead of a coordinate param.
- Committed to `main` as "Dynamic search location for Adzuna/Jooble + fix Jooble radius enum" (commit 343d16e). All 5 edits verified via direct CodeMirror document inspection + `new Function()` syntax check on the extracted inline script before committing.
- This is groundwork for the multi-city vision (DFW/Greater North Dallas now, other cities later, eventual city transit partnerships) — location is now derived dynamically rather than hardcoded, which is a prerequisite for expanding beyond one hardcoded market.
