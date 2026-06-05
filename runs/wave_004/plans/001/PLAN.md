# Pocket Aviary — Phase 1 Implementation Plan (Run 001, Wave 004)

This plan interprets the PRD into an executable engineering blueprint. It is written for a frontier engineering team that has read the PRD and needs to know what to build, how the pieces fit, what to instrument, and where the load-bearing decisions live. The plan respects every non-goal in `non_goals.md` and every constraint in `accessibility_perf.md`. Nothing in this plan implements the product; everything in this plan is meant to be turned into code by a separate team.

---

## 1. Scope

### 1.1 In scope for v1

- Web-only single-page application (SPA) served from a CDN, last two major versions of Chrome / Safari / Firefox / Edge.
- Single-user accounts, email magic-link sign-in (no passwords, no SSO at v1).
- One canonical aviary per account, with a hard cap of seven birds and a starting roster of two starter birds.
- Six-species starter pool, randomly assigned per new account (no catalog, no user selection at adoption).
- Server-side simulation tick (~once per minute, calibrated at build time) that is the sole writer of personality and mood state.
- Multi-device sync as an architectural property of the server-canonical model — not a separate feature.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only).
- Per-bird adoption + rename flow; user-assigned names with default suggestions.
- Bird-count expansion tied to aviary age, not visit count, not interaction score.
- Visit invitations: per-invite opt-in, email magic-link, read-only ambient view for visitors, host-side visit log, revocable invites, 30-day expiry.
- Procedural WebAudio call synthesis, client-side, with chorus mix and listen-in mix.
- Accessibility surfaces: screen-reader narration, call captions, reduced-motion mode (designed, not stripped), keyboard navigation, WCAG AA contrast on all user copy.
- Aggregate-only telemetry pipeline; per-bird interaction state never leaves the per-account simulation database.
- Account export (JSON snapshot) and account deletion (30-day soft, then hard).
- Synthetic account UUIDs; email stored only on the account record, never used as a key.

### 1.2 Out of scope for v1 (non-goals, restated for engineering emphasis)

- Native iOS / Android apps. The data model and protocols do not design for native-client constraints.
- Gamification of any kind: no streaks, no days-visited counter, no green-dot calendar, no achievements, no XP, no tiers, no level-up confetti, no badges, no "harmless" celebration on a milestone. There is no settings toggle that adds these. The refusal is absolute and survives future versions.
- Tamagotchi-style mechanics: birds do not die, get hungry, decay visibly, or show distress. There is no happiness meter visible or hidden to the user. The drift function is monotonic toward expressive, never downward on neglect.
- Social-network surfaces: no public feed, no discovery, no follow graph, no profiles, no comments, no chat, no leaderboards, no shared aviaries, no co-presence. The single affordance is the per-invite read-only visit.
- Notification surfaces: no push, no email-on-aviary-events, no in-product toasts or banners that announce the user. Visit notifications are an off-by-default settings toggle; everything else is silent.
- Streak counter or any widget that surfaces visit-frequency. The notebook can write that Pip greeted first today; it cannot write that the user has been here every day this week. The line between "observation of the aviary" and "observation of the user's behavior" is hard and reviewed at every notebook-content PR.
- Personality-vector numbers exposed anywhere in the product, including debug, accessibility, or stats surfaces. There is no toggle for this.
- Welcome-back toasts, banners, modals, or any textual greeting surface.
- Recorded audio of any kind. Calls are procedural; we do not ship a recorded-audio fallback when WebAudio is unavailable — silence with captions is the fallback.
- Public discovery of aviaries and ranking/leaderboards. We do not compute the underlying stats for this purpose.
- Anything from `non_goals.md`. The plan is not a venue to argue with the PRD; the PRD is the input.

### 1.3 Defensible calls on ambiguity

These are the four places where the PRD leaves a real decision unspecified. Each is resolved with a single defensible call so the engineering team is not re-litigating them:

- **Tick cadence.** "Slow cadence (~once per minute)" is the PRD phrasing. We commit to **60s tick** as the v1 target with a calibration window of 45–75s permitted during the alpha. The drift function and mood transitions are designed so that the personality-delta per tick is small and additive, not so that the tick interval itself is load-bearing.
- **Per-bird offer cooldown.** "A few minutes" is the PRD phrasing. We commit to **4 minutes** per bird, with offer type and per-bird curiosity used to vary the effective cooldown by ±1 minute. The cooldown is a functional guard against curiosity-trait saturation; we will monitor drift velocity during alpha to confirm 4 minutes is correct and adjust in the calibration pass.
- **Presence activity window.** "A few minutes" is the PRD phrasing, leaning toward the longer side. We commit to **3 minutes** as the v1 default. Presence decays 3 minutes after the last pointermove / keypress, holding all three presence signals (visible, focused, recently active) required. This is longer than most "is the user idle?" libraries default to, intentionally.
- **Activity window for the simulation tick to consider a session "recent".** Not specified. We commit to the simulation tick considering the last **15 minutes** of event log when computing the next delta. Events older than 15 minutes are still in the log for long-horizon drift, but the per-tick fast-feedback loop is bounded at 15 minutes.

Each of these is recorded as a v1 constant in the simulation service's `config/calibration.rs` (or equivalent) and is changed only through the calibration pipeline.

---

## 2. Architecture

### 2.1 High-level service shape

Pocket Aviary is a thin browser client talking to a server-owned canonical state. The server runs three logical services inside one binary or one deployable, behind one HTTPS endpoint per environment:

- **Web client** — static SPA, served from a CDN. Initial HTML, CSS, and the first JS chunk are cacheable at the CDN edge. The client holds no personality state.
- **Aviary API** — a JSON HTTP API (REST-shaped with one streaming-friendly endpoint) that serves state snapshots, accepts interaction events, and handles account operations. No business logic for simulation in the request path; the request path validates, persists the event, and returns. The simulation tick is a separate worker.
- **Simulation worker** — a background process that runs the tick loop for all active accounts. Reads from the event log, writes to the canonical state store, and is the only writer of personality vectors and mood state.
- **Auth service** — magic-link issuance, session-token minting, session revocation. Lives in the same deployable as the Aviary API for v1; this is an internal-only split and there is no auth-as-separate-service boundary yet.
- **Account data** — a small set of CRUD endpoints (export, delete-soft, delete-hard, list-sessions, revoke-session, change-email, list-visit-log, send-invite, revoke-invite).

A queueing system is not required at v1 scale. The event log is a single relational table with a (account_id, ts) index; the simulation worker polls or uses LISTEN/NOTIFY to wake on new events.

### 2.2 Client / server split — the load-bearing rule

**The server is the only writer of personality state. The client is the only producer of interaction events. The boundary is hard and is enforced by code review and an integration test that fails the build if a client endpoint that mutates personality is reachable.** This is the rule that makes the rest of the architecture honest. Drift correctness, multi-device sync, and the no-last-write-wins invariant all collapse to this single line.

The client:
- Renders state. Never derives state from raw events.
- Submits interaction events. Never submits absolute state values.
- Pulls snapshots. Never reads a "delta to apply locally" — that would re-introduce a client-side state machine we have explicitly refused to build.

The server:
- Owns canonical state. Personality vectors, mood, perch state, call timing windows, notebook cursor.
- Owns the simulation tick. Period.
- Exposes a snapshot endpoint and an event-submit endpoint. Nothing else.
- Treats the client as a possibly-malicious renderer. Validation is server-side, even on values the client could compute itself, because we want a future analytics or alternative-client surface to be safe by default.

### 2.3 Render pipeline boundary

Rendering is fully client-side. The client receives a compact JSON snapshot describing per-bird state (position on the scene, current pose, current mood, call-timing window, plumage-saturation visual hint, current "ambient motion slot") and renders using:

- A small 2D scene composed of three planes (background, midground/perch, foreground/ornament) drawn to a single canvas via WebGL (preferred) or 2D canvas (fallback).
- A bird-rendering layer that composites the current pose per bird, with personality-shaped micro-motion applied per frame.
- An audio layer (WebAudio) driven by the same snapshot — call timing windows tell the audio layer when to schedule a procedural call.
- A narration layer that emits a screen-reader prose update at the slow cadence specified in `accessibility_perf.md`.

The boundary is intentional: the client never reaches into the simulation to ask "what should happen next." It only ever asks "what is the state right now, and what changed?" and renders. This is what makes the "aviary appears already in motion" property achievable — the client opens, pulls a snapshot, places birds at their current positions in their current motions, and starts rendering as if it has been rendering all along.

### 2.4 Deployment topology

- Single-region origin (we deliberately do not need global low-latency reads; the aviary is one user's account, and a single-region read is fine at v1 scale).
- CDN edge for static assets (HTML, JS, CSS, SVG species silhouettes, plumage palettes).
- Origin handles API + simulation worker. One database, one origin, one worker. We do not split this for v1; the operational complexity of multi-service is not justified at v1 scale and the load profile is not yet a constraint.

---

## 3. Data model

The data model is the foundation that everything else in this plan rests on. Every table below is server-canonical unless explicitly noted. All account references use a synthetic UUID, never email.

### 3.1 `accounts`

| Column | Type | Notes |
|---|---|---|
| `account_id` | UUID, primary key | Generated at account creation. The only identifier used by every other table, by inter-service messages, by telemetry, by logs. |
| `email_encrypted` | bytea | AES-256-GCM, KMS-managed key. Stored once, on this row. Never used as a key, never logged, never copied. |
| `email_lookup_hash` | bytea | HMAC-SHA256 of the lowercased email under a separate KMS key. Used only for the "look up an account by email" path. The plaintext email is never compared in SQL. |
| `created_at` | timestamptz | |
| `aviary_created_at` | timestamptz | Distinct from `created_at` by a small margin in the rare case the aviary is reset; otherwise equal. Drives the bird-count expansion rule. |
| `deletion_state` | enum {active, soft_deleted, hard_deleted} | soft_deleted sets `deletion_requested_at`; hard_deleted is the terminal state after 30 days. |
| `deletion_requested_at` | timestamptz, nullable | |
| `deletion_hard_at` | timestamptz, nullable | When the 30-day window expires. |
| `visit_notifications_enabled` | bool, default false | Off by default per `social_optional.md`. |
| `current_session_token_id` | UUID, nullable | Tracks the active session for fast revocation. |
| `pending_email` | text, nullable | Used by the email-change flow until verified. |

The `email_encrypted` and `email_lookup_hash` split is non-negotiable. Email is PII; the synthetic UUID rule cuts PII out of every log line and every partition key from the source.

### 3.2 `birds`

| Column | Type | Notes |
|---|---|---|
| `bird_id` | UUID, primary key | Stable internal identifier. Never changes for the life of the bird, never reused, never reassigned. The day-one bird is *that* bird forever. |
| `account_id` | UUID, FK | |
| `species_id` | smallint, FK → `species.id` | |
| `name` | text | User-assigned. Renamable at any time. Renaming does not change personality, mood, or call. |
| `personality_vector` | jsonb | The hidden trait set. See §6.2 for shape. |
| `current_mood` | text enum | One of {wary, content, curious, drowsy, alert}. Persisted across sessions. |
| `perch_zone` | enum {front, middle, back} | Where the bird currently is. Driven by mood and personality. |
| `perch_x` | float, 0..1 | Horizontal position on the perch zone. Driven by bird-to-bird spacing and individual variation. |
| `state_version` | bigint | Monotonically increasing. Used for snapshot versioning and conflict surface detection. |
| `last_state_change_at` | timestamptz | |
| `adopted_at` | timestamptz | |
| `last_listen_in_at` | timestamptz, nullable | For drift weighting and for the listen-in cap. |
| `last_offer_at` | timestamptz, nullable | For the per-bird offer cooldown. |
| `last_presence_event_at` | timestamptz, nullable | For drift weighting. |

The personality vector is stored as a single jsonb blob with a fixed schema (5 fields, each a float in [0, 1]). It is **only** written by the simulation worker. There is no code path on the API server that mutates this column; the API endpoint that nominally updates a bird is rejected at the request handler level for any non-event-log path, and a CI test fails the build if a non-worker client can write to this table.

### 3.3 `species`

| Column | Type | Notes |
|---|---|---|
| `id` | smallint, primary key | |
| `common_name` | text | Internal label only; not user-facing. |
| `call_motif_library_id` | text | References a static asset bundle shipped with the client. |
| `default_plumage_palette` | text | |
| `is_nightjar` | bool | One species in the v1 pool is nightjar-like (active at night per `aviary_layout.md`). |
| `silhouette_asset_id` | text | |
| `size_class` | smallint | For perching layout. |

Six species are seeded at deployment. The species pool is static at v1; no per-account species variation, no rarity, no user pick.

### 3.4 `event_log` (append-only)

The event log is the source of truth for all user-originated inputs. It is the only path by which client activity influences state.

| Column | Type | Notes |
|---|---|---|
| `event_id` | UUID, primary key | |
| `account_id` | UUID, FK | Partitioning key. |
| `ts` | timestamptz | Event time on the server clock at ingestion. |
| `event_type` | enum | See §3.5. |
| `bird_id` | UUID, nullable | For per-bird events. |
| `payload` | jsonb | Event-type-specific data. |
| `ingested_at` | timestamptz | For latency monitoring. |
| `client_session_id` | UUID | The session that produced the event. |

The log is append-only. Rows are never updated, never deleted (except by hard account deletion at 30 days). The simulation worker consumes the log in `(account_id, ts)` order, applies deltas to personality state in that order, and marks events processed. Re-processing on worker crash is idempotent at the trait level — each event has a fixed delta shape — so re-application is safe.

### 3.5 Event types

The event type set is closed at v1. The set is small and the schema for each is fixed and versioned.

- `presence_tick` — fired by the client on a slow cadence (30–60s) while the three presence signals all hold. Payload: `{ts_client: number}` (used only for clock-drift detection, not for ranking).
- `presence_end` — fired when any of the three presence signals drops.
- `greeting_observed` — fired when the client renders the return-greeting for a session. Lets the server know a greeting was actually seen (some clients may not render a greeting if the user closes the tab immediately).
- `listen_in_start` / `listen_in_end` — payload: `{bird_id, duration_ms}`.
- `offer_seed` / `offer_song` / `offer_pool` — payload: `{bird_id, offer_type, accepted: bool, observed_mood: text}`. Accepted is server-determined (not client-reported) by the simulation worker, based on the bird's current mood and personality.
- `settle_gesture` — payload: `{triggered_at: ts}`. Recorded for notebook use and the gentle evening shift; not used for drift beyond ending the presence window.
- `notebook_entry_write` — internal event type emitted by the simulation worker when a notebook entry is generated. The event log records the fact of the entry; the entry text lives in `notebook_entries`.

Client-submitted events are validated for shape, for ownership (the session token resolves to the `account_id` referenced), and for rate limits. They are never trusted for state — only for "this user did this thing at this time."

### 3.6 `presence_state`

A per-session row tracking the current presence window. Updated on every `presence_tick` / `presence_end` event.

| Column | Type | Notes |
|---|---|---|
| `session_id` | UUID, primary key | |
| `account_id` | UUID, FK | |
| `last_presence_tick_at` | timestamptz, nullable | |
| `last_pointer_or_key_at` | timestamptz, nullable | From the client's presence_tick payload. |
| `ended_at` | timestamptz, nullable | Set on presence_end or session expiry. |
| `total_presence_ms` | bigint, default 0 | Accumulated presence time for this session. |

This table is the operational source for "is this user currently present?" and for accumulating presence-time per session. It is not the source for drift directly; the simulation worker reads from `event_log` to compute drift, and uses `presence_state` only as a sanity check (the worker double-checks that a presence_tick the client wrote was consistent with the prior state).

### 3.7 `notebook_entries`

| Column | Type | Notes |
|---|---|---|
| `entry_id` | UUID, primary key | |
| `account_id` | UUID, FK | |
| `written_at` | timestamptz | When the simulation worker generated the entry. |
| `entry_text` | text | The naturalist prose. Generated by a small templating system (see §6.6) — never freeform-LLM, never human-edited. |
| `entry_kind` | enum | One of {mood_observation, bird_pairing, weather_note, time_of_day, return_observation, chorus_note}. Drives templating and cadence. |
| `notebook_revision` | int | Increments on entry updates. The notebook is read-only to the user, so revisions are server-driven and visible in the UI as "edited at" markers if the user scrolls to the entry detail. |

Entries are not generic event logs. The generator produces a small, observation-style sentence specific to the moment and the birds. Cadence is sparse (see §6.6) — roughly one entry every few days for a regularly-visited aviary, more often when something noteworthy happens. The notebook is read-only and the user cannot edit, delete, or annotate. Old entries do not get archived or hidden; the user can scroll back indefinitely.

### 3.8 `visits` and `visit_invites`

| Table | Column | Type | Notes |
|---|---|---|---|
| `visit_invites` | `invite_id` | UUID, primary key | |
| | `account_id` | UUID, FK | Host. |
| | `invitee_email_encrypted` | bytea | Encrypted at rest. |
| | `invitee_email_lookup_hash` | bytea | Lookup. |
| | `created_at` | timestamptz | |
| | `expires_at` | timestamptz | `created_at + 30 days`. |
| | `revoked_at` | timestamptz, nullable | |
| | `consumed_at` | timestamptz, nullable | Set on first valid use. A used link is invalidated. |
| | `invite_token_hash` | bytea | Hash of the magic-link token. The plaintext is emailed and never stored. |
| `visits` | `visit_id` | UUID, primary key | |
| | `host_account_id` | UUID, FK | |
| | `visitor_email_encrypted` | bytea | |
| | `visitor_account_id` | UUID, nullable | The visitor's Pocket Aviary account, if they have one. Visitors without accounts still consume a visit, but their email is hashed for log purposes. |
| | `started_at` | timestamptz | |
| | `ended_at` | timestamptz, nullable | Set on tab close, session expiry, or revocation. |
| | `last_snapshot_pulled_at` | timestamptz, nullable | |

The visit is **observation, not co-presence**. The visitor pulls the host's state snapshot the same way the host's own clients do, with a read-only session token. The visitor's session is rate-limited and never allowed to submit events. The simulation does not record presence-time or interaction events from a visitor — a visitor sitting and watching for an hour does not drift the host's birds.

### 3.9 `sessions`

| Column | Type | Notes |
|---|---|---|
| `session_id` | UUID, primary key | |
| `account_id` | UUID, FK | |
| `created_at` | timestamptz | |
| `last_seen_at` | timestamptz | |
| `device_label` | text | "Safari on macOS," "Chrome on iPhone," etc. Derived from User-Agent at sign-in. |
| `revoked_at` | timestamptz, nullable | |
| `session_token_hash` | bytea | Hash of the bearer token. The plaintext is in the cookie only. |

A user can list and revoke sessions from account settings. Revocation invalidates the bearer token at the API edge immediately.

### 3.10 `telemetry_aggregate` (operational only)

This table is the only place where aggregated operational telemetry is stored. It is built from the telemetry pipeline (see §10) and is populated by the telemetry worker, not by the simulation worker. It contains counts, histograms, and p99 latencies. It contains no per-account, per-bird, or per-event linkage. The schema is intentionally narrow:

| Column | Type | Notes |
|---|---|---|
| `metric_name` | text | e.g. `api.snapshot.latency_ms.p99`. |
| `bucket_ts` | timestamptz | Time bucket (1-minute or 5-minute, by metric). |
| `value_numeric` | double precision | |
| `dimensions` | jsonb | e.g. `{region: "us-east-1", browser: "chrome"}`. No `account_id`, ever. |

The privacy boundary in `accounts_sync.md` is honored at the data-pipeline level: the telemetry worker has read access to logs and API metrics, but does not have read access to the simulation database. This is a network- and IAM-level rule, not a code-review one.

### 3.11 What is not in the data model

- No row in any table that stores a user's bird-state, notebook entry, or interaction event in a way that is reachable by a per-account aggregate query. Aggregate queries that touch the simulation tables are rejected by the database role used for analytics.
- No "global" tables: no `all_birds`, no `leaderboard_view`, no `public_aviary_directory`. The architectural absence makes the social-network feature's reappearance harder rather than easier, which is the point.

---

## 4. API surface

The API is a small set of endpoints behind a single HTTPS origin. All endpoints are JSON, all responses are versioned (`v1` in the URL path), and all requests are authenticated by a bearer session token except where noted.

### 4.1 Auth endpoints (no auth required to call, except where noted)

- `POST /v1/auth/magic_link` — body: `{email}`. Sends a magic link to the address. Rate-limited per email. Returns 204 on both success and "email not found" (the latter for enumeration resistance).
- `GET /v1/auth/magic_link/consume?token=…` — consumes a magic link, mints a session token, returns it as a Secure / HttpOnly / SameSite=Lax cookie. Used links are invalidated.
- `POST /v1/auth/sign_out` — invalidates the current session.
- `POST /v1/auth/email_change_request` — body: `{new_email}`. Sends a verification link to the new address.
- `POST /v1/auth/email_change_confirm` — body: `{token}`. Commits the change.

Magic links expire after 15 minutes. A used link is invalidated at consumption.

### 4.2 Aviary state endpoints

- `GET /v1/aviary/snapshot` — returns the current canonical snapshot. See §4.5 for shape. Auth required. Returns 200 with snapshot, 304 if `If-None-Match` matches the current `state_version`.
- `GET /v1/aviary/snapshot/stream` — long-poll or Server-Sent Events variant. Holds the connection open and pushes a snapshot on `state_version` change. Falls back to client polling at 30s intervals if the stream is unavailable.
- `GET /v1/aviary/notebook?before=…&limit=…` — paginated notebook entries, newest first. Auth required.
- `GET /v1/aviary/birds/:bird_id` — single-bird detail. Returns a snapshot of the bird plus its recent notebook mentions. The personality vector is **never** included in this response — the API omits it at serialization time, and a test asserts that the response shape is fixed.

### 4.3 Interaction event endpoints (append-only)

All of the following are POSTs with small JSON bodies, all rate-limited per session, all return 202 on acceptance and do not block on simulation processing. The event is durable the moment the API returns 202; the simulation worker picks it up at the next tick.

- `POST /v1/events/presence_tick` — body: `{ts_client, last_pointer_or_key_at}`.
- `POST /v1/events/presence_end` — body: `{}`.
- `POST /v1/events/listen_in_start` — body: `{bird_id}`.
- `POST /v1/events/listen_in_end` — body: `{bird_id, duration_ms}`.
- `POST /v1/events/offer` — body: `{bird_id, offer_type}` (one of `seed`, `song`, `pool`). Acceptance is determined server-side at the next tick, not by the client.
- `POST /v1/events/settle` — body: `{}`.
- `POST /v1/events/greeting_observed` — body: `{bird_id, greeting_kind}`. `greeting_kind` is a small enum.

Each event submission is validated for:
- Session token validity.
- `bird_id` ownership by the session's account.
- Rate limits (per session, per minute, per event type).
- Reasonable payload shape (max sizes, no script-like content).

A submission that passes validation is written to `event_log` synchronously inside a single transaction. The transaction never reads or writes to `birds` — only to `event_log` and `presence_state`. The simulation worker picks it up later. This is the structural guarantee that no client write path can mutate personality state.

### 4.4 Account endpoints

- `GET /v1/account/me` — current account summary (no email, no PII beyond what the user can see in settings).
- `GET /v1/account/sessions` — list of active sessions.
- `POST /v1/account/sessions/:id/revoke` — revoke a specific session.
- `POST /v1/account/export` — generates a JSON snapshot (birds, names, current personality vectors, current moods, notebook entries, account settings) and emails a one-time download link to the verified address.
- `POST /v1/account/delete` — initiates soft deletion. Returns the timestamp the hard delete will occur.
- `POST /v1/account/delete/cancel` — cancels soft deletion. Usable until the hard delete timestamp.
- `POST /v1/account/visit_notifications` — toggles the per-account visit notification preference.
- `GET /v1/account/visits/log` — host's visit log.
- `GET /v1/account/visits/invites` — outstanding and historical invites.
- `POST /v1/account/visits/invites` — body: `{email}`. Creates an invite and emails a magic link.
- `POST /v1/account/visits/invites/:id/revoke` — revokes an invite.
- `POST /v1/account/birds/:id/rename` — body: `{name}`. Renames a bird. Does not change personality, mood, or call. **Does not** change `state_version` (name is a separate field; personality state version is unaffected).

### 4.5 Snapshot shape

The snapshot is the single document the client renders from. It is small (kilobytes), versioned, and stable. The client can be deployed against a snapshot schema version 1 indefinitely.

```json
{
  "state_version": "017f3e2c-...-monotonic-int",
  "account_id": "uuid",
  "aviary": {
    "time_of_day_phase": "morning | midday | evening | night",
    "weather_state": "clear | light_rain | soft_wind",
    "lighting": { "warmth": 0.6, "brightness": 0.7 },
    "active_weather_until": "ts | null"
  },
  "birds": [
    {
      "bird_id": "uuid",
      "name": "Pip",
      "species_id": 3,
      "mood": "content",
      "perch_zone": "front",
      "perch_x": 0.32,
      "pose_slot": "preening | scanning | idle | head_tilt | fluffed | settled",
      "pose_phase": 0.41,
      "call_timing_window": { "next_call_at": "ts", "last_call_at": "ts" },
      "visual_saturation": 0.62,
      "size_class": 1
    }
  ],
  "ambient": {
    "leaves_active": 0,
    "feathers_active": 1
  },
  "is_settled": false,
  "server_ts": "ts"
}
```

Notes on what is and isn't in the snapshot:

- The snapshot does **not** include the personality vector. The vector is server-side state, used by the simulation worker, and is not sent to the client at any path. The client never sees the numbers.
- The snapshot does include the bird's name, current mood (as an enum, not as a label that would imply the user is "managing" it), and a pose-slot + pose-phase that the renderer animates from.
- The pose-slot is a small enum; the actual pose-frames are baked into the client assets. The pose-phase is a 0..1 progress within the slot, so the client can render the slot mid-action with no extra round-trip.
- `call_timing_window` tells the audio layer when the bird's next call is scheduled. The call itself is synthesized client-side at that time using the per-species call grammar and the bird's current `visual_saturation` (which is the only personality-shaped rendering hint the snapshot exposes — it is read by the audio and visual layers both).
- `is_settled` is the soft-settle flag; when true, the lighting has shifted to evening and the client suppresses certain micro-motion (per `interactions.md`).

### 4.6 Visit-invitation flow (sequence)

1. Host submits `POST /v1/account/visits/invites` with the visitor's email. The invite row is created with a 30-day expiry, the magic-link token is generated, and the email is sent.
2. Visitor clicks the link, lands on `GET /v1/auth/magic_link/consume?token=…&purpose=visit&host_account_id=…`. The link is validated against `invite_token_hash`, marked consumed, and the visitor is signed in as themselves (creating a Pocket Aviary account if they don't have one — sign-up is frictionless; we do not gate the visit behind a sign-up wall).
3. The visitor's session is bound to the host's `account_id` for read-only snapshot access. A `visits` row is created.
4. The visitor pulls `/v1/aviary/snapshot` as a visitor. The response is the host's snapshot, with `is_visitor: true` and a `visit_id` echoed back. The visitor session cannot submit events to `/v1/events/*`. A request filter at the API edge rejects event submissions from visitor sessions with a matter-of-fact error.
5. On the next state-snapshot pull after host revocation, the visitor session is terminated. The visitor's client renders the matter-of-fact "visit no longer available" surface.

The visitor's snapshot is the host's snapshot, exactly. There is no special rendering, no prettification, no "host mode" that hides anything. What the visitor sees is what the host sees. This is the substantive point of the visit feature.

---

## 5. Simulation engine design

The simulation engine is the part of the system that turns a stream of events into a slowly-drifting aviary. It is a single server-side worker that runs a tick for every active account on a fixed cadence.

### 5.1 Tick loop

The tick is the architectural unit of "the aviary continues without the viewer." It is also the unit at which drift is computed and applied.

For each account that is `active` (not soft- or hard-deleted), on every tick:

1. Read unprocessed events from `event_log` for this account, in `(ts)` order. The set is bounded (events since the last successful tick).
2. For each event, apply the per-event-type transformation to compute a candidate delta. The transformation rules live in `simulation/transforms/` and are pure functions from `(event, current_state) → (delta, side_effects)`. Side effects are things like "schedule a notebook entry" or "schedule a mood transition."
3. Sum the deltas by trait, clamp to [0, 1], and apply to `birds.personality_vector`. Deltas are **additive**, never absolute. A client that tried to "set boldness to 0.62" cannot — the API doesn't accept absolute values, and the worker doesn't read absolute values from the event log.
4. Apply mood transitions: read the per-bird mood timer, advance it, and transition if the time-of-day, weather, or recent-interaction signals warrant. See §6.3.
5. Update perch positions: birds choose their perch based on mood, personality, and bird-to-bird spacing. See §6.4.
6. Schedule any new notebook entries that the tick qualifies for. See §6.6.
7. Bump `state_version` for any bird whose state changed.
8. Mark events processed in the worker's local state. The mark is durable enough for crash recovery but is not a write to the event log itself.

The tick is a single transaction per account. Personality-vector deltas from the same tick are applied atomically — either all of the tick's updates commit, or none do. Cross-account parallelism is fine; intra-account serialization is required.

### 5.2 The tick's pace

The tick runs at 60s ± calibration for every account. This is the right pace for two reasons:

- Fast enough that the mood transitions through morning/afternoon/evening/night feel continuous rather than stepped.
- Slow enough that a single tick's worth of drift is small. The instrument-level drift calibration (see §6.2) is built around the assumption that drift per tick is a small fraction of a typical trait's range.

The simulation worker's "active accounts" set is the union of (a) accounts with at least one unprocessed event in the last 24 hours, and (b) accounts that have at least one active client. Accounts that have been idle for 24 hours are dropped from the active set; their next tick fires on the next event or on the next client connect, whichever comes first. Idle accounts that re-activate (e.g., a client signs in after two weeks) get a single "catch-up" tick that processes the queued events all at once — this is safe because per-tick drift is small and additive.

### 5.3 Drift function (the spine)

Drift is implemented as a low-pass filter over a small set of input signals. The exact constants are calibrated at build, but the shape is fixed:

For each trait `T` of each bird, maintain a slowly-varying accumulator. On each tick, compute the per-tick delta as:

```
delta_T = (alpha_presence * presence_signal
         + alpha_listen_in * listen_in_signal
         + alpha_offer * offer_signal)
       * (1 - current_T)        // saturation: drift slows as trait nears 1
       * tick_dt_seconds         // 60s nominal
```

Where:
- `presence_signal` is a function of accumulated presence-time in the last 7 days, normalized to [0, 1]. The function is **monotonic** in presence-time but has diminishing returns past a few hours/day. A user who watches for 2 hours/day and a user who watches for 4 hours/day produce a small but not zero difference in drift velocity.
- `listen_in_signal` is the per-bird listen-in-time fraction, normalized.
- `offer_signal` is a per-event bump that decays exponentially over a few ticks; an accepted offer bumps curiosity, an offer-near-but-not-accepted bumps boldness.
- `alpha_presence`, `alpha_listen_in`, `alpha_offer` are per-trait weights, set in the calibration config.
- `(1 - current_T)` is the saturation term. It is the implementation of "drift is monotonic toward expressive but slows as the trait approaches its expressive ceiling." This is the rule that makes "feels alive over weeks" actually true: a bird with boldness 0.9 will drift more slowly than a bird with boldness 0.5 on the same presence input, which is the felt-shape of a relationship settling rather than escalating.
- `tick_dt_seconds` is the tick length. The whole drift function is normalized to be tick-rate-independent, so changing the tick cadence does not change the long-run drift velocity.

**Neglect does not push drift down.** The signal is `presence_signal`, `listen_in_signal`, `offer_signal`. None of them are negative. A user who is absent contributes zero to all three, which contributes zero to drift, which keeps the existing personality values stable. This is the asymmetric-drift model that is the load-bearing implementation of "no Tamagotchi." The implementation does not subtract, does not decay, does not penalize absence. A bird that gets ignored does not become more wary, more silent, or less colorful — it becomes ambient: still alive, still calling, but greeting less often because less often is what's been observed.

### 5.4 Calibration targets

The drift function is calibrated against two named targets:

- **Instrument-level drift after ~1 week** of regular visits (defined as 2–3 sessions/week, 15–30 minutes each, with listen-in and offers present but not dominant). A typical bird should show a measurable change in at least two of the five traits, detectable by an automated harness that reads the personality vector from the simulation database. This is the "we can test it" target.
- **User-visible drift after ~3 weeks** of the same usage. The user should be able to look back and notice that Pip is bolder than she was, or that Wren's plumage looks richer. This is the "the user can feel it" target.

The instrument-vs-user gap is intentional. We do not want users noticing changes session-by-session; we want them noticing changes when they look back. Drift velocity is tuned to the gap, not to either target in isolation.

A daily-test harness runs in CI: a synthetic account receives a synthetic event log shaped like 30 days of regular use, and the test asserts that the resulting personality vector deltas are within the calibration window for each trait. If the test fails, the calibration constants are reviewed; if the constants are correct, the drift function is reviewed. The test is the gate; it does not auto-tune.

### 5.5 Mood transitions

Mood is per-bird, fast-timescale. It is shaped by:

- **Time of day** in the user's timezone. Morning nudges toward alert; midday toward content; late afternoon toward curious; evening toward drowsy; night toward settled (eyes closed, low on perch) for most species, with the nightjar-like species remaining active and calling into the late hours.
- **Recent interactions** in the last few ticks. A successful offer nudges toward content; a refused offer nudges toward wary briefly.
- **Ambient events**. A passing rain dampens vocal frequency across the aviary for the duration of the rain; another bird's alarm call shifts nearby birds toward wary briefly.
- **Personality**. A high-boldness bird is less likely to enter wary on the same input that would push a low-boldness bird there.

Mood transitions are a small state machine per bird with weighted edges. The exact edges and weights are calibrated; the topology is fixed (no freeform "any state to any state" transitions).

The mood a bird has at session-end is the mood it has at the next session-start, modulated by whatever the server-side tick has computed in the interim. Birds do not reset to neutral on tab open. The simulation worker persists `current_mood` and the per-bird mood timer; the client's first snapshot is the current mood, not a "default" mood.

### 5.6 Perch selection

Perch position is not user-controlled. It is a function of:

- **Perch zone (front / middle / back)** — primarily driven by `current_mood` and `boldness`. A wary bird sits in the back; a content bird sits in the middle; a bold-and-content bird approaches the front. The mapping is a small lookup table with personality-shaped noise.
- **`perch_x` (horizontal position within a zone)** — driven by bird-to-bird spacing (birds repel slightly so they don't overlap) and per-bird individual variation (a stored "favorite perch x" that drifts slowly).

The user does not have a "send Pip to the front" affordance. Perch is a *signal* the user reads, not a layout the user controls. The simulation worker writes perch on every tick, the client renders it.

### 5.7 Bird-to-bird interaction

Birds interact with each other, not just with the user. This is what makes the aviary feel like a small social system. The simulation worker models:

- **Chorus events** — when two or more birds with high vocal frequency have call-timing windows that overlap, the audio layer on the client renders a chorus (slight pan separation, slight pitch shift per voice, slight timing jitter per voice). The simulation worker does not orchestrate the chorus; it provides the per-bird call-timing windows and lets the client mix them. This is what makes the chorus feel emergent rather than cued.
- **Wary contagion** — a wary bird's pose and call tone nudges nearby birds toward wary briefly. Spread is bounded (it doesn't take over the aviary) and decays in a few ticks.
- **Greeting chains** — when one bird greets on the user's return, a high-`social_warmth` bird nearby may respond. The worker schedules the response call; the staggering is on the order of 1–3 seconds, randomized.

### 5.8 Call grammar (runtime)

Calls are not audio assets. They are synthesized client-side from a per-species call grammar. The grammar is a small set of motifs, each motif a short parameterized shape (a few hundred ms, a handful of partials, a glissando envelope). At runtime:

1. The client receives `call_timing_window` for each bird in the snapshot.
2. When a window opens, the client asks the grammar for a call: it picks a motif (weighted by the bird's current mood), picks a pitch offset (personality-shaped), picks a duration (mood-shaped), and synthesizes the call via WebAudio.
3. The synthesis is shaped by `visual_saturation` from the snapshot — a higher-saturation bird produces a richer call (more overtones, slightly faster attack). This is the only personality-shaped rendering hint the audio layer receives.
4. The audio layer mixes all active calls into the chorus bus, applies the listen-in mix, and outputs.

The call grammar is bundled with the client. It is **not** downloaded at runtime. The grammar is small (a few hundred bytes per motif) and ships in the initial JS bundle. New grammars ship with new client versions; old clients continue to render the old grammar for the species they know. Species with old grammars in a client that has a newer grammar available degrade gracefully — the newer grammar is preferred, but the older one renders the call correctly.

The "no recorded audio" rule is unconditional. The call-grammar runtime is the only audio path; there is no fallback to recorded audio. When WebAudio is unavailable, the product plays in silence with captions on.

### 5.9 Notebook generation

The notebook is auto-generated, sparse, and read-only. The generator is a small templating system, not an LLM. Templates are species-aware, mood-aware, and time-of-day-aware. Examples of template patterns:

- `"{bird.name} greeted before {other_bird.name} today, first time this week."`
- `"{bird.name} is fluffed against the cool air, watching the back perch. low calls only."`
- `"a long stretch of quiet this morning. {bird.name} preened for several minutes without looking up."`
- `a {species_descriptor} perches on the {zone} branch, calling softly.`

Each entry is generated by selecting a small set of variables (which bird, what mood, what time, what recent noteworthy signal) and applying them to a template. The output is short (one to two sentences). The generator does not produce generic event-log text. A test asserts that the entry text matches a small set of structural patterns (lowercase first character, no exclamation, no second-person, no announcement framing). Entries that fail the pattern check are not written.

Cadence is enforced by the generator, not by the UI:

- A regularly-visited aviary gets roughly one entry every 2–4 days.
- A noteworthy event (a long quiet stretch, a new bird's first day, a chorus at dawn) can produce an entry sooner.
- A user who is constantly active does not get an entry per session. The cadence rule preserves sparsity.

The notebook never writes observations of the user's behavior. The generator cannot reference the user's visit frequency, presence patterns, or any aggregate about the user. The line between "observation of the aviary" and "observation of the user" is hard-coded in the generator's available variables, and a test asserts the boundary.

### 5.10 Empty-aviary and starter-bird state

A new account starts with the empty-aviary state for a brief moment (seconds) between sign-in completion and the first bird appearing. The first bird enters with a soft fly-in to its starting perch; the second follows after a short delay. The simulation worker pre-seeds two birds at account creation with personality vectors sampled from a small "starter" distribution and an initial mood of `content`. The user does not pick from a catalog — the system picks. The user names them at adoption and can rename later.

A third bird and beyond become available based on aviary age, not visit count. The schedule is:

- Months 0–2: two birds. No offers for a third.
- Months 2–6: third bird available.
- Months 6–12: fourth bird available.
- Year 1–2: fifth and sixth bird available at intervals.
- Year 2+: seventh bird available.

A new-bird offer appears in the user's flow as a soft prompt in the aviary scene, not as a notification. The user accepts or declines. The cap is seven and is enforced at the simulation worker level — the API rejects any request to adopt an eighth bird.

---

## 6. Frontend rendering pipeline

The renderer is a single SPA's main view, plus a small set of modal surfaces (settings, account, notebook detail, visit log). The aviary view itself is the only thing the user spends most of their time looking at, and is therefore the surface the rest of this section focuses on.

### 6.1 Scene composition

The scene is a single canvas rendered to fill the viewport (or the configured `aviary` region within it) at the device pixel ratio. Three layers are drawn back-to-front:

- **Background** — sky gradient, soft foliage silhouettes. Time-of-day palette, weather-driven if active. The background is the slowest-changing layer; it repaints only on time-of-day transitions and weather changes.
- **Midground (perches + birds)** — three perch zones (front, middle, back), each with a soft horizontal rail. Birds are positioned and posed according to the snapshot. This is the layer that animates per-frame.
- **Foreground (ornament)** — occasional foreground branches and leaves. A handful of these per scene, slowly drifting across the viewport. Drawn on top of birds (so a leaf can pass in front of a perched bird).

Parallax is **subtle**. The scene is not parallax-heavy. The foreground/background separation exists to give the scene depth, not to show off.

### 6.2 Bird rendering

Each bird is rendered as a small composite:

- **Silhouette** — a per-species SVG or compact bitmap, baked into the client.
- **Plumage palette** — applied to the silhouette as a tint, with `visual_saturation` from the snapshot shaping the palette richness.
- **Pose** — a pose-slot + pose-phase from the snapshot drives the in-pose animation. Pose-slot is one of a small set (`preening`, `scanning`, `idle`, `head_tilt`, `fluffed`, `settled`, `flight_transition`, `call`). The renderer interpolates between baked pose-frames within the slot, driven by `pose_phase` in [0, 1].
- **Idle micro-motion** — applied per-frame on top of the pose interpolation. This is the personality-shaped detail: a high-`curiosity` bird head-tilts more often; a high-`social_warmth` bird glances toward the other birds; a `drowsy` bird sits lower. The micro-motion is driven by a small per-trait noise function, sampled at render time, not by the simulation worker.

Birds are never still in a way that reads as paused. Even a "settled" bird at night has a small breath-cycle pose-phase advance.

### 6.3 Frame loop and the 60fps budget

The frame loop is `requestAnimationFrame` driven. The renderer targets 60fps on a five-year-old mid-range laptop. The frame work is:

1. Advance pose phases for all birds (pure compute, cheap).
2. Advance idle micro-motion noise (per-bird, cheap).
3. Advance ambient leaves / feathers (pure client-side, no server state).
4. Repaint only the midground layer when at least one bird pose has changed. The background and foreground are repainted at lower frequencies.

The renderer uses a dirty-rect strategy: the region around each bird is recomposited each frame, and the rest of the scene is blitted from a cached framebuffer. The cache invalidates on perch position changes (rare) and on time-of-day palette transitions (very rare). This keeps the per-frame cost bounded at O(birds_in_motion), not O(scene_pixels).

A "no memory growth over 30 minutes" rule is enforced by:

- Reusing audio buffers across calls (the procedural call grammar allocates one set of AudioBuffer nodes per call shape and recycles them).
- Reusing canvas framebuffers across frames (no per-frame allocation in the render path).
- Bounding the ambient leaf / feather pool to a small fixed count (e.g., 8 leaves, 4 feathers max in flight at any time). The pool is recycled; old leaves are not retained.
- Bounding the per-frame DOM mutation in the chrome (top bar icons do not re-render per frame).

### 6.4 Transitions

Most state changes in the aviary are continuous, not transitioned. A bird moving from one perch to another is a `flight_transition` pose-slot that the renderer animates from current position to target position with an easing curve. The easing is a slow ease-in-out, on the order of 1.5–3 seconds. Birds do not "teleport" between perches; the simulation worker's perch writes are the targets, the renderer interpolates.

The exception is a fresh sign-in: the first frame the user sees has birds mid-action. There is no "wake up" animation, no fade-from-static, no entry sequence. The client opens, pulls a snapshot, places birds at their current positions in their current motions, and starts rendering. The motion in the first frame is whatever motion the birds were in on the server when the snapshot was taken.

### 6.5 Loading state

When the state snapshot takes a beat to load (slow connection, cold cache), the loading state is a quiet field — soft sky color, perhaps one or two faint motion cues (a leaf drifting, a faint bird-silhouette in the distance) — not a spinner. The empty-aviary state, between adoption flow and the first bird appearing, uses the same quiet field; the first bird then enters with a soft fly-in to its starting perch.

### 6.6 Top bar chrome

A thin top bar sits above the aviary scene. It contains exactly four icons:

- Account / settings
- Accessibility settings
- Field notebook
- Offer affordance

The top bar fades nearly to transparent after a few seconds of cursor stillness. It returns to full opacity on cursor movement or keyboard activity. The fade is the small cost of putting controls anywhere on the surface.

There is no UI chrome inside the aviary scene. No buttons, no badges, no hover-tooltips, no overlay icons, no inline labels.

### 6.7 Reduced-motion mode

Reduced-motion is a designed surface, not a stripped fallback. When `prefers-reduced-motion` is set or the user opts in via accessibility settings, the renderer switches to:

- **Pose-slot rendering** as cross-fades between still poses. A bird in the `preening` slot cross-fades between preen-poses over a slow cadence, not animated frame-by-frame.
- **Flight transitions** as cross-fades between perches rather than animated paths. The bird's silhouette fades out at the source perch and fades in at the target perch, on a slow cadence.
- **Ambient leaf / feather drift** removed. The scene is stiller.
- **Time-of-day color shifts** remain, slowed. Day-to-evening transitions take 5–10 minutes rather than the usual smooth gradient.
- **Calls** still play at full quality (or caption, per audio settings). The audio is unaffected by reduced-motion.
- **Birds still drift, mood still changes, notebook still notices things.** Reduced-motion does not stop the simulation; it changes the visual register.

The reduced-motion mode is rendered by the same code path with a small set of "motion budget" parameters set to zero or near-zero. The pose-slot cross-fade is a renderer feature, not a separate scene. This keeps the two modes in lockstep — the simulation does not fork between motion and reduced-motion.

### 6.8 Modal surfaces

A small set of modal surfaces, all keyboard-navigable, all behind a focus trap when open:

- **Account settings** — sessions list, change-email, export, delete.
- **Accessibility settings** — reduced-motion toggle, captions toggle, narration toggle.
- **Field notebook** — list of entries, scroll-back, with timestamps. Read-only.
- **Visit log** — host's visits and outstanding invites.
- **Bird rename** — small inline modal on the bird's settings entry.

Modal surfaces are code-split (loaded on demand) to keep the initial bundle within the 2MB budget. The aviary view itself loads in the initial chunk; the modals are deferred.

---

## 7. Audio pipeline

### 7.1 WebAudio graph

The audio graph is a small set of nodes per call, mixed into a chorus bus, with the listen-in mix applied at the bus output.

```
call_voice_n → gain_n (per-voice) → chorus_bus → listen_in_mix → master_gain → destination
```

Each call voice is a per-call WebAudio graph constructed when a call starts and torn down when the call ends. The graph is built from a small palette of AudioNodes (oscillators, biquad filters, gain envelopes) parameterized by the call grammar's motif output.

The audio context is created lazily on the first user interaction that requires audio (browser autoplay policies require a user gesture). The context is shared across the session; calls are spawned into the existing context.

### 7.2 Procedural call synthesis

A call is a short (300–1200 ms) synthesized tone sequence. The call grammar is a small motif library (a few KB per species, all in the initial bundle) and a parameter-shaping function. A call is built by:

1. Picking a motif (weighted by current mood — `drowsy` picks low-energy motifs, `alert` picks sharper motifs).
2. Picking a pitch offset (shaped by the bird's `visual_saturation` and a per-bird individual variation).
3. Picking a duration (mood-shaped — `drowsy` calls are longer and more breathy, `alert` calls are shorter and sharper).
4. Building a graph: 1–3 oscillators (sine, triangle, or filtered noise), a biquad filter for tonal shaping, a gain envelope (attack–sustain–release), and optionally a subtle pitch glide.
5. Scheduling the graph against the audio context's clock at the time specified by the snapshot's `call_timing_window`.

The synthesis is lightweight enough that two simultaneous calls cost only a handful of nodes and a few hundred CPU-microseconds per call. The procedural grammar can comfortably support seven simultaneous calls on a five-year-old laptop.

### 7.3 Chorus mixing

When two or more calls are active at the same time, the chorus bus mixes them. The mixing is:

- **Slight pan separation** per call (random within a small range, so voices don't stack to a single phantom-center).
- **Per-call gain reduction** in the bus to avoid clipping when many calls overlap. The reduction is small (-2 to -6 dB at 7 calls), so the chorus does not become inaudible.
- **Slight pitch jitter per voice** at the synthesis level, so the chorus is not a phase-locked stack (the phase-canceling artifact of layered recorded loops is exactly what this avoids).
- **Per-call timing jitter** at the call start (a few ms) so calls do not align to a single beat. The simulation worker schedules call windows; the client adds the jitter at the audio scheduling step.

The chorus is emergent. The simulation worker does not orchestrate "these two birds should chorus at 3pm"; it provides per-bird call windows, and the client mixes what comes.

### 7.4 Listen-in mix

Listen-in is a slow re-balance of the audio bus, not a mute. When a bird is focused:

- The focused bird's per-call gain ramps up by +6 to +9 dB over ~1.5 seconds.
- The other birds' per-call gain ramps down by -6 to -9 dB over the same period.
- The ambient bus (calls off-stage) is unaffected.

The ramp is a linear gain ramp scheduled on the audio context's clock, not a JS-timer. The ramp is the same on engage and on disengage. The other birds drop in mix but never go silent — they are still audible, just quieter. Silencing them entirely would convert the aviary into a set of soloable tracks, which is a different audio surface.

### 7.5 Settle and the evening shift

When the user triggers settle, the audio mix softens over a few seconds — the master gain ramps down by ~6 dB, the chorus bus high-pass filter opens slightly so the high-frequency content falls off, and the calls become softer and farther. The shift is gentle and matches the visual lighting shift. The aviary remains in the settled state until the user re-engages; the audio mix does not return to normal automatically.

### 7.6 WebAudio fallback

If WebAudio is unavailable (older browser, audio context permission denied, hardware issue), the aviary plays in graceful silence with captions on by default. We do not ship a recorded-audio fallback path. The "no recorded audio" rule is unconditional — recorded audio at the variation the chorus mechanic requires would blow the bundle budget, and canned audio at lower variation would feel canned.

When WebAudio is unavailable:

- The caption toggle is on by default.
- A small unobtrusive indicator in the top bar (matter-of-fact voice in the chrome settings; "calls are off — show captions") tells the user audio is silent.
- The simulation worker continues to schedule call windows in the snapshot; the client just doesn't schedule the audio graphs.

The fallback is detected at session start via a `try { new AudioContext() }` probe. The result is sticky for the session.

### 7.7 Audio captions

Captions are short prose descriptions of the call, generated at call-synthesis time from the call grammar's parameters. The grammar emits a caption string alongside the audio graph build:

- `a soft three-note rise`
- `a low trill, paused, low trill again`
- `a single sharp call from the back perch`

The caption matches what was actually played (motif, pitch offset, duration), not a fixed string per motif. Captions appear as small text near the calling bird, fading in and out with the call. The voice is the same naturalist voice as the rest of the product.

---

## 8. Accessibility surfaces

Accessibility is a designed surface, not a checklist. The implementation honors the PRD's load-bearing rule: **the user gets the actual product, not a stripped-down fallback.**

### 8.1 Screen-reader narration

A running prose narration of the aviary state is exposed via a live region in the DOM. The narration cadence is slow — roughly one prose update per 30–60 seconds at idle, faster only on user-initiated events. The narration is generated from the same state the visual surface reads from, expressed in the same naturalist voice as the field notebook.

The narration generator runs client-side from the snapshot. It maintains a small "what to mention next" state machine:

- At idle, the narrator emits a sentence every 30–60 seconds describing the aviary's general state (time of day, which birds are visible, general mood).
- On a user-initiated event (a successful offer, a settle gesture, a return-greeting), the narrator emits a sentence promptly, but still as an observation, not as a state transition.

The narrator's output is bound to an `aria-live="polite"` region. A user who has set their screen reader to a faster rate hears the narration at their own pace; we do not force a reading rate.

### 8.2 Call captions

Captions are short prose descriptions of what each call sounds like, generated at call-synthesis time. They are visual text near the calling bird, in the same naturalist voice. Captions are useful for users with audio off, hearing differences, noisy environments, or any situation where the audio isn't getting through.

The caption text is generated from the procedural call grammar at runtime, not stored as a fixed string per call. Each call's caption matches what was actually played. Captions fade in and out with the call.

### 8.3 Reduced-motion mode

Reduced-motion mode is a designed surface (see §6.7). The user can opt in via the OS-level `prefers-reduced-motion` media query or via accessibility settings. The setting is persisted per account.

### 8.4 Keyboard navigation

All interactive surfaces are reachable by keyboard. The keyboard model:

- `Tab` moves through the top bar items in order.
- `Tab` past the last top bar item enters the aviary scene and focuses the first bird.
- Arrow keys move focus between birds in scene-space order.
- `Enter` triggers listen-in on the focused bird.
- `Escape` exits listen-in (or closes the focused modal, if a modal is open).
- `O` opens the offer affordance (or a top-bar shortcut of the designer's choice; the key binding is documented in accessibility settings).
- `S` triggers the settle gesture.
- `N` opens the field notebook.

Focus indicators are visible against the aviary background — a soft, high-contrast outline that reads against both bright and dim aviary states. The visual designer specifies the exact treatment; the implementation honors it.

### 8.5 WCAG AA contrast

All user-copy text passes WCAG AA contrast at minimum. This applies to:

- Top bar icon labels (when shown).
- Settings, account, error, and accessibility surfaces.
- Captions (when shown on top of the aviary scene — captions are rendered with a soft background plate to guarantee contrast against any aviary state).
- Narration when displayed visually (e.g., on a screen where the live region is rendered with a visible text style).

The aviary scene itself does not contain user copy, so the contrast constraint applies primarily to the chrome and to captions. The design system specifies actual ratios per surface.

### 8.6 Settings surface

The accessibility settings surface is a small modal with three toggles: reduced-motion, captions, narration. The toggles are simple, named, and persist per account. The surface is reachable from the top bar's accessibility icon and is the only place these settings live.

The accessibility settings surface uses the matter-of-fact voice (per the named exception), not the naturalist voice. A user opening accessibility settings needs to know what the toggles do, not to be charmed.

---

## 9. Performance budgets and observability

### 9.1 Performance budgets

- **Initial JS bundle <2MB (gzipped) at first paint.** The bundle includes the aviary view, the simulation client, the audio pipeline, the call grammars, and the species assets. Modals (account settings, accessibility settings, visit log, notebook detail) are code-split and loaded on demand. The 2MB cap drives several downstream choices: procedural audio synthesis (we cannot ship recorded audio at the required variation within this budget), procedurally-generated or compact bird assets, and aggressive code-splitting for less-frequent surfaces.
- **Time to first bird visible <500ms** on a mid-tier mobile device over 4G. Hitting this requires the bundle budget, fast initial state-snapshot delivery (a small payload served from the CDN edge with the HTML), and a render path that does not wait for non-critical assets before drawing the first bird. The 500ms threshold is the affective-perf bridge: above it, the user notices the load; below it, they don't, and the aviary feels like it was already running.
- **60fps idle motion** on a five-year-old mid-range laptop. The constraint applies to a 30-minute session, not just the first minute.
- **No memory growth over 30 minutes.** Enforced by a CI test that runs a synthetic 30-minute session in headless Chrome and asserts that the heap size is bounded.

### 9.2 Synthetic performance checks

A fleet of automated browsers runs the aviary on a schedule from common geographies. The fleet measures:

- Time to first bird visible.
- Time to interactive (when the top bar is responsive).
- 60fps frame timing over a 5-minute session.
- Memory growth over a 30-minute session.
- Audio-context creation latency.
- Simulation-tick latency (server-side, from the worker's perspective).

The fleet is internal; results are aggregated to a dashboard; alerts fire on regression. The fleet does not test on real accounts — it uses throwaway accounts created and torn down per run.

### 9.3 Aggregate-only Real User Monitoring

Real User Monitoring collects:

- Page load timings (anonymized, no per-account dimension).
- First-bird-render timings.
- Render-frame timings (1-in-N sampled, anonymized).
- Audio-context error counts.
- Simulation-tick latencies (server-side, no per-account dimension).
- API endpoint latency histograms (per endpoint, no per-account dimension).

None of this telemetry includes per-bird state, per-account interaction history, or anything that could be used to reconstruct a user's relationship with their aviary. The privacy boundary in `accounts_sync.md` is honored at the metric definition — fields that would be per-account are not defined as fields at all.

### 9.4 Error budgets

- **Simulation-tick latency p99 <5s.** An alarm fires if p99 exceeds 5s. The tick is supposed to take much less; an alarm at 5s catches degradation early, before users notice the aviary "running slow."
- **API endpoint error rate <0.5%** per endpoint per region per 5-minute bucket.
- **Snapshot delivery p95 <300ms** from edge to client.

### 9.5 What we deliberately do not measure

- Per-account presence-time distributions.
- Per-account drift velocity.
- Per-account offer acceptance rates.
- Per-account listen-in frequency.
- Per-account notebook entry generation rates.

These are the things that would constitute "observation of the user" in violation of the privacy commitment. The metrics are not defined as fields; the data pipeline does not have read access to the simulation database. The architectural absence is the durable defense.

---

## 10. Sync model

Multi-device sync is a property of the architecture, not a separate feature. This section restates the model in engineering terms so the implementation is unambiguous.

### 10.1 The model

- The server is the only source of canonical state. Personality vectors, mood, perch state, call timing windows, notebook entries, and bird-to-bird interaction effects live in the simulation database.
- The server is the only writer of personality state. The simulation worker is the only process that mutates `birds.personality_vector` and `birds.current_mood`. There is no code path on the API server that mutates these columns; the API server's database role lacks `UPDATE` permission on them, full stop. This is enforced at the IAM / role level, not at the code-review level.
- The client is the only producer of interaction events. Clients submit events to `event_log`; they do not submit absolute state values, and the API does not accept them.
- The simulation worker consumes `event_log` in `(account_id, ts)` order, applies additive deltas, and writes to canonical state. Re-processing on worker crash is idempotent at the trait level because each event has a fixed delta shape.
- A user signing in from a second device receives the same snapshot as their first device. The two clients show the same aviary, in the same mood, with the same drift history. They do not "sync" in any operational sense — they both read the same record.

### 10.2 The no-last-write-wins rule

The simulation worker writes personality state as **additive, server-authored deltas processed in event-log order**. A client never sends "set boldness to 0.62"; a client sends "user listened in to Pip for 3 minutes," and the server decides what that means for boldness.

A last-write-wins model on personality state would let one device overwrite drift recorded from a previous session on another device. Concretely: a laptop session writes a personality update based on its presence-time; a phone session that started before the laptop session ended writes its own version of the same vector based on its older read; the lunch write wins, the morning's drift is silently deleted. The user never sees the failure — they just have a bird that's drifting more slowly than it should — and there is no log entry that says "we lost data here." Additive server-authored deltas, processed in event-log order, make this failure mode unreachable.

### 10.3 Sync conflict surface

In rare cases (magic-link replay, an in-flight session timing out mid-write, a server-side outage), the user may encounter a sync conflict surface. The API returns a small set of well-known error responses with matter-of-fact copy:

- `409 stale_state` — the client's `state_version` is behind the server's. The client re-pulls the snapshot and re-renders. This is the most common case and is not user-visible unless the user's local copy is so stale that the diff is large.
- `409 session_conflict` — two sessions for the same account on the same browser; the older one is invalidated.
- `410 account_in_deletion` — the account has been soft-deleted. The client shows the matter-of-fact "your account is scheduled for deletion" surface and offers the cancel-deletion path.
- `503 service_unavailable` — the origin is unhealthy. The client shows the matter-of-fact "something went wrong" surface with a reload affordance.

All of these use the matter-of-fact voice. Naturalist phrasing in an error context reads as evasive. The named exception applies.

### 10.4 Visitor session termination

When a host revokes an active visit, the visitor's session is terminated at the next state-snapshot pull. The visitor's client renders the matter-of-fact "visit no longer available" surface. The termination is implemented by tagging the visitor's session as revoked at the API edge; the snapshot endpoint returns 410 with the matter-of-fact body to revoked sessions.

### 10.5 What we don't do

- No client-to-client sync. There is nothing to sync.
- No eventual consistency to reconcile. Both clients read the same record.
- No offline-first. The product does not have an offline mode. The simulation is server-side; there is no local simulation to fall back to. A user without network sees the matter-of-fact "no connection" surface, not a stale aviary.
- No PWA / service worker caching of state. The client may cache static assets (HTML, JS, CSS, species assets) at the service-worker layer for fast loads, but state is always server-fresh.

---

## 11. Rollout

### 11.1 Ship v1

V1 is a single, fully-featured release. There is no v1 "soft launch" that ships without one of the v1 features. The constraint is the product's affective integrity — shipping without, say, the field notebook or the visit feature in the first release would force a "v1.1" add that re-opens design decisions and is the kind of place that a Streak Counter quietly slips in. The v1 release ships every feature listed in §1.1.

The rollout is staged:

1. **Internal alpha** — the team uses the product on their own accounts. Synthetic accounts are used for the calibration harness. This phase is for the calibration pass (see §11.3).
2. **Closed beta** — a small group of external users (target: 50–200), invited by the team. The closed beta is for the calibration pass to reach instrument-level drift targets on real usage, and for the load profile to be characterized.
3. **Public launch** — open sign-up. The release is the same build as the closed beta; the closed beta's data is preserved (the closed beta was the v1 build).

### 11.2 Ramp the bird count

The bird count starts at two and ramps with aviary age. The schedule in §5.10 is the v1 schedule. The product does not surface this ramp to the user as "level up" or "tier" — the third bird appears as a soft prompt in the aviary scene when the aviary is old enough, not as a "Congratulations, your aviary is now eligible for a third bird" notification. The soft prompt is a small affordance in the scene; the user accepts or declines.

### 11.3 Day-one instrumentation

From day one, the product is instrumented for:

- Drift velocity (instrument-level): the calibration harness tracks per-trait drift over a 30-day synthetic account. If drift velocity drifts outside the calibration window, the calibration constants are reviewed.
- Mood transition distribution: per-mood-pair transition counts over time, to detect the state machine getting stuck in a state or transitioning too frequently.
- Notebook entry cadence: entries per aviary per day, to ensure sparsity is preserved.
- API latency and error rates per endpoint.
- Simulation-tick latency and queue depth.
- Synthetic performance checks (see §9.2).
- Aggregate-only RUM (see §9.3).

Day-one instrumentation does **not** include per-account, per-bird, or per-event metrics. The data pipeline does not have read access to the simulation database.

### 11.4 Post-launch monitoring

The team monitors:

- Drift calibration on the closed beta cohort. If real-usage drift velocity differs from the synthetic calibration by more than a small factor, the calibration is reviewed and adjusted.
- Audio uncanniness reports. The procedural call grammar is the affective spine; any "calls sound off" report is investigated at the synthesis level. The 7-bird chorus is the load-bearing edge case.
- Accessibility regressions. The accessibility surfaces are tested in CI (screen reader smoke test, reduced-motion visual test, contrast test). Any regression is a P0.
- Sync correctness. The no-last-write-wins rule is tested in CI. Any test failure is a P0.

---

## 12. Risks

The risks below are the things most likely to go wrong in a way that would compromise the product's load-bearing claims. Each is named, the failure mode is described, and the mitigation is concrete.

### 12.1 Drift calibration drift

**Risk.** The drift function is calibrated against a synthetic event log. Real users' presence patterns differ from the calibration pattern, and the actual drift velocity on real accounts may differ from the design target. The "feels alive over weeks" promise requires that drift be neither too fast (Tamagotchi) nor too slow (screensaver), and the calibration window is narrow.

**Mitigation.**

- The synthetic-event-log test in CI asserts that drift on a 30-day synthetic account is within the calibration window for each trait. The test is the gate; it does not auto-tune.
- The closed beta cohort's drift velocity is measured via the calibration harness, not via per-account drift metrics (the latter would violate the privacy boundary).
- Drift calibration constants live in a single config file, reviewed by the team on every change, with a documented rationale for each constant.

### 12.2 Sync correctness

**Risk.** A code path that lets a client (or the API server on behalf of a client) mutate personality state directly would re-introduce the last-write-wins failure mode. The failure is silent — no test would catch a single lost drift update, and the user-visible symptom ("my bird is drifting more slowly than it should") is hard to attribute.

**Mitigation.**

- The API server's database role lacks `UPDATE` permission on `birds.personality_vector` and `birds.current_mood`. The mitigation is at the IAM / role level, not at the code-review level. A misconfigured role is caught by an integration test that asserts the API server cannot mutate personality state, even via raw SQL.
- An end-to-end test in CI simulates two clients on the same account, each producing events, and asserts that the resulting personality vector is the same as a single-client simulation producing the same events in the same order.
- The `state_version` field on every bird is monotonically increasing and is asserted to never decrease. A decrease is a P0.
- The event log is append-only. A `DELETE` or `UPDATE` on `event_log` from the API server's role is rejected by the database role. A `DELETE` is permitted only from the hard-deletion job, which runs as a different role.

### 12.3 Audio uncanniness

**Risk.** Procedural audio is the affective spine. The procedural call grammar can produce calls that sound unnatural, mechanical, or "off" in ways that break the felt-aliveness. The 7-bird chorus is the load-bearing edge case — seven procedural calls must mix into a chorus that sounds emergent, not stacked.

**Mitigation.**

- The call grammar is reviewed by ear at every change. The CI does not test audio quality (it cannot); the review is human.
- The chorus-mix parameters (pan separation, gain reduction, pitch jitter, timing jitter) are tuned against a small set of recorded 7-bird choruses and reviewed on every change.
- A "calls sound off" user report is a P0 investigation. The investigation includes: which call, which bird, which client, which time-of-day, what the call's grammar parameters were.
- The nightjar-like species in the species pool is a special case (it calls at night, when the other species are settled). The nightjar's call grammar is the highest-risk single component of the audio pipeline, and is reviewed against a small set of late-evening listening sessions.

### 12.4 Accessibility regressions

**Risk.** A change to the visual or audio pipeline that ships without an accessibility regression test can quietly break the screen-reader narration, the captions, the reduced-motion mode, or the contrast. The risk is amplified by the "designed surface, not fallback" rule: a stripped-down accessibility surface would land without anyone noticing the regression, because the test would pass on the stripped surface.

**Mitigation.**

- The accessibility surfaces are tested in CI: a screen reader smoke test (using a headless browser with a screen reader bridge), a reduced-motion visual test (a screenshot diff at the reduced-motion register), a contrast test (a static check on the design tokens), and a keyboard-navigation test (a scripted Tab-through of the aviary surface).
- The accessibility work is on the same critical path as the rest of the product. A reduced-motion mode that lands as a v1.1 fix is a v1 launch that quietly told reduced-motion users the product wasn't for them. The accessibility surfaces are not "post-launch" work.
- A change to the visual or audio pipeline requires a paired accessibility review. The review is a checklist plus a 10-minute listening / viewing session on the affected surfaces.

### 12.5 Bundle budget regressions

**Risk.** The 2MB initial JS bundle budget is the budget that makes the time-to-first-bird budget achievable. A feature that adds JS to the initial bundle (rather than to a code-split chunk) can quietly blow the budget and degrade the felt-aliveness of the product on slow connections.

**Mitigation.**

- A CI check on the initial bundle size. The check fails the build if the gzipped initial bundle exceeds 2MB. The check is on the deployed artifact, not on the source.
- Modals and less-frequent surfaces (account settings, accessibility settings, visit log, notebook detail, the visit-invitation flow) are code-split by default. A new surface that is added to the initial bundle requires a justification review.
- A feature-flag system allows the team to ship features that affect bundle size to a small percentage of users before they ship to all users. Flags are removed once the feature is generally available.

### 12.6 Notebook voice drift

**Risk.** The notebook's voice is the most concentrated surface of the product's personality. A template that lands as a generic event log entry ("a bird greeted you," "session started at 7:43") breaks the spell across the entire product, not just the notebook. A new template added by a well-meaning contributor can quietly drift toward gamification language.

**Mitigation.**

- The notebook generator's templates are reviewed in PR. The review includes a voice check: lowercase first character, no exclamation, no second-person, no announcement framing, no "you've been here every day this week."
- A CI test asserts that the generator's template variables do not include the user's presence-time, visit frequency, or any other observation of the user's behavior. A template that references such a variable fails the build.
- A weekly audit samples a small number of generated notebook entries and reviews them by eye. Entries that read as event-log text are removed from the generator.

### 12.7 Identity continuity

**Risk.** A migration, a seed-data change, a refactor, or a re-seed of the species pool could effectively "replace" a bird. The user would not be told — the bird would have the same name and species, but a different personality vector, a different drift history. The user would feel something was wrong without being able to name it.

**Mitigation.**

- The `bird_id` is a UUID, generated at creation, never reused, never reassigned. The species pool's `species_id` is a smallint and can change; the bird's `bird_id` cannot.
- A migration that touches the `birds` table is required to assert that the `bird_id` is preserved for every row. The migration is reviewed by two engineers.
- The user can export their aviary state (see §4.4) at any time. The export includes the `bird_id` so the user can verify identity continuity themselves.

### 12.8 "Just one streak counter" temptation

**Risk.** This is not a technical risk; it is an organizational risk. The temptation to add a "harmless" engagement feature is the most predictable failure mode of the product, and every reasonable-looking pitch to relax the rule will land. The rule has to survive every pitch.

**Mitigation.**

- The non-goals in `non_goals.md` are loud and absolute. They are restated in this plan, and they are restated in the README of the simulation worker, the README of the client, and the team onboarding doc.
- A PR that adds a streak counter, a days-visited counter, a green-dot calendar, a "you've been here every day this week" surface, a public profile, a leaderboard, an achievement, a tier, or any other gamification surface is rejected at PR review, regardless of the implementation cost. There is no version of any of these that is allowed in the product.
- A quarterly review of the product surface, performed by the team, against the non-goals. The review produces a one-page document listing the surfaces that were added in the quarter, and a per-surface judgment against the non-goals. Surfaces that touch a non-goal are removed.

---

## 13. Open questions resolved

These are the places where a careful reader of the PRD would notice an unspecified detail. The plan's defensible calls (§1.3) cover the substantive ones; this section covers the smaller ones so the engineering team is not re-deciding them.

- **Species pool composition.** Six species, one of which is nightjar-like. The other five are coherent with the v1 visual palette (soft blues, greens, warm browns, muted ochres). The species are static for v1; we may add more later, but the species count and the nightjar role are not changed in v1.
- **Starter-bird assignment.** The two starter birds are sampled from the species pool without replacement, so the user gets two distinct species. The user does not pick.
- **New-bird introduction animation.** Soft fly-in to a perch, with the new bird's initial mood of `curious`. The simulation worker pre-seeds the new bird's personality vector sampled from the "starter" distribution.
- **Naming defaults.** The user is offered a small set of name suggestions per starter bird (drawn from a small curated list of short, bird-name-friendly names). The user can type any name. The name is single-line text, max 24 characters.
- **Notebook revision history.** The notebook is read-only to the user. If a notebook entry is edited by the server (rare; only to fix a generated-text issue that the audit caught), the entry's `notebook_revision` increments. The UI shows a small "edited at" marker on entries that have been edited. The marker is in the matter-of-fact voice ("this entry was edited on …").
- **Visit notifications on the host's side.** Off by default. The user can opt in via account settings. When opted in, the host gets an email when a visit starts. The email is in the matter-of-fact voice.
- **Account export format.** JSON. The export includes bird records (without personality vectors visible to the user — the vectors are included in the export because the export is the user's own data, but the format is structured so the user can see what is being stored about their birds). The export is generated on demand and emailed as a one-time download link, valid for 24 hours.
- **Account deletion cancellation window.** 30 days from the `deletion_requested_at` timestamp. The cancel button is on every signed-in page during the window. The cancel is one click; we do not require a confirmation dialog for cancel.
- **Session token rotation.** Session tokens are rotated every 30 days. The rotation is silent — the user does not see a re-auth surface unless the rotation fails.
- **Browser support floor.** Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers receive a matter-of-fact unsupported-browser surface explaining what's needed.

---

## 14. What the team will build first

For orientation only — not a project plan, just a list of the surfaces that need to exist in roughly this order for the system to be coherent:

1. The simulation database, the event log, and the simulation worker. Without these, no other surface has state to read.
2. The auth flow and the account record. Without these, no event has an `account_id`.
3. The snapshot endpoint and the client renderer. Without these, no event has anywhere to land visually.
4. The call grammar and the audio pipeline. Without these, the aviary is silent.
5. The drift function. The simulation worker can produce mood transitions and perch selection without drift; drift is added once the basic engine is honest.
6. The field notebook generator. The notebook is a small surface, but it is the voice of the product; it lands after the engine is in place.
7. The interactions: listen-in, offer, settle. These are client-side state, but they need the audio pipeline (for listen-in) and the simulation worker (for offer acceptance).
8. The accessibility surfaces: narration, captions, reduced-motion, keyboard navigation. These are designed alongside the client, not after.
9. The visit flow. The visit is a small surface area on the API and a read-only client mode; it lands after the snapshot endpoint exists.
10. The account export, the account deletion, the session list, the visit log. These are CRUD surfaces; they land after the account record exists.

---

## 15. Summary of load-bearing decisions

For the engineering team's awareness — these are the decisions in this plan that, if changed, would compromise a load-bearing claim of the PRD:

- **The server is the only writer of personality state.** If this changes, sync collapses and the no-last-write-wins rule is unreachable.
- **Drift is additive, server-authored, monotonic toward expressive, never downward on neglect.** If this changes, the product becomes a Tamagotchi.
- **Calls are procedural; recorded audio is forbidden.** If this changes, the chorus mechanic collapses and the bundle budget is unrecoverable.
- **The user never sees personality vector values.** If this changes, the bird becomes a number and the relationship collapses.
- **The notebook is read-only and sparse.** If this changes, the notebook becomes a feed and the voice is diluted.
- **No UI chrome inside the aviary scene; the top bar fades; no toasts, no banners, no welcome-back text.** If this changes, the product announces instead of notices, and the central conceit fails.
- **The simulation tick runs server-side and is the source of canonical state.** If this changes, the aviary is no longer "continuing without the viewer."
- **The synthetic UUID rule for accounts.** If this changes, PII leaks into logs and aggregates, and the privacy commitment is broken at the source.
- **The privacy boundary at the data-pipeline level: telemetry pipelines do not have read access to the simulation database.** If this changes, the privacy commitment is broken at the architectural level.
- **Accessibility is a designed surface, not a stripped fallback.** If this changes, the product rations its actual quality by sensory ability, which is a worse failure than missing a contrast threshold.

The team is not asked to defend these decisions in PR review; the PRD has already defended them. The team's job is to implement them faithfully, and to push back loudly if a future change request would compromise one of them.

---

*End of plan. Total file write: this single `PLAN.md` and a paired `CANDIDATE_METADATA.json`. No other files were written in this slot.*
