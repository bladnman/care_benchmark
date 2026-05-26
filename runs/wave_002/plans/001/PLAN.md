# Comprehensive Implementation Plan for Pocket Aviary (v1)

## 1. Scope
### In v1
- Single horizontal browser-based aviary scene with 2–7 birds.
- Personality vectors, mood states, procedural call grammar, monotonic personality drift driven primarily by presence-time.
- Interactions: return-greeting (one bird), listen-in, offer (seed/song/pool with cooldown), settle gesture, field notebook.
- Accounts: magic-link email sign-in, synthetic UUID, single aviary per account, visit invitations (opt-in, read-only, revocable).
- Multi-device sync via server-side canonical state.
- Screen-reader narration (naturalist prose), reduced-motion mode (cross-fade poses), WCAG AA contrast, keyboard nav, call captions.
- Performance: <2MB gzipped bundle, <500ms to first bird, 60fps idle on 5yo laptop, no memory growth, WebAudio procedural synthesis.
- Server-side simulation tick (~1/min), presence accounting (visibility+focus+activity), aggregate-only telemetry respecting privacy boundary.
- Day/night (local time), ambient weather/rain/wind, leaf/feather micro-motion.
- Adoption: two starter birds from 6-species pool, user naming, stable bird IDs, renameable.
- Naturalist voice on product surfaces; matter-of-fact on system surfaces.

### Explicitly Out of Scope (per non_goals.md, product_brief.md)
- No native apps, no gamification (streaks, scores, badges, levels, counters, calendar dots), no Tamagotchi mechanics (no hunger/distress/death/happiness decay), no social network surfaces (profiles, feeds, discovery, leaderboards, comments, public aviaries).
- No panning/scrolling/zooming; one-screen scene.
- No exposed personality numbers; no notifications/push; no co-presence on visits.
- Visits default off, revocable, read-only, no drift contribution from visitors.
- No catalog selection for starters; birds "arrive."

Drift monotonic toward expressive; presence = real interaction (three-signal conjunction).

## 2. Architecture
- **Client/Server split**: Server owns canonical aviary state + personality vectors + simulation tick. Clients are thin renderers that pull snapshots and submit append-only interaction events. No client-owned personality state.
- **Service shape**: Simulation service (tick, event log consumption, drift/mood), auth service (magic links, sessions, synthetic UUID), snapshot service (state delivery to clients), notebook generator, visit service (invites/revokes).
- **Render pipeline boundary**: Client receives snapshot (bird positions, moods, call timings, active transitions, day/night/weather). Interpolates 60fps motion between snapshots; pure rendering ornaments (leaves) client-side. Stops rendering when tab hidden but simulation continues.
- **Data flow**: Client writes events (offer, listen-in, settle, presence pings); tick consumes log → updates vectors/moods → writes canonical snapshot. Clients never write vectors.

## 3. Data Model
- **Account**: synthetic UUID (primary), encrypted email (one place), session tokens (revocable), settings (audio, accessibility, visit notifications opt-in), visit log.
- **Bird**: stable UUID, account_id, species (from 6-pool), user name (renameable), personality vector (5 scalars: boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity), current mood (enum: wary/content/curious/drowsy/alert), mood timers, adoption timestamp, perch zone (front/mid/back chosen by mood/personality).
- **Personality vector persistence**: server-only, updated solely by tick via additive deltas from event log; never client-mutated, never last-write-wins.
- **Presence event**: visibilityState=visible + window focus + pointer/key activity in last N minutes (calibrate ~few min, favor longer). Accumulates presence-time.
- **Interaction event log**: append-only (offer type + bird, listen-in start/end + duration, settle, presence ping). Per-bird only, never aggregated beyond per-account sim.
- **Notebook entry**: timestamp, naturalist prose string (sparse: ~1 every few days).
- **Visit invitation**: per-invite email, expiration 30d, revocable, read-only token.
- **Snapshot**: current per-bird (position, mood, call phase, active transitions), aviary (time-of-day, weather flag).

## 4. API Surface (high-level)
- Auth: POST /magic-link, GET /verify?token → session token.
- Snapshot: GET /aviary/snapshot (auth) → current state; small payload.
- Events: POST /events (interaction payloads); append-only.
- Notebook: GET /notebook (read-only entries).
- Account: settings, export JSON (birds+vectors+moods+notebook), delete (soft 30d then hard), visit invites/revokes/logs.
- Visit: GET /visit/:token → read-only snapshot view (no event writes).
- Errors: matter-of-fact messages; never naturalist.

## 5. Simulation Engine Design
- **Tick (~1/min, always running)**: consume recent event log, apply presence-time (primary), listen-in/offer/settle (secondary) → compute additive deltas → update personality vectors (monotonic ↑ expressive only). Transition moods from interactions + local time-of-day + ambient weather + personality bias. Advance mood timers. Write new canonical state.
- **Drift function**: low-pass filter over presence/interaction signals; calibration target = measurable instrument drift ~1wk regular use, visible user drift ~3wks. No negative drift on neglect → ambient quietness.
- **Mood transitions**: recent events (offer accepted → content), time (drowsy dusk, alert morning), weather (rain damps vocal freq), personality (high-bold less wary).
- **Call grammar runtime**: per-bird motif library + personality-shaped timing/pitch + mood modulation; procedural synthesis WebAudio; chorus from real-time variation; recognizability across drift.
- **Bird-to-bird**: call response, mood spread, chorus emergence.
- **Mood persistence**: carries across sessions + tick interim.

## 6. Sync Model
- Single canonical record per aviary (server). Clients pull snapshots on visibility change, long gaps, keepalive. No client-to-client sync, no merge, no eventual consistency. Additive deltas + event-log ordering prevents last-write-wins corruption of drift. Rare sync conflicts surface as matter-of-fact errors. Client session tokens revocable.

## 7. Frontend Rendering Pipeline
- Load: pull snapshot → place birds mid-action in current motions; render immediately (no wake-up animation, no spinner; quiet field on slow load).
- Scene: single horizontal, 3 perch zones, responsive compress/widen preserving aspect & all birds visible. Subtle parallax foreground/background. Day/night local-time palette shifts, ambient weather (rain/wind short-lived).
- Idle micro-motion: mood-shaped (wary scans back, content preens, curious tilts, drowsy fluffed low) continuous; ambient leaf/feather drift client-generated.
- Transitions: listen-in mix ramp (slow rise/fall), offer reaction, settle lighting (slow evening shift + undo 5s), perch changes (interpolated paths or reduced-motion crossfade).
- Reduced-motion: cross-fade between preen poses, cross-fade perches, slowed color shifts, remove leaf drift. Still full audio/captions/notebook/drift.
- Top bar: sparse icons (account, a11y, notebook, offer), fade on cursor still, reappear on activity.

## 8. Audio Pipeline
- Procedural WebAudio synthesis from motif library per bird/species; personality (vocal freq) + mood shaped timing/pitch. Chorus = simultaneous live calls mixed.
- Listen-in: gradual rebalance (focus up, others ambient not silent).
- Settle: quiet calls.
- Fallback (no WebAudio): graceful silence + captions default-on. No recorded loops ever.
- Call captions: runtime-generated short naturalist prose per actual call, fade with call.

## 9. Accessibility Surfaces
- Screen-reader: slow naturalist prose narration (30-60s cadence idle; priority on greetings/offers/settle; same voice as notebook; event observations not state lists).
- Reduced-motion: designed surface (cross-fades), not stripped fallback.
- Keyboard: tab top-bar → aviary birds (arrows, enter listen-in, escape), offer affordance, settle.
- Captions: opt-in, naturalist prose near bird.
- Contrast: WCAG AA on all user-copy text/chrome.
- Focus: visible high-contrast outline on aviary bg.

## 10. Performance Budgets & Observability
- Bundle <2MB gzipped (drives procedural audio, small SVGs/procedural visuals, aggressive code-split for settings/visit flows).
- TTFB <500ms mid-tier mobile 4G (edge snapshot, minimal critical path).
- Runtime: 60fps idle 30min on 5yo laptop; no memory growth (reuse buffers, bounded contexts, no retained refs on scroll-out).
- Observability: synthetic browsers (common geos), RUM aggregate (load timings, frame times, audio errors, tick p99 latency alarm >5s); no per-bird/per-account data in telemetry. Privacy boundary strict.
- Browser: last 2 major Chrome/Safari/Firefox/Edge; older → matter-of-fact unsupported page.

## 11. Rollout
- v1: 2 starter birds max 7; magic-link accounts; core interactions + notebook + visits (opt-in); a11y surfaces; server tick + sync.
- Ramp bird cap gradually (instrument drift/recognition first). Day-1 instrumentation: aggregate request counts, latencies, render timings, error rates, session-duration histograms (anonymized). No per-bird telemetry.
- Ship with all listed surfaces; no deferred a11y/perf.

## 12. Risks & Mitigations
- Drift calibration off (too fast → Tamagotchi feel; too slow → lifeless): instrumented measurement + user-visible drift in 3wks target; monotonic rule protects absence.
- Sync correctness / personality loss: server-only additive deltas + event order; synthetic UUID; soft-delete recovery; no client mutation.
- Audio uncanniness / canned feel: strict procedural WebAudio, no loops; recognizability cap at 7; chorus mixing.
- Accessibility regressions: first-class designed surfaces (narration prose, reduced-motion register) shipping v1; no ARIA-only fallback.
- Presence signal corruption ("tab open" instead of 3 conditions): strict conjunction definition; test harness enforces.
- Voice leakage (announcements/toasts/streaks): explicit non-goals + mater-of-fact exception named; notebook voice enforcement in generator.

All decisions defend "feels alive, notice never announce, charm from specificity, restraint, naturalist voice, presence is real interaction, no gamification/Tamagotchi/social-network." Plan is executable by frontier team without further clarification.