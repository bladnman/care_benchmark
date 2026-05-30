# Pocket Aviary — Implementation Plan v1.0

## 1. Scope and Non-Goals

### 1.1 In Scope (v1)

- **Two starter birds** per new account; optional adoption flow to add birds up to a cap of seven
- **Single canonical aviary** per account, single-user accounts with magic-link sign-in
- **Multi-device sync** via server-side canonical state — laptop, phone, any modern browser
- **Bird engine**: procedural calls, mood system, personality vector drift, presence accounting
- **Interactions**: return-greeting, listen-in (single-bird focus with mix rebalance), offer (seed, song fragment, still pool), settle gesture, field notebook (auto-generated naturalist prose)
- **Visit invitations**: host invites a friend by email; friend sees read-only ambient view
- **Accessibility**: screen-reader narration (running naturalist prose), reduced-motion mode (cross-fade rendering, not stripped), call captions (mood-shaped prose)
- **Performance**: <2MB initial JS bundle, <500ms time-to-first-bird on 4G/mid-tier mobile, 60fps idle motion on 5-year-old laptop, no memory growth over 30 minutes

### 1.2 Out of Scope (v1)

- Native mobile apps (web-only at v1)
- Gamification: no streaks, achievements, badges, levels, scores, counters, or "days visited" surfaces — hard rule
- Tamagotchi mechanics: birds do not die, hunger, decay, or show distress; relationship is observational, not custodial
- Social network surfaces: no profiles, follows, public discovery, leaderboards, shared exploration
- Multi-aviary accounts, customizable scenes, panning/zooming, payment features

### 1.3 Non-Goal Compliance

Every feature review should ask: "Does this teach the user that presence is for a counter rather than the birds?" If yes, it does not ship. The non-goals are load-bearing design constraints, not a backlog to revisit later.

---

## 2. Architecture

### 2.1 Service Shape

The system is a **server-authoritative simulation with stateless clients**. The server holds the single canonical aviary state (birds, personality vectors, moods, positions, tick timer). Clients render what the server says; they never own state.

```
                    ┌─────────────────────────────────────────────┐
                    │            Simulation Service               │
                    │  (server-side tick ~1/min, event log,       │
                    │   personality vectors, mood state,          │
                    │   canonical bird positions)                │
                    │                                             │
                    │  ┌─────────────┐    ┌──────────────────┐   │
                    │  │ Event Log   │    │ Canonical State  │   │
                    │  │ (append-only│    │ (per-account     │   │
                    │  │  interaction│    │  birds + vectors)│   │
                    │  │  events)   │    │                  │   │
                    │  └─────────────┘    └──────────────────┘   │
                    └─────────────────────────────────────────────┘
                                     │ ▲
                         state       │ │  interaction events
                        snapshots    │ │  (offer, listen-in,
                                     │ │   settle, presence)
                    ┌────────────────┴─┴────────────────────┐
                    │              REST / SSE                 │
                    │        (small JSON snapshots)          │
                    └────────────────┬──────────────────────┘
                                       │
                    ┌──────────────────┴──────────────────────┐
                    │              Clients                    │
                    │  Browser (Chrome/Safari/Firefox/Edge,   │
                    │   last 2 versions)                      │
                    │  - WebAudio procedural synthesis        │
                    │  - Canvas/SVG bird rendering           │
                    │  - Interpolation between snapshots      │
                    └────────────────────────────────────────┘
```

### 2.2 Client/Server Split

**Server owns**: All simulation state — personality vectors, moods, bird positions, call timing, tick phase, event log, account records, visit invitations, notebook entries.

**Client owns**: Local rendering interpolation, WebAudio synthesis, DOM lifecycle, user input capture, presence signal detection.

**Client never writes personality state**. The client sends interaction events to the server; the server tick consumes them and updates personality. This is not a stylistic preference — it is the architectural property that makes multi-device sync coherent and prevents last-write-wins corruption of drift history.

### 2.3 Technology Stack (Implementation Detail)

- **Simulation service**: Node.js with a simple in-process store (or Redis-backed for horizontal scaling). Event log as an append-only series. No external database committed to until scale forces it; SQLite for canonical state at small scale is acceptable.
- **Client**: Vanilla JS with WebAudio API for procedural synthesis. No heavy framework; the bundle budget demands a lean render path. React is not ruled out but must be justified against the 2MB budget.
- **Auth**: Magic link email flow; session tokens stored in HttpOnly cookies. No passwords.
- **CDN**: State snapshot payloads served from CDN edge for fast first-bird delivery.

---

## 3. Data Model

### 3.1 Account

```json
{
  "id": "uuid-v4",           // synthetic, not derived from email
  "email_encrypted": "...",  // single encrypted field
  "created_at": "ISO8601",
  "deleted_at": null,        // soft-delete timestamp, null = active
  "settings": {
    "visit_notifications": false,
    "audio_captions": false,
    "reduced_motion": false,
    "screen_reader_narration": false
  }
}
```

### 3.2 Bird

```json
{
  "id": "uuid-v4",              // stable internal ID, never recycled
  "account_id": "uuid-v4",
  "species": "string",          // from the species pool (~6 options)
  "name": "string",             // user-assigned, editable
  "adopted_at": "ISO8601",
  "personality_vector": {
    "boldness": 0.0-1.0,        // normalized scalar, server-authoritative
    "social_warmth": 0.0-1.0,
    "vocal_frequency": 0.0-1.0,
    "plumage_saturation": 0.0-1.0,
    "curiosity": 0.0-1.0
  },
  "current_mood": "wary|content|curious|drowsy|alert",
  "mood_timer_seconds": 0,      // countdown to next mood transition
  "position": {
    "perch_zone": "front|middle|back",
    "animation_state": "preening|scanning|head_tilt|..." // for interpolation
  },
  "call_grammar_version": "v1"   // allows evolution without migration
}
```

**Constraint**: The user never sees personality vector values. No stats panel, no debug view, no "show me how my bird is doing" surface. Vector values are felt through behavior, not read as numbers.

### 3.3 Event Log Entry (Append-Only)

```json
{
  "id": "uuid-v4",
  "account_id": "uuid-v4",
  "bird_id": "uuid-v4|null",     // null for aviary-level events
  "type": "presence|listen_in_start|listen_in_end|offer|settle|adopt|rename",
  "timestamp": "ISO8601",
  "payload": {}                  // type-specific: duration for presence, item for offer, etc.
}
```

The event log is the only thing clients write. The tick consumes it in order.

### 3.4 Field Notebook Entry

```json
{
  "id": "uuid-v4",
  "account_id": "uuid-v4",
  "timestamp": "ISO8601",
  "prose": "string"             // naturalist prose, lowercase, present-tense, specific
}
```

Entries are generated by the simulation tick when something worth noting has happened — not on every session, not on every interaction. Sparsity is a design constraint: a notebook entry per session dilutes the entries that matter.

### 3.5 Visit Invitation

```json
{
  "id": "uuid-v4",
  "host_account_id": "uuid-v4",
  "visitor_email_encrypted": "...",
  "token": "uuid-v4",           // one-time link token
  "created_at": "ISO8601",
  "expires_at": "ISO8601",       // created_at + 30 days
  "revoked_at": null,
  "used_at": null
}
```

---

## 4. API Surface

### 4.1 Authentication

- `POST /auth/request-link` — body: `{email}` → emails magic link
- `GET /auth/verify?token=<uuid>` — consumes link, sets HttpOnly session cookie, redirects to aviary
- `POST /auth/sign-out` — clears session
- `GET /auth/sessions` — list active sessions (for account settings revocation)
- `DELETE /auth/sessions/:id` — revoke a session

### 4.2 State Snapshots

- `GET /aviary` — returns current canonical state snapshot (bird positions, moods, call timings, time-of-day phase, weather state, notebook recent entries). Small JSON — kilobytes, not megabytes.
- `GET /aviary/events?since=<timestamp>` — returns events since a given timestamp (for keepalive sync after gap)
- `GET /aviary/notebook` — paginated notebook entries (newest first)

### 4.3 Interaction Events (Client → Server)

- `POST /aviary/events` — body: `{type, bird_id, payload}` — write an interaction event to the append-only log. Accepted types: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`. Each write is idempotent-keyed (retry-safe).

### 4.4 Bird Management

- `PATCH /aviary/birds/:id` — rename a bird (name only; personality untouched)
- `POST /aviary/birds` — adopt a new bird (system selects species; user names it)
- `GET /aviary/birds/:id/call_grammar` — returns the motif library for the bird's species (client-side synthesis needs this)

### 4.5 Visit Invitations

- `POST /aviary/visits/invite` — body: `{email}` → sends one-time link to visitor
- `GET /aviary/visits` — visit log: list of who visited and when, current outstanding invites
- `DELETE /aviary/visits/:invite_id` — revoke an invitation

### 4.6 Visitor Path

- `GET /visit/:token` — validates one-time visit token; returns read-only snapshot stream (SSE or polling) of host's aviary state. No interaction events recorded from visitor sessions. No co-presence mechanism.

### 4.7 Account

- `GET /account/export` — generates JSON snapshot of aviary state; emails download link to verified address
- `DELETE /account` — initiates soft-delete (30-day window before hard delete)
- `PATCH /account/settings` — update accessibility and notification preferences

---

## 5. Simulation Engine Design

### 5.1 Server-Side Tick

The simulation tick runs on the server at ~1 per minute (exact value calibrated during build). The tick is the only writer of personality state.

**Tick inputs**: the event log since last tick.

**Tick outputs**:
- Updated personality vectors (drift applied)
- Mood transitions (mood timer decremented, state machine fires)
- Position updates (perch zone changes based on mood/personality)
- Notebook entry generation (if something worth noting happened)
- Weather event scheduling (ambient rain, wind — probabilistic, ~2-3x per week)
- Visit log updates (silently recorded, not surfaced to host unless they check)

### 5.2 Drift Function

Drift is a **low-pass filter over presence-and-interaction signals**, slow to protect the "no visible change in a single session" invariant.

**Inputs in rough order of weight**:
1. Presence-time (dominant) — user sits and watches; birds drift toward expressive (higher boldness, social warmth, vocal frequency, plumage saturation)
2. Listen-in — focusing a bird is a strong attention signal; social_warmth and vocal_frequency drift up for that bird
3. Offers — accepting an offer: small curiosity drift toward the offering bird; offering near a bird: small boldness drift
4. Settle — ends presence cleanly; no directional drift

**Calibration target**: measurable drift in instruments after ~1 week of regular visits; visible drift to user after ~3 weeks. No single session produces a visible change.

**Monotonicity**: traits only move up with positive presence; they do not move down on neglect. A neglected bird becomes ambient (quieter, less frequent greeting), not distressed. This is the implementation of "no Tamagotchi" — absence is fine, not penalized.

### 5.3 Mood State Machine

Mood is a small enumerated state per bird: `wary`, `content`, `curious`, `drowsy`, `alert`. Transitions are shaped by:
- Time of day in user's local timezone (drowsy at dusk, alert in early morning)
- Recent interactions (offer accepted → nudge toward content)
- Ambient events (rain → brief vocal dampening across aviary; another bird's alarm call → nearby birds nudge toward wary)
- Bird's own personality (high-boldness bird less likely to enter wary on same input)

Mood persists across sessions. At session-end, the mood is what the bird has; at session-start, it's the same unless the tick has advanced it. The user should never notice mood "snapping" to a default on tab open.

### 5.4 Call Grammar Runtime

Each bird species has a **motif library**: a small set of melodic motifs (timing patterns, pitch contours, envelope shapes). At runtime, the bird's call is composed by selecting motifs, combining them with personality-shaped timing and pitch variation, and applying the current mood's modulation.

A bird with high vocal_frequency calls more often when unobserved and joins the chorus more readily. The vocal_frequency trait affects inter-call interval and probability of joining a chorus event.

Two birds calling at once produce a real chorus — two independent synthesis streams mixed — not two identical loops in a phase-canceling stack. This is what makes seven the cap: beyond seven, the ear cannot reliably separate individual call signatures from the blended output.

### 5.5 Bird-to-Bird Interaction

Birds interact with each other: a call from one bird can prompt a response from another; a wary mood in one bird tends to spread to neighbors; chorus events emerge when two birds with high vocal_frequency happen to call in the same window. Bird-to-bird interaction is what makes the aviary feel like a small social system rather than a row of independent NPCs.

---

## 6. Sync Model

### 6.1 Multi-Device Coherence

Because the server is the only writer of personality state, multi-device sync is a property of the architecture, not a feature to implement. The user signs in on their laptop and their phone; both devices pull the same canonical state snapshot; both render the same birds, moods, positions. There is nothing to sync because both clients read from the same record.

### 6.2 Client State Consumption

The client:
1. Opens the aviary tab
2. Requests current snapshot from `/aviary`
3. Begins rendering birds at their current positions with their current animations
4. Interpolates between snapshots for smooth motion (bird at perch A in snapshot N and perch B in snapshot N+1 moves smoothly between them, no teleport)
5. Pulls a fresh snapshot on: visibility change (tab becomes visible after being hidden), long render-frame gap (laptop resuming from sleep), low-frequency keepalive while tab is visible
6. Writes interaction events to `/aviary/events` as they occur

### 6.3 Conflict Prevention

**No last-write-wins for personality state**. Personality drift is additive server-authored deltas, never client-submitted absolute values. A client never sends "set boldness to 0.62"; a client sends "user was present for 12 minutes." The server tick decides what that means for boldness.

The server tick processes the event log in order. Only the simulation service writes personality vectors. No code path exists under which a client mutates personality directly.

### 6.4 Sync Conflict Surfaces

In rare cases (magic-link replay, mid-write session timeout, server outage), the user may encounter a sync conflict surface. This surface drops out of naturalist voice into matter-of-fact tone: direct, clear, no naturalist phrasing. "We couldn't sign you in. The link may have expired. Try requesting a new link." The exception is named and intentional.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene Composition

Single horizontal scene, three perch zones (front, middle, back). No panning, no scrolling, no zooming. Birds choose perches based on mood and personality — the user reads this as signal, not as layout control.

Foreground/background separation: birds and perches on a middle plane; soft background foliage and sky behind; occasional foreground branch or leaf passes through. Parallax is subtle.

The aviary palette is calm and naturalist — soft blues, greens, warm browns, muted ochres. No saturated UI accents. Palette specifications live in the design system spec.

### 7.2 Idle Micro-Motion

Birds are never still in a way that reads as paused. Idle motion includes:
- Preening
- Scanning the scene
- Head-tilting toward sounds
- Small body-shuffle on the perch to reset weight

Idle motion runs continuously. It does not pause when the tab loses focus (the client stops rendering when hidden; the simulation continues server-side).

Idle motion is mood-shaped. A wary bird perches further back and scans more; a content bird preens; a curious bird tilts toward sounds; a drowsy bird sits low with feathers fluffed. The user reads mood from motion without being told.

### 7.3 Transitions

**First frame**: birds mid-action. No "wake up" animation, no fade-from-static, no entry sequence. The aviary appears with its motion already in progress, because it has been — the simulation has been ticking on the server.

**Perch transitions**: cross-fade between perches, not animated flight paths. (In reduced-motion mode, this is the primary transition type.)

**State transitions (mood changes, call events)**: handled by the animation state machine; no hard cuts.

**Loading state** (if snapshot is slow): a quiet field — soft sky color, faint motion cues — not a spinner. A spinner says "machine"; we are not selling a machine.

### 7.4 Reduced-Motion Mode

Users with `prefers-reduced-motion` or who opt in via settings receive a designed surface, not a stripped fallback.

- Micro-motion replaced by slow cross-fades between still poses
- Flight transitions become cross-fades between perches
- Ambient leaf drift removed; ambient color shifts (day to evening) remain, slowed
- Calls still play at full quality (or caption per user settings)
- Birds still drift, mood still changes, notebook still notices

The cross-fade rendering is its own calm aesthetic. A vestibular user gets a Pocket Aviary that is calmer and slower, not broken.

### 7.5 Responsive Handling

Scene compresses horizontally on narrow viewport; widens on wide viewport. Aspect ratio preserved. No bird is ever cropped out of frame. Maximum and minimum viewport handling are implementation details in the rendering spec.

---

## 8. Audio Pipeline

### 8.1 Procedural Call Synthesis

All calls are synthesized client-side via WebAudio from the motif library. No recorded audio. This is non-negotiable for two reasons:
1. Looped audio is the audible signature of dead software — the user hears the same call twice, exactly the same way, and the spell breaks.
2. The chorus mechanic requires real-time per-call variation. Two recorded loops layered produce phase-canceling artifacts the ear catches even when each individual call sounds procedural.

The client fetches the species' motif library on load (`GET /aviary/birds/:id/call_grammar`). Calls are composed at runtime: motifs selected and combined with personality-shaped timing and pitch variation, mood modulation applied.

### 8.2 Chorus Mixing

Two or more birds calling at once produce a real chorus — independent synthesis streams mixed in the WebAudio graph. The vocal_frequency trait affects inter-call interval and probability of joining a chorus event.

The cap of seven birds is set by this: seven is the ceiling at which per-bird call signatures remain individually recognizable to a typical listener. Beyond seven, the chorus blurs into ambient and the per-bird relationship collapses.

### 8.3 Listen-In Mix

When the user listens in on a single bird, that bird's call rises in the mix while others quiet to ambient. The mix change is gradual on engage and on disengage — a slow ramp, not a hard cut. A hard cut converts the aviary into a UI of soloable tracks; a slow ramp maintains the "place where multiple things are happening at once" feel.

Other birds drop in mix but never go silent. Silencing them entirely would teach the user that the aviary is a set of things to switch between. The listen-in mix is a re-balance, not a mute.

Listen-in disengages when: the user clicks the focused bird again, focuses a different bird, clicks empty space in the aviary, or moves keyboard focus away.

### 8.4 WebAudio Fallback

If WebAudio is unavailable (older browser, audio context permission denied, hardware issue), the aviary plays in graceful silence with captions on by default. No recorded audio fallback is shipped. The "no recorded audio" rule is unconditional — the bundle budget and the chorus mechanic both force it. Silence with captions is a better fallback than canned audio.

---

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Narration

The screen reader hears running naturalist prose narration of the aviary state, updated on a slow cadence (~one update per 30-60 seconds at idle; faster on user-initiated events like return-greeting or successful offer). Not a list of state changes — not "Pip is at perch 2" or "Wren mood: content." Running prose:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Narration is generated from the same state the visual surface reads from, expressed in the same voice as the field notebook. User-initiated events get a small priority bump but are still written as observations.

The narration cadence is slow — high-frequency narration would overwhelm the screen reader's queue. The pacing matches the slow rhythm of the visual aviary.

### 9.2 Call Captions

Users opt in via accessibility settings. Captions are short naturalist prose descriptions of what each call sounds like in the bird's current mood:

> a soft three-note rise
> a low trill, paused, low trill again
> a single sharp call from the back perch

Captions appear near the calling bird, fade in and out with the call, using the same naturalist voice as the rest of the product. Caption text is generated from the procedural call grammar at runtime — each caption matches what was actually played.

### 9.3 Keyboard Navigation

All interactive surfaces are reachable by keyboard:
- Tab moves through the top bar items
- Entering the aviary scene with Tab focuses the first bird
- Arrow keys move focus between birds
- Enter triggers listen-in on the focused bird
- Escape exits listen-in
- The offer affordance opens with a top-bar shortcut; the offer flow is fully keyboard-navigable
- Settle gesture reachable from top bar

Focus indicators are visible against the aviary background (soft, high-contrast outline; design spec specifies exact treatment).

### 9.4 Contrast

All user-copy text — top bar labels, settings, account surfaces, error surfaces, captions, visual narration — passes WCAG AA contrast minimum. The aviary scene itself carries no user copy except in the top bar, so the constraint applies primarily to chrome.

---

## 10. Performance Budgets and Observability

### 10.1 Bundle Budget: <2MB initial JS (gzipped)

Drives: lean render path (no heavy framework unless justified), procedural visual generation, aggressive code-splitting for low-frequency surfaces (account settings, accessibility settings, visit-invitation flow), small SVG/bitmap assets.

### 10.2 Time to First Bird: <500ms on mid-tier mobile over 4G

Requires: bundle budget, CDN-edge state snapshot delivery, render path that doesn't wait for non-critical assets before drawing the first bird. The 500ms threshold is the affective-perf bridge: below it, the aviary feels like it was already running; above it, the user notices load.

### 10.3 Runtime: 60fps idle motion on 5-year-old mid-range laptop

Applies to a 30-minute session, not just the first minute.

### 10.4 Memory: No growth over 30 minutes

Procedural audio buffers are reused. No per-call allocation that isn't freed. Notebook entries scrolled out of view do not retain references. Worker threads and audio contexts are bounded. This is a CI-enforced test, not a guideline.

### 10.5 Observability

**Synthetic checks**: automated browsers running the aviary on a schedule from common geographies.

**RUM (aggregate only)**: page load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies. None of this contains per-bird state or per-account interaction history — the privacy boundary is honored at the metric definition level.

**Error budget**: simulation-tick latency p99 alarms if it exceeds 5 seconds. The tick is supposed to be much faster; 5s p99 catches degradation early.

---

## 11. Rollout

### 11.1 V1 Ship

Start with two birds per new account (the adoption flow names them). User does not pick from a catalog — the first encounter is meeting an animal, not configuring an avatar.

V1 ships with: single-user accounts (magic link), multi-device sync, field notebook, presence accounting, visit invitations (off by default, opt-in per invite), screen-reader narration, reduced-motion mode, call captions.

### 11.2 Birds-Per-Aviary Ramp

New birds become available based on **aviary age** — not visit count, not interaction score, not paid tier. A new species offer appears at intervals tied to how long the aviary has existed. The pacing matches the rhythm of a relationship deepening.

This refusal of the gamification trap is deliberate: the user should not learn that more attention earns more stuff. More birds is a function of time, not effort.

### 11.3 Instrumentation from Day One

- Drift calibration monitoring (are birds drifting at the intended rate across the population?)
- Presence signal honesty (are all three conjunction conditions firing correctly? — this is the most likely silent failure mode)
- Audio-context error rates (how often is WebAudio unavailable or denied?)
- Time-to-first-bird distribution (is the 500ms budget holding across geographies and devices?)
- Tick latency distribution (is the 5s p99 alarm threshold appropriately set?)

---

## 12. Risks

### 12.1 Drift Calibration Risk

**Risk**: Drift function calibrated too fast turns birds into a stat-management exercise; too slow makes the product feel like a screensaver where nothing matters.

**Mitigation**: The calibration target (measurable in instruments at 1 week; visible to user at 3 weeks) is named and testable. Build an instrument harness that runs synthetic presence histories and asserts personality vectors change at the intended rate. Calibrate during build, not after launch.

### 12.2 Sync Correctness Risk

**Risk**: Any code path that allows a client to write personality state directly creates a last-write-wins race that silently deletes drift history across devices.

**Mitigation**: No such code path exists by architecture. The only write from client is append-only interaction events. Personality vectors are written only by the simulation tick. Review all code that touches bird state to confirm this invariant holds.

### 12.3 Audio Uncanniness Risk

**Risk**: Even procedural synthesis can produce calls that feel canned if motif variation is too limited or if phase relationships between concurrent calls produce audible artifacts.

**Mitigation**: Extensive listen-test during development. The motif library needs enough variation that a user hearing the same bird call twice in one session doesn't hear an identical pattern. The chorus mixing needs real-time variation, not just stacked loops. WebAudio fallback (graceful silence + captions) ships for cases where synthesis fails.

### 12.4 Accessibility Regression Risk

**Risk**: Reduced-motion mode or screen-reader narration is treated as a polish item and lands after launch, quietly telling accessibility users the product wasn't designed for them.

**Mitigation**: Both ship with v1. Reduced-motion is its own designed surface, not a fallback. Screen-reader narration uses running naturalist prose, not state-list labeling. Accessibility surfaces are first-class, not a checklist.

### 12.5 Presence Signal Inflation Risk

**Risk**: The presence definition (conjunction of three signals: visibility + focus + recent activity) is implemented incorrectly — perhaps only visibility is checked — silently inflating presence-time across the user base and corrupting drift.

**Mitigation**: The three-condition conjunction is documented and tested. The activity window (a few minutes, calibrated during build) must be long enough that watching birds without moving doesn't lose presence. The test suite includes a synthetic presence-signal injector that verifies the conjunction fires correctly.

### 12.6 Notebook Voice Dilution Risk

**Risk**: Field notebook entries are written as event logs ("session started at 7:43") rather than naturalist observations, breaking the voice across the product's most visible prose surface.

**Mitigation**: Notebook entry generation is a designed output of the simulation tick, not a generic event-logger. Entries are rare (~one every few days for an active aviary), specific, and written in the same naturalist voice as the rest of the product. Sparsity is a design constraint; an entry per session would dilute the entries that matter. The rule is: if it wouldn't be in a real naturalist's field notebook, it doesn't go in the product's notebook.

---

*End of plan. This plan is for a frontier engineering team. Do not implement the product. Deliverable is the plan only.*