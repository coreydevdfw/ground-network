# Ground Network — Project State
_Last verified 2026-07-16, directly from the live `main` branch and deployed site — not from chat memory._

## Confirmed from the live repo (github.com/coreydevdfw/ground-network)
- Repo renamed from `labor-network` to `ground-network`. GitHub auto-redirects the old URL.
- Live site (coreydevdfw.github.io/ground-network/) renders titled "Ground Network."
- Confirmed reconciled: `<title>`, meta tags, and canonical URL in `index.html` now say "Ground Network" — the rebrand is complete in code, not just the repo name.
- Repo root files: `index.html`, `service-worker.js`, `manifest.json`, `favicon.png`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `logo-mark.svg`, `og-image.png`, `dart-stations.json`, `dcta-stations.json` (Denton County Transit Authority, a second transit agency added alongside DART).
- **Answered (was an open question as of 2026-07-10): the Cloudflare Worker proxy source does NOT live in this git repo.** It's edited directly in the Cloudflare dashboard's "Quick Edit" (Monaco) editor at `dash.cloudflare.com/.../workers/services/edit/ground-network-proxy/production`, and isn't checked into version control anywhere. A reference-only copy is kept locally (outside this repo) and manually re-synced after each dashboard edit — there's no automated deploy pipeline for it.
- **Service worker IS registered** (was previously false/stale) — `service-worker.js` lives in the repo root, caches the app shell (`./`, `./index.html`, `./manifest.json`, `./favicon.png`, `./icon-192.png`, `./dart-stations.json`), navigate requests are network-first with cache fallback, everything else is cache-first. `CACHE_VERSION` must be bumped any time a cached file changes — currently `v5` (bumped 3x today alongside index.html deploys; see session update below).

## Backend / integrations (confirmed via direct code read)
- Firebase Auth + Firestore — collections: `employers`, `jobPostings`, `applications`.
- Employer peer-verification/vouch system is live: `window.vouchForEmployer()`, a "Vouch" button on job cards (visible only to a logged-in employer viewing another employer's direct posting), a public `verified` badge once an employer has 2+ vouches, `employer_vouched` GA4 event fires on use.
- Adzuna + Jooble routed through a Cloudflare Worker proxy at `ground-network-proxy.coreydevdfw.workers.dev`. Worker caches successful upstream responses for 600s via the Cache API to conserve Adzuna's free-tier daily quota.
- Direct-ATS integrations (bypass Adzuna/Jooble entirely, hit the employer's own applicant-tracking system): see session update below for full detail on today's additions and fixes.
- Mapbox GL JS v3.3.0 — map, 3D pins, nav mode, live traffic gauge driven by route congestion, category-tinted cluster bubbles with dominant-category emoji.
- GA4 (G-LTCHQGLGCY) — custom events tracked across seeker and employer funnels, including `job_text_search` (new today).
- Applied-jobs tracking via localStorage, shown inline as "Applied ✓" on job cards — no aggregate "My Applications" view yet.
- Adzuna free tier: 25 calls/min, 250/day, 1,000/week, 2,500/month. Jooble limits undocumented publicly — check account confirmation email.

## Direct-ATS employers (as of 2026-07-16)
| Employer | ATS | Tier | Notes |
|---|---|---|---|
| Toyota Connected | Greenhouse | 1 (free, public, no key) | `boards-api.greenhouse.io/v1/boards/{token}/jobs?content=true` |
| AT&T | Workday | 2 (semi-public CxS endpoint) | Rewritten today: paginated (3 pages × 20), city-scoped `searchText`, fallback geocoding for multi-location postings |
| Alkami | Workday | 2 | Added today, same fetcher as AT&T |
| Match Group | Lever | 1 (free, public, no key) | Added today. **Corrected from an earlier assumption of SmartRecruiters** — verified against their real careers page (mtch.com/careers → jobs.lever.co/matchgroup) before building anything |

All four share the same job schema (`{ id, title, company, url, lat, lng, category, created, source, postingId?, employerId? }`) and get geocoded via a shared Mapbox cache (`geocodeLocationText()` / `locationGeocodeCache`) so they render on the map identically to Adzuna/Jooble results.

## Open questions for Corey
1. ~~Where does `worker.js` actually live?~~ **Resolved** — Cloudflare dashboard only, not in git. If you ever want it version-controlled, that'd need a separate deliberate step (e.g. `wrangler` + a `worker/` folder in this repo).
2. Any other direct-ATS employers you want researched/added beyond Toyota Connected, AT&T, Alkami, and Match Group?
3. Is the `ops-pm-jobs` category (Operations & Project Management — added today, see below) worth promoting into the main nav/filters more prominently, or is the dropdown enough?

## Why this file exists
Chat memory (`memory.md` in the Ground Network project) is auto-generated between claude.ai conversations and can lag real progress. This file is the fix: commit it to the repo root, update it at the end of any session with real changes, and treat it — not chat memory — as the source of truth. Because it's committed and versioned, it can be pulled fresh via the same raw-GitHub method used for `index.html`, from any device, regardless of which app or session you're chatting in.

## Session update — 2026-07-16 (map exploration UX, Workday fixes, new ATS employers, ops-pm category, ZIP box doubles as job search)

**Map exploration / UX** (in response to feedback that the map felt like isolated "little worlds" per suburb, with no guided path from "near me" outward):
- Auto-geolocation on load with a DFW fallback if permission is denied/unavailable.
- Cluster bubbles are now tinted by dominant job category within the cluster, with a category emoji, instead of a flat generic color.
- Off-viewport job counts ("nudges") point toward job concentrations outside the current map view, encouraging outward exploration instead of independent free-scrolling.
- The location "pin"/locate button now force-recenters back to the user's location (clearing any open job popup/active card) once tapped again after a job's been selected — previously it silently re-ran geolocation in place with no visible effect if a job selection had panned the map away.
- Mobile layout: replaced the fixed 50/50 map-vs-list split with a Map/List toggle, since the split gave neither view enough room to be useful once pins stopped registering at zoomed-out scale.
- Fixed a cluster-label rendering bug: the point-count text field used `['concat', emojiExpr, ' ', '{point_count_abbreviated}']`, but Mapbox's legacy `'{token}'` substitution only works as the sole literal value, not nested inside `concat` — it was rendering a garbled string fragment instead of a number. Fixed by switching to `['get','point_count_abbreviated']`.

**Workday integration — verified and fixed two real bugs:**
1. Results were unfiltered/nationwide rather than DFW-scoped. Fixed by sending a city-scoped `searchText` (derived from `searchLocationText`) and paginating (3 pages × 20) instead of a single unfiltered page.
2. ~65% of multi-location postings were being silently dropped because their `locationsText` was an unparseable multi-city count-string. Fixed with fallback geocoding to the search city when the count-string can't be parsed directly.
3. Also found and fixed a server-side bug in the Cloudflare Worker's Workday route: it was hardcoding `limit:20, offset:0` regardless of what the client requested, which capped pagination. Now forwards client-supplied `limit`/`offset` (with sane defaults). Verified live via a direct pagination test.
4. **Found via Corey testing the new ZIP-box search on the live site**: typing "AT&T" returned "No jobs matching" even though AT&T is a configured direct-ATS employer. Root cause: `fetchWorkday()`'s `searchText` was the exact reverse-geocoded suburb name (e.g. "Farmers Branch," "Coppell," "Addison") — Workday's search is a literal keyword match against each posting's location text, not a radius search, and most employers tag postings with their own hub city (AT&T → mostly "Dallas," Alkami/Toyota Connected → mostly "Plano") rather than whatever suburb the user happens to be standing in. So most DFW users outside Dallas/Plano proper were silently getting zero AT&T/Alkami results. Fixed by trying the suburb first, then falling back through a short list of DFW hub cities (Dallas, Plano, Fort Worth, Irving) until one returns results. Pragmatic single-market fix, not yet tested live post-deploy — Corey should retry the "AT&T" search after this deploy propagates.

**New direct-ATS employers:** Alkami (Workday) and Match Group added — see table above. Building Match Group required correcting an earlier assumption (SmartRecruiters) after checking their real careers page and finding Lever instead; a new generic `/lever/:site` Cloudflare Worker route was built and verified live (82 postings for Match Group, 10 in Dallas) before wiring it into the app.

**New category: `ops-pm-jobs` (Operations & Project Management).** Adzuna has a fixed, closed category taxonomy with no matching slug for this, so a new `adzuna:false` flag was added to the `CATEGORIES` entries; the Adzuna fetch loop now derives a filtered `adzunaTags` list that excludes any category flagged `adzuna:false`, while Jooble (free-text keywords, no fixed taxonomy) and direct/ATS sources continue processing the full tag list unaffected.

**ZIP box now doubles as a job/company text search.** Previously the sidebar's ZIP input only accepted a 5-digit ZIP to re-center the map. Now: a 5-digit value still re-centers (unchanged behavior); anything else is treated as a free-text filter over jobs already loaded for the current area (matches against job title or company, case-insensitive substring). This was added specifically so a direct-ATS integration can be spot-checked by typing the employer's name (e.g. "AT&T") and seeing just that employer's live postings, without hunting through the full unfiltered job list. Submitting a search auto-reveals the job list (defaulting to "All Jobs" if no category was picked yet) and fits the map to the matched pins.

**Deploys today:** `index.html` uploaded 3x (recenter/exploration UX + Map/List toggle were already live from a prior session; today added the Workday/Alkami/Match Group/ops-pm-jobs bundle, then the ZIP-search bundle). `service-worker.js`'s `CACHE_VERSION` was bumped 3x (`v3` → `v4` → `v5`) to match, per the file's own header instruction to bump on any `APP_SHELL` file change.

## Session update — 2026-07-10 (dynamic search location + Jooble radius fix)
- Root cause found for why job results were always centered on a hardcoded "Dallas, TX" string sent to both Adzuna (`where=`) and Jooble (`location`), even when the user's actual location (via geolocation or ZIP) was elsewhere in DFW/Greater North Dallas.
- Added `searchLocationText` (defaults to 'Dallas, TX') plus `updateSearchLocationText()`, which reverse-geocodes the current `CENTER` coordinates via Mapbox's Geocoding API to get the real place + state (e.g. "Plano, TX" — confirmed CENTER's actual coords resolve there, not "Dallas").
- `refetchJobsAroundCenter()` now awaits `updateSearchLocationText()` before fetching, so both Adzuna and Jooble queries use the accurate current-location text.
- Fixed a real bug: Jooble's `radius` param was being sent as `"25"` (miles, invalid) — Jooble's API only accepts `0/4/8/16/26/40/80` (km). Added `joobleRadiusKm(miles)` to snap to the nearest valid step and wired it into the Jooble fetch body.
- Verified empirically before shipping: tested Adzuna's `/us/` endpoint directly with `lat`/`long` and `latitude`/`longitude` params — both returned HTTP 400. Adzuna does not support coordinate-based search on this endpoint; `where=` (text) is the only working location mechanism, hence the reverse-geocode approach instead of a coordinate param.
- Committed to `main` as "Dynamic search location for Adzuna/Jooble + fix Jooble radius enum" (commit 343d16e). All 5 edits verified via direct CodeMirror document inspection + `new Function()` syntax check on the extracted inline script before committing.
- This is groundwork for the multi-city vision (DFW/Greater North Dallas now, other cities later, eventual city transit partnerships) — location is now derived dynamically rather than hardcoded, which is a prerequisite for expanding beyond one hardcoded market.

## Backlog (not urgent, noted 2026-07-14)
- **Mobile job-scroll visibility**: the scroll-through-jobs interaction on mobile needs better visibility/affordance — worth tweaking the visual treatment or adding explicit options for how jobs are displayed while scrolling. Flagged by Corey, not yet scoped. (Partially addressed 2026-07-16 by the mobile Map/List toggle — see session update above — but the underlying scroll-affordance concern may still be worth revisiting.)
- **No in-app link to admin.html**: the master page isn't linked from the consumer-facing UI at all — reaching it currently requires knowing the direct URL. Intentional for now (keeps it off regular users' radar), but worth a deliberate decision later rather than just staying that way by default.
