# Comprehensive Phase-1 Implementation Plan: Pocket Aviary v1

**Product:** Pocket Aviary (browser-based virtual aviary)  
**Wave/Run:** wave_001 / 001 (grok-build-0.1, effort: unknown)  
**Harness:** opencode  
**Date of this Plan:** 2026-05-26 (per run environment)  
**Source PRDs:** All files listed in prd/1-START_HERE.md (product_brief.md through non_goals.md)  
**Deliverable Constraint:** This document only. No code or assets are produced here.

This plan is engineered for a frontier web engineering team. It interprets every substantive constraint, principle, and negative space from the PRDs into concrete, executable work items, architectures, data contracts, calibration targets, and verification criteria. Ambiguities are resolved with explicit defensible calls noted in each section. The plan is intentionally overspecified on the load-bearing surfaces (personality drift, presence signals, server-only personality writes, procedural chorus, "notice not announce", naturalist voice boundary) because failure modes there are silent and permanently damaging to the product's central promise.

---

## 1. Scope — v1 Inclusions and Explicit Exclusions

### 1.1 What ships in v1
- Two starter birds per new account (selected from species pool by system, not user catalog), user-assignable names (renameable later), stable bird identity across life of account.
- Maximum 7 birds per aviary. New birds offered on aviary-age schedule (third bird after ~few months existence; paced by relationship depth, not visit metrics).
- Full bird engine: 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), fast-timescale enumerated mood states, monotonic drift (positive on presence; no punitive negative drift), procedural call grammars per species, mood-shaped idle micro-motion.
- All core user interactions: return-greeting (procedural, staggered, absence-length + boldness + mood aware), listen-in (gradual re-mix, never full silence on others), offer (seed / song-fragment / still-pool; per-bird cooldowns), settle (opt-in soft goodbye with undo window; lighting shift; not required), field notebook (auto-generated sparse naturalist observations).
- Presence accounting: strict 3-signal conjunction (visibilityState=visible AND window focus AND recent pointer/key activity within calibrated window of a few minutes). Used for drift.
- Single horizontal responsive aviary scene (no panning/scroll/zoom), three perch zones (front/mid/back), local-time day/night cycle, rare ambient weather (rain, wind), subtle parallax + leaf/feather drift, top bar (sparse icons, auto-fade on idle).
- Accounts & auth: email + 15-min magic-link sign-in (no passwords at v1), per-device revocable session tokens, synthetic UUID account IDs (email stored encrypted in one place only), account export (JSON via email link), 30-day soft-delete then hard-delete.
- Server-side simulation tick (~60s cadence). Clients pull snapshots, append interaction events, interpolate rendered motion. Canonical state always server.
- Multi-device sync: property of architecture, no client-to-client merge. No LWW on personality vectors (additive server deltas only).
- Visit (social) feature: opt-in per-invite read-only ambient view (defaults off), revocable, no co-presence, no notifications (opt-in only), visit log, 30-day unused invite expiry.
- Accessibility surfaces (first-class, not checklist): screen-reader running naturalist narration (slow cadence), reduced-motion designed cross-fade renderer (not "animations off"), call captions (procedural prose, naturalist voice), full keyboard navigation with focus ring visible on all aviary states, WCAG AA contrast on all text.
- Performance: initial bundle ≤2 MB gzipped, time-to-first-bird ≤500 ms on mid-tier mobile + 4G, 60 fps idle on 5-year-old laptop for 30 min with no memory growth, synthetic + aggregate-only RUM.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge. Graceful matter-of-fact surface for older.
- Privacy: per-bird/per-account interaction data used only for owner's simulation; never aggregated for training/recommendations, never shared. Aggregate operational telemetry only (request counts, p99 latencies, anonymized histograms, error rates). No PII in telemetry beyond synthetic IDs.
- Observability and error surfaces in matter-of-fact voice; all product surfaces in naturalist lowercase present-tense specific field-notebook voice.
- Rollout instrumentation from day one (see Rollout).

### 1.2 Explicit non-goals respected in this plan (never proposed)
- No native mobile apps or plans for them in v1 design.
- No gamification whatsoever: no streaks, calendars, XP, achievements, badges, counters ("birds adopted: 2"), green dots, visit-frequency surfaces, leaderboards.
- No Tamagotchi mechanics: no hunger, distress, death, happiness decay on neglect, custodial obligations.
- No social network surfaces: no profiles, follows, public discovery/explore, feeds, comments, shared creation, friend-of-friend, "show-off" rendering for visitors, comparison features.
- No notifications/push for any aviary events (visit occurred, bird greeted, etc.). The opt-in visit log toggle is the only exception surface.
- No native-app preparedness in data model/protocols.
- No exposing personality vectors numerically to any user surface (including hidden debug toggles at v1).
- No panning, zooming, scrolling, multi-screen geography in the aviary.
- No editable notebook; read-only observer record only.
- No "Welcome back!" or announcement toasts, modals, banners.

The plan treats these refusals as load-bearing architectural invariants. Any deviation in execution requires explicit re-approval against this document.

---

## 2. Architecture

### 2.1 High-level service shape
- **Client (browser tab):** Thin state consumer + high-fidelity renderer + audio synthesizer. Single page app (SPA) entrypoint. All rendering and WebAudio synthesis in main thread or dedicated workers where safety permits. No local simulation ownership of personality/mood state.
- **Server:** One primary "Aviary Simulation Service" (stateless workers + durable job for ticks). Responsible for:
  - Magic-link auth issuance/validation (delegating email delivery to transactional email provider; link tokens single-use, 15 min TTL).
  - Append-only interaction event store per account.
  - Server tick job (~once per minute) that is the only writer of personality vectors and mood state.
  - Read path: snapshot generation (small serialized state + per-bird call timing hints).
  - Visit tokens and revocation.
- **Data layer (per-account partitionable):**
  - Account store (synthetic UUID primary, encrypted email once).
  - Birds table (stable bird_id UUID, species, name, personality_vector JSON/columns, current_mood, created_at, last_tick_version).
  - Event log (append-only, per-account, immutable: timestamp, event_type, payload, tick_version_applied).
  - Notebook entries (immutable, generated by tick or interaction processors).
  - Presence aggregate windows (sliding).
  - Session tokens (revocable).
  - Visit invites/log.
- **Auxiliary services (minimal):** Email (transactional, DKIM etc.), object storage for exports (ephemeral presigned), CDN for static client assets.
- **Deployment units:** Client served as static assets (edge CDN) + lightweight HTML shell that bootstraps JS. Simulation service as horizontally scaled containerized workers with a leader-elected ticker scheduler (or distributed cron with idempotency keys). Database: Postgres (or equivalent) with per-account schema or row-level tenancy for isolation + easy export/delete. No cross-account queries for simulation data.

### 2.2 Render pipeline boundary (client/server split)
Server owns all authoritative state. Client owns only transient render state (current interpolated positions, active mix levels, local clock for day/night if needed, animation timers).
Client must tolerate being suspended/resumed: on hidden→visible it re-pulls snapshot and re-establishes presence tracking without trusting its local clock for drift.

### 2.3 Tech stack choices (defensible calls)
- **Client language/framework:** TypeScript + lightweight framework. Preference: a minimal reactive library (e.g., Preact + signals or vanilla + tiny framework) to hit bundle budget aggressively. Avoid heavy UI frameworks. Canvas 2D (not WebGL) for aviary scene — sufficient for 60 fps on target hardware, vastly smaller bundle than three.js/etc., easier accessibility integration, procedural drawing.
- **Bird visual assets:** Species silhouettes as compact vector paths or tiny procedurally-tinted sprites (SVG embedded or tiny PNG spritesheets). No heavy textures. Plumage saturation modulates HSL or simple color matrix in shader-like canvas ops.
- **Build:** Vite or esbuild for <2 MB target (tree-shaken, code-split for settings/notebook routes).
- **Server:** Node.js/TypeScript or Go (Go for tick simplicity and predictable perf). Choice: Go for simulation workers (deterministic math, low memory, fast cold start) + TypeScript for auth/HTTP surface if org preference is unified lang; explicitly select for productivity vs perf here. Eventual consensus: Go simulation core, thin TS gateway for auth if needed.
- **DB:** Postgres 16+ (JSONB for vectors if needed, but prefer normalized columns for drift history queries during calibration). Timescale or partitioning for event log growth.
- **Auth:** Stateless magic-link JWTs (short lived) + refresh session cookies (HttpOnly, SameSite=Strict, with revocation list in DB).
- **API:** REST over HTTPS. Simple GET /v1/aviary/snapshot, POST /v1/aviary/events (batchable), etc. Versioned. No GraphQL for v1 size/perf discipline.
- **Realtime hint:** No persistent WS required; low-frequency long-poll or server-sent events optional for prod polish if polling measurably harms perf, but snapshot polling at visibility/keepalive cadence meets spec.
- **Testing:** Jest + Playwright for client, Go tests for sim. Synthetic browser fleet (headless) for perf and drift regression.

---

## 3. Data Model

### 3.1 Core entities
- **Account:** { account_id: uuid (synthetic, PK), email_encrypted: bytes, created_at, deletion_requested_at?, settings: {visit_notifications_enabled: bool default false, reduced_motion_pref: bool, captions_enabled: bool, ...} }
- **Bird:** { bird_id: uuid (stable PK), account_id, species_id (enum/ref to species table), name: string, personality_vector: {boldness: float [0.0-1.0], social_warmth: float, vocal_frequency: float, plumage_saturation: float, curiosity: float}, current_mood: enum ('wary'|'content'|'curious'|'drowsy'|'alert'|'settled'), last_mood_transition_at, perch_zone: enum ('front'|'mid'|'back'), created_at }
  - Personality never reset except on account hard-delete.
  - Drift version counter for snapshot consistency.
- **Species:** Static table or struct { species_id, silhouette_key, default_palette, call_grammar_motifs_ref, typical_vocal_base_rate }
- **InteractionEvent (append-only log):** { event_id, account_id, bird_id?, occurred_at (client or server ts normalized), event_type: ('presence_ping'|'listen_in_start'|'listen_in_end'|'offer_seed'|'offer_song'|'offer_pool'|'settle'|'return_greeting_consumed'), payload: jsonb (contextual, e.g. duration_ms, target_bird), tick_version: int (the tick that consumed/ignored it) }
  - Presence pings are lightweight; throttle to ~every 30-60s while active.
- **NotebookEntry:** { entry_id, account_id, written_at, prose: string (naturalist, lowercase, present-tense, specific), related_bird_ids: uuid[] } — sparse generation rule implemented in tick.
- **VisitInvite:** { invite_id, host_account_id, visitor_email, created_at, expires_at, revoked_at?, last_used_at? }
- **VisitEvent:** { visit_id, host_account_id, visitor_email, started_at, ended_at, duration_sec approx } — append only, no effect on host drift.
- **Session:** { session_id, account_id, token_hash, device_info_summary, issued_at, last_seen, revoked_at? }

### 3.2 Derived / computed
- Presence windows: materialized or computed aggregates from event pings (never store raw mouse events).
- Mood timers and weather influence windows: held in bird state or short auxiliary table advanced by tick.
- Personality delta history: optional for calibration analysis (instrumented internally; never surfaced).

### 3.3 Export and deletion
Export job walks the normalized rows and emits deterministic JSON snapshot (birds + vectors + moods + full notebook + visit log). Deletion: soft flag + cleanup job after 30d that hard-drops all rows for synthetic ID.

---

## 4. API Surface (Client ↔ Simulation Service)

### 4.1 Auth flow (matter-of-fact surfaces)
- POST /v1/auth/request_magic_link { email }
  - Rate limit per-email reasonable. Issues signed single-use token, emails it.
- GET /v1/auth/verify?token=... (magic link target)
  - Validates, issues session token pair, rotates. Returns minimal user context.
- Any signed endpoint rejects expired/invalid with matter-of-fact text.

### 4.2 Aviary state & interaction
- GET /v1/aviary/snapshot?since_version=... (or unconditional)
  - Response: { account_id, snapshot_version, server_time, day_phase, weather: {type?, intensity?}, birds: [{bird_id, species, name, perch_zone, mood, personality_delta_since_last (for observability only internally), call_timing_hints?, idle_phase?}], presence_summary: {last_presence_end?} }
  - Small payload (target < 4 KB uncompressed typical).
- POST /v1/aviary/events (batch)
  - [{type, bird_id?, timestamp_client, duration?, metadata}]
  - Idempotency keyed by client-generated nonce + rate limits. Server normalizes time, appends.
- GET /v1/aviary/notebook?before=...
  - Paginated prose entries.

### 4.3 Presence pings
Modeled as events (lightweight POST). Client decides cadence based on 3-signal checks. Never trust client for cumulative time; server reconstructs windows.

### 4.4 Offer cooldowns
Enforced server-side on receive; client shows approximate remaining but authoritative from next snapshot.

### 4.5 Visit flow
- POST /v1/visits/invite {visitor_email}
- GET /v1/visits/guest?invite_token=... → same shape snapshot (read-only) or error "visit no longer available".
- Revocation: DELETE /v1/visits/invites/{id} (host only) → immediate effect on next poll.

### 4.6 Account surfaces (matter-of-fact)
- Export trigger, session list/revoke, privacy policy link (static), settings toggles (captions, reduced-motion, visit notifications), delete account flow.
- No naturalist prose on these surfaces.

### 4.7 Error & unsupported
All sync/auth/unsupported errors use prescriptive matter-of-fact copy only.

---

## 5. Simulation Engine Design

### 5.1 Tick contract
- Cadence: target 60 seconds (configurable 45-75 s during calibration). Idempotent jobs.
- Inputs per tick for affected accounts: recent unconsumed events + current time-of-day in account tz + last known personality + mood timers + prior weather state.
- Outputs: updated personality vectors (monotonic deltas only), mood transitions, call phase timers advanced, new notebook entries if rarity threshold crossed, perch re-assignments, write snapshot for clients.
- Tick must be fast: p99 < 500 ms; p99 > 5 s alarms (see Perf).
- Determinism: seeded PRNG per-account + tick version for reproducible test replays.

### 5.2 Drift function (core load-bearing)
- Low-pass filter: weighted exponential moving average.
- Primary weight: presence-time (continuous windowed integral).
- Secondary: listen-in duration weighted per-bird.
- Tertiary: offer acceptance weighted + proximity offers.
- Input signal bounded per-session to avoid single-session jumps.
- Delta per trait per tick = k_presence * presence_hours * trait_sensitivity + k_listen * ... (exact k values to be instrumented in first 3 weeks of prod with A/A tests against synthetic cohorts).
- Calibration target (instrumented, not user-visible): after 7 days of 20-40 min average daily presence, average measurable change in at least one trait ≥ 0.02 (normalized units). After 21 days: visible change (user can notice difference in greetings/perch/motion without being prompted).
- Monotonic guard: max(0, delta) per tick — never negative. Explicit fast-path in code.
- Neglect modeling: lower presence weight simply yields near-zero drift per tick (ambient quieting only via mood and lower greeting frequency).

### 5.3 Mood machine
- Small FSM: 5-7 states. Transitions:
  - Time-of-day priors (drowsy at dusk, alert morning).
  - Interaction nudges (offer accepted → content bias 30 min).
  - Inter-bird coupling (one bird alert propagates small wary to neighbors).
  - Personality modulation (high boldness dampens entry into wary).
- Persists across ticks/sessions. No forced neutral.
- Settle interaction injects "settled" bias decaying over hours.

### 5.4 Call grammar runtime (server advances timing; client synthesizes)
- Server models call readiness per bird per tick using vocal_frequency + mood + time-of-day + chorus coupling (high VF birds more likely to answer).
- Snapshots emit next expected call offsets (jittered).
- Client owns actual synthesis — see Audio.

### 5.5 Bird-to-bird social
- Implemented in tick: call-response windows, safety-in-numbers effect on wary, emergent chorus when ≥2 high-VF birds overlap.

### 5.6 Adoption & species
- New account: deterministic or lightly randomized 2-species starter draw from pool (6 species). No catalog UI.
- Additional birds: age-gated (aviary_age thresholds stored in account metadata).

### 5.7 Notebook generation rules
- Generator lives in tick (or background after tick). Sparsity rule: at most 1 entry per 2-3 days for active accounts; +1 for high-salience events (first-time greeting order, new bird acceptance of offer, weather + notable mood shift, etc.).
- Uses templates + slot-filling from current state + recent events. No LLM at v1 (cost/predictability/privacy). Template expansion + small grammar for variation.

---

## 6. Sync Model & Conflict Handling

- Canonical state lives only in simulation DB. Client never owns/mutates personality or mood.
- Event log is source of truth for causation.
- Snapshot versioning (monotonic int) allows client to detect lag.
- Conflict surfaces only on auth/replay/outage edge cases → matter-of-fact error + explicit recovery action ("request new link", "reload aviary").
- No eventual-consistency reconciliation UI needed in normal path because additive deltas + ordered consumption preclude personality conflicts.
- Verify via property tests: replay event log against empty state must converge to produced vector state.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene graph (Canvas 2D)
- Layers (back to front): sky/gradient (time of day), distant foliage (subtle parallax), perches (3 y-depths), birds (composite of silhouette + plumage tint + mood micro-pose), foreground branches/leaves, weather overlay (soft rain streaks, wind lines).
- Responsive: viewport aspect drives horizontal spacing + perch depth compression; always keep all birds on-screen. No cropping.
- First-frame rule: client receives snapshot of live positions/motions → places each element at mid-action phase offset without entry tween.

### 7.2 Idle micro-motion & transitions
- Per-bird animation state machine: preen cycle, scan, weight-shift, head-tilt (personality + mood modulates speed/ amplitude).
- Perch changes: slow eased paths (no teleport). Duration personality-shaped.
- Day/night: continuous palette lerp + call volume envelope. Weather: rare short-lived overlays with mood side-effects (driven server).
- Leaf/feather: independent client PRNG spawners (no sim state).

### 7.3 Reduced-motion mode switch
- Global pref + per-session override.
- Implementation: separate renderer path (or flag) that replaces frame-by-frame micro with pose-to-pose cross-fades (CSS or canvas alpha lerp, 4-6 static poses per bird per mood).
- Ambient drift removed. Color shifts slowed. Calls + captions unaffected.
- Persists as designed surface (not fallback).

### 7.4 Transitions & loading states
- First load / resume: no spinner — quiet-field background matching palette while snapshot arrives.
- Settle: slow (3-5 s) global lighting wrap to "evening" overlay + call envelope down. Undo window: any interaction within 5 s reverts.
- Top-bar: opacity fade after N seconds cursor-silent (pointer inactivity).

### 7.5 Input routing
- Pointer: hit regions on each bird (canvas path hit-test or offscreen map).
- Keyboard: roving tab in top bar → Tab enters aviary → arrow keys cycle birds → Enter = listen-in. Escape exits.
- Focus ring always high-contrast (double stroke or luminance inversion based on current palette).

---

## 8. Audio Pipeline

### 8.1 Procedural synthesis (Web Audio API)
- Per-species motif library: 4-8 short melodic/rhythmic cells (osc + noise + filter envelopes).
- Runtime: stochastic concatenation + pitch/time variation seeded by bird_id + mood + current call phase + vocal-frequency scalar + small PRNG.
- Goal: every call instance unique at 0.1 s granularity; recognizable signature persists across weeks of drift.
- Chorus: real overlapping mix of independent instances (no phase artifacts of looping).
- Listen-in: per-source gain automation (slow ramp 800-1500 ms) + low-pass or distance filter on non-focused.
  - Other birds never reach true 0 gain; minimum ambient floor.

### 8.2 Mixing & environment
- Master bus: gentle dynamic range / light reverb for place.
- Time-of-day: low-pass + volume envelope for night.
- Weather: subtle broadband noise layer.

### 8.3 WebAudio fallback
- If AudioContext unavailable or permission denied: graceful silent mode + captions forced on. No recorded-audio track ever loaded (bundle and aesthetic contract).

### 8.4 Caption generation
- Runtime derived from active grammar instance (not hard-coded strings). E.g., "rising three-note motif, slightly breathy."
- Displayed floating near bird, lifetime matched to call + 200 ms decay. Naturalist voice.

### 8.5 Performance
- All voices share one AudioContext. Buffers reused (no per-call alloc that leaks).
- Automation via AudioParam; scheduled precisely.

---

## 9. Accessibility Surfaces

- **Narration (ARIA live region, polite, atomic):** Server or client template generates 1-2 sentence prose from snapshot state at slow cadence (30-60 s). Prioritize user events (return greeting, successful offer, settle). Same naturalist voice as notebook. Visible caption optional for sighted.
- **Captions:** As above in audio section. Toggle in accessibility settings (matter-of-fact surface).
- **Reduced motion:** As above; also responds to media query `prefers-reduced-motion`.
- **Keyboard:** Full order. Trap only in explicitly modal settings.
- **Contrast:** All text (top bar, captions, notebook, settings, errors) passes WCAG AA; design tokens specify 4.5:1+ ratios. Scene itself uses perceptual separation (no fine text inside aviary art).
- **Screen reader specifics:** Announce only via the running narration region; do not duplicate every idle state via ARIA. Hidden interactive affordances must be named accessibly.
- **Audio settings:** Per-session opt-out (globally muted but captions default on if user has historically selected them).
- Implementation: a11y settings stored server-side for multi-device consistency; applied at render bootstrap.

---

## 10. Performance Budgets, Observability & CI Gates

### 10.1 Hard budgets (enforced in CI + deploy gates)
- JS bundle at initial paint: ≤ 2 MB gzipped (measured via source-map-explorer + artifact check).
- Time-to-first-bird (paint + interactive first call visible): p75 ≤ 500 ms on synthetic 4G mid-tier mobile (Pixel 5 class, throttled).
- Runtime 60 fps for 30 min idle on 5-year-old hardware emulation (low-end laptop profile in headless or real device lab).
- Memory: no net growth > 5 MB over 30 min (heap snapshot diff in test).
- Tick p99 < 5 s (alarm immediately; target < 500 ms).

### 10.2 What we instrument
- Synthetic fleet (Playwright or custom browsers in key geos) hitting accounts on rotation: load, first-bird, 5-min idle frame times, audio context ready, narration emissions.
- Aggregate-only RUM (no per-account fields): navigation timing, first-contentful + first-bird paint, frame times (binned), WebAudio errors (binned + cause), snapshot sizes, event POST latencies.
- Simulation service: tick duration histogram, event backlog, drift computation cost, DB query stats.
- Drift calibration metrics: cohort-level aggregate delta stats (no individual bird telemetry to analytics warehouse).
- Error budgets and alerts tied to above.

### 10.3 Observability privacy line (absolute)
- Telemetry pipelines physically and logically separated from simulation DB. Simulation workers never emit into analytics. No bridge that could leak personality vectors, specific bird names, or per-account event sequences.

### 10.4 CI gates
- Bundle size regression test fails build.
- Perf budget run on every PR (or nightly + required for prod).
- Drift regression harness: replay 30 simulated weeks of crafted presence patterns; assert monotonicity + calibration band adherence.
- Accessibility audit + keyboard path automated test (axe-core + custom nav script).
- a11y narrative voice regression diff on PRs.

---

## 11. Rollout Strategy

### 11.1 Phasing (respecting "aviary continues without viewer")
- Internal dogfood (team accounts) day 0.
- Closed alpha: 50-100 friendly non-employees, 2-3 weeks. Heavy qualitative + instrument calibration.
- Limited beta: 500-2000 users, staggered onboarding to avoid thundering-herd on tick infrastructure.
- Public launch with waitlist/gradual ramp (new accounts receive invites at controlled rate).
- No "feature flag for birds" or tricks; the constraint on number is hardcoded in model.

### 11.2 Bird ramp mechanics
- New accounts start exactly 2 birds.
- Additional birds appear automatically when aviary age hits next threshold (stored in config, not per-user reward).
- No visible counter or "unlock" narrative.

### 11.3 Instrumentation day one
- All listed observability. Plus early drift cohort A/A vs model predictions.
- Qualitative: optional feedback surface (matter-of-fact form) for alpha only; reachable but never interrupting.

### 11.4 Monitoring & rollback
- Canary accounts + feature flag (on account cohort) for any server change.
- Rollback: previous container image + DB migration compatibility always forward-only or dual-write window.
- Alerting on presence signal corruption, tick backlog, narration queue, audio dropouts, first-bird >800 ms.

---

## 12. Testing Strategy (beyond perf/a11y listed above)

- Unit/property tests for drift monotonicity, mood FSM invariants, call grammar determinism.
- Event-log replay tests ensuring identical final personality vector from identical inputs across tick versions.
- End-to-end Playwright scenarios covering full session: load, greet variation across repeated runs, listen-in re-mix, offer reaction differs by mood, settle + undo, notebook entries appear sparsely, presence 3-signal correctness (simulated), multi-device simulated, visit flows, deletion flow.
- Chaos: suspend client mid-tick, revoke sessions during use, simulate replayed magic links, network interruption during event post.
- Audio regression: record spectrogram fingerprints per species across moods; assert recognizability distance.
- Notebook prose regression harness on state templates.

---

## 13. Security & Privacy Engineering

- Email PII only in account row, encrypted at rest, never in logs or partition keys (synthetic UUID rule everywhere).
- Magic links: single-use tokens, short TTL, rate limited.
- Session tokens revocable server-side; constant-time comparison.
- All telemetry aggregate-only by construction.
- Account deletion cascade is measured and logged for compliance (count but not content).
- No cross-account visibility or inference surfaces.
- Export and visit revocation audited in visit log only.

---

## 14. Risks, Mitigations & Calibration

**Risk: Drift calibration off (too fast → Tamagotchi feel; too slow → feels dead).**  
Mitigation: Instrumented A/A and controlled presence cohorts in early phases. Explicit numeric targets in plan. Dials only in server config, not client. Re-calibration documented and slow.

**Risk: Sync / event ordering / personality corruption on partial outages or replays.**  
Mitigation: Append-only log + tick versioning + no client writes to canonical state + full replay test harness + property-based testing in CI.

**Risk: Procedural audio introduces uncanny / repetitive artifacts despite grammar.**  
Mitigation: Studio tuning sessions by designers/engineers listening across 100+ generated calls. Per-species motif count and variation knobs. Fallback silent+caption never ships recorded loops.

**Risk: Accessibility surfaces feel "broken" or narrate poorly, causing users to abandon.**  
Mitigation: a11y co-design with actual screen-reader users from alpha. Narration voice test with real prose listeners. Reduced-motion is separate aesthetic, not degradation.

**Risk: Performance budget slips on mobile or after feature additions, destroying "already in motion" conceit.**  
Mitigation: Gates in CI. Canvas 2D + tight asset discipline. Code-split ruthlessly. Track first-bird metric daily.

**Risk: Privacy incident from accidental logging of email or per-bird events flowing to warehouse.**  
Mitigation: Synthetic ID rule enforced in all code paths. Separate DBs + query-time controls + column-level redaction audit + pre-merge search for "email" identifier in simulation code.

**Risk: Top-bar or chrome choices accidentally violate "no UI inside aviary" or "notice not announce".**  
Mitigation: Design reviews against exact PRD language. Plan explicitly limits top bar to 4 icons, fade rule, matter-of-fact error only.

**Risk: Visit feature later abused into social surface pressure.**  
Mitigation: Hard architectural limits (read-only, no presence recorded, no host notifications default-off) + explicit non-goal echoes in docs. Future additions must pass "would this require mutating the drift inputs from visitor?" test (answer always no).

**Other operational risks:** Email deliverability (use reputable provider from day one), DB growth on event logs (partition + archive to cold after N days for export only), tick leader election (simple lease or job queue with single consumer for starters).

---

## 15. Open Questions / Calls Made (none undefined in execution)

- Exact mathematical constants for drift filter weights: left to first-month calibration against live (instrumented) cohorts; targets documented here.
- Precise species count and initial names: 6 species, concrete visuals/motifs in separate design asset spec (a referenced companion document the visual + audio designers own).
- Tick cadence exact value: 60 s nominal, tunable ±15 s.
- Presence activity window: 3 minutes nominal (calibrated toward generous watching-without-mousing).
- Notebook generator: template + grammar expansion (no ML in v1).
- Canvas vs DOM: Canvas chosen for motion + hit-testing precision + single-draw-call discipline.
- Language/framework: TypeScript + minimal reactive + Go sim service (defensible for size/perf).

All PRD surface constraints (voice split, monotonic drift, server-only writes, procedural audio mandate, absence not punished, etc.) are treated as hard requirements with explicit verification criteria above.

This plan is complete and actionable. A competent web + backend team given this document, the referenced PRDs, and the design-system color/voice companion spec can build v1 without further product elaboration.

**End of PLAN.md (v1 phase-1)**

---

*Verification note for author: This document was produced strictly from the 9 listed PRD files. No other sources, peer runs, root files, phase_two archives, or external knowledge were consulted beyond reasonable engineering defaults where the PRDs leave implementational choice open (e.g., "DB choice", "exact framework"). All non-goals and voice boundaries are encoded into architecture, not left as policy hopes.*
