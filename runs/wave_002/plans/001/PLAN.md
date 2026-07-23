# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable engineering program. It is written so a separate engineering team can build v1 without further clarification. Where the PRD is ambiguous, we make a defensible call and flag it as a **decision**.

---

## 1. Scope

### 1.1 In scope (v1)

- Single-user accounts, email magic-link sign-in, one aviary per account.
- 2 starter birds at adoption (system-selected species, user-named), growth to a hard cap of 7 birds via age-based offers.
- Server-side simulation tick (~1/min) as the only writer of canonical aviary state: personality vectors, moods, positions, call scheduling.
- Client rendering of snapshots with interpolation; presence accounting per the three-signal conjunction definition.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook (read-only, sparse, naturalist prose).
- Scene: single horizontal screen, three perch zones, local-time day/night cycle, rare ambient weather, ambient micro-motion, top-bar chrome with fade.
- Visits: per-invite opt-in, read-only ambient view, revocable, 30-day invite expiry, silent visit log, off by default.
- Accessibility: screen-reader naturalist narration, designed reduced-motion mode, procedural call captions, WCAG AA contrast, full keyboard navigation.
- Account export (JSON, emailed link) and soft-then-hard deletion (30 days).
- Aggregate-only operational telemetry, honoring the hard privacy boundary.

### 1.2 Out of scope (v1 and beyond, per `non_goals.md`)

- No native apps; web-only, last two major versions of Chrome/Safari/Firefox/Edge.
- No gamification of any flavor (no streaks, counters, badges, calendars of visits — not even opt-in).
- No Tamagotchi mechanics: no death, hunger, distress, decay. Drift is monotonic toward expressive.
- No social-network surfaces: no profiles, follows, feeds, discovery, leaderboards, comments, co-presence, chat, avatars.
- No payments, shared aviaries, multi-aviary accounts, customizable scenes, push notifications.
- No recorded audio anywhere in the product, including fallbacks.

---

## 2. Architecture

### 2.1 Service shape

Four services behind a single API gateway; one relational store as system of record.

1. **Auth service** — magic-link issuance/verification, session token issuance and revocation, email change, account deletion lifecycle, account export.
2. **Simulation service** — owns the canonical aviary state. Runs the per-aviary tick scheduler, consumes the interaction event log, computes personality deltas, mood transitions, call schedules, notebook entries. The only writer of personality vectors.
3. **State/read service** — serves state snapshots to clients (and visitors), keeps per-account snapshot cache at the CDN edge for the fast first load.
4. **Visit service** — invite issuance, expiry, revocation, visitor session authorization, visit log. (Small enough to fold into the state service; we keep it separate because it has its own authorization surface.)

Plus shared infrastructure: an append-only **interaction event log** (per-account ordered stream; a simple partitioned log table is sufficient at v1 scale — we do not need Kafka), a **snapshot cache** (edge-cached JSON), and an **email delivery** dependency (transactional email for magic links, invites, exports).

### 2.2 Client/server split — the load-bearing boundary

- **Server owns:** personality vectors, mood state and timers, perch positions, call scheduling intents, notebook entries, presence-time accounting, drift computation, weather scheduling, day/night phase (computed from the account's timezone, transmitted in the snapshot).
- **Client owns:** rendering, interpolation between snapshots, procedural call synthesis (WebAudio) from call-schedule intents, audio mixing (listen-in mix), ambient leaf/feather ornaments (pure client-side), presence signal detection (the three-signal conjunction), interaction event submission.
- **The rule with teeth:** clients never write personality state under any code path. Clients write events; the tick computes deltas. There is no API that accepts an absolute trait value. This is enforced at the API schema level — no such field exists in any request type.

### 2.3 Render pipeline boundary

The client receives a **snapshot** (see §4) and a stream of **call intents**. Everything the client draws is derivable from the latest snapshot plus elapsed wall-clock time; nothing in the visual scene requires server round-trips per frame. Ambient ornaments (leaves, feathers) and parallax are client-only and carry no simulation state. If the client goes hidden, rendering stops entirely; the simulation continues on the server.

---

## 3. Data model

All IDs are synthetic UUIDs. Email appears only on the account record, encrypted at rest. Nothing else in the schema references email.

### 3.1 Tables

**accounts**
`account_id (uuid, pk)`, `email_encrypted`, `email_hash (for magic-link lookup only)`, `timezone`, `created_at`, `deletion_requested_at (null)`, settings JSON (visit-notifications opt-in, reduced-motion override, captions opt-in, audio mute).

**sessions**
`session_id`, `account_id`, `device_label`, `issued_at`, `expires_at`, `revoked_at`.

**magic_links**
`token_hash`, `account_id`, `issued_at`, `expires_at (issued+15min)`, `consumed_at`. Single-use; consumption invalidates immediately. Per-email rate limit on issuance.

**aviaries**
`aviary_id`, `account_id (unique)`, `created_at`, `settled_until (null)`, `current_weather (null | rain | wind)`, `weather_ends_at`.

**birds**
`bird_id (uuid, pk, stable forever)`, `aviary_id`, `species_id`, `name`, `adopted_at`,
`personality: {boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}` — each a float in `[0,1]`, seed values in `[0.3, 0.5]` (**decision**: mid-low seeds leave headroom for three weeks of visible drift),
`mood: enum(wary, content, curious, drowsy, alert)`, `mood_updated_at`,
`perch_zone: enum(front, middle, back)`, `activity_state` (current animation anchor, e.g. preen/scan/rest), `last_call_at`, `offer_cooldown_until`.

**species** (static catalog, ~6 rows)
`species_id`, display name, silhouette asset ref, plumage palette, motif-library ref, nightjar flag (one species stays active at night).

**interaction_events** (append-only, per-account ordered)
`event_id`, `account_id`, `seq` (per-account monotonic), `type` (presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo, return_greeting_shown), `payload JSON`, `client_ts`, `server_ts`, `consumed_by_tick_at (null)`.

**presence_windows** (materialized by the tick from presence_ping events)
`account_id`, `window_start`, `window_end`, `seconds`.

**notebook_entries**
`entry_id`, `aviary_id`, `written_at`, `prose`, `generation_key` (idempotency key so re-ticks never duplicate an entry).

**visits**
`visit_id`, `host_account_id`, `visitor_email_encrypted`, `token_hash`, `issued_at`, `expires_at (issued+30d)`, `revoked_at`, `first_used_at`, `last_used_at`.

**visit_log_entries**
`visit_id`, `started_at`, `ended_at`, `approx_duration_s`. Written by the visit service on session boundaries; never pushed to the host.

**aviary_age_offers**
`aviary_id`, `offered_at`, `species_id`, `accepted_at (null)`. Driven purely by aviary age (see §10).

### 3.2 Invariants

- Bird identity is immutable: `bird_id` never changes; rename touches only `name`.
- Personality vectors exist only in `birds.personality`; they are never derived at runtime, never recomputed from logs, never exposed in any client-facing payload.
- `interaction_events` is append-only; nothing updates or deletes rows except account hard-deletion.
- Visitor sessions produce **no** rows in `interaction_events` and **no** presence windows.

---

## 4. API surface

All endpoints under `/api/v1`, JSON, session-token auth except magic-link and visit-token endpoints. Versioned snapshot schema.

### 4.1 Auth & account

- `POST /auth/magic-link` — `{email}` → 202. Rate-limited per email.
- `GET /auth/verify?token=…` — validates, marks consumed, issues session token (httpOnly cookie + bearer fallback for Safari ITP edge cases).
- `POST /auth/sign-out`; `GET /account/sessions`; `DELETE /account/sessions/{id}` (revoke).
- `POST /account/email-change` → verification link to new address; old email works until verify.
- `POST /account/export` → 202; export job renders JSON (birds incl. personality vectors — export is to the user themselves, so vectors included per PRD), emails download link.
- `POST /account/delete` → soft-delete; any signed-in page shows a recover action; `POST /account/recover`.
- `GET/PUT /account/settings` — captions, reduced-motion override, visit-notification opt-in, audio mute.

### 4.2 State pull

- `GET /aviary/snapshot?since_seq=N` → snapshot (see below). Long-poll optional at v1; the default is low-frequency polling (~30s keepalive while visible) plus pull-on-visibilitychange and pull-after-frame-gap. Snapshots are a few KB; at v1 scale polling is simpler and cheaper than websockets, and 30s staleness is invisible in a product whose tick is 60s (**decision**: no realtime push in v1; revisit if greeting latency feels stale).
- Snapshot payload: aviary-level (day/night phase, weather, settled state), per-bird (bird_id, name, species, perch_zone, activity_state, mood **as animation parameters only** — see note), call schedule (next intent per bird with time offset), notebook has-unread flag (no count badge — a simple "new entries" dot is the maximum permitted chrome; **decision**: we ship the dot, as the notebook is otherwise undiscoverable, and a passive dot is noticing, not announcing).

  **Note on mood exposure:** the client needs mood to render mood-shaped idle motion. We transmit mood as an opaque animation-parameter bundle (pose set, motion tempo, perch preference) rather than a labeled enum, so no code path can ever render "mood: content" as text. The label exists only server-side.

- `GET /notebook?before=…` — paginated entries, newest first, unbounded history.

### 4.3 Interaction events (client → server)

- `POST /aviary/events` — batched array of events `{type, payload, client_ts}`. Presence pings are heartbeat-batched (one ping per ~30s while the three-signal conjunction holds; the client stops sending the moment any leg fails). Responses are 202; effects appear in the next snapshot after the tick consumes them.
- Event types: `presence_ping`, `listen_in_start {bird_id}`, `listen_in_end {bird_id, duration_s}`, `offer {kind: seed|song|pool, song_id?}`, `settle`, `settle_undo`, `adopt {bird_id, name}`, `rename {bird_id, name}`.

### 4.4 Visits

- `POST /visits/invite {email}` → 202; email with one-time link. Per-invite opt-in, no bulk.
- `GET /visits` — host's outstanding invites + visit log (visitor email, date, approx duration). On-demand only.
- `DELETE /visits/{id}` — revoke; immediate. Active visitor sessions terminate on their next snapshot pull.
- `GET /visit/{token}/snapshot` — visitor read path. Returns the same snapshot shape as the host's but with interaction affordances stripped and `read_only: true`. Token checked for expiry/revocation on **every** pull.
- Visitor clients cannot call `/aviary/events` (403 by role).

---

## 5. Simulation engine design

### 5.1 Tick scheduler

A single scheduler process iterates aviaries due for a tick (~every 60s, jittered ±10s to smooth load; exact cadence calibrated in build). Each tick for an aviary:

1. Reads unconsumed interaction events in `seq` order; marks them consumed.
2. Materializes presence windows from pings: a window extends while consecutive pings are ≤90s apart; closes on settle, tab-close signal (pings stop), or gap.
3. Computes **personality deltas** (§5.2) and applies additively.
4. Runs **mood transitions** (§5.3).
5. Updates perch zones, activity states, call schedule.
6. Runs bird-to-bird coupling (§5.5).
7. Maybe writes a notebook entry (§5.6).
8. Advances/ends weather; computes day/night phase from account timezone.
9. Writes the new canonical state and invalidates the edge snapshot cache.

The tick runs whether or not any client is connected. Tick p99 latency alarm at 5s.

### 5.2 Drift function

Low-pass filter over presence-and-interaction signals. Per bird, per tick:

```
signal(bird) = w_p · presence_active                    (1.0 while any presence window open, else 0)
             + w_l · listened_in_this_tick              (listen_in_end events for this bird)
             + w_o · offer_accepted                     (offer where bird approached/engaged)
             + w_b · offer_nearby                       (offer while bird in any perch zone)
delta(trait) = clamp(step(trait) · signal(bird), 0, max_delta_per_tick)
personality[trait] = min(1.0, personality[trait] + delta(trait))
```

- Deltas are **non-negative**. There is no code path that decrements a trait. Neglect produces absence of signal, which produces absence of drift — ambient quietness, not regression. This is enforced by construction (clamp at 0) and by a CI invariant test that fuzzes the tick with adversarial event logs and asserts monotonicity.
- Trait targeting: presence-time lifts all traits gently (dominant weight); listen-in lifts `social_warmth` and `vocal_frequency`; accepted offers lift `curiosity`; any offer lifts `boldness` slightly. Settle contributes nothing beyond closing the presence window.
- Calibration targets (testable): measurable drift in instruments after ~1 week of regular visits; user-visible drift after ~3 weeks. Concretely: a "regular" user (20 min/day presence) should accumulate ≈ +0.05–0.08 per trait-week; a single 30-minute session moves any trait < 0.005 (below perceptibility). We build a headless calibration harness that simulates synthetic presence patterns and asserts both bounds before any tuning ships.

### 5.3 Mood transitions

Mood ∈ {wary, content, curious, drowsy, alert}. Transition scoring per tick per bird:

```
score(next_mood) = base_prior(next_mood, time_of_day)
                 + recent_interaction_bias      (accepted offer → +content, +curious)
                 + ambient_bias                 (rain → −vocal, wind → +alert or +wary per species)
                 + personality_modulation       (high boldness → −wary weight; high warmth → +content)
                 + contagion                    (nearby wary bird → +wary)
```

Softmax with temperature → stochastic transition. Mood persists across sessions (it's stored state); nothing resets on tab open. Time-of-day uses the account's timezone: drowsy weight rises near dusk, alert early morning, night pushes most birds to settled/drowsy except the nightjar species.

### 5.4 Call-grammar runtime

Each species has a motif library: a set of parameterized motifs (pitch contour, note count, tempo, timbre params). The server schedules **call intents** — `{bird_id, at_offset, motif_id, variation_seed, intensity}` — driven by `vocal_frequency`, mood, time of day, weather, and chorus opportunities (when ≥2 high-vocal birds have overlapping call windows, the scheduler emits a chorus cluster with staggered offsets). The client synthesizes audio from motif params + seed, so every rendering of a motif differs in micro-timing and pitch while remaining the same recognizable signature. Per-bird signature = species motif set + a per-bird pitch offset fixed at adoption — recognizability survives drift because drift changes *how often* and *how eagerly*, never the base timbre.

### 5.5 Bird-to-bird coupling

Within a tick: one bird's call raises the chorus-join probability of high-`vocal_frequency` birds in the same window; a wary transition in one bird adds wary weight to others; greeting order on return is boldness-weighted with randomized stagger offsets (0.3–1.2s) so simultaneous cues never fire in unison.

### 5.6 Notebook generation

The tick evaluates a small rule set over the day's state (first greeter changed, unusual quiet stretch, offer accepted after long wariness, weather reactions). At most one entry per generation opportunity; opportunities gated to roughly one per few days for a typical account, with a hard cap (≤1/day) to preserve sparsity for very active users. Entries are template-composed naturalist prose over real state (bird names, perch zones, times of day), **never** numeric, **never** about the user's behavior ("you visited…" is a banned pattern, enforced by a lint rule over the template set). Idempotent via `generation_key`.

### 5.7 Return-greeting computation

On the first snapshot pull after a presence gap, the server (in the tick path or on read, flagged `pending_greeting`) selects the greeter: boldness-weighted random among birds not drowsy; greeting form selected from absence-length bucket (minutes → glance; hours → call + look; days → approach + longer call) × mood; procedural variation seed attached. Exactly one bird greets first; a second bird may respond after a stagger. The greeting is emitted as an animation/call intent, never as text.

---

## 6. Sync model

- **One canonical record.** The simulation DB row per aviary is the only state. Both devices pull snapshots from the same record; there is nothing to merge and no client-to-client path.
- **No last-write-wins.** Personality updates are additive server-authored deltas applied in event-log order. Clients physically cannot submit absolute values (no such API field). A laptop session and an overlapping phone session both append events; the tick folds both into the same vector in `seq` order. The divergent-simulation failure mode is unreachable by construction.
- **Ordering:** per-account `seq` assigned at event ingest (single-writer per account via row lock or per-account partition), so the tick's consumption order is total and deterministic.
- **Conflict surfaces** (rare: replayed magic link, expired session mid-write, server error) use the matter-of-fact voice, verbatim style per `accounts_sync.md` examples.
- **Offline:** no offline mode. A client that can't reach the server shows the quiet-field loading state; no cached-aviary rendering that could present stale state as live (**decision** — stale rendering would fake continuity, worse than an honest quiet field).

---

## 7. Frontend rendering pipeline

### 7.1 Stack

Canvas 2D renderer for the scene (birds, perches, foliage, weather, lighting) with DOM only for top-bar chrome and accessibility surfaces. **Decision:** Canvas over SVG/DOM for the scene because 60fps idle micro-motion with per-bird pose blending is cheaper in immediate mode; the a11y surfaces are parallel DOM (see §9), so we don't lose semantics. Rendering is framework-light (small custom loop + a minimal UI lib for chrome) to protect the 2MB gzip budget. Code-split: settings, account, visit flow, notebook panel load on demand.

### 7.2 Scene composition

Single horizontal scene, three perch zones (front/middle/back) mapped to depth-scaled layers; background foliage/sky behind, occasional foreground branch in front; subtle parallax (≤4% layer offset). Responsive: scene scales to viewport with perches re-spaced; birds never cropped, never offscreen (clamp + reflow rule, tested at 320px–5120px widths).

### 7.3 First-frame rule (the load sequence)

1. HTML ships from the edge **with the first snapshot inlined** (state/read service pre-renders into the page at the edge for signed-in root requests) — the client never waits for a round trip before drawing.
2. First frame: birds placed at snapshot positions mid-activity, ambient drift already running, audio graph starts on first user gesture (autoplay policy; captions cover the pre-gesture window — **decision**: the gesture gate is a platform constraint we can't design away, so the pre-gesture scene is visual-only with captions if enabled).
3. Slow-path loading state is the **quiet field**: soft sky color, faint leaf drift, no spinner, no progress bar, no fade-from-static.
4. Budget: first bird visible <500ms on mid-tier mobile over 4G. Enforced by synthetic RUM gates in CI.

### 7.4 Idle micro-motion & transitions

Per-bird animation state machine (preen, scan, head-tilt, weight-shuffle, rest) keyed by the mood-parameter bundle and personality-modulated tempo. Interpolation between snapshots: position/pose lerped over the inter-snapshot window; no teleports. Greeting and offer reactions are intent-driven one-shots layered over idle. Return-from-hidden: pull snapshot, cross-fade current render into new state over ~600ms so a day of server-side change reads as "the aviary has been living," not a jump cut.

### 7.5 Top bar

Icons only: account/settings, accessibility, notebook, offer. Fades to ~10% opacity after 4s of cursor stillness; returns on movement/keyboard. No badges except the notebook dot (§4.2). Offer panel: seed, song-fragment library (small list, keyboard navigable), still pool. Settle lives here too, with the 5s any-click undo.

### 7.6 Reduced-motion mode

A separate render path, not a degradation: micro-motion becomes slow cross-fades between still poses (pose sets sampled from the same animation state machine, so mood is still legible); flight/perch changes become cross-fades; leaf/feather drift removed; day/night color shifts remain, slowed 2×. Triggered by `prefers-reduced-motion` or the settings override. Audio, captions, notebook, drift all unchanged.

### 7.7 Hidden-tab behavior

`visibilitychange` → stop the render loop and audio scheduling within one frame; presence pings stop (conjunction fails anyway); on return, pull fresh snapshot and resume. Laptop-suspend detection via frame-gap watchdog (>5s gap → resync).

---

## 8. Audio pipeline

### 8.1 Synthesis

WebAudio graph per bird: oscillator bank + filtered noise + envelope shaping, parameterized by motif params from the call intent (pitch contour, note count, tempo, timbre) and the per-bird fixed pitch offset. Variation seed perturbs micro-timing (±30ms), pitch (±2%), and amplitude per rendering — no two plays identical, signature preserved. Motif libraries ship as compact JSON parameter sets inside the bundle (kilobytes, not audio files).

### 8.2 Chorus mixing

Per-bird gain nodes into a master bus with a soft compressor. Default mix: birds balanced by perch depth (front louder). Chorus events are emergent from the server-scheduled call intents, not a client effect.

### 8.3 Listen-in mix

On listen-in engage: focused bird's gain ramps to ~1.6× over ~1.5s; others ramp to ~0.35× (ambient, never silent). Disengage (re-click, click empty space, focus another bird, keyboard focus leave) reverses the ramp. No hard cuts anywhere in the mixer. Rain applies a global low-pass + gain dip; night applies a global quiet curve (nightjar exempt).

### 8.4 Captions

Generated client-side from the same motif params being synthesized (note count + contour → prose template: "a soft three-note rise"), so caption always matches what played. Small text near the calling bird, fade in/out with the call, naturalist voice, WCAG AA contrast.

### 8.5 Fallback

WebAudio unavailable/denied → graceful silence, captions force-on (user can still disable). No recorded-audio fallback exists; this is unconditional.

### 8.6 Resource hygiene

Audio buffers preallocated and reused; no per-call allocation; context suspended on hidden tab; bounded node pool (7 birds max → bounded by design).

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

A visually-hidden `aria-live="polite"` region carrying running naturalist prose generated from the same snapshot state the canvas reads (same generator family as the notebook, so the voice is one voice). Cadence: one update per 30–60s idle; priority bump for user-initiated events (return-greeting, offer reaction, settle) — still written as observations. Rate-limited queue so we never flood the screen reader. The canvas scene is `aria-hidden`; all semantics live in the narration region + DOM chrome. We test with NVDA, JAWS, VoiceOver.

### 9.2 Keyboard

Tab → top bar → into scene (first bird focused) → arrows move between birds → Enter listen-in → Escape disengage. Offer panel and settle fully keyboard-operable. Focus indicator: soft high-contrast outline validated against both day-bright and night-dim scenes.

### 9.3 Contrast & text

All user copy AA minimum; the scene itself carries no copy. Captions and the notebook meet AA against a backing scrim so they pass regardless of scene brightness.

### 9.4 Shipping rule

Accessibility ships in v1, not after. Reduced-motion, narration, captions are launch-blocking features, not post-launch fixes.

---

## 10. Data lifecycle features

- **Adoption:** signup → name two system-selected starters → empty-aviary quiet field → first bird soft fly-in → never empty again.
- **Age-based offers:** server job checks aviary age; offers at ~3 months, ~6, ~10, ~14, ~18 (5 offers → 7-bird cap). Appears as a quiet top-bar affordance (notebook-dot style), never a modal, never a badge count. Acceptance adds the bird (system-selected species) at default seed personality.
- **Export:** on-demand JSON (birds incl. vectors, moods, notebook, settings), emailed link, expires 24h.
- **Deletion:** soft 30 days (recoverable on any signed-in page) → hard delete job scrubs every table incl. events, visit logs, and telemetry keys (aggregate telemetry is already accountless, so nothing to scrub there).

---

## 11. Performance budgets & observability

### 11.1 Budgets (CI-enforced, not guidelines)

| Budget | Limit | Enforcement |
|---|---|---|
| Initial JS bundle | <2MB gzip | bundle-size gate per PR |
| Time to first bird | <500ms (mid-tier mobile / 4G) | synthetic RUM gate, Lighthouse-style lab run per release |
| Idle motion | 60fps on 5-yr-old laptop, sustained 30 min | automated frame-timing soak |
| Memory | no growth over 30 min | heap-snapshot diff in soak test; fail on positive slope |
| Tick latency | p99 < 5s alarm | server metrics + paging |

### 11.2 Observability (aggregate-only)

Synthetic browser fleet (common geographies, scheduled) + aggregate RUM: page-load timings, first-bird-render, frame timings, audio-context errors, tick latencies, request counts/error rates, anonymized session-duration histograms. **Hard boundary:** no metric carries account_id, bird_id, or any per-account interaction dimension. This is enforced in the metrics schema layer (field allowlist), not by policy. Telemetry pipelines never read the simulation DB; the warehouse has no path to it. Privacy policy in settings names the aggregate categories in plain text.

---

## 12. Rollout

1. **Phase A (internal):** engine + renderer + audio behind a flag, staff accounts only. Calibrate drift with the headless harness against the 1-week/3-week targets; calibrate tick cadence, presence activity window (lean long, ~5 min), offer cooldown (~3 min per bird).
2. **Phase B (invite beta):** magic-link invites to a small cohort. Watch tick p99, first-bird timing, audio error counts, caption/narration quality review (human reads samples weekly).
3. **Phase C (general availability):** open signup. Bird cap stays 2 at adoption for everyone; age-offers begin at 3 months organically — no ramp needed since growth is age-gated. Visits ship off-by-default from day one.
4. **Day-one instrumentation:** all §11.2 metrics plus funnel-lite counts (signup → adoption complete → day-7 return) at aggregate level only. No per-account engagement dashboards — deliberately unbuildable under the telemetry allowlist.

---

## 13. Risks

1. **Drift miscalibration (highest risk).** Too fast → Tamagotchi feel; too slow → screensaver. *Mitigations:* headless calibration harness with hard 1-week/3-week assertions; monotonicity fuzz tests; per-environment drift time-scale knob for QA (time-compressed simulation, never shipped); post-launch we can retune weights server-side without clients noticing since deltas are additive.
2. **Presence-signal corruption.** Any laxer presence definition silently inflates population drift. *Mitigations:* the three-signal conjunction lives in exactly one client module with unit tests per leg and integration tests per pair; server-side sanity filter (presence window > 16h/day flagged as anomalous and excluded — catches keepalive hacks); calibration harness re-run whenever the presence module changes.
3. **Sync correctness regression.** The no-last-write-wins guarantee is enforced by API shape, but a future "optimize the tick" change could reintroduce absolute writes. *Mitigation:* schema-level constraint (no endpoint accepts trait values), codeowner review on simulation service, property test asserting tick output depends only on (prior state, ordered events).
4. **Audio uncanniness.** Procedural calls that sound synthetic-cheap break the spell faster than silence; identical-sounding variation breaks it slower. *Mitigations:* variation-seed entropy budget per motif (assert two renders of same motif+seed differ measurably; same motif different seeds differ perceptibly — human listening panel in beta); recognizable-signature test (pitch offset stability across drift); chorus phase-artifact listening tests.
5. **Accessibility regressions.** Canvas scene + live region is a fragile pairing; a renderer change can silently desync narration from visuals. *Mitigations:* narration generated from the same snapshot object the renderer consumes (single source of truth); automated screen-reader snapshot tests (assert narration text tracks known states); reduced-motion path has its own visual-regression suite so it doesn't bit-rot as the "secondary" path.
6. **Tick staleness at scale.** Scheduler backlog → aviaries visibly behind. *Mitigation:* p99 5s alarm, per-aviary tick partitioning, jittered cadence; graceful degradation = clients keep rendering last snapshot (already the normal case).
7. **Scope creep toward announcements/gamification.** The most predictable failure is a well-meaning toast or streak. *Mitigation:* non-goals quoted in the repo's CONTRIBUTING; PR template checkbox "adds no announcement, counter, or engagement surface"; notebook-template lint banning user-behavior observations.
8. **Privacy boundary erosion.** A "harmless" per-account metric in a debugging hurry. *Mitigation:* metrics field allowlist rejects unlisted dimensions at ingestion; synthetic UUID rule enforced by DB constraint (no email column outside `accounts`).

---

## 14. Key decisions made where the PRD was silent

- Snapshot transport is polling (~30s + visibility/gap triggers), not websockets, at v1 — staleness is invisible against a 60s tick.
- Mood reaches the client only as opaque animation parameters, never as a labeled value.
- Notebook gets a passive "new entries" dot as the single permitted chrome badge (discoverability vs. notice-never-announce trade-off; dot is noticing-scale).
- Pre-audio-gesture window (browser autoplay policy) is visual-only with optional captions; the greeting replays visually if it fired during the gate.
- Presence activity window starts at 5 minutes, calibrated in Phase A; offer cooldown starts at 3 minutes per bird.
- Age-offer cadence: months 3, 6, 10, 14, 18 → 7-bird cap.
- No offline cached-aviary rendering; honest quiet field instead.
- Visit service folded logically into state service at implementation time if headcount demands, but its authorization checks stay a separate, separately-tested module.
