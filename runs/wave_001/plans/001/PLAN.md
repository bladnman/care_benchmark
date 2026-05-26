# Pocket Aviary — v1 Implementation Plan
Run 001 / Wave 001 — CARE Benchmark Phase 1

## 1. Scope

### 1.1 What is in v1

- **Single-user accounts**, email magic-link sign-in, per-device revocable session tokens.
- **One aviary per account**, seeded with two starter birds (from a pool of ~6 species), expandable to a hard cap of **7 birds**.
- **Server-side simulation tick** (~1/min) driving mood transitions, personality drift, ambient weather, and time-of-day state.
- **Real-time client rendering** of the aviary scene: day/night cycle, weather, perched birds with mood-shaped idle micro-motion, ambient leaf/feather drift.
- **Five core interactions**: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, and passive presence accounting.
- **Field notebook**: auto-generated, read-only naturalist prose entries, sparse (≈1 per few days), indefinite scrollback.
- **Visit invitations**: opt-in, per-invite, email-based, read-only ambient view for visitors, revocable, 30-day expiration.
- **Accessibility**: screen-reader live narration, reduced-motion cross-fade rendering, call captioning, full keyboard navigation, WCAG AA contrast on all chrome.
- **Multi-device sync** by architecture: single canonical server state, no client-to-client sync, no merge logic.

### 1.2 What is deliberately out of v1 (non-goals respected)

- No native mobile apps (iOS/Android).
- No gamification of any kind: no achievements, streaks, levels, scores, badges, XP, green-dot calendars, visit counters.
- No Tamagotchi mechanics: birds do not die, do not hunger, do not show distress, no happiness meter that decays.
- No social network surfaces: no profiles, follows, public discovery feed, friend-of-friend chains, mutual visits, comments, leaderboards, rankings, or "show-off" rendering mode for visitors.
- No push notifications, emails about aviary events, or "your friend visited!" alerts (visit log is silent by default; host may opt into notifications per-account).
- No payments, no paid tiers, no in-app purchases.
- No shared or household aviaries.
- No multi-aviary accounts.
- No customizable scenes, no drag-to-place perches, no user-arranged layout.
- No recorded audio fallback; WebAudio silence + captions is the fallback.

### 1.3 Scope ambiguities & defensible calls

| Ambiguity | Decision | Rationale |
|-----------|----------|-----------|
| Bird adoption pacing beyond "first few months" | New bird offers appear at fixed calendar intervals (e.g., 45 days, 90 days, 180 days, 365 days). No active-user threshold. | Per PRD, age—not attention—unlocks birds. Aligns with anti-gamification stance. |
| Exact mood enum size | **wary, content, curious, drowsy, alert, settled** (6 states). | Covers stated examples plus "settled" as an explicit evening/night state. |
| Presence pointer/key activity window | **3 minutes** (calibrated during build, leaning longer). | Matches PRD guidance: watching without moving is the product; window should not be so short that still observers lose presence. |
| Simulation tick cadence | **60 seconds** (exact target; ±5s jitter acceptable to avoid thundering herd). | PRD says "~once per minute." Build calibration narrows to 60s with jitter. |
| Weather frequency | **2–4 rain events per week**, **1–2 wind events per week**, each lasting 2–10 minutes. | "Rare" and "soft" per PRD; this pacing makes weather noticeable but never dominant. |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client (Browser)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Render     │  │  Audio      │  │  Interaction & Presence │  │
│  │  Pipeline   │  │  Engine     │  │  Collector              │  │
│  │  (Canvas/   │  │  (WebAudio) │  │  (visibility, focus,    │  │
│  │   WebGL)    │  │             │  │   pointer/key)          │  │
│  └──────┬──────┘  └──────┬──────┘  └───────────┬─────────────┘  │
│         │                │                       │               │
│         └────────────────┼───────────────────────┘               │
│                          │                                       │
│                    ┌─────┴─────┐                                 │
│                    │  State    │  (local snapshot + interpolate) │
│                    │  Cache    │                                 │
│                    └─────┬─────┘                                 │
│                          │                                       │
└──────────────────────────┼───────────────────────────────────────┘
                           │ HTTPS + SSE (or long-polling fallback)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway / Edge                        │
│         (CDN for static assets; edge cache for snapshots)        │
└──────────────────────────┬───────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐
│  Auth       │  │  Aviary     │  │  Simulation             │
│  Service    │  │  State      │  │  Tick Worker            │
│  (magic     │  │  Service    │  │  (event log → tick →     │
│   links)    │  │  (snapshots │  │   personality/mood       │
│             │  │   & events) │  │   update)               │
└─────────────┘  └──────┬──────┘  └─────────────────────────┘
                        │
           ┌────────────┼────────────┐
           ▼            ▼            ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Account    │  │  Aviary     │  │  Event      │
│  DB         │  │  State DB   │  │  Log DB     │
│  (emails,   │  │  (birds,    │  │  (append-   │
│   sessions) │  │   vectors,  │  │   only)     │
│             │  │   moods)    │  │             │
└─────────────┘  └─────────────┘  └─────────────┘
```

### 2.2 Client/server split

| Concern | Client | Server |
|---------|--------|--------|
| **Canonical state** | Never writes. Reads snapshots, interpolates. | Single writer via simulation tick. |
| **Personality vectors** | Never sees raw values. Renders derived behavior. | Stores, updates, protects. |
| **Mood** | Reads from snapshot, renders as idle motion / call timing. | Computes transitions per tick. |
| **Calls** | Synthesizes procedurally via WebAudio from motif grammar + personality. | Stores grammar seed, not audio. |
| **Presence** | Detects (visibility + focus + activity), sends pings. | Accumulates into presence-time, feeds drift. |
| **Interactions** | Emits events (listen-in start/end, offer, settle). | Appends to event log for tick consumption. |
| **Notebook entries** | Reads entries, renders scrollable list. | Generates prose server-side from simulation history. |
| **Time-of-day / weather** | Renders local-time cycle + weather overlay. | Computes ambient state per tick. |
| **Auth** | Collects email, consumes magic link, stores session token. | Issues/validates magic links, manages sessions. |
| **Telemetry** | Emits aggregate-only RUM (timings, errors). | Emits operational metrics (tick latency, request counts). |

### 2.3 Render pipeline boundary

The client render pipeline is entirely client-side. It does not stream animation frames from the server. The server emits **state snapshots** (bird positions, moods, current animation states, weather flags) at ~1/min (on pull) and the client interpolates between them for 60fps motion. This boundary keeps snapshots small (kilobytes) while allowing rich visual motion.

---

## 3. Data Model

### 3.1 Account

```json
{
  "account_id": "uuid-v4",           // synthetic, never derived from email
  "email_encrypted": "...",
  "created_at": "iso8601",
  "soft_deleted_at": null | "iso8601",
  "settings": {
    "reduced_motion": false,
    "call_captions": false,
    "visit_notifications": false
  }
}
```

### 3.2 Session

```json
{
  "session_id": "uuid-v4",
  "account_id": "uuid-v4",
  "device_description": "Safari / macOS",
  "created_at": "iso8601",
  "revoked_at": null | "iso8601"
}
```

### 3.3 Bird (canonical record, server-owned)

```json
{
  "bird_id": "uuid-v4",              // stable forever
  "account_id": "uuid-v4",
  "species_id": "string",            // one of ~6 species pool
  "name": "string",                  // user-editable
  "personality_vector": {
    "boldness": 0.0..1.0,
    "social_warmth": 0.0..1.0,
    "vocal_frequency": 0.0..1.0,
    "plumage_saturation": 0.0..1.0,
    "curiosity": 0.0..1.0
  },
  "mood": "wary | content | curious | drowsy | alert | settled",
  "perch_zone": "front | middle | back",
  "adopted_at": "iso8601"
}
```

**Hard rules:**
- Personality vector is never exposed to clients in raw form. API responses contain derived rendering hints (e.g., `motion_style`, `call_rate_moodifier`) computed server-side.
- Only the simulation tick writes `personality_vector` and `mood`.
- Bird identity (`bird_id`) is immutable across renames, syncs, and migrations.

### 3.4 Aviary state (snapshot)

```json
{
  "account_id": "uuid-v4",
  "snapshot_at": "iso8601",
  "local_time_hint": "HH:MM",        // client's timezone, used for day/night rendering
  "weather": null | { "type": "rain | wind", "intensity": 0.0..1.0, "remaining_seconds": int },
  "birds": [
    {
      "bird_id": "uuid-v4",
      "name": "Pip",
      "species_id": "...",
      "mood": "content",
      "perch_zone": "front",
      "render_hints": {
        "motion_style": "preen | scan | head_tilt | fluff | settle_low",
        "call_timing_next_ms": 4200,
        "plumage_color_hex": "#8B7E66",
        "plumage_detail_level": 0.72      // derived from plumage_saturation, 0–1
      }
    }
  ],
  "ambient": {
    "lighting_phase": "dawn | day | dusk | night",
    "ambient_call_volume": 0.0..1.0
  }
}
```

### 3.5 Event log (append-only)

```json
{
  "event_id": "uuid-v4",
  "account_id": "uuid-v4",
  "bird_id": null | "uuid-v4",
  "event_type": "presence_ping | listen_in_start | listen_in_end | offer | settle | tab_close | tab_visible",
  "payload": { ... },                // type-specific, e.g. { "offer_type": "seed" }
  "client_timestamp": "iso8601",
  "server_ingested_at": "iso8601"
}
```

Event log is the **only** thing clients write. The simulation tick consumes it in `server_ingested_at` order.

### 3.6 Presence window

Presence is not stored as a separate table; it is derived from contiguous `presence_ping` events. A presence window is considered open while pings arrive within the 3-minute activity window. The tick computes `presence_time_ms` per session from these windows.

### 3.7 Field notebook entries

```json
{
  "entry_id": "uuid-v4",
  "account_id": "uuid-v4",
  "written_at": "iso8601",
  "prose": "string",                 // naturalist, lowercase, present-tense
  "trigger_type": "drift_milestone | mood_notable | weather | offer_reaction | daily_observation"
}
```

Entries are generated server-side by the simulation tick or a notebook generation worker. They are read-only; no user edits.

### 3.8 Visits / Invitations

```json
{
  "invite_id": "uuid-v4",
  "host_account_id": "uuid-v4",
  "visitor_email_encrypted": "...",
  "status": "pending | active | revoked | expired",
  "created_at": "iso8601",
  "expires_at": "iso8601",
  "revoked_at": null | "iso8601"
}
```

Active visits pull the host's current snapshot read-only.

---

## 4. API Surface

### 4.1 Authentication

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /v1/auth/magic-link` | POST | Accepts `{ "email": "..." }`. Generates link, emails it. Rate-limited per email. |
| `GET /v1/auth/verify?token=...` | GET | Consumes magic link. Returns `{ "session_token": "..." }` and invalidates link. |
| `POST /v1/auth/revoke-session` | POST | Revokes a specific session by ID. Requires current session. |
| `GET /v1/auth/sessions` | GET | Lists active sessions for account. |

### 4.2 Aviary state (client pull, never push)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/aviary/snapshot` | GET | Returns current aviary state snapshot. Validates session token. |
| `POST /v1/aviary/event` | POST | Appends one interaction event to the log. Body: `{ "type": "...", "bird_id?": "...", "payload?": {...}, "client_timestamp": "..." }`. |
| `POST /v1/aviary/presence` | POST | Lightweight presence ping. Body: `{ "client_timestamp": "..." }`. |

**Snapshot fetch triggers:**
1. Initial page load.
2. `visibilitychange` → `visible`.
3. Render-frame gap > 60s (laptop sleep/resume detection).
4. Keepalive every 60s while tab is visible and focused.

### 4.3 Field notebook

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/notebook?before_id=...&limit=20` | GET | Paginated entries, newest first. |

### 4.4 Account settings

| Endpoint | Method | Description |
|----------|--------|-------------|
| `GET /v1/account` | GET | Returns settings, email (partially masked), created_at. |
| `PATCH /v1/account` | PATCH | Updates settings (reduced_motion, call_captions, visit_notifications). |
| `POST /v1/account/email` | POST | Initiates email change; verifies new address before commit. |
| `POST /v1/account/export` | POST | Queues JSON export; emails download link to verified address. |
| `POST /v1/account/delete` | POST | Initiates soft delete (30-day window). |
| `POST /v1/account/delete/cancel` | POST | Cancels pending soft delete. |

### 4.5 Visit invitations

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /v1/visits/invite` | POST | `{ "visitor_email": "..." }`. Creates pending invite, emails link. |
| `POST /v1/visits/revoke` | POST | `{ "invite_id": "..." }`. Immediate revocation. |
| `GET /v1/visits/log` | GET | Host's visit log: invites + visits. |
| `GET /v1/visits/guest?token=...` | GET | Visitor link. Returns read-only snapshot of host aviary. |

### 4.6 Bird management (minimal)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `PATCH /v1/birds/:bird_id` | PATCH | Rename only. No other mutable fields exposed. |

---

## 5. Simulation Engine Design

### 5.1 Tick architecture

A **stateless worker** (containerized, horizontally scalable) polls the event log for accounts with unprocessed events, computes the next tick for each, and writes the updated aviary state. Each tick is idempotent: given the same previous state and event batch, it produces the same next state.

**Tick cadence:** 60 seconds ± 5s jitter per account (shard by `account_id` hash to distribute load).

**Tick inputs:**
1. Previous canonical aviary state (birds, moods, vectors, weather timers).
2. Unprocessed event log entries since last tick.
3. Current actual time (UTC).
4. User's timezone (stored on account, used for time-of-day mood nudges).

**Tick outputs:**
1. Updated aviary state (new moods, updated vectors, new perch zones, weather state).
2. Zero or one notebook entry.
3. Mark events as processed.

### 5.2 Drift function

Drift is a **low-pass filter** over presence and interaction signals, producing additive deltas to personality vectors.

**Calibration target (testable):**
- After ~1 week of regular visits (defined as 20+ minutes of presence on 5+ days): instruments detect ≥0.05 change in at least one trait.
- After ~3 weeks of regular visits: a returning user can feel a difference (e.g., the bolder bird greets more readily, plumage appears richer) without being told.

**Drift formula (per trait, per tick):**

```
delta_trait = SUM(inputs_weighted) * decay_factor * global_drift_rate
```

Where:
- `inputs_weighted`: presence_time_ms (dominant), listen-in duration (moderate), offer acceptance (small), near-offer proximity (tiny).
- `decay_factor`: decreases as trait approaches 1.0 (soft ceiling, never hard-capped).
- `global_drift_rate`: tuned so the 1-week / 3-week targets are met on typical usage.

**Monotonic rule:** deltas are always ≥ 0. Neglect produces zero delta; traits never decrease. A bird that is ignored stays at its current vector values and becomes "ambient" through mood (fewer greetings) and lower accumulated presence-time, not through negative drift.

### 5.3 Mood transitions

Mood is an enumerated state machine with probabilistic transitions.

**Transition weights (per tick, per bird):**

| Current | Wary | Content | Curious | Drowsy | Alert | Settled |
|---------|------|---------|---------|--------|-------|---------|
| Wary | — | med | low | low | med | low |
| Content | low | — | med | med | low | med |
| Curious | low | med | — | low | high | low |
| Drowsy | low | med | low | — | low | high |
| Alert | med | low | high | low | — | low |
| Settled | low | med | low | high | low | — |

Modifiers:
- **Time of day:** morning → +alert; dusk → +drowsy; night → +settled.
- **Weather:** rain → +drowsy, −vocal frequency; wind → +alert, +wary.
- **Recent interaction:** offer accepted → +content; listen-in start → +curious; settle → +settled.
- **Personality:** high boldness → −wary weight; high curiosity → +curious weight.
- **Bird-to-bird:** if another bird is wary, nearby birds (same perch zone) get +wary weight.

**Mood persistence:** mood from the last tick is the starting mood for the next tick. No reset on session open.

### 5.4 Call-grammar runtime

Each species has a **motif library** (small set of pitch/timing motifs). A call is assembled at runtime by:

1. Selecting a motif sequence (2–5 motifs).
2. Applying personality-shaped variation: vocal frequency shifts timing (higher = shorter gaps between calls); boldness shifts volume; mood shifts pitch register (drowsy = lower, slower).
3. Rendering via WebAudio on the client using the motif library + variation parameters delivered in the snapshot.

**Call timing:** each bird has a `call_timing_next_ms` hint in the snapshot. The client schedules the next call when the timer expires, unless listen-in is active (which may accelerate the focused bird and suppress others).

**Chorus:** when two or more birds schedule calls within a short window (±2s), the client mixes them with slight panning and volume variation. The mix is real-time; no pre-baked chorus loops.

### 5.5 Perch selection

Perch zone is chosen per tick based on:
- Current mood (wary → back; content → middle; curious → front; drowsy → low middle; settled → low anywhere).
- Boldness (high boldness overrides mood toward front).
- Recent interaction (bird that was offered-to recently → front for a few ticks).

Transitions between perches are animated by the client; the server only emits target zones.

### 5.6 Weather generation

Weather is computed per tick with a low-probability roll:
- Rain: 3% chance per tick (≈2.6 events/week), duration 2–10 min.
- Wind: 1.5% chance per tick (≈1.5 events/week), duration 2–8 min.

Only one weather type active at a time. Weather state is included in the snapshot.

---

## 6. Sync Model

### 6.1 Canonical state principle

The server holds the **only** canonical copy of personality vectors, moods, and aviary state. Clients are thin renderers. There is no sync problem because there is no client state to merge.

### 6.2 Client behavior

1. **On load:** pull snapshot, begin rendering immediately.
2. **On visibility change to visible:** pull fresh snapshot (the aviary has been ticking while away).
3. **On long frame gap (>60s):** pull fresh snapshot (sleep/resume).
4. **While visible:** keepalive ping + snapshot pull every 60s.
5. **On interaction:** emit event to server immediately (fire-and-forget, with client-side retry on 5xx/network failure).
6. **On background/hidden:** stop rendering, stop presence pings, stop snapshot pulls. Simulation continues server-side.

### 6.3 Conflict prevention

- **Personality state:** additive server-authored deltas only. Clients never send absolute vector values. The event log is append-only and processed in order.
- **Mood:** server-computed; clients read only.
- **Bird names:** PATCH is last-write-wins, but names are user-facing cosmetic metadata with no simulation impact. A rename conflict is acceptable as last-write-wins.
- **Account settings:** PATCH is last-write-wins; settings are small and independent.
- **Notebook entries:** append-only, server-generated; no client writes.

### 6.4 Multi-device coherence

Because both devices read the same snapshot source, divergence is limited to:
- **Render interpolation differences:** device A and B may interpolate motion slightly differently between snapshots; this is acceptable and unnoticeable.
- **Event emission timing:** a listen-in on device A and an offer on device B within the same tick window may both be processed in the next tick; order is determined by `server_ingested_at`, not client timestamp.

### 6.5 Failure modes

| Scenario | Behavior |
|----------|----------|
| Server outage during session | Client renders from last snapshot, queues events locally (IndexedDB), retries with exponential backoff. Notebook not writable during outage. |
| Client offline > tick period | On reconnect, client pulls fresh snapshot. The aviary has advanced server-side; no catch-up simulation on client. |
| Simultaneous sessions on two devices | Both emit events to the same log. Tick processes in ingest order. No session locking required because events are additive (presence-time, listen-in duration). |
| Magic link replay / session timeout | Matter-of-fact error surface: "Your session timed out. Sign in again to keep watching." |

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene composition

The aviary is a single full-viewport canvas (2D Canvas API preferred for v1; WebGL reserved for future if needed). The scene layers are, back to front:

1. **Sky / background foliage** — gradient + SVG shapes, tinted by lighting phase.
2. **Back perch zone** — birds perched here render smaller, slightly desaturated.
3. **Middle perch zone** — default scale.
4. **Front perch zone** — slight scale-up, subtle shadow.
5. **Foreground branches / leaves** — occasional drift-through elements.
6. **Ambient particles** — leaves, feathers (client-generated, no server state).
7. **Lighting overlay** — global color grade for dawn/dusk/night.
8. **Weather overlay** — rain streaks or wind-ripple shader, subtle.
9. **Top bar chrome** — HTML overlay, not canvas, for accessibility and interaction.

### 7.2 First frame discipline

The first frame the user sees **must** contain birds in mid-action. Implementation:

1. HTML shell delivered from CDN edge includes a minimal inline script that draws a **quiet field** (soft sky gradient + faint ambient motion) immediately.
2. Parallel fetch of JS bundle + initial snapshot.
3. Once snapshot arrives, birds are placed at their current positions with their current motion states. No fade-in, no "wake up" sequence. The motion continues as if it had always been running.
4. If snapshot fetch is slow (>500ms), the quiet field remains; it is the loading state. No spinner.

### 7.3 Idle micro-motion

Each bird runs a continuous idle animation chosen from its mood:

| Mood | Idle motion |
|------|-------------|
| Wary | Scan (head turning side to side, 3–5s cycle), perch further back. |
| Content | Preen (beak-to-feather motions, 4–8s cycle), occasional fluff. |
| Curious | Head-tilt toward sounds, alert posture, slight forward lean. |
| Drowsy | Low perch, feathers fluffed, eyes half-closed, minimal movement. |
| Alert | Upright, quick head snaps, tail flick. |
| Settled | Eyes closed, very low movement, occasional breath/shift. |

Motion is **procedural**, not pre-authored animation cycles. Each bird's motion is seeded by its personality vector (e.g., high boldness = more forward-leaning scan). The client interpolates between key poses at 60fps.

### 7.4 Transitions

- **Perch change:** smooth bezier-curve flight path over 1–2s, with wing-flap animation.
- **Mood change:** gradual blend over 2–3s from old idle motion to new.
- **Lighting:** continuous gradient shift, no hard cuts.
- **Listen-in engage:** 2s volume ramp on focused bird, 2s ambient dim on others.
- **Settle:** 4–5s lighting shift to evening palette, calls quiet over 3s. Undoable within 5s by any click.

### 7.5 Reduced-motion mode

When `prefers-reduced-motion` is true or user opts in:

- Frame-by-frame animation is replaced by **slow cross-fades between still poses** (1–2s fades).
- Flight transitions become cross-fades between perches.
- Ambient leaf/feather drift is removed.
- Day/night color shifts remain but are slowed (10s transitions).
- Calls still play (or caption) at full quality.

This is a **designed surface**, not a fallback. The cross-fade aesthetic is intentionally calm.

### 7.6 Responsive layout

Scene scales to fit viewport while preserving aspect ratio that keeps all birds visible. On narrow viewports, perch zones compress horizontally; on wide viewports, they spread. Birds never crop offscreen. Minimum viewport: 320px wide.

---

## 8. Audio Pipeline

### 8.1 Procedural call synthesis

Each species ships with a **motif library** encoded as a compact data structure (JSON or binary blob, <20KB per species). A motif defines:

- Base pitch (MIDI note or frequency)
- Duration envelope (attack, sustain, decay in ms)
- Timbre parameters (harmonic richness, noise component)
- Timing gap to next motif

The client WebAudio graph:

```
Motif Library → Motif Sequencer → Pitch/Envelope/Timbre Variation
                                      ↓
                              Oscillator + GainNode
                                      ↓
                           Optional: Filter (mood shapes warmth)
                                      ↓
                              Master Mix Bus
                                      ↓
                           Listen-in Rebalance + Ambient Chorus
                                      ↓
                               Destination
```

**Variation from personality:**
- Vocal frequency → scales inter-motif gaps (higher = shorter gaps).
- Mood → shifts pitch register (drowsy: −2 semitones, slower attack; alert: +1 semitone, sharper attack).
- Boldness → scales output gain slightly (higher = louder).

### 8.2 Chorus mixing

When multiple birds call simultaneously:
- Each call is rendered on its own audio graph branch.
- Panning is subtle (no hard-left/right; birds are not that far apart).
- Master mix applies a gentle compressor to prevent clipping during chorus events.
- The mix is additive: two calls at once are two distinct sounds, not a layered loop.

### 8.3 Listen-in mix

- On engage: focused bird's gain ramps from ambient level to +6dB over 2s. Other birds fade to −12dB (never silent). Ambient bus remains audible at low level.
- On disengage: reverse ramp over 2s.
- If another listen-in is triggered during an active one, the old target fades down while the new fades up (cross-fade, 1.5s).

### 8.4 WebAudio fallback

If `AudioContext` is unavailable or permission is denied:
- Audio is silenced.
- Call captions are enabled by default (if not already on).
- The aviary remains fully functional; silence is preferable to canned audio.

### 8.5 Bundle and runtime constraints

- Motif libraries for all 6 species must fit within the 2MB JS bundle budget. Procedural synthesis is required for this.
- No memory growth: audio buffers for motifs are pooled and reused. A new `AudioBuffer` is not allocated per call.
- The audio context is lazily initialized on first user gesture (browser autoplay policy) and kept alive for the session.

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

A **live region** (ARIA `aria-live="polite"`) in the top-level DOM receives prose updates.

- **Cadence:** one update every 30–60s at idle. Faster on user-initiated events (return-greeting, offer reaction, settle).
- **Content:** naturalist prose, same voice as the notebook. Example:
  > "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- **Generation:** prose is composed client-side from the current snapshot + a small template/grammar system, or pulled from the server if the notebook generation service exposes a "narration snippet" endpoint. For v1, client-side composition is preferred to avoid extra latency.
- **Queue management:** updates are dropped (not queued) if the screen reader is still reading the previous update. This prevents backlog buildup.

### 9.2 Call captions

- Enabled via accessibility settings.
- When a bird calls, a small text caption appears near the bird (positioned via absolute overlay, not canvas text) and fades out after the call ends.
- Captions are generated from the motif sequence that was actually played, not pre-written strings. Example: "a soft three-note rise" or "a low trill, paused, low trill again."
- Caption text uses the naturalist voice, lowercase.

### 9.3 Keyboard navigation

| Key | Action |
|-----|--------|
| Tab | Cycles through top-bar items, then enters aviary scene. |
| Enter / Space | Activates focused top-bar item or triggers listen-in on focused bird. |
| Arrow keys | Move focus between birds in the scene (left/right for perch order, up/down for zone). |
| Escape | Exits listen-in, closes any open panel (notebook, offer menu). |
| O | Opens offer affordance (top-bar shortcut). |
| S | Triggers settle gesture. |

Focus indicator: 2px solid outline with 4px radius, color `#FFFFFF` at 80% opacity with a 1px dark shadow to ensure visibility against all aviary backgrounds.

### 9.4 Reduced-motion mode

See §7.5. This is a first-class rendering path, not a degraded fallback. It ships with v1, not post-launch.

### 9.5 Contrast

- All user-copy text in top bar, settings, captions, and error surfaces meets WCAG AA (4.5:1 for normal text, 3:1 for large text).
- Aviary scene contains no user-copy text, so contrast requirements apply only to chrome.

---

## 10. Performance Budgets and Observability

### 10.1 Budgets

| Budget | Target | Enforcement |
|--------|--------|-------------|
| Initial JS bundle (gzipped) | < 2 MB | CI build fails if exceeded. Code-split aggressively for settings, notebook, visit-invitation flows. |
| Time to first bird visible | < 500ms (mid-tier mobile, 4G) | Synthetic monitoring from 3 geographies; P95 alarm at >600ms. |
| Idle motion frame rate | 60 fps | 5-year-old laptop, 30-min session. CI performance test runs on reference hardware. |
| Memory growth over 30 min | 0 MB | Heap snapshot diff test in CI; fails if >0MB growth. |
| Snapshot payload size | < 5 KB uncompressed | Instrumented; alarm if mean >3KB. |
| Audio context init latency | < 100ms | Measured on first user gesture; alarm if p99 >200ms. |

### 10.2 Bundle composition strategy

- **Core (~1.2 MB):** framework, render engine, audio synthesis core, 2 starter species motif libraries.
- **Lazy (~0.6 MB):** remaining 4 species libraries, account settings UI, visit invitation UI, notebook rich-text rendering.
- **Assets:** bird visual assets are procedural SVGs or compact procedural drawing routines (no large bitmap atlases). Background foliage is procedural.

### 10.3 Observability

**Synthetic checks (automated browsers):**
- Load aviary from 3 geographies every 5 minutes.
- Measure: TTFB, first-bird-render time, frame rate over 60s, audio context init success rate.
- Alert: P95 > budget thresholds.

**Real User Monitoring (aggregate only, no per-account dimensions):**
- Page load timings, first-bird-render timings.
- Render-frame timings (client-side `requestAnimationFrame` instrumentation).
- Audio-context error counts.
- Simulation-tick latency (server-side, anonymized).

**Server operational metrics:**
- Tick latency p50/p99 (alarm if p99 > 5s).
- Event ingest rate, event log backlog depth.
- Snapshot API request rate, p99 latency.
- DB connection pool saturation.

**Privacy boundary:** RUM and operational metrics contain **zero per-bird state, zero per-account interaction history, zero personality vector values**. Aggregate session-duration histograms are binned and anonymized before emission. Telemetry pipeline does not read from the simulation database.

### 10.4 What we deliberately do NOT measure

- Per-bird drift rates for dashboards.
- Per-account "engagement scores."
- Individual call patterns or offer preferences for analytics.
- A/B test instrumentation that would split the user base into gamified vs. non-gamified variants.

---

## 11. Rollout

### 11.1 v1 shipping plan

| Phase | Audience | Duration | Goal |
|-------|----------|----------|------|
| Alpha | Internal team + friends | 2 weeks | Validate tick calibration, audio mix on real devices, catch first accessibility regressions. |
| Closed beta | 500 waitlist users | 4 weeks | Measure drift at real-world scale, tune presence window, fix sync edge cases, validate 500ms first-bird budget on diverse networks. |
| Open beta | Public, invite-only link | 4 weeks | Load test tick worker scaling, validate 7-bird cap audio recognizability, monitor memory growth telemetry. |
| General availability | Public | — | Full release. |

### 11.2 Birds-per-aviary ramp

- Alpha & closed beta: cap at **3 birds** (one unlock at ~2 weeks of account age).
- Open beta: cap at **5 birds**.
- GA: cap at **7 birds**.

This staged ramp lets us validate audio recognizability and rendering performance at each density before the full cap.

### 11.3 Instrumentation from day one

- Synthetic monitoring deployed before first alpha user.
- RUM enabled in all phases (aggregate-only, privacy-respecting).
- Error alerting (simulation-tick p99 latency, API 5xx rate) active before alpha.
- Accessibility audit (automated + manual screen-reader pass) before closed beta.

### 11.4 Success metrics (for v1, not user-facing)

- 7-day retention (observed, not surfaced to users).
- Session duration distribution (median, p95).
- First-bird-render time p95.
- Simulation-tick latency p99.
- Audio context init success rate.
- Accessibility: screen-reader task-completion rate in manual testing.

---

## 12. Risks

### 12.1 Drift calibration risk (HIGH)

**What could go wrong:** The 1-week instrument-visible / 3-week user-visible calibration is extremely sensitive. A drift function that's too fast makes the product feel like a Tamagotchi; too slow makes it feel like a wallpaper. There is no industry benchmark for this.

**Mitigation:**
- Build an internal "fast-forward" test harness that simulates 30 days of usage in minutes.
- Instrument all five traits numerically from alpha; require measurable change after 1 week of simulated regular use before beta.
- Schedule a dedicated "drift review" checkpoint between closed and open beta with real user qualitative feedback.
- Make `global_drift_rate` a server-side tunable (feature flag) so we can adjust without client deploy.

### 12.2 Sync correctness risk (HIGH)

**What could go wrong:** A bug in event log ordering, tick idempotency, or additive delta application could silently corrupt personality vectors. The failure is invisible: no user-visible error, just a bird that "feels wrong" after a few weeks. By the time it's reported, reproducing the corruption path is nearly impossible.

**Mitigation:**
- Event log is append-only with monotonic `server_ingested_at`; no updates, no deletes.
- Tick worker logs every delta applied (recorded in a separate audit log, not the telemetry pipeline).
- Periodic "vector integrity check" job compares recomputed drift from event log against stored vector (weekly, sampled accounts). Alert on mismatch.
- Soft-launch with 500 users gives us a small enough population to investigate reports deeply.

### 12.3 Audio uncanniness risk (MEDIUM-HIGH)

**What could go wrong:** Procedural calls that vary too little sound like loops; calls that vary too much lose per-bird recognizability. The chorus mechanic may produce phase-cancellation artifacts or "muddy" mixes at 5–7 birds. WebAudio performance may degrade on mid-tier mobile devices.

**Mitigation:**
- Audio engineer owns the motif library and variation bounds; code review gates any change to the synthesis graph.
- Per-bird recognizability is tested with blind listening tests at each birds-per-aviary ramp stage.
- Mobile audio performance is tested on reference devices (3-year-old Android, 2-year-old iPhone) in synthetic checks.
- Graceful degradation: if audio context drops frames, reduce active voice count (oldest calls finish, new calls delayed) rather than glitching.

### 12.4 Accessibility regression risk (MEDIUM)

**What could go wrong:** The screen-reader narration system is easy to de-prioritize or simplify into state-list announcements. Reduced-motion mode may lag behind visual changes and feel broken. Keyboard navigation may break when new interactions are added.

**Mitigation:**
- Accessibility is a launch blocker, not a v1.1 feature.
- Automated a11y tests (axe-core) run in CI on every build.
- Manual screen-reader pass (VoiceOver + NVDA) is required before closed beta.
- Reduced-motion and keyboard paths are tested in the same synthetic browser fleet as performance.

### 12.5 "Feels alive" death-by-a-thousand-cuts risk (MEDIUM)

**What could go wrong:** The central conceit is fragile. One spinner, one canned greeting, one "Welcome back!" toast, one achievement pop-up, one hard-cut audio transition, and the product collapses from "place" to "app." These leaks are easy to introduce in code review because each one looks harmless in isolation.

**Mitigation:**
- Code review checklist includes: "No spinners in aviary loading path. No toast greetings. No canned animations. No gamification language."
- Product review sign-off required for any UI surface that appears within the aviary viewport or during session start.
- Automated lint rule flags strings like "Welcome back," "achievement," "streak," "level up," "score" in source code.
- Design system separates "naturalist" voice components from "matter-of-fact" components; any new surface must declare which register it belongs to.

### 12.6 Performance degradation at scale risk (MEDIUM)

**What could go wrong:** The simulation tick, while stateless, must sequentially process events per account. If an account generates many events (e.g., rapid listen-in toggling), tick latency for that shard may spike. At scale, tick workers may fall behind the event log.

**Mitigation:**
- Event log is sharded by `account_id`; each worker claims a shard.
- Tick latency is alarmed at p99 > 5s.
- If backlog grows, workers can "batch-tick" — process multiple minutes of events in one computation for an account, applying the net delta. This is a safety valve, not normal operation.
- Simulation DB is provisioned with read replicas for snapshot serving, separate from the tick writer.

### 12.7 Privacy boundary erosion risk (LOW-MEDIUM)

**What could go wrong:** An engineer adds a "helpful" analytics event that includes `bird_id` or `mood` to a telemetry pipeline. A data scientist requests per-account drift aggregation. The boundary is architectural but depends on human discipline.

**Mitigation:**
- Telemetry schemas are defined in a separate repo from simulation schemas; any field addition requires PR review and privacy sign-off.
- Simulation DB credentials are not available to the analytics warehouse service account.
- Annual privacy audit includes telemetry schema review.
- The lint rule that flags gamification should also flag per-account fields in telemetry definitions.

---

## Appendix A: PRD Cross-Reference

| Plan Section | PRD Files |
|--------------|-----------|
| Scope, voice, non-goals | `product_brief.md`, `non_goals.md` |
| Data model (birds, vectors, mood, presence) | `concepts.md`, `bird_engine.md` |
| Interactions (return-greeting, listen-in, offer, settle, notebook) | `interactions.md` |
| Scene rendering, day/night, weather, top bar | `aviary_layout.md` |
| Auth, sync, tick, privacy | `accounts_sync.md` |
| Visit invitations | `social_optional.md` |
| Accessibility, performance budgets, audio constraints | `accessibility_perf.md` |

---

*End of plan. Implementation not included — this is a phase-1 planning artifact only.*
