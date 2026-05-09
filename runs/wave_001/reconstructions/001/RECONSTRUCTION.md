## System-level intent

- **Constrained web v1 rather than platform spread.** The plan frames the product as a "Browser-based virtual aviary" and repeats the boundary in "Native mobile apps (web-only for v1)" and "Web-only launch."
- **Ambient life over game pressure.** The plan rejects "achievements, streaks, levels, scores, badges" and "Tamagotchi mechanics," while the risks focus on preserving the "alive" feel without becoming a "static screensaver."
- **Gentle, non-punitive change.** The plan says there is "no death, hunger, distress; no negative drift on neglect," and the drift function "does not degrade on absence." Drift should be "perceptible only after roughly three weeks."
- **Server-authored continuity.** The architecture uses an "authoritative server," "canonical state," and an "append-only log." The sync model says the server is the "absolute source of truth" and "clients never write state directly."
- **Client sensory richness inside a server boundary.** The plan uses a "Thick client for rendering and audio synthesis," but the server dictates "where the birds are and what they are doing." The client handles "tweening/interpolation," "procedural micro-motion," rendering weather, and call synthesis.
- **Naturalist voice as an interface layer.** The "Field notebook" contains "auto-generated naturalist observations." Screen-reader narration uses "slow, naturalist prose," and captions share the same "naturalist voice."
- **Accessibility as a designed surface.** The plan groups screen-reader narration, reduced motion, captions, contrast, and keyboard navigation as explicit "Accessibility Surfaces." The risks say accessibility is "a designed surface" whose "narrative flow" must be maintained alongside visual transitions.
- **Privacy and revocability.** The plan uses "synthetic UUIDs exclusively internally," "encrypted email," "explicit, revocable email invitations," and a "Strict privacy boundary" where telemetry never includes "PII, per-bird state, or per-account interaction logs."
- **Performance in service of immediacy and aliveness.** The budgets name "<500ms time to first bird," "60fps," and "no memory growth." Rollout instrumentation monitors tick latency and audio synthesis "to ensure the 'alive' feel holds up."

## Per-feature whys

### Scope

- Browser-based virtual aviary, single horizontal scene, responsive viewport: The plan anchors v1 in a "web-only" scope and later chooses a "single horizontal HTML5 Canvas/WebGL scene" for the scene composition.
- Single-user accounts via email magic link: NOT RECOVERABLE FROM PLAN
- Two starter birds, maximum seven: The rollout says every new account starts with "2 starter birds," with later birds unlocking "purely based on aviary age," which fits the non-goal of no "achievements, streaks, levels, scores, badges."
- Procedural calls: The audio risks say the motif grammar must avoid sounding "robotic" and deliver "organic variation."
- Personality drift: The drift risk says too-fast drift feels like a "Tamagotchi" and too-slow drift feels like a "static screensaver"; the mitigation targets perceptible change after "roughly three weeks."
- Mood transitions: The simulation engine evaluates "recent events, local time of day, weather, and personality vectors" so moods reflect the aviary's current context and write to canonical state.
- Client-side procedural audio with chorus mixing: The plan says procedural synthesis avoids "phase-cancellation artifacts common when looping overlapping recorded audio."
- Return-greeting interaction: NOT RECOVERABLE FROM PLAN
- Listen-in interaction: The audio pipeline says focusing a bird "smoothly ramps up its gain node" while "ducking the ambient and other birds' gain nodes."
- Offer interaction with seed, song fragment, or pool: NOT RECOVERABLE FROM PLAN
- Settle interaction: NOT RECOVERABLE FROM PLAN
- Field notebook: The plan describes "auto-generated naturalist observations" and later a read-only notebook API, so the feature carries the product's naturalist voice into durable entries.
- Multi-device sync: Because state changes are "server-authored," devices "natively stay in sync without client-side merging."
- Read-only ambient visits by explicit, revocable email invitation: "Read-only" visits and "explicit, revocable" invitations provide a social feature while preserving the non-goal of no "profiles, follows, public feeds, leaderboards, mutual visits, or chat."
- Screen-reader narration: ARIA narration receives "slow, naturalist prose" generated from current aviary state, making nonvisual access part of the same product voice.
- Reduced-motion mode: The plan says it uses "cross-fades instead of animations" and later "slow cross-fades between static poses."
- Call captions: Captions are generated from the "procedural call grammar" and are also enabled when WebAudio is unavailable or denied.
- WCAG AA contrast: The plan requires "strict WCAG AA compliance for all text over the aviary and UI chrome."
- Keyboard navigation: The plan requires "Tab navigation through UI and birds" with "high-contrast focus indicators."
- Performance budgets: The plan connects bundle size, time-to-first-bird, frame rate, memory, tick latency, and audio synthesis performance to keeping the aviary immediate and "alive."

### Architecture

- Thick client for rendering and audio synthesis: The client is responsible for sensory detail: interpolation, micro-motion, weather rendering, and procedural calls.
- Authoritative server for persistent state, event log processing, and simulation ticking: The server supplies canonical snapshots and computes resulting state from the event log.
- Auth/Account Service: The plan gives it magic links, session tokens, and identity management, with "synthetic UUIDs exclusively internally" as the stated identity boundary.
- Simulation Service: It runs the "~1 min tick," processes client event logs, applies personality drift, and updates mood.
- API/Sync Service: It serves snapshots and ingests interaction events into an "append-only log," supporting the additive sync model.
- Render pipeline boundary: The server dictates "where the birds are and what they are doing," while the client supplies the presentation layer around those snapshots.

### Data Model

- Account synthetic UUID and encrypted email: These fields support the plan's privacy boundary and internal identity model.
- Account session tokens: NOT RECOVERABLE FROM PLAN
- Account visit log: NOT RECOVERABLE FROM PLAN
- Aviary state with current weather and local timezone anchor: Weather and local time of day are later used by mood transitions.
- Bird stable internal ID: Stable identity supports canonical state and repeated snapshots for the same bird.
- Bird species ID: NOT RECOVERABLE FROM PLAN
- Bird user-assigned name: NOT RECOVERABLE FROM PLAN
- Personality vector values: The drift function updates boldness, social warmth, vocal frequency, plumage saturation, and curiosity from presence-time and interactions; audio also varies pitch and timing by personality.
- Mood state and mood timer: Mood transitions write new moods to canonical state, so the bird needs a current mood and timer.
- Positional/render state: Current perch zone and animation state let the server snapshot "where the birds are and what they are doing" while the client tweens between states.
- Append-only event log: The sync model uses event append instead of direct state writes for "conflict prevention."
- Notebook entries: Text in "naturalist voice," timestamp, and account ID support the auto-generated field notebook.
- Visits with invitation tokens, visitor email, expiration, and revocation status: These fields support "explicit, revocable email invitations."

### API Surface

- `POST /auth/magic-link`: NOT RECOVERABLE FROM PLAN
- `POST /auth/verify`: NOT RECOVERABLE FROM PLAN
- `GET /api/state`: Returns the current aviary snapshot as "lightweight JSON" for birds, moods, positions, and weather.
- `POST /api/events`: Submits interaction events by appending them to the log and handling them asynchronously.
- `GET /api/notebook`: Fetches read-only field notebook entries and paginates them.
- `POST /api/social/invite`: Generates and sends the explicit invitation email used for ambient visits.
- `DELETE /api/social/invite/{id}`: Revokes an invitation, matching the revocable social model.

### Simulation Engine Design

- Periodic tick runtime: The tick processes the event log for each active aviary every "~1 minute."
- Drift function with low-pass filter: The low-pass filter keeps changes gradual, monotonic traits increase from accumulated presence-time and interactions, and traits do "not degrade on absence."
- Presence definition by visibility, window focus, and recent pointer/key activity: The plan defines presence "strictly," and the sync risk calls for exact timestamping of presence windows.
- Mood transitions: Mood changes depend on recent events, local time of day, weather, and personality vectors, then write new moods to canonical state.
- Call-grammar runtime: The server gives mood and vocal frequency, while the client generates "specific motif combinations and timings" from those parameters.

### Sync Model

- Canonical state: The server is the "absolute source of truth."
- Conflict prevention: Clients "never write state directly," only append to the event log, so the server authors deltas.
- Multi-device polling: Devices poll state on visibility change or low-frequency keepalive, and because all changes are server-authored they stay in sync "without client-side merging."

### Frontend Rendering Pipeline

- Scene composition with background, middle, and foreground layers: NOT RECOVERABLE FROM PLAN
- Idle micro-motion: The client adds preening and head-tilts "matching the current mood."
- Smooth tweening between perches: The stated rationale is "No teleporting."
- Reduced-motion rendering: Frame-by-frame animation is swapped for "slow cross-fades between static poses."

### Audio Pipeline

- Procedural call synthesis with WebAudio: Pitch and timing vary by the bird's personality and mood.
- Chorus mixing: Procedural synthesis avoids "phase-cancellation artifacts" from overlapping recorded loops.
- Listen-in mix decay: Focusing one bird ramps up its gain node and ducks ambient and other birds' gain nodes.
- WebAudio fallback: If WebAudio is unavailable or denied, the app falls back "gracefully to silence with captions enabled."
- No recorded audio fallback: NOT RECOVERABLE FROM PLAN

### Accessibility Surfaces

- Screen-reader ARIA live region: It receives "slow, naturalist prose" generated from current aviary state.
- Captions: Text overlays are generated from the procedural call grammar and share the "naturalist voice."
- Focus and keyboard navigation: Tab navigation through UI and birds is paired with "high-contrast focus indicators."
- Contrast: WCAG AA applies to all text over the aviary and UI chrome.

### Performance Budgets and Observability

- Initial JS bundle under 2MB: The plan treats bundle size as a tracked budget and RUM metric.
- Time-to-first-bird under 500ms: The plan gives the mechanism as "edge CDN delivery of initial state/HTML and aggressive code splitting."
- 60fps rendering and strict memory management: The plan calls for reusable WebAudio buffers and clearing detached DOM nodes to prevent memory growth.
- Synthetic checks and RUM: Observability tracks bundle size, render frame times, and simulation tick latency.
- Telemetry privacy boundary: Telemetry excludes PII, per-bird state, and per-account interaction logs.

### Rollout

- V1 web-only launch: This repeats the scope boundary that native mobile apps are out of scope.
- Core 6-species pool: NOT RECOVERABLE FROM PLAN
- Third and subsequent birds unlocking by aviary age: The plan says the unlock is "purely based on aviary age," which keeps bird growth outside achievement, streak, level, score, or badge mechanics.
- Day-one instrumentation for simulation tick latency and audio synthesis performance: The plan says this ensures the "'alive' feel holds up."

### Risks

- Drift calibration mitigation: Low-pass filter weights should make changes perceptible only after "roughly three weeks" so the app is neither a "Tamagotchi" nor a "static screensaver."
- Sync correctness mitigation: A robust append-only event queue and "exact timestamping for presence windows" protect the "illusion of continuous life."
- Audio uncanniness mitigation: Investment in WebAudio nodes and generative grammar is meant to ensure "organic variation."
- Accessibility regression mitigation: Screen-reader prose generation must be maintained alongside visual state transitions because accessibility is treated as "a designed surface" and visual changes can break "narrative flow."
