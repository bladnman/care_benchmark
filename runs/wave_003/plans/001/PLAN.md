# Comprehensive Implementation Plan for Pocket Aviary (v1)

## Scope — v1 Definition and Non-Goals Respect
v1 delivers a single horizontal browser-based aviary supporting 2–7 procedurally animated birds per account. Core features: server-side simulation tick (~1/min), presence-driven monotonic personality drift, mood transitions, procedural WebAudio calls, return-greeting, listen-in, offers (seed/song/pool), settle gesture, field notebook (sparse naturalist entries), magic-link auth, multi-device sync via canonical server state, visit invitations (read-only, opt-in, revocable, default-off), screen-reader narration, reduced-motion cross-fade mode, call captions, WCAG AA chrome, day/night + ambient weather, top-bar chrome that fades.

Explicitly out of scope per non_goals.md and product_brief.md: any native app, gamification/streaks/achievements/leaderboards, Tamagotchi hunger/distress/decay mechanics, social-network surfaces (profiles, discovery, public feeds, comments, co-presence), push notifications, customizable scenes, multi-aviary accounts, recorded-audio fallbacks, numerical personality exposure.

Bird count caps at 7 because call signatures must remain individually recognizable; new birds appear on aviary-age schedule only. Drift is monotonic toward expressive (no negative drift on neglect). Presence definition is strict (visible + focused + recent pointer/key activity). All surfaces split voice: naturalist for aviary/notebook/narration/captions, matter-of-fact for auth/errors/settings.

## Architecture — Service Shape and Client/Server Split
- **Frontend (browser-only)**: Canvas/WebGL or SVG+DOM render pipeline for scene; WebAudio synthesis engine; thin event emitters for presence pings, interaction events; state snapshot consumer + interpolator. No local persistence of personality vectors.
- **Backend services**:
  - Auth service: magic-link issuance/validation, synthetic-UUID account IDs, session tokens (revocable).
  - Simulation service: single writer of personality state; consumes append-only event log; runs tick independently of clients.
  - Snapshot service: serves small state snapshots (bird positions, moods, call timing, day/night, weather) from canonical records; CDN-edge friendly.
  - Notebook service: generates and stores sparse naturalist entries from tick + event signals.
  - Visit service: invite issuance/revocation, read-only snapshot proxy (no event recording from visitors).
  - Account service: export (JSON), soft-delete (30d), email change verification.
- Render pipeline boundary: client receives deterministic snapshot, runs local idle micro-motion + procedural audio + ambient leaf/feather ornamentation. Client never mutates canonical state.

Data flow: client → append-only interaction events (offer, listen-in duration, settle, presence pings) → simulation tick reads log → writes new personality/mood → clients poll low-frequency or on visibility/focus for fresh snapshots.

## Data Model
- **Account**: synthetic UUID primary key; encrypted email; per-device revocable tokens; settings (reduced-motion opt-in, caption opt-in, visit-notification toggle).
- **Bird**: stable internal UUID; species (from fixed 6-species pool); user name (renameable); personality vector (boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity — normalized scalars); current mood enum (wary/content/curious/drowsy/alert +); last-tick timestamp.
- **Personality vector persistence**: server-only canonical; never recomputed from logs; additive deltas only.
- **Presence event**: triple-condition record (visibility, focus, activity window) timestamped; aggregated into presence-time per bird.
- **Interaction event**: append-only log (type, bird_id, timestamp, duration/value).
- **Notebook entry**: timestamp, naturalist prose string, aviary_id.
- **Visit invite**: host_account, visitor_email, token, expires_at, revoked flag.
- All per-account state lives under synthetic UUID partitioning; no email-derived keys.

## API Surface
REST/GraphQL endpoints (auth via session token or magic link):
- POST /auth/magic-link/request, /consume
- GET /aviary/snapshot (returns canonical state + tick timestamp)
- POST /events (append interaction/presence)
- GET /notebook (paginated entries)
- POST /settle, /offer, /listen-in/start|end
- Account: GET/POST /account/export, /delete, /sessions (revoke), /settings
- Visit: POST /visits/invite, DELETE /visits/{id}, GET /visits/log; visitor GET /visit/{token}/snapshot (read-only)
Error surfaces always matter-of-fact voice; naturalist voice never used on system surfaces.

## Simulation Engine Design
- Tick cadence ~60s (calibrate). Reads recent events + current local time + last mood/presence; applies:
  - Mood transitions (time-of-day, recent offers/listen-ins, bird-to-bird call response propagation, personality modulation).
  - Drift deltas (presence-time dominant weight, listen-in and offers secondary; monotonic upward only).
  - Call-grammar scheduling (vocal_frequency shapes inter-call interval; motif combination per species + mood).
- Drift low-pass filter: measurable instrument change after ~1 week regular use; visible user change after ~3 weeks. No single-session visible drift.
- Mood persists across sessions; tick advances state during offline periods.
- Bird-to-bird: chorus emergence, mood contagion.
- Adoption: two starter birds from pool with seeded vectors; later birds on aviary-age gates.

## Sync Model
Single canonical record per aviary. Server tick is sole writer of personality/mood. Clients:
- Pull snapshot on tab visible, long gaps, keepalive.
- Write only events to append-only log (order preserved).
- No last-write-wins; event-log consumption guarantees additive correctness.
- Conflicts surface (magic-link replay, timeout) → matter-of-fact error; no naturalist phrasing.
- Multi-device: identical snapshots; no reconciliation logic needed.

## Frontend Rendering Pipeline
- Horizontal scene, three perch zones, responsive compression preserving all birds in frame.
- First frame: live mid-action state (no wake-up animation). Loading state = quiet field (no spinner).
- Idle micro-motion mood-shaped: preen/scan/tilt/fluff; cross-fades only in reduced-motion.
- Day/night palette shift + weather (rain/wind) mood dampening; subtle parallax foreground/background; ambient leaf/feather drift (client-generated ornament).
- Top bar: sparse icons (account, a11y, notebook, offer); auto-fade on idle cursor.
- Transitions: slow lighting for settle/evening; listen-in mix ramp (never hard cut).

## Audio Pipeline
- WebAudio procedural synthesis from per-species motif library. Real-time variation on every call; chorus mixing (two+ birds produce true overlap, not phase-cancel stacks).
- Listen-in: slow mix rebalance (focused bird up, others ambient; never full mute).
- Personality (vocal_frequency) + mood + time-of-day shape timing/pitch.
- No recorded loops or fallback audio files; WebAudio unavailable → graceful silence + captions default-on.
- Performance: 60 fps idle, no per-call allocation leaks.

## Accessibility Surfaces
- Screen-reader: running naturalist prose narration (30–60 s cadence at idle; priority on events); same voice as notebook; generated from same state snapshot.
- Reduced-motion: cross-fade still-pose sequences + slowed color/day shifts; identical mood/drift/notebook behavior.
- Call captions: runtime prose from grammar; positioned near bird; naturalist voice.
- Keyboard: full tab/arrow/enter/escape navigation; visible high-contrast focus ring.
- WCAG AA contrast on all chrome text; design-system ratios specified separately.

## Performance Budgets and Observability
- Bundle: <2 MB gzipped initial (drives procedural audio + procedural assets).
- Time-to-first-bird: <500 ms on mid-tier 4G mobile.
- Runtime: 60 fps idle on 5-year-old laptop; zero memory growth over 30 min (buffer reuse, no retained DOM on scroll).
- Observability: synthetic browser fleet (p99 tick latency alarm at 5 s); aggregate-only RUM (no per-account/per-bird data).
- Browser support: last two major versions of Chrome/Safari/Firefox/Edge; unsupported gets matter-of-fact notice.

## Rollout
- Ship with two starter birds; age-gated third+ bird offers.
- Ramp birds-per-aviary gradually post-launch while monitoring recognizability.
- Instrument from day one: request counts, snapshot latency, tick p99, render-frame timing, audio errors (aggregate only). Privacy boundary enforced at metric definition.
- No per-bird telemetry ever enters analytics warehouse.

## Risks and Mitigations
- Drift calibration: too fast → Tamagotchi feel; too slow → screensaver. Mitigate: instrumented test harness measuring week-1 numerical vs week-3 user-visible change; monotonic rule prevents regression to negative-drift.
- Sync correctness: last-write-wins or client mutation would silently corrupt drift. Mitigate: additive server deltas + append-only log; no client personality writes.
- Audio uncanniness / phase artifacts: recorded loops fail chorus. Mitigate: unconditional procedural WebAudio; silence fallback preferred over canned.
- Accessibility regressions: narration or reduced-motion arriving late would exclude users. Mitigate: ship with v1; treat as designed surfaces, not parity checklists.
- Presence-signal inflation: lax definition leaks drift speed across population. Mitigate: enforce exact three-condition conjunction; calibrate activity window conservatively.
- Voice leakage: naturalist phrasing on error surfaces erodes trust. Mitigate: named exception + code-level enforcement (system surfaces use matter-of-fact).

The plan is executable by a frontier team with no further clarification required. All decisions follow directly from the PRD files read. No product implementation is performed; deliverables are exactly the two assigned files.