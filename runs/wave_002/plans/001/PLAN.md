# Pocket Aviary — Implementation Plan (v1)

This plan translates the PRD into an executable build for a frontier engineering team. It does not restate the spec; it interprets it into architecture, contracts, calibration, and rollout. Where the PRD leaves room, defensible calls are made and flagged.

---

## 1. Scope

### In scope for v1

- Single-user accounts; one canonical aviary per account; magic-link email auth.
- Web client only (last-two-major Chrome, Safari, Firefox, Edge), responsive single-screen scene.
- Server-side simulation tick (~1/min cadence) as the only writer of personality vectors and authoritative mood transitions; clients read snapshots and submit append-only events.
- Bird engine: 5-dimension personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic-toward-expressive drift, daily-cadence mood, procedural call grammar, mood-shaped idle motion, ~6 species pool, 2 starter birds, hard cap of 7 birds, age-based new-bird offers.
- Interactions: return-greeting, listen-in (gradual mix ramp), offer (seed / song fragment / still pool, per-bird cooldown), settle with 5s undo, field notebook (sparse naturalist prose, read-only), presence accounting (visibility ∧ focus ∧ recent input).
- Aviary scene: three perch zones, day/night by user local time, ambient weather, ambient micro-motion, top bar (account, accessibility, notebook, offer) with idle fade.
- Multi-device sync as a property of the architecture (no client-to-client sync).
- Visit-invitation feature (off by default, per-invite opt-in, read-only ambient, host-revocable, 30-day invite TTL); silent visit log; host-side opt-in visit notifications (default off).
- Accessibility as first-class designed surface: naturalist screen-reader narration, reduced-motion mode (cross-fade aesthetic), procedural call captions, full keyboard navigation, WCAG AA contrast on chrome.
- Performance budgets: ≤2MB gzipped initial bundle, ≤500ms TTFB (time to first bird) on mid-tier mobile / 4G, 60fps idle on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export (JSON, emailed link); soft-delete (30 days) → hard delete; per-device session list and revocation.
- Aggregate-only telemetry; per-bird/per-account interaction state never enters analytics or training pipelines.

### Explicitly out of scope (respected from `non_goals.md`)

No native apps. No gamification of any kind (achievements, streaks, levels, scores, badges, calendars of green dots, "you've been here X days" surfaces, XP, ranks, tiers). No Tamagotchi mechanics (hunger, distress, decay, death, neglect-driven negative drift). No social-network surfaces (profiles, follows, public feed, discovery, leaderboards, comments, friend-of-friend). No push notifications, no marketing emails, no "welcome back" textual surface, no "your friend visited" push. No personality-vector visibility to the user (no debug, no settings toggle, no future tier). No client-side simulation tick. No recorded-audio fallback. No co-presence on visits. No editable notebook.

### Defensible calls flagged

- **Drift filter:** an EWMA over a rolling 14-day window with α tuned to hit the calibration target (instruments-detectable in ~7 days, user-perceptible in ~21). Alternative: a leaky-integrator over presence-time. Either is fine; the test harness pins behavior, not the implementation.
- **Tick cadence:** 60s. Cheap, comfortably under the p99 5s alarm budget, gives mood transitions a believable granularity.
- **Activity window for presence:** 4 minutes. Long enough to allow watching without moving (the actual product), short enough that an unattended laptop falls out of presence within reasonable time.
- **Snapshot pull frequency while visible:** 30s keepalive plus event-driven pulls on visibility/focus regain, suspend-resume detection, and listen-in/offer/settle commits.
- **Listen-in mix ramp:** 800ms equal-power crossfade on engage and disengage.
- **Settle undo window:** 5s, per PRD.
- **Notebook entry cadence target:** ≤1 entry per active aviary per 48h on average; opportunistic noteworthy events (chorus, weather + interaction co-occurrence, drift inflection) override sparsity but are themselves rate-limited.

---

## 2. Architecture

### Service shape

Five long-lived services plus the static client:

1. **Edge / web** — serves the static SPA shell and a tiny inline state-snapshot bootstrap (see `Performance` for why this lives at the edge). CDN-fronted.
2. **Auth service** — magic-link issuance, link consumption, session-token issuance, per-device session listing/revocation, email-change verification.
3. **Aviary API** — REST/JSON over HTTPS; reads canonical aviary state, accepts append-only interaction events, exposes account settings, the visit-invite flow, the visit log, the export request, narration text, captions.
4. **Simulation worker pool** — runs the per-account simulation tick. Each account is a sharded, single-writer unit; a tick worker pulls the recent event log and the current canonical state, computes deltas, writes the new state and any notebook entries.
5. **Mailer** — sends magic links, export-ready notifications, account-change confirmations. No marketing surface.

Single relational DB for canonical state (Postgres). One append-only event log table (or Kafka topic if scale demands later; v1 ships with Postgres). One narrow read-replica for the API path. A separate analytics warehouse exists but **never reads** from the canonical DB; only the operational telemetry pipeline does, and only on aggregate categories defined below.

### Client / server split

- **Server owns:** personality vectors, canonical mood, mood timers, drift history, notebook entries, presence-time ledger, aviary age, bird identity, audit log.
- **Client owns:** rendering, audio synthesis, idle micro-motion, ambient leaf/feather drift, top-bar UI, listen-in mix, captions display, narration display surface (when used as visual narration).
- **Boundary rule:** clients never write personality state. They write events. The simulation tick is the only writer.

### Render pipeline boundary

The client pulls a state snapshot, places birds at their snapshot poses, and runs:

- A **simulation interpolator** that renders smooth motion between snapshot N and N+1.
- An **idle motion layer** that runs personality- and mood-keyed micro-motions on top of the interpolated pose. Idle motion is purely client-local; it has no canonical effect.
- An **ambient layer** for leaves, feathers, parallax, weather visuals.
- A **lighting layer** that computes day/night by the user's local time (`Date`) and weather state from the snapshot.

The render pipeline does not introduce any state that survives a snapshot pull. Snapshots are always authoritative.

### Hosting and isolation

- Per-account simulation runs in a sharded worker; sharding key is the synthetic account UUID. Sharding is by hash; no PII in the partition key.
- Tick scheduling is driven by a per-account "next tick at" timestamp. A single coordinator service hands shards to workers; workers process and update.
- The simulation database is firewalled from the analytics warehouse; this is a network-level rule, not a "we promise not to query" rule.

---

## 3. Data model

### Account

```
accounts(
  id              uuid primary key,         -- synthetic, used everywhere
  email_encrypted bytea not null,           -- one place, encrypted at rest
  email_hash      bytea not null unique,    -- for sign-in lookup; salted
  created_at      timestamptz not null,
  state           enum('active','soft_deleted','hard_deleted'),
  soft_deleted_at timestamptz null,
  visit_notify    bool not null default false
)
```

Email lives only here. Every other reference (events, telemetry, logs, partitions, Kafka keys if any) uses `accounts.id`.

### Sessions

```
sessions(
  id              uuid primary key,
  account_id      uuid not null references accounts(id),
  device_label    text,                     -- user-visible "Chrome on macOS"
  created_at      timestamptz,
  last_seen_at    timestamptz,
  revoked_at      timestamptz null
)
```

### Birds

```
birds(
  id              uuid primary key,         -- stable identifier; never reassigned
  account_id      uuid not null,
  species_id      text not null,
  name            text not null,
  adopted_at      timestamptz not null,
  retired_at      timestamptz null          -- not used in v1 (no death), reserved
)
```

### Personality vector

```
bird_personality(
  bird_id         uuid primary key references birds(id),
  boldness        real not null,            -- normalized [0, 1]
  social_warmth   real not null,
  vocal_frequency real not null,
  plumage_sat     real not null,
  curiosity       real not null,
  updated_tick    bigint not null           -- last tick that wrote this row
)
```

Drift history is reconstructable from the tick log if needed for debugging; we do not surface it to the user.

### Mood

```
bird_mood(
  bird_id         uuid primary key,
  mood            enum('wary','content','curious','drowsy','alert'),
  entered_at      timestamptz not null,     -- supports "mood persists across sessions"
  next_review_at  timestamptz not null      -- earliest tick allowed to transition
)
```

### Aviary state

```
aviary(
  account_id      uuid primary key,
  weather         enum('clear','rain','wind') default 'clear',
  weather_until   timestamptz null,
  perch_layout    jsonb,                    -- per-bird current perch
  last_tick_at    timestamptz,
  created_at      timestamptz,              -- aviary age = now - created_at
  empty           bool default true         -- true between adoption and first bird-fly-in
)
```

### Event log (append-only)

```
events(
  id              bigserial primary key,
  account_id      uuid not null,
  bird_id         uuid null,                -- null for global events (settle, weather)
  type            enum('presence_ping','listen_in_start','listen_in_end',
                       'offer_seed','offer_song','offer_pool',
                       'settle','settle_undo','session_open','session_close',
                       'visit_view'),
  occurred_at     timestamptz not null,
  client_seq      bigint,                   -- monotonic per session, for ordering
  payload         jsonb null
)
```

The event log is the only path from client to canonical state. No row in `events` is ever updated or deleted (except by hard-delete at account-deletion).

### Notebook

```
notebook_entries(
  id              uuid primary key,
  account_id      uuid not null,
  written_at      timestamptz not null,
  prose           text not null
)
```

Read-only from the client perspective; written only by the simulation tick.

### Visits

```
invitations(
  id              uuid primary key,
  account_id      uuid not null,            -- host
  visitor_email_h bytea not null,           -- salted hash; cleartext email mailed out then dropped
  token           uuid not null unique,
  created_at      timestamptz,
  expires_at      timestamptz,              -- created_at + 30 days if unused
  revoked_at      timestamptz null,
  consumed_at     timestamptz null          -- first use; subsequent use re-uses the token within 30d
)

visits(
  id              uuid primary key,
  invitation_id   uuid not null,
  started_at      timestamptz,
  ended_at        timestamptz null,
  visitor_ip_hash bytea null                -- for abuse prevention only; rolling 7-day TTL
)
```

The host-visible visit log shows the cleartext visitor email re-derivable from the invitation record. Visitor email cleartext is stored on `invitations` only (encrypted), not on `visits`, since the visitor is bound to the invitation 1:1.

### Telemetry (separate cluster)

Aggregate buckets only. No `account_id`, no `bird_id`, no `event_id` references in any telemetry record. See `Performance observability`.

---

## 4. API surface

All endpoints over HTTPS. JSON. Authenticated by session token in `Authorization: Bearer`. Synthetic UUIDs only in URLs.

### Auth

- `POST /auth/magic-link` — body `{ email }`. Always returns 204; rate-limited per email and per IP. Email is the only place the cleartext lives in the request.
- `GET /auth/consume?token=...` — single-use, 15-minute TTL. On success, sets a session token and redirects to the aviary.
- `POST /auth/sessions/:id/revoke` — revokes a peer session.
- `POST /auth/email-change` — initiates verification; old email continues to work until new is verified.

### Aviary state

- `GET /aviary/snapshot` — returns the rendered-friendly state: birds, current mood, current perch, current call timing seeds, weather, day/night phase, narration prose for the last few minutes, captions in flight. Small payload (kilobytes).
- `GET /aviary/snapshot?since=<seq>` — delta form for keepalive pulls; returns just what's changed since `seq`.
- `POST /aviary/events` — body `{ events: [...] }`. Idempotent on `(account_id, client_seq)`. Returns 204. The bulk endpoint exists so a flaky network doesn't fragment the event stream.

### Notebook

- `GET /aviary/notebook?cursor=...&limit=...` — paginated, newest first. Read-only.

### Account

- `GET /account` — settings shape: email (masked), session list, accessibility prefs, visit-notify toggle.
- `PATCH /account` — accessibility prefs, visit-notify toggle, bird names. Bird names update propagates without affecting any other state.
- `POST /account/export` — generates the export job; emails a download link to verified address.
- `POST /account/delete` — soft delete; recoverable for 30 days.
- `POST /account/restore` — undoes soft delete inside the 30-day window.

### Visits

- `POST /visits/invitations` — body `{ visitor_email }`. Issues invitation, sends email.
- `GET /visits/invitations` — host-visible list of outstanding/consumed invitations.
- `POST /visits/invitations/:id/revoke` — immediate effect; next visitor pull returns the matter-of-fact "no longer available" surface.
- `GET /visits/log` — host-visible visit history (visitor email, timestamps, durations).
- `GET /visits/view?token=...` — visitor-side; returns a snapshot scoped to read-only fields. No event submission allowed; an `Allow:` header on `POST /aviary/events` for visitor sessions is `OPTIONS, GET` only (i.e., POST is rejected at the auth layer).

### Visitor session

The visitor token is its own auth class. It cannot read the host's settings, invitation list, visit log, or notebook unless the host has explicitly enabled "share notebook on visit" — which is **not** a v1 feature; for v1 the notebook is host-only. (Calling this out so it's not added by accident.)

### Visit-invitation flow (concretely)

1. Host calls `POST /visits/invitations` with visitor email.
2. Server creates `invitations` row, sends mail with one-time link encoding `token`.
3. Visitor follows link → client calls `GET /visits/view?token=...` → server validates, marks `consumed_at` if first use, opens a visitor session, returns scoped snapshot.
4. Visitor pulls snapshots normally; visitor events are dropped server-side (visit_view is recorded once at session start for the host's visit log).
5. Host revokes or 30-day window passes → next visitor snapshot pull returns matter-of-fact unavailable surface.

### Idempotency and ordering

`POST /aviary/events` carries `(client_seq, occurred_at)` per event. The simulation tick consumes events in `(occurred_at, client_seq)` order. Duplicate `(account_id, client_seq)` is dropped at insert. Out-of-order arrivals are ordered correctly by the tick because the tick reads the log and orders before processing.

### Rate limiting

Per-account event rate-limit is generous (a presence ping every 30s plus user-action events is well under any sane cap). Auth endpoints are stricter. Visit endpoints are rate-limited per host.

---

## 5. Simulation engine design

### Tick

Cadence: 60s nominal, with jitter (±5s) to spread workload. The tick for a given account does:

1. Lock the account row (advisory lock; one tick per account at a time).
2. Read `events` since `aviary.last_tick_at`.
3. Compute drift deltas (see `Drift function`).
4. Compute mood transitions (see `Mood transitions`).
5. Update perch layout if mood/personality demands a perch change (the bird-engine's planner; deterministic given the inputs).
6. Possibly emit a notebook entry (see `Notebook generation`).
7. Persist updated personality vector, mood, perch layout, weather state, and `last_tick_at`.
8. Release lock; update `next_review_at` for moods; ack consumed events.

Ticks are idempotent on rerun (deterministic from inputs); a worker that crashes mid-tick is safe to restart.

### Drift function

Inputs (per bird, per tick window):

- `presence_seconds` — total presence-time during the window where this bird existed.
- `listen_in_seconds` — total time this bird specifically was listened-in.
- `offers_directed` — count of offers in the window (offers go to the aviary; "directed" is the bird the offer-grammar resolved to per `Offer resolution` below).
- `chorus_participations` — joined-chorus events involving this bird.

Drift deltas (all non-negative; monotonic-toward-expressive is enforced as `max(0, delta)`):

- `boldness_delta = w_b * presence_seconds + w_b_offer * offers_directed_nearby`
- `social_warmth_delta = w_s * listen_in_seconds + w_s_p * presence_seconds + w_s_chorus * chorus_participations`
- `vocal_frequency_delta = w_v * listen_in_seconds + w_v_chorus * chorus_participations`
- `plumage_sat_delta = w_p * presence_seconds`
- `curiosity_delta = w_c * accepted_offers + w_c_p * presence_seconds`

Each weight is set so that, for a "regular visitor" reference profile (~10 minutes/day, ~5 days/week), `vocal_frequency` and `social_warmth` cross instrument-detectable in ~7 days and user-perceptible in ~21. The reference profile is encoded as a calibration test that runs against every release candidate (see `Tests / calibration`).

Drift is then **clipped to the upper bound** of the trait's normalized range. Once a trait saturates, additional input is a no-op for that trait.

A small **anti-thrash floor**: drift deltas under a small threshold are accumulated in a `pending_delta` row and applied only when the accumulator crosses the threshold. This avoids per-tick storage churn and keeps drift instrument-detectable rather than instrument-noise.

### Mood transitions

Per bird, on each tick, given the current mood and inputs, evaluate transition probabilities:

- `wary → content` weighted by recent positive interactions and time-of-day (morning/midday).
- `content → curious` on novel inputs (offer, weather change, another bird's chorus call).
- `* → drowsy` weighted by local time (dusk).
- `* → alert` on sharp inputs (an alarm call from another bird, weather onset).
- `wary` is biased to occur only on weather-onset, alarm calls, or low-boldness × low-presence-recency.

Transitions are gated by `next_review_at` to prevent flicker; minimum dwell time per mood is 2 ticks (~2 minutes). The bird-engine planner deterministically maps `(personality, mood, perch_layout, recent_events)` → next perch and next call-grammar parameters; that determinism is what makes tick reruns idempotent.

### Bird-to-bird interaction

Mood spread: when one bird transitions into `wary` or `alert`, neighbors with low-boldness have a heightened transition probability into `wary` for the next 1–2 ticks. Chorus events: when two birds with high `vocal_frequency` are both in `content` or `alert` and at least one has just called, a chorus event is generated and recorded on both birds.

### Call-grammar runtime

Calls are not generated by the server. The server emits per-call **parameters** in the snapshot:

```
call: {
  bird_id, motif_id, start_at, duration_ms,
  pitch_seed, timing_seed, mood_inflection
}
```

The client's WebAudio synthesizer renders the call from the motif library using these parameters. Recognizability across drift comes from a **stable per-bird voice signature**: each bird has a small immutable set of formant offsets and timbre coefficients picked at adoption from the species' allowed range, persisted on the `birds` row (or on `bird_personality` as a `voice_sig` column). Drift modulates timing and frequency, never the signature.

### Offer resolution

An offer (seed, song fragment, still pool) is a global event, not bird-targeted. The simulation runs an offer-resolution step that picks the responding birds based on (per-bird): mood, distance from the offer (which perch zone), curiosity, and the per-bird cooldown (a few minutes since last accepted offer). One or more birds may approach, ignore, or react. The resolution is recorded as bird-attributed sub-events for the drift function above.

### Server-side ticking when no client is connected

The tick continues regardless of session state. `presence_seconds` for a bird in a no-client window is zero; mood transitions can still occur from time-of-day and weather. This is what lets the user come back to "the aviary that has been ticking," not "the aviary as it was when they left."

### Notebook generation

A small content-pipeline step on each tick:

1. Collect candidate-events from this tick window: chorus, drift inflections (a trait crossed a notable bucket), first-greeter changes ("Pip greeted before Wren today, first time this week"), weather + mood co-occurrences, mood transitions of note (a long-wary bird settled to content).
2. Score candidates by salience (rarity × recency).
3. If the highest-scoring candidate is above an absolute threshold AND the account hasn't received an entry in the last 24h, emit one notebook entry; otherwise drop.
4. Render the entry through a templated naturalist-prose generator. Templates are hand-authored, parameterized on (bird name, mood, perch zone, time-of-day phrase, optional companion-bird name). They are not LLM-generated at runtime; they are stable text. This avoids both the "stock event-log" failure and the "AI-warbling" failure.

The naturalist-prose generator and the narration generator share templates so that the field notebook and screen-reader narration speak the same product voice.

### Drift safety / no-tamagotchi enforcement

A unit test asserts `personality_after >= personality_before` for every trait, every tick, every fixture. The build fails on any code path that decrements a personality value. This is the implementation-level guard for the "monotonic toward expressive" rule.

---

## 6. Sync model

### Single canonical state

The server is the only writer. Multi-device sync is a property of clients reading the same canonical record, not a separate subsystem.

### Snapshot pull pattern

Clients pull on:

- Page load.
- `visibilitychange` to visible.
- Window focus regain.
- Long render-frame gap (>2s) — laptop-suspend recovery.
- Keepalive every 30s while visible.
- On commit of a user action (listen-in start/end, offer, settle).

Snapshots include a `seq`. Subsequent pulls use `?since=<seq>` for deltas.

### No client-to-client sync

Both the laptop and the phone pull from the same canonical record. No CRDTs, no last-write-wins, no merging. The two devices never disagree because they don't have independent state.

### Conflict prevention (additive deltas, server-authored)

Personality is updated by `server_delta`-form mutations from the simulation tick, not by client-submitted absolute values. The event log is the only client-to-server channel, and it is append-only. There is no API that lets a client write personality state, intentionally or accidentally; the routes don't exist.

A test asserts that no API route mutates `bird_personality` rows other than the simulation tick worker's identity. This is enforced at the DB-role level too: the API service's DB role has no write access to `bird_personality`.

### Concurrency on the tick

A per-account advisory lock ensures one tick at a time. If the same account opens new sessions on two devices, both append events into the log; the next tick consumes them all in order. Order is `(occurred_at, client_seq)`, with `occurred_at` from the client clock; clock skew tolerance is ~1 minute (the tick cadence). For tighter ordering we use `client_seq` as the tiebreaker within a session, but two devices' events do not need cross-device ordering — the tick treats them as concurrent inputs to the additive deltas.

### Account export concurrency

Export reads canonical state at a single point-in-time consistent snapshot. It does not block ticks. The exported JSON is always consistent with one tick boundary.

---

## 7. Frontend rendering pipeline

### Stack and structure

- TypeScript, Vite for build, code-splitting on routes (account, accessibility, visit-invitation flow are split out).
- Rendering: Canvas2D primary; SVG for chrome. **Defensible call:** Canvas2D over WebGL. Bird visuals are stylized SVG-like; we don't need GPU for the visual budget, and Canvas2D keeps the bundle smaller and cross-browser concerns smaller. WebGL would be considered if profiling shows we can't hit 60fps.
- A single render loop driven by `requestAnimationFrame`, gated on `document.visibilityState === 'visible'`. Hidden-tab pauses rendering; the simulation continues server-side, so the next visible-frame is just a snapshot pull and resume.

### Scene composition

Per frame:

1. Clear canvas (or use a layered approach: ambient background, midground perches, birds, foreground ornaments — each its own canvas, composited).
2. Sample the lighting curve from `(now, user_local_time, weather)` and tint background.
3. Render perches; ambient leaves and feathers from a small particle pool.
4. For each bird: compute interpolated pose between snapshot N and N+1 (linear / eased), then layer the idle micro-motion (preen, head-tilt, body-shuffle) keyed by mood.
5. Render call captions if enabled, near the calling bird.
6. Composite top bar (SVG) with current opacity from the fade controller.

### Snapshot interpolation

A snapshot at time T provides each bird's `pose_at_T` and `velocity_at_T`. The next snapshot N+1 provides `pose_at_T+Δ`. The interpolator hermite-interpolates between the two so a bird hopping from front to middle perch renders as a smooth arc, not a teleport. When a snapshot misses or arrives late, the client extrapolates using the last known velocity for up to one tick interval; beyond that, it holds position until a fresh snapshot arrives.

### Idle motion layer

Idle motions are per-bird-per-mood. They are short procedural cycles with randomized timing offsets so the same bird in the same mood looks different across sessions. Idle motion is purely aesthetic; it is **not** in the snapshot and never flows back to the server. It's the visible expression of mood that the user reads without being told.

### Aviary first-frame conceit

The bootstrap HTML inlines a tiny initial snapshot rendered from the server (CDN edge cached for that account or, on cold cache, from a same-region edge). The SPA starts with that snapshot in memory; the first render frame already has birds in pose and ambient motion seeded. There is no spinner-resolves-into-aviary transition. **This is non-negotiable**: the spinner pattern compromises the central conceit.

When the snapshot is genuinely unavailable (very slow connection, cold edge, error), the loading state is the **quiet field**: soft sky color, subtle ambient color shift, no text, no spinner. On snapshot arrival the first bird fly-ins are skipped (because the aviary has been there); birds appear at their canonical poses.

### Reduced-motion mode

Triggered by `prefers-reduced-motion: reduce` OR the user opting in via accessibility settings.

- Idle micro-motion → cross-fade between still poses (3 poses per bird-mood, blending over 1.5s).
- Flight transitions → cross-fade pose-to-pose at the new perch over 2s instead of an animated arc.
- Ambient leaves/feathers → suppressed.
- Day/night palette shifts → retained, slowed.
- Weather visuals → cross-fade in and out.
- Calls → unchanged (audio is not motion).

The reduced-motion path is a separate render module, not a flag in the regular path. This keeps the regular path lean and makes the reduced-motion aesthetic a designed surface, not a stripped fallback.

### Top-bar fade

A small controller listens for cursor and key events, holds opacity at 1 for a few seconds, then ramps to ~0.15 over 600ms. Any input restores immediately. Controls remain keyboard-reachable even while faded; a `:focus-visible` raises opacity for the focused element.

### Settle animation

A 4–6s lighting curve from current to evening palette; calls quiet over the same window; a 5s window during which any aviary-area click reverses the curve to current-time-of-day. The reversal is animated; it is not a snap.

### Empty-aviary state

Between adoption and first bird-fly-in: same quiet field as the cold-load state. The server emits a "first bird arriving" event after a few seconds; the client renders a soft fly-in to perch. Subsequent sessions never see empty.

### Performance hygiene

- Object pooling for ambient particles, call captions, idle-motion timers.
- Bitmap caches for static visual assets; recompute only on viewport resize.
- A render-budget guardrail: if frame time exceeds 20ms for >5s, drop ambient particle count by half; this protects the 60fps floor on weaker hardware.
- The render loop is suspended when `visibilityState !== 'visible'`. The audio pipeline likewise.

---

## 8. Audio pipeline

### Procedural call synthesis

A per-call synthesizer composes a call from a motif library. Each species has 4–8 motifs; each motif is parameterized by pitch, timing, formant shape, vibrato depth, and a small randomization budget. A bird's per-call output is `motif(motif_id, params(personality, mood, voice_sig, seed))`.

### WebAudio graph (per call)

- `OscillatorNode` (or a small wavetable) → `BiquadFilterNode` (formant) → `GainNode` (envelope) → per-bird `GainNode` (mix) → master mix → `AudioContext.destination`.

Per-call nodes are cleaned up on call end. The per-bird and master gains are persistent. A small node pool reuses oscillator/biquad nodes to avoid allocator pressure.

### Voice signature (recognizability)

`voice_sig` lives on the bird record — a frozen vector of formant offsets, timbre tilt, and timing personality. It is set at adoption from the species's voice-signature distribution and never changes. Drift modulates timing and frequency only. This is the implementation of "a user who has spent two weeks with Pip should know Pip's call by ear."

### Chorus mixing

When two or more birds call within a short window, the master mix layers each procedural call. Because each call is procedurally generated with independent parameters, two calls do not produce the phase-cancel artifact of stacked recordings. A small chorus-detect pass adds a slight stereo spread (soft pan per bird based on perch zone — front center, middle slight pan, back panned away) to keep the chorus legible.

### Listen-in mix decay

On listen-in engage:

- Focused bird's per-bird gain ramps from 1.0 to 1.0 (no boost; the others drop).
- All other per-bird gains ramp from 1.0 to ~0.25 over 800ms (equal-power crossfade).
- Ambient layer (weather, leaves) holds at current.

On listen-in disengage: reverse ramp over 800ms. Other birds never go silent; 0.25 is a floor that preserves the chorus at low volume. Hard mute would convert the aviary to a mixer UI.

### Ambient

A low-volume ambient bed (wind, distant calls, leaf rustle) is procedurally generated from filtered noise. On weather events, ambient ducks toward rain-noise or wind-noise as appropriate.

### WebAudio fallback

If `AudioContext` is unavailable or the user has denied autoplay, the aviary plays in graceful silence with captions enabled by default. There is **no recorded-audio fallback path**. Silence + captions is the fallback; canned audio is not acceptable per PRD.

### Captioning generation

Captions are generated from the same call parameters that drive synthesis. A small caption-template engine maps `(motif_id, mood, vocal_inflection)` → naturalist phrase ("a soft three-note rise", "a low trill, paused, low trill again"). Templates are hand-authored and shared across motifs to keep voice consistent. A caption fades in at call start, fades out after call end + 800ms, near the calling bird (positioned by the bird's screen position, with anti-overlap handling for chorus events).

### No autoplay surprise

Browsers gate audio on user interaction. The first frame renders silently; on the user's first interaction (click anywhere, key press), the audio context resumes and ambient + calls fade in. Silence on first frame is a feature, not a bug — it pairs with "the aviary appearing already in motion": the user sees motion before audio, just like opening a window.

### Audio observability

Audio-context error counts, dropped calls (synth couldn't render in time), buffer underruns are aggregated in client telemetry. No per-call audio fingerprints.

---

## 9. Accessibility surfaces

### Screen-reader narration

A separate live region (`role="status" aria-live="polite"`) holds running prose. The narration is generated server-side (via the same prose templates as the field notebook) and delivered with the snapshot. Cadence: one update per 30–60s at idle, faster only on user-initiated events. Each prose chunk replaces the previous; we do not append (which would overflow the screen reader queue).

Narration is descriptive of the aviary as a place ("a small grey bird is perched on the front rail, calling softly"), not state-list ("Pip: content; perch: front; calling"). Voice continuity with the field notebook is enforced by template sharing.

### Captions for calls

Per `Audio pipeline`. Captions also surface to screen readers as `role="status" aria-live="polite"` on a separate region from the narration. Captions update on call event; narration updates on slow cadence; both can co-exist.

### Reduced-motion mode

Per `Frontend rendering pipeline`. Triggered by media query or settings opt-in. Designed surface, not a stripped fallback.

### Keyboard navigation

- Tab order: top bar (account, accessibility, notebook, offer) → aviary scene.
- Tab into aviary focuses the leftmost (or first-named) bird; arrow keys move focus among birds.
- `Enter` triggers listen-in on the focused bird; `Esc` exits listen-in.
- Offer affordance: opens a small picker with seed/song/pool; arrows + Enter to select.
- Settle: keyboard shortcut `Cmd/Ctrl + .` (alongside the top-bar icon) — **defensible call**, as a quiet shortcut for keyboard users who want to settle without locating the icon.

### Focus indicators

A soft, high-contrast outline rendered as part of the canvas overlay (not the OS-default outline, because the aviary is canvas-rendered). The outline reads against bright and dim aviary states by being two-tone (light edge + dark edge, alpha-composited).

### Contrast

All chrome text passes WCAG AA at minimum. The aviary scene has no user-copy text other than captions and narration overlays; both follow the contrast rule.

### Settings copy and tone

Accessibility settings live under matter-of-fact voice (per the named exception in `product_brief.md`). E.g.:

> Reduced motion: On — animations are slowed and crossfaded.
> Call captions: On — short text descriptions appear with each call.
> Screen reader narration: Always on.

Toggles are immediate (no Save button).

### "Always-on" defaults

Screen-reader narration is always emitted in the API; the client only renders to a live region if a screen reader is detected (we use a heuristic, but the live region is always present in the DOM). Captions default off but auto-enable when `AudioContext` is unavailable.

---

## 10. Performance budgets and observability

### Budgets

- **Initial JS bundle ≤2MB gzipped at first paint.** CI fails the build on regression.
- **Time to first bird visible ≤500ms** on a mid-tier mobile (reference device: a 4G Pixel 6 or equivalent in the synthetic perf fleet). Measured as time from `navigationStart` to the first paint that contains a rendered bird.
- **60fps idle motion on a 5-year-old mid-range laptop.** Reference fixture: 2019 ThinkPad-class hardware, software-rendered fallback if no GPU. Measured as p99 frame time ≤16.7ms on a 30-minute run.
- **No memory growth over 30 minutes.** A CI test runs a 30-minute headless session with synthetic interaction and asserts heap-used end-of-run is ≤ start-of-run + small floor (e.g., +5MB). Failures block the release.

### How we hit these

- Bundle: code-splitting on settings, accessibility, visit-invitation; tree-shaking on icon and motif libraries; no recorded audio in bundle.
- TTFB: SSR/edge-rendered HTML with inline initial snapshot; early bird-asset loading via `<link rel=preload>`; render path that doesn't await non-critical assets (the first bird is a primary-render asset).
- 60fps: Canvas2D, layered canvases, bitmap caches, particle pool, render-budget guardrail, idle-motion is small-vector math.
- Memory: object pooling everywhere; explicit cleanup on call end, on bird flight transitions, on notebook scroll-out (`IntersectionObserver`-based).

### Observability

- **Aggregate-only RUM**: page load timings, first-bird-render timings, render frame-time histograms, audio-context error counts, reduced-motion enabled rate (boolean histogram), captions enabled rate. No per-account or per-bird dimensions.
- **Synthetic perf fleet**: a scheduled fleet of headless browsers in 4–6 geographies running canonical aviary sessions, asserting all four budgets per release.
- **Simulation observability**: tick latency p50/p95/p99 per shard, queue depth, per-tick events-consumed histogram. Per-account data is not in this stream; it's keyed by shard.
- **Error budget**: simulation-tick p99 latency >5s alarms.
- **Auth observability**: magic-link issue rate, consume rate, expiration rate. No per-email metrics.
- **Visit observability**: invitation-issue rate, consume rate, abandonment rate. No per-host metrics.

### What we deliberately do NOT measure

- Per-bird drift trajectories aggregated across accounts.
- Per-account session frequency (would tempt us to add streaks).
- Per-user listen-in or offer rates aggregated.
- Anything that could reconstruct a user's relationship with their birds.

The "what we don't measure" list is an architectural rule: telemetry pipelines never touch the simulation DB; the DB role exposed to the analytics warehouse is `null` (i.e., the role doesn't exist).

---

## 11. Rollout

### v1 launch sequence

1. **Internal alpha (week 0–4):** team accounts, single shard, full feature set. Calibrate drift weights against the test harness; calibrate notebook entry cadence. Tune reduced-motion aesthetic; record screen-reader user feedback if available.
2. **Closed beta (week 5–8):** a small invite list. Cap birds per aviary at 4 to keep the audio-mix conservative while we observe chorus behavior at scale; widen to 7 if the listenability holds. Enable account export and account deletion.
3. **Open launch (week 9):** public sign-up; cap held at 7. Visit-invitation feature shipped on day 1; visit-notify defaults off.

We deliberately ship with the bird-cap behavior tunable per-account so that a stuck recognizability problem during beta doesn't require a release to lower it.

### Birds-per-aviary ramp

The cap is 7. New accounts start with 2. Aviary-age-driven offers introduce new birds at intervals:

- 1st new bird (3rd total): around 3–4 weeks of aviary age.
- 2nd new bird (4th): ~8–10 weeks.
- 5th: ~3–4 months.
- 6th: ~6 months.
- 7th: ~9–12 months.

Numbers tuned to "match the rhythm of a relationship deepening" per `bird_engine.md`; the offer surfaces in the field notebook + a small top-bar affordance, not as a pop-up. The user can decline; declining doesn't affect personality drift.

### Day-from-one instrumentation

- Aggregate RUM and synthetic perf live on day 1.
- Simulation-tick observability live on day 1 (we cannot ship without it).
- Privacy boundary tests in CI on day 1 — DB-role tests, telemetry-shape tests, "no email outside accounts table" tests.
- Calibration tests (drift target hits) in CI on day 1.
- Memory test, bundle-size gate, 60fps test in CI on day 1.

### Post-launch tuning

Drift weights, mood transition probabilities, notebook entry thresholds, and tick cadence are all configurable via a server-side config that ramps gradually (5%, 25%, 100%) on change. Changes ramp across the user base over a week, not a deploy. This is a soft constraint to avoid users feeling a "their birds changed" moment from a config flip.

### Migration safety

Bird identity is stable across migrations (per `bird_engine.md`). Schema migrations that touch `birds`, `bird_personality`, `bird_mood` go through the standard online-migration path (add column, dual-write, backfill, switch read, drop old). A bird's `id` column is never changed under any migration; its row is never deleted.

---

## 12. Risks and mitigations

### Drift calibration risk

**Risk:** Drift is too fast (Tamagotchi-feel) or too slow (screensaver-feel).
**Mitigation:** A calibration test harness with three reference profiles (light visitor, regular visitor, heavy visitor) asserts instrument-detectable drift at ~7 days and visible drift at ~21 days for the regular profile. Drift weights are tuned against this harness, not against a single example. The test is a release-blocker, not a manual judgment.

### Sync correctness risk

**Risk:** A code path slips that lets a client write personality state, or last-write-wins behavior is reintroduced under a refactor.
**Mitigation:**
1. The API service's DB role has no write privilege on `bird_personality`. Trying to write fails at the DB layer.
2. A test asserts that no API handler issues an UPDATE/INSERT on personality tables.
3. Personality writes happen only in the simulation worker, which is a different process with its own role.
4. Code review checklist explicitly calls this out; PRs that touch personality writes route to a small named owner list.

### Audio uncanniness risk

**Risk:** Procedural calls slip into "synthesizer-y," not "bird-y." The chorus blurs into noise. A user notices their bird's call sounds the same twice.
**Mitigation:**
1. Listening tests across the team during alpha; record canonical "Pip mornings" and review weekly.
2. Per-bird voice signature is frozen at adoption and unit-tested for stability.
3. Motif library is hand-authored per species by an audio designer; not a generic synth library.
4. A "two-call sameness" automated test: rendered call audio across two adjacent calls of the same bird in the same mood must differ above a small spectral-distance threshold.

### Accessibility regression risk

**Risk:** A new feature ships without reduced-motion support, narration, or captions, creating a "v1.1 fix" — i.e., the failure mode the PRD explicitly names.
**Mitigation:**
1. PR template requires explicit confirmation of reduced-motion path, narration update path, caption path for any visual feature.
2. CI runs a reduced-motion render snapshot test; new features must pass it.
3. Screen-reader-narration coverage is asserted for every new bird-state surface.

### Privacy boundary regression risk

**Risk:** An engineer reaches for `email` as a key somewhere; per-bird state leaks into telemetry.
**Mitigation:**
1. Static analysis: a linter rule that flags any reference to the `email` column outside `accounts.repository`.
2. Telemetry schema is allowlisted; new dimensions require schema-PR review.
3. Quarterly audit: scan all log lines and Kafka keys for any field name that could be PII.

### Gamification creep risk

**Risk:** A "harmless" engagement feature lands (a session counter, a green-dot calendar, a "you visited every day" surface).
**Mitigation:**
1. PR template includes the explicit non-goals checklist; reviewer must confirm.
2. The product brief and non-goals are loaded into onboarding and into the design-review process.
3. Notebook templates are reviewed for any phrasing that surfaces user-behavior; templates must be observations of the aviary, not of the user.

### Performance budget regression risk

**Risk:** Bundle bloat over time; frame-time regressions on weaker hardware.
**Mitigation:** All four perf budgets are CI gates, not goals. A bundle-budget regression fails the build. The 30-minute headless memory test runs on every release. The synthetic perf fleet runs on a schedule; alerts on regression.

### "Welcome back" toast risk

**Risk:** A well-meaning contributor adds a textual welcome on return.
**Mitigation:** A test asserts that no DOM node containing the strings "welcome back" / "great to see you" / "you've been gone" / "days since" / "streak" / "achievement" / "level up" / "badge" / "score" is rendered. Crude, but the kind of guard that catches the obvious mistake on PR.

### Visitor flow abuse risk

**Risk:** Open-ended visitor links could be shared beyond the named friend; abuse via spammy visit traffic.
**Mitigation:**
1. Visitor link is scoped to the named email (we don't enforce email match at visit time, but the link is single-emailed to that address).
2. Visitor sessions rate-limited per token.
3. Host can revoke per `Visit revocation surface`.
4. Visitor IP hash retained 7 days for abuse triage; nothing longer.

### "First bird" perf risk on cold edge

**Risk:** Cold-cache or slow-network user sees the quiet field for several seconds, which begins to read as "broken."
**Mitigation:** The quiet field has a slow color modulation that signals continuity (very subtle); a snapshot-arrived event triggers smooth bird arrival. We instrument cold-load time and alert if p95 exceeds 2s.

### Mood snap on tab open

**Risk:** Bug where the client renders a default mood briefly before the snapshot lands.
**Mitigation:** The bootstrap inlines the snapshot; no client-side default mood exists. The render path requires a snapshot before drawing any bird; if no snapshot, render the quiet field, never a default-mood bird.

### Notebook prose drifting toward generic

**Risk:** Templates accumulate generic phrasing over time; a "your bird is happier!" sneaks in.
**Mitigation:** Notebook templates are change-controlled; PR review by a named voice owner. The templates are tested against a "no announcement phrasing" linter (rejects "achievement", "earned", "your bird is", "great job", "X in a row", etc.).

---

## 13. Tests / calibration (cross-cutting)

This section pulls together all the named CI gates for ease of audit:

- Drift calibration test (regular profile hits ~7-day instrument detection, ~21-day user-perceptible).
- Drift monotonicity test (no negative drift under any input).
- Personality-write-by-API test (forbidden).
- DB-role privilege test (API role cannot write `bird_personality`).
- Email-as-identifier scan (no email outside `accounts`).
- Bundle-size gate (≤2MB gzipped).
- TTFB synthetic test (≤500ms on reference device).
- 60fps long-run test (p99 frame ≤16.7ms over 30 min).
- Memory growth test (no growth over 30 min).
- Two-call sameness test (spectral diff above threshold).
- Reduced-motion render snapshot test.
- Caption generation completeness test (every motif has a caption template path).
- "Welcome back / streak / achievement" anti-toast string test.
- Notebook generic-phrasing linter.
- Visit revocation latency test (next snapshot pull post-revoke is unavailable).
- Snapshot-only-source-of-truth test (mood snap on tab open is not possible from default state).

---

## 14. Open implementation details (defensible to defer)

These are real choices that don't need to be made at planning time but should be flagged so they aren't left to the last engineer:

- Exact normalized ranges for personality traits (e.g., [0, 1] vs [0, 100]) — engine internal.
- Tick cadence final value (60s nominal; calibrated during alpha).
- Activity window for presence (4 minutes nominal; calibrated during alpha).
- Listen-in disengage trigger on click-empty-aviary — define "empty" precisely (any pixel that isn't a bird's hit-region).
- Nightjar-like night-active species — exact call signature.
- Visit log retention (PRD says no per-account telemetry; the visit log is host-visible operational data, retained while account exists, hard-deleted on account hard-delete).
- Bird offer presentation surface — proposed: a single field-notebook entry plus a small top-bar pulse, never a modal. (Defensible call, refusable.)

---

## 15. Summary

Pocket Aviary v1 is a small browser product whose architectural choices fall out of two refusals: no client-owned simulation state and no negative-drift / no announcement / no gamification. Both refusals are load-bearing; both are encoded in code-level guards (DB roles, monotonicity tests, anti-toast linters, telemetry-shape tests) rather than just design intent. The plan above is sized to ship a 7-bird-cap, multi-device-synced, accessibility-first-class, performance-gated v1 that holds the affective claim — "feels alive over weeks" — under engineering pressure, not in spite of it.
