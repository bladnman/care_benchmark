# Pocket Aviary — v1 Implementation Plan

A delivery-ready plan for a frontier engineering team. The PRD establishes *what* and *why*; this document fixes the executable *how*: service shape, data model, protocols, engine math, render and audio pipelines, accessibility surfaces, perf budgets, rollout, and risk controls. Where the PRD leaves a value open ("calibrate during build"), this plan picks a defensible starting value, marks it `[CAL]`, and points at the instrument that retunes it. Where the PRD draws a hard line (no last-write-wins, presence = three-signal conjunction, monotonic drift, no gamification), this plan turns that line into an architectural property a reviewer can check, not a convention a contributor can erode.

The organizing discipline: **aliveness is the product**, and aliveness is the conjunction of four engine invariants — server-authoritative simulation, monotonic-toward-expressive drift, procedural (never recorded) audio, and notice-never-announce framing. Every component below is justified by which invariant it protects.

---

## 0. Reading guide for the build team

- `[CAL]` = calibration value with a starting number and a named instrument that retunes it. Never a magic constant; always traceable.
- `[INV]` = invariant. Code that violates it must fail review or CI, not just be discouraged. Each `[INV]` lists its enforcement mechanism.
- `[CALL]` = a defensible interpretive call this plan makes where the PRD was silent or ambiguous, with the rationale so the team can revisit it deliberately rather than by accident.
- Effort note: this is the extra-high-effort plan; it specifies engine math, wire formats, and CI gates concretely so a separate team executes without a second clarification pass.

---

## 1. Scope

### 1.1 In scope for v1

**Identity & accounts.** Single-user accounts; one canonical aviary per account; email magic-link sign-in (15-min link expiry, single-use, per-email rate limit); per-device revocable session tokens; email change with new-address verification; on-demand JSON account export delivered by emailed link; soft-delete (30-day recoverable window) then hard delete.

**The aviary & bird engine.** 2 starter birds, cap 7; ~6-species pool with per-species silhouette/palette/call-motif library; hidden per-bird personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); fast-timescale mood (wary, content, curious, drowsy, alert); server-side simulation tick (~60s `[CAL]`) that drifts personality, transitions mood, and writes canonical state; procedural call synthesis; bird-to-bird interaction and emergent chorus; stable internal bird IDs; user-assigned renameable names; age-gated availability of additional birds.

**Interactions.** Procedurally-varied return-greeting keyed to absence length + boldness + mood; listen-in (ramped mix re-balance, never mute); offer (seed / song-fragment / still-pool, per-bird cooldown); settle (evening lighting gesture with 5s undo); presence accounting (three-signal conjunction); auto-generated read-only field notebook with enforced sparsity.

**Scene & rendering.** Single horizontal non-panning scene; three perch zones (front/mid/back) as read-only signals; local-time day/night cycle; rare ambient weather; continuous ambient micro-motion and leaf/feather drift; thin auto-fading top bar (account, accessibility, notebook, offer); "motion already in progress" first frame; quiet-field loading and empty-aviary states (never a spinner).

**Sync & data integrity.** Server as sole writer of personality state; clients write to an append-only event log; snapshot-pull + interpolation; multi-device coherence as an architectural property (no client-to-client sync); additive server-authored deltas (no last-write-wins).

**Social (one feature).** Host-issued, per-invite, email-addressed, read-only ambient visits; revocable; 30-day invite expiry; silent visit log; opt-in (off by default) visit notifications; no co-presence, chat, avatars, comments, discovery, or leaderboards.

**Accessibility (first-class, day-one).** Naturalist screen-reader narration (slow cadence, prioritized user events); reduced-motion as a *designed* cross-fade rendering (not animations-off); runtime-generated call captions in naturalist voice; full keyboard navigation; WCAG AA on all user copy; WebAudio-unavailable → graceful silence + captions-on (no recorded-audio fallback).

**Privacy & telemetry.** Synthetic-UUID identity everywhere except the single encrypted email field; per-bird/per-account interaction data used *only* to drive that account's own simulation; aggregate-only operational telemetry with the per-account dimension architecturally excluded; in-product plain-text privacy policy.

**Performance.** Initial JS bundle ≤2MB gzipped at first paint; time-to-first-bird <500ms on mid-tier mobile/4G; 60fps idle on a 5-year-old laptop across a 30-min session; zero memory growth over 30 min (CI-tested); synthetic perf checks + aggregate RUM; tick p99 latency alarm at 5s.

### 1.2 Explicitly out of scope (non-goals respected as hard exclusions)

Native iOS/Android apps; **all** gamification (achievements, streaks, levels, scores, badges, adopted-count, green-dot calendars, XP, rank, tier, "every day this week" anywhere — not even as an opt-in toggle); Tamagotchi mechanics (death, hunger, distress, decaying happiness meter, negative drift on neglect); social-network surfaces (profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments); payments; shared/multi-profile aviaries; customizable scenes; multi-aviary accounts; push notifications about the aviary; recorded-audio path of any kind.

These are not "later"; the data model and protocols are deliberately built so re-adding them is *harder*, not a flag-flip (see §1.4 and §13).

### 1.3 Interpretive calls made at scope level

- **`[CALL]` Render runtime:** Canvas2D for the bird/scene layer (not WebGL, not DOM-per-bird). Rationale: ≤7 sprites + a handful of ornament particles is trivially within Canvas2D's budget at 60fps on old hardware; WebGL adds context-loss handling, shader pipeline, and bundle weight we don't need; DOM-per-bird makes 60fps idle micro-motion fragile under layout/compositing. Top bar and overlays (captions, narration mirror, settings) are DOM for accessibility. Revisit only if profiling on the reference 5-year-old laptop fails the 60fps gate.
- **`[CALL]` Monorepo, two deployables:** one repo, two services (`edge`/API + `sim` tick worker) sharing a typed schema package, plus the web client. Small team, shared types, atomic cross-cutting changes (e.g., adding an event kind touches client emit, schema, and tick consume in one PR).
- **`[CALL]` Personality numbers never cross the API boundary.** Not just "never shown in UI" — the snapshot DTO has no field that carries a raw trait scalar. The closest the wire gets is *derived presentation hints* (see §4.3). This makes "personality vector is never exposed numerically" `[INV]` enforceable by schema, not by UI discipline.

### 1.4 Anti-feature architecture (scope as enforcement)

The non-goals are enforced structurally, not by reviewer vigilance alone:

- **No cross-account aggregation pipeline exists.** The simulation database has no reader in the analytics path (§11). Because the wiring is absent, a leaderboard/"average drift" dashboard cannot "just be turned on" — someone would have to build the forbidden pipeline first, which is a visible, reviewable act.
- **No visit-frequency datum is materialized in any user-facing store.** Presence-time exists only as a drift input inside the sim DB; there is no per-day visit table, no streak field, no last-N-days array anywhere a UI could bind to. A streak counter would require *creating* the data it needs — again, a visible act, not a flag.
- **Notebook generator is constrained at the data layer** to emit observations of *the aviary*, never observations of *the user* (§7.4). The generator literally has no access to a "days visited" or "last visit" feature in its input contract.

---

## 2. Architecture

### 2.1 Service shape

Three runtime pieces plus shared schema:

1. **`edge` (API + static delivery).** Stateless request service behind a CDN. Responsibilities: auth (magic link issue/consume, session tokens), snapshot read endpoint, event-log write endpoint, account management (export, delete, email change, sessions, settings), visit invite issue/revoke/consume, visit read-only snapshot proxy, privacy policy serving. Also serves the HTML shell with an **inlined initial snapshot** (§9.2) so first-bird beats 500ms. Stateless ⇒ horizontally scalable; holds no simulation authority.
2. **`sim` (simulation tick worker).** The *only* writer of personality vectors and canonical mood/scene state. Runs the tick loop per active aviary at ~60s `[CAL]` cadence. Consumes the append-only event log in order, computes additive drift deltas and mood transitions, writes canonical state. Runs regardless of client connectivity. This is the architectural embodiment of "the aviary continues without the viewer" and of "server is the only writer."
3. **Web client.** Pulls snapshots, interpolates, renders (Canvas2D scene + DOM chrome), synthesizes audio (WebAudio), emits interaction events. Owns *zero* canonical state; owns only ephemeral render/interp state and an outbound event buffer.

Shared: a typed **schema package** (DTOs, event-kind enum, snapshot shape) imported by all three so wire contracts can't drift.

### 2.2 Client/server split — the load-bearing boundary `[INV]`

> **`[INV] Server-authoritative state.** Clients never compute or write personality vectors, never write canonical mood, never advance the simulation. Clients emit *events* ("listened in to bird X for 182s") and *render snapshots*. The server decides what events mean.**

Enforcement: (a) the event-write endpoint accepts only the event-kind union from the schema package and rejects any payload carrying a trait/mood absolute value (schema validation, 400 on violation); (b) there is no write path to the personality table reachable from `edge` — only `sim` has the DB grant/role that can write `bird.personality`; (c) a CI architecture test asserts `edge` has no import of the personality-write module and that the DB role used by `edge` lacks UPDATE on personality columns.

This single boundary makes three PRD invariants true at once: server-side continuity, multi-device coherence (both devices read one record), and no-last-write-wins (clients can't write absolute state, so they can't clobber it).

### 2.3 Render pipeline boundary

The render pipeline consumes **snapshots + interpolation targets** and produces frames. It does *not* read raw personality. The snapshot carries (a) per-bird position/perch + motion-state token, (b) current mood enum, (c) call-timing schedule + procedural call seeds, (d) scene state (time-of-day phase, weather token), (e) active transitions (greeting, offer reaction, settle). The boundary is: **simulation owns meaning; client owns motion.** A bird "looks bold" because the sim placed it on the front perch and tagged it `content/curious`; the client never knows boldness is 0.7.

### 2.4 Data flow (steady state)

```
client --(events: presence ping, listen-in, offer, settle)--> edge --append--> event_log
sim tick (every ~60s): read new event_log rows --> compute additive drift deltas
                        + mood transitions + scene advance --> write canonical aviary_state
client --(snapshot pull: on open / visibility / frame-gap / keepalive)--> edge --read--> aviary_state --> client interpolates & renders
```

Events are fire-and-forget from the client's perspective (buffered, retried, deduped by client-generated event id). Snapshots are small (KB) reads. The tick is the only thing closing the loop between input and canonical change.

---

## 3. Data model

Stored server-side. Identity uses synthetic UUIDs everywhere; email is the only PII and lives encrypted on exactly one record `[INV]`.

### 3.1 Account

```
account {
  account_id: UUID (PK, synthetic, generated at creation)  // used in ALL keys, logs, telemetry, shards
  email_encrypted: bytes                                    // envelope-encrypted; the ONLY copy of email
  email_lookup_hash: bytes                                  // HMAC(email) with server pepper, for sign-in lookup only
  created_at, updated_at
  status: enum { active, pending_delete }
  pending_delete_at: timestamp | null                        // status flips hard at +30d
  settings: {
    reduced_motion: enum { system, on, off }                 // 'system' honors prefers-reduced-motion
    captions: bool
    audio_enabled: bool
    visit_notifications: bool   // default false
  }
  pending_email_change: { new_email_encrypted, new_email_lookup_hash, token, expires_at } | null
}
```

`email_lookup_hash` is the key detail that lets us find an account by email at sign-in *without* storing email in plaintext or using it as a key. `[INV] No table outside `account` stores email or derives a key from it.` Enforcement: schema review + a CI grep/lint that forbids an `email` column on any table except `account` and forbids email as a partition/shard key.

### 3.2 Bird

```
bird {
  bird_id: UUID (PK, stable for account lifetime)            // NEVER reassigned on rename/sync/migration
  account_id: UUID (FK)
  species_id: enum (one of ~6)
  name: string (user-assigned, renameable)
  personality: { boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }  // floats in [0,1]
  mood: enum { wary, content, curious, drowsy, alert }
  mood_set_at: timestamp                                     // for mood-timer decay in the tick
  perch: enum { front, middle, back }
  adopted_at: timestamp
  drift_accumulators: { <trait>: float }                     // low-pass filter internal state (see §5.2)
  last_offer_at: { seed, song, pool }  // per-offer-type cooldown timestamps
}
```

`bird_id` stability is the foundation of drift's perceived validity (`bird_engine.md`). `[INV] No code path replaces, regenerates, or "resets" a bird_id.` Even species-pool changes or internal migrations preserve it. Enforcement: bird creation happens in exactly one service function (adoption / age-gated add); migrations are forbidden by review from issuing new ids to existing birds; a fixture test adopts birds, runs a simulated migration, and asserts ids are unchanged.

`personality` is **persisted, never derived** `[INV]`. It is never recomputed from the event log at read time. The event log feeds *deltas*; the vector is the integral and is canonical. Enforcement: there is no function that rebuilds a vector from history; the only writer is the tick's additive-apply (§5.2).

### 3.3 Aviary / canonical scene state

```
aviary_state {
  account_id: UUID (PK)
  birds: [bird_id...]                  // 2..7
  scene: {
    local_tz: IANA tz string           // captured from client, used for day/night anchoring
    weather: { kind: enum {clear, rain, wind}, started_at, ends_at } | null
    settled: bool                       // user settle gesture active
    settled_at: timestamp | null
  }
  age_tier: int                         // derived from account/aviary age; gates next-bird availability
  next_bird_available_at: timestamp | null
  last_tick_at: timestamp
  schema_version: int
}
```

This is the single canonical record both devices read. Day/night is *derived* at snapshot build from `local_tz` + server clock, not stored as a mutable "is_night" flag, so it can never desync.

### 3.4 Event log (append-only)

```
event {
  event_id: UUID (client-generated, for idempotent dedup)
  account_id: UUID
  bird_id: UUID | null                  // null for aviary-wide events
  kind: enum { presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo }
  payload: { ... kind-specific ... }    // NEVER an absolute trait/mood value
  client_ts, server_ts
  consumed_by_tick: bool                // or a high-water mark per account
}
```

`[INV] Append-only; clients write only into this log; the log carries events, never state.` Schema validation rejects any payload field that looks like an absolute personality/mood write. The tick advances a per-account high-water mark so each event is consumed exactly once, in order.

### 3.5 Notebook

```
notebook_entry {
  entry_id: UUID
  account_id: UUID
  created_at: timestamp
  prose: string                          // naturalist voice, lowercase, present-tense
  source_signal: enum (internal)         // what aviary moment generated it; never user-behavior
}
```

Read-only to the user (no edit/delete/annotate). Entries are *aviary* observations only; the generator's input contract (§7.4) excludes user-behavior signals so a "you visited every day" entry is impossible to produce.

### 3.6 Sessions, visits, magic links

```
session_token { token_id, account_id, device_label, created_at, last_seen_at, revoked }
magic_link    { link_id, email_lookup_hash, token_hash, expires_at(=+15m), consumed_at|null }
visit_invite  { invite_id, host_account_id, visitor_email_encrypted, visitor_email_lookup_hash,
                token_hash, created_at, expires_at(=+30d), revoked, status }
visit_event   { visit_id, host_account_id, invite_id, started_at, ended_at|null }  // for the silent visit log; duration approximate
```

Visit invite stores the visitor email encrypted too (it is PII), keyed by synthetic ids. The visit log shows visitor email + date + approximate duration to the host only, on demand.

### 3.7 Storage choices `[CALL]`

- **Primary store:** a single relational DB (Postgres) for accounts, birds, aviary_state, sessions, visits, notebook. Strong ordering and transactions matter for the no-LWW invariant and exactly-once event consumption; a relational DB gives us that cheaply at v1 scale.
- **Event log:** an append-only table in the same Postgres, partitioned by `account_id` + time. At v1 volume (thousands of accounts, ~1 tick/min) this is ample; the high-water-mark consumption pattern is simpler than a Kafka dependency. Revisit (move to a real log/stream) only if tick throughput demands it.
- **Encryption:** envelope encryption for email fields (KMS-managed data keys). Email lookup via HMAC with a server-side pepper.
- `[INV]` The analytics warehouse has **no connection** to this DB (§11).

---

## 4. API surface

REST/JSON over HTTPS. Auth via session token (httpOnly secure cookie). All system-voice error bodies are matter-of-fact (`accounts_sync.md`); naturalist voice never appears in an error/system response.

### 4.1 Auth

- `POST /auth/request-link { email }` → `202` always (no account-existence oracle). Issues magic link if rate-limit `[CAL: 5/email/hour]` allows; emails it. Link valid 15m, single-use.
- `POST /auth/consume { token }` → sets session cookie, returns minimal bootstrap. Consuming invalidates the link immediately (`consumed_at` set, replay → matter-of-fact error).
- `POST /auth/sign-out`, `GET /account/sessions`, `POST /account/sessions/:id/revoke`.

### 4.2 Aviary state — how clients pull

- `GET /aviary/snapshot` → the snapshot DTO (§4.3). Cheap, KB-scale, cache-busting per pull. Client calls it on: tab open, `visibilitychange` → visible, detection of a long render-frame gap (laptop resume), and a low-frequency keepalive `[CAL: every 60s while visible]`.
- The HTML shell from `edge` **inlines the first snapshot** so the first bird renders without a round trip (§9.2).

### 4.3 Snapshot DTO (the personality-safe boundary) `[INV]`

```
GET /aviary/snapshot →
{
  server_ts, tick_seq,
  scene: { tod_phase: enum{dawn,day,dusk,night}, light: {warmth, brightness}, weather, settled },
  age_tier, next_bird_available_at,
  birds: [{
    bird_id, name, species_id,
    perch: enum{front,middle,back},
    mood: enum{...},
    motion_state: token,                 // e.g. 'preening','scanning','head_tilt','shuffle','calling'
    call_schedule: [{ at_ms, motif_seed, pitch_bias, timing_bias }],  // procedural seeds, not audio
    presentation: { plumage_level: enum{1..5}, call_liveliness: enum{low,med,high} }  // DERIVED buckets
  }],
  active_transitions: [{ bird_id, kind: enum{greeting,offer_reaction,settle}, params, started_at }]
}
```

`presentation` carries **bucketed, derived** hints (e.g. plumage_level is a 1–5 visual richness band derived from `plumage_saturation`, never the float). This is the schema-level guarantee that no raw trait scalar reaches the client. `[INV]` enforced by: the DTO type has no float trait fields; a serializer unit test asserts the snapshot JSON never contains a key matching `/boldness|social_warmth|vocal_frequency|plumage_saturation|curiosity/`.

### 4.4 Interaction events — how clients submit

- `POST /aviary/events { events: [Event...] }` → `202`. Batched, append-only, idempotent on `event_id`. Validated against the event-kind union; **rejects any payload carrying an absolute personality/mood value** (400). Clients buffer offline and replay.
- Presence pings are events too: client emits `presence_ping { window_start, window_end }` only when the three-signal conjunction (§5.1) holds, at `[CAL: ~15s]` cadence while present.

### 4.5 Interactions detail

- **listen_in:** `listen_in_start{bird_id}` / `listen_in_end{bird_id}`. Mix change is *client-rendered* (ramped, §6.4); the event is the attention signal that drifts that bird's social-warmth/vocal-frequency (§5.2). Disengage triggers (re-click, focus other bird, click empty, blur) all emit `listen_in_end`.
- **offer:** `offer{bird_id|null, offer_type}`. Server enforces per-bird, per-type cooldown `[CAL: 4min]` (rejects with matter-of-fact "not yet" — but UI just shows the offer affordance disabled; no nag). Reaction is computed by the tick from mood+curiosity and surfaced as an `active_transition` on the next snapshot.
- **settle / settle_undo:** `settle{}` flips scene.settled; client renders the slow evening ramp. `settle_undo{}` (any aviary click within 5s) reverses it. Settle ends the presence window cleanly; tab-close does the same with no event (the absence of presence pings *is* the end). `[INV] settle and tab-close are engine-equivalent; neither produces negative drift.`

### 4.6 Account management

- `GET/PUT /account/settings` (reduced motion, captions, audio, visit notifications).
- `POST /account/email-change` → verification to new address; old email works until verified.
- `POST /account/export` → generates JSON snapshot (birds, names, *current* personality vectors, moods, notebook, settings), emails a download link to the verified address. (Export is the one place the user can obtain their own vectors — it's their data; this does not violate "never shown numerically in product," which is about the in-product relationship surface. `[CALL]` documented so it's a deliberate exception, not a leak.)
- `POST /account/delete` → soft-delete now, recoverable 30d; `POST /account/recover`.

### 4.7 Visit-invitation flow

- `POST /visits/invite { visitor_email }` → emails one-time link; creates `visit_invite` (expires +30d).
- `GET /visit/:token/snapshot` → read-only snapshot of host aviary (same DTO, no interaction endpoints exposed to this token). Revocation/expiry → next pull returns `410`-style matter-of-fact "this visit is no longer available." Visitor pulls do **not** emit events and do **not** drift host birds `[INV]`.
- `GET /account/visits` → host's visit log (visitor email, date, approx duration, outstanding invites). On-demand only; no badge, no push. Optional opt-in notification (off by default) is the *only* notification surface in the product, and even it is per-account and silent unless enabled.
- `POST /visits/:id/revoke`.

### 4.8 Cross-cutting

- All system surfaces matter-of-fact; `Retry-After` on rate limits; no account-existence oracle on auth; CSRF protection on state-changing routes; per-device session revocation.

---

## 5. Simulation engine design

The engine is where the product's promises become math. Four subsystems: presence accounting, drift (slow clock), mood (fast clock), and the call-grammar runtime (the schedule the client renders from). All run inside the `sim` tick.

### 5.1 Presence accounting `[INV]`

> **`[INV] A presence-event requires the conjunction of three independently-checkable client signals: `visibilityState === 'visible'` AND `document.hasFocus()` AND a pointermove/keypress within the activity window. Any one alone is insufficient.**

Client side: a presence detector samples the three signals; when all hold, it accumulates a present-window and emits `presence_ping{window_start,window_end}` at `[CAL: ~15s]` cadence. The instant any signal drops, the window closes; "tab open" alone produces *zero* presence. Activity window starts at `[CAL: 3min]`, leaning long because "watching without moving is the actual product" — retuned by the drift-calibration harness (§5.6), never by guesswork.

Server side: the tick sums presence-window seconds per account from the event log. This is the dominant drift input. The three-signal rule is the load-bearing precision that keeps drift honest across the population — a laxer rule silently inflates everyone's drift. Enforcement of honesty: the server *cannot* manufacture presence (it only sums client-reported windows), and the client emits a window only under the conjunction; a client unit test asserts no ping is emitted if any one signal is false.

### 5.2 Drift function (slow clock) `[INV] monotonic toward expressive`

Drift is a **slow low-pass filter** over presence-and-interaction signals, applied as **additive server-authored deltas** in event-log order.

Per tick, per bird, per trait:

```
raw_signal(trait) = w_presence * presence_seconds_this_window
                  + w_listen   * listen_in_seconds_on_this_bird     (→ social_warmth, vocal_frequency)
                  + w_offer    * offers_accepted_near_this_bird      (→ curiosity; offering at all → boldness)
delta(trait) = alpha * clamp_nonneg( target(raw_signal) - current(trait) )   // low-pass toward an expressive target
trait <- min(1.0, trait + delta(trait))                                       // additive, monotonic up
```

Key properties, each tied to a PRD rule:

- **`[INV] Monotonic toward expressive:** `delta` is clamped non-negative; neglect produces `delta = 0`, never a decrease.** A two-week absence yields *quieter* birds (because no recent presence raises expression) but never *warier/less colorful* ones. This is the engine-level "no Tamagotchi." Enforcement: the apply function clamps to `[current, 1.0]`; a property-based test asserts no input sequence ever lowers any trait.
- **Weights** `[CAL]`: `w_presence ≫ w_listen > w_offer` (presence dominant per PRD). Starting ratio ~ `6:3:1`.
- **`alpha` (filter slowness)** `[CAL]`: tuned so the calibration target holds — *measurable* drift in instruments after ~1 week of regular visits, *visible* drift to a user after ~3 weeks. Starting `alpha` chosen to make a single session's delta below the just-noticeable threshold for any rendered presentation bucket (so no session moves a bird visibly), while a week accumulates an instrument-detectable change. Retuned by §5.6.
- **`drift_accumulators`** hold the filter's internal state so drift is a true integral over time, not a per-tick recomputation.
- **Settle** contributes mood-quieting only; it does not push drift in any direction beyond cleanly ending the presence window.
- **No reset, ever.** Drift accumulates for account life. The vector is canonical and persisted (§3.2).

### 5.3 Mood transitions (fast clock)

Mood is an enumerated state machine per bird, re-evaluated each tick:

```
inputs: recent-session interactions (offer accepted → +content; alarm nearby → +wary),
        local time-of-day (dusk → +drowsy; early morning → +alert),
        ambient weather (rain → dampened vocal_frequency, transient; wind → some +alert, some +wary),
        own personality vector (high boldness ↓ probability of entering wary on same input)
transition: weighted-probabilistic, with hysteresis (mood_set_at gates minimum dwell time to avoid flicker)
```

- **Mood persists across sessions** `[INV]`: it is canonical state, advanced by ticks during absence, never snapped to a default on tab open. A bird that ended `drowsy` at dusk is likely `settled/sleeping` by morning *because the tick moved it through the night*, not because the client reset it.
- **Daily-ish reset** is emergent from time-of-day inputs, not a hard nightly wipe.
- Personality modulates mood (the slow current shapes the fast weather) but mood never writes back to personality except through the drift inputs above.

### 5.4 Call-grammar runtime

Each species has a **motif library** (small set of pitch/timing motifs). The tick produces, per bird, a `call_schedule`: upcoming call events with `motif_seed`, `pitch_bias`, `timing_bias` derived from vocal_frequency (rate + chorus-join readiness) and mood (timing/energy). The client synthesizes from the seed (§6) — the server schedules *when and with what character*; the client realizes the sound. This split keeps audio procedural and per-call varied while keeping call *behavior* server-authoritative and recognizable per bird.

- **Recognizability across drift/mood** `[INV-design]`: a bird's motif library is fixed for its species/identity; drift changes *frequency and timing*, not the motif identity, so Pip stays recognizably Pip by ear even as vocal_frequency rises. This is what makes the 7-bird cap meaningful.
- **Chorus** is emergent: when ≥2 birds with high vocal_frequency have overlapping scheduled calls, the schedule marks a chorus window; the client mixes real procedural calls (not stacked loops), avoiding the phase-cancel artifact recorded loops would produce.

### 5.5 Return-greeting generation

On a snapshot pull that follows a presence gap, the tick (or the snapshot builder reading canonical state + last-presence-end) selects **one** bird to greet, keyed by boldness (bolder → greets first) + current mood, and emits a `greeting` transition whose *form* scales with absence length: short gap → a glance/single note; long gap → a longer call, a step toward the front perch, possibly a second bird's staggered response. `[INV] Greeting is procedurally varied (real variation from seeds), never one of N pre-baked variants; multiple greeters stagger by a small randomized offset, never fire in unison (a unison cue would "announce" arrival).` Enforcement: greeting params include a per-event random seed; a test asserts two greetings under identical state differ.

### 5.6 Calibration harness (how the `[CAL]` values get pinned)

A deterministic simulation harness replays synthetic presence/interaction profiles (e.g., "regular daily 5-min watcher," "weekend-only," "two-week-absent then returns") through the real drift/mood code and asserts:

- regular-daily profile shows instrument-detectable drift by ~7 sim-days and a presentation-bucket change by ~21 sim-days;
- no single session crosses a presentation bucket boundary;
- absent profile shows zero negative drift and reduced *expression* (greeting frequency) but unchanged traits.

These assertions are the source of truth for `alpha`, weights, presence cadence, and activity window. `[CAL]` values ship as named config, retuned only by moving these targets, never ad hoc.

---

## 6. Frontend rendering pipeline

### 6.1 Scene composition

Single horizontal Canvas2D scene, three logical planes (background sky/foliage, mid-plane birds+perches, occasional foreground branch/leaf) with **subtle** parallax. No pan/scroll/zoom. Three perch zones map to fixed scene anchors (front/mid/back) scaled responsively (§6.6). Birds are compact SVG/procedural sprites rasterized into sprite atlases at load; ≤7 sprites + a few ornament particles keeps the per-frame draw trivial.

### 6.2 Snapshot → interpolation → frame

The client holds the last snapshot and the previous one and **interpolates**: a bird at perch A in snapshot N and perch B in N+1 renders a smooth flight path between them, never a teleport. Mood and motion_state drive the local micro-motion controller; call_schedule drives audio (§6) and the `calling` motion. A local clock advances continuously between snapshots so motion never stalls waiting for the network.

### 6.3 Idle micro-motion (mood-shaped) `[INV]`

> **`[INV] Birds are never still in a way that reads as paused.** Idle micro-motion (preen, scan, head-tilt, weight-shuffle) runs continuously and is mood-shaped: wary → further back + more scanning; content → preening; curious → tilts toward sounds/leaves; drowsy → low + fluffed. The user reads mood from motion with no label/tooltip/status-icon.**

Implemented as a small procedural animation controller per bird: a state machine of micro-gestures with personality/mood-keyed selection and *continuous low-amplitude noise* (breathing, micro weight shifts) layered on top so there is never a static frame. Enforcement of "no labels": there is no DOM/canvas affordance that prints a mood word anywhere in the scene; mood is motion only.

### 6.4 Transitions & listen-in mix

- Flight, greeting, offer-reaction, and settle are interpolated transitions driven by `active_transitions`.
- **Listen-in** is a *ramped* re-balance (§6 audio): focused bird's mix rises over `[CAL: ~800ms]`, others drop to ambient floor (never silent). Disengage ramps back over the same window. Visual: a soft focus emphasis on the listened-in bird (subtle), no hard UI selection chrome.
- **Settle:** slow evening light ramp over `[CAL: ~4s]`; 5s undo window (any aviary click reverses).

### 6.5 First frame & loading states `[INV]`

> **`[INV] First frame shows motion already in progress — no entry animation, no fade-from-static, no spinner.** The HTML shell inlines the initial snapshot (§9.2); the client places birds at their current positions/motions and starts rendering as if it had been rendering all along.**

- **Slow/cold load:** a **quiet field** (soft sky color, one or two faint motion cues) — *never a spinner* `[INV]`. The field reads as "the aviary catching up," not "the app loading." Enforcement: no spinner component exists in the codebase for the aviary surface; a review rule forbids one.
- **Empty-aviary** (post-adoption, pre-first-bird): same quiet field, then the first bird enters with a soft fly-in to its perch; thereafter the user never sees an empty aviary.

### 6.6 Responsive scene `[INV] never crop a bird`

Aspect-preserving layout; on narrow phone viewports the scene compresses horizontally (perches move closer) without cropping any bird out of frame; on wide desktop, perches spread. A layout test asserts all birds remain within the safe frame at min and max supported viewport widths.

### 6.7 Reduced-motion rendering (designed, not stripped) — see §8.3

Reduced-motion is a *second renderer*, not a flag that disables the first. Detailed in accessibility (§8.3) because it is an accessibility surface, but it lives in the render pipeline: micro-motion → slow pose cross-fades; flight → perch-to-perch cross-fade; leaf drift removed; day/evening color shift retained but slowed.

### 6.8 Ambient ornaments

Leaf/feather drift is **pure client-side rendering** at idle cadence — *not* simulation state (no per-leaf server record). Generated locally so the scene keeps its own ambient motion between bird actions. Bounded pool, reused (no per-leaf allocation churn — feeds the no-memory-growth budget, §9.4).

### 6.9 Tab-hidden behavior

When hidden/backgrounded, the client **stops rendering** (saves battery; nothing to see) but the simulation continues server-side. On return → `visibilitychange` → pull fresh snapshot → resume into the aviary-that-kept-running (and likely a return-greeting). The client never "pauses and resumes" a frozen scene.

---

## 7. Audio pipeline

### 7.1 Procedural synthesis (no recorded audio, ever) `[INV]`

> **`[INV] Calls are synthesized client-side via WebAudio from the motif library; no recorded audio ships in any path, including fallback.** Looped audio is the audible signature of dead software; the chorus mechanic requires real-time per-call variation that stacked loops cannot provide.**

Each species has a synthesis recipe: oscillator/wavetable + envelope + filter parameters realizing its motifs. A call is rendered from `{motif_seed, pitch_bias, timing_bias}` with per-call randomization so no two calls are byte-identical. Enforcement: no `.mp3/.wav/.ogg` call assets in the bundle; a CI check fails the build if audio binaries appear under the call-asset path.

### 7.2 Chorus mixing

Multiple birds' calls mix as independent synthesized voices through a shared WebAudio graph — a real chorus, no phase-cancel artifact. A bounded voice pool caps simultaneous voices; at the 7-bird cap the mix stays legible (which is *why* 7 is the cap).

### 7.3 Buffer reuse / no memory growth `[INV]`

Audio buffers and nodes are pooled and reused; no per-call allocation that isn't freed. AudioContext and worklets are bounded and singleton. This is a CI-tested budget (§9.4), not a guideline.

### 7.4 Listen-in mix decay

Focused bird's gain ramps up; others ramp to an ambient floor (never zero). Slow ramps both directions (§6.4) so it feels like *listening*, not channel-switching.

### 7.5 WebAudio fallback `[INV]`

If WebAudio is unavailable (old browser, context denied, hardware issue): **graceful silence + captions on by default.** No recorded-audio fallback. Silence-with-captions beats canned audio. The aviary otherwise behaves identically.

---

## 8. Accessibility surfaces (first-class, day-one) `[INV]`

> **`[INV] Accessibility surfaces deliver the actual product, not a stripped variant. They ship with v1, not as v1.1.** No personality vector is exposed via ARIA. Narration and captions use the naturalist field-notebook voice. Reduced-motion is a designed rendering. A reviewer treating any of these as "label every state / animations-off" has built the wrong feature.**

### 8.1 Screen-reader narration

A live region carries **running naturalist prose** (not a state list, not "Pip at perch 2"), generated from the same canonical state the visual reads, in field-notebook voice. Cadence: slow, ~1 update / `[CAL: 30–60s]` at idle; user-initiated events (return-greeting, accepted offer, settle) get a priority bump and narrate promptly — but still as observations, not state transitions. A bounded queue prevents overwhelming the SR (high-frequency narration would force the user to silence it). Voice continuity with the notebook is required (same product across surfaces).

### 8.2 Call captions (runtime-generated)

Opt-in captions render short prose descriptions of each call *in its current mood* ("a soft three-note rise"), generated from the **same procedural grammar that produced the call**, so the caption matches what actually played — never a fixed per-call string. Captions fade in/out near the calling bird, naturalist voice. On by default when WebAudio is unavailable.

### 8.3 Reduced-motion mode (designed surface)

Triggered by `prefers-reduced-motion` *or* explicit settings opt-in. A distinct renderer: micro-motion → slow pose cross-fades; flight → perch cross-fades; leaf drift removed; day→evening color shift retained but slowed. Calls still play (or caption per audio settings); birds still drift; mood still changes; notebook still notices. It is "calmer and slower Pocket Aviary," not "broken Pocket Aviary." Built as a parallel render path sharing the same snapshot input, so it can't fall behind the default renderer.

### 8.4 Keyboard navigation

Tab → top-bar items; Tab into scene focuses first bird; arrow keys move focus between birds; Enter → listen-in on focused bird; Esc → exit listen-in; offer affordance opens via top-bar shortcut and is fully keyboard-navigable; settle reachable from top bar. Focus indicator: soft high-contrast outline legible against both bright and dim aviary states (design-system-specified). Focus state maps to the same `listen_in_start/end` events as pointer interaction.

### 8.5 WCAG AA contrast

All user copy (top-bar labels, settings, account/error surfaces, captions, visually-displayed narration) passes WCAG AA minimum (design system pins exact ratios). The scene carries no user copy except the top bar, so contrast applies chiefly to chrome. Automated contrast checks run in CI on the chrome surfaces.

---

## 9. Performance budgets & observability

### 9.1 Bundle ≤2MB gzipped at first paint `[INV]`

- Aggressive code-splitting: account settings, accessibility settings, and the visit-invitation flow are lazy chunks, **not** in the first-paint bundle. First paint carries only the aviary scene renderer + audio engine + snapshot bootstrap.
- Bird assets are procedural/compact SVG; the audio engine is synthesis code (kilobytes) not audio files (the no-recorded-audio rule is partly *forced* by this budget).
- `[INV]` CI gate: bundle-size check fails the build if the first-paint gzipped bundle exceeds 2MB.

### 9.2 Time-to-first-bird <500ms (mid-tier mobile / 4G) `[INV]`

- HTML shell served from CDN edge with the **initial snapshot inlined** (small payload, no extra round trip).
- Render path draws the first bird before non-critical assets resolve; audio and ornaments initialize after first paint.
- `[INV]` Measured by synthetic checks on a reference mid-tier-mobile/4G profile; regression fails the perf gate. Above 500ms the user notices loading; below, they don't.

### 9.3 60fps idle on a 5-year-old laptop (30-min session) `[INV]`

- Canvas2D scene with ≤7 sprites + bounded ornament pool; micro-motion uses cheap transforms; no layout thrash.
- Runtime budget, not just launch: a 30-min synthetic session on the reference laptop must hold 60fps. Frame-time RUM (aggregate) catches field regressions.

### 9.4 No memory growth over 30 minutes `[INV] — real CI test`

- Pooled audio buffers/nodes; pooled ornament particles; notebook entries scrolled out drop their references (virtualized list); bounded worker/audio contexts.
- `[INV]` A CI test runs a 30-min (or accelerated-equivalent) headless session and asserts heap does not grow beyond a tight bound. This is a gate, not a guideline.

### 9.5 Observability (aggregate-only) `[INV] privacy boundary at metric definition`

- Synthetic perf fleet (scheduled automated browsers from several geographies) + aggregate RUM: page-load timings, first-bird-render timings, frame-time histograms, audio-context error counts, simulation-tick latencies, request counts/latencies/error rates, anonymized session-duration histograms (**no per-account dimension**).
- `[INV] No telemetry metric carries per-bird state or per-account interaction history.` Enforced at metric definition: the metrics schema has no account/bird dimension on behavioral metrics; a review rule + lint forbids adding one. The simulation DB is never a telemetry source.
- **Tick latency alarm:** p99 simulation-tick latency > 5s pages on-call (catches degradation before users feel "slow aviary").

---

## 10. Sync model

Restated as the concrete protocol because it is where correctness is easiest to lose.

- **One canonical record per account** (`aviary_state` + per-bird `personality/mood`). Both devices read it. There is **no client-to-client sync, no client-side personality state to merge, no eventual-consistency reconciliation** — because there is nothing on the client to reconcile.
- **Server is the sole writer of personality/mood** (`sim` only; `edge` lacks the grant — §2.2). Clients write only events.
- **`[INV] No last-write-wins on personality.** Drift is applied as additive, server-authored deltas computed from the event log in order. A client never sends "set boldness = 0.62"; it sends "listened in to Pip 182s," and the tick decides the delta. Two devices' overlapping sessions both append events; the tick consumes both in log order and applies both deltas — neither overwrites the other.** Enforcement: there is no absolute-personality write path (schema rejects it; DB grant prevents it); the tick's apply is additive and order-driven; a concurrency test fires interleaved events from two simulated devices and asserts both deltas land (no lost update).
- **Conflict surface** (magic-link replay, mid-write session timeout, outage): matter-of-fact voice only ("we couldn't sign you in…", "your session timed out…", "something went wrong loading your aviary…"). No naturalist phrasing in these.
- **Exactly-once event consumption:** per-account high-water mark; idempotent on client-generated `event_id`; client retries are safe.

---

## 11. Privacy & data boundaries `[INV]`

- **`[INV] Synthetic UUID everywhere; email encrypted on exactly one record.** No table outside `account` stores email or derives a key/shard/partition from it. Logs, telemetry, inter-service messages, shard maps all use `account_id`.** Enforcement: schema lint forbidding `email` columns elsewhere and email-derived keys; review checklist; a log scrubber that would catch raw email in log lines as a backstop.
- **`[INV] Per-bird/per-account interaction data is used only to drive that account's own simulation.** Never aggregated for training, recommendations, population analysis, or any cross-account product. The simulation DB has no reader in the analytics/warehouse path; ML (if it ever exists) never receives per-bird fields.** Enforcement: there is *no* pipeline from sim DB → warehouse (its absence is the enforcement); an architecture test asserts the warehouse has no connection string/credential for the sim DB.
- **Aggregate operational telemetry** is allowed and explicitly scoped (§9.5): health/latency/error/anonymized-duration only.
- **Privacy policy** lives in account settings as plain-text linked copy, naming the aggregate categories and explicitly excluding per-bird interaction state.
- **Account export & deletion** honor "the relationship is the user's": export on demand; soft-delete 30d then hard delete of *every* record tied to the account.

---

## 12. Rollout

### 12.1 Build sequence (dependency-ordered)

1. **Foundations:** schema package (DTOs, event union), Postgres schema + migrations, synthetic-UUID identity, envelope encryption, magic-link auth, session tokens. (Bakes in the privacy invariants from line one.)
2. **Simulation core:** event log, tick loop, drift filter, mood machine, calibration harness (§5.6). Headless; no client yet. Gate on calibration assertions.
3. **Snapshot API + personality-safe DTO:** snapshot read, event write, the no-raw-trait serializer test, no-LWW concurrency test.
4. **Client render core:** Canvas2D scene, snapshot interpolation, mood-shaped idle micro-motion, first-frame "motion in progress," quiet-field load/empty states, responsive layout. Gate on first-bird <500ms and 60fps reference checks.
5. **Audio:** procedural synthesis, chorus mixing, listen-in ramp, buffer pooling + no-memory-growth CI test, WebAudio fallback (silence+captions).
6. **Interactions:** return-greeting, offer (+cooldown + reactions), settle (+undo), presence detector (three-signal), listen-in wiring.
7. **Accessibility (in parallel from step 4, not after):** narration live region, runtime captions, reduced-motion renderer, keyboard nav, AA contrast checks. Ships *with* v1.
8. **Notebook:** sparse aviary-observation generator (aviary-only input contract), read-only viewer (virtualized).
9. **Social:** invite issue/consume/revoke, read-only visit snapshot (no event emission, no host drift), visit log, opt-in notification toggle.
10. **Account management surfaces:** settings, email change, export, soft/hard delete, sessions.
11. **Observability:** synthetic perf fleet, aggregate RUM, tick-latency alarm.

### 12.2 Birds-per-aviary ramp

- v1 launches at **2 starter birds**; the engine and audio mix support the full path to 7 from day one (cap enforced in the engine).
- **Age-gated additions** (not visit-count, not score, not paid): `age_tier` derives from aviary age; `next_bird_available_at` schedules the next offer at relationship-deepening intervals (a few months → 3rd bird; ~a year → 5–6). `[CAL]` the exact age thresholds, tuned conservatively at launch and adjusted from aggregate (anonymized) cohort *operational* signals only — never from per-account behavior.
- New species draw from the same ~6 pool; no rarity, no catalog pick (first encounter is *meeting* a bird, not configuring an avatar).

### 12.3 Instrument from day one

- Perf gates (bundle, first-bird, 60fps, no-memory-growth) in CI before launch.
- Calibration harness as a standing CI job so drift never silently de-tunes.
- Synthetic perf fleet + tick-latency alarm live at launch.
- Aggregate RUM live at launch (privacy-scoped).

### 12.4 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers → matter-of-fact unsupported-browser surface. No legacy compat paths (they'd cost bundle budget).

---

## 13. Risks

### 13.1 Drift calibration (highest engine risk)

*Too fast* → Tamagotchi (numbers move by clicking); *too slow* → screensaver (nothing matters). **Mitigation:** the calibration harness (§5.6) is a standing CI gate asserting the 1-week-instrument / 3-week-visible targets and the "no single session crosses a presentation bucket" rule; `alpha`/weights are named config retuned only by moving those targets. **Residual:** real-population presence distributions may differ from synthetic profiles; monitor *aggregate anonymized* session-duration histograms (allowed) to sanity-check that synthetic profiles span reality, and re-run the harness if they don't.

### 13.2 Sync correctness / lost drift (silent, worst-case)

A lost personality update is invisible — a bird that drifts slightly too slowly, with no error log. **Mitigation:** the no-LWW invariant is structural (no absolute-write path; additive deltas; ordered exactly-once consumption; `edge` lacks the personality grant), backed by an interleaved-two-device concurrency test asserting no lost update. **Residual:** event-log gaps from client-side loss; mitigated by client buffering+retry with idempotent `event_id`, and by drift being dominated by presence pings (frequent, so a dropped one barely matters) rather than rare high-value writes.

### 13.3 Audio uncanniness (spell-breaking)

A call heard twice identically breaks the illusion irrecoverably; stacked loops phase-cancel. **Mitigation:** procedural-only (CI-enforced no audio binaries), per-call randomization, real-voice chorus mixing, 7-bird recognizability cap. **Residual:** synthesis quality is a craft risk — schedule dedicated audio-design iteration with the species motif libraries; validate recognizability with listening tests (does a user tell Pip from Wren by ear after simulated drift?).

### 13.4 Accessibility regressions (rationing the product by sensory ability)

The cheap path (label-every-state, animations-off fallback) silently ships a worse product to SR/reduced-motion users. **Mitigation:** accessibility is built in parallel from step 4 (not after); narration/captions share the canonical state and naturalist voice; reduced-motion is a parallel *designed* renderer sharing snapshot input; CI contrast checks; the `[INV]` forbids ARIA-exposing the personality vector. **Residual:** voice drift between surfaces — guard with shared prose-generation utilities so notebook/narration/captions can't diverge in tone.

### 13.5 "Notice, never announce" erosion (the most reachable mistake)

A well-meaning contributor adds "just a small welcome toast" / "just one streak." **Mitigation:** structural anti-features (§1.4) — no welcome-toast component exists; no visit-frequency datum is materialized; no cross-account aggregation pipeline exists; the notebook generator's input contract excludes user-behavior signals. Re-adding any of these requires *building the missing data/plumbing first* (a visible, reviewable act), not flipping a flag. Plus an explicit review checklist tied to the non-goals.

### 13.6 Presence dishonesty (drift-corrupting)

A laxer presence definition silently inflates population-wide drift. **Mitigation:** three-signal conjunction `[INV]`, client unit-tested to emit no ping if any signal is false; server can only *sum* client-reported windows, never manufacture presence. **Residual:** spoofed clients could over-report presence, but the blast radius is limited to *that account's own* birds (no cross-account effect, by privacy design), so there's no population corruption — the dishonest user simply drifts their own birds; acceptable at v1.

### 13.7 Perf budget regression (kills the central conceit)

Bundle bloat or a slow first-bird re-introduces a visible load state. **Mitigation:** hard CI gates on bundle ≤2MB, first-bird <500ms, 60fps, no-memory-growth; lazy-load non-core surfaces; inlined initial snapshot; no-spinner rule. **Residual:** synthesis/render complexity creep — guard with the standing reference-device synthetic checks.

### 13.8 Day/night & timezone correctness

Storing a mutable "is_night" flag could desync across devices/ticks. **Mitigation:** day/night is *derived* from `local_tz` + server clock at snapshot build, never stored mutable; both devices derive identically.

---

## 14. Summary of invariants (review/CI checklist)

| # | Invariant | Enforcement |
|---|-----------|-------------|
| INV-1 | Server is sole writer of personality/mood | `edge` lacks DB grant; arch test; schema rejects absolute writes |
| INV-2 | No last-write-wins; additive ordered deltas | additive apply; interleaved-device concurrency test |
| INV-3 | Personality never crosses API / shown numerically | DTO has no trait floats; serializer test greps forbidden keys |
| INV-4 | Drift monotonic toward expressive | non-neg clamp; property test (no input lowers a trait) |
| INV-5 | Presence = three-signal conjunction | client emits ping only if all true; unit test |
| INV-6 | Procedural audio only; no recorded path | CI fails on audio binaries in call-asset path |
| INV-7 | First frame mid-motion; no spinner | no spinner component; quiet-field load/empty states |
| INV-8 | Notice-never-announce (no toast/streak/visit-freq) | structural anti-features (§1.4); review checklist |
| INV-9 | Bird id stable forever | single creation path; migration id-stability test |
| INV-10 | Synthetic UUID identity; email in one encrypted place | schema lint; log scrubber backstop |
| INV-11 | No cross-account aggregation of interaction data | pipeline absent; arch test (warehouse has no sim-DB access) |
| INV-12 | Accessibility ships with v1, delivers full product | parallel build track; shared voice utils; contrast CI |
| INV-13 | Bundle ≤2MB / first-bird <500ms / 60fps / no mem growth | four CI perf gates |
| INV-14 | Settle and tab-close engine-equivalent; no neglect penalty | no negative drift path; settle contributes only mood-quiet |
| INV-15 | Visitor attention never drifts host birds | visit token emits no events; arch test |

---

## 15. Open calibration values (tracked, not guessed)

| `[CAL]` | Starting value | Retuned by |
|---------|---------------|------------|
| Tick cadence | ~60s | tick-latency budget + perceived-continuity |
| Drift weights (presence:listen:offer) | ~6:3:1 | calibration harness §5.6 |
| Drift `alpha` (filter slowness) | set for 1wk-instrument/3wk-visible | calibration harness §5.6 |
| Presence ping cadence | ~15s | calibration harness |
| Activity window (pointer/key) | ~3min (leaning long) | calibration harness |
| Offer cooldown | ~4min | curiosity-saturation check |
| Listen-in mix ramp | ~800ms | feel test (listening vs switching) |
| Settle ramp / undo | ~4s ramp, 5s undo | PRD-fixed undo (5s); ramp by feel |
| Narration cadence | 30–60s idle | SR-queue-overwhelm guard |
| Magic-link rate limit | ~5/email/hour | abuse vs friction |
| Age-tier thresholds | months→3rd, ~year→5–6 | conservative launch, aggregate cohort ops signals |

This plan is executable as written: a team can build the schema, the tick, the API, the renderer, the audio engine, the accessibility surfaces, and the social feature against the invariants and `[CAL]` table above, gate them in CI, and ship a v1 that honors every line the PRD drew — most importantly the four that make the aviary feel alive rather than perform aliveness.
