# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In scope for v1
- Two starter birds per aviary, up to seven maximum
- Six bird species in the initial pool
- Server-side simulation tick (~1/minute) advancing canonical aviary state continuously
- Personality vector (5 traits) per bird, persisted server-side, never exposed numerically
- Mood system (wary, content, curious, drowsy, alert) with fast-timescale transitions
- Presence accounting: triple-conjunction check (visibilityState visible + window focus + recent pointer/key activity)
- Return-greeting: procedurally varied, absence-length-sensitive, bold-bird-ordered
- Listen-in interaction: slow audio re-balance on engage/disengage
- Offer interaction: seed, song-fragment, still-pool; per-bird cooldown ~3 minutes
- Settle gesture: soft lighting shift, 5-second undo window
- Field notebook: auto-generated naturalist prose, ~1 entry per few days, read-only
- Aviary scene: single horizontal screen, three perch zones, day/night cycle on user's local time, ambient weather (rain, wind ~few times/week), ambient micro-motion (leaves, feathers)
- Top bar: account/settings, accessibility settings, field notebook, offer affordance; fades to near-transparent after a few seconds of cursor stillness
- Email magic-link auth; 15-minute link expiry; per-device session tokens; revocable sessions
- Single canonical aviary per account; multi-device sync via server-as-single-source
- Synthetic UUID account IDs; email stored once, encrypted
- Account export (JSON on demand, emailed download link)
- Soft/hard account deletion (30-day soft window, then hard purge)
- Visit-invitation feature: email invite, read-only ambient view, revocable, off by default
- Screen-reader narration: naturalist prose, slow cadence (30–60s at idle)
- Reduced-motion mode: cross-fade between still poses, not animations-off
- Call captioning: runtime-generated naturalist prose, appearing near the calling bird
- WCAG AA contrast on all user-copy text
- Keyboard navigation: Tab → top bar; Tab → aviary → arrow keys between birds; Enter → listen-in; Escape → exit listen-in
- WebAudio procedural call synthesis; silence-with-captions fallback if WebAudio unavailable
- Synthetic performance monitoring and aggregate-only RUM; no per-bird telemetry in any analytics pipeline

### Not in scope for v1 (from non_goals.md)
- Native mobile apps (iOS, Android)
- Gamification of any kind: no achievements, streaks, levels, badges, XP, green-dot calendars
- Tamagotchi mechanics: no hunger, no distress, no decay on neglect
- Social network surfaces: no public profiles, no follows, no discovery feed, no comments on visits
- Payments, shared aviaries, multi-aviary accounts
- Password or SSO auth
- Push/email notifications (no opt-in notification surface except the per-account visit-notification toggle)
- Customizable scenes, custom palettes, drag-to-place bird positioning

---

## 2. Architecture

### Service shape

```
Browser Client
│
├── Rendering Worker (Web Worker)
│   └── Scene compositor, idle micro-motion loop, reduced-motion mode
│
├── Audio Worklet (AudioWorkletProcessor)
│   └── Procedural call synthesis, chorus mixer, listen-in re-balance
│
└── App Shell (main thread)
    ├── Presence detector (visibility + focus + activity window)
    ├── Event emitter → Event Log API
    ├── Snapshot consumer ← State Snapshot API
    └── Narration generator (screen-reader prose, caption text)

Edge Layer (CDN)
└── HTML + initial state snapshot bundled at delivery (avoids cold-cache round-trip)

API Gateway
├── POST /auth/request-link      — magic-link dispatch
├── POST /auth/consume-link      — link consumption, session token issuance
├── GET  /aviary/snapshot        — current canonical state (small JSON)
├── POST /aviary/events          — append interaction events (offer, listen-in, settle, presence)
├── GET  /notebook/entries       — field notebook entries (paginated, most-recent-first)
├── GET  /account                — account settings, session list, visit log
├── POST /account/export         — trigger JSON export, delivers via email
├── DELETE /account              — initiate soft deletion
├── POST /visits/invite          — send visit invite (email)
├── DELETE /visits/:id           — revoke invite
└── GET  /visits/:token          — visitor state snapshot (read-only; checks revocation)

Simulation Service (background)
└── Tick worker (~1/minute)
    ├── Reads event log for account
    ├── Applies drift deltas to personality vectors
    ├── Transitions mood states
    ├── Generates notebook entries when warranted
    └── Writes canonical AviarySate record

Auth Service
└── Magic link generation, session token CRUD, rate limiting per email

Notification Service (thin)
└── Email dispatch only: magic links, export download links, visit invitations
    └── (No push, no product-event email)
```

### Client/server split

The server owns:
- All canonical state (personality vectors, current moods, perch positions, mood timers)
- Event log (append-only; client writes, server reads)
- Notebook entries (generated by server tick)
- Session and account records

The client owns:
- Rendering interpolation between snapshots
- Presence detection signals (but reports them to the server as events)
- Audio synthesis (WebAudio worklet)
- Scene composition, idle micro-motion animation loop
- Narration and caption generation from the snapshot state

The client never writes personality state. The client never owns state that diverges from the server record.

### Render pipeline boundary

The main thread handles snapshot consumption, event dispatching, presence detection, and accessibility narration. Heavy rendering runs in a Web Worker via OffscreenCanvas. Audio runs in an AudioWorklet. The main thread is kept light enough that presence detection (pointer/key events) is never blocked by animation load.

---

## 3. Data model

### Account record
```
account_id: UUID (synthetic, generated at creation)
email: string (encrypted at rest; AES-256-GCM with per-account key; only record that holds email)
email_key_id: string (reference to key management service)
created_at: timestamp
deleted_at: timestamp | null (soft-deletion marker)
hard_delete_scheduled_at: timestamp | null
```

### Session token record
```
token_id: UUID
account_id: UUID (FK → account)
issued_at: timestamp
last_seen_at: timestamp
device_label: string (user-agent digest; not a fingerprint)
revoked: boolean
```

### Magic link record
```
link_id: UUID
account_id: UUID
token_hash: string (SHA-256 of raw token; raw never stored)
expires_at: timestamp
consumed_at: timestamp | null
```

### Aviary record
```
aviary_id: UUID
account_id: UUID (FK; one-to-one in v1)
created_at: timestamp
bird_slots: []bird_id (ordered; determines render Z-ordering)
ambient_state: { weather: enum, weather_ends_at: timestamp | null }
last_tick_at: timestamp
```

### Bird record
```
bird_id: UUID (stable; never reused or replaced)
aviary_id: UUID (FK)
species_id: string (from species pool enum)
name: string (user-assigned; mutable)
adopted_at: timestamp
personality: {
  boldness: float [0.0–1.0]
  social_warmth: float [0.0–1.0]
  vocal_frequency: float [0.0–1.0]
  plumage_saturation: float [0.0–1.0]
  curiosity: float [0.0–1.0]
}
mood: enum { wary | content | curious | drowsy | alert }
mood_entered_at: timestamp
perch_zone: enum { front | middle | back }
last_greeted_at: timestamp | null
call_grammar_seed: string (species-derived; fixed at adoption; shapes motif library selection)
```

### Interaction event record (append-only log)
```
event_id: UUID
aviary_id: UUID (FK)
bird_id: UUID | null (null for aviary-level events like settle)
event_type: enum {
  presence_ping | listen_in_start | listen_in_end |
  offer_seed | offer_song | offer_pool |
  settle | return
}
occurred_at: timestamp
metadata: JSON (e.g., listen-in duration, offer reaction recorded by client)
```

### Notebook entry record
```
entry_id: UUID
aviary_id: UUID (FK)
generated_at: timestamp
prose: string (naturalist field-notebook voice; lowercase, present-tense)
triggering_event_type: string | null
```

### Visit invite record
```
invite_id: UUID
aviary_id: UUID (FK → host)
visitor_email: string (encrypted)
invited_at: timestamp
expires_at: timestamp (invited_at + 30 days)
consumed_at: timestamp | null
revoked_at: timestamp | null
token_hash: string (SHA-256 of raw single-use visit token)
```

### Visit session record (ephemeral)
```
visit_session_id: UUID
invite_id: UUID (FK)
started_at: timestamp
last_seen_at: timestamp
ended_at: timestamp | null
```

### Canonical state snapshot (served to clients; derived from above records)
```json
{
  "aviary_id": "...",
  "snapshot_at": "ISO8601",
  "local_time_offset_hint": "+05:00",
  "ambient": { "weather": "clear", "time_of_day": "morning" },
  "birds": [
    {
      "bird_id": "...",
      "species_id": "warbler_a",
      "name": "Pip",
      "mood": "content",
      "perch_zone": "front",
      "plumage_saturation": 0.61,
      "call_timing_phase": 0.42,
      "is_calling": false
    }
  ],
  "notebook_preview": { "latest_entry_id": "...", "prose_snippet": "..." }
}
```

Note: personality vector values (boldness, social_warmth, etc.) are never included in the snapshot payload. Plumage_saturation is the one visual trait exposed; it drives rendering only and is surfaced as a rendering parameter, not labeled numerically to the user.

---

## 4. API surface

### Auth flow
```
POST /auth/request-link
  Body: { email: string }
  Response: 200 (always; timing-safe — never reveal whether email exists)

POST /auth/consume-link
  Body: { token: string }
  Response: 200 { session_token: string, account_id: UUID }
           | 401 { error: "expired" | "already_used" | "not_found" }
  Side effect: mark link consumed; issue session token

DELETE /auth/session/:token_id
  Auth: Bearer session_token
  Response: 204 (session revoked)
```

### Aviary state
```
GET /aviary/snapshot
  Auth: Bearer session_token
  Query: ?since=<ISO8601> (optional; if provided and state unchanged, returns 304)
  Response: 200 { snapshot } | 304

POST /aviary/events
  Auth: Bearer session_token
  Body: { events: [{ event_type, bird_id?, occurred_at, metadata? }] }
  Response: 202 (accepted for async processing by tick)
  Note: batch-friendly; client can queue and flush periodically

GET /aviary/snapshot (visitor path)
  Auth: visit_token (query param or header; no account session required)
  Response: 200 { snapshot } | 403 { error: "revoked" | "expired" }
```

### Notebook
```
GET /notebook/entries
  Auth: Bearer session_token
  Query: ?before=<entry_id>&limit=20
  Response: 200 { entries: [{ entry_id, generated_at, prose }] }
```

### Account
```
GET /account
  Auth: Bearer session_token
  Response: 200 { account_id, email_masked, sessions: [...], visit_log: [...], settings: {...} }

PATCH /account/settings
  Auth: Bearer session_token
  Body: { visit_notifications_enabled?: boolean }
  Response: 200

POST /account/export
  Auth: Bearer session_token
  Response: 202 (export queued; delivered by email link when ready)

DELETE /account
  Auth: Bearer session_token
  Response: 202 (soft deletion initiated; 30-day window before hard purge)

POST /account/undelete
  Auth: Bearer session_token (valid during soft-deletion window)
  Response: 200 (deletion cancelled)
```

### Visit invitations
```
POST /visits/invite
  Auth: Bearer session_token
  Body: { visitor_email: string }
  Response: 201 { invite_id, expires_at }
  Side effect: sends invite email to visitor_email

DELETE /visits/invite/:invite_id
  Auth: Bearer session_token
  Response: 204 (invite revoked; active visitor session terminated on next snapshot pull)

GET /visits/invite/:invite_id
  Auth: Bearer session_token
  Response: 200 { invite_id, invited_at, expires_at, consumed_at, revoked_at, visitor_email_masked }
```

### Bird naming
```
PATCH /birds/:bird_id
  Auth: Bearer session_token
  Body: { name: string }
  Response: 200 { bird_id, name }
  Validation: name 1–40 chars; no effect on personality, mood, or call
```

---

## 5. Simulation engine design

### Tick worker cadence

The tick worker runs on a per-aviary schedule at approximately once per minute. Because tick is per-aviary (not per-user-session), the aviary advances continuously whether or not any client is connected. The tick worker is implemented as a distributed job queue (one job per aviary_id, recurring) so that each aviary's tick is isolated from all others and failures don't cascade.

Exact cadence is calibrated during build; the target is ~60 seconds between ticks. Sub-minute cadence is unnecessary — mood transitions and drift updates happen over minutes-to-weeks timescales.

### Tick execution sequence

```
1. Lock the aviary_id (optimistic lock or row-level DB lock)
2. Read all interaction events since last_tick_at
3. Compute presence-time from presence_ping events
   (presence_ping at intervals ≤ activity_window_seconds apart count as continuous presence)
4. Apply mood transitions (see below)
5. Apply drift deltas (see below)
6. Advance day/night state from current UTC + account timezone offset
7. Advance ambient weather (roll for weather events, check weather_ends_at)
8. Evaluate notebook entry generation (see below)
9. Write updated AviarySate record (mood, perch_zone, call state, ambient)
10. Write updated personality vectors to Bird records
11. Update aviary.last_tick_at
12. Release lock
```

### Mood transitions

Mood is a per-bird enum. Transition rules:

**Time-of-day baselines (user's local timezone):**
- 05:00–09:00 → alert (early morning hush)
- 09:00–17:00 → content (default daytime)
- 17:00–20:00 → drowsy (dusk)
- 20:00–05:00 → drowsy/sleeping (most birds low-motion; nightjar-species remains alert)

**Interaction modifiers (applied additively per tick if events present):**
- Accepted offer → nudge toward content
- Wary bird offered-to with no acceptance → no change (don't force transition)
- listen_in_start on a bird → slight nudge toward curious
- Heavy chorus (two+ high-vocal-freq birds simultaneously) → nearby birds nudge toward alert
- Rain ambient event → nudge all birds toward drowsy
- Wind ambient event → wary birds stay wary; content birds nudge toward alert

**Personality gates:**
- High-boldness bird: wary → content transition probability doubled
- Low-boldness bird: wary persists longer
- High-curiosity bird: curious → content transition damped (stays curious longer)

Mood transitions are sampled at tick time using a small probability table seeded by the current mood, time-of-day baseline, recent events, and personality gates. No deterministic step function — mood is stochastic within the probability space, which is what makes it feel alive rather than mechanical.

### Drift function

Drift is a low-pass filter over cumulative presence-and-interaction signals. The filter time-constant is calibrated so:
- Measurable change in test instruments after ~1 week of regular visits
- User-perceptible change (visible in rendered plumage or bird behavior) after ~3 weeks

Drift is monotonic-toward-expressive. Trait values only move upward on positive presence; they do not decay on neglect.

**Drift inputs per tick (approximate relative weights):**
```
presence_time_seconds   → weight 1.0 (dominant)
listen_in_seconds       → weight 0.5 (per-bird; raises social_warmth, vocal_frequency)
offers_accepted         → weight 0.2 per event (raises curiosity)
offers_presented        → weight 0.1 per event (raises boldness, even if not accepted)
settle_events           → no drift impact (ends presence window cleanly)
```

**Per-trait delta computation:**
```
delta_boldness          = presence_weight * presence_time + offer_weight * offers_presented
delta_social_warmth     = listen_in_weight * listen_in_seconds * boldness_modifier
delta_vocal_frequency   = listen_in_weight * listen_in_seconds
delta_plumage_saturation = 0.3 * delta_boldness + 0.3 * delta_social_warmth + 0.3 * delta_vocal_frequency
delta_curiosity         = offer_weight * offers_accepted
```

The multipliers, baseline deltas, and filter constants are implementation details; the above expresses relative weights and inputs, not final coefficients. Calibration happens during build using a test harness that simulates 21 days of typical presence patterns and verifies the target drift thresholds.

Drift is applied as additive deltas:
```
personality.boldness = min(1.0, personality.boldness + delta_boldness * filter_factor)
```
`filter_factor` is the low-pass weight per tick, approximately `tick_cadence_seconds / filter_time_constant_seconds`, calibrated so weekly drift is measurable.

### Call-grammar runtime

Each species has a call-grammar motif library: a small set of motifs (3–5 per species) defined as parameterized sequences of tones (frequency, duration, attack, decay, repetition pattern). The call grammar is a generative grammar that selects and combines motifs at runtime:

```
call = [silence(duration)] [motif(params)] [silence(duration)] [motif(params) | null]
params.pitch_shift     = f(personality.vocal_frequency, mood)
params.tempo           = f(mood, time_of_day)
params.repetitions     = f(personality.vocal_frequency, mood)
params.gap             = f(personality.social_warmth)  // tighter gap = more social
```

The call-grammar seed (fixed at adoption) determines which motif library the bird draws from; the per-call variation is generated from the personality and mood parameters at synthesis time. The same bird at different moods sounds recognizably like itself but varied. Two different birds in the same mood sound distinct because their seeds differ.

Chorus emerges when two or more birds are calling in overlapping windows. The audio worklet mixes all active call streams into a single output with per-bird gain. Listen-in re-balance is applied as a multiplier on each bird's gain — the focused bird's gain ramps up over ~2 seconds, others ramp down to their ambient floor (not zero) over the same window.

### Notebook entry generation

The tick evaluates entry triggers after updating state. Entry triggers (examples; full list finalized during build):

- First time Bird A greeted before Bird B on a given day (weekly uniqueness check)
- Bird entered a mood not seen in more than 3 days
- Chorus event lasting more than 60 seconds
- Bird's first reaction to each offer type
- Aviary age milestone (first week, first month) — naturalist-voiced, not gamification language
- Long stretch of quiet (no calls for >10 minutes, which is notable)

Entry prose is generated from a template library keyed by trigger type plus current state variables. Templates are filled with bird names, mood descriptors, perch positions, time-of-day context. The output is naturalist voice (lowercase, present-tense, specific). Entry generation is rate-limited to approximately 1 entry per few days for a regularly-visited aviary; the rate limiter is per-aviary and checks last-entry timestamp before generating.

---

## 6. Sync model

The sync model is a consequence of the server-side tick architecture, not a separate sync protocol.

**Canonical state lives only on the server.** There is no client-side state that diverges from the server record. Clients are render surfaces, not state owners.

**Snapshot pull pattern:**
- On tab visible after hidden: pull fresh snapshot
- On render-frame gap >5 seconds (laptop suspend, wake): pull fresh snapshot
- On keepalive interval while visible: pull snapshot every 30 seconds (configurable)
- On initial load: snapshot is bundled in the HTML response from the CDN edge (first-paint optimization; may be up to 60 seconds stale, which is acceptable at the tick cadence)

**Conflict prevention:**
- Personality state: only the server tick writes it; no client path touches it directly. Conflict is structurally impossible.
- Events: append-only; multiple devices simultaneously writing events is fine — the event log is the journal, and the tick processes events in `occurred_at` order regardless of which device submitted them.
- Mood/perch: server-authoritative; both devices read the same canonical record.

**Multi-device behavior:**
- Both devices signed in simultaneously pull snapshots from the same record. They may be slightly out of sync (up to one tick cadence, ~60 seconds) but are never in fundamentally different states.
- If both devices submit events in the same tick window, both sets of events are recorded in the append-only log and processed by the next tick pass. This is correct; the drift function is additive and operates on totals, so double-presence from two devices in the same window does not corrupt the model (though it will inflate presence-time slightly — calibration should account for this).

**No merge, no conflict resolution UI.** The server is authoritative; clients never need to reconcile.

---

## 7. Frontend rendering pipeline

### Scene composition

The aviary scene is rendered on an OffscreenCanvas in a Web Worker. The main thread passes state snapshots and interaction events to the worker via postMessage; the worker renders and transfers frames to a canvas element on the main thread.

**Scene layers (back to front):**
1. Sky background (gradient; shifts with time-of-day)
2. Background foliage (static SVG; subtle parallax offset on scroll/tilt is NOT implemented — the scene is flat parallax, not heavy)
3. Perch structures (middle plane; SVG or compact bitmap)
4. Birds (one render element per bird; see below)
5. Foreground branches / occasional leaf drift (SVG animation, no per-leaf simulation state)
6. Weather overlay (rain: CSS filter or lightweight canvas effect; wind: leaf motion acceleration)

**Bird render element:**
Each bird is a layered sprite: base silhouette (species-specific SVG) + plumage layer (saturation driven by personality.plumage_saturation) + mood-animation state machine.

The mood-animation state machine holds the current pose sequence for the bird's mood. Pose sequences are defined per-species per-mood as a graph of poses with weighted random transitions:

```
wary:    scan-right → hold → scan-left → hold → shuffle-back → ...
content: preen → hold → preen → look-forward → ...
curious: head-tilt → step-forward → head-tilt → ...
drowsy:  low-perch → feathers-fluff → low-perch → ...
alert:   upright → scan-fast → call-posture → ...
```

Transitions between mood states trigger a cross-fade between the current pose and the first pose of the new sequence. Cross-fade duration: ~1 second.

**Perch-zone transitions:**
When a bird moves between perch zones (server-side state change propagated in snapshot), the client animates a small hop or short fly-arc. The arc duration is ~0.5–1.0 seconds. Between snapshots, the bird is rendered at its most recent known perch zone.

### Idle micro-motion

Idle micro-motion runs as a continuous background animation loop, independent of state snapshot timing. The motion is generated from the current mood-animation state machine pose sequence plus small parametric additions:

- Breathing motion: 0.3–0.5 Hz subtle up/down displacement on body center
- Blink: random interval 4–8 seconds, 80ms eyelid cross-fade
- Feather micro-ruffle: occasional 200ms local displacement on wing or tail

These are pure animation concerns — they do not affect state, do not trigger events, and do not stop when the tab is hidden (the rendering stops, but the animation state advances so that when rendering resumes, the bird is mid-pose, not reset to frame 0).

### Loading state

On first paint (before state snapshot is delivered), the scene renders a quiet field: soft sky gradient with ambient color for current time-of-day. If the bundled snapshot (delivered with the HTML) is present, birds render immediately from that snapshot. If it is absent (cache miss), the quiet field persists until the snapshot fetch completes. No spinner; no loading bar.

The quiet field uses the same sky-background layer from the scene, so the transition to the live aviary is a color/brightness shift, not a composition change. Birds appear with a brief fly-in from off-screen (first time only on the session; after that, they're already positioned).

### Reduced-motion mode

When `prefers-reduced-motion: reduce` is detected, or when the user enables it in accessibility settings, the rendering pipeline switches to reduced-motion mode:

- Mood-animation state machine: instead of frame-by-frame animation, select key poses from the sequence and cross-fade between them (300–500ms cross-fades)
- Perch zone transitions: cross-fade between bird positions (no fly-arc)
- Ambient leaf drift: disabled
- Weather overlay: rain effect reduced to a static slight desaturation of the background; wind disabled
- Day/night transitions: color shifts proceed at same rate (they are slow enough not to trigger vestibular issues)
- Top-bar fade: still applies (it is opacity, not motion)

The reduced-motion aviary is the same aviary. Mood, drift, calls, notebook — all identical. Only the visual motion register changes.

### Top-bar fade

The top bar starts at full opacity. After 3 seconds of cursor stillness (no pointermove events in the aviary viewport), it cross-fades to 8% opacity over 500ms. On any pointermove or keyboard event, it cross-fades back to 100% over 200ms. The fade is CSS `transition: opacity`; no JS animation loop required.

---

## 8. Audio pipeline

### Architecture

Audio runs entirely in an AudioWorklet (`AviarySynthProcessor`). The main thread posts call-grammar parameters to the worklet; the worklet synthesizes and mixes all active streams. The worklet outputs a single stereo stream to the audio output.

### Procedural call synthesis

Each bird's call is synthesized from its species-specific motif library. A motif is a parameterized unit:

```
motif {
  base_frequency: Hz (species-characteristic)
  frequency_sweep: [Hz] (up/down contour)
  duration_ms: number
  attack_ms: number
  decay_ms: number
  repetition_count: integer
  inter_note_gap_ms: number
}
```

At call-generation time, the grammar selects a motif (weighted by personality.vocal_frequency and mood), applies personality-driven parameter offsets:

```
effective_frequency = motif.base_frequency * pitch_factor(personality, mood)
effective_tempo     = motif.duration_ms * tempo_factor(mood, time_of_day)
repetitions         = motif.repetition_count * rep_factor(personality.vocal_frequency)
```

The motif is synthesized as a series of OscillatorNode + GainNode pairs within the AudioWorklet. Each note in the motif is an OscillatorNode with a frequency envelope and a GainNode with an ADSR envelope. Notes play sequentially within the motif.

### Chorus mixing

Each bird has a dedicated GainNode in the audio graph. The output of all GainNodes is summed into a single ChannelMergerNode → DynamicsCompressorNode → output. The compressor prevents the chorus from clipping as more birds are calling simultaneously.

Default gain per bird: scaled by `personality.vocal_frequency` and current mood (drowsy birds are quieter). Gain is normalized so that seven simultaneous birds at max volume don't clip before compression.

### Listen-in re-balance

When the user enters listen-in on bird B:

```
// Ramp focused bird's gain up over 2000ms
focusedBirdGain.gain.linearRampToValueAtTime(LISTEN_IN_LEVEL, now + 2.0)

// Ramp all other birds' gains down to ambient floor over 2000ms
otherBirds.forEach(b =>
  b.gain.linearRampToValueAtTime(AMBIENT_FLOOR_LEVEL, now + 2.0)
)
```

`LISTEN_IN_LEVEL` = 1.0 (full gain for the focused bird)
`AMBIENT_FLOOR_LEVEL` = 0.25 (audible background, not silent)

On disengage (click focused bird, click empty space, focus a different bird, keyboard escape):

```
// All birds ramp back to their default gain over 2000ms
birds.forEach(b =>
  b.gain.linearRampToValueAtTime(b.defaultGain, now + 2.0)
)
```

### Call captioning

At the moment a call motif is selected and synthesis begins, the caption text for that call is generated from the motif parameters:

```
caption = describeMotiif(motif, mood, perch_zone)
// "a soft three-note rise"
// "a low trill, paused, low trill again"
// "a single sharp call from the back perch"
```

`describeMotif` maps motif properties to naturalist-voice descriptions. The caption text is posted to the main thread, which renders it as a small fading text element near the calling bird's rendered position. Caption fade: 400ms in, hold for the call duration, 400ms out.

Captions are enabled per user setting (accessibility settings). They are also enabled by default when WebAudio is unavailable.

### WebAudio fallback

On initialization, the audio system checks `AudioContext` availability. If AudioContext is unavailable or fails to create:

- All audio synthesis is skipped
- Call captions are enabled by default (the caption system becomes the primary way to experience calls)
- A quiet notice appears once in the accessibility settings area: "Audio isn't available. Call descriptions are showing instead."
- No looped audio fallback is shipped; recorded audio at sufficient variation would exceed the bundle budget

---

## 9. Accessibility surfaces

### Screen-reader narration

The narration system maintains a live ARIA region (`aria-live="polite"` on a visually-hidden `<div>`). Narration updates are written to this region on:

- Session start (after return-greeting completes): introduce the aviary state as of now
- Every 30–60 seconds at idle: update with current scene observation
- On user-initiated events (offer result, settle, listen-in): priority narration immediately

Narration prose is generated by a `NarrationComposer` module on the client, consuming the current state snapshot. The composer has a template set for each scene state combination (mood × perch_zone × time_of_day × weather), filled with the bird's name and current descriptors. Output is naturalist field-notebook voice:

```
> a small grey bird is perched on the front rail, calling softly.
  another bird sits further back with feathers fluffed.
  it is morning in the aviary; the light is gentle.
```

Narration is never a state-list readout. No "Pip: content, front perch. Wren: wary, back perch." The narration is an observation.

Narration cadence at idle: the system checks whether a new update is warranted (state changed meaningfully) before writing to the live region. Updates are suppressed if nothing notable has changed since the last narration. Maximum update rate: once per 30 seconds.

### Captioning

Already described in section 8. Caption text is rendered in the DOM (not on the canvas) as absolutely-positioned elements near the bird's viewport position. This ensures they are readable by assistive technology in addition to being visible on screen.

### Keyboard navigation

Full keyboard navigation spec:

| Key | Context | Action |
|-----|---------|--------|
| Tab | Top bar | Move focus between top-bar icons |
| Tab (first press entering aviary) | Aviary | Focus first bird |
| Arrow keys | Aviary | Move focus between birds (left/right) |
| Enter | Focused bird | Enter listen-in |
| Escape | Listen-in | Exit listen-in, return focus to bird |
| Enter | Top-bar: Offer | Open offer panel |
| Tab | Offer panel | Move between offer types |
| Enter | Offer panel | Submit offer |
| Escape | Offer panel | Close panel |
| Enter | Top-bar: Settle | Trigger settle gesture |
| Enter | Top-bar: Notebook | Open notebook panel |
| Escape | Notebook panel | Close panel |

Focus indicators: a 2px rounded outline in a color that passes WCAG AA contrast against both the brightest (midday) and darkest (night) aviary states. Color specification lives in the design system; nominally a warm white or off-white that reads against the naturalist palette.

### Contrast requirements

- Top-bar icon labels and tooltips: WCAG AA (4.5:1 minimum for text <18pt)
- Caption text: WCAG AA against both light and dark aviary states (caption text may need a semi-transparent background pill if the aviary background varies)
- Account settings, error surfaces, accessibility settings: standard WCAG AA for all text
- Narration region: visually hidden; contrast not applicable
- Call captions: WCAG AA (because they appear on the dynamic aviary background, they use a background pill with sufficient contrast)

---

## 10. Performance budgets and observability

### Budgets

| Metric | Budget | Rationale |
|--------|--------|-----------|
| Initial JS bundle (gzipped) | <2MB | Mid-tier 4G load time constraint |
| Time to first bird visible | <500ms on mid-tier mobile, 4G | Felt-aliveness threshold |
| Idle motion frame rate | 60fps on 5-year-old mid-range laptop | 30-min sustained sessions |
| Memory growth over 30 min | 0 (flat) | Verified in CI |
| Simulation tick p99 latency | <5s alarm threshold | Early degradation detection |
| State snapshot payload | <50KB | Low cost for keepalive pulls |

### Bundle budget management

Code-splitting strategy:
- Core: App shell + scene rendering + audio worklet → must fit in the critical path
- Deferred: Account settings, accessibility settings, visit-invitation flow, notebook panel (lazy-loaded on first open)
- Never in initial bundle: any large asset library, recorded audio (explicitly banned)

Bird visual assets: procedural where possible (plumage layer is canvas-drawn); species silhouettes are compact SVGs (<5KB each). With 6 species, silhouette total <30KB.

### Initial load optimization

The HTML response from the CDN edge includes an inlined state snapshot (a small JSON blob, <5KB). This eliminates one round trip from the critical path: the client begins rendering the first bird from the inlined snapshot before the JS bundle is fully parsed. The inlined snapshot may be up to 60 seconds stale (CDN TTL), which is acceptable.

### Real User Monitoring

RUM collects (via a lightweight beacon):
- Page load timing
- First-bird-render timing (custom metric; emitted from the rendering worker when first bird is painted)
- Render frame timing distribution (p50, p95, p99 frame durations)
- AudioContext creation success/failure rate
- Simulation tick latency (server-side; included in snapshot response metadata)

All RUM metrics are anonymous aggregates. No per-account dimension. No per-bird dimension. The privacy boundary from `accounts_sync.md` is maintained at the metric definition level.

### Synthetic monitoring

A fleet of headless browsers runs the aviary from 3–5 geographies on a scheduled cadence (every 15 minutes). Synthetic checks report:
- First bird render time
- Audio context success
- Snapshot fetch latency

Alerts on p99 simulation-tick latency >5s; alerts on synthetic first-bird-render >600ms (buffer above the 500ms user budget).

### What we deliberately do not instrument

- Per-user visit counts or visit frequency
- Per-bird interaction history in aggregate analytics
- Personality vector distributions across accounts
- Any metric that could identify what any particular account's bird is doing

---

## 11. Rollout

### v1 launch sequence

**Pre-launch (internal):**
- Drift calibration: run the simulation against 3-week simulated presence patterns; verify measurable drift at 1 week, visible-to-user drift at 3 weeks
- Audio quality review: human listening panel across all 6 species, all mood states, chorus scenarios; confirm per-bird recognizability and no uncanniness artifacts
- Accessibility audit: screen-reader testing across NVDA/JAWS/VoiceOver; reduced-motion mode visual review; keyboard navigation walkthrough
- Performance audit: measure bundle sizes; measure first-bird-render on target devices; verify flat memory profile at 30 minutes; CI checks for memory growth

**Closed beta:**
- 100–500 accounts; specifically recruit reduced-motion and screen-reader users in this cohort
- Instrument: error rates, first-bird-render, audio context failures, event-log append rates
- Manual review of field notebook prose quality (no automated metric; qualitative review by two team members)
- Measure session length distribution (not per-user, anonymized histogram)

**Birds-per-aviary ramp:**
- v1 launches with the 2-bird starter configuration as described
- Third-bird availability trigger (aviary age, not visit count) is calibrated so that ~5% of accounts are eligible at launch (accounts created during the beta that have aged enough)
- Fourth through seventh bird availability intervals are decided based on observed aviary age distribution from the beta cohort
- No "more birds" announcement surface; the offer appears in the existing flow when the account qualifies

**Post-launch:**
- Monitor simulation tick latency p99; alert on >5s
- Monitor first-bird-render synthetic checks; alert on >600ms
- Monitor audio context failure rate; if >5% of sessions, investigate device/browser distribution
- Monthly qualitative review of notebook entry prose across a random sample of generated entries (no per-account data; entries are reviewed in isolation)

### Day-one instrumentation

From the first production session, we collect:
- Aggregate session duration histogram (anonymized, no per-account dimension)
- First-bird-render timing (RUM)
- Audio context error rate
- Simulation tick latency (server-side, per-tick)
- Event-log append error rate
- Magic-link conversion rate (request → consume; no per-email data in this metric)
- Snapshot fetch latency

We do not collect, and do not add later without a deliberate PRD change:
- Per-account session count
- Per-bird offer/listen-in frequency
- Any metric derived from personality vectors

---

## 12. Risks

### Drift calibration drift

**Risk:** The drift filter constants are miscalibrated. Too fast: users notice bird changes session-to-session, producing a Tamagotchi-like feel. Too slow: users see no change at all after weeks, and the promise of a living relationship collapses.

**Mitigation:** The calibration target is codified in the spec (measurable at 1 week, user-visible at 3 weeks). The test harness simulates 21 days of presence patterns and verifies against these targets before release. The filter constants are server-side configuration, not compiled into a binary; they can be adjusted without a client release if post-launch calibration indicates drift.

**Watch signal:** Session-length distribution and return rate (cohort-level, not individual). If users stop returning after week 2, drift may be too slow.

### Sync correctness under concurrent device use

**Risk:** Two devices active simultaneously submit events in the same tick window. If the tick doesn't process both sets of events correctly, drift may be inflated or events may be lost.

**Mitigation:** The event log is append-only; no event is ever overwritten. The tick processes all events since `last_tick_at` in `occurred_at` order; events from both devices appear in the log. The additive drift model means double-presence inflates drift slightly (two devices open = roughly double presence-time in that window) — this is acknowledged as an acceptable approximation. If it becomes a problem, the fix is deduplication of presence pings by session token in the event processor.

### Audio uncanniness

**Risk:** Procedural call synthesis produces calls that are technically variation of the motif but sound robotic, glitchy, or uncanny — breaking the felt-aliveness of the audio.

**Mitigation:** Human listening panels in every pre-launch review. The motif library design is the load-bearing input; motifs need to be designed for musical naturalness, not just syntactic correctness. Frequency and duration envelopes need smooth continuity. The synthesis pipeline should include a soft compressor on individual voice outputs before mixing to prevent clipping artifacts that are the most audible uncanniness signal.

**Escape hatch:** If synthesis quality is not meeting the bar at launch, the fallback is graceful silence with captions — which is the WebAudio-unavailable path. We do not fall back to recorded audio; we ship silence and captions as a designed experience.

### Accessibility regressions

**Risk:** Screen-reader narration degrades to state-list output if the NarrationComposer templates are poorly maintained. Reduced-motion mode shipping as animations-off rather than the designed cross-fade surface. Caption timing desync from calls.

**Mitigation:** Accessibility audit in closed beta with screen-reader and reduced-motion users. Narration output reviewed as part of the prose quality review process (same review cadence as notebook entry prose). Reduced-motion mode is a named rendering path in the pipeline, not a media-query that disables animation CSS — this makes it harder to accidentally degrade.

**Post-launch:** Include accessibility users in the closed beta cohort and collect qualitative feedback explicitly.

### Field notebook voice degradation

**Risk:** Notebook entry prose templates are added by contributors who don't internalize the naturalist voice constraints, producing entries that feel generic, announcement-style, or gamification-adjacent.

**Mitigation:** The voice constraints are named in the PRD and in this plan. The template library should be reviewed before each batch of new templates ships. A small test suite of generated entries is reviewed in the monthly qualitative review cadence.

### Presence-detection inflation

**Risk:** The activity window for the presence pointer/key check is calibrated too long, causing users who briefly glance at the aviary and leave their laptop open to generate presence-time.

**Mitigation:** The activity window should be calibrated toward 2–3 minutes (the user is watching birds without necessarily moving the mouse; this window gives them credit for attentive watching). At 2 minutes, a user who walks away from their laptop generates at most one additional presence-ping before presence is lost. This is acceptable. The window should not be longer than 5 minutes; beyond that, presence-time signals begin to lose meaning.

### "Notice never announce" violations in future features

**Risk:** A contributor adds a toast, a badge, or any announcement-style UI surface in a future feature, breaking the core design principle.

**Mitigation:** The principle is named and explained in the PRD with explicit examples of what would violate it. This plan names it as a risk because it is the violation most likely to occur. Code review gates on any new UI surface introduced into the product should check against this principle explicitly.

### Magic-link abuse

**Risk:** An attacker probes whether accounts exist by observing timing differences in the magic-link request response.

**Mitigation:** The magic-link request endpoint always returns 200 regardless of whether an account with that email exists (see API design). The email dispatch is async; timing is not correlated with the response. Rate-limiting is per-email at a threshold that prevents enumeration without blocking legitimate users who request multiple links.

---

## Appendix: Implementation sequencing (recommended order)

The following is advisory, not binding. A team that can parallelize may do so.

1. **Auth service** — magic link, session tokens, email dispatch; unblocks everything that requires auth
2. **Data model and DB schema** — account, bird, aviary, event log, notebook, visit records
3. **Simulation tick worker** — mood transitions first (simpler), then drift function, then notebook entry generation
4. **State snapshot API** — simple GET of canonical state; enables client development
5. **Event log API** — POST /aviary/events; enables presence and interaction event flow
6. **Core rendering pipeline** — scene composition, bird rendering, idle micro-motion; can begin with static mock snapshots
7. **Audio worklet** — procedural call synthesis for one species first; validate quality before expanding to all six
8. **Listen-in interaction** — audio re-balance; depends on audio worklet
9. **Offer interaction** — client-side, event-log, mood-driven reaction; depends on snapshot API + event log
10. **Settle gesture** — lightweight; depends on snapshot API + event log
11. **Field notebook UI** — depends on notebook entry API; notebook entry generation is in the tick
12. **Accessibility: screen-reader narration** — add ARIA live region and NarrationComposer
13. **Accessibility: reduced-motion mode** — rendering pipeline switch
14. **Accessibility: captioning** — audio worklet → caption text → DOM render
15. **Account management UI** — settings, session list, export, deletion
16. **Visit invitation flow** — invite, revoke, visitor render path
17. **Performance hardening** — bundle audit, first-bird-render optimization, memory profile
18. **Closed beta** — with 100–500 accounts, including accessibility cohort
19. **Drift calibration review** — measure against 3-week test data; adjust filter constants as needed
20. **v1 launch**
