# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable engineering program. It is written for a team that has not read the PRD; where the PRD leaves a decision open, this plan makes the call and flags it as such. The two product invariants that shape almost every technical decision below are: (1) **the server is the only writer of canonical aviary state** — clients submit events and render snapshots; and (2) **the product's value is affective** — every surface (visual, audio, accessibility, even loading) must read as "a place that was already running," and any fallback to stock app patterns (spinners, toasts, loops, state-list narration) is a product failure, not a polish gap.

---

## 1. Scope

### In scope for v1

- Single-user accounts; email magic-link auth; per-device revocable sessions; email change with verification; account export (JSON, emailed link); soft-delete (30 days) then hard-delete.
- One canonical aviary per account; two starter birds at adoption; cap of seven; new-bird offers gated on **aviary age only**.
- Server-side simulation tick (~1/min) advancing personality drift, mood, and ambient events whether or not a client is connected.
- Bird engine: hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift; fast-timescale mood (wary, content, curious, drowsy, alert as the working enum); bird-to-bird interaction (call/response, mood contagion, emergent chorus).
- Procedural call synthesis client-side via WebAudio from per-species motif libraries; per-bird recognizable call signatures; chorus mixing; listen-in mix re-balancing with slow ramps.
- Session interactions: return-greeting (one bird, varied by boldness/mood/absence length), idle presence accounting (visibility ∧ focus ∧ recent input), listen-in, offer (seed / song fragment / still pool, per-bird cooldown), settle (with 5-second undo).
- Field notebook: auto-generated, sparse (~one entry per few days), naturalist voice, read-only, infinite scrollback.
- Visual scene: single non-scrolling horizontal scene; three perch zones; local-time day/night cycle; rare ambient weather; client-side ambient ornaments (leaves, feathers); fading top bar (account/settings, accessibility, notebook, offer); quiet-field loading and empty states (no spinner ever).
- Multi-device sync as an architectural property (single canonical record; snapshot pull + event append).
- Social: per-invite, revocable, read-only visit by emailed one-time link; visit log in settings; visit notifications **off by default** with an opt-in toggle; 30-day invite expiry.
- Accessibility shipped at launch, not after: naturalist prose screen-reader narration on slow cadence; reduced-motion mode as a designed cross-fade rendering; runtime-generated call captions; WCAG AA contrast on all user copy; full keyboard navigation.
- Performance budgets as CI gates: initial JS ≤ 2MB gzipped; first bird visible < 500ms on mid-tier mobile / 4G; 60fps idle on a 5-year-old laptop sustained 30 minutes; zero memory growth over a 30-minute session; tick latency p99 alarm at 5s.
- Aggregate-only telemetry with a hard pipeline-level boundary against per-bird/per-account interaction data.

### Out of scope for v1 (enforced, not just omitted)

Native apps; any gamification surface (streaks, scores, badges, levels, visit calendars, "days visited" in any disguise); Tamagotchi mechanics (death, hunger, distress, decaying meters, negative drift); social-network surfaces (profiles, follows, discovery, comments, leaderboards, co-presence, chat, visitor avatars); payments; shared or multi-aviary accounts; customizable scenes; push/email notifications about the aviary; recorded-audio fallback; any UI exposing personality vector values.

Several of these get **structural enforcement** in this plan rather than policy enforcement — see §13 (lint rules on copy strings, absence of aggregation pipelines, API surface that cannot express the banned features).

---

## 2. Architecture

### Service shape

Three deployable units plus a static edge:

1. **API service** (stateless, horizontally scaled) — auth, snapshot reads, event-log appends, notebook reads, account/settings/visit CRUD, export and deletion jobs. HTTP + JSON.
2. **Simulation service** (the tick) — a scheduled worker fleet that advances aviary state. Logically one writer per aviary; physically a worker pool with aviaries sharded by `aviary_id` so exactly one worker owns an aviary's tick at a time (lease-based ownership, e.g. per-shard locks). The tick is the **only** code path in the entire system that writes personality vectors, moods, and bird positions.
3. **Notification/mail worker** — magic links, export links, visit invites, opt-in visit notifications. Isolated so mail-provider latency never touches the API path.
4. **Edge/CDN** — serves the app shell, and serves the **bootstrap snapshot** (see §8) so first paint doesn't wait on an origin round trip.

A single Postgres database (per-region, primary + replicas) holds canonical state. Append-heavy interaction events go to a separate events table (partitioned by day) rather than a separate broker at v1 scale; the tick reads events by `(aviary_id, seq)` cursor. This keeps "the event log" transactionally adjacent to the state it drives, which makes the no-lost-drift guarantee (§7) easy to prove. If event volume outgrows Postgres, the cursor abstraction lets us move the log to a stream later without changing tick semantics.

**Decision (PRD-open):** monolith vs. microservices — we ship API and simulation as two services from one codebase/monorepo, sharing the schema package. Two services because their scaling profiles differ (API scales with concurrent viewers, tick scales with total accounts); one codebase because the data model is shared and small.

### Client/server split

The split is asymmetric on purpose:

- **Server owns:** identity, personality vectors, mood state, bird positions at tick granularity, drift computation, notebook entry generation, day/night and weather scheduling, visit authorization.
- **Client owns:** rendering and interpolation between snapshots, idle micro-motion (cosmetic, seeded but stateless), procedural audio synthesis and mixing, ambient ornaments (leaves/feathers — explicitly not simulation state), presence detection, input capture, caption generation, screen-reader narration assembly (from server-provided observation fragments — see §10).
- **Client never:** writes any trait, mood, or position; ticks the simulation; merges state. There is no client code path that mutates canonical state. This is enforced by the API surface itself (§5): no endpoint accepts absolute state.

### Render pipeline boundary

The boundary is the **snapshot**: a small (~1–4KB) JSON document containing per-bird `{bird_id, species, name, perch_zone, position_anchor, mood, active_action, call_schedule_hints, plumage_params}` plus aviary-level `{phase_of_day, weather, settled, snapshot_seq, server_time}`. Everything visually richer than the snapshot — feather detail, motion curves, audio — is derived client-side from `(species, plumage_params, mood, personality-derived render hints)`. Personality is **not** in the snapshot; the snapshot carries only derived render hints (e.g., a `greeting_assignment`, perch tendencies are already baked into the server-chosen perch). This keeps the never-expose-the-vector rule enforceable at the wire level: the numbers physically never leave the server, so no client bug or devtools inspection can leak them.

---

## 3. Data model

All identifiers are synthetic UUIDs. **Email appears in exactly one column** (`account.email_encrypted`) and is never a key, partition value, log field, or telemetry dimension — this is a schema-review checklist item and a lint rule on log statements.

### Tables (canonical, Postgres)

**account** — `account_id (uuid pk)`, `email_encrypted`, `email_verified_at`, `created_at`, `deletion_requested_at (nullable)`, `settings (jsonb: timezone hint, captions, reduced_motion override, visit_notifications)`. Soft delete = `deletion_requested_at` set; hard delete = a scheduled job 30 days later that cascades everything.

**session** — `session_id`, `account_id`, `device_label`, `created_at`, `last_seen_at`, `revoked_at`. Listed/revocable in settings.

**magic_link** — `link_id`, `account_id`, `token_hash`, `expires_at (15 min)`, `consumed_at`. Single-use enforced by atomic consume.

**aviary** — `aviary_id`, `account_id (unique)`, `created_at` (this is "aviary age" — the only input to new-bird offers), `settled (bool)`, `last_tick_at`, `tick_seq`.

**bird** — `bird_id (uuid, stable forever)`, `aviary_id`, `species_id`, `name`, `adopted_at`, `call_seed` (stable per-bird randomness anchor so the call signature survives drift and re-renders), `retired (bool, always false in v1 — exists so future migrations never delete a row)`. The `bird_id` is the identity-continuity rule made concrete: no migration, sync, or species-pool change may ever re-issue it.

**personality** — `bird_id (pk/fk)`, `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (all `numeric` in [0,1]), `updated_tick_seq`. **Written only by the tick.** Seed values at adoption: species-typical center ± small jitter (implementation detail owned by simulation service per PRD).

**mood** — `bird_id (pk/fk)`, `mood (enum: wary|content|curious|drowsy|alert|settled_night)`, `mood_since`, `modifiers (jsonb: weather damping, recent-offer nudge, expiries)`. Written only by the tick. Persists across sessions by construction — nothing ever resets it on connect.

**interaction_event** (append-only, day-partitioned) — `event_id`, `aviary_id`, `seq (per-aviary monotonic)`, `session_id`, `type (presence_ping | listen_in_start | listen_in_end | offer | settle | unsettle)`, `bird_id (nullable)`, `payload (jsonb)`, `client_time`, `server_time`. No updates, no deletes (except account hard-delete). The tick consumes by `seq` cursor stored on `aviary.tick_cursor`.

**notebook_entry** — `entry_id`, `aviary_id`, `created_at`, `prose (text)`, `source_facts (jsonb, internal — the structured facts the prose was generated from, kept for regeneration/QA, never shown)`. Read-only to clients; infinite scrollback via keyset pagination.

**visit_invite** — `invite_id`, `aviary_id`, `visitor_email_encrypted`, `token_hash`, `created_at`, `expires_at (+30d)`, `revoked_at`, `first_used_at`. **visit_log** — `visit_id`, `invite_id`, `aviary_id`, `started_at`, `approx_duration`. Visitor identity shown to host as the email they themselves entered (re-decrypted from the invite row — visitor email is PII too and follows the same one-column rule).

**species_pool** (static config, versioned in code, ~6 species) — silhouette refs, plumage palette params, motif library ref, `nocturnal (bool)` (exactly one species true — the nightjar).

### Notes on the model

- **Presence-time is not stored as a counter the user could ever see.** Presence pings land in `interaction_event`; the tick folds them into drift and discards. There is no `days_visited`, no per-day rollup table. The absence of the table is the anti-streak rule enforced structurally — a future "harmless calendar" feature would need a schema migration, which is the speed bump we want.
- **Notebook `source_facts` discipline:** facts may describe the aviary ("pip greeted first," "rain at 9:12"), never the user's attendance pattern. The fact-extractor (tick-side) has an allowlist of fact types; "user visited N days" is not an expressible fact type.

---

## 4. API surface

All endpoints require a session token except auth and visit endpoints. Matter-of-fact voice for all error bodies. Versioned under `/v1`.

### Auth & account

- `POST /v1/auth/magic-link` `{email}` → 202 always (no account-existence oracle). Rate-limited per email.
- `POST /v1/auth/consume` `{token}` → session token + account bootstrap, or matter-of-fact error ("The link may have expired.").
- `GET /v1/account` / `PATCH /v1/account/settings` — settings include `captions`, `reduced_motion_override`, `visit_notifications`.
- `POST /v1/account/email-change` → verify-new-then-switch flow.
- `GET /v1/account/sessions` / `DELETE /v1/account/sessions/{id}`.
- `POST /v1/account/export` → 202; worker emails download link.
- `POST /v1/account/delete` / `POST /v1/account/restore` (the "I changed my mind" surface).

### Aviary state (read path)

- `GET /v1/aviary/snapshot` → the render snapshot (§2). Includes `snapshot_seq` and `greeting` block when the server decides a return-greeting applies (see §6). Served with short TTL; the **bootstrap variant** is pushed to CDN edge keyed by session for first-paint speed (§8).
- `GET /v1/aviary/notebook?before={cursor}` → keyset-paginated entries, prose only.

There is **no** endpoint that returns personality values, presence totals, visit counts toward any rank, or any per-bird numeric trait. Not under a debug flag, not under an admin scope in this service (operator tooling is a separate internal system with its own access controls and no user-facing exposure).

### Interaction events (write path)

- `POST /v1/aviary/events` — batched append: `[{type, bird_id?, payload, client_time}]`. Server assigns `seq`. Accepted types only; payloads schema-validated. Idempotency keys per batch so retries on flaky mobile networks don't double-record offers. Clients send presence pings at a slow cadence (~30s) while the presence conjunction holds (§6).
- Offers are validated server-side against the per-bird cooldown (a few minutes; start at 3, calibrate) — the client also disables the affordance locally, but the server is the enforcement point.

### Visits

- `POST /v1/visits/invites` `{email}` → mails one-time link. `GET /v1/visits/invites` / `DELETE /v1/visits/invites/{id}` (revoke; effective at visitor's next snapshot pull). `GET /v1/visits/log`.
- `GET /v1/visit/{token}/snapshot` — unauthenticated-but-tokened, **read-only**: identical snapshot the host would receive (same birds, same moods, same weather — no show-off rendering), minus anything account-scoped. Returns 410 with the matter-of-fact "visit no longer available" body once revoked/expired. **The visit path has no events endpoint at all** — visitor presence and interaction are structurally unrecordable, which implements "visitors don't drift the host's birds" as an absence rather than a filter.

### Adoption & new birds

- `POST /v1/aviary/adopt-starters` `{names: [a, b]}` — one-time; server picks the two species.
- `GET /v1/aviary/bird-offer` → present only when aviary age crosses the next threshold (server-computed schedule off `aviary.created_at`; e.g., ~3 months for the third bird, lengthening intervals after, cap 7). `POST /v1/aviary/bird-offer/accept` `{name}`.
- `PATCH /v1/birds/{id}` `{name}` — rename anytime; touches nothing else.

---

## 5. Simulation engine design

### The tick

Cadence: every 60s per aviary (configurable; calibrate during build). Each tick, the owning worker:

1. Loads aviary, birds, personalities, moods; reads `interaction_event` rows where `seq > tick_cursor`.
2. **Presence integration:** folds presence pings into presence-time for the window. A ping is only emitted by the client when visibility ∧ focus ∧ recent-input all hold, but the server additionally clamps: presence-time credited per tick ≤ tick interval (a client cannot replay pings to inflate drift), and pings from multiple devices in the same window credit the window once, not twice (presence is "the user was watching," not "how many screens").
3. **Drift:** computes per-trait deltas (below) and applies them additively. Writes `personality` with `updated_tick_seq`.
4. **Mood transitions:** evaluates the mood machine per bird (below).
5. **Ambient scheduling:** advances day-phase (computed per-account from the client-reported IANA timezone stored in settings; falls back to last-seen offset), rolls weather (target: rain ~2–3×/week for a few minutes, soft wind more often; Poisson-ish scheduling with per-aviary seed so two devices see the same weather), schedules upcoming autonomous calls and bird-to-bird exchanges as `call_schedule_hints`.
6. **Notebook fact extraction:** emits candidate facts (first-greeter changes, unusual quiet, weather moments, offer reactions) into a per-aviary buffer; a sparsity governor promotes at most ~one entry every few days (noteworthiness-scored, decaying threshold) and renders it to prose via the template-grammar system (§10 shares this machinery with narration).
7. Writes new snapshot materialization and advances `tick_cursor` **in the same transaction** as the personality/mood writes. Crash between any steps = the whole tick retries; events are never consumed without their effects being committed. This transactionality is the no-lost-drift guarantee.

Idle optimization: aviaries with no events since last tick and no client connected take a cheap "ambient-only" tick path (mood/time/weather, no drift math), and can be batch-ticked with coarser granularity (e.g., catch-up ticks computed lazily when a snapshot is requested, integrating elapsed time analytically). **Catch-up must be deterministic and time-integrated** — a user away two weeks gets the same end-state whether we ticked 20,160 times or integrated once, which is what makes lazily ticking dormant aviaries safe. This keeps tick-fleet cost proportional to active users, not total accounts.

### Drift function

Per trait, per tick: `trait += clamp(rate(trait) × signal, 0, per_tick_cap)`, where:

- `signal` is the weighted input sum for that trait this tick: presence-time (dominant, weights all traits modestly with plumage saturation keyed strongly to sustained presence), listen-in minutes on a bird (→ that bird's social warmth + vocal frequency), offer-accepted (→ curiosity), offer-near (→ boldness), settle (→ no drift; ends the presence window cleanly).
- `rate(trait)` implements the low-pass character: a slow base rate further damped as the trait rises (diminishing returns), so early weeks show clean instrument-measurable movement and the trait asymptotes rather than pinning at 1.0.
- **Monotonicity is structural:** there is no negative term in the equation. Neglect contributes zero signal, hence zero drift. Nothing in the codebase can decrease a trait (a property test asserts this over arbitrary event sequences).
- `per_tick_cap` plus the offer cooldown prevents single-session saturation.

**Calibration target (testable):** with a synthetic "regular visitor" profile (e.g., 20 min/day presence, a few listen-ins, occasional offers), traits move by an instrument-detectable margin (≥ ~0.02 absolute) within 7 simulated days, and by a render-visible margin (crosses at least one render-hint quantization band — perch tendency, greeting eagerness, plumage band) within ~21 days. A "heavy clicker" profile (max events, minimal presence) must drift **less** than the regular visitor — presence dominance is a tested property, not a stated one. These run as fast-forward simulation tests in CI against the real tick code.

**Expression of low expressiveness without negative drift:** greeting frequency, autonomous call rate, and approach behavior are computed from `trait × recency_factor(presence)`, where the recency factor eases toward a floor during absence and recovers within a session or two. The bird that was left for two weeks greets less *because less has been observed recently*, not because any trait moved down — and the floor guarantees it still greets sometimes. This implements "quieter, not punished."

### Mood machine

A per-bird stochastic state machine evaluated each tick. Transition pressure = time-of-day prior (drowsy near dusk, alert early morning, settled_night at night for non-nocturnal species) + recent-interaction nudges (accepted offer → content; alarm context → wary) + ambient events (rain → vocal damping + mild drowsy pressure; wind → alert or wary, personality-dependent) + contagion (a wary neighbor radiates wary pressure; two high-vocal-frequency birds in alert/content can lock into a chorus window) + personality gating (high boldness raises the wary threshold). Transitions are hysteretic (minimum dwell ~10–20 min except event-driven nudges) so mood reads as weather, not flicker. Mood is never reset on connect — the connect path only *reads*.

### Call grammar (server side of it)

The server schedules *when* birds call (from vocal frequency, mood, chorus windows, time of day, the nightjar's nocturnal schedule) and ships `call_schedule_hints` (next-call windows + motif-class + intensity) in the snapshot. The client synthesizes *what it sounds like* (§9). Split rationale: scheduling must be canonical (two devices must hear the same bird decide to call at roughly the same moment; a visitor must hear the host's real aviary), while waveform realization is per-client and seeded by `(call_seed, motif, mood)` so it's recognizably "Pip" everywhere without shipping audio.

---

## 6. Presence, greeting, and session semantics

### Presence detection (client)

A presence evaluator samples the conjunction: `document.visibilityState === 'visible'` ∧ `document.hasFocus()` ∧ `lastInputAt > now − WINDOW`. `WINDOW` starts at **4 minutes** (PRD says "a few minutes, lean long" — watching without moving is the product) and is a remote-config calibration knob. Input = pointermove/keypress/pointerdown/touch. While the conjunction holds, the client emits a presence ping every ~30s; the ping carries no content beyond `{type: presence_ping}`. The conjunction failing simply stops pings — there is no "presence ended" event needed, and the server treats ping-gap as the window closing. Settle and tab-close are therefore identical at the engine level by construction.

### Return-greeting

Server-decided, client-performed. When a snapshot pull follows a presence gap, the server computes a `greeting` block: which bird (weighted by boldness, mood — a drowsy bird may cede to the second-boldest; deliberately varied day-to-day via seeded jitter so it isn't always the same bird), greeting class scaled by absence length (glance < two-note call < approach-and-longer-call < re-orientation with second-bird response), and stagger offsets if a second bird responds (randomized 1–4s; never simultaneous). The client realizes the class procedurally — pose curves, call realization, timing jitter all drawn at runtime, never from a canned variant list. Greeting must begin within the first 1–2s of first render; it's part of the first-paint critical path (§8).

### Settle

Client triggers `settle` event + local lighting ramp (a few seconds). Any click within 5s sends `unsettle` and reverses the ramp. Server marks the aviary settled (calls quiet, evening lighting) until tab close or re-engagement. No drift effect beyond cleanly ending presence.

---

## 7. Sync model

Single-writer architecture makes sync mostly a non-feature, by design:

- **One canonical record** per aviary; the tick is the only writer of derived state; clients only append events and read snapshots. Two devices = two readers of one record. Nothing merges because nothing forks.
- **No last-write-wins anywhere in the state path:** the API accepts only events ("listened in to Pip for 3 minutes"), never absolutes ("boldness = 0.62"). The morning-laptop/lunch-phone overwrite hazard in the PRD is unreachable: both sessions' events land in one ordered log; the tick integrates both.
- **Snapshot freshness:** clients re-pull on `visibilitychange → visible`, on render-loop gap detection (frame delta > ~5s ⇒ machine slept), and on a slow keepalive (~30–60s) while visible. Between pulls the client interpolates: positions ease between snapshot anchors; mood changes cross-fade into idle-motion parameters; a bird at perch A then perch B renders a flight (or, reduced-motion, a cross-fade), never a teleport.
- **Event durability on flaky clients:** the event batch endpoint is idempotent (client-generated batch ids); the client queues events briefly offline and flushes on reconnect; presence pings are *not* queued (stale presence must not be back-credited — a ping older than its window is dropped server-side by `server_time`).
- **Conflict surfaces that remain** are auth-shaped, not state-shaped: expired magic links, revoked sessions, timed-out writes. All render in matter-of-fact voice per the PRD samples. There is no user-facing "sync conflict resolver" because the architecture has no conflicting writes to resolve.

---

## 8. Frontend rendering pipeline

### Stack and scene composition

- **Renderer:** Canvas2D/WebGL via a thin scene graph (PixiJS-class library or hand-rolled — decide in a one-week spike against the 2MB budget; the budget, not familiarity, decides). DOM/SVG only for top bar, settings, notebook, captions overlay.
- **Layers:** sky/background foliage (slow palette shifts, gentle parallax) → middle plane (perches, birds) → occasional foreground branch/leaf. Parallax subtle; explicitly capped.
- **Birds:** procedural/skeletal 2D — small skeletal rigs per species silhouette with parameterized plumage (palette + saturation band from the plumage trait's quantized render hint). No frame-by-frame sprite sheets for primary motion: micro-motion (preen, scan, head-tilt, weight-shuffle) is generated from per-bird seeded noise shaped by mood parameters, so it never loops detectably. Mood is readable from motion alone (wary = back perch + frequent scanning; content = preening; curious = head-tilts toward sound events; drowsy = low posture, fluffed) — this is the "no mood labels" contract.
- **Day/night & weather:** palette LUT keyed to local-time phase from the snapshot, interpolated client-side; rain/wind as light particle + foliage-ripple effects driven by snapshot weather state (same weather on every device).
- **Ambient ornaments:** leaves/feathers spawned client-side at idle cadence (stateless, unsynced — per PRD these are rendering ornaments, not simulation).
- **Responsive:** one scene scaled/recomposed across viewports; layout solver keeps all birds in frame at all sizes (compress perch spacing on narrow, widen on desktop). Never crop a bird.

### First paint (the load-state rule)

Target: **first bird visible < 500ms** on mid-tier mobile/4G, already mid-action.

- App shell + a **critical micro-bundle** (scene bootstrap + bird skeletal renderer + greeting performer, target < 300KB gz) inlined/preloaded from CDN edge; the bootstrap snapshot delivered alongside the HTML (edge-cached per session, ~few-KB) so render starts without an origin round trip.
- First frame places birds at snapshot positions **mid-pose** (poses derived from `active_action` + deterministic phase from server_time, so the pose is "wherever the motion would be right now") — no entry animation, no fade-from-static. Audio context, full motif libraries, notebook, settings are all code-split and loaded after first paint.
- Slow-path fallback: if the snapshot hasn't arrived by render-ready, draw the **quiet field** (soft sky gradient + one or two faint ambient cues). Never a spinner, never a progress bar, anywhere in the product — this is a lint-able rule (no spinner component exists in the codebase).
- Empty-aviary (post-adoption) uses the same quiet field; first bird enters with a soft fly-in; thereafter the empty state is unreachable.

### Runtime budgets

- 60fps idle on a 5-year-old mid-range laptop **for 30 minutes** — sustained, profiled in CI (headless trace on a throttled profile) and on a real reference device weekly.
- Zero memory growth over 30 minutes: object pools for ornaments and pose buffers; audio buffers reused (§9); notebook list virtualized with reference-dropping on scroll-out; bounded workers/audio contexts. Enforced as a CI test (heap snapshot delta after scripted 30-min session ≤ noise threshold).
- Hidden tab: rendering stops entirely (rAF naturally throttles; we additionally tear down the draw loop and audio), presence pings stop; simulation continues server-side; on visible, re-pull snapshot and resume mid-motion.

### Reduced-motion rendering (designed register, see §10)

A parallel pose-presentation path, not a flag that disables animation: micro-motion becomes slow cross-fades between held poses from the same pose generator; flights become perch-to-perch cross-fades; ornaments removed; day/night palette shifts retained but slowed. Same snapshot, same engine, different visual register — built as a first-class render mode with its own visual QA, shipped at launch.

---

## 9. Audio pipeline

### Synthesis

- Per-species **motif library**: small parameterized synthesis recipes (oscillator/FM/noise-shaped chirp primitives with pitch contours, timing envelopes) — code + parameters, not samples. Each bird's **call signature** = species motifs × stable `call_seed` (fixed timbre/pitch-center/ornament tendencies) × mood/trait modulation (rate, brightness, phrase length). Signature stability across mood and drift is a listening-test requirement: Pip is Pip by ear at week 0 and week 6.
- Runtime: WebAudio graph — per-bird synth voice → per-bird gain → spatial pan (subtle, by perch x-position) → chorus bus → master. Calls realized at the snapshot's scheduled windows with client-side micro-jitter; two birds calling = two live syntheses mixing naturally (the chorus is real, never layered loops; no recorded audio exists in the product).
- Performance: voices pre-allocated and pooled (cap ≈ bird count + ambience); buffers reused; synthesis parameters precomputed off the audio thread (AudioWorklet for the synth voices; main-thread fallback if worklet unavailable). Audio is loaded post-first-paint; the aviary may be visually alive ~a beat before sound fades in — fade ambience in gently rather than popping on.

### Listen-in mix

Focus bird → its bus gain ramps up while others ramp **down to ambient, never to zero** (e.g., −12 to −16dB target, never −∞), over a slow constant-power ramp (~1.5–2.5s, calibrate by ear). Disengage (click again, focus elsewhere, click empty space, keyboard blur) reverses with the same ramp. No hard cuts anywhere in the mixer — "listening, not channel-switching" is the acceptance criterion for this feature's review.

### Fallback

WebAudio unavailable/denied ⇒ graceful silence with **captions auto-enabled** (one-time matter-of-fact note in settings explaining why). No recorded-audio fallback path exists or will be built.

### Captions

Generated at realization time from the actual synthesis parameters (motif class, note count, contour, intensity, perch) through the naturalist phrase grammar: "a soft three-note rise," "a low trill, paused, low trill again." Rendered as small DOM text near the calling bird, fading with the call, AA-contrast against both day and night palettes. Because captions derive from what was actually synthesized, they never desync from the audio.

---

## 10. Accessibility surfaces

Shipped in v1, owned alongside each feature (no separate "a11y phase").

### Voice system (shared infrastructure)

One **naturalist phrase-grammar engine** powers notebook entries, screen-reader narration, and captions: structured facts in → composed lowercase present-tense prose out, with synonym/structure variation so repeated facts never render identically. Centralizing it keeps the voice continuous across surfaces (a screen-reader user moving from aviary to notebook hears one product) and gives copy review a single audit point. System-voice strings (auth, errors, settings, sync, unsupported-browser) live in a separate copy catalog with the matter-of-fact register; the two catalogs are distinct modules so the voice line is enforceable in review.

### Screen-reader narration

- An ARIA live region (polite) carrying running naturalist prose composed client-side from snapshot facts via the phrase grammar: scene-state observations every **30–60s at idle**, paced and deduplicated (don't re-describe an unchanged scene; vary phrasing when re-mentioning).
- User-initiated events (return-greeting, offer reaction, settle) get prompt narration via a priority queue — still written as observations ("pip hops down to inspect the seed"), never state transitions ("offer accepted").
- Never: trait values, mood labels as labels, coordinates, event-log dumps. The narration reads the same scene the eyes would.
- Birds are focusable elements with naturalist accessible names ("pip, a small grey bird on the front rail") — descriptions, not stat lines.

### Reduced-motion

Honors `prefers-reduced-motion` automatically; also a settings toggle (override either way). Implemented per §8 as a designed cross-fade register. Calls, captions, drift, notebook all unchanged. QA includes a vestibular-safety pass (no large translations, no parallax, no flicker) and an "is it still charming" design review — the mode has its own acceptance bar, not just a checkbox.

### Keyboard

Tab order: top bar items → aviary scene → first bird; arrows move between birds; **Enter = listen-in**, **Escape = exit listen-in**; offer affordance opens from top bar with full keyboard nav inside; settle reachable from top bar. Focus ring: soft high-contrast outline tested against brightest-day and darkest-night palettes (designer-specified). Top bar fade never hides the focused element — keyboard activity counts as activity and restores opacity.

### Contrast

All user copy (top bar, settings, errors, captions, visually-displayed narration) ≥ WCAG AA, verified per surface against the palette extremes in automated visual tests. Scene art itself carries no user copy, so the constraint concentrates on chrome.

---

## 11. Performance budgets and observability

### Budgets (CI-gated, regression = blocked merge)

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS bundle | ≤ 2MB gz (critical micro-bundle ≤ ~300KB) | bundle-size CI gate per PR |
| Time to first bird | < 500ms, mid-tier mobile profile, 4G throttle | synthetic trace in CI + lab device weekly |
| Idle frame rate | 60fps sustained 30 min, 5-yr-old laptop profile | throttled headless trace (smoke per PR, full nightly) |
| Memory | no growth over scripted 30-min session | heap-delta CI test (nightly) |
| Tick latency | p99 < 5s alarm threshold (typical ≪ 1s) | production alert |

### Telemetry — what we measure

Aggregate-only: request counts/latencies, tick compute latencies and queue depth, error rates, anonymized session-duration histograms (no per-account dimension), client render-frame timings, first-bird-render timings, audio-context error counts, audio-fallback activation counts, magic-link delivery latency, synthetic-browser fleet results from several geographies.

### What we deliberately do not measure

Per-bird state, per-account interaction history, presence totals per account, visit counts as engagement metrics, drift rates per account, anything that could reconstruct a user's relationship with their aviary — and we don't compute population aggregates over per-bird interaction data either ("average drift across accounts" is explicitly banned).

### Privacy boundary as architecture

The simulation/event database and the telemetry pipeline are physically separate: the metrics emitter is a separate module with a typed schema of allowed metric shapes, none of which admit `account_id`, `bird_id`, or event payloads as dimensions; the analytics warehouse has **no connection or credentials** to the simulation DB; no ETL job exists between them. Per-bird fields never reach any training or analytics system. Code review checklist + a CI check that greps metric emission sites for banned dimensions. The privacy policy in settings names the aggregate categories in plain text and states the exclusion.

---

## 12. Rollout

### Build sequencing (≈ two-pizza team, ~3 quarters to GA; phases overlap)

1. **Foundations (weeks 1–6):** schema + API skeleton; magic-link auth + sessions; tick worker with lease ownership and transactional event consumption; snapshot read path; drift + mood math behind fast-forward simulation tests; renderer spike (library decision vs. 2MB budget); audio synthesis spike (one species, signature-stability listening test). The two spikes are the highest-risk items and run first.
2. **Vertical slice (weeks 5–12):** two birds end-to-end — adoption, first paint < 500ms path, idle motion, presence accounting, return-greeting, procedural calls + chorus, listen-in, snapshot interpolation across two devices. Exit criterion: a cold tab on a throttled phone profile shows a bird mid-action in < 500ms and greets within 2s; the same aviary on two devices shows the same moods and weather.
3. **Full surface (weeks 10–20):** offers + cooldowns; settle/unsettle; notebook generation + sparsity governor; day/night + weather; species pool to six incl. the nightjar; reduced-motion register; narration + captions; keyboard nav; settings/account surfaces; export; deletion lifecycle.
4. **Social + hardening (weeks 18–26):** visit invite/revoke/log/expiry; visit read-only snapshot path; perf budget enforcement to green across the board; memory test stable; security review (magic-link flows, visit tokens, session revocation); accessibility audit with assistive-tech users (paid testing sessions, not just automated checks).
5. **Calibration beta (weeks 24–32):** a few hundred invited users across timezones. This phase exists primarily to calibrate what cannot be calibrated synthetically: drift feel (1-week instrument / 3-week visible targets against real presence patterns), presence WINDOW, notebook sparsity, greeting variety, call recognizability (in-beta listening surveys: "which bird called?" matched-pair tests), audio uncanniness reports. All knobs are remote-config so calibration doesn't require deploys.
6. **GA.**

### Ramping birds-per-aviary

Engine and audio mixer are built and tested for 7 from day one (chorus tests run at 7 voices), but **new-account offer schedules ramp**: at launch the age-gated schedule effectively yields up to 3 birds (aviary ages are young anyway); we extend the schedule toward 7 as chorus-recognizability listening data and audio-perf telemetry confirm the cap holds in the field. The cap itself (7) ships hard-coded in the engine.

### Day-one instrumentation

First-bird-render timing (the affective-perf bridge metric), tick latency and queue health, audio-context error and fallback rates, frame-timing distribution, magic-link delivery success/latency, error-surface display counts (a spike in matter-of-fact surfaces = something's wrong upstream), synthetic-fleet geographic checks. Nothing per-account.

### Launch checklist (product-invariant audit)

A pre-GA pass with sign-off per item: no spinner anywhere; no toast/banner/welcome text on return; no streak/visit-frequency surface in any disguise (including notebook copy audit against the "observations of the aviary, never of the user" line); no trait numbers reachable in any UI or API response; reduced-motion and narration shipping and charming; voice-register audit (naturalist vs. matter-of-fact on the correct surfaces, style-sample fidelity); neglect-return scenario QA (two-week-absent test account comes back to quieter-not-sad birds).

---

## 13. Risks and mitigations

**1. Drift calibration misses the band (top product risk).** Too fast = Tamagotchi; too slow = screensaver. *Mitigations:* drift math isolated behind a pure function with fast-forward simulation tests pinned to the 1-week-instrument / 3-week-visible targets; presence-dominance property test (heavy-clicker < regular-visitor); all rates in remote config; calibration beta phase dedicated to this; monotonicity as a property test so no calibration change can introduce negative drift.

**2. Presence signal corruption.** A lax presence definition silently inflates drift population-wide and no test catches it downstream. *Mitigations:* the three-way conjunction implemented as a single audited client module with unit tests per condition-combination; server-side clamps (per-tick credit cap, multi-device dedup, stale-ping rejection); a synthetic-browser canary that runs "tab open but unfocused for 48h" and asserts zero drift.

**3. Sync correctness / lost drift.** *Mitigations:* single-writer tick with lease ownership (no two workers tick one aviary); event consumption and state write in one transaction; no API accepts absolute state — the LWW failure mode is unrepresentable; idempotent event batches; chaos test that kills tick workers mid-tick and asserts no event is double-applied or dropped (replay from cursor must be idempotent: deltas computed from events, applied once per seq).

**4. Audio uncanniness.** Procedural calls that sound synthetic-cheap, or chorus that turns to mush, break the affective spine. *Mitigations:* audio spike first (week 1) with a kill-criterion listening test; signature-recognizability matched-pair tests at 2, 5, and 7 birds; mood-modulation kept within signature-preserving bounds; the 7-bird cap engine-enforced; a sound designer engaged from the spike, not post-hoc; beta listening surveys.

**5. Accessibility regressions / drift toward checklist-mode.** The cheap version (ARIA labels, animations-off) is always nearer than the designed version. *Mitigations:* narration and reduced-motion built on the same engines as the primary surfaces (phrase grammar, pose generator) so they can't silently fork; acceptance criteria written in product terms ("feels alive") with assistive-tech user testing before GA; reduced-motion in the visual QA matrix for every scene-affecting PR; a11y surfaces in the v1 launch gate — the product does not ship without them.

**6. Performance budget erosion.** 2MB and 500ms die by a thousand dependencies. *Mitigations:* per-PR bundle gate from week 1; critical micro-bundle isolated with an import-boundary lint (nothing heavy may be imported into it); first-paint path owns its own budget line; library choices made by measured spike, not default.

**7. Tick fleet cost/latency at scale.** Ticking every aviary every minute is O(total accounts). *Mitigations:* ambient-only cheap path for event-less aviaries; deterministic lazy catch-up for dormant aviaries (validated by equivalence tests: N small ticks ≡ one integrated tick); p99 alarm at 5s; shard-rebalance runbook.

**8. Voice erosion.** Gamified or announce-y copy leaks in through well-meaning contributions. *Mitigations:* two physically separate copy catalogs; banned-lexicon lint on the naturalist catalog ("achievement," "streak," "welcome back," "you've been," exclamation marks); notebook fact-type allowlist excludes user-behavior facts; the launch-checklist copy audit; this plan's §1 out-of-scope list referenced in CONTRIBUTING as binding.

**9. Magic-link and visit-token abuse.** Link replay, invite-token sharing, email enumeration. *Mitigations:* 15-min expiry + atomic single-consume; constant 202 on link request (no account oracle); per-email rate limits; visit tokens single-invite-scoped, revocation-checked on every snapshot pull, 30-day expiry; session list + revoke; security review in phase 4.

**10. PII leakage via convenience.** Email creeping into logs/keys/metrics. *Mitigations:* synthetic UUID rule enforced by schema review checklist, a log-field lint, and the typed metrics schema (no string dimensions that could carry email); both email columns encrypted; export and deletion jobs are the only flows that ever render an address outbound.

---

## 14. Open calls made in this plan (defensible defaults, flagged)

- Presence input window: **4 minutes**, remote-configurable (PRD: "a few minutes, lean long").
- Offer cooldown: **3 minutes per bird**, server-enforced, remote-configurable (PRD: "a few minutes").
- Mood enum: **wary, content, curious, drowsy, alert** (+ internal `settled_night`) (PRD: "exact set finalized in implementation").
- Third-bird offer at ~**3 months** aviary age, intervals lengthening to reach 6–7 around a year+ (PRD: "a few months old offers a third bird; a year-old aviary may have grown to five or six").
- Event log in **Postgres partitions with per-aviary seq cursor** rather than a message broker at v1 scale; cursor abstraction preserves a later migration path.
- Renderer library decided by **spike against the bundle budget**, not pre-committed here.
- Timezone source: client-reported IANA zone persisted in settings; last-known offset as fallback for server-side day-phase computation during absence.
- Visitor snapshot reuses the host snapshot shape minus account-scoped fields, over a tokened unauthenticated route; no separate "visit renderer" exists (structurally enforcing no-show-off-mode).
