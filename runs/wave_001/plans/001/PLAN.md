# Pocket Aviary — v1 Implementation Plan

> Audience: a frontier engineering team executing v1 without further clarification.
> This plan interprets the PRD into executable decisions. Where the PRD leaves a
> value open, this plan makes a defensible call and flags it in **§19 Assumptions**.
> The affective constraints in the PRD are treated as engineering constraints, not
> aspirations: each one is converted into a rule that can be reviewed, tested, or
> architecturally enforced. The single most important property of this plan is that
> **the product's charm is not separable from its correctness** — a canned greeting,
> a leaked numeric trait, or a "harmless" toast is a P1 bug, not a polish item.

---

## 1. How to read this plan

The PRD's five design principles ("feels alive," "notice never announce," "charm
from specificity," "restraint over richness," "naturalist vs. matter-of-fact voice")
and its non-goals are not a preamble — they are the acceptance criteria. §3 turns
each into a concrete engineering rule with an enforcement mechanism. Every later
section is expected to satisfy §3. If a later decision appears to conflict with §3,
§3 wins and the decision is wrong.

Section map: Scope (§2) → Guardrails (§3) → Architecture (§4) → Data model (§5) →
API (§6) → Simulation engine (§7) → Sync (§8) → Rendering (§9) → Audio (§10) →
Accessibility (§11) → Voice system (§12) → Performance & observability (§13) →
Privacy & data boundaries (§14) → Rollout (§15) → Risks (§16) → Testing &
calibration (§17) → Work breakdown (§18) → Assumptions (§19).

---

## 2. Scope

### 2.1 In scope for v1

- Single-user accounts, email magic-link sign-in, per-device revocable sessions,
  email change with verification, account export (JSON), soft-delete (30d) → hard-delete.
- One canonical aviary per account. Two starter birds; cap of seven.
- Server-side simulation tick (canonical state) advancing whether or not a client is connected.
- Bird engine: hidden personality vector (5 traits), mood FSM, procedural call grammar,
  monotonic-toward-expressive drift, bird-to-bird interaction.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool),
  settle (with 5s undo), presence accounting, field notebook (read-only, sparse).
- Aviary scene: one horizontal screen, three perch zones, local-time day/night, rare
  ambient weather, ambient micro-motion, thin auto-fading top bar, motion-already-in-progress load.
- Multi-device sync as an architectural property (server-canonical, client renders snapshots).
- Visit feature: per-invite, read-only ambient, revocable, expiring (30d), silent by default,
  on-demand visit log, opt-in visit notifications.
- Accessibility as first-class designed surfaces: naturalist screen-reader narration,
  call captions, full keyboard navigation, WCAG AA contrast, reduced-motion mode.
- Performance budgets enforced in CI + observability (synthetic + aggregate-only RUM).

### 2.2 Out of scope (non-goals respected — see `non_goals.md`)

Hard exclusions, each enforced architecturally so it cannot leak back in:

- **Native apps** — web-only. Data model and protocols are not shaped for a native client.
- **Gamification** — no achievements, levels, scores, badges, XP, ranks, tiers, streaks,
  "birds adopted: N," green-dot calendars, milestone celebrations, or any visit-frequency
  surface. We **do not compute** the underlying metrics (§3.2).
- **Tamagotchi mechanics** — no death, hunger, distress, decay meter. Drift is monotonic
  toward expressive; neglect produces ambient quietness only (§3.3, §7.3).
- **Social-network surfaces** — no profiles, follows, public feed, discovery, friend-of-friend,
  comments, leaderboards, co-presence, show-off rendering. Only the single read-only visit affordance.
- **Notification surface** — no push/email/in-product pings about the aviary. The only
  optional, off-by-default notification is per-visit "a friend visited," opt-in in settings.
- **Numeric exposure of personality** — no stats panel, debug view, "how's my bird" surface,
  toggle, or tier. Ever (§3.4).
- **Recorded-audio path** — calls are procedural only; the WebAudio-unavailable fallback is
  silence + captions, never recorded audio (§3.5, §10.5).
- Payments, shared/team aviaries, customizable scenes, multi-aviary accounts, species rarity,
  perch placement controls, editable notebook, panning/scroll/zoom.

---

## 3. Guardrails — design principles as enforceable engineering rules

This is the spine. Each rule states the mechanism that makes a violation either
impossible, caught in CI, or caught in review. PRs touching the listed surfaces must
cite the relevant guardrail.

| # | Rule (from PRD) | Engineering enforcement |
|---|---|---|
| G1 | **Feels alive, not robotic.** First frame is mid-motion; no spinner/entry animation; calls never repeat identically; idle motion is procedural & mood-keyed. | Render bootstrap draws birds from snapshot before non-critical assets (§9.1). "No two identical calls" and "no cycle-locked idle loop" are automated tests (§17.4). A stock spinner anywhere is a blocked pattern in lint/review. |
| G2 | **Notice, never announce.** No welcome toast/banner/modal; the bird greeting is the entire welcome. | **No toast/notification UI component exists in the codebase.** There is no notification service for the aviary. A lint rule blocks introducing one. The return-greeting is implemented as bird behavior only (§7.5, §9.3). |
| G3 | **Charm from specificity.** Naturalist, bird- and moment-specific prose; never gamification language. | All player-facing prose flows through `sim-core/prose` (§12), which has no template for generic state phrasing ("your bird is happier"). Prose snapshot tests assert specificity and ban a denylist of gamification phrases. |
| G4 | **Restraint over richness.** 2 birds start, 7 cap; one screen; no in-scene chrome; calm palette. | Bird cap enforced server-side (adoption rejects an 8th). Scene renderer has no API for in-scene buttons/badges; chrome lives only in the top-bar layer (§9.5). Palette tokens exclude saturated accents. |
| G5 | **Voice split.** Naturalist for product surfaces; matter-of-fact for system surfaces (auth, settings, errors, sync, a11y settings). | Two prose namespaces: `prose.naturalist.*` and `copy.system.*`. System surfaces import only `copy.system`. Review gate on any naturalist string in an error/auth/settings path (§12). |
| G6 | **Presence is the conjunction of three signals.** visible AND focused AND recent pointer/key activity. | Single client `PresenceDetector` (§7.2) is the only presence source; it emits a ping only when all three hold. No other code path may synthesize presence. Unit tests cover each pairwise-only case → no ping. |
| G7 | **Personality is never exposed numerically.** | Snapshot DTO (§5.4, §6.2) contains only *expressions* (perch, mood enum, render buckets, call params), never raw trait scalars. A schema test fails if any trait field name appears in any client-facing response. Export (§6.6) emits coarse render buckets, not raw scalars (§19-A7). |
| G8 | **Server is the only writer of personality; no last-write-wins.** | Clients write append-only events only; the tick is the sole personality writer; drift applied as additive deltas in event-log order (§7.3, §8.3). No client code path mutates a vector. Enforced by store permissions + code review + a test that the client API has no vector-write endpoint. |
| G9 | **Email is PII; synthetic UUID everywhere else.** | `account_id` UUID is the only identifier in logs, keys, telemetry, inter-service messages. Email is encrypted on one column. A log/telemetry schema linter denies any field resembling an email (§14.1). |
| G10 | **Per-bird/per-account interaction data is never aggregated.** | Simulation DB is network- and credential-isolated from the analytics warehouse; no ETL path exists between them. Telemetry events are defined with no per-account dimension (§14.2). |
| G11 | **Drift is monotonic toward expressive.** | The drift apply function has no negative-delta path for absence; absence simply adds nothing. Property test: no input sequence containing absence reduces any trait (§17.3). |
| G12 | **Accessibility ships with v1, as designed surfaces, in the product voice.** | Narration/captions/reduced-motion are v1 launch-gating, in the same CI as the visual path; narration uses `prose.naturalist` (G5). A "reduced-motion is a fallback" implementation is a rejected design (§11.3). |

---

## 4. System architecture

### 4.1 Service shape (client/server split, render boundary)

Three deployables plus one shared library:

1. **`api`** — stateless HTTP service. Auth, snapshot reads, event writes, settings,
   export/delete, visit invite/read, account management. Horizontally scalable;
   holds no per-session simulation state.
2. **`sim`** — the simulation worker. Owns the tick (§7.1). Sole writer of personality
   vectors, moods, and notebook entries. Reads the append-only event log; writes canonical
   aviary state. Internally partitioned by `account_id`.
3. **`web`** — the browser client. Pulls snapshots, interpolates, renders the scene,
   synthesizes audio, runs the presence detector, and submits interaction events. Renders
   only; never authoritative for personality.
4. **`sim-core`** (shared TypeScript package) — pure, deterministic functions used by
   **both** `sim` and `web`: the drift function, mood FSM, call-grammar runtime, day/night
   and weather derivation, and the naturalist prose generator (notebook + narration +
   captions). Sharing this code is a deliberate call (§19-A1): it is what keeps captions
   matching the audio actually played, narration matching the visual state, and a visitor's
   view identical to the host's — because every surface derives from one implementation.

**Render-pipeline boundary (the load-bearing split):** the **server decides slow truth**
(personality, mood, day phase, per-bird "call disposition," perch intent, weather windows);
the **client decides fast presentation** (sub-second call onsets, interpolation between
perches, idle micro-motion phase, ambient leaf/feather drift). The client never invents
slow truth; the server never dictates per-frame motion. Anything that must agree across
devices/visitor (call stream, captions, narration) is derived deterministically in
`sim-core` from server-provided seeds + wall-clock, so independent clients converge without
a sync channel (§7.6, §8.2).

### 4.2 Transport: snapshot polling, not push

The PRD describes clients that **pull** snapshots on visibility change, on long render-frame
gaps (suspend recovery), and on a low-frequency keepalive. The tick is ~1/min and clients
interpolate, so there is no need for server push. **Decision:** HTTP snapshot polling, no
WebSocket/SSE in v1 (§19-A2). This is simpler, cheaper, CDN-friendly, and matches "clients
never tick; they pull snapshots and interpolate." Snapshots are kilobytes.

### 4.3 Storage topology

- **Simulation store** (Postgres): accounts, aviaries, birds, events (append-only), notebook,
  sessions, magic links, visit invites/sessions, tick records. Strong consistency; the
  canonical record. Isolated from analytics (G10).
- **Object store**: generated account-export JSON blobs (short-lived, signed download URLs).
- **CDN/edge**: static `web` bundle + the initial HTML shell with an inlined minimal snapshot
  for the first paint (§13.2).
- **Analytics warehouse**: aggregate operational telemetry only. No network route to the
  simulation store (G10, §14.2).

### 4.4 Tech choices (defensible; see §19)

- **Language:** TypeScript end-to-end. Server runtime Node.js (LTS). This is what makes the
  shared `sim-core` real (§19-A1).
- **Chrome UI (top bar, settings, notebook, offer tray, auth, visit):** Preact + signals
  (~4–5KB) — small enough for the bundle budget, expressive enough for the chrome (§19-A3).
- **Scene renderer:** custom Canvas2D render loop behind a `SceneRenderer` interface, so a
  WebGL backend can replace it if instruments demand more headroom. Canvas2D is sufficient for
  ≤7 birds + a few parallax planes and avoids shader/bundle weight (§19-A4).
- **Audio:** WebAudio with an `AudioWorklet` synthesizer (§10).
- **DB:** Postgres. **Auth tokens:** opaque random tokens, hashed at rest, in a sessions table.

---

## 5. Data model

All identifiers are UUIDv4 generated server-side. `account_id` is the only cross-system
reference (G9). Personality scalars live only in `bird.personality` and are written only by
`sim` (G8). Times are UTC; day/night derives from a stored IANA timezone (§7.7).

### 5.1 Account & auth

```
account(
  id              uuid pk,
  email_enc       bytea,                  -- encrypted; the ONLY place email is stored (G9)
  email_verified  bool,
  tz              text,                    -- IANA tz for day/night anchoring
  settings        jsonb,                   -- reduced_motion, captions, audio_on, visit_notify
  privacy_ack_at  timestamptz,
  state           text,                    -- 'active' | 'soft_deleted'
  soft_deleted_at timestamptz null,        -- hard-delete sweep at +30d
  created_at      timestamptz
)

session(
  id           uuid pk,
  account_id   uuid fk,
  token_hash   bytea,                       -- per-device, revocable
  device_label text,                        -- coarse UA-derived label for the session list
  created_at   timestamptz,
  last_seen_at timestamptz,
  revoked_at   timestamptz null
)

magic_link(
  id          uuid pk,
  account_id  uuid null,                    -- null for first-signup-by-email flow
  email_enc   bytea,
  token_hash  bytea,
  purpose     text,                         -- 'signin' | 'email_change' | 'visit'
  created_at  timestamptz,
  expires_at  timestamptz,                  -- +15m for signin/email-change
  consumed_at timestamptz null              -- single-use; set on first consumption
)
```

### 5.2 Aviary & birds

```
aviary(
  id           uuid pk,
  account_id   uuid fk unique,              -- one aviary per account
  seed         bigint,                      -- deterministic ambient (weather, greeting variation)
  created_at   timestamptz,                 -- the AGE clock that gates new-bird offers (§7.8)
  last_tick_at timestamptz,                 -- last canonical advance (drives lazy catch-up §7.1)
  settled      bool,                        -- user-triggered evening lighting state (§7.9)
  settled_at   timestamptz null
)

bird(
  id            uuid pk,                     -- STABLE for account lifetime; never reissued (G..,§5.6)
  aviary_id     uuid fk,
  species       text,                        -- one of ~6 species (§7.10)
  name          text,                        -- user-assigned, renameable, no engine effect
  personality   jsonb,                       -- {boldness, social_warmth, vocal_frequency,
                                             --  plumage_saturation, curiosity} scalars in [0,1]
  drift_accum   jsonb,                        -- per-trait positive evidence accumulators (§7.3)
  mood          text,                         -- enum (§7.4)
  mood_since    timestamptz,
  perch         text,                         -- 'front'|'middle'|'back' (intent; client interpolates)
  last_call_at  timestamptz,
  created_at    timestamptz
)
```

### 5.3 Events (append-only) & notebook

```
interaction_event(
  id            bigserial pk,                -- monotonic; defines processing order (§8.3)
  account_id    uuid fk,
  aviary_id     uuid fk,
  bird_id       uuid null,                   -- null = aviary-wide (e.g., settle, song offer)
  type          text,                        -- 'presence_ping'|'listen_in_start'|'listen_in_end'
                                             --  |'offer'|'settle'|'settle_undo'|'adopt'|'rename'
  payload       jsonb,                        -- e.g., offer={item}, listen_in={duration_ms}
  client_ts     timestamptz,
  server_ts     timestamptz,                  -- authoritative ordering tiebreak
  consumed_tick bigint null                   -- set when the tick integrates it; idempotency guard
)

notebook_entry(
  id           uuid pk,
  aviary_id    uuid fk,
  created_at   timestamptz,
  prose        text,                          -- naturalist, generated by sim-core/prose (§12)
  trigger      text,                          -- INTERNAL only (e.g., 'first_greeter_change'); never shown
  cooldown_key text                           -- sparsity bookkeeping (§7.11)
)
```

### 5.4 Snapshot DTO (what the client receives — never raw traits, G7)

```
Snapshot {
  aviary: { day_phase: float(0..1), light: 'morning'|'midday'|'evening'|'night',
            settled: bool, weather: 'clear'|'rain'|'wind', tick_seq: int, server_now: iso },
  birds: [ {
    id, species, name,
    mood,                          // enum only
    perch,                         // 'front'|'middle'|'back'
    plumage_level: 0..N,           // COARSE render bucket derived from saturation, not the scalar
    call: { motif_weights, rate_hz, pitch_center, jitter_seed },  // disposition, not a recording
    greeting: null | { kind, intensity, stagger_ms }              // present only on session-start snapshot
  } ],
  seed: bigint                     // for deterministic client-side call scheduling & ambient
}
```

### 5.5 Visit

```
visit_invite(
  id               uuid pk,
  host_account_id  uuid fk,
  visitor_email_enc bytea,
  token_hash       bytea,
  created_at       timestamptz,
  expires_at       timestamptz,    -- +30d
  revoked_at       timestamptz null,
  last_used_at     timestamptz null
)
visit_session(
  id          uuid pk,
  invite_id   uuid fk,
  started_at  timestamptz,
  ended_at    timestamptz null,
  approx_duration_s int null       -- for the host's on-demand visit log only
)
```

### 5.6 Identity invariants (engine-level, cross-migration)

- A `bird.id` is allocated once and is immutable. No code path ("reset," "regenerate,"
  "upgrade," species-pool migration) may reassign, recreate, or swap a bird id. Renames,
  syncs, and migrations preserve `(id, personality, drift_accum)`.
- The personality vector is **stored**, never recomputed from event history at runtime. A
  migration that would rebuild vectors from logs is forbidden; migrations carry vectors forward
  verbatim. This is the foundation of perceived drift validity (loss = deleting the relationship).

---

## 6. API surface

REST/JSON over HTTPS. All authenticated routes require a valid session token; all responses
obey G7 (no raw traits) and G5 (system routes use matter-of-fact copy). Idempotency keys on
event writes.

### 6.1 Auth & account
- `POST /auth/magic-link` `{email}` → always 202 (no account-existence oracle). Sends signin
  or signup link. Rate-limited per-email.
- `GET /auth/consume?token=…` → validates (≤15m, unconsumed), marks consumed, issues a
  session token, redirects into the aviary. On failure: matter-of-fact surface (G5).
- `POST /account/email-change` `{new_email}` → sends verification to new address; old email
  works until verified.
- `GET /account/sessions` / `POST /account/sessions/{id}/revoke` → device session list + revoke.
- `GET /account/export` → enqueues snapshot, emails signed link (§6.6).
- `POST /account/delete` (soft) / `POST /account/restore` (within 30d).
- `GET/PATCH /account/settings` → reduced_motion, captions, audio_on, visit_notify, tz.

### 6.2 Aviary state (pull)
- `GET /aviary/snapshot` → triggers lazy tick catch-up (§7.1) then returns the `Snapshot`
  (§5.4). Called on load, on visibility change, on long frame-gap, and on low-frequency
  keepalive. The **session-start** snapshot includes per-bird `greeting` directives (§7.5).
- Response is small (KB), `Cache-Control: no-store`, carries `tick_seq` for client staleness checks.

### 6.3 Interaction events (write)
- `POST /aviary/events` `{events:[…]}` — append-only; the only mutation path for users.
  Accepts batched `presence_ping`, `listen_in_start/end`, `offer`, `settle`, `settle_undo`,
  `rename`, `adopt-name`. Returns 202. **No endpoint accepts a personality value** (G8).
  Offer enforces the per-bird cooldown server-side (§7.12); rename updates `bird.name` only.

### 6.4 Adoption / new birds
- `POST /aviary/birds/{id}/name` — rename (any time).
- `GET /aviary/offers/new-bird` — returns a pending new-species offer **iff** aviary age has
  crossed the next threshold (§7.8). Gated by age only; never by visits/interactions/payment.
- `POST /aviary/offers/new-bird/accept` `{name}` — adds the bird (rejects if at cap of 7).

### 6.5 Visit flow
- `POST /visits/invite` `{visitor_email}` → creates `visit_invite`, emails one-time link.
- `GET /visit/consume?token=…` → opens a read-only visitor session; starts `visit_session`.
- `GET /visit/snapshot?token=…` → returns the host's current `Snapshot` **unmodified** (no
  show-off rendering, G4/non-goals). Visitor events are **not** recorded (no drift). On
  revoked/expired/used token → matter-of-fact "visit no longer available" (G5).
- `GET /visits` (host) → visit log (visitor email, date, approx duration, outstanding invites).
- `POST /visits/{id}/revoke` → immediate; next visitor snapshot pull returns the unavailable surface.

### 6.6 Export contents
JSON: birds (id, species, name, current mood, **coarse plumage bucket** not raw scalar — §19-A7),
notebook entries, account settings. Generated on demand, emailed as a signed link to the
verified address.

---

## 7. Simulation engine

### 7.1 The tick: deterministic, time-composable, lazy-catch-up-safe

The canonical aviary advances on a server tick (~60s cadence, tunable via server flag —
§19-A5). Ticking *every* account every minute forever does not scale, and the PRD requires
the aviary to "continue without the viewer." We reconcile these with one invariant:

> **Composability invariant:** `advance(state, t0, t1)` produces the *same* canonical state
> whether applied as one step or as many sub-steps over `[t0, t1]`. Ergo "tick every minute"
> and "compute the equivalent of N minutes on next access" are identical.

Implementation: `sim` advances **recently-active accounts continuously** (a working set), and
advances **dormant accounts lazily** on the next `GET /aviary/snapshot` by calling
`advance(state, last_tick_at, now)`. A slow background sweeper also advances accounts that
have neither been accessed nor advanced in a long window (so notebook/mood reflect reality if a
visit or export arrives). Because the only time-driven inputs during absence — mood relaxation,
day/night phase, and weather windows — are **deterministic functions of (seed, wall-clock,
last state)** (§7.7, §7.13), lazy and continuous paths agree exactly. Drift during absence adds
nothing (no new presence events), consistent with monotonicity (G11).

Each tick: (1) read unconsumed events in `id` order; (2) apply additive drift deltas (§7.3);
(3) advance moods (§7.4); (4) recompute day phase & weather (§7.7, §7.13); (5) update perch
intents and call dispositions (§7.6); (6) maybe emit a notebook entry under sparsity rules
(§7.11); (7) mark events `consumed_tick` (idempotency); (8) write canonical state + `last_tick_at`.
Ticks are idempotent and replay-safe; a re-run over already-consumed events is a no-op.

### 7.2 Presence detector (client) — the honest signal (G6)

A single `PresenceDetector` in `web` is the only source of `presence_ping`. It emits a ping
**only when all three hold simultaneously**: `document.visibilityState === 'visible'` AND
`document.hasFocus()` AND a `pointermove`/`keydown` occurred within the activity window. The
activity window leans long (default 4 min — §19-A6) because "watching birds without moving is
the actual product"; presence is lost when the user shows no sign of being there *for a while*,
not the instant the mouse stops. Pings carry only "present during [t0,t1]" — never content.
Visibility loss stops rendering (§9.6) and stops pings; the server keeps ticking. No other code
may synthesize presence; the "tab is open" shortcut is explicitly forbidden (would corrupt drift
population-wide).

### 7.3 Drift function — monotonic low-pass toward expressive (G11)

Drift is the slow spine. Model per trait: a **non-decreasing positive-evidence accumulator**
`drift_accum[trait]` fed by a leaky-integrator-shaped *input* but with a **one-way apply**:
the trait value is a saturating function of accumulated evidence and **never decreases**.

- Inputs, by weight: **presence-time (dominant)** → all expressive traits, primarily
  plumage_saturation, social_warmth, boldness; **listen-in on a bird** → that bird's
  social_warmth + vocal_frequency; **offer accepted** → curiosity; **offering near a bird** →
  small boldness; **settle** → no directional drift (just closes the presence window cleanly).
- Saturating curve: `trait = trait_seed + (1 - trait_seed) * (1 - exp(-k * accum))`, so
  early evidence moves the trait more and it asymptotes — three weeks of presence keeps
  mattering but never overshoots, and no single session is visible.
- **Calibration targets (testable, §17.3):** instrument-detectable change after ~1 week of
  regular visits; user-visible change after ~3 weeks; **zero** visible movement from a single
  session; **zero** downward movement from any neglect. `k` and per-input weights are
  server-side flags so calibration is tuned without client redeploys (§15.3).
- "User-visible" is operationalized as crossing a **render bucket** boundary (plumage level,
  or a perch-preference shift), so the calibration target maps to something the test harness
  and the user both perceive (§7.14).

Absence path has no negative branch at all (G11 property test). Neglected birds become
**ambient** (greet less because less has been observed), never wary/silent/duller.

### 7.4 Mood — fast-timescale FSM

Enumerated mood per bird: `{wary, content, curious, drowsy, alert}`. Transitions are a
weighted FSM over: recent session interactions (offer accepted → toward content/curious),
local time-of-day (drowsy near dusk, alert early morning), ambient events (rain → dampened
vocal frequency; another bird's alarm → nearby birds toward wary), and the bird's own
personality (high boldness resists wary on the same input). **Mood persists across sessions**:
session-start mood = session-end mood advanced by the interim tick — never a snap to neutral
on tab open (§7.4 is enforced by the composability invariant; mood is part of canonical state).
**Night sleep** (eyes closed, low on perch) is a scene-derived state from day phase + low
arousal, distinct from the user-triggered `settled` lighting (§7.9); a nightjar-type species
stays active at night (§7.10). This avoids overloading the word "settle."

### 7.5 Return-greeting — the anchor moment (G1, G2)

Computed server-side at session start (when a snapshot is requested after a presence gap) and
emitted as per-bird `greeting` directives in the snapshot; the client renders them as bird
behavior (never as text — G2). Greeting form varies by **(boldness, current mood, absence
length)** and a per-session random draw seeded from `aviary.seed + session nonce` so it is
**real variation, not 3 rotating canned variants** (G1):

- Bird selection: which bird greets *first* honors boldness + warmth + mood (bolder/warmer
  greets first; a wary bird may not greet at all today).
- Absence length: short gap → a glance up from preening; longer gap → re-orientation (step to
  front perch, longer call, possible second-bird response).
- Multiple greeters **stagger** by a small randomized offset — never a unison chorus on cue
  (a synchronized cue would *announce* arrival; staggering reads as the aviary noticing,
  one bird at a time).

This is the highest-risk surface for compression into "play arrival animation." The
absence-length signal, boldness-driven selection, and seeded procedural variation are
acceptance-tested (§17.1). There is **no** textual welcome, "gone X days," or visit calendar
anywhere (G2).

### 7.6 Call-grammar runtime & where scheduling lives

Each species has a procedural call grammar: a small motif library combined and varied at
runtime, timing/pitch shaped by the vocal_frequency trait and current mood. **Split of
responsibility:** the **server** sets each bird's *disposition* (`motif_weights`, `rate_hz`,
`pitch_center`, `jitter_seed`, mood) in the snapshot; the **client** runs a **seeded scheduler**
(`sim-core`) that places actual call onsets within that disposition, advanced by wall-clock.
Because scheduling is deterministic from `(jitter_seed, server_now, disposition)`, **all of the
host's devices, the visitor's view, and the caption generator produce the same call stream**
(§8.2, §10, §11.2) — without a realtime audio sync channel. Per-call parameters are varied each
onset (timing/pitch jitter) so no two calls are identical (G1), while the motif core keeps each
bird's signature **recognizable** across mood and drift (the property that caps birds at 7).

### 7.7 Day/night & timezone

Day phase derives from the account's IANA `tz`: sunrise warms gradually, midday brightest,
evening warmer/quieter, night dims most birds to sleep. Computed as a pure function of
`(tz, server_now)` so it is identical on every device and reproducible in lazy catch-up. The
client renders the phase; the server uses it as a mood input. Local-time anchoring (not server
time) is required so the user's morning is the aviary's morning.

### 7.8 New-bird availability — age-gated only (anti-gamification)

A new-species offer becomes available purely as a function of `now - aviary.created_at` crossing
the next threshold — **never** visit count, interaction score, or payment. Defensible default
ladder (tunable, §19-A8): 3rd bird offered at ~8 weeks, then roughly one further offer every
~10–14 weeks, asymptotically approaching the cap of 7 around ~1 year+. The mechanic must never
teach "more attention earns more stuff." Offers are presented as "a bird has arrived," not a
catalog; the user names it (§7.10).

### 7.9 Settle (& undo)

`settle` event → aviary enters evening lighting over a slow few seconds, calls quiet, birds
drift drowsy/settled; `aviary.settled = true` until tab close or active re-engagement. **Undo:**
any aviary click within 5s emits `settle_undo`, reversing the lighting — a mercy for misclicks,
not a feature. Settle and tab-close are engine-equivalent: both end presence; neither is
penalized; there is no "you didn't settle" surface.

### 7.10 Species pool & adoption

~6 species, each with a distinct silhouette, default plumage palette, and motif library, chosen
to read as one coherent place (including one nightjar-type with a night-active call signature).
New accounts get **two** starters selected by the system (not a catalog) — "the birds that
arrived." User names them at adoption and may rename later with no engine effect. No species
rarity.

### 7.11 Notebook generation — sparse by construction (G3)

The tick may emit at most one entry per `cooldown_key` per long window, targeting **~1 entry
every few days** for a regular aviary (more only on genuinely noteworthy moments: a
first-greeter change, a first chorus, a long quiet stretch, weather). A hard sparsity gate caps
entries even for very active users (an observation-per-session would dilute the entries that
matter into noise). Entries are naturalist prose from `sim-core/prose` (§12), specific to the
moment ("pip greeted before wren today, first time this week"), **never** event-log lines and
**never** observations of the *user's behavior* ("you visited every day") — only observations of
the *aviary* (the line that keeps the notebook from becoming a disguised streak, G3/non-goals).
Read-only: no edit/delete/annotate; infinite scrollback; no archiving.

### 7.12 Offer reactions & cooldown

Offer reaction depends on the receiving bird's mood + curiosity: curious/content approaches a
seed; wary waits then nears; drowsy may ignore. Song-fragment offer plays a soft motif; response
(join / quiet / call-against) shaped by vocal_frequency + mood. Still-pool drops a reflective
surface; some drink, bathe, or watch. **Per-bird cooldown ~ a few minutes**, enforced
server-side (the event is accepted but produces no additional curiosity drift inside the
cooldown) — functional (prevents within-session curiosity saturation that would collapse the
engine), not punitive; makes an offer read as a gesture, not a button-mash.

### 7.13 Ambient weather

A few times a week: short rain or soft wind, derived deterministically from `(aviary.seed,
wall-clock window)` so all clients and lazy catch-up agree. Never assertive (no thunderstorm/
snow/event the user must notice). Effects are short-lived mood nudges: rain briefly dampens
vocal frequency aviary-wide; wind makes some birds alert, others wary.

### 7.14 Bird-to-bird interaction

Calls can prompt responses; wary mood tends to spread to nearby birds; a chorus emerges when ≥2
high-vocal-frequency birds call in the same window. This makes the aviary a small social system,
not a row of independent NPCs. Implemented in the mood FSM (neighbor influence term) and the
call scheduler (chorus-window detection) in `sim-core`.

---

## 8. Sync model

### 8.1 Canonical state, no client ownership

The server is the single source of truth and the only writer of personality (G8). Clients pull
`Snapshot`s and **interpolate** (a bird at perch A in snapshot N and perch B in N+1 renders as
smooth motion, never a teleport). There is **no client-to-client sync, no client-side state to
merge, no eventual consistency to reconcile** — multiple devices each read the same record, so
multi-device sync is a property of the architecture, not a feature.

### 8.2 Determinism across devices (no realtime channel needed)

Fast presentation that must agree across devices/visitor (call onsets, captions, ambient
weather, greeting variation) is derived in `sim-core` from server-provided seeds + `server_now`.
Two devices opening the same snapshot therefore hear the same call stream and see the same
weather without exchanging messages. This is why polling (not push) is sufficient (§4.2).

### 8.3 Conflict prevention — additive deltas in log order (G8)

No last-write-wins on personality. Clients append interaction events; the tick computes
**additive, server-authored deltas** from the event log **in `id` order** and applies them to
the existing vector. A client never sends "set boldness = 0.62"; it sends "listened in to Pip
for 3 min," and the server decides the delta. The morning-laptop/lunch-phone overwrite scenario
is **unreachable** because there is no absolute-value write path and events are processed in
order with `consumed_tick` idempotency. Vector loss is the worst failure in the product (§5.6),
so backups and migrations carry vectors verbatim and never rebuild from logs.

### 8.4 Snapshot pull triggers (client)

On load; on `visibilitychange` → visible; on a long render-frame gap (laptop suspend/resume);
and on a low-frequency keepalive while visible. Each pull first runs lazy catch-up server-side
(§7.1). Stale-snapshot detection via `tick_seq`.

---

## 9. Frontend rendering pipeline

### 9.1 Bootstrap & first frame (G1) — "already in motion"

Critical path: edge-served HTML shell carries a **minimal inlined snapshot** + a tiny critical
JS chunk that draws the **first bird mid-action within 500ms** (§13.2), before non-critical
assets (full audio worklet, settings chunk) load. There is **no spinner, no fade-from-static,
no entry/"wake-up" animation**. When the snapshot is slow (cold cache/slow link), the loading
state is a **quiet field** — soft sky color, one or two faint motion cues — never a spinner
("a spinner says machine"). The empty-aviary moment (post-adoption, pre-first-bird) reuses the
quiet field, then the first bird does a soft fly-in to its starting perch; after that the user
never sees an empty aviary.

### 9.2 Scene composition

One horizontal scene, **no pan/scroll/zoom**, responsive: compresses horizontally on phone,
widens on desktop, **never crops a bird out of frame**. Three perch zones (front/middle/back)
encode proximity; **birds choose perch from mood/personality** — the user cannot place them (no
drag, no "send to front"); perch is a signal the user reads. Subtle foreground/background
parallax (gentle, not parallax-heavy). Calm naturalist palette via design-system tokens; no
saturated UI accents. No UI chrome inside the scene (G4).

### 9.3 Idle micro-motion (G1)

Continuous, **procedural, mood-shaped**, never reading as paused and never cycle-locked: preen,
scan, head-tilt toward sounds, weight-shuffle. Driven by per-bird phase offsets + low-frequency
noise so motion never loops identically (tested, §17.4). Mood is **read from motion** (wary =
further back + more scanning; content = preening; curious = tilt/watch leaves; drowsy = low +
fluffed) — **no labels, tooltips, or status icons** (the moment a bird must be *told* to be
understood, the affective contract fails).

### 9.4 Transitions

Perch-to-perch flight as interpolated motion; staggered multi-bird greeting (§7.5); settle
lighting ramp over a few seconds with 5s undo (§7.9); day/night palette interpolation; weather
fade-in/out. Ambient leaf/feather drift is a pure **client-side idle ornament** (no per-leaf
server state) at slow random intervals.

### 9.5 Top bar (the only chrome)

A thin bar above the scene with exactly four icons: account/settings, accessibility settings,
field notebook, offer affordance. Nothing else. **Fades nearly transparent after a few seconds
of cursor stillness; returns on cursor move or key activity.** Offers open from the top bar
(not by clicking a bird). No notification/badge components exist (G2).

### 9.6 Render lifecycle & memory

rAF render loop targeting 60fps. **Render pauses when the tab is hidden** (nothing to see;
saves battery) while the server keeps ticking; on return, pull a snapshot and interpolate from
current truth (not from the frozen pre-hide frame). Reuse sprite/draw buffers; release notebook
DOM/refs on scroll-out; bounded worker/audio contexts → no memory growth over a 30-min session
(§13.4, CI-tested).

### 9.7 Reduced-motion mode (designed surface, not a fallback — G12, §11.3)

A different *rendering of the same aviary*: micro-motion → slow cross-fades between still poses;
flight → cross-fade between perches; ambient leaf drift removed; day→evening color shifts kept
but slowed. Calls, captions, drift, mood, and notebook all unchanged. Activated by
`prefers-reduced-motion` or the accessibility-settings opt-in. It is calmer and slower, not
broken or stripped.

---

## 10. Audio pipeline

### 10.1 Procedural synthesis (G1, non-goals: no recorded audio)

Calls are **synthesized client-side via WebAudio**, never downloaded as audio files. An
`AudioWorklet` runs the synthesizer off the main thread (no scheduling glitches under render
load). Per species: a motif library (oscillator/envelope/filter graphs as small data) combined
and varied per onset using the seeded scheduler (§7.6). The bundle budget *forces* procedural
synthesis (§13.1) and procedural is also what makes a real chorus possible.

### 10.2 Per-call variation & recognizability

Each onset varies timing/pitch within the bird's disposition so **no two calls are identical**
(G1, tested §17.4), while the motif core keeps each bird's **signature recognizable** across
mood and drift — the property that makes "knowing Pip from Wren by ear" work and caps birds at 7.

### 10.3 Chorus

Two+ birds calling in the same window are mixed as **real-time procedural voices**, not stacked
loops (stacked recorded loops phase-cancel audibly). Chorus events emerge from the bird-to-bird
model (§7.14).

### 10.4 Listen-in mix

Focusing a bird **gradually** raises its mix level while others **gradually** drop toward an
ambient floor — **never to silence** (a re-balance, not a mute/solo). Engage and disengage use
the same slow ramp; a hard cut would turn the aviary into "soloable tracks." Disengage on:
clicking the focused bird again, focusing another bird, clicking empty space, or moving keyboard
focus away. Listen-in is a strong attention signal feeding drift (§7.3).

### 10.5 WebAudio fallback

If WebAudio is unavailable (old browser, denied audio context, hardware issue): **graceful
silence with captions on by default**. **No recorded-audio fallback path exists** — silence +
captions beats canned audio, unconditionally.

### 10.6 Audio memory discipline

Reuse audio buffers and voices; **no per-call allocation that isn't freed**; bounded audio
contexts/worklets. Part of the no-memory-growth CI test (§13.4).

---

## 11. Accessibility surfaces (ship with v1 — G12)

### 11.1 Screen-reader narration

Running **naturalist prose** (not a state list, not "Pip at perch 2," not "mood: content"),
generated from the **same canonical state** the visual reads, via `sim-core/prose` (so it is the
same voice as the notebook and captions — a screen-reader user moving between surfaces hears one
product). Delivered through an ARIA live region. **Cadence is slow:** ~1 prose update / 30–60s
at idle, faster only on user-initiated events (successful offer, settle, return-greeting), which
get a small priority bump but are still written as observations. High-frequency narration would
flood the SR queue — explicitly avoided.

### 11.2 Call captions

Opt-in. Short prose descriptions of what each call sounds like in the bird's current mood ("a
soft three-note rise"; "a low trill, paused, low trill again"), generated **from the same
call-grammar parameters at runtime** so the caption **matches the call actually played** (§7.6).
Rendered as small text near the calling bird, fading with the call, in the naturalist voice.
On by default in the WebAudio-unavailable fallback (§10.5).

### 11.3 Reduced-motion

See §9.7 — a designed surface in the product's aesthetic, shipping in the same CI gate as the
visual path. A "turn animations off → static scene" implementation is a rejected design.

### 11.4 Keyboard navigation

Tab cycles top-bar items; Tab into the scene focuses the first bird; arrow keys move focus
between birds; **Enter triggers listen-in** on the focused bird; **Escape exits listen-in**. The
offer affordance opens from a top-bar shortcut and is fully keyboard-navigable; settle is
reachable from the top bar. Focus indicators are a soft high-contrast outline legible against
both bright and dim aviary states.

### 11.5 Contrast

All user-copy text (top-bar labels, settings, account/error surfaces, captions, any visually
displayed narration) passes **WCAG AA** as the floor; the design system sets per-surface ratios.
The scene itself carries no user copy except the top bar, so the constraint lands on the chrome.
Automated contrast checks in CI on the chrome tokens.

---

## 12. Voice system (G3, G5)

Two strictly separated namespaces, both shipped in v1:

- **`prose.naturalist.*`** (in `sim-core`) — lowercase, present-tense, bird- and
  moment-specific, no "you," no announcement framing, no gamification words. Powers the
  notebook, screen-reader narration, captions, offer prompts, and any aviary-surface copy. A
  denylist test bans gamification phrasing ("achievement," "streak," "level up," "happier!",
  "you visited," "X days"); snapshot tests assert specificity over generic state phrasing.
- **`copy.system.*`** — normal English capitalization, direct, no warmth-as-evasion. Powers
  sign-in, account settings, sync/auth errors, accessibility settings, unsupported-browser, and
  visit-revoked surfaces. Sample: "We couldn't sign you in. The link may have expired. Try
  requesting a new link."

**Enforcement (G5):** system-surface modules import only `copy.system`; a review/lint gate flags
any naturalist string in an auth/error/settings/sync path. The rule's boundary — "any surface
where the user engages with the system *as* a system (identity, money, errors, settings) drops
out of naturalist voice" — is encoded so no future surface has to relitigate it.

---

## 13. Performance budgets & observability

### 13.1 Initial JS bundle < 2MB gzipped (first paint)

Enforced in CI (build fails over budget). Aggressive code-splitting: settings, accessibility
settings, and the visit-invite flow load on demand. Bird visuals are procedural or small
SVG/compact bitmaps. Procedural audio (no audio files) is partly *driven by* this budget.

### 13.2 Time-to-first-bird < 500ms (mid-tier mobile, 4G)

Edge-delivered HTML + inlined minimal snapshot + a critical render chunk that draws the first
bird before non-critical assets. Measured in synthetic checks and aggregate RUM; a regression
over 500ms is release-blocking. This is the affective-perf bridge: above 500ms the user notices
a load and the "already running" conceit breaks.

### 13.3 60fps idle on a 5-year-old mid-range laptop

A **runtime** budget across a 30-minute session, not just first minute. Canvas2D draw batching;
capped active animations; the renderer interface allows a WebGL backend if instruments demand
headroom (§4.4). Frame timing tracked in synthetic + RUM.

### 13.4 No memory growth over 30 minutes (CI test, not a guideline)

Reuse audio buffers (§10.6); release notebook DOM/refs on scroll-out; bounded workers/audio
contexts; no unfreed per-call/per-frame allocation. A CI scenario runs a 30-min headless session
and asserts a flat heap within tolerance.

### 13.5 Observability — and what we deliberately do not measure (G10, §14.2)

Collected (**aggregate only, no per-account dimension**): page-load timings, first-bird-render
timings, render-frame timings, audio-context error counts, simulation-tick latency, request
counts/error rates, anonymized session-duration histograms. Synthetic browser fleet runs the
aviary on a schedule from several geographies. **Tick latency p99 alarms at > 5s.**

**Deliberately NOT measured / NOT computed:** per-bird state, per-account interaction history,
visit-frequency, "days visited," anything that could reconstruct a user's relationship with
their aviary, and any metric that a leaderboard/streak could later be built from. The absence is
architectural, not a setting (G10).

### 13.6 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers get a matter-of-fact
unsupported-browser surface (G5). No compatibility paths for very old browsers (bundle cost not
justified).

---

## 14. Privacy & data boundaries

### 14.1 PII isolation (G9)

Email is stored **once**, encrypted, on `account`. Everywhere else — DB keys, shard/partition
keys, inter-service messages, telemetry, logs, error messages — uses the synthetic
`account_id` UUID. A log/telemetry schema linter denies any field that looks like an email or
derives an id from email. This is honored at design time because it is impossible to retrofit.

### 14.2 Telemetry boundary (G10)

The simulation store is network- and credential-isolated from the analytics warehouse; **no ETL
path connects them.** Telemetry event schemas have no per-account dimension. Per-bird interaction
events exist **only** to drive that user's own simulation — never aggregated, never used for
training, recommendations, third-party sharing, or population analysis. "Is this account having
errors" (allowed) vs. "what is this account's bird doing" (forbidden in aggregate) is enforced at
metric definition, not policy.

### 14.3 Account lifecycle

Magic links: 15-min expiry, single-use, per-email rate limit. Per-device sessions revocable.
Email change verified before commit (old email works until then). Export: on-demand JSON, signed
emailed link. Deletion: soft for 30 days (sign-in to recover), then hard — birds, vectors,
notebook, telemetry tied to the account, all gone. Privacy policy linked in plain text in
settings, naming the aggregate categories and explicitly excluding per-bird state.

---

## 15. Rollout

### 15.1 Phases

1. **Internal alpha** — full engine + one device; calibration harness (§17.3) running against
   simulated weeks; perf budgets wired into CI; accessibility surfaces present from the start
   (not deferred — G12).
2. **Closed beta** — invite-only; multi-device sync exercised; synthetic perf fleet live;
   aggregate RUM live; drift calibration watched at the **aggregate distribution** level only
   (never per-account inspection — G10).
3. **GA** — public sign-up; bird-ramp ladder live; visit feature on (off by default per account).

### 15.2 Bird-per-aviary ramp

Start at 2; offers gated by **aviary age only** (§7.8), defaults tunable server-side. Cap of 7
enforced server-side. The cap may be revisited only if audio-mix work later raises the
recognizability ceiling; 7 is the v1 limit and is built into the engine.

### 15.3 What we instrument from day one

Perf RUM + synthetic checks; tick-latency p99 alarm; auth-funnel and error rates (aggregate);
build-time bundle-budget gate; memory-growth CI; accessibility CI (contrast, keyboard, narration
voice). **Calibration constants** (tick cadence, drift `k`/weights, presence activity window,
mood timers, weather frequency, bird-age thresholds) are **server-side flags** so calibration is
tuned without client redeploys — critical because drift calibration will need adjustment from
real aggregate data, and we must never ship a client just to nudge `k`.

### 15.4 Migration safety

Every migration preserves bird identity and personality vectors verbatim (§5.6); a migration
that rebuilds vectors from logs is forbidden. Vector backups precede any schema change touching
`bird`.

---

## 16. Risks & mitigations

- **Drift miscalibration** (too fast → numbers-Tamagotchi; too slow → screensaver). *Mitigation:*
  calibration harness simulating 1-week/3-week regular-visit profiles asserting the
  instrument-vs-visible gap; server-side tunable constants; staged rollout watching the aggregate
  drift distribution; monotonicity property test (G11).
- **Personality loss / sync corruption** (worst failure; invisible). *Mitigation:* server-only
  writer, additive deltas, log-order processing with `consumed_tick` idempotency, no LWW
  (§8.3), stable bird ids (§5.6), pre-migration vector backups, restore tests.
- **Audio uncanniness / canned-ness.** *Mitigation:* procedural-only; "no two identical calls"
  and chorus phase tests (§17.4); listen-in ramp tuning; recognizability checks in beta; the
  no-recorded-audio rule is unconditional, fallback is silence+captions.
- **Accessibility regression / second-class accessible product.** *Mitigation:* narration,
  captions, reduced-motion ship in the same v1 CI gate (G12); narration/captions use the shared
  naturalist prose (one voice); automated a11y + manual screen-reader passes each release.
- **Announcement/gamification leak** (the "harmless toast/streak"). *Mitigation:* no
  toast/notification component or visit-frequency metric exists; lint blocks introducing one;
  prose denylist; PR review gate citing G2/G3; the underlying counters are never computed.
- **PII leak via email-as-id.** *Mitigation:* synthetic UUID everywhere; email encrypted in one
  column; log/telemetry schema linter (G9).
- **Tick scalability.** *Mitigation:* composability invariant + lazy catch-up + active-set
  ticking + sweeper (§7.1); p99 5s alarm.
- **First-frame conceit failing on slow networks.** *Mitigation:* quiet field (never a spinner),
  edge-inlined snapshot, progressive first-bird draw; 500ms budget is release-blocking.
- **Mood "snapping" to default on tab open.** *Mitigation:* mood is canonical state advanced by
  the composability-correct tick; never client-reset (§7.4).

---

## 17. Testing & calibration strategy

1. **Return-greeting acceptance** (§7.5): asserts variation by absence length and boldness,
   first-greeter selection honors warmth/mood, multi-greeter staggering, and that variation is
   seeded-procedural (no fixed rotation). Asserts **no textual welcome** is rendered (G2).
2. **Voice tests** (§12): naturalist denylist (gamification phrasing), specificity snapshot
   tests, and a guard that system-surface modules never import naturalist prose (G5).
3. **Drift calibration & monotonicity** (§7.3): simulate weeks; assert instrument-detectable
   change ~1 week, visible (render-bucket-crossing) ~3 weeks, no single-session visible move;
   property test that no sequence with absence reduces any trait (G11).
4. **Audio variation/recognizability** (§10): "no two identical calls" over N onsets; chorus
   non-phase-cancellation; per-species signature stability across mood/drift; listen-in ramp
   (no silence floor breach, no hard cut).
5. **Sync correctness** (§8.3): concurrent multi-device event streams; assert additive deltas in
   log order, idempotent re-ticks, and that the morning/lunch overwrite scenario cannot lose
   drift; no client vector-write endpoint exists (G8).
6. **Presence honesty** (§7.2): each pairwise-only condition (visible+focused but idle;
   visible+active but unfocused; focused+active but hidden) produces **no** ping; only the
   conjunction does (G6).
7. **Privacy/PII** (§14): log/telemetry schema linter; assert no email-derived ids and no
   per-account telemetry dimension; assert no network route simulation-store→warehouse (G10).
8. **Performance gates** (§13): bundle-budget build gate; synthetic TTFB <500ms; frame-timing
   60fps; 30-min memory-growth CI scenario.
9. **Accessibility** (§11): contrast CI; keyboard-path tests (Tab→bird, Enter listen-in, Escape
   exit); narration cadence + live-region behavior; reduced-motion render-mode tests.
10. **Snapshot DTO guard** (§5.4): fails if any raw trait scalar field name appears in any
    client-facing response or export (G7).

---

## 18. Work breakdown (suggested milestones for an executing team)

- **M0 — foundations:** repo + `sim-core` skeleton; Postgres schema (§5); auth (magic link,
  sessions); synthetic-UUID + log linter (G9). CI: bundle-budget gate, lint, prose denylist.
- **M1 — engine core:** tick with composability invariant + lazy catch-up (§7.1); personality
  vector + additive drift (§7.3); mood FSM (§7.4); day/night + weather (§7.7/§7.13);
  calibration harness (§17.3). No client yet — validated via harness.
- **M2 — client scene:** Canvas2D renderer, three perch zones, parallax, day/night palette,
  first-frame-in-motion bootstrap (§9.1), top-bar with fade, snapshot polling + interpolation
  (§8), presence detector (§7.2). Reduced-motion mode in parallel (§9.7).
- **M3 — audio:** AudioWorklet synth, motif libraries per species, seeded scheduler, chorus,
  listen-in ramp, WebAudio fallback (§10).
- **M4 — interactions & notebook:** return-greeting (§7.5), offer + cooldown (§7.12), settle +
  undo (§7.9), notebook generation with sparsity (§7.11), prose engine (§12).
- **M5 — accessibility:** narration live region (§11.1), captions from grammar (§11.2), keyboard
  nav (§11.4), contrast pass (§11.5). Gating for v1.
- **M6 — accounts & social:** settings, sessions list, email change, export, soft/hard delete
  (§6, §14.3); visit invite/read/revoke/log + opt-in notify (§6.5).
- **M7 — hardening & rollout:** synthetic fleet + aggregate RUM (§13.5), p99 alarm, memory CI,
  beta calibration on aggregate distribution, bird-ramp flags, migration-safety drills (§15.4).

Dependency notes: M1 blocks M2/M3; `sim-core` (drift, mood, scheduler, prose) underlies M1–M5;
accessibility (M5) is **not** deferrable past v1 (G12).

---

## 19. Assumptions & defensible calls (PRD-ambiguous points)

- **A1 — One TS codebase + shared `sim-core`.** Chosen so narration, captions, the call
  scheduler, drift, and mood have a single implementation → guarantees caption/audio match,
  narration/visual match, and host/visitor parity. Alternative (separate server/client impls)
  risks the surfaces drifting into "two products."
- **A2 — Snapshot polling, no WebSocket/SSE in v1.** The PRD's pull triggers + slow tick +
  client interpolation make push unnecessary; polling is simpler/cheaper/CDN-friendly. Revisit
  only if a future feature needs sub-tick server-initiated updates.
- **A3 — Preact + signals for chrome.** Tiny footprint within the 2MB budget; the scene is a
  custom render loop, not framework-driven.
- **A4 — Canvas2D renderer behind a `SceneRenderer` interface.** Sufficient for ≤7 birds +
  parallax; lighter than WebGL/PixiJS; swappable backend if 60fps instruments demand it.
- **A5 — Tick cadence default 60s,** server-flag tunable.
- **A6 — Presence activity window default 4 min,** leaning long ("watching without moving is the
  product"); calibratable.
- **A7 — Export emits coarse plumage render buckets, not raw scalars,** to honor G7 even in an
  export the user controls; the user gets meaningful state without the product ever turning a
  bird into a number.
- **A8 — New-bird age ladder:** 3rd ~8 weeks, ~one more every ~10–14 weeks, approaching 7 around
  ~1 year+; tunable. Strictly age-gated, never attention/payment-gated.
- **A9 — Mood enum = {wary, content, curious, drowsy, alert};** night-sleep is a scene-derived
  state from day phase (distinct from user-triggered `settled` lighting), avoiding overloading
  "settle."
- **A10 — Call onset scheduling lives client-side but is seeded deterministically by the server,**
  so multi-device + visitor + caption generation all converge without a realtime audio channel.

All ambiguous calls above are reversible via server-side flags or an internal interface; none
weaken a guardrail in §3.
