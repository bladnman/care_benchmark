## System-level intent

- The aviary should feel "continuously alive" and "already ongoing." This shows up in the planning posture, the server-side tick that advances mood, drift, and activity without an open client, the frontend requirement that motion start from a "mid-action pose," the quiet field loading state, and the risk that slow or janky rendering breaks the "already alive" conceit.

- Presence is the dominant input, and the relationship must remain non-punitive. The plan repeats that "presence is the dominant input," makes presence-time the "dominant positive signal" for drift, and states that "neglect never causes negative trait deltas." The out-of-scope list also excludes hunger, health, distress, death, and punitive neglect mechanics.

- Canonical continuity belongs to the server. The plan's architecture centers on "one canonical source of truth and one simulation writer." The server owns persistent simulation, invitation, notebook, and account state; clients only send events and read snapshots; there is "no last-write-wins state merge" because clients never submit state snapshots.

- V1 should be tightly scoped rather than a platform. The release framing says V1 should ship as a "tightly scoped, high-quality experience" and that protecting "tone and architectural correctness" matters more than expanding surface area. The plan explicitly cuts features that weaken aliveness, sync correctness, or privacy.

- The product should avoid game, notification, and social-network patterns. The plan excludes scores, streaks, counters, levels, rewards, follows, profiles, comments, ranking, push notifications, public discovery, and show-off rendering. The definition of done says no surface should feel like "a game, a notification system, or a social network."

- Privacy boundaries are product boundaries. The plan forbids email as a foreign key or analytics dimension, excludes notebook text, bird names, personality values, and per-account history from telemetry, segregates simulation data from aggregate telemetry, and warns that quiet social can become "a privacy leak or engagement wedge."

- Accessibility is part of the core experience, not a fallback. The plan calls for screen-reader narration, reduced-motion, captions, keyboard navigation, contrast, and focus treatment; says reduced-motion should be a "first-class render mode"; and treats accessibility delight as a release blocker rather than only a compliance box.

- The product voice alternates between naturalist prose and matter-of-fact operations. Notebook and narration surfaces use "naturalist prose" and sparse observations, while account, conflict, and error surfaces should use "matter-of-fact" copy rather than product-surface prose.

- Conservative, hardenable implementation choices matter more than expressive breadth. The plan favors polling over websockets, rule-based notebook generation over LLM generation, bounded drift and weighted state machines over opaque learned models, and a deliberately simple architecture.

## Per-feature whys

### Scope and product surface

- Web-only product for modern browsers on desktop and mobile web: NOT RECOVERABLE FROM PLAN

- Email magic-link authentication with one aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds at account creation with automatic age-based expansion up to seven birds: NOT RECOVERABLE FROM PLAN

- Single horizontal aviary scene with perch zones, day/night, rare weather, and ambient motion: the plan ties this to the core promise that the aviary feels "continuously alive" and to a responsive scene that keeps all birds onscreen.

- Server-side simulation tick: it exists so canonical mood, drift, and activity advance "whether or not a client is open," preserving continuity when the user is absent.

- Hidden per-bird personality vectors and visible per-bird moods: hidden personality avoids exposing stats, while visible moods should remain "persistent and legible through motion."

- Passive presence: presence-time is the "dominant positive signal" and is the main input for slow drift, while neglect never creates negative deltas.

- Return-greeting: greeting plans are generated semantically server-side so the client renders a "real, state-informed return" rather than a canned local animation.

- Listen-in: the focused bird gets gradual gain and spatial clarity so the interaction reads as "attention, not track switching"; it also contributes targeted uplift to social warmth and vocal frequency.

- Offers: offers contribute "small targeted lift" based on the offered item and actual reaction, keeping interaction effects bounded rather than discontinuous.

- Settle: NOT RECOVERABLE FROM PLAN

- Field notebook and notebook viewing: the notebook is "not an event log"; it should surface sparse naturalist observations through salience detection, sparsity thresholds, deduplication, and controlled prose.

- Multi-device sync: reading the same server-authored canonical state preserves "shared canonical continuity" without client-to-client coordination.

- Accessibility surfaces: they are required so accessibility users do not get a "hollow variant"; narration and reduced-motion are built from the same semantic snapshot as the visual product.

- Optional quiet social layer: per-invite read-only visits, revocation, visit logs, and notifications off by default keep social separate from discovery, co-presence, implicit tracking, or an engagement wedge.

- Aggregate operational telemetry only: telemetry is limited to operational health so analytics cannot absorb per-account simulation history or become gamification-adjacent.

### Product architecture

- `web-client`: the browser client renders the aviary, synthesizes audio, collects presence and interactions, and presents surfaces while owning only ephemeral UI state, preserving the server/client authority split.

- `aviary-api`: the stateless API handles auth, snapshots, events, notebook reads, invitations, exports, deletion, and settings as part of an "intentionally simple" service shape.

- `simulation-service`: the service owns tick processing, canonical state updates, drift, mood, notebook candidates, and snapshot publishing because the plan requires one simulation writer.

- `notification-worker`: NOT RECOVERABLE FROM PLAN

- Relational primary datastore plus append-only interaction event table/stream: this supports one canonical source of truth, ordered event processing, idempotency, auditability, and drift processing.

- Client/server split with presentational interpolation only: clients may smooth visual motion, but never compute or store authoritative personality updates.

- Semantic snapshot and rendering boundary: server snapshots carry bird identities, moods, motion keys, call seeds, ambient state, settle state, and notebook entries so the client can map semantics into motion, audio, captions, narration, and layout.

- Client-only ambient ornaments: leaves and feathers remain non-persistent because they "do not affect semantics."

### Data model and governance

- Synthetic account IDs and encrypted email: the plan says to never use email as a foreign key, analytics dimension, log identifier, or partition key.

- Presence session / ledger: presence is derived from pings but stored explicitly "for auditability and drift processing."

- Interaction events with client context and idempotency keys: ordered append-only events let the server validate schemas, reject impossible sequences, deduplicate safely, and process state changes consistently.

- Invitation and visit session records: token hashes, revocation fields, visit logs, and encrypted visitor email support explicit read-only visits and host-only visibility without broad social discovery.

- Accessibility and product settings persistence: server-side account settings and local pre-auth persistence let reduced-motion, captions, and audio preferences remain consistent across devices.

- Telemetry schema exclusions: notebook text, bird names, personality values, and per-account interaction history are excluded to protect privacy and avoid analytics over simulation content.

### API surface

- Magic-link request and consume endpoints: rate-limiting and single-use token validation make the authentication flow safe enough for account/session creation.

- Account export and deletion endpoints: exports use secure emailed links, deletion uses soft-delete and cancellation before hard purge, matching the privacy and compliance posture.

- Aviary snapshot endpoint: the endpoint returns the current canonical snapshot with a version token and minimal first-render data so the client can render instantly.

- Event batch endpoint: batching reduces network cost, while immediate flushes for listen-in, offer, and settle keep state-shaping actions timely.

- Visitor snapshot endpoint: the endpoint is read-only and never accepts event writes or creates host presence events, so visits do not affect host simulation.

- Matter-of-fact conflict and error copy payloads: operational errors should not become product-surface prose.

### Simulation engine design

- Shardable scheduled tick jobs: they let the system support "millions of quiet aviaries" by updating only what is due and meaningfully changed.

- Time-of-day and ambient advancement without recent events: this preserves continuity even when no user event has arrived.

- Bounded additive drift: saturation guards, rolling windows, and soft caps produce slow visible change without runaway saturation or single-session discontinuities.

- Mood model with hysteresis: weighted transitions and momentum keep moods from snapping every tick, so mood feels persistent and legible rather than jittery.

- Bird-to-bird interaction: local, bounded coupling creates a "small social system" without becoming chaotic flock simulation.

- Greeting selection: absence duration, boldness, mood, and greeting frequency produce a state-informed greeting plan on re-entry.

- Call grammar runtime: species motifs and stable bird seeds preserve recognizability while mood and personality adjust timing, density, pitch, envelope, and chorus response.

- Rule-based notebook generation: rule-based templates are chosen over LLM generation because determinism, privacy, and prose control matter more than expressive breadth in v1.

### Sync and consistency model

- Simulation service as only writer, API as append-only event ingester, clients as snapshot readers/event senders: this prevents last-write-wins state merge and protects canonical state.

- Polling plus visibility-based refresh: polling is "sufficient for v1 and simpler to harden," while later SSE or websocket fanout is optional.

- Conflict prevention with sequence numbers and idempotency keys: the server can accept duplicates safely, deduplicate by key, and reject stale impossible transitions only for invariants.

- Cross-device behavior: two devices can both emit events into the same log, and presence remains per session before contributing to account-level drift.

### Frontend rendering pipeline

- Declarative scene graph and state-ingestion separation: the plan separates snapshot reducer, render model, and animation/audio layers so semantic state can drive rendering cleanly.

- Minimal first render path: drawing the scene, placing birds, and starting ambient motion before less critical chrome supports first-bird timing and the "already ongoing" conceit.

- Responsive perch layout: perch anchors reposition by viewport class while preserving all birds onscreen.

- Top bar fading with inactivity: NOT RECOVERABLE FROM PLAN

- Motion from a mid-action first frame: first meaningful render must not wait for an intro animation, because the aviary is meant to feel ongoing.

- Reduced-motion rendering: pose cross-fades, dissolves, removed drift, and slower color transitions preserve readability and comfort as a first-class mode.

- Quiet field loading state: loading is "the quiet field, not a spinner," so the cold-load path does not break the conceit.

- Empty-aviary state only between signup/adoption completion and first bird arrival: NOT RECOVERABLE FROM PLAN

### Audio pipeline

- Procedural synthesis with WebAudio, motif definitions, and bird-specific seeds: the plan uses motif envelopes and stable seeds so calls can remain recognizable while being generated live.

- Listen-in mix: slow gain and clarity ramps make the interaction feel like attention rather than track switching.

- Chorus mixing: independent call generation, voice-count limits, and compression prevent clipping, phase artifacts, and harsh stacking.

- Caption and fallback integration: captions read the same procedural call description that synthesis uses, and silence plus captions preserves the aviary if WebAudio is unavailable.

- Audio performance constraints: node reuse, bounded voices, and memory-growth checks protect long quiet sessions.

### Accessibility surfaces

- Screen-reader narration: cadence-controlled naturalist prose summarizes the current scene without exposing hidden stats or raw coordinates.

- Narration from the same semantic snapshot: using the same snapshot avoids a separate accessibility-only truth model.

- Keyboard model: birds, listen-in, offer UI, notebook, settings, and settle are reachable so the product can be used without a pointer.

- Visual accessibility: contrast, focus indicators, and captions must remain readable across day/night and weather without obscuring the scene.

- Preference persistence: `prefers-reduced-motion`, explicit overrides, captions, and audio preferences persist locally when needed and server-side by account for consistency across devices.

### Performance, observability, security, and deletion

- First-bird, bundle, frame-rate, memory, and tick budgets: the plan treats performance as part of the product promise because slow or janky sessions weaken "already alive."

- Code splitting, compact snapshots, procedural assets, and avoiding heavyweight animation frameworks: these tactics serve the published bundle, render, memory, and latency budgets.

- Synthetic monitoring, aggregate-only RUM, and service metrics: observability tracks first-bird timing, unsupported browsers, snapshot latency, frame timing, audio failures, tick duration, email delivery, and revocation propagation without per-account simulation history.

- What not to measure: the plan forbids per-bird trait histories, notebook text, bird names, interaction sequences, visit streak dashboards, and bird-count dashboards to avoid privacy and gamification drift.

- Security and privacy posture: encryption, synthetic UUIDs, signed export links, rate limits, administrative access logs, and segregated data planes protect account and simulation content.

- Deletion flow: immediate soft-delete blocks ordinary sign-in except recovery, then hard-delete removes canonical state, event logs, notebook, invitations, visit logs, and account-keyed telemetry after the grace window.

### Delivery, rollout, testing, and risks

- Phase A foundations: the exit criterion is a signed-in user loading two named birds from canonical server state on two devices.

- Phase B core aliveness loop: the exit criterion is mid-motion rendering, return greetings, and server-side state advancement during absence.

- Phase C interactions and notebook: the exit criterion is completing the core session loop and receiving sparse, high-quality notebook entries.

- Phase D sync hardening and accessibility: the exit criterion is product-complete accessibility with budgets on track.

- Phase E quiet social and account polish: the exit criterion is read-only social that does not affect host simulation.

- Phase F launch hardening: the exit criterion is a verified privacy boundary, audited telemetry, and stable target performance.

- Internal dogfood: synthetic accounts and privacy-safe fixtures let the team test without exposing real simulation content.

- Small invite-only alpha: the focus is aliveness, audio uncanniness, and reduced-motion/screen-reader validation.

- Controlled beta: operational telemetry and manual notebook prose review test readiness before public v1.

- Feature flags: bird-cap ramp, social invitations, audio tuning, narration cadence, and notebook sparsity flags allow gradual rollout and tuning without code edits.

- Day-one instrumentation: operational health metrics track first render, latency, tick duration, audio errors, unsupported browsers, invite email, and export/delete health while avoiding qualitative ranking of birds or user-behavior loops.

- Drift tests: monotonicity tests ensure positive presence never lowers traits and neglect never creates negative deltas.

- Mood tests: hysteresis tests ensure no rapid oscillation under steady conditions.

- Event tests: deduplication and ordering tests protect consistency.

- Invite tests: expiry and revocation tests protect quiet-social behavior.

- Call caption tests: captions must match the procedural grammar branch actually played.

- Multi-device integration tests: concurrent sessions must converge to the same canonical state.

- Visitor integration tests: visitor sessions must never create host presence or interaction events.

- Deletion integration tests: hard-purge must remove canonical and derived records after the grace window.

- Accessibility QA: screen-reader sessions, reduced-motion continuity and comfort, keyboard traversal, contrast, and focus checks protect accessibility completeness.

- Drift calibration mitigation: explicit calibration studies, configurable coefficients, and internal drift instrumentation guard against drift that feels fake, too fast, or inert.

- Sync correctness mitigation: a single writer, append-only event log, idempotency keys, audits, and backup/restore drills guard against losing or overwriting personality continuity.

- Audio mitigation: early motif prototyping, listening tests, and "fewer, better voices" guard against canned or harsh procedural sound.

- Quiet-social mitigation: separate endpoints and schemas, explicit invites, notifications off, no badges, and copy/telemetry review guard against privacy leaks or engagement wedges.
