# Pocket Aviary — Implementation Plan (v1)

## 1. Scope

### In scope for v1

- **Browser-only client** — single-page application, responsive from narrow phone to wide desktop, last two major versions of Chrome, Safari, Firefox, Edge.
- **Single-user accounts** — email magic-link auth, per-device session tokens, synthetic UUID account IDs, soft-then-hard deletion (30-day window).
- **Aviary simulation** — server-side tick (~1/min), personality vector drift, mood transitions, call-grammar scheduling, bird-to-bird interactions, day/night cycle, ambient weather.
- **Bird engine** — 2 starter birds per account, cap of 7, ~6 species pool, procedural personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic-toward-expressive drift.
- **Interactions** — return-greeting (procedural, absence-aware), listen-in (gradual mix re-balance), offer (seed / song fragment / still pool, per-bird cooldown), settle (opt-in soft session-end with 5s undo), presence accounting (3-signal conjunction).
- **Field notebook** — auto-generated naturalist prose, read-only, sparse (~1 entry per few days), scrollable history.
- **Audio pipeline** — procedural call synthesis via WebAudio, chorus mixing, listen-in mix ramping, graceful silence+captions fallback.
- **Visual rendering** — single horizontal scene, 3 perch zones, idle micro-motion, ambient leaf/feather drift, day/night palette cycle, top-bar chrome with auto-fade.
- **Accessibility** — screen-reader narration (naturalist prose, slow cadence), reduced-motion mode (designed cross-fade surface, not stripped fallback), call captions, WCAG AA contrast, full keyboard navigation.
- **Multi-device sync** — server-canonical state, snapshot-pull model, additive server-authored deltas, no last-write-wins.
- **Social (optional, off by default)** — visit invitations by email, read-only ambient view, no co-presence, revocable, 30-day expiry, silent visit log.
- **Account management** — export (JSON snapshot via email link), session revocation, email change with verification.
- **Observability** — aggregate-only telemetry (request counts, latencies, error rates, render-frame timing, audio errors), synthetic perf checks, RUM.

### Out of scope (non-goals, absolute)

- Native mobile apps (iOS, Android).
- All gamification: achievements, streaks, levels, scores, badges, calendars, XP, ranks, tiers, visit-frequency surfaces, "birds adopted" counters.
- Tamagotchi mechanics: death, hunger, distress, happiness decay, custodial obligation.
- Social network surfaces: profiles, follows, public feeds, discovery, comments, leaderboards, co-presence, show-off mode.
- Push notifications, email notifications about aviary state, any outbound engagement ping.
- Shared aviaries, multi-aviary accounts, customizable scenes.
- Payments, billing, paid tiers.
- Recorded audio at any level.
- Personality vector numerical exposure to the user (no stats panel, no debug view, no toggle).

---

## 2. Architecture

### Service shape

Three services plus infrastructure:

1. **API Gateway / BFF** — thin HTTP/WebSocket edge. Authenticates requests, routes to downstream services, serves the initial state snapshot inline with HTML for fast first paint. Terminates TLS, enforces rate limits.

2. **Simulation Service** — the canonical state owner. Runs the per-account tick on a ~1-minute cadence. Consumes the interaction event log, computes personality deltas, transitions moods, schedules calls, advances the day/night cycle, generates notebook entries, writes the new canonical aviary state. This is the only writer of personality vectors. Runs as a fleet of workers partitioned by account-ID hash.

3. **Accounts Service** — auth (magic-link issuance, token management), account CRUD, session management, visit-invitation flow, export generation, deletion lifecycle.

Supporting infrastructure:

- **Event Log** — append-only, per-account, ordered. Clients write interaction events here; the simulation tick consumes them. Implemented as a partitioned log (Kafka or equivalent) keyed by account UUID.
- **State Store** — the canonical aviary state per account. A document store (PostgreSQL with JSONB, or DynamoDB with a document model) holding the current snapshot: birds, personality vectors, moods, positions, call schedules, day/night phase, notebook entries.
- **Snapshot Cache** — a read-through cache (Redis or CDN edge) holding the latest snapshot per account, invalidated on each tick write. Clients pull from here.
- **Email Service** — transactional email for magic links, visit invitations, export delivery. Thin wrapper over a transactional email provider.
- **Telemetry Pipeline** — aggregate-only. Receives operational metrics (latencies, error counts, render-frame timings) from all services. Hard pipeline boundary: never reads from the state store or event log. Separate data store, separate access controls.

### Client/server split

| Responsibility | Server | Client |
|---|---|---|
| Personality vector storage & mutation | ✅ (only writer) | ❌ (never) |
| Mood transitions | ✅ | ❌ (reads from snapshot) |
| Drift computation | ✅ | ❌ |
| Call scheduling & grammar | ✅ (schedules next-call metadata) | ✅ (synthesizes audio from schedule) |
| Notebook entry generation | ✅ | ❌ (reads from snapshot) |
| Presence detection | ❌ | ✅ (3-signal conjunction, emits events) |
| Interaction event emission | ❌ | ✅ (writes to event log) |
| Rendering | ❌ | ✅ |
| Audio synthesis | ❌ | ✅ (WebAudio) |
| Snapshot interpolation | ❌ | ✅ |
| Day/night palette | ✅ (phase in snapshot) | ✅ (renders palette from phase) |

### Render pipeline boundary

The client's render pipeline takes a snapshot (current bird positions, moods, call schedules, day/night phase, ambient state) and produces frames. Between snapshots, the client interpolates: bird positions tween smoothly, idle micro-motion runs from a local state machine seeded by mood, ambient leaves/feathers are generated client-side. The render pipeline never makes simulation decisions — it only visualizes what the server has already decided.

---

## 3. Data Model

### Account

```
Account {
  id: UUID (synthetic, generated at creation)
  email_encrypted: bytes (AES-256, single storage location)
  created_at: timestamp
  deletion_state: enum {active, soft_deleted}
  deletion_requested_at: timestamp?
  settings: {
    visit_notifications_enabled: bool (default false)
    reduced_motion_override: bool?
    captions_enabled: bool?
  }
}
```

### Session Token

```
SessionToken {
  token_hash: string
  account_id: UUID
  device_label: string
  issued_at: timestamp
  expires_at: timestamp
  revoked: bool
}
```

### Bird

```
Bird {
  id: UUID (stable, never changes)
  account_id: UUID
  species: enum (from ~6 species pool)
  name: string (user-assigned, renameable)
  adopted_at: timestamp
  personality: PersonalityVector
  mood: MoodState
  current_perch: enum {front, middle, back}
  idle_state: enum (preening, scanning, head_tilt, body_shuffle, resting, calling)
  call_schedule: CallSchedule
  last_greeting_at: timestamp?
  offer_cooldown_until: timestamp?
}
```

### PersonalityVector

```
PersonalityVector {
  boldness: float [0.0, 1.0]
  social_warmth: float [0.0, 1.0]
  vocal_frequency: float [0.0, 1.0]
  plumage_saturation: float [0.0, 1.0]
  curiosity: float [0.0, 1.0]
}
```

All values server-written only. Seed values for new birds drawn from species-specific distributions. Drift is additive, monotonic toward expressive (values increase on positive presence, never decrease on neglect).

### MoodState

```
MoodState {
  current: enum {wary, content, curious, drowsy, alert}
  since: timestamp
  transition_reason: string (internal, not user-visible)
}
```

Mood persists across sessions. Resets on a daily-ish cadence modulated by interactions, time of day, ambient events, and the bird's personality vector.

### InteractionEvent (append-only log)

```
InteractionEvent {
  id: UUID
  account_id: UUID
  bird_id: UUID?
  event_type: enum {
    presence_ping,
    listen_in_start,
    listen_in_end,
    offer_seed,
    offer_song,
    offer_pool,
    settle,
    settle_undo,
    tab_open,
    tab_close
  }
  payload: JSON (event-specific metadata)
  client_timestamp: timestamp
  server_received_at: timestamp
}
```

### NotebookEntry

```
NotebookEntry {
  id: UUID
  account_id: UUID
  created_at: timestamp
  prose: string (naturalist voice, lowercase, present-tense)
  source_event_ids: UUID[] (internal provenance, not user-visible)
}
```

### VisitInvitation

```
VisitInvitation {
  id: UUID
  host_account_id: UUID
  visitor_email_encrypted: bytes
  token_hash: string
  status: enum {pending, active, revoked, expired}
  created_at: timestamp
  activated_at: timestamp?
  expires_at: timestamp (created_at + 30 days)
}
```

### VisitLog

```
VisitLog {
  id: UUID
  host_account_id: UUID
  invitation_id: UUID
  visitor_email_encrypted: bytes
  visit_started_at: timestamp
  visit_ended_at: timestamp?
  duration_approx_seconds: int?
}
```

---

## 4. API Surface

### Authentication

| Endpoint | Method | Description |
|---|---|---|
| `/auth/magic-link` | POST | Request magic link. Body: `{email}`. Rate-limited per email. |
| `/auth/verify` | POST | Verify magic link token. Returns session token. |
| `/auth/sessions` | GET | List active sessions for the account. |
| `/auth/sessions/:id` | DELETE | Revoke a specific session. |
| `/auth/logout` | POST | Revoke current session. |

### Account

| Endpoint | Method | Description |
|---|---|---|
| `/account` | GET | Account settings and metadata. |
| `/account` | PATCH | Update settings (visit notifications, accessibility overrides). |
| `/account/email` | POST | Initiate email change. Sends verification to new address. |
| `/account/email/verify` | POST | Complete email change. |
| `/account/export` | POST | Generate JSON export, email download link. |
| `/account/delete` | POST | Initiate soft deletion. |
| `/account/recover` | POST | Cancel pending deletion (within 30-day window). |

### Aviary

| Endpoint | Method | Description |
|---|---|---|
| `/aviary/snapshot` | GET | Current state snapshot. Returns full aviary state. Pulled on tab-open, visibility change, and keepalive (~30s). |
| `/aviary/events` | POST | Submit interaction event(s). Body: `{events: InteractionEvent[]}`. Append-only. |
| `/aviary/birds/:id/name` | PATCH | Rename a bird. |

The snapshot response includes:

```json
{
  "snapshot_version": 142,
  "timestamp": "2026-05-26T10:30:00Z",
  "day_phase": "morning",
  "day_phase_progress": 0.35,
  "ambient_weather": null,
  "birds": [
    {
      "id": "uuid",
      "name": "Pip",
      "species": "warbler",
      "mood": "content",
      "perch": "front",
      "idle_state": "preening",
      "idle_state_progress": 0.6,
      "call_schedule": {
        "next_call_at": "2026-05-26T10:30:45Z",
        "motif_hint": "three_note_rise",
        "intensity": 0.7
      },
      "plumage_hint": {
        "saturation_level": 0.65
      }
    }
  ],
  "notebook": {
    "entries": [
      {"id": "uuid", "created_at": "...", "prose": "pip greeted before wren today..."}
    ],
    "has_more": true,
    "cursor": "..."
  }
}
```

Note: personality vector values are **never** included in the snapshot. The client never receives them.

| Endpoint | Method | Description |
|---|---|---|
| `/aviary/notebook` | GET | Paginated notebook entries. Query: `?cursor=...&limit=20`. |

### Offer

| Endpoint | Method | Description |
|---|---|---|
| `/aviary/offer` | POST | Submit an offer. Body: `{type: "seed"|"song"|"pool", bird_id?: UUID}`. Server validates cooldown, writes event, returns reaction hint. |

### Visit / Social

| Endpoint | Method | Description |
|---|---|---|
| `/visits/invite` | POST | Create invitation. Body: `{visitor_email}`. Sends email with one-time link. |
| `/visits/invitations` | GET | List outstanding and active invitations. |
| `/visits/invitations/:id` | DELETE | Revoke an invitation. |
| `/visits/log` | GET | Visit log (who visited, when, duration). |
| `/visits/:token/snapshot` | GET | Visitor's read-only snapshot of host's aviary. Validates token, checks not revoked/expired. |

The visitor snapshot endpoint returns the same aviary state as the host's, minus the notebook and account settings. The visitor's client renders identically but disables all interaction endpoints.

### WebSocket (optional, for live updates)

A WebSocket connection at `/aviary/stream` pushes snapshot deltas to connected clients. This is an optimization over polling — the client can fall back to polling the snapshot endpoint if WebSocket is unavailable. The server pushes a new snapshot after each tick for connected accounts.

---

## 5. Simulation Engine Design

### Tick lifecycle

The simulation tick runs per-account at ~1-minute cadence. Each tick:

1. **Read event log** — consume all unprocessed interaction events since the last tick for this account.
2. **Update presence accumulator** — aggregate presence-ping events into presence-time for the current window.
3. **Compute personality drift** — apply the drift function to each bird's personality vector based on accumulated presence-time and interaction events.
4. **Transition moods** — evaluate mood transition rules for each bird based on recent interactions, time of day, ambient events, and personality.
5. **Schedule calls** — update each bird's call schedule based on vocal_frequency trait, current mood, and bird-to-bird interaction rules.
6. **Advance day/night** — update the day phase based on the account holder's local timezone.
7. **Evaluate ambient weather** — probabilistically trigger weather events (rain, wind) at the configured frequency (~few times per week).
8. **Generate notebook entries** — evaluate whether any event in this tick window is noteworthy enough for a notebook entry. Apply sparsity rules.
9. **Process bird-to-bird interactions** — evaluate chorus triggers, mood contagion (wary spreading), call responses.
10. **Write canonical state** — atomically write the updated aviary state to the state store.
11. **Invalidate snapshot cache** — signal the cache layer to serve the new snapshot.
12. **Push to connected clients** — if any WebSocket clients are connected for this account, push the delta.

### Drift function

The drift function is a low-pass filter over presence-and-interaction signals:

```
drift_delta(bird, events, presence_time) = {
  boldness: α_b * presence_time + β_b * count(events where type=offer near bird),
  social_warmth: α_s * presence_time + β_s * count(events where type=listen_in and bird=target),
  vocal_frequency: α_v * presence_time + β_v * count(events where type=listen_in and bird=target),
  plumage_saturation: α_p * presence_time,
  curiosity: α_c * count(events where type=offer and bird accepted)
}
```

Where α and β coefficients are small enough that:
- One week of regular visits produces measurable drift in instruments (numerical change detectable by test harness).
- Three weeks of regular visits produces drift visible to the user (observable behavioral/visual change).
- A single session never produces visible drift.

**Monotonicity rule**: drift deltas are clamped to `max(0, computed_delta)`. Traits never decrease. On neglect, the delta is simply zero — the bird stays where it is, it doesn't regress.

**Calibration process**: During build, run a simulation harness with synthetic user profiles (light, moderate, heavy usage patterns) and verify:
- Light user (2 sessions/week, 5 min each): measurable instrument drift at 2 weeks, visible drift at 5 weeks.
- Moderate user (daily, 10 min): measurable at 1 week, visible at 3 weeks.
- Heavy user (multiple sessions/day, 20+ min): measurable at 4 days, visible at 2 weeks, but not saturated at 3 months.

### Mood transitions

Mood is a state machine per bird with weighted transitions:

```
States: wary, content, curious, drowsy, alert

Transition weights are a function of:
  - Recent interactions (offer accepted → content; listen_in → curious/content)
  - Time of day (dusk → drowsy; dawn → alert; midday → content/curious)
  - Ambient events (rain → slight wary or drowsy; wind → slight alert or wary)
  - Personality vector (high boldness → lower weight on wary transitions;
    high curiosity → higher weight on curious transitions)
  - Bird-to-bird (nearby bird in wary → slight weight toward wary for neighbors)
```

Mood transitions are evaluated each tick but have a minimum dwell time (~5 minutes) to prevent rapid oscillation. The transition is probabilistic, not deterministic — the same inputs don't always produce the same transition.

### Call-grammar runtime

Each species has a motif library — a set of call primitives (note sequences, trills, rises, falls, pauses) parameterized by:

- **Pitch range** — derived from species defaults, modulated slightly by vocal_frequency.
- **Timing** — inter-note intervals, overall duration, shaped by mood (wary calls are shorter, sharper; content calls are longer, softer).
- **Intensity** — how loud/prominent, shaped by mood and listen-in state.

The call scheduler on the server decides *when* a bird will next call and *what motif class* it will use. The client receives this metadata in the snapshot and synthesizes the actual audio at call time using the motif parameters. This means:

- The server doesn't generate audio.
- The client doesn't decide when to call.
- Two clients seeing the same snapshot will synthesize different-but-similar calls (procedural variation), which is correct — the calls are meant to vary.

**Chorus mechanic**: When two or more birds with high vocal_frequency are scheduled to call within a short window, the server marks the overlap as a chorus event. The client's audio mixer handles the overlap naturally — procedural calls layered at runtime produce a real chorus without phase-canceling artifacts.

### Bird-to-bird interaction

Evaluated each tick:

- **Call response**: If bird A calls and bird B has high social_warmth, bird B's next-call time is pulled earlier (a response).
- **Mood contagion**: If bird A enters wary, nearby birds (same or adjacent perch) get a small weight toward wary in their next mood evaluation.
- **Chorus emergence**: If 2+ birds with vocal_frequency > threshold are calling in the same window, mark as chorus. The client renders the overlap.

### Notebook entry generation

The notebook entry generator evaluates each tick for noteworthy events:

- First-greeter changes (Pip greeted first today, first time this week).
- Unusual mood states (a bird staying wary for an unusually long stretch).
- Ambient events coinciding with behavior (rain during a long quiet period).
- Offer patterns (a bird accepting a seed for the first time in a while).
- Drift milestones (internal only — the generator knows a trait crossed a threshold, but the entry describes the *behavioral consequence*, not the number).

**Sparsity enforcement**: Even when a noteworthy event occurs, the generator applies a cooldown (~2-3 days between entries for a regularly-visited aviary). The generator maintains a "noteworthiness score" and only emits when the score exceeds a threshold and the cooldown has elapsed. This prevents active users from getting a flood of entries.

**Prose generation**: Entries are generated from templates parameterized by bird names, species, mood, perch, time of day, and the specific event. The template library is curated (not generated by an LLM) to maintain the naturalist voice. Example templates:

```
"{name} greeted before {other_name} today, first time this {period}."
"{name} is fluffed against the {weather_hint} air, watching the {perch} perch. {call_description}."
"a long stretch of quiet this {time_of_day}. {name} {idle_description} without looking up."
```

The template library is a finite, reviewed set — every entry the system can produce has been read by a human and approved for voice.

---

## 6. Sync Model

### Architecture: server-canonical, client-reads

The sync model is a direct consequence of the architecture:

1. **Server is the only writer** of personality vectors, moods, bird positions, and notebook entries.
2. **Clients write interaction events** to the append-only event log. Events are facts ("user listened in to Pip for 3 minutes"), not state mutations ("set Pip's social_warmth to 0.62").
3. **The simulation tick** consumes events in order and computes additive deltas. No client-submitted absolute values.
4. **Clients pull snapshots** and render. Between snapshots, clients interpolate locally.

### Conflict prevention

There are no sync conflicts in the traditional sense because:

- Two clients for the same account both read the same canonical state.
- Two clients both writing interaction events is fine — events are append-only and the tick processes them in server-received order.
- No client ever writes personality state, so there's nothing to conflict over.

**The anti-pattern this prevents**: If clients could write personality state, a laptop session and a phone session could each submit their own version of the vector, and last-write-wins would silently discard one session's drift. The additive-delta model makes this unreachable.

### Snapshot delivery

- **On tab open / navigation**: Client requests `/aviary/snapshot`. The API gateway serves the cached snapshot (or fetches from state store on cache miss). Target: response in <100ms from CDN edge.
- **On visibility change** (tab becoming visible after being hidden): Client pulls a fresh snapshot to catch up on ticks that occurred while hidden.
- **On render-frame gap detection**: If the client detects a gap >2x the expected frame interval (e.g., laptop waking from suspend), it pulls a fresh snapshot.
- **Keepalive**: While the tab is visible, the client polls the snapshot endpoint at ~30-second intervals (or receives pushes via WebSocket).

### Presence event delivery

The client emits `presence_ping` events at ~1-minute intervals while the 3-signal conjunction holds (visibilityState=visible, window focused, recent pointer/key activity). These events are batched with other interaction events and sent to `/aviary/events`. The server's tick consumes them for drift computation.

When any of the three signals drops (tab hidden, window loses focus, no activity for the calibrated window), the client stops emitting presence pings. The tick sees the gap and stops accumulating presence-time.

---

## 7. Frontend Rendering Pipeline

### Scene composition

The aviary scene is composed of layers:

1. **Background** — sky gradient (day/night phase), distant foliage. Static or very slow animation.
2. **Mid-ground** — perch structures (front, middle, back), branches. Static geometry.
3. **Birds** — animated sprites/procedural visuals on perches. The primary animated elements.
4. **Foreground** — occasional branch/leaf passing through frame. Ambient ornament animation.
5. **Ambient particles** — leaves, feathers drifting at slow random intervals. Client-generated, not simulation-driven.
6. **Weather overlay** — rain particles, wind-ripple effects on foliage. Activated by ambient_weather in snapshot.
7. **Top bar** — UI chrome layer, fades to near-transparent after cursor stillness.

### Rendering technology

**Decision**: Canvas 2D or lightweight WebGL (e.g., PixiJS). The scene complexity is low enough that Canvas 2D is sufficient for 7 birds + ambient effects at 60fps. WebGL is an option if visual fidelity demands it, but the calm palette and simple sprites don't require it. The choice should be made based on the visual designer's asset specifications during the first sprint.

Bird visuals: small SVG sprites or compact bitmap spritesheets per species, with pose variants for each idle state (preening, scanning, head-tilt, resting, calling). Plumage saturation is expressed as a CSS filter or sprite variant selection based on the `plumage_hint.saturation_level` from the snapshot.

### Idle micro-motion

Each bird runs a local idle-state machine seeded by its current mood:

```
idle_states: preening, scanning, head_tilt, body_shuffle, resting, calling
transition_cadence: 3-8 seconds (mood-dependent)

mood → idle_state weights:
  wary:    scanning(0.4), resting(0.3), body_shuffle(0.2), head_tilt(0.1)
  content: preening(0.4), resting(0.3), body_shuffle(0.2), scanning(0.1)
  curious: head_tilt(0.4), scanning(0.3), body_shuffle(0.2), preening(0.1)
  drowsy:  resting(0.6), body_shuffle(0.3), scanning(0.1)
  alert:   scanning(0.5), head_tilt(0.3), body_shuffle(0.2)
```

Transitions between idle states are animated (e.g., a bird shifting from preening to scanning has a short transition animation). The idle state machine runs entirely client-side — the server's snapshot provides the current idle_state and progress, and the client continues from there.

### Transitions

- **Perch changes**: When the server's tick moves a bird between perches, the snapshot reflects the new perch. The client animates the bird moving to the new perch — a short flight or hop animation over 1-2 seconds.
- **Mood changes**: No explicit visual transition. The bird's idle-state weights shift, and the next idle-state transition reflects the new mood. The user reads the mood change from the motion, not from a visual effect.
- **Day/night**: Continuous palette interpolation. The snapshot provides `day_phase` and `day_phase_progress`; the client smoothly shifts the background gradient, lighting warmth, and ambient brightness.
- **Settle**: A slow (3-5 second) lighting transition to evening palette, call volume ramping down. Reversible within 5 seconds by any click.

### Reduced-motion mode

When `prefers-reduced-motion` is set or the user opts in via accessibility settings:

- Idle micro-motion is replaced by slow cross-fades between still poses (2-3 second cross-fade, 5-8 second hold per pose).
- Perch transitions become cross-fades between perch positions rather than flight animations.
- Ambient leaf/feather drift is removed entirely.
- Day/night palette transitions remain but are slowed (10-second cross-fade instead of continuous).
- Weather effects are simplified: rain becomes a static overlay with slow opacity change rather than animated particles.
- All other functionality is preserved: calls play, moods change, drift continues, notebook updates.

### First-frame strategy

On initial load:

1. HTML is served with the aviary's background color inline (the "quiet field" — soft sky color from the current day phase, estimated from server time if the user's timezone isn't yet known).
2. JS bundle loads and requests the snapshot.
3. On snapshot receipt, birds are placed at their current positions in their current idle states, and rendering begins immediately — no entry animation, no fade-in, no spinner.
4. If the snapshot takes >500ms (slow connection), the quiet field persists. No spinner. The quiet field reads as "the aviary is here" rather than "the app is loading."

### Responsive layout

- **Narrow phone viewport** (<480px): Scene compresses horizontally. Perch zones stack closer together. Birds remain fully visible, never cropped.
- **Medium viewport** (480-1024px): Standard layout. Comfortable perch spacing.
- **Wide desktop viewport** (>1024px): Scene widens with more space between perches. Background foliage extends.
- **Aspect ratio**: The scene maintains a minimum aspect ratio. On very tall/narrow viewports, the scene is letterboxed (quiet field above/below) rather than cropped.

---

## 8. Audio Pipeline

### Procedural call synthesis

Each species has a **motif library** — a set of call primitives defined as WebAudio node graphs:

```
Motif {
  name: string (e.g., "three_note_rise", "low_trill", "sharp_alarm")
  notes: Note[]
  parameters: {
    base_pitch: float (Hz, species-specific)
    pitch_variance: float (±Hz, randomized per call)
    tempo: float (notes per second, mood-modulated)
    envelope: {attack, decay, sustain, release} (seconds)
    timbre: {oscillator_type, harmonics[]} (species-specific)
  }
}

Note {
  pitch_offset: float (semitones from base)
  duration: float (seconds)
  volume: float (0-1)
  vibrato: {rate, depth}?
}
```

At call time, the client:
1. Selects a motif from the species library based on the `motif_hint` in the snapshot.
2. Randomizes parameters within the motif's variance ranges (pitch, timing, slight timbre shifts).
3. Constructs a WebAudio node graph: oscillator → gain envelope → master gain → destination.
4. Schedules the notes and plays.

This ensures no two calls are ever identical, even for the same bird in the same mood using the same motif.

### Chorus mixing

The audio mixer maintains a gain node per bird. At any given time:

- **Ambient mix** (default): All birds at equal gain, normalized so the total output doesn't clip.
- **Listen-in mix**: Focused bird's gain ramps up over 2 seconds; other birds' gain ramps down to ~30% over 2 seconds. Other birds never go silent.
- **Disengage**: Reverse ramp over 2 seconds back to ambient mix.

When multiple birds call simultaneously (chorus), the mixer handles natural overlap. Because calls are procedural and short (1-5 seconds), overlap is brief and the mixer doesn't need complex ducking — simple gain normalization is sufficient.

### Listen-in implementation

1. User clicks/taps/keyboard-focuses a bird.
2. Client sends `listen_in_start` event to server.
3. Client begins audio mix ramp: focused bird gain → 1.0, others → 0.3, over 2 seconds (ease-in-out).
4. On disengage (click again, focus different bird, click empty space, keyboard focus moves away):
   - Client sends `listen_in_end` event.
   - Client ramps all gains back to ambient over 2 seconds.

### WebAudio fallback

If WebAudio is unavailable (AudioContext creation fails, browser doesn't support it, permission denied):

- The aviary runs in silence.
- Call captions are enabled by default (overriding the user's caption setting).
- No recorded audio fallback. The "no recorded audio" rule is unconditional.
- A small, matter-of-fact notice in accessibility settings explains that audio is unavailable and captions are active.

### Audio context lifecycle

- AudioContext is created on the first user gesture (browser autoplay policy).
- Before the first gesture, the aviary renders visually but in silence. The first bird greeting may be visual-only if the user hasn't interacted yet.
- On tab hidden: AudioContext is suspended (saves battery). Resumed on visibility change.
- On settle: Audio gain ramps to near-silence over 3 seconds. AudioContext remains active but quiet.

### Audio buffer management

- Oscillator nodes are created per-call and disconnected after playback. No persistent buffer allocation.
- Gain nodes per bird are persistent (7 max, created once on AudioContext init).
- No memory growth: all per-call resources are freed after the call completes.

---

## 9. Accessibility Surfaces

### Screen-reader narration

A narration engine generates running prose from the aviary state at a slow cadence (~1 update per 30-60 seconds at idle):

```
NarrationGenerator {
  input: current aviary snapshot
  output: prose string (naturalist voice, lowercase, present-tense)
  cadence: 30-60 seconds at idle; faster on user-initiated events
  priority_queue: user-initiated events (greeting, offer reaction, settle) jump the queue
}
```

Prose examples:
- "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- "a warbler perches on the high branch, calling softly."
- "the aviary is quiet this evening. both birds are settled, eyes closed."

The narration is delivered via an ARIA live region (`aria-live="polite"`) that the screen reader announces at its own pace. The region is visually hidden but present in the DOM.

**Cadence control**: The narration engine maintains a minimum interval between updates. Even if multiple state changes occur in quick succession, the narration batches them into a single prose update. This prevents the screen reader's queue from being overwhelmed.

**User-initiated event priority**: A return-greeting on session start is narrated within 1-2 seconds (the greeting is the welcome, and the screen-reader user needs to hear it promptly). An offer reaction is narrated as it happens. But even these are written as observations, not announcements.

### Call captions

When captions are enabled (via accessibility settings, or automatically in the WebAudio fallback):

- Each call produces a short prose caption derived from the procedural call parameters at runtime.
- Captions appear as small text near the calling bird, fading in with the call and fading out 1-2 seconds after.
- Caption text uses the naturalist voice: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch."
- Caption generation is a function of the motif and its randomized parameters — each caption matches what was actually synthesized.

### Focus and keyboard navigation

**Tab order**:
1. Top bar items (left to right): account/settings, accessibility, notebook, offer.
2. Aviary scene: first bird (front perch → middle → back, or left → right within perch).
3. No tab into ambient elements or background.

**Within the aviary scene**:
- **Tab / Shift+Tab**: Move focus between birds.
- **Arrow keys**: Move focus between birds (Left/Right cycles through birds on the same perch; Up/Down moves between perches).
- **Enter**: Trigger listen-in on the focused bird.
- **Escape**: Exit listen-in.

**Focus indicator**: A soft, high-contrast outline around the focused bird. The outline is visible against both bright (midday) and dim (night) aviary states. Exact treatment specified by the visual designer — likely a 2px outline in a warm white with a subtle drop shadow for contrast against any background.

### Contrast

All user-copy text passes WCAG AA (4.5:1 for normal text, 3:1 for large text):
- Top bar icons with labels: text on the top bar background (which fades, so contrast is measured at full opacity).
- Settings and account surfaces: standard text contrast on their backgrounds.
- Call captions: text overlaid on the aviary scene — requires a subtle text shadow or background scrim to ensure contrast against varying scene backgrounds.
- Narration (when displayed visually, e.g., in a debug mode): same contrast requirements.

The aviary scene itself contains no user copy. The only text in the scene is call captions and the focus indicator.

---

## 10. Performance Budgets and Observability

### Budgets

| Metric | Budget | Rationale |
|---|---|---|
| Initial JS bundle (gzipped) | <2 MB | Time-to-first-bird recoverable on mid-tier mobile over 4G |
| Time to first bird visible | <500ms | Below this threshold, the aviary feels already-running |
| Idle motion frame rate | 60fps | On a 5-year-old mid-range laptop, sustained over 30 minutes |
| Memory growth over 30 min | 0 MB | No leaks; procedural audio buffers reused; no retained references |
| Snapshot response time (p95) | <100ms | From CDN edge cache |
| Simulation tick latency (p99) | <5s | Alarm threshold; target is much lower (~500ms) |
| Snapshot payload size | <10 KB | Per-account snapshot is small (7 birds max + metadata) |

### How we hit the budgets

**Bundle size (<2MB gzipped)**:
- Aggressive code-splitting: account settings, accessibility settings, visit-invitation flow are lazy-loaded.
- Bird visual assets: SVG sprites or compact bitmaps, not high-res textures.
- Audio: no recorded audio files. The motif library is data (JSON), not audio.
- Tree-shaking: no heavy framework dependencies. Consider Preact or vanilla JS for the rendering layer; React only if the component tree justifies the bundle cost.
- Budget enforcement in CI: the build fails if the initial bundle exceeds 2MB gzipped.

**Time to first bird (<500ms)**:
- Inline critical CSS and the quiet-field background color in the HTML.
- Serve the initial snapshot inline with the HTML response (server-side rendered into the page, or fetched at the edge and injected).
- Render birds immediately on snapshot receipt, before loading non-critical assets (notebook data, settings, audio motif library).
- Audio can initialize after first render — the first 1-2 seconds may be silent while AudioContext warms up.

**60fps idle motion**:
- Limit draw calls: batch bird sprites, minimize per-frame allocations.
- Ambient particles (leaves, feathers) are capped at ~10 on screen at once.
- No per-frame network requests.
- requestAnimationFrame with frame budget monitoring; if frames consistently drop below 60fps, reduce ambient particle count.

**No memory growth**:
- WebAudio oscillator nodes are created and destroyed per-call. No buffer pooling that grows unbounded.
- Notebook entries are virtualized in the scroll view — only visible entries are in the DOM.
- Event listeners are cleaned up on component unmount.
- CI test: run a headless browser for 30 minutes, assert memory delta <5MB (with margin for GC timing).

### Observability

**Aggregate telemetry** (operational health, no per-account data):
- Request counts by endpoint and status code.
- Latency histograms: snapshot response, event ingestion, simulation tick.
- Error rates by service and error type.
- Client-side: first-bird-render timing, render-frame timing (fps histograms), AudioContext error counts.
- Session-duration histograms (anonymized, no account dimension).

**Synthetic monitoring**:
- A fleet of automated browsers running the aviary on a schedule from common geographies.
- Measures: time-to-first-bird, snapshot latency, audio pipeline initialization, rendering fps.
- Alerts on regression against budgets.

**What we deliberately don't measure**:
- Per-bird interaction patterns.
- Per-account drift rates.
- Per-account session frequency or duration (beyond anonymized histograms).
- Notebook content.
- Any data that could reconstruct a user's relationship with their aviary.

**Alerting**:
- Simulation tick p99 latency >5s → alert.
- Snapshot response p95 >200ms → alert.
- Error rate >1% on any endpoint → alert.
- Client-side first-bird-render p95 >1s → alert.

---

## 11. Rollout

### Phase 0: Internal alpha (weeks 1-4)

- Team-only accounts. 2 birds per aviary, no adoption of additional birds.
- Focus: simulation tick correctness, drift calibration, audio pipeline stability.
- Instrumentation: full drift logging (internal only), per-tick timing, audio error counts.
- Calibration sprints: adjust drift coefficients against synthetic and real usage data.

### Phase 1: Closed beta (weeks 5-8)

- ~100 external users, invited by email.
- 2 starter birds, third bird offer at 4-week aviary age.
- Focus: drift feel (do users report birds changing at the right pace?), audio quality (do calls feel alive or annoying?), notebook voice (do entries feel charming or spammy?).
- Feedback channel: simple email form in account settings. No in-product feedback widget (that would be an announcement surface).
- Instrumentation: aggregate session-duration histograms, drift-rate distributions (anonymized), audio error rates.

### Phase 2: Open launch (week 9+)

- Open sign-up via magic link.
- 2 starter birds, adoption pacing per the bird-engine spec (third bird at ~3 months aviary age, scaling to 7 over ~1 year).
- Visit invitations enabled (off by default).
- Full accessibility surfaces shipped (screen-reader narration, reduced-motion, captions).

### Birds-per-aviary ramp

| Aviary age | Max birds available |
|---|---|
| Day 0 | 2 (starter pair) |
| ~3 months | 3 (third bird offer appears) |
| ~6 months | 4 |
| ~9 months | 5 |
| ~12 months | 6 |
| ~15 months | 7 (cap) |

New bird offers appear as a naturalist-voiced prompt in the aviary: "a new species has been spotted near the aviary." The user can accept (name the bird, it joins) or ignore (the offer remains available indefinitely).

### Day-one instrumentation

- **Drift calibration dashboard**: distribution of personality vectors across the population (anonymized, aggregated). Are birds drifting at the expected rate? Are any traits saturating too fast or too slow?
- **Audio health**: AudioContext creation success rate, call synthesis error rate, listen-in ramp smoothness (client-reported frame timing during mix transitions).
- **Notebook quality**: entry frequency distribution (are entries too frequent for active users? too sparse for light users?).
- **Performance**: all budgets monitored from day one, with alerts configured.
- **Accessibility**: screen-reader narration generation latency, caption rendering errors, reduced-motion mode activation rate.

---

## 12. Risks

### Drift calibration

**Risk**: The drift function is too fast (birds change visibly between sessions, feeling gamey) or too slow (users feel nothing is happening after weeks).

**Mitigation**: The calibration harness runs from week 1. Synthetic user profiles at multiple activity levels verify drift rates against the targets (measurable at 1 week, visible at 3 weeks). The closed beta provides real-user data. Drift coefficients are server-side configuration, deployable without client updates.

**Contingency**: If drift is consistently miscalibrated across the population, the coefficients are tunable per-trait. The low-pass filter architecture means changes to coefficients take effect gradually — no sudden jumps for existing users.

### Sync correctness

**Risk**: A bug in the tick's event consumption causes missed events, double-counting, or out-of-order processing, corrupting personality vectors.

**Mitigation**: The event log is append-only and ordered. The tick processes events in order with idempotent application (each event has a unique ID; the tick tracks which events it has processed). Integration tests simulate multi-device scenarios: two clients submitting events concurrently, a client reconnecting after a gap, a tick failing mid-processing and retrying.

**Contingency**: Personality vectors are versioned. If a corruption is detected, the vector can be rolled back to the last known-good version. The event log is retained (for the account's lifetime), so the tick can be re-run from a known-good state if needed.

### Audio uncanniness

**Risk**: Procedural calls sound synthetic, repetitive, or unpleasant. Users hear the same motif too often and the spell breaks.

**Mitigation**: The motif library is designed with enough variance (pitch, timing, timbre) that calls are perceptually distinct even when using the same motif. The closed beta specifically tests for audio fatigue. Motif libraries can be expanded post-launch without client updates (the motif data is served from the API).

**Contingency**: If a specific species' call grammar is consistently reported as unpleasant, the motif library for that species can be revised server-side. The procedural architecture means this is a data change, not a code change.

### Accessibility regressions

**Risk**: Screen-reader narration degrades in quality (too frequent, too sparse, wrong voice). Reduced-motion mode is treated as an afterthought and ships late or broken. Captions don't match the audio.

**Mitigation**: Accessibility is a first-class workstream from day 1, not a v1.1 fix. The narration engine, reduced-motion rendering, and caption system are built alongside the main rendering pipeline, not after. Automated tests verify: narration cadence stays within bounds, reduced-motion mode renders correctly for all bird states, captions are generated for every call motif.

**Contingency**: If the narration engine produces poor prose, the template library is expandable. If reduced-motion mode has rendering issues, the cross-fade parameters are tunable. Both are data-driven, not hardcoded.

### Presence signal accuracy

**Risk**: The 3-signal conjunction (visibility, focus, recent activity) is too strict (users who are genuinely watching but not moving lose presence) or too lax (users who walk away keep accumulating presence).

**Mitigation**: The activity window for the pointer/key check is calibrated during build, leaning toward the longer side (3-5 minutes) because watching birds without moving is the actual product. The closed beta tests with real usage patterns. The window is a server-side configuration parameter.

**Contingency**: If the signal is too strict, the activity window is widened. If too lax, it's narrowed. The 3-signal conjunction itself is non-negotiable — the fix is always in the calibration of the activity window, never in dropping a signal.

### Bundle size creep

**Risk**: Dependencies, assets, or code accumulate and the initial bundle exceeds 2MB, blowing the time-to-first-bird budget.

**Mitigation**: CI enforces the 2MB budget as a hard gate — the build fails if exceeded. Code-splitting is implemented from the start for all non-critical surfaces. Asset budgets are tracked per-sprint.

**Contingency**: If the budget is tight, the first cuts are: lazy-load the notebook viewer, defer audio motif library loading until after first render, compress bird sprites more aggressively.

### Privacy boundary violation

**Risk**: A well-meaning engineer adds per-account data to the telemetry pipeline, or an analytics query joins simulation data with operational metrics.

**Mitigation**: The telemetry pipeline is a separate service with separate credentials, reading from a separate data store. The simulation database is never readable by the telemetry service. This is enforced at the infrastructure level (network policies, IAM roles), not just at the application level. Code review checklist includes a privacy-boundary check.

**Contingency**: If a violation is detected, the telemetry pipeline's access is revoked and the data is purged. The architectural separation makes this a clean cut, not a surgical extraction.

---

## 13. Implementation Order

### Sprint 1-2: Foundation

- Accounts service: magic-link auth, session tokens, synthetic UUID.
- State store schema: accounts, birds, personality vectors, moods.
- Event log infrastructure: append-only, partitioned by account.
- API gateway: auth middleware, routing.
- Client skeleton: HTML shell with quiet-field background, bundle tooling, CI with budget gates.

### Sprint 3-4: Simulation core

- Simulation tick: event consumption, mood transitions, day/night cycle.
- Drift function: initial coefficients, calibration harness.
- Snapshot endpoint: state store → cache → client.
- Client rendering: static scene (background, perches), bird sprites from snapshot data.

### Sprint 5-6: Bird engine

- Call scheduling: server-side call scheduler, motif metadata in snapshots.
- Audio pipeline: WebAudio synthesis, motif library for 2 species, ambient mix.
- Idle motion: client-side idle state machine, mood-weighted transitions.
- Bird-to-bird interactions: call responses, mood contagion, chorus detection.

### Sprint 7-8: Interactions

- Presence accounting: 3-signal conjunction, presence-ping emission.
- Listen-in: focus mechanics, audio mix ramping, event emission.
- Offer: seed/song/pool, per-bird cooldown, reaction logic.
- Settle: lighting transition, audio ramp, 5-second undo.
- Return-greeting: absence-aware bird selection, procedural variation.

### Sprint 9-10: Notebook + voice

- Notebook entry generation: event evaluation, sparsity enforcement, prose templates.
- Notebook UI: scrollable, read-only, naturalist voice.
- Screen-reader narration: prose generation, ARIA live region, cadence control.
- Call captions: runtime generation from call parameters.

### Sprint 11-12: Polish + accessibility

- Reduced-motion mode: cross-fade rendering, ambient simplification.
- Keyboard navigation: full tab/arrow/enter/escape support, focus indicators.
- Top bar: auto-fade, icon set.
- Day/night cycle: palette interpolation, species-specific night behavior.
- Ambient weather: rain, wind, mood effects.
- Responsive layout: phone to desktop.

### Sprint 13-14: Social + accounts

- Visit invitations: email flow, read-only snapshot endpoint, token validation.
- Visit log, revocation, expiration.
- Account export: JSON generation, email delivery.
- Account deletion: soft/hard lifecycle.
- Session management: device list, revocation.

### Sprint 15-16: Hardening + launch prep

- Performance optimization: bundle audit, render profiling, memory leak testing.
- Drift calibration: final coefficient tuning from beta data.
- Synthetic monitoring fleet.
- Alerting configuration.
- Species pool completion: remaining 4 species (motif libraries, visual assets).
- Documentation, operational runbooks.

---

## 14. Open Questions (defensible calls)

1. **Rendering technology**: Canvas 2D vs. lightweight WebGL. Recommendation: start with Canvas 2D (simpler, sufficient for the visual complexity), migrate to WebGL only if performance profiling demands it.

2. **Frontend framework**: React vs. Preact vs. vanilla. Recommendation: Preact for the component tree (small bundle, React-compatible), vanilla for the rendering layer (Canvas/WebGL doesn't benefit from a virtual DOM).

3. **Activity window for presence**: 3 minutes vs. 5 minutes. Recommendation: start at 5 minutes (generous, because watching birds is the product), tighten if beta data shows inflated presence-time.

4. **Snapshot polling vs. WebSocket**: Recommendation: implement both. WebSocket for connected clients (lower latency, server-push), polling as fallback (simpler, works behind restrictive proxies). WebSocket is an optimization, not a requirement.

5. **Notebook prose generation**: Template-based vs. LLM-generated. Recommendation: template-based for v1 (reviewed, voice-controlled, predictable). LLM generation can be explored post-launch if the template library becomes limiting.

6. **Tick cadence**: 30 seconds vs. 1 minute vs. 2 minutes. Recommendation: start at 1 minute. Faster ticks increase server cost without user-visible benefit (mood transitions and drift are slow). Tunable per-deployment.

7. **Database**: PostgreSQL vs. DynamoDB. Recommendation: PostgreSQL with JSONB for the aviary state (flexible schema, strong consistency, familiar to the team). DynamoDB if scale demands it post-launch.

---

*End of plan.*
