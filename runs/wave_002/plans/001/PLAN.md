# Pocket Aviary — Implementation Plan

## 1. Scope

### In v1

- Two starter birds per new account; aviary caps at seven
- Six species in the species pool (one with nightjar-like nocturnal call)
- Magic-link authentication (email only; no passwords, no SSO)
- Single canonical aviary per account
- Multi-device sync via server-canonical state (not client-to-client)
- Server-side simulation tick (~1 min cadence)
- Personality vector drift (5 traits, slow monotonic-toward-expressive model)
- Mood system (5 states: wary, content, curious, drowsy, alert)
- Procedural call synthesis via WebAudio API (no recorded audio)
- Idle micro-motion (mood-shaped, continuous)
- Return-greeting (procedurally varied by absence-length, bird boldness, mood)
- Listen-in interaction
- Offer interaction (seed, song fragment, still pool)
- Settle gesture (opt-in session-end)
- Presence accounting (3-condition conjunction)
- Field notebook (auto-generated naturalist prose, read-only)
- Day/night cycle (user's local timezone)
- Ambient weather (rare, mood-affecting rain and wind)
- Visit invitations (read-only, opt-in, per-invite, off by default)
- Visit log (host-visible, demand-pull only)
- Screen-reader narration (naturalist prose, ARIA live regions)
- Reduced-motion mode (designed cross-fade surface, not disabled animations)
- Call captioning (opt-in, runtime-generated prose)
- Keyboard navigation (full top-bar and aviary scene coverage)
- WCAG AA contrast on all user-copy text
- Account export (JSON on demand, emailed as download link)
- Soft-delete with 30-day recovery window
- Synthetic account UUID (email never used as identifier)
- Performance budgets enforced in CI: bundle <2MB gzipped, first-bird <500ms, 60fps idle, no memory growth over 30 min

### Explicitly out of v1 (see `non_goals.md`)

- Native iOS or Android apps
- Gamification of any form (achievements, streaks, levels, scores, badges, green-dot calendar, XP, rank, tier)
- Tamagotchi mechanics (no hunger, no distress, no happiness-meter decay)
- Social-network surfaces (profiles, follows, public feed, discovery, shared aviaries, comments)
- Push/email notifications (no "your friend visited!" push)
- Customizable aviary scenes
- Multi-aviary accounts
- SSO or password login
- Leaderboards or aggregate public rankings
- Payments or tiering
- Personality vector exposure to users (never shown numerically, never in any surface)

Ambiguous calls this plan makes explicitly:

- The species pool seed values and per-trait initial ranges are implementation-calibrated post-MVP, but the persistence model is fully in place at launch.
- Aviary-age-based third-bird offer timing defaults to ~90 days; this is a server config value, not a hardcoded constant, so it can be tuned without a deployment.
- Presence activity-window default is 3 minutes (longer is better per PRD; this is the starting calibration).

---

## 2. Architecture

### Service topology

```
Browser Client
    ↕  HTTPS (REST + SSE)
API Gateway (stateless, auth-validating)
    ├── Auth Service          (magic-link issuance and consumption)
    ├── Aviary API Service    (state snapshots, event log writes, notebook reads)
    ├── Visit Service         (invite issuance, revocation, visitor token validation)
    └── Account Service       (account CRUD, export, deletion)

Simulation Worker (background, not on request path)
    ← reads:  Interaction Event Log (Postgres append-only table)
    ← reads:  Bird/PersonalityVector/Mood tables
    → writes: PersonalityVector, Mood, AviaryStateSnapshot
    → writes: NotebookEntry queue

CDN / Edge Cache
    serves: static assets, HTML shell, initial state snapshot (per-account, short TTL)
```

### Client/server split

The server is the only source of truth for simulation state. Clients render and submit events; they never compute simulation outputs.

| Concern | Owner |
|---|---|
| Personality vector values | Server only (simulation worker) |
| Mood state | Server only (simulation worker) |
| Canonical bird positions (perch zone) | Server only (simulation worker) |
| Presence-event detection | Client (detects, sends ping to server) |
| Call synthesis | Client (WebAudio, from call-grammar params in snapshot) |
| Bird animation | Client (renders from snapshot state + local clock) |
| Notebook prose generation | Server (on noteworthy event detection by worker) |
| ARIA narration prose | Client (generated from snapshot using same template system as notebook) |

### Render pipeline boundary

The aviary scene is rendered in a Canvas 2D element. HTML/CSS overlays handle the top bar and all modal/overlay surfaces (notebook panel, offer picker, settings). The boundary is:

- **Canvas**: birds, perches, sky gradient, background foliage, parallax layers, weather effects
- **HTML**: top bar (position: fixed/sticky above canvas), notebook slide-in panel, offer picker overlay, accessibility settings modal, account settings modal

This boundary is clean and correct: no DOM diffing overhead in the hot render loop, no Canvas accessibility hackery needed (accessibility layer is HTML).

---

## 3. Data Model

All tables use synthetic UUIDs as primary keys. Email is stored encrypted exactly once, on the `accounts` table. No other table references email.

### accounts

```
id              UUID (PK, generated at creation)
email_encrypted TEXT NOT NULL (encrypted at rest, decrypted only for magic-link send and export)
email_hash      TEXT NOT NULL (HMAC of email, used for duplicate-check only, not as identifier)
created_at      TIMESTAMPTZ
soft_deleted_at TIMESTAMPTZ (NULL = active; non-null = pending deletion)
hard_delete_at  TIMESTAMPTZ (soft_deleted_at + 30 days, enforced by a daily cleanup job)
settings        JSONB (visit_notifications_enabled, etc.)
```

### sessions

```
id          UUID (PK)
account_id  UUID FK → accounts
device_hint TEXT (browser UA for display in session list)
created_at  TIMESTAMPTZ
revoked_at  TIMESTAMPTZ (NULL = active)
last_seen   TIMESTAMPTZ
```

### magic_links

```
token_hash  TEXT (PK; store HMAC of the raw token, never the raw token)
account_id  UUID FK → accounts
expires_at  TIMESTAMPTZ (created_at + 15 min)
used_at     TIMESTAMPTZ (NULL = not yet used)
```

### birds

```
id          UUID (PK, stable for the life of the account)
account_id  UUID FK → accounts
species_id  TEXT (references species pool enum; e.g. "warbler_1", "finch_2")
name        TEXT
created_at  TIMESTAMPTZ
display_order INT (used for left-to-right positioning hint in scene; not user-editable)
```

### personality_vectors

```
bird_id              UUID (PK, FK → birds)
boldness             FLOAT4 NOT NULL  -- range [0,1]
social_warmth        FLOAT4 NOT NULL
vocal_frequency      FLOAT4 NOT NULL
plumage_saturation   FLOAT4 NOT NULL
curiosity            FLOAT4 NOT NULL
updated_at           TIMESTAMPTZ NOT NULL
version              BIGINT NOT NULL (monotonically increasing; optimistic lock for simulation worker)
```

Constraints: all traits in [0,1]. Only the simulation worker writes this table. No client mutation path exists.

### moods

```
bird_id      UUID (PK, FK → birds)
state        TEXT NOT NULL  -- enum: wary | content | curious | drowsy | alert
entered_at   TIMESTAMPTZ NOT NULL
tick_version BIGINT NOT NULL
```

### interaction_events

Append-only. No updates. No deletes until hard account deletion.

```
id              UUID (PK)
account_id      UUID FK → accounts  -- for partition/deletion, never as identifier in cross-account queries
bird_id         UUID FK → birds (NULL for account-level events like settle)
event_type      TEXT NOT NULL  -- offer_accepted | offer_declined | listen_in_start | listen_in_end | presence_ping | settle | return_visit
occurred_at     TIMESTAMPTZ NOT NULL
idempotency_key TEXT UNIQUE  -- client-generated, prevents duplicate submission on retry
metadata        JSONB  -- e.g. {offer_type: "seed", duration_seconds: 42}
processed_at    TIMESTAMPTZ  -- NULL = not yet consumed by simulation tick
```

The simulation worker processes events in `occurred_at` order (ties broken by insertion order). `processed_at` is set by the worker after the tick that consumed the event. This table is the only write surface clients have for simulation state.

### aviary_state_snapshots

```
account_id   UUID (PK)
snapshot_at  TIMESTAMPTZ
payload      JSONB  -- per-bird: {bird_id, mood_state, perch_zone, call_grammar_params, animation_hint}
```

This is a cache, rebuilt after every tick. Clients read this; they never write it. The source of truth is `personality_vectors` + `moods` + the tick's computed positions.

### notebook_entries

```
id          UUID (PK)
account_id  UUID FK → accounts
body        TEXT NOT NULL  -- naturalist prose, lowercase, specific
written_at  TIMESTAMPTZ NOT NULL
event_ref   UUID (references the interaction_event or tick output that triggered this entry; nullable)
```

### visits

```
id                UUID (PK)
host_account_id   UUID FK → accounts
visitor_email_encrypted TEXT NOT NULL
created_at        TIMESTAMPTZ
expires_at        TIMESTAMPTZ (created_at + 30 days)
revoked_at        TIMESTAMPTZ (NULL = active)
visitor_token_hash TEXT UNIQUE (HMAC of one-time visitor link token)
```

### visit_log_entries

```
id          UUID (PK)
visit_id    UUID FK → visits
started_at  TIMESTAMPTZ
ended_at    TIMESTAMPTZ (NULL = ongoing; set on next state pull after revocation or expiry)
```

---

## 4. API Surface

All authenticated endpoints require a session token (Bearer JWT with `account_id` claim). Visitor endpoints use a separate visitor token.

### Authentication

```
POST /auth/request-link
  Body: { email: string }
  Response: 200 (always; never reveal whether email is registered)
  Side-effect: generate magic link, email it to the address

POST /auth/verify-link
  Body: { token: string }
  Response: { session_token: string, account_id: string }
  Errors: 401 if expired, already-used, or invalid
  Side-effect: mark token used, create session record
```

### Aviary state

```
GET /aviary/state
  Auth: session token
  Response: AviarySnapshot JSON (see schema below)
  Cache: per-account, max 60s CDN TTL with Cache-Control: private

POST /aviary/events
  Auth: session token
  Body: InteractionEventBatch (array of events with idempotency keys)
  Response: 202 Accepted
  Note: events are written to interaction_events; simulation worker processes them async
```

AviarySnapshot JSON shape:

```json
{
  "snapshot_at": "ISO8601",
  "aviary_age_days": 42,
  "day_night_phase": "morning | midday | afternoon | evening | night",
  "weather": "clear | rain | wind | null",
  "birds": [
    {
      "bird_id": "uuid",
      "name": "Pip",
      "species_id": "warbler_1",
      "mood_state": "content",
      "perch_zone": "front | middle | back",
      "call_grammar_params": {
        "motif_ids": ["w1_rise", "w1_trill"],
        "pitch_variance": 0.12,
        "inter_call_gap_seconds": [4, 9],
        "amplitude_envelope": "soft"
      },
      "plumage_saturation": 0.71,
      "animation_hint": "preening"
    }
  ]
}
```

`call_grammar_params` is computed by the simulation from the bird's personality vector and current mood. It is the only personality-derived data the client receives — and it is expressed as audio-shape parameters, never as trait values.

### Notebook

```
GET /notebook
  Auth: session token
  Query: ?before=ISO8601&limit=20
  Response: { entries: [{id, body, written_at}], has_more: bool }
```

### Account

```
GET /account
  Auth: session token
  Response: { account_id, created_at, settings, sessions: [{id, device_hint, created_at}] }

PATCH /account/settings
  Auth: session token
  Body: { visit_notifications_enabled?: bool }
  Response: 200

DELETE /sessions/{session_id}
  Auth: session token
  Response: 204 (revokes that session)

POST /account/export
  Auth: session token
  Response: 202 (export is generated async, emailed as download link)

DELETE /account
  Auth: session token
  Response: 202 (soft-delete initiated; 30-day window)

POST /account/recover
  Auth: session token (still valid during soft-delete window)
  Response: 200 (cancels pending deletion)
```

### Visits

```
POST /visits
  Auth: session token (host)
  Body: { visitor_email: string }
  Response: { visit_id: string, invite_link: string }
  Validation: visitor_email must be a valid email; host may not invite themselves

DELETE /visits/{visit_id}
  Auth: session token (host)
  Response: 204 (immediate revocation)

GET /visits
  Auth: session token (host)
  Response: { active_invites: [...], visit_log: [...] }

GET /visits/view/{visitor_token}
  Auth: none (visitor token in path)
  Response: AviarySnapshot (same shape as /aviary/state but with visit_mode: true)
  Errors: 410 Gone if revoked or expired
  Note: visitor sessions do not write interaction events; simulation ignores visitor presence
```

### Server-Sent Events (live updates while tab is visible)

```
GET /aviary/events/stream
  Auth: session token
  Content-Type: text/event-stream
  Events emitted: state_update (new AviarySnapshot), notebook_entry (new entry)
  Reconnect: standard SSE retry; client re-subscribes on visibility change
```

SSE is preferable to WebSocket for this use case (unidirectional, stateless reconnect, no binary frames needed). The client falls back to polling if SSE fails.

---

## 5. Simulation Engine Design

### Architecture

The simulation worker is a separate process (or a long-running task in a job queue) that:
1. Runs one tick per account approximately every 60 seconds (actual cadence is a configurable server parameter)
2. Is not on the request path — tick latency does not affect API response times
3. Uses optimistic locking (version field on `personality_vectors`) to detect concurrent tick runs and skip rather than double-apply

### Tick algorithm

```
for each active account (not soft-deleted, with at least one bird):
  1. Load current personality_vectors and moods for all birds
  2. Load unprocessed interaction_events since last tick, ordered by occurred_at
  3. Compute presence_time_seconds from presence_ping events
     - Count pings * ping_interval_seconds, capped at tick_window_seconds
     - Presence is only credited for pings received from a visible, focused, active window
       (the client-side 3-condition check gates what events are even submitted)
  4. Compute drift_deltas per bird per trait (see below)
  5. Compute mood_transitions per bird (see below)
  6. Compute new perch_zones per bird (from updated boldness)
  7. Write new personality_vectors (CAS on version; skip this account if CAS fails, retry next tick)
  8. Write new moods
  9. Write new aviary_state_snapshot
  10. Evaluate notebook trigger conditions; if met, enqueue notebook entry generation
  11. Mark all processed interaction_events with processed_at = NOW()
```

### Drift function

All drift deltas are additive and non-negative (monotonic toward expressive).

```
presence_weight   = presence_time_seconds / 3600  (hours of presence this tick window)
listen_in_seconds = sum of (listen_in_end - listen_in_start) per bird

drift_per_bird:
  boldness         += presence_weight * 0.002  (all birds drift slightly)
  social_warmth    += presence_weight * 0.001
                   += listen_in_seconds / 3600 * 0.005  (listen-in is a stronger warmth signal)
  vocal_frequency  += listen_in_seconds / 3600 * 0.005
  plumage_saturation += presence_weight * 0.001
  curiosity        += count_of(offer_accepted) * 0.002
                   += count_of(offer_near_bird) * 0.001
```

These coefficients are starting calibration values. The system targets:
- Measurable instrument-level change after ~1 week regular use (daily ~30 min presence): ~0.02-0.05 trait units
- Visible-to-user change after ~3 weeks: ~0.1-0.15 trait units

All traits are clamped to [0, 1.0] on write.

Drift is applied as a delta to the current value; no client can submit an absolute value.

### Mood transition engine

Mood is a state machine. Transition probabilities are computed per tick as:

```
inputs:
  time_of_day_bias     : map from hour → favored_mood_state
  recent_interactions  : list of interaction types in last tick window
  neighbor_moods       : moods of other birds in the same aviary (for wary spread)
  personality_boldness : current boldness trait value

transition_matrix:
  From \ To   wary  content  curious  drowsy  alert
  wary         0.6    0.2      0.05    0.1    0.05
  content      0.05   0.5      0.25    0.1    0.1
  curious      0.05   0.3      0.5     0.05   0.1
  drowsy       0.1    0.3      0.05    0.5    0.05
  alert        0.1    0.3      0.3     0.1    0.2

Modifiers (applied multiplicatively to row):
  morning (5–9):       alert ×2
  midday (10–14):      content ×1.5, curious ×1.5
  evening (17–20):     drowsy ×2
  night (21–4):        drowsy ×3
  offer_accepted:      content ×2 (for receiving bird only)
  neighbor_wary:       wary ×1.5 (spread effect)
  high boldness (>0.7): wary ×0.5 (bold bird resists wary)
```

Probabilities are renormalized after modifiers. The tick draws from the resulting distribution. This is not a deterministic function — the same state can produce different mood outputs, which is the correct model.

At full night, birds with `drowsy` state emit no calls (settled/sleeping). The one nocturnal species remains in `alert` state at night regardless of the time-of-day modifier.

### Call grammar runtime (server-side params, client-side synthesis)

Each species has a motif library defined in a static JSON configuration:

```json
{
  "species_id": "warbler_1",
  "motifs": [
    { "id": "w1_rise",  "intervals": [0, 2, 5], "duration_ms": 400, "base_freq_hz": 3200 },
    { "id": "w1_trill", "intervals": [0, 1, 0], "duration_ms": 300, "base_freq_hz": 3000 },
    { "id": "w1_single","intervals": [0],        "duration_ms": 150, "base_freq_hz": 3400 }
  ],
  "mood_motif_weights": {
    "content":  { "w1_rise": 0.5, "w1_trill": 0.4, "w1_single": 0.1 },
    "curious":  { "w1_rise": 0.3, "w1_trill": 0.3, "w1_single": 0.4 },
    "wary":     { "w1_rise": 0.1, "w1_trill": 0.1, "w1_single": 0.8 },
    "drowsy":   { "w1_single": 1.0 },
    "alert":    { "w1_rise": 0.2, "w1_trill": 0.6, "w1_single": 0.2 }
  }
}
```

The snapshot payload delivers `call_grammar_params` per bird:
- `motif_ids`: which motifs are active this tick (selected by mood-weighted sampling, stable for the tick window)
- `pitch_variance`: derived from curiosity and current mood (curious = more variance)
- `inter_call_gap_seconds`: range [min, max] derived from vocal_frequency (higher frequency = shorter gaps)
- `amplitude_envelope`: "soft" | "sharp" derived from boldness and mood

Bird-to-bird call response: when a call event fires on Bird A, the client evaluates other birds' `social_warmth` and current mood to determine whether they call back. High-warmth birds in content or curious mood have a ~30% chance of a response call within 2–6 seconds.

### Notebook entry generation

Notebook entries are generated server-side by a prose-generation function, not by an LLM. The function is a template engine keyed on trigger conditions:

| Trigger | Example entry |
|---|---|
| Bird A greeted before Bird B (unusual order) | "pip greeted before wren today, first time this week." |
| Long presence session (>30 min) | "a long stretch of quiet this morning. pip preened for several minutes without looking up." |
| Weather event during session | "wren is fluffed against the cool air, watching the back perch. low calls only." |
| Offer accepted by wary bird | "the wary one took the seed eventually. watched it for a long time first." |
| Chorus event (2+ birds calling in 10s window) | "both birds called at once just before noon." |
| Absence > 7 days on return | "the aviary has been quiet. wren is on the low perch, as usual." |

Templates are parameterized by bird names, moods, perch zones, time of day, and recent interaction history. Entries are generated conservatively — at most one per 2–3 days for a regularly active account, with triggers requiring a threshold signal strength before firing. The system maintains a last-entry-written timestamp per account to enforce the sparsity constraint.

---

## 6. Sync Model

### Canonical state ownership

| State | Owner | Client role |
|---|---|---|
| Personality vector | Server simulation worker only | Read from snapshot |
| Mood | Server simulation worker only | Read from snapshot |
| Perch zone | Server simulation worker only | Read from snapshot; interpolate motion |
| Interaction events | Client submits; server owns the log | Write via POST /aviary/events |
| Presence pings | Client detects and submits | Write as presence_ping events |

### Multi-device behavior

When the same account is open on two devices simultaneously:
- Both devices read from the same `aviary_state_snapshots` record
- Both devices submit their own interaction events to the shared event log
- The simulation worker processes all events in occurred_at order regardless of source device
- There is no device-to-device communication; the server is the only intermediary

Presence pings from two simultaneous devices do not double-count presence. The event log records each ping with its device hint. The simulation worker deduplicates overlapping presence windows by collapsing continuous ping sequences from any device into a single presence window.

### Conflict prevention

There are no client-writable personality fields, so write conflicts on personality state are structurally impossible. The only CAS risk is the simulation worker tick itself (two tick processes racing on the same account). This is handled by:
1. The version field on `personality_vectors` used as an optimistic lock
2. A per-account distributed lock in Redis (or equivalent) held for the duration of one tick
3. If the lock is already held, the account is skipped on this pass and picked up on the next tick

The interaction_event table is append-only. Duplicate submissions are handled by the `idempotency_key` UNIQUE constraint — a second submission with the same key is silently ignored.

### State snapshot delivery

Initial page load: the server renders the HTML shell with the latest `aviary_state_snapshot` for the authenticated account embedded as an inline JSON script tag. This eliminates a round-trip on page load and is the primary mechanism for achieving the <500ms first-bird budget.

The CDN caches the HTML shell at the edge. The per-account state snapshot embedded in the HTML has a short TTL (60 seconds) and is keyed on the account's session token. On cache miss, the edge fetches from origin. On cold load, the quiet-field loading state is shown (no spinner) while the snapshot arrives.

While the tab is visible, the client maintains an SSE connection to `/aviary/events/stream`. The server pushes state_update events after each tick that affects this account. The client applies the update by smoothly interpolating birds from their current animated positions to their new snapshot positions.

On tab visibility change (tab re-activated), the client immediately re-fetches the current snapshot via GET /aviary/state.

---

## 7. Frontend Rendering Pipeline

### Technology choices

- **Framework**: React (for the chrome overlay, settings, notebook panel, offer UI)
- **Scene rendering**: Canvas 2D (requestAnimationFrame loop)
- **Audio**: WebAudio API (see Audio Pipeline section)
- **Build**: Vite with aggressive code-splitting
- **State management**: minimal — snapshot from server is the source of truth; local UI state only for overlays

### Scene composition (Canvas 2D layers, back to front)

1. **Sky gradient layer** — computed from time-of-day phase and day/night cycle; updates gradually
2. **Background foliage** — static SVG composited into canvas; subtle parallax offset driven by a slow sine wave
3. **Perch structures** — branches/rails at front/middle/back zones; static per scene
4. **Bird sprites** — per-bird animated with mood-driven pose sequences (see below)
5. **Weather overlay** — rain particle system or leaf flutter; drawn on top of birds during weather events
6. **Foreground layer** — occasional leaf/feather drift; client-side only, not tied to simulation

Parallax: background layer moves at 20% of camera drift, foreground at 120%. Camera drift is a slow, barely perceptible sine wave (0.2px amplitude, ~30s period). This produces depth without apparent scrolling.

### Bird rendering

Each bird is drawn from a set of vector paths (SVG exported to canvas-draw commands) representing 6–8 pose states:
- Neutral perch
- Preen (head down)
- Scan (head lifted, alert)
- Fluffed (feathers out, drowsy)
- Call (beak open)
- Head-tilt (curious)
- Flight arc (in-transit between perches)

Plumage saturation is applied as a Canvas filter (`saturate(${plumage_saturation * 150}%)`) — a simple CSS-style filter applied per-bird draw call. This makes plumage drift visually apparent without maintaining separate sprite sheets.

Animation loop:

```
Each frame (rAF, targeting 60fps):
  1. Advance all per-bird animation timers
  2. If bird is in a flight arc, interpolate position along arc
  3. If bird is between pose keyframes, interpolate
  4. If new snapshot was received, start smooth transition to new state:
     - perch_zone change → trigger flight arc animation
     - mood change → begin cross-fading to new idle pose sequence
  5. Apply plumage saturation filter
  6. Draw bird
```

Pose sequences are looped with randomized timing jitter (±15% of nominal keyframe duration) so no two birds ever look like they're on the same clock.

### Reduced-motion mode

When `prefers-reduced-motion` is active or the user has enabled it in accessibility settings:

- The animation loop still runs (do not stop rAF)
- Instead of smooth positional interpolation, birds cross-fade between still poses
- Cross-fade duration: 400ms (CSS-style opacity blend on the canvas draw)
- Flight arcs are replaced by a cross-fade between source and destination perch poses
- Ambient leaf/feather drift is disabled
- Weather particle effects are replaced by a slow overlay opacity pulse
- Day/night transitions remain (slowed to 60s minimum)

The reduced-motion mode is detected at startup and applied to the animation controller's behavior; no separate code path is needed for the rest of the scene.

### Loading sequence

```
1. Browser receives HTML with embedded state snapshot JSON
2. React mounts; extracts snapshot from script tag
3. Canvas element is created; first frame is drawn immediately (birds mid-action)
4. rAF loop starts; WebAudio context initialized
5. SSE connection opened to /aviary/events/stream

If snapshot is not yet available (cache miss, slow connection):
  - Show quiet field (soft sky color, no spinner, no loading text)
  - Optionally: 1-2 very subtle motion cues (slow leaf drift, or a slow sky gradient pulse)
  - First bird appears as soon as snapshot arrives
```

Specifically: there is no spinner, no "loading..." text, no progress indicator. The product is either the aviary or the quiet field. The quiet field is styled like the aviary at dusk — calming, not machine-like.

### Top bar

HTML overlay (position: fixed, z-index above canvas). Contains:
- Account icon → account settings modal
- Accessibility icon → accessibility settings modal
- Field notebook icon → notebook slide-in panel (from right)
- Offer icon → offer picker overlay

Top bar fade: CSS transition on opacity; after 3 seconds of no pointermove or keydown, opacity transitions to 0.15 over 0.5s. On pointermove/keydown, returns to 1.0 over 0.2s.

The top bar is not hidden (opacity 0) — it remains at 0.15 so keyboard users always have a target to focus.

### Settle gesture

1. User triggers settle from top bar icon
2. Canvas sky gradient transitions to warm amber/indigo (evening palette) over 4 seconds
3. All call gain nodes ramp down to 20% of current level over 4 seconds
4. Settle event submitted to /aviary/events
5. For 5 seconds, any click/tap on the aviary scene cancels the settle (undo affordance):
   - Reverse sky gradient transition
   - Restore call gains
   - Submit cancel event (no-op for simulation; presence window is still active)

---

## 8. Audio Pipeline

### WebAudio graph

```
AudioContext
  ├── BirdCallSynthesizer[0..N] (one per bird)
  │     ├── OscillatorNodes (one per motif interval)
  │     ├── GainNode (amplitude envelope)
  │     └── BiquadFilterNode (species-specific timbre shaping)
  ├── BirdGainNode[0..N] (per-bird mix level; this is what listen-in adjusts)
  ├── MasterGainNode
  └── AudioContext.destination
```

### Procedural call synthesis

When a bird's call fires (driven by the `inter_call_gap_seconds` range in call_grammar_params):

1. Select a motif from the `motif_ids` list (weighted by mood_motif_weights from species config)
2. For each interval in the motif:
   - Compute frequency = base_freq_hz * 2^(interval/12) * (1 + pitch_variance * random(-1,1))
   - Schedule OscillatorNode frequency and GainNode envelope using WebAudio's `setValueAtTime` and `exponentialRampToValueAtTime`
3. Apply amplitude_envelope: "soft" = slow attack (50ms) + slow release (200ms); "sharp" = fast attack (10ms) + faster release (80ms)
4. At call-start: if captioning is enabled, generate caption text from the motif shape and dispatch to the caption rendering system

### Caption generation

Caption text is generated from the same motif parameters used for synthesis:

```
motif_intervals + mood + amplitude_envelope → caption string

Examples:
  [0, 2, 5], content, soft → "a soft three-note rise"
  [0, 1, 0], alert, sharp  → "a quick two-note call"
  [0], wary, soft           → "a single soft call from the back perch"
```

Template rules:
- 1 interval → "a single [envelope] call"
- 2 intervals → "a [envelope] two-note [direction]" (rising if ascending, falling if descending)
- 3+ intervals → "a [envelope] [count]-note [shape]" (rise, trill, cascade, etc.)

Caption appears near the bird in the canvas (or as an absolutely-positioned HTML element keyed on bird_id) with a 200ms fade-in and a 400ms fade-out at call end.

### Listen-in implementation

```
function engageListenIn(targetBirdId) {
  const rampTime = 2.5; // seconds
  const ambient = 0.2;  // other birds' relative level; never 0
  birdGainNodes.forEach((gainNode, birdId) => {
    const target = birdId === targetBirdId ? 1.0 : ambient;
    gainNode.gain.linearRampToValueAtTime(target, audioContext.currentTime + rampTime);
  });
}

function disengageListenIn() {
  const rampTime = 2.5;
  birdGainNodes.forEach(gainNode => {
    gainNode.gain.linearRampToValueAtTime(equalGain, audioContext.currentTime + rampTime);
  });
}
```

Where `equalGain = 1.0 / sqrt(birdCount)` (equal loudness summing, not equal amplitude).

### WebAudio initialization

WebAudio context must be created (or resumed) from a user gesture due to browser autoplay policies. The first click/tap/keypress on the aviary resumes the context. If the context is suspended, calls are synthesized but scheduled for when the context resumes (they do not pile up; only the most recent call's schedule is kept).

### WebAudio fallback

If WebAudio is unavailable:
- The AudioContext creation is wrapped in a try/catch
- On failure: set `audioAvailable = false`, enable captioning by default
- The scene renders normally; calls are still generated (for caption purposes); no audio is emitted
- A small indicator in accessibility settings shows "audio unavailable on this browser" in matter-of-fact voice
- No recorded audio fallback is shipped

### Memory management

- OscillatorNodes are disconnected and dereferenced immediately after their scheduled end time (+ 100ms buffer)
- A pool of 8 GainNodes per bird is maintained; nodes are reused across calls
- AudioBuffers are not used (fully procedural synthesis); no buffer pool needed
- After 30 minutes, a memory check verifies that WebAudio node count has not grown; this is a CI integration test

---

## 9. Accessibility Surfaces

### Screen-reader narration

ARIA live region setup:

```html
<div
  id="aviary-narration"
  role="region"
  aria-label="aviary"
  aria-live="polite"
  aria-atomic="false"
>
  <!-- prose updated by JS -->
</div>
```

For user-initiated events (offer reaction, return greeting, settle), the region is temporarily promoted to `aria-live="assertive"` for that single update, then returned to `"polite"`.

Narration cadence:
- Idle: new prose every 45 seconds (midpoint of the 30–60s range)
- On user event: immediate update
- On session start (return greeting): immediate update when greeting animation starts

Narration prose generator: same template system as notebook entries but with faster trigger thresholds and more frequent "ambient state" templates (no minimum sparsity constraint). Examples:

> "a small grey bird is on the front rail, calling softly. another sits further back, feathers fluffed. it is morning in the aviary."

> "wren just accepted the seed. the other bird is watching from the middle perch."

The generator never produces announcement-style text:
- ❌ "Wren's mood is content."
- ❌ "Pip has accepted the seed offer."
- ✓ "wren is settled on the low perch, calls quiet."
- ✓ "the seed sat in the dish for a moment. wren approached it slowly."

Automated test: a prose linter that flags any narration output containing title-cased mood state names, gamification words ("level", "score", "streak"), or announcement patterns ("has accepted", "mood is").

### Reduced-motion mode

Implementation details in the rendering pipeline section above. Accessibility settings UI offers an explicit toggle ("prefer calmer motion") independent of the OS `prefers-reduced-motion` media query. Either the OS setting or the in-product toggle enables reduced-motion mode. The setting persists in account settings server-side.

### Call captioning

Toggle in accessibility settings. Also auto-enabled when WebAudio is unavailable. Caption display is via HTML elements positioned using the bird's current canvas coordinates (translated to DOM coordinates via `canvas.getBoundingClientRect()`). Captions use the product's naturalist voice; no announcement-style text.

### Keyboard navigation

Full tab order:
1. Top bar: Account, Accessibility, Notebook, Offer (in left-to-right DOM order)
2. From top bar, Tab continues into aviary scene → focuses first bird
3. Arrow Left/Right: move focus between birds
4. Enter: engage listen-in on focused bird; Enter again or Escape: disengage
5. Top bar Offer icon: opens offer picker; within picker, Tab navigates offer types, Enter selects
6. Top bar Notebook icon: opens notebook panel; Tab/Arrow within panel, Escape closes

Focus ring: a soft, 2px high-contrast outline rendered as an HTML element that tracks the focused bird's canvas coordinates. It uses a color from the design system that passes WCAG AA against both the bright morning palette and the dim night palette (the visual designer specifies the exact value, but the planning constraint is that it must pass both extremes).

### WCAG AA contrast

Applies to:
- Top bar icon labels (if any)
- Accessibility settings modal text
- Account settings modal text
- Error surfaces (sign-in, sync errors, unsupported browser)
- Call captions
- Notebook panel text

Does not apply to the canvas aviary scene (no user-copy text in canvas). If captions are rendered as HTML overlays on top of the canvas, they must also pass AA.

---

## 10. Performance Budgets and Observability

### Budgets (enforced in CI)

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS bundle (gzipped) | <2MB | Vite bundle analyzer in CI; fail build if exceeded |
| Time to first bird (mid-tier mobile, 4G) | <500ms | Synthetic Lighthouse run in CI against staging; fail if p50 >400ms |
| Idle frame rate (5-year-old laptop) | 60fps | Puppeteer perf test in CI; fail if p10 frame time >20ms over 60s |
| Memory growth (30-min session) | 0 growth | Puppeteer memory snapshot at 0min and 30min in CI; fail if heap grows >5MB |
| Simulation tick latency (p99) | <5s | Server-side histogram, alarm triggers pager if exceeded |

### Code-splitting strategy

| Chunk | When loaded |
|---|---|
| `core` | Initial (contains: aviary renderer, audio engine, snapshot parser, top bar) |
| `notebook` | When user opens notebook panel |
| `settings` | When user opens account or accessibility settings |
| `visit-flow` | When user navigates to a visit invite link |
| `auth` | On sign-in page (separate route from aviary) |

The `core` bundle must be kept below 1.5MB gzipped to leave headroom for growth.

### Instrumentation (what we measure)

Collected as aggregate, anonymized telemetry. Per the privacy constraint, no per-bird or per-account fields appear in telemetry.

**Client-side (Real User Monitoring):**
- `first_bird_render_ms`: custom performance mark set when first bird is drawn on canvas
- `time_to_interactive_ms`: standard TTI metric
- `frame_time_p90_ms`: sliding 60-second p90 of rAF frame durations
- `audio_context_error_count`: count of WebAudio context failures per session
- `sse_reconnect_count`: count of SSE reconnections per session (indicates network quality)
- `session_duration_seconds`: histogram (anonymized)

**Server-side:**
- `simulation_tick_duration_ms`: histogram per tick; p50/p99 alarms
- `api_request_latency_ms` per endpoint
- `api_error_rate` per endpoint
- `snapshot_cache_hit_rate`
- `event_log_queue_depth`: count of unprocessed interaction events (alarm if growing unboundedly)

### What we deliberately do not measure

Per the privacy commitment:
- No per-account personality trait values in any telemetry pipeline
- No per-bird interaction sequences
- No cross-account drift rate analysis
- No feature usage broken down by account

The analytics warehouse has no read path to the simulation database. This is an architectural boundary, not a policy one.

---

## 11. Rollout

### Phase 1: Private alpha (pre-launch)

- Deploy all services to staging
- Manually adopt test accounts, run 2-week presence sessions
- Validate drift calibration: check that instrument-level trait changes appear after 7 days of simulated presence
- Validate audio: listening sessions with diverse motif combinations; check for phase artifacts in 2+ bird chorus
- Validate accessibility: screen-reader walk-through (with a real screen-reader user) against narration prose spec; check caption sync
- Performance benchmark suite: hit all CI budgets against staging hardware

### Phase 2: Closed beta (invite-only, ~100 accounts)

- Ship with 2 birds per account, full feature set
- Monitor: drift rate distribution, tick latency, SSE reconnect rates, first-bird timing from real devices
- Collect open-text feedback on audio quality (procedural calls)
- Run reduced-motion mode with a small percentage of accounts with `prefers-reduced-motion` set
- Measure presence-event rate to validate the 3-condition check is correctly gating idle-tab presence

### Phase 3: Launch

- Open registration
- All accounts start with 2 birds
- Third-bird offer available at aviary_age >= 90 days (server config; can be adjusted post-launch without redeployment)
- Visit invitations enabled (off by default per account)
- Full instrumentation live from day one

### Post-launch: birds-per-aviary ramping

The aviary_age thresholds for new-bird offers are server-configurable:
```json
{
  "bird_offer_thresholds_days": [90, 180, 270, 365, 450]
}
```
This produces up to 7 birds for an aviary that has existed for ~15 months, at a natural deepening cadence. The thresholds are adjusted if post-launch data shows the pacing is too fast (users feeling overwhelmed) or too slow (users disengaging).

### Day-one instrumentation checklist

Before launch:
- [ ] Simulation tick latency histogram + p99 alarm wired to pager
- [ ] first_bird_render_ms RUM collection live
- [ ] Event log queue depth alarm live
- [ ] Presence-event rate aggregate monitoring live
- [ ] Snapshot cache hit rate visible in dashboard
- [ ] Bundle size CI check passing
- [ ] Synthetic browser performance run passing

---

## 12. Risks

### Risk 1: Drift calibration too fast or too slow

**Failure mode**: Too fast → users notice their birds changing between sessions (Tamagotchi feel). Too slow → weeks of presence produce no perceptible change (screensaver feel).

**Mitigation**:
- Use accelerated-time simulation in staging: compress one week of tick cadence into 6 hours and measure trait delta magnitudes against the calibration targets
- Calibration targets are in the plan (measurable after ~1 week, visible after ~3 weeks) and are testable
- Drift coefficient values are centralized server-side config, not embedded in the tick code, so they can be adjusted without a deployment
- Post-launch: monitor the distribution of personality vector values across accounts; if all birds are converging too fast toward 1.0 across the user base, reduce coefficients

### Risk 2: Sync correctness — silent personality state loss

**Failure mode**: A double-tick, a CAS failure mishandled, or an event processed twice silently corrupts personality vectors. Users feel their birds have changed unexpectedly or are drifting slower than expected. No test catches this because the incorrect value is plausible.

**Mitigation**:
- Simulation worker uses optimistic locking (version field CAS) plus per-account distributed lock — two independent mechanisms
- Integration tests: simulate concurrent tick attempts and verify only one succeeds; verify idempotency_key prevents double event processing
- Personality vector write audit log: every write to `personality_vectors` is shadowed to an immutable audit table with tick_id, the input event_ids consumed, and the old and new values — gives a reconstruction path if corruption is detected
- Monotonicity check: a scheduled job verifies no personality trait has decreased, ever, on any bird; alarm if violated

### Risk 3: Audio uncanniness — calls sound robotic or repetitive

**Failure mode**: Procedural calls don't achieve the perceptual goal of "never sounds exactly the same twice"; the chorus mechanic produces phase artifacts; calls sound thin or buzzy.

**Mitigation**:
- Dedicated audio design phase before beta: a musician/audio designer auditions the motif libraries and synthesis parameters
- Beta feedback specifically asks about audio quality and "does it sound alive?"
- Chorus test: synthesize 7 simultaneous birds and verify (both instrumentally and by ear) that individual call signatures remain distinguishable
- Pitch variance and envelope parameters are tunable server-side via the `call_grammar_params` in the snapshot — can be adjusted per species without client redeployment
- If WebAudio synthesis is not achieving the quality target by beta, escalate: consider whether the motif complexity needs to increase, or whether a small set of pre-synthesized audio fragments (not loops — fragments) can supplement

### Risk 4: Accessibility regressions

**Failure mode**: Narration prose drifts to announcement style; captions fall out of sync with calls; reduced-motion mode is accidentally broken by a canvas rendering change; keyboard focus management breaks on a new overlay.

**Mitigation**:
- Automated prose linter (described in accessibility section above) runs on every narration output in CI
- Caption-sync test: drive a synthetic call sequence and verify each caption appears within 50ms of the corresponding audio schedule
- Reduced-motion test: Puppeteer test that sets `prefers-reduced-motion: reduce` and verifies no transform or opacity animations are running (no CSS animation values changing frame-to-frame in the canvas)
- Keyboard navigation test: Playwright test that tabs through every interactive surface and verifies focus indicators are visible
- A11y is gated on the same release criteria as performance — not a "nice to have" that ships in v1.1

### Risk 5: Presence signal inflation

**Failure mode**: A client-side bug causes the 3-condition presence check to be too loose (e.g., the activity window is accidentally longer than intended, or one of the conditions is not correctly checked). Drift runs faster than calibrated across the entire user base. Silent — no test catches it because fast drift isn't obviously wrong.

**Mitigation**:
- Client-side presence detection has its own unit tests: simulate each of the three conditions failing and verify no ping is submitted
- Aggregate monitoring: alarm if the average presence_time_per_day_per_active_account exceeds a threshold (e.g., >4 hours/day average suggests the check is too loose)
- Presence ping rate is visible in aggregate telemetry (ping count / active session count); if this ratio is higher than expected (e.g., implies users are present 20 hours a day), investigate

### Risk 6: Bundle size creep

**Failure mode**: Feature additions gradually erode the 2MB bundle budget, and the first-bird timing budget degrades before anyone notices.

**Mitigation**:
- CI enforces <2MB hard limit on every PR; the build fails if exceeded
- Bundle size dashboard visible to the engineering team (not just CI)
- Code-splitting is architected from day one with clear chunk ownership; new features must identify which existing chunk they belong to or create a new lazy chunk

### Risk 7: "Notice, never announce" violation creep

**Failure mode**: A well-meaning contributor adds a toast for a return visit, a badge on the notebook icon when there's a new entry, or a "welcome back" text somewhere. The product's affective core is eroded in small steps.

**Mitigation**:
- Automated UI tests that assert no toast/snackbar/badge-with-count elements are present during a simulated return session or offer interaction
- Product review checklist item: "Does this surface announce or notify?" is a required question in the PR template for UI changes
- This plan documents the prohibitions explicitly (no toast, no badge, no counter, no streak) so there is no ambiguity about intent

### Risk 8: Magic link email deliverability

**Failure mode**: Magic links go to spam; users can't sign in; they bounce and don't return.

**Mitigation**:
- Use a reputable transactional email provider (SendGrid, Postmark, or equivalent) with dedicated sending IP
- SPF, DKIM, DMARC configured from day one
- Monitor delivery rates and bounce rates in the email provider dashboard
- Provide clear "check your spam folder" instruction on the "we've sent you a link" page (matter-of-fact voice)

---

## Appendix: Key implementation decisions noted

1. **Canvas 2D over WebGL**: Simpler, sufficient for 7 birds with smooth animation at 60fps on a 5-year-old laptop. The canvas surface is not GPU-bound at this complexity. If future species count or weather complexity makes WebGL necessary, the rendering layer is isolated enough to swap.

2. **SSE over WebSocket**: Aviary state updates are unidirectional (server → client). SSE has simpler reconnect semantics, works over standard HTTP/2, and requires no special proxy config.

3. **Template-based notebook prose over LLM**: An LLM could generate more naturalistic prose, but introduces non-determinism in voice consistency, latency on write, and cost at scale. Template-based generation is predictable, testable, and produces consistent voice. A future enhancement could use a fine-tuned small model if the template library becomes unwieldy.

4. **Postgres for everything**: Simulation state, event log, accounts, notebook. No separate time-series DB or event streaming system at v1. The interaction_event table is effectively an append-only log; if it grows too large, archival and partitioning by account_id is straightforward.

5. **Synthetic UUID for account IDs**: Non-negotiable per the PRD. Enforced by making email_encrypted a separate non-identifier field and using UUID everywhere else.

6. **Plumage saturation as a canvas filter**: The simplest implementation that produces a visually continuous effect. Alternatives (separate sprite sheets per saturation level, shader-based tinting) are more complex with no perceptual benefit at the range values actually drift through.
