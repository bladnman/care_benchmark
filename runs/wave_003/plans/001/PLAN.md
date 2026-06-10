# Pocket Aviary — v1 Implementation Plan

This is an executable engineering plan for Pocket Aviary v1, derived from the PRD set (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). It interprets the spec into architecture, data model, API, engine design, rendering and audio pipelines, accessibility, performance, rollout, and risk. Where the PRD leaves a decision open, the plan makes a defensible call and flags it in §3.

---

## 1. Scope

### 1.1 In scope for v1

- Single-user accounts; magic-link email sign-in; per-device revocable sessions; email change with verification; account export (JSON, emailed link); soft-delete 30 days then hard-delete.
- One canonical aviary per account, advanced by a server-side simulation tick (~1/min), with multi-device read access via snapshot pulls.
- Bird engine: hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift driven primarily by presence-time; fast-timescale mood (wary, content, curious, drowsy, alert, plus settled/sleeping display states) that persists across sessions; bird-to-bird interaction (call/response, mood contagion, emergent chorus).
- Six-species pool with per-species silhouette, palette, and call-motif library; stable bird identity for account lifetime; user naming and renaming; adoption flow with two system-selected starters; age-gated bird offers up to a hard cap of seven.
- Interactions: presence accounting (three-condition conjunction), return-greeting, listen-in, offers (seed / song fragment / still pool) with per-bird cooldown, settle with 5-second undo.
- Field notebook: auto-generated, sparse, read-only, naturalist-voice entries with unlimited scrollback.
- Scene: single horizontal one-screen scene, three perch zones, local-time day/night cycle, rare ambient weather, ambient micro-motion, no in-scene chrome, fading top bar (account/settings, accessibility settings, notebook, offer), motion-already-in-progress first frame, quiet-field loading and empty states, responsive without cropping birds.
- Audio: client-side procedural call synthesis via WebAudio; per-bird recognizable signatures; real chorus mixing; listen-in mix re-balance with slow ramps; graceful-silence fallback with captions on by default.
- Accessibility as designed surfaces: naturalist screen-reader narration on a slow cadence; reduced-motion mode as a cross-fade rendering of the same aviary; runtime-generated call captions; full keyboard navigation; WCAG AA contrast on all user copy.
- Social: per-invite, email-addressed, read-only ambient visits; revocation; 30-day invite expiry; silent visit log; opt-in (default off) visit notification toggle in settings.
- Privacy: synthetic UUID account identifiers everywhere except the encrypted email-on-account-record; per-bird interaction data used only for that user's simulation; aggregate-only operational telemetry with a pipeline-level boundary.
- Performance: <2MB gzipped initial JS, first bird visible <500ms on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop, zero memory growth over a 30-minute session (CI-enforced), synthetic perf fleet + aggregate RUM, tick-latency p99 alarm at 5s.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser surface otherwise.

### 1.2 Explicitly out of scope (enforced, not just omitted)

Native apps; payments; shared/multi-aviary accounts; any gamification surface (streaks, badges, levels, counters, visit calendars, milestone celebrations — none, in any form, including settings-buried ones); Tamagotchi mechanics (death, hunger, distress, decaying meters); social-network surfaces (profiles, follows, feeds, discovery, comments, leaderboards, co-presence, chat, visitor avatars); push/email/ping notifications about the aviary; recorded-audio fallback; user-visible personality numbers in any surface at any tier; user control of perch placement; editable/annotatable notebook.

Several non-goals get *structural* enforcement in this plan, not just absence: no per-account analytics dimensions exist in the telemetry schema (so leaderboard-feeding stats cannot be "just exposed" later), the personality vector never leaves the server in raw named form (§7.6), and notebook generation has no detector class that observes user behavior (§10.4).

---

## 2. Architecture overview

### 2.1 Shape

Three deployables plus a static edge layer:

1. **Web client** — TypeScript SPA. Renders snapshots, interpolates motion, synthesizes audio, generates narration/captions client-side from the same realized state the visuals use, submits interaction events. Owns the *performance layer* (moment-to-moment motion and call timing); never owns canonical state.
2. **API service** — stateless HTTP service (TypeScript/Node). Auth, snapshot reads, event ingestion, notebook reads, visits, account management, export/delete.
3. **Simulation tick service** — worker fleet (same TypeScript codebase, shared engine package) that advances each aviary's canonical state on a ~1-minute cadence: consumes the append-only event log in order, updates personality vectors and moods, schedules weather, emits notebook candidates. The only writer of personality state in the entire system.
4. **Edge/CDN** — serves HTML shell, hashed static assets, and the bootstrap path that makes time-to-first-bird achievable (§13.2).

**Datastore: PostgreSQL** as the single source of truth (accounts, birds, vectors, moods, event log, notebook, invites, sessions). No Kafka, no separate cache tier at v1 — the snapshot is kilobytes and read-mostly; Postgres with sane indexes handles the v1 load envelope, and fewer moving parts means fewer ways to violate the single-writer rule. A Redis layer is a scale-out option later, behind the same repository interface.

**Shared engine package**: the tick function, drift math, mood transition model, call-grammar data structures, and prose grammars live in one pure, deterministic TypeScript package consumed by both server (canonical simulation) and client (interpolation, call realization, captions, narration). Determinism is a hard requirement — see §7.1.

### 2.2 Client/server split — the state/performance boundary

The load-bearing architectural line:

- **Server owns state**: personality vectors, mood, drift, presence-time accounting, weather schedule, perch assignments at tick granularity, notebook, bird offers, accounts, invites.
- **Client owns performance**: which exact frames a preen takes, exact call onset times within server-provided propensity windows, greeting choreography execution, leaf/feather ornaments, mix ramps. The client is an *interpreter* of canonical state, seeded by server-provided random seeds so two devices viewing the same aviary render coherent (not necessarily frame-identical) scenes.

Rules that fall out of this split: clients never write personality or mood directly; clients submit semantic events ("listened in to bird B for 184s"), never deltas or absolute values; everything the user sees moment-to-moment must be derivable from snapshot + deterministic client-side realization, so a refresh never visibly "resets" anything.

### 2.3 Privacy boundary as infrastructure

Two physically separate data paths from day one:

- **Simulation path**: client → API → event log → tick → canonical state. Contains per-bird, per-account interaction data. Never read by analytics.
- **Telemetry path**: client/server → metrics pipeline (counts, latencies, error rates, anonymized histograms). The metrics schema has *no per-account interaction dimensions*; account UUID appears only in operational error logs for debugging ("is this account having errors"), never joined to bird state. The analytics warehouse has no connection, credential, or replica of the simulation database. This is enforced in infra (separate networks/roles), code review checklist, and a CI lint that rejects metric definitions containing bird/personality/mood/notebook fields.

---

## 3. Defensible calls (ambiguity register)

Decisions the PRD leaves open, with the call made and why. Each is reversible behind config or interface unless noted.

| # | Call | Rationale |
|---|------|-----------|
| C1 | **Snapshot transport is HTTPS polling, not WebSockets.** Pull on visibility change, on render-gap detection (suspend recovery), and on a keepalive every ~45s while visible. | The PRD specifies exactly this pull model; the tick is 1/min, so push adds infrastructure for no perceptible freshness gain. Keeps the API stateless. |
| C2 | **Renderer is layered Canvas 2D with procedural skeletal birds; no game engine.** | ≤7 birds + sparse ornaments on one static-composition screen is comfortably 60fps in Canvas 2D on 5-year-old hardware; a WebGL engine spends bundle budget (2MB cap) and risk on capability we don't need. WebGL2 remains an internal renderer-interface swap if profiling demands it. |
| C3 | **Audio cannot start before a user gesture** (browser autoplay policy blocks AudioContext until click/tap/keydown). The aviary starts visually alive immediately; the AudioContext resumes on the first qualifying input and ambient sound fades in over ~2s mid-call, as if the window was opened. Captions (if enabled) run from frame one. No "enable sound" modal, no blocked-audio toast — a small muted-state glyph in the top bar is the only indicator. | "Calls already audible" on first frame is not implementable on the open web. Fading in on first gesture preserves the no-announcement register; a modal would violate it. Flagged as the one place the PRD's letter is physically unachievable; the spirit (no entry ceremony) is preserved. |
| C4 | **Tick cadence is adaptive in execution, fixed in semantics.** The tick function is a pure deterministic fold; aviaries with a connected client or events in the last 24h tick every minute; dormant aviaries are advanced by an exact catch-up fold (identical math, iterated minute steps) on a coarser schedule (hourly) and immediately on snapshot request. Observable state at any time t is bit-identical to having ticked every minute. | The PRD's requirement is continuity ("the aviary that has been running"), which the fold preserves exactly, while a strict 1/min tick for every dormant account forever is pure cost. The invariant is tested: `foldTicks(state, t0→t1)` must equal the minute-by-minute sequence. |
| C5 | **Raw trait values never ship to the client.** Snapshots carry derived *performance parameters* (greeting weight, approach tendency, call-rate band, plumage render params, investigation propensity) computed server-side from traits. | The "never exposed numerically" rule is about product surfaces, but a named `boldness: 0.62` in a network payload is a dev-tools dashboard waiting to happen and invites third-party stat tooling. Derivation server-side makes the rule cheap to keep. |
| C6 | **Account timezone is the IANA zone last reported by a signed-in client**, stored on the account, used by the tick for time-of-day mood inputs. The client renders day/night from device-local time regardless. | The PRD anchors the cycle to "the user's local time"; with two devices in different zones, last-reported wins for canonical mood inputs (rare case, low stakes, self-corrects on next use). |
| C7 | **Presence activity window starts at 4 minutes** (pointermove/keypress within the last 4 min), calibrated in beta, biased long per the PRD ("watching without moving is the actual product"). | The PRD says "a few minutes, lean long." 4 min is the starting value behind a config flag. |
| C8 | **Weather and perch assignments are server-scheduled** (seeded per-aviary on the tick) so all devices and visitors see the same weather and the same bird placements, and mood effects are canonical. Leaves/feathers are client-only ornaments per the PRD. | Multi-device coherence: a rain shower that exists on the laptop but not the phone breaks the "same aviary" promise. |
| C9 | **Call scheduling is client-realized inside server-set propensity bands**, seeded per (aviary, bird, time-bucket) so two devices hear a similar—not sample-identical—soundscape; captions and narration derive from the locally realized call plan so they always match what actually played. | Exact cross-device call sync would require server-clocked audio events at sub-second granularity for no user-perceivable benefit; the canonical layer (vocal-frequency band, mood) is synced, the performance layer is local. |
| C10 | **Mood model is a per-bird semi-Markov state machine** (enumerated states, dwell-time distributions, transition weights modulated by personality, time-of-day, weather, recent interactions) with hysteresis to prevent flapping. | The PRD enumerates the inputs but not the formalism; semi-Markov with dwell times gives persistent, non-twitchy moods that survive sessions naturally. |
| C11 | **Bird offer schedule (aviary age gates)**: 3rd bird at day 60, 4th at day 150, 5th at day 270, 6th at day 420, 7th at day 600 — server config, no client knowledge of the schedule. Offer appears as a quiet naturalist moment ("a new bird has been seen near the aviary") in-scene, accept/decline; declining re-offers after ~2 weeks. | The PRD specifies age-gating and the rough rhythm ("a few months old offers a third; a year-old may have five or six") without dates. These match that curve and are pure config. |
| C12 | **Drift calibration cannot use production user data** (the privacy commitment forbids population-level analysis of interaction patterns). Calibration uses the synthetic simulation harness (§16.2) plus an explicit-consent internal/beta cohort whose accounts are flagged consenting at creation. | The privacy rule is absolute; the calibration plan must work within it, and this is the only way it can. |
| C13 | **Visitors don't see the notebook.** The visit is the ambient scene + audio only; notebook, settings, offers, settle are absent from the visit surface. | The PRD says the visit is a "read-only ambient view" and "the notebook is the host's"; the notebook is the host's relationship record, not part of the ambient scene. |
| C14 | **Export is async**: generated by a job, delivered as a time-limited signed link emailed to the verified address (matches the PRD's "emailed as a download link"); link expires in 24h. | Keeps the API stateless and avoids long-poll generation requests. |

---

## 4. Data model

PostgreSQL. All foreign keys are the synthetic account UUID or entity UUIDs; **email appears in exactly one column** (`accounts.email_encrypted`) and in the transient invite/magic-link mailer payloads. Times are `timestamptz` UTC.

### 4.1 Identity & auth

- `accounts` — `account_id UUID PK` (generated at creation, used in every internal reference, log, key, and partition), `email_encrypted` (app-layer encryption, KMS-managed key), `email_hash` (HMAC, for uniqueness/lookup only), `timezone` (IANA), `created_at`, `deletion_requested_at NULL`, `settings JSONB` (visit-notification toggle, caption opt-in, reduced-motion override, audio prefs), `consent_flags JSONB` (calibration cohort, C12).
- `magic_links` — `token_hash PK`, `account_id`, `expires_at` (15 min), `consumed_at NULL`. Single-use enforced by atomic consume (`UPDATE … WHERE consumed_at IS NULL RETURNING`). Per-email request rate limit via `email_hash` counter table.
- `sessions` — `session_id PK`, `account_id`, `token_hash`, `device_label` (UA-derived, user-readable), `created_at`, `last_seen_at`, `revoked_at NULL`. Listed and revocable in account settings.
- `email_changes` — pending new-address verifications; old email remains active until the new address verifies.

### 4.2 Aviary & birds

- `aviaries` — `aviary_id UUID PK`, `account_id UNIQUE` (one per account), `created_at` (drives age-gated offers), `last_tick_at`, `last_event_seq_processed`, `weather_schedule JSONB` (seeded upcoming ambient events), `settled_state JSONB NULL`.
- `species` — static config table (six rows at v1): `species_id`, silhouette ref, default palette, call-motif library ref, idle-motion profile ref, `nocturnal BOOL` (the nightjar-like species).
- `birds` — `bird_id UUID PK` (**the stable identity; never reissued, never replaced across rename/sync/migration — enforced by the absence of any code path that swaps it**), `aviary_id`, `species_id`, `name`, `adopted_at`, `display_order`.
- `personality_vectors` — `bird_id PK`, `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (all `REAL` in [0,1]), `updated_at`, `updated_by_tick_id`. **Written exclusively by the tick service role; the API service's DB role has no UPDATE grant on this table.** The grant structure is the no-last-write-wins rule made physical.
- `bird_moods` — `bird_id PK`, `mood ENUM(wary, content, curious, drowsy, alert)`, `display_state ENUM(active, settled, sleeping)`, `entered_at`, `dwell_until`, `modifiers JSONB` (weather dampening etc.). Written only by the tick.
- `drift_ledger` — append-only per-bird daily drift deltas with input attribution (presence/listen-in/offer weights applied). Exists for auditability, the export, invariant checks (monotonicity), and disaster recovery (vectors can be re-folded from ledger + seed); it is *not* a runtime derivation source — the vector row stays canonical per the PRD.

### 4.3 Events & presence

- `interaction_events` — append-only: `event_id UUID`, `aviary_id`, `seq BIGSERIAL` (per-table ordering; tick consumes in `seq` order per aviary), `type ENUM(presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo, session_start, session_end)`, `bird_id NULL`, `payload JSONB`, `client_event_id UUID` (idempotency key — unique index on `(aviary_id, client_event_id)` makes retries safe), `received_at`. No raw input coordinates, no keystroke contents — payloads are semantic only.
- `presence_intervals` — server-coalesced from validated presence pings: `aviary_id`, `started_at`, `ended_at`, `source_session_id`. The tick reads intervals, not raw pings. Server-side validation caps ping rate (max 1 credited ping / 25s / account regardless of device count — two devices open simultaneously do not double presence) and caps creditable presence at 4h/day for drift purposes (anti-inflation; generous beyond any plausible real watching).
- `offer_cooldowns` — `bird_id`, `offer_type`, `available_at`. Server-enforced (~3 min per bird per type); client renders affordance state from snapshot.

### 4.4 Notebook, visits, export

- `notebook_entries` — append-only, immutable: `entry_id`, `aviary_id`, `written_at`, `prose TEXT`, `salience REAL`, `detector_tag` (internal only). Read-only API; no UPDATE/DELETE grants except hard-account-deletion.
- `visit_invites` — `invite_id`, `aviary_id`, `visitor_email_encrypted` (the deliberate second PII cell, scoped to this feature, shown back to the host in the visit log per the PRD), `token_hash`, `created_at`, `expires_at` (+30 days), `revoked_at NULL`, `first_used_at NULL`.
- `visit_log` — `aviary_id`, `invite_id`, `visited_at`, `approx_duration_s`. Written by the visit snapshot path; never triggers notifications unless the host's opt-in toggle is on.
- `export_jobs`, `deletion_jobs` — async job rows for §12.

### 4.5 Retention & deletion

Hard deletion (30 days after request) removes, in one transactional job: account row, sessions, magic links, birds, vectors, moods, drift ledger, events, presence intervals, notebook, invites, visit log, export artifacts, and issues a tombstone to the log pipeline to purge account-UUID-scoped operational log lines within the log retention window. Recovery during the window restores by clearing `deletion_requested_at`.

---

## 5. API surface

Stateless JSON over HTTPS. Auth via session bearer token (httpOnly cookie on web). All endpoints rate-limited per account UUID. Matter-of-fact voice on every error string. Idempotency on all mutating endpoints via `client_event_id`/request keys.

### 5.1 Auth & account

```
POST /v1/auth/magic-link        {email}                      → 202 always (no account-existence oracle)
POST /v1/auth/verify            {token}                      → session cookie; 410 with matter-of-fact copy on expired/used
POST /v1/auth/logout
GET  /v1/account                                             → settings, sessions list, timezone
PATCH /v1/account/settings      {timezone?, captions?, reduced_motion?, visit_notifications?, audio?}
DELETE /v1/account/sessions/:id                              → revoke device
POST /v1/account/email-change   {new_email} / POST /v1/account/email-change/verify {token}
POST /v1/account/export                                      → 202, job enqueued (C14)
POST /v1/account/delete         / POST /v1/account/restore   → soft-delete / changed-my-mind
```

### 5.2 Aviary state

```
GET  /v1/aviary/snapshot
```
Returns the full render-state document (~2–6KB gzipped):

```jsonc
{
  "aviary": { "as_of": "...", "day_phase_tz": "America/Chicago",
              "weather": {"active": null, "upcoming_seed": "..."},
              "settled": false, "seed": "per-time-bucket realization seed" },
  "session": { "absence_seconds": 41250 },          // drives return-greeting register
  "birds": [{
      "bird_id": "...", "name": "pip", "species": "...",
      "perch": "front", "mood": "curious", "display_state": "active",
      "performance": {                                // derived params, never raw traits (C5)
        "greeting_weight": 0.8, "approach": 0.7, "call_rate_band": [0.4, 0.9],
        "chorus_join": 0.6, "investigate": 0.75, "plumage": {"sat": 0.62, "detail": 2}
      },
      "motion_anchor": {"pose": "preen_mid", "since": "..."}  // first frame mid-action
  }],
  "offers": { "seed": {"available": true}, "song": {...}, "pool": {...} },
  "pending_bird_offer": null                          // or the quiet new-bird moment (C11)
}
```

Pull triggers (client): visibility→visible, render-frame gap >5s (suspend recovery), 45s keepalive while visible, after settle/undo, after offer submission. Snapshot request on a dormant aviary triggers synchronous catch-up fold first (C4), bounded by precomputed coarse ticks to keep p99 well under the 5s alarm.

```
POST /v1/aviary/events          [{client_event_id, type, bird_id?, payload, client_ts}]   // batched
GET  /v1/notebook?cursor=…&limit=…                       // reverse-chron, infinite scrollback
POST /v1/aviary/bird-offer/respond {accept: bool, name?: string}
PATCH /v1/birds/:id             {name}                   // rename, no engine effect
```

Presence pings post as events every 30s *only while all three presence conditions hold client-side*; the server re-validates cadence and coalesces (§4.3). Listen-in posts start/end pairs; the server derives duration (never trusts client-claimed durations beyond sanity bounds).

### 5.3 Visits

```
POST   /v1/visits/invites        {visitor_email}         → emails one-time link; 30-day expiry
DELETE /v1/visits/invites/:id                            → immediate revocation
GET    /v1/visits/log                                    → visits + outstanding invites (settings surface)
GET    /v1/visit/:token/snapshot                         → read-only; no session; no event ingestion path exists for this token class
```

The visit snapshot is the host snapshot minus host-only fields (offers, notebook access, settings, pending bird offers, session/absence data). Revocation/expiry → 410 with the matter-of-fact "this visit is no longer available" surface on the visitor's next pull. Visitor pulls log `visited_at`/duration to `visit_log` and **write nothing to `interaction_events` or `presence_intervals`** — structurally, the visit token cannot reach those tables.

---

## 6. Simulation engine — the tick

### 6.1 Determinism contract

`tick(state, events[], minute_t) → state'` is a pure function in the shared engine package: no wall-clock reads, no unseeded randomness (PRNG seeded from `(aviary_id, minute_t)`), no I/O. The worker shell does I/O (read state + events since `last_event_seq_processed`, write state, advance cursors) in one transaction per aviary per tick. Properties this buys: catch-up folds are exact (C4), replays are exact (recovery), the calibration harness can simulate months in seconds (§16.2), and a laptop/phone disagreement is impossible because there is nothing to disagree about.

Scheduling: workers claim due aviaries via `SELECT … FOR UPDATE SKIP LOCKED` on a due-time index — horizontal scaling with no coordinator and no double-tick.

### 6.2 Tick body (order matters)

1. Fold presence: close stale presence intervals; accumulate credited presence-seconds since last tick (caps per §4.3).
2. Consume events in `seq` order → semantic signals (listen-in durations per bird, offers + reactions, settle).
3. Update mood state machines per bird (§6.4) using time-of-day (account tz), active weather, signals, personality modulation.
4. Apply drift deltas (§6.3) to personality vectors; append `drift_ledger` rows.
5. Advance weather schedule; spawn/expire ambient events (seeded; a short rain a few times a week, occasional wind).
6. Recompute perch assignments where mood/personality warrants (bounded movement per tick — birds don't teleport across zones).
7. Run notebook detectors → candidate observations → sparsity governor (§10).
8. Check bird-offer age gates (C11).
9. Write state, advance `last_tick_at` and event cursor, emit tick-latency metric.

### 6.3 Drift function

Traits live in [0,1]. Per tick, per trait:

```
delta_trait = headroom × Σ_signals ( w[trait][signal] × signal_norm )
headroom    = (1 − trait)            // asymptotic approach, no overshoot, diminishing returns
delta_trait = min(delta_trait, remaining_daily_cap[trait])
trait'      = trait + max(0, delta_trait)     // monotonic: clamped at 0 below, never negative
```

- **Signals & weights (initial, config-driven):** presence-time dominates (`w ≈ 1.0` into all traits at small scale, strongest into plumage_saturation and boldness); listen-in minutes feed social_warmth and vocal_frequency of the focused bird (`≈ 0.5` relative); accepted offers feed curiosity, any near-bird offer feeds boldness (small, `≈ 0.2`); settle contributes nothing directional (mood-quieting only).
- **Daily per-trait cap** (starting value 0.01) prevents single-session saturation and is the second defense (after offer cooldowns) against engine collapse from burst interaction.
- **Monotonicity is structural**: there is no code path that produces a negative trait delta. Neglect produces *no* delta; the ambient-quietness of a neglected aviary comes from the expression layer (a bird whose warmth hasn't grown greets less often), not from trait decay. Property test: for all event sequences, traits are non-decreasing (§16.3).
- **Calibration targets (named, testable):** a synthetic "regular" user (5 visits/week, ~12 min credited presence/visit, occasional listen-ins) produces per-trait drift of ~0.02–0.05 by day 7 (instrument-measurable) and ~0.08–0.15 cumulative by day 21 on attended traits (threshold at which expression mapping makes it user-visible: greeting earlier, nearer perch, richer plumage). All constants live in a versioned server config; the harness (§16.2) regression-tests the targets on every constants change.

### 6.4 Mood model

Semi-Markov machine per bird (C10): states {wary, content, curious, drowsy, alert}; dwell-time distributions per state; transition weights modulated by — recent interaction signals (accepted offer → content), time-of-day curve (alert early morning, drowsy near dusk), weather modifiers (rain dampens vocal expression briefly, wind → alert or wary by personality), personality (high boldness suppresses wary entry), and bird-to-bird contagion (a wary neighbor raises wary weight; an alarm call propagates). Hysteresis: minimum dwell before re-transition. Display states (settled at user settle; sleeping at night for non-nocturnal species) overlay the mood without destroying it. Mood persists in `bird_moods` across sessions; the tick advances it in absence — the PRD's "drowsy at dusk yesterday → settled by morning" emerges from dwell expiry + time-of-day weights, never from a tab-open reset (there is no client-triggered mood write at all).

### 6.5 Expression mapping (state → behavior parameters)

A pure server-side function maps (personality, mood) → the `performance` block in the snapshot (C5): greeting weight (boldness × warmth × mood gate), approach tendency, call-rate band (vocal_frequency × mood × weather damping), chorus-join propensity, investigation propensity (curiosity × mood), plumage render parameters. This is the single place trait numbers become behavior, so calibrating "visible after three weeks" is a matter of tuning one mapping, and the client cannot reverse-engineer clean trait values from the composite outputs.

---

## 7. Client interaction systems

### 7.1 Return-greeting

On session start (fresh navigation or visibility-return after >10 min hidden), the client requests a snapshot and runs the greeting selector from the shared engine package: candidate birds weighted by `greeting_weight` (boldness/warmth/mood-derived), seeded by `(aviary_id, day, session_count)` so it varies across sessions but is stable within one. Selection honors the PRD's shape: bolder birds tend to greet first; a wary bird may not greet at all today. The greeting *form* is composed procedurally from a choreography grammar — gesture atoms (glance-up-from-preen, head-tilt, step-to-front, two-note call, longer call + second-bird response) assembled by absence-band:

- < 30 min: a glance or head-tilt; no call required.
- 30 min–1 day: glance + quiet short call.
- 1 day: re-orientation — approach toward a nearer perch and/or a longer call, possibly answered.

Variation is real (grammar over atoms with seeded parameters — timing, pitch contour, gaze path), not N canned variants in rotation. If multiple birds would greet, offsets are randomized 0.8–2.5s apart, never simultaneous. The greeting must fire within ~1–2s of first render; it is part of the critical path budget. `session_start` (with absence band) is logged as an event for the engine; **no textual welcome surface of any kind exists in the codebase** — enforced by the voice lint (§11.3) which has no "welcome" string class to use.

### 7.2 Listen-in

Click/tap/keyboard-Enter on a bird engages listen-in: focused bird's call bus ramps up and others ramp *down to ambient floor, never zero* over ~1.5s (equal-power curves, §9.4). Disengage (same bird again, another bird, empty-space click, focus loss, Escape) ramps back identically. Client posts `listen_in_start`/`listen_in_end`; server credits drift to the focused bird only (warmth, vocal frequency). Visual treatment is minimal — a subtle attention cue (slight camera-weightless emphasis like a soft vignette of audio, no outline ring in the mouse path; keyboard focus uses the accessible focus indicator per §11.5).

### 7.3 Offers

Offer affordance lives in the top bar only (never on the bird). Three offer types; each spawns a scene element (seed at front-ground, soft song-fragment playback through the ambient bus, still pool at front). Bird responses are realized client-side from `investigate`/mood parameters (curious+content approaches; wary waits then edges near; drowsy may ignore) — and reported as an `offer` event with the realized outcome class so the tick credits curiosity/boldness drift consistently with what the user saw. Cooldowns are server-truth (snapshot carries `available_at`); the client shows the affordance quietly dimmed during cooldown with no countdown timer (a countdown is a meter — wrong register).

### 7.4 Settle

Settle (top bar) shifts lighting to evening over ~4s, quiets the call buses, plays a soft acknowledgment (one low call), and posts `settle`. Any click within 5s posts `settle_undo` and reverses the ramp. Settled state persists until tab close or re-engagement (any interaction un-settles visually over a few seconds). Engine-wise settle only closes the presence window cleanly — identical end-state to tab close, per the presence model.

### 7.5 Presence (client side)

A presence monitor samples the three-condition conjunction — `document.visibilityState === 'visible'` AND `document.hasFocus()` AND (last pointermove/keydown within 4 min (C7)) — and emits a ping event every 30s while it holds. Conditions are evaluated at send time, not buffered: a ping is never sent retroactively. Loss of any condition stops pings; the server closes the interval after a missed-ping grace (75s). The client never computes presence-time; it only attests conditions, and the server owns crediting (§4.3).

---

## 8. Sync model

Single canonical record; sync is a property, not a feature:

- The server is the only writer of personality (DB grants, §4.2) and mood. Clients submit append-only semantic events with idempotency keys; the tick consumes them in `seq` order. "Set boldness to X" is not expressible in the API schema.
- Two devices = two readers of one record. No client-to-client path, no merge, no LWW anywhere in the design. The morning-laptop/lunch-phone overwrite scenario from the PRD is unreachable: both sessions only appended events, and the tick folded both in order.
- Stale-render handling: a snapshot pull that returns state older than the client's current interpolation epoch is impossible (server state is monotone by `as_of`); a *newer* snapshot reconciles via interpolation — birds move smoothly to new perches/moods, never teleport (bounded-movement rule in the tick keeps deltas interpolable).
- Suspend/resume: render-gap detector (>5s between frames) forces a snapshot pull before resuming motion, so a re-opened laptop shows the aviary that kept running, not a frozen-then-jumping one.
- Conflict surfaces that *can* occur (expired magic link, replayed link, session timeout, server error) get the matter-of-fact voice exactly as specified, e.g. "Your session timed out. Sign in again to keep watching."

---

## 9. Frontend rendering & audio pipelines

### 9.1 Stack & structure

TypeScript; Preact (or equivalent ~4KB view layer) for chrome surfaces only (top bar, settings, notebook, auth); the aviary itself is a hand-rolled layered Canvas 2D renderer (C2) behind a `Renderer` interface (full-motion and reduced-motion implementations; WebGL2 swap possible later). Aggressive route-level code-splitting: settings, accessibility settings, notebook, invite flow, and auth are async chunks; the critical chunk is scene + engine-runtime + audio bootstrap only.

### 9.2 Scene composition

Five layers, dirty-rect composited:
1. **Sky/light layer** — gradient field driven by local-time day-phase curve; also *is* the loading and empty states (the quiet field — same component, so "loading" is literally the aviary's sky, never a spinner; no spinner asset exists in the bundle).
2. **Background foliage** — near-static, re-rendered on light changes; subtle parallax (translation ≤1–2% on pointer drift).
3. **Bird/perch plane** — three perch zones; procedural skeletal birds: per-species silhouette skeleton + parameterized pose system (perch, preen sequence, head-tilt, weight-shuffle, fluff, call posture, hop, short flight arcs), tinted by plumage params. Poses blend continuously; idle scheduler picks mood-shaped behaviors (wary → back perch + scanning; content → preen; curious → tilt toward sound/leaf events; drowsy → low posture, fluffed).
4. **Foreground ornaments** — client-only leaves/feathers at slow random cadence from a pooled particle system (zero steady-state allocation).
5. **Weather overlay** — rain streaks/ripple, wind-driven foliage ripple, driven by server weather schedule (C8).

First-frame rule: the renderer's first paint places every bird mid-pose from `motion_anchor` (snapshot says Pip is mid-preen since t) — pose clocks are initialized to `now − since`, so frame one is mid-action with no entry transition, no fade-from-static. Tab hidden → rendering halts entirely (rAF stops, audio suspends to silence gracefully); resume → snapshot pull → reconcile (§8).

Responsiveness: layout solver keeps all birds in frame at every viewport; horizontal compression narrows perch spacing on phones, widens on desktop; nothing crops, nothing pans.

### 9.3 Audio pipeline

WebAudio graph:

```
per-bird voice (2 osc [sine/triangle] + FM mod + filtered noise + ADSR + per-call pitch contour)
  → per-bird gain (listen-in automation)
  → spatial pan (perch-zone derived)
  → chorus bus → ambience bus (wind/rain beds, also procedural) → compressor → master
```

- **Call grammar runtime** (shared engine package): per-species motif library (3–5 motifs: note cells with pitch-contour, duration, and timbre params) + per-bird signature derivation — a stable hash of `bird_id` selects motif ordering biases, pitch center offset, and timing personality so a bird's voice is *recognizable across mood and drift* (signature parameters are functions of identity; mood and vocal-frequency modulate rate, intensity, and contour expressiveness, never the signature core). Each call is realized by seeded PRNG: motif selection → variation operators (transpose within signature range, time-stretch, ornament insertion) → synthesis plan. No two calls identical; no call off-signature.
- **Scheduler**: per-bird Poisson-ish call timer inside the server-set `call_rate_band` (C9); call/response — a realized call raises short-window response propensity in high-warmth birds; chorus events emerge when ≥2 birds' timers overlap (chorus-join propensity gates joining). All scheduling uses the WebAudio clock with 200ms lookahead batching.
- **Listen-in mix**: gain automation via `setTargetAtTime` ramps (~1.5s engage/disengage); ambient floor at −18dB relative, never −∞.
- **Buffer discipline**: voices and noise buffers are pooled and reused; zero per-call allocations after warmup (feeds the no-memory-growth gate).
- **Caption hook**: every realized synthesis plan emits a structural description (note count, contour shape, tempo, intensity, source perch) consumed by the caption generator (§11.4) — captions always describe the call that actually played.
- **Fallback**: AudioContext unavailable/denied → graceful silence, captions auto-enabled, no recorded-audio path exists in the build. Autoplay gating per C3.

### 9.4 Day/night & settle lighting

A single light-state controller (time-of-day curve from device-local time, settle override, weather dimming) feeds sky layer, palette tinting, and audio ambience levels, so visual and audio "evening" always agree. Night: non-nocturnal birds in sleeping display state; the nightjar-like species stays active with occasional late calls.

---

## 10. Field notebook generation

### 10.1 Detector → governor → prose pipeline (runs in the tick)

1. **Detectors** observe aviary state transitions and emit candidates with salience: first-greeter changes ("pip greeted before wren — first time this week" requires a rolling per-aviary greeting-order memory), perch-pattern novelty, weather moments + bird reactions, chorus events, offer vignettes, long-quiet observations, new-bird arrival, plumage-richness milestones *expressed observationally* ("pip's color has come in richer this month" — never numerically).
2. **Sparsity governor** — token bucket per aviary (~1 entry / 2–4 days baseline; burst allowance of 1 for high-salience events like a new bird); below-threshold candidates are dropped, not queued — sparsity holds even for very active users.
3. **Prose realizer** — naturalist grammar (lowercase, present-tense, bird-named, specific): templates with variation slots filled from the actual state that triggered the detector, run through the voice linter at generation time. Entries are immutable rows.

### 10.4 The hard line

No detector class takes user behavior as subject matter. The detector input schema simply has no fields for visit counts, session frequency, or user actions as such — the notebook can say *pip investigated the seed slowly*, never *you visited every day this week*. Structural absence, not editorial policy.

---

## 11. Accessibility surfaces (ship with v1, not after)

### 11.1 Organizational stance

Accessibility surfaces are built by the same engineers in the same milestones as their visual counterparts (each renderer/audio work item carries its narration/caption/reduced-motion acceptance criteria). Nothing here is post-launch.

### 11.2 Screen-reader narration

A narration composer (shared prose-grammar infrastructure with the notebook) renders running naturalist prose from the same realized client state the visuals draw from — *"a small grey bird is perched on the front rail, calling softly. it is morning in the aviary; the light is gentle."* Delivery via a visually-hidden `aria-live="polite"` region; idle cadence one update per 30–60s; user-initiated events (return-greeting, offer reactions, settle) get prompt delivery via a second `aria-live="assertive"`-adjacent priority path but are still written as observations, never state transitions ("a warbler perches on the high branch, calling softly," not "warbler perched at high branch"). Queue discipline: one pending utterance max — new idle narration replaces unspoken idle narration rather than stacking. Voice identical to notebook voice (same grammar core), so the product sounds like one product across surfaces.

### 11.3 Voice & copy system (supports everything above)

All user-visible strings flow through a copy registry with a register tag: `naturalist` (scene, notebook, narration, captions, offer prompts) or `system` (auth, errors, sync, account/accessibility settings, unsupported-browser page). A CI voice-lint enforces register rules — naturalist: lowercase, present tense, no exclamation marks, no second-person announcement framing, no gamification lexicon (blocklist: streak, badge, level, score, achievement, unlock, points, welcome back…); system: plain capitalized English, states what happened and what to do. New-surface rule encoded in the registry docs: money/identity/errors/settings ⇒ `system`; everything else ⇒ `naturalist`.

### 11.4 Captions

Opt-in from accessibility settings (auto-on under the WebAudio fallback). Generated at runtime from each call's realized synthesis plan (§9.3): contour+tempo+intensity map to naturalist phrases — "a soft three-note rise," "a low trill, paused, low trill again," "a single sharp call from the back perch." Rendered as small text near the calling bird, fade in/out with the call, WCAG AA contrast against both bright and dim scene states (auto-switching text treatment), reduced-motion-safe (opacity fade only).

### 11.5 Reduced-motion mode

Triggered by `prefers-reduced-motion` or the settings override; implemented as the second `Renderer` (§9.1), a designed surface: micro-motion becomes slow cross-fades between still poses (preen = pose sequence cross-faded over seconds), flights become cross-fades between perches, leaf/feather ornaments removed, day/night color shifts retained but slowed, weather as gentle static treatments. Calls, captions, drift, mood, notebook all unchanged — same aviary, different visual register, with its own QA/design review pass (it must read as *calmer*, not broken).

### 11.6 Keyboard & focus

Tab order: top bar items → aviary scene (first bird) → arrow keys move between birds → Enter engages listen-in → Escape disengages → offer panel and settle reachable via top bar (with shortcuts), fully keyboard-operable; focus trap correctness in panels. Focus indicator: soft high-contrast outline tuned against bright and dim aviary states (design-system token, tested in both). All chrome text WCAG AA minimum per the design system's per-surface ratios.

---

## 12. Accounts, privacy, lifecycle (engineering notes beyond §4)

- Magic-link mail goes through a transactional ESP; link tokens are 256-bit, stored hashed; the verify endpoint is single-use-atomic and replay returns the matter-of-fact expired surface. Request endpoint returns 202 regardless of account existence.
- Session cookies: `Secure; HttpOnly; SameSite=Lax`, rotating token on privilege-relevant actions; device list shows label + last-seen; revocation immediate (token check per request).
- Export job: serializes birds (names, species, adoption dates), current personality vectors (the export is the named exception where the user's numbers are *theirs to take* — per the PRD's export list — but the values appear only in the exported file, never in any product surface), moods, notebook entries, settings; uploads to expiring signed storage; emails the link (C14).
- PII audit gate in CI: schema migrations and log statements are scanned for email-shaped fields outside the two sanctioned encrypted columns; metric definitions scanned per §2.3.
- Privacy policy page: plain text, linked from account settings, naming aggregate telemetry categories and explicitly excluding per-bird interaction state.

---

## 13. Performance engineering

### 13.1 Bundle budget (<2MB gz, with internal targets well under)

| Chunk | Target (gz) |
|---|---|
| Critical: bootstrap + scene renderer + engine runtime + audio synth + bird assets (procedural skeletons + 6 species silhouettes/palettes as compact vectors) | ≤ 600KB |
| Deferred-on-idle: notebook, narration grammar extensions, offer panel, weather extras | ≤ 400KB |
| Route-split: settings, accessibility settings, auth, invite flow | ≤ 300KB |

Procedural audio and procedural/SVG bird art are what make this fit — no recorded audio, no sprite sheets of consequence. CI budget gate fails the build over per-chunk targets; the 2MB cap is the hard ceiling with ~700KB headroom held in reserve.

### 13.2 Time-to-first-bird <500ms (mid-tier mobile, 4G)

Critical path: edge-cached HTML with inlined critical CSS + sky-field paint (the quiet field renders within ~100ms regardless) → critical JS chunk (preloaded, HTTP/3, edge) → state: **returning user**: last snapshot persisted in IndexedDB renders the first bird immediately (mid-pose, from cache) while the fresh snapshot fetch races; reconcile on arrival (visually a normal interpolation step). **First-ever session / cold cache**: snapshot endpoint is a single small read served from the nearest region; render-on-arrival, with the quiet field (never a spinner) covering the gap. Synthetic checks (§13.5) enforce the 500ms p75 on a Moto-class device profile over throttled 4G as a release gate.

### 13.3 Runtime: 60fps for 30 minutes

Dirty-rect compositing (sky gradient updates at low Hz; only birds/ornaments/weather repaint per frame); pose math is allocation-free (preallocated typed arrays); rAF-driven with frame-time telemetry; degradation ladder (drop parallax → reduce ornament cadence → lower offscreen resolution) before ever dropping bird motion. Target device: 2021 mid-range laptop profile in the synthetic fleet.

### 13.4 Memory: zero growth over 30 minutes — a CI test, not a guideline

Nightly (and pre-release) Puppeteer soak: scripted 30-minute session (idle + listen-ins + offers + notebook scroll) sampling JS heap + AudioContext node counts; pass = no monotonic growth trend beyond GC noise envelope. Engineering rules feeding it: pooled audio buffers/particles, notebook virtualization releases scrolled-out entries, bounded workers/contexts, no per-call or per-frame allocation in steady state.

### 13.5 Observability

- **Synthetic fleet**: scheduled headless browsers from N common geographies measuring first-bird-time, frame-rate distribution, audio-init success, snapshot latency.
- **Aggregate RUM** (no per-account dimensions): page-load and first-bird timings, long-frame counts, audio-context errors, snapshot pull latencies/failures.
- **Server**: tick latency (p99 alarm > 5s), tick backlog depth, event-ingest rates/errors, snapshot p99, auth funnel errors, mailer deliverability.
- **Engine health (privacy-safe)**: invariant violation counters (any attempted negative drift, any non-tick personality write attempt — both also hard-fail), drift-ledger consistency checks. No per-bird state, no interaction histories, no population drift analytics (C12).

---

## 14. Rollout plan

### 14.1 Milestones (gated, roughly two-week increments; workstreams parallel where noted)

- **M0 — Foundations.** Repo/CI (budget gates, voice lint, PII lint scaffolding), Postgres schema + grants (single-writer enforced from day one), auth (magic link, sessions), skeleton API, deploy pipeline, telemetry split (§2.3) stood up *before any product data exists*.
- **M1 — Engine vertical slice (headless).** Shared engine package: tick fold, drift math, mood machine, presence crediting; calibration harness (§16.2) running simulated months; determinism + monotonicity property tests green. *Gate: calibration targets demonstrable in simulation.*
- **M2 — Aviary renders.** Scene layers, procedural birds with mood-shaped idle, day/night, snapshot pull + interpolation, first-frame-mid-action, presence monitor, quiet-field load/empty states. Reduced-motion renderer developed in the same sprint as the full renderer. *Gate: first-bird <500ms in synthetic test; 60fps on target laptop.*
- **M3 — Sound & interaction.** Call grammar + synthesis, chorus, listen-in, offers + cooldowns, settle/undo, return-greeting choreography, captions, narration composer, keyboard nav. *Gate: per-bird recognizability ABX pass (§16.4); accessibility script pass (NVDA/VoiceOver).*
- **M4 — Relationship layer.** Notebook detectors/governor/prose, adoption flow + empty-aviary→fly-in, naming/renaming, bird offers (age gates), export, deletion/restore, email change. *Gate: notebook sparsity + voice-lint pass on a 60-day simulated aviary.*
- **M5 — Visits & hardening.** Invite/revoke/expiry/log, visit snapshot path, settings surfaces, unsupported-browser page, memory soak green, security review (auth, token handling, PII audit), load test of tick fleet, chaos pass on tick recovery/replay.

### 14.2 Release ramp

1. **Internal dogfood (team + friends-of-team, consenting per C12)** — 2–4 weeks. Primary purpose: presence-window calibration (C7), drift-constant tuning against real attention patterns, greeting/notebook tone review, audio uncanniness panel (§16.4).
2. **Closed beta (invited, consenting cohort)** — gates: zero sync-correctness incidents, tick p99 < 1s at cohort scale, accessibility sign-off from external screen-reader users (paid testing), perf gates green for 2 consecutive weeks.
3. **Open launch** — quiet availability (no growth mechanics exist to ramp); regional snapshot serving verified; on-call rotation + runbooks for tick backlog, mailer outage, and DB failover.
4. **Post-launch ramp** — birds-per-aviary grows organically via age gates (no action needed; the day-60 third-bird offers begin reaching the earliest accounts ~2 months post-launch — the first time chorus density rises in production, with the audio-recognizability monitoring from §16.4 re-run at each population milestone).

Instrumented from day one: everything in §13.5 — and nothing else; the discipline of *not* adding engagement instrumentation is part of the launch checklist.

### 14.3 Operational kill-switches (all config, no deploy)

Drift freeze (safe because drift is monotonic — freezing never visibly regresses a bird), weather off, visits off, bird-offers pause, narration cadence dial, presence-credit dial. Each preserves user-visible continuity.

---

## 15. Risks & mitigations

| Risk | Why it's real | Mitigation |
|---|---|---|
| **Drift mis-calibration** (too fast → Tamagotchi-feel; too slow → screensaver) | The product lives in a narrow band; no other system pins it down; production aggregate data is off-limits (C12) | Deterministic calibration harness simulating user archetypes over months (runs in CI on every constants change); named numeric targets (§6.3); consenting-cohort validation in dogfood/beta; config-driven constants + drift-freeze switch; expression-mapping tuned independently of drift rates (two knobs, not one) |
| **Sync/data-loss on personality state** (the worst, silent failure — a reset bird passes every unit test) | The PRD names this the product's worst failure | Single-writer enforced at DB-grant level; append-only events with idempotency; drift ledger enables full re-fold + daily invariant audit (vector == fold(ledger)); PITR backups; alert on any vector write outside a tick transaction (defense in depth — it's also impossible by grants) |
| **Audio uncanniness / signature blur** (calls feel synthetic-canned, or 5–7 birds blur) | Audio is the affective spine; the 7-cap premise depends on recognizability | Motif-library design with a sound designer in M3; ABX recognizability protocol (§16.4) as a release gate and re-run at each bird-count milestone; chorus mix rules (spectral spacing of signature pitch centers across the species pool); fallback = reduce chorus-join propensity via config, never add recordings |
| **Autoplay policy vs. "already audible" first frame** | Browsers hard-block pre-gesture audio | C3: visual aliveness carries the first impression; audio fades in mid-call on first gesture; captions cover the gap; copy review ensures no blocked-audio nagging |
| **Presence signal distortion** (inflated → population over-drift; too strict → watching quietly doesn't count) | Drift integrity depends on presence honesty; the PRD calls the failure silent | Three-condition conjunction implemented exactly; server-side re-validation, multi-device dedup, daily credit caps (§4.3); long activity window biased toward "sitting and watching" (C7), tuned in dogfood; engine-health counters on anomalous ping patterns |
| **Accessibility regression / drift toward checklist-mode** | The designed surfaces cost more and erode under deadline pressure | Accessibility acceptance criteria inside each feature's definition-of-done (§11.1); narration/caption/reduced-motion in CI (voice lint, axe checks, scripted SR runs); paid external screen-reader testing as a beta gate; reduced-motion has its own design review |
| **Tick fleet scaling / hot backlog** | 1-min cadence × accounts is the main scale axis | Adaptive cadence with exact catch-up (C4); SKIP LOCKED sharding scales horizontally; backlog-depth alarm well before user-visible staleness; load test at 10× projected accounts in M5 |
| **Magic-link deliverability** (the only door into the product) | ESP reputation issues lock users out entirely | Reputable transactional ESP + domain warmup, DMARC/SPF/DKIM, deliverability monitoring, secondary ESP failover behind an interface |
| **Notebook tone failure** (entries read generic/system-y at scale of generation) | The notebook is the voice's most concentrated surface; template smell compounds | Grammar with wide variation slots reviewed by the writer who owns voice; voice lint; sparsity governor keeps volume low enough for human review of samples in beta; detector ships only with ≥N realized-prose variants |
| **Timezone/DST edge cases** (mood time-of-day vs. render-local mismatch) | Two clocks (account tz for canonical, device-local for render) can disagree | C6 last-reported-wins; IANA tz handling via standard lib; DST property tests; the mismatch worst case is mood slightly out of phase with lighting for one session — acceptable and self-healing |
| **Scope creep toward announcement/gamification surfaces** | The PRD predicts well-meaning contributors will add "one harmless toast" | Non-goals encoded as lints and structural absences (no toast component, no welcome string class, no streak-capable schema, no per-account analytics dimensions); PR template includes a non-goals checklist; this plan's §1.2 is the canonical refusal list |

---

## 16. Testing & verification strategy

1. **Engine unit/property tests**: determinism (same inputs → identical state, across Node versions); monotonic drift (∀ event sequences: traits non-decreasing); fold-equivalence for adaptive cadence (C4); mood-machine dwell/hysteresis properties; presence-crediting caps.
2. **Calibration harness**: deterministic simulation of user archetypes (regular/sporadic/intense/absent-two-weeks-then-returns) over simulated months in seconds; asserts the §6.3 targets and the "returns after two weeks to quieter-not-resentful birds" scenario; runs in CI on engine-constant changes.
3. **Sync/integration**: multi-device event interleaving (the laptop/phone scenario from the PRD, asserted unreachable); idempotent retries; tick crash/replay recovery; visit-token write-path absence (attempted event POST with visit token → 403 + alert).
4. **Audio**: ABX per-bird recognizability protocol (panel distinguishes each bird's calls across moods at 2, 5, and 7 birds; gate ≥ threshold accuracy); no-two-identical-calls statistical test; mix-ramp curve tests; fallback path (context-denied → silence + captions).
5. **Perf CI**: bundle budgets per chunk; first-bird synthetic gate (throttled device profile); 30-min memory soak; frame-time regression suite.
6. **Accessibility CI + manual**: axe-core on chrome surfaces; voice lint on all registered copy + generated prose samples; scripted NVDA/VoiceOver narration sessions; keyboard-path integration tests; contrast checks in bright/dim/settled scene states.
7. **Privacy/PII gates**: schema/log/metric lints (§12); quarterly access audit of the telemetry/simulation boundary.
8. **Voice review**: human review of sampled generated prose (notebook, narration, captions, greetings) each release — the one quality bar that stays human.

---

## 17. Team & sequencing assumptions

Assumed team: ~6 engineers (2 engine/backend, 2 client rendering/audio, 1 full-stack accounts/visits/infra, 1 accessibility-focused client), a designer owning the design system + reduced-motion register, a sound designer (M3-critical), and a writer owning the voice registry and prose grammars. The writer and sound designer are not nice-to-haves; the notebook, narration, captions, and call libraries are the product's voice, and §10–11 assume their review loops. Total to open launch: roughly 6 months through M5 + ramp, with the engine calibration harness (M1) the long pole that everything affective sits on.

---

*End of plan. The deliverable is this plan; no product code is included by design.*
