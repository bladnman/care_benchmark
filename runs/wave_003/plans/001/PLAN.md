# Phase 1 Implementation Plan — Pocket Aviary (V1)

## Scope

V1 includes:
- Two starter birds per aviary (expandable to seven based on aviary age)
- Single-user accounts with magic-link sign-in
- Multi-device sync via server-side simulation tick
- Field notebook (read-only, auto-generated naturalist observations)
- Presence accounting (comprehensive, conjunction-based)
- Visit invitations (off-by-default, read-only guest viewing)
- Audio system with procedural calls and chorus mix
- Screen-reader narration (naturalist voice)
- Reduced-motion mode (cross-fade rendering)
- Call captioning (proximate prose)
- Keyboard navigation support
- Matter-of-fact error surfaces (account, sync, browser support)

V1 excludes (respecting PRD non-goals):
- Native mobile apps
- Gamification elements (scores, achievements, streaks, levels)
- Tamagotchi mechanics (bird death, hunger, distress meters)
- Social network surfaces (profiles, follows, public discovery)
- Shared/collaborative aviaries
- Custom scene layout
- Payment systems
- Notifications (push, email)
- Clone/reset bird functionality

## Architecture

**Client-server model:** Server-side simulation tick (~1min cadence) is the single source of truth for bird personality vectors, mood, and state. Clients pull state snapshots and interpolate for smooth rendering. Clients write interaction events to append-only server log.

**Service boundaries:**
- Simulation Service: Runs continuous tick, updates personality drift, transitions moods
- API Gateway: Handles auth, state snapshots, event submission
- Identity Service: Magic-link authentication, account management
- Asset Service: Bird species pool, motif libraries (static)
- Analytics/Telemetry: Aggregate-only monitoring (no per-bird state)

**Data flow:** Client → API (auth events, interaction events) → Server (tick engine) → Server (state store) ← Server (state snapshot) ← Client

**Frontend rendering pipeline:** WebAudio for procedural calls, Canvas/WebGL for bird sprites, CSS animations for transitions, server state interpolation for smooth motion.

## Data Model

**Birds:** Unique server UUID, name (user-editable), species (from 6-member pool), personality vector (5 traits: boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood (wary/content/curious/drowsy/alert), drift history (cumulative).

**Accounts:** UUID, email (encrypted), session tokens (revocable), visit log, pending invitations, subscriptionless.

**Interactions:** Events (presence, listen-in, offer, settle, greetings) with timestamps, bird/target identifiers.

**Notebook:** Auto-generated entries (naturalist prose, lowercase, present tense) with timestamps, tied to observable events.

**Personality Drift:** Server-computed daily-ish deltas from presence-time, listen-in duration, offers, with monotonic tendency (never decreases).

**Mood:** Fast-timescale enum per bird, server-computed from personality, recent interactions, time-of-day, ambient events.

## API Surface

**Authentication:** POST /auth/magic-link, GET /auth/verify?token

**State Management:** GET /api/aviary (returns snapshot), GET /api/events (since N)

**Interactions:**
- POST /api/events/listen-in (focus bird)
- POST /api/events/offer (submit offer ID)
- POST /api/events/settled (soft session end)
- GET /api/events/presence (server-side presence ping)

**Account Settings:** GET/PUT /api/account (profile), GET /api/invitations, POST /api/invitations, DELETE /api/invitations/{id}

**Visit Access:** GET /api/visit/{token} (read-only, ambient playback)

**Export:** GET /api/export (generate and email JSON snapshot)

All requests authenticated via session token; API returns JSON with metadata.

## Simulation Engine Design

**Tick Engine:** Runs server-side every ~60 seconds, regardless of client connectivity:
- Read recent interaction events (last 60min)
- Compute personality deltas: presence-time major weight, listen-in secondary, offers minor
- Apply deltas (monotonic toward expressive, never negative)
- Update mood from personality, time of day, ambient events
- Write new personality vectors and mood to canonical state

**Drift Function:** Low-pass filter across presence-and-interaction signals, visible drift expected after 3 weeks of regular use, instruments detect after ~1 week.

**Presence Detection:** Client ping on document visibility change + window focus + pointermove/keypress in last few minutes (conjunction required).

**Mood Transitions:** Deterministic based on personality traits, time zone, and ambient events (rain dampens vocal frequency, other birds trigger wary responses).

**Call Grammar:** Procedural WebAudio synthesis based on bird species motif library, personality-shaped timing and pitch, recognizability maintained across drift.

## Sync Model

**Canonical State:** Single server-side record per account; no client-side personality ownership.

**Multi-device sync:** Transparent via server as single source of truth; both devices read identical snapshots.

**Conflict prevention:** No last-write-wins; additive server-authored deltas processed in event-log order. Clients never write personality directly.

**Capture scenarios:** Magic-link replay, session timeout mid-write handled via idempotency keys; failed ticks retry with exponential backoff; sync errors surface via matter-of-fact error pages.

## Frontend Rendering Pipeline

**Scene composition:** Three-zone layout (front/middle/back), responsive across viewport sizes, single horizontal scene (no panning/scrolling).

**Micro-motion:** Mood-shaped idle animations (preening, scanning, head-tilting), ambient leaf/feather drift, cross-fades between states.

**Transitions:** Window focus recovery loads fresh state snapshot, greet animations stagger procedurally, listen-in mix changes slow-ramp.

**Reduced-motion mode:** Cross-fade between still poses, removed leaf drift, ambient color shifts slowed, calls/captions continue full quality.

**Performance:** Initial JS bundle <2MB gzipped, first bird visible <500ms, 60fps idle on 5-year laptop, no memory growth over 30min.

## Audio Pipeline

**Synthesis:** Procedural WebAudio using motif library per bird species, personality-driven timing (vocal frequency trait), mood-appropriate motifs.

**Chorus mixing:** Real-time mixing of procedural calls, focused bird rises in mix, others quiet to ambient but never silent.

**Listen-in:** Gradual mix level changes, smooth engagement/disengagement, no hard cuts.

**Fallback:** WebAudio unavailable → graceful silence with captions on by default, no recorded audio fallback.

## Accessibility Surfaces

**Screen-reader:** Server/client generated naturalist prose (~30-60s cadence), running narration (not state list), user-initiated events get queue priority.

**Keyboard navigation:** Tab through top bar → aviary → birds; arrow keys move focus; Enter triggers listen-in; Escape exits.

**Reduced-motion:** Cross-fade rendering replaces frame-by-frame animations, maintained charm, same state transition logic.

**Captions:** Prose descriptions per call (e.g., "soft three-note rise"), fade with call, same voice as notebook.

**WCAG AA:** All user-copy text meets contrast; focus indicators visible against aviary backgrounds; keyboard reachability tested.

## Performance Budgets and Observability

**Load performance:** Bundle size ≤2MB (gzipped), first bird visible <500ms on mid-tier 4G.

**Runtime:** 60fps idle motion on 5-year laptop, no memory growth over 30-minute sessions.

**Simulation tick:** p99 latency ≤5 seconds, alert on breach.

**Observability metrics:** Request counts, latencies, error rates, anonymized session durations, render-frame timings, audio-context errors.

**Privacy boundary:** No per-bird state in aggregate telemetry; simulation database isolated from analytics warehouse; telemetry can never infer interaction history.

## Rollout

**V1 ship strategy:** Launch with two birds, core bird engine, basic interactions (listen-in, offer, settle), accounts and sync, field notebook.

**Birds-per-aviary ramp:** Start at 2 birds, offer third at ~3 months aviary age, fourth at ~6 months, etc. New birds draw from same species pool, no catalog selection.

**Instrumentation:** Core KPIs: session retention, drift visibility timeline, user-reported feeling of aliveness, error rate, performance budgets. Telemetry focuses on operational health only.

**Rollout path:** Internal staging → canary deployment (5% of users) → full release; monitoring bangup for any performance regressions; rollback immediate on drift correctness issues.

## Risks

**Drift calibration:** Over- or under-sensitive drift leads to birds feeling static or overly reactive; mitigate with extensive A/B testing and phased rollout.

**Sync correctness:** Server tick lag creates state divergence across devices; prevent via single source of truth and robust event log ordering.

**Audio uncanniness:** Procedural calls sounding too synthetic or robotic; mitigate with extensive voice testing and personality-based variation.

**Accessibility regressions:** Accessible surfaces losing affective core; ensure accessibility work ships with main product, not as afterthought.

**Performance budget breach:** Bundle size or render latency issues; implement strict CI checks with performance budgets enforced at merge.

## Timeline assumptions

Based on 3-person engineering team with full-stack capabilities, this plan targets V1 launch within 6-8 months, allowing time for drift calibration, accessibility work integration, and performance optimization. The plan leaves room for iterative improvement post-launch, with core architecture designed for evolutionary scaling without compromising the foundational principles of the product.
