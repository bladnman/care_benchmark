# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In scope for v1

- Browser-based virtual aviary (web-only; no native apps)
- Two starter birds per new account, cap at seven birds per aviary; new bird slots unlock by aviary age
- Six species in the initial pool; species selection for starters is system-chosen, not user-configured
- Email + magic-link authentication; per-device session tokens; session revocation from account settings
- Single canonical aviary per account; single-user accounts only
- Multi-device sync via server-authoritative state (no client-side sync negotiation)
- Server-side simulation tick (~1 minute cadence): personality drift, mood transition, ambient events
- Procedural call grammar synthesized client-side via WebAudio API
- Presence accounting: triple-conjunction check (visibilityState visible + window focus + pointer/key activity within calibrated window)
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook
- Field notebook: auto-generated naturalist prose; read-only; sparse cadence (~one entry per few days for regular visitors)
- Day/night cycle keyed to user's local timezone
- Ambient weather events (rain, wind) a few times per week; no assertive weather
- Ambient scene micro-motion: leaves, feathers, parallax layers
- Visit invitation feature (off by default): host invites specific email address; read-only ambient view for visitor; revocable; expires after 30 days unused
- Visit log in account settings; opt-in visit notification toggle (off by default)
- Account export (JSON snapshot on demand, emailed as download link)
- Soft-delete 30-day window; hard delete after window
- Screen-reader narration in naturalist prose, slow cadence
- Reduced-motion mode: cross-fade rendering, not stripped fallback
- Call captioning in naturalist voice, runtime-generated from call grammar
- WCAG AA contrast on all user-copy text
- Full keyboard navigation; focus indicators visible against aviary background
- Performance targets: JS bundle <2MB gzipped, first bird visible <500ms (mid-tier mobile/4G), 60fps idle on 5-year-old laptop, no memory growth over 30 minutes

### Out of scope for v1 (hard boundaries)

- Native iOS/Android apps
- Gamification of any flavor (streaks, levels, badges, scores, green-dot calendar, XP, achievements, "birds adopted: N" counter)
- Any Tamagotchi-style mechanic (hunger, distress, decay-on-neglect)
- Social-network surfaces (profiles, follows, public feed, discovery, friend-of-friend, leaderboard)
- Shared or multi-user aviaries
- Multi-aviary accounts
- Push/email notifications about aviary state
- Payments / paid tiers
- Recorded audio (unconditionally; no fallback path using audio files)
- Per-bird interaction telemetry aggregated for any population-level purpose
- Personality vector exposed to users or external surfaces

---

## 2. Architecture

### Service topology

```
Browser client
  ↕  HTTPS REST + SSE (or WebSocket for low-latency presence pings)
API Gateway / Edge (CDN)
  ├── Auth Service          — magic-link issuance, token validation, session management
  ├── Aviary API            — state snapshots, interaction event ingestion, notebook reads, visit management
  ├── Simulation Worker     — periodic tick process; reads event log, writes canonical state
  └── Notification Worker   — async: email dispatch for magic links, export links, opt-in visit emails
Storage
  ├── Accounts DB           — account records (UUID pk, encrypted email, session tokens)
  ├── Aviary DB             — birds, personality vectors, moods, aviary metadata, visit records
  ├── Event Log             — append-only interaction events (presence pings, offers, listen-in, settle)
  └── Notebook Store        — generated notebook entries per aviary
```

### Client/server split

**Server owns:**
- Canonical aviary state (personality vectors, moods, bird positions at tick boundaries, aviary age)
- Simulation tick computation
- Event log persistence
- Notebook entry generation
- Account and session records
- Visit authorization

**Client owns:**
- Rendering (scene composition, idle micro-motion, transitions)
- Presence detection and local presence-ping batching
- Procedural audio synthesis (WebAudio)
- Interpolation between server-side snapshot positions
- Local time (for day/night cycle rendering)

**Client never:**
- Writes personality vector values
- Computes drift
- Generates mood transitions independently (mood comes from server snapshot; client interpolates visual expression)
- Runs a simulation tick

### Render pipeline boundary

Server delivers state snapshots. Client renders those snapshots with continuous local interpolation. The server tick fires ~once per minute; between ticks, the client drives micro-motion, call synthesis, and idle animation entirely client-side. At each tick boundary the client receives an updated snapshot and reconciles smoothly.

---

## 3. Data Model

### Account

```
account {
  id:            UUID (pk, synthetic, generated at creation)
  email_enc:     encrypted string (stored once; never used as identifier)
  created_at:    timestamp
  deletion_requested_at: timestamp | null
  hard_delete_at: timestamp | null   // 30 days after deletion_requested_at
  visit_notify:  boolean (default false)
}

session {
  id:            UUID (pk)
  account_id:    UUID (fk → account)
  device_label:  string (user-agent derived, display only)
  issued_at:     timestamp
  last_seen_at:  timestamp
  revoked_at:    timestamp | null
}

magic_link {
  token:         UUID (pk, one-time, 15-minute TTL)
  account_id:    UUID (fk → account)
  created_at:    timestamp
  consumed_at:   timestamp | null
  expires_at:    timestamp
}
```

### Aviary

```
aviary {
  id:            UUID (pk)
  account_id:    UUID (fk → account, unique — one aviary per account)
  created_at:    timestamp
  bird_slots:    int (starts 2; max 7; unlocked by aviary age schedule)
}
```

### Bird

```
bird {
  id:            UUID (pk, stable identity — never reused or reassigned)
  aviary_id:     UUID (fk → aviary)
  species_id:    string (from pool of ~6 species keys)
  name:          string (user-assigned; mutable; never affects personality)
  adopted_at:    timestamp

  // Personality vector (server-owned, never exposed to client numerically)
  pv_boldness:         float  // [0.0, 1.0]
  pv_social_warmth:    float  // [0.0, 1.0]
  pv_vocal_frequency:  float  // [0.0, 1.0]
  pv_plumage_sat:      float  // [0.0, 1.0]
  pv_curiosity:        float  // [0.0, 1.0]

  // Mood (fast timescale; persists across sessions)
  mood:          enum(wary, content, curious, drowsy, alert)
  mood_updated_at: timestamp
}
```

Personality vector fields are never returned in client-facing API responses. They are internal to the simulation service.

### Event log (append-only)

```
interaction_event {
  id:            UUID (pk)
  aviary_id:     UUID (fk)
  account_id:    UUID (fk — host account only; visitor events not recorded)
  event_type:    enum(
                   presence_ping,
                   listen_in_start, listen_in_end,
                   offer_seed, offer_song, offer_pool,
                   settle,
                   greeting_received    // recorded server-side on return-greeting fire
                 )
  bird_id:       UUID | null (target bird, if applicable)
  occurred_at:   timestamp
  session_id:    UUID (fk → session)
}
```

Visitor sessions do not write to the event log. The simulation tick only reads events from the host account.

### Presence ping

Clients POST a presence ping to the event log at a batched cadence (~every 60 seconds of active presence). The ping encodes a local measurement window: `{ started_at, ended_at, activity_confirmed: bool }`. The server validates that `ended_at - started_at` is within expected bounds; outliers are capped rather than rejected (network delays happen).

### Notebook entry

```
notebook_entry {
  id:            UUID (pk)
  aviary_id:     UUID (fk)
  generated_at:  timestamp
  prose:         string (naturalist voice, lowercase, present-tense)
  trigger_event: string | null (internal label; not exposed to client)
}
```

Entries are generated by the simulation tick or by a separate notebook-generation worker triggered on noteworthy events. Trigger conditions: a bird has greeted first for the first time in a week, a long quiet period, a bird's plumage saturation has crossed a visible threshold, etc. Rate-limiting: no more than one entry per day for a regularly-visited aviary; a cap is enforced at generation time. Client fetches entries in reverse chronological order; no pagination cap (user can scroll back indefinitely).

### Visit

```
visit_invitation {
  id:              UUID (pk)
  aviary_id:       UUID (fk → host aviary)
  visitor_email:   string (not a registered-user requirement)
  token:           UUID (one-time link token)
  issued_at:       timestamp
  expires_at:      timestamp  // issued_at + 30 days
  accepted_at:     timestamp | null
  revoked_at:      timestamp | null
}

visit_session {
  id:              UUID (pk)
  invitation_id:   UUID (fk)
  started_at:      timestamp
  last_pull_at:    timestamp
  ended_at:        timestamp | null
}
```

---

## 4. API Surface

All endpoints use HTTPS. Auth: session token in `Authorization: Bearer <token>` header, except magic-link endpoints and visit-snapshot endpoint (which uses invite token).

### Auth

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/magic-link/request` | Send magic link to email |
| GET | `/auth/magic-link/consume?token=<uuid>` | Consume token → issue session token, redirect |
| DELETE | `/auth/sessions/:session_id` | Revoke a session |
| GET | `/auth/sessions` | List sessions for current account |

### Aviary state

| Method | Path | Description |
|--------|------|-------------|
| GET | `/aviary/snapshot` | Current state snapshot (see below) |
| POST | `/aviary/events` | Submit interaction event(s) (batched) |
| GET | `/aviary/notebook` | List notebook entries (newest first) |
| PATCH | `/aviary/birds/:bird_id` | Update bird name |

**State snapshot response shape (no personality values):**

```json
{
  "aviary_id": "...",
  "snapshot_at": "ISO8601",
  "tick_at": "ISO8601",          // time of last simulation tick
  "aviary_age_days": 42,
  "day_night_phase": "morning",  // derived server-side from user's timezone offset
  "weather": { "type": "clear|rain|wind", "started_at": "ISO8601", "ends_at": "ISO8601" },
  "birds": [
    {
      "id": "...",
      "species_id": "...",
      "name": "Pip",
      "mood": "content",
      "perch_zone": "front|middle|back",
      "pose": "preening|perched|calling|alert|...",
      "call_motif_seed": 42,      // deterministic seed for client call grammar
      "call_timing_offset_ms": 0, // offset into current call cycle
      "plumage_level": "low|medium|high|rich"  // bucketed; no raw float
    }
  ],
  "next_bird_slot_unlocks_at": "ISO8601|null"
}
```

`plumage_level` is a server-computed bucketed value from `pv_plumage_sat`; the client renders visual richness from it without receiving the underlying float. This is the only personality-derived field in the client API, and it is bucketed specifically to avoid communicating numerical precision.

### Interaction event submission

```json
POST /aviary/events
{
  "events": [
    {
      "event_type": "presence_ping",
      "occurred_at": "ISO8601",
      "window_start": "ISO8601",
      "window_end": "ISO8601"
    },
    {
      "event_type": "listen_in_start",
      "bird_id": "...",
      "occurred_at": "ISO8601"
    },
    {
      "event_type": "offer_seed",
      "bird_id": "...",       // closest or most curious bird chosen client-side; server validates plausibility
      "occurred_at": "ISO8601"
    }
  ]
}
```

Server validates event plausibility (e.g., an offer cannot target a bird not in the aviary; presence window must be ≤ 10 minutes). Invalid events are logged and skipped; the batch is not rejected wholesale.

### Visit invitation flow

| Method | Path | Description |
|--------|------|-------------|
| POST | `/visits/invitations` | Host creates invitation (body: `{ visitor_email }`) |
| GET | `/visits/invitations` | Host lists outstanding invitations + visit log |
| DELETE | `/visits/invitations/:invitation_id` | Host revokes invitation |
| GET | `/visits/join?token=<uuid>` | Visitor redeems invitation → visit session token |
| GET | `/visits/snapshot` | Visitor pulls host aviary snapshot (auth: visit session token) |

The `/visits/snapshot` endpoint is a subset of the owner snapshot; it omits `next_bird_slot_unlocks_at` and any settings. Revocation check happens at snapshot-pull time: if `revoked_at` is set, returns 403 with matter-of-fact body `{"error": "visit_revoked"}`.

### Account management

| Method | Path | Description |
|--------|------|-------------|
| GET | `/account` | Account settings |
| PATCH | `/account` | Update email (requires re-verification) |
| POST | `/account/export` | Trigger JSON export (emailed as download link) |
| DELETE | `/account` | Request soft-delete |
| POST | `/account/restore` | Cancel pending deletion within 30-day window |

---

## 5. Simulation Engine Design

### Tick architecture

The simulation tick is a durable scheduled job running server-side at ~60-second intervals. It is not driven by client connections; it runs continuously. Implementation: a worker process (or serverless scheduled function with appropriate concurrency controls) that holds an advisory lock per aviary during the tick pass. Each tick:

1. Reads the event log for events since the last tick (`WHERE occurred_at > last_tick_at AND aviary_id = ?`).
2. Computes presence-time from presence-ping events (summing validated ping windows; capping any single ping window at 10 minutes; rejecting windows where `ended_at < started_at`).
3. Applies drift deltas (see below) to all birds in the aviary.
4. Transitions moods for each bird (see below).
5. Checks ambient event schedule (weather); schedules next weather event if needed.
6. Generates notebook entries if trigger conditions are met.
7. Writes updated bird records (mood, personality vector, perch zone, pose).
8. Records tick timestamp.

The tick must complete within a p99 budget of 5 seconds (alarm threshold; normal expected runtime is <200ms per aviary). Tick latency is instrumented as an operational metric.

### Drift function

Drift is computed as additive deltas per trait per bird per tick. No client submits absolute trait values. The server applies deltas and clamps results to `[0.0, 1.0]`.

**Drift is monotonic upward (toward expressive). Traits never decrease.**

```
delta_boldness       += k_presence * presence_time_this_tick
                     += k_offer * (seed_offers_near_bird + pool_offers_near_bird)
delta_social_warmth  += k_listen_in * listen_in_duration_this_bird_this_tick
                     += k_presence * presence_time_this_tick * 0.5
delta_vocal_freq     += k_listen_in * listen_in_duration_this_bird_this_tick
                     += k_presence * presence_time_this_tick * 0.3
delta_plumage_sat    += k_presence * presence_time_this_tick
delta_curiosity      += k_offer_accept * (offers_accepted_this_bird)
                     += k_presence * presence_time_this_tick * 0.2
```

Calibration targets:
- `k_presence` and related constants are set such that a bird measurably drifts (>0.01 on any trait in test harness) after ~7 days of regular visits (~30 minutes/day presence), and visibly drifts (plumage level bucket change, perch zone tendency change) after ~21 days.
- A single session (even a long one) must not shift any trait enough to move a bucketed output (plumage_level, etc.). Calibration is verified in a dedicated drift-calibration test suite that simulates synthetic event logs.
- Zero drift on neglect: the delta computation never applies negative deltas. The drift function simply accumulates less if there are fewer inputs — not more wary, not less colorful, just growing more slowly.

### Mood transitions

Mood is a per-bird state from the set: `{wary, content, curious, drowsy, alert}`.

Transition rules (evaluated each tick):

1. **Time of day** (derived from account's last-known timezone offset, sent in client requests): early morning → alert; mid-morning → curious or content; afternoon → content; near dusk → drowsy; night → drowsy or settled (eyes closed, low perch).
2. **Recent interactions this tick:** an offer accepted (bird approached) nudges toward `content`; an alarm-call event from another bird in the aviary nudges toward `wary`; a chorus event (two+ high-vocal-freq birds calling simultaneously) nudges toward `alert` for nearby birds.
3. **Personality vector influence:** high `boldness` bird resists `wary` (probability of entering wary is reduced by `pv_boldness * 0.4`); high `curiosity` bird is more likely to shift to `curious` on ambient events.
4. **Weather:** rain damps toward `drowsy`; wind shifts toward `alert` (high-boldness birds) or `wary` (low-boldness birds).
5. **Mood persistence:** the computed target mood is applied probabilistically, not deterministically — each tick, the bird has a probability of transitioning to the target based on how far away it is. This avoids snap transitions.

Mood after transition is written back to the bird record. Clients receive current mood in the snapshot and render idle animation accordingly without needing to run the transition logic themselves.

### Call-grammar runtime

Each species has a call grammar defined as a small set of motifs (2–5 motifs per species). A motif is a parameter set: base frequency, envelope shape, duration range, pitch variation curve, repetition pattern. The grammar encodes rules for motif combination (e.g., species A opens with motif 1, may follow with motif 2 or 3, rarely ends with motif 4).

The call-grammar parameter sets are authored per species and bundled client-side. The server snapshot provides `call_motif_seed` and timing offsets; the client deterministically initializes the grammar from the seed and renders calls via WebAudio. This makes calls vary procedurally (the seed changes each tick) while remaining recognizably that bird's species (the grammar structure is stable).

Personality-shaped call timing: `pv_vocal_frequency` from the server snapshot is not exposed directly, but the server derives a `call_timing_modifier` (an integer in a small range, e.g. 1–5) that the client uses to scale call frequency within the grammar. A bird with high vocal frequency calls more often and joins choruses more readily; a bird with low vocal frequency may go quiet for minutes.

Bird-to-bird call interaction: the call grammar includes cross-bird trigger events. If bird A is in a `calling` pose and bird B has high `social_warmth`, B has an elevated probability of entering a chorus call within the next 5–15 seconds. This is resolved client-side using the shared deterministic seed from the snapshot; both devices for the same account will produce the same chorus timing, since they draw from the same seed.

### Notebook entry generation

The notebook generation worker is separate from the simulation tick (it runs after the tick, or is triggered by the tick). It evaluates trigger conditions on the updated aviary state:

- First time this week that bird A greeted before bird B → entry
- A bird has been in a single zone (e.g., back perch) for more than 3 consecutive sessions → entry
- Long quiet period (no offers, low presence) detected in event log → entry (but not attributing it to the user)
- A bird's plumage bucket has just changed → entry (written as a naturalist observation, not as "plumage increased")
- A weather event occurred during the user's last session → potential entry
- Aviary age milestone (first month, etc.) → sparse entry

Rate limiting: no more than one entry per 48 hours for a regularly-visited aviary. The worker holds the same account-level advisory lock as the tick; they must not run concurrently.

Prose generation: template-driven with randomized naturalist phrasing, not LLM-generated (to avoid latency, cost, and voice drift). A library of sentence templates per trigger condition, with variable slots for bird name, species-keyed verb, mood-keyed adjective, time-of-day phrase. The output reads specific because the templates are specific, not because they're auto-generated at length.

---

## 6. Sync Model

### Single canonical state

Because the server is the only writer of personality vectors and moods, multi-device sync requires no synchronization protocol between clients. Both clients pull from the same record. There is no client-side state to merge. There is no eventual consistency window.

### How conflict is prevented (not resolved)

The design avoids conflict by construction:

- Personality vector writes: simulation tick only. No client write path exists for personality fields.
- Mood writes: simulation tick only.
- Event log: append-only; no write conflicts possible (concurrent appends from two sessions are both valid and both consumed by the next tick).
- Presence pings: additive by nature; two concurrent sessions (e.g., user left a tab open on laptop while visiting on phone) both submit pings; the server sums them. This is correct: the user was genuinely present on both devices. The tick caps total credited presence-time per window to avoid inflation.
- Notebook entries: single writer (notebook worker); no conflict surface.

The only system-level conflict to handle is concurrent magic-link consumption (two devices clicking the link simultaneously). This is handled with a database-level compare-and-swap on `consumed_at`; only one consume succeeds, the other receives a "link already used" response.

### Snapshot freshness

The client pulls a fresh snapshot:
- On tab becoming visible (`visibilityState` change to `visible`)
- On long render-frame gaps (>5 seconds gap detected, indicating laptop suspension)
- On a low-frequency keepalive while visible (~every 60 seconds, aligned loosely with tick cadence)

Snapshots are small (<10KB JSON for a full 7-bird aviary). CDN-cacheable with short TTLs (~30 seconds); the cache key is per-account, so a fresh snapshot is served quickly to both devices without hitting the database on every pull.

---

## 7. Frontend Rendering Pipeline

### Technology choices

- Framework: React (or Preact for bundle size) with a canvas-based aviary scene using WebGL (via a thin abstraction; Three.js or PixiJS are candidates, evaluated against the 2MB bundle cap)
- Alternatively: SVG + CSS animation for the scene if WebGL bundle cost is prohibitive; benchmark both and choose
- Code splitting: account settings, accessibility settings, notebook, visit invitation flow — all lazy-loaded; only the aviary scene is in the initial bundle
- Bird visual assets: SVG-based sprites, animated with GSAP or raw CSS; procedural plumage color driven by `plumage_level` bucket (HSL shift on the base palette)

### Scene composition

The aviary is rendered as a horizontal scene with three perch zones (front, middle, back) and layered planes (background foliage/sky, mid-plane perches and birds, occasional foreground branch/leaf). Layout is responsive: on narrow viewports the perch zones compress horizontally, maintaining all birds in frame; on wide viewports perch zones expand.

Bird placement at scene start: the client receives `perch_zone` and `pose` from the snapshot. It places each bird at a deterministic position within the zone (seeded by `bird.id` to maintain consistent perch positions across loads without storing per-bird pixel coordinates server-side).

### First-frame rendering

The client must render the first frame with birds already mid-action. Implementation:

1. HTML and initial JS bundle are fetched; the CDN edge pre-loads the state snapshot inline in the HTML response (as a `<script>` tag with a JSON blob, populated by server-side rendering or edge injection). This avoids a separate round-trip for the snapshot on cold load.
2. The client parses the inlined snapshot, immediately places birds at their snapshot-derived positions.
3. Idle micro-motion starts from a mid-motion offset (birds are not placed at a neutral starting pose; they are placed mid-preen, mid-call, mid-scan, using the snapshot's `pose` field and a time-offset derived from `snapshot_at - tick_at`).
4. Calls begin at the timing offset from the snapshot (`call_timing_offset_ms`).
5. No spinner, no fade-in. The first paint includes the aviary.

Loading state (for slow connections where the inline snapshot isn't immediately parseable): a quiet field — muted sky gradient, no spinner, no text — renders as a background-color CSS rule before JS runs. One or two subtle leaf-drift particles begin CSS-animating from static HTML; this gives visual continuity without JS. The word "loading" does not appear.

### Idle micro-motion system

Idle micro-motion runs as a time-driven animation loop on the client, independent of server ticks. Animations are parameterized by mood and species:

- `wary`: bird positioned toward back perch, scanning animations (head turns more frequently), feathers slightly raised
- `content`: preening sequences, slow feather settles, occasional beak-wipe
- `curious`: head-tilt toward sounds and movement, leaning forward, step toward front
- `drowsy`: low perch position, feathers fluffed, rare head-lifts, slow blink cycles
- `alert`: upright posture, scanning, more frequent brief calls

Each mood has a weighted animation playlist; the renderer picks the next animation clip probabilistically from the playlist when the current clip completes. This gives non-deterministic variation within the mood's affective register without requiring server input between ticks.

Between-tick transitions: when a new snapshot arrives with a different `mood` or `perch_zone`, the client cross-fades the animation blend rather than snapping.

### Reduced-motion mode

Triggered by `prefers-reduced-motion` media query or accessibility settings toggle.

- Micro-motion replaced by slow cross-fades between still pose images (one image per pose per species per mood, exported from the animation system)
- Cross-fade duration: 2–4 seconds (tuned so the aviary still reads as alive)
- Flight transitions: instant cross-fade between perch positions, no path animation
- Ambient leaf drift: removed entirely
- Day/night color shifts: retained, slowed by 2x
- Calls and captions: unaffected by reduced-motion mode
- Focus indicators and top-bar transitions: no flash or quick transitions

The reduced-motion rendering path shares all the same data (snapshot, mood, perch zone) as the full-motion path; it is a different renderer for the same state, not a different product.

### Top-bar behavior

The top bar contains four icons: account/settings, accessibility settings, field notebook, and offer affordance. After 3 seconds of cursor stillness, the top bar transitions to ~15% opacity (CSS transition, slow fade). Returns to full opacity on `pointermove` or keyboard activity. Touch devices: tap anywhere to restore full opacity.

### Settle UI

Settle trigger: top-bar icon. On trigger:
1. Start lighting shift toward evening palette (CSS variable transition, 3–5 seconds)
2. Reduce audio mix globally (cross-fade to quieter mix)
3. Birds begin drifting toward drowsy poses

Undo window: 5 seconds after trigger, any click/tap anywhere in the aviary reverses the transition. A small, low-contrast text label near the settle icon reads "undo settle" during the 5-second window, then fades. (This is the only tooltip/affordance inside the 5-second post-settle window; it is not permanent UI chrome.)

---

## 8. Audio Pipeline

### Architecture

All audio is synthesized client-side via WebAudio API. No audio files are fetched or bundled. The motif parameter sets (per species) are JSON, bundled in the initial JS payload, small in size (< 50KB total for 6 species).

### Call synthesis

Each bird's call is generated by a WebAudio node graph constructed from the motif parameter set:

```
OscillatorNode (base frequency, shaped by motif)
  → GainNode (envelope: ADSR from motif parameters)
  → BiquadFilterNode (formant shaping per species)
  → PannerNode (bird's horizontal position in scene)
  → GainNode (per-bird mix level; controlled by listen-in)
  → destination (main mix)
```

Pitch variation: a slight random frequency deviation is applied each time a call fires, within a range defined by the motif. This is the source of procedural variation — the same grammar with slight pitch and timing variations each time.

Timing: calls are scheduled via `AudioContext.currentTime` to avoid jitter. The `call_timing_offset_ms` from the snapshot bootstraps the timer on session start so calls don't all begin simultaneously.

Motif combination: the call grammar defines sequences. The client maintains a per-bird call scheduler state machine that selects the next motif in sequence, fires it at the appropriate time, and schedules the next. Personality's `call_timing_modifier` scales the inter-call interval.

### Chorus mixing

When two or more birds are in calling poses simultaneously, the main mix receives both call streams simultaneously. Because calls are procedural (not looped files), phase-cancellation artifacts are avoided — the two calls have slightly different pitches and timing offsets.

The listen-in interaction re-balances the mix: the focused bird's GainNode raises to 1.0; other birds' GainNodes ramp to 0.15 (not 0, so the aviary remains a place with multiple things happening). Ramp duration: 1.5 seconds. On disengage, all gain nodes ramp back to balanced mix over 1.5 seconds.

### Call captions

Generated at runtime from the motif parameters at the moment a call fires: the caption generator reads the envelope shape, pitch range, and repetition pattern and produces a short prose description (e.g., "a soft three-note rise", "a low trill, paused, low trill again"). Caption text is not stored; it is generated fresh per call. A small set of template rules maps motif parameter ranges to naturalist descriptions. Caption text appears as a small element near the calling bird, fades in with the call onset, fades out after the call ends.

### WebAudio unavailability fallback

If `new AudioContext()` throws or is unavailable:
1. Aviary renders in full visual mode.
2. Captions are enabled by default (system auto-enables them, not the user).
3. A dismissable, matter-of-fact notice in the top bar reads: "audio unavailable — captions on." One-time notice; dismissed on next tab reload.
4. No recorded-audio fallback is attempted.

---

## 9. Accessibility Surfaces

### Screen-reader narration

A live region (`aria-live="polite"`) is updated with naturalist prose on a slow cadence:
- Idle: one prose update every 30–60 seconds
- On user-initiated events (return-greeting, offer reaction, settle): updated promptly (within 2 seconds of the event)

Prose is generated from the same state as the field notebook, using shared sentence templates. The narration is not a state list; it reads as an observer's present-tense notes:

> "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."

The live region is visually hidden (off-screen CSS, not `display:none`). Screen readers announce the narration as it updates. A sighted user does not see a text block.

High-priority events (return-greeting at session start) use `aria-live="assertive"` on a separate live region to ensure prompt announcement.

### Keyboard navigation

Tab order: top bar icons (account, accessibility, notebook, offer) → aviary scene → first bird → subsequent birds (via arrow keys).

Within aviary:
- `Tab` moves focus into aviary, focuses first bird
- `ArrowLeft` / `ArrowRight` moves focus between birds
- `Enter` starts listen-in on focused bird
- `Escape` ends listen-in, returns focus to top bar
- `Tab` from within aviary exits to top bar (or next focusable element outside the aviary, if offer panel is open)

Offer panel: opened from top bar icon; all offer options (seed, song, pool) are keyboard-reachable with Tab/Enter; `Escape` closes the panel.

Focus indicators: a 2px high-contrast outline, rendered as an SVG overlay element on the aviary canvas rather than a DOM focus ring (which would not render over a canvas surface). The outline color is chosen to pass WCAG AA against both bright morning and dim evening aviary palettes.

### WCAG AA contrast

All text in the product (top bar labels, account settings, error surfaces, captions, notebook entries when rendered as text) passes WCAG AA (4.5:1 for normal text, 3:1 for large text). The aviary scene itself contains no user copy except caption text (see above). Aviary caption text passes WCAG AA against the background area it appears over; the caption element has a semi-transparent background scrim if needed.

### Accessibility settings

Accessible from the top-bar accessibility icon. Settings:
- Reduced-motion mode (toggle; reflects and writes `prefers-reduced-motion` preference; also usable as a manual override)
- Call captions on/off
- Screen-reader narration on/off (default: on for screen-reader users detected via JS; off for others)
- Text size: standard / large (applies to captions, top bar labels, and notebook prose)

These settings are persisted server-side per account, not in localStorage, so they follow the user across devices.

---

## 10. Performance Budgets and Observability

### Budgets (build constraints)

| Metric | Budget | How enforced |
|--------|--------|--------------|
| Initial JS bundle (gzipped) | <2MB | Bundle size check in CI; PR merge blocked on violation |
| Time to first bird visible | <500ms (mid-tier mobile, 4G) | Synthetic perf test in CI (Lighthouse or WebPageTest via CLI) |
| Idle frame rate | 60fps sustained | Frame-rate monitor in CI (puppeteer/headless, 5-min run) |
| Session memory growth | 0MB over 30 min | Puppeteer heap snapshot test in CI |
| Simulation tick p99 latency | <5s (alarm) | Server-side metric; p99 5s = alert threshold |

Bundle size strategy:
- Aviary scene core (renderer, audio engine, motif library): in initial bundle
- Account settings, accessibility settings, notebook, invite flow: code-split, lazy-loaded
- No recorded audio; SVG/JSON assets only
- Tree-shake dependencies aggressively; audit with `bundlephobia` before adding any npm package

### What we measure (aggregate only, no per-bird data)

- Request counts and latencies per API endpoint
- Simulation tick duration histogram (p50, p95, p99, max)
- Magic-link consume success/failure rate
- Session start rate (anonymized; no account-level attribution)
- First-bird render timing (RUM; anonymized; session-duration bucketed histogram; no per-account dimension)
- Audio context error rate (count of WebAudio init failures, by browser/OS bucket; no account attribution)
- Render frame timing histogram (aggregate across sessions; no per-account)
- Visit snapshot pull rate (anonymized count; no per-host or per-visitor dimension)

### What we deliberately do not measure

- Per-account interaction history in any telemetry pipeline
- Per-bird trait values (personality vector) in any telemetry
- Visit behavior attributable to a specific host or visitor
- Streak-like visit frequency per user
- Any metric that could reconstruct a user's relationship with their birds

The analytics warehouse has no read access to the Aviary DB (birds, personality vectors, events). This is an architectural rule enforced at the network/IAM level, not a policy.

### Synthetic observability

A fleet of synthetic browser runners (geographically distributed) execute a standard test flow every 15 minutes:
1. Load the aviary URL
2. Measure time to first bird visible
3. Run for 60 seconds; measure frame rate and memory
4. Submit one offer event
5. Report metrics to the observability pipeline

These synthetic runs use dedicated test accounts not in the user population; their interaction events do not affect user drift.

---

## 11. Rollout

### v1 launch sequence

**Pre-launch (internal):**
1. Deploy simulation worker and API services in production (inactive; no user accounts)
2. Populate species pool with all 6 species; validate call grammars and motif libraries against audio quality bar
3. Run drift-calibration test suite: simulate 7-day and 21-day presence schedules; verify measurable drift at day 7 and visible plumage/perch changes at day 21
4. Run synthetic perf suite against production infra; verify all budgets met
5. Full keyboard navigation audit (manual QA with screen reader; automated axe-core scan in CI)
6. Reduced-motion mode QA: verify all birds readable in cross-fade mode

**Soft launch (controlled; closed beta):**
- Invite-only via magic links distributed manually
- Target: 100–500 accounts
- Instrument: first-bird-render p95, simulation tick p99, WebAudio error rate
- Monitor: drift calibration (test harness runs against real event logs from beta accounts, anonymized)
- Bug bar: no p0 (data loss, auth bypass, personality reset) before broader launch

**Open launch:**
- Remove invite gate; anyone can sign up via magic link
- Bird slots remain at 2 starters; no new birds unlocked until aviary age threshold (aviary must be several months old)
- Visit invitation feature on by default for all accounts (host must still send a deliberate invite)

### Birds-per-aviary ramp

New bird slot unlock schedule (aviary-age-based):
- Aviary age < 30 days: 2 birds (starters)
- 30–90 days: 3 birds may be offered
- 90–180 days: up to 4–5 birds may be offered (one at a time, interval-spaced)
- 180+ days: up to 6–7 birds may be offered at age-appropriate intervals

The offer mechanism: the client notices `next_bird_slot_unlocks_at` in the snapshot has passed, and a new bird offer appears as a naturalist-voiced surface in the aviary (not a push notification, not a toast). "a new bird has appeared near the aviary. it is watching from the tree line." The user can name the bird; it enters the aviary with a soft fly-in.

The unlock cadence may be tightened or loosened based on beta data; the schedule above is the initial calibration.

### Day-one instrumentation checklist

- [ ] Simulation tick p99 latency alerting live
- [ ] Magic-link error rate alerting live
- [ ] First-bird render synthetic monitor live in ≥3 geographies
- [ ] WebAudio error rate dashboard live
- [ ] Drift calibration nightly test suite running against production event logs (anonymized; aggregate only)
- [ ] Memory growth CI test in CI pipeline
- [ ] Bundle size CI check in CI pipeline

---

## 12. Risks

### Drift calibration miscalibration

**Risk:** Drift moves too fast (birds change noticeably within a session; aviary becomes Tamagotchi-like) or too slow (users see no change after weeks; product feels static).

**Mitigation:** Named calibration targets with automated test harness. Synthetic 7-day and 21-day event log simulations run in CI. The calibration constants (`k_presence`, `k_listen_in`, etc.) are configuration-driven, not hardcoded; adjusting them does not require a code deploy. Monitor visible-drift complaints in beta; adjust constants before open launch. The test harness is the primary defense — if it's not automated, it will not be run regularly.

### Personality vector data loss

**Risk:** A server outage, failed migration, or bug in the tick worker resets a bird's personality vector. The user notices their bird "doesn't feel right" without being able to name why. The failure is silent.

**Mitigation:** The personality vector table has daily automated backups with point-in-time recovery. Write operations go through a single code path (the tick worker); no other process has write access to personality fields. Before any schema migration touching the bird table, a full backup is required. Alerts fire on any unexpected decrease in a personality trait value (monotonic upward is an invariant; a decrease is a bug signal).

### Sync correctness under concurrent sessions

**Risk:** Two simultaneous sessions (laptop + phone) generate presence pings concurrently; the event log receives events from both; the tick credits more presence than intended.

**Mitigation:** The tick sums presence windows and caps total credited presence per tick cycle (cap = tick interval × 1.1 to allow for clock skew). Two concurrent sessions producing overlapping presence windows are credited once for the overlapping duration, not twice. This is implemented as a windowed deduplication in the tick worker. The cap is a safety valve; the test suite includes a concurrent-session scenario.

### Audio uncanniness

**Risk:** Procedural call synthesis produces calls that are tonally correct but feel mechanical or alien rather than birdlike — the uncanny valley of synthetic audio. If users find the calls unsettling rather than charming, the audio is the spine of the product and the whole affective model fails.

**Mitigation:** The motif parameter sets are authored and tuned by a sound designer, not programmatically generated. Beta testing explicitly solicits "does this sound like a real bird?" feedback. The call grammar includes natural imperfections (slight pitch wobble, timing variation, occasional false-start motifs) that pull the synthesis away from mechanical-sounding precision. If the uncanny-valley problem appears in beta, the mitigation is to increase randomization in the motif variation parameters, not to fall back to recorded audio.

### Accessibility regression

**Risk:** The screen-reader narration voice diverges from the naturalist field-notebook voice over time (different authors, different template libraries). Or reduced-motion mode is not tested on actual hardware with the relevant accessibility setting enabled. Or a release that updates animations breaks keyboard focus indicators.

**Mitigation:** Narration and notebook prose templates share a single template library with shared tone guidelines; they are reviewed together. Reduced-motion mode has a mandatory QA step in the release checklist (tested on actual hardware with `prefers-reduced-motion` enabled; not just emulated via browser devtools). An `axe-core` automated accessibility scan runs in CI on every PR. Focus indicator rendering is tested in the synthetic browser suite (screenshot comparison with a focused-bird state).

### "Notice, never announce" violations in production

**Risk:** A well-meaning contributor adds a small toast, a welcome banner, a "your friend visited!" notification badge, or a streak counter. These are individually harmless-seeming and collectively fatal to the product's affective model.

**Mitigation:** This plan is the primary defense: contributors who read it understand why each refusal is structural. Secondary: a product-reviewer checklist item in the PR template ("does this change introduce any toast, badge, streak, or announcement surface?"). The `non_goals.md` is linked from the contributing guide. Any PR adding a toast or gamification element is blocked pending explicit product-owner sign-off, which exists to be denied.

### Bundle size creep

**Risk:** Incremental addition of dependencies crosses the 2MB initial bundle limit; the time-to-first-bird budget breaks; the "aviary already in motion" conceit fails.

**Mitigation:** CI hard-blocks on bundle size violation. A `bundle-size` job runs on every PR against the current main branch; the diff in bundle size is posted as a PR comment. Every new npm dependency requires a justification comment in the relevant PR. The code-splitting architecture (account settings, notebook, visit flow all lazy-loaded) is established at project start, not retrofitted.

### WebAudio ecosystem fragmentation

**Risk:** A browser update changes WebAudio behavior (timing precision, oscillator behavior, AudioContext restrictions). Calls change in character across browsers. The chorus sounds wrong on some combinations.

**Mitigation:** The synthetic browser suite covers the last two major versions of Chrome, Safari, Firefox, and Edge. The suite includes an audio-behavior smoke test (detects if call timing deviates beyond a threshold from expected). The motif parameters are stored as a versioned configuration; a browser-specific override mechanism is available if one browser's WebAudio implementation requires parameter adjustment.
