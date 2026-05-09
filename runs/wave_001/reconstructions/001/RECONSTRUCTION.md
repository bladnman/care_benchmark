## System-level intent

- **A constrained v1 whose refusals are part of the product design.** This shows up in the opening split between "In v1" and "Out of v1 (explicitly refused, not deferred)", and again in rollout: "all v1 features ship to all users on day 1" and "There are no feature flags in the v1 product surface" because "principle-violations are too costly".

- **Quiet companionship, not a game loop.** The plan refuses "achievements, streaks, levels, badges, scores", "hunger meters, death, visible distress from neglect", and most notification surfaces. The risk section names the core principle as "Notice, never announce" and treats toasts, badges, and status messages as blocking design violations.

- **Server-canonical simulation with the browser as a sensory surface.** The architecture says "Server owns" personality vectors, mood state, canonical layout, event log, tick scheduling, and notebook entries, while the "Client owns" rendering, interpolation, audio synthesis, presence signal reporting, and UI interactions. The boundary is stated as "everything above is rendering; everything below is simulation".

- **No client-side personality authority.** The plan repeats that "Clients never write personality state", "Clients never compute drift", and "The simulation service is the only consumer that mutates personality vectors". Sync correctness is meant to be prevented by architecture, because "No two writers of personality state exist".

- **Slow, non-punitive change over time.** Personality drift is "monotonic toward expressive, no negative drift"; the drift function clamps deltas with `max(0, delta)` and targets "measurable drift" after about 1 week and "visible drift" after about 3 weeks. The risk section frames too-fast drift as "Tamagotchi feel".

- **Presence means attentive presence, not just an open tab.** Presence accounting uses a "3-signal conjunction" of visibility, focus, and recent pointer/key activity. The tick computes "confirmed presence" from pings, and the risk section says gaming presence only changes "that user's own birds" and should not be punished or policed.

- **Privacy is a hard telemetry boundary.** Scope says "per-account interaction data never aggregated for any purpose beyond that user's simulation". Observability is "aggregate-only", with "no per-account dimension", and the plan deliberately does not instrument per-account drift rates, individual bird mood histories, per-user presence-time, or per-bird offer acceptance rates.

- **Naturalist prose is the product voice for the aviary.** The field notebook uses "naturalist field-notebook prose", screen-reader narration uses "naturalist prose" in "field-notebook voice", and call captions use "a short naturalist phrase". The accessibility settings panel is explicitly separated into a "matter-of-fact voice".

- **Accessibility is a primary aesthetic, not a fallback.** Reduced-motion mode is "a distinct aesthetic, not a stripped fallback"; screen-reader narration must not become state-list text; captions match what was actually played; keyboard flow and WCAG AA contrast are tested in CI.

- **The aviary should feel alive immediately.** The frontend has an "Initial frame 'already in motion' rule" and "MUST NOT show a blank or static scene". Performance budgets center on "Time to first bird: <500ms", "60fps idle motion", ambient micro-motion, day/night shifts, weather, and audio that starts from the snapshot timeline.

## Per-feature whys

### Scope and launch boundaries

- Browser-only web application: NOT RECOVERABLE FROM PLAN
- Single-user accounts: NOT RECOVERABLE FROM PLAN
- Magic-link sign-in only: NOT RECOVERABLE FROM PLAN
- One aviary per account: NOT RECOVERABLE FROM PLAN
- Starts with exactly 2 birds: The launch configuration says starters are selected by the server using a "randomized but balanced draw from the pool"; no extra birds are unlocked at launch.
- Max 7 birds: The audio-uncanniness risk calls the 7-bird cap a "safety valve" because more species would make call recognizability harder to maintain.
- Bird personality vectors persisted and ticked server-side: The plan makes the tick "the only path by which personality vectors are updated" so drift is canonical and not client-written.
- Mood system: Mood is used to shape call grammar, perch positions, motion state, return-greeting, and screen state; it "persists across user sessions" so session start reads the stored mood.
- Mood daily-ish reset cadence: NOT RECOVERABLE FROM PLAN
- Personality drift: Drift is "bounded additive" and "always non-negative" so birds move "toward expressive" without negative drift or neglect punishment.
- Presence accounting: The 3-signal conjunction is used to count "confirmed presence"; anomaly detection catches "impossible ping densities".
- Return-greeting interaction: The greeting is "procedurally varied" and shaped by boldness, mood, time-since-last-session, and absence length so birds respond differently to different returns.
- Listen-in interaction: The plan says listen-in is "mix re-balance, not mute"; the target bird becomes louder while other birds become "ambient-quiet", and "Silence is not an output".
- Offer interaction: Offers feed the simulation as interaction events; offer inputs affect curiosity and secondarily boldness, and `recent_offer_accepted` can shape mood transitions.
- Settle gesture: Settle is a "soft session-end" with warm dimming, lower call volumes, drowsy motion states, and a "5-second undo window" so ending remains optional and reversible.
- Field notebook: Entries are "auto-generated naturalist prose", "rare", and "read-only"; internal triggers are not exposed, keeping the notebook observational rather than user-authored.
- Procedural call synthesis via WebAudio: The plan avoids recorded audio, keeps motif grammar compact, and makes "No two calls" identical through runtime variation.
- Day/night cycle keyed to user's local timezone: Local time drives the sky gradient, mood inputs, dawn/night call activity, and scene descriptions.
- Ambient weather: Rain and wind appear in mood-transition inputs and scene layers, so weather affects both atmosphere and bird state.
- Ambient micro-motion: Leaves, feathers, and idle bird motion support the "already in motion" rule and the 60fps living-scene goal.
- Multi-device sync: Server-canonical state means all devices see "the same birds in the same moods with the same drift history"; there is no client-side merge.
- Visit invitation: Visiting is email-based, opt-in, read-only, and revocable, preserving controlled sharing without social-network surfaces.
- Screen-reader narration: The live region uses naturalist prose on a slow cadence and explicitly prevents high-frequency narration to avoid screen-reader queue overflow.
- Reduced-motion mode: It replaces frame animation with slow cross-fades and disables particles so reduced motion has its own calm visual character.
- Call captioning: Captions are generated at call synthesis time from motif and variation parameters so the text "matches what was actually played".
- WCAG AA contrast: Text, captions, panels, and error surfaces must remain readable across the day/night palette; automated contrast checks enforce this.
- Keyboard navigation for all interactive surfaces: The tab, arrow, Enter, and Escape flow gives keyboard access to the top bar, scene, birds, listen-in, offers, notebook, and settings.
- Account export: NOT RECOVERABLE FROM PLAN
- Soft-deletion: The 30-day recovery window is supported by `deleted_at`, `hard_delete_at`, and a recovery endpoint that cancels deletion after sign-in.
- Privacy per-account interaction data: The plan states interaction data is never aggregated beyond "that user's simulation"; telemetry excludes per-account histories and individual session identifiers.

### Explicit refusals

- Native mobile apps: NOT RECOVERABLE FROM PLAN
- Gamification: The plan refuses achievements, streaks, levels, badges, scores, green-dot calendars, and XP because "Notice, never announce" treats such surfaces as principle violations.
- Tamagotchi mechanics: Hunger meters, death, and visible distress are refused; the drift risk says too-fast change creates "Tamagotchi feel".
- Social network surfaces: Profiles, follows, public discovery, comments, leaderboards, and shared aviaries are refused; the plan provides only revocable, read-only visit invitations.
- Push, email, or in-product notifications: Notifications are refused except the opt-in visit log, preserving the non-announcing product posture.
- Password-based or SSO authentication: NOT RECOVERABLE FROM PLAN
- Multiple aviaries and multi-aviary accounts: NOT RECOVERABLE FROM PLAN
- Customizable scenes or user-controlled perch placement: NOT RECOVERABLE FROM PLAN
- Recorded audio fallback: The fallback path is "graceful silence + captions"; there are no audio files and no recorded-audio fallback.

### Architecture

- Service topology: The plan separates browser rendering, API/application concerns, simulation ticking, and persistent storage so rendering, auth/account/visit/notebook APIs, and simulation mutation have distinct responsibilities.
- API Gateway / Edge with initial state: CDN-delivered HTML includes the initial snapshot so first paint has "zero additional round-trips".
- Stateless application server: It is described as "horizontally scalable"; simulation state mutation is outside it.
- Simulation service as tick worker and event-log consumer: It centralizes drift, mood, call grammar, perch decisions, and notebook triggers in one canonical mutating service.
- Server owns / client owns split: The split prevents clients from becoming simulation authorities while allowing local interpolation, audio synthesis, presence reporting, and UI interaction.
- Simulation service only mutates personality vectors: The application server is "read-only with respect to personality vectors"; this removes competing writers.
- Render pipeline boundary: The client renders, synthesizes, animates, and interpolates; the server reads events, updates traits and moods, advances call timing, decides perches, and writes canonical state.
- Snapshot pull and event-log push only: This boundary keeps the browser from computing drift and keeps the server from owning frame-by-frame rendering.

### Data model and API surface

- Account synthetic ID and encrypted email: The ID is "not derived from email" and email is "stored once here only", matching the privacy boundary.
- Session device labels and revocation: Sessions have user-readable labels such as "Chrome on MacBook" and can be listed or revoked.
- MagicLinkToken hashing, expiry, and consumption: Tokens are high-entropy, stored hashed, expire after 15 minutes, and are invalidated on consumption.
- Bird stable ID and renameable name: The ID is "stable; never replaced"; the name is user-assigned and renameable but has "no effect on engine".
- Personality vector exposure: Snapshot responses return only `plumage_saturation`; other personality fields are "NEVER returned".
- Append-only interaction events: Events are never updated or deleted, and the simulation tick reads them "in chronological order".
- Client-generated event UUIDs: Retries are idempotent when network failures occur.
- Notebook trigger labels: The trigger is an internal label "not exposed", keeping notebook logic hidden from the user surface.
- Invite token, expiration, and revocation: Visit links expire, can be revoked, and return `410 Gone` if revoked or expired.
- Snapshot response short TTL and inline initial snapshot: The short TTL keeps snapshots fresh while the inline snapshot removes the first extra request.
- Visitor endpoint read-only behavior: Visitor sessions consume the same snapshot shape but "do NOT write presence events" or interaction events.
- Account delete recovery endpoint: Deletion can be canceled if the user signs in during the 30-day recovery window.

### Simulation engine

- Tick cadence with jitter: The ~60s tick uses jitter "to prevent thundering-herd on multi-account servers".
- Per-aviary serial execution and parallelism across aviaries: Serial execution protects each aviary's canonical state; parallelism across aviaries provides scale.
- Consuming events since `last_tick_at`: Older events are ignored to avoid reprocessing already-applied inputs.
- Presence-time computation: A presence ping represents seconds of confirmed presence, giving drift a time-based input.
- Weighted additive drift inputs: Presence, listen-in duration, offers, and settle events contribute weighted deltas so specific interactions shape specific traits.
- Non-negative drift invariant: `trait_new` uses `max(0, delta)` so personality only moves toward expressiveness.
- Drift calibration targets and CI tests: Tests simulate 7 and 21 days to enforce measurable and visible drift targets from day one.
- Plumage saturation from total presence-time: It has "no decay" and is the only personality-linked visual trait exposed in snapshots.
- Mood transition inputs: Time of day, accepted offers, bird-to-bird alarm, rain, wind, boldness, and social warmth all shape probabilistic transitions.
- Mood persistence: Mood is not reset at tick start and persists across user sessions, preserving continuity.
- Call motif libraries and weights: Species motif libraries are weighted by vocal frequency, current mood, and time of day to vary frequency and energy.
- Bird-to-bird response calls: A high-social-warmth call can trigger a neighbor response within 2-10 seconds, producing social chorus behavior.
- Perch and motion decisions: Boldness, mood, and time of day determine where birds sit and whether they are preening, scanning, still, alert, or calling.

### Sync model and visiting

- Single canonical aviary record: It is "the source of truth for all clients".
- No device-state merge: Devices only write events; because there are "no device states to merge", sync avoids conflict resolution.
- Additive deltas only: If concurrent ticks happen due to a bug, additive deltas commute, making the result a sum rather than a winner.
- Snapshot freshness triggers: Clients refresh after tab visibility returns, after wake-like render gaps, and on a visible keepalive.
- Visitor sync: A visitor receives owner-like snapshots but cannot write events, so visits do not affect presence or simulation state.

### Frontend rendering pipeline

- React, Canvas/SVG, Vite, and lightweight state: React owns UI chrome; Canvas is preferred for 60fps scene performance; Vite code-splits; full Redux is rejected because the product is "this size".
- Scene layers: Sky, foliage, weather, perches, birds, ambient particles, foreground branches, and top bar provide time-of-day, weather, depth, motion, and quiet UI chrome.
- Idle micro-motion: Motion states run continuously, with rendering halted only when the tab is hidden and the server continues state advancement.
- Animation cross-fades: Snapshot state changes blend over about 200ms so motion changes do not snap.
- Initial frame already in motion: Rendering starts at the animation frame implied by `snapshot_at`, avoiding a blank or static load.
- Quiet-field loading state: If no inlined snapshot exists, the fallback is a soft sky and slow leaf drift, "no spinner".
- Listen-in mix transition: Exponential 800ms gain ramps focus attention on one bird without muting others.
- Settle rendering: Lighting warms and dims, calls ramp down, birds settle, and any click during 5 seconds reverses at the same rate.
- Reduced-motion rendering: Cross-fades, disabled particles, slower day/night color shifts, and retained audio make the mode a calm alternate aesthetic.
- Responsive layout: Perch zones compress or widen while bird size and aspect ratio prevent cropping at mobile and wide desktop widths.

### Audio pipeline

- WebAudio architecture with compact grammar data: There are no downloaded audio files; motif parameters are compact JSON under 100KB.
- Motif selection and variation sampling: Seeded variation keeps calls similar to the motif but "similar-but-never-identical".
- Chorus mixing with a compressor: Dedicated bird gain nodes and a master compressor prevent clipping in dense chorus moments.
- WebAudio node pool: Pre-allocation and reuse support the "no memory growth over 30 minutes" budget.
- Autoplay or unavailable-audio fallback: Captions are enabled automatically and no audio plays; the complete fallback is "graceful silence + captions".
- Caption generation from motif parameters: Captions appear near the calling bird and describe the synthesized call that actually occurred.

### Accessibility surfaces

- Screen-reader live region: A polite live region updates with naturalist prose at idle and within 1 second for user-initiated events.
- No high-frequency narration: Updates are limited to no more than once per 15 seconds to prevent screen-reader queue overflow.
- Keyboard flow and focus indicators: The plan defines tab order, arrow navigation between birds, Enter for listen-in, Escape exits, and high-contrast focus visible in morning and night states.
- Captions setting persistence: Captions are reachable from accessibility settings and persist server-side.
- WCAG AA validation: Contrast is checked against the full day/night palette range in CI.
- Accessibility settings panel voice: Settings use "matter-of-fact voice", separating controls from the naturalist field-notebook voice.

### Performance budgets and observability

- Initial bundle budget under 2MB gzipped: CI fails if the budget is exceeded, protecting first paint.
- Code-split settings, visits, and notebook panels: Non-critical surfaces are lazy-loaded and prefetched on idle after first paint.
- Time to first bird under 500ms: Inline snapshots, critical CSS, DOMContentLoaded rendering, system fonts, and no audio dependency make the first bird visible before async work.
- 60fps idle motion: Canvas, Worker particle generation, no layout thrash, and hidden-tab halt target 16ms frames on a 5-year-old mid-range laptop.
- No memory growth over 30 minutes: CI snapshots heap growth and validates bounded pools, buffers, virtual scroll, and flushed event queues.
- Synthetic monitoring: Automated browsers from 3 geographies check first bird timing, idle frame rate, audio context creation, and snapshot latency.
- RUM aggregate-only metrics: Performance is observed by geography and device class only, preserving the privacy boundary.
- Error budgets: p99 simulation-tick latency, snapshot fetch p99, and first-bird p95 alarms define operational thresholds.
- Deliberately not instrumented measures: The plan avoids telemetry for per-account drift, individual moods, presence-time, and offer acceptance so private simulation data is not read by telemetry.

### Rollout and risks

- New-bird offers by aviary age: Later birds arrive at long age thresholds and the constants are adjustable without deploy.
- No feature flags in product surface: Partial rollouts are refused because design-philosophy-dependent feature decisions are too costly to split.
- Ramp plan: Closed beta, invite beta, and open registration are paced by "operational comfort, not by feature gates".
- Post-launch drift calibration: Aggregate p50 and p95 drift velocity are reviewed to tune `global_speed_constant` without deploy.
- Presence-time activity window review: Presence-event density is reviewed to tune how long pointer/key activity counts as presence.
- Notebook entry frequency review: Entry probability is adjusted if entries feel too frequent or too sparse.
- Call cadence review: The vocal-frequency-to-call-interval mapping is adjusted if it does not feel right.
- Drift calibration mitigation: CI simulations, server-side config, 30-day aggregate review, and realistic presence distributions prevent drift from moving too fast or too slow.
- Sync correctness mitigation: The absence of a personality endpoint, ordered append-only events, integration tests, and transaction rollback protect canonical state.
- Audio uncanniness mitigation: Sound design is grounded in real bird call families, calibrated by ear, and user-tested for "recognizable across moods".
- Accessibility regression mitigation: axe-core, manual screen-reader testing, live-region tests, reduced-motion tests, and keyboard-navigation coverage run through CI and release checks.
- "Notice, never announce" mitigation: Code review blocks textual announcements, toasts, badges, and streak surfaces; design review approves all visible UI surfaces.
- Presence signal gaming mitigation: Anomaly detection and a soft cap prevent absurd accumulation, while the plan explicitly avoids punishment or policing.
- Slow-connection performance mitigation: CDN-inlined snapshots, bundle gates, synthetic monitoring, and RUM alarms protect the first-bird target in the field.
