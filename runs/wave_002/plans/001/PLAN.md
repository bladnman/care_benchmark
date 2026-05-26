# Pocket Aviary — Implementation Plan

## 1. Scope

### What ships in v1

- **Two starter birds** per new account — selected system-side from the species pool, presented as "the birds that arrived." No catalog, no user species selection at adoption.
- **Up to seven birds** per aviary. New bird-offer events appear based on aviary age (not visit count or interaction score), paced to roughly: third bird at ~3 months, growing to five or six across the first year. No paid tiers, no unlock mechanics beyond time.
- **Single-user accounts** with magic-link email sign-in. No passwords, no SSO at v1. Synthetic UUID account identifiers; email stored encrypted, never used as a key outside the account record.
- **Multi-device sync** — the same aviary on laptop and phone, achieved via server-side canonical state (not client-to-client sync).
- **Field notebook** — auto-generated naturalist observations, read-only, prose in naturalist voice, roughly one entry every few days (sparser for active users, never per-session).
- **Presence accounting** — strict 3-condition check: `visibilityState === "visible"` AND `document.hasFocus()` AND recent pointer/key activity within a configurable timeout window (~3 minutes, calibrated during build).
- **Visit invitations** — host invites a visitor by email; visitor sees a read-only ambient view. Invites expire after 30 days unused. Revocable at any time. No co-presence, no chat, no avatars, no comments. No public discovery. No friend-visited notification by default (opt-in toggle exists in settings, off by default).
- **Screen-reader narration** — naturalist prose, updated on a slow 30–60 second cadence at idle.
- **Reduced-motion mode** — cross-fade-based rendering (not "animations off"). Honors `prefers-reduced-motion` and a manual accessibility-settings toggle.
- **Call captioning** — opt-in prose captions for each procedural call.
- **WCAG AA contrast** on all user-copy text.
- **Keyboard navigation** across all interactive surfaces.
- **Account export** (JSON snapshot, emailed) and **account deletion** (30-day soft window, then hard).

### What is explicitly out of scope

- Native mobile apps, gamification (streaks, achievements, badges, scores, green-dot calendars, leaderboards), Tamagotchi mechanics (death, hunger, distress, happiness meters), social network surfaces (profiles, follows, public feeds, discovery, comments, ranking), push notifications, payments, shared aviaries, customizable scenes, multi-aviary accounts, public discovery.
- No textual "Welcome back!" toasts, banners, or modals — ever. The bird greeting is the entire welcome surface.
- No exposing personality vector values to the user — not in stats panels, debug views, or any surface.

---

## 2. Architecture

### Service topology

A three-service system, each independently deployable:

| Service | Responsibility | Language recommendation |
| --- | --- | --- |
| **API Gateway / Auth** | Magic-link sign-in, session token issuance/revocation, email delivery, account CRUD | TypeScript (Node.js) or Go |
| **Simulation Engine** | Server-side tick, personality drift, mood transitions, call-timing events, state snapshot generation, event-log consumption | Rust or Go (CPU-bound, needs predictable latency) |
| **Client (SPA)** | Rendering, WebAudio synthesis, interaction capture, state interpolation, accessibility surfaces | TypeScript + React (or equivalent solid-state framework) |

**Database**: PostgreSQL for canonical state (birds, personality vectors, moods, notebook entries, accounts, visit logs). The simulation engine is the only writer of bird/personality/mood state. The API gateway writes account records, session tokens, and interaction events into an append-only event log table.

**Event log**: A PostgreSQL table (`interaction_events`) with account UUID, timestamp, event type, and event payload (JSONB). Append-only. The simulation tick consumes from this log in time order.

**CDN**: The client SPA is served from a CDN edge. State snapshots are small enough to serve from the same edge with a short cache TTL (no cached snapshots — each pull is live) but the server itself sits behind the CDN for TLS termination and geographic reach.

### Client/server split

- **Server owns**: personality vectors, mood state, bird position/perch assignments, drift deltas, call-gate timing events, notebook entry generation, account state, session management, visit invitations, visit logs.
- **Client owns**: rendering, procedural call synthesis (WebAudio), ambient micro-motion (leaf/feather drift), interaction capture (presence pings, offer events, listen-in start/end, settle), screen-reader narration text assembly, caption generation, reduced-motion rendering mode.
- **Boundary**: The server sends a state snapshot. The client renders it and interpolates between snapshots. The client writes interaction events to the append-only event log. The client never writes bird personality or mood state directly.

### Render pipeline boundary

The client's render loop is split into three layers:

1. **Scene compositor** — draws the aviary background (sky gradient, foliage, perches, ambient particles) and positions birds. Reads state snapshot.
2. **Bird renderer** — per-bird SVG/canvas rendering with pose selection from mood-state + personality traits. Driven by idle-motion state machine.
3. **Audio engine** — a WebAudio worklet that consumes call-timing events from the state snapshot and synthesizes calls from per-species motif libraries.

These three layers communicate via a shared state bus, not direct coupling — the scene compositor doesn't call the audio engine; both read the same snapshot.

---

## 3. Data Model

### Core entities

```
Account
  id: UUID (pk)
  email: string (encrypted at rest)
  email_verified: boolean
  created_at: timestamp
  deleted_at: timestamp | null (soft-delete)
  deletion_initiated_at: timestamp | null
  aviary_age_start: timestamp (the moment the first two birds were adopted)

Bird
  id: UUID (pk)
  account_id: UUID (fk -> Account)
  stable_id: UUID (immutable — survives rename, sync, migration)
  species_id: string (enum: 6 species)
  name: string (user-assigned, renameable)
  adopted_at: timestamp
  personality_vector: {
    boldness: float,
    social_warmth: float,
    vocal_frequency: float,
    plumage_saturation: float,
    curiosity: float
  }
  current_mood: enum (wary | content | curious | drowsy | alert)
  current_perch_zone: enum (front | middle | back)
  drift_history: jsonb[] (append-only log of delta records for audit)

NotebookEntry
  id: UUID (pk)
  account_id: UUID (fk -> Account)
  created_at: timestamp
  prose: text (naturalist voice, already rendered)

InteractionEvent
  id: UUID (pk)
  account_id: UUID (fk -> Account)
  bird_id: UUID | null (nullable for settle/presence events)
  event_type: enum (presence_ping | listen_in_start | listen_in_end |
    offer_seed | offer_song_fragment | offer_still_pool | settle |
    settle_undo | return_greeting_triggered)
  event_payload: jsonb
  recorded_at: timestamp
  consumed_by_tick: boolean (default false)

Session
  id: UUID (pk)
  account_id: UUID (fk -> Account)
  token_hash: string
  device_label: string
  created_at: timestamp
  revoked_at: timestamp | null

VisitInvitation
  id: UUID (pk)
  host_account_id: UUID (fk -> Account)
  visitor_email: string
  token_hash: string
  created_at: timestamp
  expires_at: timestamp
  revoked_at: timestamp | null

VisitLog
  id: UUID (pk)
  invitation_id: UUID (fk -> VisitInvitation)
  started_at: timestamp
  ended_at: timestamp | null
  duration_seconds: int | null
```

### State snapshot format

The client pulls a JSON snapshot per `GET /aviary/{accountId}/snapshot`:

```json
{
  "snapshot_id": "uuid",
  "generated_at": "iso8601",
  "day_phase": "morning|midday|evening|night",
  "weather": null | {"type": "rain"|"wind", "intensity": 0.0-1.0, "remaining_seconds": int},
  "birds": [
    {
      "stable_id": "uuid",
      "name": "Pip",
      "species_id": "warbler",
      "mood": "content",
      "perch_zone": "front",
      "position": {"x": 0.35, "y": 0.6},
      "pose": "preening_head_left",
      "plumage_saturation": 0.72,
      "vocal_frequency": 0.55,
      "upcoming_call_at": "iso8601|null",
      "upcoming_call_motif": "soft_two_note|null"
    }
  ],
  "notebook_recent": ["entry text", "..."],  // last 5 entries for quick display
  "settled": false
}
```

This snapshot is kilobytes, not megabytes. No personality vector values — the client receives derived pose/mood/saturation, not raw traits.

---

## 4. API Surface

### Auth endpoints (API Gateway)

| Method | Path | Description |
| --- | --- | --- |
| POST | `/auth/request-link` | Send magic link to email. Body: `{email}`. Rate-limited per email. |
| GET | `/auth/verify-link?token={token}` | Consume magic link. Issues session cookie + returns account UUID. |
| POST | `/auth/sign-out` | Revoke current session token. |
| GET | `/auth/sessions` | List active sessions (for revocation UI). |
| DELETE | `/auth/sessions/{session_id}` | Revoke a specific session. |

### Aviary state endpoints

| Method | Path | Description |
| --- | --- | --- |
| GET | `/aviary/{accountId}/snapshot` | Pull current state snapshot (authenticated, account-scoped). Returns `StateSnapshot`. Called on: tab-open, visibility-change after hidden, keepalive poll (~10s while visible). |
| GET | `/aviary/{accountId}/notebook?after={entryId}` | Pull notebook entries (paginated). |
| GET | `/aviary/{accountId}/export` | Request account export (emailed). |

### Interaction event endpoints (client writes)

| Method | Path | Description |
| --- | --- | --- |
| POST | `/aviary/{accountId}/events` | Append interaction events. Body: `{events: [InteractionEvent, ...]}`. Idempotency key per event. Returns 202. |
| POST | `/aviary/{accountId}/presence` | Lightweight presence heartbeat. Body: `{bird_ids: ["uuid", ...], timestamp}` — which birds are being observed currently (for listen-in tracking). Returns 200 with minimal ack. |

Events are batched client-side and flushed every ~5 seconds or on page unload (via `sendBeacon`), whichever comes first. The presence heartbeat is on its own timer (~30 seconds) and is separate from the interaction-event batch to avoid presence data being lost in a batch flush.

### Visit endpoints

| Method | Path | Description |
| --- | --- | --- |
| POST | `/aviary/{accountId}/visits/invite` | Create invitation. Body: `{visitor_email}`. |
| DELETE | `/aviary/{accountId}/visits/invite/{inviteId}` | Revoke invitation. |
| GET | `/aviary/{accountId}/visits/log` | Retrieve visit log. |
| GET | `/visit/{token}` | Visitor landing — returns host's aviary snapshot, read-only. No auth required, just the token. |

### Bird management endpoints

| Method | Path | Description |
| --- | --- | --- |
| POST | `/aviary/{accountId}/birds/name/{birdStableId}` | Rename a bird. Body: `{name}`. |
| POST | `/aviary/{accountId}/birds/offer` | Submit an offer event. Body: `{bird_id, offer_type}`. (Also goes through the events endpoint; this is syntactic sugar.) |

### Account management endpoints

| Method | Path | Description |
| --- | --- | --- |
| GET | `/account/{accountId}` | Account metadata. |
| POST | `/account/{accountId}/delete` | Initiate deletion. |
| POST | `/account/{accountId}/restore` | Undo deletion within soft window. |
| PUT | `/account/{accountId}/settings` | Update notification preferences. |

### Architectural note

The API Gateway routes auth/account/visit endpoints and proxies aviary-state requests to the Simulation Engine. The Simulation Engine is the authority on state snapshots. The Gateway is the authority on auth and account state. Interaction events are written to the Gateway, which appends them to the event log; the Simulation Engine reads from the event log on each tick.

---

## 5. Simulation Engine Design

### Tick cadence and lifecycle

- **Tick interval**: ~60 seconds (configurable, calibrated during build). The tick runs whether or not any client is connected.
- **Per-account tick**: Each account's aviary is ticked independently. For an account with no recent events and no active sessions, the tick does minimal work (advance mood timers, check time-of-day transitions).
- **Tick ordering**: The simulation engine maintains an `event_offset` per account — the timestamp of the last consumed event. On each tick, it reads un-consumed events from `interaction_events` WHERE `account_id = ? AND recorded_at > ? AND consumed_by_tick = false ORDER BY recorded_at ASC`.

### Tick computation sequence

For each account, per tick:

1. **Read new events** — Pull un-consumed interaction events from the event log.
2. **Compute presence-time delta** — From presence-ping events, compute how many minutes of presence occurred since last tick. Weight by presence recency (presence in the last tick window counts more than presence from hours ago — this is the low-pass filter's time-decay function).
3. **Update personality vectors** — For each bird:
   - Compute drift delta from presence-time (dominant input), weighted by per-bird factors (listen-in events on this bird increase weight).
   - Apply offer events: seed → +curiosity (small); song fragment → +vocal_frequency (small); still pool → no direct trait effect but marks bird for mood transition toward content.
   - Apply deltas additively. Traits only move upward (monotonic toward expressive). Neglect produces no downward movement.
   - Clamp all traits to [0.0, 1.0]. New birds seed at 0.3–0.5 (species-dependent).
4. **Transition moods** — For each bird:
   - Check time-of-day signal (user's local timezone, derived from account settings or IP geo): early morning → bias toward alert; midday → bias toward content; dusk → bias toward drowsy; night → bias toward sleep/settled (except nightjar species).
   - Check ambient weather: rain → dampen vocal frequency, bias toward content/drowsy; wind → bias toward alert/wary.
   - Check recent interactions: offer accepted → push toward content; listen-in recently → push toward curious; settle event → push toward drowsy/settled.
   - Apply personality filter: high-boldness birds resist wary transitions; high-curiosity birds tend toward curious more often.
   - Persist new mood. Mood does not reset to neutral.
5. **Assign perches** — For each bird:
   - Boldness-weighted probabilistic perch assignment: high boldness → higher probability of front perch; low boldness → higher probability of back perch.
   - Mood modifier: wary birds pull back one zone; curious birds may step forward.
   - Perch assignment is not persisted as a separate field; it's derived from mood + personality on each tick.
6. **Generate call-timing events** — For each bird:
   - Vocal frequency trait determines base call probability per tick.
   - Mood modifier: drowsy → fewer calls; alert → more calls; wary → fewer, shorter calls.
   - When a bird calls, assign a motif from its species library (procedural variation — pick motif + parameters: pitch bend, duration modifier, attack shape). Store the motif ID and parameters in the snapshot so the client's WebAudio engine can synthesize it identically.
   - Chorus detection: if two or more birds have calls in overlapping windows, mark the snapshot with a chorus event (affects mix levels on the client).
   - Bird-to-bird call response: if Bird A calls within a response window of Bird B's call, bias Bird B's next call probability upward (modulated by social warmth).
7. **Generate notebook entries** — Sparsity-gated:
   - Run a notebook-generation function that evaluates "noteworthy" conditions: first-time events (first chorus of the week, first time X greeted before Y), unusual perch choices, mood changes across sessions, weather events.
   - Target: ~1 entry every 2–4 days for a regularly-visited aviary. Cap: never more than 1 entry per tick, never more than 1 entry per calendar day.
   - Prose is assembled from templates keyed to the event, with species-specific and personality-specific parameter slots. E.g., template `"{bird_name} greeted before {other_bird_name} today, first time this week."` filled with actual names.
   - Notebook entries are written once and are immutable.
8. **Write canonical state** — Upsert bird records, insert notebook entries, mark consumed events as `consumed_by_tick = true`.
9. **Advance weather** — Small probability (~2–3%) of a weather event starting in a given tick for an account that doesn't already have active weather. Weather durations: rain 3–8 ticks; wind 2–5 ticks. Weather transitions are soft (ramp intensity up and down across ticks).

### Drift function calibration

The drift function is a time-decayed accumulator. For each trait:

```
delta_trait = alpha * presence_minutes * trait_weight * decay_factor
```

Where:
- `alpha`: global scaling constant (~0.001–0.005, calibrated during build to hit the "visible drift at 3 weeks" target).
- `presence_minutes`: minutes of valid presence in this tick window.
- `trait_weight`: per-trait sensitivity. Social warmth and curiosity are more presence-sensitive; plumage saturation is the slowest-moving trait.
- `decay_factor`: exponential decay applied to presence-minutes based on recency. Presence from the current tick gets weight 1.0; presence from 1 hour ago gets less; presence from 24 hours ago gets near-zero weight. This prevents overnight-tab-open from accumulating drift.

Calibration target (tested in CI): After 7 days of simulated 20-minute daily presence sessions, instruments detect measurable drift on all 5 traits. After 21 days, the drift is perceptible in render output (pose selection, perch zone distribution, plumage saturation value).

### Call grammar implementation

Each species has a motif library of 8–15 base motifs (short tonal patterns: "two-note rise," "low trill," "single sharp," "three-note descending," etc.). At call time, the engine selects a motif and applies procedural variation:

- **Pitch**: base pitch ± random micro-variation (±2–5 semitones, Gaussian).
- **Duration**: base duration × personality modifier (vocal frequency trait scales duration).
- **Attack/decay envelope**: mood-shaped (alert = sharp attack; drowsy = slow attack).
- **Timbre**: species-deterministic waveform (warbler = filtered saw; nightjar = filtered noise burst; etc.).

The variation parameters are recorded in the snapshot so the client can reproduce the exact call. The client's WebAudio worklet reads motif ID + parameters and synthesizes inline.

---

## 6. Sync Model

### Canonical state principle

The server-side simulation engine is the **sole writer** of personality vectors, moods, and all derived bird state. There is exactly one canonical state per account. The client is a state consumer + interaction producer, never a state author.

This makes multi-device sync a property of the architecture rather than a feature — no sync protocol, no merge conflicts, no CRDTs.

### How two devices see the same aviary

1. User signs in on laptop → client pulls snapshot, begins rendering.
2. User signs in on phone → client pulls snapshot, begins rendering.
3. Both clients pull snapshots from the same canonical record. Both see the same birds, same moods, same drift.
4. If the laptop is open and active, its presence pings and interaction events flow to the event log. The next server tick consumes them, updates state, and the phone's next snapshot pull (on visibility change or keepalive) reflects the updated state.

### Conflict prevention

- The client never sends personality values. It sends "user listened in to Pip for 3 minutes." The server decides what that means for boldness.
- The client never sends mood values. It sends "user offered a seed." The server decides what that means for mood.
- Interaction events are timestamped server-side on receipt (not client-side), preventing clock-skew ordering issues.
- If a client submits a stale snapshot ID in an event batch (optional optimization), the server ignores it — events are always appended regardless of what snapshot the client thinks it's on.
- No last-write-wins because there are no writes to win over. All mutations are additive deltas applied by a single writer in a single time order.

### Session timeout and reconnection

If a session token expires or the server rejects a request, the client displays the matter-of-fact surface: "Your session timed out. Sign in again to keep watching." No naturalist prose. The client does not attempt to silently reauthenticate.

### Tab lifecycle handling

- **Tab hidden** (`visibilityState = "hidden"`): Client stops rendering, stops sending presence pings, stops pulling snapshots. Audio stops (browser policy). The simulation continues server-side.
- **Tab visible again**: Client pulls a fresh snapshot. Birds have continued in the interim, mood has progressed, drift has ticked. The client interpolates from the last rendered state to the new snapshot (not a hard cut, no "waking up" animation — birds are mid-action in the new snapshot).
- **Tab closed**: Presence ends. No penalty. No settle-required. The simulation continues server-side.
- **Device suspended / laptop lid closed**: Same as tab hidden — rendering stops, simulation continues.

---

## 7. Frontend Rendering Pipeline

### Framework choice

React (or Preact for smaller bundle) with a custom Canvas/SVG hybrid renderer. The aviary scene itself is rendered on a `<canvas>` element for performance; the top bar and overlay surfaces (notebook, settings, visit flow) are standard React DOM. This split keeps the 60fps idle-motion budget hit from DOM reconciliation cost while preserving DOM accessibility for UI chrome.

### Scene composition layer

1. **Background pass**: Sky gradient (time-of-day-driven), background foliage, distant perches. Rendered once and composited; updated only on day-phase transitions (cross-faded over several seconds).
2. **Perch pass**: Three perch zones drawn as subtle branches/rails. Static geometry, single draw.
3. **Ambient pass**: Leaves and feathers drifting on slow paths. Client-side random generation; no server state. 2–6 particles at any time, recycled on drift-off-screen.
4. **Bird pass**: Per-bird rendering. The heavy pass.

### Bird rendering

Each bird is an SVG sprite sheet or a small set of body-part canvases composed at runtime:
- **Body**: species-deterministic silhouette, plumage filled with species base color modulated by `plumage_saturation` (higher saturation = richer/fuller color; lower = more muted).
- **Head/beak/eye**: composed separately for pose variation (head tilt, eye direction, beak open/closed for calls).
- **Wings/tail**: position per pose.

Pose state machine per bird:
- **Idle poses**: preening (sequence of 3–5 still frames, cross-faded in reduced-motion), scanning (slow head pan left/right, slight body sway), weight-shift (small vertical bob, occasional wing-stretch), head-tilt (toward sounds — "listening" to another bird's call), fluffed (low on perch, feathers puffed — drowsy mood).
- **Transition poses**: fly-in/fly-out between perches (arc path, wings animated), step-forward/step-back on perch.
- **Interactive poses**: greet (head cock, step forward, call animation — varies by boldness and absence-length), approach-offer (small hop toward front, head down investigating), settle (slow settle into perch, eyes closing).

Pose selection is driven by mood + personality: a bold/content bird preens more; a wary bird scans more; a drowsy bird fluffs and settles; a curious bird tilts frequently.

### Loading and first frame

**No spinner. No fade-from-static.** The loading sequence:

1. HTML shell delivers inline-critical CSS + a minimal JS bootstrap (~15KB) that paints the "quiet field" — a soft sky-gradient background with 1–2 faint ambient motion cues (a single drifting leaf).
2. Bootstrap requests `/aviary/{accountId}/snapshot` and the main JS bundle in parallel.
3. On snapshot arrival, birds are placed at their current positions with their current poses and rendered immediately — mid-motion, exactly as the snapshot describes. The first bird is visible within 500ms.
4. If the snapshot arrives before the main bundle, the bootstrap renders birds with placeholder silhouettes (species-deterministic but simplified); the full rendering pipeline replaces them seamlessly once loaded.
5. If the main bundle arrives first, it blocks on the snapshot — no rendering without state. The "quiet field" continues until data arrives.

The key property: the first frame the user sees with birds in it has birds already in motion. There is never an "entry animation" or a "birds appearing" transition after the initial adoption fly-in.

### Reduced-motion rendering mode

Detected via `window.matchMedia('(prefers-reduced-motion: reduce)')` or manual toggle. In this mode:
- All continuous animations (preening, scanning, weight-shift, ambient particle drift) are replaced with slow cross-fades (~800ms–1.5s) between still pose frames.
- Perch transitions (fly-in/fly-out) become cross-fades rather than arc-path animations.
- Ambient leaf/feather drift is disabled entirely.
- Day/night palette transitions are slowed (original transition duration × 3).
- Audio (calls) is unaffected — reduced-motion is visual only.

### Top bar

- Sits above the canvas, DOM-rendered, absolute-positioned.
- Icons: account/settings (gear), accessibility settings (universal-access icon), notebook (small book icon), offer (small seed/song/pool icon — opens a dropdown).
- Fades to near-transparent (opacity ~0.1) after 4 seconds of cursor stillness. Restores to full opacity on `pointermove` or `keydown`. Transition duration: 1 second.
- All icons are keyboard-focusable.

### Responsive layout

The canvas fills the viewport. Aspect ratio is preserved by scaling the scene to fit — letterboxing (soft background color bars) on mismatched aspect ratios rather than cropping. All birds always visible. Minimum viewport: 320×480 (small phone). Perch spacing compresses proportionally on narrow viewports.

---

## 8. Audio Pipeline

### Architecture

The audio pipeline is a WebAudio `AudioWorklet` running on a dedicated `AudioContext`. It runs client-side only — no audio streaming from the server.

### Motif library

Each of the 6 species ships with 8–15 base motifs encoded as parameter arrays (not audio files):
- Waveform type (sine, saw, triangle, filtered noise)
- Frequency envelope (array of {time, freq} points)
- Gain envelope (array of {time, gain} points)
- Filter envelope (low-pass cutoff over time, if used)

Total motif data: ~3–5KB per species, ~30KB total. Well within the 2MB bundle budget.

### Synthesis per call

When the client snapshot includes an `upcoming_call_at` timestamp with a motif ID and parameters, the audio engine:

1. Creates an `OscillatorNode` (or `AudioBufferSourceNode` for noise-based calls) with the base waveform.
2. Applies the frequency envelope via `setValueCurveAtTime` or scheduled `frequency.setValueAtTime` calls.
3. Applies the gain envelope for attack/decay shaping.
4. Applies filter envelope if specified.
5. Connects through a per-bird gain node (for mix control) → master gain → destination.
6. Schedules node start at the precise `AudioContext.currentTime` corresponding to the snapshot timestamp.
7. Auto-disconnects and releases nodes after call duration + tail.

Node pooling: `OscillatorNode` and `GainNode` instances are pre-allocated per bird and reused. No per-call allocation that isn't freed.

### Mix and listen-in

The audio graph:

```
Bird1_Osc → Bird1_Gain → Master_Gain → Destination
Bird2_Osc → Bird2_Gain ↗
...
```

Master gain is 1.0 (unity). Per-bird gain is computed from:
- **Ambient mode**: All birds at 0.5–0.7 gain (varies by perch zone — back perch slightly quieter).
- **Listen-in mode**: Focused bird ramps to 0.9 gain over 600ms; all other birds ramp to 0.15–0.2 gain over the same 600ms. Ramp uses `gain.linearRampToValueAtTime` for smooth transition.
- **Disengage listen-in**: All birds ramp back to ambient levels over 600ms.

Birds never go fully silent. The minimum gain for non-focused birds in listen-in mode is 0.1 — they remain audible as a quiet ambient presence.

### Chorus handling

When the snapshot contains a `chorus_event` flag, the audio engine applies a subtle global reverb bump (+10% wet mix on a `ConvolverNode` or simple delay-based reverb) for the duration of the chorus window. This is a gentle mix effect, not a dramatic shift — it makes overlapping calls feel like they're in the same space.

### Timing and scheduling

The snapshot provides call times as absolute timestamps. The client converts to `AudioContext.currentTime` offset and schedules ahead. The look-ahead window is 2 seconds — the audio engine schedules calls up to 2 seconds into the future and refreshes scheduling on each snapshot pull.

### WebAudio fallback

If `AudioContext` is unavailable (blocked by browser policy, older browser, hardware failure), the client enters **graceful silence with captions on by default**. The caption toggle in accessibility settings is forced on (but the user can still toggle it off — the toggle just starts on). No recorded audio fallback exists. No `<audio>` elements exist. Silence with captions is the designed fallback.

---

## 9. Accessibility Surfaces

### Screen-reader narration

- Generated client-side from the state snapshot, in the same naturalist prose voice as the field notebook.
- Narration assembly: a template system keyed to snapshot fields. E.g., `"{bird_name}, a {species_desc} {mood_desc}, perches on the {perch_zone} perch. {action_desc}."` filled from snapshot data.
- Action descriptions: "calling softly," "preening," "scanning the aviary," "settled on the perch, feathers fluffed."
- Cadence: one narration update every 30–60 seconds at idle. Faster cadence on user-initiated events: return-greeting gets narrated within 2 seconds of snapshot arrival; offer reaction gets narrated within 2 seconds; settle gesture gets narrated immediately.
- Narration is exposed via an ARIA live region (`aria-live="polite"`) on a dedicated, visually-hidden element. The narration text is accumulated and flushed to the live region at the cadence interval.
- **Priority queue**: User-initiated events queue ahead of idle updates. If an idle update was about to fire and a user event occurs, the idle update is deferred (not dropped — it fires after the user event narration).

### Call captioning

- Opt-in via accessibility settings toggle (off by default, except when WebAudio is unavailable → on by default).
- Caption text is generated from the call's motif ID + parameters at synthesis time. Motif-to-caption mapping: "soft two-note rise," "low trill, paused, low trill again," "single sharp call from the {perch_zone} perch."
- Captions appear as small, low-opacity text near the calling bird's position, fading in over 200ms and fading out over 400ms after the call ends.
- Caption text passes WCAG AA contrast against the aviary background (light text with a dark semi-transparent backdrop).
- Captions use the naturalist voice: lowercase, present-tense, specific.

### Reduced-motion mode

- Detected automatically via `prefers-reduced-motion: reduce`. Toggle in accessibility settings for manual override (force on or force off).
- Implementation described in Section 7 (Frontend Rendering).
- The manual toggle persists in `localStorage` and is synced as an account-level accessibility preference (so it follows the user across devices).

### Keyboard navigation

Tab order:
1. Top bar items (left to right): account settings → accessibility settings → notebook → offer affordance.
2. Shift+Tab reverses.
3. Tab from the last top-bar item or at any point: pressing Tab enters the aviary scene, focusing the first bird (front perch, or boldest bird if multiple).
4. Arrow keys (Left/Right) move focus between birds in perch order (front → middle → back, left to right within zone).
5. Enter/Space on a focused bird: triggers listen-in.
6. Escape: exits listen-in if active; otherwise moves focus back to the top bar.
7. When the offer affordance is focused, Enter opens the offer dropdown; arrow keys navigate seed/song/pool; Enter selects; Escape closes.

Focus indicators: a soft 3px outline in a high-contrast color (yellow-ochre, not electric blue) with a 1px dark inner shadow for visibility against both bright and dim aviary states. The indicator animates in over 150ms.

### WCAG AA contrast

All user-copy text in the top bar, settings panels, account surfaces, error surfaces, captions, and any overlay text meets WCAG AA contrast (4.5:1 for normal text, 3:1 for large text). The aviary scene canvas itself contains no user-copy text except call captions (which meet contrast), so the canvas rendering is exempt from the text-contrast requirement — but color choices for bird visibility (plumage against background) are reviewed for low-vision usability.

### Screen-reader-only notebook

Notebook entries are presented as a scrollable list with each entry in an `<article>` element. The prose is read naturally by screen readers (no special markup beyond semantic HTML). New entries are announced via a polite live region when the notebook is open.

---

## 10. Performance Budgets and Observability

### Bundle budgets

| Artifact | Budget (gzipped) | Measurement |
| --- | --- | --- |
| Initial JS (critical path) | <2MB | CI build step: `gzip-size` on the entry chunk |
| HTML shell + inline CSS | <15KB | CI build step |
| Motif library (all species) | <50KB | CI build step |
| Bird sprite assets (all species, all poses) | <300KB | CI build step (SVGs gzip well) |
| Code-split chunks (settings, visit flow, notebook) | <500KB each | CI build step |

Code-splitting boundaries:
- **Critical**: aviary renderer, audio engine, snapshot client, presence/event writer.
- **Deferred**: account settings panel, accessibility settings panel, visit-invitation flow, notebook full-view (recent entries are in the snapshot; full history loads lazily).

### Runtime budgets

| Metric | Target | Measurement |
| --- | --- | --- |
| Time to first bird visible | <500ms | RUM: `performance.mark("first-bird-visible")` timestamp minus `navigationStart` |
| Idle-motion frame rate | 60fps sustained | RUM: `requestAnimationFrame` loop sampling, p99 frame time <16.67ms |
| Memory growth over 30 min | <5MB growth | CI synthetic test: heap snapshot at t=0 and t=30min, delta <5MB |
| Audio context latency | <10ms scheduling jitter | RUM: measure difference between scheduled `AudioContext.currentTime` and actual callback time |
| Simulation tick latency | p99 <5s | Server-side metric: tick duration per account, alarmed at p99 >5s |
| Snapshot payload size | <15KB uncompressed | Server-side metric per response |
| Snapshot delivery time | p99 <200ms | RUM + server-side metric |

### No memory growth enforcement

In CI, a synthetic test opens the aviary, runs a 30-minute session (simulated presence, periodic listen-in, offers), and compares heap snapshots at t=0, t=15min, t=30min. The test fails if heap size grows by more than 5MB or if any heap snapshot shows retained objects from disposed components (audio nodes, canvas contexts, animation frame callbacks).

Key implementation rules to achieve this:
- Audio nodes are pooled and reused, never created per-call.
- Canvas rendering reuses the same context; no context recreation.
- Notebook entries loaded via infinite scroll reuse DOM nodes (virtual scrolling).
- Snapshot data replaces in-place; no accumulating snapshot history on the client.
- `requestAnimationFrame` callbacks are canceled on component unmount / tab hide.

### Observability

#### Synthetic monitoring

A fleet of headless browsers (Playwright) running from 3 geographies (US East, EU West, APAC) on a 15-minute schedule:
- Navigate to aviary, measure time-to-first-bird.
- Run a 5-minute session (simulated presence, one listen-in, one offer), measure sustained frame rate.
- Measure memory at session end.
- Alert if any metric breaches budget for 2 consecutive runs.

#### Real User Monitoring (RUM)

Aggregate-only metrics, no per-account dimensions:
- Page load waterfall (Navigation Timing API).
- Custom marks: `first-bird-visible`, `snapshot-arrived`, `audio-context-ready`.
- `Long Tasks` API for >50ms tasks on the main thread.
- Audio context error counts.
- `requestAnimationFrame` frame-time samples (sampled, not every frame).

Privacy boundary (per `accounts_sync.md`): RUM never includes per-bird state, per-account interaction history, or any field that could reconstruct a user's relationship with their aviary. Telemetry pipelines never read from the simulation database. The simulation database is never read by the analytics warehouse.

#### Server-side metrics

- Simulation tick duration histogram (per account, p50/p95/p99).
- Event log consumption lag (time between event recording and tick consumption).
- Snapshot generation time.
- API Gateway request rate, error rate, latency histograms.
- Magic-link email delivery success rate.
- Account creation rate, session creation rate.

#### Alerting

| Alert | Threshold | Severity |
| --- | --- | --- |
| Simulation tick p99 latency > 5s | 2 consecutive 5-min windows | High |
| API Gateway error rate > 1% | 5-min window | High |
| Magic-link delivery failure rate > 5% | 15-min window | Medium |
| Time-to-first-bird p95 > 1s (RUM) | 30-min window | Medium |
| Client JS error rate spike | 2x baseline | Medium |

---

## 11. Rollout

### Staging and canary

1. **Internal alpha** (week 1–2 post-build): Team members create accounts, adopt birds, run multi-device. Validate drift calibration, sync correctness, audio pipeline on real devices (mid-tier phones, 5-year-old laptops, different browsers). No external users.
2. **Closed beta** (weeks 3–4): 50–100 invited users. One species pool, two starter birds, full feature set. Primary goal: drift calibration validation and presence-accounting accuracy in real-world conditions (tabs in background, laptop sleep, phone browser). No marketing. Feedback via email.
3. **Open beta** (weeks 5–8): Unlimited sign-ups. No feature gates. Full monitoring. Gradual bird-count unlocking based on aviary age (first users won't hit the 7-bird cap for months; the cap mechanism is live from day one but practically invisible).

### Bird-per-aviary ramp

The aviary-age-based bird offering is live from day one:
- Day 0: 2 starter birds.
- ~90 days: third bird becomes available.
- ~180 days: fourth bird.
- ~270 days: fifth bird.
- ~365 days: sixth bird.
- ~450+ days: seventh bird.

This is not a rollout toggle — it's the product mechanic working as designed. The ramp is built into the simulation engine from day one. Early beta users will see third-bird offers around week 12–13 of the beta period.

### Instrumentation from day one

- All RUM metrics (time-to-first-bird, frame times, audio errors).
- All server-side metrics (tick latency, event lag, snapshot timing).
- Synthetic monitoring from launch day.
- Presence-accounting accuracy: a debug-only metric (not in RUM) that logs the ratio of presence-pings to actual session duration for internal validation. This is temporary and removed after calibration is confirmed.
- Drift calibration monitoring: a server-side audit log of personality vector deltas per account (aggregated, anonymized — just histogram of delta magnitudes) to detect calibration drift in production (e.g., if all birds are saturating traits faster than expected).

### Launch checklist

- [ ] WCAG AA contrast audit complete on all user-copy surfaces.
- [ ] Screen-reader narration tested with NVDA, VoiceOver, and JAWS on a full session.
- [ ] Reduced-motion mode tested with `prefers-reduced-motion: reduce` across all three major rendering engines.
- [ ] Keyboard-navigation audit: every interactive surface reachable, every action executable.
- [ ] Call captioning tested at all 6 species × all moods.
- [ ] 30-minute memory test passing in CI.
- [ ] Time-to-first-bird <500ms verified on a mid-tier Moto G-class device over throttled 4G.
- [ ] Bundle size check passing in CI.
- [ ] Magic-link deliverability tested across major email providers (Gmail, Outlook, Yahoo, Apple).
- [ ] Account deletion flow tested end-to-end (soft delete, restore within 30 days, hard delete after 30 days).
- [ ] Visit flow tested end-to-end (invite, accept, view, revoke, expiration).
- [ ] Multi-device sync tested: laptop + phone, simultaneous sessions, tab-hide/tab-show, laptop suspend/resume.

---

## 12. Risks

### 1. Drift calibration misses the "three weeks visible" target

**Risk**: The drift function's `alpha` is too high (birds change visibly between sessions — feels like a Tamagotchi) or too low (no perceptible change after months — feels like a screensaver). The calibration depends on real-world presence patterns that synthetic tests can only approximate.

**Mitigation**: Ship with a conservative (slow) `alpha` and a server-side feature flag to adjust it. Monitor drift-delta histograms during beta. If drift is too slow after 3 weeks of beta data, raise `alpha` in a config push (no client deploy needed — `alpha` lives on the simulation engine). If too fast, lower it. The monotonic-upward-only property means raising `alpha` later won't retroactively punish quiet birds; it just accelerates future drift.

### 2. Sync correctness edge cases

**Risk**: Event-ordering bugs where interaction events from different devices are consumed out of sequence, causing a bird to "miss" an interaction. Or a snapshot pull returns state that an inflight event batch hasn't yet been applied to, causing a minor visual inconsistency.

**Mitigation**: Event log is strictly ordered by server-receipt timestamp. Snapshot generation reads all un-consumed events first, applies them, then returns the snapshot — so a snapshot always reflects all events received before the snapshot request. The race window is: event arrives between snapshot-read and snapshot-return. This is acceptable (the event will be reflected in the next tick's snapshot, ~60 seconds later) and produces no visible inconsistency to the user (60 seconds is below the perceptible threshold for bird behavior changes).

### 3. Audio pipeline uncanniness

**Risk**: Procedural calls sound "synthy" or harsh, breaking the felt-aliveness. The WebAudio oscillator-based approach risks producing sounds that read as computer-generated rather than bird-like. Even if each call is good, the chorus might produce frequency clashes that sound artificial.

**Mitigation**: Extensive motif-library design with filtered waveforms (low-pass filtering is key to warming up oscillator output). Chorus mixing includes a subtle global reverb that smooths frequency overlap. Beta feedback specifically solicits audio quality impressions. A "call quality" RUM metric (did the audio context produce any errors? were there audible glitches?) is monitored. If WebAudio synthesis can't reach the quality bar, fallback to small pre-rendered audio buffers (~5–8 per species, procedurally combined and pitch-shifted at runtime to create variation) — this is an escape hatch that trades bundle budget for quality.

### 4. Accessibility regression during iteration

**Risk**: Accessibility surfaces (narration, reduced-motion, captions, keyboard nav) degrade as features are added or rendering is optimized. Screen-reader narration that sounded natural in v1 becomes stale when new bird behaviors are added but the narration templates aren't updated.

**Mitigation**: Narration templates are co-located with bird behavior definitions (same module, same PR review). Reduced-motion mode is tested in CI alongside the default render path — every visual change gets a reduced-motion variant test. Keyboard navigation is part of the acceptance criteria for every PR touching interactive surfaces. Accessibility audit is a launch gate, not a post-launch task.

### 5. Bundle size creep

**Risk**: Initial JS bundle grows past 2MB as features are added, codesplitting boundaries are violated by accidental imports, or dependencies bloat. This kills the time-to-first-bird budget, which kills the central conceit of the product.

**Mitigation**: Bundle size check in CI on every PR — fails the build if the critical-path chunk exceeds 2MB (gzipped). Bundle analysis tooling (webpack-bundle-analyzer or equivalent) runs in CI and flags new large dependencies. Code-splitting boundaries are enforced by directory structure (`critical/` vs `deferred/`) with an import lint rule that prevents `deferred/` imports from `critical/`.

### 6. Server-side simulation tick scaling

**Risk**: As the user base grows, ticking every account every 60 seconds becomes a scheduling problem. An account with no recent events still consumes a tick slot.

**Mitigation**: Tick scheduling is adaptive: accounts with an active client session or un-consumed events get priority; accounts dormant for >24 hours get ticked on a slower cadence (every 5 minutes) or on-demand (ticked when the client next requests a snapshot). The per-account tick cost for a dormant account is trivially cheap (no event log reads, just time-of-day mood transition check). The simulation engine is designed for horizontal scaling by account-ID shard — a consistent-hashing ring distributes accounts across engine instances.

### 7. Presence-accounting false positives in real-world conditions

**Risk**: The 3-condition presence check (visibility + focus + activity) may produce false positives (e.g., user leaves laptop open with aviary visible and a stuck key repeating) or false negatives (e.g., user watching without moving for longer than the activity timeout). Both corrupt the drift signal.

**Mitigation**: The activity timeout is calibrated during build with real-world testing — leaning longer (~5 minutes initially) to avoid false negatives for still-watching users. Stuck-key detection: require pointermove OR keypress, and keypresses are deduplicated (same key held down doesn't generate repeated presence pings). The debug presence-accuracy metric during beta will detect systematic false positives; if found, add a "human-presence heuristic" (e.g., require pointer activity at least once per presence window, not just keypresses).

### 8. Magic-link email deliverability

**Risk**: Magic links flagged as spam, delayed, or silently dropped by email providers. Users can't sign in. This is a hard failure — the entire product is gated on email.

**Mitigation**: Use a transactional email service (Postmark, SendGrid) with dedicated IP warm-up before launch. Monitor delivery rates per provider. Provide a "resend link" affordance with clear error messaging ("We couldn't sign you in. The link may have expired. Try requesting a new link."). If deliverability becomes a recurring issue, add a backup sign-in method (time-based one-time passcode via the same email flow, which doesn't rely on link-click deliverability but still doesn't require passwords).

### 9. Notebook prose quality at scale

**Risk**: Template-based prose generation produces repetitive, obviously machine-written entries that break the field-notebook illusion. The user notices "Pip greeted before Wren today, first time this week" appearing multiple times with only the bird names swapped.

**Mitigation**: Template library is large enough that no single template repeats within a 30-day window for the same account. Templates include species-specific and personality-specific variant branches. Notebook entries are sparse (1 per 2–4 days), which reduces repetition visibility. If prose quality degrades during beta, the notebook-generation function can be enhanced with more template variety or a lightweight language-model call (server-side, batched, not per-tick) — but this is an escalation, not the v1 plan.

### 10. "Feels alive" degrades under scale or optimization pressure

**Risk**: As the product matures, performance optimizations, refactors, or feature additions quietly remove the small signals that produce felt-aliveness — a micro-motion that gets dropped for frame-rate, a call variation that gets simplified to reduce CPU, a greeting stagger that gets collapsed to save code complexity. Each individual removal is defensible in isolation; the cumulative effect is the death of the product.

**Mitigation**: The design principles ("Feels alive, not robotic" and "Restraint over richness") are encoded as acceptance criteria, not as philosophy. A specific test suite — the "aliveness regression suite" — runs in CI and checks: does the return-greeting vary across 10 simulated returns? Is the greeting staggered when multiple birds would greet? Are calls procedurally varied across 50 consecutive calls from the same bird? Does idle motion run continuously for 5 minutes without repeating the exact same pose sequence? These tests are the concrete expression of the principles and they fail the build if violated.