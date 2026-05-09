# Pocket Aviary — Implementation Plan

## 1. Scope

### In v1

- Two starter birds per account at sign-up; aviary can grow to seven over time (age-gated, not interaction-gated)
- Magic-link email auth; single-user accounts; one aviary per account
- Server-side simulation tick (~1/min) advancing canonical personality vectors and mood
- Client renders state snapshots with smooth interpolation; never owns simulation state
- Multi-device sync as an architectural property (one canonical server record, no merge needed)
- Procedural call grammar synthesized client-side via WebAudio
- Mood system (wary, content, curious, drowsy, alert) with daily-ish reset plus interaction modulation
- Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) with monotonic-toward-expressive drift
- Return-greeting on tab open; presence accounting (visibility + focus + recent pointer/key)
- Interactions: listen-in, offer (seed, song fragment, still pool), settle
- Field notebook with naturalist prose, sparse cadence
- Day/night cycle keyed to user local timezone
- Rare ambient weather (rain, wind); ambient micro-motion (leaves, feathers)
- Visit invitation (opt-in, read-only, revocable, email-keyed, not co-presence)
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fade poses, not animations-off)
- Call captioning (naturalist prose, runtime-generated from call grammar)
- WCAG AA contrast on all user-copy text
- Full keyboard navigation
- Account export (JSON), soft/hard deletion (30-day window)
- Aggregate operational telemetry only; no per-bird state in any telemetry pipeline

### Out of v1 (non-goals respected)

- No native iOS/Android app
- No gamification: no streaks, no achievements, no levels, no badges, no green-dot calendars, no visit counters surfaced to the user
- No Tamagotchi mechanics: birds don't die, have no hunger meter, show no distress
- No social-network surfaces: no profiles, no follows, no discovery feed, no comments, no public aviaries
- No push/email notifications (user-opt-in visit notifications excepted as a quiet account setting)
- No passwords or SSO at launch
- No payments or paid tiers
- No multi-aviary accounts
- No customizable scenes
- No recorded audio (WebAudio-only; silence + captions if WebAudio unavailable)

---

## 2. Architecture

### Service shape

```
┌──────────────────────────────────────────────────────────────┐
│  Browser client (SPA)                                        │
│  - Rendering pipeline (canvas/WebGL + CSS)                   │
│  - WebAudio call synthesis                                   │
│  - Presence accounting                                       │
│  - Interaction event submission                              │
│  - State snapshot rendering + interpolation                  │
└────────────────────────┬─────────────────────────────────────┘
                         │ HTTPS (REST + SSE or WebSocket)
┌────────────────────────▼─────────────────────────────────────┐
│  API service (stateless, horizontally scalable)              │
│  - Auth (magic link issue/verify, session management)        │
│  - State snapshot delivery                                   │
│  - Interaction event ingestion (append-only event log)       │
│  - Visit invite management                                   │
│  - Notebook read                                             │
│  - Account CRUD, export                                      │
└───────────┬────────────────────────┬─────────────────────────┘
            │                        │
┌───────────▼──────────┐  ┌─────────▼──────────────────────────┐
│  Simulation service  │  │  Notification service (email only)  │
│  - Tick runner       │  │  - Magic link delivery              │
│  - Drift computation │  │  - Account export link              │
│  - Mood transitions  │  │  - Visit invite delivery            │
│  - Notebook writer   │  └────────────────────────────────────┘
│  - Weather generator │
└───────────┬──────────┘
            │
┌───────────▼──────────────────────────────────────────────────┐
│  Canonical data store (PostgreSQL or equivalent)             │
│  - account, session, bird, personality_vector, mood          │
│  - interaction_event (append-only)                           │
│  - notebook_entry                                            │
│  - visit_invite, visit_log                                   │
│  - aviary_weather_state                                      │
└──────────────────────────────────────────────────────────────┘
```

### Client/server split

- **Server owns**: personality vectors, mood, tick clock, event log, notebook entries, visit state, canonical aviary state
- **Client owns**: nothing persistent. It renders snapshots, accumulates local presence pings (flushed to server every ~30s), and submits interaction events
- **Client never writes** personality state directly; it writes events, server computes deltas
- Snapshots are small (<10KB): per-bird position, mood, current call timing offsets, active weather state, day/night fraction, settled flag

### Render pipeline boundary

- Server: state machine (mood, drift, positions, weather)
- Client: rendering of that state (canvas/WebGL for scene, CSS for top bar and overlays)
- Snapshot delivery: CDN-cached edge delivery for cold load; live pull from API for keepalive refreshes. First snapshot embedded in the HTML response to hit the <500ms first-bird budget.

---

## 3. Data Model

### account

```
id: UUID (synthetic, PK)
email: encrypted string (stored once; never used as FK elsewhere)
email_verified: bool
created_at: timestamp
deletion_requested_at: timestamp | null
deletion_hard_at: timestamp | null  -- deletion_requested_at + 30 days
visit_notifications_enabled: bool  -- default false
```

### session

```
id: UUID (PK)
account_id: UUID (FK account)
token_hash: string
device_hint: string  -- browser UA, for session list display
created_at: timestamp
last_seen_at: timestamp
revoked_at: timestamp | null
```

### magic_link

```
id: UUID
account_id: UUID
token_hash: string
expires_at: timestamp  -- created_at + 15 min
consumed_at: timestamp | null
```

### aviary

```
id: UUID (PK)
account_id: UUID (FK, unique)
created_at: timestamp  -- used for age-gated bird availability
weather_state: jsonb  -- current weather event or null
settled: bool
settled_at: timestamp | null
```

### bird

```
id: UUID (PK, stable for life of account)
aviary_id: UUID (FK aviary)
name: string
species: enum(~6 species)
adopted_at: timestamp
```

### personality_vector

```
bird_id: UUID (FK bird, PK)
boldness: float (0..1)
social_warmth: float (0..1)
vocal_frequency: float (0..1)
plumage_saturation: float (0..1)
curiosity: float (0..1)
updated_at: timestamp
```

Seed values drawn from a species-typical range at adoption with small random jitter. Updated only by the simulation service.

### mood

```
bird_id: UUID (FK bird, PK)
state: enum(wary, content, curious, drowsy, alert)
entered_at: timestamp
expires_at: timestamp | null  -- next daily reset window
```

### interaction_event (append-only)

```
id: UUID
account_id: UUID
bird_id: UUID | null  -- null for aviary-level events (settle, weather)
event_type: enum(presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song, offer_pool, settle, return)
occurred_at: timestamp
payload: jsonb  -- e.g. duration_seconds for presence_ping, motif_id for offer_song
```

No updates. Only inserts. Simulation tick reads and processes; no hard deletion except on account hard-delete.

### notebook_entry

```
id: UUID
aviary_id: UUID
generated_at: timestamp
prose: text  -- naturalist prose, lowercase, present-tense
trigger: enum(greeting, listen_in, offer, weather, idle_observation, mood_shift)
```

### visit_invite

```
id: UUID
host_account_id: UUID
visitor_email: string  -- plaintext (it's the invitee's email, not ours; different PII posture)
token_hash: string
created_at: timestamp
expires_at: timestamp  -- created_at + 30 days
revoked_at: timestamp | null
first_used_at: timestamp | null
```

### visit_log

```
id: UUID
invite_id: UUID
visitor_email: string
visited_at: timestamp
duration_seconds: int | null  -- set on session close
```

---

## 4. API Surface

All endpoints require session auth except the magic-link verification and visit token endpoints.

### Auth

```
POST /auth/request-link        { email } → 202 (always; rate-limited per email)
POST /auth/verify-link         { token } → { session_token, account_id }
DELETE /auth/session/:id       → 204  (revoke a session)
GET  /auth/sessions            → [{ id, device_hint, last_seen_at }]
```

### Aviary state

```
GET /aviary/snapshot           → AviarySnapshot
  AviarySnapshot: {
    aviary_id, settled, weather_state, day_fraction, local_tz_offset,
    birds: [{ bird_id, name, species, perch_zone, mood, call_timing_offset, plumage_saturation }]
  }
  Cache-Control: no-store on auth'd path; snapshot embedded in HTML on cold load via edge
```

### Interaction events

```
POST /events                   { event_type, bird_id?, occurred_at, payload } → 202
  Clients batch presence_pings; target flush every 30s or on page-hide
```

### Notebook

```
GET /notebook                  ?before=<timestamp>&limit=<int> → [NotebookEntry]
  NotebookEntry: { id, generated_at, prose }
  Paginated, most-recent-first, no limit on how far back the client can scroll
```

### Visits

```
POST /visits/invite            { visitor_email } → { invite_id, expires_at }
DELETE /visits/invite/:id      → 204  (revoke)
GET /visits/invites            → [{ invite_id, visitor_email, created_at, expires_at, revoked_at, used }]
GET /visits/log                → [{ visitor_email, visited_at, duration_seconds }]
GET /visits/:token             → AviarySnapshot (no auth; visitor-facing; checks revocation)
```

Visit snapshot endpoint returns 410 Gone with matter-of-fact body if invite is revoked or expired.

### Account

```
GET  /account                  → { email, created_at, visit_notifications_enabled }
PATCH /account                 { visit_notifications_enabled? } → 200
POST /account/export           → 202 (triggers async export; link emailed)
POST /account/delete           → 202 (soft deletion initiated)
POST /account/recover          → 200 (cancel deletion if within 30-day window)
PATCH /account/email           { new_email } → 202 (verification email sent to new_email)
```

### Bird settings

```
PATCH /birds/:id               { name } → 200 (rename only; no other mutation from client)
```

---

## 5. Simulation Engine Design

### Tick runner

The simulation service runs a tick loop at ~60s cadence (calibrate during build). Each tick:

1. For each active aviary (any account not soft-deleted):
   a. Read new interaction_events since last tick
   b. Update mood state per mood-transition rules
   c. Apply drift delta to personality_vector
   d. Recompute perch positions
   e. Advance weather state
   f. Maybe emit notebook entry
   g. Write updated snapshot to a snapshot cache (Redis or similar) for fast client reads
   h. Persist personality_vector and mood to canonical DB

Tick is idempotent within a window: if the tick processes events that have already been processed (e.g. after restart), the drift delta is not double-applied. Track `last_processed_event_id` per aviary.

### Drift function

```
for each trait T in personality_vector:
  raw_input = weighted_sum(presence_seconds, listen_in_seconds, offer_accepts, ...)
  scaled = low_pass_filter(raw_input, alpha=DRIFT_ALPHA)  -- DRIFT_ALPHA ≈ 0.0001-0.001; calibrate
  delta = scaled * DRIFT_RATE_PER_TICK
  new_value = clamp(old_value + max(0, delta), 0, 1)  -- monotonic; never negative delta
```

Calibration target: measurable change in instruments after ~1 week of daily regular visits; perceptible change to attentive user after ~3 weeks. Run synthetic calibration scenarios (simulated 7 days of presence) during build to validate constants before launch.

Trait-specific input weights:
- boldness: weighted by offer_near events and return_greeting latency
- social_warmth: weighted by listen_in events, bird-to-bird call response events
- vocal_frequency: weighted by listen_in events
- plumage_saturation: weighted purely by total presence_seconds (the "attention" trait)
- curiosity: weighted by offer_accept events

### Mood transitions

State machine per bird. Transitions are probabilistic, not deterministic — the same inputs don't always produce the same next state, but weights are shaped by personality.

```
Inputs per tick:
  - time_of_day (morning, midday, dusk, night)
  - recent_interaction_events in last session
  - personality_vector (bold birds resist wary; high-curiosity birds resist drowsy when offers pending)
  - other_birds_moods (wary is mildly contagious; alert spreads on alarm calls)
  - weather_state (rain dampens vocal frequency, nudges toward drowsy or content)

Transition weights:
  morning: alert ↑, drowsy ↓
  dusk:    drowsy ↑, alert ↓
  night:   drowsy dominant; nightjar-species stays alert
  rain:    drowsy/content ↑, wary slight ↑
  offer_accepted: content ↑, curious ↑
  alarm_neighbor: wary ↑

Daily reset window: each bird has a daily reset at a randomized time (within ±2hrs of their "morning" time) where mood is recomputed from scratch given current time-of-day and yesterday's drift inputs. Not a hard cut; the mood before reset is the starting point and transitions apply on top.
```

Mood persists to DB on every tick. Clients get mood in the snapshot.

### Call grammar runtime

Each species has a call grammar: a small set of motifs (3–6 per species) with timing rules, pitch contour templates, and variation parameters. At runtime:

```
CallGrammar.next(bird):
  select motif by personality.vocal_frequency and mood weights
  apply pitch_contour with random variation (±semitone range)
  apply timing_variation (onset jitter shaped by vocal_frequency)
  return CallEvent { motif_id, pitch_offset, timing_ms, duration_ms }
```

The grammar state is maintained per-bird in the client rendering layer (it's presentation-only; no grammar state in the server DB). On snapshot pull, the server sends `call_timing_offset` (how far into the current call cycle the bird is) so the client can join mid-call rather than starting fresh.

Bird-to-bird interaction: if Bird A calls and Bird B's `social_warmth` is high and mood is `content` or `curious`, there's a probability of B responding (a "chorus event"). Chorus events are generated in the simulation tick and included in the snapshot as pending_interactions, which the client realizes as slightly offset calls.

---

## 6. Sync Model

The sync model is deliberately simple because the server is the sole writer of canonical state.

### Single canonical record

- personality_vector and mood live in one DB table each, one row per bird
- Only the simulation service writes to those tables
- Clients write only to interaction_event (append-only)

### Client state consumption

On tab open:
1. Client fetches snapshot (embedded in HTML on cold load, else fetched from API)
2. Client begins rendering immediately from snapshot
3. Client sends a `return` interaction event (triggers greeting logic in next tick or via a fast-path event handler)
4. Client subscribes to a server-sent-events (SSE) stream for incremental snapshot updates

SSE stream sends lightweight diff events:
```
{ type: "mood_change", bird_id, new_mood }
{ type: "perch_change", bird_id, new_zone }
{ type: "weather_start", weather_type }
{ type: "weather_end" }
{ type: "notebook_entry", entry }
```

On visibility change to hidden: client disconnects SSE, stops presence pings
On visibility change to visible: client reconnects SSE, fetches fresh snapshot (handles laptop-lid-close gap)

### Conflict prevention

There are no conflicts to resolve. The server is the only writer of state. Multiple devices:
- Both read the same canonical record
- Both write interaction events to the same append-only log (order by `occurred_at`; tick processes in order)
- If two devices submit presence pings simultaneously, both are recorded; the tick sums them (presence-time is additive)

No last-write-wins, no vector clocks, no merge. The append-only event log is the conflict-free data structure.

---

## 7. Frontend Rendering Pipeline

### Technology choices

- **Canvas/WebGL** for the aviary scene: enables the procedural motion, parallax layers, and per-bird rendering at 60fps
- **CSS** for top bar, overlays, notebook, settings panels
- **WebAudio API** for call synthesis
- Code-split aggressively: aviary core in main bundle; settings, account, visit-invite, notebook in async chunks

### Scene composition

Layers (back to front):
1. Sky + background foliage (CSS gradient + SVG; animated for day/night)
2. Background perch + far birds (lower opacity at distance)
3. Middle plane perches and birds
4. Foreground perch + near birds
5. Ambient foreground leaves/feathers (canvas overlay, simple particle system)
6. Top bar (HTML/CSS, absolutely positioned, fades on idle)

### First-frame strategy

The HTML response includes:
- An inline `<script>` with the first snapshot JSON
- Critical CSS inlined
- The main bundle loaded with `<link rel="preload">`

On parse, an init script places birds at their snapshot positions in their current animation poses before the full bundle executes. The "quiet field" fallback (soft sky, faint motion cues) is CSS-only, no JS needed, so it renders even before the bundle parses.

The first frame target: bird rendered at correct position in a mid-action pose within 500ms. No entry animation. No fade-from-black. The client joins the aviary mid-scene.

### Idle micro-motion

Each bird has a motion controller driven by mood:

```
MotionController(bird, mood):
  select idle_behavior from mood_behavior_map[mood]
  -- content: preen sequences, occasional look-around
  -- wary: frequent scan, minimal preening, back-perch preference
  -- curious: head-tilts toward sounds, approach-lean
  -- drowsy: minimal motion, feathers-fluffed pose, slow blink
  -- alert: upright posture, frequent look-around, frequent short calls

  schedule next_action at now + jitter(base_interval_for_behavior)
```

Motion is purely client-side (no server state). The client interpolates between poses using spring physics for organic feel. Pose transitions are 150–400ms depending on mood (drowsy is slower; alert is snappier).

### Reduced-motion mode

Detect `prefers-reduced-motion` media query and/or explicit user setting. In reduced-motion:
- Replace sprite animation with cross-fade between still poses (CSS `transition: opacity`)
- Remove animated leaf/feather particles (purely decorative; remove entirely)
- Keep day/night color cycle but slow the transition rate further
- Keep call captions, narration, all semantic content

### Perch transitions

When a bird changes perch (server tick assigns a new perch zone based on mood/boldness change):
- Normal mode: short flight arc animation (canvas, 1–2 seconds)
- Reduced-motion: cross-fade between source and target perch poses

The snapshot includes the bird's new perch zone; the client detects the change on snapshot receipt and triggers the transition.

### Listen-in visual

When the user focuses a bird (click, tap, or keyboard Enter):
- Subtle soft-focus vignette on the non-focused birds (opacity 70%, slight blur if GPU budget allows)
- No harsh highlight ring or border on the focused bird; the audio mix change is the primary feedback
- Disengage on second click, Escape, or focus elsewhere

---

## 8. Audio Pipeline

### WebAudio graph

```
[OscillatorNode(s) + BufferSourceNode] → [BiquadFilter(formant)] → [GainNode(bird_gain)]
                                                                              ↓
                                                            [ChannelMerger (chorus mix)]
                                                                              ↓
                                                                   [GainNode(master)]
                                                                              ↓
                                                                   [AudioDestination]
```

Each bird has its own gain node. The listen-in interaction ramps the focused bird's gain to 1.0 and all others to ~0.15 over 1.5 seconds (linear ramp via `AudioParam.linearRampToValueAtTime`). On disengage, all gains return to their baseline (personality.vocal_frequency mapped to 0.4–0.8) over the same ramp.

### Procedural call synthesis

Per-species motif library stored as descriptor objects:
```typescript
interface Motif {
  base_freq_hz: number        // fundamental
  contour: PitchContour[]     // [{time_ms, freq_multiplier}]
  formant_q: number
  duration_ms: number
  variation: {
    pitch_cents: number       // max random ± variation
    onset_ms: number          // jitter in onset timing
    duration_pct: number      // variation in note length
  }
}
```

At call time:
1. Select motif by grammar rules
2. Apply pitch variation via OscillatorNode.frequency with detune offset
3. Apply onset jitter by scheduling start time with small random offset
4. Set duration via stop() scheduling
5. Apply amplitude envelope via GainNode param automation

The call grammar state per bird is a small FSM maintained in client memory. It sequences motifs per the grammar's rules, shaped by personality.vocal_frequency (inter-call gap) and current mood (motif selection weights).

### Chorus mixing

Two birds calling simultaneously: their individual gains stay at their personality-shaped values. The ChannelMerger combines them. The master gain is normalized so the total loudness doesn't spike when two birds call at once (a simple peak-limiter GainNode or a DynamicsCompressorNode at master).

### Listen-in mix decay

When the user engages listen-in, the decay on the ambient birds is gradual (1.5s ramp). This is critical to the "listening, not switching" feel. A hard cut would be jarring and would reveal the underlying gain nodes as UI controls.

### WebAudio fallback

If `AudioContext` is unavailable or the audio context cannot be resumed (permissions, hardware):
- Set captions to ON automatically (if not already)
- Proceed in silence; no recorded audio fallback
- Surface a quiet matter-of-fact note in accessibility settings: "Audio is unavailable. Call captions are on."

### Caption generation

At the moment each call event is synthesized:
1. Inspect the motif and variation applied
2. Map to a prose description template per motif type:
   - rising three-note motif → "a soft three-note rise"
   - low trill with pause → "a low trill, paused, low trill again"
   - single sharp call → "a single sharp call from the back perch"
3. Inject into the caption layer for the calling bird (HTML element near bird position, fades in/out with the call duration)

Caption text is generated at call-synthesis time, not stored per call. Each call produces its own caption; captions match what actually played.

---

## 9. Accessibility Surfaces

### Screen-reader narration

A visually-hidden `aria-live="polite"` region is updated on a slow cadence (30–60s at idle). Content is generated from the current snapshot:

```
NarrationGenerator.generate(snapshot):
  pieces = []
  for each bird in snapshot.birds:
    pieces.push(describe_bird(bird))  -- "a small grey bird is on the front perch, calling softly"
  pieces.push(describe_scene(snapshot))  -- "it is morning; the light is gentle"
  return pieces.join(". ")
```

`describe_bird` uses the bird's name (once the user has seen it), mood, perch zone, and current call state. Output is naturalist prose, lowercase, present-tense.

On user-initiated events (offer accepted, settle triggered, return greeting):
- Update the aria-live region promptly (not waiting for the 30–60s cadence)
- Priority: "polite" is sufficient for idle updates; do not use "assertive" as it interrupts the user's current screen-reader activity

The narration voice is identical to the notebook voice. A screen-reader user moving from the aviary view to the notebook hears the same register.

### Keyboard navigation

Tab order:
1. Skip-to-main link (visually hidden, appears on focus)
2. Top bar icons (left to right): notebook, offer, settle, accessibility-settings, account
3. Aviary scene: first bird in front-to-back, left-to-right order

Within the aviary:
- Tab/Shift-Tab: cycle between birds
- Enter: listen-in on focused bird (or cancel if already focused)
- Escape: exit listen-in; return focus to last focused bird
- Arrow keys: move between birds

Offer panel: standard modal focus trap while open; Escape closes.

Focus indicators: 3px solid outline, color chosen to pass WCAG AA against both bright (midday) and dark (night) aviary backgrounds. Two focus ring colors (light/dark) applied based on current day/night state. Specified in the design system; reference here as a planning commitment.

### Contrast

All user-copy text (top bar icons + labels, captions, narration display if any, system surfaces) targets WCAG AA (4.5:1 for normal text, 3:1 for large text). The aviary scene itself has no user-copy text that requires this treatment.

### Reduced-motion defaults

Detect `prefers-reduced-motion: reduce` at mount and switch to reduced-motion mode immediately. No flash of motion before detection. The detection is synchronous from a CSS media query; the initial render class is set before first paint.

---

## 10. Performance Budgets and Observability

### Initial JS bundle: <2MB gzipped

Enforcement:
- Bundle analysis in CI (fail build if gzip exceeds 2MB)
- Aviary core (rendering + audio) is the only synchronous import
- Settings, account, notebook, visit panels are dynamically imported on demand
- Bird visual assets: SVG-based with CSS parameterization for plumage saturation; no large raster bitmaps in the main bundle
- Audio motif library: small JSON descriptors, not audio files

### Time to first bird visible: <500ms

Strategy:
- First snapshot embedded in the HTML response (generated at SSR or served from edge cache)
- CSS for the aviary frame and quiet-field fallback inlined in `<head>`
- Main bundle preloaded; a lightweight init script (<5KB, inline) places birds at snapshot positions before bundle parses
- Synthetic perf tests in CI: headless Chromium on a throttled 4G connection profile; fail if first-bird-visible > 500ms

### 60fps idle motion on a 5-year-old laptop

Strategy:
- Animation frame budget monitoring: track frame times via `performance.now()` in the rAF loop; if p95 > 16ms over a 10-second window, reduce motion complexity (fewer leaf particles first, then simplify bird motion)
- GPU compositing: bird layers on their own compositing layer (CSS `will-change: transform` or canvas offscreen layer)
- Canvas draw calls minimized: dirty-rect tracking; only repaint changed regions per frame

### No memory growth over 30 minutes

Enforcement:
- CI test: run aviary in headless Chromium for 30 minutes with simulated interactions; assert JS heap < 50MB at end (or < 10% growth from stabilized value)
- Audio buffers: reuse AudioBuffer objects per motif rather than allocating new ones per call
- Notebook scroll: use virtual scrolling (only render entries in viewport); release DOM nodes on scroll-out
- Worker threads and AudioContext: a single shared AudioContext per page; no new contexts created

### Observability

Aggregate metrics collected (none contain per-bird or per-account interaction data):

| Metric | Alarm threshold |
|--------|----------------|
| Simulation tick latency p99 | > 5s |
| Snapshot fetch latency p95 | > 300ms |
| First-bird-visible (RUM, p75) | > 500ms |
| Audio context error rate | > 1% of sessions |
| Client render frame p99 | > 20ms |
| Interaction event ingest error rate | > 0.1% |
| Magic link delivery failure rate | > 2% |

Synthetic monitoring: an automated fleet of headless browsers opens the aviary from 3+ geographies every 5 minutes and reports first-bird-visible, snapshot fetch time, and audio context initialization success.

Real User Monitoring: anonymized, session-level aggregates. No per-bird fields. Emitted from the client via a small batch reporter that flushes on page-hide and periodically.

---

## 11. Rollout

### Launch configuration

- Start every account with 2 birds (species selected by server at signup; user assigns names)
- Bird-cap enforced at 7; aviary age governs when additional bird slots unlock (first unlock ~3 months; subsequent unlocks at longer intervals, calibrated during build)
- Visit invite feature ships enabled but off-by-default per account; no invite prompt during onboarding

### Ramp plan

1. **Pre-launch**: synthetic calibration runs (simulate 7-day and 30-day presence scenarios, validate drift constants match calibration targets)
2. **Closed beta** (invited accounts only): ~100 accounts, monitor simulation tick latency, presence-ping volume, first-bird-visible RUM, notebook entry cadence
3. **Soft open**: remove invite gate, monitor above metrics plus audio-context error rate and memory growth in RUM
4. **v1 GA**: when all perf budgets hold under real traffic

### Day-one instrumentation

From the moment the first account signs up:
- Every simulation tick logged with duration (no per-account detail, just latency)
- First-bird-visible RUM on every client session
- Audio context initialization result per session (success / silent-fallback)
- Presence ping volume aggregate (total pings/hour, not per-account)
- Notebook entry generation rate (entries/aviary-day, anonymized)
- Visit invite issuance count (no visitor detail, just count)

### No instrumentation of per-bird state

The analytics warehouse never receives: personality vector values, mood sequences per account, per-bird offer/listen-in history, individual presence-session durations per account. The line is drawn at the metric definition level, not enforced downstream.

---

## 12. Risks

### Drift calibration wrong at launch

**Risk**: DRIFT_ALPHA or the presence-weight constants are miscalibrated. Too fast: users notice birds changing session-to-session (Tamagotchi feel). Too slow: no perceptible change after a month (screensaver feel).

**Mitigation**: Pre-launch synthetic calibration runs (automated; no human observation needed). Define a testable assertion: "after 7 simulated days of 20-minute-average presence sessions, at least one trait moves by at least 0.05 for all seed personality vectors across all species." Run this in CI. The exact constants are not in this plan (they require calibration); the assertion is the contract.

**Recovery path**: DRIFT_ALPHA is a server-side config value, not deployed in client code. Adjusting it requires no client deploy; takes effect on the next tick cycle for all accounts.

### Sync correctness: stale snapshot on reconnect

**Risk**: A client that reconnects after a long gap (laptop lid closed for 8 hours) renders a stale snapshot for a moment before the fresh snapshot loads, producing a visible jump.

**Mitigation**: The quiet-field fallback is shown until the fresh snapshot arrives on reconnect. Do not render the stale in-memory snapshot after a reconnect; always fetch fresh. The SSE reconnect path must detect long gaps (compare snapshot `tick_id` with server's current tick) and force a full snapshot refresh rather than replaying diffs.

### Audio uncanniness

**Risk**: Procedural calls that fall into the uncanny valley — clearly synthetic but failing to feel organic. A call that sounds "MIDI" or "video-game" breaks the aliveness promise immediately and is unrecoverable in the same session.

**Mitigation**: The call grammar and motif library must be designed and tuned by someone with audio design background (not purely a developer task). Plan for multiple rounds of listening tests during build, not just one sign-off. The pitch contour templates, formant Q values, and variation ranges are the load-bearing knobs; they require human ears, not just spec conformance.

**Also**: The "recognizable across drift" requirement adds another constraint — each species call must stay distinctive as vocal_frequency rises. Test with high-drift synthetic birds during build.

### Accessibility regression: narration voice drift

**Risk**: Screen-reader narration and caption text, generated by different parts of the codebase, drift toward different voices — one starts announcing states, one uses naturalist prose. A screen-reader user hears two products.

**Mitigation**: Both narration and caption text are generated by a single shared `VoiceGenerator` module. That module has unit tests asserting the output prose follows the naturalist style constraints (no announcement verbs, lowercase, present-tense, specific). Automated style lint is a known-hard problem; the mitigation is shared code, not shared tests.

### Presence definition exploited by battery-saver modes

**Risk**: Some browser power-saving modes freeze setTimeout/setInterval but may not freeze the page's `visibilityState` signal. A frozen tab might appear visible while not running the pointer-activity check, silently inflating presence-time.

**Mitigation**: The client sends presence pings only when all three conditions are met AND the ping interval fires. If the interval is frozen, no ping is sent. The server-side tick cannot invent presence pings; it only processes what arrives in the event log. The risk is actually inverted: power-saving modes would under-report presence (fine; monotonic drift is robust to this) rather than over-report.

### Visit revocation race condition

**Risk**: A visitor's client pulls a snapshot between the host revoking the invite and the server propagating the revocation, allowing one extra snapshot view.

**Mitigation**: Acceptable. The risk window is one tick interval (~60s). The visit log records the revocation timestamp; the host can see the sequence. A hard real-time revocation would require the server to actively terminate the visitor's SSE stream, which is a valid implementation: on revocation, the server marks the invite revoked in DB and sends a `{ type: "visit_revoked" }` event on the visitor's SSE stream. Client shows the matter-of-fact surface immediately on receipt. Plan for this as a target behavior; SSE stream termination on revocation is the correct implementation.

### Notebook entry cadence calibration

**Risk**: Too many entries (one per session) dilutes the prose; too few (once a month) makes the feature feel broken.

**Mitigation**: The simulation tick's notebook-entry generator uses a probabilistic gating rule: base rate of ~1 entry per 3 active-aviary-days, with a multiplier for "noteworthy" events (first time a specific bird greeted first, first chorus event, unusual weather coinciding with a settle gesture). Calibrate the base rate and multiplier during the closed beta using the `notebook entry generation rate` metric. The prose templates are not in this plan; they're a content-design artifact that must be reviewed for voice consistency before launch.
