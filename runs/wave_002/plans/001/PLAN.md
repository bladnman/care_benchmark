# Pocket Aviary — v1 Implementation Plan

This plan translates the Pocket Aviary PRD into an executable engineering program. It is written for a team that has not read the PRD; where the PRD leaves a decision open, this plan makes a defensible call and flags it as **[call]**. Constraints inherited from the PRD's design philosophy (notice-never-announce, no gamification, monotonic drift, procedural audio, presence precision, PII isolation) are treated as acceptance criteria, not aspirations — several appear below as explicit CI gates and review checklists.

---

## 1. Scope

### In scope for v1

- Single-user accounts; email + magic-link sign-in; per-device revocable sessions; email change with verification; account export (JSON, emailed download link); soft-delete (30 days) then hard-delete.
- One canonical aviary per account, advanced by a server-side simulation tick (~1/min), with multi-device sync as a read-only-client property of that architecture.
- Bird engine: hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); monotonic-toward-expressive drift driven primarily by presence-time; fast-timescale mood (enumerated set, persisted across sessions); bird-to-bird interaction; stable bird identity; ~6-species pool; age-gated bird offers from 2 starters up to a hard cap of 7.
- Interactions: return-greeting, idle presence accounting (visibility ∧ focus ∧ recent-input), listen-in audio focus, offers (seed / song fragment / still pool) with per-bird cooldown, settle gesture with 5-second undo, read-only field notebook with sparse naturalist entries.
- Visual scene: one non-scrolling horizontal scene; three perch zones; local-time day/night cycle; rare ambient weather; client-side ambient ornaments (leaves/feathers); top bar (account/settings, accessibility, notebook, offer) with idle fade; load-with-motion-in-progress (no spinner; "quiet field" fallback).
- Audio: client-side procedural call synthesis via WebAudio; per-bird recognizable call signatures; real-time chorus mixing; listen-in mix re-balance; graceful-silence-with-captions fallback when WebAudio is unavailable. No recorded audio anywhere.
- Social: per-invite, email-named, revocable, 30-day-expiring read-only visits. Visitor presence/interactions never feed the host's simulation. Visit log in settings; visit notifications off by default with a per-account opt-in toggle.
- Accessibility shipped with v1, not after: naturalist screen-reader narration (slow cadence, event priority bumps), reduced-motion as a designed cross-fade rendering, runtime-generated call captions, full keyboard navigation, WCAG AA contrast on all user copy.
- Performance budgets as launch gates: initial JS bundle <2MB gzipped; first bird visible <500ms on mid-tier mobile/4G; 60fps idle on a 5-year-old laptop for 30+ minutes; zero memory growth over a 30-minute session (CI-enforced).
- Privacy: synthetic UUID account identifiers everywhere except the encrypted email field on the account record; per-bird interaction data used only for that account's simulation; aggregate-only operational telemetry with a pipeline-level boundary.

### Out of scope for v1 (non-negotiable)

Native apps; gamification of any form (streaks, badges, levels, counters, visit calendars — including "harmless" disguises in settings or notebook copy); Tamagotchi mechanics (death, hunger, distress, decaying meters); social-network surfaces (profiles, follows, discovery, comments, leaderboards, co-presence); push/email notifications about the aviary; payments; shared or multiple aviaries; customizable scenes; exposing personality numbers to users in any surface at any tier, ever.

Two scope rules with teeth, enforced in review (§12): (1) no surface may describe the user's behavior (only the aviary's); (2) no announcement-register UI (toasts, banners, welcome modals, badges) anywhere in the product surface.

---

## 2. Architecture

### Service shape

Three server-side components plus a static-asset edge, deliberately small:

1. **API service** (stateless, horizontally scaled): auth (magic link issuance/consumption, session tokens), state-snapshot reads, interaction-event writes, account settings, export, deletion, visit invitation/revocation, visit-session reads.
2. **Simulation service** (the tick): a scheduled worker fleet that advances each aviary's canonical state ~once per minute. Sole writer of personality vectors and canonical bird state. Consumes the per-account append-only event log in order.
3. **Notebook/narration composer**: generates field-notebook entries (sparse, server-side, during tick processing) and the prose templates the narration system draws from. Runs inside the simulation service as a post-tick step — it reads the same state transition the tick just computed, so observations are grounded in actual state changes. **[call]** Narration prose for the live screen-reader surface is composed client-side from snapshot data using a shared grammar library (see §9), while notebook entries are server-authored and persisted; this keeps narration latency low and keeps the notebook canonical.
4. **Edge/CDN**: serves the app shell and, critically, the first state snapshot (see §10 on time-to-first-bird).

### Client/server split — the one rule everything hangs on

The server is the only writer of canonical aviary state. Clients are renderers and event reporters:

- Clients **pull** snapshots and interpolate.
- Clients **push** interaction events (offer, listen-in start/end, settle, presence pings) into an append-only log.
- Clients **never** write personality, mood, perch position, or any canonical field. No code path exists for a client to submit an absolute state value. This is enforced by API design (there is no endpoint that accepts state), not by convention.

### Render pipeline boundary

The client is split into: (a) a thin **state layer** (snapshot fetch, interpolation, event queue with offline buffering), (b) a **scene renderer** (Canvas/WebGL, see §8), (c) an **audio engine** (WebAudio graph, see §9 audio), (d) **chrome** (top bar, settings, notebook, auth — ordinary DOM, code-split). The state layer is the only module that talks to the network; renderer and audio engine consume a single in-memory "current aviary frame" object it maintains. This boundary is what lets reduced-motion mode swap the renderer without touching state or audio.

### Tick mechanics

- Tick cadence ~60s per aviary **[call: start at 60s fixed, jittered ±10s per account to flatten load; calibrate during build]**.
- Scheduling: aviaries are sharded by account UUID across worker partitions; each worker sweeps its partition each minute. Aviaries with no client connected and no recent events still tick (mood/time-of-day must advance), but the tick for a fully-idle aviary is cheap (no event log to consume) — budget <5ms compute for idle aviaries, so the fleet cost scales with events, not accounts.
- Each tick: read events since last tick → compute presence-time accrued → compute drift deltas → apply mood transitions (time-of-day, ambient events, recent interactions, personality modulation) → schedule/update ambient weather → maybe emit notebook entry → write new canonical state + tick watermark, atomically.
- Idempotency: each tick writes a monotonically increasing `tick_seq` and the event-log offset it consumed through. A re-run tick (worker crash/retry) re-reads from the recorded offset and produces the same deltas; state writes are conditional on `tick_seq` to prevent double-apply.

---

## 3. Data model

All tables keyed by synthetic UUIDs. The email exists in exactly one column, encrypted at rest (application-layer envelope encryption), on `account`. No other table, log line, message, or telemetry event ever carries email. This is enforced by a lint rule on log statements and a schema review gate, not just policy.

**account** — `account_id` (UUID, PK), `email_encrypted`, `email_hash` (HMAC, for sign-in lookup only — keyed hash so the raw email is not derivable; the HMAC key lives with the encryption keys), `created_at`, `deletion_requested_at` (nullable; drives 30-day soft-delete), settings JSON (visit-notification toggle, accessibility prefs that should roam, caption preference).

**session** — `session_id`, `account_id`, device descriptor (user-agent summary, no fingerprinting), `created_at`, `last_seen_at`, `revoked_at`.

**magic_link** — `link_id`, `email_hash`, `token_hash`, `expires_at` (15 min), `consumed_at`. Single-use enforced by atomic consume.

**aviary** — `aviary_id`, `account_id` (1:1 at v1, but modeled as its own entity so the id, `created_at` (drives age-gated bird offers), `tick_seq`, `event_log_offset`, current weather state, and settled-state flag live off the account record).

**bird** — `bird_id` (stable forever; the identity rule), `aviary_id`, `species_id`, `name`, `adopted_at`, **personality vector** (5 floats, normalized 0–1), `mood` (enum), `mood_since`, `perch_zone`, current pose/motion descriptor, call-grammar seed (per-bird stable seed that makes the signature recognizable). Personality is stored here and only the tick updates it. **[call]** Also store per-trait `drift_accumulator` floats — sub-visible drift accrues continuously but is folded into the canonical trait on tick; this keeps single-tick writes small and makes drift-rate instrumentation trivial.

**interaction_event** (append-only log) — `event_id`, `aviary_id`, `session_id`, `type` (presence_ping | listen_in_start | listen_in_end | offer | settle), `target_bird_id` (nullable), payload (offer kind, etc.), `client_ts`, `server_ts`, monotonic `log_offset` per aviary. Never updated, never deleted except by account deletion. Retention: events already consumed by the tick are retained 90 days for replay/debug then dropped **[call]** — the canonical state is the system of record, not the log.

**notebook_entry** — `entry_id`, `aviary_id`, `created_at`, prose text, (internal-only) generation metadata. Read-only to clients. Never archived; paginated reads.

**visit_invite** — `invite_id`, `aviary_id` (host), visitor `email_encrypted` + `email_hash` (same one-column discipline; the visitor's email is PII too), `token_hash`, `created_at`, `expires_at` (30 days), `revoked_at`, `consumed_at`.

**visit_session** — `visit_session_id`, `invite_id`, `started_at`, `ended_at` (drives the visit-log duration display). No event-log writes ever originate from a visit session; the API enforces this by issuing visit tokens that are only valid on snapshot-read endpoints.

**species** (static config, shipped with the app + server) — silhouette/palette parameters, call-grammar motif library, night-active flag (exactly one nightjar-like species).

The export endpoint serializes: birds (names, **personality vectors — the export is the one sanctioned place raw vectors leave the server**, per the PRD's export spec), moods, notebook entries, settings. **[call]** The export labels vector fields with neutral internal names and a header noting these are internal simulation values — satisfying the export requirement without turning the export into a stats dashboard; the no-numeric-exposure rule governs product surfaces, and the PRD explicitly lists vectors in the export contents.

---

## 4. API surface

All endpoints under `/api/v1`, JSON, authenticated by session token (or visit token for the read-only visit subset). Account-scoped endpoints derive `account_id` from the session — account ids never appear in URLs.

**Auth**
- `POST /auth/magic-link` — body: email. Always returns 202 (no account-existence oracle). Rate-limited per email and per IP.
- `POST /auth/consume` — body: link token. Returns session token. Atomic single-use.
- `GET /auth/sessions` / `POST /auth/sessions/{id}/revoke`.
- `POST /account/email-change` → verification mail to new address; `POST /account/email-change/confirm`.

**State (the hot path)**
- `GET /aviary/snapshot` — returns the full render snapshot: per-bird {id, name, species, mood, perch zone, current motion descriptor + phase, next-call schedule hint, plumage render parameters (derived from saturation trait server-side — the client receives render values, never trait values)}, aviary {settled flag, weather, tick_seq, server time}. Target payload: a few KB. ETag/`tick_seq`-conditional so unchanged pulls are cheap 304s.
- Snapshot pull triggers (client behavior, not endpoint): on load, on `visibilitychange` → visible, on detected render-gap (suspend recovery), and a keepalive poll while visible — **[call]** poll at 20s, between tick cadence and perceptual freshness; with ETag this is one small request per cycle. No WebSockets at v1: at a 1-minute tick there is nothing to push at sub-poll latency, and polling removes a whole class of connection-state bugs. Revisit only if tick cadence ever drops below ~15s.

**Events**
- `POST /aviary/events` — batched append: client buffers events and flushes every few seconds and on `visibilitychange`/`pagehide` (via `sendBeacon` for terminal flushes so tab-close presence-end is not lost). Server assigns `log_offset`, returns ack watermark. Client retries unacked batches; server dedupes on client-generated `event_id` (idempotent append).
- Presence pings: client sends one ping per 30s **[call]** while the three-condition presence predicate holds (see §7). Server treats a ping as "presence held for the last ping interval," accruing presence-time on tick. Server-side sanity caps (max accruable presence per wall-clock interval) make inflated/forged pings unable to exceed real time.

**Notebook** — `GET /notebook?cursor=` paginated, read-only.

**Birds** — `POST /birds/{id}/rename`. `GET /birds/offer` (pending age-gated species offer, if any) and `POST /birds/offer/accept` with chosen name. No decline-penalty; an unaccepted offer just remains available.

**Account** — export request, deletion request/cancel, settings.

**Visits**
- Host: `POST /visits/invite` (visitor email) → system emails one-time link; `GET /visits/log`; `POST /visits/{invite_id}/revoke`.
- Visitor: `POST /visits/consume` (token) → short-lived **visit token** scoped to exactly two endpoints: `GET /visit/snapshot` (host's live snapshot, identical content to the host's — no show-off rendering) and the keepalive poll. Each snapshot pull re-checks revocation; on revoke/expiry the next pull returns 410 with the matter-of-fact "visit no longer available" payload. Visit sessions write `visit_session` rows (for the host's log) and nothing into the interaction event log.

**Voice rule for API-adjacent copy**: every error payload includes a `user_message` written in the matter-of-fact register (these are system surfaces). Naturalist copy never appears in error responses.

---

## 5. Simulation engine design

### Drift function

Per trait, drift is a bounded, monotonic low-pass accumulation:

```
accumulator[trait] += Σ (weight(input) × signal_amount) × diminishing(trait_value)
on tick: trait = min(trait_max, trait + k × accumulator); accumulator *= (1 - leak)
```

- **Inputs and weights** (initial values, to be calibrated): presence-time dominates (weight ~5× any interaction input); listen-in minutes → target bird's social warmth + vocal frequency; offer-accepted → curiosity (small); offer-made-near-bird → boldness (smaller); plumage saturation accrues from total positive presence; settle contributes nothing to drift (it only closes the presence window cleanly and applies a mood-quieting nudge).
- **Monotonicity is structural**: the accumulator only ever receives non-negative contributions, and no code path subtracts from a trait. Neglect = zero input = zero drift. There is no decay term on traits (only on the accumulator, which limits burst stacking). A property test asserts traits are non-decreasing across any event sequence — this is the "no Tamagotchi" rule as a CI invariant.
- **Diminishing returns** near trait ceiling prevent saturation and keep week-3 visibility on target.
- **Calibration harness** (build this first, in week 1 of engine work): a simulated-user driver that replays parameterized behavior profiles (daily 10-min watcher, weekend-only visitor, all-day-tab-open non-presence, click-spammer) against the real tick code at accelerated clock, asserting the two named targets: **measurable instrument drift at ~1 week of regular visits; user-visible drift (defined as Δtrait crossing the render-perceptibility threshold per trait, set with design) at ~3 weeks; no visible single-session movement.** Calibration constants live in server config with a changelog; the harness runs in CI on any constant change. This harness is the highest-leverage early artifact because the PRD pins calibration as in-scope and nothing else in the stack does.

### Mood machine

- Enum: `wary, content, curious, drowsy, alert` plus `settled/sleeping` as night states **[call: ship this set; extending it is config + animation/caption assets, not schema]**.
- Transition function per tick: `P(next_mood) = f(current_mood, time_of_day(local tz), recent session interactions, ambient events, personality)`. Personality modulates thresholds (high boldness raises the wary threshold). Daily-ish reset implemented as a slow pull toward a time-of-day-appropriate baseline rather than a discrete midnight reset, so users never observe a snap.
- Timezone: client reports IANA timezone with presence pings; the aviary stores the most recent one and ticks against it **[call]** — local-time day/night per the layout spec requires the server to know the user's local time; last-reported-tz is the simplest honest model for a single-user account.
- Mood persists in the bird record; tab-open never resets it (the snapshot simply reports current mood).
- Contagion: alarm-type transitions in one bird raise wary probability for birds in adjacent perch zones on the same tick; chorus events arise when ≥2 birds with high vocal frequency have overlapping scheduled call windows — the tick emits a chorus hint that the client audio engine uses for mix timing.

### Call grammar (shared client/server contract)

Each species ships a motif library (short parametric phrase descriptors: pitch contour, note count, timing envelope). Each bird's stable `call_seed` + personality + current mood deterministically parameterize runtime generation. The **server** schedules *when* calls happen (call-schedule hints in snapshots, so two devices hear the same aviary doing the same things and narration/captions can be consistent); the **client** synthesizes *what they sound like* at the scheduled moment (WebAudio, §9). Between snapshots the client extrapolates the schedule from vocal-frequency parameters so calls don't starve at poll boundaries; the next snapshot re-anchors. Variation is real per-rendition variation (seeded jitter on timing/pitch within signature bounds), never a rotation of fixed variants — the signature stays recognizable because the motif skeleton and pitch center are stable per bird.

### Greeting selection

On the first snapshot pull after a presence gap, the server computes the return-greeting: selects the greeting bird (weighted by boldness and current mood, with variety pressure so it isn't always the boldest), greeting form (glance / two-note call / approach / call-and-response) shaped by absence length (minutes → glance; days → re-orientation approach), and a stagger schedule if a second bird responds. Shipped in the snapshot as a one-shot animation+call directive. Server-side selection keeps multi-device consistency and lets the notebook composer observe "pip greeted before wren today" from real data.

---

## 6. Sync model

Covered structurally in §2/§4; stated as guarantees:

1. **One writer.** Only the simulation tick writes canonical bird/aviary state. The API service writes only events, account rows, and invites. There is no endpoint accepting absolute state — last-write-wins on personality is unreachable, not just discouraged.
2. **Append-only client input.** Concurrent sessions on two devices interleave their events in the log; the tick consumes in `log_offset` order. Two devices' simultaneous use is additive, never conflicting.
3. **Read-your-tick freshness.** Devices may briefly render snapshots one tick apart (≤ ~60s skew). This is acceptable by design; no UI ever displays cross-device state side by side, so skew is unobservable in practice.
4. **Conflict surfaces are auth-level only** (expired link, session timeout, load failure) and use matter-of-fact copy per the PRD's exception. There is no data-merge UI because there is no data to merge.
5. **Suspend recovery.** On wake from laptop suspend, the render-gap detector forces a fresh snapshot before resuming render — the user sees the aviary as it now is, with no fast-forward animation replaying missed time **[call]**: replaying absence would announce the absence; the aviary is simply *currently* whatever it is.

---

## 7. Presence accounting (client + server contract)

Client-side predicate, evaluated continuously:

```
present := document.visibilityState === 'visible'
        && document.hasFocus()
        && (now - last_input_event) < ACTIVITY_WINDOW
```

- `last_input_event` updates on `pointermove`, `keydown`, `pointerdown`, `wheel`, `touchstart` (passive listeners, throttled to 1 update/s).
- **ACTIVITY_WINDOW initial value: 5 minutes [call]** — the PRD says "a few minutes, leaning longer because watching without moving is the product." 5 minutes; calibrate with the drift harness.
- While `present`, the client emits a presence ping every 30s. Any predicate condition failing stops pings immediately (and emits a presence-end marker on `visibilitychange`/`blur` via the event batch; `pagehide` flush via `sendBeacon` covers tab-close).
- Server accrual: presence-time per tick = covered ping intervals, capped at wall-clock elapsed. Pings from sessions whose tokens are revoked are dropped. Visit tokens cannot emit presence events at all (different endpoint scope).
- **Honesty tests**: an integration suite drives a headless browser through the corrupting scenarios — background tab 48h (zero presence), focused-but-idle beyond window (presence ends), watching-without-mouse within window (presence holds), rapid focus flapping (no double-count) — and asserts accrued presence server-side. This is the silent-failure class the PRD warns about; it gets paid-for test coverage, not code review attention alone.

Settle: triggers evening lighting ramp (client), a settle event (log), and ends presence on the same terms as tab-close. The 5-second undo is purely client-side (any click reverses the ramp and suppresses the queued settle event if not yet flushed; if already flushed, a follow-up `settle_undone` event lets the tick ignore the pair) **[call]**.

---

## 8. Frontend rendering pipeline

- **Stack [call]:** Canvas 2D with layered compositing as the baseline renderer, WebGL only if perf testing demands it. The scene is a single non-scrolling composition with ≤7 birds, soft parallax, and ornaments — well within Canvas 2D on the 5-year-old-laptop budget, and it keeps bundle and complexity down. Framework: a lightweight reactive layer (e.g., Preact/solid-class footprint) for chrome only; the scene renderer is hand-rolled against the frame object, no virtual DOM in the render loop.
- **Scene composition:** background plane (sky + foliage, day/night-tinted), middle plane (perches + birds), occasional foreground ornament plane. Day/night palette as a continuous tint curve sampled from local time; settle overrides toward the evening point with a slow ramp.
- **Bird rendering:** procedural/parametric bird bodies per species silhouette (vector skeleton + programmatic plumage from server-supplied render parameters). Pose system: a small pose graph per species (perch-idle, preen, head-tilt, scan, weight-shuffle, fluffed, sleeping, hop, short flight) with mood-keyed selection weights and per-bird seeded timing jitter so no two birds idle in phase.
- **Interpolation:** the state layer holds last-snapshot + extrapolated schedule; birds move between perch zones with animated paths (cross-fades in reduced-motion). Snapshot deltas reconcile gently — a bird "wrong" by one zone glides, never teleports.
- **First frame:** the boot path renders sky + perch planes immediately from inline critical data, places birds mid-pose from the edge-delivered snapshot (each pose is enterable at any phase — poses are parametric loops, so "mid-preen" is just phase=0.4), and starts audio after first paint. No entry animation, no fade-from-static, no spinner; slow-snapshot fallback is the quiet field (soft sky + one drifting-leaf cue).
- **Hidden tab:** `visibilitychange` → stop rAF loop and audio scheduling entirely (battery); resume with forced snapshot pull.
- **Reduced motion** (`prefers-reduced-motion` or settings opt-in): same state layer and frame object, alternate renderer module: pose cross-fades (~2s) instead of continuous motion, flights become cross-fades between perches, leaf/feather ornaments removed, day/night tinting retained but slowed. This is a designed register — design owns the cross-fade timing curves as a deliverable, not an engineering fallback.
- **Ornaments** (leaves, feathers): client-side only, seeded RNG at idle cadence, zero simulation state, excluded from reduced-motion.
- **Top bar:** DOM layer above the canvas; fades to ~5% opacity after a few seconds of cursor stillness, restores on pointer/keyboard activity; always fully opaque while any chrome panel is open and while keyboard focus is within it (accessibility requirement overrides the fade).

---

## 9. Audio pipeline

- **Synthesis:** one `AudioContext`; per-bird call rendering via a small synthesis graph (oscillator/FM voice + noise component + band-pass + amplitude envelope) parameterized by the species motif + bird seed + mood + per-rendition jitter. No samples, no loops, no recorded audio under any code path — this is a repo-level rule (CI fails on audio file assets).
- **Voice budget:** pre-allocated pool of synthesis voices (≤7 birds, brief overlaps; pool of ~10 voices) with buffer reuse — no per-call allocation, in service of the zero-memory-growth gate.
- **Chorus:** calls are scheduled events on a shared timeline (anchored to server schedule hints, extrapolated between polls). Overlapping calls mix naturally in the context; chorus hints from the tick bias scheduling windows together. A gentle master bus (soft compressor + ambient bed gain) keeps the mix calm.
- **Listen-in mix:** focused bird's bus ramps up and others ramp down to ambient over **~1.5–2s [call]** using `setTargetAtTime` ramps; disengage ramps back identically. Others never go below an audible ambient floor (re-balance, never mute). Engage/disengage emits listen_in events with duration.
- **Settle/night:** settled state lowers overall call scheduling probability and mix level; at night only the nightjar-like species schedules calls.
- **Autoplay policy reality:** browsers block audio before a user gesture. The aviary boots visually silent-but-alive; the context starts (or resumes) on first user input. No "click to enable sound" interstitial — a muted-state icon in the top bar is the only acknowledgment **[call]**, keeping the no-announcement rule intact while handling the platform constraint honestly.
- **Fallback:** if the context fails or is denied → graceful silence with captions enabled by default for that session, plus the caption preference surfaced in accessibility settings.
- **Caption generation:** the synthesis parameter set for each rendered call maps through a small descriptive grammar to prose ("a low trill, paused, low trill again") — generated from what was actually synthesized, same naturalist voice, displayed near the calling bird with fade-in/out matching the call envelope.

---

## 10. Accessibility surfaces

Shipped in v1; tracked as launch-blocking workstreams, not post-launch fixes.

- **Screen-reader narration:** an ARIA live region (`polite`) fed by a client-side narration composer that consumes the same frame object as the renderer. Idle cadence one prose update per 30–60s; user-initiated events (return-greeting, offer reaction, settle) get a priority lane (still `polite` — never `assertive` — but jump the idle queue). Prose comes from the shared naturalist grammar library used by captions and (server-side) the notebook, so the voice is one voice. Narration is observational prose, never state lists; a copy-review gate (§12) covers narration templates the same as notebook templates.
- **Reduced motion:** see §8 — a designed alternate renderer, equal-quality product.
- **Captions:** see §9 — runtime-generated, naturalist, opt-in (auto-on under audio fallback).
- **Keyboard:** Tab traverses top bar → into scene (first bird); arrow keys move between birds (ordered by horizontal position); Enter = listen-in toggle; Escape = disengage; offer panel and settle reachable via top bar with full keyboard support; focus ring is a soft high-contrast outline speced by design against both day and night palettes. Focus management keeps the faded top bar fully visible whenever it owns focus.
- **Contrast:** WCAG AA on all user copy (chrome, captions, narration text if visually displayed, errors); verified by automated contrast checks against both lighting extremes in CI.
- **Voice discipline:** accessibility *settings* surfaces use matter-of-fact voice (system register); the narration/caption *content* uses naturalist voice. The line is exactly the PRD's: talking to the system vs. experiencing the product.

---

## 11. Performance budgets and observability

Budgets (all are CI/launch gates, measured on reference hardware profiles):

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS bundle | <2MB gzipped | CI hard fail on regression; per-PR budget report |
| Time to first bird | <500ms, mid-tier mobile / 4G | Synthetic check in CI + scheduled fleet; achieved via: app-shell + first snapshot delivered from CDN edge (snapshot embedded/streamed with HTML for warm accounts **[call]** via short-TTL edge cache keyed by session, falling back to parallel fetch), critical-path renderer in the entry chunk, everything else (settings, notebook, visits, auth flows) code-split |
| Idle frame rate | 60fps, 5-year-old mid-range laptop, sustained 30 min | Scheduled synthetic run with frame-time histogram assertions |
| Memory | Zero growth over 30-min session | CI test: scripted 30-min session in headless Chrome, heap snapshots compared after forced GC; fails on monotonic growth. Voice pools, notebook scroll-out releasing references, bounded workers/contexts are the implementation levers |
| Tick latency | p99 < 5s alarm | Production alert; tick is budgeted to run in tens of ms |

Observability:

- **Aggregate-only RUM:** page-load timing, first-bird-render timing, frame timing, audio-context errors, API latencies/errors, anonymized session-duration histograms. **No per-account dimensions, no per-bird state, no interaction-history fields.** The privacy boundary is implemented at the pipeline: the telemetry SDK has no API accepting bird/account-state objects; telemetry schema changes require privacy sign-off; the analytics warehouse has no connection to the simulation database (separate credentials, separate network policy).
- **Synthetic fleet:** scheduled browser runs from several geographies against a synthetic account pool, exercising load, greeting, listen-in, offer, settle; asserting the perf budgets and the presence-honesty scenarios continuously in production.
- **Simulation health:** tick duration, backlog depth, event-log consumption lag, drift-rate distribution across the population *as anonymized aggregate histograms only* (needed to detect calibration drift like the presence-inflation failure mode — population drift-rate is operational health for the engine, and is collected without per-account dimensions).
- **What we deliberately don't measure:** engagement funnels, retention cohorts keyed to product features, per-account visit frequency, anything that would require or tempt per-bird/per-account interaction analytics. Named here so no one adds it as instrumentation hygiene.

---

## 12. Rollout

**Phase 0 — Engine-first foundations (weeks 1–4).** Tick service skeleton; data model; event log; drift function + **calibration harness** (the harness lands before drift constants are trusted); presence accounting client predicate + honesty test suite; magic-link auth.

**Phase 1 — Vertical slice (weeks 4–9).** One aviary, two birds, real tick, snapshot→render→interpolate loop, procedural calls for two species, listen-in, presence-driven drift on the calibration harness clock, first-frame-in-motion boot path. Exit: the slice *feels alive* in an internal review explicitly scored against the five design principles — this review is a real gate, because aliveness failures (canned greeting, audible loop repetition, load spinner regressions) are product-fatal per the brief.

**Phase 2 — Full surface (weeks 9–16).** All six species + call grammars; offers; settle; notebook composer; greeting selection; day/night + weather; top bar + chrome; adoption flow; reduced-motion renderer; narration; captions; keyboard nav; export/deletion; visits.

**Phase 3 — Hardening (weeks 16–20).** Perf budget closure on reference hardware; memory-growth CI green; accessibility audit (external); presence/drift calibration pass with longitudinal dogfood data (the team runs real aviaries from Phase 1 onward — this is the only way to validate the 3-week visibility target before launch); security review of auth + visit tokens; privacy-boundary audit (telemetry schema + log scrub for email/UUID discipline).

**Launch + ramp.** Launch with adoption capped at 2 birds (as designed). The bird-offer age gates start conservative **[call: 3rd bird at ~10 weeks aviary age, then widening intervals — 4th ~6 months, tuned with dogfood data]**; gates are server config, adjustable without deploys, only ever loosened (never retroactively revoking an offered bird). Instrument from day one: perf RUM, tick health, drift-rate aggregate histograms, audio-fallback rates, narration/caption usage rates (aggregate counts only).

**Standing review gates (every PR, lightweight checklist):** (1) no announcement-register UI; (2) no surface describes user behavior; (3) no personality numbers in any client payload beyond render-derived values; (4) naturalist/matter-of-fact voice applied per the register rule, all user-facing copy through copy review; (5) no recorded audio assets; (6) no email outside the account record; (7) no client write path to canonical state. These are the PRD's load-bearing rules converted into process so they survive contributor turnover.

---

## 13. Risks

1. **Drift calibration misses the felt band** (too fast → Tamagotchi; too slow → screensaver). *Mitigation:* calibration harness with named numeric targets from day one; accelerated-clock simulation; long-running team dogfood aviaries starting Phase 1; calibration constants hot-adjustable server-side; aggregate drift-rate histograms watching the population post-launch. *Residual:* the 3-week "visible to the user" target can only be truly validated with real elapsed weeks — dogfood early, launch with the conservative end of the band (slower is recoverable; too-fast drift can't be walked back without violating monotonicity).
2. **Presence signal corruption** — the silent, untestable-by-unit-test failure the PRD warns about (lax presence inflating population drift). *Mitigation:* the three-condition predicate is a single shared module with the headless honesty suite as a permanent CI fixture; server-side accrual caps; synthetic fleet runs the background-tab scenario in production forever. Browser-specific focus/visibility quirks (Safari `hasFocus` edge cases, mobile tab freezing) get explicit per-browser test passes.
3. **Sync correctness / lost drift.** *Mitigation:* unreachable-by-construction (no state-write endpoint, single-writer tick, idempotent event append, tick-seq conditional writes). Tests: concurrent two-device event interleaving, tick crash/replay determinism, suspend-recovery snapshot ordering.
4. **Audio uncanniness** — procedural calls reading as synthetic beeps, chorus reading as noise, or signature recognizability failing well below 7 birds. *Mitigation:* audio prototyping spike in Phase 0/1 with listening reviews (can testers name the bird by ear after a week of dogfood?); signature recognizability is an explicit Phase 1 exit criterion; mix design (compressor, ambient bed, scheduling windows) treated as designed surface with audio-design ownership; the 7-bird cap honored in engine config. If recognizability ceiling lands below 7 in practice, the age-gate ramp gives us months of runway before any account reaches the cap — adjust the cap config before it binds.
5. **Accessibility regressions / accessibility shipped late.** *Mitigation:* reduced-motion renderer, narration, captions, and keyboard nav are Phase 2 deliverables with launch-gate status, not fast-follows; external audit in Phase 3; narration prose goes through the same copy review as the notebook; automated contrast + focus-order checks in CI.
6. **Performance budget erosion** (bundle creep, memory leaks in long sessions, first-bird latency on cold caches). *Mitigation:* hard CI gates on bundle and memory from Phase 1; edge-snapshot delivery designed before launch rather than retrofitted; frame-time fleet checks; code-split discipline reviewed per PR.
7. **Tone/scope erosion by well-meaning contribution** — the toast, the streak, the stats panel. *Mitigation:* the §12 standing checklist; non-goals codified in CONTRIBUTING with the PRD's reasoning; product owner sign-off required on any new user-facing surface; the absence of underlying metrics (no per-account engagement data exists) makes the worst features structurally hard to add — an intentional architectural moat.
8. **Magic-link and visit-link abuse** (link forwarding, replay, invite spam). *Mitigation:* 15-min expiry + atomic single-use on auth links; hashed tokens at rest; per-email and per-IP rate limits; visit tokens scoped to read-only endpoints with revocation checked per pull; visitor emails encrypted under the same one-column PII rule. Threat model review in Phase 3 — the asset worth protecting is account access and the privacy commitments, not in-product value.
9. **Timezone/local-time edge cases** (travel, DST, devices disagreeing). *Mitigation:* last-reported-IANA-tz model is simple and self-correcting on next presence; DST handled by computing local time per tick from the IANA zone, not stored offsets; worst case is a transiently shifted day/night palette, which degrades charm, not correctness.

---

## 14. Open calls made in this plan (summary for review)

Snapshot polling over WebSockets at v1; Canvas 2D baseline renderer; 5-minute presence activity window (calibrate); 30s presence ping / 20s snapshot poll cadences; mood enum shipped as the PRD's five + night states; server-scheduled call timing with client-side synthesis; client-side narration / server-side notebook split over one shared grammar library; export includes raw vectors per PRD with internal-value labeling; muted-state icon as the only autoplay acknowledgment; 90-day consumed-event retention; bird-offer age gates (≈10 weeks → 3rd bird) as server config; tick at 60s ±10s jitter. Each is flagged **[call]** inline with rationale; none is architecturally one-way except the polling choice, which has a clean upgrade path (the state layer is the only network-aware module).
