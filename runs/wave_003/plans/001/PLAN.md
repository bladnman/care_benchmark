# Pocket Aviary — Comprehensive Engineering & Implementation Plan (v1)

## 1. Scope and Architectural Non-Negotiables

### 1.1 In-Scope for v1
Pocket Aviary v1 is a browser-based, contemplative virtual aviary where idle attention is the primary mode of interaction.
- **Platform**: Evergreen web browsers (latest 2 versions of Chrome, Safari, Firefox, Edge). Responsive single horizontal scene without zooming or panning.
- **Population Lifecycle**:
  - Starting population: Exactly 2 starter birds randomly assigned from a 6-species pool.
  - Bird cap: Hard maximum of 7 birds per aviary (enforced by audio call discriminibability bounds).
  - Progressive adoption: Monotonically unlocked strictly by aviary age (calendar duration since creation: ~day 30 for bird 3,~day 90 for bird 4, etc.), refusing any engagement/visit-frequency gates.
- **Session Interactions**:
  - *Return-Greeting*: Procedural recognition of user arrival within 1-2 seconds, modulated by absence duration, bird boldness, and mood. Multiple birds greet with randomized staggered offsets.
  - *Presence Accounting*: Tri-condition attention detection (document visible, window focused, user activity within preceding 3 minutes).
  - *Listen-In*: Gradual audio mix rebalancing elevating focused bird to foreground while ducking others to ambient.
  - *Offers*: Gestures (seed, song fragment, still pool) with per-bird cooldowns affecting short-term mood and long-term curiosity/boldness.
  - *Settle Gesture*: Soft dusk transition with 5-second reversal window.
  - *Field Notebook*: Sparse naturalist observation log (~1 entry every few days).
- **Accounts, Sync & Privacy**: Passwordless email magic-link auth, synthetic UUID tenant isolation, authoritative server-side simulation tick (every 60s), single canonical aviary per account, real-time snapshot pull with cross-device consistency.
- **Quiet Social Affordance**: Single-guest read-only ambient view via revocable 30-day magic link; zero visitor presence drift impact; private visit log.
- **Accessibility & Performance Surfaces**: Continuous naturalist screen-reader narration, custom reduced-motion mode with tranquil cross-fades, real-time call captioning, WCAG AA contrast, comprehensive keyboard navigation. Initial JS bundle < 2MB gzipped, < 500ms time-to-first-bird, 60fps steady-state runtime with zero memory growth over 30 minutes.

### 1.2 Out-of-Scope (Explicit Non-Goals)
The engineering team must enforce absolute architectural boundaries against:
- No Native Mobile Clients: Web-only. No iOS, Android, or desktop wrappers.
- No Gamification or Streaks: No XP, leveling, streaks, green-dot activity calendars, badges, achievements, adoption counters, or visit counters.
- No Tamagotchi Dynamics: No hunger, sickness, distress, health bars, happiness meters, or mortality. Absence produces quiet, ambient behavior—never punishment or negative drift.
- No Social Network Features: No public directories, discovery feeds, profiles, follows, chat, comments, shared cursors/co-presence, or leaderboards.
- No Direct Numerical Exposure: Trait vectors are strictly internal server state and must never be exposed via API, DOM attributes, tooltips, or debug inspection panels.
- No Recorded Audio Fallback: Audio synthesis is client-side procedural WebAudio; fallback for unsupported audio environments is graceful silence with captions enabled.
- No Announcement UI: No toasts, greeting banners, or "welcome back" modals on the product surface.

---

## 2. System Architecture & Topology

Client/Server Boundary:
- **Client Application (Browser)**:
  - TypeScript + Canvas2D + WebAudio API.
  - Serves as an observational viewport and local synthesizer.
  - Renders birds at current perch positions with continuous micro-motion.
  - Renders ambient drift (leaves, feathers) procedurally.
  - Synthesizes calls from procedural motif grammars via WebAudio.
  - Tracks presence via local event hooks and batches interaction events to the server.
- **Edge Gateway & API Service**:
  - Manages magic-link authentication, session tokens, and route rate-limiting.
  - Ingests append-only interaction events into an event stream.
  - Serves cached aviary snapshots at the CDN edge (<500ms TTFBird).
  - Manages Server-Sent Events (SSE) channels for live aviary state updates.
- **Authoritative Simulation Service**:
  - Executes the authoritative simulation tick every 60 seconds per active aviary.
  - Evaluates the additive, monotonic personality drift equations.
  - Drives the Markov mood state machine according to diurnal local time and weather.
  - Generates sparse naturalist field notebook entries.
- **Data & Persistence Layer**:
  - PostgreSQL relational store for accounts, birds, aviaries, and notebook entries.
  - Append-only event log table preserving interaction provenance.

Synthetic Account Identity & Privacy Boundary:
- Every account is identified solely by an internal synthetic UUIDv4 (account_id). Email is encrypted at rest (AES-256-GCM) in a segregated table with deterministic HMAC-SHA256 blind indexing for lookup.
- Email is strictly prohibited in application logs, cache keys, telemetry, and message brokers.
- Operational telemetry collects only aggregate metrics (latencies, frame rates, tick runtimes) and is explicitly barred from containing per-bird state or interaction histories.

---

## 3. Data Models & Schema Design

Sqlte/PostgresQL Schema Overview:
- `accounts`: `id UUID PK, created_at TIMESTAMPTZ, timezone VARCHER(64), status VARCHAR(20), soft_deleted_at TIMESTAMPTZ, notification_on_visit BOOLEAN`
- `auth_identities`: `account_id UUID PK FK -> accounts.id, email_encrypted BYTEA, email_blind_index VARCHAR(64) UNIQUE`
- `sessions`: `id UUID PK, account_id UUID FK -> accounts.id, created_at TIMESTAMPTZ, last_active_at TIMESTAMPTZ, revoked BOOLEAN`
- `aviaries`: `id UUID PK, account_id UUID UNIQUE FK -> accounts.id, created_at TIMESTAMPTZ, version BIGINT, settled_at TIMESTAMPTZ, last_tick_at TIMESTAMPTZ`
- `birds`: `id UUID PK, aviary_id UUID FK -> aviaries.id, stable_index INT, species_id VARCHER(32), name VARCHAR(64), adopted_at TIMESTAMPTZ, perch_zone VARCHER(16), boldness FLOAT, social_warmth FLOAT, vocal_frequency FLOAT, plumage_saturation FLOAT, curiosity FLOAT, current_mood VARCHER(32), mood_updated_at TIMESTAMPTZ`
- `linteraction_eventsa: `id BIGSERIAL PK, aviary_id UUID, session_id UUID, event_type VARCHER(32), target_bird_id UUID, payload JSONB, created_at TIMESTAMPTZ`
- `notebook_entries`: `id UUID PK, aviary_id UUID, observed_at TIMESTAMPTZ, prose_content TEXT`
- `visit_invitations`: `id UUID PK, aviary_id UUID, token_hash VARCHAR(64) UNIQUE, invitee_email_encrypted BYTEA, created_at TIMESTAMPTZ, expires_at TIMESTAMPTZ, revoked BOOLEAN`
- `visit_logsa: `id UUID PK, invitation_id UUID, aviary_id UUID, started_at TIMESTAMPTZ, duration_seconds INT`

---

## 4. Simulation Engine Design

### 4.1 Server-Side 60-Second Tick
1. Queries all aviaries with active sessions or unprocessed events in the append-only event log.
2. Computes presence minutes validated by client tri-condition heartbeats.
3. Evaluates additive personality drift for all birds.
4. Runs Markov mood transitions based on local timeofday, current weather, and recent offers/1interactions.
5. Evaluates sparse field notebook generation rules.
6. Atomically commits new world snapshot and increments avairies.version.

### 4.2 Personality Drift Formulation & Monotonicity
- Personality traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity) are scalars in [0.0, 1.0].
- Presence-time is the dominant input; listen-in, accepted offers, and settle act as secondary inputs.
- Monotonic attention law: Delta_personality = max(0, alpha * presence_minutes + sum(beta_e * event_e)). Traits NEVER decay or drop on neglect. Absence produces ambient calm, not wariness or punishment.
- Calibration target: Detectable by test harness instruments after ~1 week of regular visits (630 presence minutes); visibly perceptible to the user after ~3 weeks (1890 presence minutes).

### 4.3 Fast-Timescale Mood State Machine
- Five core moods: WARY, CONTENT, CURIOUS, DROWSY, ALERT.
- Transitions modulated by:
  1. Diurnal Cycle: Dawn/morning boosts ALERT and CONTENT; evening/settled boosts DROWSY; night puts birds to sleep except nightjar species.
  2. Weather: Rain dampens vocal frequency and nods birds to DROWSY/quiet CONTENT; wind raises ALERT or WARY based on inverse boldness.
  3. Interactions: Offers accepted nudge toward CONTENT or CURIOUS; high-boldness birds resist WARY transitions.

### 4.4 Call Grammar & Chorus Runtime
- Each species has a motif library of 4-6 musical primitives.
- 
timing and cadence are shaped by vocal_frequency and current mood.
- Chorus assembly: When one bird calls, nearby birds with high social_warmth schedule responses within a staggered window (+600ms to +1800ms) to form harmonic chiming without phase-cancelling artifacts.

---

## 5. API Surface & Client-Server Sync Model

### 5.1 API Endpoints
- POST /api/v1/auth/magic-link (issues magic link, expires in 15 minutes)
- GET /api/v1/auth/verify (exchanges token for session JWT/cookie)
- GET /api/v1/aviary/snapshot (returns current aviary state for hydration)
- POST /api/v1/aviary/events (appends interaction events and presence heartbeats)
- GET /api/v1/aviary/stream (SSE stream for tick updates, weather, and notebook entries)
- POST /api/v1/aviary/settle (instantly settles aviary)
- GET /api/v1/notebook (reads paginated notebook entries)
- POST /api/v1/social/invites (creates 30-day read-only guest invite)
- DELETE /api/v1/social/invites/:id (revokes invitation)
- GET /api/v1/social/visits/:gest_token (read-only viewport endpoint)

### 5.2 Conflict Prevention & No-Last-Write-Wins
- Server is the sole author of world state. Clients never mutate personality or mood directly.
- Clients write interaction events into an idempotent append-only log.
- Multi-device consistency is infinitely robust: because neither laptop nor phone sends absolute state, neither can overwrite the other's attention. In-flight delays simply get processed at the next tick.

---

## 6. Frontend Rendering & Audio Pipelines

### 6.1 Canvas2D Scene Composition
- Single horizontal scene fitting wide to narrow viewports without cropping birds or panning.
- Three perch zones:
  1. Back plane (depth 0.3, muted colors, ambient)
  2. Middle plane (depth 0.6, primary resting branches)
  3. Front rail (depth 1.0, foreground, crisp detail, bold birds)
- First frame aliveness: Birds load mid-action at snapshot positions. Upon cold network fetch, a tranquil ambient sky field is displayed (never a spinner) before birds gently appear.
- Idle micro-motion: Saccadic glances, head tilts, feather fluffing, preening, and subtle breathing oscillation.

### 6.2 Reduced-Motion Mode
- Automatically honored via `prefers-reduced-motion` or settings toggle.
- Not an "animations off" degradation, but a custom designed surface:
  - Micro-motions and perch flights are replaced by slo tranquil cross-fades (600ms opacity blends) between distinct resting poses.
  - Ambient leaf/feather drift is removed.
  - Audio synthesis, call captions, mood shifts, and notebook narration remain fully active.

### 6.3 Procedural Audio Pipeline (WebAudio)
- Client-side synthesis: OscillatorNodes (sine, triangle, light FM) pledged through BiquadFilterNodes and GainNode ADSR envelopes generate every call at runtime from motif grammars. Zero recorded audio files.
- Dynamic Chorus & Listen-In Mix:
  - Ambient chorus plays at calibrated baseline levels (-18dBfS to -12dBfS).
  - Listen-in ramps focused bird to 0dZfS while ducking all other birds to -28dBfS over 800ms.
  - Smooth 1200ms exponential decay returns the chorus to baseline upon disengaging.
- Fallback: If WebAudio is unvailable or blocked, the aviary plays in graceful silence with call captions automatically enabled.

---

## 7. Interaction Protocols & UR Specifications

### 7.1 Return-Greeting Protocol
- Evaluated within 1-2 seconds of tab arrival.
- Based on absence duration, bird boldness, and mood.
- Short absences produce a glance or soft head-tilt; longer absences prompt a hop to the front rail and a full species call.
-Staggered responses from secondary birds ensure natural organic flow.
- Absolute prohibition against announcement UI: No greeting toasts, banners, or welcome-back text.

### 7.2 Presence Accounting Protocol
- Tri-condition attention detection:
  1. `document.visibilityState === 'visible'`
  2. `document.hasFocus() === true`
  3. User pointer/keyboard/click event recorded within the last 180 seconds.
- All three must simultaneously hold for presence to accrue. Pings dispatched every 60s.
- Background tabs and unattended monitors accrue zero presence.

### 7.3 Offers & Settle
- Offers (seed, song fragment, still pool) accessible from the fading top bar.
- Per-bird cooldown of 3 minutes prevents saturating traits and preserves gestural meaning.
- Settle gesture shifts lighting to warm evening dusk and quiets the chorus. A 5-second reversal window allows recovery from accidental clicks.
- No streak counters, no visit-frequency widgets, no green-dot calendars.

---

## 8. Accessibility & Voice Separation

### 8.1 Screen-Reader Narration
- Running `aria-live="polite` prose narration updated every 30-60 seconds.
- Written in naturalist voice (lowercase, present-tense, specific): e.g., "wren is on the low perch this morning, fluffed against the cool air. pip greeted first today."
- User-initiated events (offers, greetings) receive polite priority bumps.

### 8.2 Call Caption Surface
- Real-time subtitle badges rendered adjacent to calling birds, describing the procedural motif (e.g., "pip: a soft three-note rise").

### 8.3 Keyboard Navigation & Contrast
- Tab/Shift+Tab traverses top bar controls; Arrow keys traverse birds; Enter activates Listen-In; Escape exits.
- Focus rings and user-copy text pass WCAG AA contrast ratios under all diurnal lighting conditions.

### 8.4 Voice Separation
- **Naturalist Voice**: All product surfaces (aviary, notebook, narration, captions). Lowercase, present-tense, quiet, specific, no gamification jargon.
- **Matter-of-Fact Voice**: All system, auth, error, sync conflict, and accessibility settings surfaces. Capitalized, direct, clear, no affected warmth.

---

## 9. Performance Budgets & Observability

### 9.1 Performance Budgets
- Initial JS Bundle Size: < 2.0 MB gzipped.
- Time to First Bird Visible (TTFBird): < 500ms on mid-tier mobile over 4G.
- Steady-State Frame Rate: 60 fps on a5-year-old laptop.
- Memory Invariant: 0 MB heap growth across a 30-minute session.
- Simulation Tick Latency: p99 < 5.0 seconds.

### 9.2 Observability & Privacy
- Operational telemetry measures system health (request counts, latencies, errors, render frame drops, WebAudio errors).
- Privacy Boundary: No per-bird state, presence logs, or interaction events are ever exported to analytics warehouses or aggregated across accounts.
---

## 10. Rollout Strategy, Population Ramp & Risks

### 10.1 Rollout Phases
- Phase 1: Server simulation tick, additive drift math, and WebAudio motif synthesis.
- Phase 2: Canvas2D viewport renderer, return-greeting, listen-in, and offers.
- Phase 3: Magic-link authentication, SSE sync, and read-only visit links.
- Phase 4: Screen-reader narration, reduced-motion crossfades, call captioning, and 30-minute memory leak verification.

### 10.2 Population Ramp
- Day 1: 2 starter birds.
- Day 30: Bird 3 (unlocked by aviary age).
- Day 90: Bird 4.
- Day 180: Bird 5.
- Day 270: Bird 6.
- Day 360: Bird 7 (hard maximum cap).

### 10.3 Key Risks & Mitigations
- Drift Calibration Risk: Synthetic automated session tests run 90 days of accelerated presence in CI to verify instrument drift at 1 week and perceptible drift at 3 weeks.
- Audio Fatigue / Uncanniness: Procedural micro-variations in pitch, cadence, and offsets eliminate phase-cancelling and loop artifacts.
-rore/Sync Overwrites: Clients never mutate state; additive server-side event ingestion prevents drift loss across devices.
- Voice Contamination: Architectural linters prevent naturalist prose in system/sync errors and prevent system jargon in product surfaces.
