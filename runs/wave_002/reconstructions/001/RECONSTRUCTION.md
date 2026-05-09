## System-level intent

- Keep the product quiet, ambient, and non-coercive. This shows up in the explicit exclusions of "streaks, achievements, scores, levels, badges, visit-frequency counters, green-dot calendars," "Push notifications of any kind," and in the risk named "'Notice, never announce' discipline failures." The plan frames toasts, badges, textual greetings, and return messages such as "you've been gone X days" as violations of the product's "affective contract."

- Make the aviary feel alive without becoming Tamagotchi-like. This shows up in the exclusion of "hunger, distress, happiness meters, negative drift on neglect," the drift risk that traits could change too fast for a "Tamagotchi feel" or too slow for a "screensaver feel," and the calibration target that change should be "instruments-detectable after one week, user-perceptible after three weeks."

- Treat identity and accumulated attention as durable. The plan says `bird_id` is "stable forever," renaming and migration "never changes `bird_id`," traits "never decrease," and `plumage_saturation` is "the visual expression of accumulated attention." The bird identity risk says replacing a bird's personality vector causes "trust damage" that is "irreversible."

- Keep personality state canonical and server-owned. The architecture says "Server owns" personality vectors, mood state, canonical positions, and simulation tick; "Client never" writes personality state, derives personality, or ticks simulation. Sync depends on all clients reading "the same canonical aviary record."

- Prefer append-only, deterministic, conflict-preventing state flow. This appears in the event log being "append-only," the simulation tick reading events "in log order," idempotent tick windows, `personality_version`, and the sync statement that this is "Conflict prevention (not resolution, because conflicts cannot arise)."

- Preserve privacy by minimizing identifiers and keeping telemetry aggregate-only. The account model stores email once, encrypted; other services receive only `account_id`. The scope says interaction data is "never aggregated, never used for ML or third-party purposes." Observability repeats "aggregate only," "No per-bird fields," "No per-account interaction counts," and "Telemetry pipeline reads only from aggregate counters."

- Use naturalist prose as the product voice. The plan repeats "naturalist prose" for notebook entries, screen-reader narration, and call captions. Notebook templates are "lowercase, present-tense," and narration templates match the "field notebook voice."

- Make accessibility part of the primary experience, not a stripped fallback. The scope names "Screen-reader narration," "Reduced-motion mode (cross-fade rendering, not a stripped fallback)," captioning, and keyboard navigation. Reduced-motion still keeps day/night color shift and uses a pose-asset library; screen-reader updates are slow and debounced so they do not flood the queue.

- Bias toward immediate, living rendering instead of mechanical loading. The frontend plan says first render begins from an inline snapshot, the first frame is "mid-motion," particles start "from frame 1," and if a snapshot is unavailable the loading state is a "soft sky gradient" where the word "loading" never appears.

- Keep sharing ambient and revocable rather than social. Visit invitations are "per-email opt-in, read-only ambient, revocable, off by default"; visitor sessions have "No event writes." The out-of-scope list rejects profiles, follows, public feed, discovery, shared aviaries, visit comments, and co-presence visits.

## Per-feature whys

### Scope

- Browser-only: NOT RECOVERABLE FROM PLAN

- Single-user accounts: The plan ties accounts to one encrypted email, one `account_id`, and one canonical aviary, avoiding multi-user simulation or co-presence.

- Magic-link sign-in: The plan specifies short-lived links that expire after 15 minutes and are invalidated immediately on consumption; no password-based login or SSO is included.

- Synthetic UUID account identifiers: The account email is stored once and encrypted; "No other service receives the email; they receive the `account_id` UUID only."

- One canonical aviary per account: This supports the sync model where all clients read from the same canonical aviary record and the server is the sole writer of personality state.

- Two starter birds and growth to seven: The bird-per-aviary ramp is based on "calendar time from `account.created_at`, not visit count"; the pacing is "about the relationship's timeline, not the user's engagement metric."

- Server-side simulation tick: The server owns drift, mood, canonical positions, and notebook entries so clients do not derive personality or create divergent state.

- Client renders state snapshots; never writes personality state: The rationale is canonical sync and conflict prevention: clients render snapshots and interpolate only between known server states.

- Return-greeting: NOT RECOVERABLE FROM PLAN

- Listen-in: It changes the mix by bringing one bird to focus while other birds are "quiet but not silent," and listen-in seconds affect `social_warmth` and `vocal_frequency`.

- Offer: Accepted offers affect `curiosity` and `boldness`, can push mood toward `content` or `curious`, and rapid repeated offers produce "no additional signal."

- Settle: The settle gesture pushes birds toward `drowsy`; keyboard flow includes a five-second undo.

- Presence: Presence pings are the baseline signal for drift, representing confirmed attention without requiring explicit scoring or visit counters.

- Field notebook: The notebook records "noteworthy observation" using naturalist prose from concrete events; it is throttled so a recently-active aviary does not get a daily entry by default.

- Day/night cycle anchored to user local timezone: Time of day influences mood transitions and accessibility narration; the client renders continuous sky color from the user's local time.

- Ambient weather: Rain and wind give mood and call behavior low-frequency variation: rain dampens vocal frequency, wind can push alert or wary depending on personality.

- Multi-device sync: The plan says sync is "a property of the architecture": one canonical server record, append-only events, deterministic tick output, and no client-to-client communication.

- Visit invitations: Visits are per-email, opt-in, read-only, revocable, and off by default, preserving ambient sharing without social network surfaces or event writes.

- Screen-reader narration: Slow, naturalist prose updates make the aviary state available without flooding the screen reader queue; event updates are prompt for user-initiated actions.

- Reduced-motion mode: The plan explicitly says this is "not a stripped fallback"; cross-fades and static pose assets preserve mood and species posture while removing animation and particles.

- Call captioning: Captions translate motif and mood into naturalist prose and become the default path when WebAudio is unavailable.

- Keyboard navigation throughout: The plan provides complete keyboard flow so top-bar actions, bird focus, listen-in, offer, settle, and escape behavior are usable without pointer input.

- Account export: Export is the user's backup for bird identity and personality state and is available from settings at any time.

- Soft-then-hard account deletion: The 30-day window protects against accidental account deletion; personality vectors are retained until hard deletion.

- Privacy rule for interaction data: Interaction data is not aggregated, not used for ML, and not used for third-party purposes; telemetry is kept aggregate-only and isolated from simulation state.

- No native apps: NOT RECOVERABLE FROM PLAN

- No gamification: The plan rejects engagement-count surfaces and repeats "No waitlist, no count surface"; the pacing is relationship timeline rather than engagement metric.

- No Tamagotchi mechanics: The plan avoids hunger, distress, meters, and negative drift on neglect; drift is bounded so traits never decrease.

- No social network surfaces: The plan limits sharing to read-only visit invitations and rejects profiles, follows, feeds, discovery, comments, and shared aviaries.

- No push notifications: The "notice, never announce" principle treats unprompted messages and session-start text as failures of the product's affective contract.

### Architecture

- Service topology: NOT RECOVERABLE FROM PLAN

- Server-owned personality, mood, positions, notebook entries, presence totals, visit records, and simulation tick: This prevents clients from deriving or mutating canonical personality state.

- Client-owned rendered frame, presence detection, audio synthesis, interpolation, and UI state: The client can stay responsive and render between snapshots without owning simulation truth.

- Render pipeline boundary: Interpolation is only between known server states so there is "no client-side physics" that diverges from canonical positions.

### Data model

- Account email encryption: Email is stored once, encrypted, and other services receive only the UUID account identifier.

- Stable `bird_id`: The plan makes bird identity durable through renames, species updates, sync, and migrations.

- `personality_version`: The monotone version detects stale reads after simulation writes.

- Personality seed jitter: Species-keyed defaults with small random jitter make new birds start with species defaults while not being identical.

- Mood states: NOT RECOVERABLE FROM PLAN

- Presence event log schema: Events are append-only inputs to drift, mood, listen-in, settle, and presence accumulation.

- Notebook entries read-only once written: The notebook preserves ordered observations rather than mutable notes.

- Aviary snapshot computed from bird records: The snapshot is render input and avoids persisting duplicate state as a separate table.

- `call_motif_seed`: A deterministic seed derived from personality and mood lets the client vary calls consistently within a session.

- Visit record with hashed visitor email and encrypted typed email: This supports visit log display while keeping the invite token as the one-time link credential.

### API Surface

- Magic-link endpoints: Short expiration and one-time consumption keep authentication temporary and revocable.

- Session revocation endpoint: NOT RECOVERABLE FROM PLAN

- Snapshot endpoint: It returns small canonical state and is edge-cacheable for about 10 seconds, while visibility changes can force freshness with `Cache-Control: no-cache`.

- Event submission endpoint: Append-only event writes preserve log order; small batches allow pending events such as offline presence pings.

- Notebook pagination: The client loads recent entries first and fetches older entries on scroll.

- Account settings endpoint: NOT RECOVERABLE FROM PLAN

- Account export endpoint: The export is emailed asynchronously and later serves as the user's backup.

- Account deletion and recovery endpoints: They implement the soft-deletion window and recovery before hard deletion.

- Invite issue, revoke, list, and visitor snapshot endpoints: Invite validity is checked on every call so revocation takes effect even after a visitor session has started.

- Visitor read-only session: Visitors keep the view fresh through snapshot polling but cannot write events.

### Simulation Engine Design

- Active, inactive, and dormant tick cadence: Recent aviaries tick about once per minute; inactive accounts slow down while still keeping mood advancing for users who have not visited recently.

- Per-account tick isolation: Each tick reads and writes one account's bird state, events, weather, notebook entries, and `last_ticked_at`.

- Idempotent tick window: The event-log watermark prevents duplicate jobs from double-processing the same events or double-applying drift.

- Drift as weighted low-pass filter: The plan uses presence, listen-in, and offers to make personality move slowly enough to avoid "Tamagotchi feel" and fast enough to avoid "screensaver feel."

- Presence accumulation: Each ping represents about 60 seconds of confirmed presence, making attention visible to drift without requiring explicit engagement counters.

- Listen-in drift weighting: Listen-in seconds affect `social_warmth` and `vocal_frequency` more strongly than baseline presence.

- Offer drift weighting: Accepted offers affect `curiosity` and `boldness`; offer proximity affects boldness.

- One-week and three-week drift calibration: The target is measurable change after seven days and visual perceptibility after twenty-one days.

- Non-decreasing traits: Traits never decrease, matching the exclusion of negative drift on neglect.

- `plumage_saturation` as visual expression: It drifts with presence and is the only personality-driven visual appearance surface.

- Calibration test harness: A 21-day simulated scenario must pass in CI so drift remains within the intended band.

- Mood transition state machine: Mood reacts to local time, recent interactions, ambient weather, bird-to-bird contagion, and personality modulation, so the same aviary state is not static.

- Probabilistic transition table in JSON config: Probabilities can be tuned without code changes.

- Persistent mood: Mood is written back and does not reset on tab open, preserving continuity across sessions.

- Client-side call grammar runtime: The server gives a deterministic motif seed while the client handles synthesis, keeping call audio out of the server.

- Server-guided call timing: Expected next-call interval comes from `vocal_frequency`; client jitter prevents identical cadence.

- Notebook generator: It records noteworthy observations only, uses concrete bird names, moods, and events, and "never synthesizes data that didn't happen."

- Notebook throttle: At most one entry about every 48 hours prevents the notebook from becoming a daily engagement mechanism.

### Sync Model

- Sync as architecture property: Sync comes from server-only personality writes, shared canonical reads, append-only events, deterministic tick consumption, and no client-to-client communication.

- Conflict prevention: Append-only events and full-log tick consumption mean conflicting client submissions cannot arise as overwrites.

- Stale snapshot handling: `snapshot_at` and re-polling on visibility change let clients recover freshness.

- Concurrent sessions: Each device has its own session token, so there is no single "active session."

- Overlapping presence deduplication: Simultaneous devices count as one minute of presence per time window to avoid double-counting attention.

### Frontend Rendering Pipeline

- Canvas/SVG layered scene: NOT RECOVERABLE FROM PLAN

- Inline snapshot on load: It removes a round-trip and supports the time-to-first-bird target.

- First frame mid-motion: The aviary begins alive rather than as a static loading sequence.

- Soft sky gradient loading field: The fallback is quiet and avoids a spinner or the word "loading."

- Idle micro-motion: Mood-keyed motion makes each bird's posture and behavior express wary, content, curious, drowsy, or alert state.

- Personality-driven visual tinting: `plumage_saturation` maps accumulated attention into appearance, and no other visual surface directly exposes personality.

- Client-side day/night color cycle: The client computes precise sky color from local time while the server supplies the label for narration.

- Bird position transitions: Ease-in-out arcs avoid teleporting between perches.

- Reduced-motion rendering: Cross-fade pose frames preserve mood and species posture while removing animated motion and weather particles.

- Top-bar fade: NOT RECOVERABLE FROM PLAN

### Audio Pipeline

- Client-side WebAudio synthesis: No audio files are downloaded; the bundle carries compact motif JSON rather than large assets.

- Species call grammar: Hand-authored motif libraries and variation parameters make calls distinct, recognizable, and varied.

- Mood-shaped call variation: Drowsy, alert, and wary moods change tempo, pitch, call length, and silence so calls reflect state.

- Recognizable individual calls: The user should learn to recognize a bird's call across mood and drift states.

- Chorus mixing: Independent streams and per-bird `GainNode`s avoid the stacked-loop artifacts the plan calls out.

- Listen-in mix decay: The focus bird rises slowly while others become quiet but not silent, creating focus without muting the aviary.

- Call captioning: Synthesized calls emit events to prose captions near the calling bird, making motif and mood readable.

- WebAudio graceful silence: If audio fails, captions turn on and a single matter-of-fact notice appears; "Silence + captions is the correct fallback."

- No recorded audio fallback: The plan rejects recorded fallback as a path; when synthesis fails, the rest of the product remains unaffected.

### Accessibility Surfaces

- Screen-reader live region: Slow, debounced prose updates prevent rapid state changes from flooding the queue.

- Priority narration for user events: User-initiated events temporarily get assertive updates, then revert.

- Keyboard bird and top-bar flow: The flow gives full access to notebook, offer, settle, accessibility settings, account, bird focus, listen-in, and escape behavior.

- Focus ring style: The outline is high-contrast and verified against light and dark aviary states.

- WCAG AA contrast: Caption, top-bar, notebook, error, and settings text maintain 4.5:1 contrast across changing day/night backgrounds.

- Bird focus indicators: Keyboard focus is visible around the bird, and reduced-motion mode removes animation from the outline.

### Performance Budgets and Observability

- Initial bundle budget: It is enforced in CI and supported by code-splitting, compact SVGs, motif JSON, and no audio assets.

- Time to first bird visible: Inline snapshot, inline SVG, and deferred WebAudio target first bird visible in under 500ms.

- Idle frame-rate and memory budgets: Long sessions should hold 60fps and show no memory growth trend.

- Simulation tick latency budget: p99 greater than five seconds pages because timely ticks are core to the living aviary state.

- Synthetic monitors: Automated browsers repeatedly check first-bird timing, JS errors, and audio context success or failure.

- Real User Monitoring aggregate-only: The plan measures load timing, frame timing, audio failure, tick latency, session duration histogram, and event submission without per-account or per-bird dimensions.

- Telemetry privacy boundary: The analytics pipeline cannot read simulation database state and rejects deny-listed fields such as `bird_id`, `personality`, `mood`, and simulation `event_type`.

- Alarms: NOT RECOVERABLE FROM PLAN

### Rollout

- Infrastructure hardening: The first phase validates tick scheduler load, drift calibration, snapshot caching, and append-only event writes before users.

- Closed alpha: Internal users check tick latency, drift accuracy, event throughput, and accessibility; bug-fixes only means no new features during this phase.

- Invite beta: The beta includes visits, dashboards, synthetic monitors, notebook prose review, and drift calibration against actual alpha data.

- Open launch: The plan enables all v1 features while keeping "No waitlist, no count surface."

- Bird-per-aviary ramp: New birds unlock by aviary age, not visit count, because pacing is about "the relationship's timeline" rather than engagement.

- Day-one instrumentation: The product measures timing, tick latency, event volume, audio failure, and session duration from launch while avoiding metrics that touch per-account simulation state.

### Risks

- Drift calibration mitigation: CI fixtures, hidden alpha instrumentation, internal diagnostics, and config-file parameters keep drift in the narrow band between Tamagotchi and screensaver.

- Sync correctness mitigation: Watermarks, short CDN TTL, visibility no-cache, no cache for visit sessions, and concurrent tick tests prevent double-drift and stale state.

- Audio uncanniness mitigation: Hand-authored motifs, listening tests, species-specific envelopes, mood tuning, and simultaneous-call checks protect the "feels alive" goal.

- Accessibility regression mitigation: Axe-core scans, synthetic screen-reader playback, pose-asset coverage checks, and narration template tests keep alternate surfaces in sync with displayed state.

- "Notice, never announce" mitigation: Naming the principle, listing failure modes, and requiring product-owner sign-off for session-start visible text prevents quiet-contract violations.

- Bird identity loss mitigation: Audit logging, large-delta alerts, immutable `bird_id`, account export, and soft deletion preserve identity and accumulated personality state.

- Privacy boundary breach mitigation: Deny-listed telemetry fields, network isolation from analytics, and privacy review prevent per-bird or per-account simulation data from entering telemetry.
