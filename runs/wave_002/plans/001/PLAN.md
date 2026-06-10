# Pocket Aviary — v1 Implementation Plan

This is the executable engineering plan for Pocket Aviary v1, derived from the nine PRD files. It interprets the spec into architecture, data model, API, engine design, rendering, audio, accessibility, performance, rollout, and risk work. Where the PRD leaves a decision open, this plan makes a defensible call and flags it inline as **[Call]** with the reasoning. The plan assumes a frontier engineering team of roughly 6–8 engineers (2 server/simulation, 2 rendering/client, 1 audio, 1 accessibility+prose systems, 1 infra/platform, 1 floating) plus a visual designer who owns the design-system spec referenced by the PRD.

---

## 1. Scope

### In scope for v1

- Single-user accounts; email magic-link sign-in; per-device revocable session tokens; email change with verification; account export (emailed JSON snapshot); soft-then-hard account deletion (30-day window).
- One canonical aviary per account, advanced by a server-side simulation tick (~1/min), with multi-device sync as an architectural property (server is the only writer of canonical state).
- Bird engine: hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift driven primarily by presence-time; fast-timescale mood (enumerated states) persisting across sessions; bird-to-bird interaction; stable bird identity; species pool of ~6; adoption flow with two system-selected starters; age-gated growth to a cap of seven birds; user naming/renaming.
- Interactions: return-greeting (procedurally varied, absence-aware, boldness/mood-shaped, staggered multi-bird); listen-in (gradual mix re-balance, never mute); offer (seed / song fragment / still pool, per-bird cooldown); settle (opt-in soft session end with 5-second undo); precise three-condition presence accounting; read-only auto-generated field notebook with sparse naturalist entries.
- Scene: single horizontal one-screen scene, three perch zones, local-time day/night cycle, rare ambient weather, ambient client-side micro-motion (leaves/feathers, subtle parallax), no in-scene UI chrome, fading sparse top bar, load-with-motion-in-progress (quiet-field loading state, no spinner), empty-aviary state for onboarding only, responsive without ever cropping a bird.
- Audio: fully procedural client-side WebAudio call synthesis from per-species motif libraries; per-bird recognizable call signatures; real chorus mixing; listen-in mix ramps; graceful-silence fallback with captions on by default when WebAudio is unavailable. No recorded audio anywhere.
- Accessibility as a designed surface shipping with v1: slow-cadence naturalist screen-reader narration; reduced-motion mode as a distinct cross-fade rendering register; runtime-generated call captions; WCAG AA contrast on all user copy; full keyboard navigation with visible focus treatment.
- Social: per-invite, opt-in, revocable, read-only ambient visits via emailed one-time link; 30-day invite expiry; visit log in settings; visit notifications off by default with a per-account opt-in toggle. Visitors generate no presence or interaction signal.
- Privacy: synthetic UUID account identifiers everywhere except the single encrypted email field; hard pipeline-level separation between aggregate operational telemetry and per-account simulation state.
- Performance: ≤2MB gzipped initial JS; first bird visible <500ms on mid-tier mobile over 4G; 60fps idle motion on a five-year-old mid-range laptop across a 30-minute session; zero client memory growth over 30 minutes (CI-enforced); synthetic perf fleet + aggregate-only RUM; simulation-tick p99 latency alarm at 5s.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface otherwise.

### Out of scope for v1 (respected throughout this plan)

Native apps; payments; shared/multi-aviary accounts; gamification of any kind (no streaks, achievements, levels, counters, visit calendars — including all disguised variants); Tamagotchi mechanics (no death, hunger, distress, decaying meters); social-network surfaces (no profiles, follows, discovery, comments, leaderboards, co-presence, chat, visitor avatars); push/email/ping notifications about the aviary; customizable scenes; user-visible personality numbers in any form, at any tier, behind any toggle. The plan does not design data models or protocols around hypothetical future native clients.

Two structural consequences worth stating once so they propagate: (1) nothing in the schema, API, or telemetry computes or stores cross-account comparative stats — the absence is architectural, per `social_optional.md`; (2) every user-facing string is owned by the prose system (Section 9.1) so the naturalist/matter-of-fact voice split is enforced by code organization, not review vigilance.

---

## 2. Architecture

### 2.1 Service shape

Monorepo, TypeScript end-to-end. Three deployable server components plus the web client, sized for a small team rather than a microservice fleet:

1. **API service** (stateless, horizontally scaled, Node.js/TypeScript). Terminates auth, serves state snapshots, accepts interaction-event writes, serves notebook pagination, handles invites/visits, account settings, export, deletion. No simulation logic.
2. **Simulation service** (the tick runner; horizontally scaled workers with per-aviary leasing). Sole writer of canonical bird state. Consumes the append-only interaction-event log, applies drift deltas, transitions moods, advances weather and aviary timers, emits notebook candidate events, and publishes fresh snapshots.
3. **Mail worker** (small async worker). Sends magic links, visitor invitation links, export links, opt-in visit notifications, deletion confirmations via a transactional email provider (Postmark or SES). **[Call]** Postmark for v1 for deliverability quality on low-volume transactional mail; abstract behind a `Mailer` interface so the provider is swappable.

**Storage:** PostgreSQL as the single source of truth (accounts, birds, event log, notebook, invites). Redis for short-lived coordination only: tick leases, rate-limit counters, magic-link/one-time-token nonce tracking, and the active-aviary set. Nothing durable lives in Redis.

**Edge:** a CDN (Cloudflare or Fastly) serving the HTML shell and static assets, plus an edge worker that accelerates first-snapshot delivery (Section 8.2). **[Call]** Cloudflare Workers + KV for the snapshot edge cache; it has the simplest write-through API for our shape.

### 2.2 Client/server split — the one-writer rule

The split is dictated by `accounts_sync.md` and is the core invariant of the system:

- The **server** owns all canonical state: personality vectors, moods, perch assignments, aviary timers, weather, notebook. Only the simulation service writes this state. The API service writes only the event log and account/invite records.
- The **client** is a renderer and an event reporter. It pulls snapshots, interpolates between them, runs purely cosmetic local effects (leaf drift, micro-motion phase, call synthesis timing within server-provided envelopes), and appends interaction events. No code path on the client mutates personality or mood; there is no client-side write API for those records at all — the failure mode is made unreachable by the API surface, not by discipline.
- **Render pipeline boundary:** the snapshot is the boundary contract. Everything below it (drift math, mood machine, weather) is server-side and deterministic; everything above it (positions between snapshots, animation curves, audio realization, ambient ornaments) is client-side and ephemeral. A bird's *being* is server state; a bird's *appearance this frame* is client interpolation. Ambient leaves/feathers are explicitly client-only with no simulation state, per `aviary_layout.md`.

### 2.3 Logical-continuous tick, physically hot/lazy

`accounts_sync.md` requires the aviary to advance "whether or not any client is connected." Ticking every dormant aviary every minute forever is wasteful and adds nothing observable, because drift is monotonic (absence produces zero drift input) and mood/weather evolution is a deterministic function of elapsed time, timezone, and the (empty) event log.

**[Call]** Implement a *logically continuous* tick with two physical modes, plus a tested equivalence invariant:

- **Hot mode:** aviaries with an active session, or any interaction event or snapshot pull in the last 24h, are members of the Redis active set and are ticked every ~60s by simulation workers (workers claim per-aviary leases; one aviary is never ticked concurrently by two workers).
- **Lazy catch-up:** for all other aviaries, the next snapshot request (or the next hot-mode promotion) triggers a deterministic catch-up: the tick function is applied over the elapsed interval in simulation steps (coarsened where nothing can change — long empty stretches collapse to closed-form mood/weather evolution).
- **Invariant (CI property test):** for any state S, event log E, and interval T, `catchUp(S, E, T) === composeTicks(S, E, T)` bit-for-bit. The user-observable property the PRD cares about — "the aviary you return to is the aviary that has been running" — holds exactly; we just don't burn CPU proving it to nobody.

All tick computation is deterministic given (state, events, time range, seeded RNG stream per aviary). Determinism is what makes the lazy mode safe, replay debugging possible, and the drift calibration harness (Section 11) trustworthy.

### 2.4 Identity and privacy at the architectural layer

- Account IDs are synthetic UUIDv7, generated at account creation. The email address is stored exactly once, encrypted at rest (application-layer AES-GCM with a KMS-held key) on the account record. Every other table, log line, queue message, metric, and cache key uses the UUID. A CI lint forbids `email` columns/fields outside the two sanctioned tables (accounts, invites — visitor emails are likewise encrypted and live only there).
- The telemetry pipeline (Section 10) is a physically separate write path: app servers emit operational metrics to the metrics stack; nothing in the metrics stack has credentials to the simulation database, and no metric schema admits an account-ID dimension. The privacy line in `accounts_sync.md` is enforced as network topology and schema review, not policy text.

---

## 3. Data model

PostgreSQL schema (types abbreviated; all PKs UUIDv7 unless noted; all timestamps `timestamptz`).

### 3.1 Accounts and auth

```
accounts
  id                uuid PK
  email_encrypted   bytea          -- the ONLY place email lives (besides invites)
  email_fingerprint bytea UNIQUE   -- HMAC for lookup-by-email without decryption
  created_at, updated_at
  timezone          text           -- last-reported IANA tz; drives offline mood time-of-day
  deletion_state    enum('active','soft_deleted')
  delete_after      timestamptz NULL   -- hard-delete deadline when soft_deleted
  visit_notifications_enabled  bool DEFAULT false
  settings          jsonb          -- accessibility prefs (captions, reduced-motion override), audio prefs

auth_tokens (magic links)
  id, account_id FK, token_hash bytea, purpose enum('signin','email_change','export'),
  created_at, expires_at (15 min for signin), consumed_at NULL
  -- consumption is an atomic compare-and-set on consumed_at; replay is impossible

sessions (per-device)
  id, account_id FK, token_hash, created_at, last_seen_at,
  device_label text,            -- coarse UA-derived label for the session list
  revoked_at NULL
```

### 3.2 Birds and aviary

```
species (static seed data, ~6 rows)
  id, slug, silhouette_ref, default_palette jsonb,
  motif_library_ref text,       -- key into the bundled call-grammar motif sets
  is_night_active bool          -- the nightjar-like species

birds
  id                uuid PK     -- the stable identity; NEVER reissued or rewritten
  account_id        FK
  species_id        FK
  name              text        -- user-assigned, renameable, no engine effect
  adopted_at        timestamptz
  call_seed         bigint      -- immutable; derives the recognizable call signature
  personality       jsonb       -- {boldness, social_warmth, vocal_freq, plumage_sat, curiosity} ∈ [0,1]
  mood              enum('wary','content','curious','drowsy','alert','settled')
  mood_updated_at   timestamptz
  perch_zone        enum('front','middle','back')
  retired           bool DEFAULT false   -- safety valve; unused in v1 product flows

aviary_state (1:1 with accounts)
  account_id PK/FK
  last_tick_at      timestamptz
  rng_stream_state  bytea       -- seeded per-aviary RNG cursor for deterministic ticks
  weather           jsonb       -- {kind: none|rain|wind, started_at, ends_at}
  next_adoption_offer_at  timestamptz   -- aviary-age-driven growth gate
  pending_offer_species   uuid NULL
  last_presence_ended_at  timestamptz   -- drives absence-length greeting variation
```

**[Call]** Personality traits are normalized floats in [0,1], stored to full precision, seeded per species within calibrated bands (e.g., boldness seed drawn from a species-typical range so a warier species starts lower). Exact seed bands are simulation-service constants under config control (Section 5.1). The PRD says the user never sees these numbers — no API response outside the account-export endpoint ever serializes `personality`, and the export is the named, deliberate exception (it's the user's data; it is not a UI surface). **[Call]** Export includes raw vectors per `accounts_sync.md`'s explicit list; the no-numbers rule governs product surfaces, and an emailed JSON file is a data-portability surface. We will confirm this reading with product before GA; the fallback is exporting qualitative descriptors instead.

### 3.3 Event log and presence

```
interaction_events (append-only; partitioned by month)
  id                bigserial PK            -- server-assigned total order
  account_id        FK (indexed)
  bird_id           FK NULL                 -- e.g., listen-in target; NULL for aviary-wide events
  type              enum('presence_ping','listen_in_start','listen_in_end',
                         'offer','offer_reaction_ack','settle','settle_undo',
                         'session_open','visibility_change')
  payload           jsonb                   -- e.g., offer kind; presence sample window
  client_ts         timestamptz
  server_ts         timestamptz DEFAULT now()  -- authoritative for all engine math
  idempotency_key   uuid                    -- (account_id, idempotency_key) UNIQUE
  consumed_tick_at  timestamptz NULL        -- set when the tick has folded it in

presence_rollups
  account_id, hour_bucket, presence_seconds int
  -- written by the tick as it consumes presence_pings; raw pings older than 7 days
  -- are deleted by a retention job. Drift only ever reads rollups + the live log.
```

Presence is the dominant drift input, so its path is the most carefully specified: the client samples the three-condition conjunction (visible ∧ focused ∧ input within the activity window) every 15s and batches a `presence_ping` every 60s carrying the count of qualifying samples. The server clamps: pings are credited at most 60s of presence per 60s of wall-clock per account (two devices simultaneously present do not double presence — presence is attention, and one human has one attention; **[Call]** this clamp is the honest reading of `concepts.md`). Server timestamps are authoritative; client timestamps are recorded for diagnostics only.

**[Call]** Activity window for the pointer/keyboard condition: **4 minutes**, leaning long per `interactions.md` ("watching birds without moving is the actual product"), exposed as a config constant for calibration.

### 3.4 Notebook, invites, visits

```
notebook_entries
  id, account_id FK, created_at,
  body text,                    -- final naturalist prose, lowercase
  source_kind text              -- detector that produced it (greeting_order, weather, quiet_stretch, …)
  -- read-only contract: no UPDATE/DELETE grants for the API service role

invites
  id, account_id FK, visitor_email_encrypted bytea, token_hash bytea,
  created_at, expires_at (30 days), revoked_at NULL, first_used_at NULL

visit_sessions
  id, invite_id FK, started_at, last_snapshot_at, ended_at NULL
  -- feeds the visit log: who (invite → email), when, approximate duration
```

There is deliberately no `streaks`, `visit_counts`, `achievements`, or any per-user behavioral aggregate table. The visit log is the single, settings-scoped exception (transparency about sharing), and `presence_rollups` exist solely as drift input — they are never serialized to any client surface, and the notebook generator is structurally unable to read them (Section 5.4) so "you visited every day this week" entries cannot exist.

---

## 4. API surface

JSON over HTTPS; session cookie (HttpOnly, SameSite=Lax) carrying the device session token. All mutating endpoints require an `Idempotency-Key`. Matter-of-fact voice on every error string. Rate limits per session and per IP at the gateway.

### 4.1 Auth and account

```
POST /api/auth/magic-link        {email}            → 202 always (no account-existence oracle)
GET  /api/auth/consume?token=…                      → sets session cookie; 15-min expiry; single-use
POST /api/auth/signout
GET  /api/account                                   → settings, session list, deletion state
GET  /api/account/sessions                          → device list
DELETE /api/account/sessions/:id                    → revoke device
POST /api/account/email-change   {new_email}        → verify-then-commit flow
PATCH /api/account/settings      {…}                → visit-notification toggle, a11y/audio prefs
POST /api/account/export                            → 202; mail worker emails download link
POST /api/account/delete                            → soft delete, 30-day window
POST /api/account/restore                           → "I changed my mind"
```

### 4.2 Aviary state pull

```
GET /api/aviary/snapshot
```

Returns the full render-ready state, kilobytes-sized:

```jsonc
{
  "server_time": "…",
  "aviary": { "settled_hint": false, "weather": {…}, "daylight_phase": "computed client-side from local time" },
  "birds": [{
    "id": "…", "name": "pip", "species": "…",
    "mood": "content",
    "perch": { "zone": "front", "slot": 2 },
    "pose": { "kind": "preen", "phase": 0.4 },         // mid-action first frame
    "motion_profile": { "tempo": 0.62, "scan_rate": 0.3, "fluff": 0.1 },  // qualitative, mood/personality-shaped
    "call_envelope": { "density": 0.45, "chorus_affinity": 0.7, "signature_seed": 81234… },
    "plumage": { "palette_ref": "…", "richness_tier": 3 },               // quantized; never the raw trait
    "greeting": { "role": "first", "form": "two_note_call", "delay_ms": 600 }  // present on session-open snapshot only
  }],
  "narration_seed": { … }    // shared inputs for the client prose engine (Section 9.2)
}
```

Note what the snapshot does *not* contain: personality numbers. Traits reach the client only as already-shaped qualitative parameters (motion tempo, call density, quantized plumage tier, greeting assignment). This makes the "never exposed numerically" rule hold even against a user reading the network tab — the raw vector simply never leaves the server outside account export. The greeting block is computed server-side at snapshot time (which bird greets, in what form, honoring boldness/mood/absence-length) so greeting selection is canonical and consistent across a refresh; the client realizes the timing and animation.

Clients pull snapshots on: tab becoming visible, render-frame gap >5s (suspend/resume detection), a low-frequency keepalive while visible (**[Call]** every 75s, jittered ±15s — comfortably under tick staleness without chatty polling), and after submitting an `offer` (to fetch the server-adjudicated reaction promptly). No WebSockets in v1; the cadences above keep worst-case staleness near one tick, which matches the product's pace. **[Call]** Polling over push: the aviary changes on a ~minute cadence; a socket fleet buys nothing the product can feel and costs operational surface.

### 4.3 Event writes

```
POST /api/aviary/events    { events: [ {type, bird_id?, payload, client_ts, idempotency_key}, … ] }
```

Append-only batch write. The API service validates shape, stamps `server_ts`, enforces idempotency, applies per-type sanity clamps (presence crediting, offer cooldown pre-check), and inserts. It never computes drift. Offers get a synchronous lightweight response *acknowledging* the offer and the server-chosen reacting-bird outline so the client can begin the approach animation; the canonical mood consequence lands at the next tick.

### 4.4 Notebook, invites, visits

```
GET  /api/notebook?cursor=…&limit=…        → reverse-chron entries, infinite scroll-back
POST /api/invites          {visitor_email} → mails one-time link
GET  /api/invites                          → outstanding + active, with expiry
DELETE /api/invites/:id                    → immediate revocation
GET  /api/account/visit-log                → visits with email, date, approx duration

GET  /api/visit/:token/snapshot            → read-only snapshot, same shape as 4.2 minus
                                             greeting/narration-seed personalization;
                                             404-equivalent matter-of-fact surface if
                                             revoked/expired
```

The visit snapshot endpoint accepts **no** event writes — there is no visitor event route at all, so visitor presence/interactions are unrecordable by construction. Revocation works because visitors live on snapshot pulls: the next pull after revocation returns the "visit no longer available" surface. Visit tokens are long random values, hashed at rest, bound to the invite row, expiring with it.

---

## 5. Simulation engine design

The engine is a pure, deterministic module (`@aviary/engine`) shared in source between the simulation service (authoritative execution) and the calibration harness (Section 11). Signature: `tick(state, events[], tInterval, rng) → {state', notebookCandidates[], snapshotDelta}`.

### 5.1 Drift function

Drift is a low-pass filter over presence and interaction signals, monotonic toward expressive, applied per tick by the server only.

For each trait `x` with calibrated rate constant `k_x` and input signal `u(t)` ∈ [0,1] for the tick interval:

```
x ← x + k_x · u(t) · (1 − x) · Δt_eff
```

- `(1 − x)` gives natural saturation — traits asymptote toward 1.0 rather than clipping, so long-tenured birds keep drifting, ever more slowly. No term can be negative: neglect yields `u = 0` and therefore zero drift, never reversal. The monotonic rule is structural in the formula, not a guard clause.
- `u(t)` per trait composes the weighted inputs from `bird_engine.md`: presence-time dominates all traits (weight ~1.0); listen-in adds to the focused bird's social warmth and vocal frequency (~0.5); accepted offers add curiosity (~0.3) and any near-bird offer adds boldness (~0.15); settle contributes nothing to drift (mood-only). Weights are config-controlled coefficients.
- **Diminishing returns within a session:** per-bird, per-day input accumulators pass through a soft cap (`tanh`-style) so a 6-hour marathon session is worth meaningfully more than a 20-minute one but not 18×. This protects the weeks-long pacing against outlier sessions and complements the offer cooldown.
- **Calibration targets as executable tests:** with the reference "regular visitor" presence script (20 min/day, 5 days/week), the harness must show (a) instrument-measurable drift (Δtrait ≥ 0.02) at day 7, and (b) threshold-crossing of at least one *expressed* tier (perch-zone propensity, call density band, plumage richness tier — the quantized expressions the user can actually perceive) by day 21 ± 4. `k_x` values are tuned in the harness until both gates pass and a single session never moves any expressed tier (the "no visible single-session change" rule as a test assertion).

Plumage saturation drives the quantized `richness_tier` (5 tiers) so visual change arrives in rare, look-back-noticeable steps rather than per-session gradients.

### 5.2 Mood machine

Mood is a per-bird enumerated state: `wary, content, curious, drowsy, alert, settled` (**[Call]** finalizing the PRD's open set with `settled` as the night/post-settle resting state). Transitions are evaluated each tick as a weighted scoring function, not a hard FSM — each candidate mood gets a score from:

- **Time of day** in the account's stored IANA timezone (alert weighting in early morning, drowsy near dusk, settled at night — except the night-active species, which weights alert at night).
- **Recent events** from the log slice (accepted offer → content; song-fragment exchange → curious for high-vocal birds; alarm-call propagation → wary).
- **Ambient events** (rain dampens, wind splits alert/wary by personality).
- **Personality gating:** boldness subtracts from wary's score; curiosity adds to curious; so the same stimulus yields different moods across birds, per `bird_engine.md`.
- **Inertia:** the current mood receives a stickiness bonus decaying with time-in-mood, so moods change on a believable timescale (tens of minutes) and never flap tick-to-tick.

Mood persists in the DB; on the user's return, the snapshot carries whatever the (hot or lazily caught-up) tick last computed — no resets, no snapping, by construction. Bird-to-bird spread (one wary bird raising neighbors' wary scores) and chorus emergence (co-occurring high-vocal call windows flagged for the audio envelope and notebook detectors) run inside the same tick pass.

### 5.3 Call-grammar runtime (server half)

Calls are synthesized client-side (Section 8.4), but *what kind of calling a bird is doing* is canonical. The tick computes each bird's `call_envelope`: density (calls per minute band, from vocal frequency × mood × weather damping), chorus affinity, and motif-weighting hints. The bird's immutable `call_seed` (derived once from bird id at adoption) parameterizes the timbre/pitch identity so the signature survives mood and drift — recognizability lives in seed-fixed parameters (base pitch band, timbre profile, signature motif interval), while mood/drift modulate only rate, ornamentation, and intensity. The client realizes individual calls inside the envelope with its own ephemeral randomness; two devices may hear different individual calls (cosmetic), but the same bird, with the same voice, calling the same amount (canonical).

### 5.4 Notebook generator

Runs as a tick post-pass over `notebookCandidates` emitted by detectors:

- **Detectors** (engine-side, reading only aviary/bird state and event-log facts about *birds*): greeting-order firsts ("pip greeted before wren today, first time this week"), weather moments, long-quiet stretches, perch-habit changes, chorus events, a new bird's first week, offer vignettes.
- **Sparsity controller:** a per-account token bucket targeting ~1 entry per 2–4 days at baseline, with bypass priority for genuinely rare detections (first greeting-order flip, third-bird arrival). Very active accounts stay sparse; the bucket, not session count, gates output.
- **Prose realization** through the shared prose system (Section 9.1): naturalist voice, lowercase, present-tense, slot-grammar templates with per-slot variation pools and an n-gram repetition guard so consecutive entries never share a sentence skeleton.
- **Structural privacy guard:** the generator's input type contains bird/aviary observations only — no presence totals, visit counts, or session metadata cross the type boundary. The "observations of the aviary, never of the user's behavior" line is a compile-time property.

### 5.5 Adoption, growth, identity

- Account creation seeds two starter birds: species pair chosen by the server with complementary call registers and palettes (a weighted draw avoiding near-identical pairs), seeded personalities, default name suggestions surfaced in the naming step. Presented as "the birds that arrived."
- `next_adoption_offer_at` is set from aviary age only (**[Call]** schedule: third bird offered at ~90 days, then roughly every 100–140 days with jitter, server-config; cap 7 enforced in the engine). The offer surfaces as a quiet in-flow moment (a new bird appears at the edge of the scene with an accept/let-it-move-on choice in naturalist voice), never a badge or modal celebration.
- Bird ids are immutable and never reissued; renames touch only `name`; no migration may rewrite a bird row's identity or vector (enforced by migration review checklist + a canary assertion comparing vector checksums across deploys).

---

## 6. Sync model

Mostly already implied by the architecture; stated here as the contract:

1. **Single canonical record.** One aviary row-set per account; the simulation service is its only writer. Multi-device sync is reading the same record — there is no merge, no reconciliation, no client-resident authoritative state.
2. **Additive, server-authored deltas.** Clients submit events ("listened in on Pip for 3 minutes"), never values ("boldness = 0.62"). The tick folds events in `id` (server receipt) order. Last-write-wins on personality is unreachable: there is no write path that accepts a personality value.
3. **Idempotent, replayable event writes.** `(account_id, idempotency_key)` uniqueness makes client retries safe over flaky mobile networks; the append-only log plus deterministic engine makes any historical state reproducible for debugging.
4. **Concurrent sessions are normal, not a conflict.** Laptop and phone open simultaneously both pull snapshots and both append events; the presence clamp (Section 3.3) prevents double-counted attention; the tick serializes everything. **[Call]** Settle is *session-scoped* for visuals: the settling device shifts to evening locally and logs a `settle` event (a mood-quieting input); another concurrently open device does not visually settle. The canonical engine effect is identical either way, and remote-settling someone's other screen would be surprising.
5. **Conflict surfaces are auth-level only.** The only user-visible "sync" failures possible are session/auth problems (expired link, timed-out session, load failure), and they use the matter-of-fact voice verbatim per `accounts_sync.md`. There is no data-merge UI because there is no data to merge.
6. **Snapshot staleness bounds.** Visible-tab worst case ≈ keepalive (≤90s) + one tick — within the product's natural pace. Suspend/resume and tab-restore paths force an immediate pull via the frame-gap detector.

---

## 7. Frontend rendering pipeline

### 7.1 Stack and structure

**[Call]** Renderer: **WebGL2 with a thin bespoke layer** (no Pixi/Three) drawing a small set of textured quads/strips with a custom bird-pose system; Canvas2D fallback path for WebGL context loss or absence. Rationale: the scene is one screen, ≤7 birds, ~4 depth layers — a general engine costs bundle budget (Section 10) and buys nothing. UI chrome (top bar, settings, notebook, dialogs) is **Preact** (3KB) in ordinary DOM, which also gives accessibility semantics for free where they belong.

App shell composition:

- **boot chunk** (target ≤150KB gz): snapshot fetch/decode, scene renderer, bird pose/draw system, day/night palette math, presence sampler. Everything needed to put a mid-action bird on screen.
- **lazy chunks:** audio engine + motif libraries; notebook UI; settings/account; invite flow; export; reduced-motion renderer register loads in boot *if* `prefers-reduced-motion` or the stored override says so (it replaces, not supplements, the motion code path).

### 7.2 Scene composition

Layered single-viewport scene: sky/background foliage (slow palette-shifting gradient + soft sprite clusters, subtle parallax ≤2% offset), middle plane (perch structures, the three zones as slotted anchor sets), bird layer, occasional foreground branch/leaf pass. Responsive layout solves perch-slot positions from viewport width with a min/max clamp; all birds always in frame (slot solver guarantees containment; portrait phones compress inter-perch spacing first, never crop). Day/night palette is computed from client local time continuously (sunrise/dusk ramps over ~40 min), independent of snapshot cadence; weather state arrives via snapshot and renders as light rain streaks/leaf ripple.

### 7.3 Birds: procedural pose animation

Birds are parametric 2D rigs (body, head, wings, tail, ~10 control points per species silhouette) driven by a pose graph: `perched-idle, preen, scan, head-tilt, weight-shuffle, fluffed, call, hop, short-flight, drink/bathe, sleep`. Idle behavior is a per-bird stochastic scheduler seeded from bird id + mood + `motion_profile`: a wary bird scans more and sits back; content preens; curious tilts toward sound events (including real call events from the audio engine); drowsy sits low and fluffed. Micro-motion (breathing scale ~1%, feather settle noise) runs continuously on every pose so stillness never reads as paused. All animation respects a global tempo limit — nothing strobes, nothing snaps (transitions ≥300ms eased).

Interpolation: between snapshots, birds move only via in-scene plausible actions — if snapshot N+1 moves a bird's perch, the client plays a hop/short-flight along an arc, never a teleport; mood changes re-weight the idle scheduler gradually over ~20s rather than visibly switching modes.

### 7.4 First frame and load states

The boot chunk renders the scene synchronously from the inlined/preloaded snapshot (Section 10.2): birds placed at `pose.phase` mid-action, ambient drift already moving, audio fading in from its lazy chunk a beat later (audio joins in progress; it does not "start"). No entry animation of any kind. If the snapshot isn't yet available at first paint, render the **quiet field**: soft sky-color gradient at the correct local-time palette with one or two faint drifting motion cues — explicitly not a spinner, not a skeleton screen, not a progress element — and birds enter their poses (already mid-action, no fly-in) the moment state arrives. The fly-in is reserved exclusively for the one-time empty-aviary → first-adoption moment.

### 7.5 Reduced-motion register

A parallel render register, not a degraded one: the pose graph is shared, but realization swaps frame-animation for slow cross-fades between held poses (≥2s fades, ≥8s holds), flights become cross-fades between perch positions, leaf/feather ornaments are removed, and day/night palette shifts remain but slowed. Engaged by `prefers-reduced-motion` or the in-product accessibility setting (which persists in account settings and syncs across devices). The register is selected at boot — it is a first-class build target with its own visual QA pass, owned by design, per `accessibility_perf.md`.

### 7.6 Top bar and interaction affordances

Thin DOM top bar above the scene: account/settings, accessibility settings, notebook, offer. Fades to ~5% opacity after 4s of cursor stillness; restores on pointer/keyboard activity; never fades while it holds keyboard focus (focus-visible suppresses the fade — an a11y requirement). Listen-in engages by click/tap/keyboard-Enter on a bird; the focused bird gets a soft scale/attention cue (no ring, no badge); disengage per `interactions.md` (re-click, other-bird focus, empty-space click, focus departure). Offer opens a small naturalist-voiced tray from the top bar (seed / song fragment from a ~6-piece library / still pool); cooldown state is shown by quiet unavailability, not a countdown timer (a timer is a meter; the tray simply offers what can be offered). Settle triggers the slow evening ramp with the 5-second any-click undo window.

---

## 8. Audio pipeline

### 8.1 Synthesis architecture

All bird audio is synthesized in a WebAudio graph; zero recorded audio assets ship. Per-species motif libraries are compact parametric data (pitch contours as breakpoint curves, duration ranges, harmonic/noise recipes, ornament sets — a few KB per species). A bird's voice is realized as: `call_seed` → fixed identity parameters (base pitch band, timbre recipe: 2-oscillator FM pair or filtered-noise source per species family, vibrato character, signature motif interval) + mood/trait modulation (rate, ornamentation, amplitude, contour stretch). Each call is freshly generated — motif selection, micro-jitter on every breakpoint, ornament dice — so no two calls are ever sample-identical, and no phase-cancellation artifacts can exist because nothing is a copy.

**[Call]** Synthesis uses a single `AudioWorklet` voice-pool processor (pre-allocated, reused voices; no per-call node allocation) to satisfy the no-memory-growth budget and keep GC pressure off the render thread. Output chain: per-bird gain → spatial pan (perch-zone-derived stereo position) → chorus bus → soft master compressor/limiter (gentle, for safety not loudness).

### 8.2 Scheduling and chorus

A client-side **aviary audio scheduler** consumes each bird's server-provided `call_envelope` and schedules concrete calls: Poisson-ish arrivals at the envelope density, with social coupling — a bird with high chorus affinity that "hears" (receives the scheduler event for) another call may answer within a beat-window, producing emergent call-and-response and, when several high-vocal birds coincide, a true chorus of independently synthesized voices. Time-of-day shapes the ambient bed (early-morning denser, night near-silent except the night-active species). Rain audio is a soft synthesized broadband wash that also ducks call density per the engine's weather damping.

### 8.3 Listen-in mix and settle

Listen-in re-balances, never mutes: focused bird ramps +6dB-ish over ~1.5s; others ramp down to an ambient bed (≈ −12dB, floor well above silence) over the same ramp; disengage reverses identically. Implemented as smooth `AudioParam` automation on the per-bird gains — no hard cuts anywhere in the product. Settle ramps the whole mix down over the evening transition to a sparse, quiet night bed. Tab-hidden: rendering stops; audio fades to silence over ~2s (**[Call]** continuing audio from a hidden tab reads as the product demanding attention; fading out respects the user's departure — and presence has ended anyway).

### 8.4 Captions and fallback

The synthesis engine emits, alongside each realized call, a structured description of what it actually built (note count, contour direction, tempo, intensity, source perch) which the prose system renders as the caption ("a soft three-note rise", "a low trill, paused, low trill again") — captions are generated from the realized call, never canned strings, per `accessibility_perf.md`. Captions render as small fading text near the calling bird, WCAG AA against scene extremes (subtle scrim under the text where needed).

If `AudioContext` is unavailable or fails (permission, hardware, old browser slipping past support gates): the aviary runs in graceful silence, captions auto-enable (with a quiet matter-of-fact note in accessibility settings explaining why), and the scheduler keeps running so caption timing still reflects real calling behavior. No recorded-audio fallback exists, unconditionally. Autoplay policy: start the context muted-gain and ramp in on first user gesture if the browser requires a gesture; before that, behave as the silence path (captions if enabled) rather than showing any "click to enable sound" announcement — the top-bar audio toggle is the discoverable control.

---

## 9. Accessibility surfaces

### 9.1 The shared prose system (cross-cutting)

One package, `@aviary/prose`, owns every user-facing string: notebook entries, screen-reader narration, captions, offer-tray copy, system/matter-of-fact strings. It has two registers with distinct types — `naturalist()` (lowercase, present-tense, no "you", no exclamations, no announcement framing) and `system()` (plain capitalized English) — and the voice rule from `product_brief.md` (money/identity/errors/settings → system; everything else → naturalist) is encoded as which register each surface is *allowed to import*. Slot-grammar templates with variation pools and repetition guards serve notebook, narration, and captions from the same engine, which is what makes the voice continuous across surfaces ("the same product, not two products glued together").

### 9.2 Screen-reader narration

A visually-hidden `aria-live="polite"` region carries running naturalist prose generated client-side from the same snapshot + local-event stream the visual renderer reads (the `narration_seed` provides stable descriptors). Cadence: one idle update per 30–60s, composed as observation prose ("a small grey bird is perched on the front rail, calling softly…"), never state lists. A small priority queue promotes user-initiated moments — return-greeting narrated promptly at session start, offer reactions as they happen, settle acknowledged — still written as observations, via `aria-live="assertive"` only for the return-greeting (**[Call]**: it is the anchor moment; everything else stays polite). Queue discipline: never more than one pending idle utterance; idle narration drops rather than backlogs. Bird focus (keyboard) also exposes a per-bird accessible name ("pip — a small grey bird on the front perch"), generated from the same descriptors, with no mood labels and no numbers.

### 9.3 Keyboard navigation and focus

Tab order: top-bar items → aviary scene (first bird) → after-scene chrome. Within the scene: arrow keys move focus among birds (spatial order, left-to-right then depth), Enter toggles listen-in on the focused bird, Escape exits listen-in, Tab exits the scene group. Offer tray and settle reachable from the top bar with full keyboard operability; the settle undo is also keyboard-triggerable (any key with the scene focused, within the 5s window). Focus indicator: soft high-contrast outline tuned by design against bright-noon and dim-night scenes (tested at both extremes); the top bar never fades while focused. The scene's interactive birds are real focusable elements (DOM overlay hit-targets positioned over the canvas, ≥44px touch targets) so focus, ARIA, and pointer handling share one system.

### 9.4 Contrast, captions, reduced motion

Covered structurally above: WCAG AA enforced on all chrome/captions/settings/system text via design tokens + automated contrast checks in CI on both day and night palettes; captions per 8.4; reduced-motion register per 7.5. Accessibility settings (captions, reduced-motion override, narration verbosity off/normal) live in account settings and sync across devices; the settings surface itself is system-voiced.

### 9.5 Verification

Every release: axe-core automated pass in CI; narration prose snapshot tests (voice-rule lints: no uppercase sentence starts in naturalist register, no "you", no exclamation marks, banned-word list — "achievement", "streak", "level", "score" fail the build in any user-facing string); manual screen-reader passes (VoiceOver/Safari, NVDA/Firefox) on the four core flows (open/greeting, listen-in, offer, settle) as a launch-blocking checklist item, recurring monthly.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced, launch-blocking)

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS, gzipped | ≤2MB hard; boot chunk ≤150KB | size-limit check in CI; PRs over budget fail |
| Time to first bird visible | <500ms, mid-tier Android over throttled 4G | Lighthouse/WebPageTest lab run in CI on every main merge; synthetic fleet in prod |
| Idle frame rate | 60fps sustained, 5-year-old mid-range laptop, 30-min run | nightly headless perf soak on a throttled-CPU profile (≈4× slowdown) asserting long-task and frame-time distributions |
| Client memory | zero net growth over 30-min session | nightly Puppeteer soak comparing heap snapshots after forced GC at minute 2 vs minute 30; tolerance band ~1%; regression fails the build |
| Simulation tick latency | p99 < 5s alarm; target p50 ≪ 500ms | prod metric + alert |

### 10.2 Hitting time-to-first-bird

The 500ms path: CDN-edge HTML with the boot chunk preloaded (`modulepreload` + HTTP 103 Early Hints); snapshot raced with the bundle — the edge worker authenticates the session cookie and serves the account's latest snapshot from edge KV (written through by the simulation service on each hot tick for accounts active in the last 24h; ~KB values, ciphered at rest in KV, short TTL), falling back to an origin fetch for cold/lazy accounts, in which case the quiet-field state covers the gap *as a designed surface, not a failure*. Fonts: system stack only in chrome (no webfont on the critical path). Bird assets are procedural rigs + small atlas textures (single ~100–200KB atlas for all six species), decoded off the critical path where possible.

### 10.3 Observability and the privacy boundary

Two pipelines, physically separate from the simulation DB:

- **Synthetic monitoring:** scheduled headless-browser fleet from ~5 geographies running scripted sessions against dedicated synthetic accounts — measuring load, first-bird time, frame timing, audio-context init, snapshot latency, and full-flow auth (magic-link round trip against a mail-trap inbox).
- **Aggregate RUM + server metrics:** page/first-bird timings, frame-time histograms, audio errors, API latencies/error rates, tick latency and queue depth, mail delivery outcomes, anonymized session-duration histograms with **no per-account dimension**. Metric schemas are reviewed against a written allowlist; the lint that bans account-id labels on metrics runs in CI. Nothing in any telemetry stream contains per-bird state or per-account interaction history — the `accounts_sync.md` line is enforced where metrics are defined.

What we deliberately do not measure: engagement funnels, retention cohorts keyed to product mechanics, per-account visit frequency, drift distributions across the population. (Drift calibration uses synthetic + consenting dogfood accounts only — Section 11.)

Alerting: tick p99 > 5s; tick scheduler lag (active aviaries overdue >3 ticks); snapshot p99 > 300ms origin; auth mail delivery failure rate; edge-KV write-through failure rate; client error-rate spike via aggregated error reporting (Sentry with PII scrubbing and no account dimension).

---

## 11. Rollout

### 11.1 Build order (six milestones; rough team-relative durations)

- **M0 — Foundations (2–3 wk):** monorepo, CI/CD, Postgres/Redis/edge provisioning, auth skeleton (magic link end-to-end with Postmark), synthetic-ID + encryption plumbing, telemetry stacks with the privacy lints live from day one.
- **M1 — Canonical vertical slice (3–4 wk):** engine package with deterministic tick + lazy catch-up equivalence test; snapshot + event-log APIs; boot-chunk client rendering two birds from a real snapshot, mid-action first frame, day/night palette, presence sampler wired through to rollups. *Exit: open the tab on two devices and watch the same canonical aviary.*
- **M2 — Bird engine depth + audio (4–5 wk):** full drift function under config coefficients, mood machine, bird-to-bird propagation, greeting selection, offers + cooldowns, settle; AudioWorklet synth, motif libraries for all six species, scheduler, chorus, listen-in mix. Calibration harness online (below). *Exit: a dogfooder can tell Pip from Wren by ear, blind.*
- **M3 — Voice and accessibility (3–4 wk):** prose system, notebook generator + sparsity controller, narration, captions, reduced-motion register, keyboard nav, adoption flow + empty-aviary moment. Accessibility QA process starts here, not at the end.
- **M4 — Accounts/social hardening (2–3 wk):** sessions UI, email change, export, deletion lifecycle, invites/visits/visit-log/revocation, rate limiting, abuse review (invite spam, presence clamps).
- **M5 — Performance + beta (3–4 wk):** edge snapshot path, budgets to green on real mid-tier hardware, memory soak, synthetic fleet live, then a ~100-account private beta running ≥4 weeks — the drift calibration window below needs that clock time; it cannot be compressed.

### 11.2 Drift calibration plan (the schedule-critical path)

Calibration starts at M2 and runs continuously, because three-week drift targets need three weeks of wall-clock to validate honestly:

1. **Simulated cohorts:** the deterministic engine runs accelerated synthetic populations (daily visitor, weekend visitor, marathon-then-absent, two-week lapse-and-return) over simulated months; coefficient sets are tuned against the day-7 instrument gate and day-21 visible gate, plus the lapse case asserting zero negative drift and "quieter, not punished" expression.
2. **Dogfood verification:** internal accounts flagged `calibration_consented` (the only accounts whose per-bird state any dashboard may read — flag checked at the query layer) verify that real human presence patterns land within the simulated envelope.
3. **Config-driven coefficients:** drift weights, activity window, cooldowns, mood scores, notebook bucket rates are server config with audit history — recalibration post-launch never requires a client release or a schema change.

### 11.3 Launch shape and ramps

- **Launch config:** two starter birds; third-bird offers **disabled at GA** and enabled by config once ~90-day-old beta aviaries have validated the offer flow — the birds-per-aviary ramp is a config unlock on aviary age, reaching the cap of 7 only as real aviaries age into it. Visits ship enabled (off-by-default per design). Magic-link auth has no ramp; it's the only door.
- **Flagged degradations, not dark launches:** feature flags exist for operational fallback (disable edge-KV path → origin snapshots; disable audio → silence+captions; disable visits) rather than for product experimentation. No A/B framework in v1 — the product's metrics philosophy gives us nothing to A/B against, and that's by design.
- **Instrumented from day one:** every Section 10.3 metric, the privacy lints, the budget CI gates, and the tick-equivalence property test are in place before beta, not retrofitted.
- **Support surfaces:** unsupported-browser page, status-page link in the error surface, and the privacy policy (plain text, naming aggregate categories and excluding per-bird state) live in account settings at GA.

---

## 12. Risks

Ordered by (likelihood × damage to the product's actual promise).

1. **Drift calibration misses the felt target.** Too fast → Tamagotchi; too slow → screensaver; and the failure is silent (no test fails when birds feel wrong). *Mitigations:* executable day-7/day-21 gates in the harness; accelerated cohort simulation from M2; ≥4 weeks of real-time beta observation; config-only coefficients for live correction; quantized expression tiers so visible change is step-wise and verifiable. *Residual:* the visible-at-3-weeks judgment is human; beta diaries from dogfooders are the instrument of record.
2. **Sync/canonicality bugs that delete drift.** A lost personality update is invisible until a user half-feels it — the PRD's named worst failure. *Mitigations:* no client write path for state (API-shape enforcement); single-writer tick with per-aviary leases; append-only idempotent log; deterministic engine with replay; the catch-up≡ticks property test; vector-checksum canaries across deploys and migrations; backups with point-in-time recovery rehearsed before beta.
3. **Audio uncanniness.** Procedural calls that read as synthesizer beeps — or signatures that aren't recognizable — collapse the affective spine and the seven-bird cap's rationale. *Mitigations:* audio engineer + designer listening reviews as a standing M2 ritual; a blind "which bird called?" recognizability test with dogfooders (target ≥80% correct at two weeks); per-species motif tuning isolated from engine code; chorus stress tests at 7 voices; the cap stays at 7 unless the recognizability test says otherwise. *Residual:* taste risk — schedule real iteration time, treat motif libraries as content with revision cycles, not assets done once.
4. **Accessibility regression to checklist quality.** The designed surfaces (narration, reduced-motion, captions) quietly degrade into ARIA automation under deadline pressure. *Mitigations:* prose system owns narration (it cannot be a state list without failing voice lints); reduced-motion is a build register with its own design QA; manual SR passes are launch-blocking; the M3 placement (before hardening, not after) keeps it on the critical path.
5. **Presence-signal corruption.** A lax or buggy presence definition silently inflates drift population-wide. *Mitigations:* the three-condition conjunction implemented as a single audited sampler module with unit tests per condition combination; server-side crediting clamps (≤60s/60s, multi-device collapse); calibration cohorts include "tab open, user absent" scripts asserting zero credit; activity window configurable for tuning.
6. **Time-to-first-bird misses on cold paths.** Edge-KV misses (lazy accounts, new regions) push past 500ms and the conceit leaks. *Mitigations:* quiet-field state is a designed surface with design sign-off, so the miss degrades gracefully; synthetic fleet measures cold and warm paths separately; write-through covers all hot accounts; the boot chunk stays ≤150KB so the bundle is never the bottleneck.
7. **Tick fleet scaling/cost.** Hot-set ticking grows linearly with actives; a stuck scheduler stalls every aviary at once. *Mitigations:* lazy catch-up bounds the hot set; leases + overdue-aviary alarms; tick work is O(birds + recent events) with no cross-account queries; load test at 100× projected beta before GA.
8. **Magic-link deliverability.** Auth is single-path; spam-foldered mail is a hard lockout. *Mitigations:* reputable transactional provider with domain warm-up, SPF/DKIM/DMARC from M0; synthetic auth round-trip monitoring; resend affordance with rate limits; matter-of-fact guidance on the sign-in surface.
9. **Privacy boundary erosion.** Some future dashboard "just needs" a per-account dimension. *Mitigations:* network separation, schema allowlist + CI lint, the calibration-consent flag as the only sanctioned exception, and the export/deletion lifecycle tested end-to-end (hard-delete leaves no orphan rows — verified by a sweep job).
10. **Voice drift in copy.** New strings slip into announcement register ("Welcome back!", a toast, a counter) and the product reads as theater. *Mitigations:* all strings through `@aviary/prose` registers; banned-pattern lints; the non-goals list converted into a PR-review checklist; design review owns any new user-facing surface.

---

## 13. Open calls made in this plan (summary for product sign-off)

1. Presence activity window default 4 minutes (config).
2. Presence credit clamped to 60s/60s per account across devices.
3. Mood set finalized as wary/content/curious/drowsy/alert/settled.
4. Settle is visually session-scoped; engine effect canonical.
5. Snapshot keepalive 75s ± 15s; polling, no WebSockets in v1.
6. Lazy catch-up tick with proven equivalence to continuous ticking.
7. Account export includes raw personality vectors (data-portability reading; confirm before GA).
8. Third-bird offer at ~90 days, config-gated off at GA until beta validates.
9. Audio fades out when the tab hides.
10. WebGL2 bespoke renderer + Preact chrome; no game engine.
11. Cloudflare edge KV snapshot write-through for hot accounts; Postmark for mail.
12. Return-greeting is the only `assertive` narration event.

Each is reversible at config or module level except #10 (renderer choice), which is therefore the first thing M1 validates with the vertical slice.
