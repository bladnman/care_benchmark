# Pocket Aviary — v1 Implementation Plan

This is an executable engineering plan for shipping Pocket Aviary v1. It interprets the PRD into architecture, data models, APIs, simulation, sync, rendering, audio, accessibility, performance, rollout, and risks. The product's affective core is load-bearing; this plan keeps the procedural-call rule, the server-only personality writer rule, the asymmetric drift rule, the no-numeric-personality rule, and the "notice, never announce" rule visible at every layer where they bind.

When the PRD is silent, this plan makes a defensible call and flags it inline as `[Decision: …]`.

---

## 1. Scope

### In scope for v1

- Single-user, single-aviary accounts. Email + magic-link sign-in. Per-device session tokens.
- Browser-only delivery (last two majors of Chrome, Safari, Firefox, Edge).
- Two starter birds at adoption; cap of seven; new-bird offers paced by aviary age.
- ~6 species in pool, each with silhouette, default plumage palette, and call-grammar motif library.
- Server-authoritative simulation tick (~1/min) with personality drift, mood transitions, day/night, ambient weather, bird-to-bird interaction.
- Client interactions: return-greeting (server-driven, client-rendered), idle attention/presence, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), browsing field notebook.
- Multi-device sync via canonical server state (snapshots + interaction event log; no client-authored personality writes).
- Field notebook: server-generated, sparse, naturalist prose, read-only.
- Visit (read-only): per-invite opt-in, revocable, 30-day invitation expiration, off-by-default visit notifications, visit log.
- Accessibility as designed surfaces: screen-reader narration in naturalist voice, reduced-motion mode (cross-fade aesthetic, not a fallback), call captions in naturalist voice, full keyboard navigation, WCAG AA contrast.
- Performance budgets: <2 MB initial JS gzipped, <500 ms time-to-first-bird on mid-tier mobile / 4G, 60fps idle on a 5-year-old laptop, no memory growth over 30 minutes, procedural audio synthesized client-side, silence-with-captions WebAudio fallback.
- Account: settings, export to JSON (emailed link), soft-delete with 30-day recovery then hard delete, email change with verification.
- Aggregate-only operational telemetry; per-bird interaction state never enters the analytics warehouse; no ML on per-bird data.

### Explicitly out of scope (per `non_goals.md`)

- Native iOS / Android apps.
- Any gamification: streaks, achievements, levels, scores, badges, "birds adopted: N", green-dot visit calendars, XP, ranks, tiers, "you've been here every day this week" surfaces, milestone celebrations.
- Tamagotchi-style mechanics: hunger, distress, decay, death, happiness meter, negative drift on neglect.
- Social-network surfaces: profiles, follows, public feed, discovery, leaderboards, comments, friend-of-friend, co-presence in visits, chat, visitor avatars, "show-off" rendering for visitors.
- Push / email notifications about the aviary (the aviary lives where the user visits it). The single per-account, off-by-default visit-notification toggle is the named exception.
- Client-side simulation tick of any kind. Clients render snapshots; clients never write personality.
- Welcome toasts, banners, modals, or any textual greeting on return.
- Any UI surface that exposes personality vector numerics.
- Recorded-audio fallback for calls.

### Non-goal protections that bind to the codebase, not just the PR description

- No `streak`, `xp`, `level`, `score`, `achievement`, `badge` field anywhere in the data model. CI lint blocks new fields with these names.
- No telemetry pipeline path that joins per-bird state with population aggregates. Enforced by separate database for simulation vs. analytics, with no read credentials between them.
- No client mutator method on personality fields. Type system marks `PersonalityVector` as `readonly` on the client; only the simulation service has a writer.

---

## 2. Architecture

### 2.1 Service shape

A small set of services. Names are functional; the operational topology can collapse them where it makes sense.

1. **edge / web app** — Static HTML + JS bundle delivered from a CDN. Renders the aviary scene, runs WebAudio synthesis, captures interaction events, presents accessibility surfaces.
2. **api gateway** — Public HTTPS surface. Auth, snapshot reads, event-log appends, account/settings, visit invitations and consumption, account export, soft-delete. Stateless; horizontally scaled.
3. **auth service** — Magic-link issuance and consumption, session token issue/revoke. Owns the email-to-account-uuid mapping. Encrypted email at rest.
4. **simulation service** — Owns the canonical aviary state. Runs the server-side tick on a slow schedule (~1/min per active aviary; every aviary, regardless of any client connection). Reads the interaction event log. Writes personality vectors, moods, positions, current call timing, day/night state, ambient weather. The only writer of personality state, anywhere.
5. **notebook service** — Generates field-notebook entries from simulation state and event log on its own slow cadence (sparsity-tuned). Read-only to the user.
6. **narration service** — Generates screen-reader narration prose from simulation snapshots in the same naturalist voice as the notebook. Runs on a 30–60s cadence at idle, with priority bumps for user-initiated events.
7. **export / deletion worker** — Builds JSON exports on demand and emails the download link; performs hard-delete sweeps after the 30-day soft-delete window.
8. **visit service** — Issues / revokes / expires invitations; resolves visitor sessions to read-only snapshots of the host's aviary; records visit log entries on the host's account.
9. **email service** — Outbound transactional email only (magic links, export-ready notifications, optional per-host visit notifications when the host has explicitly opted in). No marketing, no engagement nudges.

The CDN-edge tier is also responsible for delivering the **first state snapshot** colocated with the HTML response (see §10.2) so the time-to-first-bird budget is reachable.

### 2.2 Client / server split

The split is ideological as well as technical:

- **Server owns**: identity, personality vectors, mood, positions, calls in flight, day/night, weather, presence accounting, drift function, notebook prose, narration prose, visit invitations, the canonical clock.
- **Client owns**: rendering, audio synthesis, interpolation between snapshots, idle ambient ornaments (leaf drift), capturing input events, optimistic visual confirmation of user-initiated actions (e.g. settle lighting begins to shift immediately while the event is in flight), accessibility surfaces (rendered from server-supplied prose).
- **Client never owns**: any field that affects future drift. Idle leaf drift is rendering ornament, not state.

The render pipeline boundary is: snapshots cross the boundary as small JSON payloads; nothing else does. The client does not run the drift function as a "preview"; the client does not project mood transitions; the client does not predict call grammar. The client interpolates positions and renders calls from the server's call-event stream, nothing more.

### 2.3 Data flow at runtime

1. Browser navigates to `/`.
2. Edge serves HTML + bundle. The HTML response embeds an initial snapshot for the user's aviary (signed, short-TTL) so the first-bird path doesn't need an extra round trip. For unauthenticated requests, the HTML embeds a sign-in landing.
3. Bundle hydrates. It renders the aviary at the snapshot's "now," begins WebAudio synthesis, kicks off a single keepalive WebSocket (or long-poll fallback) to stream snapshot deltas and call events.
4. Client emits interaction events to the api gateway as they occur. They are appended to a per-account event log.
5. Simulation service ticks every aviary on its schedule, drains the event log up to the tick boundary, recomputes deltas, writes new canonical state, and emits a new snapshot to any active client connection.
6. On tab hide, the client closes the WebSocket and stops rendering; the simulation tick continues regardless. On tab visibility return, the client opens a fresh connection and pulls a fresh snapshot.

### 2.4 Storage

- **Primary simulation DB** (Postgres). Stores accounts, birds, personality vectors, current mood, current position, mood timers, call timing, ambient weather state, day/night state cache, notebook entries, visit invitations, visit log, account export jobs, soft-deletion markers. Per-account schema isolation is logical (account UUID column on every row) with row-level access enforcement at the gateway.
- **Event log** (append-only). Per-account interaction events. Could be a partitioned table in Postgres or a Kafka topic; [Decision: start as a partitioned Postgres table keyed on account UUID, then move to Kafka if simulation throughput requires it].
- **Auth DB** (Postgres). Email-to-account mapping (email encrypted at rest with envelope encryption; the encryption key is in a managed KMS). Session tokens, magic-link nonces.
- **Object storage**. Account export JSON blobs (signed, expiring URLs).
- **Aggregate analytics warehouse** (BigQuery / equivalent). Operational metrics only — request counts, latencies, error rates, anonymized session-duration histograms. No per-bird, per-account interaction state. The simulation DB is **not connected** to the warehouse; this is a network and credentials boundary, not just a policy.

### 2.5 Identity

The synthetic UUID rule is non-negotiable. Account UUID is the only identifier used in:

- Database partition / shard keys.
- Inter-service messages.
- Logs and traces.
- Telemetry events.
- Object storage keys.
- WebSocket session identifiers.

Email lives in exactly one place: `auth.email` (encrypted). Anywhere else that needs to act on behalf of the user, the UUID is used. Any new code path that takes an email parameter for identification (rather than for delivery) is a regression and must be rejected at review.

---

## 3. Data model

All identifiers are UUIDv4 unless otherwise noted. All timestamps are UTC.

### 3.1 Account

```
accounts
  id                uuid primary key                    # synthetic UUID, the only ID used elsewhere
  email_encrypted   bytea                               # KMS-wrapped DEK + ciphertext
  email_hash        bytea unique                        # for unique-email constraint without storing plaintext
  created_at        timestamptz
  email_change_pending_encrypted    bytea null
  email_change_token_hash           bytea null
  email_change_expires_at           timestamptz null
  delete_requested_at               timestamptz null    # null = active; else 30-day soft-delete window
  visit_notifications_enabled       boolean default false
  reduced_motion_pref               text                # 'system' | 'on' | 'off'
  captions_enabled                  boolean default false
  audio_muted                       boolean default false
  preferred_locale                  text                # for time-of-day and prose locale
  timezone_iana                     text                # canonical for day/night cycle
  privacy_policy_version_accepted   text
```

### 3.2 Sessions

```
sessions
  id              uuid primary key
  account_id      uuid references accounts(id)
  device_label    text                   # client-derived UA + first-seen city; user-revocable
  created_at      timestamptz
  last_seen_at    timestamptz
  revoked_at      timestamptz null
  token_hash      bytea unique
```

### 3.3 Birds and personality

```
birds
  id                  uuid primary key                # stable, never reassigned
  account_id          uuid references accounts(id)
  species_code        text                            # references species_pool
  display_name        text                            # user-settable
  adopted_at          timestamptz
  retired_at          timestamptz null                # we don't expect to use this in v1; reserved for future
  current_perch       text                            # 'front' | 'middle' | 'back'
  current_mood        text                            # enum, see below
  mood_entered_at     timestamptz
  position_x_norm     float                           # 0..1 within the perch zone
  facing              text                            # 'left' | 'right'
  last_call_at        timestamptz null
  next_call_eta       timestamptz null

personality_vectors
  bird_id             uuid primary key references birds(id)
  boldness            float
  social_warmth       float
  vocal_frequency     float
  plumage_saturation  float
  curiosity           float
  updated_at          timestamptz
  -- no client-side accessor in the API; never serialized to the client.
```

Personality scalars are normalized to `[0.0, 1.0]` internally. They are never serialized to clients (see §4 — the API simply does not expose them). Plumage saturation is the one trait whose effect is visible directly (it modulates rendered color richness); the simulation service derives a `plumage_render_token` from `plumage_saturation` at snapshot time, and only that derived token crosses the boundary, never the float.

Mood enum (v1): `wary`, `content`, `curious`, `drowsy`, `alert`. The set is finalized here so the rendering and narration paths can build against it; widening it is a cross-cutting change and is treated as such.

### 3.4 Interaction event log

```
interaction_events
  id              bigserial primary key
  account_id      uuid not null
  bird_id         uuid null              # null for aviary-wide events (settle, weather observation, etc.)
  event_type      text not null          # see enum
  occurred_at     timestamptz not null
  payload_json    jsonb                  # type-shaped per event; small
  client_ts       timestamptz            # for ordering hints; not authoritative
  ingested_at     timestamptz default now()
```

Event types:

- `presence_ping` — emitted by the client at a calibrated cadence whenever (visibility=visible) AND (window has focus) AND (a pointermove or keypress occurred within the activity window). Carries no other data.
- `listen_in_start`, `listen_in_end` — bird_id required.
- `offer_initiated` — payload: `{ kind: 'seed' | 'song_fragment' | 'still_pool', target_bird_id?: uuid }`. Server may reject if cooldown active.
- `offer_accepted`, `offer_ignored` — server-side, written by the simulation when the receiving bird's reaction resolves.
- `settle_initiated`, `settle_undone` — aviary-wide.
- `bird_renamed` — `{ bird_id, new_display_name }`.
- `bird_adopted` — `{ bird_id, species_code, display_name }`.
- `visit_started`, `visit_ended` — written by the visit service for the host's log.

The log is append-only and never edited. The simulation service consumes events strictly in `(occurred_at, id)` order. It commits its drift / mood writes within a transaction that also marks events as processed up to a watermark.

### 3.5 Aviary-level state

```
aviaries
  account_id              uuid primary key references accounts(id)
  weather_state           text                 # 'clear' | 'rain' | 'wind'
  weather_started_at      timestamptz null
  weather_planned_until   timestamptz null
  daynight_phase          text                 # 'dawn' | 'morning' | 'midday' | 'afternoon' | 'evening' | 'night'
  daynight_t              float                # 0..1 within phase, derived from local time
  settled                 boolean default false
  settled_at              timestamptz null
  last_tick_at            timestamptz
```

`daynight_phase` is derived from the account's `timezone_iana` at tick time. The server is the source of truth for which phase the aviary is in, even though the cycle is mathematically derivable from local time; storing it makes server-driven transitions straightforward (e.g. an evening-bound mood shift fires when the phase crosses to `evening`).

### 3.6 Notebook entries

```
notebook_entries
  id              uuid primary key
  account_id      uuid references accounts(id)
  written_at      timestamptz                 # in the user's local timezone, for "tuesday — …" prose
  prose           text                        # the entry, naturalist voice
  cause_summary   jsonb                       # internal: which signals prompted the entry; not exposed
```

`cause_summary` is internal, used to rate-limit entries on the same kind of event and to debug entry generation. It is never exposed to the client and is not part of the export.

### 3.7 Visits

```
visit_invitations
  id                    uuid primary key       # the public invite code is derived from this id
  host_account_id       uuid references accounts(id)
  visitor_email_hash    bytea                  # we never store the plaintext after sending the invite email; the hash lets us match on visit
  visitor_email_encrypted_for_log  bytea       # for the host's visit log display; decryptable on demand
  created_at            timestamptz
  expires_at            timestamptz            # +30 days
  revoked_at            timestamptz null
  used_at               timestamptz null       # first consumption; subsequent visits reuse the resulting visit session

visit_sessions
  id                  uuid primary key
  invitation_id       uuid references visit_invitations(id)
  started_at          timestamptz
  ended_at            timestamptz null
  ended_reason        text null               # 'visitor_left' | 'revoked' | 'expired' | 'idle_timeout'

visit_log_entries
  -- materialized view over visit_sessions for the host's settings UI
```

### 3.8 Account export jobs

```
export_jobs
  id              uuid primary key
  account_id      uuid references accounts(id)
  requested_at    timestamptz
  completed_at    timestamptz null
  blob_key        text null                   # object-storage key
  link_emailed_at timestamptz null
  link_expires_at timestamptz null
```

### 3.9 Telemetry boundary as a schema property

The simulation DB and the analytics DB are different physical databases with different IAM identities and different network policies. There is **no service** with credentials to both. The analytics DB schema does not contain any column that can be joined back to `accounts.id`; session-duration histograms are aggregated server-side to bucket counts before they cross into the warehouse.

---

## 4. API surface

All endpoints are HTTPS. Bearer auth is the per-device session token, carried as `Authorization: Bearer …`. The api gateway enforces account-row scoping on every read and write.

### 4.1 Auth

- `POST /auth/magic-link/request` — body `{ email }`. Issues a magic link. Rate-limited per email and per source IP. Returns 202 regardless of whether the email exists (no account-existence oracle).
- `GET /auth/magic-link/consume?token=…` — first consumption issues a session token, sets a httpOnly secure cookie, redirects to `/`. The link is invalidated immediately on use; expiration 15 minutes.
- `POST /auth/sessions/revoke` — body `{ session_id }`. Revokes one of the user's sessions.
- `GET /auth/sessions` — list current sessions for the user (device_label, last_seen_at, this_one).
- `POST /auth/email-change/request` — body `{ new_email }`. Sends a verification email to the new address. Old email continues to work until verification.
- `GET /auth/email-change/confirm?token=…` — completes the change.

Account-level error surfaces (matter-of-fact voice) are returned with structured codes for the client to render: `magic_link_expired`, `session_timed_out`, `aviary_load_failed`. The client never composes its own error prose for these — the server's matter-of-fact strings are canonical.

### 4.2 Aviary state

- `GET /aviary/snapshot` — returns the current aviary snapshot (see §4.3 for shape). Used on first load (the response is also embedded in the initial HTML when possible) and on tab visibility return.
- `WS /aviary/stream` — bidirectional. Server pushes snapshot deltas and call events; client pushes interaction events. WebSocket is the normal path; long-polling fallback exists for environments that block WS.

### 4.3 Snapshot shape

```
{
  "aviary": {
    "daynight_phase": "morning",
    "daynight_t": 0.42,
    "weather": { "kind": "clear" },
    "settled": false,
    "tick_at": "2026-05-08T13:14:00Z"
  },
  "birds": [
    {
      "id": "uuid",
      "species_code": "warbler",
      "display_name": "pip",
      "perch": "front",
      "x_norm": 0.31,
      "facing": "right",
      "mood": "content",
      "plumage_render_token": "warbler_pal_07_sat_b",   // derived; no scalar
      "idle_micro_motion": "preening",                  // a small enum the renderer maps to keyframes
      "next_call": {
        "starts_at": "2026-05-08T13:14:08Z",
        "motif_id": "warbler_two_note_rise",
        "shape_params": { /* small set: pitch nudge, duration, pause, varied per call */ }
      }
    }
  ],
  "notebook": { "latest_entry_id": "uuid", "unread_since_last_open": false }
}
```

Snapshots carry **no** personality scalars and **no** raw mood timers; the renderer only needs the mood label and the derived render tokens. This is enforced at the serializer; the personality fields do not have a serializer pathway to the client at all.

### 4.4 Interaction events

- `POST /events/presence-ping` — empty body. Server validates the conjunction (visibilityState/focus/recent-activity) was claimed by the client and writes a `presence_ping`. Client must include a `client_ts` and the presence-conditions claim header for sanity; server rate-limits and dedupes.
- `POST /events/listen-in/start` — body `{ bird_id }`. Server confirms; client begins the audio mix ramp on confirmation (or optimistically, with rollback on rejection).
- `POST /events/listen-in/end` — `{ bird_id }`.
- `POST /events/offer` — body `{ kind, target_bird_id? }`. Server validates per-bird cooldown and aviary-wide rate; returns 429 with a structured `offer_cooldown_active` if rejected.
- `POST /events/settle` — empty.
- `POST /events/settle/undo` — empty. Only valid within 5 seconds of `settle`.

Personality state cannot be written by any of these. There is no `PUT /birds/:id/personality` and there will not be one.

### 4.5 Account, settings, export, deletion

- `GET /me` — account profile (matter-of-fact surfaces).
- `PATCH /me/settings` — accessibility, audio, visit-notifications, locale, timezone.
- `POST /me/export` — enqueues an export job. Returns 202. Job emails a signed download link to the verified address.
- `POST /me/delete` — soft-deletes (sets `delete_requested_at`). User can sign in during the next 30 days and recover.
- `POST /me/recover` — clears `delete_requested_at` if within window.
- `PATCH /birds/:id` — only `display_name` is mutable. Renames are written as `bird_renamed` events for the notebook to potentially reference. There is no other field a user can write on a bird.

### 4.6 Visits

- `POST /visits/invitations` — body `{ visitor_email }`. Issues an invite. Sends one-time email. 30-day expiration.
- `DELETE /visits/invitations/:id` — revokes (active or outstanding). Effective on the next snapshot pull.
- `GET /visits/invitations` — host-side list, with status and emails.
- `GET /visits/log` — host-side visit log.
- `GET /visit/:invite_code` — visitor entry. The visitor sees a sign-in-less, read-only aviary stream. The same WebSocket protocol is used, with a visitor token; visitor-issued events are rejected. The visit service writes `visit_started` / `visit_ended` to the host's log.

### 4.7 Narration / accessibility

- `GET /aviary/narration/stream` — Server-Sent Events. Pushes naturalist prose paragraphs at the calibrated cadence (~one per 30–60 s at idle, prioritized on user-initiated events). Captions for individual calls are pushed inline with call events on the main stream when the user has captions enabled, so they're synchronous with the audible call.

### 4.8 Voice / wording surface

The server is the canonical source of every user-visible string in the aviary surface (notebook prose, narration prose, captions, error surfaces). The client never composes naturalist prose locally and never templates it from raw state. This is the only way to keep the voice consistent — the moment two services produce naturalist prose with different generators, the voice fragments.

[Decision: matter-of-fact strings (sign-in errors, sync errors) live in the client bundle as a small message catalog, *because* they need to render before any server response is available (e.g. on offline entry). They are reviewed as carefully as the naturalist prose generators. Their identity is also referenced by structured server error codes so the client can pick the canonical string.]

---

## 5. Simulation engine design

The simulation service is the heart of the product. Everything that makes the aviary feel alive over weeks is computed here.

### 5.1 Tick model

- **Cadence**: target ~1 tick/min per active aviary. "Active" means: any aviary with non-zero events in the last 30 days; aviaries beyond that fall back to a coarser drift-only tick (every 10 min) until they re-activate. [Decision: this is bandwidth — reactivation is instant on next event arrival, so users don't perceive a degraded state.]
- **Per-tick work**: drain unprocessed events for the account (in `(occurred_at, id)` order) up to the tick boundary; recompute presence-time accumulator; advance mood timers; evaluate mood transitions; advance day/night phase; advance ambient weather state; schedule the next call for each bird; apply drift deltas; persist; emit a snapshot delta.
- **Determinism**: given the same starting state, same event stream, same wall clock, the tick produces the same output. PRNG seeded per `(account_id, tick_at)` so the same tick replays identically. This matters for debugging and for export-reproducibility.
- **Idempotency**: the watermark-and-events transaction is idempotent on retry. If the tick fails partway, replaying from the last committed watermark produces the same output.

### 5.2 Drift function

The drift function is a low-pass filter over scoring signals derived from the event log within the tick window.

For each trait `T` per bird `B`:

```
delta_T = filter_gain[T] * sum( signal_weight[T, signal_type] * signal_strength )
```

across the events in the window. The filter gain is small per-tick; the calibration target is:

- After ~7 days of regular visits (i.e. recurring presence-ping accumulation, periodic listen-in, occasional offers), instruments measure a clear delta on the dominant trait for the most-attended bird (~5–10% of the trait's range).
- After ~21 days, a typical user can perceive change without being told (e.g. the bird now reliably greets first; Pip's color is visibly richer).

Signal weights, in the rough order the PRD specifies:

| Signal | Trait pushed | Notes |
|---|---|---|
| `presence_ping` (per-aviary, per-window count) | All five traits, dominant input to `social_warmth` and `plumage_saturation`. | Distributes across birds; no "presence-time on a specific bird" because the user is watching the aviary, not a single bird unless listening in. |
| `listen_in_start..end` (duration) | Strong push on the *focused* bird's `social_warmth` and `vocal_frequency`. | Time-on-bird matters; very short listen-ins down-weighted to discourage flicker-tapping. |
| `offer_accepted` (per bird) | `curiosity` (strong), `boldness` (small). | An ignored offer still offers a tiny `boldness` push to the nearest bird (the user reached out). |
| `offer_initiated` near a bird | Tiny `boldness` push. | This captures the "offering at all" signal even before a bird reacts. |
| `settle_initiated` | No directional drift. | Treated as a clean presence-window terminator only. |

**Asymmetry rule (load-bearing)**: every signal contribution to drift is non-negative. There is no "absence drift," no "decay constant," no negative trait pressure from any source. A bird that gets ignored simply does not accumulate drift; its traits stay where they are. The implementation expresses this as `delta_T = max(0, delta_T)` after the sum, but the better implementation is that no individual term is negative in the first place — there is no "absence decay" term in the formula. CI test asserts this for every weight in the table.

**Saturation**: traits asymptote toward the high end of their normalized range with a soft cap; the filter approaches the cap exponentially, not linearly, so a bird that is already very expressive doesn't drift faster than a bird that is just becoming expressive. This avoids the failure mode where heavy users find their birds "maxed out" within a couple of months.

**No client-visible drift event**: no notebook entry says "Pip's social warmth went up." Notebook entries describe behaviors; the engine computes the underlying drift silently.

### 5.3 Mood transitions

Mood is a small finite-state machine per bird, driven by:

- **Recent interactions** — an `offer_accepted` nudges toward `content`; a startle (alarm call from a flockmate) nudges toward `wary`; sustained listen-in nudges toward `alert` early and `content` later.
- **Time of day** — at dawn most birds shift `drowsy → alert`; through the day, gentle drift toward `content` if no other input; at evening the population trends `alert/content → drowsy`; at night `drowsy → settled-in-place` (which we render as eyes-closed, not as a mood label change visible in the API beyond `drowsy`).
- **Ambient events** — a passing rain dampens vocal frequency *transiently* (not a drift change), and increases the chance of `wary` for low-boldness birds.
- **Personality damping** — high-boldness birds resist `wary` transitions; high-curiosity birds enter `curious` more readily.

Mood persists across tab opens. The simulation tick advances mood timers continuously; mood at session-start is whatever the tick has computed since session-end. There is no client-side mood reset on tab open.

The mood FSM is small enough to be hand-written and unit-tested: ~5 states, ~15 transitions, each with a triggering condition and a personality-modulated probability.

### 5.4 Call grammar and runtime

Each species defines a **motif library** (~6–10 short motifs per species) and a **grammar** specifying how motifs combine, with personality- and mood-modulated parameters:

- **Pitch nudge** ± a small range, modulated by mood (lower for drowsy, higher for alert) and by per-call randomness.
- **Duration** ± a small range, modulated similarly.
- **Pauses** within multi-motif calls, with mood-shaped distribution.
- **Repetition** counts.
- **Chorus joining**: a bird with high vocal frequency, when another bird is calling, has a probability of joining within a short delay window. This produces real-time chorus emergent behavior.

The grammar is procedural and per-call. Two consecutive calls from the same bird are never identical. The "no looped audio" rule is implemented at this layer: there are no audio files for calls. The motif library specifies *parameter sets for synthesis*, not recordings.

The server schedules *when* a bird calls (at tick time) and *what motif and shape parameters*; the client *synthesizes* the audio from those parameters. The server does not synthesize audio.

The server emits a `call_event` to active clients at the moment the call should begin, with motif id and shape params. The client's WebAudio path takes those params and synthesizes the call.

### 5.5 Return-greeting

Return-greeting is a server-driven event:

- The simulation observes the start of a fresh presence window (client opens the WebSocket, presence-ping arrives after a quiet window). It computes:
  - **Absence length** since last presence — categorized into stepped-away (<5min), short (5min–6hr), longer (6hr–48hr), long (>48hr). [Decision: these brackets are tunable; they shape the greeting flavor.]
  - **Greeting bird selection**: weighted by per-bird `boldness` and current `mood` (a wary bird won't greet first today). Probability of "no greeter today" exists for low-warmth aviaries; we choose to allow a session where no bird greets, because forcing a greet on every entry is exactly the canned-cue failure the PRD warns against.
- It emits a `greeting` event on the stream: `{ greeter_bird_id, flavor, absence_bracket }` plus the corresponding scheduled call/idle-motion sequence. The flavor variants (glance up from preening, two-note call and look, head-tilt and step forward, longer call followed by another bird's response) are parameterized — multiple "longer call" greetings are not the same call; they are procedurally distinct.

When several birds would greet simultaneously, the scheduler staggers their call times by randomized small offsets (200–900 ms) so the greeting reads as the aviary noticing one bird at a time, not as a chorus on cue.

There is **no** textual welcome surface, anywhere. The greeting is birds.

### 5.6 Ambient weather

A simple Markov scheduler chooses, at each tick, whether to start a weather event (very low probability per tick), what kind (rain or wind, never thunderstorm or snow), and how long it lasts (a few minutes for rain; less for wind). Effects:

- Rain: dampens vocal frequency *transiently* (modifier on call scheduling; does not write to drift). Notebook entry possibly generated ("a short rain passes; the aviary quiets briefly").
- Wind: small mood-shift probability — `alert` for high-boldness birds, `wary` for low-boldness birds.

Weather events are part of the snapshot. The renderer paints the corresponding visual (rain streaks, leaf rustle).

### 5.7 Adoption pacing and species selection

- Two starter birds at account creation. Species selection: weighted random over the 6-species pool, with the pair chosen to have *different silhouettes and call timbres* so the user immediately experiences distinguishable calls.
- New-bird offers: paced by *aviary age*, specifically `created_at`. Schedule: first offer at ~3 months, then ~9 months, ~18 months, ~30 months, ~48 months — diminishing returns up to seven. Not interaction-count gated. Not visit-count gated. The slot is offered once per scheduled window; if the user declines, it returns at the next scheduled window. (Decline is the absence of accepting; we do not show a "decline" button as a separate prompt — see §6.4 of the interactions/UI subsection.)

### 5.8 What the tick is *not*

- Not a place where any user-visible string is composed. Notebook prose comes from the notebook service; narration prose comes from the narration service. The tick produces signals; the prose services interpret signals into voice.
- Not a place where personality scalars cross any service boundary. Personality stays in this service's database.

---

## 6. Sync model

### 6.1 Single canonical aviary

There is one canonical aviary per account, stored server-side. Multi-device sync is a *property of the architecture*, not a separate component:

- Both clients ask for `/aviary/snapshot`. They both get the same record.
- Both clients open `/aviary/stream`. Both get the same emitted snapshot deltas.
- Neither client writes personality.

### 6.2 Why no last-write-wins

The PRD's rule is that personality drift is implemented as additive, server-authored deltas — never as client-submitted absolute values. This plan implements that as an architectural invariant:

- The client API has no method whose net effect is "set bird.boldness to X" or even "increment bird.boldness by X." The closest method writes an interaction event into the append-only log.
- The simulation service is the only writer of `personality_vectors`.
- The simulation service consumes `interaction_events` strictly in `(occurred_at, id)` order and writes through a single per-account tick worker. Two concurrent ticks for the same account cannot both write — the worker takes a per-account advisory lock at tick start.

### 6.3 Concurrent sessions

Two devices sending events concurrently is fine: their events both land in the append-only log, ordered by their `occurred_at` timestamps. The tick processes them in order. If clocks disagree mildly, the order is approximate but consistent (and the tick is idempotent). [Decision: client `client_ts` is informational; the server reuses `ingested_at` as `occurred_at` on conflict.]

Same-second listen-in toggling between devices is theoretically possible (laptop user starts listen-in on Pip; phone user starts listen-in on Wren a moment later). Both events land; the simulation honors them as a sequence: the laptop's listen-in started, then ended (because the phone took over), then the phone's listen-in started. The laptop *display* may show listen-in still active for a moment until the next snapshot delta; this is not a sync failure, it's the cost of separate browser tabs each rendering local audio mix optimistically. [Decision: accepted — the case is rare and self-correcting at the next snapshot.]

### 6.4 Sync conflict surface

Genuine conflicts that surface to the user are rare: magic-link replay across browsers, an in-flight session timing out mid-write. The error surface drops into matter-of-fact voice with structured codes:

- `magic_link_expired` → "We couldn't sign you in. The link may have expired. Try requesting a new link."
- `session_timed_out` → "Your session timed out. Sign in again to keep watching."
- `aviary_load_failed` → "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

None of these mention naturalist prose, the birds, or the aviary's affective surface. A user in an error state needs system clarity.

---

## 7. Frontend rendering pipeline

The renderer is a 60fps idle-motion compositor with a procedural-call audio path; the whole thing has to fit in <2 MB gzipped including the audio motif library and bird assets.

### 7.1 Stack

- **Framework**: small reactive layer (Preact-class or Solid-class — minimal runtime). [Decision: Preact + Signals; React is too heavy for the bundle budget, vanilla DOM is too verbose for the surface complexity.]
- **Renderer**: 2D, hardware-accelerated, via Canvas or WebGL. [Decision: WebGL through a tiny custom layer (no Three.js — too large). Canvas as a fallback for environments where WebGL isn't available.] The rendering uses sprite atlases per species, with skeletal joints driven by procedural motion.
- **Audio**: WebAudio. Custom synthesis nodes — no third-party audio libraries (bundle budget).
- **Idle ornament path**: ambient leaf and feather drift is generated client-side from a deterministic-but-varied PRNG; it is not in the snapshot stream, because per-leaf state would balloon the snapshot for no benefit.
- **State**: server snapshots are kept in an in-memory store keyed by `bird_id`; the renderer reads from this store every frame.

### 7.2 Scene composition

- One horizontal scene, viewport-fit, no panning / scrolling / zooming. Aspect-preserving responsive layout.
- Three perch zones — front, middle, back — each a normalized x-band with a small y-offset for parallax.
- Foreground branch and back foliage on parallax layers, subtle.
- Birds positioned by `(perch, x_norm, facing)` from the snapshot.

The first frame the user sees is *not* a load state. The render path:

1. HTML response includes a `<script type="application/json" id="initial-snapshot">` element with the user's snapshot, signed and TTL-bound.
2. As soon as the JS executes, it reads this snapshot, mounts the renderer, places birds in their snapshot positions, kicks off idle micro-motion mid-pose, begins WebAudio synthesis. The first paint — within the 500 ms budget on mid-tier mobile / 4G — has birds in motion.
3. There is no fade-from-static, no spinner, no entry animation. If the snapshot is unavailable (cold cache, slow connection, post-deploy sentinel cleared), the render shows a quiet field — soft sky color, perhaps one or two faint motion cues. **Not a spinner.** A spinner reads as "machine"; the quiet field reads as "the aviary catching up."

### 7.3 Idle micro-motion

Every bird, every frame, has subtle motion: preening, scanning, head-tilt, body-shuffle. Implemented as:

- Skeletal poses per species; idle micro-motion is a per-pose noise function applied to joints, with frequency and amplitude shaped by the bird's current `mood` (drowsy: low frequency, low amplitude; alert: higher frequency).
- The `idle_micro_motion` field in the snapshot tells the renderer which behavior to emphasize at the moment.
- Motion is keyed off wall time, not per-frame state, so motion is continuous across frames at variable framerate.

### 7.4 Transitions

- **Perch changes**: rendered as a smooth flight path between perches over a short interval. The snapshot flips `perch` at the tick boundary; the renderer sees the change and animates the transition in.
- **Mood changes**: a soft cross-blend of pose biases over ~1.5–3 s. No pose snap.
- **Call-mouth animation**: synced to the call event's start/end; the renderer opens the bird's beak in shape with the call's amplitude envelope (which the client knows because it's synthesizing the call).

### 7.5 Reduced-motion mode

Reduced-motion is its own designed surface, not a "motion off" toggle:

- Idle micro-motion is replaced by a sequence of *still poses* that cross-fade slowly (~1–1.5 s per crossfade, every 4–8 s). The bird is never frozen; it is moving between still moments.
- Flight transitions become cross-fades between the "at front perch" and "at middle perch" poses, ~1 s, no animated path.
- Ambient leaf drift is removed. Day-night color shift remains, slowed.
- Procedural calls play at full quality (unless captions-only mode); the audio path is not affected.
- Reduced-motion is triggered by `prefers-reduced-motion` *or* the explicit setting in accessibility. The aviary scene is the same aviary — same birds, same drift, same notebook — in a different visual register.

The renderer is built so reduced-motion is one branch, not a separate codepath. Same poses, same transitions; just the parameters of how they animate change. This is the only way reduced-motion stays current with feature changes.

### 7.6 Top bar

- Thin top bar above the aviary scene. Fixed icons: account/settings, accessibility settings, field notebook, offer affordance.
- After ~3 s of cursor stillness, the top bar fades to ~15% opacity. It returns to full opacity on cursor movement or any keyboard activity.
- The top bar is the only on-aviary chrome. The scene itself never carries chrome — no badges on birds, no hover-tooltips, no inline labels.

[Decision: focus indicators on the top bar use a soft high-contrast outline that reads against the dim faded state too — focusing into a faded top bar must remain visible.]

### 7.7 Tab visibility handling

- On `visibilitychange → hidden`: the renderer halts (no `requestAnimationFrame`), the WebSocket is closed, audio is suspended. The tab consumes ~zero CPU while hidden.
- On `visibilitychange → visible`: re-open WebSocket, pull a fresh snapshot, resume rendering and audio. The aviary is now in the state the simulation has computed during the hidden interval — birds may be in different perches, different moods, different times of day.

### 7.8 Empty-aviary state

The empty state — the brief moment after adoption-flow naming and before the first bird enters — uses the same quiet field as the cold-cache state. Then the first bird enters with a soft fly-in to its starting perch. After that, the user never sees an empty aviary again.

---

## 8. Audio pipeline

Audio is the affective spine. The procedural-call rule is non-negotiable.

### 8.1 Call synthesis

- Each bird has a species-keyed motif library. A motif is a parameter set: a small piecewise-linear pitch contour, an amplitude envelope, a timbre token (which selects a small bank of WebAudio oscillator/filter compositions per species), and meta-parameters (variation ranges).
- A call from the server arrives as `{ motif_id, shape_params, start_at }`. The client renders the call by:
  1. Creating a fresh audio graph (oscillators + biquad filters + a soft noise channel for breathiness, all reused from a pool).
  2. Driving it with the motif's pitch contour, modulated by the shape params (per-call pitch nudge, duration scaling, pause durations).
  3. Connecting to the master mix bus through a per-bird gain node.
  4. Releasing the graph back to the pool when the call envelope reaches zero.
- Two consecutive calls from the same bird are never identical because shape params vary every call.
- No recorded audio at any quality is shipped or played for calls. Ever. Bundle-budget rejects this even if we wanted to.

### 8.2 Chorus mixing

Two birds calling in overlapping windows produces a real chorus because both calls are synthesized live with their own per-call shape params. There is no phase-cancellation artifact characteristic of layered loops because the two call signals are not the same signal twice.

### 8.3 Listen-in mix

- Master mix sums all per-bird gain nodes through a master compressor, then to destination.
- On `listen_in_start(bird_id)`: the focused bird's gain ramps up to a higher level over ~700 ms; all other birds' gains ramp down to "ambient" (not silent) over the same window. Ambient is a background-presence level — quiet enough to read as "the others are there" but not foregrounded.
- On `listen_in_end`: gains ramp back to baseline over ~700 ms.
- Hard cuts are not used. Listen-in is "leaning in to hear," not "soloing a track."
- Other birds **never** go fully silent during listen-in. The aviary is still the aviary; what changes is where the user's ear is pointed.

### 8.4 Ambient and weather audio

- Continuous low-volume ambient bed (a procedural texture: gentle pink-noise filtered with slow LFO movement, plus a sparse layer of distant call echoes). Synthesized client-side; not a recording.
- Rain weather event: an additional procedural rain texture (filtered noise) ramps up over ~3 s, persists, ramps down. Mix is calibrated to remain calm — no sense of storm.
- Wind: a slow filtered-noise sweep, long-attack, long-release.

### 8.5 Captions

- When captions are enabled, every call event also emits a caption string (server-generated from the procedural call grammar, in naturalist voice). The client renders the caption near the calling bird as a small text element, fading in and out with the call envelope.
- Caption text is generated from the *actual* shape params, not a fixed string per motif. A two-note call with a longer second note caption-renders as "a soft two-note call, the second held"; same motif, different shape params, different caption.

### 8.6 WebAudio fallback

If WebAudio is unavailable (older browser, AudioContext permission denied, hardware issue), the aviary plays in graceful silence with captions on by default. There is no recorded-audio fallback path. Silence with captions is treated as a real product surface, not a degradation: the visual aviary continues full-quality, the captions narrate the calls in naturalist voice, and the user gets the aviary just without the audio.

The detection path:

1. On boot, attempt to construct an `AudioContext`. Catch any exception.
2. If absent, set an internal `audio.fallback = silent` flag, force-enable captions, and surface a small matter-of-fact note in accessibility settings explaining the state.
3. On `AudioContext` state change to `suspended` (autoplay policy), prompt the user once via a small affordance to "enable sound." If declined or unavailable, remain in silent + captions mode.

---

## 9. Accessibility surfaces

Accessibility is its own designed surface, not parity-by-checklist. Each surface is built to feel alive in its register, not to label the visual one.

### 9.1 Screen-reader narration

- The narration service generates running prose from the simulation snapshot at ~30–60 s cadence at idle, faster only on user-initiated events (return-greeting, offer reaction, settle).
- Voice is identical to the field notebook: lowercase, present-tense, specific:
  > "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- Delivered via an SSE stream and rendered into an `aria-live="polite"` region at the appropriate cadence. Polite (not assertive) so the screen reader's queue isn't bulldozed.
- User-initiated events get a small priority bump but remain phrased as observations:
  > "wren glances up and calls once."
  Not:
  > "Listen-in started on wren. Mood: alert."

The narration service is implementation-paired with the notebook service: they share the prose generator and voice catalog. The voice cannot fork between the two surfaces.

### 9.2 Captions

See §8.5. Captions render as small text elements in the aviary scene, near the calling bird, fading with the call. They are part of the visual surface but live near the calling bird so the spatial cue ("this bird is calling now") is preserved. For screen-reader users with audio off, captions also feed into the narration stream as small prose updates synchronized with the call.

### 9.3 Focus and keyboard navigation

- Tab order: top bar (settings → accessibility → notebook → offer → settle), then aviary surface.
- Once focus is in the aviary surface, arrow keys move focus between birds, Enter triggers listen-in on the focused bird, Escape ends listen-in, the offer affordance opens with `O` (configurable; documented in accessibility settings) and is fully keyboard-navigable internally.
- Focus indicator: a soft, high-contrast outline that reads against both bright and dim aviary states. No focus indicator is ever entirely hidden by the top bar's faded state.
- The settle gesture is reachable from the top bar; the 5 s undo is reachable as a focusable element that appears immediately after settle and traps focus until it dismisses.

### 9.4 WCAG AA contrast

All user-copy text — top bar labels, settings, account surfaces, error surfaces, captions, narration when displayed visually — passes WCAG AA. The design system specifies the actual ratios. The aviary scene itself contains no chrome user copy except in the top bar and captions.

### 9.5 Reduced-motion

Already covered in §7.5. The PRD-level point worth restating: reduced-motion is its own product surface. A reduced-motion user is not getting Pocket Aviary minus animations; they are getting Pocket Aviary in a calmer visual register. The notebook, the narration, the calls, the drift — all unchanged.

### 9.6 Audio-off mode

A user can mute audio (per-account setting, or via a top-bar mute affordance). When muted, captions auto-enable. The aviary is full-quality otherwise.

### 9.7 Settings surface

Accessibility settings live behind the accessibility icon in the top bar. Surface drops out of the naturalist register into matter-of-fact: "Reduced motion: on / off / system." "Captions: on / off." "Audio: on / muted." "Screen-reader narration cadence: slower / standard." This is the named-exception register.

---

## 10. Performance budgets and observability

### 10.1 Initial JS bundle <2 MB gzipped

The bundle includes: the Preact-class framework (~20 KB), the renderer (~80–120 KB), the audio synthesizer + motif library (~150–250 KB), bird sprite atlases for 6 species (~400–600 KB if compressed efficiently), the message catalog (matter-of-fact strings + their canonical bindings, ~10 KB), the screen-reader narration consumer / aria-live wiring (~10 KB), routing and account UI (~50–100 KB).

Code-splitting:

- Account settings, accessibility settings, the visit invitation flow, the field notebook reader, the export/delete flow are lazy-loaded on first navigation. The initial bundle is just the aviary surface.
- The species pool's full audio motif library can be split per species, with the user's adopted species' libraries in the initial bundle and the full pool's libraries lazy-loaded only when an adoption flow happens.

CI bundle-size check: hard-fails if `gzip(initial-bundle) > 2 MB`. This is a build-time constraint, not an aspiration.

### 10.2 Time to first bird visible <500 ms

On a mid-tier mobile device over 4G, the first bird is visible within 500 ms of navigation:

- The HTML response is delivered from the CDN edge with the initial state snapshot embedded inline (signed, short-TTL, sized to ~2–8 KB for a typical aviary).
- The bundle is preloaded via `<link rel="preload">` and split into `aviary-critical.js` (the minimum to render the first bird — framework, renderer, sprite atlas for the user's species) and `aviary-rest.js` (audio synthesis, full sprite atlas, narration consumer, secondary surfaces). `aviary-critical.js` size budget: ~500 KB gzipped.
- Render path: HTML → bundle → mount renderer → read embedded snapshot → composite first frame. No await on audio, no await on full motif library, no await on auxiliary chrome.
- Audio synthesis comes online a beat later (within ~1 s) with no visual gap — the bird is already on screen; the calls start when synthesis is ready.

CI synthetic check: a fleet of automated browsers runs the page load on a schedule from common geographies on emulated mid-tier mobile / 4G. p95 time-to-first-bird must be below 500 ms; alarm at p95 > 500 ms for one hour.

### 10.3 60 fps idle on a 5-year-old laptop

- Renderer enforces a frame budget: per-frame work targeted at <8 ms, hard cap at 12 ms. Above the cap, idle micro-motion noise functions are sampled at lower frequency (visually imperceptible).
- Audio: per-call audio graph allocations are pooled; at 60 fps with <=7 birds, garbage collection should not be on the hot path.
- The reduced-motion path is also 60fps but with most motion cross-fading at long intervals; trivially within budget.
- CI synthetic check: a 30-minute soak on a 5-year-old laptop CI runner; p95 frame time < 16.67 ms.

### 10.4 No memory growth over 30 minutes

- Audio buffers are pooled and reused.
- Notebook entries scrolled into view do not retain rendered DOM; the notebook surface uses a windowed list with explicit teardown on scroll-out.
- WebSocket reconnection clears any retained snapshot history beyond the current one.
- CI test: 30-minute headless session with synthetic interactions; heap snapshot at 0 and 30 min must show < 10% growth (and that growth is bounded).

### 10.5 What we measure

Aggregate operational telemetry only:

- Request counts and latencies per endpoint.
- Error rates per endpoint.
- Simulation-tick latency (per-tick wall time, distribution). Alarm at p99 > 5 s.
- WebSocket connection duration histograms (anonymized; no per-account dimension).
- Audio-context error counts (categorized by reason).
- Client render-frame timing histograms (anonymized).
- Bundle-size and time-to-first-bird from synthetic checks.
- Email service delivery rates.

Explicitly **not** measured (the privacy boundary):

- No per-account session-duration entry into the warehouse.
- No per-bird interaction counts in the warehouse.
- No "how often do users use offer," "what's the typical aviary's drift after a month," "what's the population's average mood distribution" — these queries do not exist as a capability in our analytics stack. Building them would require crossing the database boundary, which has no credentials path.
- No drift-curve analytics across accounts. The simulation DB is not joined to anything.

### 10.6 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers receive a matter-of-fact unsupported-browser surface in the simple HTML pre-bundle path. We do not maintain compatibility paths for very old browsers; the bundle bloat isn't justified.

---

## 11. Rollout

### 11.1 Phasing

- **Phase A — Closed alpha (~6 weeks)**: invite ~100 users via direct email. Feature-complete on the bird engine, mood, drift, the two starter birds, listen-in, offer, settle, the notebook, multi-device sync, accessibility surfaces. Bird cap held at 3 during alpha to focus tuning on drift calibration. Visit feature off.
- **Phase B — Open beta (~6 weeks)**: open registration with a soft cap, bird cap at 5. Visit feature on. Notebook entry frequency tuned. Reduced-motion mode in production for opt-in users.
- **Phase C — v1 launch**: bird cap at 7. New-bird offers ramped at the schedule in §5.7.

The cap-ramping during alpha and beta is a calibration affordance, not a product feature: the cap is computed in one place (a server-side config), and the species pool grows under it. The user experience is unchanged at any cap — they still get the calm, two-starter, depth-not-breadth product.

### 11.2 What we instrument from day one

- Drift function instrumentation: aggregated *internally* (not in the warehouse) on a per-deployment basis, used to validate the calibration target ("clear delta in instruments after 7 days of regular visits"). This is *not* a per-user analytic; it is used during alpha/beta to see whether drift is moving as designed across the pool, with all data scoped to the engineering team's measurement window and discarded afterward.
- Synthetic perf checks (§10.2, §10.3): ongoing.
- Error rates and tick latency: ongoing.
- Audio fallback rate: how often clients land in silent + captions mode. Used to calibrate the mute messaging.

### 11.3 What we do not instrument, ever

- Per-user retention metrics packaged for "engagement reporting." We do not have a "DAU/MAU" surface for this product. Operational health tells us the product is up and working; the success metric for Pocket Aviary is whether the users we have keep their birds, not whether some proxied count is going up. Adding a DAU dashboard would create internal pressure to optimize for it, which would convert the team's design priorities subtly toward engagement, which is exactly the rotation the PRD spends three pages refusing.
- Streak-style metrics or any surface that exposes per-user visit-frequency to internal teams in a way that could become a target.

### 11.4 Onboarding and first-encounter

- Magic-link sign-in lands on the adoption flow. The adoption flow is matter-of-fact in voice: "two birds have arrived. they are a [species] and a [species]. give them names." (lowercase prose here is fine because this is the first contact moment with the aviary, on the seam between system and product. The naturalist register starts.) [Decision: this exact wording is the seam-handling style. Subsequent prose is fully naturalist; the sign-in/error prose is fully matter-of-fact.]
- The user types names (defaults provided). On submit, the empty-aviary state shows for ~1 s; the first bird flies in to a starting perch; the second a beat later. From that moment forward, the aviary is the surface.
- No onboarding tutorial. No "click here to listen in" tooltips. No "pro tip: try offering a seed." The product teaches itself by being present; the user discovers listen-in by clicking on a bird and hearing its call rise. The discovery is part of the relationship.

### 11.5 Deployments

Standard CI/CD per service. Snapshot serializer changes are versioned (clients see a `snapshot_version` field; older client versions reject new snapshot shapes gracefully and prompt for reload — matter-of-fact voice). Simulation-service deploys are paused during a tick cycle and use a canary-then-promote pattern; ticks are idempotent so a redeploy mid-tick replays from the watermark.

### 11.6 Operability

- Each service is deployed as small, independently scalable workers.
- Simulation-service workers are sharded by `account_id` to keep per-account ticks serialized.
- Per-account tick worker takes an advisory lock at tick start to prevent double-tick.
- Backpressure: if the event log grows faster than ticks consume it for an account, the next tick processes the full window in one pass (slightly larger drift step) — no event is dropped.

---

## 12. Risks

This section is for the things that can go wrong even when everything else is right. Each risk is paired with a mitigation that is part of this plan, not deferred.

### 12.1 Drift calibration

**Risk**: drift is too fast or too slow. Too fast and the aviary becomes a Tamagotchi. Too slow and it's a screensaver. Either way, the central premise — feels alive over weeks — is broken silently; users will report something feels wrong without being able to name it.

**Mitigation**: explicit calibration targets (instruments at 7 days, user-perceptible at 21 days), instrumented during alpha/beta against a real cohort, with the drift weights treated as a tunable that can be re-deployed without code changes. Also: the asymmetric rule (no negative drift on neglect) is unit-tested per signal weight; a regression that introduces a negative weight is caught at CI.

### 12.2 Sync correctness

**Risk**: a bug introduces a path where the client effectively writes personality state — perhaps an "optimistic update" of mood or boldness that gets persisted somewhere and races with a server tick. Once any such path exists, the no-last-write-wins guarantee is theoretical.

**Mitigation**: type-system marks the personality vector as `readonly` on the client; the API has no method to write it; the simulation service is the only code with credentials to update `personality_vectors`; a CI integration test asserts that running a multi-device session for an extended window produces drift values consistent with the event log alone.

### 12.3 Audio uncanniness

**Risk**: procedural calls *sound* procedural — too synthetic, too uniform, too "FM-y." Users sense it on the first session and the aliveness premise collapses.

**Mitigation**: this is a real risk and it requires a sound designer in the loop, not just engineers. The motif libraries are designed by an audio designer with the engineering team; calibration is iterative against listen tests with the alpha cohort. The chorus mechanic is tested early because that is where uniformity is most exposed. WebAudio synthesis nodes use multi-oscillator + modulated filter compositions per species, plus per-call breath-noise modulation, so timbres differ per species and per call.

A complementary mitigation: the procedural-call rule is enforced architecturally (no recorded audio in the bundle at all), so we can't accidentally fall back to "just ship a recording for the species we couldn't get right." The rule forces us to keep tuning until the calls are good.

### 12.4 Accessibility regressions

**Risk**: a "polish" change to the visual aviary surface — a new ambient effect, a perch rearrangement, a new mood pose — ships without a matched update to the screen-reader narration generator or the captions generator. The product gets richer on the visual surface and silently flatter on the accessible one. Over a few releases, the accessible surface is a stripped variant.

**Mitigation**: every PR that touches simulation state, mood, perch logic, or call-grammar shape params is required to update the narration prose generator and the captions generator. CI lint flags PRs that change relevant files without touching the prose generators (warning, not block — but visible). More substantively: the prose generators are owned by the same team that owns the simulation, not by an "a11y" team off to the side. Voice consistency requires shared ownership.

### 12.5 PII leakage via identifiers

**Risk**: a developer reaches for email as a "convenient unique identifier" in a new service or telemetry path. PII spreads into observability tooling that no one fully audits.

**Mitigation**: the synthetic UUID rule is enforced architecturally — every shard key, partition, log line, and inter-service message carries the UUID, never the email. CI scan rejects new code that reads `accounts.email` outside the email service or the audit-logged email-display surfaces (export, settings).

### 12.6 Engagement-feature creep

**Risk**: a well-meaning contributor adds a "harmless" engagement feature — a tiny streak indicator, a notebook entry that mentions visit frequency, a quiet calendar in settings, a "your friend visited!" notification on by default. The product subtly becomes a different product over six months.

**Mitigation**: the non-goals rule lives as named constants in the codebase. Any PR adding a counter, calendar, or notification surface trips review on `non_goals.md`. The PRD is required reading on the team. The team lead is the keeper of the line. The architectural rule that the analytics warehouse cannot join per-account interaction state means that a "harmless" engagement feature can't be quietly built on the back of existing data — it would require new pipelines, which forces the question into the open.

### 12.7 Performance regression on a long session

**Risk**: a slow memory leak in the audio path or the notebook surface; users on long sessions start losing frames after 30+ minutes. The "feels alive" premise wears off in the second half-hour.

**Mitigation**: 30-minute soak tests in CI, with a hard threshold on memory growth. Audio buffer pool sizes audited per release. The notebook windowed-list has explicit teardown tests.

### 12.8 First-bird budget regression

**Risk**: a feature lands that pushes the bundle over 2 MB or pushes time-to-first-bird over 500 ms. Above these thresholds the central conceit cracks.

**Mitigation**: bundle-size hard fail in CI. Time-to-first-bird synthetic alarm in CI on new builds. New features that are essential and budget-bursting must be paid for elsewhere (drop a species, simplify a sprite atlas, lazy-load something else); the budget itself is not adjustable from inside a feature PR.

### 12.9 Empty-state on the first encounter

**Risk**: a slow CDN region or cold cache delivers the page without the embedded snapshot; the user lands on the quiet field and stays there for several seconds. They infer "the product is loading" — the spinner-failure mode without the spinner.

**Mitigation**: the embedded-snapshot path is a hard requirement of the edge response, not an optimization. Synthetic checks alarm if any region's snapshot-embedding rate drops. The fallback quiet field is itself styled to not read as "loading" — soft sky color, slow color drift, subtle motion cues that look like ambient rather than progress.

### 12.10 Voice fragmentation between services

**Risk**: notebook prose, narration prose, captions, error surfaces, and onboarding strings end up in different places, written by different teams, with subtly different voices. The product's voice — the load-bearing charm engine — fragments into "naturalist-ish."

**Mitigation**: prose generators (notebook, narration, captions) live in one shared module owned by one team; voice samples and rejection criteria are codified. Matter-of-fact strings live in a single message catalog. The handful of "seam" strings (adoption-flow prompts) are reviewed by the same prose owners.

---

## 13. What this plan does *not* spell out (intentionally)

A small list, so a downstream engineer can see the seam between PRD-level commitments and implementation-level latitude:

- Exact normalized ranges for personality scalars: implementation detail in the simulation service.
- Exact tick cadence (1/min vs 30/s): pinned by perf during alpha; the architecture supports either.
- Exact filter gains per drift signal: tuned in alpha against the calibration targets.
- Exact mood probabilities: also tuned in alpha. The transition graph is fixed; the probabilities are tuned.
- Concrete species pool: chosen by the visual designer + audio designer team; expected to be ~6 species, with care taken that their silhouettes and call timbres are mutually distinguishable.
- Exact contrast ratios: per the design system spec; floor is WCAG AA on user copy.
- Exact UI typography and palette tokens: design system spec.

These are deliberately implementation latitude. Everything above this section is committed.

---

## 14. Summary

Pocket Aviary v1 is an aviary that runs on a server-authoritative simulation, renders on a small fast browser client, and protects its central conceit — feels alive over weeks — with architectural rules: server-only personality writes, asymmetric drift, procedural calls, no engagement metrics, naturalist voice generated in one place. The accessibility and performance surfaces are first-class, designed-in constraints, not retrofits. The product's refusals — no streaks, no Tamagotchi, no social network beyond a single quiet visit — are enforced at the codebase, not just the PR description.

The plan above is executable: a frontier engineering team, with a designer and an audio designer alongside, can build this in a phased rollout (closed alpha, open beta, v1 launch) over a small handful of months while tuning drift and audio against a real cohort. The hard parts — drift calibration, audio uncanniness, voice consistency, PII discipline — are flagged and matched to mitigations that are part of this plan rather than promises against later effort.
