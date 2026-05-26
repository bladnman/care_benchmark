# Implementation Plan — Pocket Aviary v1

## 1. Scope

### In v1

- **Two starter birds per new account**, assigned from a pool of six species. Species assignment is deterministic-random seeded by account creation timestamp; the user does not pick from a catalog.
- **Aviary grows to a max of seven birds.** New species offers appear on a time-gated cadence (aviary age, not visit count). Adoption flow shows the arriving bird entering the scene.
- **Single-user accounts**, email magic-link auth, synthetic account UUID. Email change requires new-address verification.
- **Server-side simulation tick** (~1/minute) advances personality vectors, mood, mood timers, and ambient events. Tick runs regardless of client connectivity.
- **Client-side rendering**: single horizontal scene, three perch zones, day/night cycle anchored to browser local time, ambient weather (rare rain, soft wind), leaf/feather drift.
- **Interaction surfaces**: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only), and keyboard navigation.
- **Multi-device sync** as a property of single-canonical-server-state architecture. No client-to-client sync.
- **Read-only visit invitations**: host emails a one-time link; visitor sees the aviary as the host sees it. No co-presence, no chat, no comments, no visitor-driven drift. Invites revocable, expire after 30 days.
- **First-class accessibility**: naturalist screen-reader narration, reduced-motion mode (cross-fade rendering), procedural call captions, WCAG AA contrast on all user-copy surfaces, full keyboard navigation.
- **Account management**: export JSON snapshot of aviary state, soft-delete with 30-day recovery window, session token management and revocation.
- **Telemetry**: aggregate operational only (request counts, latencies, error rates, session-duration histograms), strictly bounded from per-bird/per-account interaction state.

### Explicitly out of scope

- Native mobile apps (web only).
- Any gamification surface: no streaks, scores, achievements, badges, levels, calendars, visit-frequency counters.
- Tamagotchi-style custodial mechanics: no hunger, no distress, no death, no decay-on-neglect. Drift is monotonic toward expressive.
- Social network surfaces: no profiles, no follows, no public discovery, no leaderboards, no comments, no friend-of-friend chains.
- Co-presence during visits (visitor and host never see each other in the aviary).
- Multi-aviary accounts, shared aviaries, paid tiers, push notifications.
- Recorded audio files for bird calls (procedural WebAudio synthesis only).

---

## 2. Architecture

### Service topology

```
┌───────────────┐     ┌─────────────────┐     ┌─────────────────────┐
│  Static CDN   │     │   API Gateway    │     │  Simulation Service │
│  (HTML/JS/    │     │  (auth, account, │     │  (tick runner,      │
│   assets)     │     │   state pull,    │     │   drift compute,    │
│               │     │   event append)  │     │   notebook gen)     │
└──────┬────────┘     └────────┬────────┘     └──────────┬──────────┘
       │                       │                         │
       │                       └──────────┬──────────────┘
       │                                  │
       ▼                                  ▼
 ┌─────────────────────────────────────────────────────────┐
 │                    PostgreSQL (primary)                  │
 │  accounts, birds, personality_vectors, moods, events,   │
 │       notebook_entries, visit_invites, visit_log         │
 └─────────────────────────────────────────────────────────┘
```

### Component responsibilities

| Component | Responsibility |
|-----------|---------------|
| **Static CDN** | Delivers the HTML shell, JS bundle, CSS, SVG bird assets, WebAssembly audio modules. No dynamic content. |
| **API Gateway** | Handles magic-link auth flow, session token issuance/validation, routes state-pull and event-append requests, enforces rate limits, writes interaction events to the event log. |
| **Simulation Service** | Runs the per-account simulation tick on a cron-like schedule. Reads recent events from the event log, computes drift deltas, transitions moods, advances mood timers, applies ambient weather, generates notebook entries, writes canonical state back to the database. |
| **PostgreSQL** | Single primary database. Tables are partitioned by account UUID for horizontal scaling readiness (not required at launch but designed from day one). |

### Why one database, not microservice-database-per-service

The simulation tick reads events, computes drift, and writes personality + mood + notebook all in one short transaction per account. Splitting those across services would introduce distributed-transaction coordination for a single-account operation that must be serial. The simplicity of a single relational store for a product this small outweighs the theoretical scaling benefits of separation. If the simulation tick becomes a bottleneck, we partition by account UUID and run parallel tick workers with no shared state; we do not split the schema across services.

### Language and runtime choices

| Concern | Technology | Rationale |
|---------|------------|-----------|
| **API Gateway** | TypeScript + Node.js (or Bun) on a lightweight HTTP framework (e.g., Hono) | Matches frontend language for shared types; the gateway is thin, routing auth + event append + state pull. |
| **Simulation Service** | TypeScript + Node.js, running as a scheduled worker (cron trigger or persistent loop with per-account scheduling) | Shared type definitions with API layer; simulation math is floating-point and state-machine logic, not CPU-bound. |
| **Database** | PostgreSQL 16+ | Relational with strong JSON support for personality vectors, mature partitioning, and row-level security not needed (the service layer gates access). |
| **Frontend** | TypeScript + a lightweight reactive framework (Preact or Solid) bundling to ES modules | Bundle size budget (2MB gzipped) rules out larger frameworks; WebAudio API accessed directly, no audio library dependency. |
| **Email delivery** | Transactional email service (e.g., Resend, Postmark) for magic links and export download links. | |

### Key architectural invariants

1. **The server is the only writer of personality state.** Clients append interaction events to an event log. Only the simulation tick computes drift and mutates personality vectors.
2. **The client renders snapshots, never owns state.** On tab open, the client pulls a state snapshot and interpolates between successive snapshots for smooth motion. Personality state is never cached client-side beyond the current rendering frame.
3. **No client-to-client communication.** Multi-device "sync" is not sync — it is two clients independently pulling the same canonical server state.
4. **The simulation tick runs independent of any client session.** Tick cadence is ~1/minute per account. A ticking-since-last-snapshot counter allows the server to fast-forward when a long-dormant account receives its next pull.

---

## 3. Data Model

### 3.1 Core tables

#### `accounts`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Synthetic, generated at account creation. Never derived from email. |
| `email` | TEXT (encrypted at rest) | Stored once, encrypted. Never used as a join key or identifier anywhere else. |
| `email_verified` | BOOLEAN | |
| `pending_email` | TEXT (encrypted) | Set during email-change flow before new address is verified. |
| `created_at` | TIMESTAMPTZ | |
| `deleted_at` | TIMESTAMPTZ | Soft-delete marker. NULL if active. Hard-deleted after 30 days past this timestamp. |
| `timezone` | TEXT | IANA timezone string, detected from browser on first sign-in, adjustable in settings. Drives day/night cycle for the aviary. |
| `settings` | JSONB | User-settable preferences: reduced_motion, call_captions_enabled, visit_notifications_enabled. |

#### `birds`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Stable internal identifier. Never regenerated, never reassigned. |
| `account_id` | UUID (FK → accounts.id) | |
| `species_id` | TEXT | e.g., `"warbler"`, `"finch"`, `"nightjar"`. References the species pool definition. |
| `name` | TEXT | User-assigned, renameable at any time. |
| `adopted_at` | TIMESTAMPTZ | |
| `display_order` | SMALLINT | Stable sort order for rendering (1–7). |

#### `personality_vectors`
| Column | Type | Notes |
|--------|------|-------|
| `bird_id` | UUID (PK, FK → birds.id) | One row per bird; joined directly to bird. |
| `boldness` | REAL | Range [0.0, 1.0]. Seed value for new birds: species-dependent default (e.g., 0.30–0.50). |
| `social_warmth` | REAL | Range [0.0, 1.0]. |
| `vocal_frequency` | REAL | Range [0.0, 1.0]. |
| `plumage_saturation` | REAL | Range [0.0, 1.0]. Increases monotonically. Never decreases. |
| `curiosity` | REAL | Range [0.0, 1.0]. |
| `updated_at` | TIMESTAMPTZ | Last tick that modified this row. |

#### `moods`
| Column | Type | Notes |
|--------|------|-------|
| `bird_id` | UUID (PK, FK → birds.id) | One row per bird. |
| `current_mood` | TEXT | Enum: `wary`, `content`, `curious`, `drowsy`, `alert`. |
| `mood_entered_at` | TIMESTAMPTZ | When the bird entered the current mood. Used for duration-based auto-transitions. |
| `last_transition_reason` | TEXT | For debugging: `time_of_day`, `interaction`, `ambient_event`, `personality_bias`, `bird_to_bird`. |

#### `interaction_events`
| Column | Type | Notes |
|--------|------|-------|
| `id` | BIGSERIAL (PK) | Monotonic per-account ordering. |
| `account_id` | UUID (FK → accounts.id) | Partition key. |
| `event_type` | TEXT | One of: `presence_start`, `presence_ping`, `presence_end`, `offer`, `listen_in_start`, `listen_in_end`, `settle`, `visit_start`, `visit_end`. |
| `bird_id` | UUID | Nullable; present for bird-specific events. |
| `payload` | JSONB | Event-specific data: offer type, listen-in duration, presence window ms, etc. |
| `client_timestamp` | TIMESTAMPTZ | When the client recorded the event (for ordering within a session). |
| `server_timestamp` | TIMESTAMPTZ | When the server received it (for deduplication and tick ordering). |

Index: `(account_id, id)` for tick consumption in order. Partition by `account_id` hash for horizontal scaling.

#### `notebook_entries`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | |
| `account_id` | UUID (FK → accounts.id) | |
| `entry_text` | TEXT | Naturalist prose, generated by the simulation tick. |
| `entry_tags` | TEXT[] | Optional tags for internal use: `greeting`, `weather`, `chorus`, `quiet_morning`, etc. |
| `created_at` | TIMESTAMPTZ | |

#### `visit_invitations`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | |
| `host_account_id` | UUID (FK → accounts.id) | |
| `visitor_email` | TEXT | The invited email address. |
| `invite_token` | TEXT (unique) | Opaque token embedded in the magic link. |
| `status` | TEXT | `pending`, `active`, `revoked`, `expired`. |
| `created_at` | TIMESTAMPTZ | |
| `expires_at` | TIMESTAMPTZ | 30 days from creation. |
| `revoked_at` | TIMESTAMPTZ | |

#### `visit_log`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | |
| `host_account_id` | UUID (FK → accounts.id) | |
| `visitor_email` | TEXT | |
| `visited_at` | TIMESTAMPTZ | |
| `duration_seconds` | INTEGER | Approximate; logged at session end. |

#### `session_tokens`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | |
| `account_id` | UUID (FK → accounts.id) | |
| `token_hash` | TEXT (unique) | SHA-256 of the session token. |
| `device_label` | TEXT | User-facing label e.g., "Chrome on macOS". |
| `created_at` | TIMESTAMPTZ | |
| `last_used_at` | TIMESTAMPTZ | |
| `revoked_at` | TIMESTAMPTZ | NULL if active. |

#### `magic_links`
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | |
| `email` | TEXT | |
| `token_hash` | TEXT | |
| `action` | TEXT | `signin`, `verify_email`, `export_download`, `visit`. |
| `payload` | JSONB | Extra data e.g., `invite_id` for visit links. |
| `created_at` | TIMESTAMPTZ | |
| `expires_at` | TIMESTAMPTZ | 15 minutes from creation. |
| `used_at` | TIMESTAMPTZ | Set on consumption; NULL if unused. |

### 3.2 Key data invariants

- **Personality vectors are never recomputed from event history at runtime.** The stored value is canonical. The simulation tick writes it; no other process mutates it.
- **Personality vector values are never exposed to the client numerically.** The state-snapshot API returns mood labels and procedural rendering parameters derived from the vector, not the raw floats.
- **All bird identities are stable.** `birds.id` is never regenerated across any migration, species-pool update, or name change.
- **plumage_saturation is monotonic non-decreasing.** The drift function only adds positive deltas (or zero) to this trait.
- **Event log is append-only.** No event is ever modified or deleted after write. This is the audit trail for the simulation.

---

## 4. API Surface

### 4.1 Auth endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/auth/request-link` | Body: `{ email }`. Sends magic link. Rate-limited per email. Returns 200 always (to avoid email enumeration). |
| `POST` | `/api/auth/verify-link` | Body: `{ token }`. Returns session token + account info, or 401 with matter-of-fact error. Invalidates token on use. |
| `POST` | `/api/auth/refresh` | Header: `Authorization: Bearer <session_token>`. Returns a fresh session token; rotates the token. |
| `POST` | `/api/auth/signout` | Header: `Authorization: Bearer <session_token>`. Revokes the session token. |
| `GET`  | `/api/auth/sessions` | Lists active session tokens with device labels. |
| `POST` | `/api/auth/sessions/{id}/revoke` | Revokes a specific session. |

### 4.2 Aviary state endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/aviary/snapshot` | Returns the current state snapshot for the authenticated account. Query params: `since=<timestamp>` for delta-only responses (optional optimization). |
| `POST` | `/api/aviary/events` | Appends one or more interaction events to the event log. Body: `{ events: [ { type, bird_id?, payload, client_timestamp } ] }`. Idempotency key in header. |
| `GET` | `/api/aviary/notebook` | Returns notebook entries. Query params: `before=<entry_id>` for cursor-based pagination, `limit` (default 50). |

### 4.3 State snapshot shape

```json
{
  "generated_at": "2026-05-26T14:30:00Z",
  "account": {
    "timezone": "America/New_York",
    "aviary_age_days": 120
  },
  "birds": [
    {
      "id": "uuid",
      "species": "warbler",
      "name": "Pip",
      "perch_zone": "front",
      "mood": "curious",
      "plumage_params": {
        "hue_shift": 0.02,
        "saturation": 0.72,
        "brightness": 0.65,
        "fluff_factor": 0.1
      },
      "position": { "x": 0.35, "y": 0.55 },
      "pose": {
        "type": "perched_scanning",
        "phase": 0.43,
        "cycle_progress": 0.7
      },
      "call": {
        "is_calling": true,
        "motif_id": "warble_a3",
        "pitch_base_hz": 820,
        "volume": 0.55,
        "start_offset_ms": 1200
      }
    }
  ],
  "scene": {
    "lighting": {
      "phase": "morning",
      "warmth": 0.65,
      "brightness": 0.82
    },
    "weather": {
      "active": false,
      "type": null
    },
    "settled": false
  },
  "ambient_events": [
    { "type": "leaf_drift", "at": 0.15, "trajectory": "low_left_to_right" }
  ],
  "notebook_unread_count": 2
}
```

### 4.4 Event-append payload

Each event in the batch:

```json
{
  "type": "offer",
  "bird_id": "uuid-of-bird",
  "client_timestamp": "2026-05-26T14:29:55.123Z",
  "payload": {
    "offer_type": "seed",
    "offered_near_birds": ["uuid-1", "uuid-2"]
  }
}
```

Event types and their payloads:

| Event | Payload |
|-------|---------|
| `presence_start` | `{}` (server deduces start from prior presence_pings) |
| `presence_ping` | `{ consecutive_window_ms: 180000 }` — how long the three conditions have held continuously |
| `presence_end` | `{ reason: "tab_close" \| "settle" \| "visibility_lost" }` |
| `offer` | `{ offer_type: "seed" \| "song" \| "still_pool", offered_near_birds: [uuid] }` |
| `listen_in_start` | `{ bird_id: uuid }` |
| `listen_in_end` | `{ bird_id: uuid, duration_ms: 124000 }` |
| `settle` | `{}` |

### 4.5 Visit endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/visits/invite` | Body: `{ visitor_email }`. Host sends an invitation. Rate-limited. |
| `GET` | `/api/visits/invitations` | Lists outstanding and recent invitations for the host. |
| `POST` | `/api/visits/invitations/{id}/revoke` | Revokes an invitation. |
| `GET` | `/api/visits/log` | Returns the visit log. |
| `GET` | `/api/visits/view/{invite_token}` | Visitor fetches a read-only state snapshot of the host's aviary. Returns the same shape as `/api/aviary/snapshot` but with `read_only: true`. Returns 403 if token is expired/revoked. |
| `POST` | `/api/visits/view/{invite_token}/end` | Visitor signals end of visit (logs duration). Optional; visit also terminates on token expiry or host revocation. |

### 4.6 Account management endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/account` | Returns account settings and metadata. |
| `PATCH` | `/api/account` | Updates settings (timezone, preferences). |
| `POST` | `/api/account/change-email` | Body: `{ new_email }`. Sends verification to new address. |
| `POST` | `/api/account/export` | Triggers JSON export; download link emailed. |
| `POST` | `/api/account/delete` | Initiates soft deletion. |
| `POST` | `/api/account/restore` | Cancels soft deletion within 30-day window. |

### 4.7 Error response convention

All error responses use the matter-of-fact voice. Structure:

```json
{
  "error": "session_expired",
  "message": "Your session timed out. Sign in again to keep watching."
}
```

Status codes: 401 (unauth), 403 (forbidden/revoked), 404 (not found), 409 (conflict, e.g., email already verified), 429 (rate limited), 500 (internal).

---

## 5. Simulation Engine Design

### 5.1 Tick loop architecture

The simulation tick runs as a scheduled worker process. For each active account (not soft-deleted, with `last_ticked_at` older than the tick interval), the worker:

1. **Reads** all unprocessed events from `interaction_events` since `last_ticked_at`, ordered by `(server_timestamp, id)`.
2. **Aggregates** events into a summary struct: total presence-time in the window, per-bird listen-in durations, offer counts and types, settle events, ambient time-of-day context.
3. **Computes drift deltas** for each bird's personality vector (see §5.2).
4. **Transitions moods** for each bird (see §5.3).
5. **Advances mood timers** — if a bird has been in `drowsy` for N ticks past a threshold, auto-transition.
6. **Evaluates ambient events** — passes weather, determines if a chorus event should fire, advances the day/night phase.
7. **Generates notebook entries** if criteria are met (see §5.4).
8. **Writes** personality_vectors, moods, notebook_entries, and updates `last_ticked_at` on the account — all in one transaction per account.
9. **Marks** processed events as consumed (by advancing a `last_consumed_event_id` pointer on the account row).

### 5.2 Drift function

The drift function is a low-pass filter over presence and interaction signals. The calibration target: measurable in instruments at ~1 week of regular visits; visible to the user at ~3 weeks.

For each trait `T` and each tick window:

```
delta_T = α_T * (presence_minutes * w_presence
                + listen_in_minutes * w_listen_in
                + offer_count * w_offer
                + chorus_participation * w_chorus)
```

Where:
- `α_T` is a per-trait learning rate, experimentally calibrated. Typical values: `α_boldness = 0.0003`, `α_social_warmth = 0.0004`, `α_vocal_frequency = 0.0002`, `α_curiosity = 0.0005`, `α_plumage = 0.0001`.
- Weights `w_*` are global constants, tuned during calibration. Rough ordering: presence > listen-in > offers.
- All deltas are **non-negative**. Plumage saturation is explicitly clamped: `plumage_new = plumage_old + max(0, delta_plumage)`.
- Each trait is clamped to [0.0, 1.0] after delta application.

The drift function is **not** a simple running average. It applies a mild exponential decay to inputs older than ~2 weeks so that very old presence-time fades from influence. This is implemented by maintaining a per-bird `drift_accumulator` that decays at ~5% per tick, and the tick adds new weighted inputs to it.

**Calibration strategy:** Before launch, we run a simulation harness that replays synthetic presence patterns (e.g., 10 min/day for 3 weeks, 1 hour/day for 3 weeks, 0 min for 2 weeks then 30 min/day for 2 weeks) and verify:
- Week 1: trait deltas are instrument-detectable but sub-user-perception (<0.03 change).
- Week 3: visible deltas (0.05–0.10 change) accumulate.
- 2-week absence: no negative deltas; personality plateaus, doesn't regress.
- Plumage saturation: strictly monotonic across all patterns.

### 5.3 Mood transitions

Mood is a per-bird enumerated state: `{ wary, content, curious, drowsy, alert }`.

Transitions are evaluated each tick using a weighted rule engine:

| Trigger | Effect | Weight |
|---------|--------|--------|
| Early morning (5–8am user local) | Bias toward `alert` | 0.4 |
| Midday | Bias toward `content` | 0.2 |
| Dusk (1–2 hours before user-local sunset) | Bias toward `drowsy` | 0.5 |
| Night | Most birds → `drowsy`; nightjar species → `alert` | 0.7 |
| Offer accepted in current window | Bias toward `content` or `curious` depending on personality | 0.3 |
| Listen-in in current window | Small bias toward `curious` for focused bird | 0.2 |
| Passing rain active | Bias toward `wary` (dampens vocal frequency) | 0.3 |
| Wind active | Some birds → `alert`, others → `wary` (personality-modulated) | 0.2 |
| Another bird's alarm-like call | Nearby birds bias toward `wary` | 0.25 |
| High boldness personality | Resistance to `wary` — subtracts from wary-transition weight | proportional to boldness |
| Settle gesture | All birds bias toward `drowsy` | 0.5 |

The transition engine scores each possible target mood against the current mood. The mood with the highest score (above a hysteresis threshold to prevent oscillation) becomes the new mood. Hysteresis threshold: a new mood must outscore the current mood by at least 0.15 to trigger a transition.

### 5.4 Notebook entry generation

The tick evaluates notebook-entry conditions each pass. Entries are rare — roughly one every few days for an active aviary. Generation rules:

| Condition | Entry example |
|-----------|---------------|
| Bird A greeted first, and bird B usually does | "pip greeted before wren today, first time this week." |
| Two birds called in chorus for >10s | "a brief chorus between pip and wren this afternoon, then quiet." |
| Weather event passed | "a short rain moved through the aviary this morning. wren fluffed against the damp and went quiet." |
| Long quiet period (no calls >1 hour) | "a long stretch of quiet this morning. pip preened for several minutes without looking up." |
| Bird stayed on back perch all day | "wren stayed on the back perch all day, watching." |
| Mood shift notable enough (bird went from alert → drowsy early) | "pip settled early today, a full hour before dusk." |
| Weekly summary (capped at 1/week) | "this week the aviary was mostly quiet. wren called softly at dawn most days." |

The implementation uses a template grammar with procedural fill-in (bird names, time-of-day adverbs, mood-derived descriptions). Templates are maintained in a JSON file server-side; the tick selects applicable templates, fills slots from current state, and deduplicates against recent entries (no identical entry within 48 hours).

### 5.5 Species pool and seed personality values

Six species at launch. Each is defined by a visual silhouette, a default plumage palette, a call-grammar motif library, and seed personality vector ranges:

| Species | Boldness seed | Social warmth seed | Vocal freq seed | Curiosity seed | Plumage seed |
|---------|--------------|-------------------|-----------------|---------------|-------------|
| Warbler | 0.35–0.55 | 0.45–0.65 | 0.60–0.75 | 0.40–0.55 | 0.40–0.50 |
| Finch | 0.25–0.40 | 0.30–0.50 | 0.40–0.60 | 0.50–0.70 | 0.30–0.45 |
| Sparrow | 0.40–0.60 | 0.50–0.70 | 0.35–0.55 | 0.30–0.45 | 0.45–0.55 |
| Nuthatch | 0.15–0.35 | 0.20–0.40 | 0.25–0.45 | 0.45–0.60 | 0.35–0.50 |
| Nightjar | 0.10–0.25 | 0.10–0.25 | 0.15–0.30 (day) / 0.50–0.70 (night) | 0.25–0.40 | 0.20–0.35 |
| Flycatcher | 0.30–0.50 | 0.35–0.55 | 0.50–0.65 | 0.55–0.70 | 0.40–0.50 |

Each bird's actual seed is drawn uniformly from its species range at adoption time, seeded by `(account_id + bird_id + adopted_at)` to make it deterministic across replays.

### 5.6 Bird-to-bird interaction within the tick

Each tick evaluates pairwise interactions:
- If bird A is calling and bird B has `vocal_frequency > 0.5`, B has a probability of joining the chorus (proportional to social_warmth).
- If bird A is in `wary` mood, birds within proximity (same or adjacent perch zone) receive a small wary-transition weight bump.
- If two birds share the front perch and both have `social_warmth > 0.6`, they may trigger a "close-perch" event (the rendering reflects two birds perching near each other).

### 5.7 Time-gated bird addition

New bird offers appear based on aviary age, not visit count:

| Aviary age | Bird count unlocked |
|------------|---------------------|
| At creation | 2 (starter birds) |
| ~3 months | 3 |
| ~6 months | 4 |
| ~10 months | 5 |
| ~15 months | 6 |
| ~21 months | 7 |

The offer appears as a subtle cue in the aviary scene (a new silhouette on the back perch, gradually resolving over a few sessions) rather than a modal popup. The user names the new bird at adoption.

---

## 6. Sync Model

### 6.1 Architecture

There is only one canonical aviary state per account: the row(s) in PostgreSQL, written exclusively by the simulation tick. Multi-device "sync" is a consequence of this single-writer model:

- **Device A** (laptop) pulls a snapshot → renders the aviary → sends interaction events → closes.
- **Device B** (phone) pulls a snapshot later → sees the aviary that continued ticking on the server, including drift from device A's session.

### 6.2 State snapshot pull logic (client)

1. Client requests `GET /api/aviary/snapshot` on:
   - Tab navigation (fresh open).
   - `visibilitychange` event when tab becomes visible after being hidden.
   - `focus` event on the window after being blurred for >60 seconds.
   - Low-frequency keepalive (~every 120 seconds while visible and focused).
2. Server returns current state. If the client provided `since=<previous_snapshot_generated_at>`, server may return a delta (birds that changed, scene transitions) instead of the full payload. Delta optimization is a v1 stretch goal; the full snapshot is small enough (~2KB–5KB) that it's not load-bearing at launch.
3. Client interpolates between the previous snapshot and the new snapshot over a configurable duration (default: 500ms) to avoid teleporting birds. Perch changes cross-fade; mood changes are reflected in the next idle-motion cycle; call timing adjusts on the next call boundary.

### 6.3 Conflict prevention

Conflicts are **prevented by design**, not resolved:

| Potential conflict | Prevention mechanism |
|-------------------|---------------------|
| Two devices writing personality state | Clients never write personality. Only the server tick writes personality. |
| Two devices writing events for the same session | Event log is append-only. Events are ordered by `server_timestamp`. If two devices send conflicting events (e.g., both claim `presence_start` for the same time window), the tick deduplicates by picking the first event and ignoring subsequent `presence_start` events within a short window. |
| Late-arriving events from a device that lost connectivity | Events carry `client_timestamp`. The tick processes events in `server_timestamp` order, but uses `client_timestamp` to determine whether an event falls within the current tick window. Events older than 2 tick windows are dropped (they cannot usefully contribute to drift; the drift function's exponential decay means very old events have near-zero weight). |
| Session-token replay | Tokens are SHA-256 hashed. An attacker replaying a token is harmless because the session is already valid. Token rotation on refresh limits the window. |
| Magic-link replay | Token is invalidated on first use (`used_at` set). Replay returns 401. |

### 6.4 Offline behavior

There is no offline mode. If the client cannot reach the server:
- The last-known snapshot remains rendered (birds continue idle micro-motion from the snapshot, calls play from the snapshot's call schedule, but no new server-driven events arrive).
- Interaction events are queued client-side (in memory, up to a small cap of ~50 events) and flushed on reconnect.
- The UI shows no "offline" indicator or toast. The aviary simply continues with the last snapshot; the user may notice that changes don't propagate, but the system does not announce the failure.
- On reconnect, the client flushes queued events and pulls a fresh snapshot.

The deliberate absence of an offline indicator is a "notice, never announce" decision: an "offline" banner would announce a system state, breaking the felt-aliveness. If the user notices the aviary is frozen, they'll reload; if they don't, they haven't been harmed.

---

## 7. Frontend Rendering Pipeline

### 7.1 Framework and bundling

- **Framework**: Preact (lightweight React-compatible, ~3KB gzipped) or Solid (fine-grained reactivity, ~4KB gzipped). Decision to be made during prototype phase based on animation-frame performance with 7 birds + ambient effects at 60fps. Both are well under the bundle budget.
- **Bundler**: Vite with code-splitting via dynamic `import()`. The aviary shell + first-bird rendering path is in the critical bundle; account settings, accessibility settings, visit-invitation flow are in lazy-loaded chunks.
- **TypeScript** throughout, with strict mode. Shared types between client and server live in a `shared/` package consumed by both.

### 7.2 Loading sequence (the critical path)

```
Navigation
  │
  ├─ 1. HTML shell arrives from CDN (edge-cached, ~5KB)
  │     Contains inline critical CSS for aviary background color + scene dimensions.
  │     Contains a small inline script that:
  │       a. Creates the <canvas> or SVG root at full viewport.
  │       b. Fills it with the quiet-field color (soft sky gradient).
  │       c. Optionally triggers one or two ambient motion cues (a leaf, a faint light shift).
  │
  ├─ 2. JS bundle begins downloading (deferred, non-blocking).
  │
  ├─ 3. First state-snapshot request fires (~50ms after navigation, before bundle finishes).
  │     The snapshot is delivered from a CDN edge with the HTML.
  │     This is a small GET to /api/aviary/snapshot; the response is cached at the edge
  │     for that account for a short TTL (~5 seconds) to absorb tab-reload storms.
  │
  ├─ 4. Bundle arrives. If snapshot has already arrived, rendering begins immediately.
  │     If not, the quiet-field loading state persists (no spinner).
  │
  └─ 5. First bird renders (<500ms target). Subsequent birds and full scene fill in over
        the next few frames.
```

### 7.3 Rendering technology: Canvas2D or WebGL vs. SVG vs. DOM

**Decision: Canvas2D with a thin abstraction layer.**

- **Why not DOM/SVG**: 6–7 birds with per-frame idle micro-motion, ambient leaf drift, parallax, and weather effects would push DOM layout/reflow past the 60fps budget on a 5-year-old laptop. SVG compositing of multiple animated elements with cross-fades also hits CPU bottlenecks.
- **Why not WebGL**: Overkill for a 2D scene. The complexity of shader management, text rendering (call captions, notebook), and cross-browser WebGL quirks adds engineering risk without proportional benefit. WebGL may be considered for a future release but is not v1.
- **Canvas2D approach**: A single `<canvas>` element fills the viewport. A render loop (`requestAnimationFrame`) draws the scene bottom-to-top: sky gradient → background foliage → back perch → middle perch → front perch → birds (in display_order) → foreground elements (leaves, feathers) → top bar (overlaid as DOM for accessibility).

### 7.4 Scene composition (render loop)

Each frame:

1. **Sky and lighting**: Radial or linear gradient from the lighting state (time-of-day warmth, brightness). Transitions between lighting phases use eased lerp over several minutes (so day→evening is a slow cross-fade, not a hard cut).
2. **Background foliage**: Simple silhouettes drawn from small SVG assets, positioned statically, with subtle parallax offset from the sky layer (<5px offset on scroll/device tilt if we add that; not v1).
3. **Perches**: Three horizontal lines or branch shapes at fixed Y positions (proportional to viewport height). Front perch at ~70% height, middle ~55%, back ~40%.
4. **Birds**: Each bird is drawn at its current position. Bird rendering is procedural: a species silhouette is loaded from a compact SVG asset, then transformed (scale, hue-shift, saturation, animation phase). See §7.6.
5. **Ambient elements**: Leaves and feathers are spawned at random intervals (Poisson process, λ ~1 per 10–30 seconds), follow a simple trajectory (parabolic arc with wind influence), and are removed when offscreen. No per-element server state.
6. **Weather overlay**: Rain is rendered as thin vertical lines with slight angle, semi-transparent, with a soft blue-grey overlay on the scene. Wind is visual only as faster leaf drift + slight branch sway.
7. **Top bar**: Rendered as DOM overlay (not canvas) for native accessibility tree exposure. Semi-transparent, fades after cursor stillness.
8. **Call captions** (if enabled): Positioned near calling birds as DOM overlays, fading in/out with call duration.

### 7.5 Bird rendering

Each bird is drawn per-frame using:

1. **Species silhouette**: A base SVG path for the bird's outline, rendered onto an offscreen canvas at startup, then drawn per-frame with `drawImage()`.
2. **Plumage coloration**: The base silhouette is filled with a species-palette color, then hue-shifted and saturation-adjusted based on the `plumage_params` from the state snapshot (derived from the hidden `plumage_saturation` personality trait).
3. **Pose**: The current idle-motion pose (preening, scanning, head-tilt, weight-shift, fluffed, resting) transitions between keyframes. Keyframes are defined as a small set of transforms (head angle, body lean, wing position, tail position) and interpolated with easing functions. Pose transitions are triggered by the client's idle-motion scheduler, not by snapshots — the snapshot says "bird is in content mood" and the client's idle-motion engine selects poses from the content-mood pose set.
4. **Perch transitions**: When a bird changes perch zone (between snapshots), the client animates the bird along a Bezier curve from old perch to new perch over ~1.5 seconds. The path includes a slight arc (fly-up then settle) to read as a short flight rather than a slide.

### 7.6 Idle micro-motion scheduler

The client runs a per-bird idle scheduler independent of the render loop:

- Each bird has an idle-state timer that fires at randomized intervals (Poisson with λ adjusted by mood: `drowsy` birds have longer intervals, `alert` birds have shorter).
- On timer fire, the scheduler selects a new pose from the mood's pose set, weighted randomly with no-repeat penalty (the same pose can't fire twice in a row).
- Pose transition duration is 800ms–2s depending on pose complexity.
- Between poses, a small continuous "breathing" animation (subtle body scale oscillation, ±2%) runs to prevent the bird from reading as frozen.

### 7.7 Reduced-motion rendering path

When `prefers-reduced-motion` is active or the user has toggled the setting:

1. Idle micro-motion is replaced by slow cross-fades between still poses (no frame-by-frame interpolation; the bird fades from pose A to pose B over 2 seconds).
2. Perch transitions are cross-fades (the bird fades out at the old perch, fades in at the new perch) — no animated flight path.
3. Ambient leaf drift is removed entirely.
4. Ambient lighting transitions (day/night) remain but are slowed to 2x the normal duration.
5. Call audio still plays (or captions, as configured). None of the simulation-affecting behavior changes.

### 7.8 Keyboard navigation

Implementation:
- Top bar items are in a `<nav>` with `role="navigation"`. Tab moves through them left to right.
- The aviary canvas is focusable (`tabindex="0"`). When focused, a focus ring (soft outline) appears. Arrow keys move a virtual focus indicator between birds (highlighting the focused bird with a subtle glow).
- Enter/Space triggers listen-in on the focused bird.
- Escape exits listen-in, returns focus to the aviary scene.
- The offer panel (triggered from top bar) opens as a DOM dialog with full keyboard navigation.
- Focus management: when listen-in disengages, focus returns to the previously focused bird.

---

## 8. Audio Pipeline

### 8.1 Architecture

All audio is synthesized client-side via the WebAudio API. No audio files are downloaded. The audio pipeline runs in an `AudioWorklet` or main-thread `ScriptProcessorNode` (fallback) depending on browser support.

### 8.2 Call grammar

Each bird species has a **call grammar** — a set of motifs with transformation rules:

- **Motif**: A short note sequence defined by a frequency envelope, a filter envelope, and a noise component. Stored as a small JSON structure, not a waveform.
- **Motif library per species**: ~8–12 motifs per species. Each motif is a distinct "phrase" that the bird's call is built from.
- **Runtime composition**: When the call scheduler decides a bird should call, it:
  1. Selects a motif from the library (weighted random, no immediate-repeat penalty).
  2. Applies transformations: pitch shift (±15%), tempo stretch (±20%), filter resonance variation.
  3. Sequences motifs: a call may be a single motif, two motifs in sequence, or a motif with a short pause and repeat.
  4. Synthesizes the resulting waveform via an `OscillatorNode` bank + `BiquadFilterNode` + `GainNode` chain.
  5. Applies per-bird timbre: each bird has a slight formant shift derived from its personality vector (stable across sessions) so Pip always sounds like Pip.

### 8.3 Call scheduling

- Each bird has an independent call timer driven by its `vocal_frequency` personality trait. Higher vocal frequency → calls more often.
- Call timing uses a random interval drawn from an exponential distribution with mean `base_interval / vocal_frequency`. Base interval is calibrated so that an average-vocal-frequency bird calls roughly once every 30–90 seconds when unobserved.
- Calls are not synchronized between birds — this is what makes choruses feel organic. When two birds happen to call within the same ~2-second window, the audio pipeline mixes them in real time (additive mixing with soft limiting to prevent clipping).
- During listen-in, the focused bird's call timer accelerates slightly (calls come ~30% more often) and the other birds' timers decelerate (~50% less often). This reinforces the feeling of paying attention to one bird.

### 8.4 Audio mixing

The audio graph:

```
 [Bird 1 Call Synth] ──→ [GainNode g1] ──┐
 [Bird 2 Call Synth] ──→ [GainNode g2] ──┤
 ...                                      ├──→ [MasterGain] ──→ [destination]
 [Bird N Call Synth] ──→ [GainNode gN] ──┤
                                           │
 [Ambient (wind, rain)] ──→ [GainNode a] ─┘
```

Gain nodes are modulated per-bird based on:
- **Listen-in state**: Focused bird → gain rises to ~0.8 over 1 second. Other birds → gain drops to ~0.15 over 1 second.
- **Perch zone**: Front-perch birds are slightly louder (~+2dB) than back-perch birds (spatialization by volume, not stereo panning — the aviary is small enough that stereo cues are unnecessary and add complexity).
- **Time of day**: Evening/night calls are quieter across all birds (~−3dB).
- **Settle**: All gains ramp to ~0.1 over 3 seconds, then hold.

### 8.5 Chorus mechanic

When two or more birds call within the same ~2-second window:
1. Each bird's call is synthesized independently (no pre-mixing).
2. Calls are mixed additively.
3. A soft limiter (`DynamicsCompressorNode` with ratio 4:1, knee 6dB) prevents clipping.
4. Chorus events are noted in the server's event stream (for notebook generation) but the audio is purely client-side.

### 8.6 WebAudio fallback

If `AudioContext` construction fails (browser restriction, hardware issue, permission denied):
- The aviary renders normally with all visual elements.
- Call captions are automatically enabled (the setting silently toggles on for this session; the user's persisted preference is not changed).
- No error message is shown. The aviary is silent with captions.
- There is no recorded-audio fallback path. The bundle contains zero audio files.

### 8.7 Audio performance

- `AudioContext` is created on first user gesture (click, tap, keypress) to comply with autoplay policies. Before the first gesture, the aviary is visually alive with captions default-on.
- Call synthesis is memoized: identical motif+transform combinations cache their `AudioBuffer` for ~60 seconds to avoid repeated synthesis work.
- `AudioWorklet` is used when available (off-main-thread audio processing). `ScriptProcessorNode` is the fallback.
- Audio buffer allocation is bounded: no more than 20 concurrent `AudioBuffer` instances at any time. Oldest unused buffer is evicted.

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

The narration is a live region (`aria-live="polite"`) updated on a slow cadence from a client-side narration queue.

**Generation pipeline:**
1. The state snapshot carries enough semantic information (bird species, name, mood, perch zone, calling state, time of day, weather, settled state) for the client to compose narration prose.
2. Narration prose is generated client-side using a template grammar mirroring the notebook voice. Templates map state combinations to prose sentences.
3. The narration queue holds at most 1 pending update. If a new update arrives while one is queued, the older one is dropped (the user hears the latest state, not a backlog).
4. Cadence: ~30–60 seconds at idle. User-initiated events (return greeting, offer response, settle) get a priority update immediately.

**Template examples:**
- `"a {species_name} is perched on the {perch_zone} perch{f mood == 'calling' → ', calling softly'}. {if second_bird → 'another bird sits further back'}."`
- `"it is {time_of_day_phrase} in the aviary; the light is {lighting_adjective}."`
- `"{bird_name} noticed you and tilted her head."` (on return-greeting)

### 9.2 Call captions

Captions are generated from the audio pipeline's call composition step:
- When a bird's call is synthesized, the call's structural description (motif names, pitch range, trill presence, pause pattern) is converted to prose: `"a soft three-note rise"`, `"a low trill, paused, low trill again"`, `"a single sharp call from the back perch"`.
- Caption text is rendered as a DOM overlay near the calling bird, with `aria-live="assertive"` for screen-reader users who also want call descriptions.
- Captions fade in over 300ms, hold for call duration + 500ms, fade out over 300ms.
- During chorus events, captions for both birds appear simultaneously, slightly offset vertically to avoid overlap.

### 9.3 Focus management and keyboard nav

- Focus ring: a 2px soft-glow outline using `box-shadow` with `0 0 0 2px rgba(80, 140, 200, 0.8)` against a `0 0 0 4px rgba(80, 140, 200, 0.3)` halo. Visible against both bright and dim aviary states.
- Top bar accessibility: each icon is a `<button>` with `aria-label`. The notebook icon gets `aria-label="field notebook"`. Settings gets `aria-label="account and settings"`.
- Bird focus: the virtual bird focus indicator is rendered as a subtle highlight on the canvas (a soft elliptical glow beneath the bird). Arrow-key navigation wraps (leftmost → rightmost).
- Offer dialog: a `<dialog>` element with `aria-modal="true"`, trap-focus behavior, Escape to close.

### 9.4 Reduced-motion implementation

- The `prefers-reduced-motion` media query is checked at startup and monitored for changes.
- The user can also toggle reduced motion in accessibility settings (stored server-side in `accounts.settings`).
- The rendering pipeline branches at the idle-scheduler level: if reduced-motion is active, the frame-by-frame animation path is replaced with the cross-fade path. See §7.7.
- Reduced-motion does not affect the simulation or audio; it only affects visual rendering.

### 9.5 Color and contrast

- All user-copy text (top bar labels, settings, account surfaces, error messages, captions, narration text when displayed visually) meets WCAG AA minimum contrast ratio (4.5:1 for normal text, 3:1 for large text).
- The top bar uses a semi-transparent background with a text color that maintains contrast against the aviary sky gradient behind it. When the top bar is nearly transparent (faded state), its text also fades — this is acceptable because the user is not meant to read the top bar during viewing; on cursor movement, it returns to full contrast.
- Focus indicators and interactive element borders meet 3:1 minimum contrast against adjacent colors.

---

## 10. Performance Budgets and Observability

### 10.1 Bundle budget

| Asset | Budget (gzipped) |
|-------|-----------------|
| Critical JS (aviary shell + first bird render) | <800KB |
| Full JS (all routes, lazy loaded) | <2MB |
| CSS (critical + lazy) | <50KB |
| SVG bird assets (6 species × ~3 poses each) | <200KB |
| **Total initial payload** | **<2MB** |

Enforcement: CI build step measures gzip size and fails the build if any threshold is exceeded.

### 10.2 Render performance budgets

| Metric | Threshold | Measurement |
|--------|-----------|------------|
| Time-to-first-bird (TTFBird) | <500ms | Navigation start → first bird drawn on canvas. Measured via `performance.mark('first-bird-visible')`. |
| Frame budget (idle) | 60fps | `requestAnimationFrame` timing on a 5-year-old mid-range laptop. Measured in CI on a throttled CPU profile. |
| Frame budget (listen-in transition) | 60fps | Audio mix change + camera focus must not drop frames. |
| 30-min memory | No growth | `performance.memory.usedJSHeapSize` sampled every 30s for 30 minutes. Linear regression slope must be ≤ 0. |
| Audio context latency | <10ms | Time from user gesture to `AudioContext.resume()` completion. |

### 10.3 Server performance budgets

| Metric | Threshold |
|--------|-----------|
| State-snapshot response time (p50) | <50ms |
| State-snapshot response time (p99) | <200ms |
| Event-append response time (p50) | <30ms |
| Simulation tick duration (p50 per account) | <200ms |
| Simulation tick duration (p99 per account) | <5s (alarm threshold) |
| Magic-link email delivery latency (p95) | <30s |

### 10.4 Observability stack

| Signal | Tool | Data |
|--------|------|------|
| API request metrics (count, latency, errors) | Aggregate metrics pipeline (e.g., Prometheus + Grafana) | Per-endpoint, per-status-code. No per-account dimension. |
| Simulation tick metrics | Aggregate metrics pipeline | Per-tick-phase duration, per-account drift-delta distributions (anonymized). |
| Client-side Real User Monitoring | RUM agent in browser | Page load timing, TTFBird, render-frame timing histogram, audio-context error count. No per-account, no per-bird, no interaction-event data. |
| Error tracking | Error aggregation service (e.g., Sentry) | Stack traces, error messages. **Sanitized**: account UUIDs, bird names, and emails stripped from error context before submission. |
| Synthetic monitoring | Fleet of headless browsers | Scheduled runs from 3+ geographic regions, measuring TTFBird, render-frame consistency, audio-context availability. |
| Uptime monitoring | Health-check endpoint | `GET /health` returns 200 if database + email service are reachable. |

### 10.5 What we deliberately do NOT measure

- Per-bird interaction counts in aggregate.
- Per-account drift rates in aggregate.
- "Average bird boldness across all accounts."
- Any metric derivable from `personality_vectors` or `interaction_events`.
- Any metric that could answer the question "what is this account's bird doing?"

The line is at the data pipeline level: the simulation database is on a separate PostgreSQL instance from any analytics warehouse, and no ETL job reads from it. Aggregate telemetry tables are populated from API gateway metrics only.

---

## 11. Rollout Plan

### 11.1 Phased rollout

| Phase | Duration | Scope | Gates |
|-------|----------|-------|-------|
| **Alpha** (internal) | 2 weeks | Team members only. Full feature set, 2-bird aviaries. | All CI tests green. TTFBird <500ms on lab devices. No simulation-tick p99 alarms. |
| **Closed beta** | 4 weeks | 100–500 invited users. Full feature set. Email-based invites. | Drift calibration verified (instruments show expected deltas at 1-week mark). No personality-vector data loss. Zero "Welcome back" text found in audit. |
| **Open beta** | 4 weeks | Self-serve signup, no invite gate. Capped at 10K accounts. | p99 simulation-tick latency <1s. Support ticket rate <1%. Accessibility audit passes (screen-reader narration quality, keyboard nav completeness, contrast ratios). |
| **GA launch** | — | Open signup, no account cap. | All beta gates sustained for 2 weeks. |

### 11.2 Bird-cap ramp

| Phase | Max birds per aviary |
|-------|---------------------|
| Alpha | 2 |
| Closed beta | 2 (3 unlocked at 3-week mark for early adopters) |
| Open beta | 5 |
| GA | 7 |

The ramp lets us validate audio-pipeline chorus behavior at increasing bird counts before exposing 7-bird choruses to the full user base.

### 11.3 Launch instrumentation

From day one of alpha:
- TTFBird p50/p95/p99 tracked per geographic region.
- Simulation-tick latency p99 with alarm at 5s.
- Audio-context failure rate.
- Error rate by endpoint.
- Session-duration histogram (anonymized, no per-account breakdown).

### 11.4 Pre-launch calibration work

Before alpha:
1. Run the simulation harness with synthetic presence patterns for 4 weeks of simulated time. Verify drift function calibration (measurable at week 1, visible at week 3).
2. Run the rendering pipeline on a throttled-CPU CI profile representing a 5-year-old mid-range laptop (2-core, 2GB RAM, integrated GPU). Verify 60fps idle motion.
3. Run the audio pipeline through a chorus stress test (7 birds with vocal_frequency at max, calling simultaneously) and verify no clipping, no audio-context crashes, no memory growth over 30 minutes.
4. Accessibility audit: screen-reader narration prose quality, keyboard nav completeness, reduced-motion cross-fade aesthetics.
5. Security review: magic-link replay protection, session-token rotation, email enumeration resistance, synthetic UUID hygiene.

---

## 12. Risks

### 12.1 Drift calibration — "feels wrong" risk

**Risk:** The drift function's learning rates (`α_T`) produce movement that is too fast (birds change personality between sessions → product feels like a Tamagotchi) or too slow (user feels nothing they do matters → product feels like a screensaver).

**Mitigation:**
- Calibration harness with synthetic presence patterns, run for simulated weeks before alpha.
- Beta instrumentation that measures per-account drift deltas (anonymized, aggregate distributions only) to detect population-level calibration issues.
- If drift is too fast, reduce `α_T` globally by a fixed factor (0.5×) and restart calibration. The function is designed with a single tuning knob per trait so adjustment is surgical.
- If drift is too slow, the risk is lower (users won't notice absence of drift as quickly as they notice overly fast drift) but still needs correction before GA.

**Contingency:** Drift rates are not exposed to users. If we need to recalibrate post-launch, the new rates apply to future ticks only; existing personality vectors are not retroactively adjusted. This means a user who has been using the product for 2 months at the old rate will have birds that drift at the new rate going forward — an acceptable discontinuity.

### 12.2 Sync correctness — "silent data loss" risk

**Risk:** A bug in the tick's event-consumption logic causes it to skip events, or a race condition between two tick workers processing the same account causes one worker's deltas to be overwritten. The failure is silent: no error log, no user-visible symptom except that drift is slower than expected.

**Mitigation:**
- Single-writer-per-account enforced at the database level: `SELECT ... FOR UPDATE` on the account row at the start of each tick, held until transaction commit. No two tick workers can process the same account concurrently.
- Event consumption uses a monotonic `last_consumed_event_id` pointer with `WHERE id > last_consumed_event_id` queries. No event-skipping via offset-based pagination.
- Synthetic monitoring: a special "test account" with a known sequence of events is ticked and the resulting personality vector is compared to a pre-computed expected value. Drift from expected value triggers an alarm.
- Per-account drift-delta distributions in aggregate telemetry: if the population-wide mean delta drops below a threshold, it may indicate systematic event loss.

### 12.3 Audio uncanniness — "sounds fake" risk

**Risk:** Procedural call synthesis produces calls that sound synthetic, metallic, or otherwise "off." The user hears it, the spell breaks, and the product's affective spine collapses. This is the hardest risk to mitigate because audio quality is subjective and hard to automate-test.

**Mitigation:**
- The motif library is designed by someone with synthesis expertise, not generated algorithmically. Each of the ~8–12 motifs per species is hand-tuned for naturalness.
- Motif transformations (pitch shift, tempo stretch, filter variation) are bounded conservatively: ±15% pitch, ±20% tempo. Wider ranges would produce more variation but risk entering uncanny territory.
- Chorus mixing uses soft limiting, not hard clipping. Hard clipping produces digital artifacts that are especially noticeable in layered calls.
- During beta, collect qualitative feedback specifically on call quality. If a species's calls are consistently rated poorly, redesign its motif library.

**Contingency:** If a species's audio is unsalvageable, we can ship with fewer species (e.g., 5 instead of 6) and introduce the problematic species later after redesign. The species pool is designed to be extensible.

### 12.4 Accessibility regression — "shipped without" risk

**Risk:** Accessibility surfaces (narration, captions, reduced motion, keyboard nav) are deprioritized during crunch and ship incomplete or absent. A reduced-motion user opens the aviary and sees broken animations. A screen-reader user hears "bird at perch 2" instead of naturalist prose.

**Mitigation:**
- Accessibility features are in the definition of done for every user story. A feature is not complete until its keyboard nav, screen-reader narrative, reduced-motion path, and contrast pass.
- Automated CI checks: Lighthouse accessibility score ≥ 95, axe-core passes with zero violations, contrast ratio linting on all text styles.
- Manual audit before each phase gate: a screen-reader user tests the full product flow, a keyboard-only user tests all interactions, a reduced-motion user tests session completeness.
- The PRD's "first-class, not a checklist" stance means accessibility work is not a separate track — it's the same rendering pipeline, the same prose generation, just with different output modes. There is no separate "accessibility build."

### 12.5 Bundle size creep — "over budget" risk

**Risk:** As features land, the JS bundle grows past the 2MB gzip cap. TTFBird exceeds 500ms on mobile. The product's central conceit (aviary already in motion) collapses into a loading screen.

**Mitigation:**
- Bundle size is a CI gate. Every PR's build output is measured; a PR that increases the bundle beyond the cap is blocked.
- Code-splitting boundaries are defined early (aviary shell vs. settings vs. visit flow vs. account management) and enforced by import lint rules.
- A bundle-visualization tool (e.g., `rollup-plugin-visualizer` or `vite-bundle-visualizer`) runs on every build and is published to the team dashboard. Dependencies that surprise in size are caught at review time.
- SVG bird assets are compact by design (silhouettes, not detailed illustrations). If an artist produces an asset that's too large, the CI size gate catches it.

### 12.6 Server-side tick scalability — "can't keep up" risk

**Risk:** At N accounts, the tick loop takes longer than the tick interval (1 minute) to process all accounts. The simulation falls behind; users see stale state.

**Mitigation:**
- Partition accounts across tick workers by UUID hash range. Each worker processes a subset of accounts. Workers are stateless; they can be scaled horizontally by adding more workers and shrinking each worker's hash range.
- Tick duration is measured and alarmed. If p99 exceeds 5s, on-call is paged.
- At alpha/beta scale (10K accounts), a single worker should handle all accounts comfortably (10K × 200ms avg tick = ~33 minutes of work per minute; parallelized across workers, this is trivial). At GA scale (100K+ accounts), horizontal scaling is already designed.
- If tick duration per account grows unexpectedly (e.g., notebook generation becomes expensive), optimize the slow phase or move notebook generation to an async queue.

### 12.7 "Notice, never announce" discipline erosion risk

**Risk:** During development, a "harmless" toast, banner, loading spinner, or "Welcome back!" text is added by a well-meaning contributor. The product's affective register degrades incrementally, and no single instance triggers alarm.

**Mitigation:**
- A lint rule that flags prohibited strings in the codebase: "Welcome back", "Achievement", "streak", "level", "badge", "congratulations", "great job", "you've been", "days visited". This is a mechanical gate, not a style-guide suggestion.
- PR review checklist includes "Does this surface announce the user or the system state?" If yes, flag for discussion.
- The design principles from `product_brief.md` are posted in the team's working document and referenced in code review templates.
- A "notice audit" before each phase gate: a reviewer opens every surface in the product and confirms zero announcement-style UI.

---

## Appendix A: Shared vocabulary map

| PRD term | Code identifier | Notes |
|----------|----------------|-------|
| Bird | `Bird` | Entity, never `creature`, `pet`, `animal`, `character`. |
| Aviary | `Aviary` / `aviary` | The whole scene; the account's state container. |
| Call | `call` / `BirdCall` | Never `song`, `noise`, `chirp`. |
| Mood | `Mood` / `mood` | Enumerated state per bird. |
| Personality vector | `PersonalityVector` / `personality` | Hidden numerical traits. |
| Drift | `drift` / `applyDrift()` | Slow cumulative personality change. |
| Presence | `presence` / `PresenceEvent` | The three-condition attention signal. |
| Listen-in | `listenIn` / `ListenInState` | Never `solo`, `select`, `highlight`, `pin`. |
| Offer | `offer` / `Offer` | Seed, song fragment, or still pool. |
| Settle | `settle` / `settle()` | Soft session-end gesture. |
| Field notebook | `notebook` / `NotebookEntry` | Lowercase in prose, capitalized only as UI label. |
| Visit | `visit` / `VisitInvitation` | Read-only guest viewing. |
| Tick | `tick` / `simulationTick()` | Server-side simulation step. |

## Appendix B: Calibration harness design

The calibration harness is a standalone TypeScript program that:

1. Creates a synthetic account with 2 birds (seed personalities from species defaults).
2. Replays a pre-defined presence-and-interaction schedule: e.g., "10 minutes of presence + 2 offers per day for 21 days."
3. Runs the simulation tick after each simulated day (or each simulated minute, configurable).
4. Outputs a CSV of personality vector values over time.
5. Asserts:
   - Week 1: max delta < 0.03 (user shouldn't notice yet).
   - Week 3: max delta > 0.05 (user feels it when looking back).
   - Plumage saturation: strictly non-decreasing.
   - 2-week absence window: deltas are flat (no negative drift), then resume when presence returns.
6. Runs in CI on every change to the drift function.

## Appendix C: Species SVG asset spec

Each species needs:
- **Base silhouette**: A single-path SVG, viewBox 0 0 100 100, representing the bird in side profile. The path must be a single `<path>` element (colorization is done via canvas fill).
- **Pose variants**: 3–5 pose SVGs derived from the base silhouette, each with the same viewBox but different head angle, wing position, and tail position. Naming: `{species}_perched.svg`, `{species}_preening.svg`, `{species}_scanning.svg`, `{species}_fluffed.svg`, `{species}_resting.svg`.
- **File size**: Each SVG < 5KB raw. Total per species < 25KB. Total for 6 species < 150KB raw → < 200KB gzipped (well within budget).

---

*This plan covers v1 of Pocket Aviary. It is intended to be executable by a frontend engineer, a backend engineer, and an audio engineer working in parallel. Ambiguities have been resolved with defensible calls annotated above. The plan does not implement the product; it specifies what to build, how to build it, and what could go wrong.*
