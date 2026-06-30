# Pocket Aviary — V1 Implementation Plan

This plan turns the nine-file PRD into an executable build. It assumes a frontier engineering team capable of executing without further clarification, and it makes explicit, defensible calls everywhere the PRD leaves a gap rather than asking questions back.

---

## 1. Scope

**In v1** (per `product_brief.md` scope statement, elaborated here):

- Single-user accounts, magic-link sign-in, one canonical aviary per account.
- Two starter birds at adoption, age-gated growth to a seven-bird cap.
- Six-species pool, procedural call grammar, personality drift, mood system, server-side tick.
- Multi-device sync via shared canonical server state (no client-side merge logic of any kind).
- Field notebook (sparse, naturalist, auto-generated, read-only).
- Presence accounting (the three-condition definition, verbatim).
- Visit-invitation feature: per-invite opt-in, read-only ambient visitor sessions.
- Screen-reader narration, reduced-motion mode, call captioning — built as first-class surfaces, shipped with v1, not staged afterward.
- Account export, soft-then-hard account deletion.

**Out of v1**, respected as load-bearing absences, not deferred TODOs: native apps, any gamification primitive (streaks, achievements, levels, visit-frequency surfaces of any kind), Tamagotchi mechanics (no death, no hunger, no distress, no decaying happiness meter, no negative drift), social-network surfaces beyond the single visit affordance (no profiles, follows, discovery, leaderboards, comments). These are treated as architectural constraints, not feature-flagged-off code paths — the data model below does not carry the fields a streak counter or a happiness-decay meter would need, on purpose, so that adding one later is a deliberate schema change someone has to notice in review, not a flag flip.

Two terminology collisions in the source PRD need disambiguating before any schema work starts:

1. **"Settle"** is overloaded — it names both the user-initiated evening-lighting gesture (`concepts.md`, `interactions.md`) and the time-of-day state most birds fall into at night (`aviary_layout.md`: "at full night, most birds are settled"). This plan keeps `aviary.lighting_state ∈ {active, settled}` for the user gesture and introduces a distinct bird-mood value, `roosting`, for the night-driven eyes-closed/low-perch behavior. They can co-occur (a roosting bird in an active-lighting aviary at 2am if the user is still watching) and they are driven by different inputs (user click vs. time-of-day), so collapsing them into one state machine would make the tick logic ambiguous about which input caused which transition. User-facing copy/narration is unaffected by this internal split — both still read as "settled" in prose if appropriate.
2. **"Offer" targeting.** The brief says offers come from a top-bar affordance, not from clicking a bird, but also describes "the receiving bird's" reaction in the singular. This plan resolves it as: an offer is aviary-scoped (not bird-targeted) at the point of user action, and every currently-present bird evaluates its own reaction independently against its own mood/curiosity — some approach, some watch, some ignore, per their own state. The per-bird cooldown in `interactions.md` only makes sense under this reading (a single shared cooldown would be aviary-wide and the spec is explicit it's per-bird); per-bird reactions also produce a richer scene than an arbitrarily-chosen single receiving bird would.

---

## 2. Architecture

**Service shape** (small set of focused services, not a monolith, not over-decomposed):

- **Auth service** — magic-link issuance/consumption, session tokens, revocation.
- **Simulation service** — owns the tick: personality drift, mood transitions, ambient state (weather, day/night), bird-to-bird effects, perch assignment. The only writer of personality vectors and mood.
- **Event ingestion API** — accepts client interaction events (presence heartbeats, listen-in start/end, offer, settle, undo-settle) into an append-only log. Stateless, no business logic beyond validation and cooldown rejection.
- **Snapshot API** — serves the current rendering-ready state to clients (and to visitor sessions, read-only).
- **Notebook service** — subscribes to tick completions, scores recent deltas against a sparsity budget, writes entries. Shares an "ObservationGenerator" phrase-grammar module with live narration (see §9) so the two surfaces never drift apart in voice.
- **Visit service** — invite issuance/revocation/expiry, visitor-token validation, visit log.
- **Account service** — settings, export, soft/hard deletion, email change.
- **Telemetry pipeline** — aggregate-only, physically separated from the simulation database (see §10 and the privacy note below).

**Client/server split.** The client never computes canonical state. It renders snapshots, interpolates between them, and writes interaction events. There is no client-side simulation of any kind, including offline — if the network is down, the aviary freezes on its last snapshot rather than guessing forward, because a guessed-forward client state would have to be reconciled against the real server state on reconnect, which is exactly the divergent-simulation failure `accounts_sync.md` rules out architecturally.

**Render pipeline boundary, and the personality-secrecy implication.** The PRD's "never exposed numerically" rule for personality traits is stated as an absolute, including "not in a debug view, not at any version, not in any tier" — strong enough that this plan treats it as a wire-format constraint, not just a UI omission. The snapshot payload sent to clients never contains the raw personality scalars (`boldness: 0.62`, etc.) under any field name. Instead, each tick, the simulation service derives a **presentation envelope** per bird — perch-zone weight, motion tempo, call density/pitch-range parameters, color-saturation render value, investigate-bias for offer reactions — and only the envelope is serialized. This means even a user inspecting network traffic in devtools doesn't find the underlying trait; it also means a future "debug panel" can't accidentally leak it, because the data simply isn't in the response to leak. (This plan does not attempt to defend against a determined reverse-engineer reconstructing approximate trait values from observed behavior over weeks — that's not what the PRD rule is protecting against, and chasing it would be security theater.)

**Data stores:**

- Primary OLTP store (Postgres) for accounts, birds, personality vectors, mood, notebook entries, invites, sessions.
- Append-only event table (Postgres at v1 scale, partitioned by account_id and time) with a per-account `processed_through` cursor consumed by the tick. Flagged explicitly as the first thing to migrate to a dedicated log (Kafka/Kinesis) if account volume makes single-database tick fan-out a bottleneck — not built that way at v1, because v1 scale doesn't need it and the extra infra would slow the team down on the bird engine, which is the load-bearing surface.
- CDN edge cache for the snapshot bundled with the initial HTML response (see §11, time-to-first-bird).
- Object storage for generated export JSON files, with expiring signed download links.

---

## 3. Data model

```
Account
  id (uuid, synthetic — never email)
  email_encrypted
  created_at
  status: active | pending_deletion
  pending_deletion_at, hard_delete_at
  settings: { notifications: { visit_alerts: bool=false }, accessibility: { reduced_motion: bool|null, captions: bool, narration: bool } }

Session
  id, account_id, device_label, created_at, last_seen_at, revoked_at

MagicLinkToken
  id, account_id, token_hash, expires_at (15 min), consumed_at

Aviary
  id, account_id (1:1), created_at (drives age-gated bird unlocks)
  lighting_state: active | settled
  lighting_state_changed_at
  weather_state: { kind: none|rain|wind, started_at, ends_at }

Species (static catalog, not per-account)
  id, silhouette_ref, default_palette, call_grammar_motif_library_ref

Bird
  id (stable, permanent — never reused/replaced)
  aviary_id, species_id
  name, created_at (adoption date)
  personality: { boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }  -- floats 0..1, server-only, never serialized to clients raw
  mood: { state: wary|content|curious|drowsy|alert|roosting, entered_at }
  perch_zone: front|middle|back
  perch_transition: { from, to, started_at, duration } | null
  last_offer_reaction_at  (per-bird offer cooldown)

InteractionEvent  (append-only)
  id, account_id, bird_id (nullable — aviary-scoped events), event_type, payload jsonb,
  client_seq, client_ts, server_received_ts, device_session_id

PresenceSegment  (derived/coalesced from presence heartbeats; see §6)
  id, account_id, device_session_id, started_at, last_heartbeat_at

NotebookEntry
  id, aviary_id, bird_ids[] (nullable), text, generated_at

Invite
  id, host_account_id, visitor_email_encrypted, token_hash,
  status: pending|active|revoked|expired, created_at, expires_at (30 days)

VisitSession
  id, invite_id, started_at, last_seen_at  -- duration for the visit log is last_seen_at - started_at

ExportRequest
  id, account_id, status, download_url, created_at, expires_at
```

Notes that matter:

- **Bird.id is permanent and never reused.** Renaming, re-syncing, or any species-pool migration must never substitute one bird record for another — this is the engine-level identity guarantee `bird_engine.md` calls the foundation of drift's perceived validity.
- **Drift is additive-only at the schema level too**: there is no `set_personality` write path anywhere, only `apply_delta`. This is enforced at the simulation-service code boundary (only the tick writes `Bird.personality`, and only via a delta function), not just by convention.
- No table anywhere stores "visit count," "days active," "streak," or any per-account behavioral counter intended for display back to the user. If a future feature needs one, that's a new and reviewable schema change, not a hidden column already sitting there.

---

## 4. API surface

All endpoints session-authenticated except `/visit/:token/*` (visitor-token-authenticated, read-only — there is no POST route reachable with a visit token at all, enforced at the routing layer, not just hidden client-side).

```
POST   /auth/magic-link              { email }
GET    /auth/consume?token=...       issues session, redirects
POST   /auth/sessions/:id/revoke

GET    /aviary/snapshot              ETag-aware; presentation envelopes only, no raw traits
POST   /events                       batched interaction events (see §6 for what's batched)

GET    /notebook?cursor=             paginated, reverse-chronological, no archiving

GET    /account
PATCH  /account                      settings (notifications, accessibility)
POST   /account/export
POST   /account/delete
POST   /account/delete/cancel
POST   /account/email-change         { new_email }  -- old email remains valid until new verifies

PATCH  /birds/:id                    { name }
GET    /aviary/offers                third-bird-and-beyond eligibility (age-gated)
POST   /aviary/offers/:id/accept

POST   /visits/invites               { email }
GET    /visits/invites                list + visit log, host-only
DELETE /visits/invites/:id           revoke

GET    /visit/:token/snapshot        visitor-mode snapshot; 410 with matter-of-fact payload once revoked/expired
```

Snapshot polling: immediate pull on load, on `visibilitychange` to visible, after a detected render-frame gap (suspend/resume), and a low-frequency keepalive (20–30s) while visible — matching `accounts_sync.md`'s "low-frequency keepalive" language without resorting to a push channel, since the slow ~60s tick means a sub-tick poll interval never meaningfully lags the canonical state. WebSockets are explicitly not used at v1: server-side state changes at most once a minute, so push infrastructure buys nothing here and costs bundle/ops budget the perf section won't spare.

---

## 5. Simulation engine design (the tick)

A scheduled worker processes accounts whose `next_tick_due` has passed (lease-claim a batch, process, advance `last_tick_at`). At v1 scale this is a single batch job on a ~60s cadence (PRD: "exact cadence calibrated during build" — 60s is the starting default; it's a single config constant, not load-bearing infrastructure, so retuning it post-launch is cheap). Flagged for sharded/leased workers once account volume requires it — not built that way yet, since premature horizontal-scaling infra would be wasted complexity at launch volume.

Per account, per tick:

1. **Pull unprocessed events** since the cursor, plus elapsed wall-clock time (ambient state must advance even with zero events).
2. **Compute presence-time delta** as the *union* of presence segments across *all* of the account's active device sessions for the tick window — not a sum. A user with the aviary open on a laptop and a phone simultaneously is one unit of attention, not two; summing would double-count and silently inflate that user's drift relative to the calibration target. This is the most likely subtle multi-device bug class and gets a dedicated integration test simulating overlapping multi-device presence.
3. **Apply drift deltas** per bird using a leaky-integrator/low-pass update: `trait_new = trait_old + k_signal * weight * (1 - trait_old)`. The `(1 - trait_old)` term gives natural saturation near the ceiling and guarantees monotonic non-decrease without an explicit clamp branch. `k_signal` is a single tunable constant per signal type (presence-time dominant, listen-in strong, offer-accept/offer-near small) — tuning the "measurable in ~1 week, visible in ~3 weeks" calibration target is then a config change, not a code change. There is no negative-weighted signal anywhere in this function; neglect literally cannot appear as an input with a sign, which is the structural enforcement of "drift never moves down."
4. **Transition mood** via a per-tick scoring function over candidate moods (recent-interaction inputs, local time-of-day from the account's last-reported IANA offset, ambient weather, the bird's own personality biasing certain moods). Picks the highest-scoring candidate but requires either a minimum score margin or a minimum dwell time before flipping, to avoid tick-to-tick flicker on borderline scores.
5. **Ambient/bird-to-bird pass**: weather roll (low per-tick probability of start/continue/end, tuned to "a few times a week"); day/night palette position as a continuous function of local time; chorus detection (≥2 high-vocal-frequency birds with compatible moods in-window → `chorus_active` flag, presentational only); wary-contagion (a bird entering wary nudges perch-adjacent birds' wary-likelihood for the *next tick only* — explicitly transient, never written to personality, so it can't be mistaken for drift).
6. **Perch-zone assignment** from mood+personality with hysteresis — zone changes are staged as a `perch_transition` (from/to/started_at/duration) so the client animates a flight rather than teleporting, and so a borderline bird doesn't flicker zones tick over tick.
7. **Write canonical state** in one transaction; advance the cursor.
8. **Hand off to the notebook service** (see §4/§9) for noteworthy-delta scoring.

**Return-greeting**, treated with the weight the PRD gives it: eligibility is computed when a snapshot request follows a presence gap above a minimum threshold (proposed: >2 minutes, to separate "stepped away for coffee" continuity from a real return). The server attaches a *greeting envelope* — greeter bird id (weighted random favoring higher boldness + non-drowsy mood), an absence-length tier (e.g., <10min / <1day / <1week / ≥1week) mapping to a greeting-intensity tier (glance / call / approach / call-and-response). The procedural variation *within* a tier — exact timing, pitch jitter, secondary-bird stagger offsets (200–900ms randomized, not simultaneous, per `interactions.md`'s explicit "no simultaneous chorus on cue") — is computed client-side per render so the same tier never plays identically twice.

**Offer reactions need a fast path.** A 60s tick is too slow for "watch the bird's reaction" to feel like a gesture. This plan splits offer handling into two layers: a **presentational reaction** (immediate, client-computed from the bird's last-known mood/personality-derived envelope, purely cosmetic — approach/investigate/ignore) and a **drift effect** (the actual curiosity/boldness nudge, applied authoritatively at the next tick from the logged event, same as everything else). This resolves cleanly because the PRD already says drift must never be visible within a single session — so the *fact* that the trait-level effect lands a tick late is invisible by design; only the cosmetic reaction needs to be instant, and that one is free to compute client-side. The same split applies to settle (lighting shift plays immediately client-side; the event is only flushed to the server event log after the 5-second undo window passes without a click — making "undo" simply "never send it") and to listen-in (the audio mix ramp is a fully local/client decision with no server round-trip; the event is logged in parallel purely for the drift signal).

---

## 6. Sync model

The server is the sole writer of personality and mood; clients only ever append interaction events, never absolute values (`accounts_sync.md`'s explicit example: a client sends "listened in to Pip for 3 minutes," never "set boldness to 0.62"). Multi-device sync is consequently not a protocol at all — it's a property of both devices reading the same snapshot endpoint. There is no client-to-client channel and nothing to merge.

Presence is sent as **heartbeats**, not a firehose: while all three presence conditions hold client-side (visible + focused + recent pointermove/keypress — the client enforces this locally before ever sending anything), it emits a heartbeat roughly every 20s carrying a per-device monotonic sequence number. The server coalesces consecutive heartbeats from the same `device_session_id` (gap < 2× heartbeat interval) into `PresenceSegment` rows at write time. The tick then unions segments across devices per account, as in §5 step 2. This keeps event volume bounded and keeps the presence definition exact without per-second network chatter.

Because drift writes are additive deltas computed from an ordered, append-only event log (never absolute values from a client), the last-write-wins failure case `accounts_sync.md` describes — a stale device's write clobbering a fresher device's drift — is structurally unreachable: there is no "write personality" call for a stale device to race against.

---

## 7. Frontend rendering pipeline

**Technology split.** The aviary scene itself renders on Canvas2D (kept outside any DOM-diffing framework, since it animates continuously at 60fps and a virtual-DOM layer would fight that). The top bar, settings, account flows, and notebook — all low-frequency-interaction, high-declarative-value surfaces — are a code-split, lazily-loaded React/Preact chunk layered as DOM above the canvas.

**Scene composition** (back to front): sky/gradient (day-night), background foliage parallax (subtle, per `aviary_layout.md`'s explicit "not parallax-heavy"), the three perch zones (front/middle/back) with fixed slot anchors, bird rigs, a foreground ambient-ornament layer (leaves, feathers, weather overlay — these are pure rendering ornaments per spec, not driven by the tick, generated client-side at idle cadence), and the DOM top bar on top.

**Bird rig**: a small layered 2D part-based rig (head/body/wing) per species rather than full skeletal animation or large animated-sprite sheets, kept light against the 2MB budget. Idle micro-motion (preen, scan, head-tilt, weight-shift) is continuous per-frame procedural offset driven by a slow noise function seeded by `(bird_id, mood)` — never a fixed loop, applying the same "never identical twice" principle the PRD states for calls to the visual layer too, since a looped idle animation is just as much a tell as a looped call.

**Snapshot interpolation**: the client keeps the last two snapshots and interpolates perch transitions, mood-driven pose blends, and idle-motion phase between polls; on a new snapshot arriving, the *currently rendered* state becomes the new interpolation start point (no popping/snapping to the freshly-fetched authoritative position).

**First paint and the "already in motion" conceit**: the initial HTML response is edge-rendered/edge-cached with the first snapshot inlined, so the very first frame the renderer produces places birds in their actual current positions, mid-action — no spinner asset exists in the bundle for this path, full stop. When a snapshot genuinely isn't available yet (slow network, cold cache), the fallback is a shared "quiet field" component (soft sky gradient, at most one faint ambient motion cue) — the same component used for the pre-first-bird empty-aviary state between adoption and the first bird's fly-in.

**Reduced-motion** is a mode flag (`motionMode: 'full' | 'reduced'`) threaded through the same animator and ambient-ornament systems, not a separate app or a stripped one: continuous procedural offsets are replaced by a discrete pose list cross-faded slowly on change; flight becomes a perch-to-perch crossfade instead of a path animation; ambient leaf/feather drift is removed; day/night and weather color shifts remain, slowed. Default-on from `prefers-reduced-motion`, overridable and persisted server-side in account settings (so the preference follows the user across devices, not just the browser that set the media query).

**Responsive layout**: perch anchors are viewport-relative with a minimum-spacing solver that compresses horizontal spacing on narrow viewports without ever pushing a bird outside the visible frame; resize recalculates anchors in place, reusing each bird's current animation phase rather than resetting it.

**Top bar fade**: a fade-after-N-seconds-of-stillness idle timer, restoring on pointermove/keydown — explicitly a *separate* timer from the presence-accounting activity window in §6; conflating the two would couple a UI cosmetic (chrome visibility) to the drift-bearing presence signal, which is the kind of accidental coupling that would be very hard to notice broke something.

---

## 8. Audio pipeline

Calls are synthesized client-side via WebAudio from a per-species call-grammar: a small motif library combined and varied at runtime (pitch jitter, timing jitter, motif sequencing), parametrized by personality (vocal_frequency → call rate/density; species → timbre/motif set) and mood (wary → sparser/shorter calls; content → fuller motif sequences). No audio files for calls, anywhere, at any fallback tier — this is treated as an unconditional constraint per `accessibility_perf.md`, not a default that a future "improve audio quality" ticket could quietly walk back.

**Node lifecycle**: a shared `AudioContext` with a pooled set of oscillator/gain nodes (or AudioWorklet processors instantiated once and reparametrized per call rather than recreated) — this is what satisfies the "no per-call allocation that isn't freed" / no-memory-growth performance requirement, and it's specified here as an engineering requirement, not a vague goal.

**Chorus mixing**: every concurrently-calling bird gets its own synthesis voice feeding a per-bird gain node into a shared bus — true concurrent synthesis, not pre-mixed loops, which is what avoids the phase-cancellation artifact the PRD calls out as audible even when each call sounds procedural in isolation.

**Listen-in mix**: a single `setListenTarget(birdId | null)` call drives all ramp logic in one place — target bird's gain ramps up over ~1–2s (`linearRampToValueAtTime` or exponential), all others ramp down to a reduced-but-nonzero ambient floor over the same timescale (never to 0 — per `interactions.md`, a re-balance, not a mute). All four disengage triggers (re-click, refocus elsewhere, click empty space, keyboard focus-away) funnel into the same call.

**Captions** are generated from the *same* call-grammar invocation parameters at the moment of synthesis — one shared deterministic templating layer (motif id + mood → short prose, e.g. `"rise-3note" + content → "a soft three-note rise"`) consumed by both the synth engine and the caption renderer, so a caption can never describe a call that wasn't actually the one played.

**Fallback**: if WebAudio is unavailable, synthesis no-ops and captions default ON — no recorded-audio path exists anywhere to fall back to, which is the unconditional rule stated in `accessibility_perf.md` ("silence with captions is a better fallback than canned audio"). AudioContext is created on first user gesture or first eligible visible+focused state (autoplay-policy compliant), and suspended/resumed in lockstep with the renderer's hidden-tab pause/resume.

---

## 9. Accessibility surfaces

**Narration** runs through a single ARIA live region with two priority lanes feeding the same queue rather than two separate competing regions — an ambient lane (one prose update per 30–60s) and an event lane (return-greeting, offer reaction, settle — promptly, but still phrased as observations, never as state-transition announcements). Both lanes and the field notebook draw from one shared **ObservationGenerator** phrase-grammar module — the server-side notebook writer and the client-side live-narration generator are two consumers of the same grammar/config, not two independently-authored copies, which is what guarantees a screen-reader user hears the same product voice moving between the live aviary and the notebook rather than two products glued together.

**Reduced-motion** and **captioning**: implementation detail covered in §7/§8; both are account-settings-persisted toggles, not local-only state, and both default from the relevant OS/browser signal (`prefers-reduced-motion`; WebAudio availability for captions).

**Contrast**: WCAG AA enforced on chrome surfaces (top bar, settings, captions, error/system surfaces) via design-token lint + visual-regression checks in CI; the aviary scene itself carries no user copy so the constraint doesn't apply there, per spec.

**Keyboard navigation**: top-bar items are natural DOM tab order. Birds are rendered on canvas, which is not natively focusable or AT-addressable, so each visible bird gets a thin, invisible, absolutely-positioned DOM overlay element (`role="button"`, accessible name sourced from the ObservationGenerator) repositioned each interpolation frame to track the canvas-rendered bird. Arrow keys move focus between bird overlays in a deterministic scene order (front-to-back, left-to-right); Enter triggers listen-in on the focused bird; Escape exits listen-in. Focus rings render on the DOM overlay so they stay visible regardless of what's underneath on canvas, with a treatment specified by the visual designer per the contrast requirement above.

---

## 10. Performance budgets and observability

- **<2MB gzip initial bundle**: enforced via a CI bundle-size gate (size-limit/bundlewatch) on the first-paint chunk specifically; account settings, accessibility settings, and the visit-invite flow are separate lazy chunks loaded on demand, not part of the gated bundle.
- **<500ms time-to-first-bird** on mid-tier mobile/4G: achieved by edge-caching the HTML with the first snapshot inlined (no second round-trip before the first paintable frame), `modulepreload`-prioritizing the renderer chunk, and deferring audio-engine init and the chrome bundle until after first paint. Verified continuously by a synthetic scripted-browser fleet from multiple geographies, alarming on regression — this is the metric the PRD explicitly calls the "affective-perf bridge," so it's monitored as a product metric, not just an infra one.
- **60fps idle motion / no 30-minute memory growth**: enforced by a CI performance harness running a scripted session against a throttled reference device profile, asserting frame time and heap growth; object-pooling discipline (renderer transforms, audio nodes) is a stated engineering requirement, not an aspiration, because this is explicitly called out as "a real test in CI, not a guideline."
- **Observability**: synthetic checks + aggregate-only RUM (load timing, first-bird-render timing, frame timing, audio-context error rate, simulation-tick latency, with a p99 alarm at 5s). Per the privacy boundary in `accounts_sync.md`, this pipeline is physically separated from the simulation database — the analytics warehouse never reads per-account interaction state, and no telemetry event carries per-bird or per-account dimensions. The negative space is deliberate and worth stating as plainly as the positive list: we explicitly do not build an "average drift across accounts" dashboard, a per-species engagement breakdown, or anything else that would require correlating individual accounts' behavior, even in aggregate, even with good intentions — the PRD calls this out by name as a temptation to refuse.

---

## 11. Rollout

- **Phase 0 (dogfood)**: small internal cohort, full engine and full accessibility surfaces from day one (not stripped — accessibility ships with v1, not as a v1.1 follow-up, per the PRD's explicit instruction that a late reduced-motion mode "quietly told reduced-motion users the product wasn't for them"). Goal: catch audio uncanniness and drift-feel issues that no automated check will surface.
- **Phase 1 (private beta)**: capped cohort, watching simulation-tick latency at real concurrency and — critically — instrumented (aggregate-only) drift-rate distribution against the calibration target (measurable at ~1 week, visible at ~3 weeks). This is a **go/no-go gate for public launch**, not a nice-to-have dashboard: shipping with miscalibrated drift means either a Tamagotchi-speed product or a screensaver-speed one, both of which are core-experience failures no amount of polish elsewhere fixes.
- **Phase 2 (public v1)**: two starter birds at signup; age-gated third-bird-and-beyond offers ship as designed for every account from day one (this is a product mechanic, not a rollout ramp — it isn't something the team turns up gradually); visit-invitations ship enabled (default-off per account, per spec, but the feature itself is live, not flagged off product-wide).
- **Instrumented from day one**: simulation-tick latency/error rate, presence heartbeat volume and coalescing correctness, aggregate drift-rate distribution (to validate calibration, never to inspect an individual account), notebook entry generation rate (to validate sparsity), audio-context init success rate, narration generation latency, CI bundle size, first-bird-render timing.
- **Safety valves**: the tick fleet can be paused without data loss or corruption — the aviary simply freezes (no events are lost; the durable event log replays cleanly once ticking resumes); the visit-invite creation path can be disabled independently of the rest of the product, in case the invite-email pipeline needs isolating during an incident, without taking down sign-in or the core aviary experience.

---

## 12. Risks

- **Drift calibration has no ground truth pre-launch.** The leaky-integrator design (§5) makes the calibration a single tunable constant per signal rather than a code change, specifically so Phase 1's go/no-go gate can retune it without a redeploy cycle.
- **Multi-device presence double-counting** is the most likely subtle correctness bug (§6) — mitigated by the union-not-sum design and a dedicated overlapping-multi-device integration test, called out explicitly so it doesn't get treated as an edge case nobody tests.
- **Audio uncanniness** is a product-feel risk no architecture document can fully de-risk — mitigated by starting real listening-session review in Phase 0, not waiting for Phase 1 user feedback to discover the calls sound off.
- **Personality-vector leakage** is a one-way door: the wire-format decision in §2 (presentation envelopes only, never raw traits) is what prevents this at the architecture level, but it has to survive every future PR touching bird/personality code — recommended as a standing review checklist item, since "just add a debug field for support" is exactly the kind of innocuous-looking change that would violate the rule.
- **Accessibility regression by neglect**: because reduced-motion and narration are first-class designed surfaces with their own ongoing design surface area (not a one-time fallback), they need to be in the same design review and QA checklist as every future feature touching the aviary, not a separate end-of-cycle accessibility pass — the latter is how "first-class" surfaces quietly become "checklist" surfaces over a few quarters.
- **Gamification creep is a process risk, not a technical one.** The data model deliberately has no streak/count/rank fields to repurpose, but the real defense is at design review: any future feature proposal that surfaces visit-frequency, achievement-like state, or comparative aviary data should be rejected per `non_goals.md`, and this plan recommends a standing checklist question on the team's feature-design template: *does this surface the user's own behavior back to them as a count, streak, or rank?*
- **Notebook sparsity is a tuning problem in both directions** — over-suppression reads as a dead notebook, under-suppression reads as a feed (which the PRD explicitly says would dilute the entries that matter). The scoring/suppression-window threshold (§5/§9) is instrumented and reviewed against real Phase 1 usage rather than guessed once and left alone.
- **Tick-worker scaling** is a single batch job at v1 scale by design (§2, §5) — flagged here so the sharded/leased-worker evolution is a planned next step the team revisits at a known trigger (account volume threshold), not an incident-driven scramble.
