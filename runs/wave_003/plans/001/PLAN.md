# Pocket Aviary — v1 Implementation Plan

Phase-1 plan derived from the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). Written so a separate engineering team can execute without further clarification. Where the PRD is silent, this plan makes a defensible call and marks it **[call]**.

---

## 1. Scope

### 1.1 In scope for v1

| Surface | What ships |
|---|---|
| Aviary scene | Single horizontal one-screen scene; three perch zones (front/middle/back); day/night cycle anchored to the user's local timezone; rare ambient weather (short rain a few times/week, occasional soft wind); ambient leaf/feather drift; subtle three-plane parallax; no pan/zoom/scroll; no UI chrome inside the scene. |
| Birds | 2 starter birds (species system-assigned, not user-picked, from a pool of ~6); user-assigned renameable names; cap of 7 birds; age-based adoption offers thereafter; stable internal bird IDs for life. |
| Bird engine | Server-side simulation tick (~1/min); 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift; enumerated mood states with cross-session persistence; procedural per-species call grammars; mood-shaped idle motion; bird-to-bird interaction (call response, mood contagion, emergent chorus). |
| Interactions | Return-greeting (one bird, staggered, procedurally varied, absence-length- and boldness-shaped); listen-in (gradual mix rebalance, others never silent); offer (seed / song fragment / still pool, per-bird cooldown of a few minutes); settle (opt-in, 5-second any-click undo, engine-equivalent to tab-close); presence accounting (strict 3-signal conjunction). |
| Field notebook | Auto-generated, read-only, naturalist-voice entries; sparse (~1 per few days, more when noteworthy); infinite scroll-back, no archive. |
| Accounts | Single-user accounts; email + magic-link sign-in (15-min expiry, single-use, per-email rate limit); per-device revocable session tokens; verified email change; JSON export emailed as a link; soft-delete 30 days then hard-delete; synthetic UUID as the only internal identifier. |
| Sync | Server-canonical state; clients pull snapshots and interpolate; append-only client event log; additive server-authored personality deltas (no last-write-wins, ever); multi-device coherence as an architectural property, not a feature. |
| Social | Email visit invitations only: per-invite opt-in, revocable with immediate effect, 30-day expiry on unused invites, read-only ambient visitor view (no interaction, no presence recording, no co-presence), silent visit log, host notifications opt-in and OFF by default. |
| Accessibility | Screen-reader naturalist narration (30–60s idle cadence, priority bump for user-initiated events); designed reduced-motion mode (cross-fade pose sequences, not "animations off"); runtime-generated call captions; WCAG AA contrast on all user copy; full keyboard navigation with visible focus indicators. |
| Performance | <2MB gzipped initial JS; first bird visible <500ms on mid-tier mobile over 4G; 60fps idle on a 5-year-old laptop for a 30-minute session; no client memory growth over 30 minutes (CI-enforced); WebAudio procedural audio with graceful-silence + captions fallback. |

### 1.2 Out of scope (hard refusals, per `non_goals.md`)

- **Native apps** — web-only; data model and protocols are not designed for native-client constraints.
- **Gamification** — no achievements, streaks, levels, scores, badges, counters, calendars of visits, XP, ranks, or tiers. Not as settings, not as opt-ins, not in any future version of this product. No surface anywhere observes the *user's* behavior back at them (notebook observes the *aviary* only).
- **Tamagotchi mechanics** — no death, hunger, distress, decaying happiness. Neglect yields ambient quietness, never punishment. Drift never moves a trait downward.
- **Social-network surfaces** — no profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments, chat, avatars, leaderboards, or visitor "show-off" rendering.
- Also excluded at v1: payments, shared/household aviaries, multi-aviary accounts, customizable scenes, push notifications of any kind, recorded-audio fallbacks, public aviary directories.

### 1.3 Voice governance (build-time rule, enforced in review)

Two registers, one named exception, zero drift:
- **Naturalist** (lowercase, present-tense, bird-named, specific, no exclamation, no "you"): aviary surface, notebook, narration, captions, offer prompts.
- **Matter-of-fact** (normal capitalization, direct): sign-in, account settings, sync/conflict errors, accessibility settings, unsupported-browser, visit-revocation surfaces. Any surface where the user engages the system *as a system* (money, identity, errors, settings) drops out of the naturalist register.

Enforcement: a copy-lint pass in CI that flags exclamation marks, "welcome", "congrat", "streak", "level", "score", "you visited", and announcement-framing patterns in product-surface strings, plus human copy review against the two style samples in the brief.

---

## 2. Architecture

### 2.1 Shape

Monorepo, TypeScript end-to-end (single language across client, server, and shared isomorphic modules — voice/prose generation and call-grammar descriptors are shared so client rendering and server state can never diverge in vocabulary).

```
apps/
  web/            # client: scene renderer, audio engine, a11y surfaces, thin DOM chrome
  api/            # HTTP API: auth, snapshots, event ingestion, notebook, invites, account
  sim/            # simulation workers: tick loop, drift, mood, notebook writer, weather
packages/
  domain/         # isomorphic: types, mood model, drift math, call-grammar descriptors,
                  #   narration/notebook prose generator, caption generator
  proto/          # wire schemas (TS types + zod validators shared by client and server)
infra/            # IaC, migrations, dashboards
```

Services at runtime (v1 deliberately small — one region, boring with teeth):

1. **api** — stateless HTTP service (Node 22 + Fastify **[call]**), horizontally scalable behind a CDN/edge. Serves the app shell, snapshots, event ingestion, auth, account, invites, notebook.
2. **sim** — partitioned tick workers. 64 logical shards keyed by `account_uuid`; each shard is a sequential once-per-minute loop over its accounts. Workers claim shards via short leases (Postgres advisory locks **[call]** — no extra infra at v1 scale); a dead worker's shards are re-claimed within one tick period.
3. **Postgres** — canonical store: accounts, birds, personality vectors, moods, event log, notebook, invites, sessions. Point-in-time recovery enabled; nightly logical dumps of the sim tables (see §12, personality-loss risk).
4. **Redis** — ephemeral only: rate limits (magic-link requests), magic-link token hashes with TTL, idempotency keys, edge snapshot cache (30s TTL per account, invalidated on tick write). Nothing canonical lives here; losing Redis is a performance event, never a data event.
5. **Email** — transactional provider for magic links, invites, exports. Auth-critical: provider failure = sign-in failure, so deliverability monitoring and a runbook from day one.
6. **Static edge** — app shell + assets on CDN; the initial aviary snapshot is inlined into the HTML at the edge (see §10.2).

### 2.2 Client/server split (the render pipeline boundary)

**Server owns**: personality vectors, mood state and transitions, drift, perch selection, call scheduling intent, weather state, notebook authorship, presence accounting, adoption offers, all persistence. Server is the *only* writer of personality state under any code path.

**Client owns**: rendering, interpolation between snapshots, procedural audio synthesis, idle micro-motion sequencing *within* server-provided behavioral bounds, caption/narration generation from state, presence-signal detection, event batching.

Boundary rule: the server sends *what is true* (positions, moods, current motion intents, timing hints); the client decides *how it looks and sounds this exact frame* (pose blending, jitter, synthesis variation). The server never sends frame data; the client never sends state. The client may stop rendering when hidden; the simulation never stops.

### 2.3 Why no websockets at v1 **[call]**

The canonical cadence is ~1 snapshot/minute with client-side interpolation. A 30s keepalive poll plus pull-on-visible and pull-after-long-frame-gap covers every sync requirement with stateless HTTP, trivial edge caching, and no connection inventory to manage. SSE is the designated upgrade path if a future feature needs sub-minute push; nothing in v1 does. This keeps the API service fully stateless, which keeps multi-device sync correctness a pure function of the database.

---

## 3. Data model

All internal references use the synthetic account UUID. Email appears exactly once, encrypted at rest (application-level envelope encryption), on the account row. No per-account dimension exists in any telemetry table — the telemetry schema is a separate database with an allowlisted metric definition set (§10.4).

### 3.1 Tables

**`accounts`**
```
id              uuid pk                 -- synthetic, generated at creation
email_encrypted bytea                   -- envelope-encrypted; only PII copy
email_key_id    text                    -- KMS key reference
timezone        text                    -- IANA tz; drives day/night + mood time-of-day
settings        jsonb                   -- {call_audio: bool, captions: bool,
                                        --  reduced_motion: 'system'|'on'|'off',
                                        --  visit_notifications: bool (default false)}
created_at      timestamptz
deleted_at      timestamptz null        -- set = soft-deleted; hard purge at +30d
```

**`sessions`** — per-device refresh tokens
```
id uuid pk, account_id fk, device_label text, token_hash text,
created_at, last_seen_at, revoked_at null
```

**`birds`** — one row per bird, identity immutable
```
id              uuid pk                 -- THE bird; never regenerated
account_id      fk
species         text                    -- one of ~6 pool entries
name            text                    -- renameable; no behavioral effect
personality     jsonb                   -- {boldness, warmth, vocal, plumage,
                                        --  curiosity} each real in [0,1]  +
                                        --  internal estimators (§5.3)
mood            text                    -- enum, §5.4
mood_entered_at timestamptz
perch           text                    -- 'front' | 'middle' | 'back'
pose_hint       jsonb                   -- current motion intent for clients
adopted_at      timestamptz
```

**`events`** — append-only interaction log; the only client-writable table
```
id          bigserial pk
account_id  uuid
bird_id     uuid null
type        text      -- presence_ping | presence_end | listen_in_start |
                      -- listen_in_end | offer | settle | settle_undo |
                      -- adoption_accept | rename
payload     jsonb     -- e.g. offer kind; listen-in duration; session id
client_ts   timestamptz
server_ts   timestamptz default now()
idempotency_key text unique null
```

**`tick_cursors`** — per-account consumption cursor (makes ticks idempotent)
```
account_id uuid pk, last_event_id bigint, last_tick_at timestamptz, state_version bigint
```

**`notebook_entries`**
```
id uuid pk, account_id fk, day date, kind text, text text, created_at
-- unique(account_id, day, kind) for sparsity dedupe
```

**`invites`**
```
id uuid pk, account_id fk, email text (host-entered; purged on hard delete),
token_hash text unique, status 'open'|'revoked'|'expired',
created_at, expires_at (created + 30d), revoked_at null
```

**`visits`** — host-visible visit log
```
id uuid pk, invite_id fk, started_at, last_seen_at, approx_duration_s int
```

**`aviary_state`** — per-account scene-level canonical state
```
account_id uuid pk, settled bool, settled_at null,
weather jsonb,               -- {kind: 'none'|'rain'|'wind', until}
version bigint,              -- monotonic; bumped every tick write
updated_at timestamptz
```

**`magic_links` / `exports`** — token hashes + TTLs (may live in Redis; Postgres acceptable **[call]**).

### 3.2 Serialization rules

- Personality is **never** serialized to clients as labeled trait numbers. Render-relevant consequences (e.g., plumage visual richness) are transmitted as opaque visual parameters (`plumage_visual: 0..4` band **[call]**), never named as traits, never shown in any UI, and documented as a deliberate non-statistics surface. There is no debug endpoint that exposes vectors.
- Snapshots carry `state_version`; clients discard anything older than what they hold.
- Events are immutable; corrections are new events (e.g., `settle_undo`).

---

## 4. API surface

All endpoints JSON over HTTPS. Session via httpOnly cookie (per-device refresh token). Errors use the matter-of-fact voice and a stable `code` for client-side mapping.

### 4.1 Auth & account

| Endpoint | Purpose |
|---|---|
| `POST /auth/magic-link` `{email}` | Send link (15-min TTL, single-use, per-email rate limit). Always 202 with neutral copy (no account-enumeration). |
| `GET /auth/verify?token=` | Consume link, issue per-device session, redirect to app. Expired/used → matter-of-fact error page with "request a new link". |
| `POST /auth/signout` · `GET /auth/sessions` · `DELETE /auth/sessions/:id` | Session management; revoke any device. |
| `POST /account/email/change` → `GET /account/email/verify?token=` | New address must verify before commit; old email works until then. |
| `GET/PATCH /account/settings` | Toggles: call audio, captions, reduced-motion override, visit notifications (default off). |
| `POST /account/export` | Generate JSON snapshot (birds, names, personality vectors, moods, notebook, settings); email download link to verified address. *(Export is the one place personality numbers are legible — it is the user's own data, delivered to the user only; no UI surface renders them.)* |
| `POST /account/delete` · `POST /account/recover` | Soft-delete with immediate marker; any signed-in page offers recovery for 30 days; hard purge job at +30d removes every row tied to the account (birds, vectors, notebook, events, invites, visits, sessions). |

### 4.2 Aviary state (how clients pull)

| Endpoint | Purpose |
|---|---|
| `GET /aviary/snapshot?since_version=N` | Canonical snapshot: `state_version`, `server_time`, lighting/day-phase, weather, settled flag, per-bird `{id, name, species, perch, pose_hint, mood, plumage_visual, call_intent}` (timing/energy hints only — the client synthesizes), plus `greeting` directive when the server has scheduled a return-greeting (§5.6). ETag = state_version; 304 when unchanged. Payload target <8KB. |
| `GET /notebook?before=<cursor>&limit=` | Paginated entries, oldest-scrollable indefinitely. |
| `GET /aviary/adoption-offer` | Age-gated offer if one is due (§11.3); species pre-selected by server. |
| `POST /aviary/adopt` `{name, offer_id}` | Accept adoption (starters use the same path at onboarding with two system-picked species). |

### 4.3 Interaction events (how clients submit)

| Endpoint | Purpose |
|---|---|
| `POST /aviary/events` `{session_id, events:[{type, bird_id?, payload?, client_ts, idempotency_key}]}` | Batch ingestion into the append-only log. Types: `presence_ping` (60s cadence while all 3 presence signals hold), `presence_end`, `listen_in_start/end`, `offer` (`seed|song|pool`, song id), `settle`, `settle_undo`, `rename`. 202 on accept; idempotent by key. Send via `fetch` keepalive; final `presence_end` via `sendBeacon` on pagehide. |

Offer responses are *not* request/response: the client posts the offer event, the next snapshot(s) carry the bird's reaction, and the client renders it. This preserves the single-writer discipline and makes offers feel observed rather than transactional. Per-bird offer cooldown (a few minutes; start at 4 min **[call]**) is enforced server-side; a cooled-down offer is accepted as an event but produces ambient acknowledgment rather than a fresh reaction — never an error toast.

### 4.4 Visit invitations

| Endpoint | Purpose |
|---|---|
| `POST /invites` `{email}` | Create invite, email one-time link. Host must be verified. |
| `GET /invites` | List outstanding + recent (host's visit-management surface in settings). |
| `DELETE /invites/:id` | Revoke. Effective immediately: the visitor's next snapshot pull returns `410 {code:"visit_unavailable"}` → matter-of-fact surface. Unused revoked links die silently with the same surface. |
| `GET /visits` | Host's visit log: visitor email, date, approximate duration; on-demand only, no badges. |
| `GET /visit/:token/snapshot` | Visitor's read-only snapshot (same shape minus anything host-private; notebook excluded **[call]** — the notebook is the host's). Validates invite status on every pull. |
| `POST /visit/:token/heartbeat` | Updates `visits.last_seen_at` for the host's log. Carries no behavioral payload; feeds the log only. |

Visitor clients run the same web app in **visit mode**: interaction handlers unbound, event endpoint never called, presence never recorded, no simulation input of any kind. The host's drift comes from the host alone.

---

## 5. Simulation engine design

### 5.1 The tick

- Cadence: once per minute per account (calibrate 45–75s during build), executed by the shard worker whether or not any client is connected. This is what makes "the aviary continues without the viewer" an architectural fact rather than a story.
- Per tick, per account (single transaction):
  1. Load birds, cursor, aviary_state.
  2. Consume events `id > cursor.last_event_id` in order.
  3. Update presence accounting (§5.2) and internal estimators.
  4. Apply drift deltas (§5.3).
  5. Advance mood state machines (§5.4), perch selection (§5.5), bird-to-bird effects, weather lifecycle, day/night phase from the account's timezone.
  6. Maybe emit a notebook entry (§5.7) and a return-greeting directive (§5.6).
  7. Write new state, bump `state_version`, advance cursor, commit.
- Tick is **idempotent**: cursor + monotonic version make re-running a tick after a crash safe (at-least-once worker loop, exactly-once effect).
- Budget: a tick is a few ms of math; p99 alarm at 5s catches degradation long before users feel the aviary "run slow."

### 5.2 Presence accounting

Client-side detection, server-side adjudication.

- **Client**: emits `presence_ping` every 60s only while ALL THREE hold simultaneously: `document.visibilityState === 'visible'`, window focus (`focus`/`blur` listeners), and last `pointermove`/`keydown` within the activity window (initial 5 minutes, calibrate toward longer during build — watching birds without moving is the product). Emits `presence_end` on settle, pagehide, or signal loss.
- **Server**: coalesces pings into presence-minutes per account per day (`presence_minutes` bucket), sanity-capped at 16 h/day (belt-and-braces against a client bug inflating the population's drift signal — the exact silent-corruption failure the PRD names). Presence-time is the dominant drift input; settle and tab-close are engine-equivalent window closures; neither is penalized, neither produces a recovery surface.

### 5.3 Personality drift

Five traits in `[0,1]`: boldness, warmth, vocal, plumage, curiosity. Seed values per species with small per-bird jitter (seeded RNG from bird UUID — reproducible, debuggable).

**Monotonic-toward-expressive, with honest expression decay.** The PRD requires both "traits never move down" and "a neglected bird becomes ambient, greeting less often." Reconciliation: two layers per bird —
- **Traits**: non-decreasing. Positive inputs move them up; absence moves nothing.
- **Expression estimator** `E` (per bird, per behavior class): a fast-decaying running estimate of recent attention (half-life ~4 days **[call]**). Greeting propensity, chorus-join readiness, and front-perch boldness *expression* are computed as `f(trait, E)`. Neglect decays `E` toward 0 → the bird is quieter, less forward — while the trait itself never regresses. Return of attention restores `E` quickly; the trait's accumulated growth remains. This is how "leave for two weeks, come back to quieter birds that haven't learned to mistrust you" is implemented.

**Update rule (per tick)** — low-pass filter over daily buckets:
```
trait += R_trait × I_trait(today) × (1 - trait) × (1/1440)     // per-minute portion
```
where `I_trait(today) ∈ [0,1]` is that day's input intensity for the trait:
- all traits: `min(1, presence_minutes / 120)` — presence dominates;
- warmth, vocal: + listen-in minutes on that bird (weighted ~3× presence);
- curiosity: + accepted offers; boldness: + offers made near the bird;
- settle: no directional input — it closes the presence window cleanly (mood-quieting only).
`R_trait` is the per-trait rate constant, the single calibration knob set.

**Calibration targets (testable, named in CI)**:
- *Measurable in instruments after ~1 week of regular visits* (regular = ≥5 presence-min/day, ≥5 days/wk): `Δtrait ≥ 0.01` on the dominant traits.
- *Visible to the user after ~3 weeks*: cumulative `Δtrait ≥ 0.05` with behavior-probability shifts crossing perceptual thresholds (e.g., front-perch probability for the bolder starter moving ≥10 percentage points).
Starting `R` values are set so the dominant-input path hits `≈0.02/week`; the harness in §11.4 validates, and `R` is hot-tunable via config without deploys.

### 5.4 Mood

Enum **[call, finalized here]**: `wary, content, curious, drowsy, alert`, plus `asleep` as the deep-night variant of drowsy (the PRD names "settled or sleeping" birds at night; "settled" remains the *lighting* state). One mood per bird, persisted with `mood_entered_at`, never reset on session boundaries.

Per tick, per bird, score each candidate mood:
```
score(m) = w_time(m, local_hour)          -- drowsy near dusk, alert early morning
         + w_events(m, recent events)     -- accepted offer → content nudge; alarm call → wary
         + w_personality(m, traits)       -- high-boldness resists wary on equal input
         + w_weather(m, weather)          -- rain dampens vocal; wind → alert or wary per bird
         + w_social(m, neighbors' moods)  -- wariness spreads; contentment clusters
         + inertia(current)               -- hysteresis
```
Softmax selection (temperature tuned so transitions feel gradual, not random), **minimum dwell 10–20 min** per mood to prevent flapping, and mood-tagged behavior tables downstream (idle motion, perch choice, call energy). The user's local timezone (from the account row, client-refined) drives the time term, so the aviary shares the user's day.

### 5.5 Perch selection & bird-to-bird

- Perch choice re-evaluated on mood change and on a slow random cadence: probability mass over `{front, middle, back}` = f(boldness × E, mood). Wary → back; content/curious + bold → front. Never user-placeable — perch is a signal the user reads.
- Bird-to-bird: on a call event, nearby birds sample a response (probability ∝ warmth, vocal); a wary shift in one bird adds `w_social` wary weight to others next tick; a chorus emerges when ≥2 birds' call windows overlap (vocal trait scales the join readiness). These produce a small social system, not a row of NPCs.

### 5.6 Return-greeting

The anchor moment; engineered, never canned.

- Server detects session start via the first snapshot pull after an absence gap (>~2 min **[call]**). It selects **one** greeter: score = boldness × E + warmth jitter; warier birds greet later or not at all on a given day.
- Greeting form is chosen from the greeter's mood, boldness, and **absence length**: minutes → glance up from current activity; hours → two-note call + head-tilt; days → longer call, possible approach toward front, possible response call from a second bird.
- Snapshot carries a `greeting` directive: `{bird_id, form, delay_ms, stagger_ms}`. The client renders it with live procedural variation (pose + synthesized call composed at runtime), so no two greetings are identical. If two birds would greet, stagger by a small randomized offset — the aviary notices one bird at a time, never a unison chorus-on-cue.
- No textual welcome anywhere. The bird greeting is the entire welcome surface.

### 5.7 Notebook writer

A tick-time pass that detects *noteworthy* moments and writes at most sparingly:
- Detectors: first-greeter flips vs. trailing week ("pip greeted before wren today, first time this week"); unusual quiet stretches; weather × behavior coincidences; adoption arrivals; mood-color moments ("wren is fluffed against the cool air").
- Sparsity governor: base budget ~1 entry / 3 days; noteworthy events can exceed it, but a per-day cap and `unique(account_id, day, kind)` dedupe prevent feed-noise even for very active users.
- Prose from the shared isomorphic generator (§9.4): slot-filled naturalist templates with per-species and per-mood phrase banks, lowercase present tense, always about the *aviary* — never about the user ("you visited every day" is a lint-blocked class of string).

### 5.8 Call grammar runtime (server half)

The server owns *scheduling intent*: per-bird call windows driven by vocal trait, mood, weather, time of day, and social prompts. It emits `call_intent` hints in snapshots (`energy`, `window_ms`). The client owns *realization*: composing motifs into phrases and synthesizing them with per-call jitter (§8.2). Recognizability lives in a stable per-bird **voice print** (pitch center, formant/timbre params) drawn at adoption from the bird-UUID-seeded RNG; mood and personality modulate around that center within bounded ranges, so Pip sounds like Pip in any mood at any drift level.

---

## 6. Sync model

**One canonical record, many readers, one writer.**

- The server is the only writer of personality, mood, and scene state. Clients hold no authoritative state; there is nothing to merge, nothing to conflict, no last-write-wins anywhere in the system.
- Propagation: clients pull snapshots (a) on load, (b) on `visibilitychange → visible`, (c) after any render-frame gap >5s (laptop suspend), (d) on a 30s keepalive while visible. Snapshots are kilobytes; ETag/`state_version` short-circuits unchanged polls. Client interpolates between consecutive snapshots for smooth motion (perch A→B renders as a flight/transition, never a teleport).
- Conflict prevention is structural:
  - **Personality**: additive server-authored deltas consumed from the ordered event log by the tick; clients physically cannot submit absolute values — the API has no such field. The laptop-morning/phone-lunch overwrite scenario is unreachable by construction.
  - **Events**: idempotency keys + monotonic ids; out-of-order or duplicated client batches are harmless.
  - **Clock skew**: server timestamps are authoritative for ordering; `client_ts` is advisory (used for presence-window length, clamped to sane bounds).
  - **Tick correctness**: per-account cursor in the same transaction as the state write → idempotent recovery from worker crashes; shard leases prevent double-ticking.
- Multi-device: both devices read the same record; phone at night shows the laptop-morning's drift because there is only one drift.
- Rare account-level failures (magic-link replay, session timeout mid-write, outage) surface in matter-of-fact voice with a next step: "We couldn't sign you in. The link may have expired. Try requesting a new link." / "Your session timed out. Sign in again to keep watching." / "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

---

## 7. Frontend rendering pipeline

### 7.1 Stack **[call]**

Vanilla TypeScript + hand-rolled Canvas 2D renderer — no UI framework, no WebGL dependency. Rationale: the scene is ≤7 articulated sprites + 3 parallax planes + weather; Canvas 2D comfortably holds 60fps on the target hardware; a framework would burn a third of the 2MB budget for no product value. DOM is thin (top bar, overlays, captions, settings). If profiling ever misses 60fps, the fallback is a WebGL sprite path behind the same scene graph — budgeted as a contingency, not built speculatively.

### 7.2 Scene composition

Layer stack (back→front): sky gradient (time-of-day LUT, slow continuous interpolation) → background foliage plane (parallax 0.2) → weather-behind plane → mid plane (perches + birds, the only semantic plane) → weather-front plane (rain streaks) → foreground ornaments (occasional leaf/feather, parallax 0.05) → DOM: captions, top bar, overlays.

- **Responsive**: one horizontal scene, letterboxed scale-to-fit with all birds always in frame; narrow viewports compress spacing, wide viewports widen it. Never crop a bird, never scroll, never zoom.
- **Perch zones**: three depth tiers with distinct y/scale; perch choice is server-driven (§5.5).
- **Day/night**: continuous palette interpolation over the user's local day; evening warms and quiets; night dims with most birds settled and one nightjar-like species occasionally active — night is not a dead state. The settle gesture drives the same evening ramp on demand (slow few seconds), with any click within 5s reversing it.
- **Weather**: rare, quiet, short rain (few times/week) and soft wind; streaks + leaf ripple + palette damp; mood effects per §5.4. Never thunderstorms, never snow, nothing the user must notice.

### 7.3 Bird rendering & idle micro-motion

- Birds are sprite atlases pre-rendered at startup from a procedural bird generator (per-species silhouette, palette banded by `plumage_visual`), pose set: idle, preen, scan, head-tilt, fluff, hop, fly-keyframes, sleep.
- **Idle motion** is a per-bird behavior sequencer: mood-weighted action tables (wary → back-perch scanning; content → preening; curious → tilting toward sounds/leaves; drowsy → low fluffed sit) with per-action cooldowns so each bird does something small every ~8–30s. Birds are never paused-still; motion is slow and personality-keyed. Motion continues *logically* (server) when the tab hides; rendering stops (battery) and resumes on visible by snapping to the interpolated "now."
- **First frame**: the HTML payload inlines the initial snapshot; first paint places birds mid-pose (derived deterministically from snapshot + local jitter) with calls already scheduled — no entry animation, no fade-from-static, no spinner-resolves-into-aviary. Slow-connection loading state is the **quiet field** (soft sky, faint drifting leaf) — never a spinner. The empty-aviary moment post-adoption is the same quiet field; each starter then enters with one soft fly-in, once, and the user never sees an empty aviary again.

### 7.4 Top bar chrome

Thin bar above the scene: account/settings, accessibility settings, field notebook, offer affordance. Nothing else. Fades to near-transparent after ~3s of cursor stillness; returns on pointer/keyboard activity. No badges, ever — including "new notebook entry" or "friend visited" (both deliberately unreachable as counts in the UI).

### 7.5 Reduced-motion mode

A **render strategy switch**, not an asset swap and not "animations off":
- Micro-motion → slow cross-fades (0.5–1.0s opacity lerps) between still poses from the same state-driven pose sequencer.
- Perch transitions → cross-fade between perch poses, no flight path.
- Leaf/feather drift removed; day/evening color shifts retained, slowed.
- Calls, captions, drift, mood, notebook all unchanged — the aviary is still the aviary in a calmer register.
Triggered by `prefers-reduced-motion` or the accessibility-settings override; ships in v1, gated in CI (§11.5), never a "v1.1 fix."

---

## 8. Audio pipeline

### 8.1 Why procedural, unconditionally

Two recorded loops layered are not a chorus — the ear catches the phase artifacts and the repetition, and the spell breaks permanently. Recorded audio at the required variation also cannot fit the bundle budget. So: all calls are synthesized client-side via WebAudio from the shared motif descriptors; **there is no recorded-audio fallback path at any quality**. If WebAudio is unavailable or blocked: graceful silence + captions on by default. Silence with captions beats canned audio.

### 8.2 Synthesis design

- **Motif library** (shared `domain` package): per-species motifs defined as synthesis recipes — FM/AM oscillator pairs, shaped-noise components, ADSR envelopes, vibrato — parameterized by pitch center, duration, energy. ~8–15 motifs per species **[call]**.
- **Phrase composer**: per call, select 1–4 motifs by mood and context (greeting, ambient, response, chorus-join), apply per-call jitter (pitch ± bounded, timing, gain) so no call is ever identical twice, within the bird's voice-print bounds (§5.8) so identity survives variation.
- **Graph**: per-bird synth chain → per-bird gain node → chorus bus → gentle master bus (soft limiter). Ambient bed (wind/leaf noise, procedural, low gain) under everything; weather adds/removes layers (rain patter) at the bus.
- **Autoplay reality**: browsers gate audio before a user gesture. The app attempts `AudioContext` start on load; if suspended, it resumes on the first pointer/key event (which is also our presence signal) — no "enable sound" toast, no announcement of any kind. Before unlock, the scene renders silently (with captions if enabled). Call-audio on/off lives in accessibility settings (default on) — the "mute the calls or let them play" signal from the brief is captured as an event input to drift context.

### 8.3 Chorus & listen-in mixing

- **Chorus**: overlapping birds are genuinely simultaneous synth voices — real-time mix, real variation, no loop stacking. Vocal-frequency trait scales both solo call rate and chorus-join readiness.
- **Listen-in**: engaging ramps the focused bird's gain to full over ~1.5–2s (`setTargetAtTime`, smooth exponential) while others ease down to a low ambient level (~25% **[call]**) — never to zero. Disengage (re-click the bird, focus another, click empty space, move keyboard focus away) reverses the ramp at the same rate. The mix is a re-balance, never a mute; it must feel like listening, not switching channels.

### 8.4 Captions from the same descriptor

Each emitted phrase produces a caption string from its own motif sequence ("a soft three-note rise"; "a low trill, paused, low trill again"; "a single sharp call from the back perch") via the shared caption generator — so captions always match what actually played, in the naturalist voice, rendered as small text near the calling bird, fading with the call. Opt-in via accessibility settings; default-on when audio is unavailable or off.

---

## 9. Accessibility surfaces

Designed surfaces, shipped with v1, gated in CI. The cheap version (ARIA state-list, animations-off fallback, fixed-string captions) is explicitly rejected.

### 9.1 Screen-reader narration

- An offscreen `aria-live="polite"` region carries running naturalist prose generated client-side by the shared narration generator from the same snapshots the visuals read: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- **Cadence**: one update per 30–60s at idle (jittered to avoid metronome feel); a small priority queue bumps user-initiated events (return-greeting on session start, an offer reaction, a settle) to prompt narration — written as observations, never state transitions ("pip took the seed after a long look", not "offer accepted").
- Throttle/dedupe logic guarantees the SR queue is never flooded; high-frequency narration that forces users to silence it is treated as a defect class.
- Voice continuity is a requirement: narration, notebook, and captions share one generator and one phrase bank.

### 9.2 Reduced motion — §7.5 (its own designed aesthetic, not a stripped fallback).

### 9.3 Captions — §8.4.

### 9.4 The shared prose generator

One isomorphic module produces notebook entries (server, at tick), narration (client, from snapshots), and captions (client, from phrase descriptors): slot-filled templates over per-species/per-mood/per-time-of-day phrase banks, lowercase, present-tense, specific, bird-named. Human-curated phrase banks with editorial review; template grammar prevents announcement framing ("X happened at 7:43" and "your bird is happier!" are unrepresentable). This single-source design is what makes voice consistency across three surfaces actually hold.

### 9.5 Keyboard navigation & focus

- Tab → top bar items; Tab into the scene focuses the first bird; arrow keys move between birds; Enter toggles listen-in; Escape exits. Offer affordance opens via top-bar shortcut and is fully keyboard-navigable (choose gift → it lands in the scene); settle is reachable from the top bar.
- Focus indicators: soft high-contrast outline specified by the designer to read against both bright and dim aviary states; never removed, never `outline: none` without replacement.
- All interactive surfaces (settings, invites, notebook, export/delete flows) are standard semantic HTML — the plain parts stay plain.

### 9.6 Contrast & support

All user copy (top bar, settings, errors, captions, visual narration) passes WCAG AA minimum; the design system pins per-surface ratios. Checked in CI (axe) on every chrome surface. Browser support: last two majors of Chrome, Safari, Firefox, Edge; older browsers get a matter-of-fact unsupported-browser page — no compatibility bloat.

---

## 10. Performance budgets & observability

### 10.1 Budgets (enforced, not aspirational)

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS bundle | <2MB gzipped at first paint | CI bundle-size gate; target core scene ≤250KB, settings/account/invites code-split and lazy |
| Time to first bird | <500ms on mid-tier mobile / 4G | Synthetic fleet p50/p95 per geo; release gate |
| Idle motion | 60fps sustained on 5-yr-old mid-range laptop, 30-min session | Frame-time histograms in RUM + synthetic long-session runs |
| Memory | No growth over 30 min | CI: headless 30-min accelerated session, heap snapshots, buffer-pool audits; a real test, not a guideline |

### 10.2 How <500ms first bird is achieved

Edge-served HTML with the initial snapshot inlined (edge fetches the snapshot at request time from the origin snapshot API, ~KBs), critical CSS inline, one deferred JS bundle, sprite atlas generated during the same first frames, first paint on the first rAF without waiting for non-critical assets. The metric is *first bird visible*, not `load` — and it's treated as an affective metric: above 500ms the user notices a load; below it, the aviary was simply already there.

### 10.3 Runtime discipline

rAF loop with zero per-frame allocation (object pools for poses/particles); sprite atlases pre-rendered; audio buffers pooled and reused (no per-call allocation that isn't freed); notebook list virtualized (scrolled-out entries release references); workers/audio contexts bounded; rendering halted entirely when the tab is hidden (presence detection unaffected). Day/night LUT interpolation instead of per-frame gradient rebuilds.

### 10.4 Observability — what we measure and what we deliberately don't

**Measured (aggregate-only, no account dimension anywhere):**
- Synthetic fleet (Playwright browsers on a schedule from common geographies): first-bird time, snapshot latency, audio-context failures.
- RUM: page-load timings, first-bird timings, frame-time histograms, audio errors, WebAudio-unavailable rate, API latencies, error rates — session-duration histograms anonymized, no per-account dimension.
- Sim health: tick latency (p99 alarm at 5s), tick lag per shard, event-ingestion lag, snapshot cache hit rate, email deliverability.
- Privacy policy in account settings names these categories in plain text.

**Deliberately not measured:** per-bird state in telemetry; per-account interaction history; visit-frequency per user; retention funnels with an account dimension; anything that could reconstruct a user's relationship with their aviary. The boundary is architectural: the telemetry pipeline has no read path to the simulation database, metric definitions are allowlisted at schema review, and per-account fields are absent from the metrics schema — respected at the data-pipeline level, not the policy level. Product judgment calls (drift feel, voice quality) come from the in-house calibration harness and qualitative sessions, never from per-user analytics.

---

## 11. Rollout

### 11.1 Milestones

| Milestone | Contents | Exit criteria |
|---|---|---|
| M0 — Foundations | Monorepo, CI gates (bundle size, axe, copy-lint), Postgres schema, auth (magic link, sessions), synthetic-UUID discipline | Sign-in round-trip; PII audit shows email in exactly one place |
| M1 — Engine core | Tick workers, event log, presence accounting, drift v1, mood machine, perch selection | Calibration harness (§11.4) green on 1-week/3-week targets; idempotency fault-injection passes |
| M2 — Scene | Canvas renderer, parallax, day/night, weather, first-frame-already-alive, quiet-field loading, top bar fade | First-bird <500ms synthetic; 60fps/30-min; no-chrome-in-scene review |
| M3 — Audio | Synth engine, motif libraries ×6 species, chorus, listen-in ramps, captions-from-descriptor, silence fallback | Listening review (§12); caption/audio match tests; uncanniness checklist |
| M4 — Interactions | Return-greeting, offers + cooldowns, settle + undo, listen-in input map, notebook writer | No-textual-welcome audit; stagger verification; notebook sparsity governor tests |
| M5 — Accounts & social | Export, deletion (soft→hard), email change, invites + visit mode + revocation + visit log | Revocation takes effect at next pull; visitor emits zero simulation input |
| M6 — A11y & hardening | Narration, reduced-motion, keyboard map, contrast, memory CI, fault drills | A11y release gate green; 30-day soft-delete purge drill; PITR restore drill |

### 11.2 Launch sequence

Private alpha (team + in-house synthetic households) → closed beta (~hundreds, waitlist) → waitlist ramp → GA. Feature flags: drift rate constants `R`, activity-window length, tick cadence, offer cooldown, notebook sparsity — all config-tunable without deploys. Beta cohort uses explicit opt-in feedback sessions (qualitative), consistent with the telemetry boundary.

### 11.3 Birds-per-aviary ramp

Two starters at adoption. Further offers keyed strictly to **aviary age** — never visit count, interaction score, or payment: 3rd bird ≈ 3 months, 4th ≈ 6 months, 5th ≈ 9 months, then ≈ 1/quarter to the hard cap of 7 **[call — concrete intervals matching "a few months → third; a year → five or six"]**. Offer appears quietly in the flow (top-bar affordance, naturalist voice, dismissible without penalty); species pre-selected by the server; user names the bird. The mechanic never teaches that attention earns stuff.

### 11.4 Instrumented from day one (calibration harness)

A suite of **synthetic in-house accounts** driven by scripted presence patterns (daily 10-min watcher, weekend-only watcher, two-weeks-away-and-back, offer-every-visit, never-offer) run continuously against staging and production-shadow. Asserts: measurable drift ≥0.01 after simulated week 1; visible-threshold drift by week 3; neglect produces expression decay without trait regression; presence cap rejects >16 h/day; mood never snaps on session start. This harness is how drift calibration is validated *without* per-account production telemetry — the privacy boundary forces the method, and the method is better anyway.

### 11.5 CI quality gates (every merge)

Bundle size; axe on all chrome; copy-lint (voice rules); keyboard-path e2e; reduced-motion visual-diff snapshots; narration cadence/cadence-cap tests; caption↔descriptor match tests; memory-growth session test; tick idempotency + fault injection; snapshot-contract tests (no personality numbers on the wire).

---

## 12. Risks

| Risk | Failure shape | Mitigation |
|---|---|---|
| **Drift calibration wrong** | Too fast → Tamagotchi ("move a number by clicking"); too slow → screensaver ("nothing I do matters"). Silent — no test fails; users just feel it wrong. | Named calibration targets (§5.3); synthetic-household harness asserting both bounds continuously; `R` constants config-tunable; beta qualitative reviews focused on "did you notice Pip change?" at week 3; the 1-week-instruments / 3-weeks-user gap explicitly verified, not assumed. |
| **Presence overcounting** | "Tab open" counted as attention silently inflates drift across the whole population — the exact corruption the PRD warns about; invisible in tests, visible only as birds changing faster than designed. | Strict 3-signal conjunction implemented client-side with unit + e2e tests per signal-pair failure mode; server-side 16h/day cap; drift-harness includes a "laptop left open overnight" pattern asserting ~zero drift. |
| **Sync correctness / personality loss** | A reset or lost vector deletes the bird the user knows — worst possible failure, and nearly invisible until felt. | Single-writer discipline (no client-writable personality path exists); additive deltas from an ordered log; cursor-in-transaction idempotency; PITR + nightly logical dumps of sim tables; append-only drift-delta journal as a redundant reconstruction source; restore drill in M6. |
| **Audio uncanniness** | Repetition ("I've heard that exact call") or chorus phase artifacts break the spell permanently; listen-in that feels like channel-switching; captions that don't match audio. | True per-call synthesis (no loops, ever); jitter bounds wide enough for non-repetition, narrow enough for voice-print identity; bounded chorus size (7-cap exists for this); listen-in ramps ≥1.5s with others-never-silent; captions generated from the same phrase descriptor as the audio; structured human listening reviews at M3 and pre-GA with a written uncanniness checklist. |
| **Accessibility regressions** | A11y treated as checklist → SR users get state-lists, reduced-motion users get a broken-looking scene; or a11y slips to "v1.1," which tells those users the product wasn't for them. | A11y surfaces are release gates in CI (§11.5), designed surfaces (§9) with their own aesthetics; narration cadence caps tested; reduced-motion has visual-diff snapshots; a11y ships *in* v1, full stop. |
| **"Notice, never announce" erosion** | A well-meaning contributor adds a welcome toast / streak / badge; one leak reframes the whole product as trying. | Copy-lint blocks announcement vocabulary; top bar has no badge capability in the component API (not just policy); PR review checklist names the three most-likely violations (welcome toast, streak counter, visit notification); non-goals quoted in the repo's engineering guide. |
| **Mood flapping / mood snap** | Rapid mood oscillation reads as glitch; reset-on-open breaks cross-session continuity. | Hysteresis + minimum dwell (§5.4); mood persisted with enter-time; harness asserts no visible snap across session boundaries. |
| **Magic-link deliverability** | Email delays/bounces = sign-in failure for the only auth path. | Reputable transactional provider; deliverability monitoring + alerts; neutral 202 copy; easy "request a new link"; rate limits tuned to allow retries. |
| **Autoplay audio gating** | First-load silence could read as broken. | Resume-on-first-gesture (which presence detection makes near-immediate); captions available; never an "enable sound" announcement. |
| **Notebook prose quality/sparsity** | Template-y entries or feed-frequency entries dilute the product's most concentrated voice surface. | Human-curated phrase banks with editorial review; sparsity governor with hard per-day caps; noteworthy-only triggers; harness asserts max entry rates for hyperactive synthetic users. |
| **Scale of universal ticking** | Ticking every account every minute grows linearly. | 64-shard worker model scales horizontally (re-shard to 256 by config); tick is ms-scale math; per-shard lag alarms; at v1 volumes this is trivially within a small instance count. |

---

## 13. Key decisions register (defensible calls made where the PRD is silent)

1. **Stack**: TypeScript everywhere; Node+Fastify API; Postgres canonical; Redis ephemeral-only; no websockets (polling fits the 1-min canonical cadence; SSE is the documented upgrade path).
2. **Renderer**: hand-rolled Canvas 2D, no framework; WebGL sprite path as contingency only.
3. **Drift mechanics**: non-decreasing traits + fast-decaying expression estimator `E` reconciles monotonicity with "neglected birds go quiet."
4. **Mood set**: `{wary, content, curious, drowsy, alert, asleep}` with 10–20 min minimum dwell.
5. **Presence window**: starts at 5 minutes, calibrate longer; presence pings at 60s; 16h/day server cap.
6. **Adoption ramp**: 3rd ≈ 3mo, 4th ≈ 6mo, 5th ≈ 9mo, then quarterly to 7.
7. **Offer cooldown**: starts at 4 min/bird; cooled-down offers get ambient acknowledgment, never errors.
8. **Plumage on the wire**: opaque visual band, never a labeled trait number; export-to-the-user is the only legible vector surface.
9. **Notebook excluded from visitor view**; visit mode emits heartbeats for the host's log only.
10. **Shard model**: 64 logical shards, Postgres advisory-lock leases, re-shardable by config.
