# Ground Network -- Interaction Spec

_Written 2026-07-19, grounded in a direct read of the live `index.html`. This is a companion to `PROJECT_STATE.md` -- that file tracks what's built, this one tracks how anything new should look and move._

## Why this exists

The app has been built session-by-session, each one solving whatever was broken at the time. That's produced a genuinely capable, good-looking tool -- but the motion and interaction choices were never unified, because nothing forced them to be. A direct scan of the CSS confirmed it:

- Multiple different transition durations in use across the file (0.15s, 0.18s, 0.2s, 0.3s, 0.4s, 0.45s, 0.55s, 0.6s, 0.75s)
- - Two unrelated easing curves: `cubic-bezier(0.4,0,0.2,1)` (a standard decelerate curve, used on the sidebar collapse) and `cubic-bezier(0.34,1.56,0.64,1)` (a bouncy overshoot curve, used once, elsewhere)
  - - Only one component -- the sidebar collapse -- had genuinely considered motion. Everything else defaulted to whatever felt fine in the moment it was written.
   
    - None of that is a bug. It's just what happens when every prompt is "fix this one thing." This doc is the fix at the system level: a small, fixed vocabulary that every future change -- mine or yours -- draws from instead of inventing its own.
    - ## 1. Motion system
   
    - Two durations. That's it.
   
    - | Tier | Duration | Use for |
    - |---|---|---|
    - | Micro | 160ms | Button press/hover feedback, toggle state changes, anything that responds directly to a tap |
    - | Structural | 400ms | Panels sliding/collapsing, sheets opening, view transitions -- anything that changes what's on screen |
   
    - As of 2026-07-19 these live as real CSS custom properties in `index.html`'s `:root` block: `--ease-standard`, `--dur-micro`, `--dur-structural`. The mobile sidebar's list-mode transition is the first thing wired up to them (see PROJECT_STATE.md for that change) -- everything else should be migrated to these tokens as it's touched, rather than all at once.
   
    - One easing curve for everything structural: `cubic-bezier(0.4,0,0.2,1)` (`--ease-standard`). Decelerate-into-place reads as controlled and professional, which matches the "tool, not toy" goal.
   
    - The bounce curve (`cubic-bezier(0.34,1.56,0.64,1)`) gets retired to exactly one job: a single deliberate confirmation moment (e.g. "Applied" landing, a job successfully posted). Reserving it for one specific, celebratory moment is what makes it read as intentional instead of erratic.
   
    - What not to animate: avoid animating `height`, `width`, or `max-height` directly wherever `transform` (translateY/scale) + `opacity` can do the same job. The sidebar's `max-height` transition is a legitimate exception (content needs to reflow), but it comes with a real cost specific to this app: Mapbox GL doesn't know its container resized until you tell it. Any animation that changes `#map-wrap`'s dimensions needs an explicit `map.resize()` fired on `transitionend`, not mid-animation.
    - ## 2. Mobile space allocation (updated 2026-07-19)
   
    - The original complaint that prompted this doc: in list mode, `#sidebar` was capped at `max-height:50dvh`, and after the header, locate/zip/employer rows, and filter bar each took their share of that 50%, the actual job list was landing closer to 15% of the screen, not 50%.
   
    - First fix shipped: raised `#sidebar`'s mobile list-mode `max-height` from `50dvh` to `82dvh`, using the new motion tokens for the transition. `#map-wrap` (flex:1) still keeps a visible strip above it for context. This is a same-toggle, lower-risk fix -- it does not touch the existing `#mobile-view-toggle` (map/list switch) or the `body.mobile-map-mode` override, both of which were already built and are working as designed.
   
    - Still open, if more room is wanted later: a draggable bottom sheet (peek/half/full snap points, iOS/Google Maps style) replacing the binary toggle entirely. That's a bigger change -- new drag-gesture JS, not just CSS -- and should be scoped as its own session rather than folded into a CSS tweak.
   
    - ## 3. States every view needs
   
    - Define these once, apply everywhere a view fetches or depends on data (job list, applicant list, employer dashboard):
   
    - - Loading -- a skeleton/placeholder in the shape of the eventual content, not a bare spinner. Matters more here than most apps because Adzuna/Jooble/Workday calls are real network round-trips with visible latency.
      - - Empty -- "no jobs matching X," styled consistently, not a blank scrolling area.
        - - Error / offline -- distinct from empty. The service worker already caches the app shell; an error state should make clear whether this is "no results" vs. "couldn't reach the network."
          - - Success / confirmation -- this is the one place the bounce easing from Section 1 belongs.
            - ## 4. Visual tokens already established (lock these in)
           
            - Pulled directly from the live CSS:
           
            - - Brand green: `#00ff88` (dark mode), `#006e3c` / `#005a30` (light mode equivalents)
              - - Base background: `#0a0a0a` / `#111`
                - - Font: `Inter, system-ui, sans-serif` for body copy; `Space Grotesk` (Google Fonts) is also loaded -- check current usage before assuming which one applies where.
                  - - Corner radius: 6-8px for buttons/inputs/small cards, 14px for larger cards.
                    - - The "sticker" shadow signature, repeated across cards and buttons: `0 0 0 6px rgba(0,255,136,0.08), 4px 6px 0 rgba(0,255,136,0.28)` -- a flat, hard-offset shadow with no blur.
                     
                      - ## 5. Open decisions
                     
                      - - Whether the bottom-sheet pattern (Section 2) is worth building, or whether the 82dvh fix is enough on its own.
                        - - Loading skeleton visual treatment (shimmer vs. static) isn't specified -- pick one and it becomes the standard.
                          - - Whether the bounce easing's one reserved use case should be "Applied" confirmation or job-posted confirmation -- pick one, don't use it for both, so it stays a signature rather than a general-purpose flourish.
                           
                            - This doc doesn't prescribe every future implementation -- it's the constraint set so the next round of building, whoever does it, produces one coherent tool instead of another well-made one-off.
                            - 
