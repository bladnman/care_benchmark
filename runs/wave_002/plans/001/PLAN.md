# PLAN.md — Pocket Aviary v1

Comprehensive implementation plan derived from the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). This plan is for a frontier engineering team. It interprets the spec into executable engineering decisions. It does not implement the product.

Where the PRD is explicit, this plan treats the statement as a hard constraint. Where the PRD leaves a calibration to build-time, this plan proposes a defensible default and flags it as `CALIBRATION` so it can be revisited against instruments during the build.

---

## 1. Scope

### 1.1 In scope (v1)

- Single-user accounts, one canonical aviary per account, multi-device sync.
- Email magic-link authentication. No passwords, no SSO at v1.
- Two starter birds on adoption; aviary cap of seven birds, gated by aviary age.
- Six-species bird pool with procedural call grammars, procedural plumage silhouettes.
- Personality vector (5 traits) with monotonic-toward-expressive drift over days/weeks.
- Fast-timescale mood system (wary / content / curious / drowsy / alert + finalized set in implementation), persisted across sessions.
- Server-side simulation tick at ~1/min cadence, running whether or not any client is attached.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, presence accounting.
- Field notebook: auto-generated naturalist observations, sparse, read-only, scrollable indefinitely.
- Single horizontal scene, three perch zones, local-time day/night cycle, ambient weather, ambient micro-motion, top-bar chrome that fades.
- WebAudio procedural call synthesis with listen-in mix and chorus mixing.
- Accessibility surfaces as first-class designed surfaces: screen-reader narration, reduced-motion mode, call captions, WCAG AA contrast, full keyboard navigation.
- Visit-invitation social feature: host invites a visitor by email, read-only ambient view, revocable, default off, no notifications by default, visit log.
- Account export (JSON), soft-delete (30-day window) then hard delete.
- Performance budgets: <2MB gzipped initial JS bundle, <500ms time-to-first-bird on mid-tier mobile 4G, 60fps idle on a 5-year-old laptop, no memory growth over 30 minutes.
- Synthetic account UUID; email stored once, encrypted; PII boundary hard-enforced in pipelines.

### 1.2 Out of scope (respects `non_goals.md`)

- No native mobile apps (iOS, Android). Web-only.
- No gamification of any flavor: no achievements, streaks, levels, scores, badges, counters, green-dot calendar, XP, rank, tier, milestones. Not even an opt-in toggle.
- No Tamagotchi mechanics: no death, no hunger, no decay meter, no visible suffering on neglect.
- No social network: no profiles, follows, feed, discovery, leaderboards, comments on visits, friend-of-friend chains.
- No push notifications, email notifications about the aviary, or any ping surface that reaches the user. (The single exception: the opt-in host-side visit-notification toggle, off by default — that is a settings choice by the host, not a system-driven ping.)
- No recorded audio path. No audio fallback that uses canned loops.
- No client-side tick, no client-owned personality state, no last-write-wins on personality.
- No co-presence during visits (no shared cursor, no "your friend is here too" overlay).
- No public aviary directory, no explore surface, no ranking.
- No payments, no billing, no tiers.
- No compatibility shims for very old browsers (last two major versions of Chrome/Safari/Firefox/Edge only).

### 1.3 Calibration flags (decisions deferred to build, with defaults)

- `CALIBRATION-1`: tick cadence. Default **60 seconds**. The PRD says ~once per minute; we pick 60s and confirm with mood-transition observability during the staging ramp.
- `CALIBRATION-2`: presence activity window. Default **180 seconds** (three minutes) for "pointermove or keypress occurred in the last few minutes." Leans toward the longer side, per `interactions.md`.
- `CALIBRATION-3`: drift low-pass filter constants. Target: measurable drift in instruments after ~1 week of regular visits, visible to the user after ~3 weeks. Defaults proposed in §6.
- `CALIBRATION-4`: offer cooldown per bird. Default **180 seconds** (a few minutes).
- `CALIBRATION-5`: settle undo window. Default **5 seconds** (PRD-specified).
- `CALIBRATION-6`: notebook entry frequency. Default **one entry per ~2–4 days** for a regularly-visited aviary, plus event-triggered entries. Tuned to preserve sparsity.
- `CALIBRATION-7`: magic-link rate limit. Default **5 per email per hour**.
- `CALIBRATION-8`: visit invite expiration. **30 days** (PRD-specified).
- `CALIBRATION-9`: magic-link TTL. **15 minutes** (PRD-specified).
- `CALIBRATION-10`: soft-delete window. **30 days** (PRD-specified).

---

## 2. Architecture

### 2.1 High-level service shape

Four services plus a CDN edge. Each is independently deployable and independently scalable.

1. **Edge / CDN** — serves the static JS bundle, the HTML shell, and the initial state snapshot payload inlined into the HTML for the time-to-first-bird budget. Static assets live here. No business logic.
2. **API service** ("aviary-api") — the public-facing REST + Server-Sent Events endpoint that browsers talk to. Handles auth, snapshot pulls, event-log appends, visit token resolution, account/export/delete flows. Stateless; horizontally scalable.
3. **Simulation service** ("aviary-sim") — the only writer of personality and mood state. Runs the tick. Reads the event log, computes deltas, writes canonical state. Single-writer per account (see §7).
4. **Async / email service** ("aviary-mail") — sends magic-link emails, export-link emails, visit-invitation emails. Decoupled via a queue so the API never blocks on SMTP.
5. **(Operational) Telemetry service** — aggregate-only metrics pipeline. Strictly separated from the simulation database (see §11).

### 2.2 Datastores

- **Postgres (primary)** — account records, bird records, personality vectors, mood state, notebook entries, visit invitations, visit log, session-token registry, magic-link tokens. Encrypted-at-rest. The email column is encrypted application-side as well (envelope encryption with KMS) so that even a DB dump does not expose PII.
- **Event log** — append-only, ordered, per-account interaction-event log. Implemented as a Postgres table with an autoincrementing per-account sequence, or as a partition-by-account Kafka topic with the simulation service as the single consumer group. **Postgres table is the v1 default** (simpler operationally; we are not at the scale where Kafka pays for itself). The schema is in §5.2.
- **Object storage (S3-compatible)** — account export JSON blobs (signed-URL delivery), any large immutable assets.
- **Redis (optional, ephemeral)** — rate-limit counters, magic-link attempt tracking, idempotency keys. No persistent state lives here.

### 2.3 Client/server split

The browser is a **renderer and an event forwarder**. It never owns state.

- Reads: snapshot pulls (REST + SSE for live deltas while attached).
- Writes: appends to the server-side event log only.
- Never: computes personality deltas, never writes mood, never ticks.
- The client may stop rendering when the tab is hidden (Page Visibility API) but never simulates; on visibility regain it pulls a fresh snapshot and resumes interpolation.

### 2.4 Render pipeline boundary

The render pipeline lives entirely client-side and is fed by snapshots. The boundary contract between server and renderer is the **snapshot schema** (§5.4). Anything the renderer needs to draw the next frame is either (a) in the snapshot, (b) derivable client-side from the snapshot (interpolation, idle micro-motion offsets, ambient leaf/feather ornaments, day/night palette from local time), or (c) static config (species pool, perch geometry, palette tables) shipped in the bundle.

---

## 3. Data Model

### 3.1 `accounts`

| field | type | notes |
|---|---|---|
| `account_id` | UUID (pk) | Synthetic. Generated at signup. The only identifier used downstream. |
| `email` | text (encrypted) | Stored once. Encrypted at the application layer. |
| `email_normalized` | text (blind-index) | HMAC of normalized email, for uniqueness / lookup. Not reversible. |
| `created_at` | timestamptz | |
| `soft_deleted_at` | timestamptz null | Set on delete request. Hard-purged after 30 days. |
| `settings` | jsonb | Accessibility prefs, visit-notification toggle, etc. |

Email is **never** referenced as an identifier in any other table, log, partition key, or telemetry dimension. The synthetic UUID is the only join key.

### 3.2 `birds`

| field | type | notes |
|---|---|---|
| `bird_id` | UUID (pk) | Stable internal identifier. Never reused; never reset. |
| `account_id` | UUID (fk) | |
| `species_id` | text | References species pool (static config). |
| `name` | text | User-assigned, renameable. |
| `adopted_at` | timestamptz | |
| `birth_order` | int | 0/1 for starters; subsequent birds get higher ints. |

### 3.3 `personality_vectors`

One row per bird, mutated only by the simulation service.

| field | type | notes |
|---|---|---|
| `bird_id` | UUID (fk, pk) | |
| `boldness` | real | Normalized [0,1]. |
| `social_warmth` | real | |
| `vocal_frequency` | real | |
| `plumage_saturation` | real | |
| `curiosity` | real | |
| `updated_at` | timestamptz | Last tick that wrote. |
| `version` | bigint | Monotonic per-bird counter, incremented on every delta. Used for audit; not used for conflict resolution (single writer). |

Personality values are **never exposed** to the client in numeric form. The snapshot carries derived rendering hints (perch preference, call motif weights, palette saturation multiplier) but not the raw traits. This rule is enforced at the snapshot serializer, with a unit test that asserts no numeric trait field appears in the serialized snapshot JSON.

### 3.4 `mood_state`

| field | type | notes |
|---|---|---|
| `bird_id` | UUID (fk, pk) | |
| `mood` | text | Enum: `wary`, `content`, `curious`, `drowsy`, `alert` (finalized set may extend during build). |
| `mood_intensity` | real | [0,1]. How strongly the mood is expressed. |
| `mood_entered_at` | timestamptz | |
| `updated_at` | timestamptz | |

Mood persists across sessions. The mood at session-end is the mood at the next session-start, modulo whatever the tick has computed in between.

### 3.5 `interaction_events` (the event log)

Append-only. This is the substrate the simulation tick consumes.

| field | type | notes |
|---|---|---|
| `event_id` | bigint (pk) | Global autoincrement. |
| `account_id` | UUID (fk, indexed) | |
| `bird_id` | UUID null | Null for aviary-level events (settle, presence ping). |
| `event_type` | text | `presence_ping`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song`, `offer_still_pool`, `settle`, `return_greeting_observed`, `visit_started`, `visit_ended`. |
| `event_payload` | jsonb | Type-specific details (duration, focus bird, offer kind, absence_seconds). |
| `client_ts` | timestamptz | Client's claimed time. |
| `server_ts` | timestamptz | Server-received time. Used by the tick. |
| `device_session_id` | UUID | Per-device session identifier. |

The event log is consumed in `server_ts` order. The simulation service advances a per-account cursor; crashes restart from the cursor.

### 3.6 `notebook_entries`

| field | type | notes |
|---|---|---|
| `entry_id` | UUID (pk) | |
| `account_id` | UUID (fk, indexed) | |
| `written_at` | timestamptz | |
| `body` | text | Naturalist prose. Lowercase, present-tense. |

Read-only from the client. No edit, no delete, no annotate. Old entries never archived.

### 3.7 `visit_invitations`

| field | type | notes |
|---|---|---|
| `invitation_id` | UUID (pk) | |
| `host_account_id` | UUID (fk) | |
| `visitor_email` | text (encrypted) | |
| `visitor_email_blind_index` | text | HMAC for lookup. |
| `state` | text | `outstanding`, `active`, `revoked`, `expired`, `consumed`. |
| `created_at` | timestamptz | |
| `expires_at` | timestamptz | `created_at + 30 days`. |
| `revoked_at` | timestamptz null | |
| `consumed_at` | timestamptz null | First visit. |

### 3.8 `visit_log`

| field | type | notes |
|---|---|---|
| `visit_id` | UUID (pk) | |
| `invitation_id` | UUID (fk) | |
| `host_account_id` | UUID (fk, indexed) | |
| `visitor_email_blind_index` | text | |
| `started_at` | timestamptz | |
| `last_seen_at` | timestamptz | Updated while the visit is active. |
| `ended_at` | timestamptz null | |

### 3.9 `device_sessions`

| field | type | notes |
|---|---|---|
| `device_session_id` | UUID (pk) | |
| `account_id` | UUID (fk) | |
| `issued_at` | timestamptz | |
| `last_seen_at` | timestamptz | |
| `user_agent_hint` | text | Coarse, non-PII (e.g., "Safari / macOS"). For the device-revocation list. |
| `revoked_at` | timestamptz null | |

### 3.10 `magic_link_tokens`

| field | type | notes |
|---|---|---|
| `token_id` | UUID (pk) | |
| `email_blind_index` | text | |
| `token_hash` | text | Hashed magic-link secret. |
| `created_at` | timestamptz | |
| `expires_at` | timestamptz | `created_at + 15 minutes`. |
| `consumed_at` | timestamptz null | Set on first use; further uses rejected. |

### 3.11 Static config: species pool

Shipped in the bundle and mirrored server-side. Six species. Each species defines:

- `silhouette_id` (procedural drawing parameters or SVG path).
- `default_plumage_palette` (named colors, saturation range).
- `call_motif_library` — ordered set of motifs, each motif being a small parameterized synthesis program (see §8).
- `default_trait_seeds` — distribution from which new birds are seeded.

---

## 4. API Surface

All endpoints are JSON over HTTPS. Authenticated endpoints require a `Bearer <device_session_token>` header. Tokens are revocable.

### 4.1 Auth

- `POST /v1/auth/request-magic-link` — body `{email}`. Idempotent within a short window. Rate-limited per email and per IP. Always returns 202 (never reveal whether an account exists). Email is sent by `aviary-mail`.
- `GET /v1/auth/verify-magic-link?token=...` — verifies token, invalidates it, issues a `device_session_id` + token. Sets HTTP-only secure cookie. Returns the account snapshot URL.
- `POST /v1/auth/email-change/request` — body `{new_email}`. Sends a verification link to the new address. Old email continues to work until verification.
- `GET /v1/auth/email-change/verify?token=...` — commits the email change.
- `POST /v1/auth/sign-out` — revokes the current device session.
- `POST /v1/auth/devices/{device_session_id}/revoke` — revokes a specific device session.

### 4.2 State

- `GET /v1/aviary/snapshot` — returns the current canonical snapshot (§5.4). Small (kilobytes). Cacheable on the edge for a short TTL only when no client is attached (we use `Cache-Control: private` to keep it device-scoped).
- `GET /v1/aviary/stream` — Server-Sent Events. Pushes incremental snapshot deltas while a client is attached. Reconnect uses `Last-Event-ID`.
- `POST /v1/aviary/events` — body is a batch of interaction events. Idempotent via client-supplied idempotency keys. Appends to the event log. Returns `{accepted_event_ids, snapshot_version}`.

### 4.3 Interactions (all routed through `POST /v1/aviary/events`)

The client does not call separate endpoints per interaction; it writes typed events to the event log. The simulation tick picks them up. This keeps the client dumb and the writer single.

However, certain interactions have a synchronous flavor — the return-greeting is computed immediately on snapshot pull, and offer reactions may want sub-tick feedback. For these, the API exposes:

- `POST /v1/aviary/listen-in` — body `{bird_id, action: "start"|"end"}`. Updates a transient per-session mix preference used by the audio pipeline hint in subsequent snapshots. Also writes a `listen_in_*` event to the log for drift.
- `POST /v1/aviary/offer` — body `{bird_id, offer_kind}`. Validates cooldown server-side (authoritative), writes an `offer_*` event, and returns an immediate reaction descriptor (which pose the bird adopts, whether it approaches). The reaction is computed deterministically from the current mood/personality plus a per-bird PRNG seed; the tick later reconciles any drift.
- `POST /v1/aviary/settle` — body `{undo?: boolean}`. Writes a `settle` event. Returns the lighting target state. The 5-second undo window is a client-side concern; if the client sends `undo:true` within the window, the settle event is marked reverted in the log.

### 4.4 Notebook

- `GET /v1/notebook/entries?before=<cursor>&limit=...` — paginated, descending by `written_at`. Indefinite scroll.

### 4.5 Account

- `POST /v1/account/export` — queues an export. Email with signed URL.
- `POST /v1/account/delete` — soft-deletes. Recoverable for 30 days.
- `POST /v1/account/recover` — undeletes during the soft-delete window.
- `GET /v1/account/settings` / `PATCH /v1/account/settings` — accessibility prefs, visit-notification toggle, etc.
- `GET /v1/account/devices` — list device sessions.

### 4.6 Visits

- `POST /v1/visits/invitations` — body `{visitor_email}`. Creates an invitation. Sends the visitor email.
- `GET /v1/visits/invitations` — host's outstanding + recent invitations.
- `POST /v1/visits/invitations/{id}/revoke` — immediate revocation.
- `GET /v1/visits/log` — host's visit log.
- `POST /v1/visits/resolve?token=...` — visitor-side. Resolves an invitation token. Returns a read-only snapshot stream URL and a visitor session token that has no write permissions. If revoked or expired, returns a matter-of-fact error surface.

Visitor sessions hit a read-only subset:

- `GET /v1/visits/{visit_id}/snapshot` — same snapshot shape as the host's, minus any fields the visitor must not see (host's visit log, host's settings, anything beyond the aviary scene).
- `GET /v1/visits/{visit_id}/stream` — SSE deltas.
- No `POST /v1/aviary/events` for visitors. Visitor presence is never recorded.

---

## 5. Snapshot Schema

### 5.1 Goals

- Small (kilobytes).
- Carries everything the renderer needs to draw the next frame and the audio pipeline needs to schedule the next calls.
- Carries no numeric personality traits (hard rule).
- Versioned, monotonic.

### 5.2 Top-level shape

```json
{
  "snapshot_version": 12345,
  "account_id": "uuid",
  "aviary_age_days": 187,
  "local_tz_offset_minutes": -240,
  "local_time_of_day": "morning",
  "lighting": { "palette": "morning", "intensity": 0.78 },
  "weather": { "kind": "rain", "intensity": 0.3, "ends_in_seconds": 90 },
  "birds": [ /* per-bird state, see 5.3 */ ],
  "ambient_seed": 4291,
  "notebook_hint": { "unread_count": 0 },
  "top_bar": { "fade_state": "visible" },
  "active_offer_cooldowns": [ { "bird_id": "uuid", "kind": "seed", "cooldown_remaining_s": 42 } ],
  "settled": false
}
```

### 5.3 Per-bird entry

```json
{
  "bird_id": "uuid",
  "name": "Pip",
  "species_id": "warbler_a",
  "perch": { "zone": "front", "x": 0.42, "y": 0.61 },
  "facing": "left",
  "pose": "preening",
  "pose_started_at_server_ts": "...",
  "motion_target": { "kind": "hop_to", "perch_zone": "middle", "eta_seconds": 2.1 },
  "mood_render_hint": { "posture": "fluffed", "scan_rate": 0.2 },
  "call_render_hint": {
    "next_call_in_seconds": 4.2,
    "motif_id": "warble_a",
    "pitch_offset": 1.02,
    "tempo": 0.9,
    "intensity": 0.6
  },
  "plumage_render_hint": { "saturation_multiplier": 1.08, "palette_id": "warbler_a_default" },
  "return_greeting_state": null
}
```

The `*_render_hint` fields are derived server-side from the personality + mood + species config. They are deliberately not the raw trait values; they are the minimum the renderer needs. A `mood_render_hint.posture` of `"fluffed"` is a categorical, not a number.

### 5.4 Versioning

`snapshot_version` is monotonic per account, incremented on every tick write. Clients send it back on `POST /v1/aviary/events` so the server can detect stale writes (the event is still appended — events are append-only — but the server can choose to recompute the immediate reaction if the version the client acted on is stale).

---

## 6. Simulation Engine Design

### 6.1 Tick loop

The simulation service runs a single goroutine/worker per account (sharded across a worker pool, but never two workers on the same account simultaneously — enforced by a per-account lease in Redis or a Postgres advisory lock).

Per tick (every 60s by `CALIBRATION-1`):

1. Acquire per-account lease.
2. Read cursor (last consumed `event_id` for this account).
3. Fetch new events from the event log in `server_ts` order, since the cursor.
4. For each bird in the aviary:
   a. Compute presence-time accumulated since last tick (sum of `presence_ping` event durations, with the conjunctive definition enforced client-side — see §6.4).
   b. Compute drift deltas from presence-time, listen-in durations, offers accepted, settle signals.
   c. Apply monotonic-toward-expressive rule (clamp deltas to be ≥ 0 for traits that drift up; never subtract).
   d. Update personality vector in DB, increment `version`.
5. For each bird: compute mood transition based on recent events, time-of-day (from `local_tz_offset_minutes`), ambient weather, and other birds' moods.
6. Maybe generate a notebook entry (per `CALIBRATION-6`).
7. Update the canonical snapshot row (in a `snapshots` table or materialized in the `birds`/`mood_state` rows — the snapshot is computed on demand from current rows).
8. Advance cursor. Release lease.

### 6.2 Drift function (low-pass filter, monotonic toward expressive)

Per trait `t` for bird `b` at tick `k`:

```
delta_t_b(k) = clamp_low(α_t * signal_t_b(k), 0, MAX_STEP_t)
new_t_b = clamp(current_t_b + delta_t_b(k), 0, 1)
```

Where:

- `signal_t_b(k)` is the weighted input for trait `t`:
  - **Boldness**: `w_presence * presence_s + w_offer_near * offer_near_count + w_offer_accepted * offer_accepted_count`
  - **Social warmth**: `w_presence * presence_s + w_listen * listen_in_s + w_greet_response * greet_response_count`
  - **Vocal frequency**: `w_presence * presence_s + w_listen * listen_in_s + w_chorus_join * chorus_join_count`
  - **Plumage saturation**: `w_presence * presence_s`
  - **Curiosity**: `w_offer_accepted * offer_accepted_count + w_listen * listen_in_s`
- `α_t` is the per-trait low-pass coefficient. Default proposal:
  - `α_boldness = 0.00018` per presence-second (so ~10 minutes of presence per day for 7 days ≈ +0.08, "measurable in instruments" territory).
  - `α_social_warmth = 0.00022`
  - `α_vocal_frequency = 0.00020`
  - `α_plumage = 0.00010` (slower; visible drift should be the slowest signal).
  - `α_curiosity = 0.00015`
- `MAX_STEP_t` is a per-tick cap (default `0.005`) to prevent a single heavy session from moving a trait too far.
- All weights and α values are `CALIBRATION-3` and should be validated by an instrumented staging harness that runs simulated weeks of presence against synthetic accounts.

The asymmetry rule: `delta_t_b(k)` is clamped at the floor to 0. **No input ever subtracts.** Settle does not push drift; it only ends the presence window cleanly. Neglect produces no signal — the bird's traits simply stop increasing, which manifests as "quieter over time because the bird greets less often," not "wary over time."

### 6.3 Mood transitions

Mood is a small finite-state machine. Inputs:

- `time_of_day` bucket (early_morning, morning, midday, afternoon, dusk, night) — derived from `local_tz_offset_minutes`.
- Recent events in the current tick window: offers accepted, presence detected, weather active.
- Other birds' current moods (wary spreads; chorus emerges from co-occurring high-vocal-frequency calls).
- The bird's own personality (high-boldness bird resists wary).

Transitions are stochastic: per tick, each bird has a transition probability per target mood. The probabilities are computed from a small matrix per `(time_of_day, weather, personality_bucket)`. The matrix is finalized in implementation but the structure is:

- Drowsy is more likely near dusk and at night.
- Alert is more likely in early morning.
- Content is the default attractor when nothing is happening and the bird has positive recent presence.
- Wary spreads from other wary birds with probability proportional to social warmth (high-warmth birds pick up wary faster; high-boldness birds resist it).
- Curious is elevated for ~1 minute after an offer is accepted.

Mood persists across sessions (the mood at session-end is the mood at session-start). The tick continues transitioning mood whether or not any client is attached.

### 6.4 Presence accounting

Presence is computed client-side under the strict conjunctive rule (visibility=visible AND document has focus AND a pointermove/keypress in the last `CALIBRATION-2` seconds). The client emits `presence_ping` events to the server every 30 seconds while all three hold. If any one fails, no ping is emitted; the presence window ends.

The server does not re-derive presence. It trusts the client's pings under the assumption that the client's enforcement is correct (audited by the CI test in §11). A `presence_ping` event's payload includes the duration since the last ping (so the server sums presence-time accurately even across ping jitter).

### 6.5 Return-greeting computation

On snapshot pull (or stream connect) after a session-start signal, the server determines whether to schedule a return-greeting:

- Identify the greeter bird: weighted random across birds, weighted by `boldness * social_warmth` and modulated by current mood (a wary bird is less likely to greet).
- Identify absence length from the last `presence_ping` end to the current `session_start` event.
- Pick a greeting variant from the procedural variant space, parameterized by (boldness, mood, absence_bucket). Absence buckets: `<2min`, `2min–1h`, `1h–1d`, `>1d`.
- The greeting is encoded in the snapshot as a `return_greeting_state` on the greeter bird, with start time, duration, and a procedural seed. The renderer plays it out.

Multiple-bird greetings stagger by a randomized small offset (default 0.4–1.5s), never firing in unison.

### 6.6 Notebook generation

A notebook entry is generated when (a) an interesting event occurred since the last entry and (b) at least `CALIBRATION-6` (default ~2 days) have passed since the last entry, or (c) a "noteworthy" event occurred (first-time-this-week greeting order, a chorus event, a weather-mood interaction, a long quiet stretch).

Entry text is generated from a small templating system that produces naturalist prose. The templates are deliberately not generic; they reference the specific bird, the specific moment, and avoid gamification language. The generation library is part of the simulation service, not the client, so that the voice is consistent across devices and the client bundle stays small.

Examples (matching `interactions.md` style):

- `{day} — {bird_a} greeted before {bird_b} today, first time this week.`
- `{bird_b} is fluffed against the cool air, watching the back perch. low calls only.`
- `a long stretch of quiet this morning. {bird_a} preened for several minutes without looking up.`

### 6.7 Bird-bird interaction

The tick computes simple bird-bird effects:

- A bird in `wary` mood raises the wary-transition probability for other birds proportional to their `social_warmth`.
- Two birds with `vocal_frequency > threshold` both calling in the same tick window produce a `chorus_join` event, which feeds back into the drift for both.
- A bird's call can prompt a response from another bird with probability proportional to the responder's `social_warmth` and the caller's `vocal_frequency`.

These are cheap to compute per tick; they keep the aviary feeling like a small social system rather than parallel independent birds.

### 6.8 Stable identity

`bird_id` is immutable. Renames update `birds.name` only. Sync, species-pool changes, or any future migration never replaces a `bird_id`. There is no "regenerate bird" code path. A unit test asserts that no write path in the simulation service ever creates a new `bird_id` for an existing bird.

---

## 7. Sync Model

### 7.1 Single canonical state

The server is the only writer of personality, mood, and notebook state. Multi-device sync is a property of this architecture, not a feature: both devices read the same record.

### 7.2 No last-write-wins

Clients never submit absolute personality values. They submit events. The simulation service consumes the events in `server_ts` order and computes additive deltas. There is no code path where a client sends `set boldness to X`.

### 7.3 Conflict surface

The only place where multi-device writes can "conflict" is in the event log itself, and that's resolved by the append-only single-writer model: events are appended as they arrive, ordered by `server_ts`. The simulation service processes them in that order. Two devices writing events at the same time is fine — both appends succeed, the tick picks them up in order.

The client-side snapshot version is used only for the immediate-reaction endpoints (`offer`, `listen-in`): if the client's snapshot version is older than the server's current version, the server may return a "state has changed, please re-pull" hint for that interaction. This is rare and graceful.

### 7.4 Conflict / error voice

All sync/auth/account errors drop into the matter-of-fact voice (per `accounts_sync.md`):

- `We couldn't sign you in. The link may have expired. Try requesting a new link.`
- `Your session timed out. Sign in again to keep watching.`
- `Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.`

These strings live in a separate i18n namespace from the naturalist prose so that the two voices never accidentally bleed.

---

## 8. Frontend Rendering Pipeline

### 8.1 Stack

- A modern web framework that supports aggressive code-splitting and a small initial bundle. The exact framework choice is an implementation detail, but the team should pick one that supports streaming HTML, edge rendering of the initial snapshot payload, and fine-grained lazy loading. Defaults: a React/Next-style stack, or a lighter framework if bundle size becomes a pressure.
- The renderer is a custom 2D canvas-based scene compositor (not DOM). Canvas gives us the frame budget for 60fps idle motion on a 5-year-old laptop, and lets us batch redraws. DOM is used only for the top bar and the settings surfaces.
- A WebWorker for the audio pipeline (so audio scheduling never blocks on main-thread jank).

### 8.2 Scene composition

Layers, back to front:

1. Sky (procedural gradient from local time).
2. Background foliage (static SVG / procedural shapes, subtle parallax).
3. Back perch zone.
4. Middle perch zone.
5. Front perch zone.
6. Foreground branch / occasional leaf drift (client-side ornament).
7. Captions overlay (DOM, for captions only).
8. Focus indicator overlay (DOM, keyboard focus only).

Birds are drawn at their perch zone's depth, ordered by perch `y`.

### 8.3 Idle micro-motion

Birds are never still. Each bird has a continuous idle animation driven by:

- `pose` (current categorical pose from the snapshot).
- `mood_render_hint.posture` (e.g., fluffed, sleek, low).
- A per-bird PRNG seed (stable across sessions for the same bird, so Pip's idle is recognizably Pip's).
- `mood_render_hint.scan_rate` (how often the bird scans the scene).

Idle motion is composed of small procedural deformations: head-tilt, weight-shift, preen cycles, scan sweeps. The renderer does not loop a fixed animation; it generates motion from parameters each frame. A wary bird scans more; a content bird preens; a curious bird tilts toward sounds.

### 8.4 Transitions and interpolation

The client interpolates between snapshots. A bird at perch A in snapshot N and perch B in snapshot N+1 is rendered moving smoothly between them, not teleporting. Interpolation parameters are derived from the `motion_target` field in the per-bird entry, which gives the renderer an ETA and target.

### 8.5 Reduced-motion mode

Triggered by `prefers-reduced-motion` or the user's accessibility settings. The renderer switches to a cross-fade mode:

- Micro-motion is replaced by slow cross-fades between still poses.
- Flight transitions become cross-fades between perches rather than animated paths.
- Ambient leaf drift is removed.
- Ambient color shifts (day to evening) remain, slowed.
- Calls still play at full quality. Birds still drift. Mood still changes. The notebook still notices things.

This is a designed surface, not a fallback. The cross-fade renderer has its own aesthetic and is shipped as a first-class path.

### 8.6 Loading state

The first frame the user sees is the aviary mid-motion. To achieve this:

- The HTML shell includes an inlined initial snapshot payload (small, ~few KB), fetched from the edge at HTML generation time.
- The renderer is in the critical path; it starts drawing the first bird as soon as the bundle parses and the inlined snapshot is available.
- If the snapshot is not yet available (cold cache, slow connection), the loading state is a **quiet field** — soft sky color, perhaps one or two faint motion cues — **not a spinner**. The quiet field reads as "the aviary catching up," not "the product loading."
- There is no entry animation, no fade-from-static, no "ready" pop.

### 8.7 Empty-aviary state

Between adoption flow and the first bird appearing, the aviary shows the same quiet field. The first bird enters with a soft fly-in to its starting perch. From that point forward, the user never sees an empty aviary.

### 8.8 Top bar fade

After a few seconds (default 3s) of cursor stillness and no keyboard activity, the top bar fades to ~5% opacity. Any `pointermove` or keypress restores it to full opacity. In reduced-motion mode, the fade is a slow cross-fade rather than a quick transition.

### 8.9 Responsive scene

The scene compresses horizontally on narrow viewports without cropping any bird out of frame. On wide viewports, the scene widens with more space between perches. Aspect ratio is preserved such that all birds are visible at all times.

### 8.10 Day/night cycle

Computed client-side from the user's local time (no server round-trip needed for the palette; the snapshot carries `local_tz_offset_minutes` for sanity). The palette transitions are continuous, not stepped. At full night, most birds are settled; one nightjar-like species remains active.

### 8.11 Ambient weather

Weather is snapshot-driven (`weather` field). The renderer draws rain/wind ornaments client-side. Weather is rare (a few times a week) and short. The renderer does not store weather state; it draws from the snapshot.

### 8.12 Ambient leaf and feather drift

Pure client-side rendering ornaments, generated at idle cadence from a per-scene PRNG. Not part of the snapshot. Not part of the tick.

---

## 9. Audio Pipeline

### 9.1 Procedural call synthesis

Each bird species has a `call_motif_library` — a small set of parameterized motifs. A motif is a WebAudio synthesis program: oscillators, envelopes, filters, and a small modulation matrix. A call is one motif (or a short sequence of motifs) with parameters varied at runtime by:

- The bird's `vocal_frequency` trait (affects tempo and call rate).
- The bird's current mood (affects pitch offset, intensity, motif selection).
- A per-call PRNG seed (so the same bird never produces an identical call twice).
- The chorus context (if other birds are calling, the bird may adjust timing to interleave rather than collide).

### 9.2 Call scheduling

The audio worker schedules calls based on `call_render_hint.next_call_in_seconds` from the snapshot. When a call fires, the worker picks a motif and parameters, synthesizes it, and mixes it into the chorus bus.

### 9.3 Chorus mixing

All birds' calls route to a single chorus bus. The chorus mix is automatic — multiple birds calling simultaneously produce a real chorus because the calls are procedurally varied (no phase-canceling artifacts from layered loops).

### 9.4 Listen-in mix

When a bird is focused (listen-in), the audio worker applies a slow gain ramp:

- Focused bird's bus gain rises from baseline (default 0.6) to focused (default 1.0) over ~1.5 seconds.
- Other birds' bus gains drop from baseline to ambient (default 0.25) over the same ramp.
- Other birds **never go silent**. The listen-in mix is a re-balance, not a mute.

On disengage (focus the same bird again, focus a different bird, click empty space, keyboard focus leaves), the ramp reverses over the same duration.

### 9.5 WebAudio fallback

If WebAudio is unavailable, the aviary plays in graceful silence with captions on by default. **No recorded audio fallback path exists.** This is unconditional.

### 9.6 Settle audio

The settle gesture quiets calls globally over a few seconds (gain ramp on the chorus bus to ~0.3). On undo, the ramp reverses.

### 9.7 No memory growth

Audio buffers are reused. The worker allocates a fixed pool of `AudioBufferSourceNode` and `GainNode` objects at startup and recycles them. No per-call allocation that isn't freed. This is enforced by a CI test that runs a 30-minute synthetic session and asserts no memory growth.

---

## 10. Accessibility Surfaces

### 10.1 Screen-reader narration

A live region (`aria-live="polite"`) holds the current narration prose. The narration is generated from the same snapshot the visual surface reads from, in the same naturalist voice as the notebook. Cadence: one prose update per 30–60 seconds at idle, faster on user-initiated events (return-greeting, offer reaction, settle).

Narration is prose, not state-list. **Never** "Pip is at perch 2" or "Wren mood: content." Always naturalist prose:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

The narration generator is shared between the visual captions and the screen-reader surface so the voice is identical.

### 10.2 Captions for calls

Opt-in via accessibility settings. When on, each call produces a short prose caption near the calling bird:

> a soft three-note rise
> a low trill, paused, low trill again
> a single sharp call from the back perch

Captions fade in and out with the call. The caption text is generated from the procedural call grammar at runtime — it matches what was actually played.

### 10.3 Keyboard navigation

- Tab moves through the top bar items.
- Tab into the aviary scene focuses the first bird.
- Arrow keys move focus between birds.
- Enter triggers listen-in on the focused bird.
- Escape exits listen-in.
- The offer affordance is reachable from the top bar (keyboard shortcut) and is itself fully keyboard-navigable.
- The settle gesture is reachable from the top bar.
- Focus indicators are visible against the aviary background — a soft high-contrast outline that reads against both bright and dim aviary states.

### 10.4 Reduced-motion mode

See §8.5. It is a designed surface, not a fallback.

### 10.5 Contrast

All user-copy text passes WCAG AA. The aviary scene itself does not contain user copy except in the top bar (and captions, which are user copy and must pass AA).

### 10.6 Accessibility ships with v1

Not v1.1. Not a retrofitted fix. The accessibility surfaces are part of the v1 launch.

---

## 11. Performance Budgets and Observability

### 11.1 Budgets

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS bundle (gzipped) | < 2MB | CI check on the production build. |
| Time to first bird visible (mid-tier mobile, 4G) | < 500ms | Synthetic test from common geographies. |
| Idle motion frame rate (5-year-old laptop, 30-min session) | 60fps | Synthetic test. |
| Memory growth over 30 minutes | 0 | CI test. |
| Simulation tick latency p99 | < 5s | Alarm. |
| Snapshot payload size | < 32KB | CI assertion on the snapshot serializer. |

### 11.2 Observability

- Synthetic performance checks: a fleet of automated browsers running the aviary on a schedule from common geographies, capturing page-load timings, first-bird-render timings, render-frame timings, audio-context errors.
- Aggregate-only Real User Monitoring: page-load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies.
- **Privacy boundary, hard**: none of this telemetry includes per-bird state, per-account interaction history, or anything that could reconstruct a user's relationship with their aviary. The telemetry pipeline never reads the simulation database. The simulation database is never read by the analytics warehouse. ML training, if it ever exists, never receives per-bird fields.
- Error budget: simulation-tick latency p99 alarms at 5s.

### 11.3 What we deliberately don't measure

- Per-bird personality values, in aggregate or per-account.
- Per-account interaction histories, in aggregate or per-account.
- Anything that could be aggregated into a "most-visited aviaries," "longest-running aviary," or "most birds" stat. We do not even compute the underlying metrics for this purpose, so that the architectural absence makes the feature's reappearance harder rather than easier.

---

## 12. Privacy

- Per-bird interaction events are stored only to drive that user's own simulation. Never aggregated for model training. Never used to build recommendation features. Never shared with third parties. Never used to inform population-level analysis.
- Aggregate telemetry is operational only: request counts, latencies, error rates, anonymized session-duration histograms.
- The line between "is this account having errors" (allowed) and "what is this account's bird doing" (not allowed for any aggregate purpose) is hard, named, and respected at the data-pipeline level.
- The simulation database is never read by the analytics warehouse. The telemetry pipeline never touches per-account interaction state.
- The privacy policy lives in account settings as a link, in plain text, naming the aggregate categories and explicitly excluding per-bird interaction state.

---

## 13. Rollout

### 13.1 Shipping v1

- Closed beta with a small cohort (dozens of users) for ~2 weeks. Instrumentation: aggregate perf budgets, drift calibration sanity (does a synthetic week of presence produce measurable drift? does a synthetic 3 weeks produce visible drift?).
- Open beta (hundreds of users) for ~2 weeks. Watch for: sync correctness under multi-device use, audio uncanniness reports, accessibility regressions, memory-growth anomalies.
- General availability.

### 13.2 Bird-count ramp

- New accounts start with 2 birds.
- The third bird becomes available based on aviary age (default: aviary must be at least ~60 days old).
- Subsequent birds: ~90 days between availability offers, capped at 7.
- These intervals are `CALIBRATION` and should be tuned against retention and per-bird recognizability feedback during beta.

### 13.3 Instrumented from day one

- Aggregate perf budgets (page-load, first-bird-render, frame rate, audio errors).
- Simulation-tick latency p99.
- Memory-growth CI test.
- Drift calibration sanity test (synthetic week, synthetic 3 weeks).
- Snapshot serializer test asserting no numeric personality traits leak.
- Presence-account CI test asserting the conjunctive rule is enforced.

---

## 14. Risks

### 14.1 Drift calibration

**Risk**: drift is too fast (Tamagotchi-like, users move numbers by clicking) or too slow (screensaver, nothing the user does seems to matter).

**Mitigation**: the `α_t` defaults in §6.2 are calibrated to the PRD's "measurable in instruments after 1 week, visible after 3 weeks" target. The synthetic staging harness runs simulated weeks of presence against synthetic accounts and asserts the targets. The monotonic-toward-expressive rule is enforced in the tick code itself (`clamp_low` at 0), with a unit test that asserts no input ever produces a negative delta.

### 14.2 Sync correctness

**Risk**: a client somehow writes personality state directly, or two simulation workers run on the same account and produce divergent state.

**Mitigation**: there is no code path where a client sends absolute personality values. The API only accepts events. The simulation service acquires a per-account lease (Redis or Postgres advisory lock) before ticking; two workers cannot tick the same account simultaneously. A unit test asserts that no write path in the simulation service creates a new `bird_id` for an existing bird.

### 14.3 Audio uncanniness

**Risk**: procedural calls feel canned; chorus produces artifacts; listen-in mix feels like channel-switching.

**Mitigation**: the call motif library is designed for variation within recognizability — each bird's signature stays recognizable across mood and drift. The chorus mix is automatic (no per-call coordination needed) because calls are procedurally varied. The listen-in mix uses slow gain ramps and never silences other birds. A QA pass during beta specifically listens for "this call sounds identical to the last one" and "the chorus feels phasey."

### 14.4 Accessibility regressions

**Risk**: the narration degrades into state-list ("Pip is at perch 2"); reduced-motion is a stripped fallback; captions are stock strings.

**Mitigation**: the narration generator is shared with the notebook generator and uses the same naturalist voice. The snapshot serializer test asserts no numeric traits leak. Reduced-motion is a designed surface (§8.5). Captions are generated from the procedural call grammar at runtime, not stored as fixed strings.

### 14.5 Gamification creep

**Risk**: a "harmless" engagement feature slips in (a streak counter, a calendar of green dots, a "you've been here every day this week" notebook entry).

**Mitigation**: the non-goals are explicit and absolute. The notebook generator's templates are reviewed against the rule that observations are of the aviary, not of the user's behavior. A linting rule in the notebook template library rejects templates that reference visit frequency.

### 14.6 PII leakage

**Risk**: email is used as an identifier somewhere in the stack (log line, partition key, error message, telemetry dimension).

**Mitigation**: the synthetic UUID rule is enforced at the schema level (every table uses `account_id` UUID as the join key). Email is stored once, encrypted. A grep-based CI check scans the codebase for any reference to the email column outside the account-creation and email-change paths.

### 14.7 Presence honesty

**Risk**: the client's presence enforcement is lax (e.g., counts "tab is open" as presence), inflating drift across the user base.

**Mitigation**: the client's presence check is the conjunctive rule (visibility=visible AND focus AND recent pointer/keypress), enforced in a single audited code path. A CI test simulates various failure modes (background tab, minimized window, no input for 5 minutes) and asserts no `presence_ping` is emitted.

### 14.8 Single-writer availability

**Risk**: the simulation service goes down and the aviary stops ticking.

**Mitigation**: the simulation service is horizontally scalable; the per-account lease ensures correctness, not availability. If a worker dies mid-tick, the lease expires (default 30s) and another worker picks up the account. The event log is durable (Postgres); no events are lost. Clients continue to render from their last snapshot and interpolate; the aviary appears to continue (idle motion, ambient ornaments) even if the tick is briefly delayed.

---

## 15. Implementation Order (suggested)

This is a suggested sequencing for the engineering team, not a strict Gantt. The ordering prioritizes getting the central conceit (server-side tick + snapshot + renderer with motion already in progress) working end-to-end before any of the peripheral surfaces.

1. **Foundation**: account model, magic-link auth, synthetic UUID everywhere, event log, snapshot serializer (with the no-numeric-traits test).
2. **Simulation**: tick loop, drift function (with synthetic calibration harness), mood FSM, presence accounting (with the conjunctive-rule CI test).
3. **Renderer**: canvas scene, snapshot interpolation, idle micro-motion, day/night palette, loading state (quiet field, no spinner), empty-aviary state.
4. **Audio**: WebAudio motif library, call scheduling, chorus mix, listen-in mix, WebAudio-fallback-to-silence.
5. **Interactions**: return-greeting, listen-in, offer (with cooldown), settle (with undo), field notebook generation.
6. **Accessibility**: screen-reader narration, captions, reduced-motion renderer, keyboard navigation, contrast pass.
7. **Social**: visit invitations, read-only visitor snapshot/stream, revocation, visit log, visit-notification opt-in toggle.
8. **Account surfaces**: export, soft-delete/recover, device session list, settings.
9. **Rollout prep**: synthetic perf fleet, RUM aggregate pipeline, drift calibration validation, beta cohort onboarding.

Each milestone has a CI gate: bundle size, snapshot serializer test, presence CI test, memory-growth test, drift calibration test, no-numeric-traits test.

---

## 16. Ambiguities and defensible calls

- **Framework choice**: not specified by the PRD. The plan recommends a stack that supports streaming HTML + aggressive code-splitting + canvas rendering. The team should pick the specific framework based on familiarity and bundle-size pressure, but the architecture (renderer reads snapshots, writes events, never owns state) is framework-agnostic.
- **Mood state set**: `bird_engine.md` lists wary, content, curious, drowsy, alert as examples and says the exact set is finalized in implementation. This plan uses those five as the v1 set and flags it as `CALIBRATION`. Adding more states is a matter of extending the FSM matrix.
- **Drift α values**: proposed defaults in §6.2, flagged as `CALIBRATION-3`. The synthetic staging harness validates against the 1-week / 3-week targets.
- **Tick cadence**: 60s default, flagged `CALIBRATION-1`.
- **Presence activity window**: 180s default, flagged `CALIBRATION-2`.
- **Notebook entry frequency**: ~2–4 days for a regularly-visited aviary, flagged `CALIBRATION-6`.
- **Offer cooldown**: 180s, flagged `CALIBRATION-4`.
- **Bird-count ramp intervals**: ~60 days for the third bird, ~90 days between subsequent offers, flagged as `CALIBRATION` in §13.2.
- **Visit-notification default**: off, per the PRD. The opt-in toggle exists but is not surfaced during onboarding.

---

End of plan. This document is the plan only; it does not implement the product.
