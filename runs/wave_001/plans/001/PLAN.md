# Pocket Aviary — v1 implementation plan

Prepared from the v1 PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). This document is the executable plan for the engineering team. It interprets the spec into services, schemas, algorithms, budgets, tests, and a rollout order. Where the PRD is silent or two files pull in different directions, the plan makes a call and records it in the decision log (§16) so nobody has to re-argue it.

The PRD's vocabulary (`concepts.md`) is binding in code as well as in copy: `bird`, `call`, `mood`, `personality`, `drift`, `presence`, `listenIn`, `offer`, `settle`, `notebook`, `visit`, `tick`. Identifiers like `pet`, `creature`, `solo`, `select`, `song` (outside `songFragment` offer), `streak`, `achievement`, `score`, `level`, `badge`, `toast` are banned by lint in product code and product strings.

---

## Table of contents

0. Reading guide and the twelve load-bearing invariants
1. Scope (v1 in / out)
2. Architecture
3. Data model
4. API surface
5. Simulation engine design
6. Sync model
7. Frontend rendering pipeline
8. Audio pipeline
9. Interaction flows (client/server contract)
10. Accessibility surfaces
11. Performance budgets and observability
12. Accounts, privacy, and security engineering
13. Testing, calibration harness, and CI gates
14. Rollout
15. Risks
16. Decision log (ambiguities resolved)
17. Work breakdown and sequencing

---

## 0. Reading guide and the twelve load-bearing invariants

Everything below is organized so that a team can split it by package and build in parallel after a short foundations phase (§17). Before splitting, every engineer should be able to recite the invariants. They are not preferences; each has a named test or a structural enforcement in this plan.

| # | Invariant | Enforced by |
|---|-----------|-------------|
| I1 | The server simulation tick is the only writer of personality vectors and canonical aviary state. Clients only append interaction events. | DB role separation (§3.6); API has no UPDATE grant on `birds.traits_*` or `aviary_state`; contract tests. |
| I2 | Drift is monotonic toward expressive. No code path decreases a trait. | Non-negative delta assertion in engine; DB trigger rejects trait decreases (§3.6); property tests. |
| I3 | Presence = `visibilityState === 'visible'` AND window focus AND pointer/key activity within the activity window. All three, simultaneously. | Single `PresenceMonitor` module with unit tests for each 2-of-3 failure case; server caps per-tick presence at 60 s. |
| I4 | Raw personality numbers never reach any product surface, including the network payloads the client receives. The snapshot carries a phenotype projection, not traits. | Snapshot schema has no trait fields; schema test; the only egress of raw traits is the account export artifact (§16, D4). |
| I5 | No recorded audio anywhere. Calls are synthesized client-side from the call grammar. Fallback is silence with captions. | Build-time check: no audio assets in the bundle; audio module has no `fetch` of media. |
| I6 | No announcement surfaces: no welcome toast, banner, modal, badge, streak, visit-frequency display, or friend-visited notice (except the opt-in email toggle in §9.8). | No toast/notification component exists in the product bundle; design review checklist; string lint forbids "welcome back", "you've been", "streak", "days in a row". |
| I7 | Per-account interaction state never enters telemetry, analytics, or any aggregate pipeline. | OpenTelemetry attribute allowlist (§11.4); separate DB credentials; log redaction tests. |
| I8 | The account identifier everywhere is a synthetic UUID. Email lives once, encrypted, on `account_emails`. | Schema; lint forbids `email` in log/metric/queue-key call sites outside the auth module. |
| I9 | Bird identity is a stable UUID that survives rename, sync, migration, and species-pool changes. | Immutable `birds.id`; migrations are reviewed against a "no bird replacement" checklist. |
| I10 | The notebook is read-only, sparse, and observes birds, never the user's behavior. | No write endpoint; sparsity budget in the observer (§5.10); template voice lint. |
| I11 | Visitors are read-only and never produce presence or interaction events. | Visitor sessions have no scope on the events endpoint; observer ignores visitor pulls. |
| I12 | Accessibility (narration, reduced-motion renderer, captions, keyboard) ships in v1, not after. | These are launch gates in §14; CI runs axe plus custom checks; reduced-motion visual regression suite. |

---

## 1. Scope

### 1.1 In v1

- Web app (last two major versions of Chrome, Safari, Firefox, Edge). Unsupported browsers get a matter-of-fact page.
- Single-user accounts; magic-link sign-in; per-device revocable sessions; email change with verification; JSON export by emailed link; soft delete (30 days) then hard delete.
- One canonical aviary per account; server-side simulation tick; multi-device consistency as a property of the architecture.
- Two starter birds chosen by the system from a pool of six species; user names them; cap of seven; new birds arrive by aviary age.
- Bird engine: hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic drift, mood model with persistence, bird-to-bird interaction, procedural calls with per-bird signatures, chorus.
- Scene: one horizontal screen, three perch zones, local-time day/night, rare rain and wind, ambient micro-motion, sparse fading top bar, no chrome inside the scene, motion already in progress on first frame, quiet-field loading state, empty-aviary state at adoption only.
- Interactions: return-greeting, presence accounting, listen-in, offers (seed, song fragment, still pool) with per-bird cooldown, settle with 5-second undo, field notebook.
- Social: per-invite read-only visits by email link, 30-day expiry, immediate revocation, visit log in settings, notification email off by default with an opt-in toggle.
- Accessibility: naturalist screen-reader narration, reduced-motion renderer (cross-fade register), procedural captions, WCAG AA on all user copy, full keyboard navigation with visible focus.
- Performance: initial bundle < 2 MB gz (internal target far lower), first bird visible < 500 ms on mid-tier mobile over 4G, 60 fps idle on a five-year-old laptop for 30 minutes, no memory growth over 30 minutes, synthetic monitoring and aggregate-only RUM.

### 1.2 Out of v1 (from `non_goals.md` and the brief; do not build scaffolding for these either)

Native apps; payments; shared or multi-aviary accounts; customizable scenes; public discovery, profiles, follows, feeds, comments, chat, avatars, co-presence, leaderboards; achievements, streaks, levels, scores, badges, visit calendars, any visit-frequency surface; push notifications or emails about the aviary (the single exception is the opt-in visit email); Tamagotchi mechanics (hunger, death, distress, decaying happiness); recorded audio; catalog-based bird selection; user-arranged perches; personality numbers in any UI; data-model accommodations for native clients.

### 1.3 Explicitly deferred (allowed later, not designed for now)

SSO/password auth; raising the seven-bird cap; WebSocket/SSE push (v1 polls on the tick cadence); additional species; localization beyond English (the prose grammar is built so a locale pack can be added, but v1 ships English only).

---

## 2. Architecture

### 2.1 Shape

One TypeScript monorepo. TypeScript is chosen so that the call grammar, prose grammar, bird geometry, PRNG, and state types are one implementation shared by the client, the API, the simulation worker, and the edge function. Sharing this code is what makes captions match the call that played, narration match the scene, the edge-rendered first frame match the canvas takeover, and the simulation deterministic across replay.

```
packages/
  engine/        pure, side-effect-free: types, fixed-point math, PRNG, drift, mood,
                 perch choice, call grammar (plan + caption), prose grammar (notebook,
                 narration, greeting text), species definitions, bird geometry (poses)
  client/        scene renderer (Canvas 2D), reduced-motion renderer, audio (AudioWorklet
                 synth), presence monitor, chrome (Preact), snapshot client, event queue
  api/           HTTP API: auth, state reads, event appends, offers, settings, visits,
                 export, deletion
  sim/           tick workers, catch-up sweeper, observer (notebook), edge publisher
  edge/          CDN edge function: HTML shell, inlined snapshot0, edge-rendered first frame
  mailer/        magic links, invites, export links, opt-in visit emails
  tools/         calibration harness, synthetic-user profiles, listening-test kit,
                 voice linter, bundle/perf budget checks
```

Runtime choices (defensible defaults; swap only with a written reason):

- Node 22 LTS for `api`, `sim`, `mailer`. Fastify for HTTP.
- Postgres 16 as the single system of record. Partitioning for the event log. No second datastore for canonical state.
- Redis only for rate limiting and short-lived locks; nothing canonical lives there.
- CDN with edge compute and an edge key-value store (Cloudflare Workers + KV or equivalent). The edge holds the HTML shell, static assets, and the last snapshot per aviary for first-paint.
- Object storage (S3-compatible) for export artifacts, private, signed URLs.
- Transactional email provider with DKIM/SPF/DMARC set up before beta; a second provider configured as failover because magic link is the only way in.
- Client: TypeScript, Preact for chrome (settings, notebook, adoption, visit surfaces), hand-written Canvas 2D scene renderer, AudioWorklet synth. No animation library, no game engine, no audio middleware. Vite for bundling with strict chunking (§11.1).

### 2.2 Service responsibilities

| Component | Owns | Never does |
|-----------|------|------------|
| `sim` tick worker | Personality drift, mood transitions, perch choice, weather, call schedule, greeting candidates, arrivals, notebook entries, canonical `aviary_state` writes, edge snapshot publish | Serve HTTP; read telemetry; touch email |
| `api` | Auth, sessions, event append, synchronous offer resolution (mood-level transient state only), state reads with catch-up trigger, settings, visits, export, deletion | Write traits; write `aviary_state` outside the shared engine's offer-resolution path (§5.11) |
| `edge` | Session cookie → aviary lookup, snapshot0 inline, first-frame SVG render, static assets, unsupported-browser page | Any writes; any decision about state |
| `client` | Rendering, interpolation, micro-motion, ornaments, audio synthesis and mixing, presence detection, narration and caption prose, local UI state (listen-in, top-bar fade, settle undo timer) | Own or persist personality/mood; compute drift; tick |
| `mailer` | Outbound email with templates in matter-of-fact voice | Anything with bird state |

### 2.3 The render-pipeline boundary

The server produces **behavioral intent** on a slow clock; the client produces **motion, sound, and prose** on a fast clock.

Server → client (in the snapshot, §4.3): per bird: perch zone and slot, mood, current activity kind with phase, phenotype (colors, size, temperament band), call schedule for the next ~120 s, greeting candidates, offer state and cooldowns, weather, settled flag, aviary-level observations needed for prose.

Client-only (never in the snapshot): sub-perch micro-motion (preen, scan, tilt, shuffle), flight arcs between perches, leaf and feather ornaments, palette by local time, listen-in mix, caption text, narration prose, top-bar opacity, focus rings.

Rule of thumb: if two devices should see it the same way, it comes from the server. If two devices may legitimately differ (a leaf, a preen timing, which device is listening in), it is client-local and seeded per device.

### 2.4 Request flow for a session start (the 500 ms path)

1. Browser requests `/`. Edge function reads the session cookie, looks up `session:<hash> → aviary_id` in edge KV, then `snap:<aviary_id> → snapshot0`.
2. Edge returns the HTML shell with: critical CSS inlined; `snapshot0` inlined as JSON; the first frame pre-rendered as inline SVG from `snapshot0` using the shared bird geometry (birds in their current poses, sky in the local-time palette computed from the request's `tz` hint in the snapshot with a CSS breathing animation); `modulepreload` for the critical chunk.
3. Critical JS (target ≤ 90 KB gz) boots, builds the scene from `snapshot0`, draws the same poses onto the canvas, swaps the SVG out on the same frame (no visual discontinuity), and fires the return-greeting chosen from `greeting_candidates` and the client-computed absence length (§9.1).
4. In parallel, the client calls `GET /v1/aviary/state?reason=session_start`, which runs catch-up ticks if the aviary is behind, records `session.start`, and returns a fresh snapshot. The client blends toward it using ordinary transitions.
5. Deferred chunks (audio synth, chrome, notebook, settings) load after first paint. Audio starts as soon as the AudioContext is allowed to run (§8.6).

If the edge has no snapshot (first ever load, cold KV), the shell renders the quiet field (soft sky, one faint leaf cue) and the client draws birds when the API snapshot arrives. No spinner exists in the codebase.

---

## 3. Data model

All timestamps are `timestamptz`. All primary keys are UUIDv7 generated server-side. Trait values are stored as `int4` in fixed point (units of 1e-6, range 0..1,000,000) so that drift arithmetic is integer-exact and replay is bit-identical across Node versions and CPUs.

### 3.1 Accounts and auth

```
accounts
  id                 uuid pk            -- the only identifier used anywhere else
  created_at         timestamptz
  timezone           text               -- IANA; updated on session.start
  settings           jsonb              -- {captions, reduced_motion, narration_cadence,
                                        --  volume, muted, visit_notify_email, new_arrivals_paused,
                                        --  high_contrast_focus}
  deletion_requested_at timestamptz null
  hard_delete_after  timestamptz null

account_emails
  account_id         uuid fk unique
  email_ciphertext   bytea              -- envelope-encrypted (KMS data key)
  email_blind_index  bytea unique       -- HMAC-SHA256(server_key, normalized email); lookup only
  verified_at        timestamptz
  pending_email_ciphertext bytea null   -- during change-of-email
  pending_blind_index bytea null
  pending_expires_at timestamptz null

magic_links
  id uuid pk, blind_index bytea, token_hash bytea unique, purpose text
  ('sign_in'|'verify_new_email'), created_at, expires_at (+15 min), consumed_at null,
  request_ip_hash bytea

device_sessions
  id uuid pk, account_id fk, token_hash bytea unique, created_at, last_seen_at,
  expires_at (sliding 180 d), ua_family text, revoked_at null
```

### 3.2 Aviary and birds

```
aviaries
  id uuid pk, account_id fk unique, created_at (aviary age anchor),
  species_seed bytea, max_birds int default 7,
  tick_index bigint, last_tick_at, next_tick_at, hot_until timestamptz,
  last_consumed_event_seq bigint, settled_at timestamptz null

birds
  id uuid pk (immutable), aviary_id fk, species text, adopted_at,
  name text, name_updated_at,
  signature_seed bytea             -- fixed at adoption; drives call signature and geometry jitter
  trait_boldness int4, trait_warmth int4, trait_vocal int4, trait_plumage int4, trait_curiosity int4
  attention_ema int4               -- per-bird low-pass attention state (fixed point)
  mood text, mood_since timestamptz, mood_dwell_until timestamptz,
  mood_pressure jsonb              -- decaying recent-interaction terms
  perch_zone text ('front'|'middle'|'back'), perch_slot int2, perch_since timestamptz,
  offer_cooldown_until timestamptz null,
  arrived_unnamed boolean          -- newcomer awaiting a name

aviary_state
  aviary_id pk fk, seq bigint, tick_index bigint,
  state jsonb                      -- full canonical snapshot source (server-side form, §4.3)
  presence_24h int4                -- rolling presence seconds in trailing 24 h (ring of 24 hourly buckets in jsonb)
  weather jsonb, greeting_history jsonb (last 14 days of who-greeted-first),
  observer_state jsonb             -- sparsity bucket, last entry at, seen-facts set
  updated_at
```

`aviary_state.state` is the canonical record. `birds` holds the durable per-bird truths (identity, traits, mood) as columns so constraints and triggers can guard them; the JSONB is regenerated by the tick from the columns plus transient fields.

### 3.3 Event log (append-only)

```
interaction_events  (partitioned by range on ts, monthly; older partitions detached
                     and dropped 90 days after they are fully consumed and the
                     notebook has no pending need for them)
  aviary_id uuid, seq bigint (per-aviary monotonic, assigned by trigger),
  event_id uuid (client-generated, unique per aviary), device_session_id uuid,
  ts_server timestamptz, ts_client timestamptz,
  kind text, payload jsonb
  primary key (aviary_id, seq)
  unique (aviary_id, event_id)
```

Kinds and payloads:

| kind | payload |
|------|---------|
| `session.start` | `{reason: 'navigation'|'visibility'|'resume', absence_bucket, greeter_bird_id, greeting_form, tz}` |
| `presence.start` | `{segment_id}` |
| `presence.heartbeat` | `{segment_id, seconds_since_last: ≤ 30}` |
| `presence.end` | `{segment_id, reason: 'hidden'|'blur'|'inactive'|'settle'|'unload'}` |
| `listen_in.start` / `listen_in.end` | `{bird_id}` |
| `offer.made` | `{offer_id, kind: 'seed'|'song'|'pool', song_id?}` |
| `offer.resolved` (server-written) | `{offer_id, reactions: [{bird_id, reaction}]}` |
| `settle` / `settle.undo` | `{}` |
| `bird.renamed` | `{bird_id}` (server-written; for the observer only) |

The API role has `INSERT` and `SELECT` only. There are no `UPDATE`/`DELETE` grants on this table for any runtime role; hard-deletion runs under a dedicated `purger` role.

### 3.4 Notebook

```
notebook_entries
  id uuid pk, aviary_id fk, written_at, local_date date,
  text text, facts jsonb   -- the structured observation the text was generated from
                           -- (bird ids, event kind); used for dedup, never shown
  index (aviary_id, written_at desc)
```

### 3.5 Visits

```
visit_invites
  id uuid pk, aviary_id fk, visitor_email_ciphertext bytea, visitor_blind_index bytea,
  token_hash bytea unique, created_at, expires_at (+30 d), revoked_at null, first_used_at null

visitor_sessions
  id uuid pk, invite_id fk, token_hash bytea unique, created_at, last_pull_at, ended_at null

visit_log
  id uuid pk, aviary_id fk, invite_id fk, started_at, last_seen_at
  -- duration shown to the host is last_seen_at - started_at, rounded to 5 min
```

### 3.6 Roles, constraints, triggers

- Roles: `api_rw` (insert events; select state/birds; update `accounts`, `account_emails`, `device_sessions`, `birds.name`, `birds.arrived_unnamed`, visits), `sim_rw` (update `birds` traits/mood/perch, `aviary_state`, insert `notebook_entries`, insert `offer.resolved` and `bird.renamed` events), `purger` (delete everything for one account), `readonly_ops` (no access to `interaction_events.payload`, `birds`, `aviary_state`, `notebook_entries`; used by on-call tooling).
- Trigger `birds_trait_monotonic`: on `UPDATE OF trait_*`, raise if any new value < old value unless `current_setting('aviary.allow_trait_decrease', true) = 'migration'` and the connection role is `sim_migrator`. Every migration that sets it is reviewed as a bird-integrity change.
- Trigger `birds_id_immutable`: `id`, `aviary_id`, `species`, `signature_seed`, `adopted_at` are immutable after insert.
- Check: `birds` count per aviary ≤ `aviaries.max_birds` (enforced in the arrival path under the aviary row lock, and by a deferred constraint trigger).
- Check: trait columns within `[0, 1_000_000]`.

### 3.7 Edge key-value

```
session:<sha256(session_token)>  → {aviary_id}            TTL 180 d; deleted on revoke
visit:<sha256(visitor_token)>    → {aviary_id, invite_id} TTL = invite expiry; deleted on revoke
snap:<aviary_id>                 → snapshot0 JSON (≤ 8 KB)  TTL 7 d; rewritten on every tick
                                    of a hot aviary and on every cold sweep
```

The edge KV holds bird state for rendering. It is part of the simulation data path, not telemetry, and is encrypted at rest by the provider. Nothing in it is ever read by analytics.

---

## 4. API surface

All endpoints are JSON over HTTPS under `/v1`. Account endpoints require a device-session cookie (httpOnly, Secure, SameSite=Lax). Visitor endpoints require a visitor-session cookie. Every error body is `{code, message}` with `message` in matter-of-fact voice, ready to display.

### 4.1 Auth and account

| Method | Path | Notes |
|--------|------|-------|
| POST | `/auth/magic-link` | `{email}` → 202 always (no account enumeration). Rate limit 5/h and 20/d per blind index; 30/h per IP. |
| GET | `/auth/consume?token=` | Verifies hash, expiry, single use; creates account on first sign-in; issues device session; redirects to `/` (or `/adopt` for a new account). Errors render the matter-of-fact sign-in page. |
| POST | `/auth/sign-out` | Revokes current session; deletes edge `session:` key. |
| GET | `/account` | Settings, session list (id, ua_family, created, last seen, `current` flag), deletion state. Never returns email plaintext except on the account page itself (decrypted on demand). |
| PATCH | `/account/settings` | Partial update of `accounts.settings`. |
| POST | `/account/email-change` | `{new_email}` → verification link to the new address; old continues until consumed. |
| DELETE | `/account/sessions/:id` | Revoke a device session. |
| POST | `/account/export` | Enqueue export; 1/day; emailed signed link (24 h). |
| POST | `/account/delete` | Soft delete; sets `hard_delete_after = now + 30 d`. |
| POST | `/account/undelete` | "I changed my mind" within the window. |

### 4.2 Aviary state and events

| Method | Path | Notes |
|--------|------|-------|
| GET | `/aviary/state?reason=&since_seq=&tz=` | `reason ∈ session_start|visibility|resume|keepalive`. Runs catch-up (§5.2). For `session_start`/`visibility`, records `session.start`, updates `timezone`, marks the aviary hot, clears `settled_at`. Returns the snapshot (§4.3) or `{unchanged: true, seq}` when `since_seq` is current. |
| POST | `/aviary/events` | Batch of events `[{event_id, kind, ts_client, payload}]`, ≤ 50 per batch. Idempotent by `event_id`. Returns accepted seqs. Marks the aviary hot. |
| POST | `/aviary/offers` | `{offer_id, kind, song_id?}` → synchronous resolution (§5.11); returns the reaction plan and the updated snapshot. Also appends `offer.made` and `offer.resolved`. |
| PATCH | `/aviary/birds/:id/name` | `{name}` (1–24 chars, trimmed). Clears `arrived_unnamed`. |
| GET | `/aviary/notebook?before=&limit=` | Cursor pagination, newest first, `limit ≤ 50`. |
| GET | `/aviary/config` | Client calibration values: presence activity window, heartbeat interval, keepalive interval, offer durations, greeting rate limit. Cached 1 h. |

`sendBeacon`-friendly variant: `POST /aviary/events` accepts `text/plain` bodies containing the same JSON so `navigator.sendBeacon` can deliver `presence.end` on unload.

### 4.3 Snapshot schema (server → client)

The snapshot is the only state the client renders from. It contains no trait values (I4).

```jsonc
{
  "seq": 18233, "tick_index": 412990, "server_time": "2026-…Z", "tz": "Europe/Lisbon",
  "aviary": {
    "id": "…", "age_days": 143,
    "settled": false,
    "weather": { "kind": "rain", "intensity": 0.3, "started_at": "…", "ends_at": "…" },
    "last_presence_ended_at": "…",           // for client-side absence bucket at boot
    "greeting_candidates": [                 // ranked by the server at the last tick
      { "bird_id": "…", "forms": { "short": "glance", "medium": "two_note",
                                    "long": "approach_call", "very_long": "approach_call" } },
      { "bird_id": "…", "forms": { "short": "none", "medium": "glance",
                                    "long": "response_call", "very_long": "approach_call" } }
    ],
    "greeting_rate_limit_s": 120,
    "call_schedule": [                       // next ~120 s, absolute times
      { "at": "…", "bird_id": "…", "plan_seed": 9134, "kind": "solo" },
      { "at": "…", "bird_id": "…", "plan_seed": 9135, "kind": "response", "to": "…" },
      { "at": "…", "bird_ids": ["…","…"], "plan_seed": 9136, "kind": "chorus" }
    ],
    "offer": {
      "active": [ { "offer_id": "…", "kind": "pool", "placed_at": "…", "expires_at": "…",
                    "reactions": [ { "bird_id": "…", "reaction": "bathe", "starts_at": "…" } ] } ],
      "kinds_available": { "seed": true, "song": true, "pool": false }
    },
    "newcomer_bird_id": null
  },
  "birds": [
    {
      "id": "…", "name": "pip", "species": "reed_warbler", "arrived_unnamed": false,
      "phenotype": {
        "palette": { "body": "#8a7d5c", "wing": "#6f6448", "breast": "#c9b98a", "eye": "#1f1a12" },
        "size": 0.96, "tail": 1.05,                    // geometry jitter from signature_seed
        "temperament": "easy"                          // reserved | easy | lively (coarse band)
      },
      "mood": "content", "mood_since": "…",
      "perch": { "zone": "front", "slot": 1 }, "perch_since": "…",
      "prev_perch": { "zone": "middle", "slot": 2 },
      "activity": { "kind": "preen", "phase": 0.4 },   // what the bird is doing at snapshot time
      "call": { "grammar": "reed_warbler", "signature_seed": 55121,
                "mood_shaping": "content", "mix_bias": 0.0 }
    }
  ]
}
```

Notes: `temperament` is a three-band projection of boldness and curiosity blended with mood and per-snapshot noise; it drives idle-motion style. `mix_bias` is a per-bird loudness offset derived from perch zone. `signature_seed` is a public seed for the call signature, not a trait. Names are returned as stored; product surfaces lowercase them at render time (D9).

### 4.4 Visits

| Method | Path | Notes |
|--------|------|-------|
| POST | `/visits/invites` | `{email}` → invite + email with `/visit/<token>`. Limit 10 outstanding invites per aviary, 20 sends/day. |
| GET | `/visits/invites` | Outstanding and active invites; visit log (email decrypted on demand, started, approximate duration). |
| DELETE | `/visits/invites/:id` | Revoke: sets `revoked_at`, deletes edge `visit:` key, ends visitor sessions. |
| GET | `/visit/<token>` (page) | Edge validates `visit:` key; on miss, the API validates; issues a visitor-session cookie; renders the visitor shell. Expired/revoked → matter-of-fact page: "This visit is no longer available." |
| GET | `/visitor/state?since_seq=` | Same snapshot as the host would get, minus `greeting_candidates`, `offer.kinds_available`, `last_presence_ended_at`. Each pull re-checks revocation and updates `visit_log.last_seen_at`. Marks the aviary hot. |

Visitor sessions cannot call `/aviary/*`. The visitor client build is the same bundle with `mode: 'visitor'`, which removes the offer, settle, notebook, and account affordances and never starts the presence monitor.

---

## 5. Simulation engine design

The engine (`packages/engine`) is a pure function library: `tick(state, inputs, config, rng) → state'`. The `sim` service wraps it with persistence. The `api` service calls only two engine entry points synchronously: `resolveOffer` and `rankGreeting` (read-only). Nothing else in the engine runs outside a tick.

### 5.1 Clocks and determinism

- Logical tick = 60 s. `tick_index = floor((t − epoch) / 60 s)`.
- All randomness comes from a counter-based PRNG seeded by `(aviary_id, tick_index, purpose, bird_id?)`. No `Math.random` in the engine (lint rule).
- Trait math is fixed-point integer. Mood scoring uses fixed-point too. Floats are allowed only for outputs that don't feed back into state (call schedule offsets, colors).
- Property test: applying `tick` N times sequentially equals `catchUp(state, N)`; both equal a golden replay on CI across Node versions.

### 5.2 Cadence tiers and catch-up

The aviary's state is defined at every logical tick whether or not anyone is connected. Execution is tiered to keep cost proportional to live use while preserving that definition exactly:

- **Hot**: any account or visitor pull/event in the last 5 minutes (`hot_until`). Ticked every 60 s by workers claiming due rows with `SELECT … FOR UPDATE SKIP LOCKED WHERE next_tick_at <= now()`.
- **Cold**: everything else. A sweeper catches up all cold aviaries every 15 minutes (≤ 15 logical ticks per pass, batched in one transaction per aviary).
- **On read**: `GET /aviary/state` and `/visitor/state` first call `ensureCurrent(aviary)`. If the aviary is behind by ≤ 30 ticks, catch-up runs inline under the row lock (≈ 1–3 ms). If behind by more (sweeper outage), the API returns the last snapshot immediately and enqueues catch-up; the client's next keepalive picks up the result.

Because catch-up is bit-identical to sequential ticking (§5.1 test), users cannot tell which tier ran. Notebook entries and weather that "happened" during absence are generated by catch-up with the tick-indexed RNG, so they are the same entries that minute-by-minute ticking would have produced.

### 5.3 Tick algorithm

```
tick(aviary, tickIndex):
  BEGIN; lock aviary_state row
  events   = interaction_events where seq > last_consumed_seq and ts_server < tickEnd
  inputs   = aggregate(events)       // per tick window:
             //   presenceSeconds (union across devices, capped 60)
             //   listenInSeconds[birdId] (capped 60)
             //   offersAccepted[birdId], offersMade (count)
             //   settled (bool), sessionStarted (greeter, absence bucket)
  local    = toLocalTime(tickStart, aviary.timezone)
  rng      = prng(aviary.id, tickIndex)

  weather  = advanceWeather(state.weather, local, rng)               // §5.8
  for bird in birds:
    bird.attention = lowPass(bird.attention, inputs, presence24h)    // §5.4
    delta          = driftDelta(bird, inputs)                        // ≥ 0 per trait
    bird.traits    = min(MAX, bird.traits + delta)
  moods    = transitionMoods(birds, local, weather, inputs, rng)     // §5.5
  perches  = choosePerches(birds, moods, rng)                        // §5.6
  schedule = scheduleCalls(birds, moods, weather, local, rng, 120 s) // §5.7
  greeters = rankGreeting(birds, moods, rng)                         // §9.1
  newcomer = maybeArrive(aviary, local, rng)                         // §5.9
  entry    = observer.consider(prev, next, inputs, local, rng)       // §5.10
  presence24h = rollHourlyBuckets(presence24h, inputs.presenceSeconds)
  write birds, aviary_state (seq+1), notebook entry if any, last_consumed_seq
  COMMIT
  publishSnapshot(aviary.id)   // edge KV, at-least-once, outside the transaction
```

Tick compute for seven birds is well under 1 ms; the transaction is dominated by the two row writes.

### 5.4 Drift function

Traits `T ∈ [0, 1]` (fixed point). Per bird, a low-pass attention state `A` integrates the attention inputs with a one-day time constant; the trait delta is proportional to `A` and to the remaining headroom, so drift slows as a trait approaches its ceiling. Drift is strictly non-negative because every input is non-negative.

Per tick (60 s), for each bird:

```
p   = presenceSeconds / 60                        // 0..1, aviary-wide
s   = saturation(presence24h)                      // diminishing returns after ~30 min/day:
                                                   //   1.0 for ≤ 30 min, → 0.2 by 120 min, floor 0.2
a_t = s · ( w_p · p
          + w_l · listenInSeconds[bird] / 60
          + w_o · offersAccepted[bird]
          + w_m · min(offersMade, 1) · nearBonus(bird) )
A  ← A + α · (a_t − A)          α = 1/1440        (time constant ≈ 1 day)

for trait k:
  ΔT_k = K_k · A · (1 − T_k)     // K_k per-trait gain, all ≥ 0
  T_k ← min(1, T_k + ΔT_k)
```

Starting constants (versioned in `drift_calibration.v1`, stored server-side, tuned by the harness in §13.3):

| Input | Weight | Affected traits (gain multipliers) |
|-------|--------|------------------------------------|
| presence `w_p = 1.0` | dominant | boldness 1.0, warmth 0.8, vocal 0.8, plumage 1.0, curiosity 0.5 |
| listen-in on this bird `w_l = 1.5` | strong | warmth 1.5, vocal 1.5 |
| offer accepted by this bird `w_o = 0.6` | small | curiosity 1.0 |
| offer made at all `w_m = 0.2` | small | boldness 0.3 for birds that approached (`nearBonus = 1`), 0.1 for the rest |
| settle | 0 | none; closes the presence window cleanly |

Base gain `K = 8e-4` per tick per unit of `A`. With a "regular" profile of 15 minutes of presence per day, `A ≈ 0.010` at steady state, which yields roughly +0.05 on a mid-range trait after one week (measurable by instruments) and +0.13 to +0.16 after three weeks (visible: a bird that has crossed the perch-preference threshold or greets first noticeably more often). A single one-hour session moves a trait by well under 0.01 during the session; the tail over the following day contributes ~0.03 total. The daily saturation prevents an eight-hour session from producing a week's drift in a day. The mapping from trait to observable behavior (§5.6, §9.1, §7.4) is deliberately high-gain around the 0.35–0.65 band so that +0.15 is felt without being announced.

What drift never does: decrease on absence, decrease on mute, decrease on visitor activity, react to visit counts, or react to anything the user did on a different account.

Trait seeds at adoption: each starter bird draws traits from species-typical ranges (e.g., a wren-like species seeds boldness 0.25–0.40, a robin-like species 0.45–0.60), with the two starters chosen so their boldness differs by ≥ 0.12. This guarantees a "bolder bird greets first, warier bird later" contrast from day one while leaving room for the shy one to change.

### 5.5 Mood model

Mood set (D2): `wary`, `content`, `curious`, `drowsy`, `alert`, `resting`. `resting` is the full-night state (eyes closed, low on the perch); the aviary-level lighting state uses the separate word `settled`.

Each tick, for each bird, compute a pressure vector over moods:

```
P[m] = base[m](local hour, species)              // dawn → alert; midday → content/curious; dusk → drowsy; night → resting
     + personality[m](traits)                      // high boldness lowers wary; high curiosity raises curious;
                                                   // high vocal raises alert at dawn
     + recent[m]                                   // decaying terms from offers accepted (+content, +curious),
                                                   //   listen-in (+content), settle (+drowsy)
     + ambient[m](weather)                         // rain: −alert, +drowsy, −(vocal shaping); wind: +alert for bold, +wary for shy
     + contagion[m](other birds)                   // a wary/alert neighbour adds +wary; a content majority adds +content
     + inertia · [m == current]
transition: if now ≥ mood_dwell_until, sample m' ~ softmax(P / τ); dwell_until = now + dwell(m') (8–25 min, seeded)
exception: an alarm call (bird enters alert from a wind gust or another bird's alert) may pre-empt dwell once per hour.
```

`recent` decays with a two-hour half-life and is zeroed at local dawn, which is the "daily-ish reset". The night-active species (`nightjar`) has a base curve that raises `alert` at night and lowers `resting`, so night is not a dead state.

Mood persists in `birds.mood` and continues to evolve in absence via the tick, so the bird a user returns to is the bird the ticks produced, never a default.

### 5.6 Perch choice

Nine slots: front 3, middle 3, back 3 (seven birds max leaves room for movement). Each tick a bird has a small probability (5–12%, mood-dependent) to reconsider; when it does, it scores zones:

```
score(front)  = 1.4·boldness + 0.5·[mood ∈ {curious, alert}] − 0.8·[mood == wary] − 0.6·[mood ∈ {drowsy, resting}] + 0.3·warmth·[a warm bird already on front]
score(middle) = 0.5
score(back)   = 1.0 − 1.2·boldness + 0.9·[mood == wary] + 0.5·[mood ∈ {drowsy, resting}]
```

Softmax over zones with hysteresis (+0.4 to the current zone), then a free slot in the chosen zone, preferring slots adjacent to birds the bird is warm toward. Front-perch time is one of the two most legible drift signals, so the boldness gain here is large on purpose.

### 5.7 Call scheduling and bird-to-bird interaction

The tick produces a schedule of call events for the next 120 s (overlapping the next tick so the client never runs dry):

- **Solo calls**: per bird, a Poisson process with rate `λ = base_rate(species) · f(vocal) · g(mood) · h(local hour) · w(weather)`. Vocal frequency maps `f` from 0.5× to 2.2×. `resting` → 0 except for the nightjar. Rain → 0.4×.
- **Responses**: for each scheduled call, each other bird responds with probability `0.15 + 0.5·warmth` (scaled down by wary/drowsy), delayed 0.6–2.4 s.
- **Chorus**: if two or more birds with `vocal ≥ 0.55` have solo calls within a 6 s window and it is dawn/dusk-adjacent or `content` is the majority mood, the tick merges them into a `chorus` event with per-bird offsets and a loose tempo reference; the client keeps the voices unlocked so it sounds like several birds, not one instrument.
- **Alarm**: a bird entering `alert` via wind or a random rare startle emits an alarm call; the contagion term in §5.5 responds on the next tick.
- Each scheduled call carries a `plan_seed`; the client expands it with the grammar (§8.2) so the same seed on two devices yields the same call.

If the client's schedule runs past its horizon without a fresh snapshot (network stall), the client extends locally with the same Poisson parameters until the next successful pull, then re-syncs to the server schedule at the next boundary.

### 5.8 Weather

Weather is a tick-driven aviary-level state: `none | rain | wind`. Rain onset probability is tuned to ~2–3 short rains per week (8–20 min each, intensity 0.2–0.6). Wind is ~2–4 events per week (5–12 min). Never both at once; never at the same clock hour as the last event. Weather modulates mood pressure and call rates (§5.5, §5.7) and is rendered client-side (§7.6) and sonified (§8.5). No thunder, no snow, no storms; the config schema has no room for them.

### 5.9 Arrivals (third bird and beyond)

Arrivals are keyed to aviary age only. Config `arrival_schedule.v1`: bird 3 at 75 days, bird 4 at 150, bird 5 at 240, bird 6 at 330, bird 7 at 480 (± a seeded jitter of up to 10 days so arrivals are not on the nose). The tick performs an arrival when `age ≥ threshold`, `count < max_birds`, the account's `new_arrivals_paused` setting is off, and the local time is early morning (arrivals happen at dawn, when the aviary is quiet).

The newcomer is a real bird from the first tick (stable id, traits seeded, drift active) that perches at the back, `arrived_unnamed = true`. The species is chosen to keep the aviary's call signatures separable (§8.3 recognizability check). The notebook writes it up (§5.10). There is no announcement; the bird being there is the notice. Naming lives in account settings → birds (D6). There is no decline in v1; the settings toggle `new_arrivals_paused` stops future arrivals.

### 5.10 The observer (notebook entry generation)

The observer runs inside the tick and decides, rarely, to write an entry. It has three parts:

1. **Fact extraction**: compares previous and next state and this tick's inputs to produce candidate facts with a novelty score. Examples: greeting order changed vs. the trailing 14 days; a low-boldness bird sat on the front perch for the first time this month; first rain since a given date; the nightjar called past midnight; a newcomer arrived; a bird bathed in the pool; a long quiet stretch (no calls for > 40 min during daylight); two birds chorused at dawn for the first time this week; a bird was renamed.
2. **Sparsity budget**: a leaky bucket of 1.0 entry per 3 days (capacity 2.0), plus an override lane for "landmark" facts (newcomer arrival, first-ever chorus, first front-perch by the shy bird) that can spend into a small negative balance. A fact is written only if `novelty × specificity ≥ threshold(bucket)`. Regular users land at one entry every 2–4 days; very active users are held to the same cadence because the budget is time-based, not interaction-based.
3. **Prose realization**: the shared prose grammar renders the fact in naturalist voice: lowercase (including names), present tense, no second person, no counts of the user's behavior, weekday allowed, bird names required where a bird is the subject. Each template has ≥ 4 phrasings and a "detail slot" filled from current state (perch, weather, light) so two entries about the same kind of fact read differently.

The observer has no access to visitor sessions, device sessions, or presence totals as a subject. It may use presence as a *condition* (a quiet-stretch entry only makes sense if nobody was listening in), never as content. The voice linter (§13.5) runs over every template and over 10,000 generated samples in CI.

### 5.11 Synchronous offer resolution

Offers need a reaction within a second, not at the next tick, so `POST /aviary/offers` calls `engine.resolveOffer` under the aviary row lock:

```
resolveOffer(state, offer, rng):
  reject if an offer of the same kind is active (kinds_available false)
  for bird in birds:
    if bird.offer_cooldown_until > now: reaction = 'ignore' (no drift for this bird)
    else score approach by mood/curiosity/boldness/kind:
      seed:  curious+content & curiosity high → 'approach' in 1–4 s; wary → 'approach_late' in 20–60 s;
             drowsy/resting → 'ignore' or 'watch'
      song:  vocal high & mood ∈ {content, curious, alert} → 'join' (a response call scheduled 1–3 s in);
             wary → 'quiet' (suppress calls 30 s); bold+vocal → 'call_against' (a counter-motif)
      pool:  by species and mood: 'drink' | 'bathe' | 'watch' | 'ignore'
    if reaction accepts (approach/approach_late/join/drink/bathe): bird.offer_cooldown_until = now + 4 min
  offer.expires_at = now + {seed: 2 min, song: 12 s, pool: 10 min}
  write state (seq+1), append offer.made + offer.resolved events
```

The tick later converts `offer.resolved` acceptances into drift (§5.4). Transient offer state is mood-level state; writing it from the API path is permitted by I1 because it never touches traits, and it runs through the same engine module the tick uses. The row lock serializes it with ticks.

---

## 6. Sync model

### 6.1 One canonical record, many readers

There is no client-to-client sync and no merge. Every device reads `aviary_state` (directly via the API, or its edge copy for first paint) and appends to one event log. The following rules make that correct rather than nominal:

1. **Single writer for durable state.** Only `sim_rw` updates traits, mood, perch, and `aviary_state`. The API's only state write is the offer-resolution path, which is engine code executed under the same row lock (§5.11) and never touches traits.
2. **Events are additive and idempotent.** Each client event has a client-generated `event_id`; the server ignores duplicates. Retries are safe. Events carry no absolute state, only what the user did.
3. **In-order consumption.** The tick consumes events by `seq` up to the tick boundary and stores `last_consumed_event_seq` in the same transaction as the state write. A crash mid-tick rolls back both; no event is consumed twice or skipped.
4. **Monotonic snapshot sequence.** Every canonical write increments `seq`. Clients discard any snapshot with `seq` lower than the one they hold (out-of-order responses on flaky networks).
5. **No last-write-wins anywhere near personality.** A device never sends a personality value; there is no field to send. The trait columns are guarded by I1/I2 triggers.

### 6.2 What is canonical vs. device-local

| State | Canonical (server) | Device-local |
|-------|--------------------|--------------|
| Traits, mood, perch, weather, call schedule, offers, cooldowns, newcomer, notebook | ✓ | |
| `settled` lighting state | ✓ (cleared by the next `session.start` from any device, D8) | |
| Listen-in focus | | ✓ (D7) |
| Top bar opacity, caption toggle state cache, narration timers, settle-undo timer | | ✓ |
| Accessibility and audio settings | ✓ in `accounts.settings`, mirrored locally for boot | cache |
| Presence | events → server aggregates per tick | detection |

Multi-device presence: two devices both sending heartbeats count as one presence stream because the tick caps `presenceSeconds` at 60 per minute. Two tabs on one device likewise.

### 6.3 Pull cadence

- On `session_start` / `visibility` return: immediate pull with catch-up.
- On a render-frame gap > 5 s (laptop sleep, throttled tab): pull with `reason=resume`.
- Keepalive while visible: every 60 s, aligned to a random phase so the fleet's pulls spread evenly; body is `{unchanged}` when nothing moved, so the cost is a small request.
- Hidden tab: no pulls, no rendering, no audio scheduling beyond a 2 s fade-out; the presence monitor emits `presence.end(hidden)`.
- Event delivery: events are queued locally and flushed every 5 s or when the queue reaches 10, and on `pagehide` via `sendBeacon`. Presence heartbeats are 30 s apart, so a lost flush costs at most 30 s of presence.

### 6.4 Clock handling

The server timestamps everything on receipt. The client keeps a `server_time − performance.now()` offset from each snapshot to schedule calls and reactions at absolute times. Client timestamps are stored for debugging ordering within a batch only; they never influence drift. Heartbeats claim at most 30 s each regardless of the client clock.

### 6.5 Failure handling

- API down: the client keeps rendering from the last snapshot, extends the call schedule locally, buffers events (bounded to 500, then drops the oldest presence heartbeats first), and retries with backoff. No error surface unless the user opens a system surface (settings) or the outage exceeds 10 minutes, at which point a matter-of-fact line appears in the top bar region: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- Session revoked or expired: the next API call returns 401; the client shows the matter-of-fact sign-in surface ("Your session timed out. Sign in again to keep watching.").
- Edge stale after revoke: the edge may serve one stale snapshot0 for a few seconds until KV converges; the API 401 that follows takes precedence. Acceptable and documented.

---

## 7. Frontend rendering pipeline

### 7.1 Stack and structure

- Canvas 2D for the scene. With at most seven procedural birds, four parallax layers, and a small particle pool, Canvas 2D hits 60 fps on the target laptop with headroom, avoids WebGL context loss handling, and keeps the critical bundle small. WebGL is not needed and not planned.
- Layers (each an offscreen canvas, composited per frame): sky, far foliage (parallax 0.25), perches + birds (1.0), near branch/leaves (1.3), effects (rain, drifting leaves, captions anchor points). Static layers are re-rasterized only on palette change (throttled to every 20 s, interpolated) or resize.
- `devicePixelRatio` capped at 2. Scene logical space is 1600 × 900 units mapped with "contain"; perch anchors are functions of aspect ratio (§7.7).
- Frame loop via `requestAnimationFrame`; stops on `visibilitychange → hidden`; on resume, if the gap exceeds 5 s, triggers a `resume` pull and re-seats birds at their current perches without animation (a bird cannot be mid-flight after a laptop wakes).

### 7.2 Procedural birds

Birds are drawn from parametric vector rigs, not sprite sheets. Each species defines silhouette paths (body, head, beak, tail, wing, eye) in the shared geometry module (used by the edge SVR render too). A pose is a small vector: `headYaw`, `headPitch`, `bodyTilt`, `fluff` (feather volume), `eyeOpen`, `wingLift`, `tailAngle`, `crouch`. Plumage comes from `phenotype.palette`; a light-direction shade derived from local time adds depth. Rendering cost is a few dozen path fills per bird.

### 7.3 Motion controller (idle micro-motion)

Each bird has a `MotionController` that turns the snapshot's `mood`, `temperament`, and `activity` into continuous pose changes:

- **Breathing**: sinusoidal body scale (0.6–1.2 Hz, amplitude by mood: drowsy slow and deep, alert quick and shallow). Never stops.
- **Activity generators** (seeded per device per bird, each with a duration and a blend-in/out): `preen` (head to wing, ruffle), `scan` (head sweeps with saccadic pauses), `tilt` (toward the last call's stereo position or an ornament), `shuffle` (weight reset), `fluff`, `rest` (crouch, eyes closed, occasional half-open), `watch` (fixed gaze on an offer). The scheduler picks generators by mood weights: wary → scan, back-lean; content → preen, shuffle; curious → tilt, watch; drowsy → fluff, rest-lite; alert → upright, quick scans; resting → rest.
- **Reactive hooks**: on a scheduled call from another bird, nearby birds tilt toward it with probability by curiosity band; on a leaf passing the bird's gaze cone, curious birds track it.
- **Noise**: all generator parameters are perturbed by 1/f noise so no two preens are identical.

The snapshot's `activity` field seats the controller in the right generator and phase on boot so the first frame shows the bird mid-action.

### 7.4 Transitions

- **Perch change**: hop for adjacent slots (0.4 s arc), short flight for zone changes (0.9–1.4 s, species-shaped path, wing cycle procedural). Arrival lands into a `shuffle`.
- **Mood change**: blend of generator weights and posture over 4–8 s. No pop.
- **Greeting** (§9.1): a sequenced pose script (glance, two-note call, step forward, longer call) driven by the same generators with a priority lane.
- **Settle**: palette ramps to the evening curve over 6 s; birds blend toward drowsy postures over 20 s; calls quiet (§8.4). Undo within 5 s reverses the ramp from wherever it is.
- **Fly-in** (empty-aviary → first bird only): a single arc from off-frame to the perch, 1.5 s, second bird 2–4 s later. This is the only entrance animation in the product and it runs once per account.

### 7.5 Palette and day/night

`palette(localTime)` produces sky gradient stops, foliage tints, and bird light direction from a 24-key curve (pre-dawn, dawn, morning, midday, afternoon, golden, dusk, night) interpolated continuously by the client's local clock. Settled overrides to the dusk key regardless of time. The design system supplies the actual color values; the curve structure and the AA-contrast constraint for any text over the scene (captions, focus rings) are fixed here.

### 7.6 Ornaments and weather rendering

- A fixed particle pool of 24: leaves (2–3 in flight at most in normal conditions), the occasional falling feather (1 per few minutes, tinted from a random bird's palette), rain streaks (up to 200 pooled line segments, only during `rain`), and wind-driven leaf gusts. Pool objects are reused; no allocation per spawn.
- Parallax: layer offsets follow a slow, seeded drift (not the cursor). No pointer parallax.
- Rain also darkens the sky key by intensity and adds a soft ripple on the pool offer if present.

### 7.7 Responsive layout

Perch anchor positions are functions of aspect ratio `r = width/height`:
- `r ≥ 1.6` (desktop): three zones spread horizontally, front slots low-center, back slots high and lateral.
- `1.0 ≤ r < 1.6` (tablet, landscape phone): zones compress; slot spacing shrinks to 0.8×.
- `r < 1.0` (portrait phone): zones stack diagonally (front low-left to back high-right) so seven birds remain fully visible with no cropping; bird scale reduces to 0.8×.
On resize, birds re-seat with a short hop; the scene never crops a bird (a CI visual test renders seven birds at 320×568, 390×844, 768×1024, 1366×768, 2560×1440).

### 7.8 Chrome (DOM)

- Top bar: five items (D1): account/settings, accessibility settings, field notebook, offer, settle. Icons with visually hidden labels; tooltips do not exist. Fades to 8% opacity after 4 s without pointer or keyboard activity; returns on activity; remains fully visible whenever it contains keyboard focus; is always present in the accessibility tree.
- Bird hit targets: a transparent DOM overlay with one `button` per bird positioned by the renderer each frame (cheap: seven style updates). They carry the naturalist accessible name and are the keyboard focus targets (§10.4). They have no visible styling; the focus ring is drawn by the renderer.
- Panels (notebook, offer tray, settings) open as side sheets over the scene without pausing it. The offer tray lists seed, song fragment (with a small library of ~8 short motifs, picked by name like "three notes, falling"), still pool. Unavailable kinds are shown at reduced opacity with no countdown.

### 7.9 Reduced-motion renderer

A second renderer mode, chosen by `prefers-reduced-motion` or the accessibility setting, sharing the scene graph and geometry:

- Birds are drawn from **pose keyframes** sampled from the same generators (so mood still shows) and cross-faded (1.2–2.0 s) with holds of 6–14 s. Breathing is replaced by a very slow, very small scale ease (0.05 Hz, 1%).
- Perch changes and the greeting's step-forward become cross-fades between the two positions with a brief overlap of both images at reduced alpha. Flight arcs are never drawn.
- Ornaments off; rain is a static, faint overlay that fades in and out; palette transitions run at one-third speed; settle ramps over 18 s.
- Captions and focus rings fade (opacity only). Calls play unchanged.
- Everything else (notebook, narration, drift, mood, offers) is identical.

This mode is a designed surface with its own visual regression baselines and its own review by the visual designer; it is a launch gate.

---

## 8. Audio pipeline

### 8.1 Graph

```
AudioWorkletNode "aviary-synth" (fixed pool: 16 voices) ──► per-bird bus (Gain → StereoPanner) ×7
                                                             │
ambient bus (procedural wind/rain noise, very quiet) ────────┤
                                                             ▼
                                              reverb send (small feedback-delay-network,
                                              generated at runtime, no impulse asset)
                                                             ▼
                                                  master Gain ► compressor (gentle) ► destination
```

The worklet owns synthesis and voice allocation; the main thread sends `noteOn` messages with a compact plan (arrays of `[t, f0, dur, timbre, env]` segments) and never allocates per call beyond the message. Per-bird buses are persistent nodes. Pan and mix bias follow perch zone (front: near-center, +0 dB; middle: −3 dB; back: −6 dB with more reverb send).

### 8.2 Call grammar

Per species, a motif library of 6–10 parametric motifs: `rise(n, interval, tempo)`, `fall(n, …)`, `trill(rate, dur, depth)`, `chip(sharpness)`, `whistle(dur, slide)`, `buzz(dur)`, `twoNote(interval)`, `pause(dur)`. A **call plan** is a sequence of motif instances with concrete parameters and a seed.

Per bird, a **signature** is derived once from `signature_seed`: pitch center offset (±3 semitones within the species range), tempo multiplier (0.85–1.2), timbre (harmonicity, brightness, breath amount), motif preference weights, and a characteristic ornament (e.g., always ends with a `chip`, or opens with a short `rise(2)`). The signature never changes.

Realization: `realize(grammar, signature, moodShaping, planSeed) → callPlan`. Mood shapes selection and parameters: wary → shorter, sparser, sharper; content → fuller and softer; curious → rising contours; drowsy → lower, slower, rarer; alert → chips and short calls; resting → silence (nightjar: a low, repeating churr at night). Every realization applies seeded micro-variation: pitch ±25 cents, timing ±8%, envelope ±10%, and a small chance of dropping or repeating a motif. Deviations from the signature are bounded (≤ 1 semitone, ≤ 15% tempo) so a bird stays recognizable through mood and drift.

Test: for 10,000 seeds per bird, no two realizations are parameter-identical, and every realization stays inside the signature envelope.

### 8.3 Recognizability across birds

At adoption and arrival, the species and signature of a new bird are chosen to maximize separation from the existing birds in a small feature space (pitch center, dominant motif class, tempo, timbre brightness). The engine rejects candidates closer than a minimum distance. A listening study (§14.3) validates that listeners identify the calling bird at ≥ 80% with five birds before the cap is raised beyond five.

### 8.4 Mixing behaviors

- **Listen-in**: focused bird bus +6 dB over 1.5 s; other birds −10 dB over 2.5 s (floor −14 dB; never muted); reverb send on the focused bird reduced (drier, nearer). Disengage ramps symmetrically. Focus changes cross-ramp without dipping to silence.
- **Settle**: master −8 dB over 5 s, call rate on the client's local extension halved; undo restores over 2 s.
- **Time of day**: master follows a gentle curve (night −6 dB); the nightjar's bus is exempt.
- **Hidden tab**: master fades out over 2 s and scheduling stops; on return it fades in over 2 s.
- Global loudness: normalized to a calm level (target −23 LUFS integrated for a busy aviary); a volume slider and mute live in accessibility/audio settings. Mute is not a drift input (D10).

### 8.5 Ambient and weather sound

Procedural only: filtered pink noise for wind (gain by wind intensity), band-limited noise with random droplet transients for rain. Both are very quiet under the calls and stop with the weather event.

### 8.6 Autoplay policy and fallbacks

Browsers block audio before a user gesture on a fresh origin. Behavior:

1. On boot, create the AudioContext. If its state is `running` (returning users on engaged origins, some desktop browsers), calls play from the first scheduled event.
2. If `suspended`, the aviary runs silently and captions are shown automatically for calls that would have played (the same rule as the no-WebAudio fallback), with no prompt, no "enable sound" button, no modal. The first pointer or key event anywhere resumes the context and fades the master in over 2 s; captions revert to the user's setting.
3. If AudioWorklet fails to load but AudioContext exists: a main-thread fallback synth using scheduled `OscillatorNode`/`GainNode` graphs with the same grammar (lower polyphony, 8 voices). This is code, not assets.
4. If AudioContext is unavailable or permanently denied: graceful silence, captions on by default. The setting reads, in matter-of-fact voice, "Sound isn't available in this browser. Captions are on."
5. iOS: the hardware silent switch mutes WebAudio and cannot be detected. Captions are prominently reachable in accessibility settings; the visitor and host builds behave identically.

### 8.7 Memory and performance rules

Fixed voice pool, fixed automation buffers in the worklet, per-bird buses created once, no `AudioBuffer` allocation after boot except the one-time reverb FDN state. The worklet processes at 128-frame quanta with ≤ 16 voices × 2 oscillators + noise, comfortably under budget on the target laptop. Soak test in §13.6 monitors `AudioContext.baseLatency`, dropouts (worklet reports underruns), and heap.

---

## 9. Interaction flows (client/server contract)

### 9.1 Return-greeting

Trigger: `session_start` (navigation) or `visibility` return, rate-limited to one greeting per 120 s per device (tab-switch churn must not turn into a twitchy bird).

Absence bucket (client, from `last_presence_ended_at` in snapshot0, server-verified on the `session_start` event): `short` < 20 min; `medium` 20 min–24 h; `long` 1–7 days; `very_long` > 7 days.

Greeter selection (server-ranked at each tick into `greeting_candidates`):
```
score = 1.0·boldness + 0.5·warmth + moodBonus(alert +0.3, curious +0.2, content 0, drowsy −0.4, wary −0.5, resting −1.0) + noise(σ = 0.18)
```
The noise term is sized so the shyer of two birds greets first on roughly 15–25% of days when traits are close, which is what makes "pip greeted before wren today, first time this week" a real event. `resting` birds never greet unless every bird is resting, in which case the nightjar (if present) or nobody greets, and the user sees a sleeping aviary.

Forms by bucket and bird: `glance` (look up from the current activity, hold 1.5 s), `two_note` (a short two-motif call plus a head-turn), `approach` (a step to the front slot if the bird is bold, otherwise a lean), `approach_call` (approach then a longer call; a second bird with high warmth responds 1–3 s later per the candidates list), `response_call`. The client fires the top candidate within 1 s of first paint; a second candidate is fired only for `long`/`very_long` with a 1.2–3.5 s randomized stagger; never more than two birds greet, never in unison.

The client reports `{greeter_bird_id, form, absence_bucket}` in `session.start`. The observer uses it for greeting-order facts. No textual welcome, no "gone X days", nothing but the bird.

### 9.2 Presence monitor (client)

A single module, `PresenceMonitor`, evaluates every 5 s:

```
visible = document.visibilityState === 'visible'
focused = document.hasFocus()
active  = now − lastActivityAt ≤ W    // pointermove, pointerdown, keydown, wheel, touchstart
present = visible && focused && active
```

`W` comes from `/aviary/config` (initial 4 minutes; calibration leans long because watching without moving is the product). Transitions emit `presence.start` / `presence.end` with reasons; while present, `presence.heartbeat` every 30 s. `pagehide` and `beforeunload` flush `presence.end(unload)` via `sendBeacon`. Settle emits `presence.end(settle)` and stops heartbeats until the user re-engages (any click or key), which starts a new segment.

Unit tests cover all three single-condition and three two-condition failure cases, the heartbeat cadence, the activity-window expiry, and multi-tab behavior. The visitor build never instantiates the monitor.

### 9.3 Listen-in

Pointer: tap/click a bird → engage; click the same bird, another bird, or empty scene → disengage/switch. Keyboard: Enter on a focused bird engages; Escape disengages; moving focus out of the scene disengages. The mix change is client-local and immediate (§8.4); the client also emits `listen_in.start {bird_id}` and `listen_in.end`. The focused bird gets a `tilt`-toward-viewer bias in its motion controller and, in the observer's inputs, the listen-in seconds. Listen-in state is not synced across devices (D7).

### 9.4 Offer

From the top bar tray only (never by clicking a bird). Client posts to `/aviary/offers`; the response includes the reaction plan with absolute times; the client renders the offer object (seed at the front of the scene, pool as a reflective surface at the front edge, song as a soft motif from the ambient bus) and the reactions (approach paths, drink/bathe poses, a joining or countering call). Unavailable kinds are simply unavailable in the tray while one of that kind is active; there is no countdown and no per-bird cooldown display. Reactions are narrated and, for calls, captioned.

### 9.5 Settle

Top bar item or its keyboard shortcut. Client: emits `settle`, starts the 6 s palette ramp and 5 s audio ramp, arms the 5 s undo window (any click or key anywhere reverses and emits `settle.undo`), then stops presence. Server: `settled_at` set by the tick from the event; cleared on the next `session.start` from any device or on re-engagement events (`listen_in.start`, `offer.made`, `presence.start`), all of which count as "actively re-engages".

### 9.6 Field notebook

A side sheet listing entries newest first, cursor-paginated, virtualized (DOM window of ~30 entries; entries beyond 200 are evicted from memory and re-fetched on scroll). Entries show a weekday/date line in the notebook's own lowercase style and the prose. No edit, delete, annotate, share, or export from this surface (the account export includes entries). Opening the notebook does not pause the scene or the audio.

### 9.7 Adoption

New account → `/adopt`: the two arrived birds are shown (species silhouettes, still poses) with the naturalist line "two birds have arrived." and two name fields pre-filled with suggestions from a curated list of ~200 short names (rotating, seeded). Submit → the aviary opens on the empty quiet field, then the first bird flies in, the second follows. Names are editable later in settings → birds.

### 9.8 Visits (host and visitor)

Host: settings → visits: invite by email (matter-of-fact copy), list of outstanding/active invites with revoke, visit log (email, date, approximate duration), and the notification toggle (off by default): "Email me when someone visits." The email, when enabled, is one line in matter-of-fact voice, at most one per visitor per day.

Visitor: the link opens the visitor shell (same scene, `mode: 'visitor'`): no offer, settle, notebook, or account items; the top bar has only the accessibility settings item (captions, reduced motion, volume are the visitor's own device preferences, stored in `localStorage`). Listen-in is disabled (the visitor cannot interact); keyboard focus on birds still works for narration. Polling as for hosts; a 410 on any pull shows "This visit is no longer available."

---

## 10. Accessibility surfaces

Accessibility is a designed surface here, built by the same people building the visual one, in the same sprints. Each item below has a launch gate in §14.

### 10.1 Screen-reader narration

- A single `aria-live="polite"` region (`role="log"`, `aria-relevant="additions"`) holds the running narration. The scene container has `aria-label="the aviary"` and `aria-roledescription="aviary"`; the notebook and settings are separate landmarks.
- Prose is generated **client-side** by the shared prose grammar from the current snapshot plus local scene state (which bird is preening right now, what the light is doing, whether it is raining). Same lexicon and rules as the notebook, so the voice is continuous across surfaces. Species descriptors are used where helpful ("a small grey bird") alongside names.
- Cadence: idle narration every 30–60 s (seeded jitter), skipped if nothing changed since the last line. User-initiated events insert immediately as observations: a greeting ("pip looks up from the front rail and calls twice."), an offer reaction ("wren drops to the pool and drinks."), settle ("the light softens toward evening. calls quieten."). Narration is never a state list and never includes trait words or numbers.
- Narration cadence setting: `off | slow (60–90 s) | normal (30–60 s)` in accessibility settings; default `normal`. Screen-reader users who want less get less; nobody gets a firehose.
- Visitor mode narrates too.

### 10.2 Reduced-motion mode

Specified in §7.9. Activated by `prefers-reduced-motion: reduce` (default follows the system) or the explicit setting `system | on | off`. The visual designer signs off on this register as its own aesthetic before beta.

### 10.3 Captions for calls

- Setting `captions: on | off`, default `off` unless audio is unavailable or blocked, in which case captions are shown until audio is available (§8.6).
- Caption text is produced by `caption(callPlan)` in the shared grammar from the realized plan, so it matches what played: motif classes and parameters map to phrase fragments ("a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call"), with a location fragment for back-perch birds ("from the back perch") and a mood-derived softener ("softly", "sharp"). The template set has ≥ 3 variants per motif class; a seeded pick avoids repetition.
- Rendered as small DOM text anchored near the calling bird (position from the renderer), fading in with the call and out ~1.5 s after, on a soft translucent backdrop that guarantees AA contrast against any sky key. In reduced motion they fade only. Captions are `aria-hidden` (narration covers the screen reader; double-speaking is avoided).

### 10.4 Keyboard navigation and focus

- Tab order: top bar items (account, accessibility, notebook, offer, settle) → the aviary scene group → nothing after (the scene is the last stop; Shift+Tab goes back).
- Entering the scene focuses the first bird (front zone, left to right, then middle, then back). Arrow keys move between birds in that order (Left/Right; Up/Down move between zones). Enter toggles listen-in. Escape exits listen-in, or exits the scene group to the top bar if not listening in.
- Shortcuts (documented in accessibility settings): `o` opens the offer tray, `n` the notebook, `s` settle, `,` settings, `a` accessibility. All panels trap focus while open and restore focus on close.
- Focus indicator: a soft, 3 px, high-contrast dual-tone outline (light inner, dark outer) drawn by the renderer around the focused bird, visible against every sky key and at night; a matching CSS outline for chrome. Never suppressed for pointer users.
- The top bar shows fully whenever it has focus; its fade never hides a focused control.

### 10.5 Contrast and copy

All user copy (chrome labels, settings, account and error surfaces, captions, displayed narration) passes WCAG AA; the design system specifies ratios. The offer tray and notebook use the naturalist voice; settings, account, sign-in, sync errors, unsupported-browser, and accessibility settings use matter-of-fact voice. A string-table lint tags every string with its surface and voice and rejects mismatches (§13.5).

### 10.6 Audio-off parity

A user with sound off and captions on receives the same call events, the same chorus timing, the same greeting, and the same narration. The reduced-motion user receives the same calls. No surface is the "full" one.

---

## 11. Performance budgets and observability

### 11.1 Budgets (all enforced in CI and monitored in production)

| Budget | Target | Hard limit | Enforcement |
|--------|--------|------------|-------------|
| Critical JS chunk (boot + renderer + geometry + species + palette) | ≤ 90 KB gz | 150 KB gz | `size-limit` in CI, PR fails above limit |
| Total initial JS at first paint | ≤ 350 KB gz | 2 MB gz (PRD cap) | same |
| HTML shell incl. inlined snapshot0 + first-frame SVG + critical CSS | ≤ 30 KB gz | 60 KB gz | edge build check |
| Time to first bird visible (mid-tier mobile, 4G, cold cache) | ≤ 400 ms | 500 ms | Lighthouse CI + WebPageTest profile "Moto G-class / 4G"; synthetic fleet |
| Idle frame time on 5-year-old mid-range laptop (30-min session) | p95 ≤ 12 ms | 16.6 ms | Playwright perf run on a pinned low-power VM profile; RUM frame histograms |
| Heap growth over 30 minutes | 0 ± noise | ≤ 5 MB slope | nightly soak (§13.6); PR-level 5-minute accelerated soak |
| Snapshot size | ≤ 6 KB gz | 8 KB gz | schema test with seven birds and a full call schedule |
| Tick compute (seven birds) | ≤ 1 ms | 20 ms | engine benchmark in CI |
| Tick end-to-end latency (claim → commit) p99 | ≤ 500 ms | 5 s alarm | production alarm (PRD) |
| Event-log lag (oldest unconsumed event age for hot aviaries) | ≤ 90 s | 5 min alarm | production alarm |

Chunking: `critical` (as above), `audio` (worklet loader + grammar), `chrome` (Preact + top bar + tray), `notebook`, `settings` (account, accessibility, visits), `adopt`, `visitor`, `fallback-synth`. `audio` and `chrome` are prefetched immediately after first paint; the rest on demand.

### 11.2 What we measure (aggregate only)

Server: RED metrics per route; tick latency histogram, backlog count, catch-up length histogram, tick errors; event-append rate and batch sizes; edge publish lag; magic-link send/consume funnel counts; email provider errors; export job durations; deletion job counts; DB saturation.

Client RUM (sampled 10%, no account or session identifier, no bird fields): time to first bird; SVG→canvas takeover delta; long-frame count and frame-time histogram; snapshot pull latency; audio outcome (`running_at_boot | resumed_on_gesture | worklet_fallback | unavailable`); worklet underruns; JS error signatures (sanitized, no payloads); reduced-motion / captions / narration-cadence usage counts (feature usage, no identity); anonymized session-duration histogram; anonymized presence-duty-cycle histogram (the "release canary" for I3 regressions: a sudden shift in the population's presence duty after a release indicates a monitor bug).

Synthetic: a Playwright fleet on synthetic accounts from five regions every 10 minutes measuring first-bird time, audio init, and a scripted greeting; alarms on regression.

Drift canary: a handful of synthetic accounts driven by scripted presence profiles in production (regular, sporadic, absent, binge). Their trait curves are compared nightly to the calibration bands. These are synthetic accounts; their data is engineering data, not user data.

### 11.3 What we deliberately do not measure

Per-account or per-bird anything in telemetry: no presence totals, no interaction counts, no trait values, no mood distributions, no visit counts per aviary, no retention cohorts keyed by account. No per-account experiment assignment dimensions. No cross-account aggregates of interaction behavior ("average drift", "most-used offer"), even anonymized, because they are the first step to a data product the privacy commitment forbids. Calibration questions are answered with synthetic profiles, not user data.

### 11.4 Enforcement of the telemetry boundary

- The OpenTelemetry SDK is wrapped: attribute keys must be in an allowlist (`route`, `status`, `region`, `browser_family`, `outcome`, `bucket`, …); unknown keys are dropped and the drop is counted; a unit test asserts every metric definition in the repo uses allowed keys only.
- Logging: a redaction layer strips event payloads, bird fields, and any key containing `email`, `name`, `trait`, `mood`. Account UUID is permitted in operational logs (I8) because "is this account having errors" is allowed; interaction content is not.
- The analytics/metrics store has no credentials to the simulation database. `readonly_ops` cannot read birds, state, events, or notebook (§3.6).
- Any new dashboard or query touching `interaction_events`, `birds`, `aviary_state`, or `notebook_entries` requires a privacy review. There is no BI connector to the sim database.

### 11.5 Alarms (initial set)

Tick p99 > 5 s (PRD); tick backlog > 2% of hot aviaries overdue by > 3 ticks; event-log lag > 5 min; API 5xx > 0.5% for 5 min; edge KV write failures > 1%; magic-link consume rate collapse (send/consume ratio drop > 50% over 1 h, indicating deliverability trouble); synthetic first-bird time > 500 ms in any region for 3 consecutive runs; RUM audio `unavailable` share jump > 2× baseline; heap soak failure.

---

## 12. Accounts, privacy, and security engineering

### 12.1 Magic-link sign-in

- Token: 256-bit random; only the SHA-256 hash is stored; 15-minute expiry; single use (`consumed_at` set atomically with `UPDATE … WHERE consumed_at IS NULL RETURNING`); bound to the requesting blind index.
- Consume signs in the browser that opened the link. If the user requested on a laptop and opened on a phone, the phone is signed in; the laptop's page reads "Check your email for a link. Open it in this browser to sign in here." (matter-of-fact). No cross-device polling in v1.
- Rate limits in §4.1; a generic 202 for all requests prevents account enumeration.
- New-account creation happens on first consume; an adoption flag routes to `/adopt` until two birds exist.

### 12.2 Sessions

Opaque 256-bit tokens, hashed at rest, httpOnly Secure SameSite=Lax cookie, sliding 180-day expiry, `ua_family` recorded for the session list. Revocation deletes the edge `session:` key and marks `revoked_at`; the API rejects on the next call. Sign-out on one device never affects others.

### 12.3 Email handling

Email is stored once, envelope-encrypted, with a blind index for lookup. Decryption happens in the auth module for sending links and in the account page for display. The invite feature stores the visitor's email the same way. Lint: no other module imports the decryption helper; no log, metric, queue key, or URL contains an email.

### 12.4 Export

Enqueued job builds a JSON document: account settings (no email plaintext beyond the destination), birds (id, name, species, adopted_at, current personality vector, current mood), notebook entries, visit invites (visitor emails included, since the host entered them), created_at. Stored privately with a 24-hour signed URL, emailed to the verified address, 1/day. The document labels the vector as engine values and no product surface renders it (D4).

### 12.5 Deletion

`POST /account/delete` sets `deletion_requested_at` and `hard_delete_after`. During the window, sign-in works and every signed-in page shows a matter-of-fact line with "I changed my mind" (this is a system surface, not an announcement). Ticks continue (cold) so a recovered aviary is continuous. A daily purge job, under the `purger` role, deletes all rows for accounts past `hard_delete_after`, the edge keys, queued mail, export artifacts, and writes a tombstone `(account_id, purged_at)` kept for 60 days so that a restored database backup re-purges. Backups are retained 35 days maximum; the retention document is linked from the privacy policy.

### 12.6 Visitor security

Invite tokens: 256-bit, hashed; 30-day expiry; visitor sessions bound to the invite; every pull re-validates `revoked_at` and `expires_at`. Visitor sessions have no account and no write scopes. A visitor's browser stores only its accessibility preferences.

### 12.7 Threat notes

There is little to steal, but there is something to corrupt: a user's own drift. Idempotent events, per-tick caps (60 s presence, 60 s listen-in per bird), and server-side cooldowns bound what any client can do to its own aviary; no client can affect another aviary. Standard hygiene applies: CSP with nonces (inline snapshot JSON and SVG are nonce-tagged), no third-party scripts, SRI on chunks, HSTS, dependency audit in CI.

---

## 13. Testing, calibration harness, and CI gates

### 13.1 Engine unit and property tests

- Drift: non-negativity for all inputs (property); monotonic trait series over random event streams; zero drift under visitor-only activity; per-tick caps; saturation behavior; determinism across Node 20/22 and Linux/macOS runners; catch-up ≡ sequential (§5.1).
- Mood: persistence across ticks; dawn reset of `recent`; contagion; nightjar night activity; no `resting` greeter.
- Perch: no overflow of slots; hysteresis; the seven-bird layout has a valid assignment always.
- Calls: schedule horizon always ≥ 120 s; chorus formation only with ≥ 2 eligible birds; realization uniqueness and signature envelope (§8.2); caption generation for every motif class.
- Observer: sparsity bounds under 10,000 synthetic days with varied activity; landmark lane; no user-behavior facts (a test feeds heavy presence and asserts no entry mentions visits, days, or the user).

### 13.2 API and sync tests

- Role/grant tests: the `api_rw` role cannot update traits or `aviary_state` (expect permission errors); event table has no update/delete path.
- Trigger tests: trait decrease rejected; bird id immutable.
- Concurrency: two devices offering simultaneously; offer during a tick; catch-up during a pull; duplicate event batches; out-of-order snapshot responses.
- Revocation: visitor pull after revoke → 410 within one pull.

### 13.3 Drift calibration harness (`tools/calibrate`)

Runs the engine over simulated weeks with scripted user profiles and reports trait curves against the calibration bands:

| Profile | Definition | Expected (mid-range trait) |
|---------|------------|-----------------------------|
| regular | 15 min presence/day, 1 listen-in, occasional offer | +0.04..0.07 at 7 d; +0.12..0.18 at 21 d |
| light | 5 min/day, 4 days/week | +0.02..0.04 at 7 d; +0.06..0.10 at 21 d |
| binge | 6 h on day 1, then absent | ≤ +0.06 total; no visible change on day 2 |
| absent | 2 weeks away after a regular month | no decrease; greeting-first rate unchanged; call rate unchanged |
| visitor-only | a visitor watches 2 h/day | zero drift |

The harness also produces observable-behavior curves (front-perch share, greeting-first share, call rate) so "visible after three weeks" is asserted on behavior, not only on numbers. Calibration constants change only through a versioned config with a harness report attached to the PR. Recalibration never rewrites existing traits (I2); it changes future deltas.

### 13.4 Client tests

- `PresenceMonitor` matrix (§9.2).
- Renderer: golden-image tests for each species × mood × zone; responsive no-crop tests (§7.7); SVG-first-frame vs. canvas-first-frame pixel diff ≤ 1%; reduced-motion baselines.
- Audio: worklet unit tests on plan → sample buffers (offline AudioContext) including "no two realizations identical"; underrun test under 16 voices; fallback path when the worklet module fails to load; suspended-context path shows captions.
- Accessibility: axe on every surface; keyboard walkthrough script (tab order, arrows, Enter/Escape, shortcuts, focus restore); narration region receives lines at the expected cadence and never contains banned tokens; captions match the plan.

### 13.5 Voice linter

A tool over the string table, the prose templates, and 10,000 generated notebook/narration/caption samples:

- Naturalist surfaces: lowercase only (names included), present tense verbs from an approved list, no `you`/`your`, no exclamation marks, no numerals except weekday/date lines, banned tokens (`welcome`, `back`, `streak`, `visited`, `days in a row`, `level`, `achievement`, `unlocked`, `happier`, `points`), and no trait names.
- Matter-of-fact surfaces: sentence case, no naturalist verbs (`perch`, `settle` used as product verbs), no bird names.
- Every string is tagged with its surface; untagged strings fail the build.

### 13.6 Soak and performance CI

- Nightly: a real 30-minute Playwright session on a throttled CPU profile with seven birds, rain, one offer of each kind, listen-in cycles, notebook scrolling; records heap samples every 30 s; fails on slope > 5 MB/30 min or any worklet underrun burst; also asserts frame p95.
- Per PR: 5-minute accelerated version plus bundle budgets, Lighthouse CI on the mobile profile, and the engine benchmark.

### 13.7 Manual QA before each launch gate

Screen-reader sessions with VoiceOver (Safari/macOS, iOS) and NVDA (Firefox/Chrome, Windows); reduced-motion review with the designer; listening review of ten random aviaries for uncanniness; a "spell-break" walkthrough by two people who did not build the feature, checking for any announcement, spinner, canned cue, or gamification residue.

---

## 14. Rollout

### 14.1 Stages

| Stage | Who | Config | Exit criteria |
|-------|-----|--------|---------------|
| Internal alpha | team + friends, ~50 accounts | `max_birds = 2`, arrivals off, visits off, notebook on | All invariant tests green; first-bird time and frame budgets met on target devices; screen-reader and reduced-motion reviews signed off; spell-break walkthrough passed |
| Closed beta | ~500 invited accounts | `max_birds = 3`, arrivals on (third bird at the 75-day mark, which most beta aviaries reach after launch), visits on | Two weeks of clean tick/latency alarms; drift canary inside bands; zero privacy-boundary lint violations; email deliverability ≥ 99%; memory soak green for 14 nights |
| Open launch | public sign-up | `max_birds = 3` | Synthetic fleet green across regions for one week; on-call runbooks exercised |
| Ramp to 5 | all | `max_birds = 5` | Listening study (§14.3) ≥ 80% identification at five birds; oldest aviaries reaching the 150/240-day arrivals |
| Ramp to 7 | all | `max_birds = 7` | Listening study ≥ 80% at seven birds with the recognizability separation rule; chorus mix review |

Because arrivals are age-gated, the birds-per-aviary ramp is naturally slow; the config cap exists so that a recognizability problem found late can be held at a lower count without touching any bird that already exists.

### 14.2 Configuration and flags

All tunables (presence window `W`, heartbeat interval, drift constants, arrival schedule, weather rates, observer budget, greeting rate limit, `max_birds`) live in a versioned server config with a change log. Flags are global or percent-of-accounts by UUID hash for operational rollout only; there is no per-account experiment assignment dimension in telemetry (§11.3).

### 14.3 Listening study

Before each bird-count ramp: 20+ participants, an aviary with N birds, 40 trials each of "which bird just called" after a two-day familiarization with the aviary (participants use test accounts). Pass at ≥ 80% mean identification. The study kit lives in `tools/listening`.

### 14.4 Day-one instrumentation

Everything in §11.2, the drift canary accounts, the synthetic fleet, the alarm set in §11.5, bundle and perf budgets in CI, the nightly soak, and the presence-duty histogram as the release canary for the presence monitor.

### 14.5 Runbooks (written before beta)

Tick backlog recovery (scale workers; sweeper interval); edge KV staleness; email provider failover; a bad drift-config rollback (future deltas only; never rewrite traits); a suspected personality-vector corruption (freeze `sim_rw`, snapshot the DB, compare against the previous day's backup; restoration only ever raises, never lowers, per I2, and requires the bird-integrity review); account purge failures.

---

## 15. Risks

| Risk | Why it matters | Mitigation |
|------|----------------|------------|
| **Drift calibration off** (too fast → Tamagotchi feel; too slow → screensaver) | The product's central promise sits in a narrow band; failure is invisible to unit tests | Harness with behavior curves (§13.3); versioned constants; production drift canary; daily saturation; high-gain behavior mappings so a small trait move is visible |
| **Presence inflation** (a monitor bug counts background tabs) | Corrupts drift population-wide, silently | I3 module with the failure-case matrix; per-tick cap; presence-duty histogram as a release canary; sendBeacon `presence.end` on unload; heartbeat-gap auto-close on the server |
| **Sync correctness** (double-consumed or skipped events; tick/offer races) | Lost or duplicated drift, unreachable by users | Consumed-seq stored in the tick transaction; row locks; idempotent events; catch-up ≡ sequential property test; role grants |
| **Non-determinism in catch-up** (float differences across runners) | Cold and hot aviaries diverge; replay tests flap | Fixed-point trait and mood math; counter-based PRNG; golden replays in CI on two Node versions |
| **Audio uncanniness** (calls sound like a synthesizer, or too similar between birds) | Audio is the affective spine; canned or synthetic-sounding calls break the spell | Timbre design sprint with breath noise, formant-like filtering, and micro-variation; listening reviews every sprint; recognizability separation rule; listening study before ramps |
| **Autoplay policy** | First-frame "calls already audible" is not always possible | Silent-with-captions until first gesture; 2 s fade-in; no prompt; measured via RUM audio outcome |
| **First-frame budget on 4G** | Above 500 ms the aviary reads as loading | Edge-rendered SVG first frame; critical chunk ≤ 90 KB; snapshot0 at the edge; Lighthouse CI; synthetic fleet |
| **SVG→canvas takeover pop** | A visible discontinuity is an entrance animation by another name | Shared geometry module; pixel-diff test; takeover on the same frame with matching poses |
| **Memory growth** | 30-minute sessions on laptops; audio graphs are a classic leak | Fixed pools (voices, particles, automation buffers); virtualized notebook; nightly soak with slope assertion |
| **Accessibility regressions** | Narration cadence creep, caption/plan mismatch, focus ring lost on a palette change | axe + keyboard scripts in CI; caption-from-plan test; contrast test of the focus ring against every sky key; manual SR sessions per gate |
| **Announcement creep** ("just one toast") | Violates the product's central register | No notification component in the bundle; string lint; spell-break walkthrough at each gate |
| **Notebook voice drift** (entries read like logs, or mention the user) | The notebook is where the voice is most concentrated | Template review by the writer; voice linter on 10k samples; fact extractor forbids user-behavior facts |
| **Magic-link deliverability** | It is the only door | DKIM/SPF/DMARC; second provider failover; send/consume funnel alarm |
| **PII leakage via email as identifier** | Compliance findings, impossible to retrofit | Synthetic UUID everywhere; blind index; lint on `email` in logs/metrics/keys |
| **Privacy boundary leakage** (a helpful dashboard) | Converts the relationship into a data product | Attribute allowlist; credential separation; privacy review on any sim-table query; no BI connector |
| **Tick cost at scale** | Always-on ticks for a million cold aviaries | Hot/cold tiers with exact catch-up; batched sweeper; inline catch-up bound |
| **Timezone and clock edge cases** (travel, DST, wrong device clock) | Dawn resets and night states at the wrong hours | Server uses the most recent client-reported IANA zone; DST via the zone; server timestamps only |
| **Safari/iOS WebAudio quirks** (context suspends on background, silent switch) | Silent aviary without explanation | Resume on visibility + gesture; captions reachable; matter-of-fact note in audio settings |
| **Seven-bird recognizability** | The cap is empirical; the chorus may blur earlier | Config cap; listening study gates; separation rule at arrival |
| **Newcomer discoverability** (no announcement means some users never name the bird) | A nameless bird for months looks like a bug | Notebook landmark entry; naming lives in settings → birds; unnamed birds are still fully alive, so nothing is lost |

---

## 16. Decision log (ambiguities resolved)

| # | Ambiguity | Decision | Rationale |
|---|-----------|----------|-----------|
| D1 | `aviary_layout.md` lists four top-bar items "nothing else"; `interactions.md` and `accessibility_perf.md` say settle is triggered from the top bar | Five top-bar items: account/settings, accessibility, notebook, offer, settle | Two files require settle in the top bar; the scene must stay chrome-free; the bar stays sparse and fades |
| D2 | Mood set "finalized in implementation"; PRD uses "settled" for both lighting and night birds | Moods: `wary, content, curious, drowsy, alert, resting`; `settled` is the aviary lighting state only | Avoids one word meaning two things in code and prose |
| D3 | Tick "runs whether or not any client is connected" vs. cost | Logical tick every 60 s for all aviaries; hot aviaries ticked live, cold ones caught up in batches and on read, bit-identically | Preserves the architectural property exactly while keeping cost proportional to use |
| D4 | Export "includes current personality vectors" vs. "never exposed numerically, not at any version" | Export includes them as labeled engine values; no product surface renders them | The export is a data-portability artifact the PRD explicitly specifies; the exposure rule targets product surfaces |
| D5 | How the client renders personality-shaped behavior without receiving traits | Snapshot carries a phenotype projection (palette, size, temperament band, call signature seed) and server-made decisions (perch, greeting candidates, call schedule) | Keeps raw numbers out of network payloads a user could inspect |
| D6 | How a new-bird "offer appears in the user's flow" without announcing | The newcomer arrives at dawn on the back perch as a real bird; the notebook records it; naming lives in settings → birds; no decline; a `new_arrivals_paused` setting stops future arrivals | "Birds that arrived, not birds the user chose"; no announcement surface; minimal chrome |
| D7 | Is listen-in canonical across devices? | Device-local | It is an audio focus for the person listening on that device; syncing it would surprise the other device |
| D8 | Settle persistence with multiple devices | Canonical flag; cleared by the next `session.start` from any device or any re-engagement event | Settle is a goodbye; opening the aviary anywhere is a return |
| D9 | Bird names in lowercase prose | Names are stored as entered and rendered lowercase in notebook, narration, and captions; settings show them as entered | Voice consistency in naturalist surfaces; the account surface is matter-of-fact |
| D10 | The brief mentions muting as one of the "small ways you show up"; the engine's drift inputs do not list it | Mute is not a drift input in v1 | No-punishment principle; muting for a meeting must not count against anyone; the engine file is the authority on inputs |
| D11 | Presence activity window value | Initial `W = 4 min`, server-configurable, calibrated during beta toward the longer side | PRD says lean long |
| D12 | Offer cooldown semantics | Per-bird 4-minute cooldown shapes reactions (a cooled-down bird ignores); one active offer per kind at a time gates the tray | Matches "per-bird cooldown" while keeping the tray from being a button to mash |
| D13 | Greeting on tab-switch churn | One greeting per 120 s per device | Prevents twitchy birds without touching the greeting's variety |
| D14 | Push channel | v1 polls every 60 s while visible; no WebSocket/SSE | The tick is 60 s; polling matches it and is cheaper to operate |
| D15 | Visit notification channel | Email only, opt-in, ≤ 1 per visitor per day | The product never pushes; the social file allows an opt-in toggle; email is the only channel the product has |
| D16 | Cross-device magic-link consumption | Signs in the browser that opened the link; the requesting page explains this in matter-of-fact voice | PRD: "signs them in on that browser" |
| D17 | Loading-state audio | Silent until the AudioContext is allowed; captions shown meanwhile | Browser policy; no prompt is consistent with no announcements |
| D18 | Seven-bird slot layout | Nine slots (3/3/3), diagonal stacking under portrait aspect | Leaves movement room; no cropping at any viewport |

---

## 17. Work breakdown and sequencing

Team assumption: 8–10 engineers (2 engine, 2 client-render, 1 audio, 2 backend/infra, 1 accessibility+chrome, 1 QA/tools), plus a visual designer and a writer for prose templates. Estimates are in working weeks; phases overlap after Phase 0.

**Phase 0 — Foundations (2 weeks).** Monorepo, CI with budgets and lints (voice, telemetry allowlist, banned identifiers), Postgres schema and roles/triggers (§3), engine package skeleton with fixed-point math and PRNG, species definitions v0, shared geometry v0, edge shell skeleton, design-system palette curve, string table with surface tags.

**Phase 1 — Engine alpha (4 weeks, engine team).** Drift, attention low-pass, saturation, mood model, perch choice, weather, call scheduling, greeting ranking, arrivals, offer resolution, observer with sparsity budget; calibration harness and first report; determinism tests. Deliverable: `tick()` passing §13.1 with the harness inside bands.

**Phase 2 — Scene renderer (5 weeks, render team, overlaps Phase 1).** Canvas layers, procedural bird rigs for six species, motion controllers, transitions, palette, ornaments, responsive layout, reduced-motion renderer, DOM bird overlay and focus ring, quiet-field and empty-aviary states, fly-in. Deliverable: a scene driven by fixture snapshots at 60 fps on the target laptop with golden images.

**Phase 3 — Audio (5 weeks, audio engineer with the engine team for the grammar).** Worklet synth and voice pool, motif library, signature derivation, realization with variation, caption generator, buses and mixing behaviors, ambient/weather sound, autoplay handling, fallback synth, offline tests. Deliverable: listening review passes on two-, three-, and five-bird fixtures.

**Phase 4 — Backend and sync (4 weeks, backend team, overlaps Phases 1–3).** Auth (magic link, sessions, email change), API routes, event log, hot/cold tick workers with SKIP LOCKED claims, sweeper, inline catch-up, edge publisher, snapshot0 and edge SVG first frame, export and deletion jobs, mailer, telemetry wrapper, alarms. Deliverable: two devices reading one aviary end-to-end; role and concurrency tests green.

**Phase 5 — Interactions and presence (3 weeks, client + backend).** Presence monitor, event queue with beacon flush, return-greeting, listen-in, offer tray and reactions, settle with undo, keepalive/visibility/resume pulls, error surfaces. Deliverable: §9 flows complete against the real API.

**Phase 6 — Notebook, narration, chrome, accessibility (4 weeks, accessibility+chrome engineer, writer, render team).** Notebook sheet with virtualization, prose templates for observer/narration/captions/greeting, narration region and cadence, keyboard model and shortcuts, settings and account surfaces in matter-of-fact voice, adoption flow, unsupported-browser page. Deliverable: axe and keyboard scripts green; SR review passed.

**Phase 7 — Visits (2 weeks, backend + chrome).** Invites, visitor sessions, visitor build mode, visit log, revocation, opt-in email. Deliverable: revoke-to-410 within one pull; visitor produces zero events (test).

**Phase 8 — Hardening (3 weeks, everyone).** Soak tests, first-bird budget on real devices, synthetic fleet, runbooks, drift canary accounts, spell-break walkthrough, listening study kit, privacy review of every query and dashboard. Deliverable: alpha exit criteria (§14.1).

Then the staged rollout in §14. Total to internal alpha: roughly 14–16 weeks with the overlaps above; closed beta about four weeks after; open launch when beta exit criteria hold for two weeks.

Definition of done for the whole plan: every invariant in §0 has a passing test or a structural enforcement in the repository, every budget in §11.1 is enforced in CI and observed in production, the accessibility gates in §10 are met before beta rather than after, and the twelve-line spell-break checklist (no spinner, no entry animation, no toast, no welcome text, no streak, no counter, no number for a trait, no recorded audio, no cropped bird, no announcement of a visit, no editable notebook, no visitor drift) is empty of findings on the launch build.
