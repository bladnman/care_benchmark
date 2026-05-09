# Pocket Aviary — Implementation Plan

## 1. Scope

### In v1
- Browser-based SPA (modern Chrome, Safari, Firefox, Edge — last two major versions)
- Magic-link email authentication; no passwords, no SSO
- Single account per user; single canonical aviary per account
- Two starter birds at account creation; aviary grows to a maximum of seven birds
- Six bird species in the launch pool
- Bird personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) — server-side, never exposed numerically to the user
- Mood system: wary, content, curious, drowsy, alert — persists across sessions, modulated by server-side tick
- Server-side simulation tick (~1/min cadence)
- Monotonic personality drift (additive only; no negative drift on neglect)
- Return-greeting: one bird notices the user within ~1–2s of tab open
- Interactions: listen-in, offer (seed / song fragment / still pool), settle
- Field notebook: auto-generated naturalist prose entries, sparse cadence
- Presence accounting: visibilityState + window focus + recent pointer/key activity, all three conjunctive
- Multi-device sync as an architectural property of server-canonical state
- Day/night cycle driven by user's local timezone
- Ambient weather (rain, soft wind) at rare intervals
- Ambient micro-motion (leaves, feathers) as client-side rendering ornaments
- Visit invitation feature: host invites visitor by email, read-only, revocable, off by default
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fades, not disabled animations)
- Call captions (naturalist voice, generated from call grammar at runtime)
- WCAG AA contrast on all user-copy text
- Full keyboard navigation
- Account export (JSON snapshot on demand)
- Soft-delete (30-day recovery window) then hard-delete
- Performance budgets: <2MB gzipped initial bundle, <500ms time-to-first-bird, 60fps idle on 5-year-old mid-range laptop, no memory growth over 30 minutes

### Explicitly out of v1
- Native iOS/Android apps
- Gamification of any form: achievements, streaks, levels, badges, scores, green-dot calendars, XP, visit-frequency counters
- Tamagotchi mechanics: hunger meters, distress on neglect, happiness decay
- Social network surfaces: profiles, follows, public discovery, leaderboards, comments on visits
- Shared or multi-user aviaries
- Payments or tiered features
- Push notifications of any kind
- SSO or password-based login
- Multi-aviary accounts
- Customizable scene layouts
- User control over bird perch positions
- Personality vector exposure (no debug view, no stats panel, no tier that unlocks it)

---

## 2. Architecture

### Service topology

```
Browser (SPA)
  │
  ├─── GET /api/aviary/state  ─────────────────────────────────────┐
  ├─── POST /api/events  ──────────────────────────────────────────┤
  ├─── POST /api/auth/*  ──────────────────────────────────────────┤
  ├─── GET /api/notebook  ─────────────────────────────────────────┤
  └─── GET/POST/DELETE /api/invites/*  ────────────────────────────┤
                                                                   │
                                                               API Server
                                                                   │
                     ┌─────────────────────────────────────────────┤
                     │                                             │
              Simulation Service                            PostgreSQL
              (cron tick ~60s)                          (canonical state)
                     │                                             │
                     └─────────── reads/writes ───────────────────┘
                                                                   │
                                                           Email Service
                                                    (magic links, exports, invites)
```

**API Server**: stateless HTTP service. Validates sessions, writes interaction events to the append-only event log, serves state snapshots, manages auth flows. Horizontally scalable.

**Simulation Service**: single-writer service that owns the simulation tick. Runs on a cron/timer (~60s cadence). Reads unprocessed interaction events from the event log, computes drift deltas and mood transitions, writes updated canonical state back to the database. Only this service writes personality vectors. Designed for idempotent re-runs (event log positions are tracked per tick run).

**PostgreSQL**: single source of truth for all canonical aviary state. No client-to-client replication. The simulation service is the only writer of personality_vectors and mood_state. The API server writes interaction_events and session records.

**Email Service**: transactional email only. Magic-link delivery, invite delivery, export download links. No marketing email, no notification email (except the opt-in visit notification toggle).

**CDN/Edge**: serves the SPA bundle and a pre-rendered initial HTML shell. The state snapshot endpoint is cacheable at the edge with a short TTL (a few seconds) to handle burst traffic on sign-in; individual state pulls are short-keyed on the account UUID.

### Client/server split

The client is a rendering layer. It:
- Pulls state snapshots and renders from them
- Interpolates between snapshots for smooth motion
- Synthesizes audio client-side via WebAudio
- Writes interaction events to the server's append-only log
- Never computes personality drift
- Never owns personality state
- Never resolves conflicts (there are none to resolve, because the server is the only writer)

The server is the simulation. It:
- Advances canonical state on every tick
- Is the only writer of personality vectors and mood
- Processes interaction events in log order, not wall-clock submit order
- Runs whether or not any client is connected

### Render pipeline boundary

The render boundary is the state snapshot. The snapshot contains:
- Per-bird: current position (perch zone), current mood, current call timing phase, plumage saturation (for visual rendering), current animation pose context (for reduced-motion mode)
- Aviary-wide: current day/night phase (derived client-side from local time, confirmed by server timestamp), active weather, pending notebook entries since last pull

The client derives all visual and audio parameters from this snapshot plus local time. It does not request individual bird properties; it requests the full aviary snapshot.

---

## 3. Data Model

### Tables (PostgreSQL)

#### `accounts`
```
id               UUID PRIMARY KEY  (synthetic, never derived from email)
email_encrypted  BYTEA             (AES-256-GCM; key stored in secrets manager)
email_hash       BYTEA             (HMAC for lookup; never stored in logs or telemetry)
created_at       TIMESTAMPTZ
deleted_at       TIMESTAMPTZ       (null = active; set on soft delete)
hard_delete_at   TIMESTAMPTZ       (set at deleted_at + 30 days)
```

No column referencing this table uses email as a foreign key. Every reference is to `accounts.id` (UUID). Email is used only for lookup and delivery, never as an identifier.

#### `sessions`
```
id           UUID PRIMARY KEY
account_id   UUID REFERENCES accounts(id)
device_label TEXT              (user-visible, e.g. "iPhone — Safari")
created_at   TIMESTAMPTZ
last_used_at TIMESTAMPTZ
revoked_at   TIMESTAMPTZ
expires_at   TIMESTAMPTZ
```

#### `magic_link_tokens`
```
token_hash   BYTEA PRIMARY KEY   (SHA-256 of token; token itself is single-use, in email only)
account_id   UUID REFERENCES accounts(id)
created_at   TIMESTAMPTZ
expires_at   TIMESTAMPTZ         (15 minutes from creation)
used_at      TIMESTAMPTZ
```

#### `aviaries`
```
id           UUID PRIMARY KEY
account_id   UUID REFERENCES accounts(id) UNIQUE
created_at   TIMESTAMPTZ
```

One aviary per account. `created_at` is used for aviary-age-gated bird additions.

#### `birds`
```
id                  UUID PRIMARY KEY
aviary_id           UUID REFERENCES aviaries(id)
species_id          TEXT              (references static species pool; not a FK)
name                TEXT
adopted_at          TIMESTAMPTZ
mood_state          TEXT              (wary | content | curious | drowsy | alert)
mood_updated_at     TIMESTAMPTZ
personality_vector  JSONB             ({"boldness": 0.42, "social_warmth": 0.61, ...})
last_tick_applied   BIGINT            (tick sequence number; prevents double-apply)
```

Personality vector is JSONB — the simulation service is the only writer. All five traits normalized to [0.0, 1.0]. Seed values are species-specific with small random noise at adoption.

#### `interaction_events`
```
id           BIGSERIAL PRIMARY KEY    (monotonic, used for tick watermark)
aviary_id    UUID REFERENCES aviaries(id)
bird_id      UUID REFERENCES birds(id)   (nullable for aviary-level events)
event_type   TEXT      (offer_seed | offer_song | offer_pool | listen_in_start | listen_in_end | settle | presence_ping)
metadata     JSONB     (event-type-specific; e.g. offer_accepted: true, duration_seconds: N)
occurred_at  TIMESTAMPTZ
processed_at TIMESTAMPTZ     (null until consumed by simulation tick)
```

Append-only. The simulation service sets `processed_at` after consuming a batch. The API server never updates this table — only appends.

#### `presence_windows`
```
id           UUID PRIMARY KEY
aviary_id    UUID REFERENCES aviaries(id)
started_at   TIMESTAMPTZ
ended_at     TIMESTAMPTZ    (null while open; set on settle / tab-close signal / session timeout)
duration_s   INTEGER        (computed on close; null while open)
```

Presence windows are opened by the first `presence_ping` after a visibility+focus+activity conjunction and closed when any condition breaks. Duration is the primary drift input.

#### `notebook_entries`
```
id           UUID PRIMARY KEY
aviary_id    UUID REFERENCES aviaries(id)
body         TEXT           (naturalist prose; lowercase, present-tense, specific)
generated_at TIMESTAMPTZ
```

Generated by the simulation service or a triggered generation path. Read-only from the client.

#### `visit_invitations`
```
id              UUID PRIMARY KEY
aviary_id       UUID REFERENCES aviaries(id)
visitor_email   TEXT            (plaintext; not used as identifier; used only for delivery)
token_hash      BYTEA           (SHA-256 of one-time token)
created_at      TIMESTAMPTZ
expires_at      TIMESTAMPTZ     (30 days from creation)
revoked_at      TIMESTAMPTZ
first_used_at   TIMESTAMPTZ
```

#### `visit_events`
```
id              UUID PRIMARY KEY
invitation_id   UUID REFERENCES visit_invitations(id)
started_at      TIMESTAMPTZ
ended_at        TIMESTAMPTZ
duration_s      INTEGER
```

Logged for the host's visit log. Not used as drift input.

#### `tick_runs`
```
id                 BIGSERIAL PRIMARY KEY
started_at         TIMESTAMPTZ
completed_at       TIMESTAMPTZ
last_event_id      BIGINT      (watermark: highest interaction_events.id processed)
aviaries_processed INTEGER
status             TEXT        (running | success | failed)
```

Allows the simulation service to resume from the correct watermark after a restart. Prevents double-processing events.

---

## 4. API Surface

All endpoints require a valid session token in the `Authorization: Bearer` header, except the auth endpoints and the visitor-access endpoint.

### Authentication

**`POST /api/auth/request-link`**
- Body: `{ "email": "..." }`
- Rate-limited per email (max ~5 requests/hour). Returns 204 regardless of whether email exists (prevents enumeration).
- Side effect: generates a magic link token, stores `token_hash`, emails the link.

**`GET /api/auth/verify?token=<token>`**
- Validates the token (not expired, not used, hash matches). Marks it used. Creates a session, returns a session token as an `HttpOnly` cookie + JSON body.
- On failure: 401 with matter-of-fact error body.

**`DELETE /api/auth/session`**
- Revokes the caller's current session immediately.

**`GET /api/auth/sessions`**
- Lists the caller's active sessions (id, device_label, created_at, last_used_at).

**`DELETE /api/auth/sessions/:id`**
- Revokes a specific session (even from a different device).

### Aviary state

**`GET /api/aviary/state`**
- Returns the current canonical aviary snapshot. Response:
  ```json
  {
    "snapshot_at": "<ISO UTC>",
    "server_timestamp": "<ISO UTC>",
    "birds": [
      {
        "id": "<uuid>",
        "name": "Pip",
        "species_id": "warbler",
        "mood": "content",
        "perch_zone": "front",
        "plumage_saturation": 0.61,
        "call_phase": 0.33,
        "animation_pose": "preening"
      }
    ],
    "aviary": {
      "day_phase": "morning",
      "weather": null,
      "active_weather_since": null
    },
    "unread_notebook_count": 2
  }
  ```
- The `plumage_saturation` scalar is the only personality-derived value exposed; it feeds the visual renderer only and carries no numeric label in the UI. The client never exposes this number to the user.
- Cacheable at the edge for ~5 seconds (private, per-account cache key = UUID, never email).

**`GET /api/aviary/state` (visitor)**
Visitor access uses a different path: `GET /api/visit/:token/state`. Validates the invitation token (not revoked, not expired). Returns the same snapshot shape. Does not write interaction events. Does not touch presence windows. Returns 410 if invitation is revoked.

### Interaction events

**`POST /api/events`**
- Body: array of one or more event objects:
  ```json
  [
    { "type": "presence_ping", "occurred_at": "<ISO UTC>" },
    { "type": "listen_in_start", "bird_id": "<uuid>", "occurred_at": "<ISO UTC>" },
    { "type": "offer_seed", "bird_id": "<uuid>", "occurred_at": "<ISO UTC>" }
  ]
  ```
- Appends to `interaction_events`. Returns 204.
- The client batches events and flushes on an interval (~30s) or on tab hide/settle.
- Clients send `occurred_at` from their local clock; the server records this alongside `server_received_at`. The simulation tick uses `occurred_at` for ordering, with a reasonable skew tolerance (±5 minutes).

### Field notebook

**`GET /api/notebook?before=<cursor>&limit=50`**
- Returns paginated notebook entries in reverse chronological order.
- Response: `{ "entries": [{ "id": "...", "body": "...", "generated_at": "..." }], "next_cursor": "..." }`

### Visit invitations

**`POST /api/invites`**
- Body: `{ "visitor_email": "..." }`
- Creates an invitation, emails the visitor a one-time link. Returns `{ "id": "<uuid>", "expires_at": "..." }`.

**`GET /api/invites`**
- Lists the host's invitations and the visit log: `{ "invitations": [...], "visit_log": [...] }`.

**`DELETE /api/invites/:id`**
- Revokes an invitation immediately. If a visitor is currently viewing, their next state-snapshot pull returns 410.

### Account

**`GET /api/account`**
- Returns: `{ "email_hint": "m***@example.com", "created_at": "...", "session_count": N, "visit_notifications_enabled": false }`

**`PATCH /api/account`**
- Body: `{ "visit_notifications_enabled": true }` (and future settings fields)

**`POST /api/account/export`**
- Triggers async generation of a JSON export; emails a download link to the verified address within a few minutes. Returns 202.

**`DELETE /api/account`**
- Initiates soft delete: sets `deleted_at`, schedules `hard_delete_at = deleted_at + 30 days`. Returns 204.
- Any subsequent authenticated request within the 30-day window surfaces a recovery prompt; an explicit `POST /api/account/recover` cancels the deletion.

---

## 5. Simulation Engine Design

### Tick loop

The simulation service runs a tick loop on a ~60-second cron. Each tick execution:

1. **Acquire lock**: use a database advisory lock (or distributed lock if the service scales out) to ensure only one tick runs at a time. If the previous tick is still running, skip this fire.
2. **Read watermark**: load the highest `processed_at` event ID from `tick_runs` (last successful run).
3. **Fetch unprocessed events**: `SELECT * FROM interaction_events WHERE id > :watermark ORDER BY id ASC LIMIT 5000`.
4. **Group by aviary**: partition events into per-aviary buckets.
5. **For each aviary, in parallel**: compute drift deltas and mood transitions (see below). Write updated personality vectors and mood states. Mark events as processed.
6. **Run notebook generation**: for aviaries with notable state changes, generate or queue a notebook entry (see Notebook generation).
7. **Write tick_run record**: record `completed_at`, new `last_event_id`, status.
8. **Release lock**.

The tick is idempotent: if it crashes mid-run, the watermark hasn't advanced (the tick_run record has `status = running`), so the next tick reprocesses from the same watermark. Event processing must be idempotent per aviary (re-applying the same delta set to the same initial vector produces the same result).

### Drift function

For each bird, the drift function takes:
- `presence_delta_seconds`: total presence-window duration from events in this tick batch attributed to this bird's aviary
- `listen_in_seconds`: total listen-in duration for this specific bird
- `offer_accepted`: boolean (at least one accepted offer for this bird in this tick batch)
- `offered_near`: boolean (at least one offer event for this bird regardless of acceptance)
- Current personality vector

It computes additive deltas for each trait:

```
Δboldness          = w_presence * presence_delta * rate_boldness
                   + w_offered_near * offered_near_flag * rate_boldness_offer

Δsocial_warmth     = w_presence * presence_delta * rate_warmth
                   + w_listen_in * listen_in_seconds * rate_warmth_listen

Δvocal_frequency   = w_presence * presence_delta * rate_vocal
                   + w_listen_in * listen_in_seconds * rate_vocal_listen

Δplumage_saturation = w_presence * presence_delta * rate_plumage

Δcuriosity         = w_presence * presence_delta * rate_curiosity
                   + w_offer_accepted * offer_accepted_flag * rate_curiosity_offer
```

All weights (`w_*`) and rates (`rate_*`) are configuration values, not hardcoded. The calibration target:
- After ~1 week of regular visits (~20 minutes/day presence), instruments should detect drift of ~0.02–0.04 on the dominant trait
- After ~3 weeks, visual drift is perceptible to the user (~0.08–0.15 delta on boldness/plumage)
- A single session should never move any trait by more than 0.005

Deltas are clamped: `new_value = min(1.0, old_value + delta)`. Traits never decrease. A configuration flag (`drift_enabled = true/false`) allows disabling drift in test environments.

The drift rates are in a configuration file loaded at service startup. The calibration process during build is: run the simulation against a simulated user-presence schedule (30 daily visits, each with 15 minutes presence), measure the resulting vector after 7 days, adjust rates, repeat.

### Mood transition engine

Mood is a discrete state machine. For each bird, the transition engine evaluates:

**Time-of-day input** (derived from the aviary's account timezone, or UTC if unknown):
- 5am–9am: +weight toward `alert`
- 9am–5pm: +weight toward `content`
- 5pm–8pm: +weight toward `drowsy`
- 8pm–5am: +weight toward `drowsy`/sleeping

**Interaction inputs** (from events in this tick window):
- Offer accepted → +weight toward `content` or `curious` (depending on offer type)
- Offer ignored → no change
- Listen-in → +weight toward `alert` or `content`
- Settle gesture → +weight toward `drowsy`

**Ambient events**:
- Rain active → -weight on `alert`, +weight on `content`/`drowsy`
- Wind active → +weight on `alert` for some species; +weight on `wary` for others (species-specific config)
- Nearby bird alarm call (wary bird with high vocal_frequency in same aviary) → +weight toward `wary` for neighboring birds

**Personality modulation**:
- High `boldness` → resistance to entering `wary` (multiply wary-transition weight by `1 - boldness * 0.5`)
- High `curiosity` → increased probability of entering `curious`

The transition is a weighted random draw from the current state's allowed transitions. Transition weights are summed across all inputs, then normalized. The draw is seeded with the bird ID + tick timestamp (deterministic given inputs; reproducible).

Mood state is stored as TEXT in `birds.mood_state`. Mood is updated at every tick even if no transitions occur (the time-of-day input ensures gradual drift toward drowsy in the evening).

### Call grammar runtime

The call grammar is not executed by the simulation service — it is a client-side WebAudio system. However, the server provides the parameters that drive it. The state snapshot includes `call_phase` per bird: a floating-point value [0.0, 1.0] representing where the bird is in its current call cycle. The client uses this to start the call-synthesis engine in mid-cycle, so calls appear to have been in progress before the user opened the tab.

Call grammar parameters per species (stored in a static config file loaded by the client):
```json
{
  "species_id": "warbler",
  "motif_library": [
    { "id": "A", "intervals": [0, 2, 4], "durations": [0.12, 0.08, 0.15], "base_pitch_hz": 3200 },
    { "id": "B", "intervals": [0, -1, 3], "durations": [0.20, 0.10, 0.10], "base_pitch_hz": 2900 }
  ],
  "sequence_rules": {
    "typical": ["A", "A", "B", "pause", "A"],
    "alert": ["A", "B", "A", "B"],
    "wary": ["A", "pause", "pause", "A"],
    "drowsy": ["B", "pause", "pause"]
  },
  "timing_jitter_ms": 40,
  "pitch_variation_cents": 80,
  "base_call_interval_s": 8
}
```

Client-side synthesis (see Audio Pipeline section) uses these parameters plus the bird's current `vocal_frequency` trait (scales `base_call_interval_s`) and current mood (selects sequence rule).

### Notebook generation

Notebook entries are generated by a notebook generation process that runs as part of the simulation tick (or as a separate post-tick pass). Entry triggers:

- **First-greeter flip**: the bird that greeted first today is different from the bird that greeted first in the previous observed session → generate entry "pip greeted before wren today, first time this week."
- **Extended quiet**: presence-time in the window exceeded 10 minutes with no interaction events other than presence pings → generate entry about quiet observation
- **Mood persistence**: a bird has been in the same mood for 3+ consecutive ticks → generate entry about the sustained state
- **Significant plumage drift**: plumage_saturation crossed a 0.1 threshold in the past week → naturalist observation about the bird's appearance
- **Offer notable response**: an offer was both made and resulted in a mood transition → entry about the bird's reaction

Entries are generated using a template library with slot-filling — not free-form LLM generation in v1. Each template is a naturalist-voice sentence with variable slots (bird name, perch position, time of day, mood-shaped description). Templates are stored in a config file and can be updated without a deploy.

Sparsity gate: the notebook generation pass checks the most recent entry's `generated_at` before generating a new one. No entry is generated if the most recent entry is less than 18 hours old (for a regularly-visited aviary). Notable events (first-greeter flip) can override the gate; routine observations cannot.

---

## 6. Sync Model

### Canonical state propagation

There is no sync problem in the conventional sense, because there is no client-side canonical state to sync. Both a laptop and a phone see the same aviary because both pull from the same server record.

**Path for "laptop in the morning, phone at lunch"**:
1. Laptop session: user opens the tab, pulls state snapshot, watches for 15 minutes, writes presence pings + one offer event to the event log, closes the tab.
2. Simulation tick runs: processes the laptop session's events, updates personality vectors and mood.
3. Phone session: user opens the tab, pulls state snapshot. The snapshot reflects the state after step 2 — including the personality drift from the laptop session.
4. No merge required. No conflict.

### Conflict prevention

The "no last-write-wins" rule is enforced architecturally:
- The simulation service is the only process that calls `UPDATE birds SET personality_vector = ...`
- All API server writes to interaction_events are INSERT-only
- The tick's watermark-based processing ensures events are consumed in order without double-application

Clock skew handling: client-submitted `occurred_at` timestamps are accepted with a ±5-minute tolerance window. Events outside that window are discarded with a log warning (not surfaced to the user). Events are ordered by `interaction_events.id` (insertion order, which is insertion-time-server-assigned) for tick processing, using `occurred_at` for intra-session ordering only.

### State snapshot freshness

The client pulls a fresh snapshot:
- On initial tab open (first load)
- On `visibilitychange` event (`document.visibilityState === 'visible'` transition)
- On `focus` event after >5 minutes of absence
- On a low-frequency keepalive poll (~60s interval while the tab is visible and focused)
- On a forced refresh after a connection error

The keepalive interval matches the tick cadence; the client will typically see the result of the most recent tick within ~60–90 seconds of it completing. This is intentional — per-second updates would be wasteful and per-minute updates match the slow rhythm of the product.

### Visitor sync

A visitor session pulls state snapshots on the same keepalive cadence as an owner session. A revoked invitation is detected on the next snapshot pull (the API returns 410). There is no push mechanism; the visitor poll interval is the revocation propagation delay (max ~60 seconds).

---

## 7. Frontend Rendering Pipeline

### Technology choice

The frontend is a single-page application built with a modern reactive framework (React or Svelte — to be finalized in implementation). The aviary scene is rendered on an HTML5 Canvas (primary) with SVG fallback for systems where Canvas is unavailable. CSS transitions handle top-bar fade and day/night palette shifts. WebAudio handles all sound.

### Scene composition

The scene is composed in layers (back to front):
1. **Sky/background**: CSS gradient, shifts with day/night phase using CSS custom properties updated by a `requestAnimationFrame` loop keyed to local time
2. **Background foliage**: SVG layer; subtle parallax on mousemove (max 8px offset; respects `prefers-reduced-motion`)
3. **Perch structures**: static SVG; three zones (back, middle, front) with appropriate depth blur
4. **Birds**: one canvas sprite per bird, positioned at their perch zone; each bird is animated independently
5. **Weather overlay**: Canvas or CSS particle system; rain drops generated client-side when `weather: "rain"` is active in the snapshot
6. **Foreground branch/leaf**: occasional pass-through of ambient leaf/feather as Canvas particles; generated by a bounded client-side idle timer
7. **Call captions**: absolutely positioned DOM text nodes near the calling bird, faded in/out with CSS transitions

### Bird sprites and animation

Each species has a sprite sheet (compact SVG or PNG sprite, aiming for <50KB per species) with the following pose sets:
- `idle_perch`: 2–3 standing poses
- `preen`: 3–4 preening-motion frames
- `scan`: 2 scanning poses (looking left/right)
- `head_tilt`: 1 tilted pose
- `fluff`: 1 fluffed pose
- `flight_depart` and `flight_arrive`: 3–4 frames each (used for perch transitions)
- `drink`, `bathe`: for still-pool offer reaction

Animation is driven by a per-bird animation state machine, keyed to the bird's current mood and the simulation state:
- **content**: cycles preen → idle → idle → preen at slow random intervals
- **wary**: cycles scan → idle → scan; never front perch (positions itself to middle/back)
- **curious**: head_tilt frequently; responds to ambient sounds
- **drowsy**: fluff pose; minimal movement; slow blink cue (single-frame opacity dip)
- **alert**: scan frequently; can be on any perch zone

The animation loop runs at 60fps via `requestAnimationFrame`. Each bird's frame update advances its animation state machine by `deltaTime`. On reduced-motion, the state machine selects a target pose and cross-fades to it over 1.5–3 seconds rather than animating intermediate frames.

### Idle micro-motion

Between animation state transitions, birds exhibit sub-second micro-motion:
- Subtle body sway (±1–2px, 2–3s period) — implemented as a sinusoidal position offset with phase offset per bird
- Feather detail shimmer on plumage (Canvas globalAlpha pulse at very low amplitude, 4–5s period)
- Blinking: single-frame eye-close at random ~8–12s intervals

These micro-motions do not pause when the tab loses focus; they are driven by the animation loop which halts when the document is hidden. On tab-restore, the animation loop resumes from current simulation time, so micro-motion appears continuous.

### First frame (loading path)

On navigation to the aviary:
1. Load the SPA bundle (code-split: aviary scene module is the critical path; account settings and visit-flow modules are lazy-loaded).
2. Issue `GET /api/aviary/state` immediately. This request should resolve within ~100–150ms on a warm edge cache (the state snapshot is small, ~2–4KB).
3. While the snapshot is in flight, render the **quiet field**: sky gradient at the day's current phase, no birds. No spinner. No loading text.
4. On snapshot resolution, place all birds at their current positions and poses (derived from `animation_pose` and `mood` fields). Start the animation loop.
5. The `call_phase` field ensures procedural audio picks up mid-cycle on each bird. Audio context is created on the first user gesture (click, key press) to comply with browser autoplay policies; until then, calls are captioned if captions are enabled.

If the snapshot takes >1 second (slow connection), the quiet field shows one ambient micro-motion cue (a single leaf drifting through at normal speed). This is not a spinner; it is the aviary loading.

### Perch zone transitions

When a bird transitions perch zones (driven by mood change in the state snapshot), the bird plays `flight_depart` from its origin perch, arcs across the scene on a bezier curve computed from origin/destination zone positions, and plays `flight_arrive` at the destination. Flight is fast (~0.6–0.8s), and the arc keeps the bird on screen throughout. On reduced-motion, the transition is a cross-fade of perch positions over 1.5s (the bird fades out at origin, fades in at destination).

### Top-bar fade

The top bar is a `position: fixed` element at the top of the viewport. It uses CSS `opacity` + `pointer-events: none` transition. After 3 seconds of no `mousemove` or `keydown` event on the document, the bar transitions to `opacity: 0.08`. On any `mousemove` or `keydown`, it transitions back to `opacity: 1.0`. Transition duration is 500ms. The bar is always keyboard-focusable (focus makes it opaque); `pointer-events` is restored when opaque.

### Settle animation

When settle is triggered:
1. The lighting shifts to evening palette over 3–4 seconds (CSS custom properties transition).
2. All birds gradually transition to drowsy poses (individual timings staggered by 200–500ms per bird).
3. Call volume ramps down over 5 seconds via master gain node.
4. Top bar remains accessible.

The 5-second undo window: any click in the aviary scene within 5 seconds reverses the CSS transition and ramps audio back up.

### Responsive layout

The aviary scene is a fixed-aspect-ratio canvas scaled to fill the viewport width. On narrow viewports (< 400px), the scene height is reduced to maintain all birds in frame; the three perch zones compress horizontally. The minimum scene width is 280px (below this, the scene renders in a single-column perch stack — a graceful degradation only for very small viewports).

---

## 8. Audio Pipeline

### Architecture

The audio pipeline runs entirely in the browser via the Web Audio API. No audio files are downloaded.

```
AudioContext
  └── MasterGainNode
        ├── WeatherGainNode          (0.6 during rain, 1.0 otherwise)
        │     ├── Bird[0] GainNode   (1.0 base; adjusted by listen-in)
        │     │     └── Bird[0] CallSynth (AudioWorkletNode or OscillatorChain)
        │     ├── Bird[1] GainNode
        │     │     └── Bird[1] CallSynth
        │     └── ... (up to 7 birds)
        └── AmbientGainNode          (soft background: wind, rustling)
              └── AmbientNoise (BufferSourceNode, pink noise filtered)
```

### AudioContext lifecycle

The `AudioContext` is created on the first user gesture (click, keydown, or touchstart) to comply with browser autoplay policies. Before first gesture, the audio pipeline is inactive. If the user has call captions enabled, captions display from the first frame regardless of audio state.

The `AudioContext` is suspended when `document.visibilityState` transitions to `hidden` and resumed on `visible`. This prevents background audio drain.

### Call synthesis (CallSynth per bird)

Each `CallSynth` is either an `AudioWorkletNode` (preferred, for efficient custom DSP) or a chain of `OscillatorNode` + `GainNode` + `BiquadFilterNode` (fallback).

Call synthesis algorithm per bird:
1. On each call trigger (driven by the call grammar's timing), select a motif sequence from the species config using the current mood's sequence rule.
2. For each motif in the sequence:
   - Create a short oscillator burst (or worklet-generated waveform) at `base_pitch_hz * (1 + jitter_cents / 1200)`
   - Apply the interval offsets from the motif's `intervals` array as pitch multipliers
   - Apply a short amplitude envelope (attack 15ms, decay per `durations[i]`, release 20ms)
3. Add timing jitter: each inter-note gap is varied by ±`timing_jitter_ms` ms.
4. The next call is scheduled after `base_call_interval_s * (1 / vocal_frequency_normalized) * (1 + random_variation_0.2)` seconds.

The `vocal_frequency` trait (normalized 0–1) scales the call interval: a bird with trait 0.3 calls at half the frequency of a bird with trait 0.8. This is the main behavioral output of that trait in the audio layer.

### Chorus mixing

Each bird's `CallSynth` schedules calls using `AudioContext.currentTime`-based scheduling (not `setTimeout`). Independent scheduling per bird means calls are never phase-locked to each other. The GainNode tree ensures they all mix into the same output. At up to 7 birds, the output is a natural chorus, not a layered loop.

### Listen-in mix

On listen-in engage (user clicks/focuses a bird):
- Target bird's GainNode ramps from 1.0 to 2.2 over 1.2 seconds (exponential ramp via `AudioParam.exponentialRampToValueAtTime`)
- All other birds' GainNodes ramp from 1.0 to 0.25 over 1.5 seconds
- Ambient gain node ramps to 0.5 over 1.5 seconds

On listen-in disengage:
- All GainNodes ramp back to 1.0 over 1.5 seconds
- Ambient GainNode ramps back to 1.0 over 1.5 seconds

The ramp is always gradual — no hard cuts. Even an immediate re-click triggers a smooth ramp from current value.

### Weather audio

A light rain effect is generated as filtered pink noise. The `WeatherGainNode` ramps from 1.0 to 0.6 over 3 seconds when rain starts (dampening bird call volume by 40%). A separate `AmbientRainNode` (filtered BufferSourceNode generating soft rain sound) ramps in over 3 seconds and out over 5 seconds on weather end.

### WebAudio fallback

If `AudioContext` is unavailable or throws on construction (older browser, permission denied):
- `audio_available = false` is set
- Call captions are auto-enabled (user setting overridden to "on"; the user can turn them off manually)
- The audio pipeline section of the component tree renders a non-functional but accessible "audio unavailable" notice in accessibility settings
- The rest of the product functions normally

No recorded audio fallback. Silence with captions is the complete fallback.

### Memory management

Audio buffers are reused: per-bird CallSynth nodes are persistent across calls; they do not create new OscillatorNodes per call. Instead, they maintain a small pool of 2–3 oscillators per bird, releasing and reconnecting as needed. The CI memory-growth test asserts that `performance.memory.usedJSHeapSize` does not grow by more than 5MB over a 30-minute synthetic session.

---

## 9. Accessibility Surfaces

### Screen-reader narration

A visually hidden `<div role="status" aria-live="polite" aria-atomic="true">` element at the top of the DOM receives naturalist prose updates on a slow cadence.

**Narration update schedule**:
- Idle: one update every 45 seconds (mid-point of the 30–60s spec range)
- On user-initiated events: return-greeting, offer reaction, settle — update within 1–2 seconds of the event
- On weather start/end: update within 5 seconds

**Narration prose generation**: same template library as the field notebook, sharing the naturalist voice. Templates are prose fragments, not state-dump strings. Examples:
> "pip is on the front perch, calling softly. wren sits further back with feathers fluffed."
> "a soft rain is passing through the aviary. the birds have quieted."
> "wren noticed you and called once from the back perch."

The narration service (client-side, reads from the same snapshot the visual renderer reads) selects templates based on the current per-bird state and recent events. It does not expose mood state as a label; it describes observable behavior ("feathers fluffed," "calling softly," "watching from a distance").

**Priority bumping**: user-initiated event narrations use `aria-live="assertive"` on a separate priority region to ensure they are read without waiting for the polite queue.

### Call captions

Call captions are positioned near the calling bird (absolutely positioned DOM elements, updated via the same state snapshot). Each caption is a short naturalist description of the call generated from the call grammar state at synthesis time:

Caption generation algorithm:
- Read the motif sequence and mood that drove the synthesis
- Map to a template: `"a soft three-note rise"`, `"a low trill, paused, low trill again"`, `"a single sharp call from the back perch"`
- Display for the duration of the call plus 500ms, then fade out over 300ms via CSS opacity transition
- Maximum 2 captions visible simultaneously (oldest dismissed if a third bird calls)

Captions are off by default. The user enables them in accessibility settings. The WebAudio fallback auto-enables captions.

### Keyboard navigation

```
Tab          → moves through top bar items: [account] [accessibility] [notebook] [offer] [settle]
Tab (in bar) → enters aviary scene (focuses first bird)
←/→ arrows   → moves focus between birds in scene order (left to right by perch position)
Enter        → toggles listen-in on focused bird
Escape       → exits listen-in; returns focus to aviary container
Tab (in offer panel) → navigates offer options: [seed] [song fragment] [still pool] [close]
Enter (on offer)     → triggers the offer
```

Focus indicators: a 2px `outline` in a color chosen to contrast against both the lightest and darkest aviary backgrounds (~4.5:1 against the mid-range palette). The visual designer finalizes the exact color; the constraint is enforced in the design token system.

### Reduced-motion mode

Detected via `window.matchMedia('(prefers-reduced-motion: reduce)')`. The accessibility settings UI provides a manual override toggle (stored in account settings, synced).

In reduced-motion mode:
- The animation state machine switches to `cross_fade` mode: instead of advancing animation frames, it selects a target still pose and applies a CSS `opacity` transition between the current frame and the target frame over 2 seconds
- The `requestAnimationFrame` loop still runs at full rate (for smooth cross-fade timing), but draws only at cross-fade keyframes
- Perch-zone transitions use cross-fade instead of flight arc
- Ambient leaf/feather drift is removed entirely
- Day/night palette shifts continue (no animation content; purely color)
- Weather particle effects are replaced by a color-tone overlay (a slight blue shift in the CSS gradient during rain)
- All audio runs normally in reduced-motion mode

Reduced-motion mode ships at launch alongside the full-motion mode. It is not a post-launch fix.

### WCAG AA contrast

Every text element in the product — top bar labels, settings, account surfaces, error messages, notebook entries, call captions, screen-reader narration prose — meets WCAG AA (4.5:1 for normal text, 3:1 for large text). The aviary scene does not contain text except call captions and the top bar. Contrast ratios are encoded in the design token system and tested as part of CI.

---

## 10. Performance Budgets and Observability

### Budgets

| Metric | Budget | Measurement |
|---|---|---|
| Initial JS bundle (gzipped) | < 2 MB | Webpack bundle analyzer in CI |
| Time to first bird visible | < 500ms (mid-tier mobile, 4G) | Synthetic Lighthouse, RUM p75 |
| Idle frame rate | ≥ 60fps (5-year-old mid-range laptop, 30min session) | Synthetic Chrome DevTools, CI Puppeteer test |
| Memory growth over 30min | 0 (< 5MB drift allowed) | CI Puppeteer `performance.memory` assertion |
| Simulation tick p99 latency | < 5s alarm threshold | Server-side metric, alarm in monitoring |
| State snapshot response p99 | < 200ms | Server-side metric, alarm at 500ms |

### Bundle architecture

The initial bundle (< 2MB gzipped) contains:
- Core SPA framework (~100KB)
- Aviary scene renderer (Canvas API code)
- Bird animation state machine
- Audio synthesis engine (WebAudio worklet code)
- Call grammar config (static JSON for all 6 species; ~15KB)
- Presence accounting logic
- API client (fetch wrapper)
- Auth module

Lazy-loaded (code-split):
- Account settings module (auth management, export, deletion, email change)
- Accessibility settings module
- Visit invitation flow
- Field notebook scroll component (lazy-loads entry history beyond the first 10 entries)

Bird species assets (SVG sprites) are loaded on demand: the two starter species are bundled in the initial load; additional species SVGs are fetched on first sighting.

### Time-to-first-bird path

Critical path optimization:
1. HTML shell is served with `<link rel="preload">` for the critical JS chunk and an inlined `<script>` that issues the state snapshot fetch immediately (before JS hydration)
2. The inlined fetch stores the response in a global variable; the main bundle reads it on mount without a second round trip
3. The aviary scene module renders as soon as the snapshot is available; no dependency on lazy-loaded modules
4. No web fonts in the critical path (system font stack or preloaded woff2 with `font-display: optional`)

The quiet-field fallback (while snapshot is in flight) is CSS-only: no JS execution required to show it. This ensures something appears immediately even on slow JS parse.

### Observability

**Synthetic monitoring**: a fleet of automated browsers (Puppeteer/Playwright) runs against production from multiple geographies on a 15-minute schedule. Metrics collected: TTFB, time-to-first-bird-visible, first-frame interactive, 60fps compliance over 5 minutes, audio context initialization time.

**Real User Monitoring (RUM)**: aggregate-only, no per-account dimension. Collected metrics:
- Page load timing (navigation timing API)
- First bird visible (custom PerformanceObserver mark)
- Render frame timing (rAF delta histogram, binned by fps range)
- Audio context init success/failure counts
- Simulation tick latency (server-side, from tick start to `completed_at`)
- State snapshot latency (server-side p50/p99)
- Event submission error rate

**Privacy boundary**: RUM telemetry never includes per-account identifiers, bird names, personality vector values, or interaction event content. The telemetry pipeline is architecturally separate from the simulation database. RUM events carry only anonymous session IDs (rotating per session; not linkable to account UUIDs) and aggregate metric values.

**Simulation tick SLO**: tick p99 latency is alarmed at 5 seconds. Normal tick latency (processing a few hundred events across ~1000 aviaries) should be well under 1 second. The 5-second alarm catches degradation well before users notice the aviary "running slow" (mood transitions delayed, drift not applied to recent sessions).

**Error budget**: <0.1% of state snapshot requests may return 5xx. The alarm fires at >0.5% over a 5-minute window.

---

## 11. Rollout

### Phase 1 — Internal testing (week 1–2 post-implementation-complete)

- Team accounts only; 10–20 aviaries
- Two starter birds per aviary
- Drift and mood systems enabled; calibration reviewed daily
- Audio quality review by at least 3 team members with headphones
- Accessibility review: NVDA + Chrome, VoiceOver + Safari, keyboard-only navigation
- Performance: run the full synthetic budget suite against staging

### Phase 2 — Invite-only beta (weeks 3–6)

- ~200 external users via email invitation
- Monitor drift calibration: pull personality vector distributions across the beta cohort weekly; verify the 1-week/3-week targets are met
- Monitor audio: collect audio-context error rates; review any "sounds canned" feedback from beta users
- Monitor sync: verify that users with 2+ devices see consistent state; watch for any clock-skew outliers in interaction event timestamps
- Notebook entry quality review: sample 50 generated entries per week; gate on "feels like a naturalist observation" vs. "feels like a state dump"
- Accessibility: recruit 2 screen-reader users specifically; collect qualitative feedback on narration voice and pacing
- No gamification surfaces are permitted in beta even as "test features"

### Phase 3 — Public launch

Prerequisites:
- All performance budgets met in synthetic monitoring
- Drift calibration validated over the beta period
- Audio quality validated: no reported "loop sounds" from beta users
- Accessibility review complete; WCAG AA pass on all surfaces
- Screen-reader narration reviewed and validated by users

Launch:
- Public sign-up via the magic-link flow
- All new accounts start with two starter birds
- Third-bird offer age-gated to aviary age ≥ 90 days (configurable)
- No marketing email; no push notifications; no announcement banners

### Bird-per-aviary ramp

New bird additions are age-gated, not interaction-gated:
- Aviary age 0–89 days: 2 birds
- Aviary age 90–179 days: eligible for 3rd bird (system surfaces an offer; user accepts or declines)
- Aviary age 180–364 days: eligible for 4th bird
- Aviary age 365+ days: eligible for 5th bird; 6th and 7th at 18 months and 24 months respectively

These thresholds are stored in a configuration table and can be adjusted without a deploy. The offer is surfaced as a quiet notification in the field notebook (a naturalist note that a new bird has appeared at the edge of the scene), not as a modal or announcement.

### Day-1 instrumentation

From the first day of public launch, the following are instrumented:
- State snapshot request rates and latencies
- Interaction event submission rates (type breakdown: offers, listen-ins, presence pings, settles)
- Simulation tick cadence, latency, error rate
- Session creation rates (new accounts, returning accounts)
- Visit invitation creation and redemption rates
- WebAudio context failure rate
- Account deletion initiation rate

None of this instrumentation touches per-bird or per-account interaction content. It is all aggregate operational health data.

---

## 12. Risks

### Drift calibration mis-tuning

**Risk**: the drift function is miscalibrated — either too fast (birds change noticeably between sessions, Tamagotchi-like) or too slow (three months of visits produce no perceptible change). Both are silent failures: no error is thrown, no test fails, but the product's central promise breaks.

**Mitigation**: the calibration process (simulated-user-schedule test harness against the drift function, run before launch) is treated as a ship-gate requirement. Beta telemetry collects anonymized per-cohort personality-vector distribution snapshots (no per-account data) at 1 week and 3 weeks to validate against the calibration targets. A configuration flag allows adjusting drift rates without a deploy.

**Risk of asymmetric drift getting coded as symmetric by mistake**: a developer familiar with symmetric drift models may introduce a `clamp(0, 1)` on the delta that implicitly allows negative drift. The constraint must be encoded as `max(current_value, current_value + delta)` — not `clamp(0, 1, current_value + delta)`.

### Sync correctness

**Risk**: clock skew between client and server causes interaction events to be processed out of intended order, producing unexpected drift. Edge case: a user submits events from a device with a 10-minute-wrong clock.

**Mitigation**: the ±5-minute skew tolerance window at the API layer. Events outside the window are dropped with a warning (logged but not surfaced to the user). The simulation tick processes events by insertion order (not client-submitted `occurred_at`) for the purpose of drift computation, using `occurred_at` only for intra-session ordering within a single device.

**Risk**: the simulation service crashes mid-tick, partially applying a drift delta. The `last_tick_applied` field on each bird (tick sequence number) and the idempotency of event processing prevent double-apply. A partially applied tick must be detectable via the `tick_runs.status = running` check and the service must be able to roll back a partial write. Implement the tick's per-aviary update as a database transaction.

### Audio uncanniness

**Risk**: the procedural call synthesis sounds mechanical or repetitive despite the variation built into the grammar. Users describe it as "a loop" after a few minutes.

**Mitigation**: the call grammar's pitch variation and timing jitter parameters must be empirically tuned during beta. The specific failure mode to watch for is "interval recognition" — when the same melodic interval is used in the same order too frequently, the brain pattern-matches it as a loop even if the timing varies. A minimum of 3 distinct motifs per species, with randomized selection weights shaped by mood, prevents this. Beta audio review (headphones, 20-minute session, "does this sound alive or canned?") is a ship-gate.

**Risk**: seven birds calling at once produce an indistinct wall of sound rather than a recognizable chorus. The per-bird call signature recognizability degrades above ~5 birds.

**Mitigation**: the 7-bird cap is the spec's empirical ceiling; build the audio mix with the assumption that all 7 may call simultaneously. At 7 birds, the master gain and per-bird gains should be tuned so the chorus is pleasant rather than overwhelming. Evaluate in beta when any aviary reaches 4+ birds.

### Accessibility regressions

**Risk**: a visual design change or animation update inadvertently disengages the screen-reader narration from the visual state (the narration says "pip is preening" while pip is flying).

**Mitigation**: the narration is driven by the same state snapshot as the visual renderer. The template selection uses the same field (`animation_pose`) the renderer uses. A CI test renders a synthetic session headlessly, captures narration updates and renderer state, and asserts they are consistent within the narration-update window (±5 seconds).

**Risk**: call captions fire with a perceptible delay after the call starts, breaking the audio-equivalence claim for deaf users.

**Mitigation**: the caption is scheduled at the same `AudioContext.currentTime` as the corresponding call oscillator onset, not after the call completes. A 200ms client-side scheduling tolerance is acceptable; above 500ms delay is a regression.

### Privacy architecture erosion

**Risk**: a future feature adds per-bird state to an analytics event or a telemetry pipeline, quietly breaking the architecture rule that telemetry never touches per-account simulation data.

**Mitigation**: the telemetry pipeline is a separate service with no read access to the simulation database. This is enforced at the network level (service mesh rules or security group restrictions), not just by convention. Any new telemetry event schema addition must be reviewed against the privacy constraint checklist (checklist lives in the engineering wiki).

### Performance regression on mobile

**Risk**: a new animation or audio feature bumps the bundle over 2MB or the first-bird time over 500ms on a mid-tier device.

**Mitigation**: bundle size and synthetic Lighthouse scores are CI checks, not advisory metrics. A PR that would push the bundle over 2MB gzipped fails CI. The synthetic timing check (Puppeteer with CPU throttling simulating a mid-tier device) is also a CI check with a 500ms threshold. Both checks run on every PR to `main`.

---

## Appendix: Defensible calls on ambiguous PRD points

1. **Aviary-level vs. per-bird presence-time**: the PRD defines presence at the aviary level (not per-bird). All birds in an aviary receive equal presence-time credit during a session. Rationale: the user is watching the aviary, not just one bird; penalizing background birds for not being the one being watched would break the social-system quality the product values.

2. **Notification on opt-in visit alert**: the PRD says the host can opt in to visit notifications via a settings toggle. This plan interprets this as an email notification (not an in-product toast or push), consistent with "the product does not push or ping the user." The user set the toggle; the email honors the intention without introducing a push-notification surface.

3. **Notebook generation frequency**: the PRD says "roughly one entry every few days." This plan gates entries with an 18-hour minimum gap for routine observations, with a bypass for high-signal events (first-greeter flip). During very active use, 18 hours ensures the notebook stays sparse.

4. **Call grammar motif library size**: the PRD doesn't specify minimum motif count per species. This plan specifies a minimum of 3 distinct motifs per species to prevent perceptible looping. Richer libraries (5–6 motifs) are preferred for species the user is most likely to focus on.

5. **State snapshot cache TTL**: the plan sets ~5 seconds of edge caching for state snapshots. This means two devices opening the aviary simultaneously may see a snapshot that's up to 5 seconds stale relative to each other — acceptable given the simulation tick cadence of ~60 seconds.
