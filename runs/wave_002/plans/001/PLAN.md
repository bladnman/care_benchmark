# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable engineering program. It is written for a
frontier engineering team that has not read the PRD: every load-bearing rule is restated
here as an engineering decision, an invariant, or a test. Where the PRD is ambiguous, this
plan makes a defensible call and flags it in **§14 Interpretations and defensible calls**.

The plan does not implement the product. It specifies what to build, how the pieces fit,
what to measure, and what must never be built.

---

## 1. Scope

### 1.1 V1 ships

- Single-user accounts with email magic-link sign-in (no passwords, no SSO).
- One canonical aviary per account; two starter birds at adoption; hard cap of seven.
- Server-side simulation tick (~60s idle cadence, fast-tick on interaction) advancing
  personality drift, mood, weather, and notebook generation whether or not any client is
  connected.
- Multi-device sync as a property of the architecture (server is sole writer of canonical
  state; clients render snapshots).
- The full interaction surface: return-greeting, idle watching (presence), listen-in,
  offer (seed / song fragment / still pool), settle (with 5-second undo), field notebook.
- Presence accounting with the exact three-signal conjunction (visible + focused + recent
  pointer/key activity).
- Visit invitations (read-only ambient views), off by default, per-invite opt-in,
  revocable, 30-day expiry, silent visit log, optional per-account visit-notification
  toggle (off by default).
- Accessibility surfaces as designed first-class features, shipping with launch:
  screen-reader naturalist narration, reduced-motion rendering mode, procedural call
  captioning, full keyboard navigation, WCAG AA contrast on all chrome text.
- Account export (JSON snapshot emailed as a download link) and soft-then-hard account
  deletion (30-day soft window).
- Performance budgets: initial JS < 2MB gzipped; first bird visible < 500ms on mid-tier
  mobile over 4G; 60fps idle on a 5-year-old laptop sustained over 30 minutes; zero
  memory growth over 30 minutes, enforced in CI.

### 1.2 V1 does not ship (non-goals, enforced)

- No native apps; web-only. Data model and protocols are not designed for native clients.
- No gamification of any kind: no streaks, scores, badges, levels, achievements, visit
  counters, calendars of green dots, "birds adopted: N" surfaces, XP, tiers. Not opt-in,
  not hidden, not in the notebook. The notebook may observe the aviary; it never observes
  the user's behavior ("you visited every day this week" is a banned sentence class).
- No Tamagotchi mechanics: birds do not die, hunger, suffer, or display distress. Neglect
  yields ambient quietness, never punishment. Personality drift is monotonic toward
  expressive; nothing the user fails to do moves a trait down.
- No social-network surfaces: no profiles, follows, feeds, discovery, comments, avatars,
  co-presence, leaderboards, or "most-visited" anything. We do not compute the stats that
  would power leaderboards.
- No notifications as a product surface: no push, no aviary-triggered email, no in-product
  "your friend visited" banner (the opt-in visit-notification setting is the single,
  off-by-default exception; transactional auth/export email is infrastructure, not a
  notification surface).
- No welcome toasts, banners, greeting modals, or "you've been gone X days" surfaces. The
  bird greeting is the entire welcome.
- No payments, shared/household aviaries, multi-aviary accounts, customizable scenes,
  species rarity, or recorded-audio fallback.

### 1.3 Product invariants (treat as constitutional)

These are the rules whose violation would silently produce a different product. Every one
is testable, and every one gets a test.

1. **The aviary never announces.** No toast, banner, modal, or badge ever greets,
   congratulates, reminds, or notifies the user inside the product surface.
2. **The first frame is mid-action.** No entry animation, no fade-from-static, no
   spinner. Loading fallback is the quiet field, never a loading indicator.
3. **The user never sees a number about a bird.** No personality value, drift value, mood
   label, or visit-frequency figure is ever rendered, exposed in an API response intended
   for display, or toggled into existence. (Debug tooling may read these server-side; it
   must never ship to a client surface.)
4. **Drift is monotonic toward expressive.** Presence moves traits up; absence moves
   nothing down. Enforced as a property test on the drift function.
5. **The server is the only writer of personality state.** Clients submit events; the tick
   applies additive deltas. No code path lets a client write an absolute trait value.
6. **Bird identity is immortal.** A bird's stable ID survives renames, syncs, migrations,
   and species-pool changes. There is no "regenerate bird" code path.
7. **Calls are procedural, always.** No recorded audio ships at any quality, in any
   fallback. WebAudio unavailable → silence + captions.
8. **Per-bird interaction data never leaves the account context.** No aggregation, no
   analytics-warehouse reads of the simulation DB, no ML training on per-bird fields.

---

## 2. Architecture

### 2.1 Shape

Three deployable units plus storage, deliberately boring:

```
┌────────────────────────────────────────────────────────────┐
│ Clients (browser, web-only)                                │
│  - static SPA served from CDN edge                         │
│  - pulls snapshots, submits events, renders, synthesizes   │
└──────────────▲──────────────────────────────▲──────────────┘
               │ HTTPS (snapshot pull,        │ HTTPS (event ingest,
               │  edge-cached bootstrap)      │  auth, account, invites)
┌──────────────┴──────────────────────────────┴──────────────┐
│ API service (stateless, horizontally scaled)               │
│  auth · snapshot read · event ingest · notebook read ·     │
│  account/settings · invites/visits · export/delete         │
└───────▲───────────────────────────────▲────────────────────┘
        │ reads canonical state          │ writes events
        │ (and fast-tick wait)           │
┌───────┴────────────────────────────────────────────────────┐
│ Simulation worker pool (the tick)                          │
│  per-aviary tick loop · drift · mood · weather · notebook  │
│  generation · adoption-offer scheduling                    │
└───────▲────────────────────────────────────────────────────┘
        │ sole writer of personality/mood/state
┌───────┴─────────────┐   ┌──────────────────────────────┐
│ Postgres (canonical)│   │ Object storage (exports)     │
│  accounts, aviaries,│   │ Transactional email provider │
│  birds, events,     │   │  (magic links, invites,      │
│  notebook, invites, │   │   export links)              │
│  ticks, sessions    │   └──────────────────────────────┘
└─────────────────────┘
Optional: Redis for tick scheduling/locks and rate limits.
```

- **API service**: stateless; any replica can serve any request. Holds no simulation
  state in memory except a per-request "wait for fast tick" (≤ ~1.5s, see §5.4).
- **Simulation workers**: a partitioned scheduler owns aviary tick leases. Workers are the
  only processes with write credentials on `birds`, `aviaries.state_version`,
  `notebook_entries`, and adoption offers. API replicas get read-only credentials on those
  tables and write credentials only on `interaction_events`, auth/session tables, invites,
  and account settings. The write-path split is enforced by database roles, not by
  convention.
- **Storage**: one Postgres cluster is sufficient at v1 scale (events table is the only
  high-append surface; see §9.5 for retention/roll-up). Redis is optional: convenient for
  tick leases, presence-ping dedupe windows, and rate limiting; the system must degrade to
  Postgres-only operation if Redis is absent.

### 2.2 Client/server split (the render pipeline boundary)

The boundary is the PRD's central rule: **clients render snapshots; the server simulates.**

Server owns (canonical, persistent): personality vectors, mood, perch positions and
activity states, weather state, settled state, notebook entries, adoption state, accounts,
invites, event log, presence-time accumulators.

Client owns (ephemeral, per-tab): render interpolation, pose/motion phase, procedural call
scheduling and synthesis (parameterized by snapshot values), ambient ornaments (leaves,
feathers — explicitly not simulated), audio mixing, presence detection, caption rendering,
narration prose generation (from snapshot + deltas), keyboard/focus state.

Consequences:

- A client that crashes loses nothing. A server that loses the sim DB loses the product —
  backups and PITR are launch-blocking (§12.4).
- Two devices never diverge, because neither owns state. "Sync" is read consistency, not
  merge.
- The client never recomputes drift, never infers mood, never derives a personality value
  from history. Snapshot fields are authoritative.

### 2.3 Technology selections (defensible defaults)

- **Language**: TypeScript end-to-end (API, workers, client). Shared schema/voice/prose
  packages in a monorepo.
- **API**: a thin HTTP framework (Fastify-class). No GraphQL; the surface is small and
  cache-friendly. REST-ish JSON, versioned by path prefix `/v1`.
- **Client UI chrome**: a tiny compiled component runtime (Svelte-class) for the top bar,
  settings, notebook view, auth screens. Total chrome budget ≪ the scene renderer.
- **Scene renderer**: custom Canvas 2D scene graph, no game engine (bundle budget forbids
  one). 2D skeletal bird rigs with procedurally generated sprite atlases per species (see
  §7). WebGL considered and rejected for v1: at ≤ 7 birds plus ornaments, Canvas 2D meets
  the 60fps/5-year-old-laptop budget with far less code.
- **Audio**: WebAudio API, hand-rolled graph (no audio framework). See §8.
- **DB**: Postgres 15+. Events table range-partitioned by month.
- **Email**: any transactional provider with high deliverability; abstracted behind an
  internal `Mailer` interface.
- **Infra**: containers behind a CDN; edge HTML bootstrap injection for the first-paint
  snapshot (§10.2). Any cloud; nothing exotic.

---

## 3. Data model

All tables carry `created_at`/`updated_at` unless noted. All internal references use
synthetic UUIDs. **Email appears in exactly one table** (`accounts.email_ciphertext`,
plus transiently in `magic_links`/`invites`, also encrypted); every other reference is the
account UUID. Email is never a log field, shard key, partition key, or telemetry
dimension. This is the PRD's "most important boring detail" and is enforced by a lint
rule on query construction plus a schema review gate.

### 3.1 Auth and account

- `accounts`
  - `id uuid pk` — synthetic, generated at creation; the only account identifier used
    anywhere else in the system.
  - `email_ciphertext bytea` — encrypted at rest (app-level envelope encryption).
  - `email_hash bytea` — keyed hash for lookup on sign-in; never logged.
  - `timezone text` — IANA zone captured from the client at sign-up and editable; drives
    day/night anchoring and mood time-of-day priors. DST-safe (store zone, not offset).
  - `settings jsonb` — `{ captions: bool|null, reduced_motion: 'system'|'on'|'off',
    audio: 'on'|'muted', visit_notifications: bool(false), narration: bool(true) }`.
  - `deleted_at timestamptz null` — soft-delete marker; 30 days later a hard-delete job
    removes every row in every table for the account (verified by a deletion audit test).
- `sessions` — `id, account_id fk, device_label text (user agent summary), created_at,
  last_seen_at, revoked_at null`. Session tokens are opaque, per-device, revocable;
  presented as `Authorization: Bearer`. Refresh by re-validation; no sliding complexity.
- `magic_links` — `id, token_hash, email_ciphertext, expires_at (now + 15 min),
  consumed_at null, request_ip inet`. Single-use: consumption sets `consumed_at`
  atomically; replay of a consumed token fails closed with the matter-of-fact error
  surface. Rate limit per email and per IP.

### 3.2 Aviary and birds

- `aviaries` — `id, account_id unique fk, state_version bigint (monotonic, bumped by every
  tick that changes state), settled_until timestamptz null, weather_state jsonb,
  weather_next_change_at, lighting_override text null ('settled'), created_at`.
- `species` — `id, code text unique, display_name text, silhouette_ref text, palette jsonb,
  motif_library_ref text, is_nocturnal bool` (the nightjar-like species). ~6 rows at v1;
  rarity is not modeled anywhere.
- `birds`
  - `id uuid pk` — stable identity; never reused, never regenerated.
  - `aviary_id fk`, `species_id fk`, `name text` (renameable; rename writes an event, not
    state, so the tick remains the only state writer — see §5.6).
  - `personality jsonb` — `{ boldness, warmth, vocal, plumage, curiosity }`, each a
    float in `[0,1]` (normalized range; exact seeds are sim-service constants). Updated
    only by the tick.
  - `expressiveness float` — the recency multiplier derived from trailing presence
    (§5.3). Server-computed; used by behavior scoring. Not a trait; can move down.
  - `mood text` — enum `{wary, content, curious, drowsy, alert}`; `mood_entered_at`.
  - `activity text` — current behavior state the renderer needs (`preening`, `scanning`,
    `perched_idle`, `sleeping`, `approaching_offer`, …) with `activity_phase float` so the
    client resumes motion mid-cycle.
  - `perch_zone text` — `front|middle|back`; `perch_slot smallint`, `x_norm float`
    (normalized scene position).
  - `adopted_at`, `last_greeted_at null`.
- `adoption_offers` — `id, aviary_id fk, species_id fk, offered_at, status
  (offered|accepted|expired), resolved_at null`. Generated by the age-based schedule
  (§12.2); accepting creates a bird with seed personality in the same tick.

### 3.3 Events and simulation bookkeeping

- `interaction_events` (append-only; monthly range partitions)
  - `id bigserial pk`, `aviary_id`, `account_id`, `type` enum:
    `presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo, rename,
    adoption_accept, captions_toggle, audio_mute_toggle, narration_toggle`
  - `payload jsonb` — typed per event (§4.2).
  - `client_ts timestamptz`, `server_ts timestamptz default now()`.
  - `consumed_tick_id bigint null` — set by the tick that folds the event in. NULL =
    unconsumed. This is the exactly-once ledger: a tick claims events by setting
    `consumed_tick_id` in the same transaction as the state update.
- `ticks` — `id bigserial, aviary_id, started_at, duration_ms, event_lo, event_hi,
  notes jsonb`. One row per tick; the observability spine of the sim (p99 alarm source).
- `presence_daily` — `aviary_id, day date, minutes int, by_bird_listen_in jsonb`.
  Rolled up by the tick from `presence_ping`/`listen_in_*` events; the drift function's
  input table; also caps total creditable presence per account-day (§5.2).

### 3.4 Notebook, offers cooldown

- `notebook_entries` — `id, aviary_id, entry_day date, body text, kind text
  (first|change|contrast|quiet|weather|arrival), source jsonb (event/state refs for
  audit), created_at`. Immutable; read-only to clients; never archived, never editable.
- Offer cooldowns are **derived** from `interaction_events` (last `offer` per bird) at
  tick time — no separate table to drift out of sync. The client greys the offer control
  from snapshot-provided `offer_ready_at` per bird but the server is authoritative.

### 3.5 Social

- `invites` — `id, aviary_id fk, email_ciphertext, token_hash, created_at, expires_at
  (now + 30d), consumed_at null, revoked_at null`. Outstanding = not consumed, not
  revoked, not expired.
- `visit_sessions` — `id, invite_id fk, started_at, last_seen_at, ended_at null`.
  Duration is computed from `last_seen_at - started_at` at display time; the visit log
  shows email + date + approximate duration, most-recent first. Visit sessions never
  write to `interaction_events` — visitor presence does not exist as far as the
  simulation is concerned.

### 3.6 Exports

On-demand: assemble account row, aviary, birds (including personality vectors — the
user's own data, export is the one legitimate place numbers leave the system, and the
export goes to the user, not a surface), moods, notebook entries, settings. Serialize to
JSON, store encrypted in object storage with a 7-day signed link, email the link.

---

## 4. API surface

JSON over HTTPS, cookie or bearer session auth. All errors on system surfaces use the
matter-of-fact voice (exact copy lives in a shared `voice` package so it cannot drift).
Snapshot and event payloads are small by construction (kilobytes).

### 4.1 State read

- `GET /v1/aviary/snapshot?since={version}`
  - 200: full snapshot if `since` mismatches or is absent; `{ version, unchanged: true }`
    if current.
  - Body: `{ version, server_time, aviary: { weather, lighting, settled }, birds: [ {
    id, name, species, mood, activity, activity_phase, perch_zone, x_norm,
    plumage_tier, vocal_params, offer_ready_at } ], narration_seed, calls_hint }`.
  - **Personality numbers are never serialized.** The client receives *derived rendering
    parameters* only: `plumage_tier` (a quantized visual richness level the atlas
    supports), `vocal_params` (voiceprint + call-rate parameters needed for synthesis),
    behavior states. Quantization is deliberate: the API cannot leak a trait value even
    to a curious user reading their own traffic, because it never transmits one.
    (`vocal_params` contains synthesis constants — pitch base, formant — not trait
    scores.)
  - Cache: private, no-store; ETag = state_version. Polling cadence: keepalive every 45s
    while visible; immediate pull on `visibilitychange → visible` and on render-frame-gap
    > 5s (laptop-suspend recovery). The 45s keepalive is well under the tick cadence's
    visibility horizon because ticks are usually no-ops; unchanged polls return a tiny
    body.
- `GET /v1/notebook?cursor` — reverse-chronological entries, paginated; read-only.
- `GET /v1/bootstrap` (edge) — returns the HTML shell with the current snapshot inlined
  (signed, 60s TTL) for accounts with an active session; this is how first-bird < 500ms
  is achieved (§10.2).

### 4.2 Interaction events (client → server)

- `POST /v1/aviary/events` — batch, `{ events: [ { type, payload, client_ts } ] }`.
  - 202 `{ accepted: n, server_ts, snapshot? }`. Idempotency: each event carries a
    client-generated `event_uid`; the server dedupes on `(aviary_id, event_uid)` within a
    24h window so retries from flaky mobile networks never double-apply.
  - If any event in the batch is reaction-relevant (`offer`, `settle`, `settle_undo`,
    `listen_in_start`, `adoption_accept`), the API asks the sim layer for a fast tick and
    waits up to ~1.5s; on success the fresh snapshot is included in the response so the
    client renders the reaction without an extra round trip. Timeout → 202 without
    snapshot; the next poll picks it up. The client never predicts simulation outcomes.
  - Payloads (all small):
    - `presence_ping { span_ms ≤ 60_000, tab_id }` — heartbeat while the three-signal
      conjunction holds; span capped server-side (§5.2).
    - `listen_in_start { bird_id }` / `listen_in_end { bird_id, span_ms }`
    - `offer { kind: seed|song|pool, song_id?, target_bird_id? }` — target optional; the
      sim decides who reacts based on proximity/mood/curiosity.
    - `settle {}` / `settle_undo {}`
    - `rename { bird_id, name }`, `adoption_accept { offer_id, name }`
    - `audio_mute_toggle { muted }`, `captions_toggle { on }`, `narration_toggle { on }`
      — settings mirrors; also drift-relevant (muted listening counts at reduced weight,
      never negative; §5.3).

### 4.3 Auth and account

- `POST /v1/auth/magic-link { email }` — always 200 with neutral copy (no account
  enumeration).
- `POST /v1/auth/verify { token }` → session token + account bootstrap or matter-of-fact
  failure ("We couldn't sign you in. The link may have expired. Try requesting a new
  link.").
- `GET /v1/auth/sessions`, `DELETE /v1/auth/sessions/{id}` (revoke), `POST /v1/auth/signout`.
- `GET /v1/account`, `PATCH /v1/account/settings` (captions, reduced_motion, audio,
  narration, visit_notifications, timezone).
- `POST /v1/account/email-change { new_email }` → verify-new-address flow; old address
  works until verification commits.
- `POST /v1/account/export` → 202; email with download link.
- `POST /v1/account/delete` → soft-delete now; any signed-in page shows the recovery
  affordance; `POST /v1/account/restore`. Hard-delete job after 30 days.

### 4.4 Visits (host and visitor)

- Host: `POST /v1/invites { email }` (sends one-time link), `GET /v1/invites` (outstanding
  + log), `DELETE /v1/invites/{id}` (revoke; immediate), `GET /v1/visits/log`.
- Visitor: `POST /v1/visit/redeem { token }` → read-only visit session (scoped token that
  authorizes exactly one route): `GET /v1/visit/snapshot?since=…`.
  - The visit snapshot handler checks invite validity **on every pull**; revocation takes
    effect at the visitor's next poll, returning 410 with the matter-of-fact "visit no
    longer available" surface.
  - Visit sessions record only `visit_sessions` rows (for the host's log). They cannot
    reach `POST /v1/aviary/events` (scoped token + route-level guard + DB-role guard:
    defense in depth). No greetings, offers, listen-in, settle, or notebook writes. The
    visitor sees exactly what the host sees — same snapshot fields, no special rendering.

---

## 5. Simulation engine design

The tick is the product. Everything in this section runs server-side, in workers, on the
schedule below.

### 5.1 Tick scheduler

- **Idle cadence**: each aviary ticks every 60s ± 10s jitter, whether or not any client is
  connected (exact cadence tuned in build; the PRD's "~once per minute" is the anchor).
- **Fast tick**: when unconsumed reaction-relevant events exist for an aviary, a tick is
  scheduled immediately (target latency ≤ 2s from event ingest to committed state). The
  API's 1.5s wait usually returns the fresh snapshot inline.
- **Leases**: scheduler partitions aviaries by hash; a worker takes a lease
  (Redis `SET NX PX` or Postgres advisory lock) before ticking. Crash → lease expiry →
  re-tick. Ticks are idempotent by construction (event claiming is transactional), so a
  double-tick after a crash is harmless.
- **No-op fast path**: a tick with no new events and no scheduled transitions (weather
  change, mood timer, adoption offer) writes nothing and doesn't bump `state_version` —
  keeps the event poll cheap and the `ticks` table meaningful.

### 5.2 Tick pipeline (per aviary, per tick)

One transaction:

1. **Claim events**: `UPDATE interaction_events SET consumed_tick_id = :tick WHERE
   aviary_id = :a AND consumed_tick_id IS NULL RETURNING *` (ordered by `server_ts, id`).
2. **Fold presence**: validate and sum `presence_ping` spans into `presence_daily` and
   the trailing-14-day expressiveness window. Validation rules (the honesty guards):
   - per-ping span ≤ 60s (client heartbeats at 30s; cap tolerates one missed beat);
   - creditable presence per account-day ≤ 16h (bounds stuck-tab and abuse cases);
   - multi-tab dedupe: overlapping spans from the same account within the same minute
     count once (presence is per-person, not per-tab).
3. **Update expressiveness** per bird: `E = clamp(k · Σ decayed presence over trailing
   14d)`. E can fall with absence — this is *how* birds "become ambient" without trait
   regression (§5.3).
4. **Mood transitions** (§5.5) using event impulses, time-of-day prior, weather, and
   personality bias.
5. **Behavior/perch selection**: from mood × personality × expressiveness, choose activity
   state, perch zone, and targets (e.g., approach offer). Includes bird-to-bird effects
   (§5.7) and greeting selection when a presence-window opens (§5.8).
6. **Drift accumulation** (§5.3): fold the day's inputs into per-trait accumulators;
   apply the monotonic update.
7. **Weather advance** (§5.9): rare, scheduled, short-lived.
8. **Notebook scan** (§5.10): detect noteworthy patterns; maybe write one entry.
9. **Adoption schedule** (§12.2): age-based offer generation.
10. **Commit**: update `birds`, `aviaries` (`state_version++` iff anything changed),
    write `ticks` row. Transaction boundary guarantees events-and-state move together —
    the no-last-write-wins rule is structural.

### 5.3 Drift function (the load-bearing math)

Model: per-trait **slow integrator over daily inputs**, monotonic up.

- Daily inputs per bird (from `presence_daily` + events):
  - `P` = presence minutes (account-level, dominant input), capped at 120 credited
    min/day for drift purposes (diminishing returns; prevents weekend binges from
    warping calibration).
  - `L_b` = listen-in minutes on bird *b*.
  - `O_b` = offers near/accepted by bird *b* (accepted → curiosity; near → boldness).
  - `A_m` = audio-muted fraction of presence (reduces warmth/vocal credit to a floor of
    ~60%, never below zero contribution; a muted watcher is still present, just less of
    a listener — interpretation flagged in §14).
- Update rule (per trait, per day `d`):
  - `input_boldness = w1·P_norm + w2·O_near_b`, `input_warmth = w3·P_norm·(1−0.4·A_m) +
    w4·L_b_norm`, `input_vocal = w5·P_norm·(1−0.4·A_m) + w6·L_b_norm`,
    `input_curiosity = w7·O_accept_b + w8·P_norm`, `input_plumage = w9·P_norm`
    (plumage rises with sustained attention only).
  - `Δ = rate_trait · input`, where `rate_trait` is set so that **10 min/day of average
    presence yields ≥ 0.02 trait units after 7 days** (instrument-visible) and **≥ 0.10
    after ~21 days** (user-visible). Starting `rate` ≈ 0.003–0.005/day at reference
    input; the calibration harness (§11.3) owns the final numbers.
  - **Monotonicity clamp**: `trait_{d+1} = clamp01(trait_d + max(0, Δ))`. There is no
    subtraction path. Absence yields `input = 0` → `Δ = 0` → flat.
- **Expressiveness** (separate, non-trait): `E ∈ [0,1]`, EMA of daily presence with a
  14-day-ish horizon. E scales greeting probability, chorus-join probability, and
  front-perch occupancy. A user gone two weeks returns to birds whose *traits* are
  intact but whose *behavior* is quiet — "still alive, still calling, greeting less
  often." Warming back up takes days, not weeks.
- **Why two layers**: the PRD simultaneously demands "traits never move down" and
  "ignored birds become ambient (greet less)". A single monotonic vector cannot express
  both; the E layer resolves it without violating either sentence. Flagged in §14.
- **Property tests** (CI, mandatory):
  - monotonicity: for all event streams, `trait_final ≥ trait_initial` componentwise;
  - no-single-session-visibility: one 30-min session moves any trait < 0.01;
  - calibration band: reference pattern (10 min/day + occasional listen-ins) lands in
    [0.02, 0.05] at day 7 and [0.10, 0.20] at day 21;
  - neglect-flatness: zero-event weeks change nothing except E decay;
  - offer-cooldown necessity: with cooldown removed, curiosity saturates in one session
    (guards the PRD's engine-collapse warning).

### 5.4 Event application order

Events fold in `(server_ts, id)` order — append-only log order, never client timestamps
(client clocks lie; client_ts is stored for diagnostics only). Additive deltas mean order
sensitivity is minimal, but fixed order makes ticks deterministic and replayable: the
entire sim state is reconstructible from (initial state + event log + sim version), which
is how we build the calibration harness and incident forensics.

### 5.5 Mood machine

Per-bird enumerated state `{wary, content, curious, drowsy, alert}`; `sleeping` is a
derived night-time activity, not a mood (mood persists through the night; activity is
what renders).

- **Transition scoring**: on each tick and on event impulses, compute scores
  `S_m = prior_time(m, local_hour) + Σ impulses(events) + bias_personality(m, traits) +
  ambient(weather, other_birds)` with:
  - time-of-day priors: alert peak early morning, drowsy ramp near dusk, night → drowsy
    dominance ("daily-ish reset" emerges from strong priors, not a hard reset — mood at
    session end genuinely persists into the next session, modulated by interim ticks);
  - impulses: offer accepted → +content (decays over ~30–60 min); offer ignored while
    drowsy → none; another bird's alarm → +wary neighbors; settle → +drowsy/quieting;
  - personality bias: high boldness damps +wary; high curiosity amplifies +curious on
    novel events;
  - **hysteresis + minimum dwell** (~5–10 min) so moods don't flicker tick-to-tick.
- Mood persists in `birds.mood` with `mood_entered_at`; clients only read it.

### 5.6 Renames and user-writable fields

Even renames flow through events: `rename` event → tick applies → snapshot. This keeps
the single-writer invariant absolute (no "just this one field" exceptions for future
contributors to copy). Latency is invisible to the user (fast tick ≤ 2s; the client can
echo the typed name in the edit field while pending, since text-in-an-input is chrome,
not state).

### 5.7 Bird-to-bird interaction

Within a tick: after individual mood/behavior selection, resolve social effects:

- call→response: if bird A is in a calling activity, bird B rolls `P(respond) =
  f(B.warmth, B.vocal, B.mood, E_B)`; on success B's activity becomes `responding` with a
  staggered start offset (200–1200ms, randomized per tick seed) — the same stagger rule
  the greeting uses.
- mood contagion: wary spreads with `P ∝ A.alarm_strength · (1 − B.boldness)`; content
  does not spread (calm is not contagious; alarm is).
- chorus emergence: ≥ 2 birds in calling/responding activity within the same window is
  flagged in the snapshot (`chorus: [bird_ids]`); the client mixer treats chorus members
  as an ensemble for timing (§8.4).

### 5.8 Return-greeting selection

When a presence-window opens (first valid `presence_ping` after ≥ X min closed, X ≈ 10):

1. Compute absence length from last window close.
2. Select greeter: argmax over birds of `greet_score = boldness·w + warmth·w + E·w +
   mood_fit + ε` — bolder, warmer, more-expressive birds greet first; a wary bird may not
   greet at all on a given day.
3. Select form from absence × score: glance (minutes), two-note call + head-tilt (hours),
   approach + longer call, possibly with a second bird's staggered response (days+).
4. Write `greeting` directive into the snapshot (bird, form, start delay, stagger set).
   The client renders it procedurally from the rig — never a canned animation — so no two
   greetings are identical. Multiple greeters always stagger; unison is forbidden.
5. The greeting is also a narration-priority event (§9.1) and a notebook-eligible moment
   ("pip greeted before wren today, first time this week" — the notebook generator reads
   the same greeting log).

### 5.9 Weather generator

- Schedule: per-aviary, server-seeded. Rain: Poisson, mean ≈ 2–3 events/week, duration
  5–20 sim-minutes. Wind: occasional, similar rarity. Never thunderstorms, never snow,
  never anything that demands attention.
- Effects while active: rain → temporary vocal-frequency damp across birds + slight
  content/drowsy tilt; wind → alert/wary split by boldness. Effects expire with the
  event. Weather state lives in `aviaries.weather_state` with `next_change_at` so ticks
  are self-scheduling and restart-safe.

### 5.10 Notebook generation

A generator, not a log formatter:

- **Candidate detectors** (run each tick, cheap): firsts ("pip greeted before wren,
  first time this week"), changes (perch-zone shift vs. trailing week; chorus after quiet
  days), contrasts (one bird fluffed/drowsy while other active), quiet stretches (≥ N
  hours low activity), weather moments, arrivals (adoptions). Each emits a typed
  candidate with facts (bird ids, times, comparisons).
- **Sparsity governor**: target ≈ 1 entry per 2–3 days for a regularly visited aviary;
  hard cap 1/day; minimum gap ~20h; candidates ranked by a noteworthiness score; a
  candidate that loses to the cap is dropped, not queued (stale observations are worse
  than none). Very active users get the same cap — the governor is explicitly
  anti-feed.
- **Prose writer**: template families per kind with slot-filled specifics, lexical
  variation tables, and a no-repeat window (a template may not recur within ~10 entries).
  Output voice rules are code-reviewed and snapshot-tested: lowercase, present-tense,
  bird-named, no exclamation, no "you", no announcement framing, no user-behavior
  observations (the banned class from §1.2 — a CI lint scans generated entries for
  streak/visit-count phrasing patterns).
- Entries are written at tick time and are immutable thereafter.

### 5.11 Settle semantics

- `settle` event → tick sets `aviary.settled_until = null + lighting_override='settled'`
  (persists until re-engagement or tab close; there is no auto-morning reset of a settle
  — a new presence-window clears it). Calls quiet (vocal params damped in snapshot);
  birds trend drowsy.
- `settle_undo` within 5s wall-clock (server-validated by event timestamps) reverses the
  override; any other interaction event also clears `settled` (re-engagement rule).
- Tab close without settle is engine-equivalent: presence-window closes on the final
  ping's expiry; no penalty, no recovery surface, no notification.

---

## 6. Sync model

- **One canonical record.** All state that matters lives once, server-side. Multi-device
  sync is not a feature; it is the absence of divergence. Laptop and phone pull the same
  `state_version`; both render it.
- **Conflict prevention by construction**: conflicts require two writers; there is one
  writer (the tick). The rules that keep it that way:
  1. clients can only append events (with idempotency keys);
  2. events carry no state, only observations ("listened to Pip 3 min"), never values
     ("set boldness 0.62") — the API has no endpoint that accepts a trait value;
  3. event application is ordered, transactional, exactly-once via `consumed_tick_id`;
  4. DB roles make personality tables unwritable by the API role.
- **Concurrency within one account**: two tabs heartbeating presence → server-side
  dedupe (§5.2.2). Two devices offering simultaneously → per-bird cooldown enforced at
  tick; the losing offer returns as "noted but not reacted" (the bird was already
  investigating — a naturalistic outcome, not an error).
- **Snapshot consistency**: `state_version` is the only cursor. A client that falls
  behind (suspend, tunnel) gets a full snapshot, not a delta stream — snapshots are
  kilobytes; delta infrastructure is unjustified complexity at v1. Motion continuity
  across a full-snapshot replace is handled client-side by phase-blending (§7.4), never
  by snapping.
- **Visit sync**: visitors hold scoped read tokens; revocation checked per pull → next
  poll ends the session. No realtime push anywhere in v1 (polling at 45s is
  indistinguishable for a product whose fastest canonical change cadence is ~1 min; fast
  ticks surface via the event-response inline snapshot).

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition (painter's order)

1. **Sky/lighting layer** — vertical gradient from a time-of-day palette LUT (24
   keyframes, smooth-blended by local hour from the account timezone; weather dims;
   settle/evening overrides warm-shift). Color shifts animate on a slow clock (minutes).
2. **Background foliage** — 2–3 soft shapes, parallax factor ~0.1–0.2. Subtle.
3. **Perches** — three zones (front/middle/back) with slots; static art, calm palette.
4. **Birds** — one rig per bird (§7.2).
5. **Weather overlay** — rain streaks / leaf-ripple, only when `weather_state` says so.
6. **Foreground ornaments** — rare drifting leaf/feather sprites, client-generated at
   idle cadence (pure ornament, no sim state), parallax ~1.1–1.3, never crossing birds'
   faces.
7. **Lighting grade** — global tint/vignette pass for dusk/night/settled. Night dims most
   of the scene; the nocturnal species stays active.

Viewport rules: scene scales to fit width; perches redistribute so all birds are always
in frame (narrow phone: compress inter-perch distance; wide desktop: spread). No pan,
zoom, or scroll — ever. Aspect handling is letterbox-free: sky/foliage extend, birds
never crop.

### 7.2 Bird rigs and idle micro-motion

- Per species: a small 2D skeletal rig (body, head, tail, wing, eye — ~8–12 bones)
  rendered from a build-time-generated sprite atlas (procedural feather/palette variants
  keyed by `plumage_tier`). Atlases are compact (target ≤ 150KB per species).
- **Motion is code, not footage**: animation functions (`preen(t)`, `scan(t)`,
  `head_tilt(t)`, `shuffle(t)`, `breathe(t)`) drive bones with per-bird parameter seeds
  and mood-shaped weighting — wary: more scanning, back perch; content: preening;
  curious: tilts toward sound sources (client knows call positions); drowsy: low posture,
  fluffed (scale/puff morph), slow blink. Motion never stops while visible; "paused" is
  a banned visual.
- `activity` + `activity_phase` from the snapshot set which function runs and where in
  its cycle — this is what makes the first frame mid-action: the client doesn't start
  animations, it *joins* them at phase.
- Perch transitions: short flight/hop paths between slots, initiated only by snapshot
  state changes, eased; no teleporting.

### 7.3 Boot path (no entry animation, ever)

1. Edge HTML includes inlined snapshot (§10.2) + critical CSS + bootstrap JS (one
   blocking bundle ≤ ~120KB gz of the 2MB total budget).
2. Renderer initializes synchronously from the inlined snapshot: draws sky for local
   time, perches, and each bird at its snapshot position in its current activity phase —
   first paint **is** the aviary, already moving. There is no "ready" moment.
3. Defer everything non-critical: ornament sprites, notebook data, settings bundles
   (code-split), audio graph construction (idle callback or first gesture).
4. Snapshot missing/stale (cold cache, signed-out bootstrap): render the **quiet field**
   — sky gradient + one or two faint motion cues — while fetching; birds fly in softly to
   their current perches on arrival (the same treatment as the empty-aviary first
   adoption). A spinner is a launch-blocking defect; CI greps the bundle for spinner
   assets.

### 7.4 Snapshot reconciliation

- Interpolation: keep last two snapshots; lerp positions over the poll interval; ease
  activity changes (blend pose functions over ~400–800ms). `activity_phase` continuity
  prevents motion pops; if phases disagree wildly (long gap), cross-fade poses instead of
  jumping.
- Rendering stops when `visibilityState !== 'visible'` (rAF suspended, audio scheduler
  idles to near-silent ambient; the *simulation* continues server-side). On return:
  immediate snapshot pull, phase-blend into current state — the aviary has been living;
  the client catches up.

### 7.5 Reduced-motion mode (a designed surface, not a fallback)

Same scene graph, different motion register, shipped at launch:

- Micro-motion → slow cross-fades between discrete still poses (the same animation
  functions sampled at key poses, cross-faded over 1.5–3s).
- Perch flights → cross-fade between perch poses.
- Ambient ornaments removed; day/evening color shifts remain, slowed.
- Calls, captions, notebook, drift, mood: unchanged. The aviary is still the aviary.
- Triggered by `prefers-reduced-motion` or the accessibility-settings override
  (`system|on|off`). The mode is a first-class render target in the pipeline from day
  one — both modes are exercised in the same visual-regression suite, so reduced-motion
  cannot rot.

### 7.6 Top bar and chrome

- Thin top bar above the scene: account/settings, accessibility, notebook, offer. Nothing
  else. No chrome inside the scene — no tooltips, badges, labels, hover affordances.
- Fade: after ~4s of cursor stillness → ~5% opacity; restore on pointer/key activity.
  Keyboard users: bar is always visible while focus is within it.
- Offer flow: top-bar affordance opens a small panel (seed / song fragment library /
  still pool), fully keyboard-navigable, naturalist microcopy. Offers are never triggered
  by clicking a bird (clicking a bird is listen-in).

---

## 8. Audio pipeline

### 8.1 Synthesis (procedural or nothing)

- Per species: a **motif library** — parameterized syllable types (rise, trill, chirp,
  warble, buzz) with pitch contours, durations, timbre recipes (FM index, noise blend,
  formant band). Per bird: a stable **voiceprint** derived from its ID and vocal
  parameters at adoption (base pitch ±, formant center, tempo bias, motif preference
  weights). The voiceprint is the recognizability invariant: mood and drift modulate
  *rate, energy, contour range*; pitch base and formant identity stay within a narrow
  band. Pip is always Pip by ear.
- Runtime synthesis: WebAudio graph per call = carrier osc (FM/AM) + filtered noise
  source + formant bandpass + per-syllable gain envelope. Calls are sequenced from motif
  grammar with seeded randomness: timing jitter ±10–20%, micro-pitch drift, occasional
  syllable elision/repetition. Two renderings of the "same" call are never identical —
  that is the point, and a test asserts it (sample N renders, require pairwise
  difference above threshold while classifying as same motif).
- **Caption text is generated from the same parameter set** that drives synthesis (§9.3),
  guaranteeing the caption matches what played.

### 8.2 Graph and mixing

```
per-bird: synth voice → birdGain → stereoPanner(x_norm) ─┐
                                                         ├→ master → compressor → out
ambient bed (procedural leaves/air, very quiet) → ambGain ┘
```

- Listen-in: on engage, ramp focused bird to full and others to ~0.35 (never zero) over
  ~2s (equal-power curves); disengage symmetric. Hard cuts are forbidden — the mixer has
  no instant path. Pan follows `x_norm` so the mix has gentle spatiality.
- Settled/night: master duck + call-rate damping via snapshot params.

### 8.3 Scheduling

- Client-side scheduler per bird: next-call time drawn from rate model `λ(vocal_params,
  mood, time-of-day, weather, E)`; unobserved calling continues (the tab audibly lives
  even when the user isn't interacting — subject to autoplay, §8.6).
- Call→response and chorus flags from the snapshot bias scheduling: chorus members'
  schedulers align windows with jittered offsets (staggered, never unison).

### 8.4 Resource discipline

- Preallocated node pool; calls build/disconnect subgraphs without allocation churn;
  no per-call buffer allocation (syllable envelopes are parameter-driven); audio worklet
  only if profiling demands it (main-thread synth at 7 voices is well within budget).
- The 30-minute no-memory-growth CI test runs the audio path with synthesized hours of
  calls.

### 8.5 WebAudio fallback

If `AudioContext` is unavailable or fails: graceful silence + **captions on by default**
(the caption pipeline runs off the scheduler regardless of audio output). No recorded
audio ships under any circumstances. If the context errors mid-session, same fallback,
plus an aggregate error metric.

### 8.6 Autoplay reality (platform constraint, planned)

Browsers suspend audio until a user gesture. Plan: attempt `resume()` on load; while
suspended, render captions (treated as temporarily-audio-unavailable) and queue the
scheduler silently; on first gesture (pointer/key — which our presence model already
listens for), resume with a fast, quiet fade-in of the ambient bed and current call
schedule. No "click to enable sound" banner (that would be an announcement). The first
gesture is nearly always the arrival click/keypress; the mute preference in settings is
respected across sessions.

---

## 9. Accessibility surfaces

Principle (from the PRD, treated as a budget line): accessible surfaces deliver the
*actual product*, designed for charm. This work ships at launch, in the same milestone as
the features it describes — not after.

### 9.1 Screen-reader narration

- A visually-hidden `aria-live="polite"` region carries running naturalist prose,
  generated client-side from the same snapshot fields the visuals read, using the shared
  `voice` package (the same template engine as the notebook, so the product speaks with
  one voice across aviary and notebook).
- Cadence: one prose update per 30–60s at idle, chosen by a significance gate (don't
  narrate nothing). Priority queue for user-initiated events: return-greeting, offer
  reaction, settle — narrated promptly, still as observations ("pip steps to the front
  rail and takes the seed" — never "Offer successful").
- No state-list narration, no trait/mood labels, no numbers. A lint test rejects
  narration strings matching announcement patterns ("X is at", "mood:", capitals-as-
  labels).
- Controls: narration on/off in accessibility settings (default on for SR users via
  detection heuristics + explicit toggle; never auto-announce the setting). Narration
  never fights the SR queue: single live region, replace-not-append.

### 9.2 Keyboard navigation

- Tab order: top bar (account, accessibility, notebook, offer) → aviary scene → first
  bird; arrow keys move bird focus; Enter toggles listen-in; Escape exits listen-in and
  returns focus; the offer panel and settings dialogs are fully keyboard-navigable with
  focus traps and restore.
- Focus indicators: soft high-contrast outline verified against brightest (midday) and
  dimmest (night/settled) scene states — visual-regression tests screenshot both.
- Listen-in via keyboard produces the identical mix ramp as pointer.

### 9.3 Captions

- Opt-in via accessibility settings; auto-on when audio is unavailable (§8.5/§8.6).
- Generated at schedule time from call parameters via prose templates: syllable
  descriptors → phrases ("a soft three-note rise", "a low trill, paused, low trill
  again"). Same voice rules as notebook (lowercase, present-tense, specific).
- Rendered as small text chips near the calling bird, fading in/out with the call
  envelope; max two concurrent captions (chorus captions merge: "two birds trading
  calls from the back perch"); AA contrast chip background; never overlapping bird
  focus outlines.

### 9.4 Contrast and settings surfaces

- All chrome text (top bar, settings, captions, visually-displayed narration toggle
  labels, error surfaces) meets WCAG AA against its actual backgrounds — tested
  programmatically in CI with the real palette, including night/settled states.
- Settings and error surfaces use matter-of-fact voice (shared copy package; the voice
  boundary is a copy-review checklist plus a lint that flags naturalist idioms in
  system-surface files).

### 9.5 Accessibility regression prevention

- axe-core scans on every e2e flow; keyboard-only walkthrough script in CI; SR-narration
  snapshot tests; reduced-motion visual suite (§7.5); caption/grammar golden files.
- Release gate: any a11y test failure blocks ship. "Accessibility ships with the
  feature" is enforced by making the a11y suite part of each feature's definition of
  done (§11.4).

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced gates, not aspirations)

| Budget | Threshold | Enforcement |
|---|---|---|
| Initial JS | < 2MB gz at first paint (all routes needed for first frame; settings/notebook/visits code-split out) | bundle-size gate in CI |
| First bird visible | < 500ms, mid-tier mobile, 4G, warm edge | synthetic check per release |
| Idle motion | 60fps median, no jank spikes > 100ms, 5-yr-old reference laptop, 30-min session | perf lab run nightly |
| Memory | flat heap over 30-min scripted session (audio on, notebook scrolled) | CI soak test with heap snapshots |
| Snapshot payload | ≤ 8KB typical | schema budget test |
| Tick latency | p99 < 5s (alarm); typical ≪ 1s | production alert |

### 10.2 How first-bird < 500ms is achieved

- Edge-rendered HTML with inlined snapshot for signed-in sessions (60s signed TTL;
  fallback to quiet-field + fetch).
- One small blocking bundle; renderer paints birds before ornaments/settings/fonts
  finish (fonts: system stack for chrome; no webfont on the critical path).
- Static assets on CDN with long TTLs; atlas sprites lazy after first paint of birds
  (birds paint from base-layer sprites first, detail layers fade in — mid-action is
  preserved because phase continuity doesn't depend on texture detail).

### 10.3 Observability (aggregate-only, privacy-bounded)

- **Synthetic fleet**: automated browsers from common geographies/device classes on a
  schedule measuring nav timing, first-bird paint, frame timings, audio-context errors.
- **Aggregate RUM**: the same timings as distributions (no account dimension, no per-bird
  fields), plus event-ingest error rates and snapshot sizes.
- **Sim telemetry**: tick durations (p50/p99), events consumed per tick, fast-tick
  latency, no-op ratio. p99 > 5s alarms (error budget: the tick is the product's heart;
  degradation here is the "aviary running slow" failure users feel before they can name
  it).
- **Pipeline boundary (architectural, not policy)**: the telemetry stack has no read
  credentials on the simulation database; metric definitions are code-reviewed against a
  banned-field list (bird ids, account ids, event payloads, notebook text); per-account
  debugging requires break-glass access with audit logging. Aggregate presence/session
  *histograms* (no account key) are allowed per the PRD; nothing finer.
- **What we deliberately don't measure**: per-bird anything, per-account interaction
  histories, greeting frequencies per user, drift rates per user. No analytics warehouse
  ever joins the sim DB. If a dashboard would let someone reconstruct a user's
  relationship with their birds, it does not exist.

---

## 11. Testing and calibration strategy

### 11.1 Unit/property

- Drift property tests (§5.3), mood-machine determinism tests (seeded), monotonicity,
  cooldown necessity, event-fold order invariance, presence dedupe/caps, magic-link
  single-use, invite revocation timing, hard-delete completeness.

### 11.2 Golden and voice tests

- Notebook generator: seeded golden files per detector kind; banned-phrase lint
  (streaks, counts, "you visited", exclamation, announcement frames).
- Caption generator: parameter→prose goldens.
- Narration: cadence simulator asserting ≤ 1 update/30s at idle and priority-event
  latency ≤ 5s.

### 11.3 Drift calibration harness (the instrument the PRD asks for)

- A replay rig: synthetic event streams (personas: daily-10-min watcher, weekend binger,
  two-weeks-gone, mute-always, offer-enthusiast) → run the real tick code → trait
  trajectories. Tuning targets: instrument-visible ≥ 0.02 by day 7, user-visible ≥ 0.10
  by ~day 21 for the reference persona; bingers capped by daily caps; gone-two-weeks
  shows flat traits + decayed E + quick E recovery.
- "User-visible" is proxied by trait deltas that cross behavior-threshold boundaries
  (perch-zone occupancy shift, greeting-form tier change) — we calibrate against
  *behavioral* visibility, not raw numbers, because raw numbers are never shown.
- The harness runs in CI on every sim change (drift regressions are launch-blockers) and
  is the tool for any post-launch retuning.

### 11.4 Definition of done (per feature)

Code + unit tests; a11y surface implemented and its tests green (narration/captions/
keyboard/reduced-motion as applicable); voice lint clean; perf budget unaffected (or
budget change explicitly approved); privacy review for any new event field; feature flag
wired.

---

## 12. Rollout

### 12.1 Milestones (each independently demoable)

- **M0 — Foundations (wk 1–3)**: monorepo, CI with budget gates, auth (magic link),
  accounts/sessions, synthetic-UUID discipline, schema + DB-role split, event ingest +
  append-only log, tick skeleton (no-op), snapshot endpoint, client boot with quiet
  field. Exit: sign in on two browsers, both pull identical empty-state snapshot.
- **M1 — Simulation core (wk 3–7)**: drift, mood machine, expressiveness, presence
  accounting with validation, calibration harness + property tests, weather, ticks-table
  observability. Exit: harness shows calibration band hit; property tests green.
- **M2 — Renderer (wk 5–9, overlaps)**: scene graph, rigs + atlases (2 starter species
  first), snapshot reconciliation, day/night LUT, boot path with inlined snapshot,
  reduced-motion mode. Exit: first-bird < 500ms in perf lab; 60fps/30-min soak green.
- **M3 — Audio (wk 7–11)**: synth voices, motif grammars, scheduler, mixer with
  listen-in ramps, captions from grammar, fallback paths. Exit: uncanniness playtest
  (§13.3) passed internally; memory soak green with audio.
- **M4 — Interactions (wk 9–12)**: return-greeting, listen-in, offers + cooldowns,
  settle/undo, fast-tick inline snapshot, presence heartbeats. Exit: full session loop
  e2e on two devices showing identical state.
- **M5 — Notebook + narration (wk 11–14)**: detectors, governor, prose writer, golden
  tests, SR narration region + priority queue. Exit: voice review sign-off; sparsity
  governor verified against simulated hyper-active account.
- **M6 — Accounts surface + social (wk 13–16)**: settings, export, deletion (soft/hard +
  audit test), invites, visit sessions, visit log, revocation. Exit: revocation ends a
  live visit at next poll; deletion audit proves total removal.
- **M7 — Hardening (wk 16–19)**: perf lab gauntlet, a11y audits (internal + external
  SR-user sessions), soak tests, incident drills (sim-DB restore from PITR — the
  worst-case rehearsal), browser matrix (last 2 majors of Chrome/Safari/Firefox/Edge),
  unsupported-browser surface.
- **M8 — Launch**: feature flags default-on for new accounts; existing internal accounts
  migrated (bird identity preserved — migration test: same IDs, same vectors).

### 12.2 Birds-per-aviary ramp (age-based, never engagement-based)

Starter flock: 2 (system-chosen species). Offers appear by **aviary age** only:
3rd at ~3 months, 4th at ~6, 5th at ~9, 6th at ~12, 7th at ~15–18 (final pacing tuned
against retention-safe judgment, not metrics-chasing). Offers surface quietly in the
flow (a new bird arrives; naturalist framing — "a stranger has been visiting the back
fence"), never as a reward screen, never with a count ("bird 3 of 7!" is banned
phrasing). Declining/ignoring an offer expires it without consequence; it may reappear
later. The cap of 7 is enforced in the sim (an 8th cannot be offered — engine-level,
not UI-level).

### 12.3 Instrument from day one

Tick latencies and no-op ratios; fast-tick latency; event ingest errors; first-bird
timings (synthetic + aggregate RUM); frame timings; audio-context error counts; snapshot
sizes; magic-link delivery/verification success (aggregate); invite send/redeem/revoke
counts (aggregate); notebook entries generated per day (aggregate); drift-calibration
harness results per deploy. All aggregate-only per §10.3.

### 12.4 Operational requirements at launch

- PITR backups on the sim DB (personality loss is the worst failure — restore drill is
  an M7 exit criterion), cross-region replica, tested restore runbook.
- Rate limits: magic-link per email/IP, event ingest per account (anti-flood), invite
  sends per account.
- Feature flags: every user-facing surface behind a flag; reduced-motion/captions
  defaults flaggable; fast-tick wait time flaggable.
- Incident posture: sim-worker failure → clients keep rendering last snapshot (aviary
  visually alive; drift pauses) → alert on tick staleness; API failure → quiet-field
  reconnect loop with matter-of-fact surface only after sustained failure.

---

## 13. Risks

### 13.1 Drift calibration (highest product risk)

Too fast → Tamagotchi-by-accident (users move numbers by clicking; the offer cooldown
exists precisely because curiosity would saturate in a session). Too slow → screensaver
(nothing the user does matters). Silent miscalibration is the likely failure — no test
fails, users just feel it wrong. **Mitigations**: calibration harness in CI (§11.3) with
the 1-week-instrument / 3-week-visible band as a hard gate; behavior-threshold-based
visibility proxy; daily input caps; per-persona trajectories reviewed at every sim
change; post-launch retuning only via harness-verified deploys (drift-rate changes are
flagged, logged, and never retroactive — existing birds keep their history; rates apply
forward).

### 13.2 Sync correctness

The single-writer model is simple but has real edge cases: duplicate event delivery
(retries), clock-skewed clients, lease-split ticks after worker crash, multi-tab presence
double-count. **Mitigations**: idempotency keys, server-time ordering, transactional
event-claiming (replay-safe by construction — verified by a chaos test that kills
workers mid-tick and replays), presence dedupe windows, DB-role write split, and an
e2e two-device test asserting byte-identical snapshots. The banned failure — one device
overwriting another's drift — is structurally unreachable; a regression test asserts the
API schema accepts no absolute trait fields.

### 13.3 Audio uncanniness

Procedural calls can land in three failure modes: synthetic-flat (obviously a beep),
uncanny-valley (almost-a-bird, worse than beep), and phase-cancel chorus (the PRD's
recorded-loop artifact has procedural cousins — two voices with correlated schedules
beating against each other). **Mitigations**: motif grammars designed with a sound-
designer mindset (formant noise blend, micro-pitch drift, syllable elision); scheduled
decorrelation in chorus windows; recognizability suite (voiceprint invariants across
mood renderings — automated feature-distance test: same-bird renderings cluster, cross-
bird renderings separate, across N=1000 samples); human playtest gate at M3 with the
explicit question "could you pick Pip out of the chorus?" — if not, audio doesn't ship.
Also: listen-in ramps verified to never hard-cut; non-focused floor at ~0.35 verified by
mixer unit test.

### 13.4 Accessibility regressions

The designed-surface commitment decays under schedule pressure — reduced-motion rots,
narration drifts toward state-dumps, captions diverge from audio. **Mitigations**: both
render modes in the same visual suite; narration/caption golden tests + voice lints in
CI; a11y in definition of done (§11.4); external SR-user sessions in M7 and post-launch
cadence; release gate. Specific regression watches: narration cadence creep (priority
events starving idle prose), caption overlap at chorus, focus-ring contrast at night.

### 13.5 Additional named risks

- **Autoplay policy** (§8.6): silent-first-seconds could read as "broken" — mitigated by
  captions-on-while-suspended and instant resume on first gesture; playtested.
- **Presence honesty erosion**: future pressure to count "tab open" as presence (it
  inflates every engagement-adjacent chart). The three-signal conjunction is documented
  as load-bearing; the validation caps (16h/day, dedupe) make inflation visible in
  aggregate histograms; changing the definition requires changing the calibration
  harness, which forces the conversation.
- **Notebook voice decay**: template repetition breaks the spell at week 6, not day 1.
  Mitigations: no-repeat windows, variation tables, golden-file review, sparsity
  governor (fewer entries = each one lands), and a scheduled content pass each quarter.
- **Tick cost at scale**: 60s × all aviaries is fine at v1 scale; growth path is
  adaptive idle cadence (quiet hours tick slower; presence-active accounts tick at
  cadence) — designed for but not built until fleet data justifies it. The no-op fast
  path keeps baseline cost near zero.
- **Magic-link deliverability**: auth is email; provider redundancy and delivery metrics
  from day one; matter-of-fact error copy ready for expiry/replay cases.
- **Timezone/DST**: local-time anchoring with IANA zones (never offsets); mood priors
  computed in account-local time; a DST transition test in CI.
- **Privacy boundary leak**: a well-meaning dashboard joins sim data. Mitigated by
  credential separation (no read path exists), banned-field lint on metric definitions,
  and the rule stated in §10.3.

---

## 14. Interpretations and defensible calls

Ambiguities resolved without asking, per instructions:

1. **Monotonic drift vs. "ignored birds greet less often"** — resolved with the
   two-layer model: traits monotonic up; a separate recency-based *expressiveness*
   multiplier scales behavior and can decay. Both PRD sentences hold literally. (§5.3)
2. **"~1 minute tick" vs. instant offer reactions** — adaptive ticking: 60s idle cadence,
   fast-tick ≤ 2s on reaction-relevant events, API waits ≤ 1.5s to return the fresh
   snapshot inline. Single-writer invariant preserved; affective responsiveness
   preserved. (§5.1, §4.2)
3. **Presence activity window** — "a few minutes" starts at **3 minutes**, leaning
   longer per the PRD; calibrated in build against the harness; heartbeat 30s. (§5.2)
4. **Muted-audio presence** — the brief lists muting as a drift input; implemented as a
   ~40% discount on warmth/vocal credit, never negative, never below the presence floor.
   (§5.3)
5. **No realtime push** — polling (45s keepalive + visibility/gap triggers + inline
   fast-tick snapshots) meets every UX requirement; WebSocket/SSE infrastructure
   deferred as unjustified at v1's change cadence. (§6)
6. **API never transmits trait values** — snapshots carry derived rendering parameters
   (quantized plumage tier, synthesis voiceprint) so "never show numbers" holds even
   against self-inspection of traffic. (§4.1)
7. **Tech selections** — TypeScript monorepo, Postgres, Canvas 2D custom renderer, tiny
   compiled chrome framework, hand-rolled WebAudio: chosen for the 2MB/500ms budgets and
   team scale; any substitution must re-prove the budgets. (§2.3)
8. **Adoption pacing numbers** — the PRD's "few months → third bird; a year → five or
   six" is instantiated as the §12.2 schedule; tuning the constants is a product
   decision, the age-only mechanism is not.
9. **Visit duration logging** — "approximate duration" computed from visit-session
   heartbeats (start/last-seen), never from simulation events; visitors generate no
   presence. (§3.5)
10. **Sleep vs. mood** — sleeping is a night-time *activity* layered on persisting mood,
    preserving "mood doesn't reset at session start" without a seventh mood state. (§5.5)

---

## 15. What this plan deliberately refuses to build

A summary fence for contributors: no streaks, counts, scores, badges, or calendars; no
welcome/return toasts or banners; no notifications (beyond the off-by-default visit
toggle); no public surfaces, discovery, leaderboards, comments, avatars, or co-presence;
no hunger, distress, death, or decay mechanics; no recorded audio; no spinner or entry
animation; no numeric bird stats anywhere a user can see; no native apps; no drag-to-
place perches; no species rarity; no editable notebook; no analytics on per-bird data.
Each refusal is in the plan because adding the thing is easier than refusing it, and
each has a test or a lint where a test or a lint is possible.

The product is what's left after these subtractions. This plan builds exactly that.
