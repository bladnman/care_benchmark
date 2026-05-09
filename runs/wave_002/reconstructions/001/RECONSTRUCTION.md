## System-level intent

- Server-canonical, single-writer simulation is a core design principle. It appears in "PostgreSQL: single source of truth," "No client-to-client replication," "The client is a rendering layer," "The server is the simulation," and "the simulation service is the only writer." The plan treats sync correctness as an architectural property, not a later reconciliation feature.

- The product should be slow, ambient, and non-demanding. This shows up in the explicit exclusion of "achievements, streaks, levels, badges, scores," "hunger meters," "distress on neglect," and "push notifications," plus phrases like "per-minute updates match the slow rhythm of the product," "quiet field," "No spinner. No loading text," and a new-bird offer that is "not as a modal or announcement."

- Change should come from presence and observation, not punishment. The plan repeatedly says drift is "additive only," "Traits never decrease," and "no negative drift on neglect." The calibration targets require changes to be detectable after a week and perceptible after three weeks, while "A single session should never move any trait by more than 0.005."

- The product voice is naturalist prose over numeric state exposure. This appears in "field notebook: auto-generated naturalist prose," "naturalist voice," "feels like a naturalist observation" versus "feels like a state dump," and narration that "does not expose mood state as a label" but describes "observable behavior."

- Accessibility is a launch feature and an equivalent surface, not a patch. The plan includes screen-reader narration, reduced-motion mode, call captions, WCAG AA contrast, and full keyboard navigation in v1. It says reduced-motion "ships at launch" and is "not a post-launch fix," and captions are scheduled with the same call onset to protect the "audio-equivalence claim."

- Privacy boundaries are part of the architecture. The plan uses synthetic UUIDs "never derived from email," encrypted email plus HMAC lookup, private cache keys that are "never email," aggregate-only RUM, and telemetry that "never includes per-account identifiers, bird names, personality vector values, or interaction event content."

- The experience must feel continuous and alive while staying technically bounded. This shows up in "call_phase" so calls "appear to have been in progress before the user opened the tab," "ambient micro-motion," animation that appears continuous after tab restore, and strict budgets for bundle size, first bird, frame rate, and memory growth.

- Operational recoverability and configurability are intentional. The plan uses "idempotent re-runs," tick watermarks, advisory locks, per-aviary transactions, drift rates in configuration, a `drift_enabled` flag, template libraries that "can be updated without a deploy," and age thresholds stored in configuration.

## Per-feature whys

### Scope

- Browser-based SPA (modern Chrome, Safari, Firefox, Edge - last two major versions): NOT RECOVERABLE FROM PLAN

- Magic-link email authentication; no passwords, no SSO: NOT RECOVERABLE FROM PLAN

- Single account per user; single canonical aviary per account: The plan's why is sync and identity simplicity: one aviary per account, server-canonical state, and "No merge required. No conflict."

- Two starter birds at account creation: NOT RECOVERABLE FROM PLAN

- Aviary grows to a maximum of seven birds: The plan gives the audio rationale that the "7-bird cap is the spec's empirical ceiling" because recognizability degrades above about five birds and seven simultaneous callers must remain "a natural chorus, not a layered loop."

- Six bird species in the launch pool: NOT RECOVERABLE FROM PLAN

- Bird personality vector server-side and never exposed numerically: The plan preserves mystery and avoids stats surfaces by saying "no debug view, no stats panel," exposing only `plumage_saturation` "for visual rendering," and never labeling the number in the UI.

- Mood system persists across sessions and is modulated by server-side tick: The plan's rationale is continuity: the server "runs whether or not any client is connected," and a later phone session reflects the state after the laptop session's tick.

- Server-side simulation tick at about one minute: The tick owns slow canonical advancement; the plan says the keepalive matches tick cadence and "per-minute updates match the slow rhythm of the product."

- Monotonic personality drift: The plan ties this to avoiding Tamagotchi mechanics: "additive only," "no negative drift on neglect," "Traits never decrease," and the risk note warns against accidentally allowing negative drift.

- Return-greeting within about 1-2 seconds of tab open: NOT RECOVERABLE FROM PLAN

- Interactions: listen-in, offer, settle: The plan makes these gentle state inputs. Listen-in affects warmth and vocal frequency, offers affect boldness, curiosity, and mood, and settle pushes toward drowsy with evening lighting and audio ramp-down.

- Field notebook: The rationale is sparse naturalist observation, not status reporting. The beta gate samples entries for "feels like a naturalist observation" versus "feels like a state dump."

- Presence accounting using visibilityState plus focus plus recent pointer/key activity, all three conjunctive: The plan uses presence windows as the "primary drift input," opening only after the conjunction and closing when any condition breaks.

- Multi-device sync as server-canonical state: The plan says there is "no sync problem in the conventional sense" because both devices pull from the same server record.

- Day/night cycle driven by the user's local timezone: The plan uses time of day for both mood weights and palette shifts, so local time shapes the aviary's visible rhythm.

- Ambient weather at rare intervals: Weather feeds mood, narration, rendering, and audio: rain reduces alertness, dampens bird calls, adds soft rain, and gets narrated on start/end.

- Ambient micro-motion: The plan uses micro-motion to make the aviary feel alive without heavy state changes: body sway, feather shimmer, blinking, and a leaf cue when the snapshot is slow.

- Visit invitation feature: The plan's why is bounded sharing without social-network behavior: visitor access is read-only, revocable, off by default, writes no interaction events, and "does not touch presence windows."

- Screen-reader narration: The plan makes narration a naturalist prose equivalent of the same snapshot the renderer uses, not a separate state dump.

- Reduced-motion mode: The rationale is accessible continuity: use cross-fades and overlays instead of animation content, while still preserving the aviary experience and audio.

- Call captions: The plan treats captions as an audio alternative generated from the same call grammar at runtime, with WebAudio fallback auto-enabling captions.

- WCAG AA contrast and full keyboard navigation: The plan makes all user-copy text and all core actions reachable and testable, with contrast encoded in design tokens and keyboard focus paths through the aviary.

- Account export (JSON snapshot on demand): NOT RECOVERABLE FROM PLAN

- Soft-delete with 30-day recovery then hard-delete: The plan's rationale is the recovery window: authenticated requests surface a recovery prompt, and explicit recover cancels deletion before hard delete.

- Performance budgets: The plan treats first bird, bundle size, frame rate, memory, tick latency, and snapshot latency as product constraints and CI or monitoring gates, not advisory numbers.

### Architecture

- API Server as stateless HTTP service: The plan says it is "Horizontally scalable," while keeping simulation state elsewhere.

- Simulation Service as single writer: Its why is canonical behavior and replay safety: it owns the tick, writes personality vectors and mood, and is "Designed for idempotent re-runs."

- PostgreSQL as single source of truth: The plan's rationale is no client-to-client replication and clear writer boundaries between API appends and simulation updates.

- Email Service as transactional email only: The plan uses email for magic links, exports, invites, and the opt-in visit notification, while explicitly excluding marketing email and push-like notification email.

- CDN/Edge serving shell and short TTL snapshots: The plan says the cache handles "burst traffic on sign-in," and the appendix says five-second staleness is acceptable because the simulation tick cadence is about sixty seconds.

- Client as rendering layer: The plan's rationale is that the client renders from snapshots, interpolates for smooth motion, synthesizes audio, and never computes or owns personality state.

- Server as the simulation: The server advances canonical state, processes event log order, and runs whether or not a client is connected.

- Render boundary as the full state snapshot: The plan avoids per-bird property requests; the client derives visual and audio parameters from one aviary snapshot plus local time.

### Data Model

- `accounts`: The plan uses synthetic UUIDs and encrypted/hash email so email is "only for lookup and delivery, never as an identifier."

- `sessions`: Device labels and revocation fields support user-visible active sessions and revoking a session from another device.

- `magic_link_tokens`: Hashed, single-use, expiring tokens keep the token itself only in email and mark use server-side.

- `aviaries`: The unique account relation enforces one aviary per account, and `created_at` supports aviary-age-gated bird additions.

- `birds`: The plan stores mood, perch-relevant state, and the personality JSONB in one canonical record, with the simulation service as the only writer.

- `interaction_events`: Append-only events give the tick a monotonic watermark and prevent API updates from mutating simulation state directly.

- `presence_windows`: These convert active presence into a duration because "Duration is the primary drift input."

- `notebook_entries`: They are generated prose and "Read-only from the client," preserving the notebook as an observation surface rather than an editor.

- `visit_invitations`: Token hashing, revocation, expiry, and visitor email used only for delivery keep visits bounded and revocable.

- `visit_events`: The plan logs them for the host's visit log and explicitly says they are "Not used as drift input," so visitors do not alter the aviary.

- `tick_runs`: Watermarks and status allow resume after restart and "Prevent double-processing events."

### API Surface

- `POST /api/auth/request-link`: The plan returns 204 regardless of account existence to "prevent enumeration" and rate-limits requests per email.

- `GET /api/auth/verify`, `DELETE /api/auth/session`, `GET /api/auth/sessions`, `DELETE /api/auth/sessions/:id`: The plan supports single-use token verification and immediate session/device revocation.

- `GET /api/aviary/state`: The endpoint is the canonical rendering payload. It exposes only the visual `plumage_saturation` scalar and uses a private per-account UUID cache key, "never email."

- `GET /api/visit/:token/state`: The plan gives visitors the same snapshot shape but no write path: no interaction events, no presence windows, and 410 on revocation.

- `POST /api/events`: Batching and flush-on-hide reduce chattiness while preserving an append-only log; the server keeps received time and tolerates limited skew.

- `GET /api/notebook?before=<cursor>&limit=50`: NOT RECOVERABLE FROM PLAN

- `POST /api/invites`, `GET /api/invites`, `DELETE /api/invites/:id`: The why is revocable invitation sharing with a host visit log and revocation visible on the next visitor snapshot pull.

- `GET /api/account`: NOT RECOVERABLE FROM PLAN

- `PATCH /api/account` visit notification toggle: The appendix says this is interpreted as email because it honors the user's opt-in "without introducing a push-notification surface."

- `DELETE /api/account` and `POST /api/account/recover`: The plan supports a 30-day recovery prompt before hard delete.

### Simulation Engine Design

- Tick loop lock, watermark, and ordered event fetch: The plan's rationale is only one tick at a time and ordered consumption without double-application.

- Idempotent tick re-runs: The why is crash recovery; if a tick crashes, the watermark has not advanced and the next tick reprocesses from the same point.

- Drift function inputs: Presence, listen-in, offers, and current personality are the plan's behavioral inputs for slow additive trait change.

- Drift weights and rates as configuration plus `drift_enabled`: The plan makes calibration adjustable without hardcoding and lets test environments disable drift.

- Drift calibration targets: The plan wants one week of regular visits to be instrument-detectable, three weeks to be perceptible, and one session to stay tiny so changes are not Tamagotchi-like.

- Mood transition engine: The plan blends time of day, interactions, ambient events, and personality so mood changes are contextual rather than random labels.

- Seeded weighted random mood draw: The rationale is reproducibility; the draw is deterministic from bird ID and tick timestamp given the same inputs.

- `call_phase` in the state snapshot: The plan uses it so client audio can start mid-cycle and calls "appear to have been in progress before the user opened the tab."

- Notebook entry triggers: The triggers are high-signal observations such as first-greeter flip, extended quiet, mood persistence, plumage threshold, or notable offer response.

- Template library with slot-filling, not free-form LLM generation in v1: The plan uses templates to preserve naturalist voice and allow updates "without a deploy."

- Notebook sparsity gate: The 18-hour gate keeps entries sparse; notable first-greeter flips can bypass it while routine observations cannot.

### Sync Model

- Canonical state propagation: The laptop-to-phone path demonstrates the why: the phone pulls the post-tick server state and no merge is required.

- Conflict prevention: The plan enforces "no last-write-wins" through single writer updates, INSERT-only API events, and watermark processing.

- Clock skew handling: The plan accepts a five-minute tolerance, discards outliers with a log warning, and does not surface skew errors to the user.

- State snapshot freshness: The client refreshes on open, visible, focus, keepalive, and errors because per-second updates would be wasteful and per-minute updates match the product rhythm.

- Visitor sync: Polling is enough because there is no push mechanism; the visitor poll interval is the revocation propagation delay.

### Frontend Rendering Pipeline

- React or Svelte technology choice: NOT RECOVERABLE FROM PLAN

- HTML5 Canvas primary with SVG fallback: The plan uses Canvas for the scene and SVG fallback for systems where Canvas is unavailable.

- Layered scene composition: The back-to-front layers support depth, weather, foreground particles, and captions positioned near calling birds.

- Bird sprite sets under about 50KB per species: The plan ties compact assets to the performance budget and on-demand species loading.

- Mood-keyed bird animation state machine: The plan makes mood observable through behavior: wary birds scan and avoid the front perch, curious birds tilt their heads, drowsy birds fluff and move minimally.

- Idle micro-motion: Subtle sway, shimmer, and blink cues make birds feel continuous while the loop halts when hidden to avoid background work.

- First frame quiet field: The plan avoids spinners and loading text; the quiet field is "the aviary loading" and appears immediately while the snapshot is in flight.

- Perch zone transitions: The plan keeps the bird on screen with a fast arc, while reduced-motion uses a cross-fade.

- Top-bar fade: The bar gets out of the way after inactivity but remains accessible through pointer, key, and focus behavior.

- Settle animation: Lighting, drowsy poses, and audio ramp-down create a gradual evening transition, with a five-second undo window that reverses it.

- Responsive layout: The plan keeps all birds in frame on narrow screens and provides a single-column perch stack as graceful degradation.

### Audio Pipeline

- WebAudio-only pipeline with no downloaded audio files: The plan keeps calls procedural through browser synthesis rather than recorded loops.

- AudioContext lifecycle on first gesture and suspend on hidden: The first gesture satisfies browser autoplay policies, and suspension prevents background audio drain.

- CallSynth motif algorithm: The plan uses motif rules, jitter, pitch variation, and the `vocal_frequency` trait so calls vary and the trait has an audible behavioral output.

- Chorus mixing: Independent `AudioContext.currentTime` scheduling keeps birds from phase-locking and makes the result "a natural chorus, not a layered loop."

- Listen-in mix: Gradual gain ramps let the user focus one bird without hard cuts.

- Weather audio: Filtered pink noise and rain gain changes make rain audible and dampen bird calls by design.

- WebAudio fallback: Silence with captions is the complete fallback so the rest of the product still works.

- Audio memory management: Persistent nodes and oscillator pools exist to satisfy the 30-minute memory-growth test.

### Accessibility Surfaces

- Screen-reader narration cadence: The plan uses a polite live region with slow idle updates and faster event/weather updates.

- Narration prose generation: It shares the naturalist template library and describes observed behavior rather than exposing mood labels.

- Priority bumping: User-initiated event narrations use an assertive region so they are read without waiting for the polite queue.

- Call caption generation: Captions are derived from the same motif sequence and mood that drove synthesis, preserving audio equivalence.

- Keyboard navigation: The plan lays out tab, arrow, enter, and escape paths through the bar, birds, listen-in, offers, and settle.

- Focus indicators: The contrast requirement makes focus visible against both light and dark aviary backgrounds.

- Reduced-motion mode: The plan removes animation content like flight arcs, leaf drift, and particles while preserving palette shifts and audio.

- WCAG AA contrast: The plan encodes contrast in design tokens and tests it in CI across all text surfaces.

### Performance Budgets and Observability

- Budgets table: The plan uses budgets to protect first-bird speed, idle frame rate, memory, tick latency, and snapshot latency.

- Bundle architecture: Lazy loading, on-demand species assets, and keeping only critical modules in the initial bundle support the less-than-2MB budget.

- Time-to-first-bird path: Preload, inlined snapshot fetch, global response handoff, and no critical-path fonts remove extra round trips.

- Synthetic monitoring and RUM: The plan measures first-frame, frame timing, audio initialization, tick latency, and snapshot latency so regressions are visible.

- RUM privacy boundary: Aggregate-only telemetry with rotating anonymous session IDs protects account, bird, personality, and event content.

- Simulation tick SLO and error budget: The five-second tick alarm and snapshot 5xx budget catch degradation before users notice the aviary "running slow."

### Rollout

- Phase 1 internal testing: The plan uses small team aviaries to review drift, audio, accessibility, and the full synthetic budget suite.

- Phase 2 invite-only beta: The plan validates drift distributions, audio "sounds canned" feedback, two-device sync, notebook quality, and screen-reader feedback with external users.

- Phase 3 public launch prerequisites: Launch waits for budgets, drift calibration, audio validation, WCAG/accessibility review, and screen-reader validation.

- Bird-per-aviary ramp: Additions are "age-gated, not interaction-gated," which avoids rewarding visit frequency and surfaces offers as quiet notebook notes.

- Day-1 instrumentation: The plan tracks operational health from launch while saying the instrumentation does not touch per-bird or per-account interaction content.

### Appendix: Defensible calls on ambiguous points

- Aviary-level presence-time: The plan's rationale is that "the user is watching the aviary, not just one bird," and background birds should not be penalized.

- Opt-in visit alert as email notification: The plan says email honors the setting while staying consistent with "the product does not push or ping the user."

- Notebook generation frequency: The 18-hour minimum gap keeps the notebook sparse during active use, with bypass only for high-signal events.

- Minimum call grammar motif library size: The plan specifies at least three motifs per species to "prevent perceptible looping."

- State snapshot cache TTL: The plan accepts up to five seconds of staleness because the simulation tick cadence is about sixty seconds.
