# Pocket Aviary — v1 Implementation Plan

This plan interprets the Pocket Aviary PRD into an executable design for a frontier engineering team. The deliverable is the plan; the product is not implemented in this document. Wherever the PRD names a calibration target (presence window, drift cadence, narration cadence, etc.) without a precise value, the plan names a defensible default and flags the calibration site.

## 0. North-star constraints (read first; everything else is downstream)

These five rules are non-negotiable. They are restated here as constraints, not aspirations, so every later section is forced to comply.

1. **Server is the only writer of personality state.** Clients submit interaction events; the server-side tick consumes them and emits canonical state. No client may write personality vectors under any code path.
2. **Drift is monotonic toward expressive.** Traits never decrease on neglect. The drift function is asymmetric by design.
3. **Personality values are never exposed to the user.** Not in UI, not in API responses to the client, not in a debug toggle, not at any tier.
4. **The aviary appears already in motion.** First frame = mid-action. No "wake up" animation, no spinner-resolves-into-aviary, no fade-from-static. The loading state is a quiet field, not a UI element.
5. **No announcement surfaces on return.** No "Welcome back!" toasts, no streak counters, no calendar of green dots, no "your friend visited!" badge, no "you've been gone X days" surface. The bird greeting is the only welcome.

If any later section appears to violate one of these, the section is wrong.

---

## 1. Scope

### 1.1 In scope (v1)

- Single-user accounts, magic-link sign-in, synthetic account UUIDs.
- One aviary per account, 2 starter birds, cap of 7 birds.
- Server-side simulation tick (~1 minute cadence, calibratable) that owns personality state.
- Multi-device sync (a property of the architecture, not a feature).
- Procedural call synthesis client-side; recognizability-preserving per-bird call signatures.
- Idle motion, mood-shaped motion, day/night cycle, ambient weather, ambient leaf/feather drift.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook.
- Visit invitations (opt-in, per-invite, read-only ambient view, no co-presence, no comments).
- Accessibility: screen-reader narration, reduced-motion mode (its own designed surface), call captions, WCAG AA contrast on user-copy, full keyboard navigation.
- Performance: <2MB initial JS (gzip), <500ms time-to-first-bird on mid-tier mobile over 4G, 60fps idle motion on a 5-year-old laptop, no memory growth over 30 min.
- Privacy boundary enforced at the data-pipeline level (per-bird state never reaches aggregate telemetry).
- Soft-then-hard account deletion (30-day window), account export.

### 1.2 Out of scope (v1, per `non_goals.md`)

- Native iOS / Android apps. Web-only.
- Gamification of any flavor: no streaks, no badges, no levels, no counters, no XP, no rank, no green-dot calendar, no "you've been here every day this week" surface anywhere.
- Tamagotchi-style mechanics: no hunger, no decay meters, no distress, no death, no happiness-number-surfaces.
- Social-network surfaces: no profiles, no follows, no public feed, no discovery, no comments, no leaderboards, no "show-off" rendering for visitors, no friend-of-friend chains.
- A UI surface that exposes personality vector values.
- Any notification, push, or email the user did not explicitly request (visit notifications are off by default and opt-in).
- Multi-aviary accounts, shared aviaries, household profiles.
- Customizable scenes, drag-to-place perch layout.
- Recorded-audio fallback for procedural calls.
- A "show me how my bird is doing" debug/stats panel that exposes personality numerically.

### 1.3 Explicitly named absences to defend against feature creep

- A "first time you saw this bird" recap surface is out of scope; it would be a disguised engagement metric.
- An exportable visit log surfaced as a calendar is out of scope; the visit log is for transparency, not performance.
- Notebook entries about the *user's* behavior (visit counts, time-of-day usage) are out of scope. The notebook observes the aviary, never the user.

---

## 2. Architecture

### 2.1 System shape

```
            +-----------------+         +-----------------------+
            |  Web client     | <-----> |  API gateway / BFF    |
            |  (browser)      |         |  (HTTPS, TLS)         |
            +-----------------+         +-----------+-----------+
            ^ ^ |                                    |
            | | |                                    v
            | | +-- WebAudio synth      +-----------+-----------+
            | |    (client only)        |  State service         |
            | +---- presence pings      |  - canonical aviary    |
            +----- render snapshots     |  - append-only event   |
                                       |    log                 |
                                       |  - simulation tick     |
                                       |  - visit invitations   |
                                       +-----------+-----------+
                                                   |
                                       +-----------+-----------+
                                       |  Auth service          |
                                       |  - magic-link issuance |
                                       |  - per-device sessions |
                                       |  - synthetic UUIDs     |
                                       +-----------+-----------+
                                                   |
                                       +-----------+-----------+
                                       |  Notifier (transactional|
                                       |  email only)           |
                                       |  - magic links         |
                                       |  - visit invites       |
                                       |  - export download link|
                                       +-----------------------+
```

Three services, plus a notifier for email. No analytics warehouse reads the simulation database. The boundary is enforced by IAM scopes, not just by convention.

### 2.2 Service responsibilities

**Auth service**
- Issues magic links (15-minute TTL, single-use, consumed atomically).
- On successful consumption, issues a per-device session token (opaque, server-side revocable list).
- Verifies email changes by requiring a confirm-link on the new address before commit; the old address continues to work until the new one is verified.
- Stores `email` exactly once, encrypted at rest. All inter-service references are the synthetic account UUID, never the email.
- Rate-limits magic-link issuance per email.

**State service**
- Owns the canonical aviary record: per-bird identity, name, species, personality vector, current mood, current pose/state, call timing state, presence accounting.
- Owns the append-only interaction event log per account.
- Runs the simulation tick on a scheduled cadence (default 60s, calibratable).
- Serves state snapshots to clients (read-only; the client never mutates state).
- Serves visit-invitation links (one-time, 30-day TTL, revocable).
- Computes the read-only "visit" view of a host's aviary for a visitor; the visitor's session does not write events.
- Manages soft-then-hard deletion.

**Notifier**
- Transactional email only: magic links, visit invitations, account-export download links.
- No marketing, no "we miss you" emails, no "your friend visited!" emails unless the host has explicitly opted in (and even then, only for the specific friend they have on the toggle).
- Cannot reach the per-bird interaction log; it has only the account UUID and the email to send to.

**BFF / API gateway**
- TLS termination, session-token validation, request shaping, response shaping.
- Strips any field that would expose a personality vector numerically before responding to the client (defense in depth; the state service also avoids emitting it).

**Client (web only)**
- Loads minimal initial JS.
- Pulls state snapshot, renders immediately, interpolates between snapshots.
- Synthesizes calls locally from per-bird grammar + timing cues in the snapshot.
- Reports presence pings (visibility + focus + activity conjunction) and interaction events.
- Renders the field notebook from server-authored entries (delivered with the snapshot, paginated by entry-id).
- Does not own any state that has to persist across reloads beyond a thin render cache.

### 2.3 Client / server split — the boundary that matters

The cleanest way to state it: **the server is the only thing that knows what a bird is doing; the client is the only thing that knows what a bird looks like.**

The server schedules and computes state transitions (mood timers, drift deltas, call-timing cues, greeting decisions). It emits a snapshot that says "Pip is in mood=curious, vocal-timer=12.3s, focus=front-perch, body-shuffle-pose=preen-3, greeting-class=B; this is snapshot N with these pending motion deltas." The client renders.

The client never decides what mood a bird is in. It interpolates the visual representation between two snapshots and applies the motion deltas. Mood timers run on the server; the client just renders the current value.

### 2.4 Render pipeline boundary

The rendering surface lives entirely in the client. The protocol between server and client is small, finite, and shape-stable: a snapshot is `{account_id, snapshot_id, server_time, birds: [{bird_id, name, species, mood, pose, perch_zone, motion_state, call_timing, focus_target, ...}], weather, lighting, day_phase, pending_events: [...], notebook_window: [...]}`. Personality vectors are deliberately **not** in this payload; the client gets mood (which is user-visible, naturalistic) but not the vector.

---

## 3. Data model

### 3.1 Identity and account

```
Account {
  account_uuid           UUID, primary identifier
  email_encrypted        bytes, only PII in the system
  email_verified         bool
  pending_email          text | null
  status                 enum(active, soft_deleted)
  soft_delete_at         timestamp | null
  created_at             timestamp
}

Session {
  session_id             UUID
  account_uuid           UUID
  device_label           text (user-given; "macbook — chrome")
  created_at             timestamp
  last_seen_at           timestamp
  revoked_at             timestamp | null
}

MagicLink {
  link_id                UUID
  account_uuid_or_email  text (email for issuance, UUID after consumption)
  purpose                enum(signin, change_email_confirm, visit_invite)
  target_id              UUID | null (e.g., visit_invite_id)
  expires_at             timestamp
  consumed_at            timestamp | null
}
```

### 3.2 Aviary canonical state

```
Aviary {
  account_uuid           UUID, PK
  current_server_time    timestamp
  weather_state          enum(clear, light_rain, soft_wind, ...)
  weather_started_at     timestamp
  day_phase              enum(morning, midday, evening, night) (derived; persisted for tick determinism)
  lighting_state         enum(day, settled_evening, ...)
  birds                  list[BirdState]   -- 2..7 entries
  visit_invites          list[VisitInvite]
  notebook_entries       list[NotebookEntry]
  schema_version         int               -- for future migrations
}

BirdState {
  bird_id                UUID              -- stable across rename, species-pool changes, migrations
  account_uuid           UUID
  name                   text              -- user-assigned, renameable
  species_id             text              -- references a fixed SpeciesDef
  personality_vector     {
    boldness             float (0..1, hidden)
    social_warmth        float (0..1, hidden)
    vocal_frequency      float (0..1, hidden)
    plumage_saturation   float (0..1, hidden)
    curiosity            float (0..1, hidden)
  }
  mood                   enum(wary, content, curious, drowsy, alert, settled)
  mood_entered_at        timestamp
  perch_zone             enum(front, middle, back)
  body_state             enum(perched, preening, scanning, head_tilt, fluffed, calling, settled_sleep, ...)
  motion_phase           float (0..1)      -- within current body_state pose cycle
  call_timing            {
    next_call_at         timestamp
    call_class           enum(short, two_note, soft_trill, ...)
    call_volume_hint     float
  }
  last_greeted_at        timestamp | null
  greeting_class_today   enum | null       -- once per session / per day
  created_at             timestamp
}
```

The personality vector is read only by the simulation tick. The state service has a typed interface for vector mutation that is reachable only from the tick module; client-facing snapshot serialization omits the vector entirely.

### 3.3 Event log (append-only)

```
Event {
  event_id               UUID (ULID, monotonic)
  account_uuid           UUID
  bird_id                UUID | null
  type                   enum(presence_ping, presence_end, listen_in_start, listen_in_end,
                             offer_made, offer_reacted, settle, return_greeting,
                             visibility_change, focus_change, weather_tick_consumed)
  client_event_time      timestamp          -- when the client emitted
  server_received_at     timestamp          -- when the server ingested
  payload                jsonb              -- event-specific small payload
}
```

The log is append-only by API contract and by DB constraint (no UPDATE/DELETE grants for the event role). The tick reads the log; it does not mutate individual events.

### 3.4 Notebook

```
NotebookEntry {
  entry_id               UUID
  account_uuid           UUID
  authored_at            timestamp
  prose                  text               -- lowercase, present-tense, naturalist
  referenced_bird_ids    list[UUID]
  schema_version         int
}
```

The notebook is server-authored and delivered read-only to clients. The client cannot create or modify entries; it scrolls a windowed list.

### 3.5 Visit

```
VisitInvite {
  invite_id              UUID
  account_uuid           UUID (host)
  visitor_email_encrypted bytes
  status                 enum(pending, accepted, revoked, expired)
  created_at             timestamp
  expires_at             timestamp
  accepted_session_id    UUID | null
  revoked_at             timestamp | null
}

VisitSession {
  visit_session_id       UUID
  invite_id              UUID
  visitor_account_uuid   UUID | null         -- null if visitor is not signed in (one-time link)
  started_at             timestamp
  last_snapshot_at       timestamp
  ended_at               timestamp | null    -- by revocation, expiry, or visitor tab close
}
```

The visitor's session reads the host's state snapshots; it never writes events against the host's event log. The state service enforces this at the API surface.

### 3.6 Species pool

A small static reference table (~6 entries) defining the species a starter or newly-added bird can be. Each entry has a visual silhouette, default plumage palette, call-grammar motif library (motif IDs, not audio), default trait seeds. Lives in code/config, not in the database; reference data, not user data.

---

## 4. API surface

All endpoints are HTTPS, JSON, session-token authenticated except where noted. Snapshots are server-time-stamped and versioned.

### 4.1 Auth (auth service)

- `POST /auth/magic-link` — body: `{email}`. Always returns 200 (does not leak account existence); rate-limited per email.
- `GET /auth/consume?token=...&purpose=...` — consumes magic link, returns 302 to the originating app with a new session cookie set.
- `POST /auth/session/revoke` — body: `{session_id}`. Revokes the specified session (other sessions unaffected).
- `GET /auth/sessions` — list active sessions for the account.
- `POST /auth/email/change/start` — body: `{new_email}`. Sends a confirm link to the new address.
- `POST /auth/email/change/confirm` — consumes the confirm link; old email continues to work until this completes.
- `POST /auth/account/export` — kicks off export, emails a download link.
- `POST /auth/account/delete` — soft-deletes; recoverable for 30 days.
- `POST /auth/account/restore` — cancels a soft-delete in progress.

### 4.2 Aviary (state service)

- `GET /aviary/snapshot` — returns the current canonical snapshot. Response omits personality vectors. Includes `server_time`, `pending_events` (interpolation hints for the client), `notebook_window` (recent entries).
- `GET /aviary/snapshot?since=snapshot_id` — delta snapshot. If the snapshot hasn't changed, returns 304.
- `POST /aviary/events` — appends an interaction event. Body: a single event or small batch. Server validates schema, timestamps, monotonicity of event_id, and that the event type is one of the allowed set. Returns 202 (accepted, tick will consume).
- `POST /aviary/settle` — convenience endpoint that emits a `settle` event. Idempotent within the presence-window.
- `POST /aviary/offer` — body: `{offer_type: seed|song_fragment|still_pool, target_bird_id?}`; emits `offer_made`. The reaction is computed by the tick, not by this endpoint.
- `GET /aviary/notebook?before=entry_id&limit=20` — paginated entries, oldest-first; the client can scroll indefinitely.
- `PATCH /aviary/bird/{bird_id}/name` — body: `{name}`. Validates uniqueness within the aviary (case-insensitive; we don't allow two birds named "Pip").
- `GET /aviary/accessibility` and `PATCH /aviary/accessibility` — per-account accessibility prefs (captioning, reduced-motion override, narration pacing).

### 4.3 Visit (state service)

- `POST /aviary/visit/invite` — body: `{visitor_email}`. Sends invite email. Returns invite_id for the host's records.
- `GET /aviary/visit/invites` — host's outstanding and historical invites.
- `POST /aviary/visit/invite/{invite_id}/revoke` — host revokes. Active visitor sessions terminate at the next snapshot poll.
- `GET /visit/{invite_token}` — visitor opens the link. Returns a read-only "visit" view of the host's aviary (state service enforces: no event writes, no `offer`/`settle`/`listen-in` endpoints reachable from a visit session).
- `GET /aviary/visit/log` — host's visit log (who visited, when, duration). Reachable on demand from account settings.

Visit sessions are gated by a separate token type so a stolen session token cannot be used to read another account's aviary. The state service refuses to accept interaction events from a visit session.

### 4.4 Operational endpoints (not user-facing)

- `GET /healthz`, `GET /readyz` — for the platform.
- `GET /metrics` — Prometheus format; the metric set is described in §10. Critically, none of the per-bird, per-account metrics are exposed here.

### 4.5 What is NOT in the API

- No `GET /bird/{id}/personality` endpoint. There is no shape that would return a personality vector, even to a debug client.
- No `POST /bird/{id}/personality` endpoint. The tick is the only writer.
- No `GET /stats` endpoint. No leaderboard, no aggregate, no "how is my bird doing" numeric view.
- No "send push" endpoint from any service. There is no push pipeline at v1.

---

## 5. Simulation engine design

### 5.1 Tick loop

A scheduler (cron-style with a per-account jitter window) drives a tick per account every ~60 seconds, with ±5s jitter to prevent thundering-herd. The tick is implemented as a transactional function with a small advisory lock per account, so a slow tick on account A never blocks account B.

The tick steps:

1. Read the append-only event log entries since the last tick, in event-id order.
2. Aggregate presence-time in the window (sum the duration of presence-events whose end timestamp falls within the window).
3. Update personality vectors by computing deltas from the aggregated inputs.
4. Transition moods based on (a) per-bird mood timer expiry, (b) time-of-day in the user's timezone, (c) weather, (d) nearby bird mood contagion, (e) any user-interaction events in the window.
5. Advance per-bird call timing (next_call_at, call_class).
6. Run the bird-to-bird interaction rules (chorus detection, call-response pairings, mood contagion).
7. Decide whether the next 24h window will produce a notebook entry; if so, generate the entry via a small server-side template engine seeded by current state. The engine is a deterministic picker over ~30 prose templates per surface, parameterized by bird names, mood, weather, and recent notable events.
8. Persist the new canonical state.
9. Release the lock.

The tick is a pure function of (state, events-since-last-tick) within the tick window. Given the same input, it produces the same output. This is what lets the simulation be debugged from logs.

### 5.2 Drift function (the load-bearing piece)

Drift is implemented as a low-pass filter on the per-trait accumulated input signal. A simplified model:

```
trait_input_t = (alpha_presence * presence_time_in_window
                + alpha_listen * listen_in_seconds_for_this_bird
                + alpha_offer_accept * offer_accept_count_for_this_bird
                + alpha_offer_near * offer_near_count_for_this_bird)
trait_t = trait_{t-1} + (gain_trait * trait_input_t)
```

With `trait_t` clamped to [0, 1] on the upper end only — **never clamped downward**. A trait can saturate; it cannot decay from neglect. The single failure mode we are defending against is a symmetric drift model, so the asymmetric clamp is the literal code.

Calibration target (per PRD): measurable drift in instruments after ~1 week of regular visits, visible-to-the-user after ~3 weeks. Translates to roughly `gain_trait` such that a daily ~10-minute attentive visit produces ~0.005–0.01 trait change per week, and a daily ~60-minute attentive visit produces ~0.02–0.03 per week. These are starting values for the calibration harness, not the final tuning.

Drift on `plumage_saturation` follows the same rule, with the visual effect that plumage richens over time. The rendering maps the scalar to a small set of plumage "registers" (4–5 visual treatments) rather than continuous interpolation, both for visual coherence and for keeping the trait scalar in instrument-testable territory.

### 5.3 Mood transition rules

Mood is a small enum per bird. Transitions are computed by the tick as:

```
new_mood = weighted_choice(
  self.personality_adjustments,
  time_of_day_bias,
  weather_bias,
  recent_interaction_bias,
  nearby_bird_mood_contagion
)
```

`nearby_bird_mood_contagion` is the load-bearing piece for "the aviary is a small social system": if Bird A is in `wary` and Bird B is within audio reach, Bird B has a non-zero chance of also shifting toward `wary` (modulated by B's own `boldness`). A high-`social_warmth` bird counter-acts this by calling, which can pull neighbors back toward `content`.

### 5.4 Call grammar runtime

Each species has a small motif library (3–6 motif IDs). A per-bird call is generated by:

1. The server, on tick, decides *when* the bird will next call (Poisson-ish with `vocal_frequency` as the rate parameter, modulated by mood and weather).
2. The server emits a `call_timing` cue in the snapshot: `{next_call_at, call_class, call_volume_hint, motif_seed}`.
3. The client, when `next_call_at` arrives, uses the `motif_seed` to render the call procedurally. The seed is deterministic for the snapshot, but the snapshot rotates the seed over time so the user does not hear the same call twice.

Two procedural calls played simultaneously are mixed by the WebAudio graph; phase-canceling artifacts (the giveaway of recorded-loop stacking) are absent by construction.

### 5.5 Return-greeting decision

On the first snapshot pull in a new visibility-window (page open or tab focused after long idle), the server computes a "greeting event" by:

1. Determining absence length from the last `presence_end` (or `soft_delete` boundary).
2. Selecting which bird greets first: weighted by `social_warmth` * `boldness`, with a small random component so it isn't always the boldest bird.
3. Selecting a greeting class from `{glance_up, two_note_call, head_tilt_step, longer_call_with_response}` based on absence length and the selected bird's mood.
4. Setting `greeting_class_today` so this bird is not chosen again for the same session.

The greeting class is delivered in the snapshot as a pending motion delta for the client to render. The actual visual rendering is client-side; the server only decides the class.

### 5.6 Notebook entry generation

A separate small service (or module within the state service) runs at a low frequency (roughly once every few days per account, calibrated to preserve sparsity) and decides:

- Should an entry be written now? (A function of: time since last entry, recent noteworthy events in the event log, current state.)
- Which template? (Day-of-week opener, weather observation, greeting-order observation, quiet-stretch observation, post-offer observation, bird-specific call observation, etc.)
- Which parameters? (Bird names, weather, mood.)

The templates are short, lowercase, present-tense, bird-named, specific to the moment — and they are reviewed by a human before they ship. Notebook generation is server-side, server-authoritative; the client never composes a notebook entry.

The pacing rule (roughly one entry every few days for a regular visitor) is enforced by a min-gap parameter (default 48h) plus a "worth observing" predicate that filters out trivial states.

### 5.7 Calibration harness

A test harness plays scripted user behavior (presence sequences with specific timing) against a real state-service instance, captures personality-vector deltas over a compressed-time simulated week, and asserts:

- After a week of regular visits, at least one trait has changed by ≥ 0.01.
- After a week of *no* visits, no trait has decreased from its starting value.
- Mood persists across an empty-tick window (no reset to neutral).

This is a real CI test, not a guideline.

---

## 6. Sync model

### 6.1 The whole point of sync

There is no client-side state to sync, because the server is the only writer. The user's aviary on their laptop and on their phone is the same aviary because both devices are reading the same record. The architecture is the feature.

### 6.2 Snapshot flow

1. Client opens the aviary, calls `GET /aviary/snapshot`.
2. Server returns the canonical snapshot with `snapshot_id`, `server_time`, and a `pending_events` list (motion deltas, greeting classes, call cues) the client should apply.
3. Client renders. From then on, the client polls:
   - On `visibilitychange` to `visible` after a hidden period.
   - On long render-frame gaps (laptop suspend detection).
   - On a low-frequency keepalive (~10s while tab is visible).
   - On user-initiated events (offer, listen-in, settle) to confirm the event was accepted.
4. Clients also subscribe to a server-sent events stream on the snapshot endpoint for instant updates when the simulation tick lands. The stream is server-time-stamped and replayable from `since=` for resilience.

### 6.3 Conflict surface

There is no personality conflict. The server is the only writer.

The only conflict-adjacent surface is when a client's request is in flight during a tick (rare): the response reflects whichever tick the server just finished. The next request gets the next snapshot. No merge, no version vector, no client-side reconciliation.

A user-triggered event (offer, settle) is a request to append to the event log. The log is append-only; the request either appends or is rejected. If rejected (rare — schema invalid, account soft-deleted, session revoked), the client receives a 4xx and surfaces a matter-of-fact message.

### 6.4 Visit sessions

A visit session is a *read-only* projection of the host's state. The state service does not allow the visit session to issue `POST /aviary/events`; that endpoint requires a host session. The visit session polls a different endpoint (`GET /visit/{invite_token}/snapshot`) which returns the same snapshot shape minus any per-account-private fields (notebook entries, visit log).

### 6.5 Account export, deletion, restore

- Export is generated server-side and emailed as a one-time download link (separate session token, single-use, 24h TTL).
- Soft delete: account status flips to `soft_deleted`; login is disabled; state service preserves the record for 30 days.
- Restore: account status flips back; no data was modified, so the aviary resumes from its last tick.
- Hard delete (after 30 days): all rows tied to the account_uuid are deleted, in this order — events, notebook entries, bird states, visit invites (cascading to visit sessions), sessions, the account record itself. Logs retain a tombstone row for audit (account_uuid, deleted_at) but no PII.

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- React 18 with Suspense for the lazy surfaces (settings, accessibility, visit-invite flow). The aviary scene itself is a single full-viewport canvas-rendered component (see below).
- TypeScript strict.
- State management: a thin client-side cache of the latest snapshot, mutated only by snapshot responses. No local copies of personality or mood. The cache is hydrated from the server and rehydrated on every snapshot pull.
- Routing: a tiny client router with two main surfaces (aviary, notebook overlay) and the settings pages code-split.

### 7.2 Rendering target

- WebGL2 for the aviary scene. A 2D context is insufficient for the parallax layers and ambient particle work; Canvas2D is too slow on a 5-year-old laptop at 60fps with two birds + ambient motion.
- The scene composition is one main canvas, with two parallax background layers composited as textures.
- Birds are procedurally generated SVGs rendered into a small atlas at startup (no per-frame allocation). A bird's pose is selected from a small library (~12 poses per species) and applied via vertex-uniforms-style uniform updates on the atlas quad; no per-bird re-uploads per frame.

### 7.3 First frame strategy

The single most important render-pipeline constraint. The plan is:

1. The HTML response ships with the aviary shell, the soft sky color, and a placeholder canvas.
2. The state snapshot is requested as a high-priority fetch; the response is small (kilobytes) and delivered from a CDN edge near the user.
3. As soon as the first snapshot arrives, the renderer positions birds at their current positions, applies their current motion-state pose, and begins rendering.
4. There is no "wake up" animation. The first frame is the aviary, mid-action. A bird that was mid-preen at snapshot time is rendered mid-preen.

If the snapshot is slow (cold cache, bad network), the user sees the quiet field. The quiet field is a single sky color and zero motion. It is not a spinner; it does not say "loading." A spinner would re-introduce the "machine is loading" affordance the product is shaped to refuse.

### 7.4 Idle micro-motion

- Birds run a small state machine of body-states (perched, preening, scanning, head_tilt, fluffed, calling, settled_sleep).
- Transitions are personality- and mood-shaped, driven by the server's `motion_phase` and `body_state` fields.
- The client interpolates pose within a body-state for smoothness.
- Idle micro-motion runs continuously; it does not pause on visibility hidden (server still ticks; the client can stop rendering to save battery, but the next snapshot will show a fully-alive scene).

### 7.5 Ambient particles

- Leaves, feathers, and the foreground branch pass-through are pure client-side ornaments. They are not part of the simulation state.
- They run at a slow randomized cadence, decoupled from the tick.
- They are skipped entirely in reduced-motion mode.

### 7.6 Top bar and chrome

- A thin top bar with: account/settings, accessibility settings, field notebook icon, offer affordance.
- Fades to near-transparent after a few seconds of cursor stillness; returns to full opacity on cursor movement or keyboard activity.
- Holds no personality vector data. Holds nothing per-bird numerically.

### 7.7 Field notebook overlay

- Slides in from the right (or bottom on narrow viewports) when the notebook icon is activated.
- Lists entries oldest-first; the client can scroll back indefinitely. Entries never get archived on the server.
- Read-only: no edit, no annotate, no delete. Even an undo on accidental dismissal is just "dismiss the overlay" — the entries themselves are immutable.

### 7.8 Reduced-motion mode

This is its own designed surface, not a fallback. In reduced-motion:

- Body-state micro-motion becomes a slow cross-fade between two poses within the same state. A preening bird is rendered in preen-pose-1, fades to preen-pose-2, back to preen-pose-1.
- Flight transitions are cross-fades between perches, not animated paths.
- Ambient leaves and feathers are removed.
- Day-to-evening color shifts remain, slowed.
- Calls still play at full quality. Birds still drift. Mood still changes. The field notebook still notices things.

The cross-fade rendering has its own aesthetic; it is not a degraded version of the standard rendering.

### 7.9 Responsive layout

- Single horizontal scene at any viewport.
- Narrow viewports compress horizontally without cropping any bird.
- Wide viewports widen perch spacing.
- Aspect ratio is preserved; no bird is ever cropped out of frame, ever.
- Touch and pointer both supported; gestures are tap-to-listen-in, no pinch-zoom, no pan.

---

## 8. Audio pipeline

### 8.1 Synthesis target

Calls are synthesized client-side from per-bird motif libraries using WebAudio. No audio files are downloaded. This is forced by the bundle budget and is also the chorus mechanic's correctness condition.

### 8.2 Synthesis chain

```
Motif (textual spec)  -->  ProceduralRenderer  -->  AudioBufferSourceNode
                                              -->  GainNode (per-bird)
                                              -->  BiquadFilter (mood shape)
                                              -->  StereoPannerNode (zone)
                                              -->  Master gain
                                              -->  AudioContext.destination
```

- `ProceduralRenderer` takes a motif ID, a `motif_seed`, and a per-call variation parameter, and produces a small `AudioBuffer` of a few seconds. The synthesis uses FM/AM modulation of simple oscillators with a smooth envelope; this is well within WebAudio's reach and avoids recorded samples.
- `motif_seed` is delivered by the server in the snapshot. The seed is rotated on each call so the user never hears the same call twice in the same way.
- A per-bird `BiquadFilter` shapes the timbre by mood: a `drowsy` bird's calls are low-passed; an `alert` bird's are slightly brightened.

### 8.3 The seven-bird cap and the chorus

The cap is empirical, not aesthetic. The audio system must keep per-bird calls individually recognizable. Two procedural calls mixed at the same time produce a real chorus; two recorded loops do not. The cap of 7 is the point at which per-bird signatures start to blur in informal listening tests. This is a design constraint the audio team owns; a perf test that asserts recognizability on a typical listener is part of the audio pipeline's CI.

### 8.4 Listen-in mix

Engaging listen-in on bird B:

- B's per-bird gain ramps up from its ambient level to a focus level over ~600ms.
- All other birds' gains ramp down to a quiet ambient level (~ -12dB) over the same window.
- Other birds never reach silence; they remain in the mix at low level.
- Disengaging reverses the ramp. Hard cuts are forbidden.

The mix change is the interaction; there is no separate "now listening" announcement.

### 8.5 WebAudio fallback

If `AudioContext` is unavailable (older browser, denied permission, hardware issue), the aviary renders in graceful silence with captions on by default. No recorded-audio fallback. The fallback is the same scene minus sound plus a persistent caption layer; it is not a different product.

### 8.6 Captions

- Generated from the same procedural call spec that drove the audio.
- Short prose: "a soft three-note rise," "a low trill, paused, low trill again," "a single sharp call from the back perch."
- Caption text fades in with the call and out shortly after.
- Captions can be enabled even when audio is enabled (noisy environment, hearing differences).

### 8.7 Performance constraints on the audio pipeline

- Per-call `AudioBuffer` is allocated into a small object pool; allocations are reused.
- Audio context is created on first user gesture (browser autoplay policy) and never recreated.
- The renderer is a Web Worker that pre-bakes calls for the next few seconds based on the snapshot's call-timing cues, so the audio thread is never blocked on synthesis during playback.

---

## 9. Accessibility surfaces

The PRD's stance is that accessibility is a designed surface, not a checklist. The plan treats it that way in three specific ways.

### 9.1 Screen-reader narration

- A live region (`aria-live="polite"`) receives a slow stream of prose.
- The prose is generated by the same template engine the notebook uses, parameterized by current state, with a target cadence of one update per 30–60 seconds at idle, faster only on user-initiated events.
- Voice continuity: the narration uses the same naturalist voice, lowercase, present-tense, specific. A screen-reader user moving between the aviary surface and the notebook should hear the same product.
- The prose is generated server-side and shipped with the snapshot (small, kilobytes). Generation is gated by a `narration_pacing` accessibility setting (the user can slow it down or speed it up, never off — turning it off would re-introduce the strip-it-out fallback we are refusing).

### 9.2 Reduced-motion mode (already specified in §7.8)

Its own designed surface. Not a fallback. The cross-fade rendering is its own aesthetic, not a degraded one.

### 9.3 Captions (already specified in §8.6)

Generated from the procedural spec. Always available, opt-in for sighted users with audio on, default-on for users without audio.

### 9.4 Keyboard navigation

- All top-bar items are in the tab order.
- Tab into the aviary scene focuses the first bird (perch zone order is `front` first).
- Arrow keys move focus between birds.
- Enter on a focused bird engages listen-in.
- Escape disengages listen-in.
- Top-bar shortcuts: `n` opens the notebook, `o` opens the offer affordance, `s` triggers settle, `?` opens the accessibility settings.
- Focus indicators are visible against any aviary background. A soft, high-contrast outline; the exact treatment is in the design system, but the design constraint is "reads against both bright midday and dim night aviaries."

### 9.5 Contrast

All user-copy text passes WCAG AA at minimum. The aviary scene itself carries no user copy, so the contrast constraint applies primarily to the top bar, settings, account surfaces, error surfaces, and the visual narration surface (when it is displayed visually, which it is not in v1 — narration is screen-reader only). The design system specifies actual ratios per surface; that document is a peer to this plan.

### 9.6 Accessibility settings persistence

Stored on the account (server-side), keyed by `account_uuid`. The client sends the user's settings on every snapshot request and receives a hint about which accessibility surfaces to enable. This means a reduced-motion user gets the same experience on every device they sign into, without re-configuring.

### 9.7 What is explicitly not in accessibility

- A toggle to expose the personality vector numerically. This is not an accessibility affordance; it would violate the rule.
- A toggle to disable mood transitions. Mood is part of the aviary.
- A "no ambient motion" toggle that turns the aviary into a static frame. The user can opt into reduced-motion, which is its own designed surface; they cannot opt into a degraded one.

---

## 10. Performance budgets and observability

### 10.1 The budgets, with their affective interpretation

| Budget | Threshold | Affective meaning if missed |
|---|---|---|
| Initial JS bundle | <2MB gzipped | The aviary cannot appear mid-action in the time the user is willing to wait. |
| Time-to-first-bird | <500ms on mid-tier mobile over 4G | The user sees a load. The central conceit collapses. |
| Idle motion frame rate | 60fps on a 5-year-old laptop | The aviary stops feeling alive and starts feeling laggy. |
| Memory growth | None over 30 min | The session degrades; the bird engine's felt-aliveness is ruined by stutter. |
| Simulation tick latency | p99 <5s | The aviary "runs slow"; the user notices the gap between action and persistence. |
| Magic-link email delivery | <60s p95 | The user has to retry; the calm sign-in surface breaks. |

These are not aspirational. They are enforced in CI as tests where testable, and instrumented in production where not. A budget miss blocks deploy.

### 10.2 Measurement

A synthetic perf fleet (a few headless browser instances in common geographies — us-east, eu-west, ap-southeast) runs scripted navigations on a schedule and reports:

- Time to first byte
- Time to first bird visible (the moment the first bird is composited, measured via Performance Observer)
- Time to first call audible
- 30-min frame rate percentiles
- 30-min memory growth
- Audio context error count

The fleet is anonymized (no per-account dimension) and aggregate-only.

### 10.3 Real User Monitoring

- Page load timings.
- First-bird-render timings.
- Render-frame timings (sampled, not every frame).
- Audio context errors.
- Simulation-tick ingest latency (the time from server tick completion to snapshot response to client).

No per-bird state, no per-account interaction history, no identity, no email. The metric labels never include `bird_id`, `account_uuid`, or any field that could be correlated with the simulation database.

### 10.4 Error budget

- Tick latency p99 alarm at 5s. Catches degradation early.
- Snapshot delivery p99 alarm at 1.5s. The keepalive cadence is 10s; a p99 above 1.5s would mean the user is waiting on the next snapshot.
- Audio context creation failure rate >1% alarm. Indicates a regression in browser support handling.

### 10.5 What we deliberately do not measure

- Per-bird state changes (e.g., "Pip's mood shifted to content 14 times this week"). The privacy boundary forbids it; the user-facing surface would invite it; both are answered with the same rule.
- Per-account engagement (visit counts, average session length per account). The aggregate session-duration histogram is allowed; per-account tracking is not.
- Notebook entry engagement (which entries the user expands, which they read). Notebook is read-only; if the user opens an entry, that's the read.

---

## 11. Rollout

### 11.1 Build phases (not dates — these are dependencies)

1. **Engine core**: state service, append-only event log, tick loop, drift function, mood transitions. No UI. Verified by the calibration harness in §5.7.
2. **Snapshot API + minimal client**: a single bird, no interactions, no day/night, no offers, no notebook. Just a bird rendered from a snapshot, with the tick advancing. This phase is the "does the central conceit work" gate.
3. **Two-bird aviary, day/night, idle motion**: the visible surface that proves the first-frame strategy.
4. **Procedural audio**: synthesis pipeline, listen-in, call cues from snapshots. This is the highest-risk subsystem; build it earlier than feels comfortable.
5. **Interactions**: offer, settle, field notebook.
6. **Accounts and sync**: magic-link auth, session model, multi-device, multi-tab. The architecture makes this small if §1–§5 were done right; large if they were not.
7. **Accessibility**: narration, reduced-motion, captions, keyboard nav. Built in parallel from day one; the surfaces are designed, not retrofitted.
8. **Visits**: invite issuance, read-only visitor view, revocation.
9. **Account export, soft-delete, restore**: small, but they touch the data model in ways that are expensive to retrofit.
10. **Calibration pass**: real users on real data, with the calibration harness and the perf fleet running, tuning drift gain, mood weights, tick cadence, presence window.

### 11.2 The two-bird / seven-bird ramp

- v1 launches at 2 starter birds for every new account, cap of 7. The cap is enforced server-side; the client never offers an "add an 8th bird" affordance.
- "Adding a third bird" is paced by aviary age (per PRD). The trigger logic lives in the state service: a new species offer is computed at the appropriate aviary-age interval and made available to the user via a small, opt-in flow. The user accepts; the new bird is added. The pacing constants are part of the calibration pass.
- Adding birds is never a response to user behavior (no "you've visited enough to earn a third bird" mechanic). The rule is in §1.1: aviary age, not visit count.

### 11.3 Day-one instrumentation

- The synthetic perf fleet from day one.
- Aggregate-only RUM from day one.
- The privacy boundary enforced from day one, including the IAM-scope separation between the simulation DB and any analytics warehouse. Adding it later is the kind of thing that does not get added.
- A small server-side audit log of session revocations, magic-link issuances, visit revocations, account deletions. Useful for support; deliberately not the per-bird interaction log.

### 11.4 What we ship on day one and not before

- The two-bird aviary, day/night, idle motion, calls, return-greeting, listen-in, offer, settle, field notebook, accessibility surfaces, account/sync, visits, export/delete, perf fleet, aggregate RUM. Everything in §1.1.
- Nothing in §1.2.

---

## 12. Risks and mitigations

The risks below are the ones the PRD itself names, plus a few the implementation will surface.

### 12.1 Drift calibration

- **Risk**: drift too fast (Tamagotchi) or too slow (screensaver). Both are silent failures: no unit test catches them; only the user notices.
- **Mitigation**: the calibration harness in §5.7 runs in CI. Real-user feedback loop: an internal dogfood period of 4–6 weeks before public launch, with the perf fleet reporting on the same metrics a real user would experience. The asymmetric clamp is unit-tested explicitly ("no trait decreases from its starting value over N simulated weeks of neglect").

### 12.2 Sync correctness

- **Risk**: any code path that lets a client write personality, or any path that lets a server "merge" two divergent reads, would let drift history silently delete. This is the failure mode the PRD calls out by name.
- **Mitigation**: the typed interface for vector mutation is reachable only from the tick module; linter and code-review rules enforce it; the API surface in §4 has no personality-write endpoint; the client-side cache holds no personality; the snapshot response omits the vector; the state service's serialization omits the vector. Defense in depth, every layer.

### 12.3 Audio uncanniness

- **Risk**: procedural calls that don't feel alive (mechanical FM, clicking artifacts); chorus artifacts from synthesis edges; the seven-bird recognizability ceiling dropping as we tune.
- **Mitigation**: the audio pipeline is built in §11 phase 4, earlier than feels comfortable, so the team has the time to iterate. Recognizability is tested in CI with a small listening panel — the test is "is each bird's call signature distinguishable to a typical listener at 7 birds?" If the answer degrades, the cap moves down or the synthesis is iterated. The audio team owns this and has the time to do it.

### 12.4 Accessibility regressions

- **Risk**: an a11y surface degrades silently because the team is focused on the visual product. The reduced-motion fallback, the static narration, the missing captions, the broken keyboard nav.
- **Mitigation**: a11y is in every phase of §11, not a phase of its own. CI runs axe-core and a custom narrator-prose check (asserts that the narration is in the right voice, lowercase, present-tense, not announcement-style). The reduced-motion surface is treated as its own designed product, not a fallback — same level of design scrutiny as the main surface.

### 12.5 The "just one toast" temptation

- **Risk**: a contributor adds a "Welcome back!" toast because it is conventional. Or a streak counter. Or a calendar of green dots. Or a "your friend visited!" badge. Or a notebook entry that reads "you visited every day this week." Every one of these is reachable from a reasonable argument and would re-introduce the engagement-rotation the product is shaped to refuse.
- **Mitigation**: §0 and §1.3 are the constitution. Code review rules. The PR template asks explicitly "does this surface observe the user or the aviary? If the user, reject." This is the kind of rule that survives only if it is named loudly and often; this plan names it three times and the PRD names it more.

### 12.6 The "let's expose a debug stats panel" temptation

- **Risk**: a contributor adds a "show me how my bird is doing" surface — even gated behind a feature flag, even at internal-only — that exposes personality vector values. The moment the values are visible to anyone, the bird becomes a number, and the relationship the product is asking the user to form starts to drag against the new affordance.
- **Mitigation**: the API surface in §4 has no personality-read endpoint. The client cache holds no personality. The state service's snapshot serializer omits the vector. A debug surface that wanted to display personality would have to be added at every layer. This is the protection: there is no debug surface without explicitly breaking the architecture. If a future contributor wants one, the cost of the request becomes a forcing function for the conversation.

### 12.7 The "let's just last-write-wins this" temptation on concurrent writes

- **Risk**: a contributor reaches for last-write-wins to handle a rare concurrent case. The PRD calls out exactly the failure mode this would produce: drift silently lost between devices, the user noticing nothing until they feel the bird drifting more slowly than they expected.
- **Mitigation**: the API surface has no personality-write endpoint. The event log is append-only; the tick consumes it in order. There is no merge step. There is no last-write-wins code to write. The protection is structural: the failure mode is unreachable, not just discouraged.

### 12.8 Time-of-day / timezone correctness

- **Risk**: a user's morning is the server's afternoon, breaking the day/night cycle and the mood transitions tied to local time.
- **Mitigation**: account has a `tz` field set from the browser's `Intl.DateTimeFormat().resolvedOptions().timeZone` on first sign-in, with a settings surface for the user to override. The tick computes time-of-day from the account's tz, not the server's. Day-phase transitions are tick-deterministic given the tz.

### 12.9 Privacy boundary erosion

- **Risk**: a future contributor adds a "drift-health" analytics job that reads the simulation database. Or a "popular bird species" report. Or a model-training pipeline that uses per-bird data. Each is anodyne in isolation; collectively they are the privacy commitment undone.
- **Mitigation**: the simulation database is on a separate IAM scope that the analytics warehouse cannot read. The privacy policy in account settings names the aggregate categories and explicitly excludes per-bird state. The boundary is enforced at the data-pipeline level, not the policy level — and the §11 day-one instrumentation includes the IAM-scope separation, because adding it later is the kind of thing that does not get added.

### 12.10 The empty-aviary moment

- **Risk**: between adoption flow and the first bird appearing, the aviary is empty for a beat. A spinner in that moment would re-introduce the "machine is loading" affordance.
- **Mitigation**: the empty-aviary state is the quiet field, same as the loading state. The first bird enters with a soft fly-in to its starting perch. The user never sees an empty aviary again from that point forward.

---

## 13. Open questions / calibration sites

Items the PRD leaves to implementation. The plan picks defensible defaults and flags them as calibration sites.

- **Presence activity window**: PRD says "a few minutes." Plan defaults to 4 minutes. Calibrate in dogfood.
- **Tick cadence**: PRD says "~once per minute." Plan defaults to 60s with ±5s jitter. Calibrate in dogfood.
- **Drift gain constants**: starting values in §5.2. Calibrate via the harness in §5.7.
- **Notebook pacing**: PRD says "roughly one entry every few days." Plan defaults to a 48-hour min-gap plus a "worth observing" predicate. Calibrate in dogfood.
- **Caption timing**: PRD does not specify fade duration. Plan defaults to a 300ms fade in, 800ms fade out. Calibrate in audio review.
- **Listen-in ramp duration**: PRD says "slow." Plan defaults to 600ms. Calibrate in audio review.
- **Bird-to-bird mood contagion weight**: PRD says "non-zero" but does not pin. Plan defaults to a small weight with a high-`social_warmth` counter-action. Calibrate via the harness.
- **Starter species selection**: PRD says the system picks two from the pool. The selection is a small weighted random with a "feel coherent together" predicate; the exact weighting is a designer call, owned by the visual / species team.
- **Three-bird pacing curve**: PRD says age-tied, not visit-tied. Plan defaults to a 3-month aviary age for the first third-bird offer. The pacing curve is a designer call.

---

## 14. Definition of done (v1)

A reviewer should be able to check the following:

- [ ] Two starter birds are introduced, not selected from a catalog.
- [ ] Personality vectors are server-authoritative, client-invisible, never serialized to the client.
- [ ] Drift is asymmetric (no trait decreases on neglect) and verified in CI.
- [ ] Server tick advances canonical state at ~60s cadence, advancing during absence.
- [ ] Multi-device sync is a property of the architecture, not a feature.
- [ ] The first frame is mid-action, with no "wake up" animation, no spinner.
- [ ] Calls are procedural, synthesized client-side, no recorded audio.
- [ ] The seven-bird cap is enforced server-side and not reachable from the UI.
- [ ] Listen-in is a slow mix re-balance; other birds never go silent.
- [ ] Field notebook is read-only, lowercase, present-tense, sparse.
- [ ] No "Welcome back!" toast, no streak counter, no calendar of green dots, no "your friend visited!" badge.
- [ ] No personality-vector surface anywhere in the UI.
- [ ] No Tamagotchi-style distress surfaces; no hunger meter; no death.
- [ ] No public discovery, no leaderboards, no "show-off" rendering for visitors.
- [ ] Visits are read-only ambient; visitor sessions cannot write events.
- [ ] Magic links expire in 15 minutes and are single-use.
- [ ] Soft delete with 30-day recovery; hard delete after the window.
- [ ] Email stored once, encrypted; everything else references the synthetic UUID.
- [ ] Account export available; visit log available; visit revocation immediate.
- [ ] Screen-reader narration is in the same voice as the field notebook, with a 30–60s idle cadence.
- [ ] Reduced-motion mode is its own designed surface, not a fallback.
- [ ] Captions are generated from the procedural spec, in the same voice.
- [ ] Keyboard navigation reaches every interactive surface; focus indicators are visible against any aviary state.
- [ ] WCAG AA contrast on all user-copy text.
- [ ] Initial JS bundle <2MB gzipped, time-to-first-bird <500ms on mid-tier mobile over 4G, 60fps idle on a 5-year-old laptop, no memory growth over 30 min.
- [ ] Simulation tick p99 <5s, snapshot delivery p99 <1.5s.
- [ ] Aggregate RUM is on; per-bird / per-account metrics are not.
- [ ] Privacy boundary enforced at the IAM level, not the policy level.

End of plan.
