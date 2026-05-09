# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In v1
- Browser-only web application (no native apps)
- Single-user accounts, magic-link sign-in only
- One aviary per account; starts with 2 birds, max 7
- Bird personality vectors persisted and ticked server-side
- Mood system (wary, content, curious, drowsy, alert) with daily-ish reset cadence
- Personality drift (monotonic toward expressive, no negative drift)
- Presence accounting (3-signal conjunction: visibilityState + window focus + recent pointer/key activity)
- Return-greeting interaction (procedurally varied, boldness- and mood-shaped)
- Listen-in interaction (mix re-balance, not mute)
- Offer interaction (seed, song fragment, still pool) with per-bird cooldown
- Settle gesture (soft session-end, 5s undo, optional)
- Field notebook (auto-generated naturalist prose, rare entries, read-only)
- Procedural call synthesis via WebAudio (no recorded audio)
- Day/night cycle keyed to user's local timezone
- Ambient weather (rare rain, soft wind; effects on mood)
- Ambient micro-motion (leaves, feathers, idle bird motion)
- Multi-device sync (server-canonical, no client-side merge)
- Visit invitation (email-based, per-invite opt-in, read-only for visitor, revocable)
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fades, not stripped fallback)
- Call captioning (optional, naturalist voice, runtime-generated)
- WCAG AA contrast on all user-copy text
- Keyboard navigation for all interactive surfaces
- Account export (JSON snapshot on demand)
- Soft-deletion (30-day recovery window)
- Privacy: per-account interaction data never aggregated for any purpose beyond that user's simulation

### Out of v1 (explicitly refused, not deferred)
- Native mobile apps
- Gamification: achievements, streaks, levels, badges, scores, green-dot calendars, XP
- Tamagotchi mechanics: hunger meters, death, visible distress from neglect
- Social network surfaces: profiles, follows, public discovery, comments, leaderboards, shared aviaries
- Push, email, or in-product notifications (only the opt-in visit log exists)
- Password-based or SSO authentication
- Multiple aviaries per account
- Multi-aviary accounts
- Customizable scenes or user-controlled perch placement
- Recorded audio fallback

---

## 2. Architecture

### Service topology

```
┌─────────────────────────────────────────────────────────┐
│ Client (browser)                                        │
│  ├─ Render layer (Canvas/SVG scene, WebAudio, React)    │
│  ├─ Presence tracker (3-signal conjunction)             │
│  ├─ Event emitter (interaction events → API)            │
│  └─ Snapshot consumer (pull → interpolate → render)     │
└──────────────┬──────────────────────────────────────────┘
               │ HTTPS / WebSocket (state stream)
┌──────────────▼──────────────────────────────────────────┐
│ API Gateway / Edge (CDN-delivered HTML + initial state) │
└──────────────┬──────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────┐
│ Application Server (stateless, horizontally scalable)   │
│  ├─ Auth service (magic link generation & validation)   │
│  ├─ Aviary state service (snapshot read, event write)   │
│  ├─ Visit service (invite CRUD, revocation)             │
│  ├─ Notebook service (entry generation, retrieval)      │
│  └─ Account service (CRUD, export, deletion)            │
└──────────────┬──────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────┐
│ Simulation service (tick worker, event-log consumer)    │
│  ├─ Tick scheduler (~60s cadence)                       │
│  ├─ Drift engine (presence-time + interaction inputs)   │
│  ├─ Mood engine (transitions, time-of-day, ambient)     │
│  └─ Call-grammar state (per-bird motif weights)         │
└──────────────┬──────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────┐
│ Persistent storage                                      │
│  ├─ Account DB (accounts, sessions, device tokens)      │
│  ├─ Aviary DB (birds, personality vectors, mood state)  │
│  ├─ Event log (append-only interaction events)          │
│  ├─ Notebook DB (entries)                               │
│  └─ Visit DB (invitations, visit logs)                  │
└─────────────────────────────────────────────────────────┘
```

### Client/server split decisions

- **Server owns**: personality vectors, mood state, canonical scene layout, event log, tick scheduling, notebook entries.
- **Client owns**: local rendering state, interpolation between snapshots, audio synthesis, presence signal reporting, UI interactions.
- Clients never write personality state. Clients never compute drift. The server is the only writer of canonical aviary state.
- The simulation service is the only consumer that mutates personality vectors. The application server is read-only with respect to personality vectors.

### Render pipeline boundary

The client-side render pipeline receives a snapshot from the server and is responsible for:
1. Placing birds at their current perch positions and motion states.
2. Synthesizing audio from call-grammar parameters in the snapshot.
3. Running idle micro-motion and ambient effects as a pure client-side loop.
4. Interpolating between consecutive snapshots for smooth motion.

The server-side tick is responsible for:
1. Reading accumulated interaction events.
2. Updating personality vectors (additive deltas only).
3. Transitioning mood states.
4. Advancing call-grammar timing weights.
5. Deciding perch positions and motion triggers for the next snapshot.
6. Writing the canonical state record.

The boundary is: **everything above** is rendering; **everything below** is simulation. They communicate via snapshot pull and event log push only.

---

## 3. Data Model

### Account

```
Account {
  id: UUID (synthetic, not derived from email)
  email: encrypted string (stored once here only)
  email_verified: bool
  email_change_pending: { new_email: string, token: string, expires_at: timestamp } | null
  created_at: timestamp
  deleted_at: timestamp | null  // soft-delete marker
  hard_delete_at: timestamp | null  // 30 days after deleted_at
  settings: {
    visit_notifications_enabled: bool  // default false
    accessibility: {
      reduced_motion: bool
      captions_enabled: bool
    }
  }
}

Session {
  id: UUID
  account_id: UUID
  device_label: string  // user-readable, e.g. "Chrome on MacBook"
  issued_at: timestamp
  last_seen_at: timestamp
  revoked_at: timestamp | null
}

MagicLinkToken {
  token: string (high-entropy, hashed on storage)
  account_id: UUID
  expires_at: timestamp (now + 15 minutes)
  consumed_at: timestamp | null
}
```

### Aviary

```
Aviary {
  id: UUID
  account_id: UUID
  created_at: timestamp
  last_tick_at: timestamp
  bird_ids: [UUID]  // ordered; max 7
  weather_state: { type: enum(clear, rain, wind), started_at: timestamp, ends_at: timestamp }
  next_bird_offer_eligible_at: timestamp | null  // for aviary-age-gated new bird offers
}
```

### Bird

```
Bird {
  id: UUID (stable; never replaced)
  aviary_id: UUID
  species: enum (6 species in v1 pool)
  name: string (user-assigned; renameable; no effect on engine)
  adopted_at: timestamp
  personality_vector: {
    boldness: float [0.0, 1.0]
    social_warmth: float [0.0, 1.0]
    vocal_frequency: float [0.0, 1.0]
    plumage_saturation: float [0.0, 1.0]
    curiosity: float [0.0, 1.0]
  }
  mood: enum(wary, content, curious, drowsy, alert)
  mood_set_at: timestamp
  call_grammar_state: {
    motif_weights: float[]  // per-species motif library weights, shaped by personality
    last_call_at: timestamp | null
  }
  perch_position: enum(front, middle, back)
  perch_motion_state: string  // e.g. "preening", "scanning", "still", "calling"
  offer_cooldowns: {
    seed_next_available_at: timestamp | null
    song_next_available_at: timestamp | null
    pool_next_available_at: timestamp | null
  }
}
```

### Event Log

```
InteractionEvent {
  id: UUID
  account_id: UUID
  bird_id: UUID | null  // null for aviary-level events (settle, presence)
  type: enum(
    presence_ping,
    listen_in_start,
    listen_in_end,
    offer_seed,
    offer_song,
    offer_pool,
    settle,
    session_start,
    session_end
  )
  occurred_at: timestamp
  metadata: JSON  // e.g. {duration_seconds: 180} for listen_in_end
}
```

Note: events are append-only. No event is ever updated or deleted. The simulation tick reads events in chronological order.

### Notebook Entries

```
NotebookEntry {
  id: UUID
  aviary_id: UUID
  created_at: timestamp
  prose: string  // naturalist field-notebook prose, lowercase
  trigger: string  // internal label for entry-generation logic (not exposed)
}
```

### Visit / Invite

```
Invite {
  id: UUID
  host_account_id: UUID
  visitor_email: string  // the invited person's email
  token: string (high-entropy, hashed)
  created_at: timestamp
  expires_at: timestamp  // created_at + 30 days
  consumed_at: timestamp | null
  revoked_at: timestamp | null
  active: bool  // true if consumed and not revoked
}

VisitLog {
  id: UUID
  invite_id: UUID
  host_account_id: UUID
  visitor_email: string
  started_at: timestamp
  ended_at: timestamp | null
  duration_seconds: int | null
}
```

---

## 4. API Surface

All endpoints are over HTTPS. Authenticated endpoints carry a session token in the Authorization header (Bearer). The API is RESTful with JSON payloads. WebSocket or Server-Sent Events are used for the low-frequency keepalive stream.

### Auth

```
POST /auth/magic-link
  Body: { email: string }
  Response: 200 OK (message: "check your email")
  Rate-limited per email.

GET /auth/magic-link/callback?token=<token>
  Response: 302 → /aviary with Set-Cookie: session_token=<jwt>
  Invalidates token on consumption.

DELETE /auth/sessions/:session_id
  Auth: required
  Response: 204 (revokes specified session)

GET /auth/sessions
  Auth: required
  Response: [{ id, device_label, issued_at, last_seen_at }]
```

### Aviary State

```
GET /aviary/snapshot
  Auth: required
  Response: {
    aviary_id: UUID,
    snapshot_at: timestamp,
    weather: { type, ends_at },
    local_time_of_day: string,  // "morning" | "midday" | "evening" | "night"
    birds: [
      {
        id: UUID,
        species: string,
        name: string,
        mood: string,
        perch_position: string,
        perch_motion_state: string,
        call_grammar_params: { ... },  // parameters for client-side synthesis
        plumage_saturation: float,  // only visual trait exposed
        offer_cooldowns: { seed_available: bool, song_available: bool, pool_available: bool }
      }
    ]
  }

  Note: personality vector fields other than plumage_saturation are NEVER returned.
  Note: this response is delivered from CDN edge with a short TTL; the HTML also includes
  an inline initial snapshot so the first paint has zero additional round-trips.
```

### Interaction Events (client → server)

```
POST /aviary/events
  Auth: required
  Body: { events: [InteractionEvent...] }
  Response: 200 OK
  
  Clients batch and send events. Network failures are retried with exponential backoff.
  Events have client-generated UUIDs so retries are idempotent.
  
  Valid event types: presence_ping, listen_in_start, listen_in_end, offer_seed,
                     offer_song, offer_pool, settle, session_start, session_end
```

### Field Notebook

```
GET /aviary/notebook
  Auth: required
  Query: { before: timestamp | null, limit: int (max 50) }
  Response: {
    entries: [{ id, created_at, prose }],
    has_more: bool
  }
```

### Visits

```
POST /aviary/invites
  Auth: required
  Body: { visitor_email: string }
  Response: { invite_id: UUID, expires_at: timestamp }

GET /aviary/invites
  Auth: required (host)
  Response: [{ invite_id, visitor_email, created_at, expires_at, active, revoked_at }]

DELETE /aviary/invites/:invite_id
  Auth: required (host)
  Response: 204

GET /aviary/visit/:token
  Auth: not required (visitor follows email link)
  Response: same shape as GET /aviary/snapshot, or 410 Gone if revoked/expired
  
  The visitor session does NOT write presence events.
  The visitor session does NOT write interaction events of any kind.

GET /account/visit-log
  Auth: required (host)
  Response: [{ invite_id, visitor_email, started_at, ended_at, duration_seconds }]
```

### Account

```
GET /account
  Auth: required
  Response: { email (partially masked), created_at, settings }

PATCH /account/settings
  Auth: required
  Body: { visit_notifications_enabled?: bool, accessibility?: { ... } }
  Response: 200

POST /account/export
  Auth: required
  Response: 202 (export generated and emailed asynchronously)

POST /account/delete
  Auth: required
  Response: 202 (soft-delete initiated; recovery window: 30 days)

DELETE /account/delete  [recovery]
  Auth: required (user signs in during 30-day window)
  Response: 200 (deletion canceled)
```

---

## 5. Simulation Engine Design

### Tick scheduling

The simulation service runs a background worker that triggers once per ~60 seconds (exact cadence calibrated during build; target is 60s with jitter ±5s to prevent thundering-herd on multi-account servers). The tick is the only path by which personality vectors are updated.

Tick steps per aviary (executed serially per aviary, parallel across aviaries):

1. **Load** the aviary's canonical state from the Aviary DB.
2. **Consume events** from the event log since the last tick timestamp. Events older than last_tick_at are ignored.
3. **Compute presence-time** from presence_ping events. A presence_ping represents N seconds of confirmed presence (server validates the conjunction rule by trusting client-reported signals; anomaly detection catches impossible ping densities).
4. **Apply drift deltas** to personality vectors (see below).
5. **Transition mood states** (see below).
6. **Update call-grammar state** (motif weights based on vocal_frequency and current mood).
7. **Decide perch positions** (based on boldness and mood).
8. **Decide motion state** (preening, scanning, calling, etc.) based on mood and time-of-day.
9. **Trigger notebook entry generation** if warranted (see below).
10. **Write canonical state** back to Aviary DB and Bird records.
11. **Update last_tick_at** on the aviary.

### Drift engine

Drift is implemented as a bounded additive delta applied each tick. The delta per trait per tick is the sum of weighted inputs:

```
Δtrait = (
  w_presence   × presence_time_since_last_tick_seconds  +
  w_listen_in  × listen_in_duration_for_this_bird_seconds +
  w_offer      × offer_events_for_this_bird_count +
  w_settle     × settle_events_count  // settle has minimal trait-specific weight
) × global_speed_constant

trait_new = min(trait_max, trait_current + max(0, Δtrait))
```

Key invariants:
- `Δtrait` is always non-negative (monotonic toward expressive).
- `global_speed_constant` is calibrated so that a typical bird reaches measurable drift (Δ > 0.02 on any trait) after approximately 1 week of daily regular visits, and visible drift (Δ > 0.10) after approximately 3 weeks.
- Per-trait weights differ: presence-time affects boldness and social_warmth most; listen-in affects social_warmth and vocal_frequency; offer affects curiosity (and secondarily boldness).
- Plumage_saturation is driven by total presence-time with no decay, independently of the other traits.
- The exact weight constants and global_speed_constant are an implementation artifact of the calibration process; they are not in this plan but must be validated against the 1-week/3-week calibration targets in the test harness.

**Implementation note**: the drift function is unit-tested against the calibration targets. A CI test simulates 7 days of 30-minute daily sessions (with realistic presence-time distribution) and asserts that at least one personality trait shows Δ > 0.02. A separate test simulates 21 days and asserts Δ > 0.10. These tests are live from day one.

### Mood transition engine

Mood is an enum (wary, content, curious, drowsy, alert). Transitions are probabilistic, driven by a weighted input vector:

```
MoodTransitionInputs {
  time_of_day: float (0.0=midnight, 0.5=noon, 1.0=midnight)
  recent_offer_accepted: bool
  recent_alarm_from_neighbor: bool  // bird-to-bird contagion
  rain_active: bool
  wind_active: bool
  personality_boldness: float  // high boldness suppresses wary
  personality_social_warmth: float  // high warmth promotes content
}
```

Transition probability matrix is defined for each (current_mood, inputs) combination. Example:
- Wary → content: probability increases if time_of_day is morning and recent_offer_accepted and personality_boldness > 0.5.
- Content → drowsy: probability increases near dusk (time_of_day ~0.8).
- Any → wary: probability increases if recent_alarm_from_neighbor and personality_boldness < 0.5.

Mood persists across ticks; it is not reset to neutral at tick start. Each tick evaluates whether a transition fires based on current mood and inputs. Transition can happen at most once per tick per bird.

Mood persists across user sessions: the mood state stored at tick N is what the client reads at session start.

### Call-grammar runtime

Each species ships with a motif library: a set of 4–8 atomic call motifs (defined as WebAudio parameter sequences: frequency contour, duration, envelope shape, harmonic ratios). The call-grammar state stores a weight vector over these motifs, shaped by:
- `vocal_frequency` personality trait (higher = more frequent calls, more motifs active)
- Current mood (drowsy suppresses high-energy motifs; alert amplifies them)
- Time-of-day (dawn chorus: all motifs more active; night: only low-energy motifs active)

The call-grammar state is sent to the client as part of the snapshot. The client synthesizes calls at runtime using WebAudio, picking motifs according to the weights and combining them into variations. No two calls are identical.

Bird-to-bird interaction is modeled at the call-grammar level: a call event from one bird at high social_warmth has a probability (proportional to the neighbor bird's social_warmth) of triggering a response call within a short window (2–10 seconds). This is computed server-side and represented in the snapshot as a `pending_response_window` flag on the responding bird.

---

## 6. Sync Model

### Single canonical record

There is one canonical aviary state record per account in the Aviary DB. This record is the source of truth for all clients.

The simulation service is the only writer of personality vectors and mood state. The application server is read-only with respect to these fields. Clients never write to them under any code path.

### How multi-device sync works

Device A (laptop) and Device B (phone) both call `GET /aviary/snapshot`. Both receive the same canonical record. They see the same birds in the same moods with the same drift history.

Interaction events from both devices are written to the same append-only event log (keyed by account_id). The simulation tick consumes the log in chronological order. There is no merging of device states because there are no device states to merge.

### Conflict prevention

Conflicts are prevented by architecture, not resolved after they occur:

1. **No client writes personality state.** Clients write events; the server writes state. No two writers of personality state exist.
2. **Additive deltas only.** Even if two tick executions for the same aviary were to run concurrently (a bug), additive deltas commute; the result is the sum of both, not a race condition.
3. **Event log is append-only.** No event is ever overwritten. The tick always processes events in insertion order.

### Snapshot freshness

Clients pull a fresh snapshot on:
- Tab becoming visible after being hidden (visibilitychange event).
- Long render-frame gap (>5s between frames, e.g., laptop waking from sleep).
- Low-frequency keepalive: every 30 seconds while visible (exact cadence tuned during build).

The initial snapshot is inlined into the HTML response at the CDN edge so the first paint requires zero additional round-trips.

### Visitor sync

A visitor session is identical to an owner session for snapshot consumption, with two differences:
1. No interaction events are written by the visitor.
2. The `GET /aviary/visit/:token` endpoint validates the token and returns 410 if the invite is revoked or expired.

---

## 7. Frontend Rendering Pipeline

### Technology choices

- **Framework**: React (for UI chrome and state management); the aviary scene uses Canvas 2D or SVG (decision to be made by the render lead; Canvas is preferred for performance at 60fps with many simultaneous micro-motion layers).
- **Audio**: WebAudio API (no Web Audio Worklet is required for v1's motif complexity; standard AudioContext nodes are sufficient).
- **Build**: Vite with aggressive code-splitting.
- **State**: Lightweight client store (Zustand or similar); no full Redux for a product this size.

### Scene composition

The aviary scene is composed of discrete layers, back to front:

1. **Sky layer**: gradient that shifts with time-of-day.
2. **Background foliage layer**: static or slow-parallax SVG/bitmap.
3. **Weather layer**: rain particles or leaf-ripple CSS effects (removed in reduced-motion).
4. **Perch layer**: three perch zones (back, middle, front). Perches are static geometry.
5. **Bird layer**: one element per bird, positioned at its current perch zone.
6. **Ambient particle layer**: leaves, feathers, procedurally generated client-side at idle cadence.
7. **Foreground branch layer**: subtle parallax, soft depth cue.
8. **Top bar**: always above scene, fades on cursor stillness.

### Idle micro-motion

Each bird has a motion-state machine driven by the `perch_motion_state` field from the snapshot. States: preening, scanning, calling, still, alert-scan. Each state has a looped animation (frame sequence for Canvas; CSS keyframes for SVG) that runs continuously. The animation is never paused; on tab-hidden, rendering halts (requestAnimationFrame is not called) but state continues advancing via the server tick.

On snapshot arrival, the client reconciles the bird's new motion state with its current animation frame: if the state is unchanged, the animation continues uninterrupted; if the state changed, a short cross-fade transition (~200ms) blends to the new state animation.

### Initial frame "already in motion" rule

The rendering pipeline MUST NOT show a blank or static scene at any point during load. Implementation:

1. The CDN-inlined snapshot provides bird positions and motion states.
2. The client begins rendering at the current animation frame that corresponds to elapsed time since the snapshot's `snapshot_at` timestamp. If `snapshot_at` was 12 seconds ago and a preening cycle is 8 seconds, the client enters the preening animation at second 4 (12 mod 8).
3. Audio synthesis begins immediately on snapshot receipt; calls begin based on the `last_call_at` timestamp extrapolated forward.
4. If the inlined snapshot is absent (very rare: CDN miss), the client shows the quiet-field loading state (soft sky gradient, faint slow ambient leaf drift, no spinner) until the snapshot arrives via XHR.

### Return-greeting sequence

When the client detects a new session start (session_start event fired), it reads the bird with the highest `greeting_priority` field from the snapshot (this is a server-computed value based on boldness, mood, and time-since-last-session). That bird transitions into the greeting animation:

- Short absence (<30 min): glance animation (head-tilt toward camera, brief pause).
- Medium absence (30 min–4 hours): two-note call + step toward front perch.
- Long absence (>4 hours): longer call, head-turn, and if boldness is high, a perch change toward front.

Additional birds may greet with a randomized stagger of 200ms–2000ms after the first bird, probability proportional to their own boldness.

### Listen-in mix transition

On listen-in engage:
- Target bird's call volume ramps from ambient (100%) to listen-in level (150–180%) over 800ms.
- All other birds' call volumes ramp to ambient-quiet (40%) over the same 800ms.
- The ramp curve is exponential (perceptual volume match), not linear.

On listen-in disengage:
- All volumes return to ambient (100%) over 800ms.

No bird's call ever goes to 0. Silence is not an output of listen-in.

### Settle gesture rendering

On settle trigger:
- Lighting slowly warms and dims over 3 seconds (sky gradient shifts toward evening palette).
- Call volumes ramp down across all birds over the same 3 seconds.
- All birds gradually adopt drowsy or settled motion states.
- A 5-second undo window: any click in the aviary reverses the transition at the same rate.

### Reduced-motion mode

Detected via `window.matchMedia('(prefers-reduced-motion: reduce)')` or user settings toggle. In reduced-motion:
- Frame-by-frame animations are replaced by CSS cross-fades between pre-defined still pose images (4–6 poses per motion state per species).
- Cross-fade duration: 1.5–3 seconds (slow, calm).
- Flight transitions: cross-fade from one perch still pose to the other over 2 seconds.
- Ambient particles (leaves, feathers): disabled.
- Day/night color shifts: retained, slowed to 2× their normal transition time.
- Audio: unaffected (or captioned, per audio settings).
- Top-bar fade: retained.

The reduced-motion mode is designed as a distinct aesthetic, not a stripped fallback. The cross-fade pacing is deliberately calm and has its own visual character.

### Responsive layout

Scene width fills the viewport. Perch zones scale proportionally:
- Below 400px: perch zones compress; birds scale down to ~60% of desktop size. No bird is ever cropped.
- Above 1400px: perch zones widen with more inter-zone gap; bird size stays at 100%.
- Aspect ratio of the scene is maintained at all viewport widths by adjusting scene height proportionally.

The top bar is fixed at the top, always 100% viewport width, height constant (~44px).

---

## 8. Audio Pipeline

### Architecture overview

All audio is synthesized client-side using the WebAudio API. There are no audio files to download. The bundle carries motif parameter data (frequency contours, duration distributions, harmonic ratios) as compact JSON; this data is the "call grammar" and is small (<<100KB total for all species).

### Call synthesis

A call is synthesized as follows:

1. **Motif selection**: a motif is chosen from the bird's motif library according to the current weight vector from the call-grammar state.
2. **Variation sampling**: pitch, duration, and harmonic ratios are sampled from distributions centered on the motif's parameters. The sampling uses seeded pseudo-random values so that the same motif produces similar-but-never-identical calls each time.
3. **Synthesis**: an OscillatorNode chain is constructed: fundamental frequency + 2–4 harmonics with independently varying gain envelopes. The chain is assembled in a reusable node pool (pooled, not allocated per call) and played through a GainNode for mix control.
4. **Timing**: call onset is scheduled using `AudioContext.currentTime` for sample-accurate timing. The `last_call_at` field from the snapshot seeds the initial call schedule; subsequent calls are scheduled based on the bird's `vocal_frequency` and current mood.

### Chorus mixing

Multiple birds calling simultaneously are mixed at the AudioContext level. Each bird has a dedicated `GainNode` for its mix level. The master output is a `DynamicsCompressorNode` to prevent clipping in dense chorus moments.

The listen-in interaction adjusts `GainNode` values (see §7) without interrupting ongoing call synthesis.

### WebAudio node pool

To meet the "no memory growth over 30 minutes" budget:
- OscillatorNode and GainNode instances are pooled (10 per bird, pre-allocated at session start).
- Each call reuses a pool node: `start()` is called, `stop()` is scheduled at end of call, and the node is returned to the pool after stopping.
- AudioBufferSourceNode is not used (no recorded audio).
- The pool size is validated in CI: a 30-minute synthetic session must not exceed the initial pool allocation.

### WebAudio fallback

If `AudioContext` is unavailable or the audio context is suspended due to browser autoplay policy:
- Captions are enabled automatically.
- No audio plays. No recorded-audio fallback.
- The "graceful silence + captions" fallback is the complete fallback path.

Browser autoplay policy (user has not interacted): the AudioContext is created but kept suspended until the first user interaction (any click, tap, or keypress). On first interaction, `audioContext.resume()` is called. Until then, captions are shown.

### Call caption generation

Captions are generated from the motif and variation parameters at synthesis time:
- A small lookup maps motif identity + variation bin to a short naturalist phrase (e.g., "a soft three-note rise," "a low trill, paused, low trill again," "a single sharp call from the back perch").
- The phrase is displayed as small text near the calling bird, fading in at call onset and fading out 500ms after call end.
- Captions use CSS opacity transitions (compatible with reduced-motion: only the fade is reduced-motion-sensitive; in reduced-motion, captions appear and disappear without the fade).
- Caption text is generated at call synthesis time; it matches what was actually played.

---

## 9. Accessibility Surfaces

### Screen-reader narration

A live region (`aria-live="polite"`, `aria-atomic="false"`) is maintained in the DOM, hidden visually, updated with naturalist prose on a slow cadence:
- At idle: updated every 30–60 seconds with a scene description in field-notebook voice.
- On user-initiated events (offer reaction, settle, return-greeting): updated promptly (within 1 second of the event).

Prose examples (these are samples, not templates; actual text is generated from current state):
- "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- "pip accepted the seed. she pecked twice, then looked away."
- "wren has begun a low, slow call from the back perch."

Prose generation is a server-side or client-side function that reads the current snapshot and produces sentences in naturalist voice. It must not produce state-list text ("bird 1: content, perch: front"). The voice is identical to the field notebook voice.

High-frequency narration is explicitly prevented: no update fires more often than once per 15 seconds even on rapid events. This prevents queue overflow on screen readers.

User-initiated events get priority (they insert immediately into the live region regardless of the normal cadence timer).

### Keyboard navigation

Tab order:
1. Skip-to-aviary link (first tab stop, visually hidden unless focused).
2. Top bar items left to right: notebook icon, offer icon, settle icon, accessibility settings icon, account icon.
3. Aviary scene: Tab enters the scene, focusing the first bird. Arrow keys navigate between birds. Enter triggers listen-in on focused bird. Escape exits listen-in and returns focus to the aviary scene boundary.
4. Offer panel (when open): standard tab order through offer options, Escape closes.
5. Notebook panel (when open): scrollable, standard tab order through entries, Escape closes.

Focus indicators: a high-contrast soft-glow outline (2px solid, offset 2px, color tuned against both bright and dark aviary palettes by the visual designer). The indicator must be visible in both morning and night aviary states.

### Captions

See §8 for caption generation. Captions are toggled from accessibility settings (which are also reachable via keyboard from the top bar). The setting persists server-side in the account's accessibility settings.

### WCAG AA compliance

- All user-copy text (top-bar labels, settings panels, error surfaces, captions, narration-as-text) meets WCAG AA contrast (4.5:1 for normal text, 3:1 for large text).
- The aviary scene itself contains no user-copy text except captions; caption text is overlaid on the scene with a soft background treatment to ensure contrast regardless of the underlying aviary color.
- Contrast is validated in CI using automated contrast-checking against the full day/night palette range.

### Accessibility settings panel

Reachable from the top bar. Contains:
- Reduced-motion toggle (mirrors `prefers-reduced-motion` but can be overridden by user preference).
- Captions toggle.
- (No personality-vector display, ever.)

Accessibility settings use the matter-of-fact voice, not the naturalist voice.

---

## 10. Performance Budgets and Observability

### Bundle budget

Initial JS bundle: **<2MB gzipped** at first paint.

Enforcement:
- Webpack/Vite bundle analyzer runs in CI; build fails if gzipped bundle exceeds 2MB.
- Code-split boundaries: account settings, accessibility settings, visit-invitation flow, notebook panel.
- All lazy-loaded chunks are prefetched on idle after first paint.
- No recorded audio in the bundle.
- Bird visual assets: SVG-based where possible; compact bitmaps for complex plumage rendering (max 50KB per species, all species preloaded).
- Motif grammar data: <100KB JSON total.

### Time to first bird: <500ms

Measured from navigation start to first bird visible (using PerformanceObserver or a custom paint timing mark).

Implementation path:
- HTML response from CDN edge carries inlined initial snapshot in a `<script type="application/json">` block.
- Critical CSS is inlined in the HTML `<head>`.
- The render pipeline begins immediately on DOMContentLoaded without waiting for any async asset.
- The first bird is visible before any WebAudio context is created (audio starts on first user interaction per autoplay policy anyway).
- Font loading does not block first bird paint (system fonts for the aviary scene; top-bar font is web-loaded with `font-display: swap`).

Target: p50 <200ms, p95 <500ms on mid-tier mobile over 4G.

### 60fps idle motion

Runtime budget: 16ms per frame for the full scene on a 5-year-old mid-range laptop.

Implementation:
- Canvas 2D (preferred over DOM animation for this frame budget at 7 birds).
- Off-main-thread: ambient particle generation on a Worker; results posted to main thread each frame.
- No layout thrash: all bird positions are computed in JavaScript, written to Canvas once per frame, no DOM reflow in the hot path.
- `requestAnimationFrame` halted on tab hidden (visibilitychange to hidden); resumed on visible.
- Frame timing logged in CI via synthetic 30-minute session.

### No memory growth over 30 minutes

Enforced in CI via a synthetic session that:
1. Runs for 30 minutes of simulated time (1800 frames at 60fps).
2. Takes a memory snapshot at start and at end.
3. Asserts the delta is <10MB (heap growth).

Known allocation boundaries:
- WebAudio node pool: pre-allocated, bounded, no per-call allocation.
- Canvas: single ImageData buffer, reused each frame.
- Notebook: virtual scroll; only visible entries are in the DOM; off-screen entries are unmounted.
- Event log: client-side event queue flushed on each POST /aviary/events call; does not accumulate indefinitely.

### Observability

**Synthetic monitoring**: a small fleet of automated browsers runs the aviary on a schedule from 3 geographies. Each synthetic run checks:
- Time to first bird.
- 30s idle frame rate.
- Audio context creation success.
- Snapshot fetch latency.

**Real User Monitoring (RUM)**: aggregate-only metrics collected from all sessions:
- Page load timing (navigation start → DOMContentLoaded).
- First bird render timing (custom mark).
- Per-frame render time histogram.
- Audio context error rate.
- Simulation-tick latency (server-side metric: time from tick schedule to tick commit).
- Snapshot fetch latency (p50, p95, p99).
- Session duration histogram (anonymized; no per-account dimension).

**RUM privacy boundary**: no per-bird state, no per-account interaction history, no individual session identifiers in any metric. Metrics are keyed by geography and device class only.

**Error budget**: simulation-tick latency p99 alarms if > 5 seconds. Snapshot fetch p99 alarms if > 2 seconds. First bird render p95 alarms if > 500ms.

**Deliberately not instrumented**: per-account drift rates, individual bird mood histories, per-user presence-time, per-bird offer acceptance rates. These live in the per-account simulation database and are never read by the telemetry pipeline.

---

## 11. Rollout

### v1 launch configuration

- Start: all new accounts begin with exactly 2 birds from the species pool. The 2 starters are selected by the server (not the user) using a randomized but balanced draw from the pool.
- No birds are unlocked at launch beyond the 2 starters.
- New-bird offers (3rd, 4th, etc.) become available based on aviary age. Initial pacing targets: 3rd bird offer at ~90 days, 4th at ~180 days, 5th at ~365 days, 6th and 7th at the team's discretion post-launch. These thresholds are implementation constants in the simulation service, easily adjusted without a deploy.

### Instrumented from day one

The following are live at launch:
- All synthetic monitoring checks.
- RUM pipeline (aggregate-only, as specified).
- Simulation-tick latency alarms.
- Drift calibration test suite in CI (1-week and 3-week targets).
- Accessibility CI checks (contrast, keyboard navigation flow test).
- Bundle size CI gate.
- Memory growth CI test.

### Ramp plan

Week 1: closed beta (team + 20 external testers). Week 2–4: invite-based beta (testers invite friends). Week 5+: open registration. The ramp is paced by operational comfort, not by feature gates. All v1 features ship to all users on day 1 of each cohort.

There are no feature flags in the v1 product surface (the product is too small and the principle-violations are too costly to allow partial rollouts of feature decisions that the design philosophy depends on).

The visit-invitation feature is on by default (accounts can invite) but invites are off by default per account (no one receives an invitation unless the host deliberately sends one). This is not a feature flag; it's the permanent default.

### Post-launch calibration

After launch, the following are reviewed within the first 30 days:
- Drift calibration: aggregate (anonymized) p50 and p95 drift velocity across all accounts. Target: measurable drift at ~1 week, visible drift at ~3 weeks. If drift is moving too fast, reduce `global_speed_constant`. If too slow, increase it. This is a server-side configuration change, no deploy required.
- Presence-time activity window: the window for the pointer-or-key check (how long after the last activity presence is still counted). Review the distribution of presence-event densities to tune.
- Notebook entry frequency: review whether entries feel too frequent or too sparse for median users. The entry-generation probability per tick is a configuration constant.
- Call cadence: review whether the vocal_frequency mapping to call frequency feels right. Adjust motif-weight-to-call-interval mapping.

---

## 12. Risks

### Drift calibration

**Risk**: the drift function moves personality too fast (Tamagotchi feel) or too slow (users feel nothing changes). The 1-week/3-week calibration target is not self-enforcing.

**Mitigation**: 
- CI tests enforce the calibration target against simulated sessions. If the target drifts (pun intended) during development, the CI fails.
- The `global_speed_constant` is a server-side configuration value, adjustable without a deploy.
- Post-launch, the aggregate drift velocity metric (anonymized, no per-account data) is reviewed at 30 days.
- The known danger is that beta users (who test more intensively than real users) will skew calibration. The CI simulation uses realistic presence-time distributions, not power-user distributions.

### Sync correctness

**Risk**: a bug introduces client-side personality state writes (violating the server-canonical rule), or the event log is processed out of order, producing personality corruption.

**Mitigation**:
- The `PUT /aviary/personality` endpoint does not exist. There is no code path on the client that writes personality state. Code review enforces this; any attempt to add such a path is a blocking review comment.
- The event log is append-only with insertion-order processing. The tick processes events with `ORDER BY occurred_at ASC`.
- Integration tests simulate two devices generating concurrent events and assert the final personality vector is the additive sum (not a race condition winner).
- If a tick fails mid-execution, the partial write is rolled back (transaction). The tick retries from the last committed state on the next cycle.

### Audio uncanniness

**Risk**: procedural calls are recognizable to the user as procedural — mechanical, too random, or not bird-like enough. Alternatively, calls are too similar to each other across moods and the "recognizable by ear" goal fails.

**Mitigation**:
- Motif library is designed with a sound designer (or consultant), not by engineers alone. Each species motif is based on a real bird call family; this is the ground-truth constraint.
- Variation sampling parameters (pitch jitter, duration jitter, harmonic gain variance) are calibrated by ear against recordings of real bird variation.
- "Recognizable across moods" is a user-test criterion: a blind test where testers identify which bird is calling across three different mood renderings of the same bird. This test runs in closed beta before launch.
- The 7-bird cap is a safety valve: if more species were added, call recognizability would be harder to maintain.

### Accessibility regression

**Risk**: a visual feature ships that breaks the screen-reader narration contract (e.g., state changes are not reflected in the live region), or reduced-motion mode silently breaks on a browser update.

**Mitigation**:
- Accessibility surfaces are tested in CI using axe-core (automated) and manual testing with at least one screen reader (NVDA or VoiceOver) in the release checklist.
- The live region is tested: a CI test renders the aviary with mocked state transitions and asserts that live region text updates within the expected window.
- Reduced-motion mode is tested by setting `prefers-reduced-motion` in the test browser and asserting no frame-by-frame animation CSS properties are active.
- The accessibility settings panel is covered by keyboard-navigation tests.
- These tests run on every PR, not only on release branches.

### "Notice, never announce" drift

**Risk**: a well-meaning contributor adds a toast, a badge, or a status message that breaks the core design principle. This is the highest-probability failure mode and the hardest to catch after the fact.

**Mitigation**:
- This plan names the principle explicitly and the rationale is in the PRD. New contributors read the PRD.
- Code review has a specific checklist item: "Does this PR add any textual announcement, toast, badge, or streak surface?" If yes, it is a blocking comment.
- The design review process (not just engineering review) approves all new visible UI surfaces.
- A design audit is scheduled at 60 days post-launch to check for any "harmless" additions.

### Presence signal gaming

**Risk**: a sophisticated user discovers that synthetic pointer events keep presence-time accumulating without actually watching the aviary. This would corrupt their drift signal (making birds drift faster than calibrated) without materially harming other users.

**Mitigation**:
- The presence signal is defined and enforced on the client; the server trusts it (with anomaly detection).
- Server-side anomaly detection: a presence-time accumulation rate that exceeds a calibrated maximum (e.g., 24 hours of presence per day is impossible) triggers a soft cap and a flag for review.
- The consequence of gaming is only that the user's own birds drift faster. This is not a product safety issue; it's a calibration issue for one account.
- The design philosophy is not to punish or police; if a user games presence, their birds just change faster, and that's their relationship. The soft cap prevents truly absurd outcomes.

### Performance regression on slow connections

**Risk**: the <500ms first-bird metric is met in lab conditions but not in the field on slow connections or in geographies with high CDN latency.

**Mitigation**:
- Synthetic monitoring from 3 geographies (including at least one with historically higher latency) catches regressions.
- The CDN-inlined snapshot strategy is the key mitigation: the initial state is delivered with the HTML, not as a separate round-trip.
- The bundle size CI gate prevents inadvertent growth that would push load time over the threshold.
- RUM first-bird-render p95 is an alarm; if it exceeds 500ms in real users, the engineering team investigates before the next release.
