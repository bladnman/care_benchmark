## System-level intent

- Server-owned continuity, with clients as renderers rather than authorities. This appears in "one canonical aviary per account," "server-side simulation tick governing canonical bird state," the service boundary that "the client is a renderer and event producer, never a canonical state authority," and the render boundary: "Server owns 'what is true now'; client owns 'how it is smoothly shown now.'"

- Quiet, nongamified, non-custodial product tone. The plan explicitly excludes "gamification of any kind" and "Tamagotchi mechanics," frames bird addition as "quiet and nongamified," says adoption should preserve the "birds arrived" framing, and names a product tone rule in the risk section: "notice, never announce."

- Monotonic expressive drift from qualified presence, not punishment for absence. The plan pairs "presence-driven personality drift" with "bounded monotonic increases only," "Drift never decreases traits on neglect," and the final non-negotiable phrase "monotonic expressive drift."

- Naturalist accessibility as a first-order surface. The plan says "Accessibility-first narration," "dedicated narration region with paced naturalist prose updates," and warns that narration, captions, and reduced-motion must be "primary deliverables in definition of done, not follow-up polish."

- Privacy boundaries around behavioral data. This shows up in "aggregate-only operational telemetry," analytics that "do not read simulation tables or interaction logs with per-account semantic payloads," encrypted email, synthetic IDs, and telemetry that never includes "bird names, personality vectors, notebook text, or per-account interaction histories."

- Procedural, recognizable life rather than heavy media or canned labels. The plan uses "procedural calls," "motif library," "stable signature seed ensuring recognizability across sessions," compact descriptors, and captions that match "actual runtime call shape, not a canned label bank."

- Calm startup and responsive containment. The plan centers "first-bird rendering stays under budget," "first bird visible under 500 ms," "no spinner," "one horizontal responsive aviary scene with no pan, zoom, or scroll," and preserving "all birds in frame across phone and desktop widths."

- Convergent multi-device behavior without visitor mutation. The plan calls for "multi-device sync by shared server-authored canonical state," says clients "never merge personality state," serializes events on the server, and states that "Visitors never contribute drift or host-visible presence."

## Per-feature whys

### Scope

- Browser-only product for modern Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Single-user accounts with email magic-link authentication: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: The rationale is canonical continuity. The plan repeatedly ties this to server-owned state, a single ordered event log, and snapshots that let multiple devices converge without client-side personality merges.

- Two starter birds and server-controlled age-based expansion up to seven: The plan presents two birds as the onboarding and internal vertical-slice baseline, and expansion as age-based server-side milestones. The unlock surface stays "quiet and nongamified," and broader expansion waits until "audio recognizability and performance remain healthy."

- One horizontal responsive aviary scene with no pan, zoom, or scroll: The rationale is to preserve "all birds in frame" across phone and desktop, compress or widen perch spacing without cropping, and avoid adding "in-scene chrome."

- Exclusion of gamification and Tamagotchi mechanics: The rationale is product tone and emotional safety: no "streaks, achievements, scores," no "hunger, decay, punishment, visible distress," and no "negative drift on absence."

### Product Architecture

- Web client: It renders, synthesizes audio, measures eligible presence, submits events, and renders accessible surfaces because the client is "a renderer and event producer," not a canonical state authority.

- API service: It is the only ingress for "account, session, snapshot, and interaction traffic," so it can authenticate, serve canonical snapshots, accept events, issue invites, and enforce policies consistently.

- Simulation worker: It is the only writer for personality vectors, moods, call timing seeds, weather state, and derived canonical bird state, preserving canonical simulation ownership.

- Primary database and append-only event log: The plan uses relational storage to hold accounts, birds, canonical state, interaction events, notebook entries, invites, sessions, and audit-safe operational metadata so canonical state and ordered history survive across devices and ticks.

- Queue/scheduler: Its rationale is cadence and load control: it triggers per-account ticks, queues heavier work, uses jitter to avoid "fleet spikes," and lets elapsed intervals coalesce if the system falls behind.

- Email service: NOT RECOVERABLE FROM PLAN

- Edge delivery and bootstrap snapshot path: The rationale is startup speed: the HTML shell plus small bootstrap snapshot path keep "first-bird rendering" under budget.

- Client/server render pipeline boundary: The server sends semantic and physical targets rather than frame data so it owns "what is true now," while the client owns smooth interpolation, ambience, and local display.

### Data Model and Handling

- Encrypted email and synthetic IDs: The rationale is privacy: email appears only on account and invite records in encrypted form, while logs and downstream systems use synthetic IDs.

- Personality vectors stored canonically and updated only by the simulation worker: This prevents client-facing settings or analytics from exposing personality vectors and keeps drift under server control.

- Session fields including device label and user-agent summary: NOT RECOVERABLE FROM PLAN

- AviaryState version and server time context: The rationale is conflict prevention and continuity: snapshot responses include version and server time so clients can discard stale responses and render the current day phase, settled state, and weather.

- Stable bird identity, visual seeds, and call signature seeds: The rationale is identity continuity and recognizability: `bird_id` is "stable forever," and signature seeds preserve recognizable timbre and rhythm families across sessions.

- Ordered InteractionEvent records with `dedupe_key`: The rationale is idempotent append-only ingestion. The server orders and persists events, acknowledges accepted IDs, and uses dedupe keys to prevent duplicate effects.

- PresenceWindow with validated windows rather than raw pointer paths or keystroke contents: The rationale is "presence measurement honesty" and privacy. Drift uses qualified presence seconds, while raw pointer paths and keystroke contents are not stored.

- Immutable NotebookEntry text: NOT RECOVERABLE FROM PLAN

- Invite token hashes, status, and revoke/expiry fields: The rationale is invite, privacy, and session policy enforcement, including immediate revocation and read-only visitor access.

- VisitLog approximate duration: NOT RECOVERABLE FROM PLAN

### API Surface

- Magic-link request with rate-limit-safe generic copy: The rationale is safe auth behavior: the endpoint returns an accepted response without revealing account state.

- Account export, deletion, cancel deletion, and email-change flows: NOT RECOVERABLE FROM PLAN

- Aviary snapshot endpoint: It returns canonical snapshot version, server timestamp, day phase, settled state, weather, birds, greeting hints, call motif seeds, and narration descriptors so the client can render the current aviary without becoming the source of truth.

- Aviary bootstrap endpoint: The rationale is first-bird startup, with a minimal payload optimized for rapid initial render.

- Batch interaction event endpoint: The rationale is ordered append-only event capture with dedupe keys and accepted IDs, feeding the simulation worker without client-side canonical writes.

- Optional dedicated presence endpoint: The rationale is isolated controls for presence traffic if needed; otherwise presence can be folded into the events batch.

- Visitor snapshot endpoint stripped of host-only controls and event endpoints: The rationale is that visitors may observe but cannot mutate host state or contribute drift.

- Visit open and close timing endpoints: NOT RECOVERABLE FROM PLAN

- Error and conflict behavior: Expired links, revoked invites, unsupported browsers, timeouts, and failures use "matter-of-fact system copy"; idempotent snapshot fetches retry with capped backoff; event ingestion is idempotent by `dedupe_key`.

### Simulation Engine Design

- One simulation tick per account per minute with jitter and coalescing: The rationale is regular canonical updates without fleet spikes, duplicate time-window processing, or replaying every missed minute individually when behind.

- Tick phases ending in atomic persistence with processed event watermark: The rationale is ordered, auditable folding of interaction events into presence, drift, mood, perch targets, calls, notebook moments, and canonical state.

- Low-pass drift accumulation from qualified presence seconds: The rationale is gradual expressive change. The plan wants "instrument-detectable" movement after roughly one week, "human-noticeable" change after roughly three weeks, and "no visible trait swing within a single ordinary session."

- Listen-in, offer, and settle effects on simulation: Listen-in increases social warmth and vocal frequency; accepted or near-target offers slightly increase curiosity and boldness; settle quiets mood only and does not change long-term drift direction.

- Daily and weekly drift caps: The rationale is to prevent "a single unusually long day" from compressing multi-week change into one session.

- Mood state machine: The rationale is momentary behavior that responds to current mood, personality, offers, listen-in attention, time of day, weather, and nearby bird effects while persisting canonically across sessions.

- Greeting runtime: The rationale is plausible returning-user behavior. On newly visible snapshots, the server marks one or two greeting candidates from absence length, mood, and boldness; the client resolves timing, staggers candidates, and avoids unison.

- Call grammar runtime: The rationale is recognizable procedural audio. Species motif libraries and bird signature seeds make calls recognizable, while mood and vocal frequency vary interval, density, pitch contour, and chorus participation.

- Bird addition milestones: The rationale is quiet expansion. New birds unlock through server-side aviary age milestones, with "soft offer" product voice instead of ceremony, and adoption keeps the "birds arrived" framing.

### Sync Model

- Server-owned canonical snapshots with monotonically increasing version: The rationale is that clients fetch and render snapshots, never merge personality state, and can discard stale responses.

- Conflict prevention through no absolute trait writes, dedupe keys, coarse presence, and server time: The rationale is to avoid overcount, duplicated effects, and inconsistent moods across devices.

- Multi-device behavior for laptop and phone: The rationale is convergence. Either client can submit listen-in, offer, or settle events, and both later snapshots converge because "the server serializes events."

- Recovery after suspension, offline gaps, and failed event submission: The rationale is freshness and bounded replay. Clients fetch a fresh snapshot before resuming motion/audio, retry idempotently for a short window, and drop expired events rather than replay indefinitely.

### Frontend Rendering Pipeline

- Scene composition layers: NOT RECOVERABLE FROM PLAN

- Three perch zones as semantic targets rather than freeform drag positions: The rationale is to keep server snapshots semantic and renderable, with perch zones as stable targets rather than user-manipulated coordinates.

- First-bird rendering with quiet field, no spinner, and bird already mid-action: The rationale is calm startup under budget: if the snapshot is delayed, the client shows a quiet field, and the bird appears already living in the scene rather than entering theatrically.

- Default motion system: The rationale is smooth life without server frame data: requestAnimationFrame interpolation, low-frequency pose changes, micro-motion loops, subtle parallax, and ambient ornament generation.

- Reduced-motion rendering mode: The rationale is accessible continuity: cross-fades replace continuous motion, animated paths, leaf/feather drift, while day/night transitions are slowed but retained.

- Listen-in rendering: The rationale is focused attention without overt badges: the audio mix ramps and visual posture/orientation may subtly reinforce focus.

- Offer rendering: The rationale is quiet interaction. Offers come from a top-bar affordance and create in-scene props only as long as needed.

- Settle rendering with warm/dim lighting and five-second undo: The rationale is reversible quieting: settle biases birds toward drowsy/quiet and provides an undo window without overwriting personality.

- Top bar fade on stillness: The rationale is visual quiet: top bar chrome fades to near transparency on pointer/keyboard stillness and returns on activity.

- Responsive behavior: The rationale is to keep all birds in frame, adjust spacing across widths, and preserve accessible touch and keyboard hit targets without adding in-scene chrome.

### Audio Pipeline

- WebAudio-only procedural synthesis from motif libraries: The rationale is procedural, compact, species-specific sound without heavy media, driven by snapshot descriptors.

- One audio engine instance with bounded voices and reusable nodes: The rationale is performance and resource control across a tab session.

- Listen-in audio mix: The rationale is attention shaping: the focused bird rises, others attenuate but remain audible, and weather and settle gently affect the global mix.

- Chorus handling: The rationale is clarity at up to seven birds. Calls are generated per event to avoid phasey loop artifacts, and the mixer limits concurrency.

- Subtle spatialization: The rationale is product tone: "do not turn the aviary into an exaggerated stereo toy."

- Caption coupling: The rationale is fidelity between audio and text: each generated call emits a structured descriptor, and captions match "actual runtime call shape."

- No recorded-audio fallback path: NOT RECOVERABLE FROM PLAN

### Accessibility Surfaces

- Dedicated screen-reader narration region: The rationale is paced naturalist access to the same aviary moment, prioritizing user-triggered event narration and keeping idle cadence around 30-60 seconds to avoid queue overload.

- Keyboard and focus model: The rationale is keyboard-only access through top bar controls, aviary entry, per-bird arrow navigation, Enter for listen-in, Escape for exit/close, and high-contrast focus outlines in all day/night states.

- Captions and visual text: The rationale is accessible call understanding and readability: captions follow the calling bird while avoiding collisions, and all text surfaces meet WCAG AA contrast.

- Reduced-motion and audio-off users: The rationale is no degradation in product voice. The app respects `prefers-reduced-motion`, persists overrides, and lets audio-off or hardware-muted users use captioning and notebook/noticing surfaces.

- Accessibility testing gates: The rationale is regression prevention and launch quality through screen-reader smoke coverage, keyboard-only CI tests, contrast checks, and manual narration prose review.

### Performance Budgets and Observability

- Performance budgets: The rationale is to protect the core experience: first bird under 500 ms, idle motion at 60 fps, flat memory over 30 minutes, and simulation tick p99 under 5 seconds.

- Splitting settings, invite, and export flows out of the critical path: The rationale is startup performance and first-bird budget protection.

- Compact motif/config data, reused WebAudio/render objects, small versioned snapshots, and cached descriptors: The rationale is avoiding heavy media, bundle bloat, snapshot bloat, render thrash, and unnecessary tick cost.

- Aggregate RUM, backend metrics, and synthetic checks: The rationale is operational visibility into startup, snapshot delivery, frame timing, audio init failures, unsupported browsers, tick health, ingestion, email, and invite revoke propagation.

- Telemetry privacy boundary: The rationale is privacy/compliance: metrics never include bird names, personality vectors, notebook text, or per-account interaction histories, and aggregation keys are anonymized/sessionless buckets.

### Rollout Plan

- Phase A internal vertical slice: The rationale is to validate account creation, two starter birds, snapshot rendering, basic mood/call pipeline, read-only notebook scaffolding, first-bird startup, simulation tick correctness, and presence measurement honesty.

- Phase B private alpha: The rationale is low-scale tuning with friendly testers for listen-in, offers, settle, captions, reduced motion, notebook sparsity, and weekly drift calibration review.

- Phase C closed beta: The rationale is to introduce visits, session management, export, deletion/recovery, and staged bird-count unlocks only after milestone confidence, audio recognizability, and performance remain healthy.

- Phase D v1 launch: The rationale is guarded self-serve onboarding with kill switches for invite creation, notebook generation, audio descriptor complexity, and bird unlock milestones.

- Day-one instrumentation: The rationale is to track startup speed, tick health, invite reliability, session failures, unsupported browsers, narration toggles, caption usage, and reduced-motion usage without bird-specific behavioral dashboards.
