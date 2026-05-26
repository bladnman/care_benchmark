# Pocket Aviary — Comprehensive Implementation Plan (v1)

## 1. Scope

### 1.1 In scope for v1

- **Browser-only client** — single-page web application, last two major versions of Chrome, Safari, Firefox, Edge.
- **Single-user accounts** — email + magic-link auth, synthetic UUID account IDs, per-device session tokens.
- **Aviary simulation** — server-side tick (~1 min cadence), personality vectors, mood state machine, procedural call grammar, drift function, bird-to-bird interactions.
- **Two starter birds**, cap at seven. Six-species pool. Age-gated adoption of additional birds.
- **Interactions** — return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook browsing.
- **Presence accounting** — three-signal conjunction (visibility + focus + recent input).
- **Multi-device sync** — server-canonical architecture; clients pull snapshots and post events.
- **Visit invitations** — per-invite, opt-in, read-only ambient view, revocable, 30-day expiry.
- **Accessibility** — screen-reader narration (naturalist prose), reduced-motion mode (cross-fade rendering), call captions, WCAG AA contrast, full keyboard navigation.
- **Performance budgets** — <2 MB initial JS bundle (gzipped), <500 ms time-to-first-bird, 60 fps idle motion on 5-year-old laptop, zero memory growth over 30 min.
- **Account management** — export (JSON), soft-then-hard deletion (30-day window), email change with verification, session revocation.

### 1.2 Explicitly out of scope

- Native mobile apps (iOS, Android).
- All gamification: achievements, streaks, levels, scores, badges, calendars, XP, tiers, ranks.
- Tamagotchi mechanics: death, hunger, distress, happiness decay, custodial obligation.
- Social-network surfaces: profiles, follows, public feeds, discovery, leaderboards, comments, co-presence.
- Push notifications, email notifications about aviary state (except magic-link and account-export delivery).
- Shared aviaries, multi-aviary accounts, customizable scenes.
- Payments / billing.
- Any surface that exposes personality vector values numerically to the user.

---

## 2. Architecture

### 2.1 Service topology

```
┌─────────────┐      HTTPS       ┌──────────────────┐
│  Browser     │ ◄──────────────► │  API Gateway      │
│  (SPA)       │   REST + SSE     │  (edge / CDN)     │
└─────────────┘                  └────────┬─────────┘
                                          │
                     ┌────────────────────┼────────────────────┐
                     │                    │                    │
              ┌──────▼──────┐   ┌────────▼────────┐  ┌───────▼───────┐
              │ Auth Service │   │ Aviary Service   │  │ Visit Service  │
              │ (magic-link, │   │ (snapshot, event  │  │ (invite,       │
              │  sessions)   │   │  ingest, tick)    │  │  read-only     │
              └──────┬──────┘   └────────┬────────┘  │  snapshot)       │
                     │                    │           └───────┬───────┘
              ┌──────▼──────┐   ┌────────▼────────┐  ┌───────▼───────┐
              │ Account DB   │   │ Simulation DB    │  │ Visit DB       │
              │ (Postgres)   │   │ (Postgres)       │  │ (Postgres)     │
              └─────────────┘   └─────────────────┘  └───────────────┘
                                        │
                                ┌───────▼───────┐
                                │ Tick Scheduler │
                                │ (cron / worker)│
                                └───────────────┘
```

**Rationale for three services rather than one monolith:**
- Auth is security-critical and benefits from isolation (secrets, rate-limiting, brute-force protection).
- The Aviary Service owns the simulation and is the hot path for both reads (snapshots) and writes (events, ticks).
- The Visit Service is a thin read-only projection; isolating it prevents visitor traffic from impacting the host's simulation path and enforces the "visitor never writes" rule at the network boundary.

All three services share a Postgres cluster (separate schemas, not separate instances at v1 scale) to keep operational complexity low. If the simulation DB grows, it can be split to its own instance without schema changes.

### 2.2 Client/server split

| Responsibility | Client | Server |
|---|---|---|
| Rendering, animation, idle motion | ✅ | |
| Procedural audio synthesis | ✅ | |
| Presence detection (3-signal conjunction) | ✅ | |
| Interaction event capture | ✅ | |
| Snapshot interpolation | ✅ | |
| Reduced-motion / caption rendering | ✅ | |
| Screen-reader narration generation | ✅ (from snapshot) | |
| Personality vector storage & mutation | | ✅ (only writer) |
| Mood state transitions | | ✅ |
| Drift computation | | ✅ |
| Tick scheduling | | ✅ |
| Notebook entry generation | | ✅ |
| Call-grammar parameters (per-bird motif seeds) | | ✅ (distributed in snapshot) |
| Account auth, session management | | ✅ |
| Visit invitation flow | | ✅ |

**Key invariant:** The client never writes personality state. The client never computes drift. The client renders snapshots and posts events. This is the architectural rule that makes multi-device sync correct and makes the "no last-write-wins" guarantee enforceable.

### 2.3 Render pipeline boundary

The client's render pipeline consumes a **StateSnapshot** object (see §4.3) and produces visual + audio output. The boundary is:

```
StateSnapshot ──► Interpolator ──► Scene Graph ──► Renderer (Canvas/WebGL)
                                      │
                                      ├──► Audio Scheduler ──► WebAudio Graph
                                      │
                                      └──► Narration Generator ──► ARIA live region
                                      │
                                      └──► Caption Renderer ──► DOM overlay
```

The Interpolator smooths between snapshot N and snapshot N+1 for bird positions and motion phases. The Scene Graph is a retained-mode structure of bird sprites, perch positions, ambient elements, and lighting state. The Audio Scheduler uses call-grammar parameters from the snapshot to schedule procedural synthesis in the WebAudio graph.

---

## 3. Data Model

### 3.1 Core entities

#### Account

```
account {
  id: UUID (synthetic, generated at creation)
  email_encrypted: bytea (AES-256-GCM, key from KMS)
  email_hash: bytea (SHA-256, for uniqueness check only)
  created_at: timestamptz
  deleted_at: timestamptz | null
  deletion_scheduled_at: timestamptz | null
  settings: jsonb (reduced_motion, captions_enabled, visit_notifications)
}
```

The synthetic UUID is the **only** identifier used anywhere outside the account record itself. Email is stored encrypted, hashed for uniqueness, and never appears in logs, partition keys, telemetry, or inter-service messages.

#### Session (auth)

```
session {
  id: UUID
  account_id: UUID (FK → account)
  token_hash: bytea (SHA-256 of bearer token)
  device_label: text (user-agent summary)
  created_at: timestamptz
  expires_at: timestamptz
  revoked_at: timestamptz | null
}
```

#### Bird

```
bird {
  id: UUID (stable, never changes)
  account_id: UUID (FK → account)
  species: enum (warbler, finch, wren, thrush, nightjar, bunting)
  name: text (user-assigned, renameable)
  adopted_at: timestamptz
  personality: jsonb {
    boldness: float [0..1]
    social_warmth: float [0..1]
    vocal_frequency: float [0..1]
    plumage_saturation: float [0..1]
    curiosity: float [0..1]
  }
  mood: enum (wary, content, curious, drowsy, alert)
  mood_updated_at: timestamptz
  perch_zone: enum (front, middle, back)
  call_grammar_seed: int (deterministic seed for procedural motif selection)
  last_greeting_at: timestamptz | null
}
```

Personality vector fields are floats normalized to [0, 1]. Seed values for newly adopted birds are drawn from species-specific distributions (implementation detail in simulation service config). The `call_grammar_seed` deterministically selects from the species' motif library so that each bird's call signature is stable and recognizable.

#### Interaction Event (append-only log)

```
interaction_event {
  id: UUID
  account_id: UUID
  bird_id: UUID | null (null for aviary-wide events like settle)
  event_type: enum (
    presence_ping,
    listen_in_start,
    listen_in_end,
    offer_seed,
    offer_song,
    offer_pool,
    settle,
    tab_open,
    tab_close
  )
  payload: jsonb (event-specific; e.g. listen_in_end includes duration_seconds)
  client_timestamp: timestamptz
  server_received_at: timestamptz
  consumed_by_tick: UUID | null (tick ID that processed this event)
}
```

This table is append-only. No UPDATE or DELETE operations. The tick scheduler marks events as consumed after processing. Events older than a retention window (e.g., 90 days) are archived to cold storage for account-export purposes but removed from the hot table.

#### Notebook Entry

```
notebook_entry {
  id: UUID
  account_id: UUID
  generated_at: timestamptz
  prose: text (naturalist voice, lowercase, present-tense)
  source_events: UUID[] (interaction_event IDs that contributed)
}
```

#### Visit Invitation

```
visit_invitation {
  id: UUID
  host_account_id: UUID
  visitor_email_hash: bytea
  token_hash: bytea
  created_at: timestamptz
  expires_at: timestamptz (created_at + 30 days)
  used_at: timestamptz | null
  revoked_at: timestamptz | null
}
```

#### Visit Log

```
visit_log {
  id: UUID
  host_account_id: UUID
  visitor_email_hash: bytea
  started_at: timestamptz
  ended_at: timestamptz | null
  duration_seconds: int | null
}
```

### 3.2 Personality vector — storage and mutation rules

- **Only the server-side tick writes personality vectors.** No other code path — no API endpoint, no admin tool, no migration script — mutates personality fields on the bird record except through the tick's drift computation.
- **Drift is additive.** The tick computes a delta (positive or zero) and adds it to the existing vector. The delta is clamped so the vector stays in [0, 1].
- **Drift is monotonic toward expressive.** Deltas are non-negative. A bird's traits never decrease. Neglect produces zero drift, not negative drift.
- **Personality is never exposed numerically.** No API response includes raw vector values. The snapshot sent to clients includes derived rendering hints (perch preference, call frequency, plumage detail level) but not the underlying floats.

### 3.3 Mood state machine

```
States: wary, content, curious, drowsy, alert

Transitions (weighted by personality + context):
  any → drowsy    : time-of-day near dusk/night, low interaction
  any → alert     : time-of-day early morning, ambient weather event (wind)
  any → wary      : another bird's alarm call, sudden ambient change
  any → content   : recent accepted offer, sustained presence
  any → curious   : recent offer nearby, high curiosity trait
  drowsy → content: morning time-of-day, presence detected
  wary → content  : sustained presence without interaction (patience signal)
  ...
```

The transition weights are a function of the bird's personality vector. A high-boldness bird has lower probability of entering `wary` on the same input. A high-curiosity bird has higher probability of entering `curious` on an offer event.

Mood persists across sessions. The mood at session-end is the starting mood at next session-start, modified by server-side ticks that ran in the interim (time-of-day transitions, ambient events).

---

## 4. API Surface

### 4.1 Authentication endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/magic-link/request` | Body: `{ email }`. Sends magic-link email. Rate-limited per email (5 req / 15 min). |
| POST | `/auth/magic-link/consume` | Body: `{ token }`. Validates token, creates session, returns bearer token. |
| POST | `/auth/session/revoke` | Body: `{ session_id }`. Revokes a session. Requires valid auth. |
| GET | `/auth/sessions` | Lists active sessions for the account. |
| DELETE | `/auth/session/:id` | Revokes a specific session. |

Magic-link tokens are single-use, expire after 15 minutes, and are stored as hashes. Consumption invalidates the token immediately.

### 4.2 Aviary endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/aviary/snapshot` | Returns current StateSnapshot. ETag for conditional requests. |
| POST | `/aviary/events` | Body: `{ events: InteractionEvent[] }`. Appends to event log. |
| GET | `/aviary/notebook?before=:cursor&limit=20` | Paginated notebook entries, newest first. |
| POST | `/aviary/settle` | Records settle event, triggers lighting transition. |
| GET | `/aviary/birds` | Returns bird list with names, species, rendering hints (no personality values). |
| PATCH | `/aviary/birds/:id/name` | Body: `{ name }`. Rename a bird. |

### 4.3 StateSnapshot schema

```json
{
  "snapshot_id": "uuid",
  "generated_at": "ISO-8601",
  "aviary": {
    "lighting": "morning | midday | evening | night | settled",
    "weather": "clear | rain | wind",
    "ambient_motion_seed": 12345
  },
  "birds": [
    {
      "id": "uuid",
      "name": "Pip",
      "species": "warbler",
      "mood": "content",
      "perch_zone": "front",
      "motion_state": "preening | scanning | calling | idle | fluffed",
      "motion_phase": 0.37,
      "call_grammar_seed": 8842,
      "call_scheduled_at": "ISO-8601 or null",
      "plumage_detail": 0.72,
      "greeting_pending": false,
      "offer_cooldown_until": "ISO-8601 or null"
    }
  ],
  "user_state": {
    "last_seen_at": "ISO-8601",
    "absence_duration_seconds": 3600,
    "settled": false
  }
}
```

The snapshot is ~2–5 KB. It contains everything the client needs to render the current state and schedule the next few seconds of audio. It does **not** contain personality vector values.

### 4.4 Visit endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/visit/invite` | Body: `{ visitor_email }`. Creates invitation, sends email. |
| GET | `/visit/invitations` | Lists outstanding and used invitations. |
| DELETE | `/visit/invitation/:id` | Revokes an invitation. |
| GET | `/visit/log` | Visit history for the host. |
| GET | `/visit/view/:token` | Visitor's read-only snapshot endpoint. Validates token, returns snapshot. |

The visitor's snapshot endpoint returns the same StateSnapshot structure but the visitor's client is configured to disable all interaction affordances (no offer button, no listen-in click handlers, no settle button). The visitor's client does not post events.

### 4.5 Account endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/account` | Account settings. |
| PATCH | `/account/settings` | Update settings (reduced_motion, captions, visit_notifications). |
| POST | `/account/email-change` | Body: `{ new_email }`. Sends verification to new address. |
| POST | `/account/export` | Triggers JSON export, emails download link. |
| POST | `/account/deletion` | Initiates soft deletion. |
| POST | `/account/deletion/cancel` | Cancels pending deletion. |

### 4.6 Real-time updates

The snapshot endpoint supports an optional SSE (Server-Sent Events) stream for clients that remain open:

```
GET /aviary/snapshot/stream
```

The server pushes a new snapshot on each tick (~1/min) and on significant events (mood transitions, weather changes). The client can also pull on demand (visibility change, keepalive). SSE is preferred over WebSocket because the data flow is unidirectional (server → client) and SSE has simpler reconnection semantics.

---

## 5. Simulation Engine Design

### 5.1 Tick scheduler

The tick runs on a ~60-second cadence. Implementation: a worker process (or pool) that:

1. Queries all active accounts (not soft-deleted, with at least one bird).
2. For each account, processes the tick in a single transaction.
3. Marks consumed events after successful processing.

At v1 scale (thousands of accounts), a single worker with batched queries is sufficient. The tick is idempotent: if a tick fails mid-processing, the next tick re-processes unconsumed events.

**Tick budget:** p99 latency < 5 seconds (alarm threshold). Expected per-account tick time: <50 ms. This leaves headroom for growth.

### 5.2 Tick processing (per account)

```
function processTick(accountId):
  1. Load bird records for account
  2. Load unconsumed interaction events since last tick
  3. Compute presence-time from presence_ping events
     - Validate each ping: was visibilityState=visible, focus=true, recent input?
     - (Client already filters; server trusts but logs anomalies)
     - Sum valid presence-time since last tick
  4. For each bird:
     a. Compute drift delta from:
        - presence-time (weighted by bird count — presence is shared)
        - listen_in events targeting this bird (duration-weighted)
        - offer events targeting this bird (accepted offers only)
        - Apply low-pass filter: delta = raw_delta * filter_alpha
        - Clamp: personality[trait] = min(1.0, personality[trait] + delta)
     b. Compute mood transition:
        - Evaluate transition weights given current personality, time-of-day,
          recent events, ambient weather
        - Sample next mood from weighted distribution
        - Only transition if sufficient time in current mood (minimum dwell: 5 min)
     c. Compute perch zone:
        - Function of mood + boldness trait
        - Wary → back; content/curious → front; drowsy → middle/back
     d. Compute motion state:
        - Function of mood (wary→scanning, content→preening, curious→tilting,
          drowsy→fluffed, alert→scanning)
     e. Schedule next call:
        - Interval drawn from distribution shaped by vocal_frequency trait
        - Higher vocal_frequency → shorter mean interval
  5. Compute bird-to-bird interactions:
     - If two+ birds have calls scheduled in overlapping windows → chorus event
     - If one bird enters wary → small probability of spreading to adjacent birds
  6. Compute ambient weather:
     - Stochastic: ~2-3 weather events per week per account
     - Weather affects mood transitions for duration of event
  7. Evaluate notebook entry generation:
     - Check if any noteworthy event occurred (first-greeter change, long quiet
       stretch, unusual chorus, mood shift pattern)
     - If yes and cooldown elapsed (min 2 days between entries): generate entry
     - Entry generation uses a template library + bird names + context
  8. Write updated bird records, mark events consumed, persist new notebook entries
```

### 5.3 Drift function — calibration

The drift function is a low-pass filter over presence-and-interaction signals. The calibration targets:

| Metric | Target |
|--------|--------|
| Measurable drift in instruments | ~1 week of regular visits (3+ sessions/week, 5+ min each) |
| Visible drift to user | ~3 weeks of regular visits |
| Single-session drift magnitude | < 0.005 on any trait (imperceptible) |
| Maximum trait value | 1.0 (clamped) |
| Drift on neglect | 0.0 (no negative drift) |

**Implementation:**

```
filter_alpha = 0.02  # per-tick smoothing factor
presence_weight = 0.6
listen_in_weight = 0.25
offer_weight = 0.15

raw_delta_trait = (
  presence_weight * normalized_presence_time +
  listen_in_weight * normalized_listen_in_duration +
  offer_weight * normalized_offer_events
)

# Low-pass filter
delta = raw_delta_trait * filter_alpha

# Per-trait modulation (boldness gets more from presence, vocal_frequency from listen-in, etc.)
delta_boldness = delta * 0.8
delta_social_warmth = delta * 1.0
delta_vocal_frequency = delta * (1.0 + listen_in_ratio)
delta_plumage_saturation = delta * 0.6
delta_curiosity = delta * (1.0 + offer_ratio)
```

These constants are configuration, not code. They will be tuned during build with a calibration harness that simulates weeks of synthetic user behavior and measures drift trajectories.

### 5.4 Call-grammar runtime

Each species has a **motif library** — a set of 8–12 short melodic fragments (intervals, rhythms, timbres) parameterized by:

- **Pitch base** — derived from species + individual `call_grammar_seed`
- **Pitch range** — modulated by mood (wary → narrow, content → wide)
- **Tempo** — modulated by vocal_frequency trait
- **Timbre** — species-specific base, slightly varied by plumage_saturation (richer plumage → richer harmonics)

At call time, the grammar selects 2–4 motifs, applies pitch and tempo transformations, and sequences them with small random variations in timing and pitch. The result is a call that is recognizable as "Pip's call" (same seed, same species motifs) but never identical twice.

**Chorus:** When two or more birds have calls scheduled within a ~2-second window, the audio scheduler on the client overlaps them. Because each call is procedurally generated from independent seeds, the overlap produces a real chorus — no phase cancellation, no stacking artifacts.

### 5.5 Notebook entry generation

The notebook entry generator runs as part of the tick. It evaluates a set of **noteworthy-event detectors**:

1. **First-greeter change** — the bird that greeted first today is different from the last N days.
2. **Long quiet stretch** — no presence events for >48 hours, followed by a return.
3. **Chorus event** — two+ birds called in overlapping windows (rare enough to note).
4. **Mood pattern** — a bird has been in the same mood for >3 days (unusual stability).
5. **Offer pattern** — a bird accepted N offers in a week (unusual engagement).
6. **Drift milestone** — a trait crossed a threshold (e.g., boldness > 0.7 for the first time).

When a detector fires and the per-account cooldown (min 2 days) has elapsed, the generator selects a prose template and fills in bird names, time references, and context. Templates are written in naturalist voice:

```
Template: "{bird_a} greeted before {bird_b} today, first time this {period}."
Template: "a long stretch of quiet. {bird} preened for several minutes without looking up."
Template: "{bird} is fluffed against the cool air, watching the {perch} perch. low calls only."
```

The template library is a curated set (~50 templates at v1), not generated by an LLM. LLM-generated prose would risk voice inconsistency and hallucinated details. The templates are hand-written and reviewed for voice compliance.

---

## 6. Sync Model

### 6.1 Single canonical state

The server holds the single canonical state for each aviary. All clients read from this state. There is no client-side state to merge, no eventual consistency to reconcile.

```
Client A (laptop) ──pull──► Server (canonical state) ◄──pull── Client B (phone)
Client A ──post events──► Server (event log) ◄──post events── Client B
Server tick ──reads events, writes state──► Server (canonical state)
```

### 6.2 Snapshot pull triggers

The client pulls a fresh snapshot on:

1. **Initial page load** — first snapshot, embedded in HTML for fast first paint.
2. **Visibility change** — `document.visibilityState` transitions from `hidden` to `visible`.
3. **SSE stream update** — server pushes notification of new snapshot available.
4. **Keepalive** — every 5 minutes while tab is visible (fallback if SSE disconnects).
5. **Resume from suspend** — detected by large gap between `requestAnimationFrame` timestamps (>2 seconds).

### 6.3 Conflict prevention

Personality state conflicts are **architecturally impossible** because:

- Only the server tick writes personality vectors.
- Clients write only to the append-only event log.
- The tick processes events in insertion order (by `server_received_at`).
- Drift is additive; there is no "set to value" operation that could overwrite.

The only potential conflict surface is **session-level**: two devices posting events simultaneously. This is safe because the event log is append-only and order-independent for drift computation (the tick aggregates events over a window, not per-event).

### 6.4 Snapshot interpolation

Between snapshot N and snapshot N+1, the client interpolates:

- **Bird positions:** linear interpolation between perch zones (with easing).
- **Motion state transitions:** cross-fade between animation states over ~500ms.
- **Lighting:** smooth color-temperature transition over the tick interval.
- **Mood changes:** immediate on snapshot receipt (mood is a discrete state, not continuous).

If a snapshot is missed (network hiccup), the client continues rendering the last known state with extrapolated idle motion. On the next snapshot, it snaps to the correct state with a smooth transition.

---

## 7. Frontend Rendering Pipeline

### 7.1 Technology choice

**Canvas 2D** for the aviary scene (not WebGL, not DOM-based). Rationale:

- Canvas 2D is sufficient for the visual complexity (sprites, parallax, lighting overlays).
- WebGL would add complexity without proportional benefit at this scene complexity.
- DOM-based rendering would not hit 60fps with 7 birds + ambient motion + parallax.
- Canvas 2D has excellent browser support across the target matrix.

The top bar, notebook, settings, and other UI chrome are standard DOM elements positioned above the canvas.

### 7.2 Scene composition

```
Layers (back to front):
  1. Sky gradient (time-of-day colored)
  2. Background foliage (static SVG, tinted by lighting)
  3. Back perch zone (birds at back scale)
  4. Middle perch zone (birds at middle scale)
  5. Ambient elements (leaves, feathers — client-generated)
  6. Front perch zone (birds at front scale)
  7. Foreground elements (occasional branch, rain overlay)
  8. Lighting overlay (time-of-day color wash, settle dimming)
  9. UI chrome (top bar, DOM elements above canvas)
```

### 7.3 Bird rendering

Each bird is rendered as a set of layered sprites with procedural variation:

- **Base sprite:** species-specific silhouette (SVG, <5 KB per species).
- **Plumage layer:** color-mapped from `plumage_detail` rendering hint. Higher values → richer gradients, more feather detail.
- **Pose:** selected from a pose library per species (8–12 poses: perched, preening, scanning, calling, fluffed, head-tilt).
- **Motion:** smooth interpolation between poses. Idle micro-motion is a subtle oscillation applied to the current pose.

Bird assets total: ~6 species × ~10 poses × ~2 KB avg = ~120 KB. Loaded progressively; first paint uses a single pose per species.

### 7.4 Idle micro-motion

Every bird has continuous idle motion regardless of user attention:

- **Preening:** small head movements, wing adjustments (content mood).
- **Scanning:** slow head rotation, occasional glance (wary/alert mood).
- **Fluffed:** gentle breathing oscillation, feathers expanded (drowsy mood).
- **Tilting:** head-tilt toward ambient sounds (curious mood).

Idle motion is generated client-side from the bird's `motion_state` and `motion_phase` in the snapshot. The client advances `motion_phase` locally between snapshots for continuous animation.

### 7.5 Day/night cycle

The lighting state is computed server-side from the user's local timezone (stored in account settings or inferred from browser `Intl.DateTimeFormat().resolvedOptions().timeZone`):

| Time range | Lighting state | Palette |
|------------|---------------|---------|
| 05:00–07:00 | dawn | warm pinks, soft gold |
| 07:00–11:00 | morning | gentle blues, soft light |
| 11:00–15:00 | midday | brightest, neutral |
| 15:00–18:00 | afternoon | warming, golden |
| 18:00–20:00 | evening | warm oranges, dimming |
| 20:00–05:00 | night | deep blues, dim |

Transitions are smooth gradients over the time range, not discrete switches.

### 7.6 Reduced-motion mode

When `prefers-reduced-motion: reduce` is set or the user opts in via settings:

- **Bird motion:** replaced by slow cross-fades between still poses (1–2 second transitions).
- **Ambient elements:** leaf/feather drift removed entirely.
- **Lighting transitions:** slowed to 5-second cross-fades.
- **Flight transitions:** cross-fade between perch positions rather than animated paths.
- **Calls:** still play at full quality (audio is not reduced).
- **Captions:** still appear.
- **Narration:** still generated.

The reduced-motion surface is a **designed aesthetic**, not a degraded fallback. The cross-fade rendering has its own calm quality.

### 7.7 Loading sequence

1. **HTML loads** with embedded initial snapshot (server-rendered into the page).
2. **Canvas initializes** and renders the first frame from the embedded snapshot — birds are already in position, mid-motion.
3. **Audio context** initializes on first user gesture (browser autoplay policy).
4. **Full JS bundle** loads and hydrates the interactive layer.

If the snapshot is not yet available (cold start, slow connection), the loading state is a **quiet field** — the sky gradient and background foliage with no birds. No spinner. No progress bar. The quiet field reads as "the aviary is here, the birds are just not visible yet."

---

## 8. Audio Pipeline

### 8.1 Procedural call synthesis

Calls are synthesized client-side via the **WebAudio API**. The synthesis chain per call:

```
Motif Selection → Pitch Transform → Tempo Transform → Envelope → Gain → Mixer → Destination
```

**Motif library:** Each species has 8–12 motifs, each defined as:
- A sequence of 2–6 notes (pitch intervals relative to base)
- A rhythm pattern (note durations, rests)
- A timbre descriptor (harmonic content, attack/decay shape)

**Synthesis:**
- Oscillators: sine + triangle waves with species-specific harmonic ratios.
- Envelope: ADSR (attack-decay-sustain-release) per note, shaped by mood.
- Pitch variation: ±5% random detune per note for naturalness.
- Timing variation: ±10% random timing jitter per note.

**Per-bird call signature:** The `call_grammar_seed` deterministically selects a subset of motifs and a base pitch, ensuring each bird's calls are recognizable across sessions.

### 8.2 Chorus mixing

When multiple birds call simultaneously:

- Each bird's call is synthesized on its own audio channel.
- A mixer node combines channels with per-bird gain.
- In ambient mode: all birds at equal gain (±small random variation for spatial feel).
- In listen-in mode: focused bird at 0 dB, others at -12 dB (not muted).
- Gain transitions are ramped over 1–2 seconds (no hard cuts).

### 8.3 Listen-in mix

On listen-in engage:
1. Focused bird's gain ramps from ambient level to 0 dB over 1.5 seconds.
2. Other birds' gains ramp from ambient level to -12 dB over 1.5 seconds.
3. On disengage: reverse ramps over 1.5 seconds.

The ramps use exponential curves (perceptually linear) via WebAudio's `exponentialRampToValueAtTime`.

### 8.4 Ambient audio

- **Background ambience:** a low-level generative ambient (soft wind, distant leaves) synthesized from filtered noise. Continuous, very quiet.
- **Weather audio:** rain is filtered noise with a specific spectral shape; wind is a slow LFO-modulated noise. Triggered by the weather state in the snapshot.

### 8.5 WebAudio fallback

If WebAudio is unavailable:
- **No audio plays.** The aviary is silent.
- **Captions are enabled by default** (overriding the user's setting).
- **No recorded-audio fallback.** The "no recorded audio" rule is unconditional.

Detection: attempt `new AudioContext()` on first user gesture. If it throws or returns null, enter fallback mode.

### 8.6 Audio budget

- Motif library: ~20 KB (JSON definitions, not audio files).
- Synthesis code: ~30 KB (minified).
- Total audio pipeline: <50 KB of the bundle budget.

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

**Implementation:** An ARIA live region (`role="log"`, `aria-live="polite"`) that receives prose updates.

**Generation:** The narration generator runs client-side from the snapshot state. It produces prose updates on a slow cadence (~30–60 seconds at idle):

```
Input: snapshot with 2 birds, morning lighting, Pip on front perch preening, Wren on back perch scanning
Output: "a small warbler is perched on the front rail, preening quietly. another bird sits further back, watching the branches. it is morning; the light is gentle."
```

**Priority queue:**
- User-initiated events (greeting, offer reaction, settle) → narrated within 2 seconds.
- State changes (mood transition, perch change) → narrated within 15 seconds.
- Ambient updates (time-of-day, weather) → narrated within 60 seconds.

**Voice:** Same naturalist field-notebook voice as the rest of the product. Lowercase, present-tense, specific. Never announcement-style.

### 9.2 Reduced-motion mode

See §7.6. Key implementation points:
- Detect `prefers-reduced-motion` media query on load.
- Override available in accessibility settings (persisted to account).
- Cross-fade duration: 1.5 seconds for pose changes, 5 seconds for lighting.
- No leaf/feather drift. No parallax.

### 9.3 Call captions

**Implementation:** DOM overlay positioned near the calling bird's screen coordinates.

**Caption text:** Generated from the call grammar at runtime. Each motif has a prose descriptor:

```
Motif "rising-third" → "a soft three-note rise"
Motif "low-trill" → "a low trill, paused, low trill again"
Motif "sharp-alert" → "a single sharp call from the {perch} perch"
```

The caption compositor selects descriptors for the motifs actually played and assembles them into a short phrase. Captions fade in over 300ms, persist for the call duration, fade out over 300ms.

**Opt-in:** Disabled by default. Enabled via accessibility settings. Auto-enabled when WebAudio is unavailable.

### 9.4 Keyboard navigation

| Key | Action |
|-----|--------|
| Tab | Move through top bar items |
| Tab (in aviary) | Focus first bird |
| Arrow Left/Right | Move focus between birds |
| Enter | Listen-in on focused bird |
| Escape | Exit listen-in |
| S | Trigger settle (from top bar focus) |
| N | Open notebook (from top bar focus) |
| O | Open offer menu (from top bar focus) |

**Focus indicator:** A soft, high-contrast outline (2px, color chosen to contrast both bright and dim aviary states). Specified in the design system as `outline: 2px solid var(--focus-ring)` with `--focus-ring` set to a color that passes 3:1 contrast against both the brightest and dimmest scene states.

### 9.5 Contrast

All user-copy text passes WCAG AA (4.5:1 for normal text, 3:1 for large text). The top bar uses a semi-transparent background with sufficient contrast for its labels. Caption text uses a text-shadow or background pill for contrast against the variable aviary scene.

---

## 10. Performance Budgets and Observability

### 10.1 Bundle budget

| Component | Budget |
|-----------|--------|
| Core app (router, state, API client) | 200 KB |
| Rendering engine (Canvas 2D, scene graph) | 300 KB |
| Audio pipeline (WebAudio, motif library) | 50 KB |
| Bird assets (SVGs, pose data) | 150 KB |
| Accessibility (narration, captions) | 30 KB |
| UI chrome (top bar, notebook, settings) | 100 KB |
| Code-split chunks (loaded on demand) | — |
| **Total initial bundle** | **<830 KB minified, <2 MB gzipped** |

Code-split chunks (loaded on first access):
- Account settings: ~50 KB
- Accessibility settings: ~30 KB
- Visit invitation flow: ~40 KB
- Account export/deletion: ~20 KB

### 10.2 Time to first bird

**Target: <500 ms** on mid-tier mobile over 4G.

**Strategy:**
1. **Server-side render the initial snapshot into the HTML.** The first paint includes bird positions and the sky gradient without waiting for JS hydration.
2. **CDN edge caching** of the HTML shell (with a stale-while-revalidate strategy for the snapshot).
3. **Progressive enhancement:** the first frame renders from the embedded snapshot; interactive features hydrate as JS loads.
4. **Critical CSS inlined** in the HTML head.
5. **Preconnect** to API origin.

### 10.3 Runtime budgets

| Metric | Budget | Measurement |
|--------|--------|-------------|
| Idle motion frame rate | 60 fps | `requestAnimationFrame` timing |
| Frame time (p95) | <16.6 ms | Performance observer |
| Memory growth over 30 min | 0 MB | Heap snapshots in synthetic tests |
| Audio context errors | 0 | Error counter |
| Snapshot pull latency (p95) | <200 ms | RUM |

### 10.4 Observability

**Aggregate telemetry (allowed):**
- Request counts, latencies, error rates per endpoint.
- Simulation tick latency (p50, p95, p99).
- Client-side render frame timing (aggregated, no per-account dimension).
- Audio context error counts.
- Session duration histograms (anonymized).
- Bundle size on deploy.

**Per-account telemetry (forbidden):**
- Bird states, personality values, mood transitions.
- Interaction event contents.
- Notebook entry contents.
- Presence-time per account.

**Implementation:** Two separate telemetry pipelines. The aggregate pipeline feeds an operational metrics store (e.g., Prometheus). The simulation database is never read by the analytics pipeline. This separation is enforced at the infrastructure level (network policies, separate credentials), not just at the application level.

**Alerting:**
- Tick latency p99 > 5 seconds → alert.
- Snapshot endpoint p95 > 500 ms → alert.
- Error rate > 1% on any endpoint → alert.
- Memory growth detected in synthetic tests → alert.

---

## 11. Rollout

### 11.1 Phase 1: Internal alpha (weeks 1–4)

- Deploy to internal team.
- Two birds per aviary, no adoption of additional birds.
- Instrument drift calibration: measure drift trajectories across synthetic and real usage.
- Tune drift constants, mood transition weights, notebook entry cadence.
- Validate performance budgets on target device matrix.

### 11.2 Phase 2: Closed beta (weeks 5–8)

- Invite ~200 external users.
- Enable third-bird adoption (age-gated: aviaries >4 weeks old).
- Enable visit invitations.
- Monitor drift calibration across a broader population.
- A/B test notebook entry cadence (2-day vs. 3-day cooldown).
- Collect accessibility feedback from screen-reader users.

### 11.3 Phase 3: General availability (week 9+)

- Open sign-up.
- Ramp bird adoption schedule: third bird at ~2 months aviary age, fourth at ~4 months, up to seven at ~12 months.
- Enable full species pool (6 species).
- Continue monitoring drift calibration and adjusting constants.

### 11.4 Birds-per-aviary ramp

| Aviary age | Max birds |
|------------|-----------|
| 0–2 months | 2 |
| 2–4 months | 3 |
| 4–6 months | 4 |
| 6–9 months | 5 |
| 9–12 months | 6 |
| 12+ months | 7 |

The ramp is age-based, not engagement-based. This is deliberate: it refuses the gamification trap of "earn more birds by visiting more."

### 11.5 Day-one instrumentation

- Drift trajectory tracking (per-bird, per-trait, aggregate distributions).
- Mood transition frequency (per-mood, per-species).
- Notebook entry generation rate.
- Greeting variation (no two identical greetings in a row — verified by hashing greeting parameters).
- Audio synthesis error rate.
- Presence-signal validity (fraction of presence-pings passing all three conditions).

---

## 12. Risks

### 12.1 Drift calibration

**Risk:** Drift is too fast (birds change visibly between sessions, breaking the "slow current" feel) or too slow (users feel nothing they do matters).

**Mitigation:**
- Calibration harness that simulates weeks of synthetic user behavior.
- Internal alpha with daily drift-trajectory reviews.
- Drift constants are configuration, not code — adjustable without deploy.
- Instrumentation to detect population-level drift anomalies (e.g., median boldness exceeding expected range).

### 12.2 Sync correctness

**Risk:** A bug in the tick or event-ingestion path corrupts personality vectors (e.g., a negative delta slips through, a tick processes events out of order).

**Mitigation:**
- Personality vector writes are wrapped in a transaction with a CHECK constraint (all traits in [0, 1]).
- Event log is append-only with a monotonic `server_received_at` timestamp.
- Tick processing is idempotent (re-processing the same events produces the same result).
- Drift deltas are logged for auditability.
- Integration tests that simulate multi-device scenarios (laptop + phone posting events concurrently).

### 12.3 Audio uncanniness

**Risk:** Procedural calls sound synthetic, repetitive, or unpleasant. The "never identical twice" promise is not met, and users hear the same motif sequence repeated.

**Mitigation:**
- Motif library designed by a sound designer with avian audio reference.
- Variation parameters (pitch jitter, timing jitter, harmonic content) tuned in listening tests.
- Greeting variation verified by parameter hashing (no two consecutive greetings share the same parameter set).
- WebAudio fallback to silence + captions if synthesis produces errors.

### 12.4 Accessibility regressions

**Risk:** Screen-reader narration is too verbose, too sparse, or uses the wrong voice register. Reduced-motion mode is accidentally degraded by a rendering change.

**Mitigation:**
- Accessibility review at every PR that touches the rendering pipeline, narration generator, or DOM structure.
- Automated tests for ARIA live region updates (assert prose format, assert cadence).
- Reduced-motion mode tested in CI with `prefers-reduced-motion` media query emulation.
- Screen-reader user testing during closed beta.

### 12.5 Presence signal accuracy

**Risk:** The three-signal conjunction is too strict (users who are genuinely watching but not moving lose presence) or too loose (background tabs count as presence).

**Mitigation:**
- Activity window for pointer/key check is calibrated during build (leaning toward longer: 3–5 minutes).
- Instrumentation to measure presence-session duration distributions and detect anomalies.
- Client-side logging of which signal failed when presence is lost (for debugging, not telemetry).

### 12.6 Bundle size creep

**Risk:** Feature additions push the initial bundle past 2 MB, breaking the time-to-first-bird budget.

**Mitigation:**
- CI gate: build fails if initial bundle exceeds 2 MB gzipped.
- Code-splitting for all non-critical surfaces.
- Asset budget tracked per component.
- Progressive loading for bird pose assets.

### 12.7 Privacy boundary violation

**Risk:** A well-meaning engineer adds per-account telemetry to the aggregate pipeline, or a query joins the simulation DB with the analytics warehouse.

**Mitigation:**
- Network-level separation: simulation DB and analytics pipeline on different network segments.
- Credentials for the simulation DB are not available to the analytics service.
- Code review checklist item: "Does this change read per-account simulation state for aggregate purposes?"
- Annual audit of telemetry pipelines for privacy boundary compliance.

### 12.8 Notebook voice consistency

**Risk:** Template-based prose generation produces entries that feel repetitive or break voice (e.g., accidentally using announcement-style phrasing).

**Mitigation:**
- Template library is hand-curated and reviewed for voice compliance.
- No LLM-generated prose at v1 (too unpredictable for voice consistency).
- Internal alpha reviewers flag voice violations in notebook entries.
- Template expansion is tested against a voice-compliance checklist.

---

## 13. Implementation Order

The build is sequenced to de-risk the load-bearing systems first:

| Phase | Duration | Deliverables |
|-------|----------|-------------|
| **P0: Foundation** | Weeks 1–3 | Auth service, account DB, magic-link flow, session management. |
| **P1: Simulation core** | Weeks 2–5 | Tick scheduler, personality vector storage, drift function, mood state machine, event log. |
| **P2: Rendering** | Weeks 3–6 | Canvas 2D scene, bird sprites, perch zones, day/night cycle, idle motion. |
| **P3: Audio** | Weeks 4–7 | WebAudio synthesis, motif library, chorus mixing, listen-in mix. |
| **P4: Interactions** | Weeks 5–8 | Return-greeting, offer, settle, presence accounting, field notebook. |
| **P5: Sync & polish** | Weeks 7–9 | Multi-device sync validation, snapshot interpolation, SSE stream, loading sequence. |
| **P6: Accessibility** | Weeks 6–9 | Screen-reader narration, reduced-motion mode, captions, keyboard navigation. |
| **P7: Social** | Weeks 8–10 | Visit invitations, read-only visitor view, visit log, revocation. |
| **P8: Account management** | Weeks 9–10 | Export, deletion, email change, settings. |
| **P9: Hardening** | Weeks 10–12 | Performance optimization, CI gates, synthetic testing, drift calibration tuning. |

Phases overlap intentionally. The simulation core (P1) and rendering (P2) proceed in parallel because they communicate through the snapshot interface, which is defined early and stable.

---

## 14. Testing Strategy

### 14.1 Unit tests

- Drift function: given synthetic event logs, assert personality vector changes are within expected ranges.
- Mood state machine: given inputs, assert transition probabilities match specification.
- Presence accounting: given signal combinations, assert presence-event is recorded only when all three conditions hold.
- Notebook entry generation: given event patterns, assert entries are generated at correct cadence and in correct voice.

### 14.2 Integration tests

- Multi-device sync: two clients posting events concurrently, assert server state is consistent.
- Tick idempotency: re-process the same events, assert identical output.
- Visit flow: host invites, visitor views, host revokes, visitor sees revocation surface.
- Auth flow: magic-link request, consume, session revoke, expired link handling.

### 14.3 Synthetic tests

- Fleet of automated browsers running the aviary on a schedule from common geographies.
- Measure: time-to-first-bird, frame rate, memory growth, audio errors, snapshot latency.
- Run on every deploy and on a 6-hour schedule against production.

### 14.4 Drift calibration tests

- Simulate 90 days of synthetic user behavior (various visit patterns: daily, weekly, sporadic, absent).
- Assert: drift trajectories match calibration targets (measurable at 1 week, visible at 3 weeks).
- Assert: no negative drift on neglect.
- Assert: personality values remain in [0, 1].

### 14.5 Accessibility tests

- Screen-reader narration: assert ARIA live region receives updates at correct cadence, in correct voice.
- Reduced-motion: assert no frame-by-frame animation when `prefers-reduced-motion` is set.
- Keyboard navigation: assert all interactive surfaces are reachable via Tab/Arrow/Enter/Escape.
- Contrast: assert all text passes WCAG AA against its background.

---

## 15. Open Questions and Defensible Calls

1. **Activity window for presence:** Set to 3 minutes initially. This is a defensible middle ground — long enough that a user watching without moving retains presence, short enough that a user who walked away doesn't accumulate drift. Will be tuned during build based on instrumented data.

2. **Notebook entry cadence:** Minimum 2 days between entries per account. This preserves sparsity for active users. If entries feel too rare during alpha, the cooldown can be reduced to 1.5 days.

3. **Species pool size:** Six species at v1. This provides enough variety for a 7-bird aviary (with one species appearing twice) without diluting per-species design quality.

4. **SSE vs. WebSocket:** SSE chosen for simplicity. If bidirectional communication becomes necessary (e.g., real-time co-presence in a future version), WebSocket can be added alongside SSE without replacing it.

5. **Procedural bird visuals:** Bird sprites are hand-designed SVGs with procedural color mapping from `plumage_detail`. Fully procedural bird generation (e.g., generative art) is deferred to a future version — hand-designed assets are more predictable and easier to quality-check.

6. **Timezone handling:** Inferred from browser on first visit, stored in account settings, overridable. If the user travels, the aviary follows the stored timezone until they update it. This avoids the complexity of real-time timezone detection while keeping the day/night cycle meaningful.
