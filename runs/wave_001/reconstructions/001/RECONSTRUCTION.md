## System-level intent

- Server-authoritative simulation and a single canonical aviary. This shows up in the opening assumption of a "server-authoritative simulation" and "a single canonical aviary per account"; in the client/server split where "canonical personality vectors" and "mood state transitions" belong to the server; in the render boundary where the server snapshot is the "only" source of pose, perch zone, mood, and simulation-time; and in sync where ticks consume a shared log into a "single personality truth."

- Anti-gamification and anti-punishment are invariant product boundaries. The non-goals reject "streaks, achievements, levels," "Tamagotchi mechanics," notifications, and social-network surfaces. The drift function repeats the same intent operationally: "Neglect -> delta = 0," traits are "frozen, not decay," and the observable effect is "not punishment visuals." The risks name "Gamification creep" as "Product identity erosion."

- The product voice is quiet, observational, and naturalist rather than attention-seeking. This appears in "sparse naturalist prose entries," a "quiet field" shell with "no spinner," a ship criterion that the aviary appears without "welcome toast," "Naturalist voice on product surfaces," and "matter-of-fact" system copy for failures, auth, sync, and revoked visits.

- Bird interiority should be expressive but not numerically exposed. The plan repeatedly says personality is "server-only," "never sent to client," and not exposed "numerically anywhere." Visual effects are allowed through "plumage_render_params derived from saturation trait without exposing number," while acceptance criteria require that "Personality numbers never appear in network payloads or UI."

- Aliveness comes from gradual, variable, procedural behavior. The plan uses "monotonic drift toward expressive," calibration around "weeks not sessions," mood that "persists across sessions," procedural call grammar, per-call jitter, return-greetings that "never repeat exact parameters," rare weather, and notebook triggers tied to ambient events.

- Presence should include quiet watching, but not hidden or inactive time. The presence window default is chosen to favor "watching without moving." Presence pings count only with `visible+focused+active` flags, and missing pings while hidden receive "no presence credit (by design)."

- Accessibility is part of the real aviary experience, not an afterthought. The v1 scope includes screen-reader narration, captions, reduced-motion mode, keyboard navigation, and WCAG AA. The risk table says "Accessibility as afterthought" would "exclude users from real experience," mitigated by shipping narration/reduced-motion "with core" and sharing sprint ownership with the renderer.

- Social features must stay narrow, opt-in, and read-only. The plan includes "opt-in visit invitations (off by default)," a "read-only ambient visitor view," revocable invites, no discovery endpoints, no default host notifications, and a risk that visit scope creep could create an "Accidental social network."

- Performance protects the first impression of a living, quiet scene. The budgets require "<2MB gzipped initial JS," "<500ms time-to-first-bird," and "60fps idle." The boot sequence prioritizes inline critical CSS, a minimal chunk, edge-cached snapshot fetch, and first bird paint before lazy-loading settings, notebook, visits, and export.

## Per-feature whys

### 1. Scope

- Single horizontal scene, three perch zones, no pan/zoom/scroll: NOT RECOVERABLE FROM PLAN

- Day/night from user local time: The plan ties this to `local_time_anchor`, account timezone, server UTC, and a continuous gradient driven by user timezone plus server `ambient.phase`, so the articulated why is canonical time-of-day behavior tied to the user's local time rather than local client guessing.

- Rare ambient weather: Weather is used as an ambient roll that affects call hints, mood edge weights, and notebook triggers; the plan says rain is "not a mood per se" but changes vocal dampening and edges toward `content`/`drowsy`.

- Two starter birds, age-gated growth up to seven, and bird ramp: The plan links age gates and calibration to the "weeks not sessions" promise, and the rollout monitors "audio chorus at 3-4 birds" before GA to protect recognizability at higher bird counts.

- About six species in the pool: NOT RECOVERABLE FROM PLAN

- Stable internal bird IDs: The data model makes bird IDs immutable, and the risk table says sync or personality loss "destroys trust in bird identity," so stable identity exists to preserve trust across sync and drift.

- User-assigned renameable bird names: NOT RECOVERABLE FROM PLAN

- Hidden five-trait personality vector: The plan keeps personality server-only, omits it from API responses, maps only derived rendering parameters, and bans numerical exposure because personality is private simulation state rather than a user-facing meter.

- Enumerated mood states: Mood is an explicit finite state machine with weighted edges, persists across sessions, selects animation sets, affects call hints, and provides public expressive state without exposing personality numbers.

- Monotonic drift toward expressive: The plan's why is no punishment: deltas are clamped with `max(0, computed_delta)`, neglect freezes traits rather than decreasing them, and observable changes are framed as fewer greetings or quieter chorus, "not punishment visuals."

- Return-greeting: The greeting varies by absence and bird, is selected by boldness and social_warmth, excludes sleeping birds, and must "never repeat exact parameters," making return feel personal without using a "welcome back" toast.

- Idle presence: The default activity window is chosen to favor "watching without moving"; presence-time is the dominant drift signal, while background or hidden tabs receive no credit by design.

- Listen-in: Listen-in affects that bird's social_warmth and vocal_frequency, uses smooth gain ramps, and keeps non-focused birds audible with a -9dB floor so focus does not erase the ambient aviary.

- Offers: Offers are routed through events and cooldowns; proximity and acceptance target curiosity and boldness, while reactions vary by mood and vocal_frequency, making offers influence drift without becoming a meter.

- Settle: Settle closes the presence window cleanly, creates a server-side settled state, shifts lighting and call gain, and allows undo within 5 seconds; tab close without settle is "not penalized."

- Field notebook entries: The plan makes entries sparse, read from stored server text, and constrained by a voice linter: lowercase, present tense, bird names, no user-behavior stats, and no personality numbers. The risk table says a chatty notebook would "dilute charm."

- Read-only field notebook: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: NOT RECOVERABLE FROM PLAN

- Synthetic UUID account IDs: The plan calls the UUID a "partition key everywhere," keeps email encrypted, and uses email hash lookup so account identity is not built around reversible email storage.

- Session tokens per device: The sessions model and auth routes support listing and revoking device sessions, so the articulated why is per-device session control.

- Multi-device sync via server snapshots: Clients share the same `aviary_id`, append to a shared log, and poll snapshot versions; this produces one canonical state across devices.

- Append-only interaction event log: The event log is the audit trail; events are "facts" such as listening duration rather than direct trait writes, and tick consumes ordered events from `simulation_cursor`.

- No client personality writes: The sync model rejects "No LWW on personality" because clients cannot post personality, preventing conflicts and preserving "single personality truth."

- Opt-in visit invitations off by default: The plan uses opt-in, off-by-default invitations to avoid notification and social-network creep; mitigations include no default host notifications and no discovery endpoints.

- Read-only ambient visitor view: Visitor tokens are scoped read-only to the host aviary, with no event endpoint on visitor routes, isolating visits from host simulation state.

- Visit log: NOT RECOVERABLE FROM PLAN

- Revocable invitations: Revocation is checked on each snapshot request, and acceptance criteria say revocation ends the session "matter-of-factly."

- Screen-reader narration: Narration is generated from snapshots with observational prose so screen-reader users receive the same quiet naturalist surface, not separate system instructions.

- Call captions: Captions are generated from motif metadata at play time, positioned near the calling bird, and enabled by default if audio fails, preventing silent failure.

- Reduced-motion mode: The plan honors `prefers-reduced-motion` and settings override, replaces skeletal animation with still-pose cross-fades, removes leaf/feather drift, and leaves audio, drift, and notebook unchanged.

- Keyboard navigation: The plan defines tab order, arrow-key bird focus, Enter listen-in, Escape exit listen-in, and a visible focus ring, supporting the launch gate for a keyboard-only path.

- WCAG AA chrome: Chrome, settings, and errors must meet WCAG AA so system surfaces remain readable across phases.

- Performance budgets: The budgets protect a fast, smooth aviary: first bird within 500ms, 60fps idle, controlled memory, small snapshots, and CI failure on bundle budget violations.

- JSON export on demand: The plan links export to account data including personality and uses export checksum in the "Sync / personality loss" mitigation, so the articulated why is trust and integrity around bird identity/state.

- Soft delete for 30 days then hard delete: NOT RECOVERABLE FROM PLAN

### 2. Architecture

- TypeScript monorepo with React, Canvas/WebGL, Fastify, PostgreSQL, and Redis: NOT RECOVERABLE FROM PLAN

- Server snapshot as render boundary: The snapshot is the only source of pose, perch, mood, and simulation-time, preventing the client from inferring personality or mood changes locally.

- Client render clock and interpolation: The render clock is independent of network so the client can interpolate positions and poses smoothly between snapshots.

- Client-only ornaments: Leaf/feather drift, listen-in gain ramps, settle overlays, and reduced-motion cross-fades are local ornaments so they can enrich the scene without becoming canonical simulation state.

- Stateless API service: NOT RECOVERABLE FROM PLAN

- CDN and edge snapshot cache: The boot sequence fetches an edge-cached snapshot in parallel with session restore to hit the time-to-first-bird target.

- Redis tick locks and jittered tick cadence: Redis `SETNX` prevents double tick, while jitter avoids a "thundering herd."

- PostgreSQL primary store: NOT RECOVERABLE FROM PLAN

- ETag and optional snapshot stream: ETags allow 304 when the snapshot is unchanged, while the optional SSE stream pushes new snapshot versions for visible tabs.

- Matter-of-fact error voice: Auth, sync, load, expired-link, and revoked-visit failures return matter-of-fact copy, matching the plan's split between naturalist product surfaces and system surfaces.

### 3. Data Model

- Encrypted email plus salted lookup hash: Email is stored encrypted, and lookup uses an HMAC-salted hash that is "not reversible."

- Personality stored only in `birds.personality` and account export: The plan keeps personality out of API responses while still preserving it as internal state and export data.

- Derived plumage render parameters: The snapshot builder maps `plumage_saturation` into color multipliers without revealing the scalar, allowing visible expressiveness without exposing a trait number.

- Event-sourcing without full-history personality recompute at runtime: NOT RECOVERABLE FROM PLAN

- Nightly drift checksum sampling: The nightly job verifies drift integrity through checksum sampling for ops only.

### 5. Simulation Engine Design

- Tick loop for active aviaries or lagged aviaries: NOT RECOVERABLE FROM PLAN

- Presence flags on pings: The tick counts presence only when pings include `visible+focused+active`, addressing the risk that a loose or tight definition could "corrupt drift signal."

- Seven-day low-pass drift function: The rolling window and weighted signals make presence-time dominant while listen-in, offers, and settle have smaller, targeted effects.

- Drift calibration harness: Simulated presence tests ensure measurable trait movement at 7 days and perceptible perch/greeting shift at 21 days, protecting the "weeks not sessions" promise.

- Mood finite state machine: Session-local offers and alarm calls, time-of-day, weather, and personality bias feed weighted mood edges so behavior changes are expressive but still server-governed.

- Perch selection with hysteresis: Perch zones are chosen from mood and boldness, with hysteresis "to avoid flicker."

- Call-grammar runtime: Server motif hints plus client WebAudio synthesis create procedural calls, while jitter and motif variation avoid exact repetition and recorded loops.

- Return-greeting variation memory: The server stores hashes of the last 10 greetings so greetings never repeat exact motif parameters.

- Bird-to-bird interaction: Overlapping call windows create chorus events that the notebook may note, and wary mood can spread to neighbors unless boldness is high.

- Notebook generator and voice linter: Rules-based NLG, sparsity governors, and a voice linter enforce lowercase naturalist prose with no user stats or personality numbers.

### 6. Sync Model

- Events as facts: Sync uses facts like "listened to Pip 180s" instead of setting warmth directly, preventing conflicts and keeping the server authoritative.

- Event idempotency: Client `event_id` UUIDs allow server dedupe so retried event writes do not double-count simulation input.

- Server timestamps authoritative: Client timestamps are advisory only, preventing clock skew from steering tick behavior.

- Resume after offline or suspended laptop: A full snapshot replaces local interpolator state, and hidden time receives no presence credit by design.

- Visit sync isolation: Visitor routes have separate rate limits, read-only snapshot access, no events endpoint, and revocation checked per request.

### 7. Frontend Rendering Pipeline

- Quiet field boot sequence: Inline critical CSS and a quiet field shell avoid a spinner while auth restore and snapshot fetch run in parallel.

- Lazy-loaded settings, notebook, visit flows, and export: Lazy loading keeps the minimal entry chunk small enough for fast first-bird paint.

- Canvas layer composition: NOT RECOVERABLE FROM PLAN

- Species SVG rigged layers: NOT RECOVERABLE FROM PLAN

- Idle micro-motion: Preen, scan, and fluff cycles are selected by mood, making visible motion reflect server mood.

- Perch transitions and reduced-motion cross-fades: Ease paths and fly-in paths avoid teleporting; reduced-motion uses cross-fades instead of animated movement.

- Settle lighting and audio overlay: Settle darkens/warms the scene and reduces call gain, making the settled state visible and audible.

- Top bar fade after pointer idle: NOT RECOVERABLE FROM PLAN

- Slow-network quiet field: The loading state remains a quiet field only, preserving the no-spinner product voice.

- Post-adoption soft fly-in: NOT RECOVERABLE FROM PLAN

### 8. Audio Pipeline

- Procedural WebAudio synthesis: Calls are synthesized from motifs instead of MP3s, supporting procedural variation, smaller assets, and the explicit ban on recorded fallback.

- Listen-in ducking and gain ramps: Gain ramps over 800ms prevent abrupt audio changes, and non-focused birds stay audible rather than being fully muted.

- Chorus jitter and compression: Server hints avoid unnatural sync, client jitter adds variation, and light compression keeps overlapping calls controlled.

- Song-fragment offer: The offer uses synthesized motifs, and birds may join, stay quiet, or counter-call based on mood and vocal_frequency.

- WebAudio fallback: If `AudioContext` fails, captions turn on by default and the only notice lives in a11y settings, avoiding silent failure and avoiding a toast on aviary load.

### 9. Accessibility Surfaces

- Screen-reader narration cadence and priority: Narration uses an off-screen polite live region, a 45-second idle cadence, a 15-second minimum gap, and priority for return-greeting, offer reaction, and settle so events remain observational rather than noisy.

- Captions near calling bird: Caption placement ties the textual call description to the bird producing the sound.

- Reduced-motion still-pose mode: Replacing skeletal animation with cross-faded still poses preserves the aviary state while removing motion.

- Keyboard controls and focus ring: The focus ring is visible on all phases, making bird focus and listen-in reachable without pointer input.

- Naturalist product copy and matter-of-fact system copy: The copy split keeps expressive surfaces observational and operational failures plain.

### 10. Performance Budgets and Observability

- Code splitting into `core` and `lazy`: Core includes scene, audio, and presence, while settings, notebook, visits, and export are lazy so first bird is not delayed.

- Aggregate-only RUM and server metrics: The plan measures performance and reliability while never emitting per-bird or per-account interaction fields to analytics.

- Simulation DB isolated from analytics pipeline: Isolation protects the privacy boundary around simulation state and interaction fields.

- Deliberately not measured engagement, streaks, leaderboards, or personality distributions: The plan refuses these metrics to preserve anti-gamification and privacy intent.

### 11-14. Rollout, Risks, Workstreams, Acceptance

- Bird ramp through dogfood, private beta, and GA: The staged ramp protects chorus quality and "per-bird recognizability" before enabling seven birds.

- Day-one instrumentation: Snapshot latency, tick lag, event accept rate, first-bird RUM, audio context success, and aggregate a11y adoption are measured to catch launch failures while staying aggregate-only.

- Launch gates: Seven green days of CI perf budgets, drift calibration, accessibility audit, and privacy review gate release on performance, calibration, accessibility, and telemetry partition.

- PR checklist and banned-string lint: The plan uses UI PR review, banned strings, and design gates to prevent "Welcome back" creep and gamification creep.

- Parallel team workstreams: NOT RECOVERABLE FROM PLAN
