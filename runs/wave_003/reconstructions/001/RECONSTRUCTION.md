## System-level intent

- Server-side canonical state, with clients as render surfaces. This shows up in "server-side simulation tick", "single canonical aviary per account", "server-as-single-source", and "clients are render surfaces, not state owners." The plan repeats that "canonical state lives only on the server," that the client "never writes personality state," and that there is "no merge, no conflict resolution UI."

- Felt-aliveness without mechanics that make the aviary feel like a game or obligation. The plan bans "achievements, streaks, levels, badges, XP" and "hunger, distress, decay on neglect." The mood model is stochastic because that "makes it feel alive rather than mechanical," and the drift risk says too-fast changes create a "Tamagotchi-like feel" while too-slow changes make "the promise of a living relationship" collapse.

- Notice, do not announce. The plan explicitly names the risk of "'Notice never announce' violations" and treats toasts, badges, and "announcement-style UI" as violations. It also says there is "No 'more birds' announcement surface," uses a "quiet field" instead of a spinner, and keeps field-notebook prose away from "announcement-style" or "gamification-adjacent" wording.

- Naturalist prose as the product voice. The plan repeatedly uses "naturalist field-notebook voice," "lowercase, present-tense," "specific," and "observation." Screen-reader narration is "never a state-list readout," and notebook prose is reviewed so it does not become "generic" or "announcement-style."

- Accessibility is part of the same aviary, not a secondary mode. Reduced motion is "the same aviary" where "only the visual motion register changes." Captions are visible and assistive-readable DOM text, screen-reader narration uses a slow cadence, keyboard navigation covers the aviary and panels, and all user-copy text targets WCAG AA.

- Privacy and data minimization are design boundaries, not only storage details. The account uses a "synthetic" UUID, email is "encrypted at rest" and stored in the only record that holds email, RUM is "anonymous aggregates," and the plan deliberately does not instrument "per-user visit counts," "per-bird interaction history," or "personality vector distributions."

- Presence means attentive watching, not merely an open tab. The presence feature requires visibility, focus, and recent pointer/key activity. The risk section says the activity window should give credit for "watching birds without necessarily moving the mouse" while preventing users who "leave their laptop open" from generating long presence-time.

- Performance is part of the emotional experience. The "Time to first bird visible" budget is tied to a "Felt-aliveness threshold." The edge inlines an initial snapshot to avoid a cold round trip, heavy rendering and audio leave the main thread light, and the loading state is a "quiet field" rather than a spinner or bar.

- Procedural variation should preserve recognizable continuity. Bird calls use species motif libraries, fixed adoption seeds, and personality/mood parameters so "the same bird at different moods sounds recognizably like itself but varied." Mood transitions are stochastic within a probability space for the same reason: life-like variation without arbitrary resets.

## Per-feature whys

### Scope and v1 product surface

- Two starter birds per aviary, up to seven maximum: NOT RECOVERABLE FROM PLAN

- Six bird species in the initial pool: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: The tick exists so the aviary "advances continuously whether or not any client is connected." Its once-per-minute cadence is enough because mood and drift happen over "minutes-to-weeks timescales," and per-aviary jobs keep failures from cascading.

- Personality vector per bird, persisted server-side and never exposed numerically: The plan makes personality server-owned so the client never writes it and conflict is "structurally impossible." Numerically hidden personality keeps the snapshot from exposing boldness, warmth, vocal frequency, and curiosity; only `plumage_saturation` is surfaced as a rendering parameter, "not labeled numerically to the user."

- Mood system: Mood gives fast-timescale life to birds while staying probabilistic. The plan rejects a "deterministic step function" because stochastic mood transitions make the aviary "feel alive rather than mechanical."

- Presence accounting: Presence is a triple conjunction so presence-time means visible, focused, and recently active watching. The calibration gives credit for attentive watching while limiting the extra presence generated when someone walks away from an open laptop.

- Return-greeting: NOT RECOVERABLE FROM PLAN

- Listen-in interaction: Listen-in exists to focus attention on one bird without making the rest disappear. The focused bird ramps up over about two seconds while the others ramp down to an "ambient floor (not zero)."

- Offer interaction: Offers are low-impact interaction events that feed mood, drift, and notebook triggers. Accepted offers nudge mood toward content; presented offers raise boldness even if not accepted; accepted offers raise curiosity; first reactions can generate notebook entries.

- Per-bird offer cooldown of about three minutes: NOT RECOVERABLE FROM PLAN

- Settle gesture: The settle event "ends presence window cleanly" and has "no drift impact," so it closes an interaction without changing long-term personality.

- Five-second undo window on settle: NOT RECOVERABLE FROM PLAN

- Field notebook: The notebook turns notable state changes into "naturalist field-notebook voice" instead of a feed or reward surface. It is rate-limited to about one entry per few days, uses triggers such as unusual moods or first reactions, and avoids "gamification language."

- Aviary scene: The scene combines local time, weather, perch zones, and micro-motion to create ambient life in a single horizontal screen. These elements supply observation context for rendering, captions, narration, and notebook prose.

- Top bar fade: The plan gives the behavior and implementation reason: opacity transition, no JS animation loop, and the fade still applies in reduced motion because "it is opacity, not motion." A broader product rationale is not otherwise stated.

- Email magic-link auth: Magic links avoid password/SSO auth, and the request endpoint always returns 200 so timing does not reveal whether an email exists. Email dispatch is async and rate-limited to prevent enumeration.

- Per-device session tokens and revocable sessions: Sessions let devices be listed and revoked independently. The device label is a "user-agent digest; not a fingerprint."

- Single canonical aviary per account and multi-device sync: The server-authoritative model means both devices read the same record, events are append-only, and clients never need to reconcile or show conflict UI.

- Synthetic UUID account IDs and encrypted email: These choices support the privacy boundary: account IDs are synthetic, and email is stored once, encrypted at rest with a per-account key.

- Account export: NOT RECOVERABLE FROM PLAN

- Soft and hard account deletion: The soft window allows deletion to be cancelled with undelete before the scheduled hard purge.

- Visit invitations: Visit invites provide a read-only ambient view without becoming a social network surface. They are "revocable," "off by default," token-based, and active visitor sessions terminate on the next snapshot pull after revocation.

- Screen-reader narration: Narration makes the aviary available as an observation rather than a state list. The plan explicitly rejects "Pip: content, front perch" output and uses slow naturalist prose instead.

- Reduced-motion mode: Reduced motion is designed as cross-fades and still poses, not "animations-off." The plan says mood, drift, calls, and notebook are identical; only the visual motion register changes.

- Call captioning: Captions describe runtime-generated motifs in naturalist prose, appear near the calling bird, and become the primary way to experience calls when WebAudio is unavailable.

- WCAG AA contrast: Dynamic aviary backgrounds require text, captions, labels, tooltips, and settings surfaces to remain readable; caption text may use a semi-transparent background pill.

- Keyboard navigation: Keyboard support makes top bar controls, bird focus, listen-in, offer selection, settle, and notebook access operable without pointer input.

- WebAudio procedural call synthesis: Procedural synthesis supports variation across species, mood, and personality without shipping recorded audio that would exceed the bundle budget.

- Synthetic performance monitoring and aggregate-only RUM: Monitoring exists to catch performance and reliability issues while preserving the privacy rule that analytics have no per-account or per-bird dimension.

### Architecture

- Rendering Worker: Heavy rendering moves to a Web Worker via OffscreenCanvas so the main thread stays light and presence detection is "never blocked by animation load."

- Audio Worklet: Audio runs in an `AudioWorkletProcessor` so call synthesis, chorus mixing, and listen-in re-balance happen outside the main thread as a single stereo stream.

- App Shell event emitter and snapshot consumer: The app shell reports interactions as events and consumes canonical snapshots because the client renders interpolation but does not own divergent state.

- Edge HTML with initial state snapshot: The edge bundles the snapshot at delivery to avoid a "cold-cache round-trip" and support first paint from a small, acceptable-staleness snapshot.

- Event Log API: The event log is append-only so multiple devices can submit simultaneously without overwriting; the tick later processes events in `occurred_at` order.

- Notification Service as email-only: The service is thin because v1 excludes push and product-event email; it only sends magic links, export links, and visit invitations.

### Data model

- Account record email design: Email is the "only record that holds email" and is encrypted with AES-256-GCM and a key reference, matching the privacy boundary.

- Session token record: The record supports per-device session tracking and revocation without fingerprinting.

- Magic link record: The raw token is never stored; only a SHA-256 hash, expiry, and consumed timestamp remain, which supports expiry and single-use behavior.

- Aviary record: The one-to-one account relationship enforces one canonical aviary in v1; `bird_slots` determines render Z-ordering.

- Bird record stable IDs: NOT RECOVERABLE FROM PLAN

- Bird call grammar seed: A fixed adoption seed shapes motif selection so each bird can vary by mood while remaining recognizably itself.

- Interaction event record: The append-only event record is the journal that lets tick workers process presence, listen-in, offers, settle, and return without client-owned state.

- Notebook entry record: Notebook entries store generated naturalist prose plus a triggering event so observations can be reviewed and displayed independent of live simulation state.

- Visit invite and visit session records: Visit records support expiry, revocation, encrypted visitor email, single-use token hashing, and ephemeral read-only sessions.

- Canonical state snapshot: The snapshot is small, derived state for rendering. It includes mood, perch zone, calling state, and notebook preview, but excludes hidden personality vector values.

### API surface

- `POST /auth/request-link`: Always returning 200 is timing-safe and prevents revealing whether an email exists.

- `POST /auth/consume-link`: Consuming the link marks it consumed and issues the session token, enforcing expiry, single-use, and not-found outcomes.

- `DELETE /auth/session/:token_id`: Revocation gives users device-level session control.

- `GET /aviary/snapshot`: The optional `since` parameter allows 304 when unchanged, keeping keepalive pulls cheap.

- `POST /aviary/events`: A batch-friendly 202 response lets the client queue and flush interaction events for async tick processing.

- Visitor snapshot path: Visitor access uses a visit token rather than an account session and returns revoked/expired errors, preserving read-only revocable visits.

- Notebook entries endpoint: Pagination with most-recent-first supports reading the generated notebook without turning it into live state ownership.

- Account endpoint and settings patch: The account surface exposes masked email, sessions, visit log, and settings needed for account management without exposing raw email.

- Account export endpoint: NOT RECOVERABLE FROM PLAN

- Account deletion and undelete endpoints: Deletion starts the soft window; undelete cancels deletion during that window.

- Visit invitation endpoints: Invite, revoke, and inspect endpoints support email invitation, host control, and revocation visibility.

- Bird naming endpoint: Naming is mutable identity only; validation says it has "no effect on personality, mood, or call."

### Simulation engine design

- Per-aviary tick cadence: Per-aviary scheduling keeps each aviary isolated, prevents cascade failures, and fits the minute-to-week timescales of mood and drift.

- Tick locking: The lock makes each tick apply events and write canonical state consistently for one aviary.

- Tick execution sequence: The sequence converts event log inputs into mood, drift, ambient state, notebook entries, and updated canonical records in one server-owned pass.

- Time-of-day mood baselines: Local-time baselines give birds daily rhythm: "early morning hush," "default daytime," dusk drowsiness, and low-motion night.

- Interaction mood modifiers: Offers, listen-in, chorus, rain, wind, and personality gates make mood respond to recent events and ambient conditions without forcing deterministic outcomes.

- Personality gates in mood transitions: Boldness and curiosity affect transition probabilities so personality shapes behavior rather than staying hidden data.

- Stochastic mood sampling: The plan explicitly uses stochastic sampling because it feels alive rather than mechanical.

- Drift low-pass filter: Drift changes slowly so test instruments see movement after about one week and users perceive it after about three weeks.

- Monotonic-toward-expressive drift: Traits move upward on positive presence and "do not decay on neglect," preserving the no-Tamagotchi boundary.

- Drift inputs and weights: Presence is dominant, listen-in is per-bird and social/vocal, offers are smaller signals, and settle has no drift impact, making attention the main relationship signal.

- Drift calibration harness: The 21-day simulated presence harness verifies weekly measurable and three-week visible thresholds before release.

- Call-grammar runtime: Runtime motif selection combines species seed, personality, mood, and time so calls are varied but still tied to the same bird.

- Chorus: Chorus is emergent from overlapping call windows, and the mixer combines active streams rather than authoring a separate chorus state.

- Notebook trigger evaluation: Triggers are notable observations, and rate limiting keeps notebook entries sparse enough to feel like field notes rather than a feed.

### Sync model

- Snapshot pull pattern: Fresh snapshots are pulled after hidden tabs return, after suspend/wake gaps, and on visible keepalive; initial loads use the inlined snapshot for first-paint optimization.

- Conflict prevention: Personality, mood, and perch state are server-authoritative, and events are append-only, so conflict is structurally avoided.

- Multi-device behavior: Simultaneous devices may be up to one tick out of sync, but they share the same record and both sets of events are processed on the next tick.

- No merge or conflict UI: This follows from server authority; clients never own state that needs reconciliation.

### Frontend rendering pipeline

- Scene layers: The layer stack creates visual depth while staying flat and lightweight; the plan explicitly avoids heavy parallax and per-leaf simulation state.

- Bird layered sprites: Species silhouettes plus a plumage layer let personality-driven saturation affect rendering without showing numeric personality values.

- Mood-animation state machine: Per-species, per-mood pose graphs give each mood a visible behavioral vocabulary, and cross-fades smooth mood changes.

- Perch-zone transitions: Small hop or fly-arc animations make server-side perch changes feel continuous instead of sudden between snapshots.

- Idle micro-motion: Breathing, blinking, and feather ruffles keep birds alive between state changes; animation state advances while hidden so rendering resumes mid-pose, "not reset to frame 0."

- Loading quiet field: The first paint uses the same sky-background layer and no spinner, so transition into the live aviary is a gentle color/brightness shift rather than a composition change.

- Reduced-motion rendering path: Key-pose cross-fades, no fly-arcs, disabled leaf drift, reduced weather, and retained slow day/night color shifts preserve the aviary while reducing motion.

- Top-bar fade implementation: CSS opacity transition avoids a JS animation loop and remains compatible with reduced motion.

### Audio pipeline

- Procedural motif synthesis: Motif parameters are transformed by personality and mood so the call audio reflects bird state without recorded audio assets.

- Chorus mixing: Per-bird gain and a compressor prevent clipping when multiple birds call, including the seven-bird maximum.

- Listen-in re-balance: Gain ramps create a slow focus shift; other birds remain audible at an ambient floor instead of being silenced.

- Call caption generation: Captions are generated when the motif is selected so the text describes the same call being synthesized and can fade in/out with the call duration.

- WebAudio fallback: If audio is unavailable, the designed fallback is silence plus captions, not recorded audio, because recorded audio would exceed the bundle budget.

### Accessibility surfaces

- ARIA screen-reader narration: A polite live region with 30-60 second idle cadence gives observation updates without spamming assistive technology.

- Priority narration for user events: Offer result, settle, and listen-in narration fires immediately because those are user-initiated events.

- Caption DOM rendering: Captions are not painted on canvas so they remain visible and readable by assistive technology.

- Keyboard focus indicators: The focus outline must pass WCAG AA against bright and dark aviary states so keyboard navigation remains visible across the naturalist palette.

- Caption contrast pills: Dynamic backgrounds may require a semi-transparent pill so call captions maintain WCAG AA contrast.

### Performance budgets and observability

- Initial JS bundle under 2MB: The budget is tied to "Mid-tier 4G load time constraint."

- Time to first bird visible under 500ms: The plan labels this a "Felt-aliveness threshold."

- Idle motion at 60fps and flat memory growth: These budgets support "30-min sustained sessions" and are verified in CI.

- Simulation tick p99 under 5s: This is an "Early degradation detection" alarm threshold.

- Snapshot payload under 50KB: This keeps keepalive pulls low-cost.

- Code splitting: Core rendering and audio stay in the critical path; account settings, accessibility settings, visit invitations, and notebook are deferred so the initial experience stays small.

- Compact bird assets: Procedural plumage and compact SVG silhouettes keep six species under the asset budget.

- Initial load optimization: Inlining a small snapshot eliminates a round trip and lets the first bird render before the JS bundle is fully parsed.

- RUM metrics: RUM tracks load, first-bird render, frame timing, audio context success, and tick latency as anonymous aggregate health signals.

- Synthetic monitoring: Headless browsers from multiple geographies test first bird render, audio context success, and snapshot latency on a schedule.

- Deliberately uninstrumented metrics: The plan excludes per-user visit counts, per-bird interaction history, personality vector distributions, and anything that reveals a particular account's bird behavior.

### Rollout, risks, and sequencing

- Drift calibration before launch: Calibration protects against both a Tamagotchi-like too-fast feel and an invisible too-slow relationship.

- Audio quality review: Human listening panels are needed because motif design and smooth envelopes determine whether procedural calls avoid robotic or uncanny artifacts.

- Accessibility audit and closed-beta accessibility cohort: Screen-reader, reduced-motion, keyboard, caption, and narration behavior need direct review by accessibility users.

- Performance audit: Bundle size, first-bird render, and flat memory profile are launch gates because performance is part of felt-aliveness.

- Closed beta size and instrumentation: A 100-500 account beta provides error, render, audio, and event-log signals without per-user analytics.

- Field notebook manual review: Notebook quality is reviewed qualitatively because no automated metric is treated as sufficient for naturalist prose quality.

- Birds-per-aviary ramp: Bird availability is based on aviary age, not visit count, and appears in the existing flow without a "more birds" announcement surface.

- Post-launch monitoring: Tick latency, first-bird render, audio context failure rate, and isolated notebook prose review are watched to catch degradation after release.

- Sync correctness mitigation: Append-only events and `occurred_at` processing keep concurrent devices from overwriting or losing events; deduplication by session token is the possible fix if double-presence becomes a problem.

- Audio escape hatch: If procedural synthesis misses the quality bar, the launch fallback is the designed WebAudio-unavailable path: silence with captions.

- Accessibility regression mitigation: Reduced motion is a named rendering path and narration prose is reviewed so it does not become a state-list output.

- Field notebook voice mitigation: Template review prevents new entries from becoming "generic, announcement-style, or gamification-adjacent."

- Presence-detection mitigation: A 2-3 minute activity window preserves attentive watching but prevents open laptops from generating long false presence.

- Notice-never-announce mitigation: Code review for new UI surfaces checks explicitly for toasts, badges, or announcement-style patterns.

- Magic-link abuse mitigation: Always-200 responses, async email dispatch, and per-email rate limiting prevent account enumeration through timing differences.
