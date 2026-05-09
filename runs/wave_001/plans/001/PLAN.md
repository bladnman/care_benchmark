# Pocket Aviary — v1 Implementation Plan

This plan translates the PRD into an executable v1 build. It is written for a frontier engineering team that has read the PRD; it does not restate the product, it interprets it. Where the PRD leaves a calibration call open, this plan makes a defensible choice and notes it. Defaults stated below should be treated as the starting calibration unless overridden by measured user behavior in canary.

The deliverable is the plan only. No product code is written here.

---

## 1. Scope

### 1.1 In scope for v1

- Single-user accounts, magic-link sign-in, per-device session management, account export, soft-then-hard deletion.
- One canonical aviary per account; two starter birds at adoption; cap of seven; new species offers paced by aviary age.
- Server-side simulation tick that owns canonical state (personality vectors, mood, position, call timing).
- Client rendering of a single horizontal scene with three perch zones, day/night cycle anchored to user-local time, occasional ambient weather, ambient micro-motion.
- Procedural call synthesis client-side via WebAudio with chorus mixing and listen-in re-balance.
- Interactions: idle presence, return-greeting, listen-in (engage and disengage with slow ramps), offers (seed, song fragment, still pool), settle (with five-second undo), field notebook (read-only).
- Multi-device sync as a property of the architecture (server-only personality writer; clients only emit interaction events).
- Visit feature: per-invite opt-in, read-only, revocable, no co-presence, no notification by default, visit log, 30-day invite expiration.
- Accessibility as designed surface: naturalist screen-reader narration, reduced-motion mode (cross-fade rendering), call captions, full keyboard navigation, WCAG AA contrast on chrome/copy.
- Performance: 2MB initial JS budget gzipped, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop, no memory growth over 30 minutes (CI-enforced).
- Aggregate-only operational telemetry; per-bird interaction state never aggregated.
- Last-two-major-versions browser support across Chrome, Safari, Firefox, Edge, with a matter-of-fact unsupported-browser surface elsewhere.

### 1.2 Explicitly out of scope (v1 and beyond, per PRD)

- Native apps, payments, shared aviaries, customizable scenes, multi-aviary accounts, push notifications, public discovery, leaderboards, achievements, streaks, badges, XP, calendars of visits.
- Tamagotchi mechanics: hunger, distress, decay, death, negative drift on neglect.
- Social-network surfaces beyond the single visit affordance: profiles, follows, comments, friend-of-friend chains, "explore other aviaries", co-presence, in-aviary chat, visitor avatars.
- Recorded-audio fallback path; user-controllable bird placement; user-edited or user-deleted notebook entries; any UI surface that exposes personality vector numerics.
- Any "Welcome back!" toast, banner, modal, or text greeting on return; any "you've been gone X days" or "friend visited!" notification surface.

### 1.3 Hard rules carried into every workstream

These are the rules a planner will be tempted to soften under pressure. They are restated here so each workstream can quote them back when defending a decision:

- **Notice, never announce.** No system-driven welcome, achievement, or notification surface anywhere in the product.
- **Asymmetric drift.** Personality vector traits move up on positive presence; they never move down on neglect.
- **Server is the only writer of personality state.** Clients emit interaction events to an append-only log; they never PATCH personality vectors.
- **Calls are procedural, non-negotiable.** No recorded-audio fallback at any quality.
- **Synthetic UUID for account identity everywhere except the single encrypted email-of-record.** Email is never a partition key, log field, or join key.
- **Personality vector numerics are never exposed to the user.** No debug toggle, no stats panel, no power-user view.
- **Presence is the conjunction of visibility + focus + recent pointer/key activity.** No relaxation of any of the three predicates.
- **Naturalist voice for the product surface; matter-of-fact for system surfaces (auth, errors, settings, sync conflict).**
- **Reduced-motion mode is a designed surface, not a stripped fallback.** Birds still drift, calls still play, mood still changes; only the visual register changes.

---

## 2. Architecture

### 2.1 Service shape

A small set of services with a clear authority gradient. The simulation service is the spine; everything else either feeds it events or reads its snapshots.

| Service | Responsibility | Stateful? |
|---|---|---|
| `edge` | Edge-cached static HTML, JS bundle, and a per-request bootstrap snapshot inlined into the HTML response. | No |
| `auth` | Magic-link issuance, link consumption, session token issuance and revocation, email-change verification. | Yes (small) |
| `accounts` | Account record (synthetic UUID, encrypted email, settings, soft-deletion timer). Account-scoped settings: accessibility prefs, visit-notification toggle, audio prefs. | Yes |
| `simulation` | The canonical aviary state. Owns personality vectors, mood, position, perch, call timing, drift, mood transitions, ambient events. Single writer. | Yes |
| `events` | Append-only interaction event log per account. Receives client-emitted events; consumed by simulation tick. | Yes (append-only) |
| `notebook` | Generates field-notebook entries from canonical state changes; persists notebook entries. | Yes |
| `narration` | Generates screen-reader prose snapshots from canonical state at slow cadence. May be co-located with simulation; logically separate. | Stateless |
| `visits` | Issues invite tokens, tracks invite state (outstanding, used, revoked, expired), authorizes visitor reads, records visit log entries. | Yes |
| `mailer` | Sends magic links, account-export download links, invite emails. Rate-limits per-email. | No (queue only) |
| `telemetry` | Aggregate operational metrics ingest. Hard wall against per-bird, per-account state. | Yes (aggregates) |
| `cron` | Drives the simulation tick scheduler, account-export jobs, soft-deletion sweeper, invite expirer. | No |

The shape is intentionally small. We are not building microservices for taste; we are drawing a privacy and authority boundary between the simulation database (per-account) and the telemetry pipeline (aggregate). That boundary is the single most important service-level invariant in the system and the rest of the topology is in service of it.

### 2.2 Authority gradient (which service can write what)

- Personality vectors → only the simulation service, only inside a tick.
- Mood, position, perch → only the simulation service.
- Notebook entries → only the notebook service, on simulation-event subscriptions.
- Interaction events → written by clients (authenticated session) or by the auth/accounts services for system events; consumed by simulation. Append-only; no in-place edit.
- Account settings → accounts service, authenticated by session token.
- Visit invites and visit-log entries → visits service.
- Telemetry → telemetry service. Cannot read from simulation/events/accounts databases. Hard network/IAM boundary.

### 2.3 Client/server split

- The client renders. It does not simulate. It computes interpolation between snapshots and runs the procedural audio engine, but it never decides that a bird's mood has changed or that a personality trait should move.
- The client emits interaction events to the events service. It never PATCHes personality state.
- The server holds canonical state, ticks the simulation, and serves snapshots and event ingestion.

Client-side state is purely derived and ephemeral: the most recent snapshot, the interpolation buffer, the audio scheduler queue, the UI focus state, the listen-in target. Loss of client state never loses simulation state.

### 2.4 Render pipeline boundary

The render pipeline is a separate logical module from the interaction layer. The render pipeline reads from a `SceneModel` (positions, moods, lighting, weather, current call schedule); the interaction layer writes user actions to the events service and updates the listen-in focus locally. The two communicate via an in-process event bus that the rendering loop subscribes to.

### 2.5 Hosting topology and data residency

- Edge HTML + JS bundle delivered from a global CDN with the bootstrap snapshot inlined into the HTML response (see §6.1 for why this matters for time-to-first-bird).
- Simulation service is regional, not edge — the per-tick read/write is cheap but consistent, and we are not chasing per-region tick replication. Account is pinned to its region of creation.
- Accounts/auth/events/visits/notebook are co-regional with simulation for the same account.
- Mailer is global; cron is regional.

---

## 3. Data Model

The data model lives in a relational store (PostgreSQL is the v1 choice; the simulation service uses it as both its system-of-record and its event log via a partitioned `events` table). The shape below is logical; storage details (indexes, partitions) are noted where they matter.

### 3.1 Identifiers

- `account_id`: opaque synthetic UUIDv7. The only identifier used in inter-service messages, partition keys, log lines, telemetry records.
- `bird_id`: opaque synthetic UUIDv7. Stable for the bird's lifetime.
- `event_id`: monotonic per-account sequence (a `bigint` per-account, to make event-log ordering trivially clear). Composite primary key is `(account_id, event_id)`.
- `invite_id`, `visit_id`, `session_id`: all UUIDv7.
- `email` is **only** stored encrypted on the `accounts` table. It is hashed (peppered HMAC) into `email_lookup_hash` for sign-in lookups; the lookup hash is the only column outside the encrypted email column that touches email content.

### 3.2 `accounts`

| Column | Type | Notes |
|---|---|---|
| `account_id` | uuidv7 | PK |
| `email_encrypted` | bytea | KMS-encrypted; per-account DEK |
| `email_lookup_hash` | bytea | HMAC for sign-in lookup; unique index |
| `created_at` | timestamptz | |
| `soft_deleted_at` | timestamptz nullable | non-null = pending hard delete after 30d |
| `region` | text | pinned region |
| `settings` | jsonb | accessibility prefs, audio prefs, visit-notification toggle, captions toggle, reduced-motion override |

Settings fields are documented inline in the schema; defaults are reduced-motion = off (honor `prefers-reduced-motion` query in client), captions = off, visit-notifications = off, audio = on.

### 3.3 `birds`

| Column | Type | Notes |
|---|---|---|
| `bird_id` | uuidv7 | PK |
| `account_id` | uuidv7 | FK |
| `species_id` | text | from species pool (see §3.10) |
| `name` | text | user-assigned, renameable |
| `created_at` | timestamptz | |
| `personality` | jsonb | `{boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}` — five floats in `[0.0, 1.0]` |
| `mood_state` | text | enum: `wary, content, curious, drowsy, alert, settled` |
| `mood_entered_at` | timestamptz | when current mood began |
| `current_perch` | text | enum: `front, middle, back` |
| `position_offset` | jsonb | small per-perch offset for visual variety |
| `last_call_at` | timestamptz | server-side scheduling reference |

Plumage saturation drives the bird's render-time color saturation multiplier; it never decreases (enforced by trigger on UPDATE — see §5.4).

### 3.4 `events` (append-only interaction log)

| Column | Type | Notes |
|---|---|---|
| `account_id` | uuidv7 | partition key |
| `event_id` | bigint | monotonic per-account |
| `client_session_id` | uuidv7 | for replay/dedup |
| `client_event_uuid` | uuidv7 | client-generated for idempotency |
| `event_type` | text | enum (see §3.4.1) |
| `payload` | jsonb | type-shaped |
| `received_at` | timestamptz | server receive time (authoritative) |
| `client_ts` | timestamptz | client-asserted (for sanity only; not used for ordering) |

PK is `(account_id, event_id)`. Unique index on `(account_id, client_event_uuid)` enforces idempotency; client retries are safe.

#### 3.4.1 Event types (v1)

- `presence_ping` — periodic heartbeat with `{visibility: bool, focus: bool, recent_input_ms_ago: int}`. The server records the ping; the simulation tick computes derived presence-time using the conjunction predicate.
- `listen_in_start` / `listen_in_end` — `{bird_id}`.
- `offer_made` — `{bird_id_target: nullable, offer_kind: 'seed'|'song_fragment'|'still_pool', song_motif_id: nullable}`.
- `settle_triggered` / `settle_undone` — no payload.
- `tab_visible` / `tab_hidden` — visibility transitions, mostly for client-render bookkeeping but logged for completeness.
- `bird_renamed` — `{bird_id, new_name}`.
- `bird_adopted` — system event from the simulation service when a new species offer is accepted; payload: `{bird_id, species_id}`.

The events table is the only write surface clients have for simulation-relevant state. Clients cannot insert into `birds`, `accounts`, `notebook_entries`, `personality`, or anything else. This is enforced both at the API gateway and at the database GRANT level.

### 3.5 `notebook_entries`

| Column | Type | Notes |
|---|---|---|
| `entry_id` | uuidv7 | PK |
| `account_id` | uuidv7 | partition key |
| `created_at` | timestamptz | |
| `prose` | text | naturalist-voice, lowercase, present-tense |
| `source_event_ids` | bigint[] | which events motivated this entry (for debugging only; not exposed) |

Notebook entries are read-only to the user. There is no DELETE or UPDATE path exposed to the client API.

### 3.6 `sessions`

| Column | Type | Notes |
|---|---|---|
| `session_id` | uuidv7 | PK |
| `account_id` | uuidv7 | FK |
| `device_label` | text | best-effort browser/OS string for the user-visible session list |
| `created_at` | timestamptz | |
| `revoked_at` | timestamptz nullable | |
| `last_seen_at` | timestamptz | refreshed on snapshot pull |

### 3.7 `magic_links`

| Column | Type | Notes |
|---|---|---|
| `link_id` | uuidv7 | PK |
| `account_id` | uuidv7 | FK |
| `token_hash` | bytea | hash; raw token never stored |
| `expires_at` | timestamptz | issuance + 15 minutes |
| `consumed_at` | timestamptz nullable | invalidated on consumption |
| `requested_from_ip_hash` | bytea | for rate-limiting; not exposed |

Rate limit: max 5 link issuances per email per hour, 20 per day. Soft, not user-facing as an error count; on overage the user gets the same "check your email" surface but no email is sent (matter-of-fact, not naturalist).

### 3.8 `invites`

| Column | Type | Notes |
|---|---|---|
| `invite_id` | uuidv7 | PK |
| `host_account_id` | uuidv7 | FK |
| `visitor_email_lookup_hash` | bytea | HMAC of visitor email |
| `visitor_email_encrypted` | bytea | display only, in host's visit log |
| `token_hash` | bytea | invite token; raw token never stored |
| `created_at` | timestamptz | |
| `expires_at` | timestamptz | created + 30 days |
| `revoked_at` | timestamptz nullable | |
| `first_used_at` | timestamptz nullable | |
| `last_used_at` | timestamptz nullable | |

### 3.9 `visit_log`

| Column | Type | Notes |
|---|---|---|
| `visit_id` | uuidv7 | PK |
| `host_account_id` | uuidv7 | FK |
| `invite_id` | uuidv7 | FK |
| `visitor_email_encrypted` | bytea | display only |
| `started_at` | timestamptz | |
| `ended_at` | timestamptz nullable | |
| `approximate_duration_seconds` | int nullable | rounded to nearest minute for the host's UI |

### 3.10 Species pool (configuration, not user data)

The species pool is a small set of fixed configurations shipped with the simulation service:

```
species: {
  species_id,
  silhouette_svg_id,        # client-side asset id
  default_palette,          # base hue/saturation/value
  call_motif_library_id,    # ref to procedural-grammar library
  size_class,               # affects perch offset and ambient motion
  nightjar_class: bool,     # whether this species calls into the night
}
```

Six species at v1, seeded at deploy. New species cannot be added by reaching into the database; they ship as a configuration update and require a code review.

### 3.11 Simulation snapshot (ephemeral, not stored long-term)

The simulation service exposes a snapshot endpoint that returns a derived view of `birds` joined with the current ambient state (lighting, weather, time-of-day in the account's resolved local timezone). Snapshots are not persisted as table rows; they are derived per request. A small in-memory snapshot cache per-account avoids recomputing within a tick.

```
SnapshotV1 {
  schema_version: 1,
  account_id,
  served_at,
  tick_id,                          # last applied tick
  local_time_iso,
  light_state: 'morning'|'midday'|'evening'|'night'|'pre-dawn',
  weather: { kind: 'none'|'rain'|'wind', intensity: 0..1, started_at, ends_at },
  birds: [
    {
      bird_id, species_id, name,
      mood_state, mood_entered_at,
      current_perch, position_offset,
      plumage_saturation_render,    # derived from personality but not the raw value
      call_schedule: [{ at_ms_from_served_at, motif_seed, mood_at_call }],
      idle_motion_profile: 'preen' | 'scan' | 'tilt' | 'hunch' | 'doze',
    }
  ],
  next_snapshot_after_ms,           # backoff hint
}
```

`plumage_saturation_render` is the only personality-derived value sent to the client, and it is sent as a render hint (a saturation multiplier in `[0.5, 1.5]` or similar derived range), not as the raw vector value. The raw vector is never serialized to a client.

### 3.12 Account export schema

The export endpoint emits a JSON document mirroring the user-relevant fields above: birds (id, name, species, current personality vector — yes, this is the one place the user can see numerics, and it is opt-in by way of clicking "export"; the export is the user's own data, not a UI surface; see §10 "calibration calls" for the rationale), mood, notebook entries, account settings. Export is generated to a signed expiring URL emailed to the verified address; it never sits in a public bucket.

### 3.13 Personality value range and seeding

Personality vector values normalize in `[0.0, 1.0]`. Seed values for newly-adopted starter birds are drawn from per-trait distributions designed to avoid identical starts across accounts:

- Boldness, curiosity, social warmth, vocal frequency: independent samples from `Beta(2, 5)` clipped to `[0.15, 0.7]` — biases toward modest starting values, leaves room for upward drift over weeks.
- Plumage saturation: starts at `0.4` for all birds (this is the value that drives the visible saturation multiplier; we want the user to see saturation grow, so a uniform low start makes the drift legible).

The two starter birds for any account are sampled with a constraint that their species differ and their boldness values are at least `0.1` apart (ensures one bird greets first more often than the other from day one — gives the user something to read).

---

## 4. API Surface

The API is small. There are five categories: auth, snapshot pull, event push, account, and visit.

All API responses for system surfaces (auth, errors, settings, sync conflict, visit revocation) are matter-of-fact; the client renders them in plain English. Snapshot/event surfaces are machine-shaped and don't carry user-facing prose; the prose lives in the notebook and narration endpoints.

API uses HTTPS only. JSON request/response. Versioned via `Accept: application/vnd.aviary.v1+json`. Auth via `Authorization: Bearer <session_token>` after sign-in.

### 4.1 Auth

- `POST /v1/auth/request-link` — body `{email}`. Always returns 204 (no enumeration). Triggers magic-link email if rate-limit allows.
- `GET /v1/auth/consume?token=...` — server-side; redirects to the aviary on success, to a matter-of-fact error page on failure (expired, replayed, malformed). On success, sets HTTP-only `aviary_session` cookie scoped to the apex domain. A used token is marked consumed atomically.
- `POST /v1/auth/sign-out` — revokes the current session.
- `GET /v1/account/sessions` — lists active sessions.
- `POST /v1/account/sessions/:id/revoke` — revokes a specified session.
- `POST /v1/account/email/change` — initiates email-change verification (sends magic link to the new email).
- `POST /v1/account/email/confirm?token=...` — completes email-change.

### 4.2 Snapshot pull

- `GET /v1/aviary/snapshot` — returns a `SnapshotV1`. The response also carries an `ETag` (per-tick, per-account); subsequent calls with `If-None-Match` get a `304` if the tick hasn't advanced, with a `next_snapshot_after_ms` header to set the next pull cadence.
- The snapshot is also embedded in the bootstrap HTML response on initial navigation when the user is authenticated, to remove a network round-trip from the time-to-first-bird budget.

#### Pull cadence

- On `visibilitychange` to visible.
- On long render-frame gap (>2s of skipped frames).
- On a low-frequency keepalive: every `next_snapshot_after_ms` (server-suggested; default 60s when nothing notable is pending, 15s when an offer reaction or weather event is in progress).
- Manual: never; users do not refresh the aviary themselves.

### 4.3 Event push

- `POST /v1/events` — body `{events: [{client_event_uuid, event_type, payload, client_ts}, ...]}`. Server returns `{accepted_event_ids, last_known_event_id}`. Client batches presence pings up to once every 10s; latency-sensitive events (`offer_made`, `listen_in_start`, `settle_triggered`) push immediately.
- Idempotency: the `(account_id, client_event_uuid)` unique constraint means duplicate sends are absorbed.

### 4.4 Account

- `GET /v1/account` — returns the user-visible account state (settings, list of birds with names and species, recent visit log summary).
- `PATCH /v1/account/settings` — updates settings.
- `POST /v1/account/birds/:id/rename` — renames a bird.
- `POST /v1/account/export` — schedules an export job; emails a signed download URL when ready.
- `POST /v1/account/delete` — initiates 30-day soft delete.
- `POST /v1/account/restore` — undoes a pending soft delete (only valid in the 30-day window).

### 4.5 Field notebook

- `GET /v1/notebook?cursor=...&limit=...` — paginated list of entries, most-recent first; `cursor` is opaque and forward/backward through history. The notebook returns entries indefinitely; old entries do not get archived.

### 4.6 Visit (host side)

- `POST /v1/visits/invites` — body `{visitor_email}`. Returns `{invite_id, expires_at}`. Mailer sends an invite email with a one-time link.
- `GET /v1/visits/invites` — lists outstanding invites.
- `POST /v1/visits/invites/:id/revoke` — revokes; immediate.
- `GET /v1/visits/log?cursor=...` — paginated visit log.

### 4.7 Visit (visitor side)

- `GET /v1/visits/consume?token=...` — server validates the invite token, issues a short-lived visitor session bound to the host account_id and the invite_id. Visitor sessions are NOT account_session tokens; they grant only read access to the host snapshot and disallow event push.
- `GET /v1/visits/snapshot` — visitor's read of the host's snapshot. Returns the same `SnapshotV1` shape, never includes any visitor identity in the payload.
- `POST /v1/events` returns `403` for visitor sessions; the visitor cannot emit events.
- On the next snapshot poll after revocation/expiry, the visitor receives `410 Gone` with a matter-of-fact body: `{message: "This visit is no longer available."}`.

### 4.8 Narration

- `GET /v1/narration` — returns the current naturalist-voice prose snapshot for the aviary state. Server-side; the same content is also emitted in `aria-live` regions client-side. Cadence is roughly 30–60 seconds idle, faster on user-initiated events.

### 4.9 Telemetry ingest (internal)

- `POST /internal/telemetry` — accepts only operational metrics (latency histograms, error counts, bundle-size pings, render-frame timing). Schema explicitly rejects per-bird and per-account fields. Hard-walled at IAM and at network policy from the simulation database (see §11.5).

### 4.10 Errors

All system errors use a single matter-of-fact envelope:

```
{ "error": { "code": "magic_link_expired", "message": "The link may have expired. Try requesting a new link." } }
```

Error code catalog is enumerated in the API spec; no surprise codes leak into the response.

---

## 5. Simulation Engine

This is the spine of the product. The simulation service is the only writer of personality state and is responsible for the entire "feels alive" property.

### 5.1 Tick cadence

A tick runs per-account, scheduled by the cron service:

- **Default cadence: 60 seconds.** Each account ticks once per minute.
- **Faster cadence on active session: 20 seconds** while a recent presence ping suggests the user is actively present (last presence ping received within the last 30 seconds with all three predicates true). This makes mood transitions feel responsive on-screen.
- **Slow cadence during inactivity: 5 minutes** if no presence ping in the last 24 hours (the aviary is still ticking, time-of-day is still advancing, but we don't burn cycles tightly).

We do not run a single global tick clock; each account is its own scheduled job, dispatched by a sharded worker pool. This keeps tick latency bounded per-account even at scale, and lets us alarm on per-account tick lag rather than aggregate.

### 5.2 What a tick does

In a single tick for account A:

1. Read all `events` since the last applied event for A.
2. Read all `birds` for A.
3. Resolve account local time and time-of-day light state.
4. Roll for ambient weather transitions (see §5.7).
5. For each bird:
   - Compute drift contributions from the new events (see §5.3).
   - Apply drift deltas to the personality vector, gated by the asymmetry rule (§5.4).
   - Compute the new mood (see §5.5).
   - Compute the new perch (see §5.6).
   - Schedule the next call window (see §6.4).
6. Run bird-to-bird interaction pass (chorus, alarm spread, mood contagion — see §5.8).
7. Possibly write a notebook event seed (see §5.9). Notebook entries are sparse; the seed is a candidate, not a guaranteed entry.
8. Persist the new bird state.
9. Increment `tick_id`. Mark the highest applied event_id.
10. Update the snapshot cache for A.

The tick is transactional per-account: either all of step 5–9 commit, or none of them. If the tick fails, the events are not "consumed" and the next tick reprocesses them; idempotency falls out of the deterministic event-ordered application.

### 5.3 Drift function

Drift is a low-pass filter over presence-and-interaction signals. Per-trait deltas are computed per tick from the events in that tick window, then summed and clipped:

- **Presence-time** (the dominant input): `delta_t_minutes = sum of presence-time accrued since last tick`. This contributes a small uniform upward push to all five traits, weighted toward `boldness` and `vocal_frequency` for the bird that was visible-on-screen during that presence (which in v1 is all birds, since the aviary fits at a glance).
  - Magnitude: `+0.0008` per presence-minute on weighted traits, `+0.0003` per presence-minute on the rest. Calibration target: a bird with 30 minutes of presence per week reaches a measurable instrument delta in ~7 days and a visible delta (on plumage saturation in particular) in ~21 days.
- **Listen-in**: per second of listen-in on bird B, `+0.0015` to B's `social_warmth`, `+0.001` to B's `vocal_frequency`, `+0.0005` to B's `plumage_saturation`. Listen-in is the strongest per-second drift signal; it ramps fast so the user feels recognized for paying attention to a specific bird.
- **Offer made**: `+0.005` to the receiving bird's `curiosity` if the offer is accepted (the simulation decides "accepted" based on mood + curiosity at the moment of offer); `+0.002` to all nearby birds' `boldness` regardless of acceptance (just being offered to is a small confidence push).
- **Settle**: no drift contribution. Settle is a mood-quieting signal (see §5.5) and a presence-window terminator; it does not push drift.

These coefficients are the v1 starting calibration. The simulation service exposes them as configuration (not user-visible); we plan a calibration pass during canary (see §10.3).

### 5.4 Asymmetry rule (load-bearing)

Personality vector deltas are clipped at `0` from below before being applied. Any computed delta `d` for trait `t` becomes `max(0, d)` before being added. This is enforced in two places:

- **In code**, in the simulation tick, by an explicit `max(0, _)` wrapper at the only call site that mutates the vector.
- **In the database**, by a CHECK constraint and a row-level UPDATE trigger that rejects any update where the new value of any trait is less than the prior value. This makes a "bug introduces negative drift" failure mode unreachable rather than catchable-in-review.

The trigger is intentionally double-coverage. The principle "personality only moves up" is the most affectively important invariant in the entire system; if it fails, the product becomes a Tamagotchi with extra steps and the user's trust is gone in a few sessions. Code-only enforcement is too easy to defeat with a future "small refactor"; the database constraint is the durable defense.

The trigger has one carefully-scoped exception: account hard-deletion may delete the bird record entirely. There is no UPDATE path that lowers a trait, ever, for any reason — including "fixing" a calibration mistake. A miscalibrated drift function results in birds that drifted faster than intended; the recovery is to halt the drift function, not to claw back the drift. Birds remain at their drifted values.

### 5.5 Mood transitions

Mood is a small enumerated state per bird: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`. (Six states v1; the PRD mentions wary/content/curious/drowsy/alert as examples — we add `settled` to capture the late-night/post-settle state.)

Per-tick, the new mood is the argmax of a scoring function:

```
score(new_mood | bird, events, ambient) =
    base_affinity(personality, new_mood)
  + recency_pull(current_mood, time_since_mood_entered, new_mood)
  + event_pull(recent_events, new_mood)
  + ambient_pull(time_of_day, weather, new_mood)
  + neighbor_pull(other_birds_moods, new_mood)
```

Each term is a small bounded contribution; the sum is softmaxed and the new mood is sampled (not argmax) to keep mood transitions feeling organic rather than deterministic. Sampling reduces "Pip is always content at noon"; the variation is an aliveness signal.

Specific pulls of note:

- `event_pull` for `offer_accepted` on this bird → +1.5 to `content`, +0.5 to `curious`.
- `ambient_pull` for evening light → +1.5 to `drowsy` for non-nightjar species; +1.5 to `alert` for nightjar species.
- `ambient_pull` for `rain` → +1.0 to `wary` for low-boldness birds; -0.5 to `content` for all.
- `neighbor_pull` for `wary` neighbor → small positive pull to `wary` (alarm contagion). `neighbor_pull` for `content` neighbor → small positive pull to `content`.
- `recency_pull` discourages mood-flicker: after entering a mood, there is a short refractory period (~90 seconds default) during which transitions are damped.

Settle gesture: when a `settle_triggered` event lands (and isn't followed by `settle_undone` within 5 seconds), the simulation service forces all birds toward `settled` over a slow ramp (about 5 seconds of mood-transition time across all birds, staggered slightly per bird so they settle in sequence rather than in unison). The lighting state also forces an evening cross-fade for the duration of the session.

### 5.6 Perch selection

Each bird's perch (front/middle/back) is recomputed on mood change. Mapping:

| Mood | Bias |
|---|---|
| `wary` | 70% back, 25% middle, 5% front |
| `content` | 25% back, 50% middle, 25% front |
| `curious` | 10% back, 30% middle, 60% front |
| `drowsy` | 30% back, 40% middle, 30% front (low movement on perch) |
| `alert` | 20% back, 30% middle, 50% front |
| `settled` | 50% back, 40% middle, 10% front (eyes closed, low) |

Within a perch, an `(x, y)` offset is sampled to avoid two birds rendering on top of each other. Offsets are persisted (so the bird "stays" between snapshots).

### 5.7 Ambient weather

Weather rolls per account on each tick:

- Default: `none`.
- Transition probabilities per tick at `none`: 1% to `rain` (lasts ~3–7 minutes), 2% to `wind` (lasts ~2–4 minutes). Limited by a per-account 30-minute cooldown after any weather event (so weather is "rare" — a few times a week, per the PRD).
- During `rain`: `vocal_frequency` calls scheduled less often (multiplier 0.6), wary-pull increased.
- During `wind`: ambient leaf-drift density increases client-side; some birds shift to `alert` per the pull rule above.

### 5.8 Bird-to-bird interaction

Run after individual mood updates. Three mechanics:

- **Alarm spread**: if any bird transitioned to `wary` this tick, neighbor `wary_pull` is applied retroactively (one re-pass; not an iterative cascade — we don't want feedback loops).
- **Chorus**: if two or more birds have a call window scheduled within a 1.5s overlap, the schedule is adjusted to align them, and the procedural call grammar uses the chorus-mode pitch alignment (see §6.4). Chorus probability is multiplicative with each bird's `vocal_frequency`.
- **Greeting cascade** (used at session-start, see §5.10): when one bird greets the user, neighbors get a small `social_warmth_pull` that may cause them to greet a beat later.

### 5.9 Notebook entry generation

Notebook entries are intentionally sparse. The simulation service emits a notebook *seed* — a candidate observation about a noteworthy event — and the notebook service decides whether to render it as an entry.

Seed sources include:

- A bird greeted first today for the first time this week.
- A bird's plumage saturation crossed an integer-tenth threshold (e.g., went from `0.4` to `0.5` over the past few days).
- A first chorus event between specific birds.
- A long quiet stretch (no calls for >10 minutes during a presence window).
- Weather events.
- A first-of-its-kind interaction (first time bird B accepted a song-fragment offer).

The notebook service rate-limits entry emission per-account: at most 1 entry per 24-hour period for routine seeds, with up to 2 additional entries per week reserved for "noteworthy" seeds (first-of-its-kind events). Generic per-session entries are not emitted; the rule is "observations that matter, not a feed."

Prose generation uses a small set of naturalist templates with bird-name and detail substitution. Templates are reviewed for voice (lowercase, present-tense, specific). Generated prose is committed to `notebook_entries.prose`. The system never writes about the user — no "you visited every day this week" — and the templates are audited against this rule. (See §11.7 for the voice-audit gate.)

### 5.10 Return-greeting

Return-greeting is a special tick triggered by a presence ping that was preceded by a long-enough absence:

- Short absence (<5 min): a single bird (chosen by `boldness` weighted) glances or makes a brief call. Mood pulls slightly toward `content` for that bird.
- Medium absence (5 min – 4 hours): one bird greets with a longer call; a second bird may respond after a ~600ms stagger.
- Long absence (>4 hours): one bird steps toward the front perch and calls; another bird responds; the lighting cross-fades to whatever the local time-of-day is (handles the case of returning at a different time of day than departure).

The return-greeting is procedural variation, not preset — the bird that greets, the call motif, the timing offset are all sampled from per-bird distributions that are seeded by a deterministic-but-chaotic function of `(bird_id, account_id, presence_ping_id)` so each return is fresh but reproducible for debugging.

If multiple birds would greet, they stagger by a randomized small offset (200–800ms). A simultaneous chorus on cue would announce arrival; the stagger preserves "noticed, one bird at a time."

### 5.11 Drift calibration: how we know it's right

The drift calibration target — measurable in instruments after ~1 week of regular visits, visible to the user after ~3 weeks — is testable. We build a synthetic-user harness:

- A scripted client emits the presence and interaction patterns of a "regular visitor" (e.g., 3 sessions a week at 10 minutes each).
- The simulation tick runs at a configurable accelerated wall-clock so we can compress 3 weeks of simulated time into hours of test wall-clock time.
- Instrument: a snapshot of every bird's personality vector at simulated week 1, week 2, week 3.
- Pass: vector deltas at week 1 are >0 on all traits with sustained presence; deltas at week 3 cross thresholds we judge to be "user-visible" (in particular, plumage saturation increased by ≥0.15, social_warmth increased by ≥0.10).

This harness is a CI-runnable test against the simulation service. Calibration coefficient changes go through this test.

---

## 6. Frontend Rendering Pipeline

The render pipeline is the affective surface. Two budgets dominate every choice: the 2MB initial bundle and the 60fps idle motion target on a 5-year-old laptop.

### 6.1 First paint and time-to-first-bird

The bootstrap HTML response (served from the edge for an authenticated user) includes:

- The HTML shell.
- A small inline `<script>` that decodes an inlined snapshot blob.
- A `<link rel="preload">` for the main JS bundle.
- A small inline SVG for the sky-and-perches background.

The inlined snapshot is the trick that buys the time-to-first-bird budget. By the time the bundle has fetched and parsed, the snapshot is already in memory; the first frame the renderer paints already has birds at their current positions, mid-action.

For the unauthenticated case (sign-in page), there is no aviary on first paint — the sign-in surface is matter-of-fact and minimal.

For the cold-cache or slow-network case, the bootstrap HTML still ships a tiny inline snapshot stub (the perches and the sky), and the first render is the "quiet field" loading state — soft sky color with one or two faint motion cues, no spinner. The full bird snapshot streams in within the next few hundred ms and the birds fade into position.

### 6.2 Renderer choice

Canvas2D for the aviary scene. Reasoning:

- WebGL is overkill for a single horizontal scene with ~7 birds and a few foreground/background layers, and the bundle cost of a serious WebGL setup eats too much of the 2MB budget.
- SVG is too DOM-heavy for the per-frame mutations idle motion needs (every breath, every preen frame, every leaf drift would touch the DOM).
- Canvas2D gives us precise control over per-frame composition with negligible runtime cost.

Bird visuals are composed at runtime from a small set of layered SVG "parts" (silhouette, wing, head, tail, eye, plumage overlay) rasterized on first use into Canvas. The per-frame composition is just `drawImage` calls of cached rasters at computed positions. Plumage saturation drives a hue/sat shader pass on the rasterized plumage overlay (cached by saturation value, recomputed when the saturation render hint crosses a small threshold).

### 6.3 Scene composition

Three render layers, composited per frame:

1. **Background layer**: sky gradient, distant foliage, color modulated by `light_state` and weather.
2. **Mid layer**: birds and perches. The dominant per-frame work.
3. **Foreground layer**: occasional passing leaves/feathers, foreground branch hints. Sparse.

A fourth, transparent overlay layer holds focus indicators, captions, and aria-live mirrors when needed.

### 6.4 Per-frame loop

`requestAnimationFrame`-driven. Per-frame work:

1. Tick the local interpolation clock; resolve current bird positions from the latest snapshot's interpolation targets.
2. Tick each bird's idle-motion state machine (preen frame, scan frame, head-tilt frame). Each state machine is a small finite list of poses with timing.
3. Tick the weather/lighting state (color-mod values).
4. Tick the audio scheduler (see §7); enqueue any calls due in the next audio buffer.
5. Composite layers via Canvas2D.
6. Submit aria-live updates if the narration cadence ticked over.

A frame budget of 6ms (out of the 16.6ms 60fps slot) is targeted on a 5-year-old laptop. Budget burnout (>16ms frames sustained for 2+ seconds) triggers a graceful degradation: idle micro-motion FPS drops to 30 by halving the motion-state-tick rate, while the snapshot interpolation continues at 60fps. The user sees birds preening more slowly rather than the scene stuttering.

### 6.5 Snapshot interpolation

When a new snapshot arrives:

- Diff against the current snapshot: which birds moved perch, which changed mood.
- For perch moves, compute a flight path (a smooth bezier from old position to new position with realistic timing — ~600ms for a bird's flight, slower for tired/drowsy birds).
- For mood changes, queue the idle-motion-state-machine to transition into the new mood's profile over a slow blend (~1s).
- For call schedule additions, push them onto the audio scheduler.

Birds in mid-flight when a new snapshot arrives complete their flight before adopting the new perch from the new snapshot, unless the new snapshot disagrees with the old destination — in which case the bird's path is rebezierd to the new target. We never teleport.

### 6.6 Idle motion state machine per mood

Per mood, a small state machine of poses with weighted-random transitions:

- `content` → preen, brief scan, settle-on-perch shuffle.
- `curious` → tilt-head, scan-up, scan-side, brief step-forward.
- `wary` → freeze, scan-around, retreat-step-back.
- `drowsy` → fluff, blink, low-perch-shuffle.
- `alert` → straighten, scan-fast, brief call-tilt.
- `settled` → eyes-closed, slow-breath, no head movement.

Each pose is a 6–12 frame inline animation; the per-frame poses are precomputed as Canvas2D draw operations. The state machine ticks every 200ms or so (so it's not every-frame work; the rendering is just compositing the current pose between ticks).

### 6.7 Reduced-motion mode

When `prefers-reduced-motion: reduce` is set or the user opts in:

- The idle-motion state machine is replaced with a slow cross-fade between pose snapshots. The same poses, but blended over ~3 seconds instead of stepped at frame rate.
- Flight transitions become 1.5-second cross-fades between the two perch positions, bird visible at both endpoints with reduced opacity in between, rather than animated paths.
- Ambient leaf/feather drift is omitted entirely.
- Day/night color cross-fades remain, slowed by a factor of two so they're imperceptibly slow rather than quietly slow.
- Calls play normally; mood and drift continue normally; the field notebook still updates.

The reduced-motion path is implemented as an alternate `Renderer` strategy selected at startup. It is not a runtime branch inside the regular renderer (which would be both slower and harder to test); it's a dedicated module that the boot wires in based on the preference. This lets us style and test reduced-motion as its own designed surface.

### 6.8 Top bar

Implemented as a separate React (or framework-equivalent) component tree above the Canvas. The top bar is HTML/CSS, not Canvas — it needs accessible focus management, ARIA, keyboard nav, and standard click targets.

- Top bar items: account/settings, accessibility settings, field notebook, offer affordance.
- Top bar fade: after 3 seconds of cursor-stillness AND no keyboard activity, opacity transitions over 800ms to 0.15. Any mouse move or key press returns to full opacity. The Canvas continues rendering normally; the fade is purely the overlay.
- Top bar components are code-split: opening account settings or the visit invitation flow lazily loads its module. The aviary base bundle does not pay for these surfaces.

### 6.9 Visibility, focus, and tab-hidden behavior

- On `visibilitychange` to `hidden`, the renderer pauses (no `requestAnimationFrame` calls). Audio stops scheduling new calls but lets in-flight calls finish. Presence pings stop firing.
- On `visibilitychange` to `visible`, snapshot pull is triggered; once the new snapshot arrives, the renderer resumes with smooth interpolation from the new state. There is no entry transition; we are explicit about not playing one.
- The bird positions on resume are the bird positions in the new snapshot, not the bird positions where they were when the tab hid; this honors "the aviary continues without the viewer."

### 6.10 Adoption and onboarding flow

- New account → magic-link sign-in → adoption screen (matter-of-fact: "let's name your two starter birds").
- The system has already chosen the two species (sampled at account creation, before the user reaches this screen).
- Two name fields with default suggestions; the user can change them now or later.
- On submit: the account's two birds are created server-side; the user lands in the aviary, which is empty for the brief moment before the first bird flies in.
- Empty-aviary state is the same quiet field used for the cold-cache loading state. First bird enters with a soft fly-in to its starting perch (back). Second bird enters about 4 seconds later (different perch).
- After this moment, the aviary is never empty again for this account.

There is no onboarding tutorial. There are no tooltips explaining listen-in or offer. The product is meant to be discovered by sitting with it; that's the design stance, and a planner who reaches for "first-time tooltips" is solving a problem the product specifically refuses to have.

---

## 7. Audio Pipeline

Audio is the affective spine. The non-negotiable rules are: procedural, never recorded; client-side via WebAudio; chorus is a real mix of procedural calls; silence-with-captions is the only fallback.

### 7.1 Procedural call grammar

Each species has a `motif library` — a small set of pitch contours, durations, ornaments, and rest patterns. A motif is a sequence of `(pitch_offset, duration_ms, amplitude_envelope, ornament_kind)` tuples relative to a per-species fundamental.

A call is generated by:

1. Sample a motif from the species library, weighted by current mood (a `content` bird favors longer melodic motifs; a `wary` bird favors short single-note alarms).
2. Apply per-bird pitch offset (baseline pitch is a stable per-bird value sampled at adoption, adding individuality).
3. Apply per-call random variation: small jitter on pitch and timing, ornament selection, amplitude envelope shape.
4. Modulate by `vocal_frequency` (timing density) and mood (envelope shape, ornament richness).
5. Synthesize via WebAudio nodes (see §7.3).

Each call is produced fresh every time. There is no single recording, no rotation of pre-baked variants.

### 7.2 Per-bird identifiability

Across mood and drift, each bird's call must remain recognizable. We achieve this by holding stable:

- Per-bird fundamental pitch offset (sampled at adoption from the species's pitch range).
- Per-bird ornament style preference (a sticky weighting of which ornaments this bird tends to use).
- Per-bird amplitude envelope tendency (sharp attack vs. soft, short release vs. long).

What varies with mood: motif selection bias, timing density, pitch jitter range. What varies with drift: timing density (vocal_frequency), motif richness (more elaborate motifs accessible to higher-vocal-frequency birds).

### 7.3 WebAudio graph

Per-bird audio chain:

```
Oscillator(s) → AmplitudeEnvelope → PerBirdEQ → SpatialPan → CallMixer
                                                                    ↓
                                                         AviaryReverb → Output
```

- Oscillators: a small number of harmonically-related sines summed to approximate the species's call timbre; some species use a noise source for breath texture.
- AmplitudeEnvelope: per-call ADSR derived from the call's amplitude envelope tuple.
- PerBirdEQ: a small biquad-based per-bird formant filter.
- SpatialPan: stereo position derived from the bird's perch position on screen.
- CallMixer: per-bird gain that participates in the listen-in re-balance (see §7.5).
- AviaryReverb: a small shared convolution reverb; light room sense, not concert hall.

All nodes are reused across calls; no per-call allocation. The 30-minute no-memory-growth rule is enforced by holding to a fixed audio-graph topology that is created once at audio-context startup.

### 7.4 Chorus mixing

When two or more birds have call windows scheduled in the same 1.5s window:

- Pitch alignment: their fundamentals are nudged to harmonically related ratios (octave, fifth, third) where personality permits — a high-vocal-frequency bird is more chorus-amenable, a wary bird less so.
- Timing alignment: their call onsets are quantized to a shared rhythmic pulse with small per-bird offsets (≤80ms) so the chorus doesn't sound metronomic.
- Mix balance: each chorus participant's gain is reduced by 1/√n to keep total chorus loudness comparable to a single call.

The audio engineer's calibration target: a chorus of two birds reads as "two birds together," not "two stacked tracks." If we can't hit that, the chorus rule needs more work, not the procedural-audio rule.

### 7.5 Listen-in mix

Listen-in re-balances per-bird gains:

- Focused bird's `CallMixer` gain ramps to 1.0 over ~1.2 seconds.
- All other birds' `CallMixer` gains ramp to ~0.35 (audible, attenuated; never silenced) over the same window.
- AviaryReverb dry/wet ramps slightly drier on the focused bird (cleaner direct sound).

On disengage, gains return to their default (1.0 for all) over ~1.2 seconds. The ramp curve is logarithmic, not linear, to feel like a re-balance rather than a fade.

Disengage triggers: clicking the focused bird again, focusing a different bird, clicking empty aviary space, moving keyboard focus out of the bird group, pressing Escape.

### 7.6 Audio scheduling

The audio scheduler runs in the renderer's per-frame loop. Each frame:

- Look ahead `audioContext.currentTime + 200ms` and enqueue any calls scheduled to start in that window.
- Per call: walk the procedural grammar, schedule the WebAudio nodes for that call, register a teardown timer.

Scheduling uses `audioContext.currentTime` (sample-accurate) rather than `performance.now()`. Calls scheduled for more than 5 seconds out are not enqueued (the simulation may revise the schedule before then on the next snapshot pull).

### 7.7 Captioning

Captions are generated from the call schedule. Each scheduled call carries a `caption_template` produced by the procedural grammar (e.g., for a three-note rising motif: `"a soft three-note rise"`).

Caption rendering:

- A caption is shown as small naturalist-voice text near the calling bird's screen position.
- Caption opacity ramps in over ~250ms at call onset, holds for the call duration plus 500ms, fades out over ~400ms.
- Captions are a separate render layer (HTML, not Canvas) so they are accessible to the screen reader's text content (in addition to the dedicated narration aria-live region).
- Captions do not stack: if a new call from the same bird overlaps the prior caption, the prior caption truncates and the new one replaces it.

Captions are also the audio fallback (see §7.9).

### 7.8 Audio settings

In accessibility settings:

- Audio on/off. (Default: on.)
- Captions on/off. (Default: off, but forced on if WebAudio is unavailable or audio is off.)
- Master volume. (Default: 0.7.)

Audio off is a designed surface: the visual aviary continues normally; captions appear by default. This is deliberate parity with the WebAudio-unavailable path.

### 7.9 WebAudio fallback

If `AudioContext` cannot be created or is permission-denied:

- The aviary plays in graceful silence.
- Captions are forcibly enabled.
- A small, matter-of-fact line in accessibility settings explains: "Audio isn't available in this browser or session. Captions are turned on so you can follow the calls."
- No fallback recorded audio is loaded. This is checked in the bundle audit (§11.6).

### 7.10 iOS Safari audio-context unlock

iOS browsers require an `AudioContext` to be created or resumed inside a user gesture. Strategy:

- On first navigation, audio is in a suspended state.
- The first user gesture (any click, tap, or key press anywhere) resumes the audio context.
- Until then, the visual aviary is fully active and captions show by default. The user does not see an "enable audio" prompt; the audio simply joins in once they interact.

This is intentionally noticing-not-announcing: the aviary doesn't ask permission, it just starts singing once the user engages.

---

## 8. Accessibility Surfaces

Accessibility is a designed surface, not a checklist. The plan below treats it that way.

### 8.1 Screen-reader narration

- A single `aria-live="polite"` region contains a running prose narration of the aviary, generated at slow cadence (30–60s idle, prompt on user-initiated events).
- Prose is naturalist-voiced, generated by a templated narrator that reads the same `SnapshotV1` the renderer reads.
- A second `aria-live="assertive"` region is used sparingly for user-initiated event acknowledgements (offer accepted, settle triggered, listen-in started) so they don't get lost in the polite queue. Even these are written as observations, not as state transitions ("pip leans toward the seed and tilts her head" rather than "offer accepted").
- The narrator suppresses redundant updates: if the prose hasn't materially changed (same birds in same moods on same perches in same lighting), no new line is emitted.
- Captions are also exposed in their own aria-live region (assertive), per call.

### 8.2 Focus model

- Focus trap: there isn't one. The aviary is a single page; tab traversal moves through the top bar items, into the aviary scene (which is one focusable region), and back out.
- Inside the aviary scene, arrow keys move focus between birds. Each bird is a focusable element with `aria-label="<name>, perched <position>, mood: <mood>"` (yes, even though we don't expose mood numerically, the screen-reader user benefits from the named affective state — this is the one place mood is explicitly named, and it's a usability call: a screen-reader user can't see the mood from the idle motion, so we tell them).
- Enter on a focused bird triggers listen-in. Escape exits listen-in.
- Tab moves focus to the next top-bar item.
- The currently-focused bird gets a soft, high-contrast visible focus outline drawn in the overlay layer.

### 8.3 Reduced-motion mode

Implemented as a separate renderer strategy (see §6.7). Settings carry a tri-state: `system` (honor `prefers-reduced-motion`), `on`, `off`.

### 8.4 Captions

Implemented per §7.7. Settings: tri-state — `system` (honor audio-context state), `on`, `off`.

### 8.5 Contrast

- All chrome text passes WCAG AA contrast at minimum (4.5:1 for normal text, 3:1 for large text).
- Captions and focus outlines pass the same against any aviary background state. The visual designer specifies the actual color choices; the engineering check is that both `morning` and `night` palettes pass contrast checks for caption text and focus outlines.
- A CI step renders the chrome at each light-state and runs a contrast test against the rendered aviary background underneath. A regression is a build break.

### 8.6 Auth, settings, error surfaces

These are matter-of-fact, accessible HTML forms with standard labels, error association via `aria-describedby`, keyboard-friendly buttons. They do not adopt the naturalist voice.

### 8.7 Settings panel layout

The accessibility settings panel exposes:

- Reduced motion: system / on / off.
- Captions: system / on / off.
- Audio: on / off, master volume slider.
- Visit notifications: on / off.
- (No personality vector, no debug, no "show me how my bird is doing.")

---

## 9. Performance Budgets and Observability

The PRD names the budgets; this section names how we hit and hold them.

### 9.1 Bundle budget — initial JS <2MB gzipped

- The initial bundle covers: core renderer, Canvas2D drawing, motion state machines, snapshot interpolation, audio scheduler, procedural call grammar, base UI frame, the aria-live narrator, and the input/event-pinging machinery.
- Code-split modules: account/settings panel, accessibility-settings panel, field notebook view, visit invitation flow, account export confirmation.
- Asset budget: bird silhouette SVGs combined <60KB; sky/foliage backgrounds <80KB; foreground ornaments <30KB.
- A CI gate measures the initial bundle gzipped on every PR. Budget overruns fail CI; an exemption requires a written justification merged into a "perf-debt.md" doc that the team reviews quarterly.

### 9.2 Time-to-first-bird <500ms

- Bootstrap HTML inlines snapshot for authenticated users (see §6.1).
- Bundle is delivered from CDN edge with HTTP/3 and Brotli; long-cache headers on the bundle hash.
- Critical render path is zero-network after first byte: HTML + inlined snapshot + inlined sky SVG → first bird drawable from the snapshot once the bundle arrives.
- A synthetic perf check (see §9.5) measures TTFB and time-to-first-bird from common geographies on a mid-tier mobile profile (Lighthouse "Slow 4G", CPU 4× throttling on a Pixel-class profile). If p95 over the trailing 24h exceeds 500ms, on-call is paged.

### 9.3 60fps idle motion on a 5-year-old laptop

- Per-frame budget: 6ms target on the slow-laptop profile.
- Budget assertion: a runtime perf monitor in the renderer samples frame durations; sustained budget overruns trigger graceful degradation (idle motion FPS halves to 30, see §6.4).
- A nightly automated test runs the aviary on a virtual machine sized to a 5-year-old mid-range laptop (Intel i5-8xxx-class, 8GB RAM, integrated graphics) for 30 minutes and records the frame distribution. p99 frame time >18ms is a regression.

### 9.4 No memory growth over 30 minutes (CI-enforced)

- A nightly browser-driven test runs the aviary for 30 minutes with synthetic activity (presence pings, occasional offers, listen-in cycles).
- Memory samples are taken every 5 minutes via the browser's `performance.memory` API.
- Pass: total heap growth <10MB end-to-end and no monotonic growth trend in the audio buffers (audio-buffer count steady).
- Fail: heap growth >10MB, or audio buffer count or any object pool grows unbounded.
- This is a CI test, not a runbook.

### 9.5 Synthetic perf monitoring

- A fleet of automated browsers (Chrome, Safari, Firefox; recent versions; mid-tier mobile profile and desktop profile) runs the aviary on a schedule from 4 geographies (US East, US West, EU, APAC).
- Metrics: navigation timing, time-to-first-bird, render frame distribution, audio-context startup time, snapshot pull latency.
- Aggregates ship to the telemetry service; per-bird state is never collected.

### 9.6 Aggregate Real User Monitoring

- Page-load timings, first-bird-render timing, render-frame timing histograms (binned), audio-context error counts, simulation-tick latencies (server-side).
- Schemas explicitly exclude `account_id`, `bird_id`, and any per-bird state. The metric definitions are in source control; PRs that add a metric pass through a privacy-review label that requires a maintainer ack.

### 9.7 Server-side observability

- Per-account simulation-tick latency p50/p95/p99 (without account_id in metric labels — only aggregate distribution).
- Event-ingest latency.
- Snapshot-serve latency (cache hit/miss bins).
- Notebook-emission counts (aggregate; not per-account).
- Mailer success/failure counts; magic-link replay/expiry counts.
- Tick-lag-per-shard (which shards are falling behind).

Alarms:

- Simulation-tick p99 > 5 seconds for >2 minutes → page on-call. (PRD-named.)
- Snapshot-serve p95 > 200ms for >5 minutes → page on-call.
- Event ingest 5xx rate >1% for >2 minutes → page on-call.
- Magic-link delivery failure rate >5% for >5 minutes → page on-call.
- Tick-lag on any shard >5 minutes → page on-call.

### 9.8 What we deliberately don't measure

- We do not measure per-bird drift trajectories aggregated across accounts.
- We do not measure "how many calls per session per bird across the user base."
- We do not measure session-frequency-per-account beyond what aggregate session-duration histograms (anonymized, no per-account dimension) provide.
- We do not retain interaction event logs in any analytics warehouse.
- The simulation database has no read replica that the analytics warehouse can connect to. There is no path for per-account simulation state to flow into aggregate dashboards. This is enforced at the network and IAM layer (see §11.5).

---

## 10. Calibration calls and defended choices

A list of the calls the PRD leaves open or implicit, made here so the engineering team has a starting position. These are explicitly labeled as starting calibrations subject to canary measurement.

### 10.1 Presence "recent input" window

The PRD says "a few minutes." We start at **3 minutes**. Rationale: long enough to absorb a user sitting and watching without input (the actual product), short enough that a backgrounded browser doesn't accumulate presence. We will measure during canary whether the median user's "I am watching" sessions show input within this window; if many sessions show 4–5 minute input gaps in actual usage (users genuinely just watching), we extend toward 5 minutes.

### 10.2 Tick cadence

Default 60s; active 20s; deep-idle 5min. Rationale: 60s matches the PRD; 20s during active sessions makes mood transitions feel responsive on-screen without overloading the simulation; 5min during long idle keeps time-of-day flowing without burning resources.

### 10.3 Drift coefficients

Starting numbers in §5.3. Calibration target: `+0.0008` per presence-minute on weighted traits. We test with the synthetic-user harness (§5.11) before launch and adjust if the 3-week-visible target isn't hit at the simulated regular-visitor pattern.

### 10.4 Notebook entry rate

At most 1 entry per 24h for routine seeds, with up to 2 "noteworthy" entries per week. Calibration target: a reasonably-engaged user sees ~5–10 entries in their first month, weighted toward early observations of bird behavior.

### 10.5 Mood enum count

Six states: `wary, content, curious, drowsy, alert, settled`. The PRD names four examples; we add `alert` (named separately, distinct from `curious`) and `settled` (the post-settle/late-night state). More states would dilute mood-readability via idle motion; fewer would muddle the day/night and post-settle distinction.

### 10.6 Personality vector exposure in account export

The PRD bans personality vector exposure as a UI surface. The export endpoint surfaces the values in a downloaded JSON. We argue this is consistent: the export is the user's own data being handed back to them by their explicit action; it is not a UI surface, not a stat panel, not a dashboard. The user who clicks "export my aviary" gets their data; they do not get a "your bird is at boldness 0.62" UI. If the team disagrees during build, the safe alternative is to omit `personality` from the export and ship only the user-visible state (names, species, current mood). We default to including it because the rationale "the user's data is the user's" is strong, but we flag it as a defended choice rather than a default.

### 10.7 Snapshot inlining size

The bootstrap HTML inlines a snapshot (~1–3 KB JSON). We cap the inlined size at 10 KB; above that, the snapshot is fetched via a parallel request that's dispatched before the bundle. The cap exists because the bootstrap HTML is on the critical path and we don't want to unintentionally bloat it.

### 10.8 Visit-session token lifetime

Visitor sessions last for 4 hours from first use. This balances "let the visitor sit a while if they want to" with "revocation should take effect quickly enough." Snapshot pulls from a visitor session re-validate against the invite's revoked/expired status on every request; the 4-hour lifetime is a backstop, not the primary revocation gate.

### 10.9 Light-state cross-fade durations

Sunrise: 30 minutes of real time blending across the full warm-light shift. Midday-to-evening: 60 minutes. Evening-to-night: 30 minutes. These are slow enough to not be noticed within a session, fast enough that a user returning after several hours sees the shift correctly. (The cross-fade is a continuous interpolation rather than discrete state transitions.)

---

## 11. Architectural Invariants and Enforcement

A small list of system-wide invariants and how each is enforced beyond "we will be careful." The list is short on purpose; each item is structural rather than aspirational.

### 11.1 Server is the only writer of personality state

- Database GRANT: only the `simulation_service_role` has UPDATE/INSERT on `birds.personality`. Other service roles get SELECT only.
- Application code review: a CODEOWNERS rule routes any change to the `birds` table or the `personality` JSONB to the simulation team for review.
- Lint: a custom lint rule rejects any client-side reference to `birds.personality` writes.

### 11.2 Asymmetric drift (only upward)

- Database trigger on UPDATE rejects any new row where any personality trait < the prior value. (See §5.4.)
- Application code: the only mutation site uses an explicit `Math.max(0, delta)` wrapper.
- Test: a property-based test against the simulation tick exercises arbitrary event sequences and asserts no resulting row violates the invariant.

### 11.3 Email PII is bounded

- Database: `email_encrypted` and `email_lookup_hash` are the only columns that hold email-derived data. CI grep on the schema rejects any new column whose name contains `email` outside of these.
- Lint: a custom rule rejects log statements that include the `email_*` fields. Logs use `account_id` only.
- Telemetry schema: no email-derived field is allowed.

### 11.4 Personality vector is never serialized to a client outside of export

- API gateway response inspector rejects any payload containing `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` as keys, with one whitelisted route: `GET /v1/account/export`. This is a runtime check on a sample of responses and a CI integration test on every endpoint.
- The `SnapshotV1` schema sends only `plumage_saturation_render` (a derived hint), not the raw vector.

### 11.5 Telemetry/simulation database isolation

- Network policy: the telemetry service has no network route to the simulation database. The simulation database's security group denies inbound from the analytics VPC.
- IAM: no analytics role has read access to the simulation database.
- Data pipeline: there is no ETL job that copies from simulation to analytics. A periodic audit (quarterly) confirms no such job exists.

### 11.6 No recorded audio in the bundle

- CI gate: the bundle build fails if any `.mp3`, `.wav`, `.ogg`, `.aac`, `.flac`, `.opus`, `.weba` file is included or if any URL of those extensions appears in the JS source.
- This is a hard rule. An exemption requires an architect override and merges through a `audio-bundle-exception.md` review.

### 11.7 Voice audit gate on naturalist-prose surfaces

- Any change to the notebook prose templates, the narration templates, or any user-facing string in the aviary surface goes through a CODEOWNERS review by a designated voice reviewer.
- A CI lint scans for forbidden patterns: `you visited`, `streak`, `achievement`, `unlocked`, `welcome back`, `level`, `score`, `XP`, `badge`, exclamation marks in naturalist-voice surfaces (the matter-of-fact surface allows them sparingly), `your friend`. New forbidden patterns can be added by the voice reviewer without engineering signoff.

### 11.8 Notice, never announce

- Architecturally enforced by the absence of a notification service. We do not build a notifications surface in v1; visit-notification opt-in (§social_optional) emails through the mailer queue rather than a push or in-product notification system.
- A lint rule rejects new code paths that introduce in-product banners or toast surfaces.

### 11.9 Request-level "no per-account dimension" on metrics

- The metrics SDK rejects metrics that carry an `account_id` label.
- A CI test enumerates all metric definitions and asserts none carry per-account dimensions.

### 11.10 Append-only event log

- The `events` table is append-only at the GRANT level: services have INSERT and SELECT, no UPDATE or DELETE.
- The simulation service consumes events by reading; it never marks them consumed by mutation (it tracks the highest applied event_id in its own state).
- Account hard-delete (after 30-day soft window) is the only path that removes events, and it removes them by partition drop, not by row-level DELETE.

---

## 12. Sync Model

Multi-device sync is, per the PRD, a property of the architecture rather than a feature. This section names exactly what makes that work and what failure modes we still have to handle.

### 12.1 Single canonical aviary per account

There is one row per bird, one row per account, one snapshot view derivable from those rows. Two devices reading the snapshot endpoint simultaneously read the same row through the same cache, with the same `tick_id`, and render the same scene.

### 12.2 No client-side personality state

Clients hold no persistent personality state. The local IndexedDB caches:

- The account ID and current session token.
- The most recent `SnapshotV1` (purely derived; refreshed on every visibility change and on the keepalive cadence).
- The accessibility settings (mirrored from server settings; the local cache is for offline-first feel; the server settings are authoritative on next sign-in).
- A small queue of pending event-pushes waiting for a network round-trip.

If the local cache is wiped, the next snapshot pull restores everything that matters.

### 12.3 Event-log ordering

Events from multiple devices interleave at the server based on `received_at`, not `client_ts`. The simulation tick processes events in `event_id` order. Two devices submitting events near-simultaneously interleave deterministically; the simulation tick is order-deterministic for the same event sequence.

### 12.4 Listen-in is per-device

If two devices have the same account signed in, each device's listen-in state is local. Device A focused on Pip and Device B focused on Wren is fine; each device renders its own audio mix. The events `listen_in_start` and `listen_in_end` carry the `bird_id` but the simulation only uses them for drift; the per-device audio mix is local UI state.

### 12.5 Settle is account-wide

Settle is account-wide: a `settle_triggered` event applied at the simulation level forces the lighting on the next snapshot for all clients. (This is the right behavior — settle is a goodbye gesture; it would be weird if the user's phone showed a "settled" aviary while their laptop still showed midday.)

### 12.6 Conflict surface

In rare cases — an in-flight session timing out mid-write, a server-side outage during snapshot delivery, a network partition — clients may see stale snapshots or fail to push events. Behavior:

- Failed event pushes go into the local pending queue and retry with exponential backoff. The `client_event_uuid` makes retries idempotent.
- Stale snapshots: the renderer continues with the stale snapshot until the next pull succeeds. A snapshot more than 5 minutes stale during an active session triggers a quiet "trying to reach the aviary" status in matter-of-fact voice in the top bar (very small, low contrast, not a banner — closer to a status tick).
- Hard auth failures (token revoked while in session, invite revoked mid-visit) drop the user back to the sign-in or "this visit is no longer available" surface in matter-of-fact voice.

### 12.7 Account export and account import boundary

Account export hands the user a JSON. There is no account import in v1. (This is consistent with the v1 scope: import would create a "did personality drift carry over from the imported file?" question with no clean answer that doesn't introduce an alternate write path for personality state. We close the door on that in v1.)

---

## 13. Rollout

### 13.1 Build-time staging

1. Internal alpha: a small team of ~10 internal users, real accounts, real birds. Goal: instrument validation, basic affective check ("does this feel like a real aviary?").
2. Closed beta: ~200 invited users over 4 weeks. Goal: drift calibration validation against real behavior; performance check against real devices/networks; bug surface.
3. Open beta: invite-link signup, capped at ~5,000 active accounts. Goal: server scaling shake-out, telemetry rollup validation, observability tuning.
4. Public launch: open signup with magic-link.

### 13.2 What we instrument from day one (operational)

- Synthetic perf checks running from launch (4 geographies, 3 browsers each).
- Aggregate RUM for navigation timing and time-to-first-bird.
- Simulation-tick latency per shard.
- Event-ingest latency and 5xx rate.
- Snapshot-serve latency.
- Mailer delivery success/failure.
- Audio-context error counts (aggregate; a sustained spike means a browser regression somewhere).

### 13.3 What we don't instrument

- We don't instrument session count, visit frequency, or any per-account engagement-style metric. We will know aggregate monthly active accounts; we will not know that a specific account visits daily. This is a privacy stance and a product stance — we are not building a product where engagement is a metric to optimize.
- We don't instrument per-bird interaction history outside the per-account simulation database. The simulation database does not feed analytics.

### 13.4 Bird-cap ramp

- Starter: 2 birds. Stays at 2 for several months by design.
- Third bird: offered at aviary-age ~3 months (calibration target; final value tunable per canary).
- Fourth: ~6 months. Fifth: ~12 months. Sixth: ~18 months. Seventh: ~24 months.
- The pacing is deliberately slow. Faster pacing teaches "more attention earns more stuff."
- Aviary-age is tracked from `accounts.created_at`. Bird offers appear in the top-bar offer surface as a "a new species is around" note in naturalist voice (specific, lowercase: "a small olive-green warbler has been heard near the back perch") — and the user can accept, decline, or ignore. Declining doesn't penalize; the offer reappears occasionally if not accepted.

### 13.5 Configuration for tuning

- Drift coefficients, tick cadences, mood transition pulls, weather probabilities, notebook seed thresholds, and bird-offer pacing are all configuration values in the simulation service, not hard-coded.
- Configuration changes go through code review and deploy through the normal build pipeline; there is no live-tuning admin UI in v1. (Live tuning is too easy to misuse; the asymmetry rule and other invariants are not live-tunable.)

### 13.6 Post-launch calibration

- Week 1 post-launch: read the synthetic perf and simulation-tick latency baselines; alarm if any threshold trips.
- Week 4 post-launch: review the synthetic-user drift harness against real (anonymized aggregate) presence-time distributions; if real users present materially differently than the synthetic visitor, recalibrate drift coefficients.
- Quarter 1 post-launch: review the privacy-isolation audit (no analytics path to simulation db).

---

## 14. Risks and Mitigations

The risks below are the ones most likely to silently fail. Each has a concrete mitigation; "we will be careful" is not a mitigation.

### 14.1 Drift miscalibration

- **Risk**: drift is too fast or too slow; the "feels alive over weeks" promise either exaggerates into a Tamagotchi-fast cadence or flattens into a screensaver.
- **Mitigation**: synthetic-user harness pre-launch (§5.11). Post-launch quarterly review of drift trajectories on aggregate presence-time distributions (without per-account dimensions). Drift coefficients are configuration, but downward adjustments to traits already drifted are not allowed (per the asymmetry rule); recalibration shifts the rate, not the existing values.

### 14.2 Sync correctness on personality state

- **Risk**: a code change introduces a client-side write of personality state, breaking the architectural invariant.
- **Mitigation**: database GRANT prevents the write at runtime. A CI test issues PATCH attempts against the personality field from a non-simulation role and asserts they fail.

### 14.3 Audio uncanniness

- **Risk**: procedural calls produce an uncanny-valley effect where each call sounds organic but the per-bird voice is inconsistent or chorus mixing produces an artifact.
- **Mitigation**: explicit per-bird identifiability test (§7.2). A small panel of listeners (in alpha) blind-identifies which bird made which call; pass threshold is 70% per-bird identification across mood and drift. Chorus is auditioned by an audio engineer for the "two birds together" feel; failure to pass auditioning blocks launch.

### 14.4 Accessibility regressions

- **Risk**: a feature change breaks the screen-reader narration cadence, or the reduced-motion renderer drifts into a stripped-down version of the regular renderer.
- **Mitigation**: reduced-motion is a separate renderer module (§6.7), not a runtime branch — this prevents accidental drift. Narration prose is generated by templates with a CI test that asserts cadence and presence on canonical state changes. Contrast is a CI test against actual rendered backgrounds.

### 14.5 Time-to-first-bird regression

- **Risk**: a feature change adds bytes to the bundle or a network round-trip to the bootstrap path; first-bird crosses 500ms; the central aliveness conceit collapses.
- **Mitigation**: CI gate on bundle size and on synthetic time-to-first-bird. Inlined snapshot path is documented as load-bearing; a change that removes inlining requires architect review.

### 14.6 Notebook prose drift into generic-event-log voice

- **Risk**: notebook prose drifts toward stock event-log phrasing ("a bird greeted you"), breaking the product's voice.
- **Mitigation**: voice audit CODEOWNERS review on prose template changes (§11.7). CI lint for forbidden patterns. Sample prose audit during alpha and beta; reviewer signoff required for launch.

### 14.7 Per-account state leaking into telemetry

- **Risk**: a metric is added with `account_id` as a label, leaking per-account data into aggregate telemetry.
- **Mitigation**: metrics SDK rejects per-account labels at ingest (§11.9). CI enumerates metric definitions and asserts the constraint.

### 14.8 The first "harmless" engagement feature

- **Risk**: a contributor proposes a "small" streak counter, a "quiet" calendar in settings, a "friendly" return-day notification — the foothold that compounds into a different product.
- **Mitigation**: §11.7 lint covers forbidden phrasing. The non-goals list is referenced in PR-template prompts. The product reviewer (a designated role on the team) has explicit veto on any feature that surfaces visit-frequency, achievement-style status, or notification surfaces. This is a process mitigation that has to be culturally maintained; the lint and architectural absences raise the cost of the slip but don't prevent it on their own.

### 14.9 Magic-link delivery failure

- **Risk**: the user's email provider greylists or drops the magic-link email; the user can't sign in; they bounce.
- **Mitigation**: mailer infrastructure with reputable delivery service, SPF/DKIM/DMARC properly configured. Aggregate delivery success rate alarm. A user who can't get the link sees a matter-of-fact help page with "try a different email address or get in touch" copy.

### 14.10 Visitor-session abuse

- **Risk**: a bad-actor visitor sits on a session forever; or shares the invite link with others.
- **Mitigation**: visitor sessions expire 4h after first use (§10.8). Invite tokens are one-link-per-invite; the host can revoke instantly. Visitor sessions get rate-limited snapshot pulls (no faster than the host's own keepalive cadence).

### 14.11 Server-side simulation tick falling behind

- **Risk**: a hot account or a worker incident causes tick-lag; the user opens the aviary and gets a snapshot from 10 minutes ago.
- **Mitigation**: tick-lag-per-shard alarm (§9.7). Sharded worker pool with capacity headroom. On snapshot pull, if the last `tick_id` is older than a threshold, the simulation service forces a synchronous catch-up tick before serving the snapshot.

### 14.12 Database trigger on personality update produces false positives

- **Risk**: a legitimate write (e.g., a schema migration) is rejected by the asymmetry trigger.
- **Mitigation**: schema migrations that touch the `personality` JSONB structure go through a maintenance role that has TRIGGER-DISABLE permissions on a temporary basis; the migration runs as a single atomic transaction with the trigger re-enabled before commit. Migration runbook documents this.

### 14.13 iOS Safari audio policy changes

- **Risk**: a Safari update changes audio-context unlock behavior; calls fail to play.
- **Mitigation**: synthetic perf monitoring includes Safari; an audio-context-error spike triggers an alarm. The fallback is graceful silence with captions — the aviary still works visually.

### 14.14 Account export contains too much detail

- **Risk**: the export's inclusion of personality vector numerics is later viewed as the "stat panel" we explicitly refused.
- **Mitigation**: §10.6 documents the call. If the team decides to remove personality from the export, the change is one schema field and a one-line code change; it does not affect the rest of the system. Default position is to include it because the user's data is the user's; we flag this as the most defended single call in the plan.

---

## 15. Appendix — Module map

A map of the v1 codebase shape, for the engineering team's planning.

```
client/
  bootstrap/                # inlined-snapshot decoder, audio-context shim
  renderer/
    canvas/                 # Canvas2D scene composition
    motion/                 # idle-motion state machines per mood
    interpolation/          # snapshot-to-snapshot smoothing
    reduced/                # reduced-motion renderer (separate strategy)
  audio/
    grammar/                # procedural call-grammar runtime
    scheduler/              # WebAudio scheduling
    chorus/                 # chorus mixing
    captions/               # caption generation from grammar
  events/
    queue/                  # local event push queue with retry
    presence/               # presence-conjunction monitor
  ui/
    topbar/                 # top bar (HTML/CSS, separate from canvas)
    settings/               # account/accessibility settings (code-split)
    notebook/               # field notebook view (code-split)
    visit/                  # visit invitation flow (code-split)
  narration/                # aria-live narrator
  state/                    # snapshot cache, settings cache (no personality)
  net/                      # snapshot pull, event push, auth

server/
  auth/                     # magic links, sessions
  accounts/                 # accounts, settings
  simulation/
    tick/                   # the per-account tick worker
    drift/                  # drift function (asymmetry-enforced)
    mood/                   # mood scorer
    perch/                  # perch selector
    weather/                # ambient weather rolls
    interaction/            # bird-to-bird passes
  events/                   # append-only event log
  notebook/                 # notebook seed-to-prose generator
  narration/                # naturalist narrator from snapshot
  visits/                   # invite issuance, revocation, visitor sessions
  mailer/                   # magic links, exports, invites
  cron/                     # tick scheduler, soft-deletion sweeper, invite expirer
  telemetry/                # aggregate-only ingest

infra/
  db/
    schema/
      accounts.sql
      birds.sql
      events.sql            # partitioned by account_id
      notebook_entries.sql
      sessions.sql
      magic_links.sql
      invites.sql
      visit_log.sql
    triggers/
      asymmetric_drift.sql  # the load-bearing trigger
    grants/
      simulation_role.sql   # only role with personality UPDATE
  network/
    isolation/              # telemetry-vs-simulation network policy
  ci/
    bundle_size_gate.yaml
    no_recorded_audio.yaml
    voice_lint.yaml
    metrics_no_account_id.yaml
    contrast_test.yaml
    memory_30min_test.yaml
    drift_calibration_test.yaml

design-system/              # palette, type, focus styles, contrast tables
```

---

## 16. Appendix — Decisions deferred to the visual designer or audio designer

Items the engineering plan needs to integrate with but are not engineering's call:

- Exact palette colors and contrast ratios per surface.
- The visible focus-outline treatment.
- Specific bird silhouettes per species.
- Specific motif libraries per species (the audio designer composes the motifs; the engineering plan integrates them through the procedural grammar).
- The exact size and placement of caption text near each bird.
- The specific naturalist-voice phrasing used in templates (a writer/voice owner is named on the project).

These are integration points; the plan above accommodates them without prescribing them.

---

End of plan.
