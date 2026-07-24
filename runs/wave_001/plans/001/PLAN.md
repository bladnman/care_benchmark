# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable build for a frontier engineering team. It makes defensible calls where the spec is silent and names every load-bearing rule so it survives contact with implementation.

---

## 1. Scope

### In scope (v1)
- Browser-only product (last two major versions of Chrome, Safari, Firefox, Edge).
- Single-user accounts with email magic-link auth; one canonical aviary per account.
- Two starter birds at adoption; species chosen by the system from a pool of ~6; user assigns names; cap of 7 birds, unlocked by aviary age.
- Server-side simulation tick (~1/min) owning all canonical state: personality vectors, moods, perch positions, call timing, weather, notebook.
- Client rendering of snapshots with interpolation; procedural WebAudio call synthesis; chorus mixing; listen-in.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook, presence accounting (three-signal conjunction).
- Day/night cycle anchored to user local time; rare ambient weather.
- Visit invitations: per-invite opt-in, read-only ambient, revocable, 30-day expiry, silent visit log, optional (off-by-default) visit notifications.
- Accessibility: naturalist screen-reader narration, designed reduced-motion mode, procedural call captions, WCAG AA contrast on all chrome, full keyboard navigation.
- Account export (JSON snapshot emailed as download link) and soft-delete (30 days) then hard-delete.

### Out of scope (hard rules, per non_goals.md)
- No native apps, no gamification of any flavor (no streaks, counters, badges, calendars, visit-frequency surfaces — including disguises like exportable visit logs or notebook entries about user behavior), no Tamagotchi mechanics (no death, hunger, distress, decaying meters), no social-network surfaces (no profiles, follows, feeds, discovery, leaderboards, chat, avatars, comments, co-presence).
- No recorded audio anywhere, including as a WebAudio fallback (fallback is silence + captions).
- Personality vectors are never exposed numerically on any surface, at any tier, ever.

---

## 2. Architecture

### Service shape
Three deployables plus static assets:

1. **Web client** — static SPA served from CDN. Owns rendering, audio synthesis, presence detection, interaction capture. Never owns canonical state.
2. **API service** — stateless HTTP/JSON. Handles auth (magic link issuance/verification), session tokens, snapshot reads, event-log appends, notebook reads, visit-invite lifecycle, account settings/export/deletion.
3. **Simulation service** — the only writer of personality/mood state. Runs the per-minute tick over active aviaries; consumes the append-only event log in order; computes drift deltas, mood transitions, weather scheduling, notebook entry generation, greeting decisions.

### Client/server split and render-pipeline boundary
The boundary is strict: the server computes *what is true*; the client computes *what it looks and sounds like right now*. The client receives snapshots (positions, moods, call schedules, weather, lighting phase) and renders by interpolation between snapshots. All animation timing, micro-motion selection, call synthesis, and mix decisions are client-side; all state transitions are server-side. Clients never tick.

### Data stores
- **Simulation DB** (per-account rows; PostgreSQL-class): canonical bird state, personality vectors, moods, notebook entries, event log (append-only, partitioned by account UUID). This DB is never read by analytics or telemetry pipelines — the privacy rule is enforced at the pipeline level, not the policy level.
- **Account DB**: account record (synthetic UUID key; email stored once, encrypted), sessions, visit invites, visit log, settings.
- **Telemetry store**: aggregate-only operational metrics, no per-account dimension beyond a hashed, non-reversible bucket for rate-limiting.

### Key architectural invariants
- Server is the only writer of personality state; drift is additive server-authored deltas from the ordered event log. No last-write-wins anywhere.
- Synthetic account UUID is the only account identifier in DBs, logs, partition keys, and telemetry. Email appears in exactly one column, encrypted.
- Tick runs whether or not any client is connected.

---

## 3. Data model

All identifiers are UUIDs. Timestamps UTC; client supplies timezone offset for local-time anchoring.

**Account** `{ id (uuid), email_encrypted, created_at, timezone, settings { visit_notifications: bool, reduced_motion: bool, captions: bool, audio_muted: bool }, deletion_requested_at: timestamp|null }`

**Session** `{ id, account_id, device_label, issued_at, revoked_at }`

**Bird** `{ id (stable for life of account), account_id, species_id, name, adopted_at, personality_vector { boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity } (each normalized 0..1, server-only), mood: enum(wary|content|curious|drowsy|alert), mood_entered_at, perch_zone: enum(front|middle|back), position {x,y} within zone, plumage_seed, call_grammar_seed }`

**AviaryState (canonical snapshot)** `{ account_id, tick_seq, server_time, lighting_phase (0..1 across local day), weather: enum(none|rain|wind) + ends_at, settled: bool, birds: [bird render-relevant subset: id, name, species, position, mood, plumage_saturation (drives visual richness only, not a numeric display), call_schedule_hint], pending_greeting: {bird_id, kind} | null }`

**EventLog entry** `{ id, account_id, tick_seq_consumed: int|null, type: enum(presence_ping|listen_in_start|listen_in_end|offer|settle|settle_undo|session_open|session_close), payload (bird_id, offer_kind, duration_ms, ...), client_ts, server_ts }`

**NotebookEntry** `{ id, account_id, created_at, prose (naturalist, lowercase), source_signals (internal: which events/moods generated it — used for dedup/sparsity control, never displayed) }`

**VisitInvite** `{ id, account_id, visitor_email_encrypted, token_hash, created_at, expires_at (30d), revoked_at, used_at }`
**VisitLog entry** `{ id, account_id, invite_id, visitor_email_encrypted, started_at, duration_s }`

Note the deliberate asymmetry: the visit log shows who visited (transparency to host), but no telemetry pipeline ever aggregates visit or interaction data across accounts.

---

## 4. API surface

All endpoints under `/api/v1`. Auth via bearer session token (except magic-link endpoints). All error copy uses matter-of-fact voice.

**Auth**
- `POST /auth/magic-link` `{ email }` → sends 15-minute link; rate-limited per email.
- `GET /auth/verify?token=...` → validates, invalidates token, issues session token. Replay returns the expired-link error surface.
- `POST /auth/sign-out`; `GET /auth/sessions`; `DELETE /auth/sessions/{id}` (revocation).

**State**
- `GET /aviary/snapshot?since_tick=N` → canonical snapshot (or delta). Small payload (KBs); also embedded in initial HTML at CDN edge for <500ms first bird.
- `POST /aviary/events` → batch append of interaction events (presence pings every ~60s while present; listen-in start/end; offer; settle; settle-undo). Returns accepted tick_seq for client reconciliation. Clients never submit absolute state.
- `GET /notebook?before=<cursor>` → paginated entries, oldest-to-newest scrollback, no archiving.

**Adoption / birds**
- `POST /birds/adopt` (only when an age-gated offer is active) → species drawn server-side from pool.
- `PATCH /birds/{id}` `{ name }` → rename only. No other mutable field is client-writable.

**Visits**
- `POST /visits/invite` `{ email }` → creates invite, emails one-time link.
- `GET /visits` → outstanding invites + visit log for host.
- `DELETE /visits/{id}` → revoke; active visitor sessions terminate at next snapshot pull with the "visit no longer available" matter-of-fact surface.
- `GET /visit/{token}/snapshot` → read-only snapshot for visitor; same payload shape as host snapshot minus any host-only fields; sets no presence events, writes no events; visitor client exposes no interaction affordances.

**Account**
- `GET /account`, `PATCH /account/settings`, `POST /account/email-change` (verify-new-before-commit), `POST /account/export` (emails download link), `POST /account/delete`, `POST /account/recover` (within 30-day window).

---

## 5. Simulation engine design

### Tick
Every ~60s per active aviary (all aviaries tick; cadence calibrated in build). The tick is idempotent and ordered: it consumes event-log entries with `tick_seq_consumed IS NULL` in server_ts order, then:

1. **Presence accounting.** Sum presence-ping durations (client only sends pings when all three signals hold: visibilityState=visible AND window focus AND pointer/key activity within the calibrated window, leaning long — target 3–5 min). Presence-time accumulates per bird (shared across birds, with listen-in weighting per-bird).
2. **Drift.** Low-pass filter: `trait += k * input_signal * dt`, with `k` calibrated so instruments detect change at ~1 week of regular visits and users notice at ~3 weeks. Drift is **monotonic toward expressive**: positive inputs move traits up; absence of input leaves traits unchanged (never decrements). Plumage saturation drifts up with sustained attention, never down. Implementation note: enforce monotonicity in code with a clamp and a regression test — a symmetric-drift implementation is the single most likely engine bug.
3. **Mood transitions.** Per bird, from: recent session interactions (accepted offer → content), time-of-day in user timezone (dusk → drowsy, early morning → alert), ambient events (rain damps vocal frequency aviary-wide briefly; another bird's alarm call → wary spread), and personality modulation (high boldness resists wary). Mood persists across sessions; tick continues mood evolution while no client is connected. Defensible call: use a weighted-transition Markov-ish table per mood with personality multipliers, with a daily soft-reset toward a time-of-day-appropriate baseline ("daily-ish cadence").
4. **Perch/position selection.** Bird chooses perch zone from mood + boldness; position within zone jitters. Bird-to-bird: chorus events emerge when ≥2 high-vocal-frequency birds' call windows overlap; wary spreads probabilistically.
5. **Weather.** Rare scheduler: short rain a few times/week, occasional wind; durations short; effects small and time-boxed.
6. **Greeting decision.** On `session_open`, compute absence length from last presence end; select greeter weighted by boldness and mood (bolder greets first; wary birds may not greet); pick greeting kind from absence-length band (glance < ~30min; call < ~1 day; approach/longer call beyond). Stagger multiple greeters by randomized offsets. Emit `pending_greeting` in next snapshot.
7. **Notebook.** Generator observes the tick's notable deltas (first-greeter changes, unusual quiet stretches, weather moments, offer reactions) and writes prose via a template-and-variation engine with a large phrase bank, seeded per account so entries are specific but non-repeating. Sparsity gate: ~1 entry per few days baseline for regular users, with a noteworthy-event override; hard cap per week so very active users don't get a feed.
8. **New-bird offers.** Age-based scheduler marks an adoption offer available at calibrated intervals (first at a few months).

### Call-grammar runtime
Each species has a motif library (small parametric fragments: pitch contours, syllable counts, trill patterns). Each bird has a `call_grammar_seed` fixing its recognizable signature; runtime variation (pitch/timing jitter shaped by vocal_frequency and mood) is applied per call. The server schedules call events (bird_id, motif params, timestamp) into snapshots; the client synthesizes them via WebAudio and generates the caption string from the same motif parameters, guaranteeing caption matches what played.

---

## 6. Sync model

Single canonical state; server-only writes; additive deltas. There is nothing to merge, ever.

- Client pulls snapshot on load (embedded in HTML), on `visibilitychange` to visible, on render-frame gaps > threshold (laptop suspend), and on a low-frequency keepalive (~30–60s) while visible.
- Client interpolates between snapshots for motion continuity; mood changes cross-fade into mood-shaped idle behavior.
- Writes are one-way: client appends events; tick consumes. Conflict surface exists only for auth/session/server errors, always in matter-of-fact voice.
- Multi-device is free: both devices read the same record. A phone opening after a laptop session sees the same birds in the same moods.
- Visitor sessions: same snapshot pipeline, read-only, zero event writes, revocable at next pull.

---

## 7. Frontend rendering pipeline

- **Stack defensible call:** Canvas 2D (or WebGL if profiling demands) for the scene; DOM for top bar, notebook, settings, captions, narration live-region. Birds rendered as procedurally-assembled layered sprites (small SVG/compact bitmap parts: body, head, wing, tail) with parametric plumage driven by `plumage_seed` and saturation trait.
- **First frame:** HTML ships with embedded snapshot; renderer draws birds mid-pose (phase-offset into their current idle cycle) before any non-critical asset loads. No spinner, no fade-from-static, no entry animation. Slow-connection state is the "quiet field" (soft sky, faint motion cues).
- **Scene composition:** three perch zones (front/middle/back), subtle parallax background/mid/foreground planes, day/night palette keyed to user local time (computed client-side from account timezone; smooth gradient phases), weather overlay when active, settle lighting transition over several seconds.
- **Idle micro-motion:** per-bird behavior loop (preen, scan, head-tilt, weight-shuffle) selected by mood, parameterized by personality; runs at 60fps on 5-year-old hardware; rendering halts when tab hidden (simulation continues server-side).
- **Ambient ornaments:** client-only leaf/feather drift at slow random intervals — no simulation state.
- **Responsive:** scene scales to viewport; aspect-preserving layout; invariant enforced in layout code and visual tests: never crop a bird, never let one leave frame.
- **Top bar:** account/settings, accessibility settings, notebook, offer. Fades to near-transparent after a few seconds of cursor stillness; returns on movement/keyboard. No other chrome; nothing inside the scene.
- **Reduced-motion mode:** a separate render path, not a fallback — cross-fades between still poses, cross-fade perch transitions, no leaf drift, slowed palette shifts; calls/captions/notebook/drift unchanged. Triggered by `prefers-reduced-motion` or setting.
- **Adoption:** empty aviary renders as quiet field; first bird enters with soft fly-in; thereafter the empty state is unreachable.

---

## 8. Audio pipeline

- **Synthesis:** WebAudio-only. Motif fragments rendered via oscillators/filters/noise envelopes per the call grammar; per-call parameter jitter from vocal_frequency and mood. Buffers reused; no per-call allocation that isn't freed (memory-budget test).
- **Chorus mixing:** per-bird gain nodes into a master bus. Default ambient mix; listen-in ramps focused bird up and others down (never to silence) over ~1–2s, and back on disengage (click bird again, click empty space, focus moves, keyboard focus leaves). Hard cuts are banned — it's a re-balance, not soloing.
- **Captions:** when enabled (or when WebAudio fails — then silence + captions by default), caption text is derived from the same motif parameters as the synthesized call, rendered near the calling bird, fading with the call. Naturalist voice: "a soft three-note rise."
- **Night behavior:** most birds quiet; the nightjar-like species may call late. Night is not silent-by-default dead state.
- **No recorded audio anywhere.** Bundle budget and chorus correctness both depend on this.

---

## 9. Accessibility surfaces

- **Narration:** an ARIA live-region fed naturalist prose generated from the same snapshot state the visuals use (same generator family as notebook). Cadence 30–60s at idle; priority bump for user-initiated events (greeting, offer reaction, settle), still written as observation, not state transition. Rate-limited queue so we never overwhelm the screen reader.
- **Reduced motion:** as in §7 — designed surface shipping at v1, not a later fix.
- **Captions:** as in §8.
- **Keyboard:** Tab through top bar; Tab into scene focuses first bird; arrows move between birds; Enter listen-in; Escape exits; offer flow and settle fully keyboard-operable. Visible high-contrast focus indicators against both bright and dim scene states.
- **Contrast:** WCAG AA minimum on all chrome copy, captions, and any visually displayed narration. The scene itself carries no copy.
- **Voice split honored:** settings/error surfaces are matter-of-fact; everything else naturalist.

---

## 10. Performance budgets and observability

**Budgets (CI-enforced):**
- Initial JS bundle < 2MB gzipped at first paint; aggressive code-splitting for settings/visit/notebook surfaces.
- Time-to-first-bird < 500ms on mid-tier mobile over 4G (embedded snapshot + CDN edge + render path that doesn't await non-critical assets).
- 60fps idle motion sustained for 30 minutes on a 5-year-old mid-range laptop.
- Zero memory growth over 30-minute session (automated CI test).

**Observability (aggregate-only, per the privacy boundary):**
- Synthetic fleet checks from common geographies; RUM for page-load, first-bird, frame timing, audio-context errors, tick latency.
- Simulation-tick p99 alarm at 5s.
- **Never collected:** per-bird state, per-account interaction history, anything reconstructing a user's relationship with their aviary. Telemetry pipelines have no read path to the simulation DB; metric definitions are reviewed against the boundary.
- **Deliberately not measured:** visit frequency per user, streaks, engagement counters. We don't compute stats we refuse to surface — the architectural absence makes reappearance harder.

---

## 11. Rollout

1. **Phase A — engine + skeleton:** simulation service, event log, snapshot API, magic-link auth, minimal renderer (static poses). Internal-only.
2. **Phase B — aliveness:** procedural audio, chorus, idle micro-motion, day/night, greeting pipeline, first-frame-already-moving.
3. **Phase C — relationship:** drift calibration against synthetic presence patterns (verify 1-week-instrument / 3-week-visible targets), notebook generator, offers, settle, listen-in.
4. **Phase D — access + social:** narration, reduced-motion, captions, keyboard, visit invitations.
5. **Limited beta** → calibration tuning (presence activity window, tick cadence, offer cooldown, notebook sparsity) → **GA**.

**Bird-count ramp:** v1 launches at 2 starters; age-gated offers begin at the calibrated first interval (a few months). Cap 7 enforced engine-side. Instrument from day one: tick latency, snapshot latency, first-bird timing, audio errors, narration queue depth — all aggregate-only.

---

## 12. Risks

- **Drift calibration (highest risk).** Too fast → Tamagotchi; too slow → screensaver; and failures are silent. Mitigation: synthetic-presence test harness asserting instrument-detectable drift at 1 week of simulated regular visits; monotonicity regression test; per-epoch drift-rate sanity dashboards (aggregate distribution shape only, no per-account inspection); manual long-running soak accounts reviewed weekly.
- **Presence definition corruption.** A laxer signal (e.g., "tab open") silently inflates population drift. Mitigation: the three-signal conjunction implemented in exactly one module with unit tests per signal and an integration test simulating background-tab/idle-machine cases; the activity window is a single config constant.
- **Sync correctness.** Additive-delta discipline eroding via a "convenient" client write. Mitigation: API schema makes personality fields unwritable (no endpoint accepts them); code-review rule; DB-level constraint that only the simulation role can update vector columns.
- **Audio uncanniness.** Repetition detection by the ear breaks the spell; two looped motifs phase-cancel. Mitigation: per-call jitter on pitch/timing/timbre; large motif combination space per species; listening tests including chorus overlap; recognizability testing (can a tester identify Pip after variation?) — the cap-of-7 invariant depends on this.
- **Procedural-caption mismatch.** Caption must describe what actually played; generate both from one parameter object, never separately.
- **Accessibility regressions.** Reduced-motion or narration drifting into "stripped fallback." Mitigation: accessibility surfaces are designed deliverables with their own review sign-off; CI checks for contrast, keyboard reachability, focus visibility, narration cadence limits.
- **Greeting becoming canned.** Mitigation: greeting kind selected from absence-band × personality × mood with procedural variation inside each; test asserts no identical greeting sequence repeats within N sessions.
- **Notebook dilution.** Entries trending toward feed. Mitigation: sparsity gate with hard weekly cap; prose-quality review samples.
- **Scope creep toward announcements.** Toasts, streaks, welcome text. Mitigation: PR checklist item naming the banned surfaces; this plan's §1 list is the contract.
- **Email-as-identifier leak.** Mitigation: schema makes email a single encrypted column; lint rule blocking email in log statements and partition keys.
