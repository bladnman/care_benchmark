## System-level intent

1. The aviary should feel already alive, not launched by the user. This appears in the non-negotiable experience that "the aviary appears to have been continuing before the user arrived," in the first screen being "the aviary, not a landing page or dashboard," in birds starting "mid-action," in "no wake-up animation or app-like fade-in," and in the first-load risk that a "spinner or slow first bird breaks the 'already alive' conceit."

2. The product is built around honest presence and small gestures, not maintenance. The plan says birds should respond to "honest presence and small gestures without becoming chores, counters, or pets to maintain." That intent shows up in the exclusions of "hunger, distress, death," "scores, streaks," and "visible personality stats"; in "offers are gestures, not feeding mechanics"; and in the rule that there is "no negative drift from absence or neglect."

3. Canonical bird identity and behavior belong on the server. The product boundary says "all state that matters to bird identity, mood, and long-term drift is canonical on the server." The architecture invariant repeats that the client may render and submit events, but "may not decide personality drift, persist canonical mood, overwrite bird state, or infer state from local history after reconnect." The simulation worker is "the only writer of personality vectors and canonical bird state."

4. Change should be slow, subtle, and non-punitive. Personality drift is a "low-pass filter over weekly-scale signal aggregates, not immediate event clicks." Regular visits produce "measurable numerical movement after about one week and visible/noticeable change after about three weeks." Absence should make quieter behavior emerge from mood and lack of recent positive signals, "not trait punishment."

5. Product voice is naturalist; system voice is matter-of-fact. The plan separates "naturalist" product surfaces from "matter-of-fact" account/auth/error/accessibility settings. Notebook entries and narration are "lowercase, present tense, specific," while errors use "matter-of-fact language" and unsupported browser/account errors should not force naturalist voice into "system failure states."

6. Privacy protects the user's relationship with their aviary. Operational telemetry must exclude "per-bird and per-account relationship data." Per-bird interaction events, personality vectors, mood, notebook sources, and relationship history are stored only to drive "that account's own aviary," not analytics warehouses, training pipelines, recommendation systems, or population dashboards.

7. Accessibility is part of the product experience. The plan says accessibility "ships with v1" and is "part of the product, not a later compliance pass." It also warns against "accessibility flattening," where screen-reader or reduced-motion users would receive "a state list or static fallback instead of the product."

8. Sharing is private, optional, and non-social. Visit invitations are "optional," "read-only," "off by default," and "revocable." Visitors never write interaction events, visits do not affect host bird drift, and the plan excludes public profiles, follows, comments, public discovery, leaderboards, and notification-driven engagement.

9. Performance is in service of the living-scene conceit. The performance plan sets "first bird visible under 500ms," a small initial bundle, stable frame rate, and no memory growth. The frontend boot sequence uses a "quiet field" instead of a spinner when data is late, preserving the sense that the aviary is continuing.

10. The system should be deterministic enough to test but not feel canned. The simulation worker is "deterministic enough to test, but seeded enough that the aviary never feels canned." Calls use stable signature seeds plus deterministic variation, motion should not "loop obviously," and recognizability is an explicit test.

11. This is a browser-native 2D living scene, not a game engine product. The ambiguity call says to use a "browser-native 2D renderer, not a full game engine," because the scene needs composition, animation, and audio timing, but not "physics, collisions, or large-world camera systems."

## Per-feature whys

### Product boundary and v1 scope

- Web-only boundary: NOT RECOVERABLE FROM PLAN
- One canonical aviary per account: the plan ties this to one server-owned aviary that syncs across phone and laptop with "no merge prompts or personality conflicts."
- Email magic-link account model: NOT RECOVERABLE FROM PLAN
- Two starter birds and later age-gated additions up to seven: starting with two and ramping higher is tied to recognizability and performance gates; the plan says to raise the allowed maximum only after tests pass at each count.
- User-assigned bird names: NOT RECOVERABLE FROM PLAN
- Hidden personality vectors, mood persistence, server-side simulation ticks, and append-only interaction events: these preserve server canonical state, avoid visible stats, and keep presence and gestures from becoming user-managed counters.
- Main horizontal browser aviary scene with three perch zones: the scene is meant to be one horizontal responsive scene with "no panning, scrolling, or zooming"; perch choice is server-driven and "reads as a behavioral signal."
- Local day/night cycle: it uses timezone and server time so the aviary remains locally alive; the plan explicitly says "Night remains alive."
- Rare ambient weather: weather affects mood and notebook candidates, giving sparse notable moments without becoming user-managed state.
- Idle micro-motion: birds start mid-action and retain faint motion cues so the aviary feels like it was continuing before arrival.
- Procedural calls: stable call signature seeds and grammar parameters make each bird recognizable while keeping assets compact and avoiding recorded audio.
- Listen-in: it gives attention to a focused bird, adds bird-specific weight to social warmth and vocal frequency, and ramps audio without fully muting other birds.
- Offers: offers are "gestures, not feeding mechanics," and must never create hunger, inventory, required care, or repeated reward loops.
- Settle: settle creates call quieting and a slow lighting transition, while tab close and settle both end presence; it has only a small mood-quieting effect and "no special drift reward."
- Field notebook: notebook entries surface sparse, specific naturalist observations and rare moments without reporting attendance, streaks, numerical traits, or event-log wording.
- Optional read-only visit invitations: visits allow limited sharing while staying off by default, revocable, read-only, and unable to affect host bird drift.
- Screen-reader narration, call captions, keyboard navigation, WCAG AA contrast, and reduced motion: accessibility is v1 scope and must deliver the product rather than a static fallback.
- Operational telemetry and performance observability: the system needs service health and performance visibility while excluding per-bird and per-account relationship data.

### System architecture

- Server-rendered shell, critical bootstrap data, CDN assets, and edge-cached first snapshot: these serve time-to-first-bird and make the first bird visible quickly.
- API service: it centralizes auth, settings, sessions, state reads, event writes, notebook, exports, visits, and accessibility preferences while enforcing account UUID usage and rejecting client-written mood or personality values.
- Simulation worker: it is the behavioral core, running slow ticks, consuming append-only interaction events, and being the only writer of personality vectors and canonical bird state.
- Email/background job worker: it keeps magic links, export links, invite links, email-change verification links, optional visit notifications, and hard deletion out of the interactive path.
- TypeScript shared schema types: responses are shared with the client so API and client agree on HTTPS JSON shapes.
- PostgreSQL with normalized identity/account/event/authorization records and JSON columns for flexible snapshots and generated payloads: this preserves relational boundaries while allowing shape-flexible simulation state.
- Redis or managed queue: it supports simulation tick scheduling, email jobs, export jobs, and rate limiting.
- Object storage for account exports: generated export JSON files can be delivered through short-lived signed URLs.
- Canvas or WebGL-backed 2D scene rendering: the scene needs composition, animation, and audio timing without physics, collisions, or large-world camera systems.
- Server-generated state snapshots plus client interpolation: this avoids browser-to-browser synchronization and keeps the server as the source of canonical state.

### Core data model and API surface

- Synthetic UUIDs and encrypted email: email appears only on the account table and email delivery payloads, while internal boundaries use synthetic account IDs instead of email.
- Sessions with device labels, token hashes, and revocation: per-device tokens are revocable, and raw tokens are never logged.
- Magic links with email hash, token hash, expiration, and consumed timestamp: magic links expire after 15 minutes, invalidate on use, and generic responses avoid revealing account existence.
- Starter aviary creation during magic-link consume: new accounts immediately get the starter aviary and two starter birds so the user can meet birds after sign-in.
- `GET /account` excluding hidden personality numbers except in export: account surfaces stay matter-of-fact and do not expose hidden personality numbers to the user-facing UI.
- `PATCH /account/settings`: users can control audio, captions, reduced motion, narration, timezone, and visit notification preference.
- Email change verification: NOT RECOVERABLE FROM PLAN
- Account export with short-lived link to verified address: the export contains the user's aviary state and is delivered through a short-lived link to the verified email address.
- Soft deletion, cancellation, and hard deletion after 30 days: NOT RECOVERABLE FROM PLAN
- `GET /aviary/bootstrap`: it returns the current snapshot, render settings, server time, local-time mapping, assets, and event token so the client can render quickly without exposing numerical personality vectors.
- `GET /aviary/snapshot?since_revision=...`: it refreshes canonical state on visibility return, long frame gaps, and keepalive, and can return no-change when appropriate.
- `POST /aviary/events`: batched append-only events with client IDs let the server validate ownership, cooldowns, event ordering windows, session authority, and idempotency before simulation consumes them.
- `GET /aviary/notebook`: cursor-paginated read-only notebook entries keep the notebook sparse and avoid edit/delete controls.
- `GET /aviary/narration`: server-generated or shared deterministic narration uses canonical state and constrained templates for voice consistency.
- Visit invitation creation, listing, expiry, and revocation: host sharing remains specific, one-time, time-bound, visible in settings, and revocable.
- Read-only visit snapshots: visitor snapshots never emit greetings, listen-in, offers, settle, presence, or notebook effects, so visits do not alter the host aviary.
- Physically separate visitor APIs: visitor tokens cannot call `/aviary/events`.

### Simulation engine design and sync

- Approximately once-per-minute simulation ticks: slow ticks keep mood, perch, calls, weather, notebook candidates, and snapshots moving even when no client is connected.
- Bounded catch-up and idempotent event consumption: elapsed time is processed in bounded steps to protect p99 latency, and event IDs or a monotonic cursor prevent double-applying drift.
- Presence pings only when visible, focused, and recently active: presence is meant to be honest presence, with a several-minute window so quiet watching counts.
- Server-derived presence windows with caps: the server does not trust large client totals, rejects impossible durations, and caps gaps.
- Personality drift as a weekly low-pass filter: drift is based on weekly-scale signal aggregates rather than immediate event clicks.
- Presence, listen-in, offers, and sustained attention as positive inputs: presence is the dominant input toward expressiveness; listen-in affects social warmth and vocal frequency; offers add small curiosity and boldness deltas; plumage saturation rises only with sustained attention.
- No negative drift from absence or neglect: absence should not move traits downward or create distress, hunger, death, or punishment.
- One-week measurable and three-week visible calibration: this keeps drift slow enough to avoid stat management but not so slow that users feel nothing matters.
- Mood transitions from inertia, time-of-day, weather, recent interactions, bird-to-bird calls, and personality: mood becomes the moment-to-moment behavioral surface.
- Mood persistence across sessions: navigation must not reset mood to neutral, and the first snapshot reflects current canonical mood.
- Greeting selection on bootstrap or visibility return: one bird may notice the user within one to two seconds, with no textual welcome and no absence-duration text.
- Stable call signatures and call grammar: mood and vocal frequency can modify timing, pitch, and ornamentation without making a bird sound like a different bird.
- Chorus events from overlapping call windows and response probabilities: chorus should emerge without a synchronized on-arrival chorus.
- Offer cooldowns and mood/personality reactions: cooldowns prevent single-session curiosity saturation, while reactions keep offers as gestures instead of feeding or reward loops.
- Settle start, undo, and complete: users can reverse within five seconds, and completion marks the aviary settled until close or active re-engagement.
- Sparse notebook generation from notable changes, rare moments, first-of-week ordering, weather effects, or sustained quiet: constrained templates protect voice consistency and avoid event logs.
- Multi-device overlap aggregation: simultaneous valid presence should be capped or treated as one account-level window to avoid double-counting drift.
- Rejected multi-device events with matter-of-fact guidance: initiating devices can receive guidance while the aviary itself simply continues.
- Personality vectors not rebuilt from event logs and migrations with no-reset invariant tests: stored vectors remain canonical and bird identity is protected across migrations.

### Frontend, audio, and accessibility

- First screen as the aviary rather than a landing page or dashboard: the first user experience is the living scene.
- Quiet field boot, no spinner, birds mid-action, and no app-like fade-in: this preserves the "already alive" conceit.
- Sparse top bar and no UI chrome inside the scene: account, accessibility, notebook, offer, and settle controls stay restrained while the scene remains primary.
- Responsive scene with every bird visible and no panning, scrolling, or zooming: spacing compresses rather than hiding birds.
- Bird rendering with species silhouettes, visual seeds, mood poses, and deterministic motion families: bird identity and mood remain visible without obvious looping.
- Offer affordance as a compact top-bar menu or panel: offers stay out of the scene as an overlay and use naturalist wording.
- Designed reduced-motion renderer: it replaces continuous micro-motion and flight paths with cross-fades while keeping day/night, mood, calls, captions, notebook, and simulation intact.
- Keyboard and focus model: no pointer-only functionality; keyboard access covers birds, listen-in, offers, settle, panels, settings, visits, export, and deletion recovery.
- WebAudio graph created after browser-permitted activation if needed: the aviary renders gracefully before activation and does not block on audio.
- Procedural audio components with stable signature seeds: compact motif parameters and grammar preserve recognizability without recorded audio.
- Audio pooling and node reuse: this avoids per-call allocation leaks and supports the no-memory-growth budget.
- Listen-in audio ramps: the focused bird gains presence slowly, other birds drop to ambient rather than silence, and local audio remains responsive.
- Captions generated from actual call grammar output: captions describe the calls that audio produces and can move to a caption lane for readability.
- WebAudio fallback to silence with captions on by default: no recorded-audio fallback ships, but the aviary remains usable.
- Screen-reader narration with slow naturalist prose: narration prioritizes user-initiated events and never exposes raw state labels, trait values, or event logs.
- Semantic model with ordinary accessible controls: birds, top bar, dialogs, notebook, offer menus, and visit/account settings meet standard keyboard and screen-reader expectations.
- WCAG AA contrast across day, evening, and night palettes: all text, icon labels, tooltips, settings, errors, captions, notebook entries, and narration remain readable.
- Unsupported browser and account errors in matter-of-fact language: system failure states do not use naturalist voice.

### Privacy, performance, rollout, and guardrails

- Per-bird relationship data stored only to drive the account's own aviary: it is not sent to analytics warehouses, training pipelines, recommendation systems, or population dashboards.
- Telemetry allowlist: request counts, latencies, tick duration, accept/reject counts, first-bird timing, frame timing, audio errors, bundle size, session-duration histograms without account dimension, and synthetic checks give operational visibility without relationship data.
- Telemetry disallowlist: average drift, per-bird funnels, named bird analytics, visit rankings, attendance calendars, and anything reconstructing a user's relationship are forbidden.
- Structured logs with PII filters, forbidden-field tests, SLO dashboards, alarms, and synthetic checks: these enforce privacy and service health.
- Bundle, first-bird, frame-rate, memory, and snapshot-size budgets: these protect the living-scene experience and prevent first-load failure.
- Code splitting for account settings, accessibility settings, visits, notebook history, export, and deletion flows: this keeps scene bootstrap minimal.
- Precomputed or edge-cached bootstrap snapshots where safe: this improves snapshot latency and first paint.
- RequestAnimationFrame scheduling with tab visibility pausing while simulation continues server-side: client rendering saves work without stopping canonical simulation.
- CI performance tests: bundle-size, first-render, 30-minute memory, frame-time, and audio allocation gates enforce the budgets.
- Prototype and calibration phase: deterministic fixtures, call recognizability tests, renderer spikes, and first-bird validation calibrate the product before alpha.
- Internal alpha with staff accounts: staff tuning focuses drift rate, greeting variation, notebook sparsity, and audio uncanniness.
- Accessibility-complete alpha: screen-reader narration, captions, keyboard, reduced motion, contrast, and WebAudio fallback arrive before beta, with accessibility bugs as launch blockers.
- Private beta account and visit completion: account settings, revocation, export, deletion, email change, visits, visit log, notifications, and multi-device testing come after the core loop.
- V1 launch cap ramp and visits off by default: bird counts rise only after recognizability and performance tests pass, and visits avoid onboarding prompts for social sharing.
- No database primitives for scores, achievements, streaks, public profiles, feed ranking, leaderboards, hunger, happiness meters, or visible trait summaries: this mitigates feature creep toward games, Tamagotchi mechanics, and social engagement.
- Notification default off and no onboarding prompts for visits: this avoids notification-driven engagement and social sharing pressure.
- Notebook voice tests, constrained templates, sparse entry policy, and banned phrases: these keep entries from becoming generic event logs or user-behavior summaries.
- Telemetry schema separation, allowlist, logging filters, and review gates for new metrics: these mitigate privacy boundary erosion.
