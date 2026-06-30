# Pocket Aviary — V1 Implementation Plan

Status: phase-1 plan, ready for team execution. This document interprets `prd/` into an executable engineering plan. It does not restate the spec; where the spec leaves a decision open, this plan makes a defensible call and names it as such.

The organizing principle for every section below: **the server computes meaning, the client computes appearance.** Personality, mood, perch assignment, weather, and settled-state are server-authoritative facts. Pixel position, pose, audio, and narration text are client-rendered consequences of those facts, recomputed continuously between server updates so nothing the user sees ever looks like it's waiting on a network round trip.

---

## 1. Scope

### In v1 (from `product_brief.md` scope statement, made concrete)

- Single-user accounts, magic-link auth, one canonical aviary per account, multi-device sync via server-authoritative state.
- Two starter birds at adoption, age-gated growth to a 7-bird cap, drawn from a ~6-species pool.
- Personality vector (5 traits) per bird, monotonic-toward-expressive drift, slow-timescale.
- Mood (fast-timescale, daily-ish reset, event/time/weather/personality modulated).
- Procedural call synthesis (WebAudio), chorus mixing, listen-in mix re-balance.
- Return-greeting, offer (seed / song fragment / still pool), settle (with 5s undo).
- Field notebook (sparse, templated naturalist prose, read-only, infinite scroll-back).
- Day/night cycle (client-local-time anchored), ambient weather, ambient leaf/feather ornaments.
- Visit invitations (read-only, revocable, opt-in, off by default, no notifications by default).
- Screen-reader narration, reduced-motion mode (full re-render, not a stripped fallback), call captioning, WCAG AA contrast, full keyboard navigation.
- Performance budgets: ≤2MB initial JS (gzipped), <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle motion on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export, soft-delete with 30-day recovery window then hard delete.

### Explicitly not in v1 (from `non_goals.md`)

Native apps, any gamification surface (achievements/streaks/levels/scores/badges/visit calendars), Tamagotchi mechanics (no death, no hunger, no decaying happiness meter, no visible distress), social-network surfaces beyond the single visit affordance (no profiles, follows, public feed, discovery, leaderboards). These are treated as architectural absences, not feature flags — see §12 for why no kill-switch is built for "turn gamification on."

### Scope calls this plan makes (ambiguity resolved, not deferred)

- **Final mood enum**: `wary | content | curious | drowsy | alert | settled`. The PRD lists five example moods and explicitly defers the final set to implementation; `settled` is added as a sixth because the layout and interactions files both describe a distinct settled visual/behavioral state (eyes closed, low on perch, reachable via the settle gesture and via deep night) that doesn't cleanly collapse into `drowsy`.
- **Bird age-gated unlock schedule** (`bird_engine.md` only specifies the qualitative shape — "a few months" for bird three, "a year" for five-to-six): v1 ships with placeholder thresholds of 3rd bird at 8 weeks, 4th at 16 weeks, 5th at 26 weeks, 6th at 40 weeks, 7th at 52 weeks of aviary age, stored as a single config table so they can be retuned without a deploy. Flagged in §11 as needing a calibration pass.
- **Visit token lifetime**: "one-time link" in `social_optional.md` is read as *uniquely generated per invite*, not single-use-then-dead — the visit log's duration tracking and "revoke an active invite" language imply a session-lived, revocable credential, not a link that dies after first click.

---

## 2. Architecture

### 2.1 Service shape

A modular monolith for the request-path backend, plus an independently-scaled tick worker fleet, plus an edge layer. Not microservices — the team is building one product with tightly coupled invariants (personality is server-only-written, presence must be honest, email must never leak as an identifier); a modular monolith keeps those invariants enforceable by code review and a handful of internal module boundaries rather than by inter-service contracts that can drift.

**Edge layer**
- CDN + edge compute (e.g. Cloudflare Workers / Fastly Compute) serves the HTML shell and, for signed-in requests, inlines the account's current snapshot directly into the response — this is what makes the <500ms time-to-first-bird budget reachable, because the first bird-bearing paint doesn't wait on an origin round trip.
- Static asset CDN serves the JS bundle, SVG rig parts, and motif-library JSON.

**Core backend (modular monolith, stateless, horizontally scaled behind a load balancer)**
- `auth` module — magic link issuance/consumption, session tokens, device revocation.
- `accounts` module — account CRUD, settings, export, soft/hard delete, encrypted email storage.
- `events` module — the single ingest endpoint for client interaction events (append-only).
- `aviary-read` module — snapshot serving (fallback path when edge cache is cold/stale), notebook pagination.
- `visits` module — invite issuance, revocation, visitor-facing read-only snapshot endpoint, visit log. Deliberately isolated from `aviary-read` and from any aggregate-stats code path (see §12 risk on social scope creep).
- `adoption` module — starter-bird selection, renaming, age-gated new-bird offers.

**Tick worker fleet (separate deployable, same codebase, different entrypoint)**
- Runs the simulation tick (§5) and the notebook-generation pass.
- Scaled independently from the request-path servers because its load profile (steady background compute, one pass per account per ~60s) is fundamentally different from request traffic (bursty, diurnal, latency-sensitive). Conflating the two would mean either over-provisioning request servers to cover tick load or risking tick latency spikes during traffic peaks — the PRD's tick-latency p99 alarm (§11) only makes sense if tick capacity is provisioned and scaled on its own terms.

**Datastores**
- Primary Postgres: accounts, sessions, birds (incl. personality vector + mood), notebook entries, visit invites/log, species reference table, age-unlock config.
- Event log: a Postgres table (`interaction_events`), append-only, indexed on `(account_id, sequence)`, with `sequence` assigned by a per-account monotonic counter at insert. Chosen over a dedicated log system (Kafka/Kinesis) for v1: it gets transactional consistency with the tick's read-and-advance-cursor step for free, and v1's expected event volume (a handful of events per session, presence pings at low frequency) doesn't approach the scale where Postgres write throughput becomes the bottleneck. Documented as a deliberate v1-scale call with a named migration path (partition by account, or move to a dedicated log) if event volume outgrows it — flagged once, here, so it doesn't need re-litigating later, mirroring the PRD's own treatment of the synthetic-UUID rule as "easy to honor at design time, impossible to retrofit."
- Edge snapshot cache: a fast KV (Cloudflare KV / Redis at the edge) keyed by account UUID, written by the tick worker on every tick completion, read by the edge layer for first-paint and by clients for subsequent polls when fresh. Falls through to `aviary-read` on a cache miss or staleness beyond the tick cadence.

**Third-party integrations**
- Transactional email provider (magic links, visit invites, export delivery), abstracted behind an internal `EmailSender` interface so the provider is swappable without touching call sites.

### 2.2 Client/server split

| Concern | Owner | Why |
|---|---|---|
| Personality vector values | Server (tick only writes) | No-last-write-wins correctness (§6); never serialized to any client payload in raw form (§4.2). |
| Mood state | Server (tick computes; client renders) | Drives idle-motion and audio params; must be consistent across devices. |
| Perch zone assignment | Server (derived from mood+personality each tick) | A user-visible signal the user reads, never controls (`aviary_layout.md`). |
| Pixel position / pose | Client (tweened from perch zone, procedurally animated) | Continuous 60fps motion can't be tick-cadence-limited (~60s) without looking frozen. |
| Call audio synthesis | Client (WebAudio, from server-provided call-timing params) | Bundle-budget and chorus-quality both require client-side synthesis (`accessibility_perf.md`). |
| Ambient leaf/feather ornaments | Client only, no server state | PRD is explicit: "not driven by the simulation tick (no per-leaf state)." |
| Narration / caption prose | Client, generated from the same structured facts as the notebook (§9.2) | Avoids a server round trip on every narration cadence tick; keeps voice consistent by sharing the fact-extraction module. |
| Notebook entries | Server (tick-adjacent pass writes; client only reads) | Needs cross-session history and sparsity control that only the server can see. |
| Presence detection | Client detects the 3-way conjunction; server only accumulates reported intervals | Client owns the DOM signals (`visibilityState`, focus, pointer/key activity); server is the trust boundary for what counts toward drift. |

### 2.3 Render pipeline boundary

The boundary is the snapshot. Everything left of it (simulation tick, event log, notebook generation) is server-owned and produces a versioned snapshot document. Everything right of it (scene renderer, call engine, narration generator) is a pure function of `(latest snapshot, wall-clock time, per-entity render seed)`. This means the renderer never needs to know *why* a bird is in `wary` mood — it only needs `mood`, `perchZone`, and the bird's stable id/seed to produce continuous motion and sound. This separation is what makes reduced-motion mode a second rendering strategy over the same data rather than a second data path (§8.4).

---

## 2.4 Aliveness, applied as an architecture rule

Two PRD principles (`product_brief.md`) translate directly into non-negotiable implementation rules, stated here once so every later section can cite them instead of re-arguing them:

1. **No entry animation, ever.** The renderer phase-aligns every procedural function (pose curves, call scheduling) to the server-provided tick timestamp at snapshot load, not to `performance.now()` at page load (§8.3). A bird's preen cycle resumes mid-cycle on first frame, exactly as if it had been rendering the whole time.
2. **No announcement surface, ever.** No toast, banner, or modal is permitted anywhere in the client codebase for session-start, return, or milestone events. This is enforced procedurally, not just by convention: the PR template and a lightweight lint rule (§12) flag any new top-level notification/toast component.

---

## 3. Data model

All entities use server-generated UUIDs as primary keys. Timestamps are UTC; client-local-time is only used for day/night rendering and is never persisted as a source of truth beyond a last-known-timezone hint.

```
Account
  id                  UUID (synthetic, the ONLY cross-service identifier)
  email_encrypted      bytes        -- stored once, at rest, never used as a key/identifier anywhere else
  email_verified_at    timestamp?
  pending_email_encrypted bytes?    -- set during email-change verification flow
  created_at            timestamp
  deleted_at            timestamp?  -- soft delete marker
  hard_delete_at        timestamp?  -- deleted_at + 30d, processed by a daily purge job
  timezone_hint         string?     -- last-known IANA tz from a client, day/night render hint only
  settings              jsonb       -- { reducedMotionOptIn, captionsEnabled, audioEnabled,
                                        narrationEnabled, visitNotificationsEnabled }

SessionToken
  id, account_id, device_label, created_at, last_used_at, revoked_at?

Aviary                              -- 1:1 with Account, kept distinct for clean ownership of aviary-wide state
  account_id (PK, FK)
  created_at             timestamp  -- aviary age anchor for bird-unlock gating
  settled                bool
  settled_at             timestamp?
  active_weather         enum?(rain|wind)
  weather_started_at     timestamp?
  weather_duration_s     int?
  bird_count             int        -- denormalized, kept in sync with Bird rows for cheap cap checks

Bird
  id                    UUID (stable identity, never reassigned, never reused)
  account_id            FK
  species_id            FK -> Species
  name                  string
  created_at             timestamp  -- adoption date
  personality_vector    jsonb       -- { boldness, socialWarmth, vocalFrequency, plumageSaturation,
                                        curiosity }, each float in [0,1]
  mood                  enum        -- wary|content|curious|drowsy|alert|settled
  mood_set_at            timestamp
  mood_min_dwell_until   timestamp  -- hysteresis floor, see §5.3
  perch_zone             enum       -- front|middle|back
  last_processed_sequence bigint    -- per-bird-owning-account event-log cursor advanced by the tick
  render_seed            int        -- stable per-bird seed for client-side procedural variation

Species (static reference data, ~6 rows, seeded at deploy)
  id, name, silhouette_asset_ref, default_plumage_palette,
  call_motif_library_ref, personality_seed_ranges jsonb

InteractionEvent (append-only)
  id, account_id, bird_id?, sequence (per-account monotonic, assigned at insert),
  type            enum(presence_ping|listen_in_start|listen_in_end|offer|settle|settle_undo)
  payload         jsonb          -- e.g. { offerType, durationMs }
  client_event_id UUID           -- client-generated idempotency key
  client_occurred_at timestamp   -- client-reported, ordering/UX hint only
  received_at      timestamp     -- server clock, AUTHORITATIVE for drift math (see §6.3 on clock skew)
  processed_by_tick_id UUID?

NotebookEntry
  id, account_id, text, created_at, related_bird_ids UUID[] -- for generation-side dedup only, never exposed

VisitInvite
  id, host_account_id, visitor_email_encrypted, token (opaque, indexed),
  status enum(pending|active|revoked|expired), created_at, expires_at, first_used_at?

VisitLogEntry
  id, host_account_id, visitor_email_encrypted, visited_at, duration_approx_s
```

### 3.1 Why personality is its own jsonb blob and not five columns

Five columns would work too, and either is fine relationally; jsonb is chosen because the tick is the only writer and always rewrites the whole vector as one unit (no partial-column update path exists), and it keeps the species-pool trait-range config and the bird's live vector in the same shape for diffing during calibration testing.

---

## 4. API surface

A deliberately small, mostly-generic surface. The interaction-event endpoint in particular is one endpoint for five event types, not five endpoints — this matches the PRD's "additive server-authored deltas" rule: clients report *what happened*, never *what the new state should be*, so there's no temptation to grow a per-event-type endpoint that accepts a partial state mutation.

### 4.1 Endpoints

```
Auth
  POST /auth/magic-link                {email}
  POST /auth/magic-link/consume        {token} -> session cookie
  GET  /auth/sessions
  POST /auth/sessions/{id}/revoke

Aviary state
  GET  /aviary/snapshot                -> AviarySnapshot (see 4.2). Edge-cached, short TTL.
  POST /aviary/adopt                   {names: [string, string]}   -- new-account-only, idempotent
  POST /birds/{id}/rename               {name}
  GET  /aviary/new-bird-offer           -> {eligible: bool, speciesPreview?}
  POST /aviary/new-bird-offer/accept    {name}

Interaction events (the one generic write path)
  POST /events                          {type, birdId?, payload, clientEventId, clientOccurredAt}
                                         -> 202 Accepted. Idempotent on clientEventId.

Notebook
  GET  /notebook?cursor=...             -> paginated entries, reverse-chronological

Account
  GET/PATCH /account/settings
  POST /account/email/change             {newEmail}
  POST /account/email/verify             {token}
  POST /account/export                   -> async, emailed link
  POST /account/delete
  POST /account/delete/cancel

Visits
  POST /visits/invite                    {email}
  GET  /visits                           -> outstanding + log
  POST /visits/{id}/revoke
  GET  /visit/{token}/snapshot           -> visitor-scoped read-only snapshot, separate auth path
```

### 4.2 `AviarySnapshot` — the one payload shape that matters

This is the contract the renderer, the audio engine, and the narration generator all consume. Critically, it contains **derived rendering parameters, not raw personality values** — the "personality vector is never exposed numerically" rule (`bird_engine.md`) is enforced at the API-serialization boundary, not just in the UI layer, so it can never leak via a network-tab inspection even though the client never displays it.

```
AviarySnapshot {
  tickTimestamp: ISO8601           // server tick time this snapshot reflects; client phase-aligns to this
  dayNightHint: derived client-side from timezone_hint, NOT sent — see 8.3
  settled: bool
  activeWeather: { type, startedAt, durationS } | null
  birds: [
    {
      id, name, speciesId,
      mood: enum,
      perchZone: front|middle|back,
      renderSeed: int,
      callTimingParams: {            // derived from vocalFrequency + mood, NOT the raw trait
        baseIntervalMs, jitterMs, energyLevel: 0..1
      },
      idleMotionParams: {            // derived from full personality + mood
        amplitudeScale, frequencyScale, postureBias
      }
    }
  ]
}
```

`callTimingParams` and `idleMotionParams` are computed server-side from the full personality vector and current mood, then thrown away as a vector — the client receives only the behavioral consequences. This also means a future "show me my bird's stats" feature would require a deliberate new server-side decision to compute and serialize raw values, not just a client-side UI change — exactly the friction the PRD wants here.

---

## 5. Simulation engine design

### 5.1 Tick scheduling

Each account row carries an implicit `next_tick_at` (modeled as an index on a small `tick_schedule` table rather than a column on `Aviary`, so the scheduler can use `SELECT ... FOR UPDATE SKIP LOCKED` to claim due accounts without contending with the rest of the account row). Tick workers poll for due accounts, claim a batch, process each account independently, and set `next_tick_at = now + 60s ± jitter` (jitter spreads load and avoids thundering-herd ticking of accounts that all signed up at the same moment). This is a lease/claim model, not a fixed cron-per-account job, specifically so the worker fleet scales horizontally by adding workers rather than by re-partitioning a static assignment.

### 5.2 Per-account tick steps

1. Lease the account (skip if already leased — guards against a slow previous tick still running).
2. Read `InteractionEvent` rows with `sequence > last_processed_sequence` for this account.
3. Compute presence-time accrued since the last tick from `presence_ping` events, using `received_at` (server clock) for interval math — see §6.3 on why not `client_occurred_at`.
4. For each bird, compute a drift pressure per trait (§5.4) from this tick's events, apply it, persist.
5. For each bird, compute mood transition (§5.3).
6. Run a second pass for bird-to-bird propagation (§5.5).
7. Recompute perch zone from updated mood + personality.
8. Roll weather: small per-tick probability of starting a new weather event if none active, bounded duration (a few minutes), targeting the PRD's "a few times a week" cadence.
9. Persist all updated `Bird` and `Aviary` rows in one transaction, advance `last_processed_sequence`, mark consumed events `processed_by_tick_id`.
10. Publish the new `AviarySnapshot` to the edge KV.
11. Run the notebook-noteworthiness check (§9.1) — can run every tick cheaply (it's a read of recent state diffs) even though it writes rarely.
12. Release the lease, set `next_tick_at`.

Steps 9–10 happening in the same transaction-adjacent pass as the rest of the tick is what makes "no client mutates personality directly under any code path" actually true end-to-end — there is exactly one code path that writes a `Bird.personality_vector`, and it's this one.

### 5.3 Mood transition

A scored-candidate model with hysteresis, not a hand-written if/else state machine — this keeps the four PRD-named inputs (recent interaction, time of day, ambient weather, personality) composable and independently tunable.

```
score(candidateMood) =
    basePrior[candidateMood]
  + timeOfDayWeight(candidateMood, localHour)
  + recentInteractionWeight(candidateMood, lastInteractionType)
  + weatherWeight(candidateMood, activeWeather)
  + personalityBias(candidateMood, personalityVector)   // e.g. high boldness suppresses 'wary'
  + (candidateMood == currentMood ? hysteresisBonus : 0)

nextMood = argmax(score) over all candidates,
  subject to: now >= mood_min_dwell_until, UNLESS a high-priority override fires
  (alarm-call propagation, §5.5, which can force 'wary' even mid-dwell)
```

`mood_min_dwell_until` is set to roughly 20–40 minutes of simulated time past `mood_set_at` on every transition, preventing the mood from flapping tick-to-tick on noisy inputs while still allowing same-session shifts (an offer accepted mid-session can still nudge toward `content` once the dwell floor passes, satisfying "recent interactions in the current session" without contradicting "mood resets on a daily-ish cadence").

### 5.4 Personality drift

Per trait `T` in `{boldness, socialWarmth, vocalFrequency, plumageSaturation, curiosity}`:

```
pressure[T] = sum of weighted, normalized signal contributions this tick:
  - presence-time (dominant; small uniform lift across all five traits per bird present in the aviary,
    since the PRD describes presence as driving birds "toward expressive" generally)
  - listen-in duration on THIS bird -> socialWarmth, vocalFrequency
  - offer accepted on THIS bird -> curiosity
  - any offer made while THIS bird is present -> small boldness lift
  - settle -> no directional pressure (only closes the presence window cleanly)

delta[T]  = max(0, alpha[T] * pressure[T])      // monotonicity enforced HERE, at application,
                                                  // not just in how pressure is weighted upstream —
                                                  // a bug in pressure computation can never produce
                                                  // negative drift
trait_new = clamp(trait_old + delta[T], 0, 1)    // saturating; deceleration near the ceiling is implicit
```

`alpha[T]` (per-trait pace constants) plus a global pace multiplier are the calibration surface. They are not hand-guessed in this plan — §11 specifies an offline calibration testbed that replays synthetic usage traces (e.g. "5 sessions/week, 20 min each" vs "1 session/week, 5 min") and tunes `alpha` until the PRD's named targets hold: instrument-detectable movement after ~1 week of regular use, user-noticeable movement after ~3 weeks, and — critically — no single session producing a visible jump. This testbed is a build deliverable, not a deploy-time guess.

### 5.5 Bird-to-bird interaction

A second pass after individual mood scoring: for each bird that transitioned into `wary` or `alert` this tick, neighboring birds (front/middle perch adjacency) get a small `weatherWeight`-equivalent bonus toward `wary` on their *next* scoring pass (not retroactively this tick, to keep the propagation causally one-directional and avoid oscillation). Chorus eligibility is computed separately and only affects client-side call scheduling hints (`energyLevel` in `callTimingParams`), not server state: if two or more birds with high `vocalFrequency` are both in `content`/`curious`/`alert` mood after this tick, their `callTimingParams.energyLevel` gets a small synchronized-window bump so the client's independent per-bird schedulers are more likely to produce an audible overlap — the actual chorus timing/variation still happens client-side per §7.

### 5.6 Day/night and weather

Day/night is **not simulated or persisted** — it's a pure function of `now()` and the account's `timezone_hint`, computed identically by the tick (for mood's time-of-day input) and the client (for rendering), so there's no risk of the two disagreeing. Weather *is* persisted (`Aviary.active_weather`) because it must be identical across a user's devices at any given moment — a phone and a laptop must show the same passing rain.

---

## 6. Sync model

### 6.1 Why sync is a property, not a feature

Because the tick is the only writer of personality and mood, and every client only ever reads snapshots, there is no client-to-client sync to design — both devices are reading the same record. The work in this section is entirely about making sure that single-writer property actually holds under concurrent client writes to the *event log*, which is the only thing clients write to.

### 6.2 No last-write-wins

Clients never submit absolute state. `POST /events` accepts only event facts (`listen_in_start`, `offer`, etc.); the tick is the sole translator from "what happened" to "what changed." This makes the morning-laptop / lunch-phone race condition described in `accounts_sync.md` structurally unreachable: both sessions' events land in the same append-only log in `received_at` order, and the next tick processes both, in order, as additive deltas. There is no "version" of the personality vector for two writes to conflict over, because there's only one writer.

### 6.3 Clock skew handling

`InteractionEvent.received_at` (server clock, set at insert) is authoritative for all drift-affecting interval math — presence-time accrual, listen-in duration. `client_occurred_at` is stored for debugging/ordering display only. This closes a gaming/skew vector: a client with a fast or slow clock cannot inflate or deflate its own drift contribution by misreporting timestamps, since the server never trusts client time for anything that feeds the drift function.

### 6.4 Idempotency

Every event carries a client-generated `clientEventId` (UUID). The `events` module upserts on `(account_id, client_event_id)` — a retried POST (flaky network, client reconnect logic) is a no-op on the second arrival rather than double-counting, which matters most for `presence_ping` given its periodic-retry-prone nature.

### 6.5 Snapshot delivery and freshness

Clients pull a fresh snapshot: on visibility change (tab foregrounded), after a detected long render-frame gap (laptop resume from sleep), and on a low-frequency keepalive (~60–90s) while visible — no WebSocket, no server push, matching the PRD's explicit pull-based model and keeping the architecture in the "boring with teeth" register the accounts/sync file sets. The edge KV write on every tick means a poll typically resolves from edge cache, not origin.

### 6.6 Reconciling slow-tick state with felt-immediate interactions

This is the one place the sync model needs an explicit design decision the PRD doesn't spell out mechanically, because two of its own requirements are in tension: personality/mood are server-tick-authoritative on a ~60s cadence, but an offer's reaction ("a curious, content bird approaches a seed") is described as something the user watches happen, not something that resolves a minute later.

**Resolution: separate the *behavioral reaction* from the *state mutation*.** When the user submits an offer, the client immediately renders a reaction using a deterministic, client-side reaction-selection function over the bird's *currently known* mood/personality-derived params (already present in the last snapshot) and the offer type — this is presentation, fully reproducible, and commits no state. The event is also POSTed to the event log, and its actual drift/mood effect is applied by the next tick as usual. The user sees an immediate, specific reaction; the durable consequence of that gesture lands on the normal slow cadence. This is the same pattern already implied by listen-in (the audio mix change is instant and purely client-side; the *drift* from sustained listen-in is logged and applied later) — this plan generalizes it explicitly so the team doesn't reinvent it per-interaction.

---

## 7. Audio pipeline

### 7.1 Call grammar runtime

Each species ships a **motif library**: data, not code — a small JSON/JS description per species of parametrized motif recipes (oscillator type, pitch contour, envelope shape, duration range, optional filter sweep). Keeping motifs as data is what lets six species stay inside the bundle budget and lets sound design iterate without an engineering release.

Per-call synthesis pipeline:

```
Bird's render-time inputs: species motif library, callTimingParams (energyLevel, baseIntervalMs,
                            jitterMs from the snapshot), current mood, render_seed
  -> CallScheduler rolls whether/when to call next, using a Poisson-ish interval centered on
     baseIntervalMs and scaled by mood (drowsy/settled lengthens intervals; alert shortens them)
  -> CallSpecBuilder picks a motif from the library, applies per-call jitter (seeded RNG, so output
     is reproducible for testing but never identical twice in practice) -> a CallSpec: an ordered
     list of {oscillatorType, freqContour, durationMs, gainEnvelope, filterSweep?}
  -> WebAudio graph builder schedules OscillatorNode(s) + GainNode envelope + optional
     BiquadFilterNode per CallSpec segment, routed into the bird's per-bird bus
```

### 7.2 Mixing and listen-in

Each bird has a dedicated `GainNode` feeding a shared master bus (which carries a limiter/compressor to absorb chorus-event peaks). Two named gain targets exist per bird: `ambientLevel` and `focusedLevel`. Engaging listen-in ramps the focused bird toward `focusedLevel` and all others toward a quieted-but-nonzero ambient floor using `GainNode.gain.linearRampToValueAtTime` over ~500–800ms; disengaging reverses the same ramp. The PRD's "a re-balance, not a mute" rule is enforced by clamping the quieted floor above zero, never letting other birds reach silence.

### 7.3 Chorus mixing

When the scheduler independently rolls overlapping call windows for two-plus birds (more likely when `energyLevel` carries the bird-to-bird synchronized-window bump from §5.5), each call still gets its own seeded jitter — two birds are never literally playing the same CallSpec at the same phase, which is what avoids the phase-cancellation artifact the PRD calls out. Scheduling deliberately staggers exact start times by a small random offset (tens of milliseconds) even within an "overlapping" chorus window, for the same reason the return-greeting staggers multiple greeting birds (§8 — same underlying principle, applied to audio).

### 7.4 Captioning, generated from the same source as audio

The `CallSpec` produced by step 2 of §7.1 is also handed to a small caption-phrase template grammar (motif shape + mood -> phrase, e.g. a rising 3-segment contour in `content` mood -> "a soft three-note rise"). Sharing the `CallSpec` as the single source for both the audio graph and the caption text is what guarantees the caption always matches what was actually played, per the PRD's explicit requirement.

### 7.5 Resource bounds and fallback

A bounded pool of reusable `AudioBuffer`/oscillator node instances (sized to the 7-bird cap) avoids per-call allocation churn, supporting the no-memory-growth budget (§10). If `AudioContext` construction fails or is blocked, the `CallScheduler` still runs (it's pure timing/CallSpec logic) but the graph-builder step is skipped — calls become caption-only events, with captions defaulted on, exactly matching the PRD's "graceful silence with captions on by default" fallback. No recorded-audio fallback path exists anywhere in the codebase, by design.

---

## 8. Frontend rendering pipeline

### 8.1 Rendering technology

A single `<canvas>` with an internal layered draw-list, redrawn fully every frame at 60fps (no dirty-rect tracking — the scene is small and bounded at 7 birds, so full-redraw is simpler and cheap enough, and dirty-rect bookkeeping would itself cost bundle size). No general-purpose game engine dependency (Pixi/Phaser/etc.) — a hand-rolled renderer keeps the engine itself out of the bundle-budget conversation and avoids carrying engine features (physics, tilemaps, particle systems) the product doesn't use.

Birds are rigged from small per-species SVG parts (body/head/wing/tail), rasterized once to an offscreen canvas atlas at load time, then posed at render time via 2D affine transforms (translate/rotate/scale) per part — not frame-by-frame sprite sheet animation. This is what lets idle motion be continuously parametrized (§8.2) instead of a fixed set of canned loops, and it's what lets `plumageSaturation` apply as a runtime HSL adjustment on the rasterized parts rather than requiring pre-baked art per saturation level.

### 8.2 Procedural pose system

A small library of continuous curve generators (sine/noise-based oscillators with eased transitions) produces joint-angle and position offsets for preen, scan, head-tilt, and weight-shuffle behaviors. Each is parametrized by the bird's `idleMotionParams` (amplitude/frequency scale, posture bias) from the snapshot plus its stable `render_seed`, so two birds in the same mood never move in lockstep. This directly implements "idle motion is mood-shaped" — a drowsy bird gets low amplitude, low frequency, and a posture bias toward "low on the perch"; a curious bird gets higher head-tilt frequency — without any per-mood animation clip asset.

### 8.3 Phase-aligned first frame

Every procedural function (pose curves, call scheduling) is evaluated as `f(now - tickTimestamp + serverPhaseOffset, renderSeed)`, anchored to the snapshot's `tickTimestamp`, not to the moment the page loaded. The very first rendered frame therefore evaluates these functions at whatever phase they'd be at had they been running continuously since the tick — there is no "reset to a default pose" tell on load. This is the concrete mechanism behind the PRD's "appears already in motion" requirement; it is a timing-math decision, not just an absence of a loading spinner.

### 8.4 Loading and reduced-motion as rendering-strategy swaps, not fallback codepaths

- **Loading state**: if the inlined/edge snapshot hasn't resolved within ~150–200ms, render the "quiet field" (gradient sky, zero or one ambient ornament, no bird) — under the 500ms target this is rarely perceptible. No spinner anywhere in the codebase.
- **Reduced-motion mode**: reuses the *same* pose-curve outputs and the *same* snapshot data, but samples them at a slow discrete cadence (~1.5–2.5s) and cross-fades between two rasterized poses via canvas alpha blending over ~600–900ms, instead of continuously tweening. Flight/perch-change becomes a straight cross-fade between start/end raster rather than a tweened path. The leaf/feather ornament layer is disabled; day/night color shifts remain but slow down. Implemented as a second rendering strategy consuming the identical data layer — this is what makes it "a different rendering of the same aviary," per the PRD's explicit instruction, rather than a stripped fallback.

### 8.5 Responsive layout

Perch x-positions are percentages of scene width, recomputed on resize/breakpoint change, with a minimum-spacing solver that compresses spacing (never crops) at narrow viewports.

### 8.6 Keyboard-accessible birds without aviary chrome

The PRD requires both "no UI chrome inside the aviary" and full keyboard navigation with arrow-key focus movement between birds. Resolved with invisible (zero-opacity, but accessible-name-bearing) DOM button elements absolutely positioned to match each rendered bird's current coordinates, recomputed every layout pass (resize, perch-zone change). These carry no visual chrome but are real focusable, accessible-name-bearing elements for assistive tech and keyboard users — visually the canvas alone is the surface; functionally there's a synced accessibility layer behind it.

### 8.7 Bundle composition and code-splitting

Main entry chunk: scene renderer, bird rig/pose system, call-engine core, snapshot fetch/poll logic, top-bar shell. Lazy-loaded route chunks: account settings, accessibility settings, visit-invitation flow, export/delete flows, the field notebook view. Internal target: ~800KB–1.2MB gzipped for the critical-path chunk, leaving headroom under the 2MB cap for species/motif data and the accessibility layer, enforced by CI (§10).

---

## 9. Naturalist text generation (notebook, narration, captions)

### 9.1 Shared structured-fact extraction

A single internal module, `StateFactExtractor`, consumes a snapshot (and, for the notebook, recent snapshot history) and produces structured observations: `{bird, action, perchZone, mood, intensity, comparativeNote?}` — e.g. "Pip greeted before Wren today, first time this week" starts life as a structured fact (`firstGreeterToday: Pip, novel: true`), not as freeform text. Two different template grammars consume the same fact stream:

- **Notebook grammar**: sparse, retrospective, comparative phrasing. Selection is gated by a noteworthiness score plus a minimum-gap-since-last-entry (~36–72h, configurable) to hit the PRD's "roughly one entry every few days" target even for highly active users.
- **Narration grammar**: frequent (30–60s idle cadence, faster on user-initiated events), present-tense, single-moment phrasing, with no comparative/historical framing.

Sharing the extractor is the architectural answer to the PRD's explicit worry that a screen-reader user moving between the live aviary and the notebook should "hear the same product, not two products with different personalities glued together" — voice consistency is enforced by sharing the input, not by independently hand-tuning two prose generators to sound similar.

### 9.2 Why templates, not a generative model

No LLM dependency is introduced for notebook/narration/caption text. Given the PRD's hard requirements — voice must never drift, content must never hallucinate a fact the engine didn't actually compute, entries must stay rare and specifically calibrated, and generation must run client-side for narration's low-latency cadence — a deterministic template/slot-filling grammar (structurally the same kind of system as the call grammar in §7) is the defensible choice: it's free of hallucination risk, trivially testable (assert a given `StateFact` always produces voice-correct, factually-accurate prose), and has no inference cost or external dependency. The tradeoff, named here rather than discovered later, is that template variety requires real ongoing writing investment to avoid feeling formulaic — flagged again as a risk in §13.

---

## 10. Accessibility surfaces

- **Screen-reader narration**: client-side, generated per §9.1, updated on the 30–60s idle cadence with priority bumps for user-initiated events (return-greeting, offer reaction, settle). Implemented via a visually-hidden `aria-live="polite"` region (the priority-bumped events may warrant `assertive` — to be validated with real AT during the launch-gating pass, §11) that's *replaced*, not appended, each update, so the AT queue doesn't accumulate stale narration.
- **Reduced-motion mode**: see §8.4. Triggered by `prefers-reduced-motion` media query by default, with an explicit override in accessibility settings.
- **Captioning**: see §7.4. Off by default for sighted/audio-on users, on by default whenever WebAudio is unavailable, toggleable in accessibility settings.
- **Keyboard navigation**: Tab order moves through top-bar items, then into the aviary's invisible accessible focus layer (§8.6); arrow keys move focus spatially (left-to-right by perch x-position) between birds; Enter triggers listen-in on the focused bird; Escape exits listen-in. Offer affordance and settle gesture are reachable from the top bar and are themselves fully keyboard-operable (standard button/menu semantics, no canvas-only interaction required for either).
- **Contrast**: WCAG AA minimum on all chrome text (top bar, settings, errors, captions, visually-displayed narration), enforced via the design system's token contrast checker in CI for any new chrome surface.
- **Build-in, not bolt-on**: reduced-motion mode and captioning ship in the v1 launch gate (§11), not as a post-launch fix — per the PRD's explicit instruction that a v1.1 accessibility patch is itself a launch failure.

---

## 11. Performance budgets and observability

### 11.1 Budgets (from `accessibility_perf.md`, restated as engineering targets)

| Budget | Target | Primary levers |
|---|---|---|
| Initial JS bundle | ≤2MB gzipped (internal target ~800KB–1.2MB critical path) | Code-splitting (§8.7), procedural audio/visuals instead of recorded/baked assets, motif/species data kept lean |
| Time to first bird visible | <500ms, mid-tier mobile, 4G | Edge-inlined snapshot (§2.1), render path that draws the first bird before non-critical assets resolve |
| Idle motion frame rate | 60fps, 5-year-old mid-range laptop, sustained over a 30-min session | Single-canvas full-redraw kept cheap by bounding bird count at 7, pose math kept to simple closed-form curves (no physics sim) |
| Memory growth | None detectable over 30 minutes | Reused AudioBuffer/oscillator pool (§7.5), notebook entries released on scroll-out, bounded worker/AudioContext lifecycle |

### 11.2 CI enforcement

- Bundle-size budget check fails the build on regression past the critical-path target, from the first commit that introduces the bundler config — not retrofitted later.
- A scheduled (nightly) Playwright/Puppeteer soak test runs a simulated 30-minute idle session and asserts JS heap size doesn't trend upward, per the PRD's "a real test in CI, not a guideline" instruction.
- A synthetic-browser fleet (common geographies) runs the time-to-first-bird measurement on a schedule against staging/production.

### 11.3 Telemetry boundary

Aggregate-only RUM: page load timings, first-bird-render timings, client render-frame timings, audio-context error counts, simulation-tick latencies, anonymized session-duration histograms. None of this carries a per-bird or per-account dimension — enforced by defining the metric schema itself without an account-id field for any of these emitters, not by a downstream filtering policy (the PRD treats this as an architectural rule, not a policy one; this plan follows suit, see §13 on the lint-rule mitigation).

Alarm: simulation-tick latency p99 > 5s pages on-call. Tick is expected to run well under that; the alarm is calibrated to catch early degradation (worker fleet under-provisioned, a slow query creeping in) before users perceive "my aviary feels slow to update."

---

## 12. Rollout

### 12.1 Sequencing

1. **Internal dogfood** — team accounts only, full feature set on, used to catch the obvious aliveness failures (canned-feeling motion, audio uncanniness) that automated tests can't.
2. **Calibration pass** (blocking gate) — drift-pace constants (§5.4) tuned against the offline synthetic-usage testbed; audio motif libraries get a dedicated listening-test pass, not just an engineering correctness check; narration/captions tested with real screen readers (NVDA, JAWS, VoiceOver) against real sessions.
3. **Invite-only beta cohort** — small, to get real multi-day drift behavior under observation before wider exposure, since drift calibration risk (§13) is the hardest thing to validate purely offline.
4. **Gradual percentage rollout to GA** — gated on the performance budgets (§11) holding under real traffic and the tick-latency alarm staying quiet.

### 12.2 Launch gate checklist (all required before GA, not staged into v1.1)

- Time-to-first-bird budget verified by the synthetic fleet across target geographies.
- Reduced-motion mode and captioning shipped and validated with real assistive technology.
- Bundle-size CI check green with margin.
- 30-minute memory-soak test green in CI.
- Drift calibration testbed run, targets met (instrument-detectable @ ~1wk, user-visible @ ~3wk, no single-session visible jump).

### 12.3 Bird-count ramp

The 7-bird cap is enforced in the engine from day one (not a soft-launched lower cap raised later) — the audio-recognizability ceiling it protects is a hard constraint, not a rollout lever. What ramps is the *age-gated unlock schedule* (§1, scope calls) for individual accounts, which is a config-table change, not a code change, so it can be retuned post-launch without a deploy.

### 12.4 Instrumentation and kill-switches

RUM/synthetic/tick-latency telemetry (§11.3) is live from day one. Independent kill-switches exist for the call engine (falls back to caption-only mode), the weather system, and the notebook generator (stops writing new entries, doesn't affect anything else) — scoped narrowly enough that disabling one for an incident never disables presence/drift/sync, which are the product's actual spine. No kill-switch exists for "turn on a streak counter" or any gamification surface, deliberately — that's not an incident-response lever, it's a different product, and this plan does not build the door for it to slip in through.

---

## 13. Risks

- **Drift calibration risk.** Too fast reads as Tamagotchi (users can move a number by clicking); too slow reads as a screensaver. Mitigated by the offline calibration testbed (§5.4, §12.2) as a blocking launch gate, plus aggregate-only post-launch instrumentation of drift-curve distributions (e.g. "days to first instrument-detectable change," bucketed across the cohort) — never per-account, per the privacy boundary.
- **Sync/tick correctness risk.** A tick bug double-applying events would silently corrupt drift. Mitigated structurally: `last_processed_sequence` advance and personality writes happen in the same transaction (§5.2 step 9), and `clientEventId` idempotency (§6.4) prevents duplicate event ingestion in the first place.
- **Audio uncanniness risk.** Procedural synthesis can read as "MIDI-ish" rather than alive, undermining the product's central claim. Mitigated by treating the audio motif libraries as a dedicated sound-design deliverable with a listening-test gate (§12.2), not solely an engineering correctness task.
- **Notebook/narration repetition risk.** Template-based generation (§9.2) can feel formulaic if the template library is thin, which is especially damaging here because the PRD names the notebook as the surface where the product's voice is "most concentrated and most visible." Mitigated by sizing the per-fact-type template library generously at launch and adding no-repeat-recently logic; flagged as needing sustained writing investment, not a one-time engineering task.
- **Voice-divergence risk.** If the shared `StateFactExtractor` (§9.1) is ever bypassed and the notebook/narration grammars get independent fact-deriving logic, the two voices will drift apart over time without any test catching it. Mitigated by a CI test asserting both consumers import the same extractor module, not just that their output text "looks similar."
- **Bundle-budget creep risk.** Procedural audio, the rig/pose system, and the accessibility layer all compete for the same 2MB ceiling. Mitigated by enforcing the CI bundle-size budget from the first commit (§11.2) rather than discovering the overage near launch.
- **PII-leakage risk.** The synthetic-UUID rule (`accounts_sync.md`) is easy to violate by accident — an engineer reaching for email as a "convenient" key in a new log line or partition key. Mitigated by a CI lint rule that flags any reference to the `email_encrypted`/email fields outside the `accounts` module boundary, and by documenting the UUID-only rule as a standing architectural contract in the codebase, not just in this plan.
- **Social-feature scope-creep risk.** Pressure to add visit notifications-by-default, a "frequent visitor" surface, or aggregate visit stats is predictable (`social_optional.md` names this directly). Mitigated by keeping the `visits` module structurally isolated (§2.1) from any aggregate-stats code path, so a future feature can't "just" read existing infrastructure to add a leaderboard — it would have to build new infrastructure, which is a much higher bar to clear by accident.
- **Tick scalability risk.** The lease/claim scheduling model (§5.1) and Postgres-backed event log (§2.1) are sized for v1 scale. Mitigated by designing the claim model to scale horizontally by adding workers from day one (even though v1 doesn't need many), since retrofitting ordering guarantees under load later is exactly the kind of "easy at design time, hard to retrofit" risk the PRD itself warns about for the UUID rule.
- **Gamification-creep risk.** The single highest-named risk in the PRD itself (`non_goals.md`: "the rule has to be loud here so that it survives every reasonable-looking pitch to relax it"). This plan's mitigation is architectural, not just cultural: no event type, API field, or data-model column anywhere in this plan tracks visit-frequency, streaks, or counts-of-anything-the-user-did in a form that could be surfaced as a counter. There is no telemetry table to repurpose into a streak feature later — the absence is structural, matching the PRD's own framing that the only durable defense is refusing the first one.

---

*End of plan.*
