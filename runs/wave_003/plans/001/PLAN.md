# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### What is in v1
- **Accounts & Auth**: Single-user accounts, email magic-link sign-in, multi-device session tokens, account settings with export and deletion
- **Aviary Core**: Two starter birds per new account, cap at seven birds, single horizontal scene, three perch zones
- **Bird Engine**: Personality vectors (5 traits), mood system, procedural calls, idle motion, drift function, bird-to-bird interaction
- **Interactions**: Return-greeting, listen-in, offer (seed/song/still pool), settle gesture, presence accounting
- **Field Notebook**: Auto-generated, read-only, naturalist prose observations, sparse entry cadence
- **Visit Invitations**: Opt-in, email-based, read-only ambient viewing, revocable, 30-day expiration
- **Sync**: Server-side canonical state, multi-device coherence, no client-side state merging
- **Accessibility**: Screen-reader narration, reduced-motion mode, call captioning, keyboard navigation, WCAG AA compliance
- **Performance**: Initial bundle <2MB gzipped, time-to-first-bird <500ms, 60fps idle, no memory growth over 30min

### What is explicitly NOT in v1 (non-goals respected)
- No native mobile apps (web-only)
- No gamification of any kind — no achievements, streaks, levels, scores, badges, XP, counters, calendars, green dots
- No Tamagotchi mechanics — birds do not die, hunger, show distress, or have decaying happiness meters
- No social network surfaces — no profiles, follows, public feeds, discovery, friend-of-friend chains, mutual visits, comments, leaderboards
- No push notifications, pings, or emails about the aviary (except magic-link auth and optional visit notifications)
- No shared or multi-user aviaries
- No payments or paid tiers
- No customizable scenes or multi-aviary accounts
- No recorded audio fallback — procedural synthesis only, with silence+ captions as fallback

### Scope boundaries with teeth
- The seven-bird cap is hard — audio recognizability collapses above this threshold
- Personality vectors are never exposed numerically to users, not even behind a debug toggle
- Notebook entries are never about user behavior ("you visited every day") — only about aviary observations
- The social visit feature defaults OFF for every new account, with no onboarding prompt to share

---

## 2. Architecture

### Service shape
Three services, one database per simulation domain, one CDN for static assets:

1. **Web Client** (browser — React/Vue/Svelte + Canvas/WebGL rendering + WebAudio)
2. **API Service** (stateless request handler — auth, snapshot delivery, event log append, account operations)
3. **Simulation Service** (stateful tick processor — the only writer of personality vectors, mood state, and notebook entries)
4. **Static Asset CDN** (JS bundles, species asset packs, HTML shell — edge-cached)

### Client/server split
- **Client owns**: Rendering, audio synthesis, input handling, presence detection, animation interpolation between snapshots, reduced-motion rendering, caption display, screen-reader narration generation
- **Server owns**: Personality vector storage and mutation, mood transitions, drift computation, notebook entry generation, canonical state snapshots, event log, auth, account lifecycle
- **Boundary rule**: The client renders snapshots; it never computes drift, never writes personality values, never invents mood transitions. The client is a viewer and event emitter, not a simulation participant.

### Render pipeline boundary
- The scene renders at 60fps client-side using interpolated positions between server snapshots
- Server runs tick at ~1 minute cadence; client pulls fresh snapshots on visibility changes, long frame gaps (post-sleep), and low-frequency keepalive
- Visual state (position, pose, motion phase) is derived from snapshots; audio state (call timing, mix levels) is derived from snapshots with client-side procedural synthesis

---

## 3. Data Model

### Account
```
account_id          uuid (synthetic, primary key)
email               encrypted string
email_verified      boolean
created_at          timestamp
soft_deleted_at     timestamp | null
settings            jsonb (reduced_motion, captions, visit_notifications)
```
*Rule: `account_id` is used everywhere internally; `email` lives only on this record.*

### Session
```
session_id          uuid
account_id          uuid -> Account
device_fingerprint  string (opaque)
created_at          timestamp
revoked_at          timestamp | null
```

### Aviary (one per account)
```
aviary_id           uuid
account_id          uuid -> Account
created_at          timestamp
weather_state       enum [clear, light_rain, soft_wind] | null
weather_started_at  timestamp | null
settled             boolean
```

### Bird
```
bird_id             uuid (stable forever)
aviary_id           uuid -> Aviary
species_id          uuid -> Species
name                string (user-assigned, renameable)
birth_date          timestamp (aviary creation for starters; adoption date for additions)
position_zone       enum [front, middle, back]
position_perch      int
```

### Personality Vector (server-authoritative, never client-written)
```
bird_id             uuid -> Bird
boldness            float [0, 1]
social_warmth       float [0, 1]
vocal_frequency     float [0, 1]
plumage_saturation  float [0, 1]
curiosity           float [0, 1]
last_drifted_at     timestamp
```
*Calibration target: measurable change in instruments after ~1 week of regular visits; user-visible change after ~3 weeks.*

### Mood State (per-bird, fast-timescale)
```
bird_id             uuid -> Bird
current_mood        enum [wary, content, curious, drowsy, alert, settled]
mood_entered_at     timestamp
mood_expires_at     timestamp | null (for timed transitions like weather effects)
next_scheduled_tick timestamp
```

### Event Log (append-only, client-written)
```
event_id            uuid
account_id          uuid -> Account
bird_id             uuid | null
session_id          uuid -> Session
event_type          enum [presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song, offer_pool, settle, tab_close]
payload             jsonb
timestamp           timestamp (client-provided, used for ordering within session)
server_received_at  timestamp
```
*Rule: Clients write events; Simulation Service reads and processes them.*

### Field Notebook Entry
```
entry_id            uuid
aviary_id           uuid -> Aviary
written_at          timestamp
body                text (naturalist prose, lowercase, present-tense)
trigger_events      uuid[] -> Event Log (internal reference, not user-visible)
```

### Visit Invitation
```
invite_id           uuid
host_account_id     uuid -> Account
visitor_email       string
status              enum [pending, active, revoked, expired]
created_at          timestamp
expires_at          timestamp
revoked_at          timestamp | null
last_used_at        timestamp | null
```

### Visit Session
```
visit_session_id    uuid
invite_id           uuid -> Visit Invitation
started_at          timestamp
ended_at            timestamp | null
```

### Snapshot (computed on read, not stored as a table)
A snapshot is a real-time assembly of:
- Aviary state (weather, settled, time-of-day)
- Per-bird state (position, mood, current animation seed, call timing)
- Recent events (for client-side interpolation cues)

### Species Pool (reference data, ~6 species)
```
species_id          uuid
name                string
visual_silhouette   string (SVG path or asset reference)
default_palette     jsonb
call_grammar        jsonb (motif library)
nocturnal           boolean
```

---

## 4. API Surface

### Authentication
- `POST /auth/magic-link` — body: `{email}` ; sends magic link, returns 204
- `GET /auth/verify?token={token}` — consumes magic link, sets session cookie, returns redirect to aviary
- `POST /auth/sign-out` — revokes current session
- `GET /auth/sessions` — list active sessions (for revocation UI)
- `DELETE /auth/sessions/{session_id}` — revoke a session

### State Consumption (client poll model)
- `GET /aviary/snapshot` — returns current canonical state snapshot. Called on:
  - Initial page load
  - `visibilitychange` → visible
  - Long render-frame gap detected (>5s while tab visible, indicating laptop sleep)
  - Low-frequency keepalive every ~30s while tab is visible
- Response shape: `{aviary: {...}, birds: [...], weather: {...}, timestamp}`

### Event Submission
- `POST /events` — body: `{events: [...]}` ; client batches presence pings and interaction events, appends to event log. Returns 202 immediately (fire-and-forget into queue).

### Account Operations
- `GET /account` — returns account settings, session list
- `PATCH /account` — update settings (name, preferences)
- `POST /account/export` — triggers JSON export, emailed as download link
- `DELETE /account` — initiates soft deletion
- `POST /account/recover` — reverses soft deletion within 30-day window

### Visit Feature
- `POST /visits/invite` — body: `{visitor_email}` ; creates invitation, emails link
- `GET /visits` — list invitations and visit log
- `DELETE /visits/invites/{invite_id}` — revoke an invitation
- `GET /visit/{invite_token}` — visitor endpoint: returns read-only snapshot of host's aviary. No event submission allowed from this session type.

### Notebook
- `GET /notebook` — paginated list of entries, newest first
- No write endpoint — entries are server-generated only

---

## 5. Simulation Engine Design

### Tick architecture
The Simulation Service runs a tick per aviary at ~1-minute cadence. Each tick:

1. **Read unprocessed events** from the event log for this aviary since last tick
2. **Compute presence-time** from `presence_ping` events using the three-signal rule (visibility + focus + recent pointer/key activity)
3. **Apply drift** to personality vectors based on presence-time and interaction events
4. **Transition moods** based on time-of-day, weather state, recent interactions, bird-to-bird influence
5. **Update positions** (perch zone selection based on boldness + mood)
6. **Generate notebook entries** if noteworthy events warrant it
7. **Advance weather** (rare stochastic transitions)
8. **Write new canonical state**
9. **Mark events as processed**

### Drift function (the load-bearing algorithm)
```
for each bird in aviary:
  presence_minutes = aggregate presence_time since last tick
  
  // Base drift from presence
  delta = low_pass_filter(presence_minutes, time_since_birth)
  
  // Interaction multipliers
  if listen_in_on_this_bird:
    social_warmth += delta * 1.5
    vocal_frequency += delta * 1.2
  if offer_accepted_by_this_bird:
    curiosity += delta * 0.8
  if offer_made_near_this_bird:
    boldness += delta * 0.5
  
  // Monotonic toward expressive: never subtract on neglect
  // "Neglect" simply means delta ≈ 0 for that tick
  // Drift rate naturally slows as traits approach ceiling
  
  for each trait:
    new_value = old_value + delta * (1 - old_value) * trait_specific_rate
    clamp to [0, 1]
```
*Key rule: No negative deltas. A bird ignored for weeks stays at its current trait values; it does not regress.*

### Mood transition logic
```
for each bird:
  // Time-of-day signal
  if local_time is dawn → nudge toward alert
  if local_time is dusk → nudge toward drowsy
  if local_time is night → nudge toward settled
  
  // Weather signal
  if rain_active → dampen vocal frequency intention
  if wind_active → nudge some birds alert, others wary
  
  // Recent interaction signal
  if offer_accepted_recently → nudge toward content
  if startled_recently → nudge toward wary
  
  // Personality gating
  high_boldness_threshold = 0.6
  if bird.boldness > high_boldness_threshold:
    wary_transition_probability *= 0.5
  
  // Bird-to-bird influence
  for each other_bird in aviary:
    if other_bird.mood == wary and distance < threshold:
      nudge this_bird toward wary
    if chorus_event_active and this_bird.vocal_frequency > 0.5:
      nudge toward content/alert (joining in)
  
  // Apply transition with randomized small delay per bird
  if transition_probability > threshold:
    set new mood, set mood_entered_at
```

### Call-grammar runtime
Each species defines a motif library (short melodic fragments). At call-generation time:
```
select_motifs(species_grammar, bird.vocal_frequency, bird.current_mood)
  → motif_sequence

vary_timing(motif_sequence, bird.personality, random_seed)
  → timed_notes

vary_pitch(timed_notes, bird.mood, ambient_temperature)
  → final_call
```
The client synthesizes `final_call` via WebAudio at trigger time. The server includes the next scheduled call time and motif seed in each snapshot.

### Calibration guardrails
- Instrumentation must detect if a typical synthetic account shows >5% trait change in <3 days (drift too fast)
- Instrumentation must detect if a typical synthetic account shows <1% trait change in >14 days (drift too slow)
- Tuning knobs must be exposed as service configuration, not hardcoded constants, to allow post-launch calibration without deploy

---

## 6. Sync Model

### Canonical source of truth
- The Simulation Service's database is the sole canonical store for personality vectors, moods, positions, and notebook entries.
- The API Service is stateless; it reads from the simulation database to serve snapshots.
- Event logs are append-only and processed in strict insertion order.

### Multi-device coherence
- Laptop and phone both pull from the same snapshot endpoint using the same account credentials.
- Both clients submit events to the same event log.
- There is no "merge" because there is no client-side state to merge. Clients are dumb renderers.

### Conflict prevention
- **No last-write-wins on personality**: The simulation tick is the only writer. Clients never send absolute values like "set boldness = 0.7." They send "user listened in to Pip for 3 minutes" and the tick computes the delta.
- **Event log ordering**: Events are timestamped by client for ordering but sequenced by server insertion time. The tick processes in server-insertion order to prevent race conditions between devices.
- **Mood sync**: Mood is snapshot-based. If two clients are open simultaneously, they both render the same mood from the last tick. Neither client "owns" mood.

### Offline behavior
- If the client loses connection, it continues rendering from the last snapshot with client-side ambient motion. Audio continues using cached call schedules until they run out, then falls to ambient silence.
- Reconnection triggers a fresh snapshot pull; the scene updates seamlessly from the new state (no jarring reset).
- Events queued during offline are batched and submitted on reconnect with their original timestamps.

---

## 7. Frontend Rendering Pipeline

### Scene composition
- **Layer order (back to front)**: Sky/background gradient → background foliage → middle plane (perches, birds) → foreground branches/leaves → top bar chrome
- **Bird rendering**: Each bird is a composite of species silhouette + plumage fill (tinted by `plumage_saturation`) + eye/beak detail + feather texture overlay
- **Animation system**: Two tracks per bird — (a) pose/position interpolation between snapshots, (b) idle micro-motion overlay (preen, scan, head-tilt, shuffle) driven by mood and a per-bird random seed

### Idle micro-motion
- Idle motions are categorized by mood: preen sequences for content, scan sequences for wary/drowsy, tilt-and-watch for curious, alert-posture for alert
- Each motion is a small animation clip (8-24 frames) played at low speed with randomized start offsets so birds never sync
- Motion clips are procedurally perturbed (timing jitter, amplitude variation) per bird so two content birds never preen identically

### Transitions
- Perch changes: smooth bezier path over 2-4 seconds, with a small anticipation hop at start
- Mood changes: cross-fade between idle motion sets over 3 seconds
- Weather onset: gradual opacity fade for rain particles, 10-second leaf-ripple wind effect
- Day/night cycle: continuous palette interpolation based on local time, not stepped

### Reduced-motion mode
- All frame-by-frame animations replaced by slow CSS/Canvas cross-fades between key poses (pose A → hold 2s → fade 1.5s → pose B)
- Flight paths become direct cross-fades (no visible motion between perches)
- Ambient leaf drift removed
- Day/night palette shifts remain but slowed to 2x duration
- Call audio unchanged; captions unchanged

### Loading state
- **No spinner, no fade-from-static**. On slow connections, show a "quiet field" — soft sky gradient, perhaps one faint drifting leaf. The first bird appears mid-motion as soon as the snapshot arrives.
- Initial HTML shell is <20KB, delivered from CDN edge, containing inlined critical CSS for the quiet field and async script tags for the bundle.

### Responsive layout
- Viewport < 480px: scene compresses, perch spacing reduces, birds scale to 85%, top bar icons collapse to hamburger
- Viewport 480-1024px: default layout
- Viewport > 1024px: scene widens with increased perch spacing, birds at 100%, subtle extra background detail
- Aspect ratio locked to prevent cropping birds; letterbox with soft extended background if needed

---

## 8. Audio Pipeline

### Procedural call synthesis
- **Engine**: WebAudio API — `AudioContext` with `OscillatorNode` + `GainNode` + `BiquadFilterNode` per call
- **Motif → sound**: Each motif maps to a sequence of oscillator configurations (frequency envelope, amplitude envelope, filter sweep, duration)
- **Personality shaping**: `vocal_frequency` affects call rate and motif complexity; `boldness` affects amplitude and filter brightness
- **Mood shaping**: Content → warmer filter, rounded envelope; Wary → sharper attack, higher frequency, shorter tail; Drowsy → slower envelope, lower amplitude

### Chorus mixing
- Each calling bird gets its own `GainNode` in a master `GainNode` graph
- Default mix: all birds at ~0.6 relative gain (sum compressed to prevent clipping)
- Listen-in mix: focused bird ramps to 1.0 over 2s; others drop to ~0.2 over 2s. Never to 0.
- Mix transitions use exponential ramps (`exponentialRampToValueAtTime`) for natural feel

### Listen-in decay
- On disengage, focused bird fades back to 0.6 and others rise back to 0.6 over 2s
- If user focuses a different bird during listen-in, current focus fades down while new focus fades up (cross-fade, not hard cut)

### WebAudio fallback
- If `AudioContext` creation fails or is denied: audio system initializes in "silent" mode
- Captions are enabled by default in silent mode
- No recorded audio fallback path is built or shipped
- Silent mode is communicated quietly in accessibility settings ("audio unavailable — captions on")

### Memory management
- Audio buffers for motif waveforms are pre-generated at species-load time and reused
- Per-call nodes are created and destroyed per call; no persistent node per bird except base gain nodes
- `AudioContext` is suspended when tab is hidden (`visibilitychange`), resumed on visible

---

## 9. Accessibility Surfaces

### Screen-reader narration
- **Generator**: Client-side module that consumes the current snapshot and produces naturalist prose strings
- **Cadence**: One prose update every 30–60 seconds at idle. Faster on user-initiated events.
- **Queue behavior**: New idle narration is suppressed if the previous utterance is still speaking. User-initiated events interrupt and replace the current utterance.
- **Voice**: Lowercase, present-tense, specific. Same voice as field notebook.
- **Example outputs**:
  - Idle: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
  - Event (offer accepted): "pip tilts her head toward the seed, then steps closer."
  - Event (return greeting): "wren looks up from preening and calls once."
- **ARIA**: The aviary container is a `role="region"` with `aria-live="polite"` and `aria-label="aviary"`. Navigating into it shifts focus to the first bird.

### Reduced-motion mode
- Triggered by `prefers-reduced-motion: reduce` OR explicit toggle in accessibility settings
- Rendering changes detailed in Section 7 (cross-fades instead of frame animation)
- Toggle is persistent per-account (stored server-side in `account.settings`)

### Call captioning
- Opt-in via accessibility settings
- Generated from the same procedural call grammar that generates audio — each call produces both an audio buffer and a caption string
- Caption text examples: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"
- Display: small text label near the calling bird, fades in over 0.3s, holds for call duration + 1s, fades out over 0.5s
- Font: system sans-serif, 14px, high contrast against scene (text-shadow or subtle backing box)

### Keyboard navigation
- `Tab` cycles through top bar items
- `Tab` into aviary scene focuses first bird (front-left to back-right tab order)
- `Arrow Left/Right` moves focus between birds
- `Enter` on focused bird → listen-in
- `Escape` → exit listen-in, or exit any open panel (offer, notebook, settings)
- `Space` or `Enter` on offer icon → open offer panel
- Offer panel: `Tab` through options, `Enter` to select, `Escape` to close
- Settle: reachable via top bar shortcut (`S` when focus is on top bar, or explicit button)
- Focus indicator: 2px soft outline, `#FFFFFF` with 0.8 opacity, 2px offset, visible against all aviary states

### Contrast
- All user-copy text (top bar, settings, errors, captions) meets WCAG AA (4.5:1 for normal text, 3:1 for large text)
- Aviary scene itself has no user copy, so no contrast requirement on scene elements
- Error surfaces and system surfaces use matter-of-fact voice with standard capitalization

---

## 10. Performance Budgets and Observability

### Bundle budgets
- Initial JS bundle: **< 2MB gzipped**
- Strategy: code-split by route (aviary main, settings, notebook, visit flow, auth); preload critical chunks; lazy-load non-critical
- Asset budget: species visuals as procedural SVG or compact bitmaps (<50KB each, 6 species = ~300KB); no video, no large image textures

### Time-to-first-bird
- Target: **< 500ms** from navigation to first bird visible on mid-tier mobile over 4G
- Implementation:
  - HTML shell delivered from CDN edge with inlined critical CSS (~8KB)
  - Async JS bundle load
  - Initial snapshot embedded in HTML shell (or fetched via edge function with <50ms latency)
  - First bird renders as soon as bundle parses + snapshot is available; no waiting for all assets

### Runtime budgets
- Idle motion: **60fps** on 5-year-old mid-range laptop (measured via `requestAnimationFrame` timing)
- Memory: **zero growth** over 30-minute session (CI test with Chrome DevTools heap profiler)
- Audio: no per-call allocations that aren't freed; buffer pools for synthesis nodes

### Observability
**Synthetic monitoring** (fleet of automated browsers):
- Time-to-first-bird from 5 geographies, every 5 minutes
- Snapshot delivery latency
- Audio context initialization success rate
- 60fps frame-rate check during 5-minute synthetic session

**Real User Monitoring (aggregate only, no per-account dimensions)**:
- Page load timing (Navigation Timing API)
- First bird render timestamp
- `requestAnimationFrame` frame timing (p50, p95, p99)
- Audio context error counts
- JS error counts (anonymized stack traces)

**Server-side metrics**:
- Simulation tick latency (p50, p95, p99) — **alarm if p99 > 5s**
- Event log ingestion rate and lag
- Snapshot generation latency
- Auth operation latencies

**Privacy boundary enforcement**:
- Telemetry pipelines are architecturally separated from simulation databases
- No per-bird state, per-account interaction history, or personality vectors in any telemetry
- Regular audit query: assert no `SELECT` from simulation DB in analytics warehouse

### What we deliberately do NOT measure
- Per-bird engagement funnels
- Individual user session duration (aggregate histograms only)
- "Which bird is most popular" across accounts
- Any metric that could be used to build gamification features later

---

## 11. Rollout

### Phase 0: Infrastructure (Weeks 1–3)
- Set up CDN, API Service, Simulation Service, database clusters
- Implement auth (magic-link) and account CRUD
- Build event log ingestion pipeline
- Set up synthetic monitoring and RUM

### Phase 1: Bird Engine Core (Weeks 4–7)
- Implement personality vector model and storage
- Implement drift function with calibration harness
- Implement mood system and transition logic
- Implement tick scheduler and processor
- Build instrumented test accounts for drift calibration

### Phase 2: Client Rendering & Audio (Weeks 6–9)
- Build scene renderer (Canvas or WebGL — decision point; Canvas is likely sufficient and lighter)
- Implement idle motion system
- Implement procedural call synthesis (WebAudio)
- Implement chorus mixing and listen-in
- Integrate snapshot consumption and interpolation
- Achieve 500ms time-to-first-bird target

### Phase 3: Interactions (Weeks 8–11)
- Implement presence detection (three-signal rule)
- Implement return-greeting logic (absence-length aware, boldness-weighted, staggered)
- Implement listen-in, offer (with cooldown), settle
- Implement field notebook generation logic

### Phase 4: Sync & Multi-Device (Weeks 10–12)
- End-to-end multi-device testing
- Validate no last-write-wins scenarios
- Stress-test tick latency under load

### Phase 5: Accessibility & Polish (Weeks 11–13)
- Screen-reader narration system
- Reduced-motion mode
- Call captioning
- Keyboard navigation
- WCAG audit
- Performance audit (bundle size, memory, frame rate)

### Phase 6: Social & Final QA (Weeks 12–14)
- Visit invitation flow
- Revocation and expiration
- Soft launch to internal team
- Drift calibration tuning based on real usage patterns
- Bug fix sprint

### Launch cadence for birds-per-aviary
- All new accounts start with **2 birds**
- 3rd bird unlocks at **60 days** of aviary age
- 4th bird at **120 days**
- 5th bird at **180 days**
- 6th bird at **300 days**
- 7th bird at **450 days**
*These intervals are server-configurable; initial values are starting assumptions to be tuned.*

### Day-one instrumentation
- Synthetic monitoring (time-to-first-bird, frame rate)
- RUM (aggregate load timings)
- Server tick latency
- Auth success/failure rates
- Audio context initialization rate
- Drift rate instrumentation (aggregate, no per-bird data)

---

## 12. Risks

### Drift calibration risk — HIGH
**What could go wrong**: The drift function is the product's central promise. If it's too fast, users feel they're playing a stat-management game; if too slow, the product feels like a screensaver. There is no objective unit test for "feels right."
**Mitigation**:
- Build a drift-calibration harness with synthetic accounts that simulate realistic usage patterns
- Instrument trait changes daily; alarm on out-of-bounds drift rates
- Plan a 2-week soft-launch calibration period where we tune constants based on real (anonymized aggregate) data
- Keep all drift constants in service configuration, not code, for hot-tuning

### Sync correctness risk — HIGH
**What could go wrong**: A last-write-wins bug or event ordering bug could silently corrupt personality vectors. The user would feel something is wrong but couldn't name it.
**Mitigation**:
- Strict architecture review: no client writes to personality state, ever
- Event log is append-only with server-assigned monotonic IDs
- Simulation tick processes events in strict insertion order
- Build drift-integrity invariant checks in CI (e.g., "trait values never decrease")
- Multi-device chaos testing (two clients rapid-firing events, verifying no state divergence)

### Audio uncanniness risk — MEDIUM
**What could go wrong**: Procedural calls that sound synthetic or "computery" break the aliveness spell. A single user who hears a robotic chirp may never trust the audio again.
**Mitigation**:
- Hire/contract a sound designer with procedural audio experience for motif library creation
- User testing specifically for audio believability ("does this sound like a real bird?")
- Extensive variation testing: listen to 50 consecutive calls from the same bird, assert no perceptible repetition
- Chorus mixing tested with 3, 5, and 7 birds to ensure no phase-cancellation artifacts

### Accessibility regression risk — MEDIUM
**What could go wrong**: Screen-reader narration treated as an afterthought becomes a state-list readout, violating the product's voice. Reduced-motion mode shipped as "animations off" breaks the feeling of aliveness.
**Mitigation**:
- Accessibility features are in the critical path, not a Phase-2 add-on (see rollout plan)
- Hire accessibility consultant for screen-reader UX review
- Dogfood with reduced-motion and screen-reader modes internally
- Define acceptance criteria: "A screen-reader user can describe the aviary's mood after 5 minutes" — not "ARIA labels are present"

### Performance regression risk — MEDIUM
**What could go wrong**: Bundle grows over time as features accrete; time-to-first-bird slips above 500ms; frame rate drops on target hardware.
**Mitigation**:
- CI gates: bundle size check on every PR; memory-leak test on every PR; frame-rate test on every release
- Synthetic monitoring alarms on regression
- Aggressive code-splitting policy: any PR that grows the initial bundle must justify it

### Privacy boundary erosion risk — MEDIUM
**What could go wrong**: An engineer uses per-bird interaction data for a " harmless" aggregate dashboard; an analytics pipeline accidentally reads the simulation DB.
**Mitigation**:
- Physical separation: simulation database credentials not available to analytics services
- Query auditing: any SELECT from simulation DB is logged and reviewed
- Code review policy: any telemetry addition requires privacy review
- Automated test: assert that telemetry events do not contain per-bird fields

### Team cultural risk — LOW but insidious
**What could go wrong**: A well-meaning contributor adds a "harmless" welcome toast, a streak counter, or a notification. The PR looks reasonable and passes code review.
**Mitigation**:
- Document the non-goals prominently in onboarding and CONTRIBUTING.md
- Code review checklist includes: "Does this add any announcement UI? Does it surface visit frequency? Does it gamify?"
- Maintain an internal "temptation log" — when someone proposes a gamification feature, document the proposal and the refusal so the pattern is visible
- Leadership must model the discipline: if leadership asks for an engagement metric dashboard, the team knows the boundary is negotiable

### Visit feature abuse risk — LOW
**What could go wrong**: Visit invitations used for spam; revocation UX is confusing.
**Mitigation**:
- Rate-limit invites per account
- Clear revocation UI in account settings
- 30-day expiration limits accumulation
- No public discovery means no amplification vector

---

## Appendix: Ambiguity Resolutions

1. **Render technology**: Canvas 2D is preferred over WebGL for v1. The scene is not 3D, and WebGL adds bundle size and compatibility risk. Canvas 2D can achieve the required visual quality within budget.

2. **Server framework**: Use whatever the team knows well (Node/Express, Go, Python/FastAPI). The simulation tick is the critical code, not the HTTP framework. The API Service should be horizontally scalable and stateless.

3. **Database**: PostgreSQL for relational data (accounts, birds, sessions, notebook). The event log may benefit from a time-series or append-optimized store (e.g., TimescaleDB, Kafka), but PostgreSQL with proper indexing is sufficient for v1 scale.

4. **Tick scheduling**: Use a job queue (e.g., Redis + Bull, or a cron-like scheduler inside the Simulation Service) to trigger ticks. Each aviary gets its own job; failed ticks retry with exponential backoff and alert.

5. **Client framework**: React or vanilla JS with a lightweight state manager. The render loop is Canvas-based and mostly outside the framework's reconciliation cycle. Choose based on team's existing expertise.

6. **Call caption generation**: Generate caption strings on the client from the same motif-variation function that produces audio parameters. This ensures captions always match what was played.

7. **Weather randomization**: Weather is seeded by aviary creation date + day-of-year so that weather is deterministic per-aviary on a given day (simplifies testing and snapshot caching) but varies across accounts.
