# Pocket Aviary — V1 Implementation Plan

This plan translates the Pocket Aviary PRD into an executable engineering specification. It does not implement the product; it provides the architecture, data model, API contracts, pipeline designs, and operational boundaries a separate engineering team needs to build v1 without further clarification.

Where the PRD leaves an ambiguity, this plan makes a defensible call and notes it.

---

## 1. Scope

### What is in v1

- **Aviary runtime:** 2 starter birds per new account, cap at 7 birds.
- **Account system:** Single-user accounts; email magic-link sign-in; per-device revocable sessions.
- **Bird engine:** Personality vectors (5 traits), mood system, drift function, procedural call grammar, idle motion system, bird-to-bird interaction.
- **Interactions:** Presence accounting (strict three-factor detection), listen-in, offer (seed / song fragment / still pool), settle gesture, return-greeting.
- **Field notebook:** Auto-generated, read-only, sparse naturalist prose entries.
- **Scene rendering:** Single horizontal scene, 3 perch zones, day/night cycle (local time), ambient weather, ambient micro-motion, top-bar chrome with auto-fade.
- **Audio:** Procedural call synthesis via WebAudio, chorus mixing, listen-in mix decay.
- **Accessibility:** Screen-reader narration (naturalist prose, not state lists), reduced-motion mode (cross-fade rendering, not stripped fallback), call captioning, full keyboard navigation, WCAG AA contrast on all chrome text.
- **Sync & simulation:** Server-side canonical simulation tick (~60s cadence), multi-device sync by shared snapshot reads, additive personality deltas, no last-write-wins.
- **Social:** Opt-in visit invitations (email, one-time link), read-only ambient visitor view, revocable, no notifications by default.
- **Privacy & compliance:** Synthetic UUIDs for all internal references, encrypted email storage only, soft-then-hard deletion (30 days), per-bird interaction data isolated from aggregate telemetry.
- **Performance:** 2MB initial JS bundle (gzipped), <500ms time-to-first-bird on mid-tier mobile / 4G, 60fps idle motion on 5-year-old laptop, zero memory growth over 30 minutes.

### What is out of v1 (non-goals respected)

- Native mobile apps (iOS, Android).
- Gamification of any kind: achievements, streaks, badges, levels, scores, XP, calendars, leaderboards, visit counters.
- Tamagotchi mechanics: hunger, distress, decaying happiness meters, death, visible neglect punishment.
- Social network surfaces: profiles, follows, public discovery feeds, friend-of-friend chains, comments, mutual visits, "explore" surfaces.
- Push notifications, email digests, or any outbound messaging about aviary state.
- Shared or multi-user aviaries; household profiles.
- Payments, subscriptions, tiered features.
- Customizable scenes, multi-aviary accounts, panning/zooming/scrolling.
- Recorded-audio fallback; any form of looping audio assets.
- Per-bird interaction data used for model training, recommendation features, or population-level analysis.

---

## 2. Architecture

### Service shape

Three backend services and one client application:

1. **API Gateway** — HTTPS ingress, auth/session validation, rate limiting, routing to downstream services. Stateless. Auto-scaled.
2. **Simulation Service** — The canonical state owner. Runs the tick scheduler, maintains per-account aviary state (bird records, personality vectors, moods, event logs), owns the notebook generation logic, and serves state snapshots. Single-writer for personality state.
3. **Auth Service** — Magic-link generation and verification, session token minting/revocation, account lifecycle (creation, email change, export, soft deletion, hard deletion). Holds the encrypted email↔UUID mapping.
4. **Web Client** — Single-page application (SPA). Renders the aviary scene, synthesizes audio via WebAudio, captures presence and interaction events, consumes state snapshots.

### Client / server split

- **Server owns:** Personality vectors, moods (canonical), notebook entries, account metadata, session tokens, event logs, tick scheduling, visit-invitation state.
- **Client owns:** Rendering frame loop, audio synthesis and mixing, input event capture, local interpolation between snapshots, ambient leaf/feather drift (pure rendering ornaments), accessibility narration generation (from snapshot data).
- **Boundaries:**
  - Client never computes personality drift.
  - Client never writes mood or personality directly.
  - Client writes interaction events to an append-only log; the simulation tick consumes it.
  - Client pulls snapshots; it does not push state.

### Render pipeline boundary

The client rendering pipeline is divided into three layers:

1. **Scene composition layer** — Computes bird positions, perch assignments, lighting, weather overlay from the latest server snapshot + local time. Runs at the simulation snapshot cadence (~30s keepalive) and on every visibility change.
2. **Animation layer** — Interpolates between snapshot states for smooth motion, drives idle micro-motion (preen, scan, head-tilt), manages cross-fade poses in reduced-motion mode, handles day/night palette transitions. Runs at 60fps.
3. **Audio layer** — WebAudio context, procedural call synthesis per bird, chorus mixing, listen-in gain ramps, reverb/ambient bed. Runs on its own audio-thread clock.

---

## 3. Data Model

### Account

```
account
  account_uuid          UUID (PK, synthetic, all internal refs)
  email                 string (encrypted at rest)
  created_at            timestamp
  deleted_at            timestamp (nullable, soft-delete marker)
  email_verified        boolean
  settings              JSONB
    reduced_motion      boolean (default: follows OS preference)
    call_captions       boolean (default: false)
    visit_notifications boolean (default: false)
    audio_enabled       boolean (default: true)
  aviary_age_days       integer (derived, reset-on-recovery)
```

### Bird

```
bird
  bird_uuid             UUID (PK, stable forever)
  account_uuid          UUID (FK)
  species_id            string (FK to species pool)
  name                  string (user-assigned, renameable)
  created_at            timestamp
  adopted_at            timestamp
```

### Personality Vector (canonical, server-only)

Stored as a single JSONB column on `bird`, updated exclusively by the simulation tick:

```
personality_vector
  boldness              float [0.0, 1.0]
  social_warmth         float [0.0, 1.0]
  vocal_frequency       float [0.0, 1.0]
  plumage_saturation    float [0.0, 1.0]
  curiosity             float [0.0, 1.0]
```

- Seed values for starter birds: drawn from species-default ranges with small per-bird random variance.
- Never exposed to client; never serialized into API responses.
- The simulation tick writes additive deltas only.

### Mood (canonical, server-only)

```
mood
  bird_uuid             UUID (FK)
  current_mood          enum: wary | content | curious | drowsy | alert | settled
  mood_entered_at       timestamp
  mood_expires_at       timestamp (nullable)
```

- Mood transitions are computed server-side on tick.
- Snapshot API includes current mood per bird (client needs it for rendering).

### Presence Event Log (append-only)

```
presence_event
  event_uuid            UUID (PK)
  account_uuid          UUID (FK)
  device_session_id     string
  event_type            enum: ping | pause | resume | end
  duration_seconds      integer (for "end" events, the continuous presence block length)
  recorded_at           timestamp
```

- The client emits `ping` events at a cadence (every 30s while presence conditions hold).
- A terminal event (`end` or `pause`) closes the presence block. The tick aggregates total presence-time per block.

### Interaction Event Log (append-only)

```
interaction_event
  event_uuid            UUID (PK)
  account_uuid          UUID (FK)
  bird_uuid             UUID (FK, nullable for aviary-level events)
  event_type            enum: listen_in_start | listen_in_end | offer | settle
  metadata              JSONB (e.g., offer_type: seed|song|pool)
  recorded_at           timestamp
```

### State Snapshot (derived, served to client)

```
snapshot (ephemeral, computed on read)
  account_uuid
  generated_at
  local_time_anchor     timestamp (client's timezone, provided by client in request)
  birds[]
    bird_uuid
    name
    species_id
    current_mood
    current_perch_zone    front | middle | back
    current_pose          enum: preen | scan | settle | call | idle
    call_timings          { next_call_at, motif_id }
    idle_motion_seed      int (deterministic seed for procedural micro-motion)
  aviary_state
    day_phase             dawn | morning | midday | evening | night
    weather_event         null | light_rain | soft_wind
    settle_active         boolean
  visit_token             string (nullable, for visitor sessions)
```

### Notebook Entry

```
notebook_entry
  entry_uuid            UUID (PK)
  account_uuid          UUID (FK)
  written_at            timestamp
  body                  text (naturalist prose, lowercase, present-tense)
  tags                  string[] (internal, for generation logic: e.g., "greeting_order", "weather")
```

- Read-only to users. Immutable after creation.
- Sparsity target: ~1 entry every 2–3 days for active users, tunable via generation heuristics.

### Visit Invitation

```
visit_invitation
  invite_uuid           UUID (PK)
  account_uuid          UUID (FK, host)
  visitor_email_hash    string (SHA-256, for lookup without storing raw email)
  token                 string (one-time URL token)
  status                pending | active | revoked | expired
  created_at            timestamp
  expires_at            timestamp (+30 days)
  revoked_at            timestamp (nullable)
```

- Visitor sessions authenticate via token in URL query param; no account required.

---

## 4. API Surface

### Authentication

All authenticated endpoints require a `Session-Token` header (per-device JWT issued by Auth Service).

| Endpoint | Auth | Description |
|----------|------|-------------|
| `POST /v1/auth/magic-link` | None | Request sign-in link. Body: `{ email }`. Rate-limited per email. |
| `POST /v1/auth/magic-link/verify` | None | Consume token from magic link. Body: `{ token }`. Returns `{ session_token, account_uuid }`. |
| `DELETE /v1/auth/sessions/:id` | Session | Revoke a device session. |
| `GET /v1/auth/sessions` | Session | List active sessions. |

### Aviary State

| Endpoint | Auth | Description |
|----------|------|-------------|
| `GET /v1/aviary/snapshot` | Session | Returns current state snapshot. Query: `timezone_offset`. |
| `GET /v1/aviary/snapshot?visit_token=...` | None (token auth) | Visitor read-only snapshot. Same shape, but `visit_token` scoped. |
| `POST /v1/aviary/events` | Session | Append interaction/presence event. Body: `{ events[] }`. Batchable. |
| `GET /v1/aviary/notebook` | Session | Paginated notebook entries. Query: `before`, `limit` (max 50). |

### Account & Settings

| Endpoint | Auth | Description |
|----------|------|-------------|
| `GET /v1/account` | Session | Returns account settings, email (masked), deletion status. |
| `PATCH /v1/account/settings` | Session | Update settings (reduced_motion, call_captions, etc.). |
| `POST /v1/account/export` | Session | Trigger JSON export. Async; emailed as secure download link. |
| `POST /v1/account/delete` | Session | Initiate soft deletion. Returns recovery deadline. |
| `POST /v1/account/recover` | None | Recover soft-deleted account within 30 days (requires fresh magic link). |

### Visit Invitations

| Endpoint | Auth | Description |
|----------|------|-------------|
| `POST /v1/visits/invitations` | Session | Create invitation. Body: `{ visitor_email }`. Returns `{ invite_url }`. |
| `GET /v1/visits/invitations` | Session | List host's invitations and visit log. |
| `DELETE /v1/visits/invitations/:id` | Session | Revoke invitation. Immediate effect. |

### Key API behaviors

- **Snapshot polling:** The client polls `GET /v1/aviary/snapshot` on tab visibility change, on a 30s keepalive while visible, and after any interaction event submission. It also polls with backoff after long render-frame gaps (e.g., laptop wake from sleep).
- **Event batching:** The client batches interaction and presence events locally and flushes every 10s or on pagehide, whichever comes first. This reduces request count without losing critical events.
- **Idempotency:** Event submission includes a client-generated `batch_uuid`; the server deduplicates on `(account_uuid, batch_uuid)` to safely handle retries.
- **Visitor sessions:** The visitor client receives the same snapshot shape but all mutation endpoints return `403`. The visit token is passed as a query parameter, not a header, so it works without an account session.

---

## 5. Simulation Engine Design

### Tick cadence and scheduling

- The tick runs on a per-account basis, scheduled via a lightweight job queue (e.g., a time-series ordered set in Redis or a scheduled task in the application runtime).
- Target cadence: once per minute ± jitter (±5s random) to prevent thundering herd across accounts.
- Ticks are idempotent and retrievable: if a tick fails, it retries with the same `tick_id` and timestamp, and the drift calculation uses the actual wall-clock delta since the last successful tick, not the nominal cadence.

### Tick execution steps

For each account due for a tick:

1. **Load state:** Pull current personality vectors, moods, aviary age, and unprocessed interaction events for the account.
2. **Process events:** Mark events as processed. Compute:
   - Total presence-time since last tick.
   - Listen-in durations per bird.
   - Offer counts and types per bird.
   - Settle event (boolean, used to cleanly close presence windows).
3. **Compute drift deltas:** For each bird, apply the low-pass filter:
   - `delta = f(presence_time, listen_in_time, offers, settle_flag, personality_vector)`.
   - The function is monotonic: all deltas are ≥ 0. Neglect produces zero delta, never negative.
   - Trait-specific weighting: presence_time feeds all traits; listen-in feeds social_warmth and vocal_frequency; offers feed curiosity (acceptance) and boldness (proximity); settle feeds nothing but ends presence cleanly.
   - Calibration constants (tunable via configuration, testable):
     - After ~1 week of regular visits (defined as 30 min/day presence), instruments should measure a total vector change ≥ 0.05 across at least one trait.
     - After ~3 weeks, visible drift should be perceptible to a human observer (e.g., bird greets more readily, perches closer, calls more often).
4. **Apply deltas:** Add deltas to personality vector, clamp to [0.0, 1.0]. Write updated vectors.
5. **Transition moods:**
   - Compute time-of-day signal from user's timezone (stored on account, updated by client in snapshot requests).
   - Apply ambient weather effect if active.
   - Apply bird-to-bird influence (a wary bird nudges neighbors toward wary; high vocal-frequency birds trigger chorus alignment).
   - Apply personality gating (high boldness resists wary; high curiosity resists drowsy).
   - Set new mood and `mood_entered_at`.
6. **Advance call schedule:** Compute next call times per bird based on `vocal_frequency`, current mood, and chorus probability.
7. **Generate notebook entry (conditional):** Evaluate heuristics:
   - Minimum time since last entry (72h base).
   - Noteworthiness score: greeting-order reversals, first offer acceptance after a drought, weather events, drift milestones.
   - If score exceeds threshold, generate prose entry and write to notebook.
8. **Persist canonical state:** Commit personality vectors, moods, call schedules, processed event markers, and any notebook entry in a single transactional write.

### Drift function — implementation notes

- The drift function is a configured set of constants, not hardcoded magic numbers. Store constants in a configuration table or feature-flag system so calibration can be tuned in production without a deploy.
- Provide a deterministic test harness: given a fixed event log and initial vector, the function produces an expected output vector. CI must enforcedrift calibration tests.
- The "no negative drift" rule is structural: the delta computation discards any negative component before application. This is a hard invariant tested in unit tests.

### Call-grammar runtime

- The simulation tick does not synthesize audio. It computes *when* each bird should call next and *which motif family* to use.
- The client receives `call_timings` in the snapshot and synthesizes the actual audio waveform.
- The tick computes chorus windows: if two or more birds have `next_call_at` within a 2s window, mark them as a chorus group so the client can apply chorus mixing (slight timing variance, shared harmonic bed).

---

## 6. Sync Model

### Canonical state propagation

- The server maintains one canonical aviary state per account in the primary database.
- All clients (laptop, phone, tablet) read from the same record via snapshot API.
- There is no client-side state that needs reconciliation. The client is a renderer, not a simulation participant.

### Conflict prevention

- **Personality state:** Only the simulation tick writes. Clients append events; the tick consumes them. Because deltas are additive and commutative (within the monotonic constraint), event-log order is the only ordering needed. There are no conflicts to resolve.
- **Mood state:** Only the tick writes. Clients read snapshots.
- **Notebook:** Append-only, generated by the tick. No client writes.
- **Account settings:** The user may change settings from multiple devices. These are small, infrequent, and last-write-wins is acceptable for preferences (the surface area is tiny and user-intent is clear). Use optimistic concurrency with an `etag`/`version` field on the account record.
- **Bird names:** Rename is a client-submitted event, but it is a single-field overwrite, idempotent, and user-intent is unambiguous. Store `name_updated_at` to allow simple last-write-wins with timestamp check.

### Offline / reconnect handling

- If the client loses network, it continues rendering from the last snapshot. Idle motion and ambient drift continue locally; audio continues based on cached call schedules.
- On reconnect, the client requests a fresh snapshot. The renderer interpolates from its extrapolated local state to the new canonical state over ~1s to avoid jumps.
- Interaction events queued during offline are batched and sent on reconnect. The tick processes them on its next pass; drift is applied retroactively based on event timestamps, not arrival time.

### Presence sync across devices

- Presence is per-device. If a user has the aviary open on both laptop and phone, each device records its own presence events.
- The tick sums presence-time across all devices for a given account. This prevents double-counting only insofar as the user is unlikely to be actively present on two devices simultaneously; if they are, the presence-time is additive, which is acceptable (the user is, in fact, present).

---

## 7. Frontend Rendering Pipeline

### Scene composition

- The scene is a single full-viewport `<canvas>` element (preferred for performance and pixel-level control over ambient motion) or a WebGL context if the team is comfortable with the complexity. A high-performance 2D Canvas API is sufficient for the visual density described.
- **Layers (back to front):**
  1. Sky / background gradient (computed from local time + day-phase).
  2. Background foliage (subtle parallax, slow ambient sway).
  3. Perch structures and back perch zone.
  4. Middle perch zone + birds.
  5. Front perch zone + birds.
  6. Weather overlay (rain, wind-ripple) if active.
  7. Foreground branch / leaf drift (pure rendering ornaments, no simulation state).
  8. Top bar (HTML overlay, not canvas, for accessibility and input handling).

### Idle micro-motion

- Each bird's idle motion is driven by a deterministic seed (`idle_motion_seed` in snapshot) combined with real time. This ensures that two clients viewing the same aviary at the same moment see the same micro-motion without requiring high-frequency sync.
- The animation layer samples motion curves (sine-composite for breathing, Perlin-like noise for head scan) at 60fps.
- Mood shapes the motion set: wary → more scanning, less preening, head snaps; content → preening, slow blinks; curious → head-tilts toward sounds; drowsy → minimal motion, eyes closed; alert → upright, quick scans.

### Transitions

- **Perch changes:** When a snapshot places a bird at a new perch zone, the animation layer animates a smooth path over 2–4s with ease-in-out easing. The path is a gentle arc, not a straight line, to feel like flight.
- **Mood changes:** The animation layer cross-fades motion sets over 3–5s. No sudden snap.
- **Day/night:** Palette interpolates continuously based on local time; no discrete jumps.
- **Settle trigger:** Lighting shift to evening over 4s; calls quiet by ramping global mix gain down (birds don't stop instantly).
- **Undo settle (within 5s):** Reverse the lighting ramp over 1s; resume normal day-phase palette and audio mix.

### Reduced-motion mode

- Detected via `matchMedia('(prefers-reduced-motion: reduce)')` or user settings override.
- **Rendering changes:**
  - Idle motion: instead of frame-by-frame animation, render a sequence of 3–5 still poses per mood, cross-fading between them every 4–8s.
  - Perch transitions: cross-fade between start and end poses over 3s, no animated path.
  - Ambient leaf drift: removed entirely.
  - Day/night palette shifts: remain but slowed to 2× normal duration.
- **Preserved:** Call audio (unless user also disables audio), mood changes, drift, notebook, all interaction affordances.

### Loading state

- The first paint is a "quiet field": a soft sky gradient matching the current day-phase, plus a single faint ambient motion cue (e.g., one slow drifting cloud shape).
- No spinner. No progress bar. No "Loading..." text.
- Once the snapshot arrives, birds are placed mid-motion as if they have always been there. The transition from quiet field to full scene is an opacity fade over 300ms.
- Empty-aviary state (post-adoption, pre-first-bird): same quiet field, then a single bird flies in to its starting perch over 2s. This is the only "entrance" animation in the product.

---

## 8. Audio Pipeline

### Procedural call synthesis

- Each species has a **motif library**: a set of atomic sound fragments (e.g., a rising chirp, a low trill, a sharp note) parameterized by pitch base, duration variance, and harmonic envelope.
- Each bird's call is assembled at runtime by:
  1. Selecting a motif from the species library (weighted by mood and personality).
  2. Applying personality transforms: `vocal_frequency` shifts timing (higher → shorter gaps between calls); `boldness` slightly raises amplitude and brightness.
  3. Applying mood transforms: wary calls are shorter and more staccato; content calls are longer and more melodic; drowsy calls are quieter and slower.
  4. Adding micro-variance: ±5% pitch jitter, ±10% timing jitter, per-call random seed. This ensures no two calls are identical.
- Synthesis runs in a dedicated WebAudio `AudioWorklet` (or `ScriptProcessorNode` fallback for older browsers) to keep audio off the main thread.

### Chorus mixing

- When the snapshot marks a chorus window, the audio layer:
  - Slightly detunes each bird's pitch (±3 cents) to prevent phase cancellation.
  - Applies a shared "chorus reverb" send to a common convolution reverb node.
  - Reduces individual bird dry-gain by 2dB so the blended sound doesn't clip.
- The chorus should feel like birds calling together, not like layered loops.

### Listen-in mix decay

- On listen-in engage:
  - Focused bird dry-gain ramps +6dB over 1.5s.
  - Other birds dry-gain ramps -4dB over 1.5s.
  - Ambient bed (wind, foliage rustle) unchanged.
- On listen-in disengage:
  - Reverse ramp over 1.5s.
- Never mute non-focused birds entirely. They remain audible at ambient level.

### WebAudio fallback

- If `AudioContext` creation fails or the user has denied audio permission:
  - Enter "graceful silence": all synthesis stops.
  - Automatically enable call captions if not already enabled.
  - Display a small, non-intrusive indicator in the top bar (muted speaker icon) that the user can click to retry audio initialization.
- No recorded-audio fallback path. Silence + captions is the only fallback.

### Audio budget

- Procedural synthesis code (motif libraries, envelope generators, worklet) must fit within the 2MB JS bundle cap. Target: <300KB for all audio code + motif data.
- Do not load external audio assets. All sound is synthesized from oscillators, noise buffers, and wavetable lookups included in the bundle.

---

## 9. Accessibility Surfaces

### Screen-reader narration

- **Implementation:** A hidden live region (`aria-live="polite"`, `aria-atomic="false"`) in the DOM receives narration prose generated from the snapshot.
- **Generation logic:** Runs client-side (to avoid extra server round-trips). A small narration module consumes the snapshot and composes prose sentences:
  - "a small grey bird is perched on the front rail, calling softly."
  - "another bird sits further back with feathers fluffed."
  - "it is morning in the aviary; the light is gentle."
- **Cadence:** Pushes new prose every 45s at idle. User-initiated events (offer, settle, listen-in) push immediately, replacing the idle queue.
- **Voice continuity:** Uses the same naturalist register as the field notebook. Never state-list format. Never expose mood labels or perch zone names.
- **Priority queue:**
  1. Immediate: return-greeting, offer reaction, settle trigger/unsettle.
  2. Fast (within 10s): listen-in engage/disengage, mood transitions the user initiated.
  3. Normal (45s): ambient observation updates.

### Call captions

- When enabled, each call triggers a small text element near the calling bird's position (HTML overlay, absolutely positioned, fading in/out with the call).
- Text is generated from the same procedural call parameters: "a soft three-note rise", "a low trill, paused, low trill again".
- Captions use the naturalist voice and pass WCAG AA contrast against a semi-transparent dark background pill.
- Caption duration matches call duration + 1s fade-out.

### Focus and keyboard navigation

- `Tab` cycles through top bar items in DOM order.
- Entering the aviary scene (after top bar, or via a dedicated "Enter aviary" skip link): focus lands on the first bird.
- `ArrowLeft` / `ArrowRight` move focus between birds in visual left-to-right order.
- `Enter` on a bird triggers listen-in (if not already focused) or disengages listen-in (if already focused).
- `Escape` exits listen-in and returns focus to the aviary container.
- `Space` or `Enter` on the offer affordance opens the offer palette; `ArrowUp`/`ArrowDown` selects offer type; `Enter` confirms; `Escape` closes.
- Settle is reachable from the top bar via `Tab` and activated with `Enter`.
- Focus indicator: a 3px soft outline (`#ffffff` with 0.8 opacity, 2px blur) that is visible against all day-phase palettes.

### Contrast

- All text in the top bar, settings panels, account surfaces, error messages, and captions must pass WCAG AA (4.5:1 for normal text, 3:1 for large text).
- The aviary scene itself contains no text (except captions, which have their own background pill), so contrast requirements apply to chrome only.
- Color palette for chrome: derived from the design system; avoid relying on color alone for state indication.

---

## 10. Performance Budgets and Observability

### Bundle size

- **Hard cap:** 2MB gzipped for the initial JS bundle at first paint.
- **Code-splitting boundaries:**
  - Core aviary runtime (renderer, audio, presence): loaded synchronously.
  - Account settings, accessibility settings, visit-invitation flow: lazy-loaded on first access.
  - Notebook: loaded eagerly (it's a lightweight text list) but can be code-split if bundle pressure demands.
- **Asset strategy:** Bird visuals are procedural SVGs or compact vector paths generated from species templates + plumage_saturation. No large bitmap assets. Background foliage is tiled SVG patterns.

### Time to first bird

- **Target:** <500ms from navigation start to first bird visible on mid-tier mobile / 4G.
- **Implementation path:**
  1. Inline a minimal ~15KB boot script in HTML that paints the quiet field immediately.
  2. Stream the main bundle via `<script type="module">` with `async` loading.
  3. Snapshot request is fired by the boot script as soon as `navigator.sendBeacon` or `fetch` is available, parallel to bundle download.
  4. First bird renders as soon as both bundle init and snapshot response complete; typical path is bundle wins, then snapshot arrives 50–150ms later.
  5. Use HTTP/2 server push (or 103 Early Hints) for the snapshot endpoint if the auth token is present in a cookie.

### Runtime budgets

- **Idle motion:** 60fps on a 5-year-old mid-range laptop (e.g., 2019 MacBook Air, Intel i5).
  - Target: <8ms per frame for scene update + render.
  - Use `requestAnimationFrame`; throttle non-visual updates (e.g., narration text composition) to every 5 frames.
- **Memory:** Zero net memory growth over 30 minutes.
  - Procedural audio: pre-allocate AudioBuffer pools for motif fragments; do not allocate per-call.
  - Canvas: reuse offscreen canvases for compositing; do not create new canvas elements per frame.
  - Notebook: virtualized scroll list; detach DOM nodes for off-screen entries.
  - Workers: one dedicated animation worker (if using OffscreenCanvas), one audio worklet. No unbounded spawning.
  - **CI test:** Run a 30-minute automated session in Chrome headless; fail if `performance.memory.usedJSHeapSize` delta > 1MB.

### WebAudio latency

- Audio context latency mode: `latencyHint: 'interactive'`.
- Call scheduling: use `AudioContext.currentTime` + lookahead (100ms) for buffer scheduling to avoid glitches.

### Observability

- **Synthetic monitoring:** A fleet of headless browsers (e.g., Playwright) running from 3+ geographies, continuously loading the aviary and recording:
  - Time-to-first-bird (median and p95).
  - Bundle download time.
  - Snapshot API latency.
  - Audio context initialization success rate.
- **Real User Monitoring (RUM):** Aggregate only.
  - Page load timing (TTFB, FCP, LCP).
  - First-bird-render timing (custom metric).
  - Render-frame timing (median frame duration, dropped frame count per session).
  - Audio-context error rate.
  - Simulation-tick latency (measured server-side, not client).
- **Server metrics:**
  - Tick execution latency (p50, p99).
  - Snapshot API latency (p50, p99).
  - Event ingestion rate and lag.
  - Error rates per endpoint.
- **Alert thresholds:**
  - Tick latency p99 > 5s → page on-call.
  - Snapshot p99 latency > 1s → investigate.
  - Audio-context init failure rate > 2% → investigate browser compatibility.
  - Time-to-first-bird p95 > 1s on synthetic → investigate CDN / bundle.

### What we deliberately do not measure

- **Per-bird behavioral analytics:** We do not track which bird is clicked most, which offer is most popular, or how long users listen-in per bird. These would violate the privacy boundary and create data that could later justify gamification or leaderboards.
- **Individual-user session recordings:** No FullStory / Hotjar-style replay tools.
- **Engagement funnels:** No "activation" metrics, no "session depth" scoring. The product is not optimized for engagement time.

---

## 11. Rollout

### Phased launch

1. **Internal alpha (weeks 1–4):** Team + friends. 50 accounts. Goal: validate tick cadence, drift calibration, audio mix on real devices. Birds capped at 2. Notebook generation tuned.
2. **Closed beta (weeks 5–10):** 1,000 invite-only accounts. Goal: stress-test sync, multi-device usage, presence accounting accuracy, visit feature. Birds capped at 3. Monitor tick latency and event-log growth.
3. **Public beta (weeks 11–16):** Open sign-up, waitlist if needed. Birds capped at 5. Goal: validate performance budgets at scale, gather accessibility feedback, tune call-caption heuristics.
4. **General availability (week 17+):** Full cap at 7 birds. Remove waitlist.

### Ramping birds-per-aviary

- The bird cap is controlled by a feature flag / configuration value `max_birds_per_aviary` per environment.
- New birds become "available" based on `aviary_age_days`, not user interaction. The server checks age on tick and, if a new-bird threshold is crossed, sets a `pending_adoption` flag on the account. The client surfaces a subtle indicator (a soft chirp cue from an unseen bird) that a new bird is available.
- **Age thresholds (v1 calibration):**
  - 3rd bird: 60 days.
  - 4th bird: 120 days.
  - 5th bird: 180 days.
  - 6th bird: 270 days.
  - 7th bird: 365 days.
- These are tunable constants. The key constraint: new birds must never feel earned by attention; they feel like the aviary maturing.

### Instrumentation from day one

- All server metrics and synthetic checks running from alpha.
- Privacy audit in week 2: verify that per-bird event logs are not flowing into analytics warehouse.
- Accessibility audit in week 3: screen-reader walkthrough with actual assistive-tech users, not just axe-core.
- Load test in week 6: simulate 10,000 concurrent accounts, verify tick scheduling and snapshot p99 latency.

### Feature flags

- `max_birds_per_aviary`
- `visit_feature_enabled` (default true, but gated for quick kill-switch)
- `call_captions_enabled`
- `reduced_motion_override`
- `drift_constants_v2` (for A/B calibration tuning)

---

## 12. Risks

### Drift calibration risk (HIGH)

**What could go wrong:** The drift function is tuned too fast or too slow. Too fast → birds feel like Tamagotchis; too slow → the product feels inert. Calibration after launch is hard because changing constants retroactively changes the perceived continuity of birds users have already bonded with.

**Mitigation:**
- Make drift constants externally configurable (feature-flag / config table) so tuning does not require deploys.
- Run a 3-week internal alpha with mock-heavy usage to instrument before any public user sees drift.
- Provide a `drift_constants_v2` flag for gradual rollout; if a cohort reports birds feeling wrong, roll back constants for that cohort without affecting others.
- Automated calibration tests in CI: given a synthetic event log representing "30 min/day for 7 days," assert that total vector change > threshold; given "30 min/day for 21 days," assert visible heuristic changes (e.g., vocal_frequency > 0.3).

### Sync correctness risk (HIGH)

**What could go wrong:** A misunderstood client-side caching layer, an optimistic update, or a mobile background-tab edge case causes the client to compute or cache a stale personality vector. The user sees a bird "snap back" to an earlier state.

**Mitigation:**
- Strict separation: client code has no write path to personality or mood. Any PR that adds client-side state mutation to these fields is a build-blocking lint rule.
- Snapshot versioning: each snapshot includes `state_version` (a monotonic counter or timestamp). The client logs `state_version` with any bug report; support can detect desync.
- Event-log durability: client events are written to `localStorage` / `IndexedDB` until ACK'd by server. On pagehide, flush via `navigator.sendBeacon`. Never drop events silently.

### Audio uncanniness risk (MEDIUM-HIGH)

**What could go wrong:** Procedural calls sound synthetic or "bloopy." Users hear the algorithm. The chorus mixes poorly. The listen-in ramp feels like a UI effect, not like moving closer.

**Mitigation:**
- Hire or contract a sound designer with procedural audio experience (SuperCollider, Max/MSP, WebAudio) for the motif library. This is not a generalist engineering task.
- A/B test call quality internally: "Does this sound like a real bird?" Pass/fail gate before beta.
- Accept that some users will prefer silence: the fallback (silence + captions) is dignified and does not punish.

### Accessibility regression risk (MEDIUM-HIGH)

**What could go wrong:** A well-meaning performance optimization removes the live region; a refactor of the animation layer breaks `prefers-reduced-motion`; a new feature adds hover-only tooltips that are unreachable by keyboard.

**Mitigation:**
- Accessibility tests are CI-blocking: axe-core for contrast and structure, custom linter rules for `aria-live` presence, keyboard-nav traversal tests for every interactive element.
- Reduced-motion and screen-reader user flows are part of the Definition of Done for every feature.
- Budget for an accessibility audit by a third-party firm in week 3 of beta.

### Presence accounting corruption risk (MEDIUM)

**What could go wrong:** A browser update changes `visibilityState` behavior; a laptop sleep/wake cycle generates spurious presence events; a user keeps the tab visible but walks away for hours.

**Mitigation:**
- The three-factor check (visibility + focus + recent input) is robust but must be tested across Chrome, Safari, Firefox, Edge on Windows, macOS, iOS, Android.
- Client-side input detector: track `lastInputAt` timestamp. Re-evaluate presence every 10s. If `lastInputAt` is older than the calibrated window (start with 3 minutes, tune in beta), stop emitting pings even if visibility and focus are true.
- On `pagehide` / `beforeunload`, emit an `end` event to close the presence block cleanly.
- Server-side sanity filter: if a single device reports presence > 4 hours in one block, flag for review but do not reject it automatically (the user might genuinely be watching).

### Privacy boundary erosion risk (MEDIUM)

**What could go wrong:** An engineer joins the analytics team and asks for "just a histogram of offer types per species." The pipeline is built; later, someone asks for "average drift speed per cohort." The boundary dissolves.

**Mitigation:**
- Architectural enforcement: per-account simulation DB and event-log DB are on isolated network segments with read-only service accounts. The analytics warehouse has no credentials to these DBs.
- Code review checklist: any query touching `interaction_event` or `presence_event` requires security-team approval.
- Telemetry schema review: RUM events are reviewed in public RFCs; per-bird fields are forbidden by schema contract.

### Performance budget overrun risk (MEDIUM)

**What could go wrong:** The procedural audio engine + motif library + SVG bird assets exceed 2MB. The team considers "just raising the cap to 3MB."

**Mitigation:**
- Bundle size is a CI gate: `webpack-bundle-analyzer` output is attached to every PR. PRs that increase the initial bundle by >50KB require explicit approval.
- Audio is the likely overrun: set a sub-budget of 300KB for the audio module. If motifs exceed this, reduce polyphony or species count (from 6 to 4) rather than raise the cap.
- If 2MB proves genuinely impossible, escalate to product; do not silently raise. The 500ms time-to-first-bird depends on it.

### Visit feature abuse risk (LOW-MEDIUM)

**What could go wrong:** Users share visit links publicly; a host's aviary becomes an ambient stream for strangers.

**Mitigation:**
- Invitations are per-email (hashed) and tokenized; there is no global URL pattern.
- Tokens expire in 30 days; no renewal.
- Host can revoke at any time.
- No embedded player or iframe support; the page requires a full browser context.
- Terms of service (handled by legal, not engineering) prohibit public redistribution of visit links.

---

## 13. Engineering Discipline & Guardrails

### Code-level rules

1. **No client-side personality mutation.** Any code that writes to personality vector fields outside the simulation tick is a build failure.
2. **No numerical exposure.** Personality vectors are never serialized to client-facing JSON. If a debug panel is needed, it is behind a feature flag that is off in all production builds.
3. **No gamification primitives.** No counters, no streak logic, no achievement checks, no "days since" surfaces. Lint rules block common terms (`streak`, `achievement`, `level`, `score`, `badge`, `XP`) in UI string files.
4. **No recorded audio.** No `.mp3`, `.ogg`, `.wav` assets in the audio pipeline. Lint rule blocks audio file imports in the client bundle.
5. **No announcement toasts.** No `toast`, `banner`, `notification`, `alert` components may be used in the aviary scene or on session start. System surfaces (auth, errors, settings) are the only allowed locations.

### Review & QA gates

- Every PR requires: unit tests, bundle-size report, accessibility scan, and a manual check of the "notice, never announce" principle for any new UI.
- Weekly calibration review during alpha/beta: examine instrumented drift rates across 10 internal test accounts; adjust constants if needed.
- Pre-release checklists include: screen-reader narration flow, keyboard-only session, reduced-motion session, 30-minute memory profile, 4G throttled load test.

---

## 14. Ambiguities & Defensible Calls

1. **Tick cadence:** PRD says "~once per minute." This plan chooses 60s ± 5s jitter. Actual wall-clock delta since last tick is used for drift computation, so jitter does not affect correctness.
2. **Presence activity window:** PRD says "a few minutes" and suggests calibrating during build. This plan starts with 3 minutes and proposes tuning during beta. The server does not enforce the window; the client stops pinging after inactivity, and the server trusts the event log.
3. **Mood exact set:** PRD lists `wary, content, curious, drowsy, alert` and notes "exact set finalized in implementation." This plan adds `settled` as a mood state (distinct from `drowsy`) to represent the post-settle evening state, because `settled` is used as an adjective for the lighting state and should be represented in the mood enum for consistency.
4. **Notebook generation heuristics:** PRD specifies sparsity and naturalist voice but does not specify the algorithm. This plan proposes a noteworthiness-score threshold model; the exact scoring function is left to implementation but must be instrumented and tunable.
5. **Canvas vs. DOM for scene:** PRD does not mandate a rendering technology. This plan chooses Canvas 2D for performance and visual control, with the caveat that WebGL is an acceptable alternative if the team has the expertise. The critical requirement is the 60fps budget.
6. **Audio synthesis detail:** PRD mandates procedural synthesis but does not specify synthesis method (FM, subtractive, wavetable, etc.). This plan leaves the synthesis technique to the audio engineer but mandates the budget, the variation requirements, and the WebAudio-only constraint.
7. **New bird age thresholds:** PRD describes the concept but does not give exact days. This plan proposes a graduated schedule (60/120/180/270/365 days). These are tunable and should be validated against beta cohort retention data.

---

*Plan end. This document is a specification for engineering execution, not a product requirements document. Implementation code must trace back to the decisions above, and deviations require an explicit engineering decision record (EDR).*
