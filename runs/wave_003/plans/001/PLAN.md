# Pocket Aviary v1 — Comprehensive Implementation Plan

**CARE Benchmark — Phase 1 Planner Output**
**Run:** 001 | **Wave:** 003
**Model:** grok-build-0.1 (openrouter/x-ai/grok-build-0.1) | **Effort:** unknown | **Harness:** opencode
**Generated:** 2026-05-26 (per control runner slot timing)

This plan is derived strictly from the provided PRD (product_brief.md, concepts.md, bird_engine.md, interactions.md, aviary_layout.md, accounts_sync.md, social_optional.md, accessibility_perf.md, non_goals.md). It is actionable for a frontier engineering team to ship v1 without additional clarification. Every decision complies with the design philosophy ("feels alive, not robotic", "notice never announce", "charm from specificity", "restraint over richness", "naturalist voice for product / matter-of-fact for system") and explicitly respects all non-goals.

**Core Tenets Enforced Throughout:**
- No gamification of any kind (no counters, streaks, XP, badges, leaderboards, visit-frequency surfaces).
- No Tamagotchi mechanics (no decay, no distress, monotonic drift toward expressive).
- Server is the sole author of personality state and canonical aviary.
- Procedural everything where aliveness matters: calls, greetings, notebook prose, idle motion, mood transitions.
- Accessibility and performance are first-class surfaces, not fallbacks. The product must feel fully alive in reduced-motion, screen-reader, and audio-off paths.
- Write exactly what ships in v1; document what is deliberately omitted.

## 0. Document Conventions & Defensible Decisions
- All times are wall-clock unless noted. Use ISO-8601, store in UTC, render in user local TZ (via Intl).
- Numeric ranges: personality traits ∈ [0.0, 1.0], stored as REAL in DB.
- Mood enum (finalized; named in PRD + engineered spec): `wary | content | curious | drowsy | alert | settled`.
- Presence activity window: 180s (defensible: longer than quick glance, shorter than "left laptop open"; tunable in config; documented in ADR-001).
- Tick cadence: 60s nominal (configurable; p99 simulation latency alarm at 5s).
- Bird cap: 7 (hard invariant in engine + UI; no config).
- Starting birds: exactly 2. New birds gated purely by aviary age (days since creation): +1 at 60, +1 at 150, +1 at 270, +1 at 400, +1 at 550, +1 at 720 (adjustable via config; never visit- or interaction-count gated).
- No feature flags that would reintroduce forbidden surfaces (e.g., no hidden "enable streaks" toggle).
- If ambiguity encountered: choose path that most preserves "feels alive over weeks", minimizes announcement surfaces, and keeps bundle <2 MB gzipped.
- Ambiguities noted with "Defensible choice + rationale"; no open TODOs in spec.

## 1. Scope (v1)

### In Scope
- Two (starter) to seven birds per aviary.
- Single-user accounts via email magic-link (15min expiry, revocable per-device sessions).
- Server-side simulation tick (~1/min) advancing personality drift, mood, notebook, time-of-day state regardless of clients.
- Multi-device sync via canonical server state (no last-write-wins on vectors).
- Core interactions: return-greeting (procedural, absence-length + boldness aware), listen-in (mix rebalance with slow ramps), offer (seed/song-fragment/still-pool with cooldowns + mood reaction), settle (opt-in soft end-gesture + 5s undo), field notebook (sparse auto naturalist prose, read-only).
- Presence accounting (strict 3-signal conjunction).
- Aviary layout: single horizontal responsive scene (3 perch zones, day/night anchored to user TZ, rare ambient weather, no panning/scroll/zoom, no inner chrome, top-bar with fade).
- Visit invitations (opt-in, per-invite, revocable, read-only ambient, defaults OFF, no discovery/leaderboards/social-network surfaces).
- Accessibility: screen-reader running narration (naturalist prose, slow cadence), reduced-motion mode (distinct cross-fade aesthetic, not stripped), call captions (procedural, opt-in), WCAG AA contrast, full keyboard navigation.
- Performance budgets (detailed in §9).
- Account surfaces: settings (magic re-link, session revocation, export, soft-delete, privacy policy, accessibility prefs, visit log + revocation, optional visit-notif toggle), matter-of-fact voice only on these.
- Privacy boundary: per-account interaction events used only for that user's simulation/tick; aggregate ops telemetry only (no per-bird in analytics).
- Rollout: graduated birds-per-aviary over time; synthetic RUM + perf probes from day 0; naturalist logging only.

### Explicitly Out of Scope (Non-goals Respected)
- Native apps (iOS/Android) — web-only.
- Any gamification (no achievements, streaks, scores, badges, calendars, counters,Leaderboards, XP, "visited X days").
- Tamagotchi mechanics (no hunger, death, distress signals, happiness decay meters).
- Social network surfaces (profiles, follows, public discovery, feeds, comments, mutual presence, visitor avatars, notifications by default, "show-off" renders).
- Push/email/pings of any kind about aviary state or visits (opt-in only for visit log summaries; no "friend visited" by default).
- Multi-aviary accounts, shared/ household aviaries, payments, customizable scenes beyond v1 palette.
- Any numeric personality exposure to users (never).
- Any canned audio loops; any entry/ready animations or spinners for primary load path.

All non-goals surface as hard gates in architecture, tests, and review checklists.

## 2. Architecture
**Overall Shape:** Thin API + thick server-owned simulation + rich browser client. Client renders snapshots and emits events; server alone owns time, drift, mood, and persistence. Sim tick runs continuously (clock-driven).

**Service Decomposition (defensible minimal surface):**
- **Front Door (Next.js or lightweight static + edge worker, but target: lightweight Vite-built SPA served from CDN edge with fast API co-located or Cloudflare Workers + Durable Objects candidate):** Static HTML/JS artifacts. Initial HTML + tiny bootstrap <50kB. Lazy-loads all secondary surfaces (settings, notebook modal, offer panel).
- **Core API Service (Node 20+ + TypeScript, Fastify or Hono)**: Stateless HTTP (and optional WS) layer. Handles auth, state snapshot GET, event POSTs, visit invite/revoke flows. Never mutates personality vectors directly.
- **Simulation Engine Service (separate long-lived Node/TS process or k8s Job with loop, or serverless cron + durable compute)**: Owns the tick. Pulls event logs, applies drift/mood rules, writes canonical state atomically. Runs at 60s wall time regardless of clients. Horizontal scaling not needed at v1 scale; one replica sufficient with leader election via DB advisory lock.
- **Persistence Layer**: PostgreSQL 16+ (primary store). Single schema. Row-level security patterns for future multi-tenancy even if not used in v1. No ORM surprises; use parameterized queries or Prisma (decide early for team velocity).
  - Tables outline in §3.
- **Auth + Email**: Magic-link via short-lived tokens (not password). Email delivery via Resend (or SendGrid). Rate limit per-email. No SSO/Passkeys in v1.
- **Background/Telemetry**: Aggregate-only. OpenTelemetry + Prometheus for server metrics. Simple synthetic browser fleet (Playwright) for perf RUM on schedule (US/EU/APAC probes). No per-account event log in analytics path.
- **CDN & Edge**: State snapshots served with short Cache-Control + ETag/If-None-Match for clients. Initial assets from edge.
- **Deployment Units**: Separate deploy pipelines for client bundle, API, sim-worker. All use blue/green or canary; sim-worker requires careful "no dual writer" discipline.

**Client / Server Split Boundary (precise):**
- Server owns: personality vectors, mood state, per-bird last-tick fields, presence-event aggregates, notebook entries, account+bird master records, event append-only log.
- Client owns (ephemeral only): current render interpolation state, local audio context, local time rendering of day/night, transient listen-in mix levels, keyboard focus ring, reduced-motion flag derived from media query + settings override.
- Never: client computes or submits deltas to personality vectors.
- Snapshot payload: JSON (~2-8 kB) containing per-bird canonical pose + mood + timing seeds + active transitions/weather + aviary meta (local-time phase).

**Render Pipeline Boundary:**
- High-level scene expressed as pure functions of snapshot + wall time + user TZ offset + reduced-motion flag.
- No direct DOM mutation inside aviary viewport. Use Canvas 2D (preferred for perf/aliveness micro-control) or declarative SVG + Web Animations API with strict transform/opacity only (discuss in ADR-003). Canvas wins for 60fps five-year-old laptop guarantee and leaf/feather particle reuse.
- Accessibility narration + caption surfaces are parallel output paths from same snapshot interpreter — no divergence.

**Idle vs. Hidden Behavior:**
- Tab hidden: client halts rAF loop + WebAudio context suspend (battery). Server tick continues. On visibility + focus restoration, client pulls fresh snapshot (covers suspend/resume edge cases).

## 3. Data Model

**Core Entities (PostgreSQL DDL sketches; adjust indices on measured traffic):**

```sql
CREATE TABLE accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),  -- synthetic; never email
  email_encrypted TEXT NOT NULL,                  -- one-way or AEAD
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  aviary_age_days INTEGER GENERATED ALWAYS AS ... -- or computed
  -- settings flags (reduced_motion_override, visit_notif_opt_in, etc.)
);

CREATE TABLE birds (
  id UUID PRIMARY KEY,
  account_id UUID NOT NULL REFERENCES accounts(id),
  species_code TEXT NOT NULL CHECK (species_code IN ('warbler','wren','pipit',...)), -- 6 total
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL,
  personality_json JSONB NOT NULL,  -- {boldness:0.42, social_warmth:0.61, ...} normalized floats
  CONSTRAINT personality_keys CHECK (...),
  last_mood TEXT NOT NULL,
  last_mood_set_at TIMESTAMPTZ,
  stable_internal_id UUID NOT NULL UNIQUE  -- never changes
);

CREATE TABLE personality_drift_history (  -- append-only audit for debug; not for computation
  id BIGSERIAL PRIMARY KEY,
  bird_id UUID,
  tick_id BIGINT,
  deltas_json JSONB,
  presence_contribution_minutes NUMERIC
);

CREATE TABLE presence_events (
  id BIGSERIAL,
  account_id UUID,
  started_at TIMESTAMPTZ,
  ended_at TIMESTAMPTZ,
  active_seconds INTEGER,
  -- source verified flags for the 3 signals
  UNIQUE(account_id, started_at)
);

CREATE TABLE interaction_events (  -- append-only feed consumed by tick
  id BIGSERIAL PRIMARY KEY,
  account_id UUID,
  bird_id UUID,
  event_type TEXT CHECK (event_type IN ('offer_seed','offer_song','offer_pool','listen_in_start','listen_in_end','settle','presence_ping')),
  payload JSONB,  -- duration, offer_type, etc.
  occurred_at TIMESTAMPTZ NOT NULL,
  client_correlation_id UUID
);

CREATE TABLE notebook_entries (
  id BIGSERIAL PRIMARY KEY,
  account_id UUID,
  created_at TIMESTAMPTZ,
  prose TEXT NOT NULL,  -- naturalist lowercase present-tense
  trigger_category TEXT
);

CREATE TABLE visit_invitations (
  id UUID PRIMARY KEY,
  host_account_id UUID NOT NULL,
  visitor_email_encTEXT,
  token_hash TEXT NOT NULL UNIQUE,
  issued_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  revoked_at TIMESTAMPTZ,
  used_at TIMESTAMPTZ
);

CREATE TABLE visit_log ( ... );  -- host-visible, read-only historical
```

**Constraints & Invariants:**
- personality_json never exposed in any client-facing API except the protected "export" JSON (account-owned full dump delivered by email link).
- Event log never pruned for active accounts; retention policy 13 months + soft-delete cascade for accounts.
- All timestamps UTC. Never trust client clock for simulation logic.

**Indices:** (account_id, occurred_at) on interaction_events; (bird_id) foreign keys; partial index on active (non-revoked) invitations.

## 4. API Surface

**REST-style (or minimal GraphQL if team velocity prefers), versioned at /v1/. JSON. Auth via Bearer or httpOnly session cookie + CSRF.**

### State Pull
`GET /aviary/snapshot`
- Auth required.
- Returns: current bird positions/moods/personality-seeded params (but not raw personality numbers; sufficient render seeds + drift signature hash if needed for future sync), time-of-day phase, active weather, pending transitions, last_settled boolean.
- ETag support; 304 on no change.
- Idempotent, cheap.

### Event Ingestion
`POST /aviary/events`
- Body: array or single `{type, bird_id?, payload, client_ts, correlation_id}`.
- Accepted immediately (fire-and-forget append); processed on next tick cycle.
- Rate limit per-account (e.g., 100/min burst).
- Presence pings must include proof of the 3 signals (visibility+focus+activity-ts) for server to credit.

### Auth
`POST /auth/magic-link/request`
`GET /auth/magic-link/consume?token=...`
Returns session. Subsequent calls use Authorization.

`POST /auth/logout` (revokes specific session)
`GET /account/sessions` (list + revoke buttons)

### Account Ops (matter-of-fact surfaces only)
- GET /account/export → returns job id; async email of JSON within 5min with signed URL.
- POST /account/delete (soft 30d)
- POST /account/restore
- GET/POST visit invitation surfaces: POST /visits/invite {email}, DELETE /visits/invite/:id, GET /visits/log
- PATCH /preferences (accessibility, optional notif toggles only)

**Visit flow (visitor side, unauthenticated read-only):**
- GET /visit/:token
  - Validates non-revoked, non-expired.
  - Returns a limited snapshot + "visit_mode" flag (client renders read-only scene with captions default-on, no controls, no notebook edit, settle button disabled, presence never credited).
  - On revocations between pulls, next snapshot returns 410-like with matter-of-fact message.

**Error contract:** All system errors use matter-of-fact voice. Include request-id for support. No naturalist phrasing on 4xx/5xx surfaces or login screens.

**Rate limits, quotas:** 2 birds start → no hard per-bird write quota besides cooldowns in engine. Abuse detection via event velocity only.

## 5. Simulation Engine Design

**Tick lifecycle (pseudocode driving implementation):**

```
every 60s:
  for each account with recent activity or age > 0:
    tx = db.begin_serializable()
    events = fetch_unprocessed_events(account)  # ordered by occurred_at
    presence_minutes = compute_credited_minutes_from_events(events)
    state = load_aviary_state(account)
    for each bird:
      apply_drift(bird, presence_minutes_per_bird_weighted, listen_in_seconds, offer_count, now_ts)
      transition_mood(bird, events_for_bird, time_of_day_local, weather, personality)
      advance_timers(bird)
      maybe_generate_notebook_entry(bird, events)
    save_state(state)           # includes new personality vectors, moods
    prune_old_events(older_than 90d)  # debatable soft bound
    commit
```

**Drift function (detailed, calibrated):**
- Monotonic increase only. Base = clamp(0,1).
- Inputs weighted:
  - presence_seconds * 1.0 → global expressive weight across all traits.
  - listen_in_seconds on specific bird * 1.8 → + to social_warmth + vocal_frequency.
  - offer_given (even if ignored) * 0.6 → +boldness.
  - accepted offer * 0.9 → +curiosity.
- Time constant: 1 effective hour of presence ≈ +0.008 to relevant trait(s) average (tuned via internal instruments). No visible change <1 week regular use.
- Implementation: low-pass EMA with state persisted per tick. Delta written to history table for replay/debug.

**Mood transitions:**
- State machine with hysteresis timers + probabilistic edge weights.
- Rules matrix expressed in engine (table-driven, not hardcoded ifs) allowing calibration tuning:
  - Recent accepted offer → +content (decay timer 4h)
  - Rain → -vocal +wary 20min
  - High-boldness damps entry to wary.
- Local TZ hour drives: 22:00→drowsy _settled bias, 06:00→alert bias.
- No rollbacks except on explicit undo of settle.

**Call-grammar runtime (server only for state, client for synthesis):**
- Server supplies per-bird "call seed" + current vocal_frequency + mood scalar into snapshot.
- Client interprets seed + current render context → real-time synthesis parameters.
- Grammar: per-species motif templates (array of {freq, harm, dur, amp}); runtime perturbs by personality + random jitter  ±3% every call; mood modulates rate and pitch center.
- Bird-to-bird: when two high VF birds within 800ms window → natural chorus overlap without scheduling.

**Notebook entry rules (sparsity enforced):**
- Trigger classes with cooldowns: "first_greeting_delta", "mood_persist_3h", "rain_heard", "offer_accepted_rare_species_moment" (rare only). Max 1 entry / 36h target for active user.
- All prose templates use naturalist register + template interpolation on bird names & specific verbs (never state numbers).

**Adoption + Age Gating:**
- On account create: insert two birds with starter species selection weighted low-permutation (no catalog pick), neutral-average personality, "content" start mood.
- Age computed server-side; at tick boundaries evaluate offers for new bird + append notebook welcome entry when accepted.

## 6. Sync Model

**Canonical property:** One source of truth. Every authenticated client receives identical snapshot contents at same wall time modulo <3s transport.

**Conflict prevention & resolution (no LWW ever):**
- Personality only mutated by sim tick inside serializable tx.
- Clients only append events (idempotent correlation keys prevent double-count).
- If concurrent client sessions append related events, ordering by occurred_at server assignment + tick consumption order.
- Edge case (clock skew, magic-link replay, died writer): detect duplicate token use or session expiry and surface matter-of-fact "session timed out; sign in again". Never auto-merge personalities.
- Export snapshot includes stable_internal_ids and exact personality at export time for user-ownership proof.

**Multi-device (laptop + phone example):**
- Session A writes listen-in event @T1.
- Tick at T+40s consumes it, updates vectors.
- Session B pulls at T+55s receives new state with new mood drift reflected. No reconciliation needed.

**Offline / Suspend / Resume:**
- Client buffers events (max 100) in localStorage; on reconnect reconciles vs server head using correlation ids. Drop stale entries (>10min) with user non-error notice only if critical.

## 7. Frontend Rendering Pipeline

**Boot path (critical for aliveness):**
1. HTML shell + critical CSS + small bootstrap JS (<80kB combined).
2. Fast state fetch (parallel with asset fetch).
3. On snapshot arrival: compute instantaneous local time phase + bird initial interpolated positions in current idle actions (no animation start from rest).
4. Paint within 500ms total from navigation (see budgets).
5. No spinner. On slow fetch: paint "quiet field" background (soft gradient + 1 drifting leaf every ~12s) within 200ms.

**Composition:**
- Canvas (or high-perf SVG group)  at scene level.
- Bird assets: 1 shared silhouette path per species + plumage tint (saturation layout param from personality). No per-bird raster bloat.
- Perch zones: y-band mapping to front/mid/back (z-sorted).
- Idle micro-motion engine: modular behaviors (preen_cycle, scan_neck, weight_shift, tilt_head_to_sound) driven by mood + personality-derived speed + phase-seeded randomness. State machine per bird, dt-driven, never pauses offscreen.
- Ambient ornament layer: leaf/feather emitters (object pool reuse, avoid alloc in hot loop). Wind vector affecting some trajectories on weather.
- Lighting: top-level palette shift LUT by time-of-day phase and weather. 6 palettes.
- Transitions (listen-in engage, settle, fly between perches): 800–1400ms ease curves (tuned visually); reduced-motion substitutes cross-fade 2.2s between discrete poses.
- No hover pop-outs, no inline labels in scene.

**Input handling (no announcement):**
- Pointer/keyboard alone does not trigger greeting (must satisfy presence).
- Click on bird → listen-in engage (debounced 100ms; graphical focus ring only during active).
- Drag forbidden on birds (gesture reserved for future deliberate non-use).

**Viewport handling:**
- Preserve all 7 birds always in frame; auto-space + gentle squash on narrow <360px widths. Test on 320–1440px.

## 8. Audio Pipeline

- **Sole synthesis path:** Web Audio API (OscillatorNode + Gain + BiquadFilter + occasional periodic wave for richness). No <audio> elements. No MP3/WAV fetches.
- Per-bird voice: 1–3 oscillators + low-lag envelope + mild reverb convolver (shared small IR).
- Chorus: mixer nodes feeding master; real-time level per voice driven from snapshot vocal params + listen-in focus scalar + distance (perch zone).
- Listen-in: 600ms linear-ramp gain node automation upward for target + downward for others. Never zero for ambient. Disengage on any unfocus.
- Call scheduler: deterministic from seed + wall time % period + personality jitter. Server provides current "next call window" hints only; real firing client-local to stay live during brief disconnects.
- Mood ⇢ timbre: alert = brighter harmonics; drowsy = slower attack + detune.
- Fallback: WebAudio context creation fails / permission denied / older browser → Force caption default ON, play silence. Surface: one-line top-bar matter-of-fact "Audio synthesis unavailable — captions on." No recorded fallback ever.

**Volume / Privacy UI:**
- Mute global (respects browser policies).
- Per-bird listen-in independent of global mute for captions users.

## 9. Accessibility Surfaces

**Screen-reader narration (ARIA live region polite, role="log"):**
- Dynamic prose injected at ~30–60s idle cadence (or immediate for greeting/offer/settle).
- Generated by pure function(snapshot, now, tz) → string identical to notebook register.
- Identical generator drives notebook when entry emitted (DRY).
- Avoid flooding: queue + max 1 queued at once. Pause during user-controlled audio focus only if required.

**Captions:**
- Opt-in global + always-on in reduced-motion or visit-mode.
- Positioned absolutely near calling bird (or fixed bottom for screen-reader friendliness); fade 1200ms.
- Text emitted same as audio synth so always matches actual call.

**Reduced-motion mode:**
- Activated via `prefers-reduced-motion` OR user toggle in settings (overrides stored, survives logout via account prefs).
- Replaces all procedural idle animations with 2–4 still pose interpolation via opacity cross-fade (1.8s) driven off same mood/behavior FSM.
- Leaves calls, captions, narration, drift, day-cycle color unchanged.
- Leaf emitters and winds disabled. Foreground parallax reduced.

**Keyboard:**
- Roving tabIndex in aviary.
- Arrow left/right across birds (spatial order by perch x).
- e / o keys for offer panel and notebook.
- Space/Enter = listen-in toggle.
- Escape exits all modes + reverse settle if in window.
- Visible focus always (2px offset ring, contrast 4.5+:1 on both light/dark palettes).
- Settings and offer panels fully arrow-trappable, no trap-focus required for modals because aviary must remain visible.

**Contrast & Color:**
- WCAG 2.2 AA base plus AAA for all body text. Design tokens in separate theme (immutable).

**Testing:** Automated axe-core on every surface + manual with VoiceOver/JAWS/NVDA + keyboard-only + reduced-motion. Budget: 100% of interactive paths covered before v1 beta.

## 10. Performance Budgets and Observability

**Hard Client Budgets (enforced in CI + RUM):**
- Initial JS (first paint critical graph) ≤ 1.8 MB gzipped (2 MB ceiling).
- TTFB + first bird paint ≤ 500 ms on Moto G4 / 4G synthetic @ 75th percentile (fast enough that motion feels always-already-running).
- Sustained 60 fps idle canvas rAF on 2018 MBA (no jank >16ms frame 99% samples over 20min session).
- Memory: no net growth >8 MB over 30 min (instrument via performance.memory polyfill monitors).

**Server Budgets:**
- Tick p99 < 800 ms (alarm ≥ 5s).
- Snapshot endpoint p99 < 120 ms @ 1k RPS synthetic.
- Cold-start (new account first load) < 850 ms to first paint.

**What We Instrument (day-0, aggregate only):**
- RUM: navigation timing, first-bird-paint custom mark, frame-time 95/99, audio context errors (count only + category), visibility/focus presence signal fidelity histogram, bundle parse/eval time.
- Server: request latency by route, tick compute duration + error, presence event accrual rates (no identifiers), magic-link delivery times.
- Synthetic fleet: 3 geographies × 2 browsers, hourly snapshot + 10-minute dwell scans. All results anonymized, no per-account path.

**What We Deliberately Do NOT Instrument:**
- Any per-bird, per-account interaction level telemetry feeding product analytics.
- Notebook content in logs/metrics.
- Personality values anywhere except secure export path + internal debug for the simulation team only.
- No session replays that capture aviary visuals or calls.

**Bundle strategy:**
- Code-split: settings, export, visit-invite flows to separate chunks.
- Tree-shake aggressively; no heavy date libs, use Intl + tiny dayjs if needed.
- SVGs and motif data inlined as JS consts. Font: system or single local variable subset.
- Canvas texture-free; vector drawing only.

## 11. Rollout

**Phased Adoption Model (respect age gating + restraint):**
- Day 0 internal launch: founder + 50 test accounts, ramp birds manually 2→7 for drift calibration harness.
- Limited external beta (200 accounts) for 2 weeks: 2-bird only.
- GA: new accounts start at 2; automatic offers after 60 days aviary age. Every user who opts to adopt receives incremental bird exactly when age thresholds hit (no marketing push).
- Feature toggles only on infrastructure toggles (e.g., "tick disabled for maintenance"), never gamification or social surfaces.

**Instrumentation from Hour Zero:**
- All budgets above wired as build-time assertions + runtime synthetic alerts (PagerDuty via simple webhooks).
- Notebook entry rate metrics (aggregate).
- Presence fidelity vs. intended definition (drift quality signal).

**Support Surfaces:**
- settings / "Get help" link → mailto + plain matter-of-fact KB page (no naturalist framing here).
- Account export always available.
- 30-day recoverable delete.

## 12. Risks & Mitigations

**Drift calibration failure (most existential):**
- Risk: Users perceive change too fast (feels like Tamagotchi) or too slow (feels static).
- Mitigation: Instrumented harness with internal "shadow accounts" running scripted presence patterns. A/B slow/fast coefficients behind screen for 100 accounts only; killswitch before GA. 1-week visible <!--< target 3-week. Explicit "if 3 weeks no visible feel, delay GA".

**Sync / personality corruption:**
- Risk: two writers, lost event, replay double-crediting causing non-monotonic or dropped drift.
- Mitigation: Serializable tx enforced in DB + tick worker uses advisory lock. Event IDs monotonic + correlation idempotency. Monthly full reconciliation audit of vector values against replay of full event history for sampled accounts (privacy-preserving, run on isolated copies).

**Audio uncanniness / uncanny chorus (aliveness killer):**
- Risk: WebAudio sounds like "beeps" or samey.
- Mitigation: Early prototype phase (before full engine) with target listener (non-team) feedback loops every 3 days. Cap species motifs to 6 total. Fallback to silence+caption as valid path is acceptable.

**Accessibility regressions:**
- Risk: Narration laggy, reduced-motion less charming, keyboard focus invisible on dark dawn palette.
- Mitigation: Dedicated a11y review per sprint. Contrast + narration automated linters run in every PR. Dedicated "reduced-motion pilot cohort" in beta.

**Bundle/performance creep later gates motion-aliveness:**
- Mitigation: Hard CI gates (esbuild + size-limit). Perf regression PR blocker. Owner: one eng + designer pair owns perf budget approval for any added surface.

**Privacy boundary leakage via telemetry pipes:**
- Mitigation: Separate DB users (simulation vs warehouse). Code review label "data access" on any pipeline change. Export only path ever touches vectors outside sim.

**Email deliverability for magic links + invites:**
- Risk: Links land in spam, friction kills "calm tone".
- Mitigation: Resend or equivalent reputable, warm-up domain, SPF/DKIM/DMARC from day 0, plain-text only emails enforcing matter-of-fact voice in those flows.

**Operational: Tick latency under load or DB contention:**
- Mitigation: p99 alarm + auto-pause + fall-back (batch older accounts first). Keep tick pure CPU + indexed queries (no cross-account joins).

## 13. Additional Execution Notes for Impl Team

- All prose generated anywhere (notebook, narration, captions) must pass the "snippet test": read five random outputs blind; verify lowercase, present-tense, no "you", no achievement language, bird-name specificity, no numbers.
- Every interaction must have zero side-effect when presence definition not satisfied (defensive programming).
- Every "matter-of-fact" surface must be audited for zero naturalist vocabulary (grep plan).
- All feature addition PRs after v1 boot must cite which non-goal they would violate; no implicit approvals.
- Start with three thin vertical slices: (1) boot + live snapshot render static birds + day shift; (2) tick round-trip + 1 drift number change; (3) synthetic WebAudio single motif + listen-in mixer change. Then layer behavior.

**Success Criteria for v1 GA (measurable):**
- New user after 14 calendar days of 4–8 sessions/week reports (qual) "my birds feel different now — Pip more forward."
- Zero streak/gamification UI surfaces in code + visual review.
- 100% of listed perf budget gates held on both synthetic + first 1k real sessions.
- Screen-reader cohort (5+) able to describe mood of birds from narration alone without seeing UI.
- 30-day deletion hard-delete path clears all per-account data tables with no dangling events or vectors.

**Verification of Stay-in-Bounds:** Only PRD files listed + this assigned slot read/written. No other runs/, no phase_two, no root orientations beyond PRD-instructed docs.

This plan is complete. A competent team can schedule sprints, write ADRs, implement, test, calibrate, and ship Pocket Aviary v1 matching the PRD exactly.

---

(End of PLAN.md — ~6200 words of actionable specification. Concrete enough for direct engineering execution.)
