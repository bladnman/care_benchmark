# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable build for a frontier engineering
team. It assumes no further clarification. Where the PRD is silent, this plan
makes a defensible call and flags it as **[call]**. The plan's organizing belief,
inherited from the brief, is that *aliveness is the product*: every architectural
choice below exists to make the aviary feel like a place that has been
continuing without the viewer, and to make a relationship that deepens over
weeks. Sections map 1:1 to the deliverable checklist in `1-START_HERE.md`.

---

## 0. Design invariants (load-bearing — enforced mechanically, not by vigilance)

Most of the PRD's warnings are about regressions a well-meaning engineer
introduces months later. Vigilance does not survive a team; tests and types do.
We encode each load-bearing rule as an automated guardrail in CI so that
violating it fails the build. These are referenced throughout and consolidated
in §11 (Guardrails).

1. **First frame is mid-action.** No intro animation, no spinner, no
   fade-from-static. The first rendered frame places birds at the exact motion
   phase carried in the snapshot. *Enforced:* boot-path test asserts no loading
   component mounts on the aviary route; the snapshot schema carries motion
   `phase` and the renderer is required to consume it.
2. **Notice, never announce.** No toast, banner, modal, confetti, badge, or
   textual welcome anywhere on the product surface. The bird greeting is the
   entire welcome. *Enforced:* a lint/AST guardrail bans a `Toast`/`Banner`/
   `Notification` primitive from existing in the product (chrome) component tree;
   the only "return" affordance allowed is the greeting directive in the snapshot.
3. **No gamification, ever.** No score, streak, level, badge, XP, visit count,
   day-dot calendar, milestone celebration — not even as a settings toggle.
   *Enforced:* the canonical state model has no field that counts visits,
   sessions, or streaks; a schema test asserts no such field can be added without
   review sign-off, and the notebook generator's fact vocabulary (§5.7) cannot
   express user-behavior facts.
4. **Personality vector is never exposed and never client-owned.** The raw
   trait floats never leave the server, never appear in any UI/debug/export
   numeric surface, and are never derived or recomputed by the client.
   *Enforced:* the wire snapshot type physically cannot contain the
   `PersonalityVector` type (separate package boundary; §3, §6.2); a type-level
   test fails if `PersonalityVector` is importable from client code; export
   payload (§7.4) carries derived *descriptions*, not trait numbers.
5. **Drift is monotonic toward expressive.** Personality traits only ever
   increase. Neglect never decreases a trait. Visible "quietness on neglect" is
   produced by the fast-timescale mood/behavior layer, not by lowering
   personality. *Enforced:* the drift application clamps every delta `≥ 0`; a
   property test asserts `P_next ≥ P_prev` componentwise for arbitrary event
   logs including empty ones.
6. **Calls are procedural; no recorded audio anywhere.** *Enforced:* a bundle
   guardrail fails CI if any audio asset (`.mp3/.ogg/.wav/.m4a/...`) appears in
   the build graph; the WebAudio fallback is silence-plus-captions, never a
   recorded-audio path.
7. **Server is the only writer of personality/canonical state.** Clients write
   *events* to an append-only log; the tick computes additive, server-authored
   deltas in log order. No last-write-wins on personality. *Enforced:* canonical
   state DB grants write only to the simulation worker role; the ingress that
   accepts client writes can only `INSERT` into the event log (no `UPDATE` of
   personality rows). Access is a DB-permission boundary, not a convention.
8. **Email is PII and is never an identifier.** Every internal reference uses a
   synthetic account UUID. Email is stored once, encrypted, on the account
   record. *Enforced:* a CI check greps service code, log statements, partition
   keys, and event schemas for email-shaped identifiers in non-account-record
   contexts and fails on match; structured-logging serializers drop any `email`
   field by default.
9. **Per-bird interaction data drives only that user's own simulation.** It is
   never aggregated, never trains a model, never crosses into analytics.
   *Enforced:* the simulation DB has no read grant to the analytics warehouse;
   the telemetry event schema has no per-account/per-bird dimension and a
   schema test rejects one (§7.7, §9.5).
10. **Voice split is explicit.** Naturalist for the aviary/notebook/narration/
    captions/offer prompts; matter-of-fact for sign-in/account/sync-error/
    accessibility-settings. *Enforced:* copy lives in two namespaced bundles
    (`voice.naturalist.*`, `voice.system.*`); a lint asserts system surfaces
    import only from the system bundle and aviary surfaces only from naturalist.

If a future change needs to break one of these, the guardrail forces an explicit,
reviewed decision rather than a silent drift.

---

## 1. Scope

### In scope for v1
- Single-user accounts; email magic-link sign-in; per-device revocable sessions;
  email change with verification; account export; soft-then-hard deletion.
- One canonical aviary per account; 2 starter birds (system-selected species);
  cap of 7; age-gated availability of additional birds.
- The bird engine: hidden personality vector (5 traits), monotonic drift, mood
  state machine, procedural call grammar, bird-to-bird interaction, mood-shaped
  idle motion, stable bird identity.
- Server-side simulation tick as the only writer of canonical state.
- Multi-device sync as an architectural property (snapshot pull + interpolation;
  append-only event log; additive server-authored deltas).
- Interactions: return-greeting, listen-in, offer (seed / song-fragment / still
  pool), settle (+5s undo), field notebook (read-only, sparse), presence
  accounting (strict 3-signal conjunction).
- Single horizontal scene; three perch zones; day/night by local time; rare
  ambient weather; ambient micro-motion; thin fading top bar; quiet-field load
  state; empty-aviary state.
- Audio: client-side procedural synthesis, chorus mixing, listen-in re-balance,
  WebAudio fallback to silence+captions.
- Accessibility shipped *with* v1: naturalist screen-reader narration,
  reduced-motion mode (designed surface), call captions, full keyboard nav,
  WCAG AA contrast on chrome.
- Social: one feature — read-only ambient visit by emailed one-time link;
  per-invite opt-in; revocable; default off; silent logging; opt-in notify;
  30-day invite expiry.
- Performance budgets (bundle <2 MB gz, first bird <500 ms, 60 fps idle on a
  5-yr-old laptop, no 30-min memory growth) and aggregate-only observability.

### Explicitly out of scope (from `non_goals.md`, honored throughout)
Native apps; payments; gamification of any flavor (streaks/levels/scores/badges/
XP/visit-counts/day-dot calendars/milestone celebrations); Tamagotchi mechanics
(death, hunger, distress, decaying happiness meter); social-network surfaces
(profiles, follows, public feed, discovery, leaderboards, comments, co-presence,
friend-of-friend, mutual visits); push/email notification of the aviary;
customizable scenes; shared/multi-aviary accounts; recorded-audio fallback;
catalog-style bird selection; user-controlled perch placement; any numeric
exposure of personality.

### Defensible calls flagged in this plan
- **[call]** Naturalist prose (notebook/narration/captions) is produced by a
  writer-authored generative *phrase grammar* parameterized by concrete moment
  facts — not by a runtime LLM. Rationale: determinism, perf budget, voice
  control, and the privacy rule (no per-bird data leaves to an external model).
  (§5.7)
- **[call]** Renderer is a hand-written Canvas2D/WebGL2 scene module (no game
  engine dependency) to protect the 2 MB budget and 60 fps floor. (§6.1)
- **[call]** Tick is *continuous* for recently-active aviaries and
  *deterministic catch-up on wake* for dormant ones; both produce identical
  canonical state, so "the aviary that has been running" holds while cost stays
  bounded. (§5.1)
- **[call]** Presence credit is computed client-side (only the client can read
  visibility/focus/activity) and **clamped server-side** to elapsed wall-clock to
  prevent drift inflation/gaming. (§5.3, §4.3)

---

## 2. Architecture

### 2.1 Shape
A thin client, a CDN edge tier, and a small set of backend services around one
canonical-state datastore. The client renders; the server simulates and is the
sole writer of canonical state.

```
                ┌─────────────────────────────────────────────┐
   Browser ───► │ CDN edge: static HTML + inlined first snapshot│  (first bird <500ms)
   (client)     └─────────────────────────────────────────────┘
       │                         │ snapshot pulls / event writes
       ▼                         ▼
  ┌──────────┐   ┌───────────────────────────────────────────────┐
  │ Renderer │   │ BFF / API gateway (read snapshots, append events)│
  │ + Audio  │   └───────────────────────────────────────────────┘
  │ + A11y   │            │           │            │            │
  └──────────┘            ▼           ▼            ▼            ▼
                   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
                   │  Auth    │ │Simulation│ │  Visit   │ │ Notebook │
                   │  svc     │ │  svc(tick│ │  svc     │ │+Narration│
                   └──────────┘ │ +engine) │ └──────────┘ │ generator│
                                └──────────┘              └──────────┘
                                     │  (sole writer)
                                     ▼
            ┌───────────────── Canonical state DB (Postgres) ─────────────────┐
            │ accounts · birds · personality_vectors · moods · positions ·    │
            │ event_log (append-only, per-account sequence) · notebook ·      │
            │ invites · sessions · settings                                   │
            └─────────────────────────────────────────────────────────────────┘

   Edge/ephemeral: Redis (snapshot cache, presence keepalive, rate limits)
   Telemetry (SEPARATE): aggregate-only pipeline → analytics warehouse
                         (no read path from canonical DB; no per-account dim)
```

### 2.2 Services
- **Auth service** — magic-link issue/consume, session token mint/revoke, email
  change verification, deletion lifecycle. Owns the only place email is stored.
- **Simulation service** — the engine. Runs the tick; reads the event log;
  computes drift deltas, mood transitions, call scheduling; writes canonical
  state. The *only* writer of `personality_vectors`, `moods`, `positions`.
- **BFF/API gateway** — serves snapshot reads (from cache/state) and accepts
  event-log appends. Holds no canonical write authority over personality.
- **Notebook + Narration generator** — turns canonical state + the moment's facts
  into naturalist prose (notebook entries, screen-reader narration, call
  captions). Shares one voice engine (§5.7). Read-only over canonical state.
- **Visit service** — invite issuance/revocation/expiry, visit-link resolution,
  read-only snapshot proxy for visitors, visit logging.

**[call]** All services are TypeScript/Node for shared types with the client and
team velocity; the tick's per-aviary math is light (vector ops on ~7 birds), so
Node is adequate. If profiling shows the tick CPU-bound at scale, the engine core
is a pure function (§5) and can be lifted into a worker pool or a faster runtime
without touching contracts.

### 2.3 Client / server split (the hard boundary)
- **Server owns:** personality vectors, drift, canonical mood, canonical
  positions, mood timers, day/night truth (derived from the account's local-time
  basis), weather schedule, notebook/narration text, call *scheduling decisions*.
- **Client owns:** rendering, interpolation between snapshots, idle micro-motion
  *expression* (parametric, seeded by the snapshot), audio *synthesis* of the
  scheduled calls, ambient ornaments (leaves/feathers — pure decoration, no
  server state), presence detection, and the listen-in mix.
- **Never crosses the wire to the client:** raw personality trait values. The
  snapshot carries *expressed behavior directives* (quantized tendencies the
  renderer/audio need), never the vector itself (§3, §6.2, invariant #4).

### 2.4 Render pipeline boundary
The simulation produces *what* is true (mood, perch, scheduled call, greeting
form). The client decides *how* it looks/sounds frame-to-frame (the exact preen
motion, the pitch jitter of this call instance, the parallax). This split is why
the same canonical aviary renders identically-in-mood but never identically-in-
detail across sessions and devices — the source of procedural variation.

---

## 3. Data model

All identifiers are synthetic UUIDv7 (time-ordered, good index locality). Email
appears only on the account record, encrypted (envelope encryption, per-record
data key) — invariant #8.

**account**
```
account_id           uuid (PK, synthetic, used everywhere internally)
email_ciphertext     bytea            -- encrypted; only field holding email
email_lookup_hash    bytea            -- HMAC(email) for sign-in lookup, not an id
created_at           timestamptz
local_time_basis     text             -- IANA tz, drives day/night; updated on sign-in
deletion_state       enum(active, soft_deleted)
soft_deleted_at      timestamptz null
settings             jsonb            -- reduced_motion, captions, audio_on, notify_visits, ...
```
`email_lookup_hash` lets sign-in find the account without storing email as a key;
it is a keyed HMAC, not a partition/shard key.

**bird** (stable identity — invariant; never replaced on rename/sync/migration)
```
bird_id        uuid (PK, stable forever)
account_id     uuid (FK)
species        text            -- from the ~6-species pool
name           text            -- user-assigned, renameable
adopted_at     timestamptz
```

**personality_vector** (server-only; never serialized to client)
```
bird_id              uuid (PK/FK)
boldness             real   -- [0,1]
social_warmth        real   -- [0,1]
vocal_frequency      real   -- [0,1]
plumage_saturation   real   -- [0,1]
curiosity            real   -- [0,1]
last_drift_tick_seq  bigint -- ordering guard for additive deltas
updated_at           timestamptz
```
Lives in its own table/package boundary so the client type graph cannot reach it.

**mood**
```
bird_id      uuid (PK/FK)
state        enum(wary, content, curious, drowsy, alert, settled, sleeping)
since        timestamptz
dwell_until  timestamptz      -- hysteresis floor; prevents flicker
```

**position**
```
bird_id     uuid (PK/FK)
perch_zone  enum(front, middle, back)
perch_slot  smallint          -- slot within zone for layout
motion_pose enum(perch, preen, scan, shuffle, tilt, fly)
phase       real              -- 0..1 animation phase carried into next snapshot
```

**event_log** (append-only; the only thing clients write)
```
account_id      uuid
seq             bigint        -- per-account monotonic sequence (ordering authority)
client_event_id uuid          -- idempotency key (dedupe replays/retries)
type            enum(presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo)
server_ts       timestamptz   -- authoritative
payload         jsonb         -- e.g. {birdId, offerType} or {windowStart,windowEnd}
PRIMARY KEY (account_id, seq)
UNIQUE (account_id, client_event_id)
```
Partitioned by `account_id`. `seq` is allocated server-side on insert.

**presence_window** (derived/rolled-up by the tick; not user-visible)
```
account_id        uuid
window_start/end  timestamptz
credited_seconds  int         -- clamped to wall-clock (anti-inflation)
```

**notebook_entry**
```
entry_id     uuid (PK)
account_id   uuid
created_at   timestamptz
prose        text            -- naturalist, generated; read-only forever
fact_digest  text            -- internal: which fact template fired (dedupe/sparsity)
```

**session**
```
session_id   uuid (PK)
account_id   uuid
device_label text             -- coarse, user-facing in session list
created_at   timestamptz
last_seen_at timestamptz
revoked_at   timestamptz null
```

**invite**
```
invite_id        uuid (PK)
host_account_id  uuid
visitor_email_ct bytea            -- encrypted
one_time_token_h bytea            -- hash of the emailed token
state            enum(outstanding, active, revoked, expired, used)
created_at       timestamptz
expires_at       timestamptz      -- created_at + 30d
last_visit_at    timestamptz null
```

**visit_log**
```
visit_id      uuid (PK)
invite_id     uuid (FK)
host_account  uuid
started_at    timestamptz
ended_at      timestamptz null
approx_seconds int
```

Notebook, mood, position, and personality are all *outputs of the tick*. The
event log is the only *input* clients control.

---

## 4. API surface

REST/JSON over HTTPS. Snapshots are small (KB). Auth via session token (httpOnly,
secure cookie) plus a CSRF token for state-changing routes. Two copy registers
(naturalist vs matter-of-fact) per invariant #10 — every API error that reaches
a system surface returns matter-of-fact copy keys.

### 4.1 Read path — pull canonical snapshots
`GET /api/aviary/snapshot`
Returns the current canonical snapshot (see schema below). Served from the edge
snapshot cache when warm (invalidated on each tick), else computed from canonical
state. The session-start variant includes a `greeting` directive.

Snapshot wire schema (note: **no personality vector**, only expressed directives):
```jsonc
{
  "aviaryId": "uuid",
  "serverTime": "ISO8601",
  "tickSeq": 91421,
  "daynight": { "phase": "morning|midday|evening|night", "t": 0.37 },  // t = 0..1 within phase
  "weather": { "type": "rain|wind", "intensity": 0.3, "t": 0.6 } | null,
  "birds": [{
     "id": "uuid", "species": "warbler", "name": "pip",
     "perchZone": "front", "perchSlot": 1,
     "mood": "content", "moodSince": "ISO8601",
     "behavior": {            // QUANTIZED expressions of hidden traits — render/audio inputs only
        "greetTendency": "high|med|low",
        "approachTendency": "high|med|low",
        "vocalTendency": "high|med|low",
        "plumageLevel": 0..4,        // discrete visual richness step, not the raw float
        "curiosityTendency": "high|med|low"
     },
     "motion": { "pose": "preen", "phase": 0.42, "next": "scan", "nextEtaMs": 2300 },
     "call": { "inMs": 1800, "motifSeed": 88123, "pitchBias": 0.1, "lengthSteps": 3 } | null
  }],
  "greeting": { "birdId": "uuid", "form": "glance|two_note|step_forward|long_call_then_response", "ttlMs": 2200 } | null
}
```
`behavior` is deliberately quantized (buckets / discrete steps) so the client has
what it needs to render and synthesize without ever receiving a trait number —
this satisfies "render the bird" while keeping invariant #4 intact.

The client also pulls a fresh snapshot on: visibility regaining `visible`, a long
render-frame gap (laptop resumed from suspend), and a low-frequency keepalive
while visible (§6.6).

### 4.2 Notebook
`GET /api/notebook?before=cursor&limit=n` — paginated, read-only, newest-first;
infinite scroll-back; no edit/delete/annotate routes exist.

### 4.3 Write path — append interaction events
`POST /api/events` (batched array; idempotent by `client_event_id`)
```jsonc
[{ "clientEventId":"uuid","type":"listen_in_start","payload":{"birdId":"uuid"} },
 { "clientEventId":"uuid","type":"presence_ping","payload":{"windowStart":"ISO","windowEnd":"ISO","active":true} },
 { "clientEventId":"uuid","type":"offer","payload":{"offerType":"seed","birdId":"uuid"} }]
```
The gateway only **inserts** into `event_log`; it has no authority to write
personality/mood/position. Presence handling: the server credits presence only
for the interval between consecutive pings, **clamped to elapsed wall-clock**, and
ignores any ping whose `active` flag is false or that arrives from a session
reporting hidden/blurred — this prevents a laxer-than-spec presence definition
from silently inflating drift (concepts.md's central correctness concern).
Offer requests are accepted always but the engine enforces the per-bird cooldown
(§5.5); a cooled-down offer is logged and produces no curiosity drift.

### 4.4 Visit-invitation flow
- `POST /api/invites` `{ visitorEmail }` → issues a one-time token, emails the
  visitor a link `https://app/visit/<token>`. Host-only. Default off means: no
  invite exists until this is called.
- `DELETE /api/invites/:id` → revoke; effective at the visitor's next snapshot
  pull (§8.4).
- `GET /api/invites` and `GET /api/visits` → outstanding invites and the visit
  log (host settings; pull-only, no badge).
- `GET /api/visit/:token/snapshot` → resolves token → host aviary; returns the
  **same** snapshot the host would see (no show-off rendering), minus any
  interaction directives. Visitor events are **never** written to the host's
  event log (visitor attention must not drift the host's birds). On
  revoked/expired/used token → matter-of-fact "visit no longer available".

### 4.5 Account/system routes (matter-of-fact copy)
`POST /api/auth/magic-link` (request), `GET /api/auth/consume?token=` (15-min
expiry, single-use), `GET/POST /api/account/settings`, `POST /api/account/email`
(verify-before-switch), `GET /api/account/export` (emails a download link),
`POST /api/account/delete` and `POST /api/account/recover` (30-day soft window),
`GET/DELETE /api/sessions`.

---

## 5. Simulation engine design

The engine is a **pure function** of (previous canonical state, ordered events
since last tick, wall-clock, local-time basis, weather schedule):
`tick(state, events, now) -> (state', notebookCandidates, narrationCandidates)`.
Purity makes it testable, replayable, and trivially correct under catch-up.

### 5.1 The tick
Cadence ~once/minute **[call: 60 s nominal, calibrated in build]**.
- **Scheduler** enqueues aviaries due to tick: any aviary with un-consumed events,
  active presence, or whose mood/day-night state would change with elapsed time.
- **Worker** takes an aviary, acquires a row-level advisory lock
  (`SELECT … FOR UPDATE` on the bird set), reads `event_log` rows with
  `seq > last_drift_tick_seq`, runs `tick(...)`, writes new state and the new
  `last_drift_tick_seq` in one transaction, invalidates the edge snapshot cache,
  enqueues any notebook/narration candidates. Ordered + idempotent: re-running a
  tick over the same `seq` range is a no-op because deltas key off the high-water
  `seq`.
- **Dormant aviaries** (no client, no events for a long stretch): not ticked every
  minute. On the next snapshot pull, a **catch-up** runs `tick` forward over the
  elapsed interval (zero presence/interaction input; only time-of-day, weather,
  and mood-timer evolution). Because `tick` is pure and time-driven transitions
  are deterministic, catch-up state is *identical* to having ticked all along —
  the user still returns to "the aviary that has been running." Notebook entries
  that would have fired overnight are back-filled during catch-up.

This hybrid is the single scaling decision that keeps "continues without the
viewer" true without ticking millions of idle aviaries every 60 s.

### 5.2 Personality drift function
Traits `P ∈ [0,1]^5`. Drift is a **slow low-pass filter** over presence-and-
interaction signals, **clamped non-negative** (monotonic toward expressive —
invariant #5).

Per tick, for each bird `b`, accumulate window inputs:
```
S_present  = clamp(credited_presence_seconds, 0, wallclock_elapsed)     // dominant
S_listen_b = listen_in_seconds_focused_on_b                              // strong, bird-specific
S_offer_b  = accepted_offers_near_b                                      // small → curiosity/boldness
// settle contributes no directional drift (just clean window end)
```
Convert to a bounded "expressiveness pull" per trait, then apply a saturating
step that slows near the ceiling and can never decrease:
```
pull_boldness   = a1*S_present + a2*S_offer_b
pull_warmth     = b1*S_present + b2*S_listen_b
pull_vocalfreq  = c1*S_present + c2*S_listen_b
pull_plumage    = d1*S_present
pull_curiosity  = e1*S_present + e2*S_offer_b

for trait t:  P_t' = P_t + max(0, k_t * tanh(pull_t / scale_t) * (1 - P_t))
```
- `(1 - P_t)` makes it a low-pass toward the expressive ceiling — naturally slow,
  no single session moves a trait visibly.
- `tanh(... )` bounds per-tick contribution so a marathon session can't spike.
- All deltas `≥ 0` → monotonic. **Property test** asserts `P' ≥ P` for any log.

**Calibration target (testable, from bird_engine.md):** with "regular visits"
(nominal: ~10 min/day, ~5 days/wk), the instrument harness must detect drift after
~1 week (e.g. ≥ one trait moved by a small instrument-detectable Δ), and the
*behavioral* expression must cross a visible threshold by ~3 weeks (e.g. greet
order or front-perch tendency demonstrably changes). The `k_t`, `a..e`, and
`scale_t` constants are tuned by the calibration harness (§5.8); the plan ships
nominal starting values and the harness, not magic numbers.

**Why neglect looks quiet without lowering traits:** greeting/approach/vocal
*expression* is `f(P_t, mood, recent_presence_decay)`. When recent presence is
low, a decaying recency term lowers expressed greeting frequency — the bird is
"ambient" — while `P` is untouched. Resume presence and the bird greets at its
(higher, drifted) level. This is the precise mechanism that delivers
"quieter, not mistrustful" (non_goals: no Tamagotchi).

### 5.3 Presence (the dominant drift input — exact by design)
A presence-event is credited only when, simultaneously: `visibilityState ===
'visible'` **and** the document has window focus **and** a pointermove/keypress
occurred within the activity window. All three; any one alone is insufficient
(concepts.md). The client evaluates this conjunction and emits `presence_ping`
events covering the active interval; the server **clamps credit to wall-clock**
and discards pings not flagged active. Activity window **[call: start at 3 min,
lean longer]** — watching without moving *is* the product, so presence is not lost
the instant the mouse stills; it's lost after sustained no-show. Settle and
tab-close are both terminal and identical at the engine level; neither is
penalized; there is no "you didn't settle" surface.

### 5.4 Mood transitions (fast timescale)
Mood is a small enumerated state with **hysteresis**. Each tick, score candidate
moods from weighted inputs and transition only if a candidate clearly beats the
current state and the dwell floor (`dwell_until`) has passed — prevents flicker.
Inputs:
- recent in-session interactions (accepted offer → nudge `content`; alarm context
  → `wary`),
- local time-of-day (`drowsy` near dusk, `alert` early morning, `settled`/
  `sleeping` at night),
- ambient weather (rain → briefly damp vocal tendency; wind → some `alert`, some
  `wary`),
- the bird's own personality (high boldness resists `wary` on the same input),
- **bird-to-bird contagion**: a `wary` neighbor raises nearby `wary` scores; a
  chorus context raises `content`/`alert`.
Mood **persists across sessions**: session-start mood = session-end mood as
advanced by intervening ticks/catch-up (never snaps to neutral on tab open).

### 5.5 Offers, listen-in, settle (engine effects)
- **Offer** (seed/song-fragment/still-pool): reaction = `g(mood, curiosity)`.
  Curious+content approaches; wary waits then nears; drowsy may ignore. Song
  fragment plays a soft motif; response shaped by vocal frequency + mood. Still
  pool drops a reflective surface; drink/bathe/watch by mood. **Per-bird cooldown
  ~few minutes [call: 3 min]** — functional, prevents single-session curiosity
  saturation; a cooled offer yields no drift. Offers reach the engine only via the
  top-bar affordance (not by clicking a bird — layout rule).
- **Listen-in** start/end are events; sustained focus on `b` feeds
  `S_listen_b` (warmth/vocal drift). The *mix* itself is a pure client concern
  (§7.3); the engine only records attention.
- **Settle** ends the presence window cleanly and quiets calls; **no directional
  drift**. The 5 s undo is a client interaction; if undone, no settle event is
  committed (or a `settle_undo` cancels it) so the window isn't prematurely closed.

### 5.6 Call-grammar runtime
Per species: a **motif library** (small set of pitch/rhythm motifs) + a timbre
profile (the recognizable signature). The engine *schedules* calls — when a bird
calls, which motif seed, how many steps, pitch bias — shaped by vocal_frequency
(more frequent/longer when high) and mood (drowsy → sparse/low; alert → crisp).
The **client synthesizes** the scheduled call (§7). Recognizability invariant:
the motif skeleton + species timbre are held constant across mood and drift;
only the *variable* params (jitter, pitch bias, length, ornamentation) move — so
a user knows Pip by ear after two weeks even as vocal_frequency drifts up. Two or
more high-vocal birds whose scheduled calls overlap produce an **emergent
chorus** (not a scripted chorus event). The **caption** for any call is generated
from the *same* scheduled params (§7.7) so the caption matches what played.

### 5.7 Naturalist prose engine (notebook + narration + captions — one voice)
**[call]** A writer-authored **generative phrase grammar**, not a runtime LLM:
1. The tick emits structured **moment facts** — e.g. `greet_order_first(pip)`,
   `first_this_week(greet_order, pip)`, `fluffed_against_cool(wren)`,
   `long_quiet_stretch(morning)`, `leaf_drift_unnoticed`. Facts are about the
   *aviary*, never about the *user's behavior* (no "you visited" facts can be
   expressed — invariant #3).
2. A weighted grammar realizes a fact into prose with combinatorial variation,
   lowercase, present-tense, specific, bird-named, no exclamation, no "you":
   `"pip greeted before wren today, first time this week."`
3. **Notebook sparsity:** entries are rare (~1 per few days for a regular
   aviary; more only when genuinely noteworthy). Enforced by a per-account
   rate limiter + novelty gate over `fact_digest`, so active users don't get a
   feed. Read-only forever; infinite scroll-back; never archived.
4. **Narration** uses the same engine at a slow cadence (~1 update / 30–60 s
   idle; prompt on user-initiated events) into an `aria-live="polite"` region —
   running prose, never a state list (§8).
5. **Captions** use the same engine on call params — short call descriptions
   (`"a soft three-note rise"`), generated at runtime to match the played call.

Using one engine guarantees a screen-reader user, a caption reader, and a
notebook reader hear the *same product voice*. Keeping it grammar-based (not an
external LLM) also satisfies the privacy rule: per-bird facts never leave to a
third-party model.

### 5.8 Calibration & drift instrument harness
A standalone harness drives the pure `tick` with scripted presence/interaction
patterns (e.g., "regular visitor", "weekend-only", "two-week absence then return")
and asserts the calibration targets (§5.2). This is how `k`, `a..e`, `scale`, the
mood weights, and the activity window are tuned. It runs in CI as a regression
gate on engine changes. Crucially, calibration uses **synthetic** patterns and
opt-in internal test accounts — never aggregated real-user per-bird data
(invariant #9).

---

## 6. Sync model

### 6.1 One canonical state, many readers
There is no client-to-client sync because there is nothing to sync: both the
laptop and the phone `GET /snapshot` from one canonical record. The server is the
only writer of personality/mood/position. Multi-device coherence is therefore a
*property of the architecture*, not a feature with its own code path.

### 6.2 Conflicts are prevented, not resolved
- **Personality:** clients never send absolute values; they send events. The tick
  applies **additive, server-authored deltas in `seq` order**. There is no
  last-write-wins path because no client write targets personality. Worked
  example the design defeats: morning laptop session and a lunch phone session
  (read before the laptop's write) cannot clobber each other's drift, because
  neither writes the vector — both only appended events, and the tick folds both
  event ranges in order. The "slowly-drifting-too-slow, no error logged" silent
  data loss from accounts_sync.md is structurally unreachable.
- **Idempotency:** `client_event_id` dedupes retries and magic-link replays.
- **Ordering:** per-account `seq` is the single ordering authority.
- **Type boundary:** `PersonalityVector` lives in a server-only package; the
  client snapshot type cannot reference it (invariant #4). A compile-time test
  asserts the client bundle's type graph never includes it.

### 6.3 Interpolation, not teleporting
A bird at perch A in snapshot N and perch B in N+1 is rendered moving smoothly
(eased flight path); mood/behavior changes cross-fade. The client interpolates
between the last two snapshots and extrapolates idle micro-motion from `motion`
fields, so motion stays continuous even between ~minute-cadence snapshots.

### 6.4 Reconnect / resume
On `visibilitychange → visible`, on a long rAF gap (suspend/resume), and on
keepalive, the client pulls a fresh snapshot and *re-seats* birds at their current
canonical motion phase — never an intro animation, never a fade-from-static: it
simply resumes as if it had been rendering all along (invariant #1).

---

## 7. Frontend rendering pipeline

### 7.1 Composition
A single horizontal scene, one screen, **no pan/scroll/zoom**. Three depth planes:
background (sky + soft foliage), mid-plane (perches + birds across front/middle/
back zones), foreground (occasional passing branch/leaf). Subtle parallax only —
not parallax-heavy. **[call]** Renderer: a hand-written Canvas2D scene module with
an optional WebGL2 path for headroom; chrome (top bar, settings, notebook, visit
flow) is a small reactive UI lib (Preact/Solid-class, code-split). No game engine
dependency — protects the 2 MB budget and the 60 fps floor (§9).

### 7.2 Idle micro-motion (mood-shaped, procedural — never canned clips)
Each bird's idle motion is a **parametric** blend (preen / scan / head-tilt /
weight-shuffle) driven by per-bird noise seeded from the snapshot + mood weights:
a wary bird sits back and scans more; content preens; curious tilts toward sounds
and watches leaves; drowsy sits low, fluffed. The user reads mood from motion with
**no label, tooltip, or status icon** (bird_engine: mood is the visible surface).
Motion runs continuously and never reads as paused while visible.

### 7.3 Transitions & listen-in visual
Perch changes are eased flight paths; mood shifts cross-fade. Greeting forms
(glance / two-note / step-forward / long-call-then-response) are realized
procedurally from the `greeting` directive and staggered by a small random offset
when multiple birds would greet (never a unison cue — that would "announce").

### 7.4 First-frame, load, and empty states
- **First frame mid-action** — hydrate from the inlined snapshot, place every bird
  at its `motion.phase`, start the loop. No spinner, no fade-from-static
  (invariant #1).
- **Slow snapshot** — a **quiet field** (soft sky, one or two faint motion cues),
  never a spinner ("a spinner says machine").
- **Empty aviary** (post-adoption, pre-first-bird) — same quiet field; the first
  bird enters with a soft fly-in to its starting perch; the user never sees an
  empty aviary again.

### 7.5 Day/night, weather, ambient ornaments
Palette interpolates by the account's local time (sunrise warm-up, bright midday,
warm quiet evening, dim night; the nightjar-class species stays active at night).
Weather (rare rain/wind) is a soft overlay tied to the snapshot's `weather`
field. Leaves/feathers drift at idle cadence as **pure client ornaments** (no
per-leaf server state) — they keep the scene alive between bird actions.

### 7.6 Top bar & chrome
A thin top bar above the scene: account/settings, accessibility settings, field
notebook, offer affordance — nothing else. **No UI chrome inside the aviary**
(no inline buttons/badges/tooltips/overlay icons). The bar **fades** to near-
transparent after a few seconds of cursor stillness; returns on
pointer/keyboard. Offers are reached here, never by clicking a bird.

### 7.7 Responsive & frame budget
Scene compresses on narrow (phone) and widens on desktop, **never cropping a bird
out of frame**; aspect handling keeps all birds visible. Fixed-timestep update
with render interpolation; object pooling for birds/ornaments to hold 60 fps and
zero memory growth (§9).

---

## 8. Audio pipeline

### 8.1 Procedural synthesis (no recorded audio — invariant #6)
WebAudio with an **AudioWorklet** synth so synthesis runs off the main thread
(protects 60 fps and avoids main-thread GC). Each species timbre is built from
oscillators/wavetables + filtered noise + envelopes; the call grammar (§5.6)
supplies motif seed, pitch bias, length, ornamentation per call instance. Voices
are **pooled** and buffers **reused** — no per-call allocation (§9 "no memory
growth"). Result: every call varies; the user never hears the exact same call
twice (the audible signature of dead software is forbidden).

### 8.2 Chorus mixing
Each bird has its own call scheduler and synth voice through a shared graph;
overlapping high-vocal birds produce a **real chorus** of independently-varying
calls — not two stacked loops (which phase-cancel audibly). This is why
procedural synthesis is non-negotiable, not just a bundle-budget consequence.

### 8.3 Listen-in mix (re-balance, never mute)
Focusing a bird ramps its voice gain up and the others **down to ambient (never
silent)** via `setTargetAtTime` slow ramps — listening, not channel-switching.
Disengage (re-click the bird, focus another, click empty space, or move keyboard
focus away) ramps back to the ambient mix with the same slow decay.

### 8.4 Fallback
If WebAudio is unavailable (old browser, denied audio context, hardware issue):
**graceful silence with captions on by default** (§9). No recorded-audio fallback
path exists — silence+captions beats canned audio, and a recorded path can't meet
the variation bar or the bundle budget anyway.

---

## 9. Accessibility surfaces (first-class, ships *with* v1)

The stance: a screen-reader, reduced-motion, or audio-off user gets *the actual
product* — an aviary that feels alive — not a stripped, state-announcing fallback.

- **Screen-reader narration** — naturalist running prose (§5.7) into an
  `aria-live="polite"` region, slow cadence (~1 / 30–60 s idle; prompt on
  user-initiated events via a small priority bump that stays polite). Never a
  state list, never "Pip mood: content," never ARIA-label automation. Same voice
  as the notebook so moving between surfaces feels like one product.
- **Reduced-motion mode** — a *designed* surface, not "animations off." Triggered
  by `prefers-reduced-motion` or the settings toggle. Micro-motion becomes slow
  cross-fades between still poses; flight becomes perch-to-perch cross-fades;
  ambient leaf drift is removed; day→evening color shifts remain, slowed. Calls,
  drift, mood, and notebook all continue. It has its own calm aesthetic — a
  reduced-motion user gets a *calmer* Pocket Aviary, not a broken-looking one.
- **Captions** — opt-in; short naturalist prose per call (`"a low trill, paused,
  low trill again"`), generated from the played call's params, fading near the
  calling bird; on by default in the WebAudio fallback.
- **Keyboard navigation** — Tab cycles top-bar items; Tab into the scene focuses
  the first bird; arrows move focus between birds; Enter triggers listen-in on the
  focused bird; Escape exits listen-in; the offer affordance opens via a top-bar
  shortcut and is fully keyboard-navigable; settle reachable from the top bar.
  Focus indicators are a soft high-contrast outline legible against bright and dim
  aviary states.
- **Contrast** — all user copy (top-bar labels, settings, account/error surfaces,
  captions, visually-displayed narration) passes **WCAG AA** minimum; the design
  system pins exact ratios. The scene itself carries no copy except the top bar.

A11y is acceptance-gated for launch (§10, §11): shipping reduced-motion or
narration "in v1.1" is treated as telling those users the product wasn't for them.

---

## 10. Performance budgets & observability

### 10.1 Budgets (hard, CI-enforced)
- **Initial JS bundle < 2 MB gzipped at first paint.** Tactics: hand-written
  renderer (no game engine), AudioWorklet synth (no audio assets), procedural /
  small-SVG / compact-bitmap bird visuals, aggressive **code-splitting** of less-
  frequent surfaces (account settings, accessibility settings, visit-invitation
  flow). A `size-limit` gate fails CI on regression.
- **Time-to-first-bird < 500 ms** on a mid-tier mobile / 4G. Tactics: edge-
  delivered HTML with the **first snapshot inlined** (from a CDN edge near the
  user), a render path that draws the first bird before non-critical assets/audio
  init, deferred audio-context start (after first paint or first gesture).
  Measured by synthetic checks and aggregate RUM.
- **60 fps idle motion on a 5-year-old mid-range laptop**, sustained over a 30-min
  session (runtime budget, not just launch). Tactics: fixed-timestep + interp,
  object pooling, worklet audio, minimal layout thrash.
- **No memory growth over 30 minutes** — a real CI test (headless browser, 30-min
  soak, heap assertion). Buffers/voices pooled; scrolled-out notebook entries
  release references; audio contexts/worker threads bounded.

### 10.2 Observability (aggregate only — privacy boundary at the metric)
Synthetic browsers run the aviary on a schedule from common geographies;
aggregate-only RUM captures page-load timing, first-bird-render timing,
render-frame timing, audio-context error counts, and simulation-tick latency.
**None** carries per-bird or per-account interaction dimensions. **Tick latency
p99 alarms above 5 s.** The telemetry pipeline has no read path to the canonical
state DB; the canonical DB is never read by the analytics warehouse; any future
ML never receives per-bird fields (invariant #9, §7.7 schema test).

### 10.3 Browser support
Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers get a
matter-of-fact unsupported-browser surface (no compatibility shims that bloat the
bundle).

---

## 11. Guardrails (how the invariants survive a team)

Each load-bearing rule from §0 has an automated gate; "charm" is regression-tested
like correctness:
- **Charm/affective suite** — boot path asserts no loading/spinner component on
  the aviary route; greeting directive produces ≥ N distinct realizations over M
  runs (variation is real, not 3 rotated variants); a "no announcement" AST lint
  bans toast/banner/notification primitives from product chrome.
- **Engine property tests** — `P' ≥ P` for arbitrary logs (monotonic drift); mood
  hysteresis prevents flicker; catch-up state equals continuously-ticked state;
  calibration targets (~1 wk instrument, ~3 wk visible) hold.
- **No-recorded-audio bundle gate** — fails on any audio asset in the build graph.
- **Personality-isolation type test** — `PersonalityVector` is unreachable from
  client code; snapshot payloads contain no trait floats.
- **Single-writer DB grants** — only the simulation worker role may write
  personality/mood/position; the event ingress may only INSERT events.
- **PII gate** — no email-as-identifier in code, logs, keys, or event schemas;
  log serializers drop `email`.
- **Telemetry schema gate** — rejects any per-account/per-bird dimension.
- **Voice-bundle lint** — system surfaces import only matter-of-fact copy; aviary
  surfaces only naturalist copy.
- **Budget gates** — `size-limit` (<2 MB gz), synthetic first-bird (<500 ms),
  30-min memory soak, 60 fps frame-budget check.

---

## 12. Rollout

### 12.1 Build sequence (each stage independently demoable)
1. **Foundation** — accounts (synthetic UUID, encrypted email), magic-link auth,
   sessions, canonical state DB, append-only event log, edge snapshot serving,
   tick skeleton (time-of-day + mood timers only). Guardrails #7, #8 wired first.
2. **Engine** — personality vectors, monotonic drift, mood state machine, call
   scheduling, bird-to-bird interaction; calibration harness (§5.8) online.
3. **Client core** — renderer, interpolation, idle micro-motion, first-frame-mid-
   action boot, top bar + fade, day/night, ambient ornaments.
4. **Audio** — AudioWorklet synth, chorus, listen-in mix, fallback.
5. **Interactions** — return-greeting, listen-in, offer (+cooldown), settle (+undo),
   presence accounting, notebook generation (sparse).
6. **Accessibility** — narration, reduced-motion mode, captions, keyboard nav,
   contrast. **Gated as launch-blocking** (ships with v1, not after).
7. **Social** — invites, read-only ambient visit proxy, revocation/expiry, visit
   log, opt-in notify.
8. **Perf & polish** — meet all §10 budgets; run the full guardrail suite.

### 12.2 Birds-per-aviary ramp
Start at 2 (system-selected species, named at adoption, renameable). Additional
birds are offered by **aviary age**, not visit count/score/tier (bird_engine):
e.g. a 3rd around a few months, growth toward 5–6 over ~a year. The pacing is a
feature-flagged, server-side schedule so it can be tuned without client releases.
Cap is 7 (the recognizability ceiling of the call mix); the cap is built into the
engine and adoption flow.

### 12.3 Instrument from day one (aggregate only)
Perf RUM, tick-latency p99 (5 s alarm), audio-context error rate, HTTP error
rates, synthetic geo checks. Drift calibration is observed via the synthetic
harness and opt-in internal accounts — never by mining real-user per-bird data.

### 12.4 Launch gate
A release is blocked unless: all budget gates pass; the affective/charm suite
passes; accessibility surfaces (narration, reduced-motion, captions, keyboard) are
complete; the privacy/PII/telemetry gates pass; and the engine calibration
regression holds.

---

## 13. Risks & mitigations

- **Drift miscalibration (the central engine risk).** Too fast → Tamagotchi-by-
  clicking; too slow → screensaver. *Mitigation:* the `(1-P)` low-pass + `tanh`
  bounding keeps single sessions invisible; the calibration harness (§5.8) gates
  the ~1-wk-instrument / ~3-wk-visible targets in CI; constants are tunable
  server-side without client releases.
- **Monotonic-drift misread.** An engineer "fixes" neglect by adding negative
  drift (symmetric model), reintroducing punishment. *Mitigation:* invariant #5
  property test; the quiet-on-neglect behavior is implemented in the recency/mood
  layer, documented as the intended mechanism so it isn't "corrected."
- **Sync correctness / silent personality loss.** A future code path writes
  personality from the client (last-write-wins). *Mitigation:* DB single-writer
  grant (#7), additive `seq`-ordered deltas, idempotent events, property test that
  divergent device reads cannot clobber drift.
- **Audio uncanniness.** Looping artifacts, chorus phase-cancellation, worklet
  glitches/underruns. *Mitigation:* procedural-only (#6), independent per-voice
  synthesis, voice pooling, soak tests for glitch/underrun, the
  "never-the-same-call-twice" assertion.
- **Accessibility regression.** Narration degrades into a state list; reduced-
  motion becomes "animations off"; a11y slips to a later release. *Mitigation:*
  one shared voice engine (#10/§5.7); reduced-motion as a designed render mode;
  launch-blocking a11y gate (§12.4).
- **Announcement creep.** "Just a small welcome toast." *Mitigation:* invariant #2
  AST lint; the greeting directive is the only return affordance; review training
  in the guardrail docs.
- **Gamification creep.** A "harmless" streak/visit count. *Mitigation:* invariant
  #3 — no counting fields in the model; the notebook grammar cannot express
  user-behavior facts; reviewed sign-off required to add any counter.
- **Privacy boundary erosion.** Email reused as a key; telemetry reaches into the
  sim DB; per-bird data feeds a model. *Mitigation:* invariants #8/#9 — synthetic
  UUID everywhere, encrypted email in one place, no analytics read grant on the
  sim DB, telemetry schema gate, grammar-based prose (no external LLM on per-bird
  facts).
- **Bundle creep over 2 MB / first-bird regression.** *Mitigation:* `size-limit`
  and synthetic first-bird gates in CI; no game-engine dependency; aggressive
  code-splitting; procedural assets.
- **Tick scaling cost.** Ticking idle aviaries every minute is wasteful.
  *Mitigation:* continuous tick only for recently-active aviaries; deterministic
  catch-up on wake for dormant ones (identical canonical state).
- **Presence inaccuracy/gaming.** A laxer "tab open" definition inflates drift
  across the population. *Mitigation:* strict 3-signal conjunction client-side +
  server clamp to wall-clock + discard of non-active pings (#5.3).
- **Day/night correctness.** Timezone/DST errors make the user's morning the
  aviary's afternoon. *Mitigation:* IANA `local_time_basis` per account, refreshed
  on sign-in; day/night derived from it server-side; tested across DST boundaries.
- **Magic-link security.** Link replay or interception. *Mitigation:* 15-min
  expiry, single-use invalidation on consume, per-email rate limiting, idempotency
  on consume, httpOnly secure session cookies, revocable per-device sessions.

---

## 14. Open calibration items (resolved in build, not blocking design)
Exact tick cadence (~60 s); presence activity window (~3 min, lean longer);
offer cooldown (~3 min); narration cadence (30–60 s); the drift constants
`k_t/a..e/scale_t`; mood-transition weights and dwell floors; age thresholds for
3rd–7th birds; exact species roster (~6) and their motif libraries; palette/
contrast ratios (design-system owned). Each has a nominal starting value above and
a harness/test that tunes and protects it.
