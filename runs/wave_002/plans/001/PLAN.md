# Pocket Aviary — V1 Implementation Plan

This plan turns the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable engineering plan. It does not restate the spec's rationale except where a decision in this plan needs to be justified against it. Where the PRD leaves an implementation detail open ("we'll calibrate during build"), this plan makes a specific, defensible choice and flags it as `[CALIBRATED]` so it's visibly a planning decision, not spec text.

---

## 1. Scope

### In v1 (per `product_brief.md` scope statement)

- Single-user accounts, magic-link sign-in, one canonical aviary per account, multi-device sync.
- Two starter birds at adoption, cap of seven, species-pool-driven new-bird offers tied to aviary age.
- Bird engine: personality vector, mood, procedural calls, drift, idle motion, bird-to-bird interaction.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook.
- Presence accounting (the three-condition definition) as the drift engine's primary input.
- Single horizontal aviary scene, day/night cycle, ambient weather, top-bar chrome, responsive layout.
- Visit-invitation (read-only, opt-in, revocable, no co-presence).
- Screen-reader narration, reduced-motion mode, call captioning, full keyboard navigation, WCAG AA.
- Performance budgets: <2MB initial JS, <500ms time-to-first-bird, 60fps idle on a 5-year-old laptop, no 30-minute memory growth.
- Account export, soft-then-hard account deletion, aggregate-only telemetry.

### Explicitly not in v1 (per `non_goals.md`)

- Native apps (iOS/Android). No native-client constraints baked into protocols or data model.
- Any gamification primitive: achievements, streaks, levels, scores, badges, counters of any visit/usage statistic, green-dot calendars. This is enforced architecturally (§3.6) as well as by review discipline, because the PRD names this as the most likely accidental regression.
- Tamagotchi mechanics: no death, hunger, distress, or decaying happiness meter. Drift is monotonic toward expressive (§4.2) — there is no code path that decreases a personality trait.
- Social-network surfaces beyond the single visit affordance: no profiles, follows, public feed, discovery, friend-of-friend chains, comments, leaderboards.

### Scope risk note

The biggest in-scope risk is not a missing feature, it's an over-built one. Several PRD passages (gamification, streaks, Tamagotchi) read as warnings to future contributors, not just to the v1 team. This plan treats those warnings as architectural constraints (§3.6, §4.2) rather than relying on review vigilance alone, because review vigilance is exactly the protection that erodes over a multi-year product lifetime.

---

## 2. Architecture

### 2.1 Service shape

Five services, one shared event log, one simulation core:

1. **Edge/BFF (Backend-for-Frontend)** — serves the initial HTML + state snapshot from a CDN edge (critical for the <500ms time-to-first-bird budget), proxies API calls, terminates auth sessions. Stateless, horizontally scaled.
2. **Account service** — magic-link issuance/verification, session tokens, account settings, export, soft/hard deletion, visit-invitation lifecycle. Owns the only table that stores email (encrypted at rest).
3. **Simulation service** — owns personality vectors, mood state, perch/position state, the species pool, and bird identity. Runs the tick (§5). The only writer of personality and mood. Exposes a read API (snapshot) and an event-ingest API (append-only).
4. **Event log** — append-only store of interaction events (presence pings, offers, listen-in start/end, settle, visit-session events). Partitioned by account UUID (never email — see §3.1). The simulation service's tick is the sole consumer that mutates state from this log; nothing else reads it for product purposes (§3.7 privacy boundary).
5. **Notebook service** — consumes simulation-tick outputs and a curated subset of interaction events to generate field-notebook entries via the narrative-generation pipeline (§6). Entries are stored, immutable once written, keyed by account UUID.

A sixth, non-product system: **telemetry/observability pipeline**, intentionally air-gapped from the simulation database (§3.7).

### 2.2 Client/server split

The client is a rendering and input surface. It never computes personality, mood, or drift. It:

- Pulls state snapshots (full bird state + scene state) from the edge/BFF.
- Renders the scene by interpolating between the last two snapshots.
- Synthesizes audio client-side from a motif-library + the bird's current call-grammar parameters (delivered in the snapshot), via WebAudio.
- Generates purely cosmetic ambient ornaments (leaf/feather drift) with no server roundtrip and no persisted state — these are explicitly excluded from the simulation per `aviary_layout.md`.
- Captures presence signals (`visibilitychange`, `focus`/`blur`, `pointermove`, `keydown`) and batches them into presence-ping events sent to the event-ingest API.
- Sends interaction events (offer, listen-in start/end, settle) to the event-ingest API. Never sends absolute personality/mood values.

This split is the direct implementation of "the client never owns state" (`accounts_sync.md`).

### 2.3 Render pipeline boundary

The render pipeline is a pure function of `(latest snapshot, previous snapshot, client clock, accessibility settings, viewport)`. It does not consult the event log, does not consult personality vectors directly (it consults only what the snapshot exposes, which is rendering-relevant derived state — see §2.4), and never blocks the first paint on anything beyond the initial snapshot fetch (§9).

### 2.4 What's in a snapshot

A snapshot is small (kilobytes, per the perf budget) and contains only what the client needs to render plus the public surface, never raw personality numbers:

```
{
  aviary_id, server_time, day_phase, weather: { type, intensity, started_at },
  birds: [
    {
      bird_id, name, species, mood, mood_started_at,
      perch_zone, perch_slot, facing,
      idle_state, idle_state_seed,
      call_state: { motif_set_id, tempo_band, pitch_band, call_likelihood_band, last_called_at },
      plumage_render_band,      // coarse banding of plumage_saturation, e.g. 1-10, not the raw float
      render_pose_hint
    }
  ],
  active_offer: null | { kind, target_bird_id, started_at, ttl },
  settle_state: { settled: bool, since: timestamp | null },
  notebook_unread_marker: bool,   // existence-only; never a count, see §3.6
  narration_text: string | null  // current screen-reader prose, see §8.1
}
```

Note the personality vector is never serialized to the client even in banded form for raw traits except `plumage_render_band`, which is unavoidable (plumage is directly visual). Boldness, social warmth, vocal frequency, and curiosity influence *which* `idle_state`, `call_state`, and `perch_zone` values the server computes — the client receives only the behavioral output, not the trait that produced it. This is the literal implementation of "personality is never exposed numerically" (`bird_engine.md`): there is no field, banded or otherwise, the client could read to reconstruct boldness.

`notebook_unread_marker` is boolean existence, not a count, to foreclose any "you have 3 new entries" framing creeping toward a badge/counter pattern (§3.6).

---

## 3. Data model

All tables keyed by synthetic UUIDs; email never appears outside `accounts.email_encrypted` (§3.1).

### 3.1 Accounts

```
accounts
  account_id          UUID PK
  email_encrypted     bytea
  email_verified_at   timestamp
  created_at          timestamp
  deletion_status      enum(active, soft_deleted, hard_deleted)
  deletion_requested_at timestamp | null
  visit_notifications_enabled boolean default false   -- §10.4 host opt-in
  pending_email_encrypted bytea | null                -- email-change flow

sessions
  session_id    UUID PK
  account_id    UUID FK
  device_label  text          -- user-set or derived (e.g. "Chrome on macOS"), never email
  created_at    timestamp
  last_seen_at  timestamp
  revoked_at    timestamp | null

magic_links
  link_id       UUID PK
  account_id    UUID FK | null   -- null for not-yet-existing accounts (signup)
  email_encrypted bytea
  token_hash    bytea           -- never store the raw token
  expires_at    timestamp        -- created_at + 15 min
  consumed_at   timestamp | null
```

Every foreign key elsewhere in the system references `account_id`, never `email_encrypted`. This is enforced by a lint rule on the data-access layer (§3.1 reasoning: see `accounts_sync.md` — the synthetic ID rule is named non-negotiable, and a code-level lint rather than a doc convention is the only way that survives team turnover).

### 3.2 Aviary and birds

```
aviaries
  aviary_id     UUID PK
  account_id    UUID FK, unique  -- one aviary per account, enforced by uniqueness constraint
  created_at    timestamp        -- aviary age, drives the new-bird-offer cadence (§4.6)
  timezone      text             -- last-known IANA tz from client, for day/night anchoring

birds
  bird_id           UUID PK   -- stable identity, never reused, never regenerated (bird_engine.md "Bird identity")
  aviary_id         UUID FK
  species_id        text FK -> species_pool
  name              text
  created_at        timestamp  -- adoption date
  -- personality vector, server-authored only:
  boldness          float  -- normalized range, see §3.3
  social_warmth     float
  vocal_frequency   float
  plumage_saturation float
  curiosity         float
  -- mood, server-authored only:
  mood              enum(wary, content, curious, drowsy, alert)  -- §4.3, extensible
  mood_started_at   timestamp
  mood_source       enum(interaction, time_of_day, ambient_event, personality_bias)  -- diagnostic, not exposed to client
  -- position/render state, server-authored:
  perch_zone        enum(front, middle, back)
  perch_slot        int
  last_tick_at      timestamp
```

`species_pool` is a static reference table (~6 rows at v1) with silhouette asset ref, default palette, and call-grammar motif-library ref.

### 3.3 Personality vector ranges and seeding

`[CALIBRATED]` The PRD leaves exact ranges/seed values to "the simulation service" as an implementation detail. This plan fixes: all five traits normalized to `[0.0, 1.0]`, stored as `float4`. Seed values for new birds (starters and later adoptions) are species-flavored: each species in the pool has a seed-range per trait (e.g. a naturally bolder species seeds boldness in `[0.4, 0.6]` rather than `[0.0, 0.2]`), randomized within that band at creation. This gives starter birds an immediate, legible species-flavored personality difference from session one, rather than two identical 0.0-vectors that happen to drift apart — important because the PRD's "feels alive" promise can't depend entirely on a slow-drift signal that takes a week to register even in instruments.

### 3.4 Presence and interaction events (append-only)

```
events
  event_id      UUID PK
  account_id    UUID FK
  aviary_id     UUID FK
  event_type    enum(presence_ping, listen_in_start, listen_in_end, offer, settle, visit_session_start, visit_session_end)
  bird_id       UUID FK | null      -- null for account-level events
  payload       jsonb               -- event-specific, schema per type, §3.5
  occurred_at   timestamp           -- client-observed time
  received_at   timestamp           -- server receipt time, used for tick ordering (§5.1)
  source_session_id UUID FK
```

Append-only, partitioned by `account_id` for the privacy boundary (§3.7) and by `received_at` for retention/compaction. No update or delete path except the hard-deletion job (§3.8).

### 3.5 Event payload schemas (representative)

- `presence_ping`: `{ window_started_at, window_ended_at }` — client batches presence into windows rather than per-pointermove events, to avoid event-log flooding. Client-side debounce: a presence window is open while all three conditions hold (§7.1) and closes when any condition lapses; the client flushes a closed window (or a still-open window every ~60s as a heartbeat) to the event-ingest API.
- `listen_in_start` / `listen_in_end`: `{ bird_id }`, paired by `source_session_id` + bird_id for duration computation.
- `offer`: `{ kind: seed|song|pool, target_bird_id | null }` — `target_bird_id` null is valid; some offers (song, pool) are aviary-wide rather than bird-targeted, per `interactions.md`.
- `settle`: `{ }` — no payload beyond the event itself; settle also implicitly closes any open presence window (§7.2).
- `visit_session_start` / `visit_session_end`: `{ invite_id }` — recorded for the host's visit log (§10.3) only; explicitly never consumed by the drift tick (§10.2 — visitor presence must never drift host birds).

### 3.6 Anti-gamification data-model constraint

No table in this schema stores a derived "visit count," "streak length," "days active," or any per-account aggregate of visit frequency. This is deliberate and structural, not just a UI omission: `non_goals.md` calls gamification "the rule that compounds" — the moment a count exists in the schema, a future PR can expose it cheaply. The only frequency-shaped data that exists is the raw `events` log (needed for drift and notebook generation) and `aviaries.created_at` (needed for the age-gated new-bird-offer feature, which is explicitly not a frequency metric — see `bird_engine.md`: "Not visit count... Not interaction score... Age."). Any future feature proposal that wants a visit-frequency aggregate has to add new schema, which is the friction this plan intends.

### 3.7 Privacy/telemetry boundary

The `events`, `birds`, and `aviaries` tables live in the simulation database. The analytics/telemetry warehouse has zero read access to this database — enforced at the infrastructure level (separate VPC/network policy, no shared credentials, no ETL job between them). Aggregate telemetry (§9.5) is emitted directly by services as pre-aggregated metrics (counters, histograms) with no per-account dimension, never derived by querying the simulation database after the fact. This closes the most common version of this leak, where an engineer adds "just one more dimension" to an analytics query against the primary database.

### 3.8 Account deletion implementation

- `deletion_status = soft_deleted`, `deletion_requested_at = now()` on user-initiated delete. Account remains fully intact; sign-in still works and surfaces a restore affordance ("I changed my mind").
- A scheduled job runs daily, finds accounts with `deletion_requested_at < now() - 30 days` and `deletion_status = soft_deleted`, and hard-deletes: cascading delete across `accounts`, `aviaries`, `birds`, `events`, `magic_links`, `sessions`, notebook entries, visit invitations. Hard delete is a real DELETE, not a tombstone, to honor "every record tied to the account, gone."
- Any in-flight visit invitations are revoked as part of hard deletion.

### 3.9 Field notebook entries

```
notebook_entries
  entry_id      UUID PK
  aviary_id     UUID FK
  generated_at  timestamp
  prose         text       -- final naturalist-voice text, immutable once written
  trigger_kind  enum(scheduled_observation, notable_event)  -- internal only, not exposed
  source_event_ids uuid[]  -- provenance for debugging, not shown to user
```

Read-only to the user (no edit/delete/annotate endpoints exist at all — not just hidden in UI, per `interactions.md`'s "the notebook is read-only").

### 3.10 Visit invitations

```
visit_invitations
  invite_id       UUID PK
  aviary_id       UUID FK (host)
  visitor_email_encrypted bytea
  token_hash      bytea
  status          enum(pending, active, revoked, expired)
  created_at      timestamp
  expires_at      timestamp   -- created_at + 30 days
  first_used_at   timestamp | null
  revoked_at      timestamp | null

visit_sessions
  visit_session_id UUID PK
  invite_id        UUID FK
  started_at       timestamp
  ended_at         timestamp | null
```

`visit_sessions` backs the host's visit log (§10.3); it is never joined into the drift-input query (§10.2).

---

## 4. Simulation engine design

### 4.1 Tick architecture

The simulation service runs a tick per aviary on a fixed cadence. `[CALIBRATED]` Cadence: 60 seconds, matching the PRD's "~once per minute." Implementation: a partitioned tick scheduler (e.g. a sharded cron-like worker pool keyed by `aviary_id` hash) rather than one global tick over all aviaries, so tick latency doesn't scale with total account count — this is what keeps the p99 5s tick-latency alarm (`accessibility_perf.md`) meaningful at scale.

Per tick, for a given aviary:

1. Read all `events` with `received_at` since the aviary's `last_tick_at`, in `received_at` order.
2. Compute presence-time accrued in this window from `presence_ping` events (§7).
3. Compute drift deltas per bird from presence-time + listen-in + offer events (§4.2).
4. Apply drift deltas additively to each bird's personality vector (clamped to `[0,1]`).
5. Recompute mood per bird from: time-of-day, recent interaction events, ambient weather state, personality bias (§4.3).
6. Recompute perch zone/slot per bird from mood + personality (boldness primarily).
7. Resolve bird-to-bird interaction effects (§4.7): call-response chance, wary-mood spread, chorus eligibility.
8. Roll ambient weather state (§4.8) and day-phase (derived from aviary timezone + server clock, not stored as drifted state).
9. Write updated `birds` rows, advance `aviaries... last_tick_at` (modeled as a per-aviary tick cursor), and emit any notebook-trigger candidates to the notebook service (§6).

This is a single-writer model per aviary (only the tick mutates `birds`), which is what makes the no-last-write-wins guarantee (§5.3) hold without needing distributed locking beyond "one tick worker owns one aviary's tick at a time" (a per-aviary advisory lock or partition-owned queue).

### 4.2 Drift function

Drift is a low-pass filter, implemented as an exponential moving average target with a small per-tick step size, applied per trait:

```
delta_raw = w_presence * presence_minutes_this_window
          + w_listenin * listenin_minutes_this_window  (per-bird, social_warmth + vocal_frequency)
          + w_offer_accept * offer_accept_count_this_window  (curiosity)
          + w_offer_proximity * offer_near_bird_count_this_window  (boldness)

delta_clamped = min(delta_raw, max_step_per_tick)   // hard ceiling per trait per tick
new_value = clamp(old_value + delta_clamped, 0.0, 1.0)
```

Key properties enforced by construction, not by convention:

- **No negative term exists in the formula.** There is no decay term, no "ignored_minutes" input, no subtraction anywhere in the drift computation. This is the literal implementation of "drift is monotonic toward expressive... traits move up on positive presence and never move down on neglect" (`bird_engine.md`). A code reviewer can verify monotonicity by confirming the function has no subtraction over personality fields, which is checked by a unit test that fuzzes the function with adversarial event sequences (including zero-event windows) and asserts `new_value >= old_value` for every trait every tick.
- **`max_step_per_tick` is what prevents single-session visible movement.** `[CALIBRATED]` Tuned so that a maximally-engaged single 2-hour session (the high end of plausible session length) moves any one trait by no more than ~0.5–1% of its range — invisible session-to-session, but accumulates to the PRD's calibration targets over the timescales below.
- **Calibration targets** (from `bird_engine.md`): instrument-visible drift after ~1 week of regular visits, user-visible drift after ~3 weeks. This plan operationalizes "regular visits" as `[CALIBRATED]` ~20–30 minutes of presence-time/day, 5+ days/week, and sets weights so that under that usage pattern, a trait starting at 0.3 reaches a numerically-detectable delta (>2% movement, used as the test harness's instrument-visible threshold) within 7 days and a perceptually-distinct delta (>10% movement, calibrated against the perch-zone/idle-state behavioral bands that actually change what the user sees) within 21 days.
- **Weight ordering matches the PRD's stated weight order**: presence-time dominant, listen-in second, offers third, settle non-directional (settle only closes the presence window cleanly — `w_settle` does not appear in the drift formula at all, consistent with `bird_engine.md`: "it does not push drift in any particular direction").

### 4.3 Mood model

Mood is a small state machine per bird, five states initially (`wary, content, curious, drowsy, alert` — PRD says "the exact set is finalized in implementation," so this plan fixes these five as the v1 set and leaves the enum extensible at the schema level for post-v1 tuning).

Transition inputs, combined into a weighted score per candidate mood state each tick, with personality acting as a per-bird bias on transition thresholds (a high-boldness bird needs a stronger wary-pushing input to actually transition to wary):

- Recent interaction events (offer-accepted nudges toward `content`; an aviary-wide alarm-call event nudges nearby birds toward `wary`).
- Time-of-day (drowsy bias near dusk/night, alert bias in early morning) — computed from the aviary's stored `timezone`, not server local time, per `aviary_layout.md`'s "local-time anchoring."
- Ambient weather (rain dampens vocal-frequency-linked alert/curious propensity; wind raises alert in some birds, wary in others — species-flavored).
- The bird's own personality vector, as a bias term on every transition threshold, not as a separate input — this directly implements "the bird's own personality vector (a high-boldness bird is less likely to enter wary even on the same input)."

Mood does not reset between sessions: it is read from the persisted `birds.mood` row at the start of every tick, never reinitialized at session boundary, which is enforced by construction — there is no "session start" event type that the mood transition function consumes, and `mood` is mutated only inside the tick (§4.1 step 5), never by any session-lifecycle handler.

### 4.4 Idle motion mapping (server-computed, client-rendered)

The tick computes an `idle_state` enum per bird (e.g. `preening, scanning, head_tilt, fluffed_low, alert_watch, ...`) as a function of current mood, with some intra-mood variety so the same mood doesn't always render identically. The client receives `idle_state` + a `idle_state_seed` (for procedural micro-variation within that state — e.g. varying the preen-cycle timing) and renders accordingly; the client never infers idle state from mood itself, keeping the mood→motion mapping server-owned and easy to retune without a client release.

### 4.5 Call grammar runtime

Call grammar is a client-side runtime: the server sends `call_state` (motif_set_id keyed to the bird's species, tempo_band and pitch_band derived from vocal_frequency + mood + personality, and `last_called_at`/`call_likelihood_band` for scheduling). The client's WebAudio engine:

- Holds a small library of motif fragments per species (shipped as part of the species-pool asset bundle, lazy-loaded per adopted species so the bundle budget isn't paying for unadopted species — see §9.1).
- Synthesizes a call by selecting and recombining motifs at runtime with per-call randomized timing/pitch jitter within the server-specified bands — never replaying a fixed sample, which is what "procedural, not recorded" requires structurally (a motif fragment alone is not a complete call; recombination happens client-side per call event).
- Schedules calls probabilistically using `call_likelihood_band` and an internal Poisson-ish process per bird, biased so two high-vocal-frequency birds calling in the same scheduling window produce a genuine chorus event (overlapping, separately-synthesized calls mixed in the same WebAudio graph) rather than a server-dictated "now both birds call" trigger — this avoids canned-feeling simultaneity and lets recombination noise differ between birds in the same chorus, which is what avoids the phase-canceling artifact the PRD calls out for stacked identical loops.
- Recognizability constraint: each species' motif library is built around 2–3 fixed melodic/timbral signature elements that persist across all of a bird's synthesized calls regardless of mood/personality drift, with mood/personality only modulating tempo, pitch offset, and motif selection frequency around that fixed signature — this is what keeps "a user who has spent two weeks with Pip should know Pip's call by ear... even when Pip's vocal frequency has drifted up" true: the signature elements don't move, only the surrounding variation does.

### 4.6 New-bird offer cadence (age-gated)

A scheduled job (not part of the per-minute tick — this runs at a much lower frequency, e.g. daily) evaluates each aviary's `created_at` age against a `[CALIBRATED]` age curve: third bird offered at ~8–10 weeks, fourth at ~16–20 weeks, fifth/sixth/seventh spaced further out, capped at seven and never auto-offered again once seven is reached. The offer is presented as an in-aviary event (a new bird arriving, framed identically to the original two starters — "the birds that arrived," not a catalog pick) rather than a notification, consistent with `interactions.md`'s "notice, never announce." The age-gate job reads only `aviaries.created_at` and `birds` count — it explicitly does not read the `events` table, which is what keeps the mechanic verifiably untied to visit frequency or interaction score per `bird_engine.md`.

### 4.7 Bird-to-bird interaction

Within a tick, after individual mood/perch updates, a second pass resolves cross-bird effects for aviaries with 2+ birds:

- **Call-response**: if bird A's `call_likelihood_band` fires in a tick window, birds with high social_warmth and proximity (same or adjacent perch zone) get an elevated chance of also firing in the same window — implemented as a conditional probability bump applied before the per-bird call scheduling in §4.5, computed server-side and expressed to the client only as each bird's own `call_state`.
- **Wary spread**: an aviary-wide ambient alarm-type event (modeled as a rare ambient-event type, distinct from weather) pushes nearby birds' mood-transition score toward `wary`, dampened by each bird's personality bias (§4.3).
- **Chorus emergence**: when 2+ birds with high vocal_frequency have overlapping elevated `call_likelihood_band` in the same tick, the tick flags a chorus window in the snapshot's weather/ambient state (not a separate field — modeled as a transient ambient_event) so the client can lean into a slightly richer audio mix during that window, but the actual chorus *sound* is emergent from independent client-side synthesis (§4.5), not server-authored audio.

### 4.8 Ambient weather

A separate, lower-frequency scheduled process per aviary (independent of the per-minute tick, since weather is described as "a few times a week") rolls a weather event with small probability; when it fires, it sets a `weather` object (type, intensity, started_at, planned duration) on the aviary, visible in the snapshot, and the next several ticks apply the short-lived mood dampening/alerting effects described in `aviary_layout.md` (rain dampens vocal frequency briefly; wind raises alert/wary in different birds) as a temporary modifier layered on top of the normal mood-transition score, decaying back to zero influence once the weather event's duration elapses.

---

## 5. Sync model

### 5.1 Single canonical writer

The simulation tick (§4.1) is the only process that writes `birds.boldness/social_warmth/vocal_frequency/plumage_saturation/curiosity/mood/perch_zone/perch_slot`. All API write paths for clients go through the event-ingest API (`events` table), never directly to `birds`. This is enforced at the database layer: the simulation service's tick worker is the only credential with UPDATE grant on `birds`' state columns; the account/edge services have no such grant.

### 5.2 Snapshot propagation

Both laptop and phone clients call the same read API (`GET /aviary/snapshot`), authenticated by their own session token but both resolving to the same `aviary_id` via `account_id`. There is no per-device snapshot variant and no per-device cache that could diverge — the read path is a straight read of the current `birds`/`aviaries` rows, optionally behind a short-TTL (a few seconds) read-through cache for load, never a per-device materialized copy.

### 5.3 No last-write-wins; additive deltas only

Because clients never submit absolute personality values (§5.1) and the tick processes the event log in `received_at` order across *all* devices for that account uniformly (§4.1 step 1 reads all unconsumed events regardless of `source_session_id`), there is no scenario where one device's write clobbers another's. A laptop session's events and a phone session's events occurring in an overlapping window both get folded into the same tick's delta computation — order within the window doesn't matter because the inputs are additive (presence-minutes, listen-in-minutes, offer counts), not last-value-wins fields. This is the direct implementation of `accounts_sync.md`'s "No last-write-wins for personality state."

### 5.4 Client snapshot refresh triggers

The client refetches a snapshot on: `visibilitychange` to visible, a detected long render-frame gap (>~2s, indicating suspend/resume), and a low-frequency keepalive poll while visible (`[CALIBRATED]` every 20–30 seconds — frequent enough that a second device's changes become visible within about the cadence of one tick, infrequent enough to stay well inside the perf and battery budget). Between snapshots, the client interpolates only cosmetic motion (position, pose) — it never extrapolates mood or personality changes locally.

---

## 6. Field notebook / narrative generation pipeline

### 6.1 Triggering

Two trigger kinds, matching `notebook_entries.trigger_kind`:

- **scheduled_observation**: a low-frequency background process (`[CALIBRATED]` roughly every 2–4 days per actively-visited aviary, randomized to avoid a fixed cadence the user could learn) samples the aviary's current state and recent tick history and, if nothing more notable has happened recently, writes a low-key observational entry.
- **notable_event**: emitted as a candidate from the tick itself (§4.1 step 9) when a tick detects something structurally interesting — a first-time-today ordering (Pip greeted before Wren), a long quiet stretch, a mood/weather coincidence, a chorus event, a bird crossing a perceptible drift threshold. These candidates are rate-limited (`[CALIBRATED]` no more than ~1 notable entry per 1–2 days even if multiple qualify) so notable events don't turn the notebook into a feed, per `interactions.md`'s explicit sparsity requirement ("entries are rare... not on every session").

### 6.2 Generation

A notable-event or scheduled-observation candidate carries structured facts (which birds, what state, what changed) — never raw personality numbers (§2.4's no-numeric-exposure rule applies here too) — into a generation step that produces naturalist prose. This plan specifies a template-grammar approach rather than a general-purpose LLM call in the generation hot path: a constrained, hand-authored phrase-grammar (parameterized by bird name, species, mood, perch zone, time-of-day, and the specific notable fact) that composes naturalist sentences from vetted fragments, with enough combinatorial variety (per-species vocabulary banks, varied sentence templates) to avoid repetition fatigue. Rationale: the voice is the product's most exposed surface (`interactions.md`: "a stock event-log treatment would break the spell across the entire product"), and a constrained generator is auditable and won't drift off-voice the way a raw LLM completion could; it's also far cheaper and faster than an LLM call per entry at the cadence needed. The generation grammar is built and reviewed by a writer/engineer pair, not auto-generated from a model with no voice guardrail.

### 6.3 Entry storage and access

Entries are written once, immutable (§3.9), and served via a paginated read API ordered newest-first, scrolling back indefinitely (no archival/hiding), with no edit/delete/annotate endpoints existing at all.

---

## 7. Presence engine

### 7.1 Client-side presence detection

Implemented as a small dedicated client module, independent of the rendering pipeline, that tracks three boolean signals continuously:

- `document.visibilityState === 'visible'`
- the document has window focus (`document.hasFocus()`, updated on `focus`/`blur`)
- "recent activity": a `pointermove` or `keydown` has occurred within the activity window

`[CALIBRATED]` Activity window: 5 minutes, leaning toward the long side per the PRD's explicit guidance ("leaning toward the longer side because watching birds without moving is the actual product"). A presence-event is true only while all three booleans are simultaneously true; the module opens a presence window the instant all three become true and closes it the instant any one becomes false, sending `presence_ping` events with the window's start/end (§3.5) — batched/flushed periodically (every ~60s while open, and immediately on close) rather than one event per micro-window, to bound event-log volume.

### 7.2 Settle and tab-close symmetry

Both settle (an explicit client action) and tab-close (detected via `visibilitychange`/`pagehide`, which closes any open presence window the same way a focus-loss would) terminate the presence window through the *same* code path — there is no special "penalize ungraceful exit" branch. This directly implements `interactions.md`'s "the engine treats the end of presence the same way regardless of whether the user clicked settle first."

### 7.3 What presence does and doesn't drive

Presence-time is the dominant drift input (§4.2) but is explicitly never used to compute or surface any frequency/count metric to the user (§3.6) — the same raw presence windows feed two consumers (drift computation and, indirectly, notable-event detection for the notebook) but never a third consumer that would expose "days visited" or similar.

---

## 8. Frontend rendering pipeline

### 8.1 Scene composition

Layered canvas/WebGL (or a lightweight 2D-canvas + DOM-overlay hybrid, finalized in the rendering spec referenced by the PRD) with three render planes: background (sky/foliage, day-night gradient), middle (perches + birds), foreground (occasional branch/leaf occlusion) — implementing the "quiet foreground/background separation" with subtle, non-layered-illustration parallax.

The scene boots directly into motion: on snapshot arrival, every bird is placed at its current `perch_zone`/`perch_slot` already mid-`idle_state` (not at a neutral/idle-zero pose), ambient leaf/feather ornament generators start immediately, and day/night gradient is set from `day_phase` with no transition-in. There is no loading-spinner state in the happy path; on a slow snapshot fetch, the fallback is the quiet-field placeholder described in §9.3, never a spinner.

### 8.2 Idle micro-motion and mood-shaped rendering

Each `idle_state` value maps to a small library of procedural motion clips/pose-interpolation curves (preening, scanning, head-tilt-toward-sound, weight-shuffle) parameterized by the `idle_state_seed` for natural variation between birds in the same state and between repeat occurrences of the same state on the same bird. Mood is never rendered as a label, icon, or color tag — it is read purely through which `idle_state` family and perch zone the bird is in, per `bird_engine.md`'s "the moment the user has to be told what a bird is feeling, the product has failed."

### 8.3 Return-greeting implementation

On snapshot arrival at session start (fresh nav, tab refocus after absence, return after long absence), the client computes an "absence length" locally (time since last known presence-window end, tracked in client storage) and selects which bird greets based on a `[CALIBRATED]` weighted choice: each bird's `boldness` *band* (already present indirectly via behavioral fields, not raw boldness — the *selection itself* happens server-side at snapshot-generation time, not client-side, specifically so the client never needs the raw trait) and `mood` bias the likelihood of being "the noticing bird" for this session-start event, with the response *amplitude* (glance vs. call vs. approach-and-call) scaled by the absence-length bucket the server computes from the account's last recorded presence-window end. The server includes a `greeting` directive in the snapshot for session-start fetches specifically (`{ bird_id, amplitude_band, stagger_offset_ms }` for one or, rarely, more birds), and the client renders it as a one-time animation cue layered on top of normal idle rendering — never as a separate "arrival" scene state. If multiple birds greet, the client staggers their start times by the server-provided random offsets rather than firing them in the same frame, per `interactions.md`'s explicit "stagger... rather than firing in unison" requirement. Critically: there is no toast, banner, modal, or text string rendered alongside this — the greeting is exclusively the bird's animation + call, with the narration channel (§8.5) optionally describing it in prose for screen-reader users only, never as a visual banner.

### 8.4 Listen-in mix and visual focus

Listen-in is a client-local interaction (instant from the user's perspective) that (a) ramps the focused bird's WebAudio gain node up and all other birds' gain nodes down over a slow ramp (`[CALIBRATED]` ~800ms–1.2s ease, not a hard cut, per `interactions.md`), with other birds' gain floor never reaching zero, and (b) sends a `listen_in_start` event to the server for drift accounting. Disengage (re-click, click-elsewhere, focus-away, click a different bird) reverses both effects with the same ramp curve and sends `listen_in_end`. The audio ramp logic lives entirely client-side for responsiveness — it does not wait on a server round-trip — while the event log captures start/end for the tick to fold into drift later.

### 8.5 Reduced-motion rendering mode

A parallel rendering mode (not a flag that disables the normal renderer's animation loop) selected when `prefers-reduced-motion` is set or the user opts in via accessibility settings. In this mode, the same `idle_state`/mood/perch data drives a different renderer: instead of continuous motion clips, the bird is rendered as a sequence of distinct held poses (preen-pose-1, preen-pose-2, ...) connected by slow cross-fades; flight/perch-change transitions are perch-to-perch cross-fades rather than animated flight paths; ambient leaf/feather drift ornaments are omitted entirely; the day/night color gradient still shifts but on a slowed easing curve. This is built and tested as its own designed surface from day one (it ships in the same release as the primary renderer, not as a follow-up), per `accessibility_perf.md`'s explicit requirement that it not land as a "v1.1 fix."

### 8.6 Responsive layout

The scene container queries viewport width/height and recomputes perch x-positions and inter-perch spacing to keep all birds in frame at all aspect ratios from narrow-phone-portrait to wide-desktop, never cropping a bird or scaling it off-canvas; perch *zone* (front/middle/back) maps to a y-band, perch *slot* within a zone maps to an x-position computed from available width divided by birds-in-that-zone, recalculated on resize.

---

## 9. Audio pipeline

### 9.1 Asset strategy

The species-pool motif libraries (§4.5) are small parameterized waveform-generation definitions (oscillator types, envelope shapes, motif-fragment sequences expressed as data, not audio files) rather than audio assets — this is what keeps the bundle small while supporting per-call variation. Only the motif libraries for the account's *currently adopted* species are loaded (lazy-loaded on first need, e.g. when a bird's species is in the initial snapshot); the full six-species pool is not loaded upfront for a two-bird aviary.

### 9.2 Synthesis and mixing

One `AudioContext` per session, one gain node per currently-rendered bird feeding a shared master bus, plus a "weather/ambient" bed at a low fixed level. Calls are scheduled and synthesized just-in-time (each call event creates short-lived oscillator/noise nodes that are started, played, and disposed — no persistent per-call buffer retained, which is required for the no-memory-growth budget in §11.4) using Web Audio's scheduling primitives so multiple birds' calls can genuinely overlap and phase-interact rather than being pre-mixed.

### 9.3 Listen-in mix decay

Implemented as Web Audio `GainNode.gain` automation (`linearRampToValueAtTime` or `setTargetAtTime` for a more natural decay curve) rather than discrete volume steps, giving the "rise/fall" feel `interactions.md` requires rather than a stepped channel-switch feel.

### 9.4 WebAudio fallback

On `AudioContext` construction failure or permission denial, the client falls back to a silent audio path with captions forced on by default (overriding the user's caption preference toggle only in the sense that it's pre-enabled, not that the toggle is removed) — no recorded-audio fallback exists in the codebase at all, which removes the temptation to "just ship an MP3 fallback" later, since there's no code path that even accepts an audio file asset.

---

## 10. Visit-invitation flow (social, optional)

### 10.1 Invite lifecycle

Host enters visitor email in account settings → account service creates a `visit_invitations` row (`status = pending`), emails a one-time link via the account service's existing email infra (shared with magic-link, different template, matter-of-fact voice). Visitor follows the link → token validated against `token_hash` → on first valid use, `status` transitions to `active` and `first_used_at` is set; a `visit_sessions` row opens. Subsequent visits via the same link reuse the `active` invite (it's not single-use in the sense of one viewing — it's revocable/expiring, not one-shot, since the PRD describes ongoing visit access until revoked or expired, not a single viewing).

### 10.2 Read-only enforcement

The visitor's client authenticates via the invite token (not an account session) and hits a *separate, read-only* API surface (`GET /visit/:invite_token/snapshot`) that returns the same snapshot shape as the host's `GET /aviary/snapshot` for that aviary, but the visitor client ships with no event-ingest calls wired up at all for offer/listen-in/settle/presence — not just hidden buttons, but no client code path that can construct those requests, and the server-side visit-read endpoint is served by a credential with no write access to the event-ingest API. This double enforcement (client has no write capability; server endpoint has no write grant) is what guarantees "the simulation does not record presence-time or interaction events from a visitor" structurally rather than by trusting the visitor client not to call write endpoints. A `visit_session_start`/`visit_session_end` pair *is* recorded, but exclusively for the host's visit log (§10.3) via a distinct, separate event type that the drift-computation step (§4.1 step 2-3) explicitly excludes from its query (filters `event_type NOT IN (visit_session_start, visit_session_end)` when computing presence/drift inputs — visit events exist in the same `events` table for storage convenience but are structurally invisible to the tick's drift math).

### 10.3 Visit log

Host-facing read API in account settings listing `visit_invitations` joined with `visit_sessions`, showing visitor email (decrypted only in this host-facing, host-authenticated context), date, and approximate duration (computed from `visit_sessions.started_at`/`ended_at`), ordered most-recent-first. No badge, no unread marker, no count surfaced anywhere outside this on-demand page — consistent with the no-counter rule (§3.6).

### 10.4 Revocation and expiration

Revocation (host action) sets `status = revoked`, `revoked_at = now()` immediately; the visit-read endpoint checks `status` on every snapshot request (not just at link-click time), so an active visitor's *next* snapshot poll (within the same ~20-30s cadence as §5.4) returns a matter-of-fact "visit no longer available" response instead of a snapshot, which the visitor client renders as a system-voice surface. A daily job marks `pending` invites with `expires_at < now()` as `status = expired`.

### 10.5 Visit notifications (off by default)

`accounts.visit_notifications_enabled` (default false) gates whether a `visit_session_start` event triggers a notification (email, matter-of-fact voice) to the host. When false (the default for every account, with no onboarding prompt ever surfacing this toggle), `visit_session_start` writes only to the log; no notification side-effect fires.

---

## 11. Accessibility surfaces

### 11.1 Screen-reader narration generation

A narration text stream, generated by the *same* narrative-generation pipeline data inputs as the field notebook (§6.2) but at a different cadence and granularity: a lightweight version of the phrase-grammar runs continuously (server-side, attached to the snapshot as `narration_text`, refreshed alongside the normal snapshot poll cadence but gated to actually change only on the cadence below) describing current aviary state in the same naturalist voice. `[CALIBRATED]` Cadence: one narration update per 30–60s at idle (per the PRD's explicit range), with priority-bumped narration emitted promptly (within the next client poll, not held to the 30-60s idle cadence) for user-initiated/notable events: return-greeting, offer reactions, settle. The client exposes `narration_text` via an `aria-live="polite"` region (`aria-live="assertive"` reserved only for the priority-bumped event class, to avoid overwhelming the screen-reader queue per the PRD's explicit warning about queue-overwhelm).

### 11.2 Reduced-motion mode

Covered in §8.5 as a first-class rendering mode, shipped in the same release.

### 11.3 Call captioning

Caption text is generated at call-synthesis time client-side (§4.5/§9.2): when the call-grammar runtime selects and recombines motifs for an actual call event, it derives a short caption string from the same motif/parameter choices (e.g. mapping motif shape + tempo/pitch band to one of a set of caption-template fragments — "a soft three-note rise," "a low trill, paused, low trill again") so the caption always matches what was actually synthesized, not a fixed per-call-type string. Captions render as small fading text anchored near the calling bird's position, opt-in via accessibility settings, with audio-unavailable (§9.4) forcing captions on by default.

### 11.4 Keyboard navigation

Full keyboard map implemented as part of the core interaction layer, not bolted on: `Tab` traverses top-bar items then into the aviary (first bird focused on entry), arrow keys move focus between birds (focus order following perch zone/slot, front-to-back then left-to-right, recalculated whenever perch assignments change), `Enter` triggers listen-in on the focused bird, `Escape` exits listen-in, a top-bar keyboard shortcut opens the offer affordance (fully keyboard-navigable once open), and the settle control is a normal focusable top-bar button. Focus rings use a fixed-contrast outline treatment (not theme-blended into the aviary palette) so they remain visible against both bright daytime and dim night scene states.

### 11.5 Contrast

All user-copy surfaces (top bar labels, settings, account/error surfaces, visible captions, visually-displayed narration) pass WCAG AA at minimum, verified by an automated contrast-check in CI against the design system's token values (not a one-time manual audit) so future palette tweaks can't silently regress contrast.

---

## 12. Performance budgets and observability

### 12.1 Bundle budget (<2MB gzipped initial JS)

Enforced by a CI bundle-size gate on the initial-route bundle specifically (not total app size — code-splitting, §12.5, keeps secondary surfaces out of the initial measurement). Procedural audio (no audio files in the initial bundle) and procedurally-generated/small-SVG bird visuals are what make this budget achievable; the species-pool asset strategy (§9.1) keeps unadopted-species data out of the initial payload too.

### 12.2 Time-to-first-bird (<500ms on mid-tier mobile/4G)

Achieved via: (a) the initial snapshot delivered from a CDN edge alongside the HTML shell (server-side rendered or edge-injected initial state, avoiding a separate client-initiated fetch-then-render round trip on cold load), (b) a render path that draws the first bird as soon as its pose/position data is available, without waiting on non-critical assets (notebook data, account settings bundle, unadopted-species motif libraries) which are deferred, (c) the bundle budget itself. Measured continuously via synthetic monitoring (§12.6), not just at ship time.

### 12.3 60fps idle motion, 5-year-old laptop

Idle motion render loop budgeted to a fixed per-frame cost ceiling; profiled against a representative low-end device tier in CI performance testing (not just modern dev-machine testing). Procedural ornament generation (leaves/feathers) is rate-limited and object-pooled rather than allocating new DOM/canvas objects per ornament.

### 12.4 No memory growth over 30 minutes

Audio: no persistent per-call buffer retention (§9.2) — nodes are created and disposed per call. Notebook: entries scrolled out of the viewport are virtualized/unmounted, not retained in a growing in-memory list. Workers/AudioContexts: a single bounded set, never recreated per-interaction. This is enforced as an actual CI test: a scripted 30-minute (or accelerated/simulated-time-compressed) session run in a headless browser instrumented with heap snapshots at intervals, asserting flat memory within a tolerance band — not a manual guideline.

### 12.5 Code-splitting

Initial bundle = core aviary scene renderer, core audio engine, presence module, snapshot client. Deferred/lazy-loaded: account settings, accessibility settings panel (loaded eagerly enough to not block a user who needs it on first load, but as a separate chunk), visit-invitation flow, field notebook UI, unadopted-species motif libraries.

### 12.6 Observability

Synthetic monitoring: scheduled headless-browser runs from multiple geographies hitting the real aviary load path, measuring time-to-first-bird and bundle delivery timing. Aggregate RUM: page-load timing, first-bird-render timing, render-frame-time histograms, audio-context error counts, simulation-tick latency — all collected with no per-account dimension, emitted directly by services/client as pre-aggregated metrics (§3.7), never derived from the simulation database. Alarm: simulation-tick latency p99 > 5s pages the on-call simulation-service owner.

### 12.7 Browser support

Last two major versions of Chrome/Safari/Firefox/Edge are the supported/tested matrix; older browsers get a matter-of-fact unsupported-browser page (system-voice, per `product_brief.md`'s voice-exception rule) rather than a degraded attempt to render.

---

## 13. Rollout

### 13.1 Phasing

1. **Internal alpha**: simulation engine + tick + single-bird rendering, no auth (test accounts), used to calibrate drift weights (§4.2) and mood-transition thresholds (§4.3) against real usage patterns before any external user sees the product — calibration work needs real session-shaped data, not just unit-test fuzzing.
2. **Closed beta**: full account/auth/sync stack, two-starter-bird adoption flow, core interactions (greeting, listen-in, offer, settle), field notebook, accessibility surfaces all present (not deferred — per `accessibility_perf.md`, accessibility ships with the rest, including in beta). Visit-invitation feature included but limited to a small allow-listed cohort initially to validate the read-only enforcement boundary (§10.2) under real cross-account traffic before wider exposure.
3. **V1 launch**: full feature set as scoped (§1), performance budgets gated in CI and verified against the synthetic-monitoring baseline established in beta, browser-support matrix finalized.

### 13.2 Birds-per-aviary ramp

The age-gated new-bird-offer cadence (§4.6) is itself the ramp mechanism — there's no separate "ramp birds-per-aviary" rollout lever beyond tuning the age-curve constants, which this plan treats as a post-launch-tunable config (not a code change) so the age thresholds can be calibrated against real long-term cohort behavior without a deploy.

### 13.3 Day-one instrumentation

Ship with the full observability story (§12.6) live from v1, not added after — tick latency, render timing, audio-context error rates, and the drift-calibration instrumentation (§4.2's instrument-visible-by-week-1 threshold) all need to be live from the first real cohort to validate the calibration targets against actual usage, since those targets can only be checked against real multi-week account histories.

---

## 14. Risks

### 14.1 Drift calibration risk (highest risk in the plan)

The instrument-visible-week-1 / user-visible-week-3 targets (§4.2) are calibrated against an assumed "regular visit" usage pattern that is itself a planning estimate, not measured data — real usage distributions (session length, frequency) won't be known until the alpha/beta phases. Mitigation: drift weights and `max_step_per_tick` are runtime-tunable config, not hardcoded constants, specifically so the alpha phase (§13.1) can recalibrate against real session-length/frequency distributions before any external user's drift history is affected; once a real cohort exists, retroactively changing weights would change the felt pace of drift for accounts already mid-relationship with their birds, which the product can't really afford post-launch, so calibration work has to land *before* the closed beta, not be deferred to "tune after launch."

### 14.2 Sync correctness risk

The no-last-write-wins guarantee (§5.3) depends on every personality-affecting client interaction routing through the event log and never a direct write path. The main way this regresses is an engineer adding a "quick" direct-write debug endpoint or admin tool that bypasses the tick for support purposes (e.g. "let me just reset this user's bird's mood for a support ticket"). Mitigation: no direct-write path exists in the codebase at all (not gated by a feature flag — simply not implemented), and any future support tooling for personality state has to go through synthetic event injection into the log, preserving the tick as sole writer even for support cases.

### 14.3 Audio uncanniness risk

Procedural synthesis is the harder technical path and carries real risk of sounding worse than a well-produced recorded loop, especially early in development before the motif libraries and synthesis parameters are tuned. Mitigation: the alpha phase (§13.1) is explicitly scoped to include audio-engine iteration time before any external listener hears it, and the per-species "fixed signature elements + variable surround" design (§4.5) is structured so that even early/rough synthesis preserves recognizability (the thing users will actually judge "is this the same bird I know") even if the raw timbre quality needs more tuning passes than the visual side.

### 14.4 Accessibility regression risk

The reduced-motion and narration surfaces (§8.5, §11.1) are designed as first-class parallel surfaces rather than fallbacks, which means they require ongoing parallel maintenance as the primary renderer evolves — any future visual feature added to the primary scene needs a corresponding reduced-motion treatment and narration-grammar update, or the two surfaces drift apart and the reduced-motion/screen-reader product quietly becomes a stale "v1 snapshot" while the primary surface keeps evolving. Mitigation: treat "ships a reduced-motion + narration treatment" as a hard merge-gate for any PR that adds a new visible aviary-scene behavior, enforced by a PR checklist/review requirement rather than relying on it being remembered.

### 14.5 Gamification creep risk

Covered architecturally in §3.6, but named again here because the PRD itself identifies this as the single most likely failure mode over the product's lifetime ("the foothold that makes the next harmless engagement feature easier to argue for"). Mitigation beyond the schema-level friction in §3.6: any future feature proposal that would introduce a per-account frequency/count/streak concept requires explicit named sign-off referencing this section, not just normal code review, because normal code review is exactly the protection that erodes over a multi-year, multi-team product lifetime as the original PRD's reasoning fades from institutional memory.

### 14.6 Visit read-only boundary risk

The structural double-enforcement in §10.2 (no client write path + no server write grant) is the right design, but a future feature that wants to add *any* visitor-facing interactivity (even something that looks harmless, like a "wave" gesture) would need to either go through full design review against `social_optional.md`'s explicit "visitor cannot trigger anything" rule, or be rejected — this is named as a risk because "let visitors do one small thing" is a plausible future feature request that directly contradicts the read-only design this plan implements.
