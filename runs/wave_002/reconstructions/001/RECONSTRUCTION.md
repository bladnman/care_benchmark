## System-level intent

1. **Treat warnings as architectural constraints, not review vigilance.** This appears in the scope risk note: "The biggest in-scope risk is not a missing feature, it's an over-built one," and the plan says gamification, streaks, and Tamagotchi warnings are "architectural constraints" because "review vigilance is exactly the protection that erodes over a multi-year product lifetime." The same philosophy returns in anti-gamification schema friction, visit read-only double enforcement, and merge gates for accessibility.

2. **The client is a rendering and input surface; the server owns state.** The architecture names this as the direct implementation of "the client never owns state." The client pulls snapshots, renders by interpolation, sends events, and "never computes personality, mood, or drift." The simulation service is "the only writer of personality and mood."

3. **Expose behavior, not numbers.** The snapshot section says raw personality numbers are never serialized; boldness, social warmth, vocal frequency, and curiosity influence server-computed outputs instead. The plan calls this the literal implementation of "personality is never exposed numerically." The same principle applies to notebook generation: structured facts carry "never raw personality numbers."

4. **Avoid engagement framing by removing footholds.** `notebook_unread_marker` is boolean existence, "not a count"; no schema stores visit counts, streaks, days active, or frequency aggregates. The plan says "the moment a count exists in the schema, a future PR can expose it cheaply," so it makes future frequency concepts require new schema.

5. **Drift is positive, slow, and relationship-shaped.** The drift function enforces "No negative term exists," no decay, no ignored-minutes input, and no subtraction. `max_step_per_tick` prevents "single-session visible movement," with instrument-visible drift after about a week and user-visible drift after about three weeks.

6. **Use additive event folding instead of last-write-wins.** Sync correctness is grounded in append-only events, a single canonical writer, and additive deltas. The plan says overlapping laptop and phone events "both get folded into the same tick's delta computation" because inputs are additive, not clobbering fields.

7. **Make the product feel alive through procedural variation and persistence.** Species-flavored seed ranges create "immediate, legible" personality differences; mood does not reset between sessions; calls are "procedural, not recorded"; the scene "boots directly into motion"; greetings are animation and call only, never a banner.

8. **Keep the voice naturalist, sparse, and auditable.** Notebook generation uses constrained phrase grammar because "a stock event-log treatment would break the spell," while the constrained generator is "auditable and won't drift off-voice." Notebook entries are rare so they do not "turn the notebook into a feed."

9. **Accessibility is a first-class designed surface.** Reduced motion is "a parallel rendering mode," "not a flag," and ships in the same release. Screen-reader narration uses the same naturalist voice and a calibrated cadence to avoid "overwhelming the screen-reader queue." Rollout says accessibility is "not deferred."

10. **Privacy boundaries are infrastructure boundaries.** Email appears only in encrypted account fields; foreign keys use synthetic UUIDs; telemetry has "zero read access" to simulation data through separate infrastructure. Aggregate telemetry is emitted pre-aggregated, with "no per-account dimension."

11. **Optional social must remain read-only and non-influential.** Visit invitations are read-only, opt-in, revocable, and no co-presence. Visitor events are "structurally invisible to the tick's drift math," and the plan names "let visitors do one small thing" as a risk because it contradicts the read-only design.

12. **Performance budgets are product requirements, not afterthoughts.** Time-to-first-bird, bundle size, idle FPS, and memory growth are enforced by CI gates, synthetic monitoring, and explicit architecture choices such as edge snapshots, lazy-loaded motif libraries, object pooling, and one bounded AudioContext.

13. **Calibrate before users build relationships.** The rollout and risks section say drift weights and thresholds need real session-shaped data in alpha, before external users, because changing weights later would alter the felt pace of drift for accounts "already mid-relationship with their birds."

## Per-feature whys

### 1. Scope

- **Single-user accounts, magic-link sign-in, one canonical aviary per account, multi-device sync:** NOT RECOVERABLE FROM PLAN
- **Two starter birds at adoption, cap of seven, species-pool-driven new-bird offers tied to aviary age:** The cap and age tie are justified later as avoiding visit frequency and interaction score; age is "explicitly not a frequency metric." The plan does not articulate a separate why for exactly two starter birds or exactly seven beyond scope and later cadence.
- **Bird engine: personality vector, mood, procedural calls, drift, idle motion, bird-to-bird interaction:** The plan's rationales are spread through the simulation sections: personality differences should be "immediate, legible"; drift is monotonic toward expressive; procedural calls avoid fixed samples and canned-feeling simultaneity; idle motion keeps mood legible without labels; bird-to-bird interaction creates call-response, wary spread, and chorus emergence.
- **Interactions: return-greeting, listen-in, offer, settle, field notebook:** Return-greeting is meant to be exclusively animation and call, never a toast/banner. Listen-in is instant client-side for responsiveness and uses a slow ramp instead of a hard cut. Offers feed curiosity/boldness without becoming score. Settle closes presence cleanly without pushing drift. Field notebook protects the naturalist voice and avoids stock event-log treatment.
- **Presence accounting as drift engine primary input:** Presence-time is dominant because the plan follows the PRD's weight order and defines presence as three simultaneous conditions; it also avoids using presence to surface frequency/count metrics.
- **Single horizontal aviary scene, day/night cycle, ambient weather, top-bar chrome, responsive layout:** The plan gives rationale for responsive layout: keep all birds in frame across aspect ratios, never cropping or scaling off-canvas. Day/night is anchored to local time. Ambient weather creates short-lived mood dampening/alerting. Top-bar chrome: NOT RECOVERABLE FROM PLAN.
- **Visit-invitation:** The rationale is a limited social surface: read-only, opt-in, revocable, no co-presence, and visitor presence must never drift host birds.
- **Screen-reader narration, reduced-motion mode, call captioning, keyboard navigation, WCAG AA:** The rationale is first-class accessibility: reduced motion must not land as a "v1.1 fix"; narration must avoid queue-overwhelm; captions should match what was actually synthesized; keyboard navigation is part of the core interaction layer; automated contrast checks prevent palette regressions.
- **Performance budgets:** The plan ties each budget to specific design choices and continuous enforcement: edge-delivered snapshots for first bird, CI bundle gates, low-end-device profiling, heap snapshot tests, synthetic monitoring.
- **Account export, soft-then-hard deletion, aggregate-only telemetry:** Hard delete is real DELETE to honor "every record tied to the account, gone." Aggregate-only telemetry keeps analytics air-gapped from simulation data. Account export: NOT RECOVERABLE FROM PLAN.

### 2. Architecture

- **Edge/BFF:** It serves initial HTML plus state snapshot from CDN edge because this is "critical for the <500ms time-to-first-bird budget."
- **Account service:** Owns email because it is "the only table that stores email (encrypted at rest)" and handles account lifecycle, sessions, export, deletion, and visit invitation lifecycle.
- **Simulation service:** Owns personality, mood, species pool, and bird identity because it is the only writer of personality and mood.
- **Event log:** Append-only event storage exists so the simulation tick can consume interaction and presence events to mutate state while nothing else reads it for product purposes.
- **Notebook service:** It consumes tick outputs and curated events to generate immutable field-notebook entries, keeping notebook generation separate from simulation writes.
- **Telemetry/observability pipeline:** It is non-product and air-gapped to preserve the privacy boundary.
- **Client/server split:** The client never computes state because "the client never owns state"; it renders snapshots and sends events.
- **Render pipeline boundary:** Pure function rendering keeps first paint unblocked and keeps rendering from consulting event logs or personality vectors directly.
- **Snapshot shape:** The snapshot is small for perf and hides raw personality, exposing only render-relevant derived state and a boolean notebook marker to prevent count/badge framing.

### 3. Data model

- **Synthetic UUID keys and encrypted email:** This implements the "synthetic ID rule" and makes email never appear outside encrypted account storage; the plan uses a lint rule because a code-level rule survives team turnover better than a doc convention.
- **Sessions:** Device labels are user-set or derived and never email, supporting multi-device sessions without leaking identity into other tables.
- **Magic links:** Raw tokens are never stored, only `token_hash`, to protect sign-in links. Expiration is set to 15 minutes. Other rationale: NOT RECOVERABLE FROM PLAN.
- **One aviary per account:** Enforced by uniqueness constraint; rationale beyond scope: NOT RECOVERABLE FROM PLAN.
- **Aviary timezone:** Stored for day/night anchoring to local time.
- **Stable bird identity:** `bird_id` is stable, never reused, never regenerated, matching "Bird identity."
- **Species pool:** Static v1 reference table gives silhouette, palette, and motif-library references; rationale beyond supporting species-flavored rendering/audio: NOT RECOVERABLE FROM PLAN.
- **Personality vector ranges and seeding:** Normalized traits and species-flavored seed ranges make starter birds immediately legible and avoid identical zero-vectors that only diverge after slow drift.
- **Presence and interaction events:** Append-only events are needed for drift and notebook generation; partitioning by account supports the privacy boundary and retention/compaction.
- **Presence ping payload:** Batched windows avoid event-log flooding.
- **Listen-in payloads:** Start/end are paired for duration computation.
- **Offer payloads:** `target_bird_id` can be null because some offers are aviary-wide rather than bird-targeted.
- **Settle payload:** Empty because the event itself is enough and settle implicitly closes presence.
- **Visit-session events:** Recorded only for the host's visit log and never consumed by drift because visitor presence must not drift host birds.
- **Anti-gamification data-model constraint:** No visit count, streak, days active, or frequency aggregate exists because "the moment a count exists in the schema, a future PR can expose it cheaply."
- **Privacy/telemetry boundary:** Separate infrastructure prevents "just one more dimension" analytics queries against primary simulation data.
- **Account deletion:** Soft deletion allows restore; daily hard deletion cascades with real DELETE to honor "every record tied to the account, gone."
- **Field notebook entries:** Immutable read-only entries support the read-only notebook; provenance exists for debugging, not user display.
- **Visit invitations and visit sessions:** The invitation rows support revocable/expiring access, and sessions back the host's visit log without feeding drift.

### 4. Simulation engine design

- **Per-aviary tick architecture:** A 60-second sharded scheduler keeps p99 5s tick-latency alarms meaningful at scale and avoids tick latency scaling with total account count.
- **Single-writer model per aviary:** Only the tick mutates birds so no-last-write-wins holds without distributed locking beyond one owner per aviary tick.
- **Drift function:** It is monotonic by construction, with no negative term, and uses max step ceilings so a single session is invisible while week/three-week targets accumulate.
- **Presence/listen-in/offers/settle weight order:** Matches the stated weight order: presence dominant, listen-in second, offers third, settle non-directional.
- **Mood model:** Five states and weighted transitions make mood respond to recent interactions, time of day, weather, and personality bias; persisted mood prevents reset between sessions.
- **Idle motion mapping:** Server-owned `idle_state` lets mood be readable through behavior and retunable without a client release.
- **Call grammar runtime:** Client-side procedural synthesis avoids fixed samples, phase-canceling stacked loops, and canned simultaneity while fixed signature elements preserve recognizability.
- **New-bird offer cadence:** Age-gated daily evaluation keeps the mechanic verifiably untied to visit frequency or interaction score and frames arrival as "notice, never announce."
- **Bird-to-bird interaction:** Call-response, wary spread, and chorus emergence make group behavior arise from proximity, social warmth, vocal frequency, ambient events, and independent synthesis.
- **Ambient weather:** Low-frequency weather creates temporary mood effects a few times a week without making weather part of per-minute drift state.

### 5. Sync model

- **Single canonical writer:** Database grants enforce that only tick workers update bird state, preserving server-owned state and preventing client direct writes.
- **Snapshot propagation:** All devices read the same `GET /aviary/snapshot` so there is no divergent per-device snapshot variant.
- **No last-write-wins; additive deltas only:** Events from overlapping devices fold into the same tick because inputs are additive rather than absolute values.
- **Client snapshot refresh triggers:** Refetch on visibility, resume, and low-frequency keepalive makes second-device changes visible near tick cadence without harming perf or battery; the client never extrapolates mood or personality.

### 6. Field notebook / narrative generation pipeline

- **Scheduled observation:** Low-frequency randomized entries avoid a fixed cadence the user could learn.
- **Notable event:** Structurally interesting events can become entries, but rate limiting keeps them from turning the notebook into a feed.
- **Template-grammar generation:** Chosen because voice is exposed, stock event-log treatment would break the spell, and a constrained generator is auditable, off-voice-resistant, cheaper, and faster than hot-path LLM calls.
- **Writer/engineer review:** Ensures phrase grammar has a voice guardrail.
- **Immutable entries and paginated access:** Supports a read-only notebook with no edit/delete/annotate endpoints.

### 7. Presence engine

- **Three-condition presence detection:** Visibility, focus, and recent activity define presence; the 5-minute activity window leans long because "watching birds without moving is the actual product."
- **Presence windows and batching:** Open/close semantics capture true presence while periodic flushing bounds event volume.
- **Settle and tab-close symmetry:** Same code path prevents any "penalize ungraceful exit" branch.
- **Presence consumers:** Presence drives drift and indirectly notebook detection, but never any user-facing days-visited or frequency metric.

### 8. Frontend rendering pipeline

- **Layered scene composition:** Background, middle, and foreground planes implement quiet foreground/background separation with subtle parallax.
- **Scene boots directly into motion:** Avoids neutral/idle-zero poses and loading spinners; slow fetch fallback is a quiet-field placeholder.
- **Idle micro-motion and mood-shaped rendering:** Mood is not label/icon/color; it is readable through motion and perch behavior because needing to tell the user a bird's feeling would mean the product failed.
- **Return-greeting:** Uses server-selected greeting directive so the client never needs raw traits; amplitude scales with absence; stagger avoids unison; no banner/toast/modal keeps greeting as bird animation and call.
- **Listen-in mix and visual focus:** Client-local ramping gives instant response and slow rise/fall while server events preserve drift accounting.
- **Reduced-motion rendering mode:** Separate designed renderer avoids merely disabling animation and ships in v1 with the primary renderer.
- **Responsive layout:** Recomputes perch positions so all birds stay in frame on phone portrait through wide desktop.

### 9. Audio pipeline

- **Asset strategy:** Parameterized waveform definitions and lazy-loaded adopted-species motif libraries keep the bundle small while preserving per-call variation.
- **Synthesis and mixing:** One AudioContext, per-bird gain nodes, and short-lived oscillator/noise nodes allow genuine overlap and prevent memory growth.
- **Listen-in mix decay:** Gain automation creates the required rise/fall feel rather than stepped channel switching.
- **WebAudio fallback:** Silent path with captions forced on avoids shipping an MP3 fallback and keeps captions available when audio is unavailable.

### 10. Visit-invitation flow

- **Invite lifecycle:** Ongoing active invites are revocable/expiring, not one-shot, because the plan reads the access as ongoing until revoked or expired.
- **Read-only enforcement:** Separate read-only API, no visitor event-ingest client code, and no server write grants make visitor non-interaction structural instead of trusting hidden buttons.
- **Visit-session events:** Recorded for host log only and excluded from drift to ensure visitor presence never affects host birds.
- **Visit log:** On-demand settings page shows visitor email, date, and approximate duration, but no badges, unread markers, or counts outside that page.
- **Revocation and expiration:** Checking status on every snapshot request makes revocation take effect on the next poll; daily expiration handles stale pending invites.
- **Visit notifications:** Default false and no onboarding prompt keep notifications from becoming an engagement surface.

### 11. Accessibility surfaces

- **Screen-reader narration generation:** Same naturalist inputs and cadence preserve voice while idle cadence and polite/assertive split avoid overwhelming the queue.
- **Reduced-motion mode:** It is first-class and ships with v1.
- **Call captioning:** Captions derive from the actual motif and parameter choices so they match what was synthesized, not a fixed call type.
- **Keyboard navigation:** Built into the core interaction layer; focus order follows perch structure and fixed-contrast rings remain visible across day/night.
- **Contrast:** Automated CI checks against tokens keep future palette tweaks from silently regressing WCAG AA.

### 12. Performance budgets and observability

- **Bundle budget:** Initial-route CI gate and lazy/code-split assets keep initial JS under budget.
- **Time-to-first-bird:** Edge-delivered initial snapshot, immediate first-bird render, deferred non-critical assets, and bundle budget combine to hit the 500ms target.
- **60fps idle motion:** Fixed frame cost, low-end CI profiling, rate-limited ornaments, and object pooling protect idle performance.
- **No memory growth:** Dispose per-call audio nodes, virtualize notebook entries, and avoid recreating workers/AudioContexts; heap snapshot CI enforces this.
- **Code-splitting:** Keeps secondary surfaces and unadopted species out of the initial bundle while preserving core aviary startup.
- **Observability:** Synthetic monitoring and aggregate RUM validate performance and tick latency without per-account dimensions or simulation database reads.
- **Browser support:** Last two major browser versions are tested; unsupported browsers get a matter-of-fact page rather than degraded rendering.

### 13. Rollout

- **Internal alpha:** Needed to calibrate drift weights and mood thresholds against real usage patterns because calibration needs "real session-shaped data."
- **Closed beta:** Full account/auth/sync, starter birds, core interactions, notebook, and accessibility all present because accessibility is not deferred; visit invitations are limited initially to validate read-only enforcement under cross-account traffic.
- **V1 launch:** Full feature set launches only after CI performance gates and beta synthetic-monitoring baselines.
- **Birds-per-aviary ramp:** The age-gated cadence is the ramp lever; age thresholds are post-launch tunable config so they can be calibrated without deploy.
- **Day-one instrumentation:** Tick latency, render timing, audio errors, and drift-calibration instrumentation must be live from the first cohort because multi-week calibration can only be checked against real histories.

### 14. Risks

- **Drift calibration risk:** Highest risk because assumed regular visits may not match reality; weights and max step are runtime config so alpha can recalibrate before external users build histories.
- **Sync correctness risk:** Direct-write debug/support endpoints could bypass the tick, so no direct-write path exists; support tooling must use synthetic events.
- **Audio uncanniness risk:** Procedural synthesis may sound worse than recorded loops, so alpha includes tuning time and fixed signature elements preserve recognizability.
- **Accessibility regression risk:** Parallel reduced-motion and narration surfaces can drift stale, so any visible aviary-scene behavior needs corresponding treatments as a hard merge gate.
- **Gamification creep risk:** Because the PRD calls it "the foothold," future count/streak/frequency proposals need explicit sign-off beyond normal review.
- **Visit read-only boundary risk:** Future visitor interactivity, even a "wave," must be rejected or fully reviewed because it contradicts "visitor cannot trigger anything."
