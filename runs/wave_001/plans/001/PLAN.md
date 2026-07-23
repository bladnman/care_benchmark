# Pocket Aviary — V1 Implementation Plan

Prepared from the PRD set (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). This plan is written for a separate engineering team to execute without further clarification. Where the PRD is deliberately silent (exact ranges, cadences, thresholds), this plan makes a defensible starting call and marks it as a **calibration target** to be tuned in the harness, not guessed at in production.

Throughout: **bold "hard rule"** markers flag PRD constraints that must be enforced in code review and CI, not just documented.

---

## 1. Scope

### 1.1 In scope for v1

- **Platform:** web-only, modern browsers (last two major versions of Chrome, Safari, Firefox, Edge). No native clients, and no protocol concessions made for hypothetical native clients.
- **Accounts:** single-user accounts, one aviary per account, email + magic-link sign-in (15-minute expiry, single-use, rate-limited per email), per-device revocable sessions, verified email change, JSON account export, 30-day soft then hard deletion.
- **Birds:** two system-selected starter birds per new account; species pool of ~6; user-assigned, renameable names; aviary-age-gated adoption offers up to a hard cap of 7 birds per aviary.
- **Bird engine:** hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift driven primarily by presence; 5-state enumerated mood with daily-ish cadence and cross-session persistence; procedural per-bird call grammar; bird-to-bird interaction (call response, mood contagion, emergent chorus).
- **Interactions:** return-greeting (one bird, staggered, absence-length- and boldness-shaped, procedurally varied); listen-in (gradual mix re-balance, never mute); offers (seed, song fragment, still pool; per-bird cooldown of a few minutes); settle (opt-in, 5-second undo, engine-equivalent to tab-close); field notebook (auto-generated, read-only, sparse, indefinite scrollback); presence accounting (strict three-signal conjunction).
- **Scene:** single horizontal scene, no pan/zoom/scroll; three perch zones (front/middle/back) chosen by birds, never by the user; local-time day/night cycle with a nightjar-like nocturnal species; rare ambient weather (rain a few times a week, occasional wind); ambient micro-motion (leaves, feathers — client-side ornaments only); thin top bar (account, accessibility, notebook, offer) that fades to near-transparent on cursor stillness; first frame renders mid-motion; quiet-field loading and empty-aviary states.
- **Sync:** server-side simulation tick (~1/min) as the only writer of canonical state; snapshot-pull clients; multi-device coherence as an architectural property.
- **Social:** email-based visit invitations only — per-invite opt-in, default OFF, read-only ambient view, revocable with immediate effect at next snapshot pull, 30-day invite expiry, silent visit log, opt-in (default-off) visit notifications. No co-presence; visitor attention never feeds the host's drift.
- **Accessibility:** screen-reader naturalist narration (30–60s idle cadence, priority on user events), designed reduced-motion mode (cross-fade rendering, not animation-off), runtime-generated call captions, full keyboard navigation with visible focus indicators, WCAG AA contrast on all user copy. **Hard rule: accessibility ships with v1, not after.**
- **Performance:** <2MB gzipped initial JS; <500ms time-to-first-bird on mid-tier mobile over 4G; 60fps idle motion on a 5-year-old laptop for a 30-minute session; zero client memory growth over 30 minutes (CI-tested); client-side WebAudio procedural synthesis with graceful-silence-plus-captions fallback.

### 1.2 Out of scope (non-goals, enforced)

From `non_goals.md` and `product_brief.md`, treated as **hard rules** with a review checklist and (where mechanically possible) CI guards:

1. **No gamification, ever.** No achievements, streaks, levels, scores, badges, XP, ranks, tiers, visit counters, green-dot calendars, "you visited every day this week" surfaces — including in notebook prose, settings, exports-as-feature, or opt-in toggles. The notebook observes the aviary, never the user's behavior.
2. **No Tamagotchi mechanics.** No death, hunger, distress, decaying happiness. Neglect yields ambient quietness, never punishment. Drift never moves a trait downward.
3. **No social-network surfaces.** No profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments, avatars, chat, leaderboards — and no aggregation of the cross-account stats that would make leaderboards computable later.
4. **No native app at v1.**
5. **Also excluded:** payments, shared/multi-user aviaries, multi-aviary accounts, customizable scenes, push notifications of any kind, any numeric exposure of personality vectors (no stats panel, no debug toggle, no tier), any "welcome back" text/toast/banner/modal, recorded-audio call assets.

---

## 2. Architecture

### 2.1 Service shape

A **modular monolith** with three deployable processes sharing one TypeScript codebase and one Postgres database. This is the smallest shape that satisfies the hard architectural rules (server-only canonical state, tick runs with no clients connected) without distributing a system that doesn't need distribution yet.

```
┌─────────────────────────────────────────────────────────────┐
│  CDN edge (static assets + inlined boot snapshot)            │
└───────────────┬─────────────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────────────┐
│  api-server (stateless, N replicas)                          │
│   - auth (magic link issue/consume, sessions)                │
│   - snapshot reads (current canonical state)                 │
│   - interaction-event ingest (append-only, idempotent)       │
│   - notebook reads, adoption, invites/visits, account mgmt   │
│   - export generation + deletion workflows                   │
├──────────────────────────────────────────────────────────────┤
│  sim-worker (single-writer, partitioned)                     │
│   - 60s tick over active aviaries, sharded by account UUID   │
│   - consumes event log in order; writes personality, mood,   │
│     weather, call-schedule hints, notebook candidates        │
├──────────────────────────────────────────────────────────────┤
│  mailer (magic links, invites, export links) — queue-driven  │
└──────────────────────────────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────────────┐
│  Postgres (system of record) + object storage (exports)      │
└──────────────────────────────────────────────────────────────┘
```

**Technology calls (defensible defaults, replaceable at module boundaries):**

- **Language:** TypeScript end-to-end. Shared packages: `sim-core` (pure simulation functions — drift, mood, call-grammar parameterization, notebook/narration prose generation), `protocol` (snapshot and event schemas, versioned), `web` (client), `api`, `worker`. Sharing `sim-core` between server and client guarantees the screen-reader narration, captions, and notebook prose come from *one* prose generator, and lets the client pre-compute call render-plans identically to how the server parameterizes them.
- **Server runtime:** Node 22 LTS, Fastify (or equivalent thin HTTP layer). Chosen for shared-language velocity; the sim-worker is CPU-light at v1 scale (per-tick work is small linear algebra over ≤7 birds per aviary).
- **Database:** Postgres 16. All canonical state lives here. The interaction event log is a Postgres append-only table (partitioned monthly) — no Kafka at v1 scale; the tick consumes via `(account_id, seq)` watermarks. Migration tooling with reversible migrations only.
- **Client build:** Vite + TypeScript, no UI framework for the aviary scene (imperative Canvas 2D renderer with a small retained scene graph); a micro-framework or vanilla DOM for chrome surfaces (top bar, dialogs, settings). Rationale: bundle budget (<2MB gz) and full control over the render loop; the scene is one canvas, not a component tree.
- **Email:** transactional provider behind an internal `mailer` interface with idempotent send + delivery logging.
- **Hosting:** containerized behind a CDN; HTML served with the initial aviary snapshot inlined (see §10.2) from the edge.

### 2.2 Client/server split — the render pipeline boundary

The boundary is the PRD's central architectural rule made concrete:

- **Server owns:** personality vectors, mood, perch/position decisions, call scheduling parameters, weather, day/night phase computation inputs, notebook entries, all account/invite state. Server is the *only* writer of all of these. **Hard rule: no client code path mutates personality state — enforced by schema (no endpoint accepts it), by type (protocol package exposes no such message), and by review checklist.**
- **Client owns:** pixels and audio. Snapshot interpolation, idle micro-motion playback within server-declared activity states, ornamental layer (leaves/feathers — pure client-side, never simulated, never synced), listen-in mix levels, caption rendering, narration queue draining.
- **Shared (via `sim-core`):** call-grammar interpretation (server emits a *call render-plan*; client synthesizes it), prose generation templates (notebook server-side; narration/captions client-side from the same state and templates).

The client's render state is derived: `snapshot(t)` → interpolated presentation state at display time. Ornaments are layered on top and are explicitly out of the canonical model, per `aviary_layout.md`.

---

## 3. Data model

Postgres schema (logical; exact DDL in implementation). **Hard rule: every internal reference to an account is the synthetic `account_uuid`. Email appears exactly once — on the account record, encrypted at rest (application-layer encryption with key management, plus pgcrypto at column level is acceptable) — and never in logs, telemetry, partition keys, or error messages.**

### 3.1 Accounts and auth

- `accounts(account_uuid uuid pk, email_ciphertext bytea, email_hash bytea unique, created_at, timezone text, deleted_at timestamptz null, settings jsonb, notify_on_visit bool default false, reduced_motion_opt_in bool default false, captions_opt_in bool default false, audio_muted bool default false)`
  - `email_hash` (HMAC) allows login lookup without plaintext indexes. `deleted_at` non-null = soft-deleted; a nightly job hard-deletes at `deleted_at + 30d` (birds, events, notebook, invites, telemetry links — everything keyed by `account_uuid`).
- `sessions(session_id uuid pk, account_uuid fk, token_hash bytea unique, device_label text, created_at, last_seen_at, revoked_at null)` — per-device, revocable from settings.
- `magic_links(token_hash bytea pk, account_uuid fk, issued_at, expires_at (= issued+15m), consumed_at null)` — single-use; per-email issue rate limit enforced in the API layer.
- `email_change_requests(account_uuid fk, new_email_ciphertext, new_email_hash, token_hash, expires_at)` — switch commits only after new-address verification; old email works until then.

### 3.2 Aviary and birds

- `aviaries(aviary_id uuid pk, account_uuid unique fk, created_at, settled bool default false, weather_state jsonb, day_phase_hint text)`
- `species(species_id text pk, display_name, silhouette_asset_ref, palette jsonb, motif_library_ref, nocturnal bool)` — seed data, ~6 rows.
- `birds(bird_id uuid pk, aviary_id fk, species_id fk, name text, adopted_at, perch_zone smallint, position_x real, activity_state text, activity_started_at, mood text, mood_entered_at, boldness real, social_warmth real, vocal_frequency real, plumage_saturation real, curiosity real, drift_carry jsonb)`
  - Personality traits are scalar columns (normalized range, e.g. [0,1]; exact range/seed values are a **calibration target** owned by the sim team). `drift_carry` holds sub-threshold accumulated signal between ticks (see §5.2).
  - **Hard rule: `bird_id` is immutable and never reused.** Renames, migrations, and species-pool changes never replace a bird. Migration tests assert bird-id continuity.
- `adoption_offers(offer_id uuid pk, aviary_id fk, species_id fk, becomes_available_at, accepted_at null, declined_at null)` — generated purely from `aviary.created_at` thresholds (**calibration targets:** third bird ≈ 3 months, subsequent ≈ every 2–4 months, hard-capped so total ≤ 7).

### 3.3 Events, presence, notebook

- `interaction_events(event_id uuid pk, account_uuid fk, client_event_id uuid, bird_id uuid null, type text, payload jsonb, client_ts timestamptz, server_ts timestamptz default now(), session_id uuid, unique(account_uuid, client_event_id))` — append-only; `unique(account_uuid, client_event_id)` makes ingest idempotent against client retries. Types: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `reengage`, `rename`, `adoption_response`, `session_open`, `session_close`.
- `presence_windows(account_uuid fk, started_at, ended_at null, seconds int, source_session uuid)` — materialized by the tick from validated `presence_ping`s; the only presence representation the drift function reads.
- `notebook_entries(entry_id uuid pk, account_uuid fk, bird_ids uuid[], body text, created_at, generator_key text, source_ref jsonb)` — `generator_key` (e.g. `first_greeter_swap`) supports sparsity budgeting and dedup.
- **Internal-only:** `drift_observations(account_uuid fk, bird_id fk, trait text, value real, observed_at)` — append-only instrumentation written by the tick so the calibration harness can verify "measurable after one week" on synthetic accounts. Never exposed via any API, export excluded (see note), and **never** rendered. (Decision: the user's JSON export includes current personality vectors per `accounts_sync.md`; `drift_observations` is a derived internal log and is not part of the export.)

### 3.4 Social

- `invites(invite_id uuid pk, host_account_uuid fk, visitor_email_ciphertext bytea, token_hash bytea unique, status text check in ('outstanding','active','revoked','expired'), created_at, expires_at (= created+30d), revoked_at null)`
- `visit_log(visit_id uuid pk, invite_id fk, started_at, ended_at null, approx_seconds int)` — written by the visit session handler; surfaced read-only in settings.

### 3.5 Ops

- `tick_watermarks(account_uuid pk, last_event_seq bigint, last_tick_at)`; `weather_schedule(aviary_id fk, kind text, starts_at, ends_at)`; `export_requests(...)`; `deletion_queue(...)`.

---

## 4. API surface

Plain HTTPS + JSON, cookie-session auth (HttpOnly, SameSite=Lax) with per-device session tokens. Polling transport — no WebSockets at v1: the canonical cadence is 60s, snapshots are kilobytes, and polling on visibility-change + a 45–60s keepalive is sufficient and radically simpler to make correct. (SSE is a noted future option if keepalive load warrants it.) All API errors on system surfaces render in the **matter-of-fact voice** (hard rule); bodies carry a stable `error_code` for the client to map to copy.

### 4.1 Auth & account

- `POST /auth/magic-link` `{email}` → 202 always (no account enumeration). Rate-limited per email and per IP.
- `GET /auth/consume?token=…` → validates single-use, unexpired token; issues session; redirects to app. Expired/used/replayed → matter-of-fact error surface ("We couldn't sign you in. The link may have expired. Try requesting a new link.").
- `POST /auth/signout`; `GET /account/sessions`; `POST /account/sessions/:id/revoke`.
- `GET/PATCH /account/settings` (timezone, notify_on_visit, reduced-motion opt-in, captions opt-in, audio muted — **no** aviary state).
- `POST /account/email-change` + `GET /account/email-change/confirm?token=…`.
- `POST /account/export` → enqueues export; emails a time-limited download link (birds, names, current personality vectors, current moods, notebook entries, account settings).
- `POST /account/delete` → sets `deleted_at`; any signed-in page shows the recovery affordance during the 30-day window; `POST /account/recover` clears it.

### 4.2 Aviary state & events

- `GET /aviary/snapshot` → current canonical snapshot (schema versioned; see §4.5). Also inlined into initial HTML at the edge for first-paint (§10.2).
- `POST /events` `{events: [{client_event_id, type, bird_id?, payload?, client_ts}]}` → 202 + per-event acks (accepted/duplicate/rejected-with-code). This is the **only** write path for interaction data. Batched by the client every ~5s and on pagehide (via `sendBeacon`).
- `GET /notebook?before=<cursor>&limit=…` → reverse-chronological entries.
- `GET /offers/catalog` → the small static offer library (seed; song fragments as motif parameterizations, not audio files; still pool).
- `POST /adoption/respond` `{offer_id, accept, name?}`; `POST /birds/:id/rename` `{name}`.

### 4.3 Visits

- `POST /invites` `{visitor_email}` → creates + emails one-time link. `GET /invites` (outstanding + history). `POST /invites/:id/revoke` → immediate; the visitor's next snapshot pull gets `410 visit_unavailable` and the matter-of-fact surface.
- `GET /visit/:token` → validates; on success issues a short-lived **visit ticket** scoped `read:snapshot` only.
- `GET /visit/snapshot` (ticket auth) → same snapshot shape minus anything user-private beyond what the host sees; the visit client posts **no** events (the API rejects event writes from visit tickets at the gateway level — belt-and-braces with the client simply never sending them). Visit tickets are checked against invite status on every pull, so revocation takes effect within one poll interval.
- `GET /account/visit-log` → who/when/approx duration + outstanding invites.

### 4.4 Event semantics the engine depends on

- `presence_ping` payload: `{visible: bool, focused: bool, last_activity_age_ms: int}`. The client sends a ping every 15s **only while all three conditions hold** (visibilityState == visible AND window focus AND pointer/key activity within the calibrated window — starting value 3 minutes, leaning longer per `interactions.md`; **calibration target**). Server-side plausibility validation: reject pings claiming >60s of presence per 60s of wall clock, clamp overlapping windows, ignore pings with non-monotonic client_ts from one session.
- `listen_in_start/end`, `offer {kind, bird_id}`, `settle`, `reengage` (the 5-second settle-undo click), `session_open/close`.

### 4.5 Snapshot format (v1)

```jsonc
{
  "schema": 1,
  "server_time": "…",
  "aviary": { "day_phase": "morning", "lighting_t": 0.32, "weather": {"kind":"rain","ends_in_s":140}|null, "settled": false },
  "birds": [{
    "bird_id": "…", "name": "Pip", "species": "warbler",
    "perch_zone": 0, "position_x": 0.71, "mood": "curious",
    "activity": {"kind":"preen","started_ms_ago":4200},
    "next_call_hint_ms": 9000,           // scheduling hint; render-plan fetched on fire
    "plumage_render_tier": 2              // derived from saturation; numbers never shown
  }],
  "greeting": {"bird_id":"…","style_seed":"…","delay_ms":600} | null  // only on session_open
}
```

**Hard rule:** the snapshot never contains raw personality values. `plumage_render_tier` is a bucketed render hint; trait numbers exist only in the sim DB and the user's own export.

---

## 5. Simulation engine design

`sim-core` is a pure, deterministic-given-seed function library; the `sim-worker` is its driver. Per-account RNG is seeded `(account_uuid, tick_index)` so replays of the same event log reproduce identical states — this property is CI-tested and is the backbone of drift-calibration science.

### 5.1 The tick

- Cadence: **60s** (calibration target; keep in [30s, 120s]).
- Driver: sim-worker iterates aviaries sharded by `account_uuid`; per aviary, per tick:
  1. Read events since `tick_watermarks.last_event_seq` (ordered by `(server_ts, event_id)`).
  2. Validate & fold `presence_ping`s into `presence_windows`.
  3. Apply drift deltas (§5.2) to personality vectors with `drift_carry`.
  4. Run mood-transition step (§5.3) per bird, including time-of-day (account timezone) and weather inputs.
  5. Run bird-to-bird step (§5.5).
  6. Advance/schedule weather (§5.6); advance perch/activity decisions; emit call-schedule hints.
  7. Generate notebook candidates (§5.7) under the sparsity budget.
  8. Write all state in one transaction; bump snapshot version; write `drift_observations`.
- Runs with zero connected clients — the aviary genuinely continues. Tick latency is budgeted: p99 < 5s alarm (§10.3); expected per-aviary tick cost is milliseconds, so a single worker handles thousands of aviaries before sharding matters.

### 5.2 Drift function

- **Shape:** per-trait low-pass filter over interaction signals. Signals per tick: presence-minutes (dominant), listen-in minutes per bird, offer-accepted per bird, offer-near per bird. Settle contributes nothing directionally (it ends the presence window cleanly — that's its whole drift role).
- **Update rule (starting point, calibration target):** `trait += clamp(rate_trait × signal_weight × minutes_in_window, 0, max_delta_per_tick)`, with a per-trait half-life on the order of **10–20 days of typical use** and hard per-day caps so no single marathon session moves a trait visibly. `drift_carry` accumulates sub-cap signal so slow-but-steady users are not rounded down to zero.
- **Monotonicity (hard rule):** trait deltas are never negative. Neglect produces *ambient expression*, implemented not as trait decay but as **expression scaling**: behavior probabilities (greeting likelihood, chorus-join likelihood, call base rate) are multiplied by a recency-weighted presence factor that relaxes toward a quiet baseline when presence is absent and recovers with returning presence. A bird ignored for two weeks reads as quieter on return — with its traits intact.
- **Calibration targets (testable):** on the synthetic harness (§11.3), a "regular visitor" persona (≈10 min/day) must produce per-trait movement detectable by `drift_observations` within **7 simulated days**, and movement crossing the user-visible behavior thresholds (greeting-first frequency shift, perch-zone distribution shift, plumage tier change) within **21 simulated days**. A "marathon" persona (4h in one day, then nothing) must **not** cross visible thresholds. CI runs these personas against `sim-core` on every build.

### 5.3 Mood

- Per-bird enumerated state: `wary, content, curious, drowsy, alert` (final set owned by implementation; start with these five).
- Model: semi-Markov with per-state base durations and input-modulated transition scores. Inputs: recent session events (offer accepted → nudge `content`; offer ignored while `wary` → stay), local time-of-day curve (dusk → `drowsy`, early morning → `alert`), weather (rain → brief vocal dampening; wind → split `alert`/`wary`), ambient events (another bird's alarm call → `wary` contagion), and personality modulation (high boldness lowers `wary` transition scores).
- **Daily-ish cadence:** a soft re-centering pull during the local night hours (not a hard reset — a pull toward the time-of-day-appropriate state), so moods feel fresh each day without snapping.
- **Persistence (hard rule):** mood is a stored column; session boundaries never reset it. The tick advances mood through the user's absence (drowsy-at-dusk → settled/sleeping overnight), so the user never sees a snap-to-default on tab open.

### 5.4 Call grammar runtime

- Per species: a **motif library** (tens of motifs; each motif = a compact parameterization: pitch contour anchors, syllable timings, amplitude envelope, timbre recipe for the WebAudio voice). Per bird: a fixed **identity seed** that selects timbre formants and a motif subset — this is what keeps Pip recognizable as Pip across mood and drift (**hard rule: identity seed immutable for the life of the bird**).
- Runtime generation: server picks `(bird, motif, variation_seed, timing_params)` at tick time as call-schedule hints; when a call fires, the client (or server for the notebook/caption path) instantiates a **render-plan**: concrete note list with per-note pitch/duration/level, varied within the motif's tolerance by `variation_seed`, with timing density shaped by `vocal_frequency` and pitch offset shaped subtly by mood. **Every call is a fresh variation — no two calls render identically (hard rule).**
- The same render-plan drives three consumers: audio synthesis (§8), caption text (§9.3), and (when noteworthy) notebook phrasing. One plan, three expressions — guarantees "the caption matches what was actually played."

### 5.5 Bird-to-bird interaction

- Call events enter a short rolling window; other birds' response probability is a function of `social_warmth`, `vocal_frequency`, and current mood. Two or more high-`vocal_frequency` birds calling in the same window trigger a **chorus event** (concurrent overlapping calls) — emergent, never scripted.
- `wary` contagion: an alarm-class call raises `wary` transition scores for nearby birds for a short window. `content` is mildly contagious during chorus.
- Greeting arbitration (for return-greeting): candidate greeters scored by `social_warmth × boldness × absence-length factor × mood suitability`; winner greets, others stagger by randomized small offsets if they also greet. **Hard rule: never a simultaneous unison greeting.**

### 5.6 Weather & ambient events

- Scheduler per aviary: rain 2–3×/week (short, minutes), occasional wind; never thunderstorms/snow; effects brief and small (vocal dampening, alert/wary nudges). Night: most birds drift `drowsy`/settled; the nocturnal (nightjar-like) species remains active. Weather is canonical (server-scheduled) so all devices and visitors see the same sky.

### 5.7 Notebook generation

- Tick computes **observation candidates** from state diffs: first-greeter swaps vs. trailing week, unusual perch choices, long quiet stretches, preening streaks, weather reactions, chorus events, offer episodes.
- **Sparsity budget (hard rule):** exponentially-decaying credit allowing ≈1 entry per 2–3 days median for regular users, hard-capped (e.g. ≤3/week) even for hyperactive aviaries; candidates ranked by novelty against recent `generator_key`s; below-threshold candidates discarded silently.
- Prose: template slots + lexical variation in `sim-core`'s prose module, naturalist voice (lowercase, present-tense, bird-named, specific; e.g. "tuesday — pip greeted before wren today, first time this week."). **Hard rule: generator templates may reference birds, weather, light, time — never the user, never visit frequency, never trait numbers.** CI lint over template strings bans the user-observing patterns listed in `interactions.md` §"No streak counter".

---

## 6. Sync model

- **One canonical record.** All canonical state lives in Postgres, written only by the tick (birds/personality/mood/weather) or by transactional API handlers (names, invites, settings). Multi-device sync is therefore a non-feature: every signed-in client pulls the same snapshots. There is no client-to-client sync, no merge, no CRDT, no eventual consistency.
- **No last-write-wins, structurally (hard rule):** personality updates exist in exactly one code path — the tick's additive delta application, consuming the event log in `(server_ts, event_id)` order. There is no API message type that carries trait values. A device that goes silent mid-session loses nothing: its already-ingested events are consumed by the next tick; its un-sent events are simply absent (and batched retry with `client_event_id` dedup makes reconnects safe).
- **Conflict surface:** the only user-visible "conflicts" are auth/session failures (expired/replayed magic link, timed-out session, server error) — all rendered matter-of-fact. Data conflicts as a category are designed out, not resolved.
- **Client pull discipline:** snapshot on load (inlined), on `visibilitychange → visible`, after render-loop gaps > threshold (laptop suspend), and on a 45–60s keepalive. Snapshot `schema` and monotonic `state_version` let the client detect and recover from staleness by re-pulling.
- **Visitors** read through revocable tickets; revocation is effective at next pull (≤ one poll interval). Visitor sessions write nothing and accrue nothing.

---

## 7. Frontend rendering pipeline

### 7.1 Stack and scene composition

- Canvas 2D renderer with a retained layer graph (back-to-front): sky/lighting gradient → background foliage → back perch zone → middle zone (birds + perches) → front zone → foreground ornaments (passing branch/leaf) → weather overlay. Subtle parallax offsets between the three scenic layers only.
- Birds: compact vector sprite parts (body, head, wing, tail, eye) posed by a skeletal-ish transform rig — plumage detail scales with `plumage_render_tier`. Silhouettes per species keep birds distinguishable at a glance. Assets are small SVG-derived path data or compact bitmaps, consistent with the 2MB budget.
- Top bar and all dialogs/settings are **DOM** (real focus, real semantics), visually floating above the canvas, fading to ~5% opacity after ~3s of cursor stillness, restoring on any pointer/keyboard activity.

### 7.2 Boot sequence — "already in motion" (hard rule)

1. Edge serves HTML with critical CSS, a tiny boot script, and the **inlined current snapshot**.
2. First paint draws the scene from the snapshot with activity clocks backdated by `started_ms_ago` — birds appear mid-preen, mid-call; ambient ornaments start immediately; audio engine initializes and (per autoplay policy, §8.4) begins as soon as allowed.
3. If the snapshot cannot be inlined or is slow: render the **quiet field** (soft sky gradient of the correct local time-of-day, one or two faint motion cues) and pop birds in place when the snapshot lands — never a spinner, never a fade-from-static, never an entry animation. (The single permitted entrance: the adoption fly-in from the empty-aviary state, once per bird.)
4. Return-greeting fires within the first 1–2s per the snapshot's `greeting` block.

### 7.3 Motion system

- Per-bird idle state machines keyed to mood: `wary` → back-zone perch bias, scan cycles; `content` → preen bouts; `curious` → head-tilts toward sound sources and drifting leaves; `drowsy` → low posture, fluffed feathers, slow blink. Micro-motion runs continuously while the tab renders; procedural blink/fidget jitter ensures no two idle loops look alike.
- Transitions: perch changes are short flight arcs (server decides *that* a bird moves; client choreographs *how*). Interpolation between snapshots smooths any server-side position change — never a teleport.
- **Hidden tab:** render loop stops (rAF ceases; we also explicitly suspend), simulation continues server-side; on return, a fresh snapshot replaces state and motion resumes mid-stream.
- Responsive: scene scales to viewport with per-zone anchoring; **hard rule: all birds in frame at every viewport** — compression before cropping, minimum/maximum viewport behaviors defined in the rendering spec.

### 7.4 Reduced-motion mode (a designed surface, hard rule)

Triggered by `prefers-reduced-motion` or the accessibility setting. The same state renders in a cross-fade register: pose-sequence stills cross-fading slowly (preen as 3–4 dissolving poses), perch changes as cross-fades, no leaf/feather drift, day/evening color shifts retained but slowed. Audio, drift, mood, notebook, captions, narration all unchanged. Implemented as an alternate `MotionDriver` behind one interface, selected at boot and switchable live from settings.

---

## 8. Audio pipeline

### 8.1 Synthesis

- All calls synthesized client-side via WebAudio from the render-plan (§5.4). **Hard rule: zero recorded audio assets, at any fallback tier.**
- Per-bird voice chain: excitation source (oscillator/noise blend per timbre recipe) → formant filtering (identity seed) → amplitude/pitch envelopes per note → per-bird gain node → chorus bus → master (with user volume/mute). Buffers and nodes are pooled; no per-call allocations that outlive the call (memory budget, §10.1).
- Chorus emerges from concurrent per-bird voices; the mixer adds gentle bus compression so a 7-bird chorus stays legible and per-bird signatures remain recognizable (the empirical basis of the 7-bird cap).

### 8.2 Listen-in mix

- Engage (click/tap/keyboard Enter on a focused bird): focused bird's gain ramps up over ~1.5s; others duck over the same window to an **ambient floor (~25–35% level, never silence — hard rule)**. Disengage (re-click, focus another bird, click empty space, keyboard focus moves away) ramps back symmetrically. Mix state is client-local; `listen_in_start/end` events feed drift.

### 8.3 Call scheduling on the client

- Client consumes `next_call_hint_ms` plus per-bird base rates and fires render-plans with small local jitter; on each snapshot, hints re-sync. If the client is closed, calls simply don't render — the canonical state doesn't depend on rendered audio.

### 8.4 Autoplay & the WebAudio fallback

- Browsers may block audio before a user gesture. Policy: attempt `AudioContext` start on load; if blocked, run the aviary silently with a one-time, non-toast, in-chrome affordance (a small speaker state in the top bar) that unlocks audio on first interaction; from then on audio starts with the tab. This is a platform constraint surfaced quietly, **not** an announcement surface.
- If WebAudio is unavailable or errors: graceful silence + **captions default ON** (hard rule). No recorded fallback path exists.

---

## 9. Accessibility surfaces

Planned from day one, shipping with v1 (hard rule). An accessibility review is a launch gate, not a follow-up.

### 9.1 Screen-reader narration

- An `aria-live="polite"` off-DOM narration region carries running naturalist prose generated by the shared prose module from the same snapshot state the canvas renders. Idle cadence: one update per 30–60s (randomized within band, matching the visual rhythm); user-initiated events (return-greeting, offer reactions, settle) get a polite priority bump. Prose style: "a small grey bird is perched on the front rail, calling softly." — never state-lists, never trait values.
- Implementation: a `NarrationQueue` (dedup, coalesce, min-interval) fed by the snapshot-interpolation layer, so narration describes what is *on screen*, not raw state.

### 9.2 Reduced motion — §7.4.

### 9.3 Call captions

- Opt-in via accessibility settings (default-on only in the WebAudio-fallback case). Caption text is generated from the call's render-plan ("a soft three-note rise", "a low trill, paused, low trill again") by the shared prose module, rendered as small text near the calling bird, fading with the call. Because caption and sound derive from one plan, they always agree.

### 9.4 Keyboard & focus

- Tab → top bar items → into the scene: first bird focused; arrow keys move between birds; Enter toggles listen-in; Escape exits; offer dialog and settle reachable from the top bar and fully keyboard-operable. Focus indicators: soft high-contrast outlines specified against both bright and dim scene states. All interactive DOM has real roles/labels; canvas birds are exposed via an off-DOM list mirroring focus.

### 9.5 Contrast & voice

- WCAG AA minimum on all user copy (chrome, dialogs, captions, narration if ever displayed, settings, errors). The naturalist/matter-of-fact voice split is enforced in the copy deck and review: product surfaces naturalist; sign-in, settings, sync errors, accessibility settings matter-of-fact.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced, not guidelines)

| Budget | Threshold | Enforcement |
|---|---|---|
| Initial JS | < 2MB gzipped at first paint | bundle-size CI gate; aggressive code-splitting for settings/invite/account flows |
| Time to first bird | < 500ms on mid-tier mobile / 4G | synthetic WebPageTest-style runs per release; edge-inlined snapshot; render path never blocks on non-critical assets |
| Idle motion | 60fps sustained on 5-year-old laptop, 30-min session | synthetic long-session runs with frame-timing capture |
| Memory | no growth over 30 min | CI heap-delta test with pooled audio buffers; notebook virtualization frees scrolled-out references |
| Sim tick | p99 < 5s | server metrics + alarm |

### 10.2 First-paint path

Edge HTML → inlined snapshot → first bird drawn without waiting for JS chunk graph beyond the boot renderer; chrome, audio engine, and settings hydrate after first paint.

### 10.3 What we measure (and what we deliberately never measure)

- **Collect (aggregate only):** request counts/latencies/error rates, sim-tick latencies, page-load and first-bird timings, render-frame timings, audio-context error counts, anonymized session-duration histograms (no account dimension). Synthetic checks from common geographies on a schedule.
- **Never collect (hard rule, enforced at the pipeline level):** per-bird state, per-account interaction history, anything reconstructing a user's relationship with their aviary. Telemetry schemas live in a separate repo module with explicit field allowlists; the telemetry pipeline has no network or IAM path to the simulation database; the analytics warehouse never reads it; no aggregate "average drift" dashboards exist. Drift calibration happens on **synthetic harness accounts**, never production data. The privacy policy (plain text, linked in settings) names the collected categories and the exclusions.

---

## 11. Rollout

### 11.1 Phases

1. **Foundations (wks 1–4):** repo, protocol package, Postgres schema, auth/magic-link, event ingest + watermarks, snapshot endpoint, deploy skeleton, CI gates (bundle size, banned-pattern lints).
2. **Engine core (wks 3–8, overlapping):** `sim-core` drift/mood/weather/call-scheduler + tick worker; synthetic-time calibration harness; drift persona CI.
3. **Client scene (wks 5–10):** renderer, boot path, motion system, day/night, weather visuals, top bar; quiet-field and empty-aviary states.
4. **Audio (wks 7–11):** WebAudio voices, grammar interpreter, chorus mixing, listen-in, captions-from-render-plan, fallback.
5. **Interaction & notebook (wks 8–12):** presence pipeline, offers with cooldowns, settle (+undo), return-greeting arbitration, notebook generator + sparsity.
6. **Accessibility (wks 9–13, parallel, not trailing):** narration queue, reduced-motion driver, keyboard/focus, contrast pass — launch-gated.
7. **Accounts/sync/social (wks 10–14):** sessions mgmt, export, deletion, invites/visits/visit-log, revocation path.
8. **Hardening & launch (wks 13–16):** perf gates green, synthetic fleet live, error budgets armed, closed beta (staff + invited), then open web launch.

### 11.2 Bird-count ramp

- Launch: 2 starters, cap 7 in engine but adoption offers unlocked purely by aviary age (third ≈ 3 months; **calibration target**). Before any pacing change, run the chorus-recognizability listening protocol (can a two-week user pick each bird's call from a mixed chorus?) — the cap follows recognizability, not the reverse.

### 11.3 Day-one instrumentation

- Synthetic performance fleet; aggregate RUM; tick-latency alarms; audio-error counters; plus the internal `drift_observations` harness running synthetic personas continuously so calibration drift (in the statistical sense) is caught before users feel it. Feature flags: global kill switches for weather and notebook cadence; visits ship dark-launchable but per-account default-off forever.

---

## 12. Risks

| # | Risk | Why it matters | Mitigation |
|---|---|---|---|
| R1 | **Drift miscalibration** — too fast → Tamagotchi; too slow → screensaver | The product lives in the narrow band; failure is silent and only felt by users weeks in | Synthetic-persona CI with the 7-day-measurable / 21-day-visible gates; per-day delta caps; expression-scaling separated from traits so "quiet when neglected" doesn't require trait decay; post-launch tuning only via harness, never prod data (privacy rule) |
| R2 | **Presence-signal corruption** — laxer client check inflates population drift | `concepts.md` names this the load-bearing precision | Three-signal conjunction implemented in one audited module; server plausibility validation; activity-window length biased long; CI test that background-tab and unfocused-window scenarios produce zero presence |
| R3 | **Sync correctness regression** — a future endpoint accidentally accepts trait writes | One LWW path silently deletes drift with no log | Protocol package exposes no trait-carrying write type; schema-level rejection; architecture review gate for any new event type; replay-invariance property test (same log → same state) |
| R4 | **Audio uncanniness** — procedural calls read as synthetic or repetitive | One repeated-sounding call breaks the spell permanently | Motif-library breadth budget per species; per-call variation enforced by render-plan test (N calls ⇒ N distinct plans); identity-seed recognizability listening tests; chorus phase-artifact checks (the reason loops are banned) |
| R5 | **Accessibility regressions** | Accessible surfaces are the actual product for those users; retrofitting is the named failure mode | a11y launch gate; narration cadence/voice snapshot tests; reduced-motion visual-diff tests; keyboard-path e2e in CI; axe-style automated contrast checks |
| R6 | **"Notice, never announce" erosion** — a well-meaning toast ships | One announcement reframes the whole session | Banned-pattern lint over product-surface copy (welcome/streak/visit-frequency phrases); copy-deck review; product-surface vs system-surface voice checklist in PR template |
| R7 | **Personality-vector loss or bird-identity break** | The worst invisible failure: a reset bird un-reveals itself slowly | Point-in-time recovery on the sim DB; bird-id continuity asserted in migration tests; user export as an additional escape hatch; identity seed + traits treated as immutable-key data |
| R8 | **Tick staleness at scale** — users return to an aviary that "ran slow" | Breaks the continuing-without-you conceit | p99 5s alarm; per-shard lag metrics; worker autoscale trigger on watermark lag |
| R9 | **Scope creep toward gamification/social** | The compounding failure every adjacent product slides into | `non_goals.md` quoted in the PR template; any metric proposal cross-checked against the never-collect list (no leaderboard-computable stats exist to expose) |
| R10 | **Notebook voice collapse into event-log** | One "session started at 7:43" unmasks the whole voice as performance | Template lint; sparsity budget prevents dilution; editorial review of generator output samples each release |

---

## 13. Notable calls made where the PRD was open

1. **Tick cadence 60s** and **presence activity window ~3 min (biased longer)** — flagged calibration targets.
2. **Canvas 2D + DOM chrome** over WebGL/SVG-DOM — budget and 60fps control.
3. **Polling over WebSockets/SSE** — canonical cadence is 60s; polling is simpler to keep correct.
4. **Personality vectors included in the user's data export** (per `accounts_sync.md`) while remaining numerically invisible in-product; internal `drift_observations` excluded from export.
5. **Trait monotonicity + separate expression scaling** as the mechanism for "quieter on return after neglect" without downward drift.
6. **Autoplay-blocked audio** handled as silent start + quiet chrome affordance — a platform constraint, explicitly not an announcement surface.
7. **Drift calibration runs on synthetic harness accounts only**, reconciling the calibration targets with the aggregate-telemetry privacy boundary.
