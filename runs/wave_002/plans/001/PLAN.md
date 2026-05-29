# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable build for a frontier engineering team. It assumes the affective constraints in the PRD are non-negotiable acceptance criteria, not aspirations: aliveness, "notice never announce," specificity, restraint, the naturalist/matter-of-fact voice split, monotonic-toward-expressive drift, server-authoritative state, and the explicit non-goals. Where the PRD leaves a value open ("calibrate during build"), this plan names a starting value and a calibration method rather than a final number.

---

## 1. Scope

### 1.1 In scope for v1

- **Aviary runtime**: single horizontal scene, 2 starter birds, cap 7, three perch zones, day/night anchored to user local time, rare ambient weather, ambient leaf/feather drift, top-bar chrome with fade.
- **Bird engine (server)**: per-bird persistent personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood enum (wary, content, curious, drowsy, alert), procedural call grammar parameters, slow-tick simulation (~1/min), monotonic-toward-expressive drift, bird-to-bird interaction, ~6 species pool, stable per-bird identity, age-gated bird offers.
- **Interactions**: return-greeting, listen-in, offer (seed / song-fragment / still-pool) with per-bird cooldown, settle (with 5s undo), field notebook (sparse, read-only naturalist entries), presence accounting (the three-signal conjunction).
- **Accounts/sync**: magic-link auth, single-user single-aviary, synthetic UUID account id, per-device revocable sessions, email change with verification, account export (emailed link), soft-delete 30d then hard-delete, server-authoritative state, append-only client event log, snapshot pull + client interpolation, additive server-authored personality deltas (no last-write-wins).
- **Social**: per-invite opt-in read-only ambient visit, revocable, 30-day invite expiry, silent visit log, opt-in (off by default) visit notification toggle. No co-presence, no visitor drift.
- **Accessibility (first-class, ships with v1)**: naturalist screen-reader narration on slow cadence, reduced-motion as a designed cross-fade surface, runtime-generated call captions, WCAG AA on all user copy, full keyboard navigation, visible focus indicators.
- **Performance/observability**: ≤2MB gzipped initial bundle, <500ms time-to-first-bird (mid-tier mobile / 4G), 60fps idle on 5-year-old laptop, zero memory growth over 30 min (CI-enforced), client WebAudio procedural synthesis with silence+captions fallback, aggregate-only telemetry honoring the privacy boundary, synthetic perf monitoring, tick-latency p99<5s alarm.

### 1.2 Explicitly out of scope (enforced as build rules, not just omissions)

Per `non_goals.md` and scattered prohibitions: native apps; **all** gamification (achievements, streaks, levels, scores, badges, counts like "birds adopted: 2", green-dot calendars, XP, tiers, "you've been here every day" surfaces — in any form, including settings toggles or opt-in dashboards); Tamagotchi mechanics (death, hunger, distress, decaying happiness, negative drift on neglect); social-network surfaces (profiles, follows, public feed, discovery, leaderboards, comments, co-presence, friend-of-friend); push/email notifications about the aviary; recorded-audio fallback; any per-bird/per-account data feeding aggregate analytics or ML; numeric exposure of the personality vector anywhere, any tier; "Welcome back" toast/banner/modal; spinner/fade-from-static load sequences; user-controlled perch placement; panning/scrolling/zooming the scene.

These are encoded as automated guards (§11.4) so they cannot leak in through "harmless" PRs.

### 1.3 Defensible calls made where the PRD is open

- **Presence activity window**: start at **3 minutes** of pointer/key inactivity before presence drops; calibrate (§5.4). Bias long because "watching without moving is the product."
- **Tick cadence**: **60s** canonical tick; drift integration uses elapsed wall-time so a slower/faster tick changes cost, not calibration.
- **Drift calibration targets**: instrument-detectable change after ~7 days regular visits; user-perceptible after ~21 days. Encoded as a golden simulation test (§5.3, §12).
- **Offer cooldown**: **3 minutes** per bird per offer-type.
- **Notebook sparsity**: target ~1 entry / 2–4 days for a regular visitor; hard rate cap (§4.6).
- **Species pool size**: 6 species, one of which is the nightjar-like night-active signature.

---

## 2. Architecture

### 2.1 Service shape

Three backend services plus a static client, behind an edge/CDN. Keep services few; the engine is the value, not the topology.

1. **Auth/Account service** — magic-link issuance/consumption, sessions, email change, export, deletion lifecycle, invite issuance/revocation, visit-log reads. Owns the (encrypted) email and synthetic-UUID account record. Stateless app tier + Postgres.
2. **Aviary state service (read/write API)** — serves snapshots to clients, accepts interaction events into the append-only log, serves narration text, serves visit (read-only) snapshots. Does **not** mutate personality. Stateless app tier in front of the simulation datastore.
3. **Simulation tick worker** — the only writer of personality vectors. Runs the slow tick over all live aviaries, consumes the event log in order, updates vectors/moods, writes canonical state, emits notebook entries. Horizontally shardable by account-UUID hash.

**Datastores**: Postgres (accounts, sessions, invites, birds, personality vectors, mood, notebook, visit log) as system of record; the append-only interaction event log as an ordered, partition-by-account stream (Postgres table partitioned by account, or Kafka/Kinesis with account-UUID partition key — choose Postgres-first for v1 simplicity, see §6.3). A small Redis/edge KV holds the latest serialized snapshot per account for sub-100ms snapshot reads and CDN-edge seeding of first paint.

```
              ┌────────────┐
  browser ◀──▶│   Edge/CDN │──▶ static client bundle + HTML + seeded snapshot
              └─────┬──────┘
                    │ HTTPS (JSON)
     ┌──────────────┼───────────────────────┐
     ▼              ▼                         ▼
┌─────────┐   ┌───────────────┐        ┌──────────────┐
│ Auth/Acct│  │ Aviary State  │        │  Snapshot KV │
│ service  │  │  service (R/W)│◀──────▶│  (Redis/edge)│
└────┬─────┘  └──────┬────────┘        └──────┬───────┘
     │               │ append events           │ reads snapshot
     ▼               ▼                          │
  ┌─────────────────────────────────┐          │
  │           Postgres              │          │
  │ accounts/sessions/invites       │          │
  │ birds/personality/mood/notebook │          │
  │ append-only event log (part.)   │◀─────────┘ writes snapshot
  └───────────────┬─────────────────┘
                  │ consume log in order, write canonical state
                  ▼
          ┌──────────────────┐
          │ Simulation tick  │  (only writer of personality)
          │ worker (sharded) │
          └──────────────────┘
```

### 2.2 Client/server split (the load-bearing boundary)

- **Server owns**: personality vectors, mood, drift, mood timers, canonical positions/intentions, notebook generation, narration prose generation (or its structured inputs), call-grammar parameters per bird, weather events, day/night phase (derived from snapshot timestamp + client-reported timezone offset, see §2.4).
- **Client owns**: rendering, interpolation between snapshots, idle micro-motion within the mood envelope the server sets, ambient ornaments (leaves/feathers — pure client, no server state), procedural audio synthesis from server-provided grammar params, the local presence-signal detection, the listen-in mix, the settle lighting animation and its 5s undo, caption text rendering from the same grammar params.
- **Hard rule**: the client never computes or submits personality. It submits *events*; the server decides meaning. There is no code path where a client PUTs a vector. Enforced by API shape (§4) — there is no write endpoint for personality.

### 2.3 Render pipeline boundary

The simulation produces *intentions and envelopes* ("Pip: mood=curious, target perch=front, call-density=high, next-call window=…"); the client produces *motion* (the actual preen frames, the flight arc, the call audio). This separation is what lets the tick run at 60s while the client renders at 60fps: the client is never waiting on the server for a frame, only for the occasional state update it interpolates toward.

### 2.4 Time and timezone

Day/night is the user's **local** time. Client sends its IANA timezone (and raw UTC offset as fallback) on snapshot requests; the snapshot carries the server UTC timestamp and the resolved local phase so the client and the narration agree. The tick computes mood time-of-day effects using each account's last-known timezone (stored on the account; updated on snapshot pulls). This keeps "morning in the user's timezone is morning in the aviary" true even while the user is away and only the server is ticking.

---

## 3. Data model

All identifiers are synthetic UUIDs. Email appears **only** on `account.email_encrypted`.

### 3.1 Account / auth

- `account` — `id (uuid pk)`, `email_encrypted`, `email_hash` (keyed HMAC, for login lookup only, never logged), `timezone`, `created_at`, `state` (`active|pending_delete`), `delete_requested_at`, `visit_notify_optin (bool, default false)`, `privacy_policy_version_ack`.
- `pending_email_change` — `account_id`, `new_email_encrypted`, `new_email_hash`, `token_hash`, `expires_at`.
- `magic_link` — `token_hash`, `email_hash`, `purpose (login|visit|email_change)`, `expires_at (now+15m for login)`, `consumed_at`. Single-use; consumed atomically.
- `session` — `id (uuid)`, `account_id`, `device_label`, `created_at`, `last_seen_at`, `revoked_at`. Token is a random opaque secret; only its hash is stored.

### 3.2 Aviary / birds

- `aviary` — `id`, `account_id (1:1)`, `created_at` (drives age-gated offers), `settle_state (active|settled)`, `settled_at`.
- `bird` — `id (uuid, stable for account lifetime)`, `aviary_id`, `species_id`, `name`, `created_at`, `seed` (deterministic RNG seed for call grammar / visual variation).
- `personality_vector` — `bird_id (pk)`, `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (all float, normalized 0..1), `updated_at`, `version (monotonic int, bumped per tick write)`. **Only the tick writes this row.**
- `mood_state` — `bird_id (pk)`, `mood (enum)`, `entered_at`, `mood_expires_hint`, `current_perch (front|middle|back)`, `last_call_at`.
- `bird_offer` (age-gated availability) — `account_id`, `species_id`, `available_at`, `accepted_at`. Created by the tick when aviary age crosses the next interval and bird count < 7.

### 3.3 Events and notebook

- `interaction_event` (append-only, partitioned by `account_id`) — `id (uuid)`, `account_id`, `bird_id (nullable)`, `type (presence_ping|listen_in_start|listen_in_end|offer|settle|adopt_name)`, `payload (jsonb, bounded)`, `client_ts`, `server_ts`, `processed_by_tick_at (null until consumed)`. Inserted by clients via the state service; consumed in `server_ts`/`id` order by the tick.
- `notebook_entry` — `id`, `aviary_id`, `created_at`, `prose (text)`, `dedupe_key`. Read-only to users.
- `tick_cursor` — `account_id`, `last_processed_event_id`, `last_tick_at`. Guarantees in-order, exactly-once-ish consumption.

### 3.4 Social

- `invite` — `id`, `host_account_id`, `visitor_email_encrypted`, `visitor_email_hash`, `token_hash`, `created_at`, `expires_at (now+30d)`, `revoked_at`, `last_used_at`, `state (outstanding|active|expired|revoked)`.
- `visit_log_entry` — `id`, `host_account_id`, `visitor_email_encrypted`, `started_at`, `approx_duration_s`.

### 3.5 Snapshot (derived, not a table — serialized to KV + sent to client)

Bounded, kilobytes. Contains: per-bird `{id, species, name, mood, perch, call_grammar_params, next_call_window, active_transition}`, aviary `{local_phase, palette_phase, weather_event?, settle_state}`, server UTC ts, snapshot version. **Never** contains raw personality vector numbers — only the derived expressive parameters the client needs to render. This keeps the "never exposed numerically" rule true even at the wire level.

---

## 4. API surface

JSON over HTTPS. All authenticated routes require a session token. No personality-write route exists anywhere.

### 4.1 Auth / account (Auth service)

- `POST /auth/request-link {email}` → 204 always (no account enumeration). Rate-limited per email-hash.
- `POST /auth/consume {token}` → sets session, returns `{account_id, session_id}`. Invalidates link atomically.
- `GET /account` → settings view (sessions list, invites, visit-notify toggle, privacy link). Matter-of-fact voice.
- `POST /account/sessions/:id/revoke`.
- `POST /account/email-change {new_email}` → emails verification to new address; commit only on verify.
- `POST /account/export` → 202; generates snapshot JSON, emails download link to verified address.
- `POST /account/delete` → marks `pending_delete`. `POST /account/undelete` restores within 30d.
- `PUT /account/visit-notify {enabled}`.

### 4.2 Aviary state (State service)

- `GET /aviary/snapshot?tz=…` → current snapshot (from KV, fallback compute). Cheap, called on load, on visibility-change, after long frame-gap, and on a low-frequency keepalive. Also updates account `timezone` / `last_seen`.
- `POST /aviary/events` → append one or a small batch of interaction events to the log. Body validated and bounded. Returns 202. This is the **only** state-affecting write clients make.
  - `presence_ping`: sent only while the three-signal conjunction holds (client-gated, §5.4), carrying a short covered-interval so the server accrues presence-time without per-second spam.
  - `listen_in_start` / `listen_in_end {bird_id}`.
  - `offer {bird_id?, offer_type}` — server enforces cooldown; over-cooldown events are accepted but flagged no-op for drift.
  - `settle` / `settle_undo`.
  - `adopt_name {bird_id, name}` (also reachable via a dedicated rename route; rename never touches identity).
- `GET /aviary/narration` → current naturalist narration prose (server-generated; see §8) for screen readers; slow-cadence, ETag/If-None-Match so the client polls cheaply.
- `GET /aviary/notebook?cursor=…` → paginated read-only entries, newest first, infinite scroll-back.
- `POST /aviary/birds/:id/rename {name}`.
- `POST /aviary/bird-offer/:id/accept` → adopts the offered species as a new bird (only if count<7 and offer available).

### 4.3 Visit (read-only) flow

- Host: `POST /invites {visitor_email}` → emails one-time visit link. `GET /invites`, `POST /invites/:id/revoke`.
- Visitor: `GET /visit/:token/snapshot` → read-only snapshot of host aviary, **identical** to what host sees (no show-off rendering). No event endpoint is reachable with a visit token; visitor cannot write anything. On revoke/expire, next pull returns a matter-of-fact "visit no longer available." Server records a `visit_log_entry` and (if host opted in) a single visit notification; otherwise silent.

### 4.4 Voice enforcement at the API/copy layer

All user-facing strings are centralized in two namespaces: `naturalist.*` (aviary, notebook, narration, captions, offer prompts) and `system.*` (auth, account, sync errors, accessibility settings, unsupported browser, visit-unavailable). A lint rule (§11.4) fails the build if a `system.*` string is rendered in an aviary surface or vice-versa, and if any naturalist string contains an exclamation mark, "you", "welcome back", or gamification tokens.

---

## 5. Simulation engine design

### 5.1 The tick

A worker pass every ~60s per active aviary (an aviary is "active" if it has unprocessed events or stale mood timers; dormant aviaries tick lazily on next access to avoid scanning the whole table every minute). Each pass, for one account, in a single transaction:

1. Read `tick_cursor`; load unprocessed `interaction_event`s in `(server_ts, id)` order.
2. Compute **presence-time** accrued since last tick (from `presence_ping` covered-intervals, clamped so overlaps don't double-count).
3. Compute **drift deltas** per bird (§5.2) from presence-time + interaction signals. Apply as **additive** updates to `personality_vector`, clamp to [0,1], bump `version`. Never decrease on neglect (§5.3).
4. Run **mood transitions** (§5.5) using elapsed time, time-of-day (account timezone), weather events, interaction recency, and the bird's personality.
5. Run **bird-to-bird** coupling (§5.6).
6. Decide perches and call windows from mood+personality; write `mood_state`.
7. Maybe emit a **notebook entry** (§4.6 sparsity gate) and refresh **narration** input.
8. Maybe create an age-gated **bird offer**.
9. Recompute and write the **snapshot** to KV. Advance `tick_cursor`.

Idempotency: the cursor + per-event `processed_by_tick_at` makes reprocessing safe; a crashed tick re-runs from the cursor without double-applying drift (drift is keyed to the consumed event range, not wall-clock).

### 5.2 Drift function

Personality is a low-pass filter over presence-and-interaction signals. For trait `t` of bird `b` over a tick covering presence-time `P` (seconds) and interaction set `I`:

```
raw_signal_t = w_presence[t]*P_norm
             + w_listen[t]*listen_seconds_on_b_norm
             + w_offer_accept[t]*offers_accepted_near_b
             + w_offer_proximity[t]*offers_made_near_b
delta_t      = k_t * raw_signal_t                 # k_t small, per-trait
delta_t      = max(0, delta_t)                    # MONOTONIC: never negative (§5.3)
vector_t    += delta_t ; clamp [0,1]
```

- `P_norm` normalizes presence-seconds so a daily ~realistic watch session moves traits by a calibrated tiny amount.
- Weights per `interactions.md`: presence dominates; listen-in chiefly raises social_warmth + vocal_frequency of the focused bird; offer-accept raises curiosity; offering near a bird raises boldness; settle contributes none to drift (only ends presence cleanly). Plumage_saturation rises with sustained presence-time.
- `k_t` is tuned to the calibration targets (§5.3) via the golden test, not hand-set in prod code; values live in a single `drift_calibration` config object so calibration is one reviewable diff.

### 5.3 Monotonicity (the no-Tamagotchi engine rule)

`delta_t` is clamped to ≥0 at the engine level — there is no neglect term, no decay term, no negative weight anywhere in the drift path. A bird that is ignored simply receives small/zero deltas and stays where it is; it appears "quieter" only because mood/time-of-day reduce its *call frequency expression*, not because any trait dropped. This is enforced by a unit test asserting no code path can produce a negative personality delta, and by the golden test (§12) asserting a 2-week-absence trajectory is flat, not declining.

### 5.4 Presence accounting (the three-signal conjunction)

Client emits `presence_ping` only when **all three** hold simultaneously: `document.visibilityState === 'visible'` AND `document.hasFocus()` AND a `pointermove`/`keydown` occurred within the activity window (start 3 min). The client accrues local presence in covered-intervals and pings periodically (e.g. every 30s while present) with the interval it covers; it stops pinging the instant any condition fails. The server treats absence of pings as absence of presence — it never infers presence from connection liveness or tab-open. Visit sessions never emit presence pings (visitor attention must not drift host birds). Calibration of the activity window is a config value validated against a manual study during beta; bias toward longer because still-watching is the product.

### 5.5 Mood

Enum: `wary, content, curious, drowsy, alert`. Mood is a fast-timescale state machine per bird. Transition inputs: recent-interaction nudges (accepted offer → toward content/curious; alarm nearby → toward wary), time-of-day (dusk→drowsy/settled, early morning→alert), weather (rain→dampened vocal expression briefly; wind→some alert, some wary), and the bird's personality as a bias (high boldness resists wary; high curiosity favors curious). Mood **persists across sessions**: session-end mood is session-start mood, modulated only by what the tick did meanwhile (a drowsy-at-dusk bird is likely settled by morning). Mood is expressed to the client as the rendering envelope (perch tendency, idle-motion style, call density), never as a label.

### 5.6 Bird-to-bird coupling

Within a tick, a calling bird raises the near-term call probability of nearby high-vocal-frequency birds (chorus emergence); a wary mood has a small contagion radius; high-social-warmth birds bias toward perching near others and calling back. These are bounded couplings computed in the tick so chorus events are emergent, not scripted.

### 5.7 Call-grammar runtime

Each species has a motif library (a small set of pitch/rhythm motifs). Each bird's call signature = species motifs + personality/seed-shaped timing and pitch transforms, kept stable across mood and drift so the bird stays recognizable by ear (the cap-of-7 affordance). The server hands the client per-bird grammar parameters + next-call windows; the **client** synthesizes the actual call (§7) so every utterance varies. Captions (§9) are generated from the same grammar params so caption text always matches what was played.

---

## 6. Sync model

### 6.1 Single canonical state

There is exactly one writer of personality (the tick) and one canonical snapshot per account. Every device `GET`s the same snapshot and renders it; there is no client-to-client sync, no client-side authoritative state, nothing to merge. Multi-device "sync" is therefore a *property*, not a feature: laptop and phone read the same row.

### 6.2 No last-write-wins

Clients never submit absolute personality values; they submit interaction events into the append-only log. The tick applies additive, server-authored deltas in event-log order. A late-arriving event from an older client read cannot overwrite drift — it is just another appended event consumed in order. This makes the "morning drift silently deleted by a stale lunch write" failure unreachable by construction.

### 6.3 Event log ordering & conflict prevention

- Events carry `client_ts` (advisory) and are stamped `server_ts` on append; the tick consumes strictly by `(server_ts, id)`. Per-account partitioning guarantees per-aviary order without global ordering cost.
- v1 starts with a **Postgres partitioned table** for the log (simplest correct option; ordered reads, transactional cursor). If volume warrants, migrate to a partitioned stream (Kafka/Kinesis, account-UUID key) behind the same State-service interface — the client and tick contracts don't change.
- Snapshot writes are last-tick-wins *for the snapshot only* (it's derived, regenerable); canonical personality is never last-write-wins.

### 6.4 Interpolation & freshness

Client interpolates bird motion between snapshots (perch A→B rendered as a smooth move, never a teleport). It pulls fresh snapshots on visibility-change, after long render-frame gaps (laptop resume), and on a low-frequency keepalive. Snapshots are kilobytes, so polling is cheap and a websocket is **not** required for v1 (revisit only if perceived latency demands it).

### 6.5 Sync/error surfaces

Magic-link replay, mid-write session timeout, and outages surface in **matter-of-fact** voice ("We couldn't sign you in…", "Your session timed out…", "Something went wrong loading your aviary…"). These are the named exception to naturalist voice and live in `system.*`.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

Single horizontal scene, no pan/scroll/zoom, responsive so all birds stay in frame at any viewport (compress spacing on narrow, widen on desktop; never crop a bird). Layers: background sky/foliage, mid-plane perches+birds (front/middle/back zones map to scale + y-position + subtle blur for depth), occasional foreground branch/leaf. Subtle parallax only. Renderer: Canvas2D or lightweight WebGL (PixiJS-class) chosen for the 60fps-on-old-laptop + 2MB budget; birds are small SVG/procedural sprites with rigged parts (head, body, wings) for micro-motion. **No** entry animation, spinner, or fade-from-static.

### 7.2 First frame already in motion

On load: HTML ships with a CDN-edge-seeded snapshot (§10) so the client can place birds at their *current* positions/motions immediately and start rendering mid-action — a bird mid-preen, another calling, a leaf drifting. The "loading" state, when a snapshot must still be fetched, is a **quiet field** (soft sky, one or two faint motion cues), never a spinner. The empty-aviary state (post-adoption, pre-first-bird) reuses that quiet field, then the first bird soft-flies in to its perch; the user never sees an empty aviary again.

### 7.3 Idle micro-motion (mood-shaped)

Continuous per-bird micro-motion within the server-set mood envelope: preen, scan, head-tilt toward sounds, weight-shuffle. Wary→further back + more scanning; content→preening; curious→tilts toward sounds/leaves; drowsy→low + fluffed. Motion is procedurally varied (phase/seed offsets) so it never reads as a loop. The user must be able to read mood from motion **without any label**.

### 7.4 Transitions & interpolation

Perch changes and greetings are interpolated from snapshot deltas. Return-greeting (§7.6) and offer reactions are short client animations triggered by snapshot/event acknowledgement. Settle is a client-driven slow lighting shift to evening with the 5-second undo (any aviary click within 5s reverses it).

### 7.5 Reduced-motion as a designed surface

When `prefers-reduced-motion` or the opt-in is set: micro-motion → slow cross-fades between still poses; flight → cross-fade between perches (no path); ambient leaf/feather drift removed; day→evening color shifts retained but slowed. Calls, drift, mood, notebook all continue unchanged. This is a distinct, calmer aesthetic, **not** "animations off" — verified by a dedicated visual review, not just a CSS toggle.

### 7.6 Return-greeting implementation

On session start (fresh nav, tab-return, or return-after-absence) the client requests a snapshot that includes an **absence-length** signal (server computes from last presence) and a **greeter selection** (which bird greets first today — honoring boldness + mood; bolder greets first, warier later or not at all). Greeting *form* scales with absence (coffee-break → a glance; two days → a closer approach / longer call) and varies procedurally inside those rules (real variation, not 3 canned variants). Multiple greeters **stagger** by a small randomized offset — never a unison chorus on cue. There is **no** textual welcome anywhere.

### 7.7 Top bar & chrome

Thin top bar above the scene with exactly: account/settings, accessibility settings, field notebook, offer affordance. No chrome inside the scene (no badges, tooltips, overlay icons, inline labels). Top bar fades to near-transparent after a few seconds of cursor stillness; returns on cursor/keyboard activity.

### 7.8 Ambient ornaments

Leaves/feathers drift at slow random client-side cadence — pure rendering ornaments, **no** server/per-leaf state. They keep the scene alive between bird actions.

---

## 8. Audio pipeline

### 8.1 Procedural synthesis (non-negotiable)

Calls are synthesized client-side via WebAudio from the per-bird grammar params + seed; **no recorded audio anywhere**, no fallback to recorded audio. Each utterance is generated fresh (oscillator/wavetable + envelope + formant-ish filtering per species motif), so the same call is never heard twice identically. This is both a bundle-budget necessity (<2MB) and the only way the chorus mechanic works (layered recordings phase-cancel; real-time procedural calls mix into a true chorus).

### 8.2 Chorus mixing

A shared audio graph mixes all active birds. Chorus events (from §5.6 coupling) are rendered as genuinely simultaneous, slightly-detuned/offset procedural calls. Per-bird gain nodes feed a master bus with gentle limiting to avoid clipping when several birds call at once.

### 8.3 Listen-in mix

Engaging listen-in on a bird **slowly** ramps that bird's gain up and the others down to ambient (never silent — re-balance, not mute). Disengage (click bird again / focus another / click empty space / move keyboard focus away) ramps back to ambient with the same slow envelope. No hard cuts — it must feel like leaning in to listen, not switching a track.

### 8.4 Resource discipline (perf)

A single bounded `AudioContext`; a pool of reused buffers/nodes; no per-call allocation that isn't returned to the pool. This is the audio half of the "no memory growth over 30 min" CI test (§11.3).

### 8.5 WebAudio fallback

If WebAudio is unavailable or denied: the aviary runs in **graceful silence with captions on by default** (§9). No recorded-audio path is ever shipped.

---

## 9. Accessibility surfaces (first-class, day-one)

### 9.1 Screen-reader narration

Running **naturalist prose** (same voice as the notebook), generated server-side from the same canonical state the visual reads, served via `GET /aviary/narration` and surfaced in a polite `aria-live` region. Cadence ~1 update / 30–60s at idle; user-initiated events (return-greeting, offer reaction, settle) get a small priority bump and prompt narration, still written as observations ("a warbler perches on the high branch, calling softly") never as state lists ("Wren mood: content"). High-frequency narration is explicitly avoided so the SR queue isn't overwhelmed.

### 9.2 Captions for calls

Opt-in. Short naturalist prose describing each call in its current mood ("a soft three-note rise"), generated at runtime **from the same grammar params** that drove synthesis so the caption matches what played. Rendered as small text near the calling bird, fading with the call. On by default in the WebAudio-fallback silent mode.

### 9.3 Keyboard navigation & focus

Tab through top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds; Enter triggers listen-in on the focused bird; Escape exits listen-in; offer opens via a top-bar shortcut and is fully keyboard-navigable; settle reachable from the top bar. Focus indicator: a soft high-contrast outline legible against both bright and dim aviary states.

### 9.4 Contrast

All user copy (top-bar labels, settings, account/error surfaces, captions, visually-displayed narration) passes WCAG AA minimum; exact ratios from the design system. The scene itself carries no copy except the top bar.

### 9.5 Stance

Accessibility is a **designed** surface, not a stripped fallback, and ships **with** v1 (a reduced-motion mode arriving as a "v1.1 fix" is treated as a launch defect). Acceptance includes SR walkthroughs and reduced-motion visual review, not just automated axe checks.

---

## 10. Performance budgets & observability

### 10.1 Budgets (acceptance criteria)

- **Initial JS bundle ≤ 2MB gzipped at first paint.** Aggressive code-splitting: account settings, accessibility settings, and the visit/invite flow are lazy chunks; the core aviary + engine-client + audio are the only first-paint code. Bird visuals procedural/small-SVG. Budget enforced in CI (size-limit) — build fails over budget.
- **Time-to-first-bird < 500ms** on mid-tier mobile / 4G. Achieved via: edge-seeded snapshot delivered with the HTML (no round-trip to draw the first bird), a render path that draws birds before non-critical assets, and synth audio warmup deferred until after first paint. Verified by synthetic checks (§10.3).
- **60fps idle motion on a 5-year-old mid-range laptop**, sustained over a 30-min session (runtime budget, not just launch).
- **No memory growth over 30 min** — CI test (§11.3). Reused audio buffers, no leaked notebook DOM/refs after scroll-out, bounded workers/audio contexts.

### 10.2 What we deliberately don't measure

No per-bird, per-account interaction telemetry feeds aggregate analytics — ever. The simulation DB is never read by the analytics warehouse; telemetry pipelines never touch per-account simulation data; ML (if ever) never receives per-bird fields. This is an architectural boundary, enforced at metric-definition and data-access level, not a policy footnote.

### 10.3 Observability (aggregate only)

Synthetic browser fleet runs the aviary on a schedule from common geographies (load timing, first-bird-render, frame timing, audio-context errors). Aggregate-only RUM: page-load, first-bird-render, render-frame timing, audio errors, tick latency — **no per-account dimension**, anonymized session-duration histograms only. Alarm: simulation-tick latency **p99 > 5s**. Privacy boundary is honored in each metric's definition.

### 10.4 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers get a matter-of-fact unsupported-browser surface. No legacy compat shims (they'd cost the bundle budget).

---

## 11. Engineering structure & guardrails

### 11.1 Repos / packages

- `client/` — rendering, audio, presence detection, a11y surfaces, copy namespaces.
- `engine/` — pure, deterministic simulation library (drift, mood, coupling, call-grammar param derivation, notebook/narration generators). **Framework-free and seedable** so it runs identically in the tick worker and in tests.
- `services/auth`, `services/state`, `services/tick` — thin app tiers wrapping `engine` + datastores.
- `shared/` — snapshot/event schemas, copy namespaces, UUID/PII helpers.

### 11.2 Determinism

The `engine` is deterministic given (state, events, seed, clock). This makes drift calibration testable (golden trajectories, §12) and makes the tick replayable/idempotent.

### 11.3 Memory CI test

Headless browser drives a 30-min accelerated session (sped-up clock, real audio graph + render loop); asserts heap and audio-node counts are bounded at the end. Fails the build on growth.

### 11.4 Anti-leak guards (encode the non-goals)

A CI lint/check that fails on:
- any `naturalist.*` string containing `!`, "you", "welcome back", "streak", "level", "badge", "achievement", "score", or a "days visited" pattern;
- a `system.*` string rendered in an aviary surface (or naturalist in an error surface);
- any endpoint/handler that writes a personality vector outside the tick worker;
- any code path producing a negative personality delta;
- any telemetry event carrying a per-account/per-bird dimension;
- any route exposing personality numbers to the client/snapshot/ARIA;
- introduction of a spinner component on the aviary load path.

These guards exist because the PRD repeatedly names these as the temptations a well-meaning contributor will reach for; making them build failures is cheaper than re-litigating them.

---

## 12. Drift calibration & test harness

- **Golden trajectories**: deterministic `engine` simulations of canonical user behaviors — "regular daily 10-min watcher", "weekend-only", "two-week-absence-then-return", "heavy listen-in on one bird". Assertions: instrument-detectable trait change by ~day 7, user-perceptible by ~day 21, **flat (never declining)** across the absence window, focused bird's social_warmth/vocal_frequency outpaces peers under heavy listen-in.
- Calibration constants (`k_t`, weights, normalization, activity-window, cooldown, tick cadence) live in one `drift_calibration` config so tuning is a single reviewable diff; the golden tests gate changes to it.
- Beta calibration of the presence activity-window against a small manual watch study before GA.

---

## 13. Rollout

1. **Foundations**: auth (magic link, synthetic UUID, sessions), account record + encrypted email, datastores, snapshot KV, edge-seeded HTML. Anti-leak guards + voice namespaces in CI from commit one.
2. **Engine core**: deterministic `engine` lib + golden tests (drift, mood, coupling, call-grammar params) before any UI polish — calibration is the riskiest thing, de-risk it first.
3. **Tick worker**: event-log consumption, additive deltas, snapshot generation, idempotent cursor.
4. **Client aviary**: render pipeline, mood-shaped idle motion, interpolation, first-frame-in-motion, top bar + fade, presence detection.
5. **Audio**: procedural synthesis, chorus, listen-in mix, silence+captions fallback; wire the 30-min memory CI test.
6. **Interactions**: return-greeting, offer (+cooldown), settle (+undo), notebook (sparsity-gated), narration.
7. **Accessibility pass**: SR narration cadence, reduced-motion designed surface, captions, keyboard nav, contrast — gated as v1 launch criteria.
8. **Social**: invite/visit read-only, revocation, visit log, opt-in notify.
9. **Account lifecycle**: export, email change, soft/hard delete, privacy-policy surface.
10. **Perf hardening + synthetic monitoring + tick-latency alarm**, then GA.

**Bird-count ramp**: start every aviary at 2; age-gated third-bird offer fires at the first age interval (target a few months) and subsequent intervals up to the cap of 7, **never** tied to visit count/score/payment. Instrument the *offer cadence* (aggregate, anonymized) only to verify pacing — not per-account.

**Instrument from day one** (aggregate-only): first-bird-render time, frame timing, audio errors, tick latency, bundle size in CI. Never per-bird/per-account.

---

## 14. Risks & mitigations

- **Drift miscalibration** (Tamagotchi-fast or screensaver-slow): the central risk. Mitigation: deterministic golden trajectories as the gate; calibration constants isolated in one config; monotonicity enforced by test so "punish absence" can't sneak in.
- **Sync correctness / lost drift**: mitigated structurally — single server writer, additive deltas, in-order event-log consumption, no last-write-wins path exists. Test: replay out-of-order/stale events and assert no drift loss.
- **Audio uncanniness / phase-cancel chorus**: mitigated by procedural-only synthesis, per-utterance variation, detuned chorus offsets, and listen-in slow ramps. Risk if synthesis sounds synthetic — budget design-time for motif sound-design and per-species recognizability testing (can a listener tell Pip from Wren by ear after two weeks?).
- **Accessibility regression / flattening**: SR narration degenerating into state lists, reduced-motion shipping as "animations off". Mitigation: voice continuity tests on narration/captions, reduced-motion as a reviewed designed surface, a11y as launch criteria not v1.1.
- **"Notice never announce" erosion**: a toast/banner/streak slips in. Mitigation: the §11.4 anti-leak CI guards make these build failures.
- **PII leakage via email-as-identifier**: mitigated by synthetic-UUID-everywhere + email-only-on-account-record + guard against per-account telemetry dimensions; the `email_hash` is HMAC-keyed and never logged.
- **First-bird >500ms / bundle >2MB**: mitigated by edge-seeded snapshot, aggressive code-splitting, procedural assets, size-limit + synthetic perf gates in CI.
- **Tick scalability**: lazy-tick dormant aviaries, shard the worker by account-UUID hash, p99 latency alarm; migrate the event log to a partitioned stream behind the same interface if Postgres-partitioning hits limits.
- **Memory growth over long sessions**: mitigated by pooled audio buffers, bounded contexts/workers, notebook DOM cleanup, enforced by the 30-min CI test.

---

## 15. Open items deferred to specs (named, not invented here)

- Exact palette colors and per-surface contrast ratios — design-system spec (visual designer).
- Final perch geometry / responsive breakpoints — rendering spec.
- Final motif libraries per species and sound-design parameters — audio spec.
- Final calibration constants — output of §12 golden tests + beta study.

These are deliberately left to their owning specs; this plan defines the interfaces and acceptance criteria they must satisfy, not their internal values.
