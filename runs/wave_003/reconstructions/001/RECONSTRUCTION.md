## System-level intent

1. Build an ambient relationship product, not a game or care-taking loop. This shows up in Scope's "No gamification of any kind," "No Tamagotchi mechanics," and "No push notifications of any kind." The plan repeats the same intent in return-greeting ("The bird greeting is the entire welcome surface"), notebook generation ("never about the user's behavior"), top-bar rendering ("No hover tooltips, badges, or inline labels"), and risks ("A single toast, streak, or 'welcome back' re-frames the product").

2. Keep aviary state canonical, server-owned, and structurally conflict-free. This appears in the hard rules ("The server is the only writer of personality state"), Architecture ("The server is the only writer of all aviary state"), and Sync model ("One canonical record," "there is nothing to merge"). The plan frames this as making "the last-write-wins failure mode structurally unreachable."

3. Preserve personality as felt behavior rather than visible stats. The plan says "The user never sees personality vector values," the client receives only "renderable consequences," and raw floats are bucketed "so the vector stays genuinely hidden even from DevTools inspection." Frontend rendering carries the same principle: "Motion is the mood readout; there are no mood labels or icons." The risk table explains the product why: exposing values "collapses the relationship into stats."

4. Make drift expressive and non-punitive. Scope says drift is "monotonic-toward-expressive," and neglect produces "ambient quietness only." Drift design repeats that "deltas are never negative" and explains the two-layer model as honoring both "monotonic drift" and "quieter after absence." The intended feel is "quieter, not warier."

5. Enforce privacy through architecture, not trust. Hard rules require email to be "stored once, encrypted" and all other references to use "a synthetic UUID." Telemetry is "physically separate from the simulation store," and the boundary is "a build artifact, not a review checklist." Observability repeats the philosophy: some metrics are "not computed, so they can't leak."

6. Use a strict voice split. The hard rules define "naturalist (lowercase, present-tense, specific)" for the aviary, notebook, narration, captions, and offer prompts, and "matter-of-fact" for sign-in, account, sync errors, and accessibility settings. Notebook generation adds "no exclamations, no 'you'-sentences," and narration shares the same prose engine where possible.

7. Make the aviary feel alive through procedural variation, not canned loops. Call grammar uses motif libraries, fixed call signatures, variation seeds, chorus timing nudges, and response calls. Frontend rendering rejects "entry animation," "fade-from-static," and "spinner"; birds resume "mid-action." The audio risk says repeated-sounding calls, "phase-y chorus," or "canned feel breaks the entire product."

8. Treat accessibility as a first-version surface, not an afterthought. Scope includes screen-reader narration, reduced-motion mode, captions, keyboard navigation, and contrast. Accessibility surfaces say they "ship with v1, not after" and are "in the critical path." Reduced motion is "a parallel render path, not a stripped one."

9. Keep the product quiet at the edges: sparse social, sparse chrome, sparse notebook. Scope allows only a "read-only visit invite" and rejects profiles, follows, feeds, discovery, comments, and mutual visits. Notebook entries are "sparse naturalist entries" with a gate of "one entry per few days." The top bar is "sparse" and fades, with "No chrome inside the scene."

10. Make performance and observability part of the product contract. Scope lists concrete budgets; the performance table gives CI and RUM enforcement; rollout says the system is "Instrumented from day one." These budgets protect "time to first bird," idle motion, memory stability, and tick latency.

## Per-feature whys

### Scope

- Web-only product, modern browsers: The plan's rationale for no native apps is that "v1 is web-only" and "data model and protocols are not designed for native-client constraints."

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: The risk table ties magic-link constraints to avoiding "Magic-link abuse / replay" and "Account takeover via forwarded links."

- Per-device revocable sessions: The risk mitigation gives "per-device sessions, session list + revoke" as protection against magic-link abuse and replay.

- One canonical aviary per account: The Sync model rationale is "One canonical record" with "nothing to merge," avoiding aviary-state conflict UI.

- Two starter birds: NOT RECOVERABLE FROM PLAN

- Cap of 7 birds: The plan says the cap is "enforced in the engine, not just the UI," and risk mitigation frames architectural absence as a guard against social/gamification creep.

- Additional birds offered by aviary age: The plan specifies age "not activity, not payment," aligning the feature with the non-gamified, non-payment scope. Rollout repeats "unlock purely on aviary age."

- Server-side simulation tick as only writer of canonical state: The rationale is the single-writer guarantee: clients write events only, the server owns state, and sync has "no merge logic."

- Personality vector: The rationale is to drive expressive behavior while never surfacing stats; the vector is "SERVER ONLY" and client DTOs receive only bucketed consequences.

- Mood system: The plan uses mood to shape actions, calls, rendering, and return-greeting; specific product rationale beyond making behavior expressive is NOT RECOVERABLE FROM PLAN.

- Procedural call grammar: The plan's why is recognizability without repetition: the call signature makes a bird "always recognizably" itself, while variation seeds mean no two renders are identical.

- Bird-to-bird interaction: The plan ties this to calls scheduling response calls and alarm-call actions shifting nearby mood scores; a broader rationale is NOT RECOVERABLE FROM PLAN.

- Return-greeting: The rationale is to make the bird greeting "the entire welcome surface" with "No canned animations, no textual welcome, ever."

- Listen-in: The audio pipeline keeps the focused bird audible while other birds remain at "an ambient minimum -- never silent," preserving the aviary as a shared ambient scene.

- Offer (seed / song fragment / still pool, per-bird cooldown): NOT RECOVERABLE FROM PLAN

- Settle (with 5s undo): The plan says settle carries a gesture so the tick applies "the small mood-quieting signal" and the client renders "the lighting shift." The rationale for the 5s undo specifically is NOT RECOVERABLE FROM PLAN.

- Presence accounting: Presence pings are "the raw material" for presence-time, and the three-signal conjunction plus anti-inflation tests prevent silently accelerated drift.

- Field notebook: The rationale is sparse, naturalist observation rather than logs: detectors emit "candidate observation," the prose is "never about the user's behavior," and risk mitigation avoids entries degrading into event logs that "break the spell."

- Multi-device sync: The rationale is that all state is server-side, so there is "nothing to merge" and "last-write-wins" becomes structurally unreachable.

- Visit invitations: The rationale is to provide the single social feature while avoiding social-network surfaces; visitor tokens are read-only, write "no events," and generate "no drift."

- Visit log: NOT RECOVERABLE FROM PLAN

- Visit notifications default off: NOT RECOVERABLE FROM PLAN

- Account export: The plan says export is "the user's own data" and includes vectors because of "the PRD's explicit list."

- Account deletion: The plan ties deletion to complete removal: hard delete removes "birds, vectors, notebook, events, telemetry joins -- every record keyed by the account UUID."

### Architecture

- Edge/static tier: The embedded bootstrap renders the "quiet field" immediately and starts the first snapshot fetch, supporting first-paint speed.

- API service: NOT RECOVERABLE FROM PLAN

- Simulation service: The rationale is to consume events, advance canonical state on the minute cadence, and preserve "Single-writer per account."

- Client/server split: Server owns canonical state; client owns rendering, interpolation, audio synthesis, presence detection, and accessibility surfaces so the client "never computes drift" and "never mutates personality."

- Semantic render pipeline boundary: The rationale is hiding traits while giving the client enough "semantic state" to render pixels and audio.

- Bucketed renderable consequences: The plan states the why directly: "so the vector stays genuinely hidden even from DevTools inspection."

- Primary store: NOT RECOVERABLE FROM PLAN

- Event log: The plan chooses Postgres append-only at v1 scale and keeps an "abstraction boundary" so it can swap later "without touching the tick."

- Snapshot cache: NOT RECOVERABLE FROM PLAN

- Telemetry pipeline: The rationale is privacy by physical separation, different credentials, a schema allowlist, and no per-bird/per-account interaction fields.

### API surface

- REST-ish JSON over HTTPS: NOT RECOVERABLE FROM PLAN

- No WebSocket at v1: The plan says snapshots are "kilobytes" and the tick cadence is "a minute," so polling is "simpler and cheaper."

- Versioning under `/v1/`: NOT RECOVERABLE FROM PLAN

- `POST /v1/auth/magic-link`: The stated rationale is abuse prevention through a "15-minute, single-use link" and per-email rate limiting.

- `GET /v1/auth/consume`: The rationale is replay protection: validate, invalidate immediately, then issue per-device session cookie and CSRF token.

- Sign-out, session list, and session revoke: The risk table connects these to mitigation for magic-link abuse and replay.

- Email change verify-new-before-commit: The plan states the behavior but the rationale is NOT RECOVERABLE FROM PLAN.

- Snapshot endpoint: The rationale is low-cost state pull, "few KB" responses, delta/full behavior, and initial HTML embedding for "first-paint speed."

- Client snapshot pull triggers: The rationale is to refresh on visibility, render gaps, and low-frequency visible keepalive while interpolation "hides the pull cadence."

- No long-polling at v1: The rationale is that "the interpolation layer hides the pull cadence."

- Batched interaction events with idempotency: The plan's rationale is idempotent retry, ordered per-account sequence assignment, validation, and making absolute personality state unrepresentable.

- Notebook API: It is reverse-chronological, cursor-paginated, read-only, with older entries reachable; broader rationale beyond read-only sparse notebook is NOT RECOVERABLE FROM PLAN.

- Visit invite creation: The rationale is to support a revocable ambient session while ensuring the emailed token "grants no account."

- Visit invite revocation: The rationale is immediate host control; revoke is "effective at the visitor's next snapshot pull."

- Visit snapshot: The rationale is read-only access "minus anything identifying the host beyond the aviary itself" and explicit active/revoked/expired status.

- Visit log API: NOT RECOVERABLE FROM PLAN

- Export queue and emailed download link: The rationale is account export of the user's data to the verified address.

- Soft-delete and hard-delete: The rationale is a 30-day restore window followed by complete hard deletion keyed by the account UUID.

### Simulation engine design

- Tick cadence with jitter: The rationale is to avoid "thundering herd."

- Sharding by `account_id`: The rationale is that each account is advanced by "exactly one worker at a time," giving the single-writer guarantee "without a global lock."

- Ordered event consumption: The rationale is deterministic additive state advancement from events after `last_consumed_seq`.

- Presence folding: The rationale is to convert raw pings into counted presence-time and avoid counting gaps beyond the activity window.

- Mood transitions: Hysteresis is used "to prevent flicker," and mood persists so there is "no reset-to-neutral anywhere."

- Drift deltas: The rationale is monotonic drift through additive-only, clamped updates where deltas are "never negative."

- Drift low-pass filter: The rationale is that "no single session moves a trait visibly."

- Drift calibration targets: The rationale is testability against regular visitor and absence profiles, including "7 simulated days" and "21 simulated days."

- Neglect handling: The rationale is to make birds read as "quieter, not warier" while the stored vector "never moves down."

- Per-bird action/position state machines: NOT RECOVERABLE FROM PLAN

- Call scheduling: The rationale is to shape calls by vocal frequency, mood, weather, chorus emergence, and greeting logic; a deeper why is NOT RECOVERABLE FROM PLAN.

- New-bird offer from aviary age: The rationale is age-based growth rather than activity or payment.

- Mood scoring function: The plan uses time of day, personality, recent events, weather, and other birds so mood emerges from state; a broader rationale is NOT RECOVERABLE FROM PLAN.

- Call signature: The rationale is fixed per-bird recognizability: "Pip is always recognizably Pip."

- Variation seeds: The rationale is non-repetition while preserving signature; the same seed makes captions match audio exactly.

- Chorus: The rationale is that overlapping calls read as "a chorus" and are "never stacked identical loops."

- Bird-to-bird response calls and alarm influence: The plan states social_warmth weighting and wary mood shifts; broader rationale is NOT RECOVERABLE FROM PLAN.

- Noteworthiness detectors: The rationale is to select observations like first-greeter changes, quiet stretches, weather reactions, and habit changes rather than raw session logs.

- Notebook sparsity gate: The rationale is "at most one" selected observation, with hard minimum spacing and priority for rarer event classes.

- Notebook prose engine: The rationale is curated naturalist prose: lowercase, present-tense, bird-named, no exclamations, no "you"-sentences.

- Return-greeting selection: The rationale is one greeter, weighted by bird state and absence buckets, with staggered responses "never in unison."

- On-demand greeting computation: The plan says greetings need "sub-tick latency."

### Sync model

- One canonical record: The rationale is that clients are renderer plus event emitter and "there is nothing to merge."

- Additive deltas only: The rationale is that concurrent phone and laptop sessions both count and neither overwrites the other.

- Per-account ordering and idempotency: The rationale is strictly ordered tick consumption and deduped retries.

- Matter-of-fact conflict surfaces: The rationale is that real conflicts are limited to auth/session edges, while aviary-state conflict UI is unnecessary because state cannot conflict.

- Visitor isolation: The rationale is that visitors cannot reach the event endpoint; their presence creates "no events and no drift."

### Presence accounting

- Three-signal conjunction: The rationale is accurate attention: visible, focused, and recently active.

- Four-minute activity window: The plan says it is "biased long because motionless watching is the product."

- Presence ping every 30 seconds while present: The rationale is raw material for presence-time.

- Final ping on conjunction loss: The rationale is to end presence at the engine when any signal is lost.

- Hidden-tab battery behavior: The rationale is battery: rendering stops, RAF cancels, and audio suspends after fade.

- Anti-inflation tests: The rationale is to assert zero presence-time for backgrounded, unattended, and unfocused windows.

### Audio pipeline

- WebAudio synthesis: The rationale is procedural calls; the risk table says fallback is "silence+captions, never loops."

- Chorus mixing: The rationale is an ambient shared mix across birds.

- Listen-in mix: The rationale is focus without erasing the aviary: other birds floor at ambient minimum, "never silent."

- AudioContext-clock scheduler: The rationale is to follow snapshot call schedules with lookahead and controlled variation.

- Captions from motif and seed: The rationale is that caption "matches audio exactly."

- Graceful silence fallback: The rationale is no recorded-audio fallback path and captions default on if WebAudio is unavailable or denied.

- Audio memory allocation: The rationale is preventing growth; buffers are allocated once and checked by the 30-minute no-growth CI test.

### Frontend rendering pipeline

- Canvas-based scene renderer with WebGL and 2D fallback: The plan gives technical choices, but a specific product rationale is NOT RECOVERABLE FROM PLAN.

- Procedurally generated birds and shader-level plumage saturation: The rationale is to render personality-influenced, bucketed consequences without exposing raw vectors.

- Code-splitting account settings, accessibility settings, visit flow, and notebook UI: The rationale is supporting the initial JS budget.

- First paint with inline bootstrap and initial snapshot: The rationale is first-paint speed and placing birds at snapshot positions "mid-action."

- Quiet field loading state: The rationale is to avoid spinner/static loading; it shows soft sky color and faint motion cues when snapshot is delayed.

- Empty aviary fly-in: The plan states "never shown again after adoption"; deeper rationale is NOT RECOVERABLE FROM PLAN.

- Interpolation between snapshots: The rationale is that motion "never teleports."

- Idle micro-motion: The rationale is that "Motion is the mood readout" with no mood labels or icons.

- Ambient ornaments: The rationale is subtle client-side atmosphere with "Zero server state."

- Day/night: The rationale is gradual local palette change; at night most birds settle while nightjar-like species stays active.

- Weather: The rationale is rare, short-lived palette/audio effects; broader rationale is NOT RECOVERABLE FROM PLAN.

- Top bar: The rationale is sparse chrome: it fades, returns on movement or keyboard, and keeps "No chrome inside the scene."

- Responsive layout: The rationale is that "all birds always in frame" and narrow viewports "never crop."

- Reduced-motion mode: The rationale is a designed equivalent surface: "a parallel render path, not a stripped one," preserving audio, drift, notebook, and mood.

### Accessibility surfaces

- Screen-reader narration: The rationale is to read the same snapshot state as the renderer in naturalist prose without flooding; user events get priority but remain observations.

- Shared narration/notebook prose engine: The rationale is keeping the "Same voice as the notebook."

- Keyboard navigation: The rationale is full reachability of top bar, scene, birds, listen-in, offer palette, settle, and dialogs by keyboard.

- Focus indicator: The rationale is legibility against "bright and dim aviary states."

- WCAG AA contrast: The rationale is readable chrome/copy, including night palette, checked in CI.

- Audio-off and captions: The rationale is fallback access to calls when audio is unavailable, denied, or disabled.

- Ship gate: The rationale is that accessibility ships "with v1, not after" and is in the critical path.

### Performance budgets & observability

- Initial JS bundle budget: The rationale is enforced first-paint performance through a hard CI bundle-size check.

- Time to first bird budget: The rationale is fast first visible bird on mid-tier mobile and 4G, monitored synthetically and with RUM.

- Idle motion budget: The rationale is 60fps idle on older laptops.

- Memory budget: The rationale is no growth over a 30-minute session, enforced by CI heap-delta tests.

- Tick latency budget: The rationale is server health with p99 alarm threshold.

- Aggregate RUM and server metrics: The rationale is performance and reliability measurement without per-bird or per-account interaction data.

- Deliberately unmeasured per-bird/per-account interaction analytics: The rationale is privacy: no account dimension, no simulation-store reads, no average drift dashboards, and fields that could leak are "not computed."

- Unsupported browser surface: The rationale is "matter-of-fact" communication and "no compatibility shims."

### Rollout

- Phase 0 foundations: The rationale is to land the privacy boundary "before any interaction data exists."

- Phase 1 living aviary: The rationale is internal dogfood of vector, mood, drift, procedural motion/audio, day/night, and greeting behind a flag.

- Phase 2 relationship: The rationale is locking drift calibration against "7-day/21-day targets" using fast-forward simulation.

- Phase 3 access and polish: The rationale is to bring narration, reduced motion, captions, keyboard nav, contrast, and performance hardening into the release path.

- Phase 4 quiet edges: The rationale is to complete visits, export, deletion, sync conflict surfaces, and unsupported-browser handling after core relationship and access work.

- Invite-only beta: The rationale is to re-check drift calibration against real instrument readings before general availability while remaining "aggregate-free, per-account-internal-only."

- Birds-per-aviary ramp: The rationale is age-based pacing, server-configurable thresholds, and engine-enforced cap.

- Drift-calibration dashboard over synthetic test accounts: The rationale is visibility into calibration drift "without touching user data."

### Risks

- Drift calibration mitigations: The rationale is avoiding "Tamagotchi feel" if too fast and "screensaver feel" if too slow.

- Presence inflation mitigations: The rationale is preventing lax detection from silently accelerating drift population-wide.

- Sync correctness mitigations: The rationale is avoiding client-side personality writes that "silently deletes drift."

- Audio uncanniness mitigations: The rationale is that repeated, phase-y, or canned calls "breaks the entire product."

- Accessibility regression mitigations: The rationale is preventing flooded SR queues, stripped reduced-motion, or accessibility being deferred.

- Personality-vector persistence and hiding: The rationale is that losing a vector "deletes a bird the user knows" and exposing values "collapses the relationship into stats."

- Voice drift copy rules: The rationale is that one toast, streak, or welcome-back surface "re-frames the product."

- Notebook voice safeguards: The rationale is avoiding entries that become event logs and "break the spell."

- Tick cost mitigations: The rationale is keeping per-minute ticks affordable through jittered sharding, tiny per-account work, stateless workers, and cache.

- Magic-link abuse mitigations: The rationale is preventing account takeover via forwarded links.

- Scope-creep safeguards: The rationale is resisting the "just one harmless counter" pitch by making reappearance "a build, not a toggle."

### Appendix A

- Tick cadence default: The plan marks it "calibrate"; further rationale is NOT RECOVERABLE FROM PLAN.

- Presence activity window default: The plan says "calibrate long"; the presence section says motionless watching is the product.

- Mood enum final set: NOT RECOVERABLE FROM PLAN

- Event log default: The rationale is a "swap-ready boundary."

- Scene renderer default: NOT RECOVERABLE FROM PLAN

- Visit token model default: The rationale is revocable short-lived read-only sessions.

- New-bird age thresholds default: The rationale is config-driven age pacing.
