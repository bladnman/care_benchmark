## System-level intent

- "Feels alive, not robotic" is the lead principle for v1. It shows up in the scope line itself and is carried through "birds appear mid-action," "continuous micro-motion," "real-time per-call variation," "never identical recordings," and chorus mixing that should "blend naturally, not layered."
- The product should remain intimate and small. The plan starts with "Two birds per aviary," keeps a "7bird cap never exceeded," and later says to "Maintain sense of intimacy." Starter birds are "arrivals, not choices."
- The aviary is canonical, slow, and server-authored. The plan repeats "server owns all simulation state," "single source of truth," "server-side canonical state," "single-writer guarantees," and "No Last-Write-Wins." This is meant to make drift and sync consistent rather than device-dependent.
- Growth is additive and non-punitive. The drift function is "drift additive, no negative drift" and "Monotonic toward expressive: traits never decrease on neglect." This aligns with the explicit exclusion of Tamagotchi-style mechanics such as "bird death, hunger, distress meters."
- Social surfaces are quiet, opt-in, and read-only. Visit invitations are "quiet opt-in read-only viewing," visitor sessions see the host state, and the notification surface has "Matter-of-fact error messages only" that "never announces presence."
- Accessibility is part of the aviary, not an alternate product. Screen-reader narration uses "naturalist prose," reduced motion is "cross-fade rendering, not animations off," captions are prose descriptions of calls, and the phase gate says accessibility surfaces must be "fully functional (not checklist)."
- Privacy and telemetry are deliberately separated from simulation state. The plan uses "Synthetic UUID (never email)," encrypted email, telemetry that is "Aggregate-only," "no per-account state," and a privacy boundary where telemetry pipelines "never access simulation database."
- Performance is a product constraint, not a late optimization. The plan gives explicit budgets for bundle size, time-to-first-bird, fps, memory growth, WebAudio recovery, and simulation-tick p99, and says "Bundle size drives all asset decisions downstream."

## Per-feature whys

### Scope

- Two birds per aviary: The rationale is intimacy and constraint. The plan says v1 starts with two birds, later calls this the "inverse rule from day one," and says to "Maintain sense of intimacy" while never exceeding the cap.
- Single-user accounts with magic-link sign-in: The rationale is an "email-only" onboarding path with "Immediate aviary experience (no setup required)" and no shared or multi-user aviaries.
- Multi-device sync: The rationale is that devices should see the "same canonical aviary" by pulling the same server snapshots.
- Server-side simulation tick: The rationale is that the server "owns all simulation state" and advances canonical state on a slow tick.
- Personality vector system with 5 traits: The rationale is drift and expressive variation; traits drive mood, calls, boldness, social warmth, vocal frequency, plumage saturation, and curiosity while remaining "stored server-side, never exposed."
- Mood system: The rationale is fast-timescale variation. Moods respond to time-of-day, ambient events, recent interactions, nearby bird calls, and personality bias.
- Procedural call synthesis with chorus mixing: The rationale is variation and naturalism: "never identical recordings" and overlapping calls that "blend naturally, not layered."
- Listeners-in: The rationale is to make focused birds perceptually and behaviorally responsive; listen-in elevates one bird in the audio mix and gives a social warmth/vocal frequency boost.
- Offer interactions: The rationale is expressive drift and mood response; offer acceptance drives curiosity, offering near a bird drives boldness, and acceptance nudges mood toward content.
- Return-greeting: The rationale is that "one bird notices user on session return," supporting the plan's "feels alive, not robotic" principle.
- Settle gesture: The rationale is a soft close to a session, expressed as a "soft session-end lighting transition" and a slow shift to evening.
- Field notebook: The rationale is "human-consumable prose" in a naturalist voice, with auto-generated observations and timestamped notebook entries.
- Visit invitations: The rationale is "quiet opt-in read-only viewing" rather than a social network surface.
- Screen-reader narration: The rationale is accessible presentation in "naturalist prose" with "slow cadence" and priority for user-initiated events while staying prose.
- Reduced-motion mode: The rationale is "Same aviary, different rendering intensity," using cross-fades instead of turning animations off.
- Call captions: The rationale is prose access to calls and an audio fallback path; captions describe calls and are on by default if WebAudio is unavailable.
- Presence accounting: The rationale is "honest drift" through a "precise 3-signal definition" and pings only when all three signals are true.
- Keyboard navigation: The rationale is "full accessibility compliance" with keyboard access to birds, listen-in exit, and offers.
- WebAudio with graceful fallback: The rationale is to preserve the aviary when audio is unavailable: silence with captions and visual aviary only.
- Prior-day bird species pool (~6 species): NOT RECOVERABLE FROM PLAN
- User name assignment per bird: NOT RECOVERABLE FROM PLAN

### Architecture

- Monolithic simulation service with single-writer architecture: The rationale is consistency: "single-writer guarantees," ordered event application, and no conflict resolution for personality state.
- Separate authentication service: The rationale given is the "magic-link workflow."
- Content delivery network for static assets: The rationale is fast delivery of bundle, bird assets, and calls under the bundle and time-to-first-bird budgets.
- Database with account and per-bird state separated from telemetry: The rationale is the privacy boundary: telemetry pipelines never access the simulation database.
- Real-time WebSocket for event streaming: The rationale is streaming "presence, interactions" into the client and state management layer.
- CDN for state snapshots with edge caching: The rationale is rapid loading from edge snapshots and a small state payload.
- Server owns all simulation state: The rationale is canonical state for personality vectors, mood, and drift history.
- Clients pull small state snapshots and render interpolated motion: The rationale is small payloads in kilobytes and fast rendering while the server remains authoritative.
- Clients send append-only interaction events: The rationale is ordered consistency and no need for conflict resolution.
- Client-side procedural call synthesis: The rationale is that it is "CPU-bound" and provides "per-user variation."
- Visual bird assets generated/client-rendered: The rationale stated is "no external dependencies."
- Accessibility surfaces co-rendered with visual aviary: The rationale is that narration and captions come from the same state as the visual aviary.

### Data model and API surface

- Synthetic UUID: The rationale is privacy; the account identifier is "never email."
- Email address encrypted, single write: The rationale is privacy and limiting email exposure.
- Revocable per-device session tokens: The rationale is device-level session control.
- Visit log: NOT RECOVERABLE FROM PLAN
- Settings for notification toggles and reduced-motion preference: The rationale is user preference for notification and rendering intensity.
- Soft deletion for 30 days then hard deletion: NOT RECOVERABLE FROM PLAN
- Stable internal bird ID: The rationale is stable identity, "immutable across all changes."
- Species classification from 6-v1 pool: NOT RECOVERABLE FROM PLAN
- Personality vector stored server-side and never exposed: The rationale is canonical hidden state for drift and behavior.
- Current mood as fast-timescale state: The rationale is short-term behavior that can change with time, ambient events, interactions, and session continuity.
- Per-bird event log: The rationale is to record offers, listen-ins, and presence timestamps used by drift and state updates.
- Per-bird positions: NOT RECOVERABLE FROM PLAN
- Call timing parameters: The rationale is that pitch and rhythm are based on vocal frequency.
- Weather effects: The rationale is ambient influence: rain dampens calls and wind affects alertness.
- Notebook entries: The rationale is timestamped "naturalist prose."
- Append-only event log: The rationale is log-structured consistency for interactions, presence pings, and tick events.
- Presence pings only from host devices when all 3 signals are true: The rationale is "honest drift."
- Tick events: NOT RECOVERABLE FROM PLAN
- `GET /api/aviary/state`: The rationale is a small current state snapshot, under 50KB, for rapid loading.
- `POST /api/events`: The rationale is appending interaction events with "single-writer guarantees."
- Visit create, validate, and revoke endpoints: The rationale is host-controlled read-only access through invitations that can be revoked.
- Magic-link endpoints: The rationale is the email-only sign-in path.
- `GET /account/export`: The rationale is a JSON download of the user's full aviary state.
- Notification surface: The rationale is quietness and privacy: error messages are "Matter-of-fact" and "never announces presence."

### Simulation engine design

- Server-side tick: The rationale is to advance canonical state about once per minute from recent ordered events.
- Reading recent event log in order: The rationale is consistent application of events.
- Updating personality vectors with additive drift: The rationale is expressive change without negative drift.
- Mood transitions: The rationale is behavior that reflects time-of-day, ambient events, recent interactions, bird-to-bird effects, and personality bias.
- Advancing call timing parameters: The rationale is connecting the vocal frequency trait to calls.
- Immutable append to state history: NOT RECOVERABLE FROM PLAN
- Presence-time as dominant drift input: The rationale is that presence, defined precisely, should be the main driver of drift.
- Listen-in drift: The rationale is that a focused bird gains social warmth and vocal frequency.
- Offer drift: The rationale is that acceptance drives curiosity and proximity drives boldness.
- Monotonic toward expressive: The rationale is that traits "never decrease on neglect."
- Drift calibration: The rationale is slow visible change: measurable after one week and visible after about three weeks.
- Time-of-day mood reset: The rationale is natural daily rhythm, with drowsy near dusk and alert mornings.
- Offer acceptance mood effect: The rationale is that accepted offers nudge a bird toward content.
- Bird-to-bird mood effect: The rationale is that nearby calls can shift other birds toward wary.
- Personality bias in mood: The rationale is that high-boldness birds resist wary transitions.
- Mood persistence: The rationale is continuity from session end to next session start.
- Species motifs in call grammar: The rationale is a species library for procedural synthesis.
- Timing shaped by vocal frequency: The rationale is that personality traits shape call timing.
- Pitch modulated by mood and personality: The rationale is that calls express current mood and personality.
- Real-time per-call variation: The rationale is "never identical recordings."
- Chorus mixing: The rationale is overlapping calls that "blend naturally, not layered."

### Sync model

- Single source of truth: The rationale is server-side canonical state.
- No client-to-client syncing: The rationale is that users sign in on devices and both pull the same snapshots.
- No personality state to sync: The rationale is that "server owns it."
- Event log consistency across devices: The rationale is ordered application of interactions.
- No Last-Write-Wins: The rationale is that personality updates are server-authored additive deltas and no device can overwrite another's drift history.
- Magic-link replay handled by invitation token validation: NOT RECOVERABLE FROM PLAN
- No conflict resolution for personality state: The rationale is that server-side consistency eliminates that failure mode.
- Visitor sessions see host's exact state: The rationale is read-only visiting with stale snapshots acceptable.

### Frontend rendering pipeline

- Three-perch zone system: The rationale is to let birds choose perches based on mood and personality.
- Local-time day/night cycle: The rationale is that local time affects lighting, calls, and moods.
- Ambient weather effects: The rationale is rain dampening calls and wind alertness.
- Subtle parallax: The rationale is foreground/background separation.
- Continuous micro-motion: The rationale is birds that feel alive through preening, scanning, and head-tilting.
- Mood-shaped idle motion: The rationale is that wary, content, and curious birds move differently.
- No pausing when tab not focused: The rationale is that "simulation continues."
- Birds appear mid-action on load: The rationale is avoiding an artificial entry animation.
- Listen-in transition: The rationale is gradual mix elevation over one to two seconds.
- Offer reaction: The rationale is a smooth bird approach/animation.
- Settle transition: The rationale is a slow lighting shift to evening over several seconds.
- Reduced-motion cross-fade rendering: The rationale is lower motion while keeping the same aviary.
- Flight transitions as perch-to-perch cross-fades: The rationale is reduced rendering intensity.
- Ambient leaf drift removed and color shifts slowed: The rationale is reduced motion without removing the aviary.
- Rendering engine using WebGL/2D canvas: NOT RECOVERABLE FROM PLAN
- UI layer using React/Vue: NOT RECOVERABLE FROM PLAN
- State management using Redux/Redux-Saga with WebSocket updates: NOT RECOVERABLE FROM PLAN

### Audio pipeline

- Client-side WebAudio nodes: The rationale is client-side procedural synthesis.
- Motif library per species: The rationale is species-based procedural calls.
- Real-time pitch/timing variation: The rationale is personality-driven variation.
- Mood-based timbre changes: The rationale is that calls reflect mood.
- Listen-in mix elevation: The rationale is that the focused bird rises gradually while other birds remain ambient.
- Other birds drop to ambient, not silenced: The rationale is preserving the chorus instead of muting the aviary.
- Quiet power-down when tab not visible: The rationale is avoiding hard cuts.
- Gradual return when tab visible again: The rationale is immersion, specifically "No hard cut-downs that break immersion."
- WebAudio fallback with captions on by default: The rationale is accessible operation when WebAudio is unavailable.
- No recorded audio fallback: The rationale is to preserve the "variation property."
- Initial load under 2MB gzipped: The rationale is the bundle size budget.
- Time-to-first-bird under 500ms: The rationale is quick first visible aviary on mid-tier mobile over 4G.
- 60fps idle motion on a 5-year-old laptop: The rationale is smooth idle motion on older hardware.
- No memory growth over 30-minute sessions: The rationale is stable long sessions.

### Performance budgets and observability

- Bundle size budget: The rationale is staying under the 2MB ceiling; procedural audio avoids recorded files, assets stay generated or small, and rare surfaces are code split.
- Time-to-first-bird budget: The rationale is rapid loading, with CDN edge snapshots and drawing the first bird before non-critical assets.
- Runtime budgets: The rationale is stable performance: 60fps idle, no memory growth, WebAudio recovery, and simulation-tick p99 alarms.
- WebAudio context restart within 2 seconds: The rationale is recovery from context loss.
- Simulation-tick p99 under 5 seconds: The rationale is an alarmed backend performance threshold.
- Aggregate-only telemetry: The rationale is "no per-account state."
- Synthetic performance checks: The rationale is distributed browser performance monitoring.
- Real User Monitoring: The rationale is anonymized session durations and error rates.
- Client-side render-frame timings and audio-context errors: The rationale is observing render and audio failure surfaces.
- Bundle size measurement per deployment: The rationale is enforcing the bundle budget continuously.
- Per-account interaction events never leave user's own simulation: The rationale is the privacy boundary.
- Telemetry pipelines never access simulation database: The rationale is separation of telemetry from simulation state.
- Analytics warehouse separate from production systems: The rationale is privacy and production separation.
- Account export includes full aviary state: The rationale is user access to a complete JSON state export.

### Rollout

- Limited beta of 1000 users: The rationale is launching with instrumentation before broader release.
- Performance monitoring second-by-second: The rationale is close tracking during launch.
- Error rates per feature surface: The rationale is feature-surface reliability tracking.
- User feedback on affordability and charm: The rationale is learning whether the product feels affordable and charming.
- Bird cap ramp: The rationale is to start with two birds, offer a third based on aviary age after three to four months, and "Maintain sense of intimacy."
- Instrumentation for session duration, presence-time, audio-play counts, adoption patterns, species preferences, abandonment, settlement, and accessibility uptake: The rationale is launch learning across behavior, charm, and accessibility surfaces.
- Magic-link onboarding: The rationale is "email-only."
- Immediate aviary experience: The rationale is "no setup required."
- Two starter birds as arrivals, not choices: The rationale is preserving the immediate aviary experience and product voice.
- Phase gate performance budgets: The rationale is that bundle size, first bird, and fps must be met before launch.
- Phase gate accessibility surfaces: The rationale is that accessibility must be fully functional, "not checklist."
- Phase gate non-goals: The rationale is strict enforcement of non-goals, including zero gamification.
