# Pocket Aviary — v1 Implementation Plan

**Status:** phase-1 plan, ready for engineering execution
**Source:** `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals)
**Audience:** a frontier engineering team executing without further clarification from the PRD authors

---

## 0. How to read this plan

The PRD is unusually opinionated about *feel*, and most of its hard constraints are affective ones with engineering consequences. This plan's job is to convert those into structures that make the wrong thing hard to build. Where the PRD names a principle, this plan names the mechanism that enforces it:

| PRD principle | Enforcing mechanism in this plan |
|---|---|
| Feels alive, not robotic | Server tick is canonical and deterministic; client never has an "initial" state, only a "current" state (§5, §7); no load-state code path exists in the render pipeline (§8.2) |
| Notice, never announce | No toast/banner/notification component exists in the design system; CI lint bans the primitives (§13.3) |
| Personality never exposed numerically | Trait values are stripped at the service boundary — the snapshot DTO physically cannot carry them (§4.4, §6.3); contract test asserts it |
| No gamification | No counter/aggregate of user visits is persisted in a queryable shape; notebook generator has a hard "subject must be a bird" rule (§9.4, §13.3) |
| Monotonic drift | Drift deltas are clamped non-negative at the single write site; property test asserts monotonicity over random event streams (§6.2) |
| Procedural audio only | No audio file format in the asset pipeline allowlist; bundle gate fails on `.mp3/.ogg/.wav` (§10.6) |
| Accessibility is the product | Narration and captions are generated from the same state machine as the visuals, in the same voice module (§11); ships in M4, not after launch |

Sections marked **[Decision]** are calls this plan makes where the PRD left an implementation detail open. Sections marked **[Assumption]** are places where the PRD is genuinely ambiguous and a different reading would be defensible; each states what would change if the assumption is wrong.

---

## 1. Scope

### 1.1 In scope for v1

**Accounts and identity**
- Email + magic-link sign-in; 15-minute link expiry; single-use consumption; per-email rate limiting
- Per-device session tokens, listed and revocable from account settings
- Email change with new-address verification before commit (old address valid until verified)
- Synthetic account UUID as the only identifier used anywhere but the account record
- JSON account export, generated on demand, delivered as an emailed download link
- Soft deletion for 30 days with in-product recovery, then hard deletion

**Aviary and bird engine**
- One canonical aviary per account; 2 starter birds at adoption; hard cap of 7
- Species pool of 6, each with silhouette, plumage palette, and motif library
- Hidden 5-trait personality vector per bird, persisted server-side, server-write-only
- Monotonic-toward-expressive drift driven primarily by presence-time
- 5-state mood machine per bird with daily-ish reset cadence, persisting across sessions
- Server-side simulation tick at ~60s, running independently of client connections
- Bird-to-bird interaction: call-and-response, wary contagion, emergent chorus
- New-species offers gated on aviary age only
- Stable internal bird IDs, decoupled from name and species, invariant across all migrations

**Interactions**
- Return-greeting: single bird, procedurally varied, keyed to boldness/mood/absence length, staggered when multiple birds greet
- Listen-in: click, tap, or keyboard focus; slow mix ramp both directions; other birds attenuate but never mute
- Offer: seed, song fragment, still pool; reachable only from the top bar; per-bird cooldown of a few minutes
- Settle: top-bar gesture, slow evening lighting shift, 5-second any-click undo
- Field notebook: auto-generated, read-only, sparse, infinitely scrollable backwards
- Presence accounting under the strict three-condition conjunction

**Scene**
- Single non-panning, non-scrolling, non-zooming horizontal scene, responsive without ever cropping a bird
- Three perch zones (front/middle/back), system-assigned from mood and personality
- Local-timezone day/night cycle with a nightjar-like species active at night
- Rare, non-assertive ambient weather (short rain, soft wind) with short-lived mood effects
- Ambient micro-motion: leaf and feather drift, subtle parallax, continuous idle motion
- No UI chrome inside the scene; thin top bar with exactly four affordances; top bar fades on cursor stillness

**Social**
- Per-invite, email-addressed, revocable visit invitations; off by default; 30-day unused expiry
- Read-only ambient visitor view with no interaction, no co-presence, no visitor representation
- Visitor activity contributes nothing to host drift
- Host-side visit log in account settings, pull-only, no badge
- Optional per-account visit-notification toggle, off by default, not surfaced in onboarding

**Accessibility**
- Running naturalist screen-reader narration on a 30–60s idle cadence with priority bumps for user-initiated events
- Reduced-motion mode as a designed cross-fade rendering, not a stripped fallback
- Runtime-generated call captions matching what was actually played
- WCAG AA contrast on all user copy; full keyboard navigation with visible focus against all aviary states

**Performance**
- <2 MB gzipped initial JS; <500 ms time-to-first-bird on mid-tier mobile over 4G; 60 fps idle motion on a 5-year-old mid-range laptop sustained over 30 minutes; zero memory growth over 30 minutes
- Client-side procedural WebAudio synthesis; graceful-silence-plus-captions fallback
- Last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface below that

### 1.2 Explicitly out of scope

Native iOS/Android apps (and no data-model or protocol accommodation for them). Any gamification: achievements, streaks, levels, scores, badges, XP, ranks, tiers, adoption counters, visit calendars, "days visited," exportable visit logs of the *user's own* behavior. Tamagotchi mechanics: death, hunger, distress, decaying happiness, negative drift on neglect. Social-network surfaces: profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments, chat, avatars, leaderboards, show-off rendering. Push notifications and marketing email about the aviary. Payments and tiers. Shared or multi-aviary accounts. Customizable scenes. Panning/scrolling/zooming. User-controlled perch placement. Any numeric surfacing of personality traits, in any tier, behind any toggle, including internal debug builds shipped to production.

### 1.3 The "one screen" scope test

Any proposed v1 feature must pass: *does this make the user look at a bird, or at the product?* Features that add a surface the user reads instead of the aviary are out by construction. This test is the tiebreaker for scope arguments during build and belongs in the team's PR template.

---

## 2. Architecture

### 2.1 Shape

Five services, one client, one shared TypeScript types package. Small on purpose — the PRD's product is small, and a microservice-per-noun decomposition would add sync surfaces the "one canonical record" model exists to eliminate.

```
                    ┌─────────────────────────────────────────┐
   browser ────────▶│  edge (CDN)  —  HTML + JS + inlined      │
                    │  bootstrap snapshot                      │
                    └───────────────┬─────────────────────────┘
                                    │
              ┌─────────────────────┴────────────────────┐
              │            aviary-api (Fastify)          │
              │  read: snapshot, notebook, settings      │
              │  write: event ingest (append-only)       │
              └───┬──────────────┬───────────────┬───────┘
                  │              │               │
        ┌─────────▼────┐  ┌──────▼───────┐ ┌─────▼─────────┐
        │  identity    │  │  Postgres    │ │  Redis        │
        │  (magic link,│  │  canonical   │ │  tick queue,  │
        │   sessions,  │  │  state +     │ │  snapshot     │
        │   invites)   │  │  event log   │ │  cache, rate  │
        └──────────────┘  └──────▲───────┘ └───────────────┘
                                 │
                      ┌──────────┴──────────┐
                      │  sim-worker         │
                      │  the tick. sole     │
                      │  writer of          │
                      │  personality state  │
                      └─────────────────────┘

                      ┌─────────────────────┐
                      │  notebook-worker    │  (reads sim output,
                      │  observation gen    │   writes entries)
                      └─────────────────────┘

                      ┌─────────────────────┐
                      │  ops telemetry      │  ◀── hard boundary:
                      │  (separate store)   │      never reads the
                      └─────────────────────┘      simulation DB
```

**[Decision] Runtime and language.** TypeScript end to end (Node 22 LTS server, strict mode, no `any` in the sim engine). One language means the simulation's pure-function core — drift, mood transition, call grammar — is a single shared package compiled to both server and client. The client needs it for interpolation and for caption generation; the server needs it as the authority. Sharing the code, not just the spec, is what keeps client-predicted motion and server-canonical motion from diverging visibly.

**[Decision] Datastore.** PostgreSQL 16 as the single canonical store. The whole product is small-row, high-fanout-read, low-write; there is no scale story here that justifies a second database. Redis is a cache and a queue, never a source of truth — if Redis is lost entirely, the product degrades to slower snapshot reads and nothing else.

### 2.2 Client/server split — the invariant

The split is not a performance decision, it is the correctness decision the PRD names repeatedly. Stated as an invariant the whole team can hold:

> **The server owns what the bird *is*. The client owns what the bird *looks like right now*.**

- Server owns: personality vectors, mood, mood timers, perch assignment, call scheduling decisions, weather, drift, notebook entries, day-phase derivation, presence accounting.
- Client owns: sub-second motion, interpolation between snapshots, idle micro-motion selection, leaf/feather ornaments, audio synthesis, caption rendering, camera-free layout, focus state.

The dividing line is time: anything with a timescale longer than the snapshot interval is server-owned; anything shorter is client-owned. A leaf drifting through frame has no server representation (the PRD says so explicitly). A bird's decision to move to the front perch does, because the phone must agree with the laptop about it.

### 2.3 Render pipeline boundary

The client renders from a **scene model**, not from the API DTO. The API DTO is deserialized into the scene model exactly once per snapshot, and the renderer only ever reads the scene model. This boundary is what makes the visitor view, the reduced-motion view, and the narration view three consumers of one state rather than three parallel implementations.

```
snapshot DTO ──▶ scene model ──┬──▶ canvas renderer (default)
   (server)       (client       ├──▶ canvas renderer (reduced-motion profile)
                   truth)       ├──▶ audio graph driver
                                ├──▶ a11y DOM mirror + narration generator
                                └──▶ caption generator
```

Every consumer is a pure function of `(sceneModel, t)`. No consumer mutates the scene model; only the snapshot applier does. This is the single most important structural rule on the client, because it is what prevents the accessibility surfaces from drifting out of sync with the visual one over six months of feature work.

### 2.4 Why no WebSocket at v1 **[Decision]**

The tick is ~60 s. Snapshots are kilobytes. A persistent socket per open tab would buy sub-second freshness the product has no use for, and would cost a stateful connection tier, reconnection logic, and a second delivery path to keep correct. v1 uses HTTP polling: on visibility change, on long render-frame gap, and on a keepalive aligned to the tick with jitter. Conditional requests (`ETag`/`If-None-Match`) make the steady-state poll a 304 with a near-zero body.

*Revisit trigger:* if we ever add co-presence (explicitly out of scope) or drop the tick below ~10 s, revisit. Neither is planned.

---

## 3. Domain model (conceptual)

Before the schema, the object graph, because several of these have non-obvious lifetimes:

- **Account** — synthetic UUID, encrypted email, settings, lifecycle state. Lives until hard deletion.
- **Aviary** — one per account. Carries `created_at` (the sole input to new-bird pacing), timezone, and the canonical simulation clock `simulated_through`.
- **Bird** — stable UUID, species, user-assigned name, personality vector, mood, mood timers, perch, adoption timestamp. Lives as long as the account. **Never** replaced, regenerated, reseeded, or migrated into a different row identity.
- **Personality vector** — five scalars in `[0,1]`, server-write-only, never serialized to any client.
- **Mood state** — enum + entered-at + dwell-until. Fast timescale.
- **Interaction event** — append-only, client-authored, immutable, idempotent. The only thing a client may write that affects simulation.
- **Presence window** — derived server-side from presence-ping events; the drift input.
- **Notebook entry** — immutable, system-authored prose, sparse, read-only forever.
- **Invitation** — per-visitor, per-invite, revocable, 30-day unused expiry.
- **Visit** — a record of a redeemed invitation being used; contributes nothing to simulation.

---

## 4. Data model

### 4.1 Schema

All timestamps `timestamptz`, stored UTC. All identifiers UUIDv7 (time-sortable, index-friendly). No natural keys anywhere.

```sql
-- ─── identity ────────────────────────────────────────────────────────────────
create table account (
  id                  uuid primary key,              -- synthetic; the ONLY id used elsewhere
  email_ciphertext    bytea not null,                -- envelope-encrypted, KMS data key
  email_kid           text  not null,                -- key id for rotation
  email_lookup_hash   bytea not null unique,         -- HMAC-SHA256(email, pepper); sign-in lookup only
  status              text  not null default 'active'
                        check (status in ('active','pending_deletion','deleted')),
  deletion_requested_at timestamptz,
  created_at          timestamptz not null default now()
);

create table pending_email_change (
  account_id          uuid not null references account(id) on delete cascade,
  new_email_ciphertext bytea not null,
  new_email_lookup_hash bytea not null,
  token_hash          bytea not null,
  expires_at          timestamptz not null,
  primary key (account_id)
);

create table magic_link (
  token_hash          bytea primary key,             -- SHA-256 of the emitted token; raw never stored
  account_id          uuid not null references account(id) on delete cascade,
  issued_at           timestamptz not null default now(),
  expires_at          timestamptz not null,          -- issued_at + 15 min
  consumed_at         timestamptz
);

create table session (
  id                  uuid primary key,
  account_id          uuid not null references account(id) on delete cascade,
  token_hash          bytea not null unique,
  device_label        text,                          -- coarse UA-derived: "Safari on iPhone"
  created_at          timestamptz not null default now(),
  last_seen_at        timestamptz not null default now(),
  revoked_at          timestamptz
);

-- ─── aviary + birds ──────────────────────────────────────────────────────────
create table aviary (
  id                  uuid primary key,
  account_id          uuid not null unique references account(id) on delete cascade,
  created_at          timestamptz not null,          -- sole input to new-bird pacing
  timezone            text not null default 'UTC',   -- IANA; client-reported, server-stored
  simulated_through   timestamptz not null,          -- canonical sim clock
  tick_seq            bigint not null default 0,     -- monotonic tick counter; PRNG stream index
  weather_state       jsonb not null default '{"kind":"clear"}',
  settled_until       timestamptz,                   -- non-null while in settled state
  rng_seed            bigint not null                -- per-aviary base seed
);

create table bird (
  id                  uuid primary key,              -- STABLE FOR LIFE. never reassigned.
  aviary_id           uuid not null references aviary(id) on delete cascade,
  species_id          text not null,                 -- references the code-side species pool
  name                text not null,
  adopted_at          timestamptz not null,
  -- personality vector: server-write-only, never serialized to a client
  boldness            real not null check (boldness between 0 and 1),
  social_warmth       real not null check (social_warmth between 0 and 1),
  vocal_frequency     real not null check (vocal_frequency between 0 and 1),
  plumage_saturation  real not null check (plumage_saturation between 0 and 1),
  curiosity           real not null check (curiosity between 0 and 1),
  -- mood: fast timescale
  mood                text not null check (mood in ('wary','content','curious','drowsy','alert')),
  mood_entered_at     timestamptz not null,
  mood_dwell_until    timestamptz not null,          -- hysteresis floor
  mood_day_epoch      date not null,                 -- for daily-ish reset in aviary-local time
  perch_zone          smallint not null check (perch_zone in (0,1,2)), -- 0 front, 1 mid, 2 back
  perch_slot          smallint not null,             -- lateral slot within the zone
  next_call_at        timestamptz,
  voice_seed          bigint not null,               -- FIXED AT ADOPTION. timbre identity. never drifts.
  last_greeted_at     timestamptz
);
create index on bird (aviary_id);

create table bird_offer_cooldown (
  bird_id             uuid primary key references bird(id) on delete cascade,
  cooldown_until      timestamptz not null
);

-- ─── event log ───────────────────────────────────────────────────────────────
create table interaction_event (
  id                  uuid primary key,              -- client-generated UUIDv7 = idempotency key
  aviary_id           uuid not null references aviary(id) on delete cascade,
  bird_id             uuid references bird(id) on delete cascade,   -- null for aviary-scoped events
  kind                text not null
                        check (kind in ('presence_ping','listen_in_start','listen_in_end',
                                        'offer','settle','settle_undo','session_open')),
  payload             jsonb not null default '{}',
  client_ts           timestamptz not null,          -- advisory only; never trusted for ordering
  server_ts           timestamptz not null default now(),  -- authoritative ordering
  seq                 bigserial not null
);
create index on interaction_event (aviary_id, seq);
create unique index on interaction_event (id);

create table sim_cursor (
  aviary_id           uuid primary key references aviary(id) on delete cascade,
  consumed_through_seq bigint not null default 0
);

create table presence_window (
  id                  uuid primary key,
  aviary_id           uuid not null references aviary(id) on delete cascade,
  started_at          timestamptz not null,
  ended_at            timestamptz,
  credited_seconds    integer not null default 0,    -- capped, de-inflated
  consumed_by_tick    bigint                          -- tick_seq that folded this into drift
);
create index on presence_window (aviary_id, started_at desc);

-- ─── notebook ────────────────────────────────────────────────────────────────
create table notebook_entry (
  id                  uuid primary key,
  aviary_id           uuid not null references aviary(id) on delete cascade,
  observed_at         timestamptz not null,          -- aviary-local moment being described
  body               text not null,                  -- final naturalist prose
  template_id        text not null,                  -- for voice QA + dedupe, never shown
  subject_bird_ids   uuid[] not null default '{}',
  novelty_score      real not null,
  created_at         timestamptz not null default now()
);
create index on notebook_entry (aviary_id, observed_at desc);

-- ─── social ──────────────────────────────────────────────────────────────────
create table invitation (
  id                  uuid primary key,
  aviary_id           uuid not null references aviary(id) on delete cascade,
  invitee_email_ciphertext bytea not null,
  invitee_email_lookup_hash bytea not null,
  token_hash          bytea not null unique,
  created_at          timestamptz not null default now(),
  expires_at          timestamptz not null,          -- created_at + 30 days
  first_used_at       timestamptz,
  revoked_at          timestamptz
);
create index on invitation (aviary_id, created_at desc);

create table visit (
  id                  uuid primary key,
  invitation_id       uuid not null references invitation(id) on delete cascade,
  started_at          timestamptz not null,
  last_seen_at        timestamptz not null,
  approx_duration_s   integer not null default 0
);

-- ─── settings ────────────────────────────────────────────────────────────────
create table account_settings (
  account_id          uuid primary key references account(id) on delete cascade,
  reduced_motion      text not null default 'system'  -- 'system' | 'on' | 'off'
                        check (reduced_motion in ('system','on','off')),
  captions            text not null default 'auto'    -- 'auto' | 'on' | 'off'
                        check (captions in ('auto','on','off')),
  audio_enabled       boolean not null default true,
  visit_notifications boolean not null default false, -- OFF by default, per PRD
  narration_verbosity text not null default 'standard'
);
```

### 4.2 What is deliberately absent from the schema

There is no `visit_count`, no `session_count`, no `streak`, no `days_active`, no `last_visit_date` on the account, no per-day rollup table of user activity. These absences are load-bearing: the non-goals file says the refusal has to survive future reasonable-looking pitches, and the most durable way to refuse a streak counter is to not have the column that would make it a two-hour ticket. `presence_window` rows are the only record of when the user was present, they are consumed by the tick, and §4.3 retires them.

### 4.3 Retention

- `interaction_event`: 90 days, then dropped by partition. The tick has long since folded them into canonical state; retaining them longer creates a behavioral history with no product use and a real privacy cost.
- `presence_window`: 180 days (long enough to re-derive drift if a calibration bug requires a replay), then dropped.
- `notebook_entry`: forever, per the PRD's "scroll back indefinitely."
- `visit`: 12 months.
- Partition `interaction_event` by month; dropping a partition is the retention job.

### 4.4 The trait-exposure boundary

`bird` is never selected directly into a response DTO. The only read path to bird state for clients goes through `renderBird(bird, ctx) → BirdRenderState`, which emits qualitative, quantized fields (see §6.3). Enforced three ways: (1) the DTO type has no numeric trait fields; (2) a repository-layer lint rule forbids importing the `Bird` row type into the `api/dto` module; (3) a contract test asserts the JSON of a snapshot response for a fixture aviary with extreme trait values is byte-identical to one with mid-range values *except* for the fields §6.3 permits.

---

## 5. Simulation engine

### 5.1 The tick — cadence, sharding, and the cold-account problem

The tick advances canonical state every **60 s** of aviary time. Ticking every account every minute is trivially affordable at launch and quadratically wasteful at scale, so:

**[Decision] Hot/cold tick scheduling with deterministic catch-up.**

- **Hot set** — aviaries with a presence window in the last 30 minutes, or a snapshot read in the last 10 minutes. Ticked live every 60 s by `sim-worker`, sharded by `hash(aviary_id) % N` over a Redis sorted-set schedule.
- **Cold set** — everything else. Not ticked live. On the next snapshot read (or notebook pass), the engine runs **catch-up**: it replays the missed ticks deterministically and writes the result before serving.

This is only legitimate because of the next rule, which the team must treat as a hard invariant:

> **Tick determinism invariant.** For any aviary state S and any elapsed interval, stepping the simulation live minute-by-minute and replaying the same interval in catch-up produce byte-identical output state.

Enforced by: (a) all randomness comes from a counter-based PRNG seeded `(aviary.rng_seed, bird_id_low64, tick_seq, channel)` — never from wall-clock or `Math.random()`; (b) the step function is pure `(state, inputsForThatMinute) → state`; (c) a property test replays random event streams both ways and asserts equality. Any code that breaks this breaks multi-device coherence and the "aviary that has been running" promise simultaneously, so the test is a merge blocker.

Catch-up is bounded: a maximum of 2,880 steps (48 h) is replayed at full fidelity; beyond that the engine coarsens to hourly steps for the interior and replays the final 48 h at full fidelity. Coarsening is safe because absent-user intervals have zero drift input by definition and mood dynamics over multi-day gaps are dominated by the day/night signal, which is closed-form. The coarsening boundary is itself deterministic (keyed to `tick_seq`), so the invariant holds.

**[Assumption]** The PRD says "the tick runs whether or not any client is connected." Read literally that mandates live ticking for every account forever. This plan reads it as a statement about *observable semantics* — the aviary the user returns to must be the aviary that ran — and satisfies it exactly via determinism. If product insists on literal live ticking for cold accounts, the change is a config flag (`SIM_COLD_LIVE=true`) plus capacity: at 1 M accounts a 60 s tick is ~17 k aviary-steps/s, which is affordable but pointless. Nothing else in the design changes.

### 5.2 What one tick does

```
tick(aviary, now):
  1. load aviary, birds, settings, and new interaction_events above sim_cursor
  2. fold presence pings → presence windows → credited presence-seconds     (§5.3)
  3. compute environment: local time-of-day phase, day/night light, weather (§5.6)
  4. for each bird, in stable id order:
       a. mood transition                                                    (§5.5)
       b. perch selection                                                    (§5.7)
       c. call scheduling                                                    (§5.8)
       d. bird-to-bird influence accumulation                                (§5.9)
  5. apply bird-to-bird influences (computed from step-start state, applied
     simultaneously — no order-dependence between birds)
  6. drift update, from this minute's credited presence + interaction signals (§5.4)
  7. advance simulated_through, tick_seq; write state; advance sim_cursor
  8. emit notebook candidates to notebook-worker                              (§9)
  9. invalidate snapshot cache key for this aviary
```

Steps 4–5 are split deliberately: computing every bird's influence from the *step-start* state and applying simultaneously makes the tick order-independent, which keeps the determinism invariant robust against future changes to iteration order.

Latency budget: p50 < 15 ms, p99 alarm at 5 s per the PRD's error budget. The step is pure arithmetic over ≤7 birds; anything approaching 5 s means an I/O regression, which is exactly what the alarm should catch.

### 5.3 Presence accounting

The client evaluates the three-condition conjunction locally and emits a `presence_ping` every **15 s** while all three hold:

```ts
function presentNow(now: number): boolean {
  return document.visibilityState === 'visible'
      && document.hasFocus()
      && (now - lastPointerOrKeyAt) <= ACTIVITY_WINDOW_MS;   // 4 min, see below
}
```

**[Decision] `ACTIVITY_WINDOW_MS = 240_000` (4 minutes).** The PRD says "a few minutes," leaning long, "because watching birds without moving is the actual product." Four minutes is long enough that a genuinely still watcher keeps presence, short enough that an unattended open-and-focused laptop stops crediting within one notebook-irrelevant interval. Calibration in M5 against session telemetry (aggregate only): target ≥95% of sessions with a subsequent interaction showing no presence gap.

Server-side hardening, because the client is not trusted:
- Pings are credited only if the session token is valid and the ping arrives within a ±90 s window of its `server_ts`. `client_ts` is advisory and never used for crediting.
- **Credit cap:** at most 60 presence-seconds credited per wall-clock minute per aviary, across all devices. Two tabs open do not double drift.
- Consecutive pings within 30 s extend the open `presence_window`; a gap >30 s closes it. A closed window is credited `min(sum of covered intervals, wall-clock span)`.
- Windows longer than 4 h are truncated with a telemetry counter (`presence.window_truncated`) — a 6-hour "watching" session is a bug or an abuse, not a user.
- Visitor sessions never create presence windows. Enforced by the visitor token lacking the `events:write` scope entirely, not by a check in the handler.

Both `settle` and session end (tab close, detected as ping cessation) terminate the window identically. There is no penalty, no difference in credit, no recovery surface.

### 5.4 Drift

Traits live in `[0,1]`. Drift is a **monotonic, saturating low-pass filter** over presence and interaction signals.

Per tick, for trait `T` on bird `b`:

```
signal_T(b)  = w_presence·P̂ + Σ_i w_{T,i}·I_i(b)          // all terms ≥ 0
gain_T       = k_T · (1 − T)                                // saturating: slows near 1
ΔT           = max(0, gain_T · signal_T · dt_minutes)       // clamped non-negative
T'           = min(1, T + ΔT)
```

`P̂` is credited presence-minutes in this tick, normalised to `[0,1]` (a full attentive minute = 1.0). The `max(0, …)` clamp is written **once**, in `applyDrift()`, and is the only place a trait is mutated. That function is 20 lines, has a property test asserting `∀ event streams: T' ≥ T`, and is the mechanical implementation of the no-Tamagotchi rule.

Weights, in the PRD's stated order:

| Signal | boldness | social warmth | vocal freq | plumage | curiosity |
|---|---|---|---|---|---|
| presence-minute (aviary-wide) | 0.35 | 0.35 | 0.30 | 0.45 | 0.25 |
| listen-in minute (focused bird) | 0.20 | 0.90 | 0.85 | 0.15 | 0.20 |
| offer accepted (that bird) | 0.15 | 0.20 | 0.05 | 0.05 | 0.80 |
| offer made (all birds, proximity-weighted) | 0.35 | 0.10 | 0.05 | 0.02 | 0.15 |
| settle | 0 | 0 | 0 | 0 | 0 |

Settle contributes nothing to drift, exactly as specified — it cleanly ends the presence window and quiets mood, nothing more.

**Calibration.** Base rate `k_T` is solved from the PRD's two named targets rather than guessed:

- Define **instrument-detectable** = Δtrait ≥ 0.010 (well above float noise, trivially assertable in tests).
- Define **user-visible** = Δtrait ≥ 0.080 for a trait with a visible expression (a boldness shift of 0.08 moves a bird's front-perch probability enough to change where the user habitually finds it).
- Define a **regular-visit week** = 5 sessions × 11 min attentive presence ≈ 55 presence-minutes, plus ~4 listen-ins and ~3 offers.

Solving for the aviary-wide presence path at mid-range trait 0.5: `ΔT_week = k_T · 0.5 · 0.35 · 55`. Setting `ΔT_week = 0.012` gives `k_presence ≈ 1.25 × 10⁻³ per presence-minute`. Over three weeks with the saturation term, that path alone accumulates ≈ 0.034; adding listen-in and offer contributions on the birds the user actually attends to brings *those* birds to ≈ 0.09–0.11 — over the visibility threshold at three weeks, under it at one. That is precisely the instruments-at-one-week / eyes-at-three-weeks gap the PRD asks for, and the asymmetry across birds (the listened-to bird drifts faster) is a feature: the user notices Pip changed, not "the birds changed."

These constants live in one versioned file, `sim/calibration.ts`, with the derivation in comments and a **calibration harness** (§13.2) that fails CI if a simulated user profile falls outside the target bands.

**Migration safety.** Changing `k_T` retroactively changes nobody's birds — drift is applied incrementally and never recomputed from history. Calibration changes are forward-only, versioned (`calibration_version` recorded per aviary), and rolled out to new aviaries first, then ramped.

### 5.5 Mood

Five states: `wary`, `content`, `curious`, `drowsy`, `alert`.

Mood is a weighted-softmax transition evaluated per tick, with hysteresis:

```
score(m) = base(m)
         + timeOfDay(m, phase)            // drowsy↑ at dusk, alert↑ early morning
         + recentInteraction(m, window)   // offer accepted → content↑; startle → wary↑
         + ambient(m, weather)            // rain → vocal-damping, wary↑ mildly; wind → alert↑
         + personality(m, bird)           // high boldness → wary penalty; high curiosity → curious↑
P(m) = softmax(score / τ)
```

Hysteresis rules, which matter more than the scores:
- `mood_dwell_until = mood_entered_at + dwell(m)`, with dwell drawn per-mood from `[6, 25]` minutes. No transition may fire before dwell expires, except on a **startle** input (alarm call from another bird, or a weather onset), which can force `alert`/`wary` immediately.
- Transitions are sampled at most once per tick and are *sticky*: `P(stay)` gets a +0.35 additive bonus so moods persist for tens of minutes rather than flickering minute to minute.
- **Personality gating:** `score(wary) -= 0.9 · boldness`. A high-boldness bird is materially less likely to enter wary on identical input, as specified.

**Daily-ish reset.** At the bird's first tick after local 04:00, mood re-seeds from a personality-weighted prior rather than to a neutral default. "Daily-ish" is implemented by jittering the reset hour per bird by ±90 min from `voice_seed`, so an aviary doesn't reset in unison.

**Cross-session persistence.** Mood is never touched on session open. The snapshot read path does catch-up ticking (§5.1) and returns whatever mood the simulation produced. There is no "session start" code path in the mood machine at all — the absence of that code is the enforcement.

### 5.6 Environment: day/night and weather

- **Day phase** is derived from `aviary.timezone` (IANA, reported by the client on session open, persisted, never inferred from IP). Phases: `night`, `dawn`, `morning`, `midday`, `afternoon`, `dusk`, `evening`. Boundaries are computed from local clock time, not solar position — solar would be more "correct" and would require a location we deliberately do not collect.
- **Nightjar exception:** the species pool contains one species whose call scheduler ignores the night vocal-damping term and whose mood prior at night favours `alert`. Night is not a dead state.
- **Weather** is an aviary-level state machine: `clear → rain → clear` and `clear → wind → clear`. Onset probability per tick is tuned to ~3 rains/week and ~5 wind events/week, drawn from the aviary PRNG so it is deterministic and replay-safe. Duration 4–12 minutes. Effects: rain multiplies call scheduling interval by 1.8 and adds a small `wary` term; wind adds `alert` for high-boldness birds and `wary` for low-boldness ones. Effects decay over ~10 minutes after the event ends. No thunderstorm, no snow, no user-facing weather affordance, ever.

### 5.7 Perch selection

Perch is a *signal the user reads*, never a control. Assignment per tick:

```
zonePreference = clamp(0.65·boldness + 0.25·moodZoneBias(mood) + 0.10·socialPull(neighbours))
   → maps to P(front), P(middle), P(back)
```

`moodZoneBias`: wary → back, drowsy → back/middle low, content → middle, curious → front/middle, alert → any, higher motion. `socialPull` draws high-social-warmth birds toward occupied slots. Zone changes are rate-limited (no more than one zone move per 3 ticks, absent a startle) so birds don't ping-pong, and the client renders the move as a flight, not a teleport (§8.4).

Lateral slot assignment avoids collisions and, on narrow viewports, the *renderer* compresses slot spacing rather than the simulation reassigning slots — this keeps the phone and laptop showing the same bird in the same place, which multi-device coherence requires.

### 5.8 Call scheduling and the call grammar runtime

**Scheduling (server).** Each bird carries `next_call_at`. On firing:

```
baseInterval = lerp(210s, 55s, vocal_frequency)          // chattier birds call more often
interval     = baseInterval
             · moodFactor(mood)                          // drowsy 1.9, alert 0.7, content 1.0…
             · dayFactor(phase)                          // night 3.0 (except nightjar), dawn 0.8
             · weatherFactor(weather)                    // rain 1.8
             · jitter(0.7, 1.4)                          // from PRNG — never a fixed cadence
```

The server decides *that* a call happens, its `motif_id`, and its `phrase_seed`. It does not synthesize audio. The snapshot carries scheduled and recent calls with their seeds; the client synthesizes.

**Grammar (shared package, runs on client).** A three-level structure:

```
signature  (per bird, FIXED at adoption from voice_seed)
   ├── timbre: harmonic profile, formant pair, attack/decay envelope shape, vibrato depth
   └── motif subset: 3–4 motifs drawn from the species' 8-motif library

phrase     (per call, from phrase_seed)
   └── sequence of 1–4 motif instances with rests, chosen by a small weighted grammar:
       Phrase → Motif | Motif Rest Phrase | Motif Motif' (Motif' = varied repeat)

realisation (per playback)
   └── motif → note events, with mood/personality modulation applied to:
       tempo (±25%), pitch centre (±2 semitones), rest length, phrase length,
       ornament density (trill count, glide depth)
```

> **Recognizability invariant.** Mood, drift, weather, and time of day modulate *tempo, pitch centre, phrase length, and ornament density*. They **never** modulate timbre, formants, or motif-subset membership. Those come from `voice_seed`, which is fixed at adoption and is not a drifting trait.

This is the mechanism behind "a user who has spent two weeks with Pip should know Pip's call by ear." It is stated as an invariant, not a convention, because the natural instinct during audio polish is to let mood tint the timbre — and that is precisely what would erode per-bird identity over time. A unit test asserts the timbre parameters produced for a bird are identical across all five moods and across trait vectors at both extremes.

**Chorus.** Chorus is emergent, not scheduled: when two or more birds have overlapping call windows, a high-`social_warmth` bird gains a call-response bonus that pulls its `next_call_at` earlier (bounded, one response per triggering call). Because each call is independently realised with its own seed, two simultaneous calls genuinely differ — which is the property the PRD says stacked loops cannot deliver.

**Cap:** at most 3 concurrent voices; a 4th scheduled call is deferred by 1–3 s rather than dropped. This is a mix-clarity rule as much as a CPU one.

### 5.9 Bird-to-bird interaction

Three effects, all computed from step-start state and applied simultaneously:

1. **Call-and-response** — a call from bird A gives bird B a response bonus scaled by `B.social_warmth · A.social_warmth`, decaying over ~20 s.
2. **Wary contagion** — a bird entering `wary` adds a `wary` score term to birds in adjacent perch zones for ~3 ticks, damped by the receiver's boldness. Contagion is capped at one hop per tick so an aviary cannot cascade into all-wary.
3. **Perch gravity** — high-social-warmth birds bias toward occupied zones (§5.7).

### 5.10 Return-greeting

Greeting is the anchor moment and gets a dedicated, testable subsystem. On `session_open`:

```
absence   = now − last presence-window end        (per aviary)
band      = short (<15 min) | medium (<24 h) | long (<7 d) | very_long (≥7 d)

eligible  = birds where mood ≠ drowsy-at-night and cooldown elapsed
weight(b) = 0.55·boldness + 0.30·social_warmth + 0.15·moodGreetBias(mood)
            − 0.25·(greeted_in_last_session ? 1 : 0)      // rotate, gently
greeter   = weighted sample from eligible using session-open PRNG
```

- **Exactly one bird** greets in the first 1–2 s. If a second bird responds, it is scheduled at a randomized `+900–2600 ms` offset, never in unison.
- **Form** is selected by `(band, boldness, mood)` from a *composition space*, not a variant list:
  - `short` → a glance up mid-preen; no call, or one short motif at low amplitude
  - `medium` → head-tilt + one call phrase; a bold bird adds a step toward the front perch
  - `long` → longer phrase (3–4 motifs), a hop forward one zone, higher chance of a second bird responding
  - `very_long` → a re-orientation: forward zone move, extended phrase, slower settle back
- **Variation is real:** phrase, ornamentation, motion timing offsets, head-tilt angle, and step timing are all drawn from a per-session seed. There is no enumerable set of greeting animations. A test asserts 100 successive greetings from a fixed bird produce ≥95 distinct realisation fingerprints.
- **A wary or low-boldness bird may not greet at all on a given day** — `eligible` can be empty for a bird, and if it is empty for all birds, no greeting fires. Silence is a valid greeting outcome and is not backfilled with a fallback cue. This is deliberate: a guaranteed greeting is a canned greeting.

**No textual welcome exists anywhere.** There is no toast component, no banner component, no snackbar, and no "welcome" string in the i18n catalogue. §13.3 makes adding one fail CI.

---

## 6. API surface

### 6.1 Conventions

- Base `/v1`. JSON. `Authorization: Bearer <session token>` (httpOnly `Secure` `SameSite=Lax` cookie for the browser; bearer for the API contract).
- All mutating requests carry a client-generated UUIDv7 `Idempotency-Key`; replays return the original result.
- Errors: `{ "error": { "code": "...", "message": "..." } }`. `message` is **matter-of-fact voice**, always. Naturalist voice never appears in an error body; a lint rule flags lowercase-leading error strings in the error catalogue.
- Rate limits per session and per IP; per-email limits on link and invite issuance.

### 6.2 Endpoints

**Auth / identity**

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/auth/link` | `{email}` → always `202`, regardless of account existence (no account enumeration). Rate-limited per email hash. |
| `POST` | `/v1/auth/consume` | `{token}` → sets session cookie; marks link consumed; single-use. `410` on expired/consumed with the PRD's exact copy. |
| `GET` | `/v1/sessions` | List of `{id, device_label, created_at, last_seen_at, current}`. |
| `DELETE` | `/v1/sessions/{id}` | Revoke. Revoking current session signs out. |
| `POST` | `/v1/account/email` | Start email change; sends verification to new address. Old address remains valid until verified. |
| `POST` | `/v1/account/email/verify` | Commit the change. |
| `POST` | `/v1/account/export` | Enqueue export; emailed download link (signed URL, 24 h TTL). |
| `POST` | `/v1/account/delete` | Soft-delete; sets `pending_deletion`. |
| `POST` | `/v1/account/restore` | Recover during the 30-day window. |

**Aviary read**

```http
GET /v1/aviary/snapshot
If-None-Match: "a1f3-2291"
```

```jsonc
{
  "aviary": {
    "id": "…",
    "serverTime": "2026-07-26T14:03:11Z",
    "localPhase": "afternoon",
    "light": { "warmth": 0.62, "brightness": 0.81 },   // quantized to 2dp
    "weather": { "kind": "clear" },
    "settled": false,
    "tickSeq": 184920,
    "nextSnapshotAfterMs": 60000
  },
  "birds": [
    {
      "id": "0192f0…",                    // stable for life
      "name": "pip",
      "species": "warbler",
      "mood": "curious",                   // qualitative, user-visible via motion
      "perch": { "zone": "front", "slot": 1 },
      "render": {                          // see §6.3 — quantized, qualitative
        "plumage": 3,                      // 0–4 band, NOT saturation value
        "posture": "alert",
        "motionEnergy": "moderate",
        "approachBias": "forward"
      },
      "voice": {
        "signatureSeed": "9f2c…",          // timbre identity; opaque to the client's UI
        "motifSet": ["m2","m5","m7"],
        "callCadence": "frequent"          // qualitative band, NOT vocal_frequency
      },
      "scheduledCalls": [
        { "atMs": 4200, "motifId": "m5", "phraseSeed": "31ab…" }
      ]
    }
  ],
  "greeting": {                            // present only on session_open snapshots
    "birdId": "0192f0…",
    "band": "medium",
    "seed": "77c1…",
    "staggerMs": [0, 1450]
  }
}
```

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/aviary/snapshot` | Triggers catch-up tick if cold. `ETag`; `304` on no change. p95 < 120 ms warm, < 400 ms with catch-up. |
| `GET` | `/v1/notebook?before=<cursor>&limit=30` | Reverse-chronological, keyset paginated, read-only. |
| `GET` | `/v1/settings` / `PATCH` | Account + accessibility settings. |
| `GET` | `/v1/narration` | Optional server-generated narration text (see §11.1); returns the current prose paragraph with an `ETag`. |

**Aviary write — events only**

```http
POST /v1/events
{ "events": [
    { "id": "0192…", "kind": "presence_ping", "clientTs": "…" },
    { "id": "0193…", "kind": "listen_in_start", "birdId": "0192f0…", "clientTs": "…" },
    { "id": "0194…", "kind": "offer", "birdId": null,
      "payload": { "offerKind": "seed", "targetZone": "front" }, "clientTs": "…" }
] }
→ 202 { "accepted": 3, "rejected": [] }
```

Batched (client flushes every 15 s or on page-hide via `sendBeacon`), idempotent by event `id`, append-only. **There is no endpoint anywhere in the surface that writes personality, mood, or perch.** The absence is the enforcement; an API-surface test enumerates all routes and asserts none has a write path reaching the `bird` trait columns.

**Adoption / birds**

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/aviary/adopt` | Onboarding only. Server selects 2 species; body carries only names. No catalogue is returned — the user is not offered a choice of species, per the PRD. |
| `PATCH` | `/v1/birds/{id}` | `{name}` only. Rename has no effect on any other field. |
| `GET` | `/v1/aviary/newcomer` | Returns `{available: bool, species, suggestedName}` when aviary age crosses a threshold. Age is the sole gate. |
| `POST` | `/v1/aviary/newcomer/accept` | Adopts, up to the cap of 7. |

**Visits**

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/invitations` | `{email}` → issues one-time link, 30-day expiry, emails the invitee. Host-authenticated. |
| `GET` | `/v1/invitations` | Outstanding + used, for the settings surface. |
| `DELETE` | `/v1/invitations/{id}` | Revoke. Effective at the visitor's next snapshot pull. |
| `GET` | `/v1/visits` | The visit log: `{visitorEmail, startedAt, approxDurationS}`, newest first. Pull-only; no badge, no unread state. |
| `POST` | `/v1/visit/redeem` | `{token}` → issues a **visitor token**: scope `snapshot:read` only, bound to one aviary, 2 h TTL, no `events:write`, no notebook, no settings. |
| `GET` | `/v1/visit/snapshot` | Same DTO as the host's snapshot, minus `greeting`. Returns `410 visit_unavailable` if revoked/expired — the visitor's client renders the matter-of-fact surface. |

The visitor token's *scope* is what makes "visitors cannot interact and do not drift the host's birds" true. There is no code path where a visitor token reaches the event ingest handler; the check is at the auth middleware, not in business logic.

### 6.3 The render-state projection

The snapshot never carries a trait number. `renderBird()` projects the vector into qualitative, quantized fields:

| Trait | Projected as | Quantization |
|---|---|---|
| plumage_saturation | `render.plumage` | 5 bands (0–4) |
| boldness | `render.approachBias` | `retiring \| neutral \| forward` (also expressed by perch, which is already visible) |
| vocal_frequency | `voice.callCadence` | `sparse \| occasional \| frequent` |
| curiosity | folded into mood weighting + `render.motionEnergy` | `still \| gentle \| moderate \| lively` |
| social_warmth | not projected directly; visible only through greeting order and chorus behaviour | — |

Coarse banding is deliberate on two counts. It makes the numbers unreconstructable from the wire (a user cannot watch the API and chart their bird's boldness), and it forces the client to express personality through *behaviour* rather than through a parameterized visual dial — which is what the PRD means by "personality should be felt by watching the bird."

---

## 7. Sync model

### 7.1 The model in one paragraph

There is one canonical record. The server is its only writer. Clients read snapshots and write events. Therefore there is nothing to sync, nothing to merge, and no conflict class to resolve — which is why this section is short and why the PRD spends its words on *why* rather than *how*.

### 7.2 Rules

1. **Server-only writes to personality.** Only `sim-worker`'s `applyDrift()` writes trait columns. Enforced by a Postgres role: `api_rw` has `SELECT` on `bird` and `UPDATE` on `name` only; the trait columns are `UPDATE`-revoked at the column grant level. `sim_rw` is the only role with trait `UPDATE`. This makes the rule a database permission, not a code review habit.
2. **Additive deltas, never absolute values.** Client events say "user listened in to Pip for 3 minutes." The server decides what that means. No request body anywhere in the API contains a trait value.
3. **Ordered log consumption.** The tick consumes `interaction_event` by `(aviary_id, seq)` above `sim_cursor.consumed_through_seq`, in a single transaction with the state write. Cursor advance and state write commit atomically, so a crash mid-tick reprocesses the same events and — by determinism — produces the same state.
4. **Single-writer per aviary.** `sim-worker` takes a Postgres advisory lock on `hash(aviary_id)` for the duration of a tick. Two workers cannot tick one aviary. Catch-up on the read path takes the same lock, so a read-triggered catch-up and a scheduled tick serialize.
5. **Optimistic concurrency on the state row.** The state write is `... WHERE tick_seq = :expected`. A mismatch aborts and retries from a fresh read. Belt-and-braces behind the advisory lock.

### 7.3 Multi-device behaviour

Two devices open: both pull snapshots; both see identical mood, perch, drift, weather. Both write events; events from both land in one log; the tick folds both. The 60-second credit cap (§5.3) prevents two open tabs from double-crediting presence — a real correctness detail, since a user with a laptop and a phone on the same desk is a normal case, and "two devices = double drift" would silently corrupt calibration for exactly the users the multi-device feature is for.

Listen-in is **per-device, client-local** for the audio mix, but the *event* is logged and drifts the bird once (dedup: overlapping listen-in windows on the same bird from different sessions are merged into a single interval before crediting). **[Decision]** The alternative — making listen-in a synced aviary-wide state so the phone's mix follows the laptop's — was rejected: listen-in is an act of attention by a person at a device, not a property of the aviary.

### 7.4 Conflict surfaces (the rare cases)

Only three states can surface to a user, all in matter-of-fact voice, all copy taken verbatim from the PRD:

- Expired/replayed magic link → *"We couldn't sign you in. The link may have expired. Try requesting a new link."*
- Session expiry mid-view → *"Your session timed out. Sign in again to keep watching."*
- Snapshot fetch failing repeatedly → *"Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."*

Transient single-fetch failures do **not** surface anything: the client retries with jittered backoff and keeps rendering the last scene model. A bird that keeps preening through a 3-second network blip is both better UX and more honest than an error banner — the aviary genuinely is still there. The error surface appears only after ~20 s of continuous failure.

---

## 8. Frontend rendering pipeline

### 8.1 Technology **[Decision]**

**Canvas 2D with a pre-rasterized sprite/atlas pipeline, plus a parallel non-visual DOM accessibility tree.**

Considered and rejected:
- *WebGL/WebGPU* — comfortably fast enough, but adds shader tooling, context-loss handling, and a driver-bug surface for a scene of ≤7 birds and some leaves. Overkill against a 2 MB budget and a 5-year-old-laptop target where Canvas 2D is hardware-composited on all four supported browsers.
- *SVG/DOM-per-bird* — best accessibility story for free, but per-frame attribute mutation on dozens of nodes at 60 fps on a 2019 laptop is the classic way to miss the frame budget.

Canvas 2D + a *separate*, visually-hidden DOM mirror gets both: a cheap render loop and a real, focusable, screen-reader-navigable element per bird. The DOM mirror updates on state change (seconds), not per frame.

Framework: **Preact** for the chrome (top bar, settings, notebook, onboarding) — small, sufficient, and it never touches the render loop. The aviary canvas is framework-free.

### 8.2 There is no load state

The render loop starts on the *first* frame with a real scene model. This is achieved by delivering the bootstrap snapshot with the HTML:

1. Edge HTML request → the edge worker authenticates the session cookie, fetches (or catch-up-ticks) the snapshot, and **inlines it into the HTML** as a JSON script tag.
2. The critical bundle (renderer + scene model + one species' sprite set) is preloaded via `<link rel=modulepreload>` in the same document.
3. First bird draws as soon as the critical bundle parses — typically before the full bundle, always before audio init.

The consequence: **the code has no "loading" branch**. There is no spinner component, no skeleton, no fade-from-static, no entry animation. If the inlined snapshot is absent (cold edge, slow origin), the client renders the **quiet field** — sky gradient at the correct local-time colour, one or two faint ambient motion cues — which is the same surface as the empty-aviary state (§8.7). It is a real scene, not a placeholder, and it transitions to birds by the birds *arriving*, not by a crossfade from a loader.

`prefers-reduced-motion` is read before first paint from the inlined settings so the correct render profile is used on frame one — the reduced-motion user never sees a flash of full motion.

### 8.3 Frame loop and layer budget

```
rAF(t):
  dt = t − lastT  (clamped to 50 ms to survive tab-throttle wakeups)
  if dt > 2000: request fresh snapshot (suspend/resume detection), continue rendering
  advance interpolators
  advance procedural idle motion
  advance ambient ornaments
  composite layers:
     [sky+light]        redraw only on phase change (≈ every 30 s) → cached bitmap
     [background]       static + slow parallax offset → cached bitmap, translated
     [midground perches] static → cached bitmap
     [birds]            redrawn every frame; ≤7 sprites with transform + 2–3 part offsets
     [foreground]       occasional leaf/branch pass → cached bitmap, translated
     [weather]          only during weather events
  draw dirty regions only when the scene is otherwise still
```

Budget on the reference machine (2019 mid-range laptop, integrated graphics): **≤ 6 ms/frame** for the whole composite, leaving headroom. Birds are drawn from pre-rasterized part atlases (body, head, wing, tail per species per plumage band) composed with transforms; there is no per-frame vector rasterization.

### 8.4 Interpolation between snapshots

Snapshots arrive ~60 s apart; the scene must never teleport.

- **Perch changes** are rendered as a *flight*: a short easing arc with wing-beat animation, 500–1200 ms, triggered when the applier sees a zone change. The bird departs immediately on snapshot apply; it does not wait.
- **Mood changes** cross-fade the posture parameters over 2–4 s. A bird does not snap from fluffed to alert.
- **Calls** are scheduled ahead of time in the snapshot (`scheduledCalls[].atMs`), so audio and beak/throat motion are frame-accurate without a round trip.
- **Between scheduled events**, motion is entirely client-procedural (§8.5) — this is what makes a 60-second snapshot interval invisible.

If a snapshot arrives that contradicts in-flight client motion (rare: a bird mid-flight to front is now reported at middle), the applier **completes the current motion, then re-targets**. Never a snap-back.

### 8.5 Idle micro-motion

Idle motion is a per-bird small state machine driven by a client PRNG seeded from `(bird.id, tickSeq)` — so two devices showing the same bird show *similar* idle behaviour without needing to be frame-synced (they are not, and needn't be).

Behaviours: `preen` (head to wing, 2–5 s), `scan` (head sweep, 1–2 s), `tilt` (toward a recent sound source), `shuffle` (weight reset, 400 ms), `fluff` (feather puff, 1 s), `settle-low` (drowsy posture hold), `watch` (track a passing leaf).

Selection weights are mood-shaped, exactly as the PRD specifies: wary → `scan` heavy, further-back framing, quicker head moves; content → `preen` heavy; curious → `tilt`/`watch` heavy, more forward lean; drowsy → `settle-low`/`fluff`, slow blink, minimal motion; alert → upright posture, fast short `scan`s.

> **No bird is ever in a hold pose with zero motion.** The lowest-energy state (drowsy) still carries breathing (a ±1.5 px body scale oscillation at ~0.4 Hz) and occasional blinks. A test asserts that for every mood, the per-bird motion energy over any 10-second window exceeds a floor. "Reads as paused" is a bug with a test.

Micro-motion never fires in unison across birds: each bird's behaviour timer carries a per-bird phase offset.

### 8.6 Ambient ornaments

Leaves and feathers are pure client ornaments with no server state, spawned on a Poisson process (~1 per 12 s, jittered), following a slow sinusoidal fall path with per-instance drift, and pooled (fixed array of 8, recycled — no allocation per leaf, which matters for §10.4). Parallax is subtle: background offset ≤ 6 px, foreground ≤ 14 px across the full scene width, driven by nothing but a slow autonomous oscillation (there is no camera and no pointer-parallax — pointer parallax would make the scene respond to the cursor, which is an app behaviour, not a window behaviour).

### 8.7 Scene layout, responsiveness, and the empty state

- The scene is laid out in a **virtual coordinate space** (1600×900 units) and mapped to the viewport by a fit transform that preserves aspect while guaranteeing every perch slot's bird bounding box lies inside the safe area. On narrow viewports the mapping *compresses horizontal spacing* between zones and slots rather than cropping; on wide viewports it *expands* spacing up to a max, then letterboxes with continued sky/background.
- A layout invariant test renders all 7 birds at every zone/slot across 12 viewport sizes (320×568 → 2560×1440) and asserts zero bounding boxes are clipped and no two overlap by more than 15%.
- **Empty-aviary state** (post-signup, pre-first-bird) is the quiet field: sky at correct local-time colour, perches, one faint ambient cue. The first bird then *flies in* to its starting perch, followed by the second at a staggered offset. After that, the aviary is never empty again — a code invariant, since birds are never removed.

### 8.8 Top bar

Four affordances only: account/settings, accessibility settings, notebook, offer. Plus the settle gesture (grouped with the offer affordance in a single "gestures" cluster to keep the count at four visual anchors) — **[Assumption]**: the PRD lists the top bar contents as account/settings, accessibility, notebook, and offer, and separately says settle is triggered "from the top bar." This plan renders settle as a small companion control adjacent to the offer affordance rather than a fifth top-level icon, keeping the bar sparse. If the design review prefers a fifth icon, the change is cosmetic.

Fade behaviour: after **3.5 s** of cursor stillness and no keyboard activity, the bar animates to `opacity: 0.12` over 800 ms. It returns to full opacity in 120 ms on pointer move, key press, or focus entering the bar. It **never** fades while a top-bar element has keyboard focus, while a panel is open, or when `prefers-reduced-motion` is on (in that mode it steps between the two opacities over 400 ms rather than easing). Keyboard-only users are never chasing an invisible control.

### 8.9 Reduced-motion rendering profile

Not a flag inside the renderer — a **second render profile** the same scene model drives:

| Aspect | Default | Reduced-motion |
|---|---|---|
| Idle micro-motion | continuous animation | 2.5 s cross-fades between still poses drawn from the same pose set |
| Flight between perches | eased arc with wing-beats | 1.8 s cross-fade: bird fades out at origin, in at destination |
| Breathing | 0.4 Hz oscillation | none; pose held between cross-fades |
| Ambient leaves/feathers | active | removed entirely |
| Parallax | subtle | none |
| Day/night colour shift | continuous | retained, slowed ~2× |
| Weather | animated rain streaks | colour/light shift + caption/narration mention; no particle motion |
| Calls, drift, mood, notebook | unchanged | unchanged |
| Top bar fade | eased | stepped |

The pose sets are authored deliberately (a preen sequence *is* a set of 4 poses), so the cross-fade mode has its own slow, quiet aesthetic rather than looking like a broken animation. Design signs off on the reduced-motion surface as a designed surface, in the same review as the default surface — this is on the M4 milestone, not a follow-up.

---

## 9. Field notebook

### 9.1 Generation architecture

`notebook-worker` consumes tick outputs and runs a three-stage pipeline:

```
detectors  →  novelty scoring  →  sparsity governor  →  prose realisation  →  entry
```

**[Decision] Deterministic template-and-slot realisation, not an LLM.** Reasons: the voice must be exactly right every time and an LLM's failure mode here is *generic* prose, which is the one failure the PRD says breaks the spell across the whole product; entries must be reproducible for QA; latency and cost are irrelevant to the user but a per-entry model call adds an operational dependency to a feature that writes ~10 entries per user per month; and the per-account privacy boundary (§12) means shipping interaction history to an inference provider is a boundary we would rather not open. The combinatorial phrase bank (below) gives more real variation than a small model would, under our control.

### 9.2 Detectors

Each detector emits a candidate observation with a novelty score. v1 detectors:

- `greeting_order_change` — a different bird greeted first than usual ("pip greeted before wren today, first time this week")
- `first_of_period` — first call at dawn, first bird to the front perch this morning
- `long_quiet` — an unusually long interval with no calls
- `sustained_behaviour` — a bird preening or scanning far longer than its own baseline
- `chorus_event` — two or more birds calling together
- `weather_moment` — behaviour during/after rain or wind
- `perch_novelty` — a bird occupying a zone it rarely uses
- `offer_reaction` — a notable acceptance or a pointed refusal
- `drift_landmark` — a trait crossing a band boundary, described **behaviourally**: "pip has been coming to the front rail more often lately." Never numerically, never as a change announcement.
- `night_activity` — the nightjar calling late
- `bird_to_bird` — a call-and-response exchange, a wary mood spreading

### 9.3 Sparsity governor

Target: **~1 entry per 3 days** for a regularly-visited aviary, more when something genuinely unusual happens, never one per session.

```
budget(aviary) = 1 entry per 72 h, with a burst allowance of 2 within any 24 h
emit if:  novelty ≥ threshold(recent_entry_density)
      and template_id not used in the last 10 entries
      and subject/template pair not used in the last 30 days
      and daily cap (2) not exceeded
threshold rises as recent density rises  →  active users do not get a feed
```

Novelty is computed against the *aviary's own* baseline (this bird's usual perch, this aviary's usual quiet), never against a population baseline — the latter would require cross-account aggregation, which §12 forbids.

### 9.4 Voice rules — mechanically enforced

Realisation composes: `[time-or-weather opener] + [subject clause] + [specific detail]`, drawn from per-template phrase banks of 8–20 alternatives per slot, selected by seeded PRNG with recency exclusion. A typical template yields >2,000 distinct realisations.

Hard rules, checked by a lint test over the entire phrase bank at build time:

- Lowercase throughout (except bird names, which are user-supplied and rendered as given).
- Present tense. Past tense permitted only in explicit day-reference constructions ("greeted before wren today").
- **No second person.** The strings "you", "your", "you've" are banned from the notebook phrase bank outright. This single rule mechanically forecloses "you visited every day this week," "welcome back," and the whole class of user-behaviour observations.
- **The subject of every entry is a bird, the aviary, or the weather** — never the user. Enforced by the template schema: `subject` is a typed enum with no `user` variant.
- No numbers describing traits. No event-log phrasing ("session started"). No exclamation marks. No gamification vocabulary — a banned-lexicon test covers achievement, unlocked, streak, level, score, badge, milestone, congratulations, and ~40 more.

### 9.5 Surfacing

Notebook opens from the top bar as an overlay panel over a still-running aviary (the aviary keeps animating behind it — closing the notebook must not feel like returning to an app). Keyset pagination, infinite scroll backwards, virtualized list that releases DOM nodes and any retained entry objects on scroll-out (§10.4). Read-only: no edit, no delete, no annotate, no share, no export-as-image. Entries carry a date label in naturalist form ("tuesday"), not a timestamp.

---

## 10. Audio pipeline

### 10.1 Graph

```
                                   ┌──────────────┐
 per bird:  voice pool ──▶ voice gain ──▶ pan ──▶ │              │
            (oscillators,    (listen-in           │  aviary bus  │──▶ soft limiter ──▶ dest
             noise, filters)  automation)         │              │
                                   ┌──────────────┘
 ambient bed ─────────────────────▶│
 weather layer ───────────────────▶│
```

- One `AudioContext`, created on the first user gesture (browser policy), and never recreated.
- **Voice pool:** a fixed pool of 4 voices, each a pre-built subgraph (2 oscillators + 1 noise source + biquad + envelope gain), allocated once at init and **reused**. No `createOscillator()` per call — a call is a parameter automation over a pooled voice. This is what makes §10.4 (no memory growth) achievable rather than aspirational.
- Panning is subtle stereo, mapped from the bird's horizontal scene position (±0.4 max) so the audio scene matches the visual one.

### 10.2 Synthesis

A call motif is a list of note events `{ startOffset, duration, pitchCurve, timbreMix, ornament }`. Realisation:

- **Pitch curve** — piecewise exponential ramps on the oscillator frequency; glides and trills are curve shapes, not separate samples.
- **Timbre** — per-bird from `voice_seed`: harmonic mix ratio, a formant-ish biquad pair (centre + Q), noise-to-tone ratio, attack/decay shape, vibrato rate/depth. **Never modulated by mood or drift** (§5.8 invariant).
- **Modulation** — mood and traits scale tempo, pitch centre, rest lengths, phrase length, ornament density.
- Every playback is unique because `phraseSeed` is unique per call. There is no cached rendered buffer per motif; there are no audio files in the bundle at all (§10.6).

### 10.3 Listen-in mix

```
engage(bird):
  focused.gain.setTargetAtTime(1.0, ctx.currentTime, 0.45)     // τ ≈ 0.45 s → ~1.4 s to settle
  for each other: gain.setTargetAtTime(AMBIENT_FLOOR, ctx.currentTime, 0.55)
  ambientBed.gain.setTargetAtTime(0.55, ctx.currentTime, 0.55)

disengage():  all voices ramp back to 1.0 with the same τ
```

**`AMBIENT_FLOOR = 0.28` (≈ −11 dB), never 0.** The others quiet; they never mute. `setTargetAtTime` is an exponential approach, so the ramp is a *listening* change rather than a channel switch. Rapid focus changes are handled by re-targeting the same automation (no cancel-and-jump), so switching birds is a smooth re-balance, never a click.

Disengage triggers, all four from the PRD: clicking the focused bird again, focusing a different bird, clicking empty aviary space, moving keyboard focus away (including Escape).

### 10.4 Memory discipline

- Fixed voice pool; zero per-call node allocation.
- Ambient bed is a single looping procedural noise chain, built once.
- Scene ornaments pooled (§8.6).
- Notebook virtualization drops entry objects and DOM on scroll-out.
- Snapshot applier mutates the existing scene model in place; it does not build a new object graph per snapshot.
- Event queue is bounded (2,000 events) with drop-oldest on overflow plus a telemetry counter.
- CI soak test (§13.2): 30-minute headless session, heap sampled every 30 s; fails if the post-GC trend line slope exceeds 0.5 MB per 10 min or if `AudioNode` count grows at all.

### 10.5 Fallback

If `AudioContext` is unavailable, construction throws, or the context stays `suspended` after a gesture: the aviary plays in **graceful silence with captions forced on**. No recorded-audio path exists in the codebase. The user gets a single matter-of-fact line in accessibility settings explaining audio is unavailable in this browser; there is no in-aviary error surface (that would be an announcement).

Audio is also silent — without the caption forcing — when the user has audio off in settings or the tab is hidden. Suspend the context on `visibilitychange: hidden`; resume on visible.

### 10.6 Asset pipeline rule

The bundler's asset allowlist excludes `.mp3`, `.ogg`, `.wav`, `.m4a`, `.flac`, `.opus`. A build that imports one fails. This is the "no recorded audio, unconditional" rule as a build error.

---

## 11. Accessibility surfaces

### 11.1 Screen-reader narration

**[Decision] Client-side generation, sharing the notebook's realisation engine and phrase banks.** The narration describes the *current* scene, which the client already holds; generating it server-side would add a request per update and would desync from what is actually rendered. Sharing the phrase engine is what makes the voices identical rather than merely similar — the PRD's "same product, not two products glued together" requirement.

- Rendered into `<div role="status" aria-live="polite" aria-atomic="true">` in a visually-hidden region.
- **Cadence: one update per 30–60 s at idle**, jittered. A scheduler enforces a minimum 25 s gap between polite updates.
- **Priority bump** for user-initiated events (return-greeting, offer reaction, settle, listen-in engage): these use a separate `aria-live="polite"` region with a shorter minimum gap (6 s), and reset the idle timer. Still `polite`, never `assertive` — `assertive` interrupts, which is announcing.
- Content: a 2–3 sentence paragraph composed from scene state — who is where, in what posture, what the light is doing, what was just heard:

  > a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

- Composition rules identical to §9.4 (lowercase, present tense, no second person, observation not state-transition). One shared banned-lexicon test covers narration, captions, and notebook. A phrase-bank recency window prevents the same sentence twice in a session.
- Narration verbosity setting: `standard` (default) or `brief` (one sentence). No `verbose` — more narration is not more access.

### 11.2 The accessibility DOM mirror

A visually-hidden, focusable element per bird, kept in the scene's spatial order:

```html
<div role="group" aria-label="the aviary">
  <button id="bird-0192f0" aria-describedby="nar-0192f0">pip, on the front rail</button>
  <span id="nar-0192f0" hidden>a small grey bird, calling softly</span>
  …
</div>
```

Updated on state change only (seconds), never per frame. This mirror is what makes keyboard focus, screen-reader navigation, and hit-testing share one source of truth with the canvas.

### 11.3 Keyboard model

Exactly as the PRD specifies:

- `Tab` cycles top-bar items, then enters the aviary group and focuses the first bird.
- Arrow keys move focus between birds in scene order (left/right within the row, up/down between perch zones), using a roving-`tabindex` pattern so the bird group is a single tab stop.
- `Enter` triggers listen-in on the focused bird; `Escape` exits listen-in and returns focus to the bird.
- The offer affordance is reachable from the top bar and its panel is fully keyboard-navigable (arrow keys between the three offers, `Enter` to offer, `Escape` to close).
- Settle is reachable from the top bar; the 5-second undo is available via `Escape` as well as any click.
- Focus indicator: a 2 px soft outline plus a 4 px low-opacity halo, drawn in *both* a light and a dark variant, with the variant chosen per current aviary luminance so it reads at midday and at night. Contrast validated at ≥3:1 against the sampled background behind the focused bird, in every day-phase, in an automated test.

### 11.4 Captions

Generated from the same call AST that drives synthesis, at the moment of realisation, so the caption matches what actually played:

```
phrase [motif m5 ×2 with rest, low register, slow tempo]
   → "a low trill, paused, low trill again"
```

Composition maps AST features to phrase-bank fragments: register (`low`/`soft`/`bright`/`sharp`), shape (`rise`/`fall`/`trill`/`chatter`), count (`two-note`/`three-note`), and location when the bird is not the focused one (`from the back perch`). Rendered as small text near the calling bird — this is the one permitted text inside the scene region, and only while a caption is active. It fades in over 200 ms, holds for the call plus 1.2 s, fades out over 400 ms. WCAG AA against the sampled local background, using a subtle scrim if the sampled contrast falls short.

Settings: `auto` (on when audio is unavailable or muted), `on`, `off`. Default `auto`.

### 11.5 Reduced-motion

Covered in §8.9. The settings toggle is three-state (`system`/`on`/`off`) and defaults to `system`, honouring `prefers-reduced-motion` on first paint (§8.2).

### 11.6 Contrast and system-surface voice

All chrome text passes WCAG AA (4.5:1 body, 3:1 large). Accessibility settings, account settings, sign-in, and every error surface use **matter-of-fact voice** — normal capitalization, direct, no naturalist phrasing. A copy-review checklist item in the PR template names the boundary explicitly so it does not get re-litigated per surface.

### 11.7 Verification

Automated: axe-core on every chrome surface in CI; contrast assertions across all day-phases; keyboard-path E2E tests (tab order, arrow navigation, listen-in engage/disengage, escape from every panel); live-region cadence test asserting no more than one polite update per 25 s at idle.
Manual, before launch: a paid session with screen-reader users (NVDA/Firefox, VoiceOver/Safari desktop, VoiceOver/iOS) and with vestibular-sensitive users on the reduced-motion surface. The acceptance question is not "can they operate it" but **"does it feel alive to them"** — and it is a launch blocker, per the PRD's stance that accessibility ships with the product.

---

## 12. Privacy and the telemetry boundary

### 12.1 Architectural boundary

```
   simulation DB (Postgres)          telemetry store (separate)
   ├── accounts, birds, vectors      ├── request counts, latencies
   ├── interaction events            ├── error rates
   ├── notebook                      ├── anonymized session-duration histograms
   └── visits                        ├── render-frame timings
                                     ├── audio-context error counts
        ▲                            └── simulation-tick latencies
        │  NO PIPELINE CROSSES HERE
        ▼
   analytics warehouse: does not exist for this product
```

Enforced at four levels, because a policy alone is not a boundary:

1. **Network** — the telemetry collector runs in a separate service with no credentials for, and no network route to, the simulation database. Egress policy denies it.
2. **Schema** — the metrics schema is a closed allowlist of dimension names. `account_id`, `aviary_id`, `bird_id`, `email`, and any free-form dimension are rejected at ingest, with a counter for rejections.
3. **Code** — telemetry emitters take a typed `Metric` whose dimension type is a closed union; adding an account-scoped dimension is a type error.
4. **Review** — schema changes to the metrics allowlist require a named privacy reviewer on the PR.

Operational debugging of a specific account is done via authenticated, audited, time-boxed admin reads of the simulation DB — not via telemetry. Those reads are logged to an append-only audit table.

### 12.2 PII handling

- Email is stored once, on `account`, envelope-encrypted with a KMS data key. Lookup for sign-in uses `HMAC-SHA256(email, pepper)`, not the plaintext.
- Every other reference — logs, queue keys, cache keys, metrics, error reports, Sentry-equivalent payloads — uses the synthetic account UUID. A CI grep + a structured-logging serializer that redacts anything matching an email pattern enforce it. The serializer runs in production, not just in tests.
- Invitee emails on `invitation` follow the same rule. The visit log decrypts on read for display to the host, per request, never into a cache.
- Account export contains only the requesting account's data; delivered as a time-limited signed URL to the verified address.

### 12.3 Deletion

Soft-delete sets `status='pending_deletion'` and stops tick scheduling immediately (a pending-deletion aviary does not drift). Any signed-in page shows a matter-of-fact restore affordance. At day 30, a hard-delete job removes account, aviary, birds, events, presence windows, notebook, invitations, and visits, with `ON DELETE CASCADE` doing most of the work; backups age out on their own 35-day cycle, so the retention promise holds within one backup generation. The job emits a per-run count (not per-account) to telemetry.

---

## 13. Testing, tooling, and guardrails

### 13.1 Test layers

- **Unit** — pure simulation functions: drift, mood scoring, perch selection, call grammar, caption realisation, presence crediting.
- **Property-based** (fast-check):
  - `∀ event streams: traits are non-decreasing` (monotonic drift)
  - `∀ intervals: live-step == catch-up-replay` (determinism invariant)
  - `∀ mood inputs: no transition before dwell expiry` (unless startled)
  - `∀ traits, ∀ moods: timbre parameters identical` (recognizability invariant)
  - `∀ presence event streams: credited seconds ≤ wall-clock span`
- **Contract** — snapshot DTO carries no trait numbers; no route writes trait columns; visitor token cannot reach event ingest; error catalogue is all matter-of-fact.
- **Integration** — tick correctness against a seeded database; multi-device event interleaving; idempotent replay.
- **E2E** (Playwright) — onboarding through first bird; return-greeting fires within 2 s; listen-in mix ramp measured via `AudioContext` inspection; offer cooldown; settle + undo; notebook pagination; visit redeem and revoke; keyboard-only traversal.
- **Visual regression** — deterministic renders (PRNG pinned) at 12 viewports × 4 day-phases × 2 motion profiles.

### 13.2 Purpose-built harnesses

1. **Calibration harness** — simulates user profiles (`daily-watcher`, `weekly-visitor`, `lapsed`, `binge-then-absent`, `two-device`) over 1, 3, and 8 simulated weeks and asserts drift lands in the target bands: instrument-detectable (≥0.010) at week 1, user-visible (≥0.080 on attended birds) at week 3, and *no* trait decrease for the lapsed profile. Runs nightly; a regression here fails the build.
2. **Aliveness harness** — records 10 minutes of headless client render + audio events and asserts: no motion-energy floor violations per bird per 10 s window; no two identical call realisations; no two identical greetings across 100 sessions; no bird stationary in one perch zone for a full 10 minutes across a full-day simulation.
3. **Memory soak** — §10.4.
4. **Perf lab** — WebPageTest-equivalent on a throttled mid-tier Android profile over emulated 4G: asserts first-bird-visible < 500 ms and gzipped initial JS < 2 MB, per commit to main.
5. **Voice lint** — the banned-lexicon and composition rules (§9.4) run over the entire notebook + narration + caption phrase banks at build time.

### 13.3 Anti-drift guardrails (product integrity as CI)

These exist because the PRD says, repeatedly, that the failure mode is a well-meaning contributor adding a reasonable-looking thing:

- **Banned-component lint** — no module may export or import a component named `Toast`, `Snackbar`, `Banner`, `Notification`, `Badge`, `Confetti`, `Achievement`, `Streak`, `ProgressBar`, `Leaderboard`, or `Spinner`. None exists in the design system; adding one fails lint.
- **Banned-string lint** — the i18n catalogue may not contain "welcome back", "you've been", "days in a row", "streak", "achievement", "unlocked", "level up", "congratulations", "great to see you", "keep it up".
- **Second-person lint** — scoped to the naturalist surfaces (notebook, narration, captions, aviary chrome copy). System surfaces may and should use "you."
- **No-numbers contract test** — §4.4.
- **Schema guard** — a migration test fails if a new column matching `/streak|visit_count|days_active|score|level|xp|rank|points/` is added to any table.
- **PR template** — three checkboxes: *does this add a surface that announces? does this surface a number about the user? does this add a counter?* Any "yes" requires an explicit product sign-off, which for these categories the PRD has already refused.

None of this is bureaucracy for its own sake. The PRD's clearest engineering claim is that these refusals erode by increments; encoding them as build failures is the only form of refusal that survives a year of staffing changes.

---

## 14. Rollout

### 14.1 Milestones

**[Assumption]** Team of ~7: 2 backend, 2 frontend/render, 1 audio/DSP-leaning frontend, 1 design, 1 QA/infra part-time. Durations are engineering weeks and assume design runs one milestone ahead.

| # | Milestone | Weeks | Exit criteria |
|---|---|---|---|
| M0 | Foundations | 1–2 | Repo, CI, shared types package, Postgres schema + migrations, magic-link auth end to end, session management, synthetic-UUID and PII lint in CI |
| M1 | Simulation core | 3–6 | Tick worker with determinism invariant proven by property test; drift, mood, perch, weather, call scheduling; calibration harness green; snapshot + events API; catch-up path |
| M2 | Render + first bird | 5–9 | Canvas pipeline, scene model, interpolation, idle micro-motion, perch flight, day/night, ambient ornaments, responsive layout invariant test, top bar with fade, inlined bootstrap snapshot, **no load-state code path** |
| M3 | Audio | 8–11 | Voice pool, call grammar realisation, per-bird signatures, chorus, listen-in ramp, WebAudio fallback, memory soak green, recognizability invariant test green |
| M4 | Accessibility (ships with, not after) | 9–13 | Narration engine, DOM mirror, keyboard model, captions, reduced-motion profile with authored pose sets, axe + contrast + live-region tests, **design sign-off on reduced-motion as a designed surface** |
| M5 | Interactions + notebook | 11–15 | Return-greeting subsystem with variation test, offers + cooldown, settle + undo, notebook detectors/governor/realisation, voice lint green |
| M6 | Accounts, social, privacy | 14–17 | Settings, export, soft/hard deletion, invitations, visitor token + read-only view, revocation, visit log, telemetry boundary enforced at all four levels |
| M7 | Hardening + calibration | 17–20 | Perf budgets met on the reference devices, aliveness harness green, moderated a11y sessions passed, private beta feedback folded in, alerting live |

Critical path is M1 → M2 → M3; accessibility (M4) is deliberately overlapped with audio rather than queued behind it, because captions and narration depend on the call AST and the scene model, both of which exist by week 9.

### 14.2 Launch sequence

1. **Internal dogfood (week 14+)** — team accounts, real multi-day usage. The only reliable way to catch "feels canned" is to live with it for two weeks.
2. **Private beta, ~200 invited users (week 18)** — the key measurement is drift calibration against real presence patterns, gathered from the calibration harness re-run on aggregate presence *distributions* (bucketed session-length histograms), never from per-account inspection.
3. **Waitlist ramp (week 20+)** — 500 → 2,000 → open. Ramp gates: tick p99 < 1 s, snapshot p95 < 150 ms, first-bird p75 < 500 ms on the RUM histogram, zero calibration-band regressions.

### 14.3 Ramping birds-per-aviary

New-species availability is gated **only** on aviary age, per the PRD. Proposed schedule, to be confirmed against beta:

| Aviary age | Birds available |
|---|---|
| 0 | 2 (starters) |
| ~10 weeks | 3rd offered |
| ~7 months | 4th |
| ~12 months | 5th |
| ~18 months | 6th |
| ~26 months | 7th (cap) |

The offer appears as a quiet in-flow moment (a new bird has been around the aviary lately; would you like it to stay?) — not a notification, not a badge, not an email. If the user ignores it, it remains available; it is never re-pitched. Because v1 launches with no aviary older than zero, only the 3rd-bird threshold ships live-critical; the rest are configuration with a scheduled review.

**Audio-mix ramp risk control:** the 7-bird cap rests on call-signature recognizability. Before enabling the 5th bird in production, run a listener study (n≈20) asking participants to identify birds by call in 4-, 5-, 6-, and 7-bird aviaries. If recognizability drops below ~80% at 6, hold the cap lower and revisit with mix work. The PRD authorizes revisiting the cap only upward and only on audio-mix evidence; this study is the evidence.

### 14.4 Day-one instrumentation

Within the §12 boundary: request rates and latencies per endpoint; tick latency histogram with the p99>5 s alarm; tick backlog depth; catch-up replay length distribution; snapshot cache hit rate; first-bird-visible RUM histogram; render frame-time p95 histogram; audio-context init failure rate; caption/reduced-motion/audio-off adoption rates (as counts, no account dimension); magic-link issue/consume/expire counts; invitation issue/redeem/revoke counts; error rates by code. Synthetic checks: a browser fleet from 4 geographies loading a seeded demo account every 10 minutes, asserting first-bird timing and audio start.

Explicitly **not** instrumented: per-account session frequency, retention cohorts keyed to individual behaviour, per-bird interaction counts, any drift value per account, any "engagement" metric. The product does not have a north-star engagement metric, and building one would create exactly the incentive the non-goals file forbids. Operational health and aggregate performance are what we watch.

---

## 15. Risks

**R1 — Drift calibration is wrong in production.** *Likelihood: high. Impact: high.* The bands are derived from assumed session behaviour; real users will differ. Mitigation: calibration is a versioned constants file with a nightly harness; changes are forward-only and never recompute history; beta measures against aggregate presence distributions. Detection: the harness plus a synthetic-cohort dashboard. Fallback: adjust `k_T` forward for new aviaries first, then ramp — existing birds keep their history either way.

**R2 — Determinism invariant silently breaks.** *Likelihood: medium. Impact: severe.* A `Math.random()` or `Date.now()` reaching the step function makes catch-up diverge from live ticking, which corrupts multi-device coherence *without any error*. Mitigation: the step function is in a package with a lint rule banning `Math.random`, `Date.now`, and `new Date()`; time and randomness are injected. Property test on every build. Detection: a canary job that ticks a fixture aviary both ways hourly in production and alarms on divergence.

**R3 — Personality state loss.** *Likelihood: low. Impact: catastrophic and near-invisible.* The PRD names this as the worst possible failure. Mitigation: PITR on Postgres with 5-minute RPO; nightly logical backups of the `bird` table retained 35 days; a daily consistency job asserting every bird's traits are non-decreasing versus yesterday's snapshot (a *decrease* means corruption, since drift is monotonic — this is a uniquely strong integrity check the monotonicity rule hands us for free); restore drill quarterly.

**R4 — Audio uncanniness / procedural calls sound synthetic.** *Likelihood: medium-high. Impact: high.* Procedural bird calls are genuinely hard; a bad synth reads worse than no audio. Mitigation: audio prototype in week 3 as a standalone page, evaluated by ear against field recordings before the engine is built around it; DSP-experienced contractor budgeted for motif design; a listening review gate at M3 exit with a named accept/reject decision. Contingency: if the synth is not convincing by M3 exit, ship with a quieter, sparser mix (fewer, shorter calls with more silence) rather than louder or more frequent — silence is more alive than bad synthesis, and captions carry the meaning. Recorded audio is not a contingency at any point.

**R5 — Recognizability erodes as features land.** *Likelihood: medium. Impact: high.* The natural instinct during polish is to let mood tint timbre. Mitigation: the recognizability invariant test (§5.8), plus the 5th-bird listener study (§14.3).

**R6 — Accessibility regression after launch.** *Likelihood: medium. Impact: high.* Narration and captions share the scene model, but a renderer refactor could bypass it. Mitigation: the single-scene-model rule (§2.3) enforced by a lint rule preventing renderer modules from importing the snapshot DTO directly; axe + keyboard E2E in CI; a quarterly moderated screen-reader session as a standing calendar item, not a project task.

**R7 — Product erosion by increments.** *Likelihood: high over 12 months. Impact: total.* The non-goals file predicts this in detail. Mitigation: §13.3 guardrails as build failures, plus the scope test in §1.3 in the PR template.

**R8 — Time-to-first-bird missed on mobile.** *Likelihood: medium. Impact: high (it is an affective metric, not a perf metric).* Mitigation: budget enforced per commit from M2, not measured at the end; the edge-inlined snapshot removes a round trip; critical bundle is renderer + one species only, with the rest lazily loaded. Contingency: reduce first-paint fidelity (fewer plumage bands, simpler background) before reducing motion — the bird must be moving in the first frame even if it is drawn more simply.

**R9 — Presence signal inflation or gaming.** *Likelihood: low. Impact: medium.* A modified client could spam presence pings. Mitigation: the 60-seconds-per-wall-clock-minute server cap makes inflation structurally impossible regardless of client behaviour; window truncation catches the pathological case. Note this cap also silently protects against the honest two-device case, which is the more likely source of inflation.

**R10 — Cold-account catch-up latency spike.** *Likelihood: medium. Impact: medium.* A user returning after 3 months triggers a long replay on the read path. Mitigation: the 48 h full-fidelity / hourly-coarse split bounds replay to a few thousand cheap steps (< 50 ms measured on the reference schema); a lazy pre-warm job catch-up-ticks accounts that receive a magic-link request, so the replay usually happens before the aviary is even opened.

**R11 — Notebook prose reads as templated.** *Likelihood: medium. Impact: high (the PRD calls the notebook the most concentrated expression of the voice).* Mitigation: phrase banks sized for >2,000 realisations per template with 30-day subject/template recency exclusion; a writer (not an engineer) authors the banks; a dogfood review at M5 where the team reads two weeks of generated entries end to end and rejects any that read as filled-in.

**R12 — Magic-link deliverability.** *Likelihood: medium. Impact: high (it is the only front door).* Mitigation: reputable ESP with a dedicated subdomain, SPF/DKIM/DMARC from day one, delivery-failure alerting, and a matter-of-fact "didn't get the link?" surface with a resend and a plain-text explanation of spam-folder checking.

---

## 16. Open items for product/design (non-blocking)

These do not block engineering; each has a working default in this plan and a scheduled decision point.

1. **Activity window length** (default 4 min, §5.3) — confirm against beta presence histograms at M7.
2. **Settle as a fifth top-bar icon vs. a companion to offer** (§8.8) — design review at M2.
3. **New-bird pacing schedule** (§14.3) — only the 10-week threshold is launch-critical; confirm the rest before the first cohort reaches 7 months.
4. **Species pool composition** — six species including one nightjar-like; the other five are a design call, needed by M2 for silhouettes and by M3 for motif libraries.
5. **Notebook date labels** — naturalist weekday form assumed ("tuesday"); confirm behaviour for entries older than a week (proposed: "tuesday, in early june").
6. **Visit session TTL** (default 2 h, §6.2) — a visitor who leaves the tab open all day re-redeems; confirm this is acceptable versus a longer-lived visitor token.

---

## 17. Definition of done for v1

- Every in-scope item in §1.1 is shipped and behind no flag.
- All five harnesses in §13.2 green on main for seven consecutive days.
- All four performance budgets met on the reference devices, measured from the RUM histogram, not the lab.
- Moderated screen-reader and reduced-motion sessions passed, with the acceptance question being whether the aviary felt alive.
- The §13.3 guardrails are enforcing in CI, not merely documented.
- A team member has used the product daily for three weeks and can say, without prompting, that a specific bird changed.
