# PLAN.md — Pocket Aviary v1 Implementation Plan

This plan interprets the PRD into an executable program of work for a frontier engineering team. It does not restate the spec; it makes load-bearing calls and pins down boundaries the PRD intentionally leaves to implementation.

Where the PRD is ambiguous, this plan picks a defensible default and notes it inline as **(call)**.

---

## 1. Scope

### In scope for v1

- Single-user accounts; magic-link email auth; per-device sessions; account export; soft-then-hard delete.
- One canonical aviary per account, 2 starter birds, cap of 7.
- Server-authoritative simulation tick (~60s cadence).
- Personality vector with monotonic-expressive drift; mood with daily-ish reset; procedural calls; idle micro-motion.
- Single horizontal scene with three perch zones, day/night by user-local time, ambient weather, leaf/feather drift, top-bar fade.
- Interactions: return-greeting, listen-in, offer (seed / song-fragment / still-pool), settle (with 5s undo), field notebook (read-only).
- Multi-device sync via canonical-state model (no client-to-client sync).
- Visit feature: per-invite opt-in, read-only ambient, revocable, 30-day expiry, host-side visit log, off-by-default visit-notification toggle.
- Accessibility as designed surface: naturalist screen-reader narration, designed reduced-motion mode, procedural call captions, full keyboard nav, WCAG AA contrast.
- Performance: ≤2 MB initial JS (gzipped); first-bird ≤500 ms p75 mid-tier-mobile/4G; 60 fps idle on a 5-year-old laptop; flat-memory 30-min sessions.
- Aggregate-only operational telemetry; no per-bird/per-account state in analytics.
- Browser support: latest two majors of Chrome, Safari, Firefox, Edge.

### Out of scope (PRD-named, planned-around)

Native apps; gamification of any flavor (achievements, streaks, levels, scores, badges, "days visited" surfaces); Tamagotchi mechanics (death, hunger, distress, decay-on-neglect); social network surfaces (profiles, follows, feeds, comments, leaderboards, discovery); push notifications; payments; multi-aviary accounts; shared aviaries; public discovery; recorded-audio fallback; animated entry/loading transitions.

These are not just deferred features — they are architectural absences. The data model and the metrics pipeline are designed so these features are *harder to add later*, not easier.

### Anti-features explicitly designed against

The plan must, at design-review time, fail the following checks:

- No string anywhere in the codebase containing `welcome back`, `streak`, `days visited`, `level up`, `achievement`, `badge`, `xp` (excluding tests that assert their absence).
- No code path that surfaces personality vector values to the client.
- No code path that writes personality vectors from the client.
- No telemetry event with a per-bird or per-account dimension beyond the synthetic UUID needed for operational error correlation (and even that bounded; see §11).

---

## 2. Architecture

### 2.1 Service shape

Five logical services. Two are load-bearing; three are conventional.

1. **Edge / Web** (Node, Cloudflare-or-equivalent edge). Serves HTML, the JS bundle, and the *initial state snapshot inlined into the HTML response*. The inline-snapshot trick is what makes the 500 ms first-bird budget reachable; see §10.
2. **Auth service**. Magic-link issuance, token validation, session management. Stateless beyond a small token store. Owns the email-to-UUID mapping (the only place email is stored).
3. **Aviary API** (REST + WebSocket fallback to long-poll). Read endpoints serve state snapshots; write endpoints accept interaction events into the append-only event log; visit endpoints handle invites and read-only state for visitors. **Never** writes personality vectors.
4. **Simulation service**. Owns the tick. Pure consumer of the event log + previous state; pure writer of canonical aviary state. Single writer per account (sharded by account UUID). The only process in the system that mutates personality vectors.
5. **Notebook service**. Generates and stores notebook entries. Read-only to clients. Listens to canonical state changes; emits naturalist-prose entries on a sparsity-controlled schedule.

Two ancillary stores: an **email service** (transactional outbound for magic links, exports, invitations) and a **CDN** (bundle, sprites, motif library — though motifs are tiny and often inlined).

### 2.2 Client/server split — the load-bearing line

- Server owns: personality vectors, mood state, simulation time, notebook entries, account records, sessions, invitations, the event log.
- Client owns: rendering state, audio mix state, focus/listen-in target, reduced-motion preference (mirrored from settings), input event capture, *transient* presence-event accumulation that is flushed to the server.

The client never owns canonical anything. This is restated because every casual feature ask will try to nudge it.

### 2.3 Render-pipeline boundary

Client receives **state snapshots** (small JSON, typically <8 KB), not animation timelines. The renderer translates a snapshot into:

- A target perch position per bird with a *transition mode* (cross-fade in reduced-motion; animated path otherwise).
- A target idle-motion preset per bird, keyed by mood + personality.
- A current call schedule per bird (next-call window + grammar seed; client synthesizes within that window).
- Scene state: time-of-day phase, current weather, ambient-particle seed.

Two snapshots, plus interpolation between them, are enough to render any frame. The client never extrapolates beyond the next snapshot's expected arrival; on stale snapshots, it holds last-known and quietly catches up on resume.

---

## 3. Data Model

All IDs are synthetic UUIDs (v7 — time-ordered for index locality). Email is stored once on the account record, encrypted at rest, never used as a key.

### 3.1 Account

```
account {
  id: uuid (PK, partition key everywhere)
  email_encrypted: bytes
  email_lookup_hash: bytes  // HMAC for sign-in lookup; not reversible
  created_at: timestamp
  deletion_marked_at: timestamp | null   // for soft-delete window
  visit_notifications_enabled: bool      // off by default
  reduced_motion: bool | null            // null = follow OS pref
  captions_enabled: bool
  audio_enabled: bool
  high_contrast: bool
  privacy_policy_version_acked: string
}
```

### 3.2 Bird

```
bird {
  id: uuid (PK; immutable for life of bird)
  account_id: uuid (FK)
  species_id: enum (~6 species)
  name: string (user-editable)
  adopted_at: timestamp
  // Personality vector — only the simulation service writes these.
  trait_boldness: float     // [0.0, 1.0]
  trait_social_warmth: float
  trait_vocal_frequency: float
  trait_plumage_saturation: float
  trait_curiosity: float
  // Fast-timescale state.
  mood: enum {wary, content, curious, drowsy, alert}
  mood_entered_at: timestamp
  // Render hints — derived, but persisted to keep snapshots cheap.
  current_perch: enum {front, middle, back}
  last_call_at: timestamp
  next_call_window_start: timestamp
  next_call_window_end: timestamp
  // Drift bookkeeping.
  presence_seconds_consumed: int64  // cumulative, monotonic
  last_drift_applied_at: timestamp
}
```

Personality fields are stored as floats but **never serialized to any client surface** (see §4 for snapshot shape). The schema enforces this with a separate snapshot projection.

### 3.3 Event log (append-only)

```
event {
  id: uuid (v7, time-ordered)
  account_id: uuid
  bird_id: uuid | null  // null for aviary-wide events
  type: enum {
    presence_ping, listen_in_start, listen_in_end,
    offer_seed, offer_song_fragment, offer_still_pool,
    settle_start, settle_undo, session_start, session_end,
    weather_observed_rain, weather_observed_wind  // server-emitted, but flow through same log
  }
  occurred_at: timestamp
  client_seq: int  // for idempotency from a single client
  payload: jsonb   // small, type-specific
}
```

Append-only. The simulation tick consumes events in `(account_id, occurred_at, id)` order and marks a high-watermark per account. Events older than 90 days are aggregated into a per-bird drift summary and pruned (PRD doesn't specify retention; **(call)** 90 days balances drift transparency with storage).

### 3.4 Notebook entry

```
notebook_entry {
  id: uuid
  account_id: uuid
  written_at: timestamp
  prose: string    // naturalist voice; ≤280 chars typical
  trigger_kind: enum {greeting_pattern, mood_persistence, weather, drift_milestone, quiet_session, sparse_default}
}
```

The trigger_kind exists for QA (so we can verify mix), not for client display.

### 3.5 Invitation / Visit

```
invitation {
  id: uuid
  host_account_id: uuid
  visitor_email_hash: bytes   // hash for revocation lookup
  visitor_email_encrypted: bytes
  token: opaque (random, 128-bit)
  created_at: timestamp
  expires_at: timestamp        // created_at + 30d
  revoked_at: timestamp | null
  consumed_first_at: timestamp | null
}

visit {
  id: uuid
  invitation_id: uuid
  started_at: timestamp
  ended_at: timestamp
  approximate_duration_seconds: int   // bucketed; not exact
}
```

Visitor sessions never write events to the host's event log.

### 3.6 Session

```
session {
  id: uuid
  account_id: uuid
  device_label: string   // user-editable in settings
  created_at: timestamp
  last_seen_at: timestamp
  revoked_at: timestamp | null
}
```

### 3.7 Storage choices

- Account / Bird / Session / Invitation / Notebook entries → Postgres, sharded by `account_id` (citus or app-level shard map). One canonical writer per shard for the simulation tick.
- Event log → append-only table with `(account_id, occurred_at)` partitioning, monthly partitions, retention as above. Could also be Kafka topic + materialized table; **(call)** start with Postgres partitioned table — operational simplicity wins until volume forces otherwise.
- Email-to-account lookup → separate small table indexed by `email_lookup_hash`.

---

## 4. API Surface

REST + JSON. WebSocket only as an optimization for snapshot freshness; everything also reachable over plain HTTPS for the long-poll fallback.

### 4.1 Auth

- `POST /auth/magic-link`  body: `{email}`  → 202 always (no enumeration).
- `GET /auth/consume?token=…` → sets session cookie; redirects to `/`.
- `POST /auth/sign-out` → revokes current session.
- `GET /account/sessions` → list of sessions (device label, last_seen).
- `POST /account/sessions/:id/revoke`.

### 4.2 Aviary

- `GET /aviary/snapshot` → state snapshot (see §4.5). Supports `If-None-Match` etag.
- `POST /aviary/events` → batch of client-originated events. Idempotent on `(session_id, client_seq)`. Body bounded in size; server rejects events beyond a small batch size (**(call)** 32) to prevent flooding.
- `WS /aviary/stream` → server pushes snapshots when state changes meaningfully (mood transition, perch change, weather change, settle ack). Heartbeats double as keepalive presence pings under §6.
- `GET /aviary/notebook?cursor=…` → paginated notebook entries (newest first).
- `PATCH /aviary/birds/:id` → rename only. Body: `{name}`. Returns 200; rename does not enter event log (no drift impact).

### 4.3 Account

- `GET /account` → profile, settings, deletion status.
- `PATCH /account` → settings (visit notifications, accessibility prefs, audio default, etc.).
- `POST /account/email-change` → starts verification of new email.
- `POST /account/export` → enqueues export; emailed when ready.
- `POST /account/delete` → marks soft-delete; immediate UI lockdown but data preserved 30 days.
- `POST /account/undelete` → during the 30-day window.

### 4.4 Visits

- `POST /invitations` body: `{visitor_email}` → returns invitation id (no token; token is in email).
- `GET /invitations` → host's outstanding + recent invitations, with state.
- `POST /invitations/:id/revoke`.
- `GET /visit/:token` → public-ish endpoint; if valid, sets a short-lived visitor session cookie (no account binding).
- `GET /visit/snapshot` (visitor session) → snapshot for the host's aviary, *projected* (see §4.5).
- `GET /account/visit-log` → host-visible visit log.

### 4.5 Snapshot shape

Two projections, identical fields except where noted. **No personality vector values appear in either.**

```
{
  "snapshot_id": "uuid",
  "server_time": "ISO8601",
  "tick": int,                  // monotonic per account
  "scene": {
    "time_of_day_phase": "predawn|dawn|morning|midday|afternoon|evening|dusk|night",
    "weather": {"kind": "clear|rain|wind", "intensity": 0..1, "until": "ISO8601"},
    "ambient_seed": uint32      // for client-side leaf/feather PRNG
  },
  "birds": [
    {
      "id": "uuid",
      "name": "string",
      "species": "string",
      "perch": "front|middle|back",
      "mood": "wary|content|curious|drowsy|alert",
      "next_call_window": ["ISO8601","ISO8601"],
      "call_grammar_seed": uint32,
      "render_hints": {           // categorical only, never numeric trait values
        "approach_eagerness": "low|medium|high",
        "vocal_density": "sparse|moderate|frequent",
        "plumage_tier": 1|2|3|4,
        "idle_motif": "preening|scanning|head_tilt|fluffed|bouncing"
      }
    }
  ],
  "host_session": { ... } | null  // omitted on visitor projection
}
```

The `render_hints` are coarsened, named buckets — they expose enough to drive rendering without exposing the underlying floats. A user sniffing the network sees `"plumage_tier": 3`, not `"plumage_saturation": 0.624`. This is the implementation of the "personality is never numeric to the user" rule at the protocol level.

### 4.6 Visit flow

1. Host posts `visitor_email`. Server creates `invitation` with random token; emails the visitor a link `/visit/<token>`.
2. Visitor opens link. Server checks token, expiry, revocation. If valid, creates short-lived **visitor session** (no account bind, ~2h max), returns the read-only client.
3. Visitor client polls or WebSockets `/visit/snapshot` for fresh state. The endpoint returns the host's current state snapshot, projected to remove host_session and any host-only fields.
4. On revoke, the next snapshot pull returns 410 Gone with the matter-of-fact "visit no longer available" message.
5. Visit duration is recorded on the host side, bucketed (e.g., to nearest minute) — **(call)** to discourage exact stalking.

### 4.7 Error responses

JSON envelope:

```
{ "error": { "code": "magic_link_expired", "message": "...", "matter_of_fact": "We couldn't sign you in. The link may have expired. Try requesting a new link." } }
```

The client renders `matter_of_fact` text verbatim on system surfaces; the `code` is for instrumentation only.

---

## 5. Simulation Engine

The simulation tick is the heart of the product working at all. It must be deterministic given the same inputs (replayability for debugging), single-writer per account (no merge), and bounded in cost (p99 latency budget 5 s).

### 5.1 Tick cadence

- Default cadence: **once every 60 seconds per account**, *only if there are unprocessed events or pending mood/weather timers*. Idle accounts (no events for ≥1 hour and no pending timers) drop to a 5-minute heartbeat tick.
- This idle dropoff makes total system load bounded by active accounts × 1/min, not all accounts × 1/min.
- The tick is enqueued via a per-account-shard scheduler; only one tick per account in flight at a time.

### 5.2 Tick algorithm

```
tick(account_id):
  state = load_canonical_state(account_id)
  events = load_events_after(account_id, state.event_watermark)
  presence_seconds = compute_presence_seconds(events)
  for each bird:
    apply_drift(bird, presence_seconds, events)
    update_mood(bird, events, time_of_day, weather)
    update_perch(bird)
    schedule_next_call(bird)
  update_weather(state)
  state.event_watermark = events.last.id
  save_canonical_state(state)
  emit_render_hints(state)
  maybe_write_notebook_entry(state, events)
```

Each substep is small, pure, and individually unit-testable.

### 5.3 Drift function

Drift is a low-pass filter over presence-time and selected interactions. Specifically, per trait, per tick:

```
delta = clamp(
  k_presence * presence_seconds_in_tick * trait_responsiveness[trait]
  + k_listen * listen_in_seconds_for_this_bird
  + k_offer  * offers_to_this_bird,
  0, max_delta_per_tick
)
trait += delta * trait_responsiveness_curve(current_value)
trait = clamp(trait, 0, 1)
```

- `delta` is **always non-negative**. The clamp at 0 is the asymmetry rule made code; never a `-=` anywhere. A static-analysis CI rule rejects negative coefficients in this function.
- `trait_responsiveness_curve` is a saturating function so traits move fastest in the mid-range and barely at all near 1.0 (so birds asymptote rather than max out instantly).
- `max_delta_per_tick` is small enough that no single tick visibly moves a trait to the user — calibrated so cumulative motion across a *typical* week of regular visits is just-detectable in instruments and three weeks is just-visible to a user. Initial calibration values **(call)**:
  - For the calibration target "≈3 weeks visible" with ≈30 min/day presence, target ~+0.20 cumulative on the dominant trait. That gives ~+0.001 per minute of presence at mid-range, scaled by `trait_responsiveness`. We will fit these constants against the simulation harness in §13 before launch and tune from telemetry-free synthetic data after.

### 5.4 Mood transitions

Mood is a finite state machine with weighted transitions:

- Triggers: time-of-day boundaries; weather start/end; offer accepted/declined; another bird's alarm-call; chorus event nearby; sustained listen-in.
- Transition weights are personality-shaped: `P(enter wary | trigger=alarm) = base − α * boldness`.
- Mood persists across sessions: end-of-tick mood is start-of-next-tick mood. The server has no concept of "session"; only presence windows.
- Mood timers (e.g., "drowsy lasts ~20–40 min") are stored on the bird and decremented per tick.

Five mood states per PRD: `wary`, `content`, `curious`, `drowsy`, `alert`. Transition table is data, not code, and lives in `simulation/mood_transitions.json` under version control with calibration tests.

### 5.5 Call grammar runtime

Call generation is split:

- **Server**: schedules the *next* call window per bird per tick (`window_start`, `window_end`, `grammar_seed`). Windows respect vocal-frequency trait, mood, weather, time-of-day. Two birds whose windows overlap form a chorus event.
- **Client**: at the scheduled window, generates the actual audio waveform from the species' motif library + grammar seed via WebAudio. This means: server doesn't ship audio; client doesn't decide *when* a bird calls.

Motif library per species: a small set of motifs (rises, trills, single notes, two-note descents) plus connection rules. Personality and mood select from the motifs and shape timing/pitch — e.g., a high-vocal-frequency bird in `content` mood emits longer chains; the same bird in `wary` mood emits short single calls.

Recognizability discipline: each species has at least one **identifying motif** that always appears with high probability in the first call after a perch change. The user's ear locks onto species through repeated re-anchoring of that motif. We reserve at most ~8 motifs per species to keep recognizability high (per PRD, 7-bird ceiling).

### 5.6 Weather

Weather is global per aviary, server-driven on a stochastic schedule (rain a few times/week; wind a bit more often; nothing else at v1). Weather events go through the event log so the tick treats them like any other input. Intensity ramps over a few minutes; effects on mood are in the mood transition table.

### 5.7 Notebook entry generation

Sparsity is a feature, not a bug. The notebook generator:

1. After each tick, evaluates a small set of *potential* observations (greeting-order shift, mood-persistence-across-day, weather-with-bird-response, drift-milestone, "long quiet" sessions).
2. Each potential observation has a *salience score*.
3. We only emit if (a) salience > threshold, AND (b) we haven't emitted within a cooldown window (**(call)** 24 h baseline, with rare-event override for true milestones).
4. Generation is templated naturalist prose, parameterized by bird names, perches, and observed pattern. Templates are written by-hand, lowercase, present-tense, no announcement framing. Each template has multiple realizations with light procedural variation (clause order, optional adverbs) to avoid sameness.
5. The generator never emits user-behavior observations. A static prompt-template review checks that no template references "you," "every day," "this week," "streak", "visit", etc. CI enforces a denylist on template literals.

For very active users, a hard cap of ~3 entries/week prevents the notebook from becoming a feed.

### 5.8 Determinism and replay

Tick is deterministic: given the same canonical state + same event slice + same time bucket, the output is the same. RNG is seeded from `(account_id, tick_index)`. This makes:

- Replay testing trivial.
- Bug investigations safe (no production data exfiltration; replay against synthetic accounts).
- Future migrations possible without "what would the bird have done" ambiguity.

---

## 6. Sync Model

### 6.1 Single canonical writer

Personality, mood, perches, scheduled calls, notebook entries — all written *only* by the simulation service, *only* one tick at a time per account. A row-level lock (or a per-account lease in the scheduler) enforces single-writer.

### 6.2 Snapshot delivery

Two paths to the client:

1. **Inline-on-HTML** (initial load): the edge server fetches a tiny snapshot from a per-account cache (warmed on auth) and inlines it into the HTML response, so the client has frame-zero data before the JS bundle parses. This is what makes the 500 ms first-bird budget reachable.
2. **Pull / push after init**: client uses `If-None-Match` + WebSocket push when the canonical state's tick number advances. Client polls every ~30 s as fallback when WebSocket isn't connected.

Snapshots are eventually consistent at <2 s lag in steady state. The PRD allows for this — the user shouldn't be able to detect the lag visually because no single tick produces a visible change.

### 6.3 Why no last-write-wins

Reiterated as code rule: clients submit *events*, not *state*. Servers apply *deltas*, not *replacements*. The DB schema literally lacks a "set personality_vector" code path; the only mutator is `apply_drift_delta(bird_id, ...)` and it only adds non-negative values.

### 6.4 Multi-device behavior

Two devices reading the same account: each opens a WebSocket; each receives identical snapshots; each interpolates locally. If both devices are foregrounded simultaneously, presence-time for that interval is *not* double-counted: the server keys presence accumulation by `(account_id, time_window)` and dedupes overlapping presence pings across sessions. The harder version of this — measuring multi-device attention precisely — isn't required by the PRD; dedupe is sufficient.

### 6.5 Visibility/suspend handling

- On `visibilitychange → visible`, client sends `session_resume` and pulls a fresh snapshot.
- On long render-frame gap (e.g., laptop wake), client pulls a fresh snapshot before the next render.
- If the client clock and the server clock diverge by more than a small window (**(call)** 5 s), client trusts the server snapshot's `server_time`.

---

## 7. Frontend Rendering Pipeline

### 7.1 Tech choices **(call)**

- TypeScript.
- Custom render loop on `<canvas>` (2D for v1; we'll evaluate a WebGL2 micro-renderer if perf headroom isn't met on the target laptop). React only for chrome (top bar, settings, notebook UI), not for the aviary.
- The aviary is one canvas, full-bleed, with a thin React overlay for the top bar and modal surfaces.
- Build tooling: Vite + esbuild. Strict bundle-size budgets enforced in CI.

The aviary-on-canvas-with-React-chrome split keeps render-hot paths out of React's reconciler, which is the only way to stay at 60 fps idle on a 5-year-old laptop.

### 7.2 Scene composition

Three logical layers:

- **Background**: sky gradient + far foliage. Updated on time-of-day phase change. Mostly static between updates.
- **Mid**: perches + birds. Where the action lives.
- **Foreground**: occasional leaves, feathers, a soft branch sway. Driven from a deterministic PRNG seeded each minute so cross-tab consistency holds approximately and there's no per-leaf state.

Birds are rendered from a small skeleton-and-pose system: each species has ~8–12 named poses (preen, scan, head-tilt, fluffed, calling-open-beak, calling-closed, mid-hop, settled). Poses are SVG paths or compact bitmap atlases (**(call)** SVG to keep bundle small and to allow procedural color recoloring for plumage tier).

### 7.3 Idle micro-motion

- Each pose has a small set of micro-deformations (subtle breath rise, head tilt, eye blink) animated procedurally rather than baked.
- Motion is mood-driven via a parameter pack: e.g., `wary` lifts head higher and increases scan-frequency; `drowsy` fluffs feathers and lowers stance; `curious` adds head-tilt-toward-sound triggers.
- All motion is interpolated from the snapshot's render-hints; no client-side state machine *decides* mood — only *renders* it.

### 7.4 Transitions between perches

Perch changes in snapshots trigger animated paths (eased curves between perch anchor points), with wing-flap pose during the path. Path duration is short (<800 ms) so it doesn't dominate the scene.

In reduced-motion: no path. Cross-fade between "sitting at perch A" frame and "sitting at perch B" frame over ~1 s.

### 7.5 First-frame strategy

- HTML response is small (<8 KB) and contains an inlined snapshot.
- A tiny inline script (bundled as a separate ≤10 KB chunk loaded eagerly) reads the snapshot, draws a static first-frame of the aviary into the canvas using inline SVG bird poses, and starts a slow ambient-color-shift loop. This is the "first bird visible" path and is independent of the main bundle parsing.
- The main bundle then loads, hydrates the renderer, and continues motion *from where the static frame is*. No fade, no transition between the two — the user sees one continuous aviary.
- On cold-state (snapshot not yet warmed at the edge), the inline script renders the "quiet field" loading state — soft sky, no spinner — and swaps to the populated aviary when the snapshot arrives.

### 7.6 Reduced-motion mode

- Detected from `prefers-reduced-motion` and overridable in settings.
- Engages a different render mode: no animated paths, no micro-motion deformation, no leaf drift, no top-bar fade animation (chrome stays at constant opacity).
- Calls and audio still play unchanged (reduced-motion is about motion, not audio).
- Day/night color shifts remain, just slowed.
- The reduced-motion mode is *not* a degraded fallback; it has its own first-frame, its own mood-rendering (cross-fade between named poses), and its own narrative continuity — the same product, slower.

### 7.7 Top bar

Thin top bar, four icons: account/settings, accessibility, notebook, offer. Top bar fades to ~10% opacity after **(call)** 4 s of pointer stillness; returns to full opacity on pointermove or any keypress. Reduced-motion users get instant opacity changes (no fade).

---

## 8. Audio Pipeline

### 8.1 WebAudio graph

Per-bird subgraph:

- An oscillator-and-noise source bank generating motif voicings (mix of sine, triangle, filtered noise per species).
- A simple ADSR envelope per call.
- A pitch/timing modulator parameterized by mood and personality.
- A bird-specific filter chain (formant-ish for each species).
- A per-bird gain node feeding the master.

Shared:

- A small reverb (impulse response, ~30 KB convolution) to give the aviary its sense of space.
- A "wind/rain" ambient layer modulated by weather state.
- A master gain feeding `AudioContext.destination`.

### 8.2 Chorus mix

Two or more birds calling concurrently is the chorus. Because each call is generated live, two simultaneous calls don't phase-cancel — they layer naturally. Chorus events are *not* a separate code path; they're a consequence of overlapping per-bird call windows.

### 8.3 Listen-in

Listen-in is a slow gain re-balance. On engage:

- Focused bird's gain ramps from 1.0 to ~1.4 over ~1.5 s.
- Other birds' gains ramp from 1.0 to ~0.3 over the same interval.
- Ambient layer ramps to ~0.6.

On disengage: same ramp in reverse. Other birds never reach 0 gain — the aviary remains a place, not a bird-soloing UI. Implemented via `GainNode.gain.linearRampToValueAtTime`; no JS-driven volume polling.

### 8.4 Procedural call generation

- Motif library per species lives as data: a sequence of `(pitch_cents, duration_ms, source_kind, envelope_params)` tuples per motif.
- A motif is realized at runtime by:
  1. Reading `(grammar_seed, mood, personality bucket)` from snapshot.
  2. Sampling from the species' motifs deterministically given the seed (so a snapshot's seed always produces the same call — useful for QA replay).
  3. Applying mood-and-personality modulation: pitch jitter, timing jitter, motif selection bias.

### 8.5 WebAudio fallback

If `AudioContext` is unavailable, fails to start, or the user has audio off:

- Playback path is bypassed entirely.
- Captions auto-enable (regardless of caption setting) with a quiet first-time notice (matter-of-fact, dismissible).
- The aviary continues otherwise unchanged.
- No recorded-audio fallback ships in the bundle.

### 8.6 Memory hygiene

- AudioBuffers for the impulse response and any one-shot wind/rain samples are loaded once and shared.
- Per-call nodes are created, used, and torn down deterministically; we maintain a small node pool to avoid GC churn.
- A 30-min CI test asserts heap snapshot delta below a small threshold (see §10.4).

---

## 9. Accessibility Surfaces

Accessibility is a designed surface of v1, shipped on day one. Sections below are design+engineering pairs.

### 9.1 Screen-reader narration

- Narration is generated server-side as part of the snapshot (`narration_prose: string`), refreshed when the snapshot changes meaningfully.
- The client renders narration into a visually-hidden, ARIA-live polite region. We use polite (not assertive) so calm narration doesn't barge into the SR queue.
- Cadence: at most one narration update per ~30–60 s in idle, faster only on user-initiated events. The server gates this.
- Voice: same naturalist prose as the field notebook. Templates are reviewed by the writer-of-record and version-controlled.
- Priority: user-initiated events (offer accepted, settle gesture, return-greeting) get a narration update via a *separate* assertive region with a single short prose bump, then quiet again.
- Determinism: seeded from snapshot data so a given snapshot always produces the same narration (replayable).

### 9.2 Captions for calls

- Generated client-side from the same `(grammar_seed, mood)` used to synthesize audio. The motif library carries both an audio realization and a caption realization per motif (e.g., `"a soft three-note rise"`).
- Captions render as small text near the calling bird's screen position, fade in over ~200 ms, hold for the call's duration, fade out.
- Reduced-motion users get instant in/out instead of fade.
- WCAG AA contrast on caption text against the aviary background; we render a subtle dark scrim under captions if local contrast is insufficient.

### 9.3 Keyboard navigation

- `Tab` cycles through top-bar items first (settings, accessibility, notebook, offer).
- `Tab` again moves into the aviary; first bird is focused.
- `←` / `→` (and `↑`/`↓`) move focus between birds in left-to-right (then back-to-front) order.
- `Enter` toggles listen-in on the focused bird.
- `Esc` exits listen-in.
- A second-level shortcut `O` opens the offer affordance from anywhere in the aviary.
- `S` settles the aviary; a brief in-aviary affordance shows the 5-s undo timer; pressing `S` again or `Esc` undoes during that window.
- Focus indicator: a soft, high-contrast outline (designer-specified) that maintains visibility against bright and dim aviary states. We render the focus ring on the canvas itself (overlay) so it sits above the scene without leaking through React.

### 9.4 Contrast

- All chrome text passes WCAG AA. The design system specifies the values; the build pipeline runs a contrast-token verifier as a CI check on the design-token package.
- High-contrast mode (settings opt-in) increases chrome contrast and also strengthens the aviary's bird-perch silhouettes (subtle outline) without changing the palette character.

### 9.5 Settings surface

The accessibility settings panel has matter-of-fact copy:

```
Reduced motion: [Off | On | Match my system]
Captions for calls: [Off | On]
Audio: [On | Off]
High contrast: [Off | On]
```

Any change applies live, with no save button. (Forms with save buttons here would over-engineer a low-stakes surface.)

---

## 10. Performance Budgets and Observability

### 10.1 Bundle budget

- Initial JS bundle (the chunks needed to render the aviary): **≤2 MB gzipped**, measured on every PR via CI.
- Top-bar React chrome and modals (settings, notebook, accessibility, account, visit-invitation flow) live in deferred chunks loaded on-demand.
- The motif library is data, not code — measured separately. Target ~150 KB gzipped for all six species combined.
- A bundle-analyzer report is published on every PR; budgets fail the build if exceeded.

### 10.2 Time-to-first-bird

Target: **≤500 ms p75** on a mid-tier mobile device on 4G to first bird visible.

Implementation strategy:

1. Edge inlines the initial snapshot into the HTML.
2. A small (≤10 KB) inline script renders the static first-frame.
3. The full bundle loads asynchronously and continues from there.
4. We pre-warm the per-account snapshot cache when the magic-link consume call lands, so the snapshot is already at the edge before the user's next navigation.

Synthetic perf checks: a fleet of headless browsers run from at least three geographies, on emulated mid-tier mobile with 4G throttling, on a schedule (every 5 min). Alarm on p75 breach.

### 10.3 60 fps idle on a 5-year-old laptop

- The render loop uses `requestAnimationFrame` and skips frames when the tab is hidden.
- Per-frame work is bounded: idle micro-motion uses precomputed sin tables; pose interpolation runs on a small fixed number of birds (max 7).
- Worst-case profiler target: <8 ms render time per frame on a 2020 mid-range laptop with 60-Hz display.
- Performance test: a CI "scene benchmark" runs the aviary headless with 7 birds, all in motion, for 60 s, and reports p99 frame time. Fails the build if p99 > 12 ms.

### 10.4 No memory growth in 30-min sessions

CI integration test:

1. Run the aviary in headless Chrome with 7 birds, simulated active session, for 30 min.
2. Take heap snapshots at minute 1 and minute 30.
3. Difference must be below threshold (**(call)** 5 MB, allowing for some non-GC'd cache fill).
4. Specific checks:
   - AudioBufferSourceNodes count returns to baseline between calls.
   - Notebook entries scrolled out are released (WeakMap-backed list views).
   - Worker threads are bounded to a fixed pool.

### 10.5 Observability

- Aggregate-only metrics: request counts, latencies, error rates, simulation tick latency p50/p95/p99, audio context error count, render-frame timing histograms (RUM, anonymized).
- **Forbidden in metrics**: any per-bird value, any per-account value beyond an account-UUID error correlation ID *only when escalated to oncall* (not in default dashboards), any interaction count by type per account, anything that could reconstruct a relationship.
- Per-tick latency p99 alarm at 5 s.
- Sign-in error rate alarm.
- CDN bundle-size alarm if any deploy increases total bundle size by >5%.
- Privacy invariants enforced at the metrics-pipeline layer: a denylist of dimensions (`bird_id`, `account_email`, etc.) is checked at metric emission and at warehouse ingestion.

---

## 11. Privacy / Telemetry Boundary

This is part of the architecture, not an addendum.

- Email lives in exactly one place (the encrypted `email_encrypted` column on `account`) plus a one-way `email_lookup_hash` for sign-in lookup. No other table, log, message queue, metric, or telemetry event references email.
- Synthetic UUIDs are the only identifiers that cross service boundaries.
- Per-bird and per-account interaction state never leaves the simulation database into:
  - the analytics warehouse (no ETL job exists; we will not write one);
  - any ML training pipeline (no such pipeline exists at v1; if one is ever proposed, this rule must be re-affirmed);
  - any third-party tool — error reporters, analytics SDKs, marketing tools.
- The error-reporting tool (e.g., Sentry-equivalent) is configured with PII scrubbing and a dimension allowlist.
- The privacy policy is a plain-text, link-from-settings document, listing aggregate categories explicitly and stating per-bird state is excluded.

---

## 12. Rollout Plan

### 12.1 Staging order (engineering)

1. **Foundations (weeks 1–3)**: schema, auth, magic-link, edge HTML, empty-aviary client, CI bundle/perf budgets in place from day one.
2. **Simulation v0 (weeks 3–6)**: tick scheduler, drift function, mood FSM, event log, snapshot endpoint, single bird, no audio.
3. **Render pipeline v0 (weeks 5–8)**: canvas renderer, one species, idle motion, perch transitions, day/night cycle, top bar, settings.
4. **Audio (weeks 7–10)**: WebAudio graph, motif library for one species, listen-in, fallback, captions.
5. **Birds 2–7 + adoption flow (weeks 9–12)**: full species pool, naming, two-starter adoption, return-greeting, multi-bird chorus.
6. **Notebook (weeks 11–13)**: entry generator, sparsity controller, naturalist templates, denylist enforcement.
7. **Accessibility designed surface (weeks 11–14, parallel)**: screen-reader narration server-side, ARIA-live wiring, reduced-motion render mode, captions polish, keyboard nav, focus rings, high-contrast.
8. **Visits (weeks 13–15)**: invite flow, visitor session, projected snapshots, revoke, visit log, expiration.
9. **Account export, deletion, change-email (weeks 14–16)**.
10. **Hardening (weeks 16–18)**: synthetic perf fleet, memory tests at scale, calibration of drift constants, browser matrix sweeps, accessibility audits with real assistive-tech users, security review.

### 12.2 Bird-count ramp

- Closed alpha (10s of accounts, internal): cap at 7 from day one to surface chorus issues early. We don't ramp; we want early evidence that 7 is sustainable.
- Closed beta (low hundreds of users): same cap.
- Public v1 launch: same cap.

The PRD treats the cap as empirical. If beta evidence is that recognizability collapses earlier (say, at 5), we lower the cap. We don't raise it.

### 12.3 Drift calibration

- Pre-launch: synthetic populations of simulated users (presence patterns from real-ish distributions) feed the simulation; we tune `k_*` constants until calibration targets are met (~1 week instruments, ~3 weeks visible).
- Post-launch: aggregate-level monitoring of drift distribution — the *shape* of the trait distributions over weeks, not per-account values. If the distribution moves faster or slower than the calibration intends, we tune constants in code (not retroactively per-account).
- All such tuning is versioned; we record the calibration version on every bird record so we can interpret historical drift correctly.

### 12.4 Day-one instrumentation

- Health: tick latency, edge response p99, snapshot freshness lag, error rates.
- Perf: synthetic first-bird, RUM render-frame histograms.
- Privacy invariants: denylist verification job.
- Accessibility: a synthetic SR-runner that loads the aviary and verifies live-region content updates on a slow cadence (no faster than spec).
- Audio: WebAudio init failure rate.

### 12.5 Launch checklist (selected)

- All bundle/perf/memory budgets green for two weeks of synthetic runs.
- Accessibility audit by a qualified third party with at least one screen-reader user, one reduced-motion user, one keyboard-only user, one user with Deafness or hearing differences, one user testing in a low-contrast environment.
- Penetration test on the magic-link flow and the visitor-session flow.
- Privacy review confirming no per-account telemetry dimension and an export of warehouse schemas to verify.
- Disaster recovery drill: restore a Postgres snapshot to a staging environment, validate that birds and personality vectors round-trip.
- Static-analysis CI rule confirming no negative drift coefficient.

---

## 13. Risks and Mitigations

### 13.1 Drift calibration is wrong

- **Risk**: drift is too fast — birds feel volatile session-to-session — or too slow — three weeks pass with no perceived change. Either failure quietly invalidates the product's central claim.
- **Mitigation**: synthetic-population calibration before launch; aggregate distribution monitoring after launch; calibration constants are data, not code, and roll forward with version tags so historical drift remains interpretable.

### 13.2 Personality vector loss

- **Risk**: the worst possible failure — a bug, a migration error, a restore-from-stale-backup that resets a bird's personality. The user wouldn't see a stack trace; they'd just feel off.
- **Mitigation**: server-only writer; row-level immutability of bird `id`; Postgres point-in-time recovery; daily logical backups of the simulation database; integration tests that round-trip personality through the entire stack; on every migration that touches the bird table, a checksum-of-traits per-account is computed before/after and compared.

### 13.3 Sync correctness

- **Risk**: client-side write paths sneak in. A future contributor "just" lets a client patch a personality value because it's quicker.
- **Mitigation**: schema and API have no write path for personality. The DB ORM model marks trait fields read-only at the application layer; a code review checklist item "did you add a personality writer?" is required on simulation-touching PRs; a runtime invariant in the simulation service rejects any write that didn't originate from the tick.

### 13.4 Audio uncanniness

- **Risk**: procedural calls land in an uncanny valley — too synthetic to feel like birds, too organic to feel like UI.
- **Mitigation**: motif libraries and synthesis params are designed by an audio designer and tuned with a small panel of users in alpha; we A/B procedural variation against itself for "feels like a bird" judgments; calls are tweakable via per-species data files so we can iterate post-launch without code releases.

### 13.5 Accessibility regression

- **Risk**: visual or audio polish lands and quietly breaks SR narration timing or focus indicator visibility.
- **Mitigation**: SR-narration-cadence test in CI; focus-ring contrast test against rendered scenes (snapshot diff with a contrast threshold); accessibility regression suite runs nightly with assistive-tech automation.

### 13.6 Engagement-feature creep

- **Risk**: "harmless" engagement feature requested by a stakeholder later — a streak indicator, a "you've been here every day" easter egg. Once one lands, the product's affective contract leaks.
- **Mitigation**: refusal is in the PRD with reasoning, repeated here. The denylist of forbidden strings (in CI) and forbidden user-behavior templates (in notebook generator) are concrete defenses. Anti-feature design reviews on any PR that touches the chrome, notebook, or settings surfaces.

### 13.7 Privacy / PII leakage

- **Risk**: email or per-bird state ends up in a log, an error reporter, a metric, a warehouse.
- **Mitigation**: dimension allowlist at metrics emission; PII scrubbers in error reporters; structural separation of simulation DB from analytics; periodic random-sample audits of log lines for forbidden patterns.

### 13.8 Performance creep

- **Risk**: bundles grow with each feature; first-bird budget slips by a few ms per release; one day it's no longer met.
- **Mitigation**: per-PR budgets fail the build, not just warn; the synthetic perf fleet alarms on p75 regressions; before any new feature ships, its perf cost is measured in the same CI suite.

### 13.9 Visit-feature abuse

- **Risk**: visit links forwarded; visitor surface used for harvesting host activity over time; visit revocation race conditions.
- **Mitigation**: visitor sessions are short (~2 h) and bound to invitation tokens; visit-snapshot endpoint is read-only and rate-limited; revocation takes effect at next snapshot pull; visit duration is bucketed in the visit log to reduce stalking utility; invitation tokens are 128-bit random, single-bind on first claim **(call)** to a visitor session (subsequent claims fail).

### 13.10 Single-writer simulation bottleneck

- **Risk**: as accounts grow, the per-account-shard simulation can't keep up; tick latency p99 climbs.
- **Mitigation**: simulation service is horizontally shardable by account UUID; tick cadence drops to 5-min for idle accounts; per-tick work is bounded; load tests run before major launches.

---

## 14. Open / Deferred Decisions

These are calibrations or details we deliberately defer past PRD reading and will close before launch.

- Exact presence-event activity window (a few minutes; final value from beta data).
- Exact mood timer durations per state.
- Drift constants `k_presence`, `k_listen`, `k_offer` and their per-trait responsiveness curves (calibrated via synthetic populations).
- Exact bird-adoption-by-aviary-age cadence (PRD says months apart; beta will tune).
- Exact notebook entry sparsity threshold for very active vs. infrequent users.
- Exact retention window on the event log (currently 90 days **(call)**; revisit with privacy review).
- Final palette and exact contrast tokens (design system).
- Whether to ship an offline cache of the most recent snapshot for "the aviary appears even before the network responds" (likely yes; will assess against bundle budget).

None of these defaults block v1 design. Each has a clear owner and a measurement gate before launch.

---

## 15. Out-of-Band Notes for the Engineering Team

A short list of things that will not appear in any spec but matter:

- Every PR description has a "what does this teach the user?" line. If a feature subtly teaches the user that engagement is for the counter, it doesn't ship — even if the PR otherwise looks correct.
- The naturalist voice in narration and notebook is *load-bearing* style. Reviews on those code paths require a non-engineering reviewer (the writer-of-record).
- Time and timezone bugs are likely in the day/night code; we set a regression test for each timezone discovered to misbehave.
- The "first frame is alive" promise can be broken by a single dropped optimization (a spinner sneaking in during a refactor). The launch checklist contains a manual visual verification on cold-cache load over throttled connection.
- The product is small enough that careful work fits inside reasonable timelines. The risk is not scope; the risk is drift away from the affective contract under cumulative reasonable-looking pressure.

---

## End of plan

The above is comprehensive enough to execute v1 without further clarification on substance. Calibration and design-system specifics are explicitly deferred and named, with their gates.
