# Pocket Aviary — Implementation Plan (v1)

> **Phase 1 plan.** Produced from the full PRD (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals). This plan translates the spec into an executable engineering roadmap for a frontier team. It does not implement the product; it tells another engineering team how to do it without further clarification.

---

## 1. Scope — what is and isn't in v1

### In scope (v1 ships all of this at launch)

- **Aviary** — single horizontal scene per account; two starter birds; cap at seven; day/night cycle locked to user's local timezone; ambient weather (rain, soft wind, a few times per week); three perch zones (front, middle, back) with bird positioning driven by mood/personality, not any user drag-to-place affordance.
- **Birds** — species pool of ~6 with distinct silhouettes, plumage palettes, and call-grammar motif libraries; each bird carries a stable synthetic UUID identity, user-assigned name, hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood state, and drift history. Birds are adopted via aviary-age gating (not visit count, not score); the first two birds are selected by the system from the pool, not chosen by the user from a catalog.
- **Server-side simulation** — canonical aviary state advances on a slow tick (~1/min) regardless of client connectivity. Only the server writes personality vectors; clients write interaction events to an append-only event log; the tick consumes the log and updates drift/mood.
- **Personality drift** — monotonic toward expressive (up only on positive presence; never down on neglect). Calibrated for measurable instrument drift at ~1 week, visible user drift at ~3 weeks. Primary input: presence-time. Secondary: listen-in, offers, settle. No symmetric negative drift.
- **Mood** — fast-timescale enumerated state per bird (wary, content, curious, drowsy, alert; finalized in implementation). Resets on a daily-ish cadence; modulated by recent interactions, time of day, ambient events, personality. Persists across sessions — no reset to neutral on tab open.
- **Interactions** — return-greeting (one bird notices within 1–2s, procedurally varied by absence length, boldness, mood; never identical twice), listen-in (click/tap/keyboard focus on a bird; its call rises in mix, others quiet to ambient, never silent; gradual ramp, not a hard cut), offer (seed, song fragment, still pool; reachable from top-bar affordance; per-bird cooldown ~minutes), settle (opt-in evening lighting gesture from top bar; 5s undo window), field notebook (auto-generated naturalist-prose observation log; read-only; ~one entry every few days; never generic event logs).
- **Presence accounting** — three-condition conjunction: `document.visibilityState === "visible"` AND `document.hasFocus()` AND a pointermove/keypress within a calibratable window (~few minutes). Settle and tab-close both terminate presence identically; no penalty for tab-close. Presence-time is stored server-side as the dominant drift input.
- **Accounts** — single-user, magic-link email sign-in; synthetic UUID account ID (email never used as identifier anywhere else); per-device revocable session tokens; email change requires new-address verification; account export (JSON snapshot emailed); account deletion (soft 30d then hard).
- **Multi-device sync** — property of the single-canonical-server architecture, not a separate feature. Both clients pull the same snapshot; no client-to-client sync; no merge reconciliation.
- **Visit invitations** — host invites visitor by email; visitor gets a one-time read-only ambient link; no co-presence, no chat, no avatars, no comments, no public discovery; invites expire 30d if unused; visitor presence does not drift host's birds; host can revoke anytime; visit log reachable from account settings; visit notifications off by default, opt-in toggle available.
- **Accessibility** — screen-reader narration in naturalist prose (~1 update per 30–60s at idle, priority bump for user-initiated events); reduced-motion mode as its own designed surface (cross-fades between still poses, not "animations off"); call captions from procedural grammar; WCAG AA contrast on all user-copy text; full keyboard navigation (Tab through top bar, arrow keys between birds, Enter for listen-in, Esc to exit).
- **Audio** — procedural call synthesis client-side via WebAudio; per-bird recognizable call signatures across mood/drift; chorus mixing from real-time procedural calls, not stacked loops; listen-in mix decay; WebAudio-unavailable fallback = silence with captions on by default, no recorded-audio fallback.
- **Performance** — initial JS bundle <2MB gzipped; time-to-first-bird <500ms on mid-tier mobile over 4G; 60fps idle motion on 5-year-old laptop; no memory growth over 30-minute session; CI test for memory stability; aggregate RUM and synthetic perf checks; simulation-tick p99 alarm at 5s.
- **Browser support** — last two majors of Chrome, Safari, Firefox, Edge. Unsupported browsers get a matter-of-fact notice.

### Explicitly out of scope (from non_goals.md)

- Native mobile app (iOS/Android)
- Gamification of any form (achievements, streaks, scores, badges, counters, leaderboards, XP, ranks, green-dot calendars, visit-frequency indicators anywhere in the product)
- Tamagotchi-style mechanics (birds do not die, get hungry, show distress, or have decaying happiness meters)
- Social network surfaces (profiles, follows, feeds, public discovery, friend-of-friend chains, comments, rankings)
- Payments or billing flows
- Shared/multi-aviary accounts
- Push notifications (except the opt-in visit notification toggle)
- Customizable scenes or species purchasing
- Co-presence in visits

---

## 2. Architecture — service shape and client/server split

### High-level topology

```
┌─────────┐    HTTPS/WSS     ┌──────────────┐    read/write     ┌───────────────┐
│ Browser  │◄────────────────►│  API Gateway  │◄────────────────►│ Backend Svcs  │
│ (Client) │   state snapshots│  (edge/CDN)   │   event log      │               │
└─────────┘   + event writes  └──────────────┘                   │ ┌───────────┐ │
                                               │ ├─ Auth service│ │
                                               │ ├─ Aviary API  │ │
                                               │ ├─ Sim ticker  │ │
                                               │ ├─ Notebook    │ │
                                               │ └─ Visit mgr   │ │
                                               └───────────────┘
```

### Services (each deployable independently)

1. **Auth service** — email/magic-link sign-in, session token issuance, device revocation. Stateless except for issued token store (Redis or similar). Synthetic UUID assigned at account creation; email stored encrypted, never used as secondary key.

2. **Aviary API** — the read/write surface for clients. Handles `GET /state` (current canonical snapshot), `POST /events` (interaction events appended to the event log), notebook queries, visit-invitation management, account settings reads/writes. Stateless; reads canonical state from the simulation database, writes events to an append-only event-store service (Kafka topics, or a simpler append-only table for v1).

3. **Simulation ticker** — a scheduled worker (serverless cron or long-lived process) that runs once per minute per active account (where "active" = any interaction in the last 30 days; accounts with zero recent activity are lazy-ticked on next client pull instead of every minute). The ticker reads the event log since its last pass, computes drift deltas, transitions moods, updates mood timers, advances time-of-day state, and writes the new canonical state atomically. The ticker is the only writer of personality vectors.

4. **Notebook generator** — a separate scheduled job (runs less frequently, e.g., every few hours) that reads recent aviary events and canonical state and generates naturalist-prose notebook entries. Because entry generation is creative-text work at a sparse cadence, this is a separate concern from the per-minute simulation tick.

5. **Visit manager** — handles visit-invitation creation (host enters visitor email → system emails a one-time view link), visit link resolution (visitor follows link → serves read-only aviary snapshot, flagged as `role: visitor`), revocation (host revokes → next visitor snapshot pull returns `visit_unavailable`), and visit logging.

### Database topology

- **Canonical state store** (PostgreSQL or similar relational DB) — one row per account containing: account metadata, bird records (one row per bird with stable_id, name, species, personality vector as JSONB, current mood, current perch, last-tick timestamp), notebook entries, visit log, account settings. Personality vectors are a single JSONB column per bird; never decomposed into per-trait columns that would invite direct SQL reads by non-simulation code paths.
- **Event log** (append-only, time-ordered) — one event per row: account_id, bird_id (nullable), event_type, payload (JSONB), client_timestamp, server_timestamp. The event log is consumed by the simulation ticker and the notebook generator. It is not read by the Aviary API for state snapshots (the API reads canonical state directly).
- **Auth store** — accounts table (synthetic UUID PK, email encrypted, created_at, deleted_at, deletion_state), session token table (token hash, account_id, device label, created_at, last_seen_at), magic link table (token hash, email, expires_at, consumed_at).

### Client/server boundary

The boundary is sharp: the client renders snapshots, never owns state. The client never computes drift, never transitions moods, never writes personality vectors. The client's responsibilities are:

1. Pull current state snapshot on open / visibility change / keepalive.
2. Render birds, scene, audio from the snapshot.
3. Synthesize procedural calls from the motif library (shipped as part of the JS bundle).
4. Interpolate between snapshots for smooth motion.
5. Send interaction events to the append-only log via `POST /events`.
6. Manage WebAudio context lifecycle.
7. Run accessibility surfaces (narration queue, caption display, keyboard navigation, reduced-motion rendering).

### Render pipeline boundary

The rendering pipeline is a pure function of the state snapshot + local time + audio state. It has no side effects that write back to server state (all server-state writes go through `POST /events`). The render loop runs on `requestAnimationFrame`, consumes the latest interpolated state, and produces frames. Audio synthesis runs in a separate `AudioWorklet` context (or `ScriptProcessorNode` fallback) that reads from the same state and the motif library.

---

## 3. Data model

### Account

```
account {
  id: UUID (synthetic, never email-derived)
  email: encrypted string (single column, never used as lookup key outside auth service)
  created_at: timestamp
  deletion_state: "active" | "soft_deleted" | "hard_deleted"
  soft_deleted_at: timestamp | null
  aviary_created_at: timestamp (used for age-gating bird offers)
  timezone: string (user-set or auto-detected; drives day/night cycle)
  settings: {
    reduced_motion: boolean (default: prefers-reduced-motion media query)
    captions_enabled: boolean (default: false)
    visit_notifications_enabled: boolean (default: false)
  }
}
```

### Bird

```
bird {
  id: UUID (stable, invariant across renames/syncs/migrations — same bird forever)
  account_id: UUID (FK to account)
  name: string (user-assigned, renameable anytime)
  species: string (from the ~6-species pool)
  adopted_at: timestamp
  position_order: int (order in which bird was adopted; 1 = first starter, 2 = second starter, etc.)

  personality_vector: {
    boldness: float (normalized small range, e.g. 0.0–1.0)
    social_warmth: float
    vocal_frequency: float
    plumage_saturation: float
    curiosity: float
    version: int (incremented on each tick that modified this vector)
  }

  current_mood: enum("wary", "content", "curious", "drowsy", "alert")
  current_perch: enum("front", "middle", "back")
  last_mood_transition_at: timestamp
  last_tick_at: timestamp
}
```

**Personality vector rules:**
- Never exposed to the user numerically (no stats panel, no debug view, no API surface that returns raw values to the client).
- Stored as a single JSONB blob; versioned so the ticker can detect stale reads.
- Only the simulation ticker writes it. No client path, no admin path, no migration script writes it directly.

### Presence session

```
presence_session {
  id: UUID
  account_id: UUID
  started_at: timestamp
  ended_at: timestamp | null
  total_presence_ms: int (accumulated)
  last_activity_at: timestamp (most recent pointermove/keypress that satisfied the conjunction rule)
  terminated_by: enum("settle", "tab_close", "timeout") | null
}
```

### Interaction event (event log)

```
interaction_event {
  id: UUID
  account_id: UUID
  bird_id: UUID | null (null for global events like settle)
  event_type: enum(
    "presence_ping",       // client sends every ~30s while all three conditions hold
    "presence_ended",      // client sends on settle or tab-close
    "listen_in_start",
    "listen_in_end",
    "offer_seed",
    "offer_song",
    "offer_pool",
    "settle",
    "settle_undo",
    "greeting_observed",   // logged when client renders a return-greeting
    "session_opened",
    "session_closed"
  )
  payload: JSONB (per-type fields, e.g. listen_in_duration_ms, offer_accepted boolean)
  client_timestamp: timestamp (from client clock — informational only)
  server_timestamp: timestamp (set by API on write, used for ordering)
}
```

### Notebook entry

```
notebook_entry {
  id: UUID
  account_id: UUID
  created_at: timestamp (ties to a specific aviary moment)
  prose: text (naturalist voice, lowercase, present-tense, specific)
  entry_type: enum("greeting", "mood_shift", "milestone", "weather_moment", "quiet_observation")
  referenced_bird_ids: [UUID] (empty for aviary-wide entries)
}
```

### Visit

```
visit_invitation {
  id: UUID
  host_account_id: UUID
  visitor_email: string
  token: string (one-time use, hashed in DB)
  status: enum("pending", "active", "revoked", "expired")
  created_at: timestamp
  expires_at: timestamp
  first_used_at: timestamp | null
}

visit_session {
  id: UUID
  invitation_id: UUID
  started_at: timestamp
  ended_at: timestamp | null
  duration_ms: int | null
}
```

### Time-of-day state (computed, not stored as a column)

Computed server-side on each tick from the account's timezone and current UTC time. No separate storage; the ticker derives it inline. Clients receive `time_of_day: { phase: "morning"|"midday"|"evening"|"night", progress: 0.0–1.0 }` in each state snapshot for rendering.

### Weather state (computed, not stored as a persistent column)

The ticker, on each pass, rolls a low-probability check for weather events. Weather events are short-lived (a few minutes of sim time). The current weather state is written into the canonical state snapshot as `weather: { type: "clear"|"rain"|"wind", intensity: 0.0–1.0, started_at: timestamp, remaining_ticks: int }`.

---

## 4. API surface

### REST endpoints (all over HTTPS; auth via Bearer token)

#### State pull

**`GET /api/v1/aviary/state`**

Returns the current canonical state snapshot. Response shape:

```json
{
  "account_id": "uuid",
  "aviary_age_days": 42,
  "time_of_day": { "phase": "morning", "progress": 0.35 },
  "weather": { "type": "clear", "intensity": 0 },
  "birds": [
    {
      "id": "uuid",
      "name": "Pip",
      "species": "warbler",
      "current_mood": "content",
      "current_perch": "front",
      "position_order": 1,
      "active_animation": "preening",
      "last_greeting_at": "iso-timestamp",
      "plumage_level": "computed from plumage_saturation, rendered client-side"
    }
  ],
  "settled": false,
  "snapshot_version": 8472
}
```

Key design decisions:
- Personality vector values are **never** included in this response. Plumage level is transmitted as a client-renderable value derived from plumage_saturation (e.g., a `plumage_level` integer 0–3 that maps to visual feather-detail tiers) — the raw float never crosses the wire.
- `snapshot_version` is an opaque monotonic integer the client sends back with events for ordering.
- The snapshot is small; target wire size under 20KB for a full seven-bird aviary.

**`GET /api/v1/aviary/state?since_version=8470`** (incremental pull for keepalive/visibility-change). Returns a diff: only birds whose state changed since the given version. If `since_version` is too old (the server has compacted), returns a full snapshot with a flag `full_snapshot: true`.

#### Event writes

**`POST /api/v1/aviary/events`**

Body:

```json
{
  "snapshot_version": 8472,
  "events": [
    {
      "type": "listen_in_start",
      "bird_id": "uuid",
      "client_timestamp": "iso-timestamp"
    }
  ]
}
```

Events are acknowledged immediately (202 Accepted); the server assigns a `server_timestamp` and appends to the event log. Events are processed asynchronously by the simulation ticker on its next pass. The API validates: `snapshot_version` must not be older than the server thinks the client last saw (stale-write protection — if older, the API rejects with 409 Conflict and the client must pull a fresh snapshot first). Bird IDs must belong to the authenticated account.

**`POST /api/v1/aviary/events/presence`**

Specialized lighter endpoint for presence pings. Accepts a single `presence_ping` event. Clients call this every ~30s while all three presence conditions hold. The server updates the active `presence_session` row's `last_activity_at` and accumulates `total_presence_ms`. Returns `presence_session_id` for the client to reference on session-end.

#### Notebook

**`GET /api/v1/notebook?before=entry_id&limit=20`**

Returns notebook entries in reverse chronological order, paginated by `before` cursor (entry ID). Each entry is `{ id, created_at, prose, entry_type }`. The prose text is generated server-side; the client never generates notebook prose.

#### Account

**`POST /api/v1/auth/request-link`** — body: `{ email }`. Sends magic link. Rate-limited per email.

**`POST /api/v1/auth/verify-link`** — body: `{ token }`. Returns `{ account_id, access_token, refresh_token }`.

**`POST /api/v1/auth/refresh`** — body: `{ refresh_token }`. Returns new access token.

**`POST /api/v1/auth/revoke-session`** — body: `{ session_id }`. Revokes a device session.

**`GET /api/v1/account/settings`** — returns current settings.

**`PATCH /api/v1/account/settings`** — updates settings (reduced_motion toggle, captions toggle, visit_notifications toggle, timezone).

**`POST /api/v1/account/birds/rename`** — body: `{ bird_id, new_name }`. Renames a bird.

**`POST /api/v1/account/export`** — triggers export generation; emailed as download link.

**`POST /api/v1/account/delete`** — initiates soft deletion.

**`POST /api/v1/account/undelete`** — recovers from soft deletion within 30-day window.

#### Visit invitations

**`POST /api/v1/visits/invite`** — body: `{ visitor_email }`. Creates invitation; system emails the one-time link. Returns `{ invitation_id, expires_at }`.

**`POST /api/v1/visits/revoke`** — body: `{ invitation_id }`. Revokes immediately; any active visit session terminates on next snapshot pull.

**`GET /api/v1/visits/log`** — returns `[{ visitor_email, visited_at, duration_approx, invitation_status }]`.

**`GET /api/v1/aviary/visit-state?token=...`** — unauthenticated endpoint for visitors. Returns a read-only subset of the state snapshot (birds, mood, time_of_day, weather; no account metadata, no settings). Validates the one-time token, checks expiry and revocation. Returns `{ state: {...}, visit_active: true }` or `{ visit_active: false, reason: "expired"|"revoked" }`.

### API design principles

- All client-facing endpoints return JSON. No HTML templating served from the API (the frontend bundle is static; served from CDN).
- Error responses use matter-of-fact prose: `{ error: "session_expired", message: "Your session timed out. Sign in again to keep watching." }`.
- Personality vector values are never present in any API response body, including admin/debug endpoints accessible only internally. The only code that reads raw personality values is the simulation ticker.
- All timestamps are ISO 8601 UTC.
- Rate-limiting applies to auth endpoints, event-write endpoints, and the visit-invitation create endpoint. State-pull and notebook-read endpoints are un-rate-limited beyond standard DoS protection.

---

## 5. Simulation engine design

### The tick loop (runs server-side, ~once per minute per active account)

```
for each active account:
  lock account row (advisory lock or SELECT ... FOR UPDATE on the account's bird rows)
  read current canonical state (birds, moods, personality vectors, last tick timestamp)
  read new interaction events since last_tick_at from event log
  compute_time_of_day(account.timezone, now())
  compute_weather(account_id, current weather state)
  for each bird:
    delta = compute_drift_delta(bird.personality_vector, new_events, presence_session)
    bird.personality_vector = apply_drift(bird.personality_vector, delta)  // monotonic up only
    bird.current_mood = transition_mood(bird.current_mood, bird.personality_vector,
                                         new_events, time_of_day, weather,
                                         other_birds_moods)
    bird.current_perch = select_perch(bird.current_mood, bird.personality_vector.boldness)
  write updated state atomically
  increment snapshot_version
  unlock
```

**Active account definition:** any account that has had a presence_session or interaction_event within the last 30 days. Accounts beyond this window are lazy-ticked: the tick is computed on-demand when the next `GET /state` arrives, and the ticker skips them in its scheduled loop. This limits infrastructure cost while preserving correctness — the user who comes back after 6 months still sees an aviary that advanced, but the server hasn't been ticking their account twice a minute for half a year.

### Drift function

**Calibration target:** measurable instrument drift at ~1 week of regular visits (~30 min/day presence); visible user drift at ~3 weeks.

**Drift delta per tick (per trait):**

```
drift_increment = base_rate * presence_factor * bird_specific_factor * noise

base_rate = 0.0001  // calibrated so a week of ~30min/day presence produces measurable change
presence_factor = min(presence_minutes_in_tick_window / 1.0, 1.0)  // capped at 1.0
  where presence_minutes_in_tick_window is total presence-time in the ~60s since last tick
bird_specific_factor:
  - boldness: + (0.3 * listen_in_count_this_window) + (0.1 * offer_count_nearby)
  - social_warmth: + (0.4 * listen_in_count_this_window) + (0.05 * chorus_participation_count)
  - vocal_frequency: + (0.4 * listen_in_count_this_window) + (0.1 * greeting_count_this_window)
  - plumage_saturation: + (presence_factor * 0.8)  // drifts mostly on pure presence
  - curiosity: + (0.3 * offer_accepted_count) + (0.1 * listen_in_count_this_window)
noise = random(0.8, 1.2)  // small per-tick variation so drift isn't perfectly linear
```

**Critical constraint: drift is monotonic toward expressive.** The delta is always ≥ 0. No trait ever decreases. The `apply_drift` function is `new_value = min(old_value + delta, 1.0)`. This is the engine-level expression of "no Tamagotchi punishment."

**Footnote:** The exact `base_rate` and per-factor coefficients above are initial calibration values. They must be tested against the calibration target (measurable drift at ~1 week) using a simulation test harness before finalization. The test harness runs a synthetic user that generates a known presence/interaction pattern over simulated weeks and asserts that personality values move by the expected amount.

### Mood transition function

Mood is an enumerated state per bird. Transitions are probabilistic, shaped by weighted inputs:

```
transition_weights(current_mood, personality, recent_events, time_of_day, weather, nearby_bird_moods):
  weights = { "wary": 0, "content": 0, "curious": 0, "drowsy": 0, "alert": 0 }

  // time-of-day pushes
  if time_of_day.phase == "evening": weights["drowsy"] += 0.3
  if time_of_day.phase == "night":   weights["drowsy"] += 0.5
  if time_of_day.phase == "morning": weights["alert"]   += 0.2

  // personality modulates
  weights["wary"]    -= personality.boldness * 0.2     // bolder birds less likely to be wary
  weights["curious"] += personality.curiosity * 0.15

  // event-driven pushes
  if offer_accepted_in_window:  weights["content"]  += 0.4
  if alarm_call_from_nearby:    weights["wary"]     += 0.3
  if listen_in_active_in_window: weights["content"]  += 0.1  // being paid attention to

  // weather
  if weather.type == "rain": weights["drowsy"] += 0.2

  // self-reinforcement: current mood has inertia
  weights[current_mood] += 0.25

  // normalize and sample
  return weighted_random_sample(weights)
```

Mood transitions are not applied on every tick; a mood has a minimum dwell time (e.g., birds stay in a mood for at least ~5 ticks / 5 minutes before transitioning). This prevents mood flicker.

**Mood persistence across sessions:** mood is stored server-side and is NOT reset on session open. The mood from session-end carries forward through the server tick and into the next session's first snapshot.

### Perch selection

Deterministic from mood + boldness:

```
select_perch(mood, boldness):
  if mood == "wary":    return "back"    // always
  if mood == "drowsy":  return "middle" if boldness > 0.4 else "back"
  if mood == "content": return "front" if boldness > 0.6 else "middle"
  if mood == "curious": return "front" if boldness > 0.3 else "middle"
  if mood == "alert":   return "front"
```

The user never controls perch position. Birds choose; users read the signal.

### Call grammar runtime (server-side reference; client-side execution)

The call grammar is not stored as raw audio but as a motif library — a set of parameterized sound-descriptor objects per species. Each species has:

- **Motif pool:** 8–12 short melodic fragments described by: base_frequency, frequency_envelope (attack/decay/sustain/release curve), harmonic_profile (which overtones and at what amplitudes), modulation_params (vibrato depth/rate, pitch-bend curve).
- **Call-composition rules:** how motifs combine into a call (single motif, two-motif sequence with gap, three-motif phrase). Mood shapes which rules are active (drowsy = short single-motif calls only; alert = complex phrases).
- **Timing parameters:** call_interval_base (seconds between calls, modulated by vocal_frequency trait), chorus_join_likelihood (probability of joining when another bird calls), call_variation (small random perturbation to frequency/timing per call).

On each tick, the server computes per-bird call timing: `next_call_at` timestamps are scheduled based on vocal_frequency and mood. When the client renders a bird's call at the scheduled time, it synthesizes the call audio from the motif library client-side via WebAudio. The motif library (the parameter descriptors, not the audio itself) ships as part of the JS bundle (~50KB of JSON descriptors for six species * ~10 motifs each).

**Why server-side timing, client-side synthesis:** The timing is part of canonical state (the ticker decides when Pip's next call is). The audio generation is client-side to satisfy the bundle budget (no recorded audio files) and to enable real-time chorus mixing (the WebAudio graph mixes synthesized calls at runtime).

### Return-greeting logic

Computed server-side on session open (the client's first `GET /state` in a new presence window triggers the ticker to compute a greeting):

1. Select greeter: the bird with the highest `social_warmth * boldness` product whose mood is not "drowsy." Break ties with `position_order` (older birds).
2. Generate greeting call: procedurally varied by absence_duration (short absence → short single-motif call; long absence → longer multi-motif call with approach-to-front-perch animation) and the bird's personality.
3. Stagger: if multiple birds would greet, stagger their call timings by a randomized offset (0.8–3.0 seconds) so they don't fire in unison.

The greeting is encoded in the state snapshot as a `greeting` field: `{ bird_id, call_type ("glance"|"single_call"|"approach_and_call"), animation_sequence }`. The client renders it, then logs `greeting_observed` to the event log.

---

## 6. Sync model

### Single canonical aviary

There is one database row per bird per account. That row is the canonical source of truth. The server simulation ticker is the sole writer. Clients read snapshots; they never write state directly.

### How multi-device works (no sync, just reads)

1. User signs in on laptop. Laptop calls `GET /state` → renders aviary from snapshot v8472.
2. User closes laptop. Tab becomes hidden; client sends `presence_ended`.
3. Server ticker continues running. Aviary advances to snapshot v8480 over the next 8 minutes.
4. User opens phone. Phone calls `GET /state` → receives snapshot v8480. Renders the aviary as it is now, not as it was when the laptop closed. Birds have moved, moods may have transitioned, a weather event may have started.
5. Both devices see the same aviary because both read the same row.

### Conflict prevention (the `snapshot_version` mechanism)

The only write conflict possible is: a client sends events referencing an old snapshot, and the server has advanced since. This is handled by the `snapshot_version` check on `POST /events`:

- Client sends events with `snapshot_version: 8472`.
- Server checks: is 8472 within the server's compaction window? If the server has advanced past the compaction horizon (say, 100 versions behind), reject with 409. The client discards its local interpolation and re-pulls a fresh snapshot. If within the window, accept — the event log preserves the ordering, and the ticker processes events in `server_timestamp` order, not `snapshot_version` order.
- For v1 simplicity: set the compaction window large (e.g., 1000 versions). A 409 is a rare event that resolves with a re-pull. We can tighten later.

### No last-write-wins for personality

Personality drift is additive server-authored deltas, never client-submitted absolute values. The only write path for personality is the simulation ticker's `apply_drift` function. There is no API endpoint that accepts personality vector values from clients. This is an architectural constraint enforced by the absence of any route that accepts personality data — not a policy enforced by validation. The code literally cannot receive personality writes from outside.

### Stale state during tick

The ticker locks the account's bird rows during computation to prevent a concurrent tick on the same account. The lock is brief (<100ms for a 7-bird aviary). During tick, `GET /state` reads the pre-tick snapshot (the server can serve from the previous version rather than blocking). A client that pulls state 50ms before the tick completes gets the pre-tick state; the next pull gets the updated state. This is fine — the tick is slow enough (~1/min) that no client notices the boundary.

---

## 7. Frontend rendering pipeline

### Boot sequence (target: first bird visible <500ms)

```
Phase 1 (0–50ms): HTML shell delivered from CDN edge. Inlined critical CSS sets the aviary background color to the sky palette for the current time-of-day (no flash of white). Small inlined JS snippet starts fetching the state snapshot and the deferred JS bundle in parallel.

Phase 2 (50–200ms): State snapshot arrives. JS bundle is still loading. The inlined snippet renders the sky gradient + perch silhouettes from the snapshot data (static, no animation yet). This is the "quiet field" — no spinner, no skeleton screen. The visual says "the aviary is here, catching up."

Phase 3 (200–500ms): JS bundle hydrates. The rendering engine reads the snapshot, instantiates bird sprites/graphics, positions them on perches, starts the idle-motion loop. First bird appears in a still pose (mid-action, not fading in). The next frame adds micro-motion. The frame after that, the audio graph initializes and the first call may sound.

Phase 4 (500ms+): All birds visible, idle motion running, audio pipeline active, return-greeting rendered. The scene settles into its continuous render loop.
```

**Key: no entry animation.** The first frame with birds is the aviary mid-action. Birds are placed at their current positions in their current poses. The next frame they move. This is achieved by (a) the snapshot including `active_animation` and `animation_progress` fields so the client knows what a bird was doing, and (b) the renderer starting its loop from those states rather than from an "entry" state.

### Render loop

```
each frame (requestAnimationFrame):
  interpolate bird positions between snapshot perches
  advance idle micro-motion state machines (preen cycle, head-tilt, weight-shift, scan)
  advance ambient leaf/feather drift (client-only ornaments)
  advance time-of-day sky gradient (slow lerp toward next tick's time-of-day)
  if reduced-motion mode: cross-fade between still poses instead of tweening
  draw scene: background → middle perch plane → birds → foreground → top bar overlay
  if captions enabled: draw caption text near calling birds
```

### Idle micro-motion state machines

Each bird has a small state machine for idle behavior, driven by mood:

- **Preening** (content, drowsy moods): cycles through a sequence of preen poses (beak-to-wing, wing-extend, body-shake). Each pose lasts 2–5 seconds with variable timing.
- **Scanning** (alert, wary moods): head moves in small arcs, eyes shift. Frequency higher for wary.
- **Head-tilt** (curious mood): toward sounds (calls, weather) or toward the viewer.
- **Weight-shift** (all moods): small body shuffle every 5–15 seconds.
- **Feather-fluff** (drowsy, content): periodic fluff-and-settle.

These are implemented as procedural tweening between key poses, not as sprite-sheet animation loops. Poses are defined in a small vector-art format (SVG paths with color palette overlays, or compact JSON vertex descriptors). The tweening engine interpolates path points for smooth morphing.

### Scene composition

**Layers (back to front):**
1. Sky gradient (CSS or canvas fill, time-of-day phased)
2. Background foliage/leaves (SVG, subtle parallax at ~0.2x)
3. Back perch (SVG)
4. Middle perch (SVG)
5. Birds on perches (rendered per layer 3/4/5)
6. Front perch (SVG)
7. Foreground branch/leaf occluder (SVG, ~0.5x parallax, rare)
8. Ambient leaf/feather drift particles (canvas overlay or CSS-animated elements)
9. Caption text overlay (if enabled)
10. Top bar (HTML-overlaid, fades on cursor stillness)

### Transitions

**Perch-to-perch:** When a bird moves perches (ticker output), the client interpolates a soft arc path (not a straight line). The bird lifts off the source perch with a wing-flap animation, arcs through a Bézier curve, lands on the target perch with another wing-flap. Duration: ~1–1.5s. Driven by the snapshot diff — if `current_perch` changed between snapshots, animate the transition.

**Day/night:** The sky gradient color stops lerp smoothly over the course of the time-of-day phase. No abrupt color change; the lerp distributes over the full duration of a phase (e.g., morning-to-midday transition takes ~3 hours of real time, at one lerp step per frame).

**Settle:** On settle event, the sky shifts toward evening over ~3 seconds (accelerated lerp), calls quiet (the audio mix applies a gain reduction), and birds shift toward drowsy postures. On settle-undo (within 5s), reverse the lerp over ~1 second.

**Reduced-motion transitions:** Perch changes become cross-fades (old perch → opacity 0, new perch → opacity 1 over ~2s). No flight arc. Day/night remains (color lerp is non-vestibular). Settle is a slower cross-fade (~5s).

### Rendering tech choices

- **Canvas 2D** or **WebGL** for the aviary scene (birds, perches, foliage, particles). WebGL is preferred for smooth 60fps morphing on mobile but Canvas 2D is the fallback. The rendering engine abstracts the backend via a `Renderer` interface.
- **HTML/CSS** for the top bar and all chrome surfaces (account settings, notebook panel, offer panel, accessibility settings). These are standard React/Vue/Svelte components — framework choice is secondary to the perf budget.
- **SVG** for bird species silhouettes and perch assets. Small, scalable, tintable via the plumage palette.
- **CSS custom properties** for the aviary color palette so reduced-motion mode and accessibility high-contrast overrides can swap palettes without re-rendering.

---

## 8. Audio pipeline

### Architecture

```
┌──────────────────────────────────────────────────┐
│                   AudioWorklet                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │ Bird 1   │  │ Bird 2   │  │ Bird N   │        │
│  │ Synth    │  │ Synth    │  │ Synth    │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │gain 1       │gain 2       │gain N         │
│       ▼             ▼             ▼               │
│  ┌────────────────────────────────────┐          │
│  │        Mixer (GainNode)            │          │
│  └──────────────┬─────────────────────┘          │
│                 │                                 │
│                 ▼                                 │
│  ┌──────────────────────────────┐                │
│  │  Ambient Reverb (Convolver)  │                │
│  └──────────────┬───────────────┘                │
│                 │                                 │
│                 ▼                                 │
│  ┌──────────────────────────────┐                │
│  │  Master Gain → destination   │                │
│  └──────────────────────────────┘                │
└──────────────────────────────────────────────────┘
```

Each bird gets a dedicated gain node. The mixer sums them into a single output. Ambient reverb (a small convolution or delay-based reverb) applies to the mixed output for spatial depth.

### Procedural call synthesis

Each bird's call is synthesized on-demand when the client's audio scheduler determines that `next_call_at` has arrived (based on the state snapshot's call schedule). The synthesis pipeline:

1. **Select motifs:** based on species, current mood, and vocal_frequency trait. Motifs are JSON descriptors: `{ base_freq, freq_envelope: [attack, decay, sustain, release], harmonics: [{multiple, amplitude}], modulation: {vibrato_rate, vibrato_depth, pitch_bend} }`.

2. **Build oscillator graph:** For each harmonic, create an `OscillatorNode` at `base_freq * multiple` with amplitude from the descriptor. Apply the frequency envelope (attack ramp, decay ramp, sustain level, release ramp) via `setValueAtTime` / `linearRampToValueAtTime` on the frequency and gain params.

3. **Apply modulation:** if vibrato, connect an LFO `OscillatorNode` (low frequency, <20Hz) to the frequency param of the tone oscillators.

4. **Apply personality variation:** small random offsets to timing, frequency (±5%), and envelope shape per call, so the same motif never sounds identical twice.

5. **Schedule release:** after the call duration (from the envelope descriptors), stop and disconnect all nodes. Nodes are reused from a pool to prevent GC pressure (the "no memory growth" constraint).

### Chorus mixing

When two birds call in overlapping windows, the WebAudio graph naturally mixes their output — this is why procedural synthesis matters. Two synthetically-generated calls mixing in the audio graph produce a true chorus, with beat frequencies, phase interactions, and harmonic interplay that recorded loops cannot achieve (two recorded loops stacked produce phase cancellation artifacts because their waveforms are identical at the sample level; procedural calls generate different waveforms each time).

The chorus also has a spatial element: birds on the left side of the scene get a subtle stereo pan left; right-side birds pan right. This is achieved via `StereoPannerNode` per bird, connected before the gain node and after the synthesis graph.

### Listen-in mix decay

When listen-in is active on bird B:

- Bird B's gain node: ramp from current value to 1.0 (full volume) over ~1s.
- All other birds' gain nodes: ramp from current value to 0.15 (ambient, not silent) over ~1s.
- On listen-in end: reverse ramp back to the pre-listen-in gains over ~1s.
- The ramp uses `linearRampToValueAtTime` for smoothness.

### WebAudio lifecycle

- `AudioContext` is created on first user gesture (click, tap, keypress) to comply with browser autoplay policies. Before the first gesture, the aviary renders visually with captions on if audio is unavailable.
- On tab hidden (`visibilityState !== "visible"`), the `AudioContext` is suspended via `context.suspend()` to save CPU and battery.
- On tab visible, resume the context. Calls that would have fired during the hidden period are skipped (the aviary continues server-side, but the client doesn't retroactively play missed calls — it picks up the current state on resume).

### WebAudio fallback

If `AudioContext` is unavailable (older browser, denied permission, hardware failure), the client enters silent mode: captions are enabled by default, the audio graph is never instantiated, and all call events render as caption text only. No recorded audio fallback. No polyfill for `AudioContext`. The WebAudio-unavailable surface is just the aviary without sound — still visually animated, still interactive.

---

## 9. Accessibility surfaces

### Screen-reader narration

**Architecture:** A client-side narration queue consumes the aviary state snapshot and produces prose updates on a slow cadence. The queue uses an `aria-live="polite"` region to announce prose to screen readers without interrupting the user's current context.

**Implementation:**

1. On each state snapshot pull, a diffing function compares the previous snapshot to the current one and generates a set of "observable changes": bird moved perch, bird changed mood, time_of_day phase changed, weather started/ended, greeting occurred, offer was accepted, settle happened.

2. Each observable change maps to a naturalist prose sentence via a template engine that selects from multiple phrasings per change type:

   ```
   mood_change(bird_name, old_mood, new_mood):
     choices = [
       "{bird} is {new_mood} now, {mood_description}",
       "a shift — {bird} seems {new_mood}",
       "{bird} has settled into a {new_mood} state"
     ]
     return random_choice(choices)
   ```

3. Prose updates are enqueued. The queue drains at most one update per 30 seconds at idle; user-initiated events (greeting, offer, settle) get a priority bump and are narrated within 5 seconds of the event.

4. The prose is never diagnostic ("Pip is at perch 2"), never numeric ("boldness changed by 0.03"), and never gamified ("Pip greeted first!" — the exclamation is a gamification cue). It follows the field-notebook voice: lowercase, present-tense, specific.

5. The narration generator runs client-side so it can respond to user events in real time. It reads from the same state snapshot the visual renderer uses. It does not call a separate API.

### Reduced-motion mode

**Detection:**
- Media query `prefers-reduced-motion: reduce` → auto-enable.
- Account setting `reduced_motion: true` → manual enable (persisted server-side, synced across devices).

**Rendering changes:**
- **Idle micro-motion:** Replaced by slow cross-fades between still poses. Each idle state machine still runs internally (to compute which pose should be displayed), but instead of tweening between poses, the renderer cross-fades: old pose opacity → 0, new pose opacity → 1 over ~1.5s.
- **Perch transitions:** Cross-fade between perch positions (bird renders on new perch with opacity fade-in while old position fades out). No flight arc.
- **Ambient particles:** Leaf/feather drift removed entirely.
- **Day/night color shifts:** Retained (slow color lerp is non-vestibular).
- **Settle transition:** Extended to ~5s (slower cross-fade to evening palette).
- **Top bar fade:** Retained or user-controlled opacity.

**Important:** Reduced-motion is not a CSS `animation: none` override. It's a parallel rendering path. The scene still has visual life; it just moves differently.

### Call captions

- **Generation:** When the audio engine schedules a call, it also produces a caption string derived from the procedural call parameters. The caption generator inspects the chosen motifs and their parameters to produce naturalist prose:

  ```
  generate_caption(bird_name, motifs, mood):
    if mood == "drowsy":    return "a soft single note from {bird}"
    if len(motifs) == 1:    return "a brief call from {bird}"
    if len(motifs) == 2:    return "a two-note rise from {bird}"
    if len(motifs) >= 3:    return "a winding call from {bird}, climbing then settling"
  ```

- **Display:** Small text near the calling bird's position, fading in over 0.3s, holding for the call duration, fading out over 0.5s. CSS-animated opacity on an absolutely-positioned text element. The text uses a high-contrast color against the aviary background (meets WCAG AA).
- **Semantic markup:** Caption text is wrapped in `<span role="status" aria-label="...">` so screen readers can optionally announce it (this is separate from the narration prose — captions are per-call, narration is per-moment).

### Keyboard navigation

**Tab order:**
1. Top bar items (left to right): settings icon → notebook icon → offer affordance → settle button → accessibility settings.
2. When a bird is focused, the Tab key within the aviary cycles: Bird 1 → Bird 2 → ... → Bird N → back to top bar.

**Bird focus:**
- Focus indicator: a soft 2px outline around the bird, high-contrast against any background state (uses a dual-color outline technique: dark outline with a light outer glow, or vice versa depending on the time-of-day palette behind the bird).
- Arrow Left/Right: move focus to previous/next bird.
- Enter / Space: toggle listen-in on the focused bird.
- Escape: exit listen-in, return focus to the aviary container.

**Offer panel (opened from top bar):**
- Arrow keys navigate between offer types (seed, song, pool).
- Enter selects the offer and dismisses the panel.
- Escape closes the panel without offering.

### Contrast

All user-copy text (top bar labels in icon tooltips, settings pages, error messages, notebook prose, captions, narration when displayed visually in the accessibility panel) meets WCAG AA contrast ratio minimum (4.5:1 for normal text, 3:1 for large text). The aviary background colors are specified against these contrast targets in the design system.

### Accessibility settings panel

Reachable from the top bar. Exposes:

- **Narration:** on/off toggle, narration speed (slow/normal), last narration text (visible for review).
- **Captions:** on/off toggle (overrides the per-session default from WebAudio-unavailable fallback, but defaults are sticky).
- **Reduced motion:** on/off toggle (overrides `prefers-reduced-motion`; the media query still auto-enables on first visit, but the user can toggle after).
- **Keyboard shortcuts:** a reference list.

All labels and error states in this panel use the matter-of-fact voice (this is a system surface, not the naturalist surface).

---

## 10. Performance budgets and observability

### Bundle budget

- **Initial JS bundle:** <2MB gzipped. Enforced in CI via a build-size check that fails the build if exceeded.
- **Breakdown target:** ~1.2MB for the rendering engine + bird species assets + motif library JSON; ~300KB for the framework runtime; ~300KB for the audio synthesis engine; ~200KB reserve for code split surfaces (account settings, notebook, visit flow).
- **Code splitting:** Account settings, accessibility settings, visit-invitation flow, and the full notebook panel are lazy-loaded. Only the aviary surface + top bar + offer panel are in the initial bundle.
- **Asset strategy:** Bird visuals are procedural SVG (or compact JSON vertex descriptors), not PNG spritesheets. The ~6 species at a few plumage variations produce maybe 150KB of vector descriptors total, not MB of raster assets. No recorded audio anywhere in the bundle. Motif library JSON is ~50KB.

### Time-to-first-bird <500ms

- Measured from navigation start to the first frame where a bird is rendered (not just the quiet sky field).
- Achieved by: inlined critical CSS + state-snapshot fetch in the HTML shell; parallel JS bundle fetch; rendering engine starts from snapshot state without waiting for non-critical assets or a "load complete" event.
- CDN edge delivery for the HTML shell and static assets, with the state-snapshot API behind the same CDN (cache hit for read-only state pulls from unauthenticated visitors; authenticated pulls are uncached but lightweight).
- CI synthetic check: a Lighthouse or custom Puppeteer-based test that navigates to the aviary on a throttled 4G profile and asserts the first-bird-render timestamp is under 500ms.

### Runtime performance

- **60fps idle motion on 5-year-old laptop:** The render loop targets 16ms per frame. All bird state machines, tweening calculations, and particle updates must complete within a frame budget. The rendering engine profiles in CI with a frame-budget test (simulated 30-minute session, assert 99th percentile frame time < 16ms on a throttled CPU profile).
- **No memory growth over 30 minutes:** CI test runs a 30-minute synthetic session (automated browser, idle with occasional interactions), captures heap snapshots at start and end, and asserts heap size has not grown beyond the initial + a small buffer (~5MB for unavoidable DOM accumulation from notebook scrolling). Audio node pools are reused; no per-call `OscillatorNode` allocations (nodes are pre-allocated in a pool at context creation, borrowed per call, returned on release).
- **Audio pipeline:** The `AudioWorklet` runs at the audio hardware's sample rate (typically 44.1kHz or 48kHz); the synthesis of a single call is <1ms of worklet time, and the worklet is idle between calls (the `OscillatorNode` pool connects/disconnects but does no work when silent). The audio render quantum (128 samples) never blocks the main thread.

### Observability

**What we measure:**
- **Synthetic checks (fleet of automated browsers):** Page load timing, first-bird-render time, 30-minute session memory stability, audio-context creation success rate, time-to-first-call (time from navigation to first audible call).
- **RUM (Real User Monitoring, aggregate only):** Navigation timing, first-bird-render timing (measured via `PerformanceObserver` for a custom mark), render frame timing histograms, audio-context error rate, WebAudio-unavailable rate, session duration histograms (anonymized, no per-account dimension).
- **Server-side telemetry:** `GET /state` latency p50/p95/p99, `POST /events` latency, simulation tick duration per account (p50/p95/p99), tick failure rate, event-log write throughput, notebook-generation latency.
- **Error budget:** Simulation tick p99 latency alarm at 5s. Page-load p95 alarm at 3s.

**What we deliberately do NOT measure:**
- Per-account interaction history in any aggregate pipeline.
- Per-bird drift trajectories or personality vector values in telemetry.
- Visit-frequency per account.
- Any metric that could reconstruct a user's relationship with their birds.
- The telemetry pipeline never reads the simulation database. The simulation database is never read by the analytics warehouse. This is an architectural rule enforced at the network/permission level (separate database credentials, separate VPC subnets if applicable, no cross-read IAM roles).

---

## 11. Rollout

### Ship sequence

**Phase A — Internal alpha (weeks 1–4 of development):**
- Ship the server-side simulation ticker, the event log, and the Aviary API on staging.
- Ship the rendering engine (aviary scene, birds, idle motion, day/night cycle) connected to a static snapshot (no live server — birds rendered from hardcoded state).
- Ship the audio synthesis engine with motif library for 2 species, connected to the static snapshot.
- Ship the account/auth service with magic-link flow on staging.
- Goal: a complete rendering + audio pipeline, no live simulation yet.

**Phase B — Integration alpha (weeks 5–8):**
- Connect the rendering engine to the live Aviary API.
- Ship presence accounting (client-side detection + server-side session tracking).
- Ship simulation ticker consuming real events and producing real state snapshots.
- Ship return-greeting logic.
- Ship listen-in and offer interactions end-to-end.
- Ship settle gesture.
- Ship multi-device sync (same account, two browsers, same aviary).
- Goal: a complete, living aviary for internal testers.

**Phase C — Polish + accessibility (weeks 9–12):**
- Ship screen-reader narration (prose generation engine + aria-live queue).
- Ship reduced-motion mode (parallel rendering path).
- Ship call captions.
- Ship keyboard navigation.
- Ship field notebook (notebook generator service + client panel).
- Ship visit invitations (visit manager + visitor view).
- Ship account export and deletion flows.
- Performance tuning against the bundle budget and first-bird budget.
- Goal: feature-complete v1, passing CI performance and accessibility gates.

**Phase D — Pre-launch (weeks 13–14):**
- Ship to a small beta cohort (invited users, not public).
- Monitor RUM, tick latency, audio error rates, memory stability.
- Tune drift calibration based on beta telemetry (are birds drifting at the right speed?).
- Fix critical bugs surfaced by beta.
- Accessibility audit pass (manual screen-reader testing, keyboard navigation review, contrast verification).
- Goal: validated v1, ready for public launch.

**Phase E — Launch (week 15):**
- Public launch with 2-bird starter aviaries.
- Synthetic perf checks running in production.
- RUM dashboards live.
- Error budget alerting active.

### Bird count ramp

- Launch: 2 birds.
- Aviary age ~3 months: third bird available. No user action required; the system surfaces the offer in the aviary UI (a quiet moment: a new bird arrives).
- Aviary age ~6 months: fourth bird available.
- Aviary age ~12 months: fifth bird available.
- Aviary age ~18 months: sixth bird available.
- Aviary age ~2 years: seventh bird available (cap).

The user never purchases birds, never earns birds through engagement, never selects birds from a catalog. Birds arrive based on aviary age. The arrival is a scene moment (a fly-in), not a notification. We never send email about new bird availability (no push).

### Day-1 instrumentation

- Synthetic perf checks from 3–4 geographic regions, running aviary page loads and 30-minute idle sessions on a schedule.
- RUM beacon for: page load metrics, first-bird-render, audio context errors, WebAudio-unavailable rate.
- Server-side: tick duration histogram, state-snapshot latency histogram, event-write throughput, auth endpoint latency, error rates on all endpoints.
- Error budget alerting on tick p99 latency >5s and page-load p95 >3s.
- No per-account metrics in any of these pipelines. Privacy boundary enforced at the metric-collection point.

---

## 12. Risks

### Risk 1: Drift calibration failure (HIGH)

**What could go wrong:** The drift function is tuned too fast, and birds change visibly between sessions, turning the product into a Tamagotchi-like stat-manager. Or it's tuned too slow, and users feel that nothing they do matters — the birds are static.

**Mitigation:**
- Build the drift simulation test harness early (Phase A) — a test that simulates synthetic user presence/interaction patterns over simulated weeks and asserts trait movement within the calibration bands.
- Run the harness in CI on every change to the drift function.
- Encode the calibration targets as assertion thresholds, not as comments.
- During beta (Phase D), monitor drift trajectories from real accounts (with opt-in consent from beta testers, within the privacy boundary — these accounts consent to drift monitoring for calibration purposes). If drift is outside the expected band, adjust the `base_rate` before launch.

### Risk 2: Sync correctness — personality data loss (HIGH)

**What could go wrong:** A bug in the event-log ordering, the ticker's locking, or the snapshot-version mechanism causes a tick to overwrite drift deltas from a previous tick (the "last-write-wins with extra steps" failure mode). Because personality vectors are never exposed to the user, this failure is invisible until someone notices "my bird feels different."

**Mitigation:**
- The ticker's `apply_drift` is additive-only. Design it so there is no code path that sets a personality value absolutely from an external input — the only operation is `value += delta`.
- Version the personality vector JSONB field with a `version` integer. Every tick write increments the version. If a tick reads version N and tries to write version N but the database row is now at version N+1 (another tick snuck in), the write is rejected (optimistic locking via `WHERE personality_version = N`).
- CI test: simulate two concurrent ticks, assert that both deltas are applied and neither is lost (the second tick retries after seeing version N+1, re-reads, re-computes with any new events, and writes version N+2).
- Audit log: every tick write to personality vectors also writes an audit record (account_id, bird_id, old_vector, delta_applied, new_vector, tick_timestamp) to a separate append-only table. This is not user-facing; it's for debugging and recovery. Rotated after 90 days.

### Risk 3: Audio uncanniness (HIGH)

**What could go wrong:** The procedural call synthesis produces sounds that are perceptibly synthetic — too pure, too regular, too machine-like. Users feel the audio is "off" and the product's affective core collapses. This is the audio equivalent of the canned-greeting risk from the PRD.

**Mitigation:**
- Ship the audio engine to internal testers in Phase A and collect qualitative feedback on call naturalness before the rest of the product depends on it.
- The motif library must be designed with an audio designer (or a strong synthesis model). Each motif descriptor should produce a call that has some organic irregularity — micro-pitch variation, harmonic complexity, slight envelope asymmetry.
- Add subtle pitch-drift within each call (a very slow LFO on frequency, <2Hz, shallow depth) to simulate organic voice variation.
- Add bandpass-filtered noise to the harmonic profile to simulate breath/air noise in the call.
- Test against the "recognizability" constraint: record test sessions where 7 birds of different species are calling in a chorus, and ask listeners to identify whether they can distinguish individual birds. If not, the cap must be lowered or the species pool redesigned.
- For the WebAudio fallback, ensure captions are tested with users who have audio off — the caption prose must carry the affect that the audio would.

### Risk 4: Accessibility regression — narration quality (MEDIUM)

**What could go wrong:** The screen-reader narration prose engine produces generic, repetitive, or robotic prose that fails to convey the aviary's felt-aliveness. The screen-reader user gets an aviary that feels like a state machine being reported, not a living place being observed.

**Mitigation:**
- The narration prose engine must use a template library with multiple phrasings per change type (minimum 5 variants per change type). Random selection prevents repetition.
- Test the narration output against the field-notebook prose from the PRD examples — a human reviewer should not be able to distinguish narration prose from notebook prose in a blind test.
- Include a screen-reader user in beta testing (Phase D). Their qualitative feedback on narration feel is a launch gate.
- Build the narration engine to be self-testable: a CI test generates narration for 100 simulated aviary state transitions and checks that no output matches the generic/announcement-style patterns from the PRD's negative examples (no "perched at," no "mood: content," no state-list phrasing).

### Risk 5: First-bird budget missed on slow connections (MEDIUM)

**What could go wrong:** The 500ms first-bird budget is achieved in synthetic tests on clean staging, but real users on congested 4G in variable geographies routinely see 800ms+ first-bird times. The "aviary already running" conceit fails for a non-trivial fraction of real users.

**Mitigation:**
- Ship the HTML shell with inlined critical CSS (sky gradient only) and a tiny inline script (<5KB) that fetches the state snapshot and begins rendering the quiet field. The quiet field must render in <200ms on any connection where the HTML shell arrived.
- The first bird render does not depend on the JS bundle arriving — the inline script renders a static bird SVG from the snapshot data. The full rendering engine hydrates after.
- RUM must include a percentile breakdown of first-bird times by geography and connection type (EffectiveConnectionType from the Network Information API, where available). If a geography or connection type consistently exceeds 500ms, the CDN edge configuration or the HTML shell's critical path must be optimized.
- Pre-warm the state-snapshot endpoint at CDN edges in target geographies.

### Risk 6: "Just one" feature creep (LOW at plan stage, HIGH over product lifetime)

**What could go wrong:** A well-meaning contributor adds a streak counter, a "birds adopted" label, a visit counter, or a "friend visited!" toast, arguing it's "just one harmless feature." The cumulative effect breaks the product's refusal of gamification.

**Mitigation:**
- The non-goals section of the PRD is a design document with teeth. Encode it as an architectural linter rule in CI: any UI surface that displays a count tied to user behavior (visits, streaks, days, sessions, birds adopted, offers made) fails the build. The linter checks for strings matching known gamification patterns in UI templates.
- Code review checklist includes a "gamification check" item.
- The "notice, never announce" principle is a review gate for any new UI surface.

### Risk 7: Event-log growth and ticker lag (MEDIUM)

**What could go wrong:** Over months, the per-account event log grows large, and the ticker's "read new events since last tick" query becomes slow. Tick latency creeps up, the p99 exceeds 5s, and users perceive lag in the aviary's responsiveness to their actions.

**Mitigation:**
- The event log is partitioned by account_id. The ticker reads only events with `server_timestamp > last_tick_at` and `account_id = X`, which is an indexed range scan, not a full log scan.
- Event rows are stored in a time-partitioned table (e.g., monthly partitions). Old partitions are archived to cold storage after 90 days (the ticker only needs recent events; the archive preserves the full history for account export and notebook generation).
- The ticker's `last_tick_at` is stored per-account and is monotonic so the read window is always small (~60s of events).
- Monitor tick duration per-account in production; if the p50 starts creeping up, investigate.

### Risk 8: Browser audio policy changes (LOW)

**What could go wrong:** A browser update changes the autoplay policy, requiring a user gesture before any `AudioContext` can produce sound. Calls that should fire on page open are silent until the user clicks.

**Mitigation:**
- Already handled: `AudioContext` is created on first user gesture, not on page load. The aviary renders visually from frame 1 with captions enabled. After the first gesture (click/tap/keypress anywhere on the page), the `AudioContext` is created and the audio pipeline starts. The interim silence is visually bridged by captions.
- Test against latest Chrome, Safari, and Firefox autoplay policies in CI.

---

## Appendix A: Implementation unknowns to resolve during build

These are not risks; they are calibration decisions the PRD explicitly defers to implementation. Each must be resolved before Phase D (pre-launch).

1. **Exact presence activity window** for the pointer/key condition. Start with 3 minutes. Validate during Phase B with internal testers — does presence feel like it "sticks" too long after walking away? Adjust downward (to 2 minutes) if so.

2. **Exact tick cadence.** Start at 60 seconds. If the tick duration for a 7-bird aviary is <10ms, consider 30 seconds for more responsive mood transitions. If it's >50ms, stay at 60 seconds.

3. **Exact mood enumerated set.** Start with `{wary, content, curious, drowsy, alert}`. During Phase B, observe whether the set captures the observable bird behaviors adequately. Add or refine as needed.

4. **Drift `base_rate` coefficient.** The value 0.0001 is a starting point. Calibrate against the 1-week instrument test during Phase A–B. The calibration test harness is the gate.

5. **Notebook entry sparsity.** Start with a generation trigger threshold (e.g., entry generated only if at least 2 "noteworthy" events have occurred since the last entry). Tune during Phase D beta so that a regularly-visited aviary produces ~2–3 entries per week.

6. **Weather event frequency.** Start at ~2 rain events and ~1 wind event per week of simulated time. Tune during Phase B — too frequent and weather becomes noise; too rare and it might as well not exist.

7. **Species pool exact composition.** 6 species, designed with an audio designer. Species must have visually and audibly distinct silhouettes and call signatures. Finalize during Phase A.

8. **Motif library per species.** Target 8–12 motifs per species. Validate the chorus recognizability test (7 birds distinguishable by ear) during Phase A–B. If not, adjust motif count or species count.

---

## Appendix B: Architectural rules that must survive into implementation

These are not implementation details; they are invariant constraints derived from the PRD that the codebase must enforce structurally.

1. **Personality vector values never cross the API boundary.** No route returns them. No client parses them. The rendering engine's `Bird` data model has no `personality` field — only derived visual traits (plumage_level, call_frequency_hint).

2. **Only the simulation ticker writes personality vectors.** No migration, no admin endpoint, no debug CLI, no integration test helper mutates a personality vector directly. Enforce via database permissions (the simulation ticker's DB user has write on the `bird` table; the API's DB user has read-only on the `personality_vector` column).

3. **Email is never a lookup key outside the auth service.** The synthetic account UUID is the only identifier in inter-service messages, database foreign keys, event log partitions, and telemetry. Enforce via code review and schema design (no `email` column in any table outside the auth service's `account` table).

4. **Presence requires all three conditions.** No code path records presence from visibility alone, focus alone, or activity alone. The presence-ping function is a single, tested function (not three scattered checks).

5. **No streak counter. No gamification language.** Enforce via CI linting against known patterns and via the "gamification check" in code review.

6. **No recorded audio in the bundle.** No `.mp3`, `.wav`, `.ogg` files in the source tree. The audio engine is synthesis-only. Enforce via a build-time check that scans for audio file extensions.

7. **Accessibility surfaces ship with the product, not after.** The reduced-motion rendering path, narration engine, caption system, and keyboard navigation are in the initial branch from Phase A. They are not tagged as "post-launch improvements."

---

_This plan is the comprehensive, executable interpretation of the Pocket Aviary PRD for v1. An engineering team reading this document should have no ambiguity about what to build, how to build it, and what constraints are non-negotiable._