# Pocket Aviary — V1 Implementation Plan

## 0. Reading this plan

This plan is organized so a frontier engineering team can execute it without
returning to the PRD for clarification. Where the PRD leaves a decision
explicitly open ("exact cadence calibrated during build"), this plan makes a
defensible default and flags it as `[CALIBRATE]` so it's visible as a tunable
rather than buried. Where the PRD is silent on something engineering needs
(e.g., specific tech choices), this plan makes a call and flags it as
`[DEFAULT]`.

---

## 1. Scope

### 1.1 In scope for v1

- Single-user accounts, magic-link auth, one canonical aviary per account.
- Two starter birds at adoption, server-controlled growth to a 7-bird cap,
  paced by aviary age (not engagement).
- Server-side simulation tick driving personality drift and mood.
- Client rendering: full visual scene, idle motion, day/night, weather,
  return-greeting, listen-in, offer, settle.
- Procedural WebAudio call synthesis and chorus mixing; silence+captions
  fallback when WebAudio is unavailable.
- Field notebook (auto-generated, read-only, sparse).
- Multi-device sync via server-canonical state (no client-side merge logic).
- Visit-invitation (read-only ambient view, opt-in, revocable).
- Screen-reader narration (naturalist prose), reduced-motion mode (designed
  surface, not animations-off), call captions, full keyboard navigation,
  WCAG AA contrast.
- Account export, soft-then-hard account deletion (30-day window).
- Aggregate-only operational telemetry; per-bird/per-account interaction data
  never leaves the simulation database boundary.

### 1.2 Explicitly out of scope (respecting `non_goals.md`)

- Native iOS/Android apps. No native-client constraints baked into the
  protocol or data model at this stage — the wire protocol uses ordinary
  HTTP/JSON snapshots, nothing that assumes a particular client runtime, but
  no native SDK work is planned or scaffolded.
- Any gamification primitive: achievements, streaks, levels, scores, badges,
  visit calendars, "birds adopted: N" counters. These must not exist even as
  disabled/internal-only code paths — see §13 (Risks) for why "build it
  disabled" is itself a risk here.
- Tamagotchi mechanics: no death, no hunger, no decaying happiness meter, no
  distress states. Drift is monotonic toward expressive only (§4.3).
- Social-network surfaces beyond the single visit affordance: no profiles,
  follows, public feed, discovery, friend-of-friend chains, comments.
- Multi-aviary accounts, shared/household aviaries, customizable scenes,
  payments, push notifications.

### 1.3 v1 boundary calls (made here, not asked back)

- **`[DEFAULT]` Bird species pool count: 6**, per `bird_engine.md`. Each
  species is defined by a static asset (silhouette + palette + call-motif
  library) versioned in a species registry service/table — see §3.6.
- **`[DEFAULT]` Aviary-age bird-unlock schedule**: third bird unlocked at 90
  days of account age, then roughly one additional bird per ~120 days,
  capped at 7. This is a server-side cron/scheduled job reading account
  creation timestamp, not engagement signal. Exact pacing is a product/design
  tuning knob exposed via a config table, not hardcoded, so it can be
  recalibrated without a deploy.

---

## 2. Architecture

### 2.1 Service shape

Three deployable units plus a CDN edge:

1. **Edge/CDN** — serves the static client bundle and, critically, a fast
   path for initial state-snapshot delivery alongside the HTML shell (see
   §9.2, the 500ms time-to-first-bird budget). `[DEFAULT]`: snapshot
   pre-fetch is implemented as an edge function that reads a replicated
   read-optimized snapshot store (see §2.3) and inlines the first snapshot
   into the HTML response as a `<script type="application/json">` block, so
   the client doesn't need a second round-trip before first paint.

2. **API service** (stateless, horizontally scaled) — handles:
   - Auth (magic-link issuance/verification, session tokens).
   - Snapshot reads (`GET /aviary/state`).
   - Event writes (`POST /aviary/events`) — append-only, never mutates
     personality directly (§6.4).
   - Account management (export, deletion, email change, session revocation).
   - Visit invitation management.
   - Notebook reads.

3. **Simulation service** (the tick worker) — a scheduled/continuously
   running worker, decoupled from the API service, that:
   - Reads the event log since its last cursor, per account.
   - Computes mood transitions and personality deltas.
   - Advances ambient world state (weather, bird-to-bird interaction,
     greeting eligibility).
   - Writes canonical aviary state and a denormalized read snapshot.
   - Runs whether or not any client is connected (§4, §6.1).

This separation matters because the tick's correctness requirements (ordered
event consumption, exclusive write access to personality vectors) are
different from the API's requirements (low-latency reads, horizontal
scaling, no shared mutable state). Conflating them would make it tempting to
let an API request mutate personality synchronously, which reintroduces the
last-write-wins failure mode the PRD explicitly rules out (§6.4).

### 2.2 Client/server split

The client **never** simulates. It:
- Pulls snapshots and interpolates positions/animations between them.
- Renders idle micro-motion, ambient ornaments (leaves, feathers — these are
  pure client-side rendering, not server state, per `aviary_layout.md`).
- Synthesizes audio from the call grammar using parameters embedded in the
  snapshot (current mood, vocal-frequency-derived timing, personality-shaped
  pitch ranges) — the *grammar* is shared code (compiled to both server
  validation/instrumentation and client synthesis), but the actual audio
  buffer generation happens client-side only.
- Writes interaction events to the API; never computes or sends absolute
  personality/mood values.

The server:
- Owns all canonical state.
- Computes the call-grammar *parameters* a client should use for the current
  moment (motif selection, tempo, pitch envelope ranges) — not waveforms —
  so calls remain server-determined in character but client-synthesized in
  audio output. This split keeps "procedural, not looped" honest while
  keeping bytes-over-the-wire small.

### 2.3 Render-pipeline boundary

A clean boundary between **simulation state** (mood, personality, perch
assignment, active animation/transition descriptor) and **render state**
(interpolated position this frame, ambient ornament instances, parallax
offsets, idle-motion pose blend weights). The snapshot the server emits is
simulation state only — small, infrequent (≥ once per tick, plus targeted
event-triggered pushes for the return-greeting, see §5.1). The client's
render loop runs at 60fps independently, interpolating toward the latest
simulation-state target. This boundary is what lets the client "stop
rendering when hidden" (§9) without ever stopping the simulation, and what
lets idle micro-motion feel continuous despite snapshots arriving only once
a minute.

### 2.4 `[DEFAULT]` Storage architecture

- **Source-of-truth store**: a transactional database (e.g., a managed
  Postgres) holding accounts, birds (personality vectors, identity, name,
  species), event log, notebook entries, visit invitations, sessions. Keyed
  throughout by synthetic UUIDs (§7.1), never by email.
- **Read-snapshot store**: a low-latency cache (e.g., Redis or equivalent)
  holding the latest denormalized per-account snapshot, refreshed by the
  tick on every pass. The API's `GET /aviary/state` reads from here, not
  from the transactional store, to keep snapshot pulls cheap and to keep the
  tick the sole writer of derived state.
- **Event log**: append-only table partitioned by account UUID, consumed
  in-order by the tick with a per-account cursor. This is the only path by
  which client interactions affect personality/mood (§6.4).

---

## 3. Data model

All IDs are synthetic UUIDs. No field anywhere is keyed or partitioned by
email except the single encrypted email column on the account record
(§7.1).

### 3.1 Account

```
Account {
  id: uuid (pk)
  email_encrypted: bytes
  email_verified_at: timestamp
  created_at: timestamp
  deletion_state: enum(active, pending_deletion, deleted)
  deletion_requested_at: timestamp | null
  accessibility_prefs: { reduced_motion: bool, captions: bool,
                          narration_enabled: bool }
  visit_notifications_enabled: bool (default false)
}
```

### 3.2 Bird

```
Bird {
  id: uuid (pk)                       // stable identity, never reassigned
  account_id: uuid (fk)
  species_id: string (fk -> species registry)
  name: string (user-editable, no effect on engine)
  created_at: timestamp                // adoption time

  // Personality vector — slow timescale, server-authored only
  personality: {
    boldness: float [0,1]
    social_warmth: float [0,1]
    vocal_frequency: float [0,1]
    plumage_saturation: float [0,1]
    curiosity: float [0,1]
  }
  personality_updated_at: timestamp
  personality_version: int             // monotonic, for delta-application
                                        // ordering/idempotency, not for
                                        // exposing to client

  // Mood — fast timescale
  mood: enum(wary, content, curious, drowsy, alert)  // `[CALIBRATE]` exact
                                                       // enum finalized in
                                                       // implementation per
                                                       // bird_engine.md
  mood_set_at: timestamp
  mood_decay_due_at: timestamp         // daily-ish reset boundary

  // Derived/runtime (recomputed by tick, part of snapshot not canonical
  // long-term storage)
  current_perch_zone: enum(front, middle, back)
}
```

The personality vector is **never** serialized into any API response that
reaches the client in numeric form (§4.5). The snapshot endpoint omits the
`personality` object entirely; it only ever emits mood, perch zone, and
call-timing parameters derived from personality server-side.

### 3.3 Species registry (static-ish reference data)

```
Species {
  id: string (pk)
  silhouette_asset_ref: string
  default_palette: string
  call_motif_library_ref: string       // shared grammar definition,
                                        // versioned
  is_nightjar_class: bool              // the "stays active at night" species
}
```

Six rows at v1 launch, per `bird_engine.md`.

### 3.4 Event log (append-only)

```
InteractionEvent {
  id: uuid (pk)
  account_id: uuid (fk)
  bird_id: uuid | null (fk)            // null for account-level events
                                        // (settle, presence ping)
  type: enum(presence_ping, listen_in_start, listen_in_end,
             offer_seed, offer_song, offer_pool, settle, undo_settle)
  occurred_at: timestamp                // client-observed time, but tick
                                        // processes in server receipt order
  payload: jsonb                        // e.g., offer subtype, listen-in
                                        // duration on _end
  sequence: bigint (monotonic per account, assigned by API on write)
}
```

`sequence` is what the tick uses for in-order, idempotent consumption — not
`occurred_at`, which is client-supplied and untrusted for ordering.

### 3.5 Aviary world state (per account, singleton)

```
AviaryWorldState {
  account_id: uuid (pk)
  local_tz_offset_hint: string          // last-known client-reported tz,
                                         // used to anchor day/night locally
                                         // per concepts.md ("their morning
                                         // is the aviary's morning")
  weather: { kind: enum(none, rain, wind), started_at, expected_end_at }
  last_tick_at: timestamp
  last_tick_cursor: bigint              // event-log sequence consumed
                                         // through
  next_bird_unlock_eligible_at: timestamp | null
}
```

`[DEFAULT]` Day/night is anchored to client-reported timezone offset, not
geolocation. The client sends its IANA timezone (or UTC offset) on snapshot
pull and on a low-frequency keepalive; the tick uses the most recent value
to compute local time-of-day for mood transitions and rendering parameters.
This avoids any location-permission prompt, which would be a system-surface
intrusion the naturalist product surface shouldn't need.

### 3.6 Notebook entry

```
NotebookEntry {
  id: uuid (pk)
  account_id: uuid (fk)
  written_at: timestamp
  prose: string                         // naturalist voice, generated by
                                         // the tick's narration generator
  trigger_kind: string                  // internal only — what pattern
                                         // fired this entry; never exposed
                                         // to client in this form
}
```

Entries are read-only to the user (no edit/delete API at all — not just a
hidden one).

### 3.7 Visit invitation

```
VisitInvitation {
  id: uuid (pk)
  host_account_id: uuid (fk)
  visitor_email_encrypted: bytes
  token_hash: bytes                     // one-time link token, hashed
                                         // at rest
  state: enum(pending, active, revoked, expired)
  created_at: timestamp
  expires_at: timestamp                 // created_at + 30 days
  first_used_at: timestamp | null
  last_seen_at: timestamp | null
}
```

Visitor sessions are tracked by invitation `id`, not by a visitor account —
visitors never get an Account record; they're not users of the product in
the account sense.

### 3.8 Session

```
DeviceSession {
  id: uuid (pk)
  account_id: uuid (fk)
  token_hash: bytes
  created_at: timestamp
  last_used_at: timestamp
  user_agent_hint: string                // for the revocation UI ("Safari
                                          // on iPhone, last used 2 days ago")
  revoked_at: timestamp | null
}
```

---

## 4. Simulation engine design

### 4.1 Tick cadence and execution model

`[CALIBRATE]` Default cadence: **60 seconds**, per the PRD's "~once per
minute." Implemented as a worker pool that, per pass:

1. Queries accounts with unconsumed events since `last_tick_cursor`, OR
   accounts whose `mood_decay_due_at` has passed, OR accounts due for
   ambient-only advancement (weather, time-of-day mood shaping) even with
   zero events — **the tick must run for accounts with no connected client
   and no new events**, because mood transitions through time-of-day and
   weather happen regardless of interaction. This is what makes "the aviary
   continues without the viewer" true at the data layer, not just the
   rendering layer.
2. For each due account, processes in a single transaction: consume new
   events in `sequence` order → compute deltas → apply to personality and
   mood → advance world state (weather rolls, bird-to-bird propagation) →
   write canonical state → write denormalized snapshot to the read store →
   advance `last_tick_cursor`.
3. Idempotency: the transaction commits cursor advancement atomically with
   state writes, so a crashed/retried tick pass never double-applies a
   delta.

Sharding: accounts are processed independently and in parallel across
worker instances (partition by account UUID hash), since there's no
cross-account interaction in the simulation (visits are read-only and don't
feed back into the host's simulation, per `social_optional.md`).

### 4.2 Mood transition function

Inputs per `bird_engine.md` §Mood: recent in-session interactions, local
time-of-day, ambient events (weather, another bird's alarm call), the bird's
own personality (a high-boldness bird resists wary).

`[DEFAULT]` Implementation: a weighted scoring function over mood states,
computed fresh each tick pass:

```
score(mood, bird, context) =
    w_time_of_day(mood, context.local_hour)
  + w_recent_interaction(mood, bird.recent_events)
  + w_ambient(mood, context.weather, context.nearby_alarm)
  + w_personality_bias(mood, bird.personality)
```

The bird transitions to `argmax(score)` unless it's already there, with a
small hysteresis margin (`[CALIBRATE]`: new-mood score must exceed
current-mood score by ≥10%) to prevent flapping between two adjacent moods
on borderline ticks. Mood persists across sessions and across ticks where no
score crosses the hysteresis threshold — this is what satisfies "mood does
not reset on tab open" (§ concepts.md, bird_engine.md).

Daily-ish reset: `mood_decay_due_at` is set to roughly 20-28 hours
`[CALIBRATE]` after the last mood-changing tick (jittered per bird, not a
synchronized midnight reset, so birds don't all change in lockstep — that
would read as systemic rather than alive). On reaching it, the scoring
function gets a stronger time-of-day weight pull, nudging back toward a
time-appropriate baseline mood.

### 4.3 Personality drift function

Implemented as a low-pass filter: each tick computes a small non-negative
delta per trait from the event log since last cursor, and applies
`new_value = min(max_trait_value, old_value + delta)`. **Deltas are never
negative** — this is enforced at the type level (the delta-computation
function returns `NonNegativeFloat`, not `Float`), not just by convention,
because the PRD calls the monotonic-toward-expressive rule load-bearing and
a sign error here is the single most damaging regression this service could
ship silently.

Delta weighting, in the PRD's stated order of dominance:

1. **Presence-time** (dominant): every accumulated presence-event (§4.4)
   contributes a small delta spread across all traits, weighted toward
   `plumage_saturation` and `vocal_frequency` (sustained attention → visual
   richness and willingness to call) — `[DEFAULT]` exact per-trait weight
   split, tunable via the same config table as the bird-unlock pacing.
2. **Listen-in**: contributes to the listened-in bird's `social_warmth` and
   `vocal_frequency` specifically, scaled by listen-in duration (capped per
   session to avoid one long listen-in dominating a day's drift).
3. **Offers**: accepting an offer → small `curiosity` delta on the receiving
   bird; offering near a bird at all (regardless of accept/ignore) → small
   `boldness` delta.
4. **Settle**: no directional drift delta; only closes the presence window
   cleanly (§4.4).

Calibration targets (testable, per `bird_engine.md`): a synthetic-presence
test harness simulating ~1 week of "regular visits" (`[DEFAULT]`: defined
as ~20-30 min/day of qualifying presence, 5-6 days/week, for calibration
purposes) should produce a measurable (instrument-detectable, e.g. >0.02 on
a [0,1] trait) but not user-visible delta; ~3 weeks of the same pattern
should produce a delta large enough to cross a `[CALIBRATE]` visibility
threshold the design/animation team defines (likely tied to which idle-pose
or call-timing bucket the trait value falls into, since the user perceives
traits through their rendered effects, not the numbers). This calibration
harness is a required CI fixture, not a one-off manual check, because drift
calibration is named in the PRD as one of the primary risk areas (§13).

### 4.4 Presence accounting

Implemented client-side as a presence detector that emits `presence_ping`
events to the API at a steady low-frequency interval (`[DEFAULT]`: every 30
seconds) **only while all three conditions hold simultaneously**:
`document.visibilityState === 'visible'`, `document.hasFocus()`, and a
pointermove or keypress has occurred within the trailing activity window
(`[CALIBRATE]`: default 3 minutes, "leaning toward the longer side" per
`interactions.md`).

The client does not decide presence-time on its own — it just reports
qualifying ticks (pings) and the tick on the server accumulates ping count
× ping interval into presence-time per accounting period. This keeps the
authoritative definition server-enforced (a compromised or buggy client
can't inflate presence by lying about elapsed time) while keeping the
three-condition check itself client-side, where the signals
(`visibilityState`, focus, input events) actually live.

`presence_ping` events stop being emitted the instant any condition lapses
— no client-side "grace period" smoothing that would itself loosen the
definition. The server-side activity window check in §4.3's delta
computation uses ping density, not a single boolean, so a client that pings
intermittently near the activity-window boundary doesn't get rounded up to
full presence.

### 4.5 Personality vector never exposed

Enforced architecturally, not just by client discipline: the snapshot
serializer (the function that turns canonical `Bird.personality` into the
wire-format snapshot) has no code path that includes the `personality`
object. There is no debug flag, no internal-only query param, no admin
endpoint that returns it in a form reachable from a browser session — admin
tooling that needs to inspect personality for support/debugging purposes
reads directly from the source-of-truth database via an internal-only
tool, never through the public API surface, so "is personality reachable
from the client" stays a single, auditable boundary.

### 4.6 Call-grammar runtime

The call grammar is a shared definition (motifs, allowed combination rules,
tempo/pitch parameter ranges per species) versioned alongside the species
registry. Two runtime halves:

- **Server-side (parameter selection)**: each tick, for birds expected to
  call before the next tick (probability driven by `vocal_frequency` and
  mood), the tick selects which motif(s), a tempo multiplier, and a pitch
  envelope range, and includes these as part of the bird's snapshot entry.
  This keeps the *decision* of what a bird sounds like driven by
  server-authoritative state.
- **Client-side (synthesis)**: WebAudio nodes generate the actual waveform
  from the selected motif + parameters, with real-time micro-variation
  (slight timing jitter, slight pitch jitter) applied independently each
  time it's triggered, so the same motif never sounds byte-identical twice
  — satisfying "calls vary every time" without needing the server to
  enumerate every variation.

Chorus: when the snapshot indicates two or more birds with calls scheduled
in overlapping windows, the client mixes them through a shared WebAudio
graph in real time (not pre-mixed), which is what produces genuine
phase/timing interaction instead of the stacked-loop phase-cancellation
artifact called out in `bird_engine.md`.

### 4.7 Bird-to-bird interaction

Modeled at the tick level: when processing a bird whose mood just shifted to
`wary` (e.g., from an alarm-type event), the tick applies a smaller,
decaying wary-bias to other birds' mood scores for the next 1-2 ticks
(propagation, not instant synchronization — `[CALIBRATE]` propagation
strength and decay). Chorus eligibility (§4.6) is itself a bird-to-bird
interaction: high-vocal-frequency birds get a small score bonus to call
sooner when another high-vocal-frequency bird is already scheduled to call
in a nearby window, producing emergent joint-calling without being scripted.

---

## 5. API surface

All endpoints require a valid device-session token except magic-link
issuance/verification and the visit-token redemption path. All responses
use the synthetic account UUID; no endpoint accepts or returns a raw email
except account-settings email-change flows (which require the new address
specifically, by design).

### 5.1 State

- `GET /aviary/state` → current denormalized snapshot (read from the fast
  snapshot store, §2.4): per-bird `{id, name, species_id, mood,
  current_perch_zone, call_schedule: [{motif_id, tempo, pitch_range,
  trigger_at}], active_transition}`, world state `{time_of_day_phase,
  weather, settled: bool}`. **No personality fields, ever** (§4.5).
- `GET /aviary/state` supports a `?reason=visibility_regain|resume|keepalive`
  hint purely for server-side metrics (does the snapshot-pull pattern match
  expectations) — it has no effect on response content.
- Return-greeting is not a separate endpoint: it's encoded as a
  `greeting_event` field on the snapshot returned at the start of a session
  (first pull after a gap), computed by the tick at the moment it detects a
  fresh presence window opening, using absence-length (time since last
  presence-event) and the greeting bird's boldness/mood to select a
  greeting intensity bucket and a small randomized stagger offset for any
  secondary birds that also greet. The client renders the greeting
  client-side from this descriptor; the server doesn't ship audio, just the
  descriptor (motif + intensity + stagger timing), reusing the call-grammar
  runtime (§4.6).

### 5.2 Events

- `POST /aviary/events` — body: `{type, bird_id?, payload?,
  client_occurred_at}`. Server assigns `sequence` and persists to the event
  log; returns `202 Accepted` with the assigned sequence. This is the only
  client-to-server mutation path for anything that affects mood/personality.
  Rate-limited per account to prevent a misbehaving client from flooding the
  log (`[DEFAULT]`: 10 req/sec/account burst, well above any legitimate
  interaction rate including presence pings).
- Presence pings, listen-in start/end, offer events, settle/undo-settle all
  go through this single endpoint with different `type` values — kept as
  one endpoint rather than one-per-type because they share identical
  auth/validation/ordering concerns and the type enum is small and stable.

### 5.3 Notebook

- `GET /notebook?cursor=&limit=` → paginated, most-recent-first, read-only.
  No write/edit/delete endpoints exist at all.

### 5.4 Account

- `POST /auth/magic-link` — `{email}`, always returns `202` regardless of
  whether the email is registered (no account-enumeration leak).
- `POST /auth/magic-link/verify` — `{token}` → issues `DeviceSession`,
  invalidates the link.
- `GET /account/sessions`, `DELETE /account/sessions/:id` — list/revoke.
- `POST /account/email-change` — `{new_email}` → sends verification to new
  address; old email remains active until confirmed.
- `POST /account/export` → generates JSON snapshot async, emails a download
  link to the verified address (not returned synchronously in the API
  response — the PRD specifies email delivery).
- `POST /account/delete` → sets `deletion_state = pending_deletion`,
  `deletion_requested_at = now`.
- `POST /account/undelete` → available any time within the 30-day window
  while signed in; sets `deletion_state = active`.
- A scheduled job hard-deletes accounts where
  `deletion_requested_at < now - 30d` and `deletion_state =
  pending_deletion`: cascading delete across birds, events, notebook,
  sessions, visit invitations, telemetry references.

### 5.5 Visit invitations

- `POST /visits/invite` — `{visitor_email}` (host-authenticated) → creates
  `VisitInvitation`, emails one-time link.
- `GET /visits` (host-authenticated) → visit log: invitee email, dates,
  approximate duration, outstanding invites.
- `DELETE /visits/:id` (host-authenticated) → revokes immediately.
- `GET /visit/:token` (unauthenticated, token-based) → if valid/active,
  redeems into a short-lived visitor session scoped to read-only snapshot +
  audio access for that one host account; if revoked/expired, returns the
  matter-of-fact "visit no longer available" surface descriptor.
- Visitor sessions hit a restricted variant of `GET /aviary/state` (no
  event-write access at all — `POST /aviary/events` rejects any request
  authenticated as a visitor session, by token scope, not just by UI
  omission) and never write presence/interaction events, per
  `social_optional.md`'s requirement that visitor attention never drifts
  the host's birds.

### 5.6 Accessibility settings

- `GET/PUT /account/accessibility` — `{reduced_motion, captions,
  narration_enabled}`. Reduced-motion also auto-applies from
  `prefers-reduced-motion` client-side without requiring this endpoint, but
  the explicit setting overrides/persists across devices.

---

## 6. Sync model

### 6.1 Canonical-state propagation

Multi-device sync requires no client-to-client logic at all (§2.1, §6
overview in `accounts_sync.md`). Both devices call the same `GET
/aviary/state`, which always reads the single canonical snapshot the tick
last wrote. There is no per-device state to reconcile because no device
holds authoritative state. This is the architecture, not a feature layered
on top — there is no sync service, no conflict-resolution module for
personality, because nothing client-originated is ever treated as
authoritative for personality.

### 6.2 Snapshot refresh triggers (client)

The client re-pulls `GET /aviary/state`:
- On `visibilitychange` → visible.
- On detecting a long render-frame gap (`[DEFAULT]`: >5s since last
  `requestAnimationFrame` tick, indicating likely suspend/resume).
- On a low-frequency keepalive while visible (`[DEFAULT]`: every 60s,
  aligned with tick cadence so the client is never far behind the server's
  latest pass).
- Immediately after writing an interaction event expected to produce a
  visible reaction (e.g., right after an offer), with a short bounded
  retry/poll (`[DEFAULT]`: poll up to 3 times over 5s) since the tick may
  not have processed the event yet — the client doesn't block the UI on
  this, it just opportunistically refreshes the reaction sooner than the
  next scheduled pull.

### 6.3 Preventing conflicts (not resolving them)

This is the section title deliberately worded "preventing" not "resolving"
per `accounts_sync.md`: there is structurally no conflict to resolve because
clients never submit absolute state. The only place a real conflict-like
surface exists is auth (magic-link replay, session race) and those are
handled as ordinary auth edge cases (link single-use + hashed + short TTL),
not as data-sync conflicts.

### 6.4 No last-write-wins for personality

Enforced by construction: the only write path to `Bird.personality` is the
tick's delta-application step (§4.3), which runs inside a single-account
transaction holding a row lock for the duration of delta computation +
write, so even two tick passes racing for the same account (shouldn't
happen given the scheduling model, but defensively) can't interleave
writes. The API layer has **no code path** that accepts a personality value
in a request body — request schemas for `POST /aviary/events` don't even
have a field shaped like a personality value, so this isn't just
"unused," it's structurally absent.

---

## 7. Privacy and identity

### 7.1 Synthetic account ID

Every table, log line, queue message, partition key, and telemetry event
keyed by `account_id` (the synthetic UUID) — never email. Email exists in
exactly one column (`Account.email_encrypted`), encrypted at rest with a
key-management setup standard for the org `[DEFAULT — implementation detail
left to platform security team]`. A lint/CI check
(`[DEFAULT]`: a repo-level grep-based check in CI, e.g. flag any new
schema/log-schema field literally named `email` outside the account
service's own module) guards against regression, given the PRD's framing of
this as the single most important rule in the file and a class of mistake
that's "impossible to retrofit."

### 7.2 Telemetry boundary

Two physically/logically separate data paths:
- **Simulation database** (accounts, birds, events, notebook) — never read
  by the analytics warehouse, never exported to any ML/training pipeline.
- **Aggregate telemetry pipeline** (request counts, latencies, error rates,
  anonymized session-duration histograms, render-frame timings,
  audio-context error counts, tick latency) — collects no per-account
  dimension beyond what's needed for operational alerting (and even that is
  scoped to "is this account/shard having errors," never "what is this
  bird doing"). `[DEFAULT]`: enforced by having the telemetry SDK used in
  simulation-service code reject (at the type level — the emit function's
  parameter type has no `account_id`-shaped field) any event carrying
  per-bird or per-account interaction fields; only the API/infra layer's
  generic request-metrics middleware (which only ever sees route + status +
  latency) is allowed to emit telemetry at all from request-handling code.

### 7.3 Account export and deletion

Export: async job assembles a JSON document (birds + names + current
personality values + current moods + notebook entries + account settings)
and emails a signed, expiring download link to the verified address — not
returned in an API response body, consistent with "emailed to the verified
address" in the PRD.

Deletion: `deletion_state` state machine `active → pending_deletion →
deleted`, with `pending_deletion → active` reachable any time within 30
days via the "I changed my mind" affordance on any signed-in page. Hard
deletion is a scheduled job, cascading across every table keyed by
`account_id`, including telemetry records that happen to carry the account
UUID for operational-alerting purposes (even though those records don't
carry per-bird state, they still get cleaned up since they're still
account-linked).

---

## 8. Frontend rendering pipeline

### 8.1 Scene composition

Single horizontal scene, three perch-zone layout (front/middle/back),
foreground/background parallax layers (subtle, per `aviary_layout.md` — no
heavy layered-illustration parallax). `[DEFAULT]` rendering approach: a
canvas or WebGL-lite 2D scene graph (not DOM-animated elements), so 60fps
idle motion on a 5-year-old laptop (§9.3) is achievable without layout
thrash; birds are sprite/vector entities with pose-blend state, not DOM
nodes with CSS animations, given the volume of continuous micro-motion
required.

### 8.2 No-load-state-as-spinner

The client's bootstrap sequence: HTML arrives with the first snapshot
inlined (§2.1) → client immediately renders that snapshot's bird positions
mid-pose (not at a neutral/idle default) → render loop starts. If the
inlined snapshot is stale or missing (cold cache, very slow connection),
the loading state is the "quiet field" — soft sky color, at most one or two
faint motion cues, explicitly never a spinner — until the first real
snapshot arrives, per `aviary_layout.md`. This is implemented as a distinct
rendering mode (`quiet-field`), not a generic loading-spinner component
reused from elsewhere in the codebase — a shared spinner component must not
be reachable from this code path at all, to keep the rule enforceable by
code review rather than by hoping nobody reaches for the convenient
existing component.

### 8.3 Idle micro-motion and mood-shaped rendering

Each bird has a pose-blend state machine per mood (preening, scanning,
head-tilt-toward-sound, weight-shuffle for "content/curious/alert"; perched
low/fluffed for "drowsy"; further-back perch selection + more scanning for
"wary"). Mood drives both perch-zone selection (already encoded server-side
in the snapshot's `current_perch_zone`, §3.2) and the client's pose-blend
weighting — the client doesn't infer mood from anything; it reads mood
directly off the snapshot and selects the corresponding pose-blend table
entry.

### 8.4 Day/night, weather, ambient ornaments

Day/night palette interpolation driven by `world_state.time_of_day_phase`
from the snapshot (computed server-side from the client-reported tz hint,
§3.5) — smooth client-side interpolation between phase keyframes so the
transition itself is gradual even though the snapshot only updates once a
tick. Weather (`rain`/`wind`/`none`) is a snapshot field that triggers a
client-side particle/overlay effect; ambient leaf/feather drift (§
`aviary_layout.md`) is explicitly **not** server-state-driven — pure
client-side ornament generator at idle cadence, since per-leaf state would
be pointless server load for something with no simulation meaning.

### 8.5 Responsive scene

Viewport-responsive scene graph: horizontal compression on narrow viewports
without cropping any bird (perch positions recalculated proportionally, not
clipped), more inter-perch spacing on wide viewports. `[DEFAULT]`: perch
zones are defined as proportional viewport regions (not fixed pixel
coordinates), and the bird-placement layer clamps every bird's rendered
position to stay within the visible frame at all supported aspect ratios —
implemented as a layout constraint solved per-frame from current
viewport size, not a fixed set of breakpoint-specific layouts, to avoid
popping when resizing (e.g., rotating a phone).

### 8.6 Reduced-motion mode (designed surface)

A parallel render mode, not a flag that disables animation calls. Idle
micro-motion is replaced by a curated set of still poses per mood per
species, cross-faded slowly between them (`[CALIBRATE]` cross-fade
duration, default 2-4s) instead of continuously animated. Flight/perch
transitions become cross-fades, not animated path traversal. Leaf/feather
ornaments are removed entirely (not just slowed) per the PRD; ambient color
shifts remain but slowed. This requires its own pose-art and transition
logic authored alongside the full-motion version from the start (§10.1),
not derived automatically from it, since "automatically derived" is exactly
the stripped-fallback failure mode the PRD calls out.

### 8.7 Transitions and the settle gesture

Settle: client-triggered lighting shift to evening palette over several
seconds (`[CALIBRATE]`: ~4-6s), calls quiet (audio mix fades toward
near-silence ambient), `settle` event written to the server. A 5-second
client-side undo window: any click anywhere in the aviary within 5s reverts
the lighting shift locally and writes an `undo_settle` event; past 5s, the
undo affordance disappears and the aviary stays in settled visual state
until the user re-engages or closes the tab. Note: settle is a **rendering
and engagement-window** gesture, not a different engine state — at the
tick/presence level, settle simply ends the presence window the same way
tab-close does (§4.4); it carries no drift weighting of its own per
`bird_engine.md`.

---

## 9. Audio pipeline

### 9.1 Procedural call synthesis

WebAudio-based synthesis engine consuming the call-grammar parameters from
the snapshot (§4.6): oscillator/noise-based motif primitives, combined per
the species' motif library, with runtime jitter (timing ±a few percent,
pitch ±a few cents, `[CALIBRATE]` exact jitter ranges tuned by sound design)
applied independently on every trigger so no two plays of "the same" call
are byte-identical. Buffers/audio nodes are reused across triggers (object
pool, not per-call allocation) to satisfy the no-memory-growth budget (§11).

### 9.2 Chorus mixing

A single shared `AudioContext` per session; each calling bird gets its own
gain-node chain feeding a shared mix bus, so simultaneous calls genuinely
mix in real time (true chorus) rather than being pre-rendered and layered
(which produces the phase-cancellation artifact the PRD calls out).
Bird-to-bird chorus eligibility is server-decided (§4.7); the client just
renders whatever overlapping call schedule the snapshot describes.

### 9.3 Listen-in mix decay

Listen-in engage/disengage drives a gain-automation ramp
(`AudioParam.linearRampToValueAtTime` or equivalent) on each bird's gain
node: focused bird ramps up over `[CALIBRATE]` ~1-2s, all others ramp down
to an ambient floor (never to zero — "never go silent," per
`interactions.md`) over the same window. Disengage reverses the ramp
symmetrically. This is implemented as a single mix-state machine per
session (current focus target + ramp progress), not per-bird ad hoc
fades, so simultaneous engage/disengage (switching focus directly from one
bird to another) ramps correctly without a momentary double-ramp glitch.

### 9.4 WebAudio fallback

Feature-detect `AudioContext` availability and permission state at startup.
If unavailable or denied: render the aviary normally (full visual fidelity)
but with audio synthesis disabled and captions (§10.3) **on by default**
for that session, per `accessibility_perf.md`. No recorded-audio fallback
path exists in the codebase at all — this is enforced by simply not
shipping any recorded-audio asset pipeline, so there's no "quick fallback"
available to reach for under deadline pressure.

---

## 10. Accessibility surfaces

### 10.1 Screen-reader narration

A narration generator — sharing its underlying voice/style logic with the
field-notebook generator (§3.6, §12) but operating on a faster, live cadence
— produces naturalist prose strings from current snapshot state, exposed via
an `aria-live="polite"` region (`[DEFAULT]`) updated on a `[CALIBRATE]`
30-60s idle cadence, with a priority bump (more assertive timing, not
necessarily `aria-live="assertive"` since that risks interrupting screen
reader output and is itself an "announcement" the PRD's voice rules are
wary of — `[DEFAULT]`: stays `polite` even for prioritized events, just
queued sooner) for return-greetings and offer reactions. Narration text
generation must run through the same prose-template/voice system as the
notebook (§12.1) so the two surfaces don't read as differently-voiced
products glued together, per `accessibility_perf.md`.

### 10.2 Reduced-motion mode

Covered in §8.6 as a rendering concern; from the accessibility surface side,
triggered automatically from `prefers-reduced-motion: reduce` and
overridable/persistable via `PUT /account/accessibility` so the preference
follows the account across devices, not just the one browser that has the
OS-level setting.

### 10.3 Captioning for calls

Caption text is generated at the moment of synthesis from the actual
selected motif + parameters (§9.1) — not a fixed string table per
motif-id — via the same naturalist-voice template system, so the caption
always matches what was actually played (per `accessibility_perf.md`'s
explicit requirement). Rendered as small fading text near the calling bird;
toggled from accessibility settings, and forced on by default under the
WebAudio-unavailable fallback (§9.4).

### 10.4 Keyboard navigation

Full tab-order: top bar items → aviary scene (Tab focuses first bird, arrow
keys move focus between birds, Enter triggers listen-in on the focused
bird, Escape exits listen-in). Offer affordance keyboard-reachable via a
top-bar shortcut and fully keyboard-navigable once open. Settle reachable
from the top bar. Focus indicator: a soft, high-contrast outline rendered
as a scene-graph overlay (not a DOM `:focus` outline, since birds are
canvas/WebGL entities, §8.1) that's legible against both bright and dim
aviary palettes — `[DEFAULT]`: implemented as a contrasting ring drawn with
a subtractive/inverted blend so it stays visible regardless of underlying
scene color, exact treatment finalized by the visual designer per the PRD.

### 10.5 WCAG AA contrast

Applies to all user-copy text (top bar labels, settings, account/error
surfaces, captions, visually-displayed narration). Enforced via a
design-token contrast-ratio check in CI against the design system's defined
palette (`[DEFAULT]`: automated contrast-ratio assertion run against
rendered chrome components in the visual regression suite), since the
aviary scene itself carries no user copy outside the top bar.

---

## 11. Performance budgets and observability

### 11.1 Bundle budget: <2MB gzipped initial JS

Enforced via CI bundle-size check that fails the build above threshold.
Drives: procedural (not recorded) audio (§9), procedurally-generated or
compact-SVG bird visual assets, aggressive code-splitting for
infrequently-reached surfaces (account settings, accessibility settings,
visit-invitation flow) loaded as separate chunks fetched on navigation to
those surfaces, not in the critical path.

### 11.2 Time-to-first-bird <500ms (mid-tier mobile, 4G)

Achieved via: (a) snapshot inlined into the HTML response from the edge
(§2.1) — zero extra round-trip before first paint; (b) critical render path
draws the first bird from the inlined snapshot before any non-critical
asset (audio engine init, settings-surface code, notebook code) loads; (c)
bundle budget (§11.1). `[DEFAULT]`: measured via synthetic Lighthouse-style
checks plus aggregate field RUM (§11.5), both gated in CI/release process —
a release that regresses this budget on the synthetic check blocks deploy.

### 11.3 60fps idle motion, 5-year-old laptop, sustained 30 minutes

Canvas/WebGL scene graph (§8.1) chosen specifically to keep this budget
reachable; render loop profiled against a representative low-end device
tier in CI perf testing (`[DEFAULT]`: an automated perf-regression suite
running against a throttled-CPU headless browser profile, not just manual
spot checks, since this is called out as a runtime budget over a 30-minute
session, not just a launch-moment one).

### 11.4 No memory growth over 30 minutes

Real CI test (`[DEFAULT]`: a long-running headless-browser test that drives
the aviary through simulated interaction for 30 minutes and asserts heap
size stays flat within tolerance), per the PRD's explicit "this is a real
test in CI, not a guideline." Audio buffer pooling (§9.1), notebook-entry
virtualization (entries scrolled out of view release references, not just
visually hidden), and bounded worker/AudioContext lifecycle (one
AudioContext per session, not recreated per call) are the primary
mechanisms.

### 11.5 Observability

- **Synthetic checks**: scheduled automated-browser runs from multiple
  geographies exercising load → first-bird-render → basic interaction,
  feeding the time-to-first-bird and bundle-delivery metrics.
- **Aggregate RUM**: page load timings, first-bird-render timings,
  render-frame timing distributions, audio-context error counts,
  simulation-tick latency — all aggregate-only, no per-account dimension
  beyond what's needed for alerting routing (§7.2).
- **Tick latency alarm**: p99 simulation-tick latency alarms above 5s, per
  the PRD's explicit error budget.

### 11.6 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers
get a matter-of-fact unsupported-browser page (no attempt at graceful
feature-by-feature degradation below that line, per the PRD's explicit
cost-benefit call).

---

## 12. Field notebook generation

### 12.1 Generation logic

Notebook entries are generated by the tick (or a closely-coupled
notebook-generation pass run alongside it), pattern-matching against recent
simulation history for "noteworthy" moments — `[DEFAULT]` candidate
patterns: a first-time-this-week ordering change in which bird greets
first; an unusually long quiet stretch; a bird settling into a new
mood-and-perch combination it hasn't shown recently; a notable weather
event coinciding with a behavior change. Each matched pattern feeds a
naturalist-voice prose template system (shared with narration, §10.1) that
fills in bird names, specifics, and timing — never generic phrasing like
"your bird is happier."

### 12.2 Sparsity control

`[CALIBRATE]` Target: roughly one entry every few days for a
regularly-visited aviary, more frequent only when multiple noteworthy
patterns genuinely fire close together — never one entry per session.
Implemented as a per-account cooldown/budget the generator respects (a
minimum inter-entry gap, `[DEFAULT]` ~36-48 hours baseline, overridable
only by a small set of explicitly higher-priority pattern types) rather
than a pure pattern-frequency cap, so the sparsity holds even for very
active users, per the PRD's explicit requirement.

### 12.3 Strict observation/behavior boundary

The pattern library is restricted, by design, to patterns about the
*aviary* (bird behavior, bird-to-bird dynamics, weather, time) — never
patterns about the *user* (visit frequency, days-since-last-visit,
streaks). This boundary is enforced at the pattern-definition level: the
pattern-matching function's only inputs are bird/world state history, not
account-level visit-frequency aggregates, so a "you've been here every day
this week" entry isn't a content-moderation problem to police, it's
structurally unreachable from the data the generator is given.

---

## 13. Rollout

### 13.1 Launch sequencing

Ship accessibility surfaces (narration, reduced-motion, captions,
keyboard nav) **with** v1, not as a fast-follow, per
`accessibility_perf.md`'s explicit instruction that a later-landing
reduced-motion mode is itself a launch failure for those users. This is a
go/no-go gate on the launch checklist, not a stretch goal.

### 13.2 Bird-count ramp

Starts at 2 birds per new account (server-selected species, not
user-chosen, per `bird_engine.md`'s adoption-flow rule). Third-bird-and-
beyond unlocks are purely aviary-age-driven (§1.3), implemented as a
scheduled check the tick performs per account (compare account age against
the unlock schedule in the config table) — never engagement-driven, so
there's no path by which "interact more to unlock birds" can creep in.

### 13.3 Day-one instrumentation

From day one: aggregate RUM (§11.5), tick-latency alarms, drift-calibration
test harness results tracked over time (not just pass/fail — trend the
measured drift magnitude so calibration regressions are visible before
they're reported by users), bundle-size CI gate, WebAudio
availability/fallback-rate aggregate counter (to know how many sessions are
running in silence+captions mode, since that's a meaningfully different
experience worth watching at the aggregate level).

### 13.4 Gradual exposure

`[DEFAULT]`: standard staged rollout (internal → small % → full) gated on
the synthetic performance checks and the memory-growth/perf CI suites
passing, plus a manual review of a sample of generated notebook entries and
narration strings before full exposure, specifically to catch voice
drift (generic-sounding output) before it reaches users at scale — this is
a content-quality gate, not just a systems-health gate, because voice
consistency is called out repeatedly as load-bearing for the whole product.

---

## 14. Risks

### 14.1 Drift calibration

The single highest-risk piece of the system. Too fast and the product
becomes a Tamagotchi-shaped thing the user can visibly move by clicking
(violates the explicit non-goal); too slow and nothing the user does
appears to matter (the "screensaver" failure named directly in
`bird_engine.md`). Mitigation: the calibration test harness (§4.3) as a
required, tracked CI signal rather than a one-time tuning pass; expose the
delta-weighting constants via the same config table as the bird-unlock
pacing so recalibration is a config change, not a code deploy, since this
will almost certainly need adjustment post-launch based on real usage
patterns that synthetic harnesses can't fully predict.

### 14.2 Sync correctness under load

The no-last-write-wins guarantee (§6.4) depends on the tick holding an
exclusive per-account lock during delta computation + write. Under high
concurrency (e.g., a burst of accounts due for tick processing
simultaneously), lock contention or a poorly-tuned worker pool could create
processing delays that look like "my bird didn't react." Mitigation: tick
latency alarm (§11.5) as an early-warning signal, and load-testing the tick
worker pool specifically for the "many accounts due simultaneously" case
(e.g., a synchronized cadence where many accounts share similar tick-due
timestamps) before launch, not just steady-state throughput testing.

### 14.3 Audio uncanniness

Procedural synthesis that sounds robotic or repetitive defeats the entire
rationale for not using recorded audio (§ `bird_engine.md`). This is a
sound-design risk as much as an engineering one. Mitigation: budget real
sound-design iteration time against the motif library and jitter parameters
(§9.1) before launch — this is explicitly not a "build it once and ship"
component; treat it like the drift calibration, with a listening-review
gate in the rollout checklist (§13.4) alongside the notebook/narration
voice-quality gate.

### 14.4 Accessibility regression risk

Because accessible surfaces are bespoke designed experiences (reduced-
motion poses, narration prose, captions) rather than mechanically derived
from the full-fidelity surface, they're easy to let drift out of sync when
the full-fidelity surface changes post-launch (new species added, new
mood added, new interaction added) and someone forgets to extend the
accessible variant. Mitigation: treat "does this PR touch bird
moods/species/interactions" as a checklist trigger requiring the
corresponding reduced-motion pose set / narration template / caption
template to be updated in the same change, enforced via PR template/review
checklist rather than left to memory.

### 14.5 Gamification creep

Named directly as a risk in `non_goals.md`: the cheapest, most tempting
features to add post-launch (a streak, a visit calendar, a "birds adopted"
counter) are exactly the features explicitly forbidden. Mitigation: no
underlying data is even computed in a form that would make these features
cheap to bolt on — notebook pattern-matching has no access to visit-
frequency aggregates (§12.3), telemetry has no per-account interaction
dimension (§7.2), and there's no "achievements" table or feature-flag
scaffold sitting dormant in the schema waiting to be turned on. The
prevention is structural (the data isn't there to build on), not just
policy (a rule that could be relaxed in a future planning meeting).

### 14.6 Visit feature scope creep toward co-presence

The most natural-seeming enhancement to "watch a friend's aviary" is some
form of shared presence (seeing that a friend is watching too, a shared
cursor, live chat). `social_optional.md` rules this out explicitly because
it would require a fundamentally different multi-user simulation model.
Mitigation: visitor sessions are architecturally incapable of writing
events (§5.5, token-scope-enforced, not UI-hidden) — extending toward
co-presence would require a new write path to be added deliberately, not
something that falls out of relaxing a UI constraint.
