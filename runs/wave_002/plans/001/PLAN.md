# Pocket Aviary — v1 Implementation Plan

**Status:** ready for execution
**Audience:** the engineering team building v1 (client, simulation, platform, design-engineering)
**Source of truth for product intent:** the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`)

---

## 0. How to use this document

This plan interprets the PRD into an executable build. Three kinds of statement appear here, and they carry different weight:

- **Invariants** (§2) are binding. Each one has a named enforcement mechanism and a test. They are not style preferences; several of them are the product. Changing one requires changing the PRD first.
- **Specifications** (§3–§14) are the design the team should build. Deviate where implementation reality demands it, but record the deviation against the invariant it touches.
- **Assumptions** (§18) are calls this plan made where the PRD left a gap. Each is numbered `A-nn` and referenced inline. They are defensible, not sacred — but they are decided, so no one has to re-open them mid-build.

The glossary in `concepts.md` governs all naming, in code as well as in prose. `bird`, `aviary`, `call`, `mood`, `personality vector`, `drift`, `presence`, `listen-in`, `offer`, `settle`, `field notebook`, `visit`, `tick`. Type names, table names, event names, and metric names use these words and no synonyms. A lint rule enforces the banned synonyms (`pet`, `creature`, `character`, `song` for a call, `chirp`, `solo`, `select`/`highlight`/`pin` for listen-in) in identifiers and user-facing strings.

One naming exception this plan introduces deliberately: the bird's night state is called **`roosting`**, not "settled." The glossary reserves *settled* for the aviary's evening lighting state after the settle gesture, and overloading it in the mood enum would produce exactly the kind of quiet ambiguity that leads to a bug where triggering settle changes a bird's mood record. See A-04.

---

## 1. Scope

### 1.1 In scope for v1

| Area | v1 content |
|---|---|
| Aviary | One horizontal single-screen scene; three perch zones; day/night on the user's local clock; rare ambient weather; ambient leaf/feather drift; foreground/background parallax; no in-scene chrome |
| Birds | 2 starter birds per account, hard cap 7; ~6-species pool; hidden 5-trait personality vector; 6-state mood; procedural calls; mood-shaped idle motion; bird-to-bird interaction; stable identity; user-assigned renameable names |
| Simulation | Server-side tick at ~60s; presence accounting; monotonic drift; mood transitions; call scheduling; weather; perch selection; notebook entry generation; greeting selection |
| Interactions | Return-greeting; listen-in; offer (seed / song fragment / still pool); settle (with 5s undo); presence as interaction; field notebook (read-only, sparse) |
| Accounts | Single-user; magic-link sign-in; per-device revocable sessions; email change with verification; JSON export; 30-day soft delete then hard delete |
| Sync | One canonical server-side aviary; snapshot pull + client interpolation; append-only client event log; server-only personality writes |
| Social | Per-invite opt-in visits, read-only ambient, revocable, 30-day invite expiry, on-demand visit log, off-by-default visit notification toggle |
| Accessibility | Naturalist screen-reader narration; designed reduced-motion mode; runtime-generated call captions; full keyboard navigation; WCAG AA on all user copy |
| Platform | Web only; last two majors of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface |

### 1.2 Out of scope for v1 (refused, per `non_goals.md`)

Native mobile apps. Gamification of any kind — achievements, streaks, levels, scores, badges, XP, ranks, tiers, adoption counters, visit calendars, "you've been here every day" surfaces, milestone celebrations, opt-in dashboards of any of the preceding. Tamagotchi mechanics — death, hunger, distress, decaying happiness meters, negative drift on neglect. Social-network surfaces — profiles, follows, public feeds, discovery, friend-of-friend chains, mutual visits, comments, leaderboards, co-presence, avatars, chat. Push notifications and re-engagement email. Payments. Shared or multi-aviary accounts. Customizable scenes. Panning, scrolling, zooming.

These are not backlog items. §2 gives each one a mechanical guard so it cannot arrive by accident.

### 1.3 Deferred but not refused

SSO and password auth (magic link is v1). Native clients. Raising the seven-bird cap if future audio-mix work raises the recognizability ceiling. Additional species beyond the launch pool. Localization beyond `en` (the prose generator is built with a locale seam — see §10.2 — but only `en` ships).

### 1.4 What "v1 is done" means

1. All invariants in §2 pass their tests in CI.
2. Performance budgets in §14.1 hold on the reference device profile.
3. A 21-day live calibration cohort (§15.3, §16.1/M6) shows drift landing inside the target band.
4. The accessibility surfaces ship in the same release as the visual surfaces — not after (`accessibility_perf.md` is explicit that a v1.1 reduced-motion mode is a failed v1).
5. Manual affective QA (§15.4) signs off that the aviary reads as continuing, not starting.

---

## 2. Product invariants (binding)

Each invariant has an ID, an enforcement mechanism, and an automated check. "Enforcement" means the design makes the violation hard or impossible, not that we intend to be careful. Reviews catch what mechanisms miss; mechanisms catch what reviews miss.

| ID | Invariant | Enforcement mechanism | Automated check |
|---|---|---|---|
| INV-01 | The first frame the user sees has birds mid-action. No spinner, no entry animation, no fade-from-static, no "ready" transition. | Critical render path draws from an inlined/cached snapshot with in-flight transition phases (§8.4). No loader component exists in the codebase. | Source lint bans `spinner`, `loader`, `skeleton` identifiers in the aviary bundle. Playwright filmstrip test asserts a bird silhouette is painted in the first captured frame and that no full-scene opacity ramp occurs. |
| INV-02 | Slow-connection / cold-cache state is a quiet field, never a spinner. | The only pre-snapshot render path is `QuietField` (§8.5). | Test forces a 5s snapshot delay and asserts painted output is the field gradient plus ≤2 ornament sprites, and that the DOM contains no `role="progressbar"`. |
| INV-03 | No "Welcome back!" toast, banner, modal, or textual welcome on return. No "you've been gone X days" surface. | There is no toast/banner/modal primitive in the app shell at all. The only overlay primitives are the settings sheet, the notebook sheet, and the offer menu — all user-initiated. | Lint bans `toast`, `banner`, `snackbar`, `welcome` identifiers and `aria-live="assertive"`. Copy-registry test (§10.1) rejects any string matching the welcome lexicon. |
| INV-04 | No streak counter, visit calendar, days-visited count, session count, adoption count, or any surface that reflects the user's own behaviour back at them. | The client is never sent user-behaviour aggregates. The snapshot serializer has no field for them; the notebook template corpus is linted against user-subject templates (§10.3). | Snapshot schema test asserts no field name matches the behaviour-metric lexicon. Notebook corpus test rejects any template whose subject is the user. |
| INV-05 | Drift is monotonic toward expressive. No trait ever decreases. Neglect produces quietness, never distress, decay, or reversal. | `applyDrift()` clamps every per-trait delta to `>= 0` and the repository layer re-asserts `new >= old` before write, failing the tick on violation. | Property test: 10⁶ random (state, event-window) pairs, assert no trait decreases. Golden test: 60 simulated days of zero presence produces zero trait change. |
| INV-06 | Calls are procedurally synthesized. No recorded audio ships, in any path, including fallback. | No audio asset formats are permitted in the bundler config; `.mp3/.wav/.ogg/.m4a/.aac/.flac/.opus` are hard build errors. WebAudio unavailability routes to silence + captions (§9.7). | Build-time asset-type gate. Runtime test asserts `decodeAudioData` is never called. |
| INV-07 | A given bird's call is never byte-identical twice, and its timbral signature never changes. | Call plans are generated from a per-call PRNG draw; timbre comes from an immutable per-bird `voice_seed` that no code path writes after adoption (§9.3). | Variation test: 5,000 consecutive calls from one bird → 0 duplicate plan hashes. Recognizability test: cosine distance of MFCC centroids across all mood × drift combinations for one bird stays below threshold, and above threshold between any two birds. `voice_seed` column has no UPDATE path; a migration test asserts this. |
| INV-08 | Personality vector values are never exposed numerically on any rendered surface — no stats panel, no debug view, no tier, no toggle. | Raw traits are stripped in the snapshot serializer; the client receives only quantized render tiers (§6.6). The client bundle has no type that holds a raw trait. | Serializer test asserts the snapshot JSON matches no trait key. Client type-level test asserts `PersonalityVector` is unimportable from client packages. (Export exception: see A-11.) |
| INV-09 | Presence requires all three conditions simultaneously: `visibilityState === 'visible'` AND `document.hasFocus()` AND pointer-or-key activity within the activity window. | Single `PresenceDetector` with one boolean expression; no other code path emits presence (§6.4). Server re-validates window arithmetic and caps credited presence at wall-clock. | Unit matrix over all 8 condition combinations asserts presence only in the all-true case. Server test rejects overlapping/inflated windows and multi-device double-credit. |
| INV-10 | Only the server simulation tick writes personality vectors. Clients submit events, never state. | Postgres role used by the API service has no UPDATE grant on `bird.traits`; only the tick worker's role does. | Integration test attempts a trait write through the API role and asserts a permission error. Schema test asserts grant configuration. |
| INV-11 | No last-write-wins on personality. Drift is applied as additive, server-authored deltas processed in event-log order. | The tick reads events by monotonically increasing `seq` watermark and applies deltas; there is no "set trait" function in the codebase. | Concurrency test: two overlapping device sessions submitting interleaved events produce the same final vector as their serialized union. |
| INV-12 | Account identity everywhere except the single encrypted email column is a synthetic UUID. | `account.email_encrypted` is the only email column. All FKs, partition keys, cache keys, log fields, metric dimensions, and message envelopes use `account_id`. | CI scans logs and metric definitions for email-shaped strings; a structured-logging serializer redacts by default and fails the build if an email-typed value reaches a log call site. |
| INV-13 | Per-bird / per-account interaction state never enters analytics, aggregate telemetry, or any training corpus. | Physical separation: the telemetry emitter is a separate service with credentials that have no read grant on simulation tables; there is no network route from the warehouse to the simulation database (§12.7). | Metric-registry allowlist test rejects any dimension in the forbidden set (`bird_id`, `account_id`, trait names, mood, perch, offer type). Grant test asserts the analytics role cannot select from simulation tables. |
| INV-14 | Visitors cannot influence the host's aviary. Visitor presence and interaction never reach the simulation. | The visit route mounts a router with no event-ingestion endpoint. There is no code path from a visit session to the event log. | Route-table test asserts the visit router exposes exactly the read endpoints. Integration test posts every event type with a visit token and asserts 404 (route absent), then asserts the host's event log is unchanged. |
| INV-15 | Visitors see the host's aviary exactly as it is — no "show-off" rendering. | The visitor snapshot is produced by the *same* serializer function as the host snapshot; only interaction affordances are omitted client-side. | Test asserts `serializeSnapshot(state, HOST)` and `serializeSnapshot(state, VISITOR)` are byte-identical in every scene field. |
| INV-16 | Product surfaces use naturalist voice; system surfaces (identity, money, errors, settings) use matter-of-fact voice. | Two-namespace copy registry with per-surface allowlists; cross-namespace import is a build error (§10.1). | Registry lint on both namespaces' lexical rules; surface-to-namespace binding test. |
| INV-17 | Listen-in is a re-balance, never a mute or a cut. Unfocused birds stay audible. | The unfocused bus gain floor is a constant `LISTEN_IN_FLOOR_DB = -12`, and the ramp helper has no zero-target overload. | Audio graph test asserts the unfocused bus gain never reaches 0 and that both engage and disengage take ≥1.2s. |
| INV-18 | The field notebook is read-only and immutable. | No update/delete endpoints exist for notebook entries; the table has no UPDATE grant for the API role. | Route-table and grant tests. |
| INV-19 | Notebook entries observe the aviary, never the user. | Template corpus is authored with an explicit `subject: 'aviary' | 'bird' | 'weather' | 'time'` field; `'user'` is not a permitted value. | Corpus schema test plus a forbidden-lexicon scan over all generated output in a 10,000-entry fuzz run. |
| INV-20 | Perch position is a signal the birds produce, not a layout the user controls. | No API accepts a perch assignment; no client gesture maps to placement. | Route-table test; input-map test asserts drag gestures on birds are unbound. |
| INV-21 | Settle is optional. Tab-close and settle are equivalent at the engine level. | Presence windows close on the same code path for both; settle emits an additional mood impulse only. | Test asserts identical presence accounting and identical drift for a session ended by settle vs. by close. |
| INV-22 | The aviary never sends push notifications or re-engagement email. Transactional email only, always triggered by a user action. | No push subscription code, no VAPID keys, no service-worker `push` handler. Email sending goes through a template allowlist of exactly six transactional templates. | Source scan for `PushManager`/`Notification`; email service test rejects any template ID outside the allowlist. |
| INV-23 | Bird identity is stable forever. No reset, regenerate, swap, or migration replaces a bird. | `bird.id` is immutable; species-pool changes are additive-only; any migration touching `bird` requires a reviewed exemption. | Migration lint fails on `DELETE FROM bird`, `UPDATE bird SET species_id`, or any statement that reassigns `bird.id`. |
| INV-24 | The aviary cap is 7 birds; new accounts start with 2; new birds arrive on aviary age only. | `MAX_BIRDS = 7` enforced at insert; the arrival scheduler's only input is `aviary.created_at`. | Constraint test; scheduler unit test asserts no engagement input reaches the arrival decision (function signature excludes it). |
| INV-25 | The simulation tick is server-side and runs regardless of client connection. | The client bundle has no tick implementation; the client's clock only drives presentation interpolation. | Client bundle scan asserts drift/mood-transition modules are absent. Test asserts an account with no client sessions still advances mood and time-of-day. |
| INV-26 | Reduced motion is a designed render mode, not disabled animation. | `RenderMode.CrossFade` is a first-class branch in the renderer with its own pose sampler and its own visual QA checklist. | Visual regression suite runs the full scene matrix in both modes; test asserts cross-fade mode still animates (non-zero inter-frame delta) and still plays calls. |
| INV-27 | Screen-reader narration is naturalist prose on a slow cadence, never a state list. | Narration is produced by the same prose engine as the notebook, from the same templates family; there is no ARIA-label-per-state code path. | Output lint asserts every narration string is a lowercase present-tense sentence with a verb, and matches no `key: value` shape. Cadence test asserts idle interval ∈ [30s, 60s]. |
| INV-28 | Captions describe the call that actually played. | `describeCall(plan, mood)` takes the same `CallPlan` object the synthesizer consumes. Captions cannot be produced without a plan. | Test asserts caption text is a pure function of the plan and that syllable count in the caption matches the plan's. |
| INV-29 | Initial JS ≤ 2MB gzipped; time-to-first-bird ≤ 500ms at p75 on the reference profile; 60fps idle on the reference laptop; no heap growth over a 30-minute session. | CI budget gates on every PR; the memory soak is a real test, not a guideline (§14.6). | Bundle-size gate, Lighthouse/WebPageTest synthetic gate, frame-timing gate, heap-growth gate. |
| INV-30 | No third-party model or service ever receives bird, mood, trait, notebook, or interaction data. | The prose engine is fully local (§10.6). No LLM SDK is a dependency of the simulation or prose packages. | Dependency allowlist test on the simulation and prose packages; egress policy on the simulation VPC allows only the mail provider and the metrics sink. |

---

## 3. Architecture

### 3.1 Shape

```
                         ┌──────────────────────────────────────┐
   browser               │  edge (CDN + edge function)          │
 ┌──────────────┐        │  · static assets, immutable-hashed   │
 │ aviary client│◄──HTML─┤  · document assembly: inlines the    │
 │  · renderer  │        │    session's snapshot mirror         │
 │  · audio     │        └──────────┬───────────────────────────┘
 │  · presence  │                   │
 │  · a11y      │        ┌──────────▼───────────────────────────┐
 └───┬──────┬───┘        │  api service  (stateless, HTTP/JSON) │
     │      │            │  · auth / magic link                 │
   snapshot events       │  · snapshot read  (Redis → PG)       │
     │      │            │  · event ingest   (append-only)      │
     └──────┴───────────►│  · notebook / account / visits       │
                         └───┬──────────────────┬───────────────┘
                             │                  │
                    ┌────────▼───────┐   ┌──────▼──────────┐
                    │ Redis          │   │ Postgres        │
                    │ · snapshot     │   │ · canonical     │
                    │   cache/mirror │   │   aviary state  │
                    │ · tick claims  │   │ · event log     │
                    │ · rate limits  │   │ · notebook      │
                    └────────▲───────┘   │ · accounts      │
                             │           └──────▲──────────┘
                    ┌────────┴──────────────────┴───────────┐
                    │ tick worker fleet (only writer of     │
                    │ personality state)                    │
                    │ · scheduler → claim → step → persist  │
                    └───────────────────────────────────────┘

                    ┌───────────────────────────────────────┐
                    │ telemetry emitter (separate creds,    │
                    │ NO read grant on simulation tables)   │
                    └───────────────────┬───────────────────┘
                                        ▼  metrics sink / warehouse
```

### 3.2 Services

| Service | Responsibility | Scaling shape |
|---|---|---|
| `edge` | Static asset delivery, document assembly, snapshot inlining, unsupported-browser surface | CDN-native |
| `api` | Auth, snapshot read, event ingest, notebook read, account operations, visit flow | Stateless, horizontally scaled, CPU-light |
| `tick` | The simulation. Claim due aviaries, step them, persist, emit notebook candidates and snapshot cache updates | Horizontally scaled by account shard; CPU-bound |
| `mailer` | Transactional email only (six templates) | Queue-driven, low volume |
| `telemetry` | Aggregate operational metrics emission | Separate credentials; no simulation read access |
| `opsconsole` | Internal calibration/support tooling; the only place raw traits are ever displayed, behind SSO + audit log | Internal-only, not deployed to the public edge |

### 3.3 Technology choices

- **Language:** TypeScript on both sides. The call grammar, prose templates, mood constants, and the snapshot schema are in a shared `@aviary/core` package so the caption text on the client and the notebook prose on the server cannot drift apart.
- **Client rendering:** Canvas 2D with layered offscreen canvases. Not WebGL. Rationale: shader compilation and context creation cost tens of milliseconds against a 500ms first-bird budget, WebGL context loss adds a recovery path that would visibly violate INV-01, and the scene's draw-call count (≈7 birds × ≈8 parts, plus ≤12 ornaments and 4 static layers) sits comfortably inside Canvas 2D's budget on the reference laptop. Revisit only if profiling in M3 says otherwise.
- **Client chrome:** Preact + a small signal store for the top bar, notebook, settings, offer menu, and the accessibility DOM mirror. DOM (not canvas) for anything focusable, so keyboard navigation, focus rings, and screen-reader semantics are native rather than reimplemented.
- **Audio:** WebAudio with a persistent `AudioWorkletNode` per bird (§9). No `OscillatorNode`-per-call — one-shot nodes would allocate per call and fight INV-29.
- **Server:** Node 22 + TypeScript for `api` and `tick`. Sharing `@aviary/core` between the tick and the client is worth more here than raw tick throughput; the tick is arithmetic over small state and profiles comfortably (§14.8). If per-account tick cost exceeds budget at scale, the hot inner loop is a well-isolated pure function and can be ported without touching the surrounding system.
- **Storage:** Postgres 16 (canonical state, event log, notebook, accounts), Redis 7 (snapshot cache, tick claim coordination, rate limits, presence de-duplication).
- **Email:** one provider; magic-link, email-change verification, export-ready, invite, visit-notification (opt-in), and account-deletion-confirmation templates. Six, allowlisted (INV-22).

### 3.4 The authority line

This is the single most load-bearing boundary in the system, so it is stated as a rule rather than left implied:

> **The server owns state. The client owns presentation.**

The client may: interpolate between snapshots, run idle micro-motion, schedule and synthesize calls within the timing envelope the snapshot declares, generate ornament drift, compute local time-of-day for palette, and render captions from call plans it generated. The client may **not**: decide a mood, apply drift, choose which bird greets, decide a perch move, decide whether a bird accepts an offer, or persist anything the simulation reads back as truth.

The practical test for any new client feature: *if two devices ran this logic independently with the same snapshot, would they disagree about something the user would notice as state?* If yes, it belongs on the server. Ornament leaves may differ between devices; a bird's perch may not.

### 3.5 Environments

`dev` (per-engineer, seeded fixture accounts with time-scaling enabled), `staging` (full topology, synthetic cohort of ~2,000 simulated accounts running continuously from M4 onward so drift and tick cost are observed long before launch), `prod`. Time-scaling (`SimClock` multiplier) exists in dev and staging only and is compiled out of production builds.

---

## 4. Data model

### 4.1 Postgres

Types elided where obvious; all timestamps are `timestamptz`; all IDs are UUIDv7 unless noted.

```sql
-- ── identity ────────────────────────────────────────────────────────────────
account (
  id                 uuid primary key,              -- the ONLY identifier used anywhere else
  email_encrypted    bytea not null,                -- envelope-encrypted; the sole email storage
  email_hash         bytea not null unique,         -- HMAC(pepper, lower(email)) for lookup only
  email_verified_at  timestamptz,
  pending_email_encrypted bytea,                    -- set during email change, cleared on verify
  created_at         timestamptz not null,
  tz                 text not null default 'UTC',   -- IANA; last known, from the most recent presence-bearing session
  deletion_requested_at timestamptz,                -- soft-delete marker; NULL when active
  visit_notify_enabled boolean not null default false
)

session (
  id            uuid primary key,
  account_id    uuid not null references account,
  token_hash    bytea not null unique,
  device_label  text,                               -- coarse UA-derived label, e.g. "Safari on iPhone"
  created_at    timestamptz not null,
  last_seen_at  timestamptz not null,
  revoked_at    timestamptz
)

magic_link (
  id           uuid primary key,
  account_id   uuid not null references account,
  token_hash   bytea not null unique,
  created_at   timestamptz not null,
  expires_at   timestamptz not null,                -- created_at + 15 minutes
  consumed_at  timestamptz                          -- single use
)

-- ── aviary ──────────────────────────────────────────────────────────────────
aviary (
  account_id      uuid primary key references account,
  created_at      timestamptz not null,             -- drives the new-bird arrival schedule; nothing else does
  state_version   bigint not null default 0,        -- monotonic; snapshot ETag
  last_tick_at    timestamptz not null,
  next_tick_at    timestamptz not null,             -- scheduler index
  tick_class      text not null default 'hot',      -- hot | warm | cold  (§6.2)
  prng_state      bytea not null,                   -- serialized xoshiro256** state; determinism
  weather         jsonb not null default '{}',      -- {kind, started_at, ends_at, intensity}
  lighting        text not null default 'auto',     -- auto | settled   (settle gesture)
  settled_at      timestamptz,
  presence_open_window jsonb,                       -- {started_at, last_credited_at, session_id}
  last_presence_end_at timestamptz,                 -- drives absence bucket for the greeting
  notebook_tokens real not null default 3,          -- sparsity budget (§6.16)
  notebook_tokens_at timestamptz not null
)
create index on aviary (next_tick_at) where deletion_hard_at is null;

bird (
  id            uuid primary key,                   -- IMMUTABLE. never reassigned (INV-23)
  account_id    uuid not null references account,
  species_id    text not null,                      -- immutable after adoption
  name          text not null,                      -- user-assigned, renameable, no engine effect
  voice_seed    bytea not null,                     -- IMMUTABLE. timbral fingerprint (INV-07)
  adopted_at    timestamptz not null,
  slot          smallint not null,                  -- stable draw/focus ordering, 0..6

  -- personality vector: server-written only (INV-10), never rendered numerically (INV-08)
  t_boldness    double precision not null,
  t_warmth      double precision not null,
  t_vocal       double precision not null,
  t_plumage     double precision not null,
  t_curiosity   double precision not null,
  -- per-bird asymptotic ceilings, sampled at adoption; preserves individuality forever (A-06)
  c_boldness    double precision not null,
  c_warmth      double precision not null,
  c_vocal       double precision not null,
  c_plumage     double precision not null,
  c_curiosity   double precision not null,

  mood            text not null,                    -- alert|curious|content|wary|drowsy|roosting
  mood_entered_at timestamptz not null,
  mood_impulses   jsonb not null default '[]',      -- decaying [{kind, weight, at}]
  perch_zone      smallint not null,                -- 0 front, 1 middle, 2 back
  perch_slot      smallint not null,                -- position within the zone
  transition      jsonb,                            -- {from_zone,from_slot,to_zone,to_slot,start_at,dur_ms} | null
  next_call_at    timestamptz,
  offer_cooldown_until timestamptz,
  unique (account_id, slot)
);
create index on bird (account_id);

-- ── events (append-only; the only thing clients write) ──────────────────────
interaction_event (
  account_id      uuid not null,
  seq             bigserial,                        -- global order; the tick's watermark
  client_event_id text not null,                    -- ULID from the client; idempotency
  session_id      uuid not null,
  kind            text not null,                    -- presence|listen_in_start|listen_in_end|
                                                    -- offer|settle|settle_undo|session_start|rename
  payload         jsonb not null,
  client_at       timestamptz not null,             -- advisory only
  received_at     timestamptz not null,             -- authoritative
  primary key (account_id, seq),
  unique (account_id, client_event_id)
) partition by hash (account_id);

-- ── notebook ────────────────────────────────────────────────────────────────
notebook_entry (
  id           uuid primary key,
  account_id   uuid not null references account,
  written_at   timestamptz not null,
  local_date   date not null,                       -- in the account's tz at write time
  text         text not null,                       -- rendered once; immutable (INV-18)
  template_id  text not null,                       -- for corpus analytics (aggregate, no per-account dimension)
  salience     real not null
);
create index on notebook_entry (account_id, written_at desc);

-- ── visits ──────────────────────────────────────────────────────────────────
visit_invite (
  id                 uuid primary key,
  account_id         uuid not null references account,   -- host
  visitor_email_encrypted bytea not null,
  visitor_email_hash bytea not null,
  token_hash         bytea not null unique,
  created_at         timestamptz not null,
  expires_at         timestamptz not null,                -- created_at + 30 days
  revoked_at         timestamptz
);

visit_session (
  id            uuid primary key,
  invite_id     uuid not null references visit_invite,
  started_at    timestamptz not null,
  last_seen_at  timestamptz not null                      -- duration is derived, approximate
);
```

`species` and the motif libraries are static build-time data in `@aviary/core`, not a table — the pool is fixed at ~6 for v1 and versioning it in code keeps client and server literally identical.

### 4.2 Redis keyspace

| Key | Contents | TTL |
|---|---|---|
| `snap:{account_id}` | Serialized snapshot + `state_version`; written by the tick, read by `api` and the edge | 10 min |
| `tick:claim:{shard}` | Worker claim coordination | 60s |
| `pres:{account_id}:{minute}` | Multi-device presence de-duplication bitmap (§7.3) | 10 min |
| `rl:magic:{email_hash}` | Magic-link rate limit | 1 h |
| `rl:ev:{session_id}` | Event ingest rate limit | 1 min |
| `visit:{token_hash}` | Cached invite validity (host id + revoked flag) | 5 s — bounds revocation latency (§13.3) |

Redis holds no durable truth. A total Redis loss costs a cold read from Postgres and a slower first bird; it costs no state.

### 4.3 Retention and lifecycle

- `interaction_event`: retained 90 days, then dropped by partition. The tick has long since folded them into state; they exist for replay/debug of recent drift and for the calibration harness. Nothing downstream needs them beyond that, and keeping interaction history indefinitely would contradict the privacy commitment.
- `notebook_entry`: retained for the life of the account. Scrollable indefinitely, never archived or hidden (`interactions.md` is explicit).
- `visit_session`: retained for the life of the account (it is the visit log).
- Soft delete: `deletion_requested_at` set → account excluded from tick scheduling, snapshot reads return the recovery surface, invites revoked. Recovery clears the marker and resumes ticking with a catch-up (§6.2).
- Hard delete at +30 days: a job deletes every row keyed to `account_id` across all tables and purges Redis keys and the encrypted email. Verified by a post-delete assertion job that scans for orphans by `account_id` and alarms on any hit.

### 4.4 Deliberately not stored

No raw email outside `account.email_encrypted` / `pending_email_encrypted`. No IP addresses in the simulation database. No per-account rows in any analytics store. No user-agent strings beyond the coarse device label on `session`. No visit-frequency aggregate, no session count, no "days active" column — not because we would show it, but because a column that exists is a column someone can later render (INV-04).

---

## 5. API surface

### 5.1 Conventions

- JSON over HTTPS, `/v1/…`. HTTP/3 where available.
- Auth: `Authorization: Bearer <session token>` or the `aviary_session` cookie (`HttpOnly`, `Secure`, `SameSite=Lax`). Visit routes use a path token instead and are mounted on a separate router (INV-14).
- Idempotency: all writes carry a `client_event_id` (ULID); duplicates return the original result.
- Errors: `{ "error": { "code": "...", "message": "..." } }`. `message` is drawn from the **system** copy namespace only (INV-16) and is safe to display verbatim.
- Every response carries `state_version` where meaningful; the snapshot endpoint uses it as an `ETag`.

### 5.2 Auth

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/auth/link` | `{email}` → always `204`, regardless of whether the account exists (no enumeration). Rate-limited per `email_hash`: 3/15min, 10/day. |
| `POST` | `/v1/auth/consume` | `{token}` → issues a session; marks the link consumed. Expired/consumed/unknown all return the same `AUTH_LINK_INVALID`. |
| `POST` | `/v1/auth/signout` | Revokes the current session. |
| `GET` | `/v1/auth/sessions` | List of `{id, device_label, created_at, last_seen_at, current}`. |
| `DELETE` | `/v1/auth/sessions/:id` | Revoke a device session. |

Magic links expire at 15 minutes and are single-use (`accounts_sync.md`). The consume endpoint is POST, and the emailed link points at an interstitial page that POSTs on load, so email-scanner prefetches cannot silently burn links — a real failure mode with corporate mail security that would otherwise present to the user as "the link never works."

### 5.3 Snapshot

```
GET /v1/aviary/snapshot
  If-None-Match: "<state_version>"
→ 200 { snapshot }  |  304
```

Served from Redis when warm; falls back to Postgres and triggers a catch-up tick if the aviary is overdue (§6.2). Target payload ≤ 4KB gzipped, p95 server time ≤ 25ms warm.

The client re-pulls on: `visibilitychange` → visible; a render-frame gap > 5s (laptop suspend); and a keepalive every 45s while visible (`accounts_sync.md` names exactly these three triggers).

### 5.4 Events

```
POST /v1/aviary/events
  { events: [ { client_event_id, kind, at, payload } ] }   -- batched, ≤ 20 per call
→ 202 { accepted: n, state_version }
```

Kinds and payloads:

| Kind | Payload | Emitted when |
|---|---|---|
| `session_start` | `{tz, reduced_motion, captions, audio_available}` | Client boot, once per session |
| `presence` | `{window_start, window_end}` | Every 30s while present; also flushed on `pagehide` via `sendBeacon` |
| `listen_in_start` / `listen_in_end` | `{bird_id}` | Focus engage / disengage |
| `offer` | `{kind: seed|song|pool, near_bird_id?}` | Offer issued |
| `settle` / `settle_undo` | `{}` | Settle gesture; undo within 5s |
| `rename` | `{bird_id, name}` | Bird renamed |

Events are appended and acknowledged; they are never applied synchronously. The client does not wait for the simulation to reflect them (see §8.3 on optimistic presentation).

Note what is absent: there is no endpoint that sets a mood, a trait, a perch, or a bird's acceptance of an offer. That absence is INV-10 and INV-20 expressed as an API shape.

### 5.5 Notebook

```
GET /v1/notebook?before=<cursor>&limit=30 → { entries: [{id, local_date, text}], next_cursor }
```

Read-only. Keyset pagination, unbounded scroll-back. No POST, PATCH, or DELETE exists (INV-18).

### 5.6 Account

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/account` | Email (masked), tz, settings, session list, visit-notify toggle |
| `PATCH` | `/v1/account/settings` | Reduced motion, captions, keep-controls-visible, visit notifications |
| `POST` | `/v1/account/email` | Starts change; sends verification to the new address. Old address keeps working until verified. |
| `POST` | `/v1/account/export` | Enqueues export; emails a signed, single-use, 24h download link to the **verified** address |
| `POST` | `/v1/account/delete` | Sets `deletion_requested_at` |
| `POST` | `/v1/account/undelete` | Clears it — the "I changed my mind" affordance available on any signed-in page during the window |

### 5.7 Visits

Host routes (session-authenticated):

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/visits/invites` | `{email}` → creates invite, emails one-time link, 30-day expiry |
| `GET` | `/v1/visits/invites` | Outstanding invitations |
| `DELETE` | `/v1/visits/invites/:id` | Revoke; effective within the 5s cache bound |
| `GET` | `/v1/visits/log` | `[{visitor_email, first_seen_at, last_seen_at, approx_duration}]`, most recent first |

Visitor routes (separate router, token-authenticated, read-only — INV-14):

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/visit/:token` | Resolves the invite; establishes a `visit_session` |
| `GET` | `/v1/visit/:token/snapshot` | Same serializer as the host's (INV-15); `403 VISIT_UNAVAILABLE` when revoked/expired |

The visitor router registers exactly these two routes. There is no event endpoint to filter, disable, or accidentally re-enable.

### 5.8 Snapshot payload

```jsonc
{
  "state_version": 90412,
  "server_time": "2026-07-26T14:03:12.480Z",
  "aviary": {
    "lighting": "auto",                  // "auto" | "settled"
    "settled_at": null,
    "weather": { "kind": "rain", "started_at": "...", "ends_at": "...", "intensity": 0.28 },
    "tz": "America/Chicago"
  },
  "birds": [
    {
      "id": "b_01J...",
      "name": "pip",
      "species": "warbler",
      "slot": 0,
      "mood": "curious",                 // enum only — never a numeric affect value
      "perch": { "zone": 0, "slot": 1 },
      "transition": { "from": {"zone":1,"slot":0}, "to": {"zone":0,"slot":1},
                      "start_at": "2026-07-26T14:03:11.100Z", "dur_ms": 1400 },
      "render": {                        // QUANTIZED. no raw traits ever (INV-08)
        "plumage_tier": 3,               // 0..5
        "detail_tier": 2,                // 0..3
        "idle_energy": "low",            // low | mid | high
        "approach_bias": "forward"       // back | neutral | forward
      },
      "voice": {
        "seed": "9f2c…",                 // immutable timbral fingerprint
        "tempo_scale": 1.08,
        "repeat_bias": 0.42,
        "next_call_at": "2026-07-26T14:03:19.900Z"
      }
    }
  ],
  "greeting": {                          // present only on the first snapshot of a session
    "bird_id": "b_01J...",
    "form": "approach_and_call",
    "absence_bucket": "day",
    "start_at": "2026-07-26T14:03:13.000Z",
    "stagger": [ { "bird_id": "b_02K...", "delay_ms": 2400, "form": "glance" } ]
  },
  "narration_seed": 448192               // so narration is stable across devices in a session
}
```

Three things about this payload are load-bearing.

**It carries phase, not just position.** `transition.start_at` may be in the past. A client joining mid-flight renders the bird partway through the arc, which is what makes INV-01 true rather than aspirational.

**It carries no trait numbers.** The `render` block is quantized. The client cannot leak what it does not have, and no future "debug overlay" can be built from this payload.

**Host and visitor get the same object.** INV-15 is a property of there being one serializer.

### 5.9 Error taxonomy

All messages below are **system** namespace, matter-of-fact voice, and are the exact strings shipped.

| Code | HTTP | Message |
|---|---|---|
| `AUTH_LINK_INVALID` | 401 | We couldn't sign you in. The link may have expired. Try requesting a new link. |
| `AUTH_SESSION_EXPIRED` | 401 | Your session timed out. Sign in again to keep watching. |
| `SNAPSHOT_UNAVAILABLE` | 503 | Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch. |
| `VISIT_UNAVAILABLE` | 403 | This visit is no longer available. |
| `RATE_LIMITED` | 429 | Too many requests. Wait a moment and try again. |
| `BROWSER_UNSUPPORTED` | 200 (page) | Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge. |
| `ACCOUNT_PENDING_DELETION` | 200 (page) | This account is scheduled for deletion. You can restore it until <date>. |

No naturalist phrasing appears in any of these. A user blocked at sign-in needs system clarity, and warmth in that position reads as evasion.

---

## 6. Simulation engine

### 6.1 Tick architecture

The tick is a pure step function plus a persistence shell:

```ts
type Emission =
  | { kind: 'notebook_candidate'; templateId: string; slots: Slots; salience: number }
  | { kind: 'metric'; name: string; value: number }        // aggregate only

function step(
  state: AviaryState,          // birds, weather, lighting, prng
  dt_ms: number,               // nominally 60_000
  events: InteractionEvent[],  // in seq order, since the last watermark
  wall: WallClock,             // absolute time + the account's IANA tz
): { next: AviaryState; emissions: Emission[] }
```

`step` is deterministic: every random draw comes from `state.prng`, which is advanced and persisted. Given the same `(state, dt, events, wall)`, two workers produce byte-identical output. This is what makes replay-based debugging, the calibration harness, and catch-up (§6.2) all possible, and it is worth the discipline of never calling `Math.random()` in the simulation package (a lint rule enforces it).

Scheduling:

```sql
UPDATE aviary SET next_tick_at = now() + interval '60 seconds'
WHERE account_id IN (
  SELECT account_id FROM aviary
  WHERE next_tick_at <= now() AND deletion_requested_at IS NULL
  ORDER BY next_tick_at
  FOR UPDATE SKIP LOCKED
  LIMIT 200
) RETURNING account_id;
```

Workers claim batches of 200, process, and write back in a single transaction per account. `SKIP LOCKED` gives horizontal scaling with no external coordinator. Backlog (count of aviaries with `next_tick_at < now() - 120s`) is the primary saturation signal (§14.8).

Ordering within a tick, per aviary:

1. Fold new events since the watermark → presence credit, mood impulses, offer records, listen-in windows, settle/undo, tz update.
2. Advance wall-clock-driven inputs: local time-of-day curve, weather sampling and expiry, dawn reset if due.
3. Apply drift (§6.5) — the only place traits are written.
4. Transition moods (§6.7), including bird-to-bird contagion (§6.8).
5. Resolve perch intentions into transitions (§6.10).
6. Schedule calls (§6.11).
7. Evaluate the new-bird arrival schedule (§6.15).
8. Emit notebook candidates (§6.16).
9. Bump `state_version`, persist, write the snapshot cache.

### 6.2 Catch-up and dormancy

Ticking every account every 60 seconds forever is wasteful for accounts nobody is watching, but the PRD is explicit that the aviary continues without the viewer, so "just stop ticking" is not available either. The resolution rests on one observation:

> **During an absence there are no drift inputs.** Drift is driven by presence, listen-in, and offers — all of which are user-generated. An aviary with no client sessions receives zero drift signal, so the personality vector is provably unchanged across the absence. Only mood, time-of-day, weather, calls, and perch state advance.

That makes long absences analytically collapsible without changing what the user sees.

| Class | Condition | Cadence | Catch-up on read |
|---|---|---|---|
| `hot` | Presence in the last 6h, or a live session | 60s | none needed |
| `warm` | Activity in the last 30 days | 60s | none needed |
| `cold` | No activity for 30+ days | 6h coarse tick | full replay on snapshot request |

Catch-up on a `cold` aviary's snapshot request replays real 60s steps up to a cap of 1,440 steps (24h). Beyond that, it invokes `collapseAbsence(state, from, to)`, which advances time-of-day and weather analytically, moves each bird's mood along the no-input stationary distribution, and asserts (in test) that the trait vector is unchanged. The user's first snapshot after a long absence therefore costs a bounded ~40ms of catch-up, not a 60,000-step replay.

The correctness requirement is stated as a test, not a hope: `collapseAbsence` and a full replay must produce the same trait vector exactly, and mood distributions within a Kolmogorov–Smirnov tolerance over 10,000 seeds.

Cold-account promotion happens on `session_start`.

### 6.3 Determinism

`prng_state` is a serialized xoshiro256** state persisted with the aviary. Sub-streams are derived by splitting on a domain tag (`'mood'`, `'call'`, `'weather'`, `'greeting'`) so adding a new random consumer does not perturb existing sequences — otherwise every calibration golden file breaks on every feature addition, and the team quietly stops trusting them.

### 6.4 Presence accounting

Client detector (INV-09), one expression, one owner:

```ts
const ACTIVITY_WINDOW_MS = 240_000;   // 4 minutes; server-delivered config (A-01)

present =
  document.visibilityState === 'visible' &&
  document.hasFocus() &&
  (now - lastPointerOrKeyAt) <= ACTIVITY_WINDOW_MS;
```

`lastPointerOrKeyAt` updates on `pointermove` (throttled to 1Hz), `pointerdown`, `keydown`, `wheel`, and `touchstart`. Focus is tracked via `focus`/`blur` plus a poll of `document.hasFocus()` on the presence interval, because focus events are unreliable across iframe and OS-level window-manager transitions.

Four minutes is chosen at the long end of "a few minutes" because `interactions.md` is explicit that watching birds without moving is the actual product — presence should be lost when the user has stopped showing any sign of being there, not the moment the mouse stops. A-01 records this as tunable via server config so we can move it without a client release.

The client emits a `presence` event every 30s with the `[window_start, window_end]` interval it observed, and flushes a final partial window on `pagehide` with `navigator.sendBeacon`.

Server-side credit rules — each one closes a way the signal could inflate:

1. **Clamp to receipt.** Credited time is intersected with `[received_at - 90s, received_at]`. A client cannot claim an hour in one ping.
2. **Never exceed wall clock.** Total credit for an aviary across a wall-clock minute is capped at 60s regardless of how many devices report (§7.3).
3. **Cap the day.** Credited presence is capped at 4h/day (A-02). Beyond that the signal is almost certainly not a person watching birds, and the cap bounds the drift contribution of outliers.
4. **Presence windows close on both settle and tab-close identically** (INV-21).

### 6.5 Drift

Drift is a saturating low-pass integrator, per trait, per bird:

```
signal_τ  = Σ over the tick window of weighted inputs mapped to trait τ
Δτ        = max(0, k_τ · signal_τ · (ceiling_τ − t_τ))
t_τ'      = min(ceiling_τ, t_τ + Δτ)
```

The `max(0, …)` and the repository-level `new >= old` assertion are INV-05. There is no branch in this function that can produce a negative delta; neglect contributes `signal = 0`, which contributes `Δ = 0`.

Input weights (per `bird_engine.md`'s stated ordering — presence dominant, then listen-in, then offers, settle neutral):

| Input | Unit | boldness | warmth | vocal | plumage | curiosity |
|---|---|---|---|---|---|---|
| Presence (aviary-wide) | per credited minute | 0.6 | 0.8 | 0.6 | 1.0 | 0.6 |
| Listen-in (that bird) | per focused minute, capped 12 min/session | 0.5 | 2.5 | 2.5 | 0.8 | 0.6 |
| Offer accepted (that bird) | per event | 0.4 | 0.5 | 0.2 | 0.2 | 1.6 |
| Offer issued (birds in the near zone) | per event | 0.9 | 0.2 | 0.1 | 0.1 | 0.4 |
| Settle | per event | 0 | 0 | 0 | 0 | 0 |

Settle contributes a mood impulse (§6.7) and nothing to drift — `bird_engine.md` says it does not push drift in any direction beyond closing the presence window cleanly.

Rate constant: `k = 6.0e-4` per unit-signal-minute, uniform across traits (per-trait shaping lives in the weight table so there is one knob to calibrate, not five).

Calibration derivation, with "regular visits" defined as 15 minutes/day, 5 days/week ⇒ **75 presence-minutes/week** (A-03):

- Seed traits ≈ 0.35, ceiling ≈ 0.85. Headroom ≈ 0.50.
- Week 1, plumage (weight 1.0): `75 × 1.0 × 6.0e-4 × 0.50 ≈ 0.0225`. The instrument threshold is 0.010, so this is comfortably **measurable in instruments after ~1 week**. ✔
- Week 3 cumulative, with the shrinking-headroom term: **≈ 0.062**. The render-tier width is 0.055 (§6.6), so the first tier boundary is crossed in week 3 — **visible to the user after ~3 weeks**. ✔
- A single 15-minute session moves plumage by ≈ 0.0045 — an order of magnitude below a tier boundary, so **no single session is ever visible**. ✔

These constants are the starting point, not the answer. The calibration harness (§15.3) refits `k` against the acceptance band before launch and re-checks it against the live cohort at M6.

Per-bird ceilings are sampled at adoption from `U(0.72, 0.98)` per trait (A-06). Without them, every long-lived bird converges to the same maximum and the aviary loses its individuality in year two — a slow failure that no launch test would catch.

### 6.6 Quantization: the instruments-vs-user gap

`bird_engine.md` asks for drift that instruments can measure in a week but users only notice after three. That gap is not achievable with rate tuning alone — a continuously-rendered trait is either visible or it isn't. The mechanism is **quantized rendering of continuous traits**:

| Trait | Rendered as | Quantization |
|---|---|---|
| Plumage saturation | `plumage_tier` 0–5 and `detail_tier` 0–3 | Tier width 0.055, with 0.008 hysteresis so a bird sitting on a boundary does not flicker between tiers across snapshots |
| Boldness | `approach_bias` ∈ {back, neutral, forward}, feeding the perch-choice distribution | Thresholds at 0.42 / 0.62 |
| Social warmth | Greeter-selection weight and call-response probability | Continuous internally, but its *observable* — who greets first — is discrete by nature |
| Vocal frequency | `tempo_scale`, `repeat_bias`, and the call hazard rate | Continuous, but the audible step is coarse relative to the drift rate |
| Curiosity | Offer-approach probability and head-tilt frequency | Banded at 0.40 / 0.65 |

Traits change every day; what the user *sees* changes when a boundary is crossed. That is exactly the felt experience the PRD describes: the user does not notice the current, they notice where the bird used to perch.

### 6.7 Mood

States: `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting` (A-04).

Mood is a continuous-time Markov chain evaluated per tick. Each ordered pair `(from, to)` has a base rate `λ₀`, multiplied by context factors:

```
λ(from→to) = λ₀(from,to)
           · f_time(to, local_hour)          // drowsy↑ near dusk; alert↑ early morning
           · f_personality(to, traits)        // high boldness suppresses wary entry; high curiosity raises curious
           · f_impulse(to, active_impulses)   // decaying interaction effects
           · f_weather(to, weather)           // rain: vocal damping + drowsy↑; wind: alert↑ / wary↑ split by boldness
           · f_neighbors(to, nearby_moods)    // contagion (§6.8)
           · f_inertia(time_in_current_mood)  // suppresses churn in the first ~4 minutes
```

Transition is sampled with `p = 1 − exp(−λ·dt)` per target, resolved by the mood PRNG sub-stream.

Impulses are `{kind, weight, at}` with exponential decay (half-life 6 min for offer reactions, 12 min for listen-in, 20 min for settle). Examples: offer accepted → `content +0.9`; song-fragment offer → `curious +0.7`, `alert +0.3`; settle → `drowsy +1.1` aviary-wide, plus a lighting change; return-greeting → `alert +0.5` on the greeter.

`roosting` is entered only when local time is past the species' roost hour and the aviary is dim, and it is suppressed entirely for the nightjar-like species, which keeps calling into the late hours (`aviary_layout.md`: night is not a dead state).

**Dawn reset.** `concepts.md` says mood resets on a daily-ish cadence; `bird_engine.md` says mood must never snap to a default on tab open. Both hold if the reset acts on the *priors*, not on the value. At local `05:30 ± 40min`, jittered per bird, the tick clears decayed impulses, re-samples the day's baseline mood weights from personality and season, and resets mood inertia — but it does not assign a mood. The bird transitions out of `roosting` through the chain, at its own rate, shaped by its own personality. A bird that ended yesterday wary softens toward content overnight if the time-of-day signal pushes that way; it is never *set* to content.

Mood persists across sessions because it lives in the canonical record and is only ever advanced by the tick (INV-25). There is no session-start mood initialization code path.

### 6.8 Bird-to-bird interaction

Adjacency is derived from perch geometry: birds in the same zone are adjacent; adjacent zones are half-weighted; front-to-back is unweighted.

- **Wary contagion.** A bird entering `wary` raises `λ(*→wary)` for neighbours for 90 seconds, scaled by adjacency weight and by `(1 − boldness_neighbour)`. A bold bird beside a spooked one mostly does not catch it, which is exactly the individuality the trait is supposed to produce.
- **Call response.** When a bird calls, each other bird gets a temporary call-hazard boost over a 0.8–3.5s response window, weighted by its `social_warmth` and dampened if it is `roosting` or `drowsy`.
- **Chorus.** There is no chorus object and no chorus scheduler. A chorus is what happens when two or more high-vocal-frequency birds' hazard rates overlap and the response boost cascades. Emergence is the design: a scheduled "chorus event" would be a cue, and cues are what the product is refusing.

### 6.9 Weather

Sampled by the tick as an inhomogeneous Poisson process per aviary (A-05):

| Kind | Rate | Duration | Intensity | Effect |
|---|---|---|---|---|
| Rain | 2.5 / week | 4–11 min | ≤ 0.40 | Vocal hazard × 0.55 aviary-wide during and for ~8 min after; `drowsy` ↑ slightly |
| Wind | 4 / week | 6–20 min | ≤ 0.30 | Leaf-motion amplitude ↑; `alert` ↑ for bold birds, `wary` ↑ for shy birds |

Intensity is capped in the type, not by convention. There is no thunderstorm, no snow, no weather the user must notice. Two strong events never overlap; a new sample is rejected if one is active.

### 6.10 Perch selection and motion

Each tick, a bird evaluates a desire to move with a low base rate modulated by mood and by the mismatch between its current zone and its preferred zone. Preferred zone is a distribution over `{front, middle, back}` from `approach_bias` (boldness) and mood: `wary` pushes back, `curious` and `content` pull forward, `drowsy` prefers wherever it already is.

When a move fires, the tick writes a `transition` with `start_at` and `dur_ms` (hop 400–700ms within a zone, short flight 900–1,600ms between zones) and updates the target perch. The client renders the arc. Because the snapshot carries `start_at`, a client that joins mid-flight renders the bird mid-flight (INV-01).

The user cannot move a bird (INV-20). There is no drag handler, no perch API, no "come here" gesture.

### 6.11 Call scheduling

Each bird carries `next_call_at`, sampled from an exponential distribution whose hazard rate is:

```
rate = base(species)
     · g_vocal(t_vocal)          // the vocal-frequency trait, principal driver
     · g_mood(mood)              // roosting ≈ 0 (nightjar excepted), drowsy 0.3, alert 1.4
     · g_time(local_hour)        // dawn chorus bump, midday lull, dusk taper
     · g_weather(weather)        // rain damping
     · g_response(recent_calls)  // the response-window boost from §6.8
     · g_presence(user_present)  // modest lift when the user is present (A-07)
```

The server schedules *when*; the client synthesizes *what* (§9.2). The split matters: `next_call_at` must be canonical so two devices hear the same bird call at the same moment, but the waveform is generated locally so it costs no bandwidth and can vary per render.

A-07 is a small, deliberate call: birds call a little more when someone is present. `bird_engine.md` describes vocal frequency as governing how often a bird calls "when unobserved," implying observation matters. The lift is capped at ×1.25 so it reads as the aviary being livelier when watched, not as the birds performing for the user.

### 6.12 Greeting selection

On `session_start`, the tick (or an inline fast path on the first snapshot, using the same function) computes:

1. **Absence bucket** from `aviary.last_presence_end_at`: `brief` < 30 min, `short` < 12h, `day` < 48h, `long` < 7d, `extended` ≥ 7d.
2. **Greeter** = argmax over birds of `1.15·boldness + 0.85·warmth + moodBonus(mood) + noise`, where `moodBonus` favours `alert`/`curious`, penalizes `drowsy`, and excludes `roosting` birds unless all birds are roosting (in which case the greeting is a small stir, not a call). `noise` is a seeded draw, so the same bird does not greet every single time while boldness still shows through across many sessions.
3. **Form** sampled from a bucket-conditioned distribution:

| Bucket | glance | two-note call | step forward | approach + call | long call + response |
|---|---|---|---|---|---|
| brief | 0.70 | 0.25 | 0.05 | — | — |
| short | 0.35 | 0.35 | 0.20 | 0.10 | — |
| day | 0.10 | 0.25 | 0.25 | 0.30 | 0.10 |
| long | 0.05 | 0.15 | 0.20 | 0.40 | 0.20 |
| extended | — | 0.10 | 0.15 | 0.40 | 0.35 |

4. **Stagger.** Additional birds may greet, each with an independent probability from `social_warmth`, each offset by a randomized 1.2–4.5s delay. Never in unison — a simultaneous chorus on cue announces the user's arrival, which is the wrong register (`interactions.md` says so directly).
5. **Procedural variation inside the form.** The motion parameters and call plan are generated from a fresh seed per greeting. This is the point `interactions.md` warns about hardest: three pre-recorded variants in rotation is a canned cue with extra steps. The variation test in INV-07 covers greetings specifically.

There is no textual accompaniment to any of this (INV-03). The greeting is the welcome.

### 6.13 Offers

Offers are issued from the top bar, never by clicking a bird (`interactions.md`). The offer lands in the scene; birds react.

Reaction is resolved by the tick per bird:

```
p(approach) = base(offer_kind)
            · h_curiosity(t_curiosity)
            · h_mood(mood)          // content 1.2, curious 1.5, alert 1.0, wary 0.35, drowsy 0.15, roosting 0
            · h_distance(zone)
```

- **Seed** — a curious, content bird approaches; a wary bird waits and often comes near after a delay (sampled 20–90s, so the waiting is legible); a drowsy bird frequently does not come at all.
- **Song fragment** — a soft melodic motif plays in the aviary. The bird's response — joining, going quiet, calling against it — is drawn from `vocal_frequency` and mood.
- **Still pool** — a soft reflective surface at the front of the scene for ~3 minutes. Birds drink, bathe, or watch, by mood and curiosity.

**Cooldown: 4 minutes per bird** (A-08). The cooldown is functional. Without it, curiosity drift saturates inside one session and the engine collapses; with it, an offer reads as a gesture. When a bird is in cooldown the offer still lands in the aviary and other birds may respond — the cooldown is per-bird, not per-aviary, and the offer affordance is never disabled or greyed out, because a greyed-out button with a countdown would turn a gesture into a cooldown timer, which is a game mechanic wearing a naturalist coat.

### 6.14 Settle

Settle sets `lighting = 'settled'` and `settled_at`, emits a `drowsy` impulse aviary-wide, and closes the presence window. The client cross-fades to the evening palette over 4.5s and lowers the ambient mix.

Undo: any click anywhere in the aviary within 5 seconds emits `settle_undo`; the client reverses the lighting ramp immediately (optimistically — this is presentation, so the client may lead), and the server clears the state. This is a mercy for accidental clicks, and it is deliberately not surfaced as a button, an "undo" toast, or a countdown.

The aviary stays settled until the tab closes or the user actively re-engages (a click, a keypress, an offer, a listen-in). At the engine level settle and tab-close are the same event (INV-21): the presence window ends, nothing is penalized, no recovery surface exists, and no notification is sent.

### 6.15 New-bird arrival

Availability is a function of `aviary.created_at` and nothing else (INV-24). The function's signature does not accept an engagement parameter, which makes the rule structural rather than remembered.

Thresholds (A-09): 3rd bird at 90 days, 4th at 180, 5th at 300, 6th at 450, 7th at 600.

**How the offer is surfaced is a design decision this plan makes explicitly**, because `bird_engine.md` calls it "an offer that appears in the user's flow" while the brief forbids announcement surfaces. Resolution: *the new bird simply shows up.*

When a threshold is reached, an unnamed bird arrives at the back perch on the user's next session. It is tentative — rarely comes forward, calls infrequently — and it is present for 7 days. If the user notices and focuses it, a small naming affordance appears (naturalist voice: `a new bird. what will you call it?`); naming completes adoption and the bird becomes permanent with a freshly seeded personality vector. If the user never notices, the bird drifts off at day 7 and returns after 21 days.

This is "notice, never announce" applied to the one feature that most invites a modal. The user's noticing *is* the accept. If usability testing in M5 shows a majority of users never notice a new arrival across two 7-day windows, the fallback is a single extremely quiet top-bar affordance — a small perched-bird glyph, no badge, no count, no dot — and never a toast, modal, or banner.

### 6.16 Notebook entry generation

The tick emits candidate observations with a salience score. Candidates are generated from state comparisons the tick already computes: greeting order changed, a bird crossed a plumage tier, a long quiet stretch, first rain in a while, a chorus involving three or more birds, a bird using the front perch for the first time in a fortnight, a bird ignoring an offer it usually takes.

**Sparsity** is a token bucket on the aviary: capacity 3, refill 1 per 40 hours. Writing an entry costs 1 token, and the salience threshold rises as the bucket empties (`threshold = 0.35 + 0.25 × (3 − tokens)`). The steady state is roughly one entry every 2–3 days for a regularly-visited aviary, with genuinely noteworthy days able to spend down the bucket. A very active user does not get more entries — that is the sparsity requirement in `interactions.md`, and the bucket makes it structural rather than a tuning parameter someone later relaxes.

Entries are rendered once at write time and stored as text (immutable, INV-18). Subject is always the aviary, a bird, the weather, or the time — never the user (INV-19).

---

## 7. Sync model

### 7.1 The single-writer rule

One canonical aviary record per account. The tick worker is the only writer of personality state, enforced by Postgres grants rather than by convention (INV-10). Clients write append-only events; the tick folds them in `seq` order and applies additive deltas (INV-11).

The failure this design forecloses is worth spelling out because it is silent when it happens: with last-write-wins, a phone session that read the vector at noon and wrote it back at 12:30 would erase drift a laptop session recorded at 12:15. The user never sees an error — they just have a bird that drifts more slowly than it should, and no log line says data was lost. Additive server-authored deltas processed in log order make that state unreachable rather than unlikely.

### 7.2 Snapshot delivery and versioning

`state_version` increments on every tick that changes anything. Clients send `If-None-Match`; unchanged states cost a 304. Two devices reading the same version render the same aviary.

Clients pull on the three triggers `accounts_sync.md` names: visibility change to visible, a render-frame gap over 5s, and a 45s keepalive while visible. There is no WebSocket in v1 — a 45s pull of a 4KB payload is cheaper to operate and to reason about than a persistent connection, and the simulation's 60s tick means a socket would deliver nothing sooner. (Revisit only if a future feature needs sub-tick latency.)

### 7.3 Multi-device presence de-duplication

If a user has the aviary open on a laptop and a phone, both may satisfy the presence conditions. Crediting both would double the drift signal for one person sitting in one chair.

Credit is therefore taken as a **union over wall-clock seconds**, not a sum. `pres:{account_id}:{minute}` holds a 60-bit bitmap; each presence window sets the seconds it covers; the tick credits `popcount` and clears. Two devices reporting the same minute credit 60 seconds, not 120.

### 7.4 Failure modes and conflict surfaces

| Situation | Behaviour | User-visible surface |
|---|---|---|
| Magic-link replay (already consumed) | Reject | `AUTH_LINK_INVALID` |
| Session revoked mid-use | Next request 401 | `AUTH_SESSION_EXPIRED` |
| Snapshot read fails | Client retains the last snapshot and keeps rendering; retries with backoff | Nothing for the first 10s — the aviary keeps going, which is honest, since the server-side aviary is fine. After 10s of failures, `SNAPSHOT_UNAVAILABLE` as a small matter-of-fact line in the field |
| Event POST fails | Queue in memory (cap 200 events), retry with backoff, flush on `pagehide` | Nothing. Losing a presence ping is not worth telling the user about |
| Clock skew | Client uses `server_time` from the snapshot to compute an offset; all scheduling uses the corrected clock | None |
| Tick backlog | Snapshot serves the last good state; catch-up runs on read | None unless prolonged, then `SNAPSHOT_UNAVAILABLE` |

### 7.5 Not built

No client-to-client sync. No CRDTs. No offline write queue that survives a reload (an unsent presence window is not worth durable storage). No client-side simulation of any kind (INV-25).

---

## 8. Frontend rendering pipeline

### 8.1 Layer stack

| Layer | Content | Update cadence | Technique |
|---|---|---|---|
| L0 sky | Day/night gradient | ~1Hz | Two pre-rendered key-time gradients cross-faded; a full gradient re-bake at most every 60s |
| L1 far background | Soft foliage, horizon | On resize + parallax | Offscreen canvas, blitted with a small translate |
| L2 mid | Perches, mid foliage | On resize | Offscreen canvas |
| L3 birds | Bird rigs, still pool, offer items | 60fps | Direct draw, dirty-rect where profitable |
| L4 foreground | Drifting leaves/feathers, occasional near branch | 60fps | Pooled sprites |
| DOM chrome | Top bar, sheets, captions, focus targets, live regions | Event-driven | Preact |

Parallax is a translate of ≤ 6px on L1 and ≤ 14px on L4 against pointer position, heavily damped. The scene is not a layered illustration trying to show off (`aviary_layout.md`).

Everything focusable is DOM. Nothing focusable is canvas. This single rule is what makes §11 tractable.

### 8.2 Bird rig and procedural pose

Each bird is a small 2D skeletal rig: `body, head, beak, eye, wing_far, wing_near, tail, leg_l, leg_r`. Species geometry ships as `Path2D` built from compact path strings at module init (no SVG parsing at runtime, no image decode).

Pose is composed from three layers, summed:

1. **Breath** — continuous, low-amplitude value noise on body scale, head bob, and tail angle. Amplitude and frequency scale with `idle_energy`. This runs always. A bird is never still in a way that reads as paused (`bird_engine.md`).
2. **Action** — a small state machine: `perched, preen, scan, tilt, shuffle, hop, flight, drink, bathe, roost`. Selection weights are mood-shaped: `wary` → scan-heavy and further back; `content` → preen; `curious` → tilt toward sounds and track passing leaves; `drowsy` → low posture, fluffed silhouette, slow blink. The user reads mood from motion; there is no label, tooltip, or status icon anywhere (INV-03 territory, and `bird_engine.md` says failing this fails a primary affective contract).
3. **Directive** — server-declared transitions (flight arcs, greeting forms, offer approaches), which override action selection for their duration.

Plumage rendering: base species palette modulated by `plumage_tier` (saturation and value curve) with `detail_tier` gating a feather-detail overlay pass. Tier changes cross-fade over 8 seconds so a snapshot boundary never produces a visible pop.

### 8.3 Interpolation and mid-action join

The client keeps `snapshotPrev` and `snapshotNext` and a corrected clock. Discrete fields (mood, perch target) take effect at the new snapshot's `state_version`; continuous presentation interpolates.

For transitions, the client computes `phase = (now − start_at) / dur_ms`, clamps to `[0,1]`, and renders the arc at that phase — including when `phase` starts above zero because the page just loaded. A bird at zone 1 in snapshot N and zone 0 in snapshot N+1 flies; it never teleports.

**Optimistic presentation** is permitted for pure presentation and nothing else: the settle lighting ramp starts on click (§6.14), listen-in mix starts ramping on focus, and the offer item appears in the scene immediately. Bird *reactions* to an offer wait for the server, because reaction is state. If the server later disagrees with an optimistic presentation, the client reconciles by animating, never by snapping.

### 8.4 First-frame path

Target: first bird painted ≤ 500ms at p75 on the reference profile (mid-tier Android, 4G) — A-10 records the interpretation of the PRD's `<500ms` as a p75 target with a p95 guardrail of 900ms.

1. **Edge document assembly.** An edge function resolves the session cookie to `account_id`, reads `snap:{account_id}` from a regional replica of the snapshot cache, and inlines it into the HTML as a JSON script tag. Miss → the document ships without it and the client fetches in parallel.
2. **Inline critical module** (~26KB gzipped, budgeted): canvas bootstrap, species path data, pose evaluator, sky gradient, snapshot decoder. Nothing else is on the critical path.
3. **First paint** draws the sky (time-of-day computed locally — no server needed for the palette) plus every bird at its current pose and transition phase.
4. **Deferred** (in priority order, after first paint): full animation system, audio worklet + grammar, DOM chrome, accessibility mirror, notebook, settings, offer menu, visit flow.
5. **No web font on the critical path.** Top bar uses a system stack; if a display face is used at all it loads with `font-display: optional`.
6. **Repeat visits.** A service worker precaches hashed static assets (never snapshots), and the last snapshot is mirrored to `localStorage`. A repeat visit paints from cache in ~120ms.

**Stale-cache reconciliation.** A `localStorage` snapshot older than the current state is used only for composition — which birds exist, roughly where they were, palette from the local clock. When the live snapshot lands, differences are rendered as ordinary motion: a bird hops or flies to where it actually is. Since the aviary carries no labels or counters, there is nothing that can be *wrong* on screen in the interim — only slightly out of date, resolved by the aviary doing something. Beyond 6 hours of staleness, the cache contributes species and count only, and birds are placed by mood-agnostic priors. The cache is cleared on sign-out.

Budget breakdown at p75 on the reference profile:

| Step | Budget |
|---|---|
| DNS + TCP + TLS (cold, HTTP/3, edge PoP) | 120ms |
| Edge document TTFB (incl. snapshot mirror read) | 60ms |
| HTML + inline module transfer (~40KB gz) | 70ms |
| Parse + execute critical module | 90ms |
| First bird painted | 50ms |
| **Total** | **390ms** (110ms headroom) |

### 8.5 The quiet field

When no snapshot is available yet, the client renders `QuietField`: the sky gradient for the local hour, the far-background silhouette, and one or two faint ornament cues (a slow leaf). No spinner, no progress bar, no percentage, no shimmer, no skeleton (INV-02). The field is the aviary catching up, not the product loading.

This same surface is the empty-aviary state during adoption. After naming, the first bird enters with a soft fly-in from off-frame to its starting perch, the second follows a few seconds later. From that point the user never sees an empty aviary again — and note that the fly-in exists *only* here, at genuine first arrival. It is not a load animation and must never be reused as one.

### 8.6 Day/night and weather rendering

Day/night follows the user's local clock via `Intl.DateTimeFormat().resolvedOptions().timeZone`, sent to the server on `session_start` for the tick's time-of-day inputs and for notebook date words. Sunrise warms the palette gradually across the early hours; midday is brightest; evening warms and dims; night dims most of the scene while leaving the nightjar-like species active.

DST and travel are handled by recomputing from the IANA zone every render rather than caching an offset (A-12). Multi-device timezone conflicts resolve to the tz of the most recent presence-bearing session.

Rain renders as a subtle vertical streak field at `intensity ≤ 0.4` plus a slight desaturation and a damped ambient mix. Wind renders as increased leaf-drift amplitude and a slow foliage sway. Both are short-lived. There is no weather UI, no icon, no forecast.

### 8.7 Ambient ornaments

Leaves and feathers are pure client-side rendering ornaments with no simulation state (`aviary_layout.md` is explicit). Spawned from a pool of 16 at a Poisson rate of ~1 per 7s, with wind raising the rate and amplitude. They are the visual signal that the aviary continues between bird actions. They are pooled and never allocated per spawn (INV-29).

### 8.8 Top bar

Four items: account/settings, accessibility settings, field notebook, offer. Nothing else. No badges, no counts, no dots, no indicators of any kind.

Fade: after 3.5s of cursor stillness, opacity ramps to 0.06 over 900ms (opacity only — no transform, no layout change). Returns to full on `pointermove`, any keydown, or focus entering the bar. Keyboard focus pins it open — a control that fades while focused is a keyboard trap in slow motion. The accessibility setting **"keep controls visible"** disables the fade entirely, for low-vision users for whom near-transparent chrome is a barrier.

No UI chrome exists inside the aviary scene itself: no buttons, no badges, no hover tooltips, no overlay icons, no inline labels (`aviary_layout.md`). Bird names are not drawn in the scene. The scene is birds and place.

### 8.9 Reduced-motion render mode

Triggered by `prefers-reduced-motion: reduce` or the accessibility setting. This is `RenderMode.CrossFade`, a first-class branch, not a disabled-animation flag (INV-26).

| Element | Default mode | Reduced-motion mode |
|---|---|---|
| Idle micro-motion | Continuous noise-driven pose | Discrete pose keys, 900ms cross-fade, 3–6s hold |
| Preen / scan / tilt | Animated | Cross-fade between 2–4 authored poses of the same action |
| Flight between perches | Animated arc | 1.2s cross-fade between the two perch positions, no path |
| Ambient leaves | Present | Removed |
| Parallax | Present | Removed |
| Day/night shift | Continuous | Continuous, 2× slower |
| Weather | Particles + tint | Tint and mix change only |
| Top-bar fade | 900ms | 300ms, opacity only |
| Calls | Full | Full — unchanged |
| Drift, mood, notebook | Unchanged | Unchanged |

The cross-fade register is its own quiet aesthetic, and it gets its own visual design review and its own screenshot suite. A user who set `prefers-reduced-motion` for vestibular reasons should get a Pocket Aviary that is calmer, not one that looks broken.

### 8.10 Responsive layout

One horizontal scene, always fully visible. No panning, scrolling, or zooming (`aviary_layout.md`).

Layout is computed from viewport width against a 16:9 design frame. Narrow viewports compress perch spacing horizontally and shrink bird scale to a floor of 0.72× before compressing further; wide viewports widen spacing to a ceiling and letterbox the sky rather than stretching. **A bird is never cropped and never drifts offscreen** — bird positions are computed in normalized scene space and clamped to a safe inset, and a layout test asserts every bird's bounding box lies inside the viewport across a matrix of 14 viewport sizes from 320×568 to 2560×1440.

High-DPI: render at `min(devicePixelRatio, 2)`. Above 2× the cost is real and the visible gain on this art style is not.

### 8.11 Hidden-tab behaviour

When the tab is hidden, the client cancels its `requestAnimationFrame` loop, suspends the `AudioContext`, and stops presence reporting. It keeps nothing running. The simulation continues server-side (`interactions.md`), so on return the client pulls a fresh snapshot and renders the aviary that has been running — not the one it left.

---

## 9. Audio pipeline

### 9.1 Graph

```
per bird (max 7):
  BirdVoiceWorklet ──► GainNode(birdGain) ──┬──► BiquadFilter(lowpass, unfocused bus) ──┐
                                            └──► (focused path, no filter) ─────────────┤
                                                                                        ▼
  ambientBed (worklet: wind/leaf noise, −34 dB) ─────────────────────────────► MixBus ──► FDN reverb (worklet)
                                                                                        │
                                                                              masterGain ──► DynamicsCompressor ──► destination
```

Node count is fixed at boot: 7 voice worklets + 1 ambient + 1 reverb + a handful of gains and filters. Nothing is created per call (INV-29).

### 9.2 Call grammar and synthesis

A **motif** is a sequence of **syllables**. A syllable is:

```ts
type Syllable = {
  shape: 'tone' | 'sweep' | 'trill' | 'chip' | 'buzz';
  f0_hz: number; f1_hz: number;      // start / end pitch (equal for 'tone')
  dur_ms: number; gap_ms: number;
  amp: number; attack_ms: number; decay_ms: number;
  vibrato_hz: number; vibrato_cents: number;
  harmonics: number[];               // relative amplitudes of partials 2..5
  noise_mix: number;                 // breathiness
};
```

Each species ships 3–6 motif templates. The grammar is a weighted production system:

```
call     → phrase (gap phrase){0..repeat_bias·3}
phrase   → motif | motif tail | intro motif
motif    → syllable{2..5}                      -- drawn from the species library
```

Generation applies, in order: (1) the bird's **immutable voice constraints** (§9.3); (2) mood shaping — `alert` shortens gaps and raises pitch ~1.5 semitones, `drowsy` lengthens and lowers, `wary` shortens phrases and raises the chip ratio, `curious` favours rising sweeps; (3) drift shaping — `tempo_scale` and `repeat_bias`; (4) fresh per-call jitter on every numeric field (±3% timing, ±25 cents pitch, ±8% amplitude) from a per-call PRNG draw.

The result is a `CallPlan`, which is posted to the bird's worklet and passed to `describeCall()` for the caption. One object, two consumers — which is how INV-28 holds by construction.

The worklet renders the plan sample-by-sample: a small bank of phase accumulators for the partials, a state-variable filter for formant shaping, a noise generator for `noise_mix`, and an envelope follower. No `OscillatorNode`, no `decodeAudioData`, no buffers to allocate.

### 9.3 Voice identity — the recognizability contract

`bird_engine.md` requires that a user who has spent two weeks with Pip knows Pip's call by ear across mood changes and drift. That is a strong constraint and it needs a mechanism, not good intentions.

Each bird's `voice_seed` (immutable, generated at adoption, never updated — INV-07) deterministically derives a **timbral fingerprint**:

- base pitch offset (±4 semitones from species centre)
- harmonic amplitude profile (the partial ratios — the dominant contributor to perceived timbre)
- formant filter centre and Q
- vibrato rate and depth
- syllable-shape ordering bias (which motif this bird favours opening with)
- attack sharpness

**These parameters never change.** Mood and drift modulate only *tempo, repetition count, amplitude, gap length, and small pitch offsets* — the prosody, not the voice. Prosody varies, timbre does not, which is exactly how humans recognize a familiar voice saying something new.

Verification is a real test, not a listening session: compute MFCC centroids for 200 calls per bird across the full mood × drift-tier matrix. Assert intra-bird centroid distance stays below a threshold and inter-bird distance above it, for all 21 pairs of 7 birds and all species pairs. This test is what makes the seven-bird cap defensible — if a future species pool breaks the separation, the test fails before the cap does.

### 9.4 Chorus

All voices mix live into a shared bus. Because every call is generated fresh, two birds calling together produce genuine interference and beating — a real chorus. Two recorded loops layered produce a characteristic phase-cancellation artifact the ear catches even when each individual call sounds procedural (`bird_engine.md`), which is one of the two reasons the no-recorded-audio rule is unconditional.

The master compressor is gentle (ratio 2:1, −18 dBFS threshold, 30ms attack, 250ms release) — enough to keep a 4-bird chorus from clipping, not enough to pump.

### 9.5 Listen-in mix

Engage: focused bird's gain → 0 dB and the unfocused bus → **−12 dB** (never 0 — INV-17), each via `setTargetAtTime` with a 0.45s time constant, reaching effective target in ~1.4s. A 2.5kHz lowpass fades in on the unfocused bus to push those birds perceptually back. Disengage uses the identical ramp in reverse.

It must feel like listening, not like switching channels. There is no hard cut anywhere in the path, and there is no code path that sets a bird's gain to zero for listen-in purposes. Other birds drop in the mix but never go silent — silencing them would teach the user the aviary is a set of tracks to switch between rather than a place where several things happen at once.

Disengage triggers, per `interactions.md`: clicking the focused bird again, focusing a different bird, clicking empty space in the aviary, or moving keyboard focus away.

### 9.6 Scheduling

`next_call_at` is server-canonical (§6.11). The client converts it through the server-clock offset and schedules the worklet with WebAudio's sample clock, ~250ms of lookahead. Calls whose scheduled time has already passed by more than 3s at snapshot load are dropped, not fired late — a burst of stale calls on tab return would be the audible version of a load state.

### 9.7 Fallback

If `AudioContext` construction fails, is blocked by autoplay policy until a gesture, or the worklet module fails to load, the aviary plays in graceful silence and **captions turn on by default**. There is no recorded-audio fallback path in the codebase (INV-06). Silence with captions is a better fallback than canned audio.

Autoplay policy is handled without a "click to enable sound" modal (that would be a load-state announcement). The aviary starts silent-with-captions; the first user gesture anywhere resumes the context, captions revert to the user's setting, and the transition is a 1.5s fade-up, not a pop.

### 9.8 Memory discipline

Fixed node graph. `CallPlan` objects come from a pool of 32 and are recycled. Worklet-side scratch buffers are allocated once at construction. The `postMessage` payload is a flat `Float32Array` copied into a preallocated ring buffer rather than a fresh object graph per call. Target: zero net allocation in steady-state audio.

### 9.9 Anti-canned tests

- 5,000 consecutive calls from one bird → 0 duplicate plan hashes (INV-07).
- No exact repeat of the greeting motion-parameter tuple within 200 consecutive greetings.
- Timbre separation matrix (§9.3).
- Automated listening artifact: nightly, render 60s of 4-bird chorus to a WAV in CI and attach it to the build. Not asserted mechanically, but reviewed weekly by the audio owner. Audio uncanniness is not detectable by unit test, and pretending otherwise is how it ships.

---

## 10. Voice, copy, and generated prose

### 10.1 Two registers, mechanically enforced

Every user-visible string lives in exactly one namespace.

**`naturalist.*`** — aviary, notebook, narration, captions, offer prompts, bird naming. Lint rules: lowercase first character; no `!`; no second-person pronouns (`you`, `your`, `you're`); present tense (checked against a verb list); no gamification lexicon (`streak`, `badge`, `level`, `score`, `achievement`, `unlocked`, `milestone`, `days in a row`, `welcome back`, `congratulations`, `progress`).

**`system.*`** — sign-in, account settings, accessibility settings, sync and error surfaces, unsupported-browser, deletion, export, invite management. Lint rules: normal sentence capitalization; no naturalist lexicon (`perch`, `flutter`, `settles`, `nestles`) in an error or identity context; states what happened and what to do.

Each UI surface declares its namespace. Importing a string from the other namespace is a build error. When a future surface lands in between — a billing flow, say — the rule from `product_brief.md` decides it: any surface where the user is engaging with the system *as a system* (money, identity, errors, settings) drops out of the naturalist register.

### 10.2 The prose engine

One generator serves the notebook, the narration, and the captions, so the three cannot drift into three different products (`accessibility_perf.md` is explicit that a screen-reader user moving between surfaces should hear the same product).

```ts
type Template = {
  id: string;
  subject: 'aviary' | 'bird' | 'weather' | 'time';   // 'user' is not a value (INV-19)
  surface: ('notebook' | 'narration' | 'caption')[];
  text: string;                                       // "{bird} is on the {perch} this morning, {posture}."
  slots: Record<string, SlotSpec>;
  conditions: StateCondition[];
};
```

Slots are filled from a curated lexicon with per-slot synonym sets and a **recency penalty**: the generator tracks the last 40 slot fills per account and down-weights recent choices, so "fluffed against the cool air" does not appear three days running. Selection is seeded by `(account_id, entry_id)` so an entry, once written, is stable.

The locale seam exists (templates are data, not string literals in code) but only `en` ships.

### 10.3 Notebook prose

Authored by a writer, not a developer, and reviewed as prose. Launch corpus: ~120 templates across ~25 observation types, with an authoring target of at least four distinct phrasings per type.

Examples of the register (`interactions.md`):

> tuesday — pip greeted before wren today, first time this week.

> wren is fluffed against the cool air, watching the back perch. low calls only.

> a long stretch of quiet this morning. pip preened for several minutes without looking up.

What the corpus must never contain — and what the linter rejects at build time — is the event-log register: `session started at 7:43`, `a bird greeted you`, `Pip's vocal frequency changed by 0.03`, or anything whose subject is the user's behaviour. A user who opens the notebook and finds a log line has been told, in one entry, that the rest of the product's voice is performance.

**Charm decay** is a real risk with template-generated prose over months. Mitigations: the recency penalty above; a corpus-coverage metric (aggregate, no per-account dimension) showing template usage distribution so we can see flattening; and a standing commitment to add ~20 templates per quarter post-launch, treated as content work rather than as a bug fix.

### 10.4 Narration

Rendered into a visually-hidden `aria-live="polite"` region. Never `assertive` — assertive interrupts, and interrupting is announcing (INV-03's spirit, INV-27's letter).

Two regions: `#narration-ambient` (idle) and `#narration-event` (user-initiated). When an event narration fires, pending ambient text is cleared so the queue does not back up behind it. User-initiated events — a return-greeting, a successful offer, a settle — get the priority bump `accessibility_perf.md` describes, and are still written as observations rather than as state transitions.

Cadence: idle updates every 35–55s (jittered). Faster only for the event region. High-frequency narration overwhelms the screen-reader queue and forces the user to silence it, which is the system pushing the accessibility surface aside.

Register, from the PRD:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Not `Pip is at perch 2` and not `Wren mood: content`. A planner who treats this as ARIA-label automation has built the wrong feature; INV-27's shape lint (no `key: value` forms, every string a sentence with a verb) is the mechanical version of that warning.

Narration is generated client-side from the same snapshot the visual surface reads, using the shared engine, with `narration_seed` from the snapshot so two devices in the same session narrate consistently.

### 10.5 Captions

`describeCall(plan, mood)` maps a `CallPlan` to prose, deterministically:

> a soft three-note rise

> a low trill, paused, low trill again

> a single sharp call from the back perch

Syllable count, contour direction, and dynamic come from the plan; the adjective comes from mood and amplitude. Because the caption is derived from the same object the synthesizer consumed, the caption always describes what actually played (INV-28) — a stored per-call string could not make that promise.

Captions render as small DOM text near the calling bird, fading in with the call and out over ~1.2s after it, positioned with collision avoidance so two simultaneous calls do not overlap. WCAG AA contrast against both the brightest and dimmest aviary states, achieved with a soft backing scrim rather than an outline. Opt-in from accessibility settings; on by default when audio is unavailable.

### 10.6 No third-party model in the prose path

Tempting and forbidden. `accounts_sync.md` commits that per-bird interaction data is never shared with any third party and never used for training. Calling a hosted model to write notebook entries would send bird state, mood, and interaction history to a third party on every entry — a direct violation dressed as a quality improvement.

All prose is generated locally by the template engine (INV-30). Enforced by a dependency allowlist on the `@aviary/prose` and `@aviary/sim` packages and by an egress policy on the simulation VPC that permits only the mail provider and the metrics sink.

---

## 11. Accessibility

Built in the same milestones as the visual surfaces, not after (`accessibility_perf.md`: a reduced-motion mode landing as a v1.1 fix is a v1 launch that told reduced-motion users the product wasn't for them).

### 11.1 Semantic mirror and focus model

The canvas is `role="img"` with a short static description. Over it sits a transparent DOM layer of one `<button>` per bird, absolutely positioned and updated to follow bird positions (transform-only, updated at 15Hz — sufficient for hit-testing and focus-ring placement without costing frame budget).

Real buttons mean native focus, native `Enter` activation, native screen-reader semantics, and DOM-rendered focus rings. Each button's accessible name is naturalist and current: `pip, on the front perch, calling softly`.

### 11.2 Keyboard map

| Key | Context | Action |
|---|---|---|
| `Tab` | Anywhere | Top bar items → aviary scene → sheets, in DOM order |
| `Tab` | Entering the scene | Focuses the first bird (roving tabindex thereafter) |
| `←` / `→` | Scene | Move focus between birds in slot order |
| `↑` / `↓` | Scene | Move focus between perch zones |
| `Enter` / `Space` | Bird focused | Toggle listen-in |
| `Escape` | Listen-in active | Exit listen-in |
| `Escape` | Sheet open | Close sheet, return focus to its trigger |
| `N` | Anywhere outside a text field | Open the field notebook |
| `O` | Anywhere outside a text field | Open the offer menu |
| `S` | Anywhere outside a text field | Settle |

Sheets trap focus while open and restore it on close. Keyboard focus pins the top bar visible (§8.8).

### 11.3 Focus indicators

A dual-tone ring — 2px dark inner, 2px light outer halo — so it reads against both the brightest midday sky and the dimmest night. Rendered in DOM (`outline` plus `box-shadow`), never drawn into the canvas, so it cannot be lost to a repaint. Verified by an automated contrast check against sampled aviary background colours at eight times of day.

### 11.4 Contrast

All user copy — top bar, settings, account surfaces, error surfaces, captions, visually-displayed narration — passes WCAG AA (4.5:1 body, 3:1 large). Automated axe-core checks in CI across both render modes and eight times of day. The scene itself carries no copy, so the constraint applies chiefly to chrome, as `aviary_layout.md` notes.

### 11.5 Accessibility settings surface

Matter-of-fact voice (INV-16). Options: reduced motion (`auto` / `on` / `off`, defaulting to the media query); call captions (`on` / `off` / `auto-when-silent`); keep controls visible; narration detail (`normal` / `brief`).

### 11.6 QA

Automated: axe-core on every surface in both render modes; keyboard-only traversal test reaching every interactive element and returning; live-region cadence test; focus-ring contrast test; `prefers-reduced-motion` visual regression suite.

Manual, required before launch and each quarter after: NVDA/Firefox and VoiceOver/Safari full-session walkthroughs including adoption, listen-in, offer, settle, and notebook; a keyboard-only session with no mouse attached; a session with audio disabled and captions on; and a paid session with at least two screen-reader users from outside the team. The test is not "can they operate it" — it is whether the aviary feels alive to them. That is the stated bar, and it is the only one worth measuring against.

---

## 12. Accounts, auth, and privacy

### 12.1 Magic link

Email → 15-minute, single-use link. Consumption is a POST from an interstitial, so mail-scanner prefetches cannot burn links. Rate limits: 3 per 15 minutes and 10 per day per `email_hash`, with a matter-of-fact `RATE_LIMITED` message. Sign-in requests always return 204 regardless of account existence.

### 12.2 Sessions

Per-device tokens, `HttpOnly; Secure; SameSite=Lax`, 90-day sliding expiry. Listed in account settings with a coarse device label and last-seen time; revocable individually. Revocation is immediate: the session row is marked and the API checks it per request (cached 5s).

### 12.3 Email change

`POST /v1/account/email` stores `pending_email_encrypted` and sends verification to the new address. The old address keeps working until the new one verifies; on verification, the swap is committed atomically and a notice goes to the old address. Unverified pending changes expire after 7 days.

### 12.4 Export

On demand, generated asynchronously, emailed as a signed single-use link valid 24 hours to the **verified** address. Contents: birds (id, name, species, adoption date, current personality vector, current mood), notebook entries, account settings, visit log. JSON, documented schema.

The export includes personality vectors because `accounts_sync.md` explicitly lists them. That is in tension with INV-08, and A-11 records how the tension resolves: the export is a portability artifact delivered as a file, not a product surface. No in-product view renders export contents, and no endpoint returns the vectors to a browser session. The rule stays "the user never *sees* the numbers in the product," which is what the numbers-are-hidden rule is protecting.

### 12.5 Deletion

`deletion_requested_at` set immediately; the account is excluded from tick scheduling and all invites are revoked. Any signed-in page during the 30-day window shows the restore affordance ("I changed my mind"), which clears the marker and resumes ticking with a catch-up. At +30 days a job hard-deletes every row keyed to `account_id` across all tables, purges Redis, and drops the encrypted email. A verification job then scans for orphans by `account_id` and alarms on any hit.

### 12.6 The synthetic-ID rule

`account.id` (UUID) is the identifier in every table, every cache key, every log field, every metric dimension, every queue partition key, every inter-service message, and every error message. Email lives in exactly two encrypted columns and is used for exactly two things: sending mail, and lookup via `email_hash` at sign-in.

`accounts_sync.md` calls this the single most important boring detail in the PRD, and it is right: an engineer reaching for email as a convenient unique key sprays PII across observability tooling nobody can fully audit, and it is impossible to retrofit. Enforcement (INV-12): the structured logger redacts by default and fails the build if an email-typed value reaches a log call site; a CI scan greps logs and metric definitions for email-shaped strings; the `Email` type is a branded type that the logger and metric emitter refuse to accept.

### 12.7 Privacy boundary architecture

The commitment is that per-bird interaction events exist only to drive that user's own simulation — never aggregated, never used for training, never shared, never used for population-level analysis. That reads as a privacy claim and behaves as an architectural rule, so it is built as one:

1. The analytics warehouse has **no credentials** for the simulation database and no network route to it. Not a filtered view — no route.
2. The telemetry emitter is a separate service whose DB role can read nothing in the simulation schema. It emits from request-scoped operational data only.
3. The metric registry is an allowlist. New metric names and dimensions require a PR against the registry, and the CI check rejects `bird_id`, `account_id`, trait names, mood values, perch, offer kind, and notebook template IDs at per-account granularity.
4. Even anodyne aggregates over bird state — "average drift across all accounts" on an internal dashboard — are out. That is the exact framing `accounts_sync.md` rules out, and the only reason to build the pipeline would be to later expose it. The calibration harness (§15.3) runs on **synthetic cohorts in staging**, not on production user data, which is what makes this refusal affordable rather than merely principled.
5. Support access to a specific account's raw state requires an audited `opsconsole` action with a stated reason.

### 12.8 Email policy

Exactly six transactional templates: magic link, email-change verification, export ready, visit invite, visit notification (opt-in only), deletion confirmation. Every one is triggered by a user action within the last few minutes. There is no lifecycle email, no digest, no re-engagement, no "your birds miss you," ever (INV-22). Enforced by a sending allowlist keyed by template ID.

---

## 13. Visits

### 13.1 Flow

Host enters a visitor's email in account settings → the system creates an invite and emails a one-time link → the visitor opens it and sees the host's aviary as a read-only ambient view. Visits are off for new accounts, per-invite opt-in, and never prompted during onboarding.

### 13.2 Read-only enforcement

Structural, not conditional (INV-14). The visitor router exposes exactly two GET routes. There is no event-ingestion endpoint mounted under the visit path, so there is nothing to filter, disable, or later re-enable by accident. A visitor's presence and interactions never reach the simulation — the host's drift comes from the host's presence only, which protects the host's relationship with their birds from being reshaped by attention they did not sign up for.

The visitor client boots in `viewer_role: 'visitor'`, which omits the offer affordance, the settle gesture, the notebook, and account settings, and disables listen-in and the presence detector. But that omission is a UI convenience — the security property lives in the absent routes.

### 13.3 Revocation and expiry

Invites expire 30 days after creation if unused, and cannot be revived — the host issues a new one. Revocation from account settings takes effect within the 5-second `visit:{token_hash}` cache bound; the visitor's next snapshot pull returns `403 VISIT_UNAVAILABLE` and the client shows the matter-of-fact surface. Revoking an unused invite silently stops the link working; the visitor sees the same surface. The host gets no confirmation that revocation succeeded — the visitor's absence from the visit log is its own confirmation.

### 13.4 Visit log

In account settings, on demand: visitor email, date, approximate duration, most recent first, plus outstanding invitations. Approximate duration comes from `visit_session.last_seen_at − started_at`, bucketed to the nearest 5 minutes because precision here would be surveillance rather than transparency.

No badge appears on the settings icon when a new visit happens. No indicator anywhere. The log is just there if the host wants to look, and the reason it exists at all is transparency about what has been visible to whom.

By default the host gets no push, no email, no in-product notice when a friend visits. A per-account toggle, off by default and not surfaced in onboarding, enables an email — for the host who specifically wants to know when a friend stops by.

### 13.5 Refusals

No chat. No avatars or visitor markers of any kind. No comments on the aviary or on birds. No public discovery, directory, or explore surface. No leaderboards — and, per `social_optional.md`, **we do not compute the underlying statistics** that a leaderboard would need, which is why the refusal compounds: no cross-account metrics means no pipeline that could later "just be exposed." No co-presence, no shared cursor, no "your friend is here" overlay. No show-off rendering (INV-15).

---

## 14. Performance budgets and observability

### 14.1 Budgets

Reference profiles: **mobile** — mid-tier Android (≈ Pixel 6a class), 4G (9 Mbps down, 170ms RTT), cold cache. **laptop** — 5-year-old mid-range (dual-core i5-1035G1 class, integrated graphics).

| Budget | Target | Gate |
|---|---|---|
| Initial JS, gzipped | ≤ 2.0 MB (working ceiling **1.2 MB**) | CI, hard fail |
| Critical inline module | ≤ 30 KB gzipped | CI, hard fail |
| Time to first bird (mobile, cold) | p75 ≤ 500ms, p95 ≤ 900ms | Synthetic gate + RUM alarm |
| Time to first bird (repeat, warm SW cache) | p75 ≤ 200ms | RUM |
| Idle frame rate (laptop, 4 birds) | ≥ 60fps sustained; ≤ 0.5% frames > 20ms over 30 min | CI frame-timing gate |
| Idle frame rate (laptop, 7 birds + weather) | ≥ 55fps | CI |
| Heap growth over 30 min | ≤ 2 MB retained; 0 detached nodes | CI soak, hard fail |
| Audio glitch rate | 0 worklet underruns per 30 min on the laptop profile | CI soak |
| Snapshot payload | ≤ 4 KB gzipped p95 | CI |
| Snapshot API server time | p95 ≤ 25ms warm, ≤ 120ms cold | SLO |
| Tick latency per aviary | p50 ≤ 8ms, p99 ≤ 5s (alarm) | SLO, paging alarm |

The working ceiling of 1.2MB against a 2MB cap is deliberate: budgets consumed to their limit on day one have nowhere to go, and the 2MB number in `accessibility_perf.md` is the point past which the first-bird budget becomes unrecoverable, not a target to grow into.

### 14.2 Bundle plan

| Chunk | Est. gz | Loading |
|---|---|---|
| Critical inline (canvas, species paths, pose eval, sky, snapshot decode) | 26 KB | Inlined in HTML |
| Animation system + ornaments + weather | 85 KB | Deferred, high priority |
| Audio (worklet, grammar, motif library, reverb) | 70 KB | Deferred, after first paint |
| Chrome (Preact, top bar, sheets) | 95 KB | Deferred |
| A11y (mirror, narration, prose engine, templates) | 110 KB | Deferred, high priority |
| Notebook | 40 KB | On open |
| Settings + account | 55 KB | On open |
| Offer menu | 25 KB | On open |
| Visit flow | 20 KB | On the visit route only |
| **Initial (through a11y)** | **≈ 386 KB** | |

Comfortably inside budget, which is the point: procedural audio and procedurally-generated bird art are what buy that headroom, and both are required for other reasons anyway.

### 14.3 Runtime discipline

Object pools for ornaments, call plans, and pose buffers. No allocation in the render loop — enforced by an allocation-tracking test in dev builds that fails on any GC-visible allocation across 600 frames. Offscreen canvases for static layers, re-baked only on resize or day-phase change. `transform`-only DOM updates for the bird-button mirror. Audio nodes fixed at boot.

### 14.4 The memory test

`accessibility_perf.md` says the no-memory-growth rule is a real test in CI, not a guideline. Two tests:

1. **Time-scaled CI soak** (every PR): the app supports an injected `SimClock`; in test mode the render loop, audio scheduler, and snapshot cadence run at 20×. 95 seconds of wall clock ≈ 30 minutes of session. Heap snapshots at 0/25/50/75/100%; assert retained-size growth ≤ 2MB and detached-node count 0.
2. **Real-time nightly soak** (device lab): a genuine 35-minute session on the reference laptop and a mid-tier Android, with full audio and a 7-bird aviary, capturing heap, frame timing, and audio underruns.

Both, because time-scaling can hide leaks driven by real elapsed time and real-time-only would be too slow to gate every PR.

### 14.5 CI gates

Bundle size (per chunk and total) · critical-module size · Lighthouse mobile-4G first-bird timing · frame-timing soak · memory soak · allocation-in-loop check · axe-core across surfaces and both render modes · keyboard traversal · copy-registry lint · forbidden-lexicon scan over generated prose (10k-entry fuzz) · call-variation and timbre-separation tests · drift monotonicity property test · snapshot-schema no-trait-keys test · route-table tests · DB grant tests · metric-registry allowlist · dependency allowlist · migration lint.

Every invariant in §2 maps to at least one of these. An invariant with no gate is an intention.

### 14.6 RUM and synthetic

RUM, aggregate-only, no per-account dimension: navigation timing, first-bird-render, frame-timing histograms, audio-context init failures, worklet underrun counts, snapshot fetch latency and error rate, JS error rate. Sampled at 10%.

Synthetic: a fleet of automated browsers running full sessions every 15 minutes from four geographies on both device profiles, exercising cold load, return-greeting, listen-in, offer, settle, and notebook, and asserting first-bird timing.

### 14.7 Server SLOs and alarms

| Signal | SLO | Alarm |
|---|---|---|
| Snapshot availability | 99.9% | 5-min error rate > 1% |
| Snapshot latency | p95 ≤ 25ms warm | p95 > 60ms for 10 min |
| Tick latency | p99 < 5s | **p99 > 5s → page** (the PRD names this threshold explicitly) |
| Tick backlog | 0 aviaries > 120s overdue | > 1,000 overdue for 5 min |
| Event ingest | 99.9% accepted | error rate > 0.5% |
| Magic-link delivery | p95 < 30s to provider-accepted | p95 > 120s |
| Hard-delete job | Runs daily, zero orphans | Any orphan found → page |

Tick-latency p99 is the aviary's health in one number: the tick is supposed to take milliseconds, and a 5s p99 catches degradation well before users experience the aviary "running slow."

### 14.8 Forbidden metrics

Named explicitly, checked mechanically (INV-13): no metric may carry `account_id`, `bird_id`, `email`, a trait name, a mood value, a perch zone, an offer kind, or a notebook template ID at per-account granularity. No metric may count visits per account, sessions per account, or days active. Session-duration histograms are anonymized and carry no per-account dimension.

Aggregate template-usage counts (for charm-decay monitoring, §10.3) are permitted because they carry no account dimension and describe the corpus, not any user.

---

## 15. Testing strategy

### 15.1 Layers

| Layer | Coverage |
|---|---|
| Unit | Drift, mood chain, presence arithmetic, call grammar, prose generation, snapshot serialization |
| Property | Drift monotonicity, presence never exceeding wall clock, trait ceiling respect, snapshot schema conformance |
| Golden / replay | Fixed seed + fixed event stream → byte-identical state after N ticks. Regenerating a golden requires an explicit flag and a reviewer |
| Integration | Full API surface, auth flows, visit flows, grant enforcement, route-table assertions |
| Client | Renderer under a stubbed clock, interpolation, mid-flight join, reduced-motion mode, audio graph shape |
| E2E | Playwright: adoption, return-greeting, listen-in, offer, settle+undo, notebook, settings, export, invite, revoke, delete, restore |
| Perf | §14.5 gates |
| Accessibility | Automated (§11.6) + scheduled manual sessions |
| Long-horizon | §15.4 |

### 15.2 Invariant tests

Every row in §2 has a test file named for its ID (`inv-05-drift-monotonic.test.ts`). They live in a dedicated suite that runs on every PR and cannot be skipped by tag. A PR that touches an invariant test requires a second reviewer.

### 15.3 Calibration harness

A deterministic simulator that runs synthetic user cohorts through the real `step()` function at accelerated time.

Cohorts: `daily-15min`, `daily-45min`, `weekend-only`, `sporadic` (2×/week), `heavy` (2h/day), `lapsed` (3 weeks on, 4 weeks off, return), `listen-in-focused`, `offer-heavy`.

Acceptance band, run at every change to drift constants or weights:

| Assertion | Band |
|---|---|
| `daily-15min` at 7 days: max trait delta | ≥ 0.010 and ≤ 0.045 |
| `daily-15min` at 21 days: at least one trait crosses a render tier | true |
| `daily-15min` at 21 days: no more than 2 traits cross a tier | true (drift should be felt, not dramatic) |
| Single 15-min session: max trait delta | ≤ 0.010 (well under a tier) |
| `lapsed` during the 4-week absence: any trait change | exactly 0 (INV-05) |
| `heavy` at 90 days: no trait pegged at its ceiling | true (A-06 working) |
| `sporadic` at 90 days: visible drift present | true (the product still works for casual users) |

Post-launch, the harness re-runs against the staging synthetic cohort continuously, and M6 compares the model's predictions against a consented live cohort using **aggregate drift-rate distributions only** — no per-account inspection (§12.7).

### 15.4 Affective QA

Some of what this product must get right cannot be asserted. A weekly 30-minute review, owned rotating across the team, with a fixed checklist:

- Does the first frame look like the aviary was already running? (Watch the recording at 0.25×.)
- Did any call sound repeated?
- Did any surface announce anything?
- Did any notebook entry read like a log line?
- Does listen-in feel like listening or like switching channels?
- Does the reduced-motion mode have its own charm, or does it look broken?
- Did anything anywhere show the user a number about themselves?

Findings are filed as bugs against the relevant invariant. This is the only mechanism that catches the failures the PRD warns about most — the ones the user will not name but will feel.

### 15.5 Long-horizon simulation

A permanent staging cohort of ~2,000 synthetic accounts running from M4 through launch and beyond, at 1× real time, exercising the real tick fleet. It surfaces what short tests cannot: trait saturation at 6 and 12 months, notebook template exhaustion, tick cost per account under realistic distribution, cold-account catch-up correctness, and multi-year drift shape. Two years of simulated aviary time before we have a single one in production.

---

## 16. Rollout

### 16.1 Milestones

| # | Milestone | Content | Exit criteria |
|---|---|---|---|
| M0 | Foundations (2w) | Repo, `@aviary/core`, CI skeleton with budget gates, copy registry + lints, DB schema + grants, edge/CDN topology | All §2 gates exist and fail correctly against seeded violations |
| M1 | Simulation spine (3w) | `step()`, tick scheduler, presence accounting, drift, mood chain, determinism, golden tests, calibration harness | Calibration acceptance band met on synthetic cohorts; INV-05/09/10/11 tests green |
| M2 | State plane (2w) | Snapshot API, event ingest, Redis cache, catch-up/dormancy, auth + magic link, sessions | Two devices show identical state; concurrency test green; multi-device presence de-dup verified |
| M3 | Scene (4w) | Renderer, bird rigs, pose system, interpolation, mid-flight join, first-frame path, quiet field, day/night, weather, ornaments, top bar, responsive | First bird ≤ 500ms p75 on the reference profile; 60fps at 7 birds; INV-01/02 filmstrip tests green |
| M4 | Voice of the aviary (3w) | Audio worklet, call grammar, voice identity, chorus, listen-in mix, fallback; prose engine + notebook corpus v1 | Timbre-separation matrix passes at 7 birds; variation tests green; audio memory soak clean; long-horizon cohort live |
| M5 | Interactions & accessibility (4w) | Return-greeting, offer, settle+undo, notebook UI, semantic mirror, keyboard, narration, captions, reduced-motion mode, settings | Full a11y CI green; two external screen-reader sessions completed; reduced-motion design review signed off |
| M6 | Accounts, visits, calibration (3w) | Export, deletion lifecycle, email change, visit flow end-to-end, privacy boundary verification, live calibration cohort | Grant/route/metric tests green; hard-delete verification job clean; drift band confirmed against the 21-day cohort |
| M7 | Hardening (3w) | Perf tuning, soaks, synthetic monitoring, affective QA sweep, unsupported-browser surface, load test at 10× projected launch volume | All budgets met; zero open invariant violations; affective checklist clean two weeks running |
| M8 | Launch | Staged ramp (§16.3) | — |

≈ 24 weeks. Sizing assumes 2 client engineers, 2 backend, 1 design-engineer (rendering/audio), 1 visual designer (part-time, owns palette, reduced-motion register, focus treatments), 1 writer (part-time, owns the prose corpus and the copy registry). The writer is not optional: the notebook and narration corpora are the product's charm engine, and a developer-authored corpus is how the voice becomes generic.

### 16.2 Launch ramp

Closed alpha (~50 accounts, internal + friendly) at M6 → private beta (~1,000, invite-only) at M7, running at least four weeks so the three-week drift horizon is actually observed by real users before public launch → public launch with waitlist-gated signup ramping 500/day for the first two weeks, watching tick backlog and cost-per-account.

The beta must run past three weeks. Launching before anyone has experienced visible drift means launching without having tested the central promise.

### 16.3 Bird-count ramp

The aviary itself ramps on age (§6.15): 2 at signup, 3rd at 90 days, then 180 / 300 / 450 / 600. No account reaches 7 birds until it is ~20 months old, so the seven-bird audio load arrives long after launch. That is fortunate but not a reason to defer testing it — the timbre-separation matrix and the 7-bird perf soak run from M4, and the long-horizon cohort exercises full aviaries continuously.

Thresholds are server config, changeable without a release. If the recognizability ceiling turns out lower than seven in real listening, we lower the cap; the engine supports any value ≤ 7.

### 16.4 Day-one instrumentation

Operational and aggregate only (INV-13): first-bird timing, frame timing, audio init failures and underruns, snapshot latency and error rate, tick latency and backlog, magic-link delivery, event ingest rate, error rates by surface, anonymized session-duration histogram, corpus template-usage distribution.

Deliberately not instrumented: retention cohorts, DAU/MAU, session counts per user, visits per account, offers per session, days-active. Not because they would be hard, but because the moment they exist someone will optimize against them, and the optimizations that improve those numbers are precisely the features `non_goals.md` refuses. Refusing to build the metric is how the refusal stays durable.

We will know if the product works by whether people keep coming back and what they say, not by a graph that rewards adding a streak counter.

### 16.5 Configuration and kill switches

Server-delivered config, changeable without a release: presence activity window, tick cadence, drift rate constant `k`, weather rates, notebook token refill rate, new-bird thresholds, listen-in ramp constants, snapshot keepalive interval. Kill switches: weather off, new-bird arrivals paused, visits disabled, RUM sampling to zero, catch-up replay cap.

No kill switch disables accessibility surfaces, and no config value can set the drift rate negative — the type is a positive-only branded number.

---

## 17. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | Drift mis-calibrated — too fast reads as Tamagotchi, too slow as screensaver | High | High | §15.3 harness with an acceptance band; quantized rendering (§6.6) decouples instrument sensitivity from user visibility; `k` is server config; four-week beta before launch |
| R2 | Procedural calls sound synthetic or uncanny | Medium | High | Timbre fingerprint + prosody separation (§9.3); weekly listening review with nightly rendered artifacts; audio owner staffed from M0, not M4 |
| R3 | Per-bird recognizability fails at 7 birds | Medium | High | MFCC separation matrix from M4; cap is config; if it fails at 7 we ship 5 and say so |
| R4 | Sync correctness bug silently loses drift | Low | Very High | Grants make client trait writes impossible (INV-10); additive deltas make LWW unreachable (INV-11); replay goldens; concurrency tests; the loss would be silent, so the defenses are structural rather than detective |
| R5 | Accessibility regresses after launch | Medium | High | Full a11y suite in CI on every PR; quarterly external sessions; reduced-motion has its own visual regression suite; INV-26/27 are gated, not reviewed |
| R6 | An announcement surface creeps in ("just a small toast") | **High** | High | No toast/banner/modal primitive exists in the app shell; copy-registry lint; affective QA checklist item; this is the failure `product_brief.md` predicts most confidently, so it gets the most mechanical guard |
| R7 | A gamification feature is argued for post-launch | High | Very High | No behaviour metrics exist to justify one (§16.4); INV-04 gates; the absence of the underlying data is the durable defense |
| R8 | First-bird budget missed on real mobile networks | Medium | High | Edge snapshot inlining, 26KB critical module, SW precache, 110ms headroom; synthetic monitoring from four geographies; if missed, the quiet field degrades gracefully — it is never a spinner |
| R9 | Tick cost per account exceeds budget at scale | Medium | Medium | Dormancy classes + analytic absence collapse (§6.2); `SKIP LOCKED` horizontal scaling; long-horizon cohort measures real cost from M4; `step()` is a pure function and portable if needed |
| R10 | Notebook prose flattens over months | Medium | Medium | Recency penalty; 120-template launch corpus; aggregate usage distribution monitoring; standing quarterly content commitment |
| R11 | Magic-link deliverability (spam folders, corporate scanners) | Medium | High | POST-on-interstitial consumption; SPF/DKIM/DMARC from M0; delivery-latency SLO; clear matter-of-fact retry copy; deliverability monitoring as a launch-blocking check |
| R12 | Traits saturate, all birds converge | Medium | Medium | Per-bird ceilings (A-06); long-horizon cohort verifies at 6 and 12 months |
| R13 | Users never notice a new bird arriving (§6.15) | Medium | Low | M5 usability testing; documented fallback is a single quiet top-bar glyph, never a modal |
| R14 | Presence over-credited via multi-device or automation | Medium | Medium | Union-over-seconds de-dup (§7.3), wall-clock clamp, 4h/day cap, server-side window validation |

The four risks `1-START_HERE.md` names — drift calibration, sync correctness, audio uncanniness, accessibility regression — are R1, R4, R2, R5. Each has a structural mitigation rather than a review-time one, because each fails silently: none of them produces an error the on-call engineer sees.

---

## 18. Assumptions and judgment calls

The PRD leaves these open; this plan decides them so the build does not stall on them. Each is defensible and each is reversible.

| # | Gap | Call | Rationale |
|---|---|---|---|
| A-01 | "Activity window is a few minutes; calibrate during build, leaning long" | **240s**, server config | `interactions.md` explicitly says watching without moving is the actual product; four minutes is at the long end of "a few" without counting an empty chair |
| A-02 | No stated cap on daily credited presence | **4h/day** | Bounds outlier drift contribution; well above any plausible real session; recorded so it is a decision, not an accident |
| A-03 | "Regular visits" undefined for calibration | **15 min/day, 5 days/week = 75 presence-min/week** | Matches the brief's "short and uneventful by design" session; all calibration numbers derive from it and move together if it changes |
| A-04 | `bird_engine.md` lists an unfinalized mood set; `settled` is overloaded | Moods = `alert, curious, content, wary, drowsy, roosting`; `settled` stays reserved for aviary lighting | Prevents a class of bug where the settle gesture appears to mutate bird mood records |
| A-05 | Weather is "a few times a week" | Rain 2.5/wk (4–11 min), wind 4/wk (6–20 min), intensity capped ≤ 0.4 | Frequent enough to be part of the place, rare enough never to become a feature |
| A-06 | Trait ranges and seeds are "an implementation detail" | Traits in [0,1], seeded ≈ 0.35, **per-bird ceilings ~ U(0.72, 0.98)** | Ceilings preserve individuality at 12+ months; without them every mature bird converges and the aviary flattens |
| A-07 | Whether presence affects call rate | Yes, capped at ×1.25 | `bird_engine.md` frames vocal frequency as "how often the bird calls *when unobserved*," implying observation matters; the cap keeps it from reading as performance |
| A-08 | Offer cooldown is "a few minutes" | **4 minutes per bird**; the affordance is never disabled or shown with a countdown | Long enough to prevent curiosity saturation, short enough that a gesture is not a wait; a visible countdown would be a game mechanic |
| A-09 | New-bird pacing is "tied to aviary age" | 90 / 180 / 300 / 450 / 600 days | Matches "a few months old offers a third; a year-old may have grown to five or six"; server config |
| A-10 | "First bird within 500ms" — no percentile given | **p75 ≤ 500ms**, p95 ≤ 900ms, on the stated mid-tier-mobile/4G profile | A hard p100 on an unbounded network is unmeasurable; p75 with a p95 guardrail is enforceable and honest |
| A-11 | Export includes personality vectors, but vectors are never exposed | Export ships them as a portability file; no product surface ever renders them; no session endpoint returns them | Honors both statements. The hidden-numbers rule protects the *relationship*, and a JSON file the user requested and downloaded is not the relationship surface. Flagged for product sign-off |
| A-12 | Timezone handling unspecified in detail | IANA zone from the client, recomputed per render (DST/travel safe); multi-device conflicts resolve to the most recent presence-bearing session | Day/night must follow the user's day, including when they travel |
| A-13 | Transport for state | HTTP polling on the three PRD-named triggers; no WebSocket | A 60s tick makes a socket deliver nothing sooner at meaningfully higher operational cost |
| A-14 | How the new-bird "offer" surfaces without announcing | The bird simply arrives at the back perch for 7 days; noticing it *is* accepting it | The only reading that satisfies both `bird_engine.md`'s offer and the brief's ban on announcement. Documented fallback in §6.15 |
| A-15 | Whether the notebook/narration can use a hosted language model | No | Would send per-bird interaction data to a third party, contradicting the privacy commitment outright |
| A-16 | Species pool composition | 6 species: warbler-like, finch-like, thrush-like, wren-like, tit-like, nightjar-like (the night-active one `aviary_layout.md` requires) | A coherent set from one place, with the night species the layout spec names |

Two of these — A-11 and A-14 — resolve genuine tensions inside the PRD rather than filling silence. Both are flagged for product sign-off before M5; the build does not depend on the outcome, only on a decision.

---

## 19. Open items owned outside this plan

- **Design system spec** (visual designer): exact palette values per day-phase, contrast ratios per surface, focus-ring treatment, top-bar iconography, reduced-motion pose sets, still-pool and seed art. Needed by M3.
- **Prose corpus** (writer): ~120 notebook templates, narration templates, caption vocabulary, the six transactional email bodies. Notebook corpus needed by M4; narration by M5.
- **Species art and motif libraries** (design-engineer + audio owner): 6 silhouettes with rig geometry, 3–6 motifs each. Needed by M3 (visual) and M4 (audio).
- **Privacy policy text** (legal + writer): plain-language, naming the aggregate telemetry categories and explicitly excluding per-bird interaction state, linked from account settings. Needed by M6.
