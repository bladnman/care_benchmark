# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In v1

- Browser-only (no native apps)
- Single-user accounts, magic-link sign-in, synthetic UUID account identifiers
- One canonical aviary per account
- Two starter birds at account creation, capacity to grow to seven over aviary age
- Server-side simulation tick (~1 min cadence): personality drift, mood transitions, call timing
- Client renders state snapshots; never writes personality state
- Five interaction types: return-greeting, listen-in, offer (seed / song-fragment / still pool), settle, presence
- Field notebook (auto-generated naturalist prose, ~1 entry per few days)
- Day/night cycle anchored to user local timezone
- Ambient weather (light rain, soft wind) a few times per week
- Multi-device sync (architecture property, not a separate feature)
- Visit invitations: per-email opt-in, read-only ambient, revocable, off by default
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fade rendering, not a stripped fallback)
- Call captioning in naturalist voice
- Keyboard navigation throughout
- Account export (JSON, emailed on demand)
- Soft-then-hard account deletion (30-day window)
- Privacy: per-account interaction data never aggregated, never used for ML or third-party purposes

### Explicitly out of v1

- Native apps (iOS, Android)
- Gamification of any kind: streaks, achievements, scores, levels, badges, visit-frequency counters, green-dot calendars
- Tamagotchi mechanics: hunger, distress, happiness meters, negative drift on neglect
- Social network surfaces: profiles, follows, public feed, discovery, shared aviaries, visit comments
- Push notifications of any kind (no email, no push, no in-product pings unprompted by user action)
- Multiple aviaries per account
- Customizable scenes or palettes
- Password-based login or SSO
- Multi-user simulation or co-presence visits
- Recorded audio fallback

---

## 2. Architecture

### Service topology

```
Browser client
  ├── Rendering engine (Canvas/WebGL or SVG, TBD per §6 rendering spec)
  ├── WebAudio call synthesizer
  ├── Presence tracker (visibility + focus + activity)
  └── HTTP client (REST/JSON)

Edge CDN
  └── Serves initial HTML + JS bundle + state-snapshot cache

API Gateway
  ├── /auth            — magic link issue + verify, session tokens
  ├── /aviary          — snapshot reads, event writes
  ├── /notebook        — entry reads
  ├── /account         — settings, export, deletion, visit log
  └── /visit           — invite issue, invite verify, revocation

Simulation service (internal)
  ├── Tick scheduler (cron, ~1 min)
  ├── Drift engine
  ├── Mood transition engine
  └── Notebook entry generator

Database layer
  ├── Account store      — UUID → encrypted email, session tokens
  ├── Aviary store       — per-account: birds, personality vectors, moods, aviary state
  ├── Event log          — append-only: presence pings, offers, listen-in start/end, settle
  ├── Notebook store     — per-account: ordered prose entries
  └── Visit store        — invites, visit log
```

### Client/server split

- **Server owns**: personality vectors, mood state, canonical aviary positions, notebook entries, presence accumulation totals, visit records, simulation tick.
- **Client owns**: rendered frame, local presence detection, audio synthesis, interpolated motion between snapshots, UI state (listen-in focus, top-bar visibility).
- **Client never**: writes personality state, derives personality from event history, ticks the simulation.

### Render pipeline boundary

State snapshots from the server are the render input. The client renders a snapshot, interpolates until the next snapshot arrives, and re-renders. There is no client-side physics for bird positions that diverges from server-side positions over time; interpolation is always between known server states.

---

## 3. Data model

### Account

```json
{
  "account_id": "uuid-v4",
  "email_encrypted": "...",
  "created_at": "ISO8601",
  "deleted_at": null,
  "deletion_scheduled_at": null,
  "session_tokens": [{"token_id": "...", "issued_at": "...", "last_seen": "..."}],
  "settings": {
    "visit_notifications_enabled": false,
    "reduced_motion": false,
    "captions_enabled": false
  }
}
```

The account's email is stored once, encrypted, on this record. No other service receives the email; they receive the `account_id` UUID only.

### Bird

```json
{
  "bird_id": "uuid-v4",
  "account_id": "uuid-v4",
  "name": "Pip",
  "species": "warbler",
  "adopted_at": "ISO8601",
  "personality": {
    "boldness": 0.42,
    "social_warmth": 0.61,
    "vocal_frequency": 0.55,
    "plumage_saturation": 0.38,
    "curiosity": 0.50
  },
  "personality_version": 1047,
  "mood": "content",
  "mood_updated_at": "ISO8601",
  "last_perch": "front"
}
```

`bird_id` is stable forever. Renaming, species updates, sync, or migration never changes `bird_id`. `personality_version` is a monotone integer incremented by the simulation tick on each write; used to detect stale reads.

Personality trait range: `[0.0, 1.0]`. Seed values for new birds: species-keyed defaults with small random jitter (~±0.05).

### Mood states

`enum: wary | content | curious | drowsy | alert`

Stored per bird on the aviary record. Transitions are computed by the simulation tick (see §5).

### Presence event (event log entry)

```json
{
  "event_id": "uuid-v4",
  "account_id": "uuid-v4",
  "event_type": "presence_ping | offer | listen_in_start | listen_in_end | settle",
  "timestamp": "ISO8601",
  "payload": {}
}
```

`payload` contents by type:
- `presence_ping`: `{}` (emitted by the client every ~60s while presence conditions are met)
- `offer`: `{"offer_type": "seed|song_fragment|still_pool", "target_bird_id": "...", "accepted": true}`
- `listen_in_start`: `{"bird_id": "..."}`
- `listen_in_end`: `{"bird_id": "...", "duration_seconds": 142}`
- `settle`: `{}`

### Notebook entry

```json
{
  "entry_id": "uuid-v4",
  "account_id": "uuid-v4",
  "created_at": "ISO8601",
  "prose": "pip greeted before wren today, first time this week."
}
```

Entries are ordered by `created_at`. No upper-bound on count. Read-only once written.

### Aviary state snapshot

Returned by `GET /aviary/snapshot`. Not persisted as a separate table — computed from the current bird records.

```json
{
  "snapshot_at": "ISO8601",
  "local_time_of_day": "morning",
  "weather": null,
  "birds": [
    {
      "bird_id": "...",
      "name": "Pip",
      "species": "warbler",
      "mood": "content",
      "perch": "front",
      "motion_state": "preening",
      "plumage_saturation": 0.38,
      "call_motif_seed": 12345
    }
  ]
}
```

`call_motif_seed` is an integer derived from the bird's personality and current mood, giving the audio synthesizer a deterministic seed for motif selection and variation within a session.

### Visit

```json
{
  "invite_id": "uuid-v4",
  "host_account_id": "uuid-v4",
  "visitor_email_hash": "sha256-of-visitor-email",
  "created_at": "ISO8601",
  "expires_at": "ISO8601",
  "revoked_at": null,
  "used_at": null,
  "visit_log": [
    {"visited_at": "ISO8601", "duration_seconds": 340}
  ]
}
```

The visitor's email is stored hashed for visit log display purposes; the host sees the email they typed at invite creation stored plaintext on the invite record (also encrypted at rest). The `invite_id` is what's embedded in the one-time link.

---

## 4. API surface

All endpoints authenticated by session token (Bearer header). Magic-link and invite endpoints are unauthenticated.

### Auth

| Method | Path | Description |
|---|---|---|
| POST | `/auth/magic-link` | Issue magic link; email address in body |
| GET | `/auth/magic-link/verify?token=…` | Consume token, issue session, redirect |
| DELETE | `/auth/sessions/{session_id}` | Revoke a session |

Magic links expire after 15 minutes. Consuming a link invalidates it immediately.

### Aviary

| Method | Path | Description |
|---|---|---|
| GET | `/aviary/snapshot` | Current canonical state snapshot |
| POST | `/aviary/events` | Submit an interaction event (presence_ping, offer, listen_in_start, listen_in_end, settle) |

The snapshot endpoint returns the small JSON described in §3. It is cacheable at the CDN edge for ~10 seconds; cache-control headers allow the client to receive a fresh snapshot on visibility change via `Cache-Control: no-cache`.

`POST /aviary/events` is append-only. Body is a single event object (see §3 event log schema). Returns `201 Created` or `202 Accepted`. Clients may batch up to ~5 pending events in a single request (e.g., presence pings accumulated while offline) using an array body.

### Notebook

| Method | Path | Description |
|---|---|---|
| GET | `/notebook/entries?before={entry_id}&limit=20` | Paginated notebook entries, newest-first |

Cursor-based pagination. The client loads 20 entries on open; scrolling to the bottom fetches the next page.

### Account

| Method | Path | Description |
|---|---|---|
| GET | `/account` | Account settings and metadata |
| PATCH | `/account/settings` | Update settings (reduced_motion, captions_enabled, visit_notifications_enabled, etc.) |
| POST | `/account/export` | Trigger export; email sent asynchronously |
| DELETE | `/account` | Initiate soft deletion |
| POST | `/account/recover` | Cancel pending deletion during 30-day window |

### Visit / social

| Method | Path | Description |
|---|---|---|
| POST | `/visit/invites` | Issue invite (host auth required); body: `{visitor_email}` |
| DELETE | `/visit/invites/{invite_id}` | Revoke invite |
| GET | `/visit/invites` | List outstanding invites and visit log |
| GET | `/visit/{invite_id}/snapshot` | Visitor reads host aviary snapshot (no auth, invite_id is the token) |

The visitor snapshot endpoint checks invite validity (not revoked, not expired) on every call. If the invite has been revoked since the visitor's session started, the endpoint returns `403` with a matter-of-fact body.

Visitor session: read-only. The visitor's browser can call `/visit/{invite_id}/snapshot` on a keepalive cadence to keep the view fresh. No event writes.

---

## 5. Simulation engine design

### Tick scheduler

A background job scheduler (e.g., cron via a job queue such as BullMQ, Sidekiq, or cloud-native equivalent) fires the simulation tick for each active account approximately once per minute. "Active" is defined as: an account whose aviary has had any event in the past 48 hours, OR whose last personality_version update was less than 7 days ago (to keep mood advancing even for users who haven't visited recently). Fully inactive accounts (no events, old tick) reduce to once-per-hour ticks for day/night mood transitions; once-per-day for dormant accounts.

Each tick is isolated per account. The tick reads:
1. The bird records (personality vectors, current moods, current perch positions)
2. The interaction event log entries since the last tick timestamp
3. The current server time (for time-of-day mood transitions)
4. Recent ambient weather state (generated independently at low frequency)

Then writes:
1. Updated personality vectors (drift deltas applied)
2. Updated moods (transition state machine)
3. Updated perch positions (derived from mood + personality)
4. New notebook entries (if any)
5. Updated `last_ticked_at` on the account

The tick must be idempotent for a given tick window. If the job fires twice for the same minute, the second run detects no new events and produces no drift delta (double-processing the same events is guarded by tracking the watermark position in the event log).

### Drift engine

Drift is implemented as a weighted low-pass filter over accumulated presence-time and interaction signals.

**Presence accumulation**: Each `presence_ping` event represents ~60 seconds of confirmed presence (client emits one ping per minute while conditions hold). The tick sums presence pings since the last tick to get `presence_seconds` for this tick window.

**Drift delta computation** (per bird, per tick):

```
delta(trait) = learning_rate × (
    presence_weight    × presence_seconds / normalization_constant
  + listen_in_weight   × listen_in_seconds_on_this_bird
  + offer_accepted_w   × offer_accepted_count (curiosity, boldness only)
  + offer_proximity_w  × offer_issued_near_bird (boldness only)
)
```

Weights (initial calibration; to be validated against the 1-week-instruments / 3-week-user-visible targets):

| Signal | Traits affected | Weight relative to presence |
|---|---|---|
| presence_seconds | all traits (weakly) | 1.0 (baseline) |
| listen_in_seconds | social_warmth, vocal_frequency | 3× |
| offer_accepted | curiosity, boldness | 2× per event |
| offer_proximity | boldness | 1× per event |

`learning_rate` is a small constant (~0.0002 per normalized unit per tick) calibrated so a trait moves ~0.03 over one week of daily 20-minute sessions. This satisfies: instruments-detectable after one week, user-perceptible after three weeks.

Drift is bounded at `[trait_value, 1.0]` — traits never decrease. If a trait is already at 1.0, no further drift is applied (saturated state).

`plumage_saturation` drifts identically to the general presence signal; it is the visual expression of accumulated attention.

**Calibration test harness**: A unit test fixture runs a simulated 21-day scenario with a daily 20-minute presence session and asserts that all five traits show measurable change after 7 days and that `plumage_saturation` passes a visual-perceptibility threshold (defined as ≥0.05 change from seed value) after 21 days. This test must pass in CI.

### Mood transition state machine

Mood is a per-bird enum (`wary | content | curious | drowsy | alert`). Each tick evaluates the following inputs:

- **Time-of-day signal** (derived from last-known user timezone, stored on account):
  - early morning (5–8 AM): pushes toward `alert`
  - midday: pushes toward `content` or `curious`
  - late afternoon: neutral
  - dusk (6–9 PM): pushes toward `drowsy`
  - night (9 PM–5 AM): strong push toward `drowsy`; nightjar species resists this

- **Recent interaction signals** (from events since last tick):
  - successful offer acceptance → push toward `content` or `curious`
  - settle gesture → push toward `drowsy`
  - rapid repeated offers (cooldown check) → no additional signal

- **Ambient weather**:
  - rain event: dampens vocal_frequency for this tick, slight push toward `wary` for high-wary-personality birds
  - wind event: push toward `alert` for high-curiosity birds, `wary` for low-boldness birds

- **Bird-to-bird contagion**: if another bird in the aviary transitioned to `wary` in this tick, nearby birds (low boldness) have a 30% chance of also shifting toward `wary`

- **Personality modulation**: a high-boldness bird requires a larger wary-push signal to enter `wary`; a high-curiosity bird is more likely to transition to `curious` on any offer event.

Transitions are probabilistic, not deterministic. A transition table defines probabilities given each combination of current state + dominant input signal. This table is an implementation artifact (JSON config file) that can be tuned without code changes.

Mood is written back to the bird record. Mood persists across sessions; it does not reset on tab open.

### Call-grammar runtime

Call grammar is not executed server-side. The server provides `call_motif_seed` in the snapshot, derived from `(bird_id, mood, personality_version) → deterministic hash`. The client audio synthesizer uses this seed to select motifs and variation parameters.

Call timing is server-guided: the snapshot includes the bird's expected next-call interval in seconds (derived from vocal_frequency). The client schedules calls accordingly and adds ±15% jitter.

### Notebook entry generation

The notebook generator runs as part of the simulation tick but only fires when there is a noteworthy observation. Conditions that trigger an entry (examples; implementation expands this list):

- A bird greeted first today when it usually doesn't (greeted_first_count change)
- A long period of quiet in the aviary (no calls in the event log for >20 minutes of a presence window)
- An offer was accepted by a bird that has historically been slow to approach
- A bird entered alert mood during a rain event

The generator uses a small template library with slot-fill from concrete bird names, moods, and events. Templates are naturalist prose, lowercase, present-tense. Template selection randomizes within the applicable category. The raw event data for the observation is passed as context; the generator never synthesizes data that didn't happen.

Entry rate is throttled: at most one entry every ~48 hours per aviary, with the exception that the system may generate an entry when something genuinely unusual happens (first-of-its-kind event for this aviary). A recently-active aviary does not get a daily entry by default.

---

## 6. Sync model

Multi-device sync is not a protocol; it is a property of the architecture:

- The server is the sole writer of personality state (via simulation tick only)
- All clients read from the same canonical aviary record
- Clients write events to an append-only log; events are never overwritten
- The simulation tick reads events in log order; its output is deterministic given the same inputs
- No client-to-client communication

**Conflict prevention** (not resolution, because conflicts cannot arise):

- A client submitting an event when another client submitted a conflicting event: impossible, because event submission is append-only and the tick consumes the entire log. Two presence pings from two devices in the same minute are both recorded; both contribute to drift.
- A client reading a stale snapshot: the snapshot includes `snapshot_at`. The client uses this to determine whether to re-poll. On visibility change (tab becomes visible), the client always re-polls regardless of age.
- Session token conflicts: each device has its own session token. Both are valid concurrently. There is no concept of "the active session"; the server serves any valid session.

**Edge case**: a user is signed in on two devices simultaneously. Both devices submit presence pings. Both sets of pings land in the event log. The tick processes them and accumulates the higher of the two presence counts (not both — the tick deduplicates overlapping presence windows by timestamp to avoid double-counting simultaneous presence). Deduplication logic: the tick groups presence pings by 1-minute windows; if N pings arrive within the same window, they count as 1 minute of presence.

---

## 7. Frontend rendering pipeline

### Rendering technology choice

The aviary scene is rendered via an HTML Canvas element using a 2D rendering context (falling back to WebGL for complex particle effects if perf budget allows). SVG is used for bird silhouettes (compact, scalable, personality-tinted via CSS filter). The scene is composed of:

- **Background layer**: static sky gradient (CSS, transitions via CSS custom properties for day/night)
- **Mid-far layer**: foliage, branches (SVG or bitmap sprites)
- **Perch layer**: three perch zones (front, middle, back), each a fixed position in the composition
- **Bird layer**: each bird is an SVG group positioned at its perch zone, with transform-based animation for micro-motion
- **Foreground particle layer**: leaf/feather drift (Canvas 2D, requestAnimationFrame, client-generated, no server state)
- **Ambient weather layer**: rain (Canvas particle system), wind (CSS ripple on leaves)

### Scene composition on load

1. Client requests snapshot from CDN-cached endpoint (served with initial HTML as inline JSON to eliminate a round-trip)
2. Client places birds at their snapshot positions in their snapshot motion states
3. Rendering begins immediately — first frame is mid-motion (bird is mid-preen, etc.)
4. Particle layer starts independently (leaves begin drifting from frame 1)

If the inline snapshot is unavailable (cold cache miss), the loading state is a soft sky gradient with gentle color shift — not a spinner. The client polls for the snapshot while this quiet field is displayed. The word "loading" never appears.

### Idle micro-motion

Each bird has a motion state machine (client-side, not simulation-driven):

- `preening`: series of head-bow and wing-ruffle transforms, looped with random timing variation
- `scanning`: slow head-pan left and right, small body sway
- `calling`: beak-open/close synchronized with audio synthesis events
- `perching_idle`: minimal weight-shift, subtle feather settle
- `approaching` (front-perch transition): short hop/glide from back to front perch

Motion parameters are keyed by mood:
- `wary`: tighter body posture, faster scan rate, perched further back
- `content`: slow preening, relaxed posture
- `curious`: frequent head-tilts toward sounds, longer beak-opens
- `drowsy`: low-slung posture, minimal motion, beak occasionally drops
- `alert`: upright, frequent scanning, wings slightly extended

Mood is read from the snapshot; the client applies mood-keyed motion parameters immediately on snapshot receipt.

### Personality-driven visual tinting

`plumage_saturation` maps to a CSS filter (`saturate()`) applied to the bird's SVG group. Range: 0.3 (desaturated, new bird) to 1.0 (richly colored, long-attended bird). This is the only visual surface where personality directly affects appearance.

### Day/night cycle (client-side)

The client reads the user's local time and applies a continuous gradient to the sky background and a warm-to-cool color filter on the scene. Implementation: CSS custom properties driven by a `requestAnimationFrame` loop that computes `hour + minute/60` and maps to a color table. The table has smooth easing at sunrise (5–7 AM) and sunset (6–8 PM). The server provides the `local_time_of_day` label in the snapshot for accessibility narration; the client computes the precise color itself.

### Transitions

All bird position transitions (perch changes) use CSS transitions or Web Animations API with duration ~1.5–2s and an ease-in-out curve. No teleporting. Cross-perch moves use a short arc path (transform keyframe) rather than a straight line.

### Reduced-motion mode

Detect `prefers-reduced-motion: reduce` via `matchMedia`; also check the user's account settings `reduced_motion: true`.

In reduced-motion mode:
- All CSS `animation` and `transition` durations are set to 0
- A cross-fade layer is placed above the bird SVGs; bird pose changes are achieved by dissolving between static SVG frames (each pose is a separate SVG asset, pre-built for each mood × species combination)
- Ambient leaf drift is removed (particle loop not started)
- Ambient weather: color shift (day/night) remains; rain particle system is removed
- Listen-in audio mix change: still gradual (no visual change needed)

The cross-fade rendering requires a pose-asset library: for each species × mood combination, 3–4 static SVG pose frames covering the key postures. This is a design deliverable. The rendering code dissolves between frames on the same 30–60s cadence that would drive animation transitions in the full-motion mode.

### Top-bar fade

A CSS opacity transition on the top-bar element, driven by an idle timer. Timer resets on `pointermove`, `keydown`, `touchstart`. After 3 seconds of cursor stillness, top bar transitions to `opacity: 0.15` over 1s. On cursor movement, returns to `opacity: 1.0` over 0.3s.

---

## 8. Audio pipeline

### Architecture

All call synthesis occurs client-side in the WebAudio API (`AudioContext`). No audio files are downloaded. The bundle carries a compact motif library (sequences of frequency ratios, rhythm templates, pitch contours) for each species — this is a few KB of JSON, not audio assets.

### Call synthesis

Each species has a call grammar with:
- **Motif library**: 8–12 base motifs (short note sequences defined as pitch ratio arrays)
- **Variation parameters**: timing jitter range, pitch transpose range, repetition probability
- **Rhythm template**: the characteristic time pattern of the species' call

At runtime, the audio synthesizer:
1. Selects a motif using the `call_motif_seed` + a time-based entropy source (so successive calls within a session aren't identical)
2. Applies mood-shaped variations: drowsy → slower tempo, lower pitch; alert → faster tempo, higher pitch; wary → shorter calls, longer silences
3. Applies vocal_frequency scaling to inter-call intervals
4. Synthesizes via WebAudio oscillators (sine + small noise component for texture) and a custom envelope (attack, sustain, decay per note)

The synthesis produces a distinct, recognizable call per species while varying every instance. The user should learn to recognize Pip's call across mood states and drift states.

### Chorus mixing

Two or more birds calling simultaneously are independent audio streams mixed in the `AudioContext`. Each bird has its own `GainNode`. The default mix level for each bird is derived from the bird's vocal_frequency trait and its current mood.

No phase cancellation mitigation is needed (procedural synthesis avoids the artifact that stacked loops produce).

### Listen-in mix decay

When the user triggers listen-in on a bird:
1. The focused bird's gain ramps to ~1.0 over 2 seconds (slow rise)
2. All other birds' gains ramp to ~0.15 over 2 seconds (quiet but not silent)
3. On disengage (click elsewhere, Escape, focus change): both ramps reverse over 2 seconds

Implementation: `GainNode.gain.linearRampToValueAtTime()` with the 2-second target time.

### Call captioning

When a call is synthesized, the synthesizer emits an event to a caption renderer. The caption renderer maps the call's motif + mood to a naturalist prose description:
- Motif 1 in `drowsy` mood → "a slow, low two-note murmur"
- Motif 3 in `alert` mood → "a sharp ascending trill"

The caption library is a small JSON map from `(species, motif_id, mood)` to prose string, with 2–3 variants per combination. The renderer selects a variant randomly. Caption text appears as a small fade-in/fade-out label positioned near the calling bird's perch zone; visible for ~2s then fades.

### WebAudio fallback

If `new AudioContext()` throws, or if the audio context fails to resume after a user gesture, the client enters graceful silence:
- No audio plays
- Call captions are enabled by default (not requiring the user to opt in)
- A small matter-of-fact notification appears once in the top bar: "Audio isn't available. Captions are on." (fades after 5 seconds)
- The rest of the product is unaffected

No recorded audio fallback path exists. Silence + captions is the correct fallback.

---

## 9. Accessibility surfaces

### Screen-reader narration

A visually-hidden ARIA live region (`aria-live="polite"`, `aria-atomic="false"`) is updated on a slow cadence:
- **Idle cadence**: one prose update every 30–60 seconds, drawn from current snapshot state
- **Event cadence**: promptly on return-greeting, offer reaction, settle gesture

Prose is generated client-side from the current snapshot using a template library matching the field notebook voice. Example template:

> `{bird_name} is on the {perch_position} perch, {motion_state_description}. {second_bird_name} {optional_second_observation}.`

Concrete output:
> "pip is on the front perch, preening slowly. wren is further back, watching."

The live region is never updated more than once per 10 seconds (debounced), so rapid state changes don't flood the screen reader's queue.

On user-initiated events, the region receives priority updates via `aria-live="assertive"` temporarily switched on an inner element, then reverted.

### Keyboard navigation

Full keyboard flow:
- `Tab` cycles through top-bar icons (notebook, offer, settle, accessibility settings, account)
- From top bar, `Tab` enters the aviary scene
- Inside the aviary, `Tab` / `Shift+Tab` cycle through birds by perch order (front to back, left to right within perch)
- `Enter` on a focused bird: triggers listen-in
- `Escape`: exits listen-in, returns focus to aviary
- `Enter` on top-bar offer: opens offer panel (keyboard-navigable: arrow keys choose offer type, `Enter` submits)
- `Enter` on top-bar settle: triggers settle gesture; `Esc` or any key within 5 seconds undoes it

Focus ring style: a 2px soft-glow outline in a high-contrast color (verified against both light and dark aviary states; exact color specified by design system).

### WCAG AA contrast

- All top-bar icon labels: minimum 4.5:1 against the top-bar background
- Caption text: minimum 4.5:1 against the aviary background at both morning and evening lighting states
- Notebook prose: minimum 4.5:1 (notebook panel has a solid background overlay)
- Error and settings surfaces: minimum 4.5:1

The aviary scene background changes with day/night; caption text adapts via a background-color behind the caption element that ensures the contrast ratio is met regardless of scene lighting.

### Bird focus indicators

When a bird is keyboard-focused, a soft outline appears around the bird's SVG bounding box. The outline uses a fixed high-contrast color that reads against all aviary backgrounds. In reduced-motion mode, the outline does not animate; it appears and disappears without transition.

---

## 10. Performance budgets and observability

### Budgets

| Metric | Target | Enforcement |
|---|---|---|
| Initial JS bundle (gzipped) | < 2 MB | CI bundle-size check (e.g., bundlesize or size-limit) |
| Time to first bird visible | < 500ms on mid-tier 4G | Synthetic Lighthouse CI check |
| Idle motion frame rate | 60fps for 30 min on 5-year-old mid-range laptop | Manual QA + synthetic browser check |
| Memory (30-min session) | No growth trend | `performance.measureUserAgentSpecificMemory()` check in CI E2E |
| Simulation tick p99 latency | < 5s (alert), < 1s (target) | Server-side metric alarm |

**Bundle budget strategy**:
- Code-split account settings, accessibility settings, visit flow, notebook view into separate chunks (loaded on first navigation to those surfaces)
- Bird SVG assets: compact inline SVG, < 5 KB per species, generated at build time
- Motif library (audio): < 20 KB total JSON
- No audio assets
- No large image assets

**Time-to-first-bird strategy**:
- State snapshot inlined as JSON in the HTML `<head>` (generated at SSR time, served with the initial HTML)
- First render begins from the inline snapshot without a network round-trip
- Bird SVG rendered inline in HTML (no image load needed for first bird)
- WebAudio context creation deferred to after first paint

### Observability

**Synthetic monitors**:
- Fleet of automated browsers (Playwright) running the aviary from 5 geographies, every 15 minutes
- Checks: first-bird-visible timing, absence of JS errors, audio context success/failure rate

**Real User Monitoring (aggregate only)**:
- Page load timing (navigation → first-bird-visible, via `PerformanceObserver`)
- Render frame timing (P50, P95 frame duration, via `PerformanceObserver` on `"frame"` type)
- Audio context success/failure count
- Simulation-tick latency (server-side, P50/P95/P99 per tick execution)
- Session duration histogram (anonymized; no per-account dimension)
- Event log submission success/failure rate

**Privacy boundary on telemetry**:
- No per-bird fields in any telemetry event
- No per-account interaction counts in any telemetry event
- No session identifiers that could be joined to account records
- Telemetry pipeline reads only from aggregate counters, never from the simulation database

**Alarms**:
- Tick p99 latency > 5s → PagerDuty
- Audio context failure rate > 5% → Slack alert (non-paging)
- Bundle size CI check failure → blocks deploy

---

## 11. Rollout

### v1 ship plan

**Phase 0: Infrastructure hardening** (weeks 1–3)
- Simulation tick scheduler deployed, tested under load for N=1,000 accounts
- Drift calibration fixture passing in CI (1-week measurable, 3-week user-visible targets)
- State snapshot endpoint deployed with CDN caching
- Event log append-only write path deployed and load-tested

**Phase 1: Closed alpha** (weeks 4–6)
- 50 internal users; 2 starter birds each
- Monitor: tick latency, drift accuracy, event log throughput
- Bug-fixes only; no new features
- Accessibility: screen-reader and reduced-motion verified with assistive technology QA

**Phase 2: Invite beta** (weeks 7–10)
- 500 users, email-invite-only
- Full feature set including visit invitations
- Monitoring dashboard live; synthetic monitors running
- Notebook entry quality review (random sampling of generated prose by the team)
- Drift calibration checked against actual 1-week data from alpha

**Phase 3: Open launch** (week 11+)
- No waitlist, no count surface
- All v1 features enabled
- Bird count ramping: all accounts start at 2 birds; third-bird offer timing begins accruing from account creation date (first offer at 60 days of account age)

### Bird-per-aviary ramp

- Account creation: 2 starter birds (selected by the system; user names them)
- 60 days of aviary age: third-bird offer appears
- 180 days: fourth bird available
- 365 days: fifth bird available
- 18 months: sixth bird available
- 24 months: seventh (cap) bird available

"Aviary age" is calendar time from `account.created_at`, not visit count. A user who signed up 60 days ago and visited once gets the third-bird offer the same as a user who visited daily. The pacing is about the relationship's timeline, not the user's engagement metric.

### Day-one instrumentation

From the moment the product is live, we instrument:
- First-bird-visible timing (P50, P95)
- Tick latency distribution (P50, P95, P99)
- Event log submission volume (signals whether presence tracking is firing)
- Audio context failure rate
- Session duration histogram

We do not instrument: per-bird interaction rates, offer acceptance rates, specific bird greeting events, or any metric that would require touching per-account simulation state.

---

## 12. Risks

### Drift calibration miss

**Risk**: The drift function parameters are miscalibrated — traits change too fast (Tamagotchi feel) or too slow (screensaver feel). The product experience depends on the narrow band between these.

**Mitigation**:
- The CI calibration fixture (simulated 21-day scenario) gates all deploys
- Alpha participants receive a hidden instrumentation layer that tracks personality vector movement against the intended calibration targets, without exposing the numbers to users
- A "drift diagnostic" internal admin endpoint shows per-account personality history for the engineering team; this never touches the product surface
- Calibration parameters (`learning_rate`, weights) live in a config file, not hardcoded — can be adjusted without a deploy

### Sync correctness edge cases

**Risk**: A duplicate tick run processes the same event log twice, causing double-drift. A stale snapshot served from CDN causes a client to display the wrong state after a personality update.

**Mitigation**:
- Tick idempotency: the tick watermark (last processed event_id) is updated atomically with the personality vector write (single transaction). A duplicate tick sees no new events and produces no delta.
- CDN snapshot TTL: 10 seconds. On visibility change, client always bypasses CDN with `Cache-Control: no-cache`. For visit sessions, snapshot is not CDN-cached (vary by invite_id).
- Integration test: simulate two concurrent tick runs for the same account; assert no double-drift.

### Audio uncanniness

**Risk**: The procedural call synthesis sounds mechanical, uncanny, or too similar across birds. The "feels alive" goal collapses if the audio reads as generated noise.

**Mitigation**:
- Motif library is hand-authored per species by a sound designer, not algorithmically generated. The synthesizer applies variation; the motifs themselves are designed.
- A/B listening test with 10 internal participants before alpha: can they distinguish species by call alone? Can they detect the "loop?" Failure gates the audio implementation.
- The synthesis envelope (attack, decay, sustain, release) is species-specific and mood-tuned. "Alert warbler" sounds distinct from "drowsy warbler."
- Cross-bird phase artifact check: two birds calling simultaneously are auditioned to confirm no audible phase-cancellation artifact.

### Accessibility surface regression

**Risk**: A visual or audio change causes the screen-reader narration or caption layer to fall out of sync with the displayed aviary state, or the reduced-motion mode's cross-fade rendering breaks on a new species or mood combination.

**Mitigation**:
- Automated accessibility tests run on every deploy: Axe-core scan of all primary surfaces, synthetic screen-reader playback check (using NVDA/VoiceOver in CI via a headless browser harness)
- The pose-asset library (for reduced-motion cross-fades) is tested in CI: assert that every `(species × mood)` combination has ≥ 3 pose assets. Missing assets fail the build.
- Narration template test: for every snapshot state combination, assert that the template renderer produces non-empty naturalist prose. Template coverage check is part of the snapshot test suite.

### "Notice, never announce" discipline failures

**Risk**: A well-meaning contributor adds a toast, a badge, or a textual greeting that breaks the product's affective contract. These additions look harmless from the outside and are easy to rationalize.

**Mitigation**:
- This principle is named, enumerated, and its specific failure modes are listed in the PRD (welcome toast, "you've been gone X days," streak counter). The planning document repeats the rule.
- Code review checklist includes: "Does this PR add any user-visible text on session start or return?" If yes, it requires explicit PRD-justified sign-off from the product owner.
- A product principle document (separate internal doc) names the violations explicitly for every future contributor.

### Bird identity loss

**Risk**: A data migration, a bug in the bird-creation flow, or a poorly-scoped "reset" operation replaces a bird's personality vector with defaults. The user notices something is wrong without being able to name it; the trust damage is irreversible.

**Mitigation**:
- The simulation database has row-level audit logging: every write to `birds.personality` is logged with the tick watermark and the size of the delta. A write of size > 0.1 on any single trait in a single tick fires an automated alert.
- `bird_id` is immutable at the schema level (`NOT NULL, IMMUTABLE` or equivalent ORM annotation). Any code path that attempts to overwrite a bird_id fails at the database constraint.
- Account export (§3) serves as the user's own backup. Export is available from account settings at any time.
- Soft-deletion (30-day window) protects against accidental account deletion; personality vectors are retained until hard-deletion.

### Privacy boundary breach in telemetry

**Risk**: An engineer adds a telemetry event that accidentally includes a per-bird field or a per-account interaction count, violating the privacy commitment named in `accounts_sync.md`.

**Mitigation**:
- The telemetry pipeline is configured with a deny-list of field names (`bird_id`, `personality`, `mood`, `event_type` from the simulation event log). Any event containing a deny-listed field is rejected at the pipeline ingestion layer.
- The simulation database is not readable by the analytics warehouse (network-level isolation, not just policy).
- Privacy review is required for any new telemetry event type (PR checklist item: "Does this event contain per-bird or per-account simulation state?").
