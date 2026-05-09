# Pocket Aviary — Implementation Plan (v1)

## 1. Scope

### In scope for v1
- Browser-only (no native apps)
- Single-user accounts, magic-link sign-in only
- One aviary per account; starts with two birds, caps at seven
- Server-side simulation tick (~1/min cadence) driving all canonical state
- Procedural call synthesis via WebAudio, client-side
- Mood system (wary, content, curious, drowsy, alert) with daily-cadence resets
- Personality vector: boldness, social warmth, vocal frequency, plumage saturation, curiosity — slow drift over weeks
- Five interactions: return-greeting, listen-in, offer (seed/song-fragment/pool), settle, field-notebook browsing
- Presence accounting with three-signal conjunction (visibilityState + window focus + recent pointer/key activity)
- Field notebook: auto-generated naturalist-prose entries, read-only, sparse (~one entry per few days)
- Day/night cycle anchored to user's local timezone
- Ambient weather (rain, wind) a few times per week
- Multi-device sync as architectural property (server is sole canonical writer)
- Visit invitations: host invites visitor by email, read-only ambient view, revocable, off by default
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fade rendering, not a stripped fallback)
- Call captions (naturalist prose, generated from call grammar at runtime)
- WCAG AA contrast on all user-copy text
- Full keyboard navigation
- Account export (JSON on demand, emailed)
- Account deletion (30-day soft delete then hard delete)
- Synthetic UUID for all internal account references (email stored once, encrypted)
- New-bird unlocking by aviary age (not interaction count, not paid tier)

### Explicitly out of scope for v1
All items in `non_goals.md`: native apps, gamification of any kind (achievements, streaks, badges, counters, levels, green-dot calendars), Tamagotchi-style mechanics (hunger, visible distress, decay), social-network surfaces (profiles, follows, public feed, discovery, comments). Also out: payments, shared/multi-aviary accounts, multi-user sim, SSO, password login, push notifications, recorded audio fallback, personalizable aviary scenes.

---

## 2. Architecture

### Service shape

```
Browser Client (SPA)
   │  HTTPS/REST + SSE
   ▼
API Gateway / Edge (CDN)
   │
   ├── Auth Service          — magic-link issuance, session management
   ├── Aviary API Service    — snapshot delivery, event ingestion, visit management
   ├── Simulation Service    — tick worker, event-log consumer, state writer
   ├── Notebook Service      — entry generation from state snapshots
   └── Notification Service  — email dispatch (magic links, export links)
```

All services talk through an internal message bus (e.g., Kafka or a managed equivalent) for async operations. The Simulation Service is the only writer to the canonical personality-vector record. The Aviary API Service is the only reader the client talks to — it never calls Simulation Service directly.

### Client/server split
- **Server owns**: all canonical state (personality vectors, moods, presence accumulation, notebook entries, aviary age); simulation ticks; event-log persistence.
- **Client owns**: rendering, local presence-event detection, interpolation between snapshots, call synthesis, reduced-motion mode, caption rendering, keyboard and pointer event handling. Client writes only to the append-only interaction-event log via the API.
- **No client-side state is ever authoritative** for personality or mood.

### Render pipeline boundary
The client receives a state snapshot and renders it. The rendering loop is entirely local — no round-trip per frame. Snapshots are pulled: on tab-visible transition, on long render-gap detection (laptop resume), and on a low-frequency keepalive (~30 s) while the tab is visible and present.

---

## 3. Data Model

### Account record
```json
{
  "id":           "UUID",
  "email_enc":    "<AES-256-GCM encrypted email, key managed by KMS>",
  "email_hash":   "<HMAC, used only for uniqueness checks — not stored in logs or external services>",
  "created_at":   "ISO8601",
  "deleted_at":   "ISO8601 | null",
  "sessions":     [{ "token_hash": "...", "device_label": "...", "created_at": "...", "revoked_at": "..." }],
  "settings": {
    "visit_notifications_enabled": false,
    "reduced_motion":              false,
    "captions_enabled":            false
  }
}
```

Internal references everywhere else use the UUID only.

### Aviary record
```json
{
  "id":           "UUID",
  "account_id":   "account UUID",
  "created_at":   "ISO8601",
  "bird_ids":     ["UUID", ...],
  "next_bird_unlock_at": "ISO8601 | null"
}
```

### Bird record
```json
{
  "id":           "UUID (stable, never replaced)",
  "aviary_id":    "UUID",
  "species":      "species_key",
  "name":         "string",
  "adopted_at":   "ISO8601",
  "personality":  {
    "boldness":          0.0–1.0,
    "social_warmth":     0.0–1.0,
    "vocal_frequency":   0.0–1.0,
    "plumage_saturation":0.0–1.0,
    "curiosity":         0.0–1.0
  },
  "mood":         "wary | content | curious | drowsy | alert",
  "mood_set_at":  "ISO8601",
  "perch_zone":   "front | middle | back",
  "call_state":   { "last_call_at": "ISO8601", "current_motif_seed": "int" }
}
```

Personality vector is never returned to the client in raw form. The snapshot omits numeric values; it returns only rendered outputs (perch zone, mood, call parameters, plumage saturation as a visual token, not a number).

### Presence event (append-only)
```json
{
  "id":         "UUID",
  "account_id": "UUID",
  "started_at": "ISO8601",
  "ended_at":   "ISO8601 | null",
  "duration_s": "int | null"
}
```

### Interaction event (append-only)
```json
{
  "id":         "UUID",
  "account_id": "UUID",
  "bird_id":    "UUID | null",
  "type":       "listen_in_start | listen_in_end | offer | settle | return_view",
  "payload":    { "offer_type": "seed | song_fragment | pool" },
  "occurred_at":"ISO8601"
}
```

### State snapshot (server → client, ephemeral)
```json
{
  "snapshot_at": "ISO8601",
  "aviary_id":   "UUID",
  "local_time_of_day": "float 0–24 (user's local hour)",
  "weather":     "clear | rain | wind | null",
  "birds": [
    {
      "id":               "UUID",
      "name":             "string",
      "species":          "species_key",
      "mood":             "wary | content | curious | drowsy | alert",
      "perch_zone":       "front | middle | back",
      "plumage_level":    "low | medium | high",
      "call_params":      { "motif_seed": int, "pitch_offset": float, "timing_scale": float },
      "offer_cooldown_ends_at": "ISO8601 | null"
    }
  ],
  "settled": false,
  "notebook_entry_count": int
}
```

### Notebook entry
```json
{
  "id":         "UUID",
  "aviary_id":  "UUID",
  "written_at": "ISO8601",
  "prose":      "string (naturalist field-notebook text)"
}
```

### Visit invitation
```json
{
  "id":           "UUID",
  "host_account": "UUID",
  "visitor_email_enc": "<encrypted>",
  "token_hash":   "<bcrypt hash of the one-time token>",
  "issued_at":    "ISO8601",
  "expires_at":   "ISO8601 (+30 days)",
  "revoked_at":   "ISO8601 | null",
  "used_at":      "ISO8601 | null"
}
```

---

## 4. API Surface

### Auth
| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/magic-link` | Issue magic link to email |
| POST | `/auth/verify` | Consume magic-link token → issue session |
| DELETE | `/auth/sessions/{session_id}` | Revoke a session |
| GET | `/auth/sessions` | List active sessions |

Magic-link tokens: cryptographically random 32-byte tokens, stored as bcrypt hashes, expire at 15 minutes, one-use invalidation on consumption.

### Aviary state
| Method | Path | Description |
|--------|------|-------------|
| GET | `/aviary/snapshot` | Fetch current canonical state snapshot |
| GET | `/aviary/snapshot/sse` | Server-sent events stream; server pushes snapshot diffs on tick or notable event |
| POST | `/aviary/events` | Append interaction events (offer, listen-in start/end, settle, presence ping) |

Snapshots are signed with an `ETag`; clients cache and send `If-None-Match`. The `/sse` endpoint delivers lightweight diff objects, not full snapshots; the client merges diffs into its in-memory snapshot.

### Field notebook
| Method | Path | Description |
|--------|------|-------------|
| GET | `/notebook/entries?before={ISO8601}&limit={n}` | Paginated notebook entries, newest first |

No write endpoints; the notebook is server-generated.

### Account
| Method | Path | Description |
|--------|------|-------------|
| GET | `/account` | Account settings |
| PATCH | `/account/settings` | Update settings (reduced_motion, captions, visit_notifications) |
| POST | `/account/email-change` | Initiate email change (sends verification to new address) |
| POST | `/account/export` | Request JSON export (generates async, emails download link) |
| DELETE | `/account` | Initiate soft deletion |
| POST | `/account/restore` | Cancel pending deletion (within 30-day window) |

### Visits
| Method | Path | Description |
|--------|------|-------------|
| POST | `/visits/invitations` | Create a visit invitation (host) |
| DELETE | `/visits/invitations/{id}` | Revoke an invitation (host) |
| GET | `/visits/invitations` | List host's outstanding invitations |
| GET | `/visits/log` | Host's visit log |
| GET | `/visits/view/{token}` | Visitor: validate token, get read-only snapshot |
| GET | `/visits/view/{token}/sse` | Visitor: SSE stream for read-only snapshot updates |

Visitor endpoints do not require account authentication. The visit token is sufficient and short-scoped. Visitor sessions never write to the event log.

---

## 5. Simulation Engine Design

### Tick runner
The Simulation Service runs a scheduler that triggers a tick per account approximately every minute. Exact cadence is calibrated during build; the tick is staggered across accounts to avoid thundering-herd on the database.

Each tick:
1. Lock the account's simulation record (advisory lock or optimistic CAS).
2. Read interaction events since the last tick from the append-only event log.
3. Compute mood transitions for each bird.
4. Compute personality drift deltas.
5. Advance bird-to-bird interaction state.
6. Generate notebook entry if conditions met.
7. Advance weather state if due.
8. Write the updated bird records (mood + personality vector + call state).
9. Write a new snapshot record.
10. Emit an SSE diff event to any connected clients.
11. Release lock, record tick metadata (latency, events consumed).

### Mood transitions
Mood is a small enumerated state: `wary, content, curious, drowsy, alert`. Transitions are probabilistic, governed by:

- **Time-of-day bucket** (morning, midday, afternoon, evening, night) in the user's local timezone. Each species has a time-of-day preference table. Early morning → alert; late evening → drowsy; night → settled/drowsy.
- **Recent interaction signal**: an `offer` accepted in the last tick nudges toward `content`; a `listen_in` nudges toward `alert`; weather `rain` nudges vocal-frequency down and may nudge toward `wary`.
- **Personality modulation**: a high-boldness bird has a lower probability of entering `wary` on the same input signal.
- **Bird-to-bird contagion**: wary mood in one bird raises the probability of `wary` in nearby birds (same perch zone).

Mood transitions are computed as a weighted probability lookup table, not a formula, so they can be tuned during calibration without code changes. The table is stored as a JSON config artifact versioned alongside the simulation service.

Mood persists across ticks and across the user's sessions; it does not reset on tab open.

### Personality drift function
Drift is a low-pass filter applied as an additive delta to each personality trait once per tick.

```
delta(trait) = learning_rate × (signal(trait) - current(trait))  [monotonic clamp]
```

Where `signal(trait)` is derived from presence time and interaction events since the last tick:
- `boldness` signal rises with offer events (any offer near a bird) and presence time.
- `social_warmth` signal rises with listen-in events.
- `vocal_frequency` signal rises with listen-in events and presence time.
- `plumage_saturation` signal rises with presence time.
- `curiosity` signal rises with accepted offer events.

Learning rate is set very low (empirically calibrated so that measurable drift occurs after ~one week of regular visits, visible drift after ~three weeks).

**Monotonic clamping**: delta is never negative. Traits only increase toward expressive; they do not decrease on neglect. The drift function checks `max(0, delta)` before applying.

The canonical test: an automated test harness simulates 7 days × 60-minute sessions of synthetic presence and confirm measurable trait movement; simulates 14 days of absence and confirms no trait regression.

### Call grammar runtime
Each species has a call-grammar definition: a small library of motifs (pitch contour, duration, amplitude envelope), a timing rule (inter-call interval drawn from a distribution keyed to `vocal_frequency`), and combination rules (motifs that can be chained, repeated, or responded to).

At tick time, the Simulation Service computes `call_params` (motif seed, pitch offset, timing scale) from the bird's current mood and personality and stores them in the snapshot. The client reads `call_params` and synthesizes the call via WebAudio without further server contact.

The motif-seed and pitch-offset are chosen so that:
- The same bird's calls are recognizable across mood and personality drift (stable motif base).
- No two consecutive calls are identical (seed is updated each call using a deterministic PRNG that includes the call timestamp, so the client can re-derive it deterministically).
- High vocal frequency → shorter inter-call interval drawn from the distribution.

Bird-to-bird response: the client detects when one bird's call ends and, if a second bird's call_params indicate a response window is open (a time range after the first call, encoded in the snapshot), triggers the second bird's call synthesis.

### Weather state machine
Weather state: `clear (default) → rain → clear` or `clear → wind → clear`. Transitions are triggered by the tick runner on a stochastic schedule targeting roughly 2–3 weather events per week per aviary. Weather events have a duration (rain: 5–15 minutes, wind: 3–10 minutes). The weather state is included in the snapshot and drives client-side visual and audio adjustments. The simulation service also applies the mood-nudge for weather-exposed birds during the ticks that overlap with active weather.

---

## 6. Sync Model

Multi-device sync is an emergent property, not a separate system.

- Canonical state lives in one database record per bird, written only by the Simulation Service tick.
- Every client pulls from the same record via the Aviary API Service.
- No client holds mutable personality state. Clients hold only the current rendered snapshot and the local interaction event queue.
- Interaction events are append-only. The client fires a POST to `/aviary/events` and forgets; the simulation tick consumes them in order on the next pass.

### Conflict avoidance (not resolution)
The system is designed so that personality-state conflicts cannot occur:
- Only the server tick writes personality vectors → no client can overwrite another.
- Interaction events are append-only with server-assigned timestamps → no ordering ambiguity.
- Session tokens are per-device; two devices operating simultaneously both write their events to the same append-only log, and the tick processes both.

### Snapshot freshness
A client that has been dormant (laptop suspend, background tab for >5 minutes) pulls a fresh snapshot on next visibility. The `ETag`-based cache means that if nothing changed, the pull is a single round-trip that returns 304.

### Visit client consistency
A visitor client follows the same pull model as an authenticated client, minus write access. Visit tokens are scoped to read-only snapshot access for the host's aviary. Revocation takes effect at the next snapshot pull: the API returns 403 with a matter-of-fact body; the client renders the "visit no longer available" surface.

---

## 7. Frontend Rendering Pipeline

### Technology choices
- **Framework**: React (or Preact for bundle savings) with a custom canvas/WebGL layer for the aviary scene. The top bar, notebook panel, and account/settings flows are standard DOM. The aviary scene is rendered on a `<canvas>` element.
- **Scene renderer**: WebGL-accelerated (via a thin wrapper — no full 3D engine). Falls back to Canvas 2D API if WebGL is unavailable; the visual fidelity budget for Canvas 2D is defined during visual design.
- **Audio**: WebAudio API exclusively. No `<audio>` elements, no recorded files.
- **State management**: Minimal; snapshot + local interaction queue held in a single store. No global state framework needed at v1 scale.

### Scene composition
The scene has three rendering layers drawn back-to-front:
1. **Background layer**: sky gradient (time-of-day + weather state), background foliage, soft parallax offset keyed to a very slow drift function.
2. **Bird layer**: per-bird sprite/vector asset drawn at current perch position, animated by mood-keyed idle motion rigs.
3. **Foreground layer**: ambient leaf/feather drift (generated client-side, no server state), foreground branch/foliage elements, still-pool overlay when active.

### Bird animation rigs
Each species has a set of pose keyframes for each mood (wary, content, curious, drowsy, alert) and for each action (call, head-tilt, preen, perch-shuffle, fly-to-perch, bathe). The renderer interpolates between keyframes for smooth motion.

Reduced-motion mode replaces frame-by-frame interpolation with cross-fades between still poses: the renderer picks a target pose based on the current mood, cross-fades at ~2 s intervals. The cross-fade is itself a subtle motion that preserves presence without vestibular risk.

### First frame strategy
On navigation, the client:
1. Fetches the initial HTML shell from the CDN edge (includes the inline initial snapshot embedded as a `<script type="application/json">`).
2. Hydrates the React app.
3. Initializes the canvas, positions birds at their snapshot perch zones, and begins the animation loop — all before any subsequent network requests complete.
4. Birds appear mid-motion: the animation loop starts at a random frame offset within the idle-motion cycle so no bird begins from a "start" pose.

The inline snapshot is embedded in the HTML response by the CDN edge worker, which fetches it from the Aviary API on the user's first request. Total round-trips to first bird: 1 (the HTML itself). Subsequent snapshots arrive via the SSE stream.

### Loading state (when snapshot is unavailable)
If the inline snapshot is unavailable (cold cache, first-ever load race), the client renders the quiet-field state: soft sky color, one or two ambient motion cues (slow leaf drift). No spinner. No "Loading…" text. This state resolves as soon as the SSE stream delivers the first snapshot diff, typically within 200–300 ms.

### Top bar fade
The top bar's opacity CSS property is animated on a timer that resets on pointer movement or keyboard activity. After 4 seconds of inactivity, the bar fades to ~15% opacity via a CSS transition. On activity, it returns to 100% with a short ease-in (150 ms). The fade is disabled when keyboard focus is within the top bar.

### Idle micro-motion loop
Each bird's idle animation runs at 60 fps via `requestAnimationFrame`. The loop samples the current mood every frame and applies the corresponding keyframe interpolation. The loop is paused when `visibilityState` is `hidden` (no rendering needed). The simulation continues server-side regardless.

### Transition animations
- **Perch-to-perch move**: fly-arc animation (300–500 ms) when a bird changes perch zone. The perch change is communicated via SSE diff; the client triggers the fly animation and resolves the bird at the new perch.
- **Settle lighting shift**: a 3-second CSS gradient transition on the background layer warming the palette toward evening hues, combined with a gradual volume ramp-down in the audio graph.
- **Settle undo**: if the user clicks anywhere in the aviary within 5 seconds of triggering settle, the CSS transition is reversed; the settle flag is cleared on the next event POST.

### Listen-in mix
The audio graph has one `GainNode` per bird connected to a master mixer. On listen-in start, the focused bird's gain ramps up to 1.0 over 2 seconds; all other birds' gains ramp to 0.2 over the same period. On listen-in end, all gains ramp back to their default values (0.6 for all birds, modulated by vocal_frequency). The ramp uses `linearRampToValueAtTime` on the Web Audio API.

---

## 8. Audio Pipeline

### Call synthesis
Each bird's call synthesis is driven by its `call_params` snapshot value. The synthesis chain:

```
Motif generator (motif_seed → frequency sequence)
   → OscillatorNode (pitch_offset applied)
   → BiquadFilterNode (species-specific formant shaping)
   → EnvelopeNode (GainNode with ADSR from species motif definition)
   → BirdGainNode (per-bird gain, controlled by listen-in mix)
   → MasterGain
   → AudioContext.destination
```

All oscillator and gain nodes are pooled and reused. No per-call node allocation that isn't returned to the pool. This is the implementation of the "no memory growth" contract for audio.

### Motif library
Each species ships with 4–8 base motifs stored as compact parameter objects (not audio files):
```json
{ "pitch_contour": [440, 520, 490], "duration_ms": [80, 120, 100], "amplitude": [0.8, 1.0, 0.7], "inter_note_gap_ms": 30 }
```

The motif generator selects and sequences motifs according to the species call-grammar rules, uses the `motif_seed` as PRNG input, and produces a sequence of OscillatorNode parameter commands. The same seed deterministically produces the same call, so the client can regenerate captions without additional server state.

### Caption generation
The caption text is generated from the same parameter commands that drive the synthesis. A simple mapping from pitch contour shape and duration characteristics to prose:
- Short high notes → "a quick bright call"
- Slow descending contour → "a soft descending call"
- Repeated motif → "a low trill, paused, low trill again"

This logic runs synchronously alongside synthesis and produces the caption string before the call begins playing. Caption text is positioned near the calling bird (CSS `position: absolute` over the canvas) with a fade-in at call-start and fade-out at call-end.

### Chorus mixing
Multiple birds calling simultaneously is handled naturally by the per-bird gain architecture. No special chorus mixer is needed. The `vocal_frequency` trait shapes the inter-call interval distribution so that high-frequency birds call more often, but the independence of the call schedulers means calls can naturally overlap or stagger.

### WebAudio fallback
If `AudioContext` construction fails (permissions denied, old browser, hardware absence), a `webAudioUnavailable` flag is set. The aviary renders silently with captions forced on. No recorded-audio fallback is attempted. The user sees a single matter-of-fact line in accessibility settings: "Audio is unavailable on this device. Captions are on."

### Audio context activation
Browsers require a user gesture to activate an `AudioContext`. The first user interaction with the aviary (any pointer or keyboard event) triggers `audioContext.resume()`. Before activation, the aviary renders visually; calls are queued and fire on activation. No special "click to enable audio" surface — the first natural interaction is sufficient.

---

## 9. Accessibility Surfaces

### Screen-reader narration
A `<div aria-live="polite" aria-atomic="true">` region is maintained in the DOM, off-screen but not `display: none`. Its text content is updated on a slow cadence (30–60 s at idle). Text is naturalist prose generated server-side or client-side from the same snapshot state the visual renderer reads.

Narration generation:
- Base state: describe the birds currently visible (perch, mood-expressed behavior, light quality). One or two sentences.
- Prompt refresh: on user-initiated events (return-greeting, offer reaction, settle), narration updates promptly (within the current event handling cycle, not on the slow timer). These updates use `aria-live="assertive"` on a second region for timely delivery.
- Content examples:
  - Idle: "a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning; the light is gentle."
  - On offer accepted: "the bird approaches the seed, tilting its head once before taking it."

The narration prose generator is a dedicated module that takes the snapshot as input and produces prose strings. It is tested independently of the rendering pipeline.

### Reduced-motion
`prefers-reduced-motion: reduce` is detected via `window.matchMedia`. The setting is also toggleable from accessibility settings (stored in `account.settings.reduced_motion`). When active, the renderer substitutes the cross-fade mode for all bird animations. CSS `transition` durations on the top bar and lighting shifts are halved. Ambient leaf/feather drift is suppressed. The audio pipeline is unaffected.

The reduced-motion setting takes effect immediately on change; no reload required.

### Call captions
Captions are togglable from accessibility settings (`account.settings.captions_enabled`) and also auto-enabled on WebAudio failure. Each caption element is positioned absolutely over the canvas at the bird's current pixel position. WCAG AA contrast is enforced by a semi-transparent background on the caption chip. Caption elements are not in the ARIA live region (they would produce duplicate narration for screen-reader users who also use captions).

### Keyboard navigation
Tab order: top-bar icons (left to right) → aviary scene. When focus enters the aviary scene, it is held on a single `<div tabindex="0">` with `role="application"`. Within the application region:
- Arrow keys cycle focus between birds (using an internal bird-focus state).
- `Enter` triggers listen-in on the focused bird; `Escape` exits listen-in.
- `Space` triggers the offer affordance (opens the offer panel).
- Tab while in the aviary exits to the next top-bar element.

A visible focus indicator (2px solid outline, high-contrast color defined in design system) appears on the currently focused bird's bounding box. The indicator is rendered on the canvas, not as a DOM overlay, to keep it accurately positioned across responsive scaling.

### WCAG AA compliance scope
Text covered: top-bar labels, settings surfaces, account surfaces, error messages, caption text, notebook prose (when displayed). The aviary scene background is not user-copy and is exempt, but the caption chips must meet contrast. All ratios are validated in automated CI using an ARIA/contrast testing tool (e.g., axe-core) on every PR.

---

## 10. Performance Budgets and Observability

### Bundle budget: <2MB gzipped initial JS
- The aviary scene renderer, call synthesis engine, and presence-accounting code must all be in the initial bundle.
- Deferred chunks (code-split): account settings, accessibility settings, notebook panel, visit-invitation flow, adoption flow. Each of these is lazy-loaded on first user interaction.
- Bird visual assets: SVG or compact bitmap sprite sheets; no raster assets >100KB in the initial load.
- Audio motif parameters are inlined in the bundle as JSON objects — they are small (a few KB per species).
- Tracked in CI with a bundle-size check that fails the build if the initial chunk exceeds 2MB gzipped.

### Time to first bird visible: <500ms (mid-tier mobile, 4G)
- Inline snapshot in HTML response eliminates a round-trip for initial state.
- The renderer begins placing birds on the canvas before deferred assets load.
- Synthetic performance tests (Lighthouse CI or equivalent) run on every commit to main; a regression >10% from baseline fails the build.

### 60fps idle on a 5-year-old laptop
- The animation loop uses `requestAnimationFrame`. Frame budget: 16.7ms.
- Heavy computations (call synthesis, snapshot merging) run in a Web Worker and post results to the main thread.
- No DOM mutations during the animation loop; all visual state is written to the canvas.
- Tested with automated browser benchmarks on a reference device profile in CI.

### No memory growth over 30 minutes
- Audio node pooling: oscillator and gain nodes are reused, not created per call. Pool size is fixed at a small constant (e.g., 2× the max bird count).
- Notebook entries: only entries currently scrolled into view are kept in DOM; others are virtualized.
- Snapshot diffs: the client merges diffs into a single in-memory snapshot, discarding old snapshots.
- Leaf/feather particle system: fixed pool of N particles; re-used on recycling.
- A Playwright-based test holds the aviary open for 30 minutes (time-accelerated), runs the GC, and asserts that heap size has not grown by more than a small threshold.

### What we measure
- **Synthetic**: Lighthouse CI on every commit (FCP, TBT, bundle size, first-bird-render via a custom Playwright assertion).
- **RUM**: anonymized page-load timings, first-bird-render timings, render-frame timings (median and p95), audio-context error counts, simulation-tick p50/p95/p99 latencies. No per-account, per-bird fields.
- **Server**: simulation-tick latency histogram; p99 > 5s triggers an alert.
- **Privacy boundary**: telemetry pipelines are explicitly forbidden from joining on account UUID or reading the simulation database. Enforced by IAM policy and a quarterly audit.

### What we deliberately do not measure
- Per-account visit frequency.
- Per-account interaction patterns.
- Per-bird drift rates across the population.
- Any metric that, if aggregated, would constitute per-bird interaction history.

---

## 11. Rollout

### Pre-launch
- Internal alpha: team members only, all seven bird species available, aviary age accelerated (1 hour of real time = 1 week of aviary age) to validate drift calibration.
- Drift calibration test: confirm measurable trait change in instruments after the equivalent of 7 real days; confirm no visible change to non-instrumented reviewers before the equivalent of 21 days.
- Accessibility audit: external audit with screen-reader users and reduced-motion users before open beta. Findings treated as launch blockers if they affect the affective quality of the accessible surface.

### Launch (v1.0)
- Open signup with magic-link auth.
- New account starts with two birds; aviary caps at seven.
- Bird species pool: six species, available at account creation.
- New-bird unlock schedule: third bird available at 90 days of aviary age; fourth at 180 days; fifth at 365 days; sixth at 548 days (18 months); seventh at 730 days (2 years). These numbers are starting calibration; the schedule is stored in config and adjustable without code deploy.
- Visit invitations available on launch; off by default.
- Screen-reader narration, reduced-motion mode, call captions all ship at v1.0 (not as v1.1 fix).
- Performance budgets enforced in CI from day one.

### Bird-count ramp
No artificial ramp on bird-per-aviary count — the unlock schedule is the ramp. The seven-bird cap is enforced server-side.

### Day-one instrumentation
- Simulation-tick latency histogram.
- First-bird-render time (RUM, anonymized).
- Audio-context error count.
- Active-session count (no per-account detail).
- Account creation rate and deletion rate.
- No per-account or per-bird telemetry.

### Feature flags at v1
No feature flags on core surfaces. Code-splitting already handles the deferred-load surfaces. A simple config flag is used for the bird-unlock schedule so it can be adjusted post-launch without a code deploy.

---

## 12. Risks

### Drift calibration
**Risk**: the low-pass filter learning rate is too fast (birds change visibly between sessions, feeling like a stat game) or too slow (no perceived change after months of visits).
**Mitigation**: Calibration is done in the internal alpha against the named targets (measurable change after 7 days, visible change after 21 days). Learning rates are stored in a config artifact, not hardcoded. An automated test harness validates the targets on every change to the drift function.

### Sync correctness under multi-device concurrency
**Risk**: two devices submit interaction events simultaneously; the tick processes them in an order that produces unexpected drift.
**Mitigation**: Events are append-only with server-assigned timestamps. The tick processes events in timestamp order. No client writes personality directly. The risk reduces to "events processed in unusual order," which is acceptable; the drift function is insensitive to short-term reordering because the learning rate is so low.

### Audio uncanniness
**Risk**: procedural call synthesis that sounds mechanical, uncanny, or repetitive even without exact loop repetition. Users consciously or unconsciously sense the procedural quality.
**Mitigation**: Motif libraries are designed and tested by someone with musical/ornithological ear. Call grammar combination rules are validated by listening, not only by automated test. The internal alpha includes a specific "does Pip sound alive?" evaluation with non-engineer listeners. Recognizability across mood states is tested by presenting calls to alpha listeners without labeling mood and asking them to identify the bird.

### Accessibility regression
**Risk**: a visual-rendering change inadvertently breaks the narration text, or a CSS change breaks the focus indicator against a new aviary palette state.
**Mitigation**: axe-core runs on every PR against a set of representative aviary states (morning, night, settled, listen-in active). The narration text generator has its own unit tests against snapshot fixtures. Focus-indicator contrast is validated in CI against both the lightest and darkest aviary background states.

### Presence-signal inflation
**Risk**: a browser or platform change causes `visibilityState` or `focus` events to behave differently, inflating presence time and corrupting drift.
**Mitigation**: The three-signal conjunction is implemented in a single isolated module with clear contracts. Browser-compatibility tests run against the last two versions of each supported browser. The presence module emits a per-session presence-time count that is visible in aggregate (anonymized) telemetry; an anomalous spike in median session-presence-time triggers a review.

### "No toast" discipline
**Risk**: a well-meaning contributor adds a notification or announcement surface that violates the "notice, never announce" principle.
**Mitigation**: The principle is documented in the product brief and in `interactions.md` with explicit examples. Code review includes a "did you add a toast?" check for any PR touching session-start or interaction-response code paths. A linting rule or comment convention flags the relevant component files.

### Performance regression after launch
**Risk**: a subsequent feature adds to the initial bundle, crossing the 2MB threshold and breaking the <500ms first-bird target.
**Mitigation**: Bundle-size CI check fails the build above 2MB. First-bird-render synthetic check is part of the same CI pipeline. Lazy-loading of non-critical surfaces is enforced in code review.

### Personality vector loss
**Risk**: a database migration, a bug in the tick runner, or a deployment error resets or corrupts a personality vector. Users notice their bird "forgot" them.
**Mitigation**: Daily point-in-time backups of the bird record database. The tick runner writes a before-and-after audit log for each personality-vector update (retained for 90 days). A soft-corruption canary: if any single tick produces a delta that would move any trait more than 3× the maximum expected learning rate, the write is rejected and the anomaly is logged for review. Account export (on demand) gives users a personal copy of their bird state.

---

## Ambiguity decisions (defensible calls made by the planner)

1. **SSE vs. WebSocket**: SSE chosen for snapshot delivery. It's simpler, HTTP/2-friendly, and read-mostly. The client writes via REST. No bidirectional communication is needed; SSE is sufficient.

2. **Inline snapshot delivery via edge worker**: the HTML response embeds the initial snapshot as a `<script type="application/json">` injected by a CDN edge worker (e.g., Cloudflare Worker or Fastly Compute). This eliminates the first round-trip for state without requiring the client to make a separate API call before rendering.

3. **Canvas + WebGL for scene, DOM for chrome**: the aviary scene is canvas-rendered for animation-loop control and memory management; all other surfaces (top bar, notebook, settings) are DOM. This boundary is clean and avoids the accessibility complications of a fully canvas-rendered UI.

4. **Notebook entry generation**: notebook entries are generated by the Notebook Service as a side effect of the simulation tick, triggered when a noteworthy state transition is detected (first-greeting-order change, unusual mood persistence, a multi-bird chorus event). The generation cadence is governed by a "min gap" config (never more than one entry per ~48 hours for a given aviary, regardless of activity) to preserve sparsity. Entries are generated by a small LLM-based prose generator or a template system — the choice between these is deferred to implementation; both can honor the naturalist-voice constraint if the prompt/template is designed carefully.

5. **Adoption flow on first sign-in**: the first-sign-in flow shows a brief one-page adoption surface (species announced, naming prompt). No species catalog; the two starters are presented as "the birds that arrived." The adoption flow is the only time before the aviary first renders; it is brief by design.

6. **Bird visual assets**: SVG vector assets per species, with plumage saturation expressed as a CSS filter (`saturate()`) applied to the asset. This allows plumage drift to be rendered without separate asset variants, keeping the initial bundle within budget.

7. **Notebook LLM vs. templates**: if an LLM-based prose generator is used, it is called server-side by the Notebook Service, receives only the event type and bird names (no personality vectors, no numeric drift values), and produces naturalist prose. The prompt is the voice and tone spec. Output is reviewed in the internal alpha before launch. If the prose quality is insufficient, a deterministic template system with name-interpolation and mood-specific variants is the fallback.
