## System-level intent

- Treat affect as architecture, not polish. In "Product Framing and Execution Posture," the plan says the PRD's "affective constraints" are "hard technical requirements" and ends with the goal that the product should feel like "a small living window that continues without the user, notices them when they arrive, and never turns their attention into a score."

- Keep canonical life on the server. The plan repeats that "the server owns the aviary's canonical simulation state," the simulation worker is "the only writer of personality vectors," clients "never mutate personality state," and multi-device sync comes from reading "the same canonical state, not by syncing clients to each other."

- Let the browser express, not decide. In the client/server split and render pipeline boundary, the browser draws snapshots, interpolates, synthesizes calls, renders captions, manages focus, and sends events, while snapshots contain "semantic and parametric descriptors, not pixels."

- Make the first visible frame feel already alive. The plan states that "the first visible frame must be an aviary already in motion, not an app loading into existence," then reinforces "No spinner," "No fade-from-static," "No entry sequence once the aviary exists," and a "quiet field" fallback.

- Separate naturalist product voice from system voice. The plan says "User-visible product surfaces use naturalist voice; system surfaces use matter-of-fact voice," then applies that to notebook entries, narration, captions, settings, errors, auth, export, and deletion.

- Treat presence as a precise, quiet signal rather than engagement. The core thesis says "Presence is a precisely measured input to slow drift, not a generic engagement metric." Presence must use visibility, focus, and recent activity, lean toward "quiet watching," and never become "visit counts, streaks, calendars, or user-facing progress."

- Preserve non-punitive continuity. The plan excludes Tamagotchi mechanics, says personality drift "never moves downward on neglect," and allows absence to create "ambient quietness" or "slower re-orientation" but not "guilt," "visible suffering," or "distress."

- Enforce privacy by boundaries, not policy text. The plan says "Privacy is enforced by data boundaries," keeps email encrypted and fingerprinted, keeps event logs "for simulation only," disallows per-bird analytics, and allows only "aggregate operational telemetry."

- Make accessibility the product experience. The plan says "Accessibility surfaces deliver the actual product experience, not a simplified status feed," and requires screen-reader narration, reduced motion, captions, keyboard navigation, and contrast to ship in V1.

- Keep social and gamified pressure out. The scope excludes achievements, streaks, leaderboards, public discovery, profiles, comments, co-presence, and notification surfaces; visits stay "quiet," "optional," "read-only," "off by default," "revocable," and with a host log that is "reachable on demand and never pushed."

## Per-feature whys

### Product Framing and V1 Scope

- Modern web-only Pocket Aviary optimized for current Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN.

- Single-user accounts: The plan ties this to "one canonical server-side aviary per account," no "shared aviaries or multi-user simulation," and a product boundary against public social surfaces and co-presence.

- Email magic-link sign-in: The plan articulates security and system-surface behavior: single-use tokens, 15-minute expiration, rate limits, generic responses, and matter-of-fact email/sign-in handling. It does not articulate why magic links were chosen over other account methods.

- One canonical aviary per account: The why is sync correctness. All devices read the "same canonical state"; stale clients cannot overwrite state, client retries cannot duplicate events, and conflicts are meant to be "architecturally impossible."

- Two starter birds at adoption: The plan uses two birds as the first usable state: a new user names "two system-selected starter birds" and then sees an aviary already in motion. It does not articulate why the number is specifically two.

- Additional birds unlocked by aviary age: The why is to make arrival "part of the aviary's life, not a reward." The schedule must not depend on "visit count, offers, listen-in totals, payments, or engagement."

- Hard cap of seven birds: The plan tests recognizability "up to seven birds" and sets a V1 cap, but the specific rationale for seven is NOT RECOVERABLE FROM PLAN.

- User-assigned and renameable bird names: NOT RECOVERABLE FROM PLAN.

- Small coherent species pool of about six species: The plan wants species to provide silhouette, palette, pose set, motif, and accessibility descriptors while avoiding rarity and avoiding species stereotypes: "individual bird identity should matter more than species stereotype."

- Server-selected starter birds rather than a user catalog: The why is to enforce "balanced variety rules," avoid a "catalog of choices," and keep common virtual-pet affordances from becoming customization or optimization.

- Hidden server-side personality vectors: The why is to support distinct long-term bird identity and slow drift while avoiding optimization surfaces. Personality fields are not returned to ordinary UI endpoints, and the client cannot write trait deltas or absolute values.

- Fast-timescale mood: The plan uses mood to make state visible through "motion, perch, calls, and offer reactions" without exposing raw labels. It also lets birds be quieter after absence without reducing personality values.

- Server-side simulation tick at a slow cadence: The why is canonical continuity. The aviary should reflect elapsed time and prior inputs when the user returns, "not a frozen client resume."

- Append-only interaction event ingestion: The why is ordered, validated, deduplicated simulation input. The event log is "for simulation only" and is not copied into analytics.

- Precise presence accounting: The why is calibrated drift. Presence counts only visible, focused windows with recent pointer/key activity, tolerates quiet watching, and rejects background tabs, laptop sleep gaps, and stale or impossible pings.

- Return-greeting by one bird within the first second or two: The why is recognition without announcement. One primary greeter varies by absence, mood, personality, and history, and "No text accompanies this greeting."

- Listen-in interaction: The why is focused attention without isolation. The focused bird's calls ramp up gradually, other birds ramp down but "never silent," and listen-in contributes targeted social warmth and vocal-frequency signals.

- Offer interaction for seed, song fragment, and still pool: The why is gentle interaction whose consequences are mediated by mood, curiosity, proximity, state, cooldowns, and saturation, so the user cannot "spam offers to force drift."

- Settle gesture: The why is a quiet evening mode, not a reward or obligation. It closes the presence window, quiets mood, has "no direct long-term trait push," and closing the tab without settle is "equally valid."

- Field notebook: The why is sparse, stable naturalist observation rather than a log or achievement feed. It is read-only, uses no "you," no streaks, no numeric trait values, and stores final entry text so it remains stable.

- Thin top bar: The why is to keep chrome available without taking over the scene. It contains settings, accessibility, notebook, and offer affordance, fades on stillness, returns on activity, and follows "notice, never announce."

- Responsive single-scene aviary with no panning, scrolling, zooming, or cropped birds: The why is to preserve a single living window where every bird remains visible across viewports.

- Day/night cycle anchored to local timezone: The plan uses local timezone to drive day/night, local time context, and time-of-day mood inputs.

- Rare quiet ambient weather and visual micro-motion: The why is to help the scene feel alive and modulate mood quietly without producty announcements.

- Procedural client-side calls: The why is to avoid recorded loops while keeping calls varied, recognizable per bird, and memory bounded.

- Call captions from the same procedural grammar as audio: The why is fidelity. Captions match generated calls rather than being hard-coded labels, and become the fallback when WebAudio is blocked or unavailable.

- WebAudio graceful-silence fallback: The why is browser policy. Autoplay may block audio, so the aviary still renders normally in "graceful silence" with high-quality captions and unobtrusive audio recovery.

- Screen-reader narration in naturalist prose: The why is to expose the same aviary charm from the same snapshot state without raw mood labels, perch numbers, personality values, or event-log phrasing.

- Reduced-motion rendering: The why is to be a "designed alternate visual register" where users still see bird identity, current mood, calls, captions, narration, and drift, not a "broken static fallback."

- Keyboard navigation: The why is that no core interaction should require pointer-only access. Keyboard users must be able to sign in, adopt, experience, listen in, offer, settle, open notebook, manage settings, export/delete, and manage visits.

- Account export: The why is private account data access, including vectors because the PRD requires export of current aviary state. The plan keeps it machine-readable, behind account settings, emailed to the verified address, expiring, and not a naturalist product surface.

- Account deletion with 30-day soft deletion and hard deletion afterward: The why is privacy and recovery. Soft deletion immediately hides/disables the account; hard deletion removes simulation data, vectors, moods, notebook, event logs, invites, sessions, and export jobs.

- Optional read-only visit invitations: The why is quiet sharing without social creep. Visits are off by default, revocable, expiring, scoped to one aviary, and visitors cannot affect presence, drift, offers, listen-in, settle, notebook, or host notifications by default.

- Host visit log in settings: The why is host control without attention pressure: it is "reachable on demand and never pushed" and has no badge.

- Aggregate-only operational telemetry: The why is reliability and performance observability without privacy erosion or engagement drift. Allowed metrics are request rates, latencies, errors, first-bird timing, frame timing, audio errors, and tick health, without account or bird dimensions.

### Architecture Overview

- Web client with TypeScript, component chrome, scene renderer, WebAudio, and accessibility layers: The why is separation of concerns. React or equivalent handles top bar and panels, while a dedicated Canvas/WebGL-backed renderer avoids DOM-heavy animation.

- Public API service: The why is to centralize auth, snapshots, events, settings, export, deletion, visits, and matter-of-fact error responses behind server validation and synthetic account IDs.

- Simulation worker service: The why is to make one service responsible for ticks, mood, drift, calls, notebook candidates, weather, adoption availability, canonical snapshots, and all personality writes.

- Email service integration: The why is account and data flow delivery: magic links, export links, and visit invitations. It explicitly has "No aviary-state engagement email."

- Relational persistence plus queue/job system: The why is durable canonical records, ordered events, scheduled simulation, email/export/deletion jobs, and audit metadata.

- Low-latency cache that is not canonical: The why is fast session/config/species/bootstrap access without allowing cache to become the simulation store.

- Static asset/CDN edge: The why is fast HTML, JS, assets, and initial snapshot bootstrap that support "fast first-bird render without waiting for settings/account chunks."

- Aggregate observability pipeline: The why is operational visibility only, with no per-bird state, notebook content, account-specific interaction history, or account email.

- Snapshot descriptors instead of pixels: The why is preserving "server's canonical continuity" while keeping client rendering "fast and expressive."

### Data Model

- Account record with synthetic UUID and encrypted/fingerprinted email: The why is to avoid using email as database key, log key, queue key, telemetry dimension, partition key, or URL parameter.

- Device sessions: The why is revocable per-device access, matter-of-fact timeout errors, and session tokens that authorize event submission but not direct simulation writes.

- Aviary record: The why is one-to-one V1 ownership, local timezone day/night inputs, settled state, weather, tick state, snapshot versioning, and non-public default behavior.

- Bird record with stable IDs, seeds, personality, mood, perch, cooldowns, and greeting state: The why is persistent identity. Species migrations, asset updates, and call grammar updates must preserve `bird_id`, seeds, personality, mood, and notebook references.

- Species metadata as versioned configuration: The why is to define visual, call, night, and accessibility variation without making species rarity a V1 feature.

- Interaction Event Log: The why is idempotent, ordered, bounded simulation input that visitors cannot use to affect host state and that is not copied into analytics.

- Presence Window: The why is to simplify drift from validated pings while counting only valid visible, focused, recently active presence and hiding it from user-facing progress.

- Mood and Expression State: The why is to reconcile no downward neglect drift with quieter, more ambient returns after absence. It modulates expressiveness without becoming happiness, hunger, affection, or obligation state.

- Notebook Entry: The why is naturalist, sparse, stable observations without achievement framing, numeric trait values, system event-log language, or user visit-frequency claims.

- User Settings: The why is direct control over audio, captions, reduced motion, narration, visits, timezone, and contrast, with matter-of-fact settings copy.

- Visit Invite and Visit Session: The why is bounded, revocable, read-only access that does not make the emailed token a reusable bearer token and cannot feed host simulation.

- Export Job: The why is on-demand private export through a verified, expiring email link, not an advertised naturalist product feature.

### API Surface

- Structured API errors and idempotency keys: The why is predictable system surfaces, retry safety, and avoidance of duplicate mutations.

- Auth magic-link request and consume endpoints: The why is normalized lookup, rate limiting, single-use token consumption, generic responses, session issuance, and safe new-account creation.

- Session list and revoke endpoints: The why is account settings control over active device sessions.

- Adoption endpoint: The why is server-selected starter or age-unlocked availability with descriptors sufficient for naming, without a catalog or gamified adoption counters.

- Bird rename endpoint: The why is name validation while preserving personality, mood, call signature, and notebook history.

- Aviary snapshot endpoint: The why is a small, private, versioned rendering contract with day phase, bird descriptors, call schedule, transitions, presence config, and rendering settings, avoiding unnecessary payloads and private cache leakage.

- Bootstrap snapshot and quiet-field fallback: The why is to avoid a spinner and make the first frame feel alive even when a snapshot is delayed.

- Event ingestion endpoint: The why is host-only, validated, bounded, idempotent input where acceptance is acknowledged separately from durable simulation results.

- Offers endpoint: The why is immediate render descriptors with durable effects left to the tick, so cooldowns and current state are respected.

- Notebook endpoint without edit/delete: The why is read-only observations that do not become engagement achievements or notification counts.

- Account settings, email change, export, delete, restore endpoints: The why is matter-of-fact account control over preferences, verified email changes, data access, soft deletion, and restoration.

- Visit invite, consume, snapshot, revoke, and log endpoints: The why is non-public, revocable, read-only visiting where revoked sessions get matter-of-fact unavailable surfaces and never emit host events.

### Simulation Engine Design

- Adaptive tick lifecycle: The why is to preserve correct canonical state at return while avoiding unnecessary minute-by-minute work for inactive accounts.

- Per-aviary locks and transactional tick persistence: The why is no partial tick writes, no duplicate tick corruption, and one tick writer per aviary.

- Personality ranges and seeds: The why is distinct but not extreme starter birds, with individual identity mattering more than species stereotype.

- Drift function as slow low-pass filter: The why is one-week measurable and three-week felt movement, no single-session visible movement, saturation under heavy activity, and no downward movement from neglect.

- Neglect handling through expression envelope: The why is ambient quietness and changed return-greeting form without guilt, suffering, distress, or personality regression.

- Mood transition model: The why is fast visible state that persists across sessions, responds to time/weather/events/personality/bird influence, and outputs rendering descriptors rather than labels shown to the user.

- Return-greeting algorithm: The why is a single varied greeting that notices the user quickly without text, avoids the same bird always greeting, and avoids simultaneous greetings.

- Perch and motion state: The why is that perch position is a "signal, not user-controlled layout," using hysteresis to avoid jitter and keeping all birds visible.

- Offer handling: The why is soft reaction logic, cooldown protection, and non-punitive no-reaction states, with the tick responsible for mood and drift consequences.

- Field notebook generation: The why is rules-first, sparse, specific naturalist prose with strict filters and lint against achievement words, direct "you," trait numbers, and event-log phrasing.

### Sync Model and Correctness

- Canonical State: The why is cross-device consistency from server state, with one personality vector per bird, one tick writer, server-assigned event order, idempotent retries, and no stale overwrite.

- Event Ordering: The why is safe duration interpretation without trusting client time for ordering.

- Concurrency model: The why is lock-protected ticks while ingestion appends concurrently for the next tick.

- Client snapshot refresh: The why is to reconcile local interpolation with server truth after open, visibility changes, sleep/wake, keepalive, or mutation acknowledgement.

- Conflict surfaces: The why is to keep conflicts rare and matter-of-fact: expired links, revoked sessions, snapshot load failure, revoked visits, unsupported browser, or outage.

### Frontend Rendering Pipeline

- Canvas/WebGL scene renderer with retained scene graph: The why is 60fps idle motion, reduced-motion cross-fades, predictable main thread, and avoiding per-frame React state for birds.

- Scene composition with background, depth-aware perches, birds, weather, captions, and focus layer: The why is to keep the scene visual and keep controls, badges, labels, and hover UI out of it.

- First frame and loading path: The why is to meet the "already running" conceit with first bird visible under budget, no spinner, quiet-field fallback, and birds drawn mid-action once the snapshot arrives.

- Responsive layout solver: The why is all birds in frame on narrow phones, expanded spacing on desktop, no cropping, no panning, no scrolling, and no zooming.

- Normal motion system: The why is continuous, mood/personality-keyed life through preen, scan, head tilt, body shuffle, call posture, ambient drift, gradual palette changes, and quiet weather.

- Hidden/background tab behavior: The why is to stop rendering, pause WebAudio as required, stop counting presence, and refresh snapshot on return.

- Top bar fade/return: The why is to keep affordances available while protecting the aviary scene from badges, notification pressure, and permanent chrome.

- Listen-in UX: The why is accessible focus on a bird, subtle visual focus, graceful disengagement, and audio rebalancing without labels over birds.

- Offer UX: The why is top-bar mediated, keyboard navigable interaction without direct click-on-bird feeding, punitive cooldown copy, or rejection-like no-reaction messaging.

- Settle UX: The why is slow evening lighting, quieter calls, a five-second undo, and no implication that the user must settle before leaving.

- Field notebook UI: The why is readable field-note texture, read-only indefinite scrolling, and no edit/delete/comment affordance.

### Audio Pipeline

- WebAudio procedural synthesis engine: The why is seeded, species-specific, personality/mood-modulated calls with runtime variation, matching captions, bounded memory, and no recorded call assets.

- Server call scheduling descriptors: The why is canonical call timing and chorus relationships while client scheduling adapts to local `AudioContext.currentTime`.

- Listen-in mix buses and ramps: The why is gentle focus, no hard cuts, no full muting of non-focused birds, and symmetric engage/disengage.

- Browser audio policy handling: The why is a launch-risk mitigation for blocked autoplay: unlock where possible during onboarding, persist audio preference, graceful silence when blocked, no "Welcome back" or loud permission banner.

- Captions: The why is generated, contrast-safe, bird-adjacent call description that aligns with audio, supports fallback, and respects reduced motion.

### Accessibility Plan

- Accessibility ships in V1: The why is explicit: "Accessibility ships in V1, not after launch."

- Screen-reader narration generator: The why is naturalist prose over the same snapshot state, slow idle cadence, priority bumps for meaningful changes, and no raw state, personality, or event-log phrasing.

- Keyboard Navigation: The why is complete access to top bar, birds, listen-in, offer, settle, notebook, settings, export/delete, and visits without pointer-only requirements.

- Focus indicators: The why is usability across bright, dim, rainy, and night palettes while fitting the surface.

- Reduced Motion: The why is first-class access from OS preference or user override while preserving mood and identity.

- Contrast and Text: The why is WCAG AA readability for controls, settings, errors, captions, notebook, and displayed narration while keeping the scene free of labels except captions/focus surfaces.

- Accessibility Testing: The why is proof that screen-reader, keyboard, touch, reduced-motion, audio-disabled, blocked-WebAudio, and contrast modes can complete core flows.

### Privacy and Security

- PII Boundary: The why is to keep email encrypted, indexed only by keyed fingerprint, absent from logs/URLs/telemetry/partition keys, and to treat even account IDs as sensitive operational data.

- Interaction Data Boundary: The why is to keep per-bird and per-account interaction state inside that user's simulation only, not analytics, model training, recommendations, population dashboards, or third parties.

- Auth Security: The why is short-lived single-use magic links, hashed tokens, rate limits, generic responses, secure sessions, CSRF protection, revocation, and scoped read-only visit links.

- Deletion: The why is to hide/disable accounts immediately, allow 30-day recovery, then remove simulation data and make queued work no-op after hard deletion.

### Performance and Observability

- Hard V1 budgets: The why is to preserve the "already running" experience: small bundle, first bird under 500ms, 60fps idle motion, no memory growth, and tick p99 alarm at 5 seconds.

- Client performance strategy: The why is to keep scene rendering fast by splitting chunks, using compact assets, avoiding recorded audio, pooling objects/nodes, batching draws, pausing hidden tabs, and measuring key first-frame marks.

- Server performance strategy: The why is small snapshots, indexed access, bounded tick batches, lock timeouts, deterministic notebook latency, and safe catch-up for long-inactive accounts.

- Observability: The why is operational insight through aggregate request, snapshot, event, tick, render, audio, memory, and accessibility metrics without engagement metrics or per-account/bird dimensions.

- Synthetic monitoring: The why is external verification of first-bird, snapshot, frame timing, tick health, and normal/reduced-motion behavior on seeded non-sensitive accounts.

### Rollout Plan

- Phase A Foundations: The why is to prove sign-in, canonical aviary records, privacy-safe logging, and no-op ticks before product complexity.

- Phase B Simulation Core: The why is to prove no client personality writes, calibrated drift, no presence inflation, and non-punitive absence before rendering polish.

- Phase C Aviary Rendering: The why is to prove first-bird budget, no cropped birds, no spinner/entry sequence, and sustained 60fps scene behavior.

- Phase D Audio and Captions: The why is to prove recognizable, non-looped calls, gentle listen-in, caption fidelity, and stable memory.

- Phase E Accessibility and System Surfaces: The why is to prove core flows work for screen-reader and keyboard users, reduced motion is intentional, system voice is matter-of-fact, and export/deletion do not leak PII.

- Phase F Visits: The why is to prove visitors cannot affect host simulation, revocation works, and no profile/discovery/follow/comment surfaces exist.

- Phase G Beta Hardening: The why is to meet performance budgets, tick headroom, privacy review, accessibility audit, content/voice QA, browser support, runbooks, and product sign-off against gamification, announcements, and social creep.

- Launch cohorts: The why is controlled ramp from internal dogfood to private beta, broader beta, and general V1 launch while age scheduling naturally limits near-term bird count.

- Feature flags and kill switches: The why is to control visits, notebook generation, weather, age-based offers, audio captions, tick cadence, drift coefficients, WebAudio scheduling, weather, and visit invitations. There is "No feature flag for gamification or engagement surfaces."

### Testing Strategy

- Unit tests: The why is to validate presence, event schemas, auth tokens, sessions, drift, mood, offers, notebook guardrails, visits, export, and deletion at small scope.

- Integration tests: The why is to prove multi-device consistency, concurrent ingestion/ticks, retry safety, stale-client protection, sleep/wake refresh, visitor isolation, soft-delete blocking, and hard-delete removal.

- Simulation Calibration Tests: The why is to verify one-week measurable drift, longer-term visible descriptor changes, no negative drift, no background presence inflation, and no single-session trait jumps.

- Frontend Tests: The why is to prove viewports, captions, top bar fade, keyboard order, listen-in, offer, settle undo, reduced motion, and no-spinner first frame.

- Audio Tests: The why is to prove varied but recognizable calls, correct mix ramps, audible non-focused birds, caption alignment, graceful WebAudio failure, and memory stability.

- Accessibility Tests: The why is to prove chrome/panel checks, manual screen-reader use, keyboard-only flow, reduced motion, palette contrast, and multi-bird captions.

- Privacy Tests: The why is to prove logs and telemetry exclude email, tokens, bird names, notebook text, event payloads, and per-bird/per-account simulation fields, while export and deletion behave correctly.

### Key Product Decisions, Risks, Guardrails, Workstreams, Definition of Done

- Early rendering technology decision: The why is to choose Canvas/WebGL stack and asset format before building bird assets.

- Early trait ranges and drift coefficient config shape: The why is to lock calibration shape before implementation dependencies spread.

- Early presence activity-window calibration: The why is to balance quiet watching against false presence inflation.

- Early mood enum and transition model: The why is to keep visual, audio, offer, and narration behavior consistent.

- Early species pool and motif library structure: The why is to align visual identity, call identity, and accessibility descriptors.

- Early snapshot schema versioning: The why is that the snapshot contract integrates simulation, rendering, audio, and accessibility.

- Early notebook/narration/caption template style guide and automated lint rules: The why is to keep voice from becoming logs or achievements.

- Early browser audio policy UX: The why is that first-ever audible calls are constrained by browser policy.

- Early initial bird age-unlock schedule: The why is to configure non-engagement-based arrivals.

- Early raw consumed interaction event retention period: The why is privacy and continuity after events have been consumed.

- Drift calibration mitigations: The why is to avoid stat-management feel, irrelevant attention, or punitive neglect.

- Sync correctness mitigations: The why is to protect bird identity, drift, and personality from duplicate ticks, stale clients, or client-authored state.

- Audio mitigations: The why is to avoid canned calls and handle autoplay blocking through early prototypes, audio expertise, recognizability tests, and graceful silence.

- Performance mitigations: The why is to protect the "Already Running" conceit from spinners, late birds, and frame drops.

- Accessibility mitigations: The why is to prevent screen-reader and reduced-motion experiences from becoming flat fallbacks.

- Privacy mitigations: The why is to stop "helpful" analytics from reading simulation data into dashboards, logs, or training sets.

- Social-feature mitigations: The why is to stop visits from expanding into profiles, badges, friend lists, comments, notifications, or leaderboard-like stats.

- Voice mitigations: The why is to keep notebook, narration, captions, and offers from reading like logs or achievements.

- Customization mitigations: The why is to avoid scene customization, bird catalogs, and placement controls, with perch remaining simulation output only.

- Engineering guardrails: The why is code review and launch readiness enforcement against personality writes outside simulation, trait fields in client payloads, gamification words, bird/account analytics, logged email, notification badges, visitor-to-host simulation paths, recorded calls, spinners, broken reduced motion, and visible personality numbers outside export.

- Suggested team workstreams: The why is early integration through snapshot schema and event log across simulation, rendering, audio, accounts/privacy, accessibility, visits, and performance/observability.

- Definition of Done for V1: The why is to ensure the finished product meets the plan's core behavior: sign in, two starter birds, already-in-motion aviary, varied greetings, slow valid-presence drift, persistent personality, complete interactions, visitor isolation, complete accessibility paths, performance budgets, aggregate-only telemetry, and no gamification, Tamagotchi, notification, public social, or native-app surfaces.
