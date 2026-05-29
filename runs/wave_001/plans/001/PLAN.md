# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable engineering program. It is written for a frontier team that will build it without further clarification. It interprets the spec rather than restating it: where the PRD names an affective contract, this plan names the mechanism that enforces it and the test that proves it. Where the PRD leaves a value to be calibrated during build, this plan picks a defensible starting value and says how it gets tuned.

The single organizing idea: **the server owns the bird; the client renders it.** Personality is a server-authored, additive-delta record advanced by a slow tick that runs whether or not anyone is watching. Everything else — sync coherence, the "continues without the viewer" conceit, the no-Tamagotchi asymmetry, the privacy boundary — falls out of holding that line without exception.

---

## 1. Scope

### 1.1 In scope for v1

- Single-user accounts, magic-link email sign-in, per-device revocable sessions, email change with verification, account export (JSON via emailed link), soft-then-hard deletion (30 days).
- One canonical aviary per account. Two starter birds at adoption; cap of seven; additional birds offered on aviary-age cadence (server-side selection from a ~6-species pool, no catalog).
- Server-side simulation tick (~60s cadence) that is the **only** writer of personality vectors and mood; consumes an append-only interaction-event log in order.
- Bird engine: hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic-toward-expressive drift, enumerated mood states with fast-timescale transitions, stable per-bird identity, mood persistence across sessions, bird-to-bird interaction.
- Client render pipeline: single horizontal three-perch scene, day/night anchored to user local time, ambient weather, ambient leaf/feather drift, top-bar chrome with fade, "loads with motion already in progress," empty/loading quiet-field state, responsive scene that never crops a bird.
- Interactions: procedurally-varied return-greeting, listen-in (gradual mix re-balance), offer (seed / song-fragment / still-pool with per-bird cooldown), settle (with 5s undo), field notebook (auto-generated, sparse, read-only), precise presence accounting (visibility ∧ focus ∧ recent activity).
- Audio: client-side WebAudio procedural call synthesis, per-bird recognizable call signatures, real-time chorus, listen-in mix decay, graceful-silence + captions-on fallback when WebAudio is unavailable.
- Accessibility as first-class designed surface: naturalist screen-reader narration, reduced-motion as its own rendering, runtime-generated call captions, keyboard navigation, WCAG AA contrast on all user copy.
- Social: single read-only ambient visit affordance (off by default), per-invite email opt-in, revocable, 30-day invite expiry, silent visit log, optional per-account visit-notification toggle (off by default).
- Performance: <2MB gzipped initial bundle, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop, zero memory growth over 30 minutes (CI-enforced), aggregate-only telemetry, synthetic perf checks, p99 tick-latency alarm at 5s.

### 1.2 Out of scope (respected, not deferred-with-hooks)

Native apps; payments; shared/multi-profile aviaries; customizable scenes; multi-aviary accounts; public discovery; leaderboards; achievements/streaks/scores/badges/levels/XP; push notifications; any user-behavior surface (visit counts, calendars, "you've been gone X days"); any numeric exposure of personality; recorded-audio fallback; co-presence in visits; chat/avatars/comments on visits; symmetric (punishing) drift; client-side simulation.

These are not "later." We will not build data-model fields, telemetry pipelines, or protocol affordances that make them cheap to add. The PRD is explicit that the *architectural absence* is the defense (e.g. "no leaderboards means no underlying metrics aggregated across accounts"). We honor that: no cross-account aggregation of bird/interaction state exists anywhere in the system.

### 1.3 Defensible calls on PRD ambiguities

The PRD invites us to "make a defensible call and note it." The notable ones:

- **Presence activity window:** start at **3 minutes** of no pointer/key activity before presence is considered lost, biased long per the PRD ("leaning toward the longer side because watching birds without moving is the actual product"). Tunable server-side config; see §5.4.
- **Tick cadence:** **60s** canonical tick; client keepalive presence ping at **20s** while visible (3 pings per tick window). See §5.1, §6.3.
- **Drift calibration:** target ~1 week to instrument-detectable, ~3 weeks to user-perceptible. We model this with an explicit low-pass time constant and a perceptibility-quantization layer (§4.4) so "no single session moves a trait visibly" is structural, not hoped-for.
- **Mood set:** `wary, content, curious, drowsy, alert, settled` (settled = night/post-settle resting). See §4.5.
- **Notebook sparsity:** target ≤1 entry / 3 days for a regular visitor, with a hard rate cap and a noteworthiness threshold (§7.6).
- **Species pool size:** 6 species, one of which is a nightjar-like night-caller.

---

## 2. Architecture

### 2.1 Service shape

Five backend services plus a static client, deliberately kept small. The privacy boundary (§9) dictates which services may talk to which datastore, so service boundaries are drawn on the data-access line, not just on feature lines.

1. **auth-svc** — magic-link issuance/consumption, session-token mint/revoke, email-change verification. Owns the `account` record (the only place email lives, encrypted). Issues the synthetic account UUID at creation.
2. **aviary-svc** — read API for state snapshots; write API for the append-only interaction-event log; adoption flow; bird rename; account export; deletion lifecycle. Owns `bird`, `aviary`, `event_log`, `notebook` tables. **Does not** compute drift; it records events and serves snapshots.
3. **sim-svc** — the simulation tick worker. The **only** writer of `bird.personality_vector` and `bird.mood`. Reads `event_log`, computes additive deltas, advances mood timers, writes canonical state, generates notebook entries and narration text. Has no public HTTP surface; it is a scheduled/queue-driven worker.
4. **visit-svc** — visit invitations, one-time visitor links, read-only visitor snapshot proxy (reads canonical state via aviary-svc's read path; never records presence/events from visitors), visit log, revocation.
5. **telemetry-svc** — aggregate operational metrics ingestion. **Physically isolated** from the simulation datastore (separate DB credentials with no read grant to the sim/aviary tables). This is the architectural enforcement of the privacy commitment.

A thin **edge/CDN** layer serves the static client bundle and an inlined first-state-snapshot for time-to-first-bird (§8.2).

```
            ┌──────────┐   magic link / sessions   ┌──────────────┐
  browser ──┤ auth-svc ├───────────────────────────┤ account DB   │ (email, encrypted)
     │      └──────────┘                            └──────────────┘
     │ snapshot pull / event write
     ▼
 ┌────────────┐   read snapshot / append events   ┌──────────────────┐
 │ aviary-svc ├───────────────────────────────────┤ simulation DB    │
 └────────────┘                                    │ bird, aviary,    │
     ▲  read-only snapshot                         │ event_log,       │
 ┌────────────┐                                    │ notebook         │
 │ visit-svc  ├────────────────────────────────────┤ (visitor: read) │
 └────────────┘                                    └────────┬─────────┘
                                                            │ tick (sole writer of vectors/mood)
                                                   ┌────────┴─────────┐
                                                   │     sim-svc      │
                                                   └──────────────────┘

 telemetry-svc ── aggregate metrics DB   (NO grant to simulation DB)
```

### 2.2 Client/server split — the hard line

- **Server owns:** personality vectors, mood, canonical bird positions/perch choices at tick granularity, drift, notebook entries, narration prose, day/night phase math inputs, weather events, adoption/age-based bird offers.
- **Client owns:** rendering, interpolation between snapshots, idle micro-motion playback, procedural call **synthesis** (timing/pitch parameters come from the snapshot; the actual audio graph is local), ambient ornaments (leaves/feathers — pure render, no server state), top-bar fade, reduced-motion rendering, caption rendering.
- **Never on the client:** any write of personality state; any tick; any authoritative mood transition. The client emits *events* ("listened in to bird X for N seconds"); the server decides meaning.

This split is the load-bearing rule from `accounts_sync.md` and `bird_engine.md`. We enforce it in code review and with an architectural test: grep/lint rule that fails CI if any client module imports or references a personality-vector write path, and a server contract test asserting that the only DB role with `UPDATE` on `bird.personality_vector` is the sim-svc role.

### 2.3 Render pipeline boundary

The render boundary is the **snapshot**. The client receives a small (kilobytes) snapshot describing per-bird state and active transitions, and renders forward by interpolation + local procedural motion until the next snapshot. The client never extrapolates personality or invents drift; between snapshots it only interpolates motion and runs ambient ornaments. See §6.

### 2.4 Tech choices (defensible defaults)

- Client: TypeScript, a thin reactive view layer (Preact/Solid-class, chosen for bundle size against the 2MB cap — not React-default), Canvas2D or lightweight WebGL for the scene (decided in the rendering spike, §11), WebAudio for synthesis, Web Worker for audio scheduling and snapshot interpolation math to protect the main-thread 60fps budget.
- Server: a single language/runtime across services (Go or TypeScript/Node; pick one in week 0) for staffing simplicity; Postgres for the simulation DB (row-level ownership, ordered event consumption, transactional delta application); a durable queue (e.g. Postgres-backed or a small broker) to drive sim-svc ticks and absorb backpressure.
- Edge: static hosting + CDN with edge-inlined first snapshot.

---

## 3. Data model

All identifiers are synthetic UUIDs. **Email never appears as a key, partition, shard, or log field** — it lives once, encrypted, on `account`. (`accounts_sync.md`: "the single most important boring detail.")

### 3.1 `account`
- `account_id` (UUID, PK)
- `email_encrypted` (bytes; the only email storage in the system)
- `email_verified_at`
- `pending_email_encrypted`, `pending_email_token`, `pending_email_expires_at` (email-change flow)
- `created_at`
- `status` (`active | pending_deletion`)
- `deletion_requested_at` (nullable; hard-delete job fires at +30d)
- `settings` (JSON: reduced_motion_pref, captions_on, audio_on, visit_notifications_enabled=false, …)

### 3.2 `session`
- `session_id` (UUID, PK), `account_id` (FK)
- `device_label` (best-effort, user-facing in session list), `created_at`, `last_seen_at`, `revoked_at`

### 3.3 `aviary`
- `aviary_id` (UUID, PK), `account_id` (FK, unique — one aviary per account)
- `created_at` (drives bird-offer age cadence)
- `tz` (IANA timezone string; resolved client-reported, used for day/night and offers)
- `last_tick_at`, `tick_seq` (monotonic)
- `weather_state` (current ambient weather + expiry), `lighting_phase` (derived, cached)

### 3.4 `bird`
- `bird_id` (UUID, PK — **stable forever**; never reissued on rename/sync/migration)
- `aviary_id` (FK)
- `species` (enum from the 6-species pool)
- `name` (user-assigned; renameable; independent of id)
- `personality_vector`: `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` — floats normalized to `[0,1]`, server-only writer
- `personality_seed` (the at-adoption baseline, retained so drift is always "from baseline" auditable)
- `mood` (enum), `mood_entered_at`, `mood_timer_expires_at`
- `perch` (front/middle/back), `pose_state` (preen/scan/tilt/shuffle/rest — render hint)
- `drift_accumulator` (internal low-pass state per trait; see §4.4)
- `last_offer_at` per offer-type (cooldown bookkeeping)
- `created_at` (adoption time)

Personality vector is **persisted, never derived from history at runtime** (`bird_engine.md`). The `event_log` exists to *drive the tick*, not to recompute state on read.

### 3.5 `event_log` (append-only)
- `event_id` (UUID, PK), `aviary_id` (FK), `bird_id` (nullable — presence is aviary-level, listen-in/offer are bird-level)
- `seq` (monotonic per aviary; defines processing order — the basis of no-last-write-wins)
- `type` (`presence_tick | listen_in_start | listen_in_end | offer | settle`)
- `payload` (JSON: e.g. offer type, listen-in duration on `end`)
- `client_ts`, `server_received_ts`
- `consumed_by_tick_seq` (nullable; set when sim-svc processes it — makes consumption idempotent)

Events are immutable and **never carry absolute personality values**. The client cannot say "set boldness=0.62"; it can only say "user listened in to bird X for 180s" (`accounts_sync.md`).

### 3.6 `notebook_entry`
- `entry_id` (UUID, PK), `aviary_id` (FK)
- `created_at`, `local_day` (for the "tuesday —" prefix)
- `prose` (naturalist text, server-generated, immutable)
- `noteworthiness_score` (internal; used for sparsity gating, not shown)

Read-only to the user. No edit/delete/annotate API exists.

### 3.7 `visit_invite`
- `invite_id` (UUID, PK), `aviary_id` (FK)
- `visitor_email_encrypted` (PII; same encryption discipline)
- `one_time_token`, `issued_at`, `expires_at` (+30d), `revoked_at`, `consumed_at`
- `status` (`outstanding | active | revoked | expired | used`)

### 3.8 `visit_log_entry`
- `log_id` (UUID, PK), `aviary_id` (FK), `invite_id` (FK)
- `started_at`, `approx_duration_s`, `visitor_email_encrypted` (shown to host in visit log)

### 3.9 What is deliberately absent from the model

No `streak`, `visit_count`, `last_visit`, `days_active`, `level`, `score`, or any cross-account aggregate table. No `happiness`/`hunger`/`health` field. No negative-drift state. Their absence is the enforcement of the non-goals — adding any later requires a schema change, which is the friction we want.

---

## 4. Simulation engine design

The engine is the product. This section is the most prescriptive.

### 4.1 The tick

`sim-svc` advances each aviary on a ~60s cadence. A tick for one aviary:

1. **Lease the aviary** (advisory lock on `aviary_id`) so two workers never tick the same aviary concurrently — protects ordered, single-writer delta application.
2. **Read new events** since `consumed_by_tick_seq`, in `seq` order.
3. **Compute presence-time** in this window from `presence_tick` events (§4.3).
4. **Apply drift deltas** (§4.4) — additive, monotonic non-decreasing.
5. **Advance mood** for each bird (§4.5): apply time-of-day, weather, recent-interaction nudges, personality damping; tick mood timers.
6. **Update perch/pose hints** from mood+personality (§4.6).
7. **Maybe emit a weather event** (§4.7).
8. **Maybe write a notebook entry / refresh narration** (§7.6, §9-narration).
9. **Write canonical state**, bump `tick_seq`, mark events consumed (idempotent via `consumed_by_tick_seq`), set `last_tick_at`.

The tick must run for accounts with **no connected client** — that is the "continues without the viewer" property. Scheduling: a sweep enqueues any aviary whose `last_tick_at` is older than the cadence. Idle aviaries with no events still get cheap ticks (mood/time-of-day/weather advance; drift delta is zero because presence-time is zero — and zero is non-decreasing, satisfying the no-punishment rule for free).

**Why the tick is server-side and sole-writer:** §2.2 and `accounts_sync.md`. If the client ticked, two devices would diverge and any merge corrupts drift. We never enter that state.

### 4.2 Personality vector

Five traits in `[0,1]`, seeded per species with small per-bird randomization at adoption (`personality_seed`). The starter pair is chosen by the server from the pool; the two species are picked to feel coherent (not rarity-weighted). Seeds are never shown and never exposed numerically anywhere, any tier (`bird_engine.md`, hard rule). We enforce this with a test asserting no API response (snapshot, export, narration, caption) contains raw trait floats; the export ships *current* vectors only as part of the user's own data download, which the PRD explicitly allows ("current personality vectors" in `account_export`) — note: this is the one place vectors leave the server, and it goes only to the verified account owner as their own data, never rendered in-product as numbers.

> Calibration note: account-export including raw vectors is in tension with "never exposed numerically." We resolve it by treating export as *data portability of the user's own record* (a different context from in-product display), but we round/label them as opaque ("expressiveness markers") rather than `boldness: 0.62` so the export can't become a stat dashboard by inspection. Flagged for product sign-off.

### 4.3 Presence computation

A `presence_tick` event is only emitted by the client when **all three** hold simultaneously (`concepts.md`, `interactions.md`):

- `document.visibilityState === 'visible'`, **and**
- the document has window focus (`document.hasFocus()`), **and**
- a `pointermove` or `keydown` occurred within the activity window (default 3 min).

The client emits a `presence_tick` on its keepalive (20s) only while all three hold. The server sums presence-time as `count(presence_tick in window) × keepalive_interval`, clamped to the wall-clock window length so a burst of events can't inflate it. Visitor sessions never emit presence (`social_optional.md`). Background/hidden tabs stop rendering and stop pinging; the sim keeps ticking but records zero presence for that account — exactly the "laptop open all night ≠ watching" guarantee.

### 4.4 Drift function

Drift is a **slow low-pass filter over presence-and-interaction signals**, **monotonic toward expressive** (`bird_engine.md`).

Per trait, per tick:

```
raw_signal      = w_presence * presence_time_norm
                + w_listen   * listen_in_signal     (warmth, vocal_freq)
                + w_offer    * offer_signal          (curiosity; boldness if offered near bird)
delta           = max(0, alpha * (raw_signal))      # clamp ≥ 0  → never negative
accumulator    += delta                              # low-pass integrator
trait           = min(1, trait + quantize(delta))    # additive, capped at 1
```

- `alpha` (time constant) is set so a *regular* visitor (say ~10 min/day) crosses the instrument-detectable threshold (~Δ0.01–0.02 on a primary trait) in ~7 days and the user-perceptible threshold (visible behavioral change — greets-first frequency, front-perch tendency, plumage richness) in ~3 weeks.
- **Perceptibility quantization (`quantize`)**: behavioral/visual expression reads trait *bands*, not raw floats, so within-session float movement never crosses a band boundary. This makes "no single session moves a trait visibly" a structural guarantee rather than a tuning hope. Drift moves the float continuously; expression changes only when a band boundary is crossed, which takes the calibrated weeks.
- **Monotonicity** is enforced by the `max(0, …)` clamp: neglect contributes zero, never negative. Plumage saturation, explicitly, only ever rises (`bird_engine.md`). A neglected bird becomes *ambient* (greets less because less has been observed — an expression effect of low recent presence, **not** a downward trait drift). We implement "greets less" as a *mood/recency* effect (§4.5), not as trait decay.

> This asymmetry is the load-bearing "no Tamagotchi" implementation. A reviewer's instinct will be to add symmetric decay "for realism." It is explicitly wrong here. Test: a simulated account that goes idle for 14 days shows **no decrease** in any trait; greeting frequency drops via the recency path and recovers on return.

Inputs weighting (rough, from `bird_engine.md`): presence-time dominant; listen-in strong (→ warmth, vocal frequency of the focused bird); offers small (accept → curiosity; offering near a bird → boldness); settle contributes no directional drift, only clean presence-window termination.

### 4.5 Mood

Enum: `wary, content, curious, drowsy, alert, settled`. Mood is fast-timescale, **persists across sessions** (the session-end mood is the session-start mood, modulo intervening ticks — `bird_engine.md`). Transitions per tick are a weighted function of:

- recent in-session interactions (offer accepted → toward content; alarm/wary spread from a neighbor → toward wary),
- local time of day (drowsy toward dusk, alert early morning, `settled` at deep night except the nightjar species),
- ambient weather (rain → dampen vocal frequency briefly; wind → some alert, some wary),
- the bird's own personality (high boldness damps entry into wary on the same input).

Transitions are stochastic with personality-weighted probabilities, never a hard state machine the user could decode. Mood never "snaps to neutral" on tab open — the client renders whatever canonical mood the snapshot carries. Mood timers (`mood_timer_expires_at`) prevent flicker.

### 4.6 Perch & pose as readable signal

Perch (front/middle/back) and pose (preen/scan/tilt/rest) are derived each tick from mood+personality and shipped as render hints. Wary/low-boldness → back + scan; content → middle + preen; curious → forward + tilt-toward-sound; drowsy/settled → low + rest. The user reads mood off motion with **no label/tooltip/icon** (`bird_engine.md`, `aviary_layout.md`). The user never arranges birds; perch is signal, not control.

### 4.7 Call grammar runtime

Each species has a **motif library** (a small set of pitch/rhythm motifs). The runtime (client-side synthesis, server-side timing parameters) combines and varies motifs with personality-shaped timing/pitch so:

- a call is **never identical twice** (variation seeded per emission),
- each bird's signature is **recognizable across mood and drift** (the motif identity is preserved; mood/drift modulate timing/pitch/density within recognizable bounds),
- **chorus** emerges when ≥2 high-vocal-frequency birds call in the same window — real-time mixing, not stacked loops (this is *why* synthesis is client-side; §8).

The server snapshot carries: which bird is calling, motif id, variation seed, and modulation parameters (mood, vocal-frequency band). The client synthesizes from those. Caption text (§ accessibility) is generated from the *same* parameters so the caption matches what was actually played.

### 4.8 Bird-to-bird interaction

The tick models simple inter-bird coupling: a call can raise a neighbor's call probability next sub-step; a wary mood has a spread probability to nearby birds; chorus is the emergent co-occurrence of high-vocal-frequency callers. This is what makes the aviary "a small social system, not a row of NPCs" (`bird_engine.md`). Kept cheap: coupling is computed within an aviary's small bird set (≤7) per tick.

### 4.9 Bird offers (growth) and adoption

- **Adoption:** new account → server selects 2 coherent species from the pool, seeds vectors, lets the user name them (defaults provided). Presented as "the birds that arrived," not a catalog (`bird_engine.md`).
- **Growth:** additional-bird offers are gated on **aviary age** (`aviary.created_at`), never on visit count/score/payment. A few-months aviary may offer a third; a year-old one may reach 5–6; hard cap 7. The offer appears in-flow (a new bird arriving), not as a store. Accepting names the new bird; declining is fine and re-offered later.

---

## 5. Sync model

### 5.1 One canonical record, many readers

Multi-device sync is a **property of the architecture, not a feature** (`accounts_sync.md`). The server holds one canonical aviary; every client (laptop, phone) reads the same snapshots. There is no client-to-client sync, no client state to merge, no eventual consistency to reconcile.

### 5.2 No last-write-wins

Personality is written **only** by the tick, as **additive server-authored deltas applied in `event_log.seq` order**. A client never PUTs a vector. The lunch-phone-overwrites-morning-laptop failure (`accounts_sync.md`) is unreachable because no client write path to personality exists, and the tick processes the single ordered log. We enforce with:

- DB grant: only the sim-svc role has `UPDATE` on personality columns.
- Idempotent consumption (`consumed_by_tick_seq`): re-processing the same events is a no-op.
- Advisory lock per aviary during a tick: no concurrent delta application.

### 5.3 Snapshot consumption

Clients pull a fresh snapshot:
- on load,
- on `visibilitychange` → visible (returning from a hidden tab),
- on a long render-frame gap (laptop resumed from suspend),
- on a low-frequency keepalive while visible.

Snapshots are kilobytes (per-bird position, mood, call timing, active transitions). The client interpolates between snapshots for smooth motion; a bird at perch A (snapshot N) → perch B (snapshot N+1) is rendered moving, not teleporting (`accounts_sync.md`).

### 5.4 Conflict & error surfaces (matter-of-fact voice)

Rare cases — magic-link replay, in-flight session timeout mid-write, server outage — surface in **matter-of-fact** tone (the named voice exception, `product_brief.md`, `accounts_sync.md`):

- "We couldn't sign you in. The link may have expired. Try requesting a new link."
- "Your session timed out. Sign in again to keep watching."
- "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

No naturalist phrasing on system/identity/error/settings surfaces. A lint/style check on copy strings flags naturalist tokens (lowercase-leading, bird verbs) in the system-surface string bundle.

---

## 6. Frontend rendering pipeline

### 6.1 Scene composition

Single horizontal scene, three perch zones (front/middle/back), foreground/background with **subtle** parallax, sky + soft foliage behind, occasional foreground branch. No panning/scrolling/zoom. Responsive: compress horizontally on narrow viewports, widen perch spacing on desktop, **never crop a bird out of frame** (`aviary_layout.md`). Birds are small SVG/compact-bitmap/procedural assets (bundle budget, §8).

### 6.2 Loads with motion already in progress

The first frame has birds mid-action — a bird mid-preen, another calling from the high perch, a leaf drifting. **No** spinner, wake-up animation, fade-from-static, or entry sequence (`aviary_layout.md`). Implementation: client pulls (or receives edge-inlined) the current snapshot, places birds at their current positions/poses, and starts rendering as if mid-stream. When the snapshot is slow (cold cache/slow net), the loading state is a **quiet field** (soft sky, one or two faint motion cues), never a spinner. Empty-aviary (post-adoption, pre-first-bird) uses the same quiet field; the first bird enters with a soft fly-in, and the user never sees an empty aviary again.

> This is a graded affective contract the reviewer will check hard. A spinner-then-fade is an explicit failure. The render path must draw the first bird without waiting for non-critical assets (§8.2).

### 6.3 Idle micro-motion & transitions

Continuous mood-shaped micro-motion (preen, scan, tilt, weight-shuffle) runs locally, driven by the snapshot's mood/pose hints; it does not pause when the tab is visible-but-idle. When the tab is hidden, rendering stops (battery) but the server keeps ticking; on return the client pulls a fresh snapshot and resumes from canonical state. Flight/perch changes interpolate smoothly. Ambient leaf/feather drift is a **pure client ornament** (no server state) at idle cadence.

### 6.4 Reduced-motion mode (its own designed surface)

For `prefers-reduced-motion` or opt-in: micro-motion → slow cross-fades between still poses; flight → cross-fade between perches; ambient leaf drift removed; day/evening color shifts retained but slowed. **Calls still play (or caption), birds still drift, mood still changes, the notebook still notices.** It is a different *register*, not "animations off," and not a stripped fallback (`accessibility_perf.md`). It ships **with** v1, not after.

### 6.5 Top-bar chrome

Thin top bar above the scene with exactly: account/settings, accessibility settings, field notebook, offer affordance. Nothing else. No chrome inside the scene (no inline labels/badges/tooltips/overlay icons). Top bar fades to near-transparent after a few seconds of cursor stillness, returns on cursor/keyboard activity (`aviary_layout.md`).

### 6.6 Day/night & weather rendering

Lighting phase follows the user's **local** time (sunrise warm-up, bright midday, warm quiet evening, dim night; nightjar species active at night). Weather is rare and never assertive (short rain a few times/week, soft wind) and is driven by server `weather_state` so it's coherent across devices; leaf/feather ornaments remain client-local.

### 6.7 Performance discipline in the pipeline

Interpolation math and audio scheduling run in a Web Worker to protect the main-thread 60fps budget. Object pools for birds/particles; no per-frame allocation; no per-call audio-buffer allocation (§8). Render loop is `requestAnimationFrame`, paused on hidden.

---

## 7. Audio pipeline

### 7.1 Procedural synthesis (non-negotiable)

Calls are synthesized client-side via WebAudio from the per-species motif library — **never recorded audio loops** (`bird_engine.md`, `accessibility_perf.md`). Looped audio is "the audible signature of dead software"; once a user hears the same call twice identically, the spell breaks.

### 7.2 Per-call variation & signature

Each emission varies (timing, pitch micro-variation, motif recombination) seeded per call, while preserving the species/bird **signature** so a user knows Pip by ear across mood and drift. Vocal-frequency trait shapes call density when unobserved and chorus readiness.

### 7.3 Chorus

≥2 high-vocal-frequency birds calling in the same window mix in **real time** into a true chorus — not stacked loops (stacking recorded loops phase-cancels audibly; §7.1). The mixer sums independent synthesized voices.

### 7.4 Listen-in mix

Focusing a bird raises its mix level and lowers others to ambient — **gradual ramps** on engage and disengage (slow rise/drop), never a hard cut, never full silence for the others (a re-balance, not a mute; `interactions.md`). Disengage on: click the focused bird again, focus a different bird, click empty space, or move keyboard focus away. Same slow ramp back to ambient.

### 7.5 Fallback (graceful silence + captions)

If WebAudio is unavailable (old browser, denied context, hardware): the aviary plays in **graceful silence with captions on by default** (`accessibility_perf.md`). **No recorded-audio fallback path exists** — unconditional rule. Captions (§ accessibility) carry the call information instead.

### 7.6 Memory & lifecycle

Audio buffers/nodes are reused (pooled); audio contexts bounded; no per-call allocation that isn't freed. This feeds the "no memory growth over 30 min" CI test (§8.4).

---

## 8. Accessibility surfaces

Designed for charm, **not parity-by-checklist** (`accessibility_perf.md`). All accessible surfaces ship with v1.

### 8.1 Screen-reader narration

Running **naturalist prose** (same voice as the notebook), generated from the same canonical state the visual reads, on a **slow cadence** (~1 update / 30–60s at idle; faster only on user-initiated events: return-greeting, accepted offer, settle). Not a state list, not "Pip mood: content," not ARIA-label automation. Implemented as an `aria-live="polite"` region fed server-generated (or client-generated from snapshot) prose; user-initiated events get a small priority bump but are still written as observations. Cadence cap protects the SR queue.

### 8.2 Reduced-motion

Covered in §6.4 — its own designed cross-fade rendering, full audio/drift/mood/notebook retained.

### 8.3 Call captions

Opt-in; default-on when audio is unavailable. Short naturalist prose per call ("a soft three-note rise", "a low trill, paused, low trill again"), appearing near the calling bird, fading with the call. **Generated at runtime from the same call-grammar parameters** that drove synthesis, so the caption matches the actual call — never a fixed per-call string (`accessibility_perf.md`).

### 8.4 Keyboard navigation

Tab cycles top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds; Enter triggers listen-in; Escape exits listen-in; offer opens via top-bar shortcut and is fully keyboard-navigable; settle reachable from the top bar. Focus indicator: soft high-contrast outline legible against both bright and dim aviary states.

### 8.5 Contrast

All user copy (top-bar labels, settings, account/error surfaces, captions, visually-displayed narration) passes **WCAG AA** minimum; design system specifies exact ratios. The scene carries no copy except the top bar, so the constraint lands on chrome.

---

## 9. Privacy, performance budgets & observability

### 9.1 Privacy as an architectural rule

Per-bird/per-account interaction events drive **only that user's own simulation** — never aggregated for training, recommendations, third parties, or population analysis (`accounts_sync.md`). Enforcement is architectural, not policy:

- telemetry-svc's DB role has **no read grant** on the simulation DB; the analytics warehouse never ingests `bird`/`event_log`/`notebook`; any future ML path is structurally cut off from per-bird fields.
- Synthetic account UUID everywhere; email encrypted in one place; no email in any key/partition/log/metric.
- A CI test asserts no metric definition carries a per-account or per-bird dimension.

### 9.2 Aggregate telemetry (allowed)

Request counts, latencies (incl. tick-compute latency), error rates, anonymized session-duration histograms (no per-account dimension), client render-frame timing, audio-context error counts. Privacy policy linked in account settings names these categories and explicitly excludes per-bird state.

### 9.3 Performance budgets

- **Initial JS bundle < 2MB gzipped** at first paint. Aggressive code-splitting for less-frequent surfaces (account settings, accessibility settings, visit-invite flow). Procedural assets/synthesis partly *because* of this cap.
- **Time to first bird < 500ms** on mid-tier mobile / 4G. Requires the bundle budget, an edge-inlined small first snapshot delivered with the HTML, and a render path that draws the first bird before non-critical assets load.
- **60fps idle on a 5-year-old laptop**, sustained over a 30-min session (runtime budget, not just launch).
- **No memory growth over 30 min** — a real CI test: audio buffers reused, notebook rows released on scroll-out, bounded workers/contexts.

### 9.4 Observability

Synthetic perf checks (scheduled headless browsers from several geographies running the aviary) + aggregate-only RUM (load/first-bird/frame timings, audio-context errors, tick latencies). **Error budget:** sim-tick latency p99 alarms at >5s (the tick should be far under; this catches degradation before users feel "slow"). Browser support: last two majors of Chrome/Safari/Firefox/Edge; older browsers get a matter-of-fact unsupported-browser surface.

---

## 10. Rollout

### 10.1 Shipping v1

Single coordinated launch; accessibility and reduced-motion ship **in** v1 (not v1.1). Magic-link auth, two-bird adoption, the engine, render/audio pipelines, notebook, presence, visits-off-by-default, and all performance/a11y budgets are launch gates.

### 10.2 Ramping birds-per-aviary

Cap stays at 7. Bird-offer cadence is age-based and tuned conservatively at launch (start the third-bird offer window generous, observe via *aggregate* health metrics only — never per-account inspection). The cap is built into the engine; raising it would require revisiting audio recognizability (out of scope for v1).

### 10.3 Day-one instrumentation

Instrument from day one (aggregate-only): bundle size in CI, time-to-first-bird (synthetic + RUM), frame timing, tick latency (p99 alarm), audio-context error rate, memory-growth CI gate. Drift calibration is observed through a **synthetic-account harness** (simulated presence schedules) in staging — never by reading real users' birds.

### 10.4 Calibration loop

`alpha` (drift time constant), presence activity window, tick cadence, and notebook sparsity are server-side config tuned against the synthetic harness and the named calibration targets (1wk instrument / 3wk perceptible; ≤1 notebook entry / 3 days). Changes ship as config, validated against the harness before rollout.

---

## 11. Risks

### 11.1 Drift calibration miss (highest engine risk)
Too fast → Tamagotchi-by-clicking; too slow → screensaver. **Mitigation:** explicit low-pass time constant + perceptibility quantization (§4.4) makes within-session invisibility structural; a synthetic-account harness replays presence schedules in CI and asserts the 1-week-instrument / 3-week-perceptible targets and the **monotonicity** invariant (idle 14 days → no trait decrease). This is the first thing built (week-0 spike) because everything else hangs off it.

### 11.2 Sync correctness / personality loss (worst-case failure)
Losing a vector = deleting the bird the user knows; silent and untestable by unit test alone. **Mitigation:** sole-writer DB grant, additive ordered deltas, idempotent consumption, per-aviary advisory lock, no client write path (architectural test in CI). Backups of the simulation DB are point-in-time recoverable; the `personality_seed` + ordered `event_log` give an audit trail, but the canonical persisted vector — not recomputation — remains source of truth.

### 11.3 Audio uncanniness
Procedural calls that sound synthetic, or a chorus that phase-cancels. **Mitigation:** week-0 audio spike to validate motif libraries, per-call variation, signature recognizability (a listening test: can a naive listener distinguish two birds after a short exposure?), and real-time chorus mixing. The "never identical twice / recognizable across drift" pair is tested by ear, not just by code.

### 11.4 Accessibility regressions
Narration drifting into state-list announcements; reduced-motion decaying into "animations off"; captions falling back to fixed strings. **Mitigation:** voice lint on narration/caption/notebook strings (flag announcement style), reduced-motion treated as a first-class rendering with its own visual QA, caption-matches-call test (caption generated from the same parameters as synthesis), keyboard-path E2E tests.

### 11.5 The "harmless feature" leak (product-integrity risk)
A toast, a streak, a "you've been gone X days," a visit notification on by default — each looks harmless and each breaks the product. **Mitigation:** the non-goals are encoded as *absences in the data model and telemetry* (no count/streak/aggregate tables), a copy lint that flags announcement-style strings and "welcome back"/"X days" patterns, and a PR checklist line: "does this announce, gamify, or surface user behavior? If yes, reject." The architectural absence (no cross-account metrics) makes leaderboards/discovery expensive to add, by design.

### 11.6 Performance budget erosion
Bundle creep past 2MB; first-bird past 500ms; frame drops on old hardware; memory growth. **Mitigation:** CI gates on bundle size and the 30-min memory test; synthetic first-bird timing in CI/staging; Web-Worker offload for audio/interpolation; object pooling; code-splitting deferred surfaces.

### 11.7 Privacy boundary erosion
An engineer reaching for email as a convenient key, or a telemetry event quietly carrying a bird field. **Mitigation:** synthetic-UUID-everywhere rule with a CI check that no log/metric/partition field is email-derived and no metric carries a per-account/per-bird dimension; telemetry-svc DB role physically lacks read access to the simulation DB.

### 11.8 Timezone / day-night correctness
DST transitions, travel, stale client-reported tz. **Mitigation:** store IANA tz on the aviary; resolve lighting from canonical tz on the server tick so all devices agree; update tz on a deliberate signal, not silently per-request, to avoid the aviary "jumping" time when a user travels.

---

## 12. Build sequence (suggested)

0. **Spikes (week 0–1, parallel):** drift-calibration harness + monotonicity test; audio motif/synthesis/chorus listening test; render-spike for "loads-in-motion" + first-bird-<500ms path; choose runtime/render tech against bundle budget.
1. **Core data model + sim-svc tick** with sole-writer grants, additive ordered deltas, idempotent consumption; synthetic-account harness green on calibration targets.
2. **aviary-svc** snapshot read + event-log write; **auth-svc** magic-link + sessions + synthetic UUID + encrypted email.
3. **Client render pipeline**: scene, three perches, day/night, interpolation, idle micro-motion, top-bar + fade, loads-in-motion, empty/quiet-field states.
4. **Audio pipeline**: synthesis, per-call variation, chorus, listen-in ramps, fallback-silence+captions.
5. **Interactions**: return-greeting (absence-length + boldness wired through, staggered multi-bird), offer (3 types + cooldown), settle (+5s undo), notebook (sparse, read-only), presence accounting (3-signal conjunction).
6. **Accessibility**: narration (slow, naturalist, prioritized events), reduced-motion designed rendering, runtime captions, keyboard nav, AA contrast.
7. **Accounts/privacy**: export, email-change, soft/hard deletion, session revoke, privacy-policy surface; telemetry-svc isolated.
8. **Visits**: invite/one-time link/read-only ambient proxy, revoke, expiry, silent visit log, opt-in notification toggle (off).
9. **Observability + budgets in CI**: bundle gate, 30-min memory test, synthetic first-bird, tick-latency p99 alarm, voice/announcement copy lints, architectural sole-writer + privacy lints.
10. **Hardening + calibration pass**, then v1 launch (a11y + reduced-motion as launch gates).

---

*Plan complete. The deliverable is this plan; the product is not implemented here.*
