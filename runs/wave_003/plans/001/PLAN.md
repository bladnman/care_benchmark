# Pocket Aviary — V1 Implementation Plan

## 1. Scope

**In v1:**
- Single-user accounts with email magic-link auth (no passwords, no SSO).
- One canonical aviary per account; two starter birds, cap of seven, further birds offered by aviary age.
- Server-side simulation tick (~1/min) driving mood, drift, bird-to-bird behavior; multi-device sync as an architectural property.
- Session interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook, presence accounting.
- Full scene: three perch zones, day/night cycle on user's local time, ambient weather, ambient micro-motion, top-bar chrome with fade.
- Procedural WebAudio call synthesis with chorus mixing; silence+captions as the only audio fallback.
- Accessibility: naturalist screen-reader narration, designed reduced-motion mode, runtime call captions, WCAG AA contrast, full keyboard navigation.
- Social: per-invite opt-in read-only visits, revocable, visit log, opt-in visit notifications (off by default), 30-day invite expiration.
- Account export (JSON via email link) and soft/hard deletion (30-day window).

**Not in v1 (per non_goals.md):** native apps, any gamification (no streaks, achievements, levels, counters, visit calendars), Tamagotchi mechanics (no death/hunger/distress/negative drift), social-network surfaces (no profiles, follows, discovery, leaderboards, comments, chat, avatars, co-presence), payments, shared aviaries, customizable scenes, multi-aviary accounts, push notifications, recorded audio anywhere.

## 2. Architecture

**Service shape.** A small set of services behind a single API gateway:
- **auth-service** — magic-link issuance/validation, session tokens, email change, account deletion.
- **sim-service** — the canonical simulation: owns bird records, personality vectors, mood state; runs the per-account tick; consumes the event log; writes snapshots and notebook entries.
- **api-service** — client-facing state pulls, interaction-event ingestion, visit-invitation flow, account settings, export generation.
- **web client** — single-page app, statically hosted on CDN, WebAudio + Canvas/WebGL rendering.

**Client/server split.** The server is the only writer of all bird state. Clients render snapshots and append interaction events; they never compute drift, never mutate personality, never advance mood. The render pipeline boundary is sharp: client input → events → server tick → snapshot → client interpolation. This split is what makes multi-device sync, "feels alive without the viewer," and no-last-write-wins all true at once.

**Deployment.** Snapshots for the initial load are inlined with the HTML response from a CDN edge to hit the <500ms first-bird budget. Subsequent pulls go to the API. The tick runs in a worker fleet keyed by account UUID.

## 3. Data model

All identifiers are synthetic UUIDs; email appears once, encrypted, on the account record.

- **Account** — id (UUID), email (encrypted, unique), created_at, settings (visit-notify opt-in, reduced-motion override, captions opt-in), deletion state (none / soft-deleted-at).
- **Session** — id, account_id, device label, issued_at, revoked flag.
- **Bird** — id (stable UUID), account_id, species_id, name, adopted_at, personality vector (boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity — normalized scalars), current mood enum, perch zone, drift history (for instruments only, never surfaced).
- **Species** — id, silhouette/SVG assets, palette, call-grammar motif library (six species at v1, including one nightjar-like nocturnal signature).
- **InteractionEvent** (append-only log) — id, account_id, bird_id (nullable), type (presence_ping, listen_in_start/end, offer, settle, session boundary), payload, client_timestamp, server_timestamp. Ordered per account; consumed by the tick in order.
- **Snapshot** — derived, per-account canonical state at tick N: per-bird position/perch/mood/call timing/active transitions, scene state (time-of-day lighting, weather, settled flag), notebook pointers.
- **NotebookEntry** — id, account_id, timestamp, prose text, generation context (internal only).
- **VisitInvite** — id, host account_id, visitor email (encrypted), token, issued_at, expires_at (30 days), revoked flag, consumed flag.
- **VisitLogEntry** — id, host account_id, visitor email, started_at, duration.

## 4. API surface

All endpoints authenticated except auth. Per-account scoping everywhere; visitors use invite-scoped tokens.

- `POST /auth/magic-link` — request link (rate-limited per email). 15-minute expiry, single-use.
- `POST /auth/consume` — exchange link token for session token.
- `GET /state/snapshot?since=N` — current canonical snapshot (small, KB-scale). Used on load, visibility change, long frame gaps, keepalive.
- `POST /events` — append interaction events (presence pings, listen-in start/end, offer, settle). Idempotent via client-supplied event ids. Never carries absolute personality values.
- `GET /notebook?cursor=` — paged notebook entries, oldest scrollable indefinitely.
- `POST /birds/{id}/rename`, `GET /birds` — naming and list.
- `POST /visits/invite` — create invite (host). `DELETE /visits/invite/{id}` — revoke (immediate effect). `GET /visits/log` — host's visit log.
- `GET /visit/{token}/snapshot` — visitor read-only snapshot pull; returns "visit no longer available" (matter-of-fact) when revoked/expired.
- `GET /account/export` — enqueue export, email download link. `POST /account/delete`, `POST /account/recover`.
- Adoption: `POST /birds/adopt` gated server-side on aviary-age offers.

Visitors have access only to the read-only snapshot endpoint under their invite token; their clients emit no events, and the tick ignores visitor sessions entirely.

## 5. Simulation engine design

**Tick.** Every ~60s per account (cadence calibrated in build), a worker: loads the account's birds + recent unconsumed events; applies mood updates, drift deltas, bird-to-bird interactions; advances scene state (weather scheduler, settle decay); writes snapshot N+1 and marks events consumed. Runs with zero clients connected. p99 latency alarm at 5s.

**Drift function.** A low-pass filter over presence-and-interaction signals:
`trait_t+1 = clamp(trait_t + Σ w_i · signal_i)`.
Weights (descending): presence-time (dominant), listen-in (per-target bird: social_warmth, vocal_frequency), offers (accepted → curiosity; nearby offer → boldness). Settle only ends the presence window cleanly — no directional drift. **Monotonic toward expressive: no trait ever decreases.** Absence produces ambient quietness (lower greeting frequency expression), never negative trait movement. Calibration targets, testable in the harness: measurable instrument drift after ~1 week of regular visits; user-visible drift after ~3 weeks. We will build a time-compressed simulation harness (synthetic presence streams at accelerated tick rates) to tune weights against these targets before launch.

**Mood.** Small enum (wary, content, curious, drowsy, alert). Transition function inputs: recent events this session, user's local time-of-day, ambient weather events, bird's personality (high boldness suppresses wary entry). Mood persists across sessions — stored, never reset on connect; the tick continues mood evolution during absence (drowsy-at-dusk → settled by morning).

**Call-grammar runtime.** Per species: a motif library (timing patterns, pitch contours, syllable primitives). At tick time, the engine schedules call events per bird (probability shaped by vocal_frequency, mood, chorus opportunities from co-calling birds, weather dampening, night/nocturnal exception). Snapshots carry call schedules with per-instance variation seeds; the client synthesizes audio deterministically from (species motif, seed, mood, personality) so the caption, the audio, and any narration all describe the same call. Call signatures are stable per bird across drift — recognizability is the design invariant behind the 7-bird cap.

**Bird-to-bird.** Within a tick: a call by bird A rolls response checks for other birds (warmth-weighted); wary spreads locally; chorus events emerge from overlapping call windows. Implemented as a small per-tick interaction pass, not an agent framework.

**Notebook generation.** The tick emits candidate observations (greeting order changes, unusual quiet stretches, weather reactions, offer outcomes). A sparsity gate (~1 entry per few days for typical use; noteworthy events can break through) selects candidates; a prose generator (templated naturalist voice with slot-filling from real state — never numeric, never user-behavior observations) writes the entry. Hard rule enforced in the generator's data access layer: entries may reference bird behavior and scene state only, never visit frequency or user stats.

## 6. Sync model

One canonical state per account; server is the only writer. Clients pull snapshots and interpolate; they write events only. Multi-device sync requires no merge logic — both devices read one record. Conflicts are prevented by construction:
- Personality is additive server-authored deltas applied in event-log order; no client ever submits absolute trait values.
- Events are idempotent and ordered per account; replayed events are deduped by id.
- The tick is single-writer per account (partitioned by account UUID; one in-flight tick per account enforced by lock/lease).
- Client staleness is handled by `since=N` snapshot pulls; the client never writes state, so there is nothing to conflict.
- Failure surfaces (magic-link replay, mid-write timeout, outage) present matter-of-fact error copy, never naturalist voice.

## 7. Frontend rendering pipeline

**Stack.** Canvas (WebGL where available, 2D fallback) for the scene; DOM for top bar and settings surfaces. Bird assets: procedural where possible, otherwise compact SVG/bitmap sprites; pose cross-fade and motion done in the render loop.

**Scene composition.** Single horizontal scene, three perch zones, middle-plane birds/perches, soft background, occasional foreground ornaments, subtle parallax. Responsive: scale-and-respace strategy that never crops a bird at any viewport.

**Boot path.** HTML response inlines the first snapshot + minimal boot JS → first bird painted before any non-critical asset loads → scene starts already mid-motion (birds placed at snapshot positions mid-action; no entry animation, no spinner — the cold-load fallback is the quiet-field state). Code-split: settings, visits, notebook load on demand.

**Idle micro-motion.** Continuous mood-shaped behaviors (preen, scan, head-tilt, weight-shift) driven from snapshot state + local animation clock; client interpolates between snapshots for perch moves. Rendering pauses when the tab is hidden (simulation continues server-side); resume re-pulls snapshot to avoid catch-up snapping.

**Transitions.** Greeting, listen-in focus, offer reactions, settle lighting shift, settle 5-second undo, top-bar fade — all slow ramps, no hard cuts.

**Reduced-motion mode.** A separate render register, not "animations off": pose cross-fades replace frame animation, perch changes cross-fade, leaf drift removed, color cycles slowed. Triggered by `prefers-reduced-motion` or settings opt-in.

## 8. Audio pipeline

**Synthesis.** WebAudio graph: per-bird synth voices built from species motif primitives (oscillator/noise + envelope + filter recipes per syllable), sequenced from the snapshot's call schedule with per-call seeded variation. No audio files anywhere.

**Chorus mixing.** Per-bird gain/pan buses into an ambient master. Chorus is real-time mixing of independently varied procedural calls — never stacked loops.

**Listen-in mix.** Engage: focused bird's bus ramps up over ~1.5s; others ramp down to ambient floor (never zero). Disengage (re-click, other bird, empty space, focus loss): symmetric slow ramp back.

**Fallback.** WebAudio unavailable/denied → graceful silence + captions enabled by default. No recorded fallback path exists, by design.

## 9. Accessibility surfaces

- **Screen-reader narration.** Live-region prose generated from the same snapshot state, naturalist voice, ~1 update per 30–60s at idle, priority bump for user-initiated events (greeting, offer reaction, settle). Written as observations ("a warbler perches on the high branch, calling softly"), never state lists. Queue management caps pending utterances so the reader is never flooded.
- **Captions.** Per-call prose generated from the same call-grammar parameters that produced the audio; rendered near the calling bird, fade with the call. Opt-in via settings; default-on in the no-WebAudio fallback.
- **Reduced-motion.** As in §7 — a designed surface shipping at v1, not a follow-up.
- **Keyboard.** Tab through top bar → into scene; arrows move between birds; Enter listen-in; Escape exits; offer palette fully keyboard-navigable; settle from top bar. Visible high-contrast focus indicators against all lighting states.
- **Contrast.** WCAG AA minimum on all user copy (chrome, settings, captions, errors); verified in CI with automated checks per lighting state.

## 10. Performance budgets and observability

- Initial JS < 2MB gzipped; first bird visible < 500ms on mid-tier mobile over 4G; 60fps idle on a 5-year-old laptop sustained over 30 minutes; zero memory growth over 30 minutes (CI test: heap snapshots across a scripted session; audio buffer pooling; no retained notebook DOM).
- Instrumentation: synthetic browser fleet from common geographies; aggregate-only RUM (load, first-bird, frame timings, audio-context errors, tick latencies). Tick p99 alarm at 5s.
- Privacy boundary at the metric schema level: no per-account, per-bird, or interaction dimensions in any telemetry stream; simulation DB is never read by analytics. Browser support: last two majors of Chrome/Safari/Firefox/Edge; matter-of-fact unsupported-browser surface otherwise.

## 11. Rollout

1. **Foundation:** auth + synthetic-UUID account model + event log + tick skeleton with static snapshots.
2. **Scene first:** boot path, three-perch scene, interpolation, quiet-field loading — validated against the <500ms budget early.
3. **Engine:** drift + mood + calls behind the time-compressed calibration harness; notebook generator with sparsity gate.
4. **Audio:** synth voices, chorus mix, listen-in ramps, captions, fallback.
5. **Accessibility surfaces** ship with the features they describe — not a later pass.
6. **Social last** (visits are read-only consumers of existing snapshots).
7. **Ramp:** internal accounts → invite-only beta with two birds only → open third-bird age offers once drift calibration is confirmed in production instruments → approach the 7-bird cap gradually, listening for chorus-recognizability degradation.
8. **Instrumented from day one:** performance budgets, tick latency, error rates — all aggregate-only.

## 12. Risks

- **Drift calibration.** Too fast → Tamagotchi; too slow → screensaver; the failure is silent. Mitigation: accelerated-time harness with the 1-week/3-week targets as automated gates; per-trait drift-rate telemetry (aggregate) post-launch; weights adjustable server-side without client deploys.
- **Presence-signal honesty.** Any laxer definition (tab-open) silently inflates drift population-wide. Mitigation: the three-signal conjunction implemented once in a shared presence module, unit-tested against background/focus/idle cases; longer activity window favored.
- **Sync correctness.** The no-last-write-wins rule fails only if a code path lets a client write state. Mitigation: personality columns writable only by the sim role at the DB level; event ingestion schema has no trait fields; single-tick-per-account lease tested under failover.
- **Audio uncanniness.** Repetitive or phase-locked calls break the spell. Mitigation: seeded per-call variation with anti-repetition windows; staggered multi-bird greetings; listening tests against the 7-bird recognizability ceiling.
- **Accessibility regressions.** Narration cadence flooding screen readers; reduced-motion shipped as a stripped fallback. Mitigation: narration queue caps with soak tests; reduced-motion treated as a parallel render register with its own design review; axe CI on all copy surfaces.
- **Voice drift.** A toast, a streak, a numeric stat, or an event-log-style notebook entry sneaks in via well-meaning contribution. Mitigation: copy linting / content review checklist encoding "notice never announce," "no user-behavior observations in the notebook," and the naturalist/matter-of-fact split as merge-blocking checks.
- **Performance regressions.** Bundle creep (audio/asset additions). Mitigation: bundle-size CI gate at 2MB; first-bird synthetic check per release.
