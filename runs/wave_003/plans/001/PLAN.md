# Pocket Aviary — V1 Implementation Plan

This plan interprets the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable build for a frontier engineering team. It does not restate the spec; it makes the calls the spec leaves to implementation and lays out the system that delivers them.

---

## 1. Scope

### In v1 (confirmed from the brief/scope statement)

- Single-user accounts, magic-link auth, one canonical aviary per account, multi-device sync via server-canonical state.
- Two starter birds at adoption, server-selected species, user-named; cap of seven birds; age-gated new-bird offers.
- Server-side simulation tick (~60s cadence) driving mood transitions and personality drift; client is a pure renderer of snapshots.
- Procedural, client-side WebAudio call synthesis; no recorded audio, ever, including as a fallback.
- Return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (sparse, naturalist, read-only).
- Single horizontal scene, three perch zones, day/night cycle on local time, rare ambient weather, top-bar chrome that fades on idle.
- Visit-invitation social feature: per-invite opt-in, read-only ambient, revocable, off by default, no notifications by default.
- Screen-reader narration (naturalist prose, not state-list), reduced-motion mode (its own designed rendering, not animations-off), call captioning, WCAG AA contrast, full keyboard navigation.
- Performance budgets: ≤2MB gzipped initial JS, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle motion sustained over 30 minutes, no memory growth over 30 minutes.
- Account export (JSON snapshot, emailed), 30-day soft account deletion.

### Explicitly not in v1 (per `non_goals.md` and brief)

Native apps, gamification of any kind (no achievements/streaks/levels/scores/badges/visit-frequency surfaces of any form, including disguised variants like a settings calendar), Tamagotchi mechanics (no death, hunger, decay, distress states — drift is monotonic toward expressive only), social-network surfaces beyond the single visit affordance (no profiles, follows, public discovery, leaderboards, comments), payments, shared/household aviaries, customizable scenes, multi-aviary accounts, push notifications of any kind, SSO/password auth, co-presence in visits.

### Defensible calls on ambiguous points (noted per PRD instruction, not deferred as open questions)

- **Tick cadence**: fixed at 60s. The PRD says "~once per minute, calibrated during build" — 60s is the calibration default; it's tunable via a single config value, not re-architected.
- **Presence pointer/key activity window**: fixed at 3 minutes (PRD says "a few minutes, leaning toward the longer side"). Configurable server-side constant, not hardcoded into client logic, so it can be tuned post-launch without a client release.
- **Mood enum**: finalized as `{wary, content, curious, drowsy, alert, settled}` — the five named in `bird_engine.md` plus `settled` as the terminal state produced by the settle gesture and full-night, since the spec treats "settled" as a distinct rendered/narrated state from "drowsy."
- **Personality trait ranges**: stored as floats normalized to `[0, 1]`, seeded for new birds in `[0.35, 0.65]` (mid-range, avoids a new bird reading as maximally bold or maximally shy on day one, which would look like the engine already "knows" the bird).
- **Offer cooldown**: 5 minutes per bird per offer-type, configurable server constant.
- **Bird species pool**: 6 species as stated; exact species/silhouettes are an asset-design deliverable tracked outside this engineering plan, but the engine and asset pipeline must treat species as a swappable data table (id, silhouette asset, palette, call-grammar motif library reference) so adding a 7th species later is a data change, not a code change.

---

## 2. Architecture

### Service shape

```
┌─────────────┐      HTTPS/JSON       ┌──────────────────────┐
│   Client     │ ───────────────────▶ │   API Gateway / BFF   │
│ (browser)    │ ◀─────────────────── │  (auth, snapshot read,│
└─────────────┘    snapshot pull,      │   event write)        │
                    event push          └──────────┬────────────┘
                                                    │
                          ┌─────────────────────────┼─────────────────────────┐
                          ▼                         ▼                         ▼
                 ┌────────────────┐       ┌──────────────────┐      ┌──────────────────┐
                 │  Account Svc    │       │  Aviary State DB  │      │ Event Log (append │
                 │ (auth, magic-   │       │ (canonical: birds,│      │  -only: offers,    │
                 │  link, sessions,│       │  personality,     │      │  listen-ins,       │
                 │  export/delete) │       │  mood, perch pos) │      │  settle, presence) │
                 └────────────────┘       └─────────▲──────────┘      └─────────┬──────────┘
                                                      │                          │
                                            ┌─────────┴──────────────────────────┘
                                            │
                                   ┌────────▼─────────┐
                                   │  Simulation Tick   │
                                   │  Worker (cron, ~60s│
                                   │  per active aviary)│
                                   └────────┬──────────┘
                                            │
                                   ┌────────▼─────────┐
                                   │ Notebook Generator │
                                   │ (consumes tick      │
                                   │  output + event log,│
                                   │  writes sparse       │
                                   │  prose entries)      │
                                   └───────────────────┘

                 ┌────────────────┐
                 │  Visit Service  │ (invitations, revocation, read-only token issuance)
                 └────────────────┘
```

This is five services behind one BFF, not a monolith, because the simulation tick has a fundamentally different scaling and failure profile from request/response auth and snapshot serving — it must run unconditionally on a schedule regardless of read traffic, and a slow tick must never block a client's ability to read its last-known snapshot. Co-locating them in one deployable would couple their failure domains; splitting them lets the tick worker degrade (queue backlog, alarm) without taking down sign-in or snapshot reads.

### Client/server split (hard boundary, restated as an engineering rule)

- The client never computes personality, mood transitions, or drift. It renders the last-pulled snapshot, interpolates between two snapshots for smooth motion, and runs purely cosmetic ornamentation (ambient leaf/feather drift — explicitly "not driven by the simulation tick" per `aviary_layout.md`) locally.
- The client writes only to the event log (offer, listen-in start/end, settle, presence pings). It never writes to personality or mood fields, at any layer, under any code path — enforced at the API schema level (the event-write endpoint's request schema has no fields for personality/mood; there is no endpoint that accepts them from a client).
- The simulation tick is the only writer of canonical state. This is the implementation of the "no last-write-wins" rule in `accounts_sync.md`.

### Render pipeline boundary

Rendering is split into three layers composited in a single canvas/WebGL surface (see §7 for detail):
1. **Simulation-driven layer** — bird positions, poses, moods; sourced from snapshots, interpolated.
2. **Ambient ornament layer** — leaves, feathers, parallax; client-local pseudo-random generation, no server round-trip, no persisted state.
3. **Environment layer** — sky/light palette computed from local time + weather flag in the snapshot.

The boundary matters because layer 2 must never be confused with simulated state in telemetry or in the snapshot payload — it is rendering noise, not product state, and keeping it client-only is what keeps snapshots small (performance budget) and keeps the "no per-leaf state" rule honest.

---

## 3. Data model

All identifiers are synthetic UUIDs (per `accounts_sync.md` — email is never a key anywhere outside the account record).

### Account
```
Account {
  account_id: UUID (PK)
  email_encrypted: bytes        // encrypted at rest, the only place email lives
  email_verified: bool
  created_at, deletion_requested_at: timestamp | null
  hard_delete_at: timestamp | null   // created_at_deletion + 30d
  visit_notifications_enabled: bool  // default false
  reduced_motion_opt_in: bool        // explicit opt-in distinct from prefers-reduced-motion media query
  captions_enabled: bool             // default false; true if WebAudio unavailable
}
```

### Aviary (1:1 with Account at v1, modeled as separate entity so the schema doesn't have to change if multi-aviary is ever revisited)
```
Aviary {
  aviary_id: UUID (PK)
  account_id: UUID (FK, unique)
  created_at: timestamp           // anchors "aviary age" for new-bird-offer pacing
  timezone_offset_minutes: int    // last-known client-reported offset, for day/night calc when no client connected
  weather_state: enum(clear, rain, wind) 
  weather_started_at: timestamp | null
}
```

### Bird
```
Bird {
  bird_id: UUID (PK)               // stable for the bird's lifetime; never reused, never reassigned
  aviary_id: UUID (FK)
  species_id: UUID (FK -> SpeciesDefinition)
  name: string                     // user-assigned, renameable any time
  adopted_at: timestamp
  personality: {
    boldness: float [0,1]
    social_warmth: float [0,1]
    vocal_frequency: float [0,1]
    plumage_saturation: float [0,1]
    curiosity: float [0,1]
  }
  mood: enum(wary, content, curious, drowsy, alert, settled)
  mood_entered_at: timestamp       // for mood-duration-shaped transitions
  perch_zone: enum(front, middle, back)
  last_tick_at: timestamp
}
```
Personality fields are never returned by any API the client can introspect outside of what's needed to *render* (i.e., the client receives derived render parameters — pose bias, call timing bias — not raw trait values. See §4, "what the snapshot contains.")

### SpeciesDefinition (static reference data, not per-account)
```
SpeciesDefinition {
  species_id: UUID (PK)
  display_silhouette_asset: string
  default_palette: ColorRamp
  call_motif_library_ref: string
  is_nightjar_type: bool           // the one species active at full night
}
```

### InteractionEvent (append-only log; source of truth the tick consumes)
```
InteractionEvent {
  event_id: UUID (PK)
  aviary_id: UUID (FK)
  bird_id: UUID | null             // null for aviary-wide events like settle
  event_type: enum(offer_seed, offer_song, offer_pool, listen_in_start, listen_in_end, settle, presence_ping)
  occurred_at: timestamp            // client-reported, server-validated against skew tolerance
  received_at: timestamp            // server clock, authoritative for tick ordering
  metadata: jsonb                   // e.g. listen-in duration once listen_in_end pairs with start
}
```
This table is the only thing a client ever writes to regarding interaction state. It is partitioned by `aviary_id` and the tick consumes it in `received_at` order per aviary — this ordering is what makes additive deltas correct under concurrent multi-device writes (see §5/§6).

### PresenceWindow (derived, computed by the tick from consecutive presence_ping events, not stored as raw client state)
```
PresenceWindow {
  aviary_id, started_at, ended_at, accumulated_seconds
}
```

### NotebookEntry
```
NotebookEntry {
  entry_id: UUID (PK)
  aviary_id: UUID (FK)
  written_at: timestamp
  prose: string                     // generated, naturalist voice
  trigger_event_ids: UUID[]         // provenance, for debugging/QA — never exposed to client
}
```

### VisitInvitation
```
VisitInvitation {
  invitation_id: UUID (PK)
  aviary_id: UUID (FK, host)
  visitor_email_encrypted: bytes
  status: enum(pending, active, revoked, expired)
  created_at, expires_at: timestamp   // expires_at = created_at + 30d
  token_hash: bytes                   // one-time link token, hashed at rest
  last_accessed_at: timestamp | null
  approx_duration_seconds: int        // accumulated, for the host's visit log
}
```

### Session (per-device, revocable)
```
Session {
  session_id: UUID (PK)
  account_id: UUID (FK)
  device_label: string              // user-agent-derived, shown in account settings
  created_at, last_seen_at: timestamp
  revoked: bool
}
```

---

## 4. API surface

All endpoints are under `/api/v1`. Auth via per-device session token (bearer, short-lived access token + refresh, both opaque, both invalidated on revoke). Visit sessions use a separate, narrower-scoped read-only token tied to a `VisitInvitation`.

### State pull
- `GET /aviary/snapshot` — returns the current render-ready snapshot. **Contains derived render parameters, not raw personality values**: per-bird `{ bird_id, name, species_id, perch_zone, mood, pose_seed, call_timing_bias, plumage_render_value, position_interp_target }`. `call_timing_bias` and `plumage_render_value` are server-computed projections of personality (e.g. `plumage_render_value = plumage_saturation`, directly usable by the renderer) — the point is the client never receives a field labeled "boldness: 0.62"; it receives only what's needed to draw the frame, even though some of those numbers are mathematically derived 1:1 from personality. This is the literal implementation of "personality is never exposed numerically": the API contract has no endpoint, field, or debug flag that returns the named trait vector.
  - Also returns: `weather_state`, `time_of_day_phase` (derived server-side from aviary's last-known timezone offset, recomputed per-request to avoid stale day/night on long-idle reconnects), `settled: bool` (aviary-wide), `etag`/`snapshot_version` for client-side change detection.
  - Polling/refresh triggers (client-side logic, not server push): on `visibilitychange` to visible, on long render-frame-gap detection (>2s, suspended-laptop heuristic), and a low-frequency keepalive (every 20s) while visible. No WebSocket/SSE in v1 — snapshot payloads are kilobytes and the tick cadence (60s) makes push infrastructure unjustified complexity for the freshness gain it'd buy.
- `GET /aviary/notebook?cursor=` — paginated, reverse-chronological, read-only.
- `GET /account/sessions`, `DELETE /account/sessions/{id}` — session list/revoke.
- `GET /account/export` — triggers async export job, emails download link; returns 202.
- `POST /account/delete`, `POST /account/delete/cancel` — soft delete / restore within 30-day window.

### Interaction events (write path)
- `POST /aviary/events` — body: `{ event_type, bird_id?, occurred_at, metadata? }`. Validates `event_type` against the closed enum; rejects anything resembling a state mutation. Idempotency key required (client-generated UUID) so retries on flaky connections don't double-count offers or double-pair listen-in start/end.
- Presence pings are batched client-side (one `POST /aviary/events` with `event_type: presence_ping` roughly every 30–60s while the three presence conditions hold continuously) rather than streamed per-second — keeps write volume bounded and matches the "few minutes" granularity the drift function actually needs.

### Auth
- `POST /auth/magic-link` — `{ email }`, rate-limited per email.
- `POST /auth/magic-link/consume` — `{ token }`, issues session, invalidates token atomically (single-use enforced via a DB-level consumed flag with a unique constraint, not a soft check, to close the replay race).
- `POST /auth/logout`.

### Visit flow
- `POST /visits/invite` — host-only, `{ visitor_email }`, issues one-time link, creates `VisitInvitation(status=pending)`.
- `POST /visits/revoke/{invitation_id}` — host-only.
- `GET /visits/consume?token=` — visitor-facing, validates token, issues visit session token, marks `status=active`.
- `GET /visits/snapshot` — visit-token-scoped variant of `/aviary/snapshot`; same payload shape, read-only token can't reach `/aviary/events`.
- `GET /account/visits` — host's visit log.

### Accessibility-supporting endpoints
- `GET /aviary/narration` — server-generated naturalist narration string, regenerated at the same slow cadence as the tick's notebook-eligible output but independently throttled (30–60s) per `accessibility_perf.md`. Returned alongside the snapshot poll rather than a separate channel, to avoid a second connection class — the client's accessibility layer reads `snapshot.narration` and feeds it to a live region.
- Call captions are **not** a separate endpoint: caption text is derived client-side at call-synthesis time from the same motif/parameter data driving the WebAudio graph (per `accessibility_perf.md`, "generated from the procedural call grammar at runtime, not stored as a fixed string"), so captions stay in sync with whatever the audio engine actually plays without a round trip.

---

## 5. Simulation engine design

### Tick loop (runs as a scheduled worker, fan-out per aviary, not per-bird)

Every 60s, for every aviary with at least one event since its last tick **or** whose mood timers warrant a transition even absent new events (time-of-day always advances):

1. **Pull unconsumed events** for the aviary from `InteractionEvent`, ordered by `received_at`, since `last_tick_at`.
2. **Compute presence-time delta**: reconstruct presence windows from consecutive `presence_ping` events (a gap larger than the ping interval + tolerance closes a window). Sum accumulated seconds since last tick.
3. **Compute drift deltas** (additive, never absolute — see §6):
   - `presence_delta = f(presence_seconds)` — dominant weight, diminishing-returns curve (not linear) so an open-all-night tab doesn't dwarf an attentive hour; diminishing returns is also the practical enforcement of "tab open ≠ engagement" at the *magnitude* level even though the binary presence definition already handles it at the *event* level.
   - `listen_in_delta` — per bird focused, weighted by listen-in duration (capped per session to avoid a single long listen-in session producing an outsized one-day jump — this cap is the engineering implementation of "no single session shifts a trait visibly").
   - `offer_delta` — small fixed deltas per accepted offer type, gated by the cooldown (cooldown enforced at write time in the events API, not at tick time, so a spammed offer never even reaches the log as a countable event past the first).
   - All deltas pass through `max(0, delta)` before applying — this is where "drift never moves down on neglect" is enforced as code, not policy: the tick literally cannot subtract from a trait.
   - Deltas are scaled by a global calibration constant tuned to hit the named targets: measurable in instruments after ~1 week of regular visits, visible to users after ~3 weeks. (See §11 for how this gets tuned/tested.)
4. **Apply deltas** to `Bird.personality`, clamped to `[0,1]`.
5. **Advance mood state machine** per bird (see below).
6. **Recompute perch_zone** from updated boldness + mood (boldness biases toward front, wary mood biases toward back; the assignment is a weighted pick, not deterministic, so two birds with similar boldness don't always sit in the same configuration).
7. **Resolve bird-to-bird interaction pass**: after individual mood updates, run one cross-bird pass — wary-mood contagion (a wary bird raises wary-transition probability for birds on adjacent perches), chorus detection (≥2 birds with high `vocal_frequency` and `mood ∈ {content, alert, curious}` in the same tick window flip a transient `in_chorus` flag consumed only by the call-timing renderer hint, not persisted state).
8. **Persist new state**, bump `last_tick_at`, advance `snapshot_version`.
9. **Hand off to Notebook Generator** (see below) with the tick's delta summary and raw events for entry-eligibility scoring.

Tick failures: a single aviary's tick failure (e.g., a malformed event) must not block other aviaries' ticks. The worker processes aviaries independently (queue-per-aviary or partitioned batch with per-item error isolation) and logs/alarms on the failed aviary without blocking the batch. p99 tick latency alarm at 5s per `accessibility_perf.md`.

### Mood transition model

Mood is a weighted state machine, not a lookup table, because four independent signals (recent interactions, time of day, ambient weather, personality) all need to combine into one transition probability per tick:

```
mood_transition_score(bird, signals) =
    w_interaction * recent_interaction_bias(bird, signals.recent_events)
  + w_time        * time_of_day_bias(signals.local_hour, mood_target_for_hour)
  + w_weather      * weather_bias(signals.weather_state)
  + w_personality  * personality_resistance(bird.personality, current_mood)
```
Each bias term nudges toward a candidate target mood; `personality_resistance` is the term that makes a high-boldness bird resist sliding into `wary` on the same input that would tip a low-boldness bird — implemented as a per-trait multiplier on the wary-target bias specifically (boldness dampens wary-bias; social_warmth dampens contagion susceptibility; vocal_frequency raises chorus-join probability; curiosity raises offer-approach probability). The transition fires probabilistically (weighted random, not a hard threshold) so the same inputs don't always produce the same output — this is part of "calls vary every time" applied to mood, not just audio.

Mood persists across sessions by construction: it's read from `Bird.mood` at the start of every tick, never reset to a default. A bird's mood at session-end is whatever the last tick wrote; the next tick (which has run on schedule regardless of client connection) picks up from there.

### Drift calibration as a tunable, testable surface

The global calibration constant in step 3 is the single knob that determines whether the product feels like a Tamagotchi (too fast) or a screensaver (too slow). It ships as a config value, not a magic number buried in the delta math, and the simulation worker exposes an instrumented "synthetic aviary" harness (constant simulated presence input) used in CI to assert the 1-week-instrument / 3-week-visible targets against a fixed seed, so calibration drift is caught in CI rather than discovered by users. "Visible to users after 3 weeks" is operationalized as: the rendered `plumage_render_value` and `call_timing_bias` deltas exceed a perceptual-difference threshold (established via design review against the rendering spec) over a 3-week synthetic run.

### Call-grammar runtime (client-side, driven by server-supplied bias parameters)

The call synthesis engine itself runs entirely client-side (WebAudio), per `accessibility_perf.md` and `bird_engine.md`. The server's role is limited to supplying, per snapshot, the parameters that shape — not generate — the call: `call_timing_bias` (derived from `vocal_frequency`), `mood` (shapes pitch/timbre selection within the species' motif library), and the transient `in_chorus` hint. The client's WebAudio engine:
- Loads each adopted bird's `species_id → motif_library_ref` once (small, code-split, loaded lazily as birds are adopted — not all 6 species' motif libraries upfront, only the ones the user's aviary actually has).
- On each call trigger (timer driven by `call_timing_bias` + mood + a randomization jitter so timing is never metronomic), selects and combines 2–4 motifs from the library with personality/mood-shaped pitch and tempo variance, synthesizes via oscillator/noise nodes + envelope, never replays a stored buffer verbatim.
- Mixes multiple simultaneous bird calls through a single `AudioContext` graph so true real-time phase interaction occurs (this is what avoids the "stacked recorded loops" phase-cancellation artifact named in `bird_engine.md` — it's a direct consequence of synthesizing into a shared graph rather than rendering each bird to an independent buffer and summing).

---

## 6. Sync model

Multi-device sync is **not** a synced-state feature; it is a read-many/write-one architecture, restated here as concrete mechanics:

- **Single source of truth**: `Bird.personality` and `Bird.mood` live in one row per bird in the canonical Aviary State DB. There is no per-device copy.
- **Writers**: exactly one writer process class — the Simulation Tick Worker. No API endpoint, no client SDK path, no admin tool, writes directly to `personality` or `mood` fields. This is enforced at the schema/permissions layer (the tick worker's DB role has UPDATE on these columns; the API service's DB role does not).
- **Client writes go through the event log only**, and the event log is append-only (INSERT-only permissions for the API write path; no UPDATE, no DELETE on `InteractionEvent` from the API role). This makes the "additive deltas, never absolute values" rule physically true, not just conventionally true — a compromised or buggy client literally cannot send `{boldness: 0.62}` because no write path accepts a personality field.
- **Conflict resolution**: there is no conflict to resolve in the traditional sense, because there are no concurrent writers to the same canonical field. Two devices both writing `presence_ping` and `offer` events concurrently just produce two interleaved rows in the append-only log, ordered by `received_at`; the tick consumes them in that order and the deltas are additive, so interleaving order doesn't change the result (commutative under addition) — this is explicitly why additive-only deltas were chosen over any operation that isn't order-independent.
- **Snapshot freshness across devices**: each device polls its own snapshot independently; there's no cross-device push. A user actively watching on a laptop while picking up their phone will see the phone catch up to the same canonical mood/position within one poll cycle (≤20s keepalive) — not instantly, and that's an accepted tradeoff given the 60s tick cadence makes sub-20s freshness unnecessary.
- **Session-level conflicts** (not personality conflicts): magic-link replay, in-flight session timeout — these are auth/session issues, not state-sync issues, and are surfaced via the matter-of-fact error surfaces specified in `accounts_sync.md`, not via any "merge" UI (there is nothing to merge).

---

## 7. Frontend rendering pipeline

### Stack choice

Canvas2D for the scene (not WebGL, not DOM/CSS-animation-driven), for three reasons tied directly to the budgets in `accessibility_perf.md`: (a) Canvas2D ships with zero additional runtime bundle weight versus a WebGL/three.js dependency, which matters hard against the 2MB cap; (b) the scene's visual complexity (procedurally-composed sprite/vector birds, simple parallax, soft color fields) doesn't need a 3D pipeline; (c) Canvas2D's imperative draw model makes the "first frame has motion already in progress" requirement straightforward — there's no framework mount/hydration sequence between snapshot arrival and pixels on screen.

### Scene composition

- Render loop driven by `requestAnimationFrame`, structured as the three layers from §2 (simulation-driven, ambient ornament, environment), composited back-to-front each frame: environment → background foliage/parallax → birds/perches → foreground ornament (passing leaf/branch).
- Bird rendering: each bird is a small state machine of pose-sprites (procedurally composed from a base skeleton + species palette, not one bitmap per pose-per-species, to keep the asset budget down) driven by `mood` + idle-motion timers (preen, scan, head-tilt, weight-shift) layered with the server-supplied position/pose targets, interpolated via easing between consecutive snapshots.
- Position interpolation: a bird moving from perch A (snapshot N) to perch B (snapshot N+1) is animated as a flight arc over a few seconds client-side, not a teleport — the client owns the *tween*, the server owns the *target*.

### First-frame requirement (no spinner, no fade-from-static)

Implementation: the initial HTML response includes the first snapshot inlined (small JSON payload server-rendered into the page, not a separate fetch-then-render round trip) so the very first JS execution has state to draw immediately — this is what makes <500ms time-to-first-bird achievable, since it removes one network round trip from the critical path. If the inlined snapshot is stale (cache edge case) or absent (cold cache), the loading state is the specified "quiet field" — a soft sky-color canvas fill with one or two faint motion cues — never a spinner component. This quiet-field state is itself just an early frame of the same render loop (environment layer only, simulation layer pending), not a separate component swapped out later — avoiding any seam between "loading" and "loaded."

### Empty-aviary state

Same quiet-field rendering, held until the post-adoption first-bird-assigned event arrives, then the first bird's entry is rendered as a scripted fly-in (a one-time client-side animation, not server-driven) to its starting perch. This is the one place a "fly-in" entrance animation is allowed — explicitly for the one-time adoption moment, not for returning sessions.

### Reduced-motion mode (separate render path, not a flag on the same path)

Implemented as a distinct renderer module sharing the same simulation-state input but a different draw strategy: pose cross-fades (timed opacity blends between discrete pose keyframes) instead of continuous procedural motion; flight transitions render as cross-fades between perch positions instead of arcs; ambient leaf/feather ornament layer is disabled entirely; day/night palette shifts remain but are slowed (longer cross-fade duration). Triggered by `prefers-reduced-motion` media query by default, overridable (in both directions) via an explicit accessibility-settings toggle so a user can opt in even without the OS-level flag, or opt out if they prefer full motion despite the OS flag (the PRD specifies opt-in via settings as an addition to the media query, not a replacement).

### Top bar

Separate DOM layer (not canvas-rendered) over the canvas scene, for accessibility (real focusable DOM elements, not canvas hit-testing) and for the CSS-opacity-driven idle fade (a few seconds of cursor/keyboard stillness → fade near-transparent; movement/keypress → restore). Contains exactly four icons per `aviary_layout.md`: account/settings, accessibility settings, field notebook, offer. No other chrome.

### Code splitting (budget enforcement)

Critical path bundle = scene renderer + snapshot fetch/interpolation + WebAudio call engine core + top-bar shell. Lazy-loaded on demand: account settings, accessibility settings panel, visit-invitation flow, field notebook entry list (loaded when the notebook icon is opened, not preloaded), per-species motif libraries beyond the user's current adopted species. This split is what keeps the always-loaded critical path under budget while the full feature surface stays under the 2MB cap in aggregate.

---

## 8. Audio pipeline

- **Single shared `AudioContext`** per session, created on first user gesture if the browser requires it (autoplay-policy handling — if no gesture has occurred yet, the engine queues calls silently and a one-time, unobtrusive "tap to enable sound" affordance appears in the top bar only if needed, not as a blocking modal).
- **Per-bird call scheduler**: each bird has an independent timer (jittered interval derived from `vocal_frequency` + mood) that, on fire, requests a motif sequence from that species' motif library, applies personality/mood-shaped pitch/tempo/timbre parameters, and schedules oscillator/noise/envelope nodes onto the shared graph.
- **Chorus mixing**: because all active calls share one `AudioContext` graph, simultaneous calls sum and interact in real time (true chorus, not pre-mixed stems) — this directly satisfies the "two procedural calls mixed at runtime, not stacked loops" requirement.
- **Listen-in mix**: implemented as per-bird gain nodes; engaging listen-in ramps the focused bird's gain up and all others' gain down via `GainNode.linearRampToValueAtTime` over ~1–2s (slow, not a hard cut, per `interactions.md`); disengaging ramps symmetrically back to ambient baseline. Others' gain floor is clamped above zero — never fully muted.
- **Buffer/node reuse**: oscillator nodes are short-lived (per the no-memory-growth budget, nodes are created and disposed per call, not pooled indefinitely — but envelope/gain node *templates* and the motif-library data itself are loaded once and reused, and the audio graph topology (gain nodes per bird) is created once at session start and reused for the session, not recreated per call).
- **Caption generation**: at the moment a call is scheduled, the same parameters that drive synthesis (motif selection, mood, pitch contour) are passed through a small text-template function that produces the caption ("a soft three-note rise") — guaranteeing caption-audio match without a separate content pipeline.
- **WebAudio fallback**: if `AudioContext` construction throws or remains in a permanently suspended state after a user gesture, the engine flips to a silent mode and force-enables captions (overriding the user's caption preference to "on" for that session, since silence-with-no-captions would be a worse failure than overriding a default) — per `accessibility_perf.md`'s explicit "no recorded-audio fallback, ever" rule.

---

## 9. Accessibility surfaces

- **Screen-reader narration**: a visually-hidden ARIA live region (`aria-live="polite"`, `aria-relevant="additions"`) updated with naturalist prose pulled from `snapshot.narration`. Update cadence matches the server's narration regeneration (30–60s idle, faster on user-initiated events — return-greeting and offer-reaction narration are pushed with the priority bump described in §4, meaning the client checks for a "priority" flag on the narration payload and, if set, uses `aria-live="assertive"` for that single update only, then reverts to polite). The narration text is never auto-generated client-side from raw state — it's the same server-authored prose the visual/notebook surfaces use, so voice is identical across modalities (explicit requirement in `accessibility_perf.md`).
- **Captions**: rendered as small fading DOM text positioned near the calling bird's canvas coordinates (canvas position translated to DOM overlay coordinates each frame the caption is visible), using the same naturalist phrase generated in §8.
- **Keyboard navigation**: Tab order = top-bar icons (in visual order) → aviary scene (Tab into the scene focuses the first bird by current front-to-back perch order) → arrow keys move focus laterally between birds (left/right within a perch zone, up/down between zones) → Enter triggers listen-in on focused bird → Escape exits listen-in and returns focus to the previously-focused element. Offer affordance: opens via top-bar activation (Enter/Space on the icon), then the offer panel itself is a standard focusable list (seed/song/pool), keyboard-dismissible via Escape. Settle: reachable and triggerable via the top-bar settle icon, standard activation.
- **Focus indicator**: a high-contrast outline (exact treatment from the design system, but functionally: sufficient luminosity contrast against both the brightest daytime palette and the darkest night palette — likely a two-tone outline (dark core + light halo or vice versa) rather than a single color, since no single outline color survives both extremes of a calm naturalist palette).
- **Contrast**: all top-bar/settings/account/error/caption/narration-when-visually-displayed text is checked against WCAG AA in the design system's component library; this is enforced via automated contrast-checking in the visual regression/CI suite for chrome components (the aviary canvas scene itself is exempt since it carries no required-reading user copy per the PRD).
- **Reduced-motion as default product quality bar**: QA explicitly tests the reduced-motion render path as a first-class surface (not a "does it also work" afterthought) — same acceptance bar as the full-motion path for "feels alive," verified via design review, not just a functional toggle check.

---

## 10. Performance budgets and observability

| Budget | Target | Primary lever |
|---|---|---|
| Initial JS bundle | ≤2MB gzipped | Canvas2D over WebGL; code-splitting settings/visit/notebook surfaces; procedural audio (no audio files); lazy per-species motif loading |
| Time to first bird visible | <500ms, mid-tier mobile/4G | Snapshot inlined in initial HTML; render loop starts on first paint with no fetch-then-render gap; CDN-edge delivery of initial HTML+snapshot |
| Idle motion frame rate | 60fps sustained, 5-year-old laptop, 30-min session | Canvas2D with minimal per-frame allocation; pose-sprite compositing instead of per-frame procedural geometry generation; rAF loop profiled in CI on a throttled-CPU headless browser |
| Memory growth | None over 30 min | Oscillator/audio-node disposal per call; notebook entries virtualized (DOM nodes for scrolled-out entries released, not retained); bounded worker/AudioContext count (one shared context, no per-call context creation) |

**Observability** (aggregate-only, per the privacy boundary in `accounts_sync.md` — never per-bird/per-account state):
- Synthetic checks: scheduled headless-browser runs from multiple geographies hitting `/aviary/snapshot` and measuring full page-load-to-first-bird-paint.
- RUM: page load timing, first-bird-render timing, render-frame timing histograms, audio-context error counts — all anonymized, no per-account dimension.
- Simulation-tick latency: p50/p99 tracked per tick batch (not per account); **p99 alarm at 5s** as specified.
- Error budgets surfaced on an ops dashboard separate from any product-facing surface; none of this data is queryable by account or bird, enforced by the telemetry pipeline never having read access to the Aviary State DB or Event Log (it reads only from a separate, pre-aggregated metrics stream the services emit).

---

## 11. Rollout

### Phasing

1. **Foundation** (engine-first, not UI-first): Aviary State DB schema, Simulation Tick Worker with the synthetic-aviary calibration harness, event log + API write path, auth (magic-link). Ship-gate: synthetic aviary hits the 1-week-instrument / 3-week-visible drift targets in CI before any client work begins, since getting this wrong invalidates everything downstream.
2. **Core client**: snapshot pull/render loop, Canvas2D scene with the two starter species, procedural audio engine for those species, return-greeting, listen-in, settle. Ship-gate: <500ms time-to-first-bird and 60fps idle on target hardware, measured before adding remaining species/features, since these are infrastructural and easier to fix before more surface area is built on top.
3. **Full interaction surface**: offer (all three types), field notebook, remaining 4 species, age-gated bird unlocks up to 7.
4. **Accessibility surfaces built in parallel with phase 2–3**, not after — per `accessibility_perf.md`'s explicit instruction that this must ship with v1, not as a fast-follow. Narration, reduced-motion render path, and captions are developed against the same milestones as their full-motion/audio counterparts, with their own acceptance review.
5. **Accounts/sync hardening + social (visits)**: multi-device session list/revoke, account export/deletion, visit invitation flow. Land last because they're additive to a working single-device single-user core and have no engine dependency.
6. **Launch readiness**: synthetic perf monitoring live, RUM live, error-budget alarms wired, browser-support matrix (last 2 versions of Chrome/Safari/Firefox/Edge) verified, unsupported-browser matter-of-fact surface in place.

### Ramping birds-per-aviary

The age-gated unlock schedule (third bird at "a few months," growing to five-or-six by a year) is a config table (`aviary_age_days → max_unlockable_birds`) read by a lightweight eligibility check, not hardcoded logic — this makes the pacing curve tunable post-launch (e.g., if early cohort data suggests the curve is too slow/fast to keep the relationship-deepening feel right) without a deploy touching engine code, only a config change.

### Day-one instrumentation

Aggregate RUM and synthetic checks (§10) go live at launch, not added later. Simulation-tick latency alarms active from day one. No product-usage analytics beyond the aggregate operational categories explicitly allowed in `accounts_sync.md` — there is no "day-one engagement dashboard" because the PRD forecloses the entire category of per-account behavioral aggregation.

---

## 12. Risks

- **Drift calibration is the single highest-risk unknown.** The 1-week-instrument / 3-week-visible target is a felt-experience claim, not a number anyone can derive analytically — it has to be tuned against real cohort behavior, and the only way to validate it before launch is the synthetic-aviary CI harness plus a private beta cohort with informed instrumentation review (still subject to the privacy boundary — beta drift validation uses opt-in cohort accounts whose interaction data is reviewed under the same access controls as any other account, not a special "ML training" exception). Mitigation: ship the calibration constant as a single tunable config, gate launch on the CI harness passing, and treat the constant as expected to need at least one post-launch adjustment based on real engagement patterns — but adjusting it must never become a way to quietly make the product more "engaging" in the gamification sense; it tunes felt-aliveness pacing only, and any proposed adjustment is reviewed against the non-goals file before shipping.
- **Sync correctness depends on disciplined enforcement of "server is the only writer," and that discipline is easy to erode under deadline pressure** (a future contributor adding a "quick fix" endpoint that writes mood directly to fix a support ticket). Mitigation: enforce at the DB permission layer (API service role has no UPDATE grant on `personality`/`mood` columns), not just at the code-review layer, so the rule survives even a rushed change.
- **Audio uncanniness**: procedural synthesis that doesn't sufficiently vary, or that produces audibly similar output to itself across calls, fails the "looped audio is the audible signature of dead software" bar even if it's technically "procedural." Mitigation: the motif-combination + pitch/tempo jitter parameters need real listening-test review (not just a "it's procedural so it's fine" checkbox) before each species ships, and recognizability-across-mood/drift (a user should still know Pip's call after weeks of drift) needs its own design-review pass per species, since the engine could technically satisfy "procedural" while still drifting a bird's call signature into unrecognizability if motif-selection weights aren't bounded correctly.
- **Accessibility regressions from feature velocity**: because accessibility surfaces (narration, reduced-motion render path, captions) are full parallel implementations rather than ARIA-label automation, they carry real ongoing maintenance cost — every new bird behavior, mood, or interaction needs a narration phrase and a reduced-motion pose set, not just a visual implementation. Mitigation: treat "narration phrase set" and "reduced-motion pose set" as required deliverables in the definition-of-done for any new bird behavior or species, not an accessibility-team follow-up ticket.
- **Performance budget erosion**: the 2MB/500ms/60fps budgets are easy to violate incrementally (one more species' motif library, one more settings surface inlined instead of split) with no single change being the obvious culprit. Mitigation: bundle-size and time-to-first-bird checks gate CI on every merge (hard failure, not a warning), not just at major milestones.
- **Gamification creep via "harmless" feature requests** is named explicitly in the PRD as the most predictable failure mode, and it's a process risk as much as a technical one. Mitigation: any feature proposal touching visit-frequency, streaks, counters, or achievement-shaped UI gets routed through an explicit non-goals review against `non_goals.md` before implementation starts, not just at code review — the refusal has to happen at the planning stage, where it's cheap, not after a sprint's been spent building it.
- **Visit feature scope creep toward co-presence**: because read-only ambient visits are a deliberately constrained subset of what a "social" feature could be, there will be repeated, reasonable-sounding pressure to add small co-presence touches (a viewer-count, a "someone's watching" indicator). Mitigation: explicitly out of scope per `social_optional.md`'s reasoning — any such request needs a structural redesign of the simulation (multi-source presence/drift), which this plan does not provision for, and should be treated as a different product decision, not a small addition.
