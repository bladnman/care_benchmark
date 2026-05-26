# Phase 1 Implementation Plan for Pocket Aviary — v1

**Wave:** 002  
**Run:** 001  
**Candidate:** grok-build-0.1 (effort: unknown) via opencode harness  
**Date:** 2026-05-26 (per env)  
**Objective:** Produce a self-contained, executable engineering plan for a frontier team to ship Pocket Aviary v1 in a browser-only web product. The plan respects every non-goal, voice rule, and calibration target in the PRD. No code, assets, or prototypes are generated here.

## 1. Executive Summary and Scope
Pocket Aviary v1 is a calm, single-aviary, browser-based companion product. Two starter birds (capped at seven total), personality vectors that drift monotonically toward expressiveness over weeks of honest presence, fast-timescale mood, procedural calls, field notebook, read-only visit invites (off by default), magic-link accounts with server-authoritative sync, full accessibility surfaces including designed reduced-motion mode and self-describing narration, no gamification whatsoever.

**In v1 (explicitly):**
- Two starter birds at account creation; birds added solely by aviary age.
- Personality: 5-tuple (boldness, social warmth, vocal frequency, plumage saturation, curiosity), scalar, persisted only on server.
- Mood enum (wary/content/curious/drowsy/alert + one additional in final spec) with personality- and time-of-day- and event-modulated transitions.
- Procedural call grammars per species (≈6 species), synthesized via WebAudio.
- Server tick (~1/min cadence) that:
  - Applies drift from presence-time + selected interactions (additive deltas only).
  - Transitions moods.
  - Writes next canonical snapshot.
- Presence strictly defined (visibility + focus + recent pointer/key activity).
- Field notebook (sparse, naturalist prose, server-grade generation).
- Settle (opt-in), listen-in (mix rebalance, not mute), offers (seed/song/pool with per-bird cooldowns).
- Magic-link email auth, synthetic account UUID, encrypted email at rest, per-device revocable tokens.
- Multi-device: clients are pure renderers + event appenders; server owns all personality and mood state.
- Visit invitations: one-time email links, revocable, 30-day expiry, read-only ambient only, no presence recording from visitor, notifications opt-in and off-by-default.
- Accessibility first-class:
  - Running naturalist screen-reader narration at slow cadence.
  - designed reduced-motion cross-fade surface (not "animations off").
  - Call captions generated from the grammar at runtime.
  - Full keyboard nav + WCAG AA.
- Performance: <2 MiB gzipped initial bundle, <500 ms time-to-first-bird on mid-tier 4G, 60 fps idle on 5-year laptop, no client memory growth over 30 min.
- Privacy wall: per-bird events never leave the simulation service for any aggregate/ML purpose; only operational aggregate telemetry collected.

**Explicitly out of v1 (non-goals + explicit refusals honored):**
- No native apps (web-only).
- No gamification of any kind (streaks, scores, achievements, visit counters visible to user, etc.).
- No Tamagotchi mechanics (no hunger, distress, negative drift, death).
- No social network surfaces beyond the single quiet visit affordance (no profiles, discovery, leaderboards, comments, public aviaries).
- No push notifications, no "welcome back" announcements, no visible visit frequency surfaces.
- No catalog selection of birds; adoption is "the birds that arrived."
- No numerical trait exposure to users or debug surfaces.
- No client-side ownership of personality vectors or last-write-wins anything for state.
- No recorded audio loops for calls.

The plan below converts each of the preceding constraints into concrete work items, data shapes, sequencing, and risk mitigations for a team that will never need to re-ask "why".

## 2. High-Level Architecture
### 2.1 Service Topology (minimal, ownership clear)
- **Frontend SPA** (static assets + hydration):
  - Pure rendering + interaction capture + WebAudio context. Zero ownership of persistent state.
  - Bundled once; code-split for settings/notebook/visit flows.
- **Auth Service (thin)**:
  - Magic-link issuance / consumption, session token issuance/revocation, email change verification.
  - Owns only account identity tables (synthetic UUID primary, encrypted email secondary). Never authoritative for aviary state.
- **Simulation Service (core)**:
  - Canonical owner of every aviary record, bird personality vectors, moods, notebook.
  - Runs the periodic tick for all active aviaries even with zero connected clients.
  - Exposes:
    - Snapshot pull endpoint (small).
    - Append interaction-event endpoint (append-only log shard per account).
    - Tick trigger (internal cron or queue).
    - Visit token validation (read-only projection).
- **Asset / CDN layer** for the small static SVGs, palettes, motif data (procedural generators ship as code).
- **Email Delivery** (transactional only; no marketing).
- Single database logical boundary: simulation primary store is the source of truth; auth store is separate with foreign synthetic UUID only.

Rationale: server tick + additive deltas + synthetic IDs permanently eliminate the usual "two clients diverge" and PII sprawl classes of bugs before the first line of client code is written.

### 2.2 Client / Server Split
- Client: receives snapshot + renders + interpolates + emits events.
- Server tick consumes events in order, produces next snapshot + drift + mood.
- No client ever mutates personality; the event "listen-in on Pip for 184s" becomes a drift-delta inside the tick only.

### 2.3 Render / Simulation Boundary
- Snapshot contains authoritative "positions + velocities + mood enums + call-phase offsets + timing for next events" sufficient for the client to deterministically interpolate 10–60 s worth of micro-motion.
- Client does not run its own stochastic bird behavior generator except for pure-ornament leaves/feathers (client-side, non-authoritative).

## 3. Data Model (server-canonical only)
All persistent shapes live server-side; clients never persist personality vectors or notebook entries beyond session cache.

### 3.1 Core Tables / Documents
- Account
  - id: synthetic UUID (primary, immutable)
  - email_encrypted
  - created_at, last_activity
  - settings: {visit_notifications_enabled: bool (default false), reduced_motion: bool (opt-in override), captions_enabled: bool, ...}
  - visit_invites: list of pending + historical (see below)
- Bird (per aviary; aviary == one per account)
  - id: stable internal UUID (never changes even on rename/species migration)
  - account_id
  - species_id (enum 0-5 → look up silhouette + palette + grammar)
  - name: user string, renameable
  - adopted_at
  - personality: 5 floats [0.0-1.0] normalized
    - boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity
  - current_mood: enum
  - mood_last_transition_at (server wall time)
  - last_presence_window_end (timestamp)
- PresenceEvent (append-only)
  - account_id, bird_id? (global or per-bird), started_at, ended_at, duration_seconds (computed)
  - source: visibility+focus+activity conjunction record (audit)
- InteractionEvent (append-only log, sharded by account)
  - ts, kind (offer/ listen_start / listen_end / settle / presence_ping / ...)
  - payload: {target_bird?, offer_type?, duration_ms?}
- NotebookEntry
  - id, account_id, created_server_ts, prose: string (naturalist, lowercase, present-tense)
  - sparse creation policy (not per session)
- VisitInvite
  - token (one-time use, hashed), host_account_id, visitor_email, created_at, expires_at, revoked_at?, used_at?
- VisitLogEntry (host-only, append)
  - visitor_email, started_at, ended_at (or approx duration)

Personality vectors: store as 5-tuple of floats + last_drift_applied_tick_id so deltas can be replayable for repair.

Mood state machine: transitions table + decay timers per mood. Exact enum + transition matrix part of simulation-engine work package (see §5).

## 4. API Surface (exactly what clients and tick need)
REST or simple gRPC; keep small.

### 4.1 Snapshot Pull (GET /aviary/snapshot)
- Auth via session token (header).
- Response (tiny JSON):
  {
    server_ts,
    aviary_state: {day_phase, weather?, ambient},
    birds: [{
      id, species, name, perch_zone,
      mood, mood_params,
      personality_fingerprint_hash (for drift change detection, never numbers),
      call_phase, next_call_offset_ms,
      idle_motion_seed
    }],
    notebook_head: [last 3 entry ids + headline prose for prefetch]
  }
- Client may pass If-Modified / ETag + last_seen_ts to skip if unchanged.
- CDN edge cacheable for at most 30–60s per account (cache key = account+session nonce).

### 4.2 Event Append (POST /aviary/events)
- Idempotency key per request.
- Batch of one or more InteractionEvent.
- Server appends to log; returns 202. Tick will consume later.
- Presence pings are special-cased lightweight heartbeats inside the window rule.

### 4.3 Auth Endpoints (thin magic-link service)
- POST /auth/request_link {email}
- GET /auth/consume_link?token=... → session cookie or bearer + device id
- POST /auth/revoke_session
- POST /auth/request_email_change, verify flow (double opt-in)

### 4.4 Notebook Pull (paginated, cursor)
- Pure read of prose entries. Coarse page size; infinite scroll client ok.

### 4.5 Visit Flows
- Host: POST /visits/invite {email} → returns invite record
- Host: DELETE /visits/invites/{token}
- Host: GET /visits/log (paginated)
- Visitor (no account): GET /visit/{one_time_token} → returns projection same shape as snapshot but flagged read_only, visitor_mode
  - Token is single-use or time-bounded; simulation service validates + emits no events.

### 4.6 Settings / Account
- Matter-of-fact voice endpoints only: PATCH settings, export aviary JSON, initiate delete, etc.

### 4.7 Error Surfaces
All error surfaces use matter-of-fact voice per spec. No naturalist prose on system failure paths.

## 5. Simulation Engine Design
### 5.1 The Tick (~60 s ideal cadence)
Single logical writer per account. Pseudocode outline (actual impl must be pure and testable):

```
for each aviary due:
  now = server_time()
  events = InteractionEventLog.since(last_tick_ts)
  // 1. Drift deltas (presence first)
  presence_seconds = sum(PresenceEvent windows sliced to interval)
  for bird:
    delta = drift_function(presence_seconds, events, bird.personality)
    bird.personality = clamp_additive_expressive_only(bird.personality + delta)
  // 2. Mood transitions
  for bird:
    bird.mood = next_mood(
      current=bird.mood,
      personality=bird.personality,
      tod=local_time_for_account,
      recent_events=subset(events, last_15m),
      weather=ambient
    )
  // 3. Notebook generation (probabilistic sparse)
  if worth_observing(events, personality_delta, tod_shift):
    append NotebookEntry( naturalist_prose_from_state_diff(...) )
  // 4. Write next snapshot atomically + advance last_tick_ts
  persist(birds, notebook_delta)
  enqueue next scheduled tick (wall + jitter)
```

Drift function must be:
- Monotonic expressive: every component ↑ or flat.
- Presence dominant, listen-in secondary, offer marginal.
- Slow: design target: 1 week regular use ⇒ measurable in instrumentation harness; 3 weeks ⇒ user-visible character shift without numbers.

Calibration harness (synthetic accounts + scripted presence patterns) is required before first real-user ship. See §12.

### 5.2 Call Grammar Runtime (pure function of bird + mood + phase)
- Each species owns motif library (3–6 short motifs).
- Grammar: probabilistic finite state machines or tiny DSL for concatenation/variation.
- Output: per-call (frequency curve, amplitude envelope, micro-pitch variation, duration) and caption string in naturalist voice.
- Engine exposes deterministic(seed, phase, mood, personality) ⇒ render params.
- Client receives initial phase + seed; advances local clock (interpolates).

No stored audio buffers for primary path.

### 5.3 Mood State Machine & Ambient Drivers
- Inputs ordered: personality bias > time-of-day > recent interaction history (offer acceptance strong) > ambient weather (rare).
- Transitions must respect "no snapping" — afternoon drowsy should drift into evening settled, not reset randomly.
- Persistence: mood at session end is input to next snapshot.

## 6. Sync Model and Conflict Prevention
- One canonical writer (tick) + append-only log. Never reconcile divergent vectors.
- Clients are allowed to be arbitrarily stale (suspended laptop) and simply pull fresh snapshot on visibility restoration.
- Explicit conflict surfaces only on:
  - Magic-link replay / token replay → matter-of-fact "link expired".
  - Session timeout mid-write → re-auth.
  - Outage during tick window → client poll model + eventual consistency message.
- Export / hard-delete are the only two account-level operations that must coordinate across auth + simulation (soft-delete tombstone + 30d vacuum).

## 7. Frontend Rendering Pipeline
### 7.1 Scene Layers (z-order, canvas or WebGL2 choice to be made in first sprint)
- Background sky + distant foliage (parallax 0.05).
- Perch zones (front/mid/back) + branches.
- 2–7 birds (SVG or small textured quad + procedural feather detail).
- Foreground leaves / pass-through branches (client-ornament, 5–8 instances).
- Subtle sky color shift (tod).

### 7.2 Idle Micro-Motion System
- Per-bird anim graph driven by mood + personality (blend weights).
  - Wary → back perch + frequent scan
  - Content → preen cycles + occasional small shuffle
  - Curious → head tilts + weight shifts toward ambient sounds
  - etc.
- All animations authored as short idle loops or spline curves; never rely on CSS for personality.
- Frame timer: requestAnimationFrame + delta clamp; pause rendering entirely when document.hidden (but simulation marches on).

### 7.3 Transitions
- Birthing (only at initial adoption): soft 800 ms fly-in path from off-screen to perch.
- Perch change: smooth arc or hop (personality-modulated).
- Listen-in mix cross-fade (1.2 s) on audio side; visual cue: gentle scale or lighting lift on focused bird (purely visual affordance, no state change).
- Settle: 3–4 s global warm-dim + calls → ambient wind-down.
- Undo settle: fast reverse.

### 7.4 Reduced-Motion Mode
- Two completely separate render paths, not toggled filters:
  - Normal: full micro-motion + drift animation.
  - Reduced: slow (≈1.5–2.5 s) cross-fade between 4–6 canonical postures per bird per mood. Precomputed or interpolated on GPU.
- Day-night palette shifts remain (slowed).
- Leaf ornaments removed entirely; only static foliage.
- No vestibular-safe fallback can ever look "off"; this must ship as designed surface.

### 7.5 First-Frame Contract
- Snapshot delivers enough to render frame N immediately (current perch targets, base pose seeds).
- No spinner ever on main path; quiet-field placeholder only during very slow first pull (<300 ms target).

## 8. Audio Pipeline
### 8.1 WebAudio Graph (mandatory primary path)
- Master -> ChorusBus + AmbientBus.
- Per-bird procedural oscillator chain or grain synth (small footprint):
  - 1–2 detuned oscillators or noise+filter for timbre.
  - Amplitude envelope + gentle LFO per call motif.
  - Spatializer: very light stereo pan by perch zone (never distracting).
- Listen-in: smooth ramp of gain on focused bird (0.9 → 1.0 target) + sidechain ducking on others (0.6 → 0.2) over 800–1200 ms. Never 0.0 for any bird.
- Chorus blending must be phase-aware; procedural variation eliminates the dead-loop artifacts of layered tracks.

### 8.2 Call Scheduling & Grammar Invocation
- Client owns timing extrapolation between server snapshots.
- Server supplies next_call_offsets[] and motif seeds per snapshot.
- Grammar is JS module (shared between tick for state decisions and client renderer).

### 8.3 Fallback (no recorded audio)
- WebAudio unavailable (permission, old browser, broken hardware): full silence + captions forced ON.
- No audio file assets shipped for voices. This is a hard bundle + uncanny-valley preventative.

## 9. Accessibility Surfaces (ship concurrent with core)
### 9.1 Screen-Reader Narration
- Live region (polite) receives prose strings.
- Server can pre-generate the narration blobs on snapshot or client can deterministically produce from same state machine.
- Cadence: idle 40–60 s; bump priority +25 % on user actions.
- Text must survive switch between visual and pure-narration contexts without rephrasing voice.

### 9.2 Call Captions
- Generated identically to audio call metadata: same seed + grammar → prose caption.
- Positioned near bird; fade timing matches audio call duration.
- Respect reduced-motion (slower fades).

### 9.3 Keyboard & Focus
- Roving tabindex on birds + top-bar items.
- Arrow keys: spatial? or linear order along perch line.
- Enter = listen-in toggle.
- Escape = disengage.
- Focus ring: high-contrast soft halo (white + subtle drop) against any palette state.

### 9.4 Contrast & Palette
- Design-system tokens must deliver ≥ WCAG AA on all text over aviary and chrome.
- No reliance on hue for information (use luminance + pattern when needed).

## 10. Performance Budgets & Observability
Hard gates (failing any = blocking for v1):

- Gzipped main entry + critical path chunks ≤ 2 MB.
- LCP / first-bird-visible ≤ 500 ms on Moto G (4G) synthetic in 75th percentile.
- Idle 60 fps sustained 30 min on 2019 MacBook Air / mid-tier Windows laptop.
- Memory: no measurable heap growth (>3 MB tracked) over 30 min session (DevTools + synthetic).
- Audio context: reuse single context; no per-call buffer allocations that grow.
- Tick p99 latency < 5 s (alarm at 3 s).

Telemetry (aggregate-only, never per-account personality):
- Synthetic browser fleet (Playwright or equivalent) in 5 geographies, 1-min cadence.
- RUM: paint, frame, audio-error, tick-consumption client round-trip (anonymized).
- Error budgets: explicit SLOs written into runbooks.

Deliberately not measured:
- Any per-bird interaction histogram that could reconstruct relationships.
- Session duration exact for individuals.
- Click counts (presence is the metric).

## 11. Rollout & Ramp
### 11.1 Technical Ramp Sequence
1. Internal dogfood (team + 20 synthetic "bird watchers").
2. 50 alpha accounts (invite-only, friends/family).
3. 500 closed beta.
4. Gradual public: 100 new accounts/day → 500 → 2000, with kill-switch + rollback plan.
5. Bird count ramp: start all accounts at 2 birds; age-based gating revealed only after 60/180/270 days of account age respectively for #3,4,5.

### 11.2 Instrumentation Day-One
- All operational aggregates above.
- Explicit "did any notebook entry get written?" daily health check.
- Drift velocity instrumentation on synthetic harness (alert if median > 2σ from calibration target).
- Presence definition test harness (headless browser scripts exercising the three-conjunct logic; failure = regression in every future account).

### 11.3 Ship Criteria (non-negotiable)
- All perf budgets met on target devices.
- Narrated experience reviewed by accessibility specialist + blind tester.
- Reduced-motion reviewed by 2 vestibular-sensitive users; signed off as "calmer, not broken".
- Calibration harness shows week-1 and week-3 drift feel targets.
- Zero gamification strings or counters in shipped bundle.
- Visit feature exercised by 20 end-to-end invite/revoke/visit flows with no presence leak.

## 12. Risks & Mitigations (explicit table)
| Risk Area                        | Concrete Failure Mode                                      | Leading Indicators (instrument)                  | Mitigation (in plan)                                   | Owner Assignment (by role) |
|----------------------------------|------------------------------------------------------------|--------------------------------------------------|--------------------------------------------------------|----------------------------|
| Drift calibration                | One week of presence produces visible character shift or zero perceptible change | Synthetic drift-velocity p50/p95 | Dedicated calibration sprint + offline Markov model tuning before any real accounts created | Simulation Engineer + Stats |
| Sync correctness                 | Personality reset or last-write-wins during 2-device handoff | Deterministic replay test failures               | Pure event-log + tick only; 100 % coverage of replay tests using captured real sequences | Backend + Tests |
| Audio uncanny / canned feel      | User hears exact same motif twice in first two sessions    | Self-similarity metric on 1000 procedurally generated calls | Grammar must pass "repeat test" (nebula score) + listen-in A/B in alpha | Audio + Frontend |
| Accessibility regression         | Narration lags or repeats state dumps instead of continuous prose; reduced-motion feels stuttery | SR user sessions + motion-preference opt-in cohort | Accessibility owner embedded in every vertical slice from sprint 1 | A11y + Design |
| Bundle bloat / perf miss         | Time-to-first-bird crosses 500 ms on target hardware        | Auto synthetic CI gate                           | Aggressive code-split, procedural assets only, early perf sprints with real Moto G device | Frontend |
| Privacy wall breach (accidental) | Per-bird event row lands in analytics warehouse            | Column-level DLP + query log scanning            | Simulation DB physically/network-isolated from analytics DB; strict schema contracts | Data + Backend Arch |
| Social feature scope creep       | "Just a little comment box" or discovery feed appears      | Design review + PRD diff linters (literal string "leaderboard") | Explicit non-goal review at every milestone; no-discover surface review gate | PM + Eng |

## 13. Work Breakdown & Sequencing (macro sprints)
Sprint 0 (2 wks) — Foundation & Calibration
- Data model + tick skeleton + deterministic replay harness.
- Presence definition implementation + synthetic verifier.
- Personality drift math + first calibration targets.
- Basic auth + magic-link.

Sprint 1 (3 wks) — Core Alive Surface
- First 2 birds procedural idling + 1 species call grammar.
- Snapshot + event pipeline.
- Day/night + weather ambient.
- First naturalist notebook generator (sparse).

Sprint 2 — Frontend Visual Core + Listen-In & Offers
- Scene composition + micro-anim.
- Listen-in mix + offer system (cooldowns).
- Settle gesture + undo.

Sprint 3 — Accounts/Sync Polish + Multi-device Dogfood
- Session model + revokes.
- Conflict surfaces (matter-of-fact).
- Cross-device drift continuity verification.

Sprint 4 — Audio Production + Polish
- Full grammar library for 6 species.
- WebAudio graph + all transitions.
- Silence + caption fallback.

Sprint 5 — Accessibility Complete + Reduced-Motion
- Screen reader prose pipeline running live.
- Reduced-motion visual variant as first-class.
- Keyboard fully done.
- Caption generation finished.

Sprint 6 — Social (Visit) Minimal + Hard Privacy Gates
- Invite / revoke / logging flows end-to-end.
- Visitor projection (read-only).
- DLP/warehouse isolation enforcement.

Sprint 7–8 — Perf, Calibration Validation, Beta
- Bundle & runtime gates in nightly.
- Full calibration harness run on 30 synthetic weeks.
- Alpha → closed beta ramp.

Sprint 9 — Hardening & Ship
- All sign-off criteria checklists.
- Rollback, monitoring, runbooks.

## 14. Documentation Deliverables Alongside Code
- Every module must include a one-paragraph "why this protects the product contract" note (e.g., why the tick owns personality).
- ADRs for: presence definition, monotonic drift, procedural audio only, reduced-motion as designed surface, privacy wall architecture.
- Runbook: "what to do when tick p99 > 3 s".
- Calibration notebook (living doc) that survives the team that produced it.

## 15. Open Implementation Decisions (to be resolved in Sprint 0)
- Exact canvas/WebGL vs. DOM+SVG bird rendering (perf vs. asset size).
- Grammar DSL (tiny json rules? JS functions? compiled Wasm tiny interpreter? Keep the surface area tiny).
- Narration synthesis location (client or server pre-render cached by mood vector).
- Tick scheduling: queue vs. cron with jitter (must guarantee at-least-once per account).
- Exact presence inactivity window (this will be tuned on live users early; 3–5 min candidate stated in PRD; 180–300 s range).

These decisions are narrow; all major affective and architectural constraints have already been locked by the PRD.

## 16. Deliverable Confidence & Handoff
This plan is deliberately over-specified precisely where the product is most fragile (drift honesty, server-authoritative state, first-class accessibility, no-gamification boundary). A competent frontend + backend team with ownership discipline can take this document, the PRD corpus, and a design-system Figma and be shipping v1 within 12–14 weeks without returning to the spec authors except for the few explicit calibration choices recorded above.

Nothing in the plan implements or ships any part of Pocket Aviary. It only describes an executable path that honors every constraint given.

(End of comprehensive plan — 001 for wave_002 — grok-build-0.1 at unknown effort)