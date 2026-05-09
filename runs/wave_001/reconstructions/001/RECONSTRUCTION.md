## System-level intent

- Server-owned canonical state and conflict prevention. This shows up in "Server owns all state," "Client writes events only," "Simulation tick is server-only," and the sync claim that "there is only one state to see." The plan prefers "conflict prevention (not conflict resolution)" through append-only events and clients that never write personality state.

- Slow, monotonic, non-punitive relationship over weeks. The plan says drift is slow enough that "user perceives change after ~3 weeks," requires the "Monotonic constraint" that "Traits never decrease," and lists "no hunger, distress, or death; no decay on neglect" as hard non-goals. The product should "feel alive over weeks," not act like a fast counter.

- Calm, non-gamified, no-pressure product posture. The hard non-goals reject "streaks, achievements, badges, scores, green-dot calendar, visit counters" and "push notifications." Bird-count ramping appears only in account settings, "not as a popup, not as a streak reward," with "no pressure, no expiration countdown."

- Naturalist voice instead of technical or announcement framing. The field notebook is "sparse naturalist prose," entries cannot reference trait values or event counts numerically, captions use the "same naturalist voice as the notebook," and screen-reader narration must be "lowercase, present-tense, specific" and never use "announcement framing."

- Privacy boundary by architecture, not just policy. This shows up in encrypted email plus non-reversible hash lookup, "pv_* fields are never returned," "Aggregate operational telemetry only," "zero per-account data in analytics pipeline," and the rule that the telemetry pipeline is "architecturally forbidden from reading the Aviary DB."

- Accessibility as a v1 surface, not a later patch. The plan includes screen-reader narration, reduced-motion mode, call captions, WCAG AA contrast, keyboard navigation, and launch checklist testing on VoiceOver and NVDA.

- "Already in motion" and smooth presence as product feel. The first-frame strategy targets birds "mid-action within 500ms," bans a loading spinner on that path, separates rendering from network jank, and budgets for "60fps idle at 30 minutes."

- Gentle, opt-in, read-only social access. Social is limited to "one-to-one visit invitations," "off by default," email-based, revocable, with no profiles, follows, public discovery, comments, leaderboards, or visitor event writes.

## Per-feature whys

### 1. Scope

- Single-user accounts; email magic-link sign-in; no passwords, no SSO: NOT RECOVERABLE FROM PLAN
- One aviary per account; two starter birds at account creation, cap of seven: NOT RECOVERABLE FROM PLAN
- Five personality traits per bird: NOT RECOVERABLE FROM PLAN
- Server-side simulation tick: The plan keeps drift, mood transitions, ambient weather, notebook generation, and canonical state on the server so "No tick logic runs on the client" and the client "never simulates drift or mood transitions."
- Multi-device sync: The plan makes this an architectural property of "the canonical server state" so both devices "see the same state because there is only one state to see."
- Client interactions: The interactions are written as events, not state, so they can feed the tick through the append-only event log while remaining "fire-and-forget" from the client.
- Presence accounting: The "triple-condition signal" exists because over-reporting presence would "inflate drift across all accounts"; the plan wants visibility, focus, and recent activity all satisfied.
- Procedural audio: The plan uses client-side WebAudio because "No recorded audio files are shipped," no audio files are downloaded, and motif iteration is cheap because the motif library is code.
- Aviary visual scene as a single horizontal scene with three perch zones, day/night cycle, rain, and wind: NOT RECOVERABLE FROM PLAN
- Field notebook: It is read-only, auto-generated, rare, and "sparse naturalist prose" so users receive behavioral observations, not trait values, event counts, or template strings.
- One-to-one visit invitations: Visits are off by default, email-based, revocable, and read-only to avoid social network surfaces while still allowing a host to share an aviary.
- Visit log in account settings; no notifications on visit by default: The plan places visibility in account settings and avoids default pings, consistent with the hard non-goal against push notifications and counters.
- Accessibility surfaces: Screen-reader narration, reduced motion, and call captions are included in v1 so the aviary remains usable when motion or audio are not available or appropriate.
- Account export and soft-then-hard deletion: Export is available "on demand"; deletion is recoverable for 30 days, giving a data-control path with a recovery window.
- Aggregate operational telemetry only: The plan measures operations without per-account dimensions so analytics cannot include per-account interaction history, presence totals, or notebook content.
- WCAG AA contrast: The plan requires contrast on all user-copy text and specific icon, focus, and caption contrast rules so the aviary remains readable across day and night palettes.

### 2. Architecture

- Service decomposition into Browser Client, API Gateway, services, and stores: NOT RECOVERABLE FROM PLAN
- Server owns all state: The client never writes personality because server ownership preserves canonical state and makes personality conflicts impossible.
- Client renders snapshots and interpolates: The client can stay responsive between low-frequency snapshots while not treating interpolation as authoritative state.
- Client writes events only: Events are accepted asynchronously and retried, while authoritative state remains the result of server ticks.
- Simulation tick is server-only: Keeping tick logic off the client prevents device-specific drift or mood decisions.
- Render Worker and State Client boundary: The boundary "protects the animation loop from network jank" because the Render Worker does not touch the network.

### 3. Data Model

- Account UUID, encrypted email, and email hash: The account id is synthetic and "never email"; encrypted email and non-reversible hash support lookup without making email the identifier.
- Sessions with device_hint, last_seen_at, and revoked_at: `device_hint` is "display only," and revoked sessions support explicit session management.
- Magic links with token_hash, expiry, and consumed_at: The one-time token is "never stored plain," expires after 15 minutes, and is marked consumed to prevent reuse.
- Aviary day_night_offset_minutes: NOT RECOVERABLE FROM PLAN
- Bird stable id, name, species_id, adopted_at, and perch state fields: NOT RECOVERABLE FROM PLAN
- Personality vector columns on the bird row: The plan says they are "always read/written together by the tick," and row-level locking is "the correct conflict model."
- Personality vector fields excluded from client APIs: The plan hides `pv_*` so users and clients never receive numerical personality state.
- Mood persists across sessions: Persistence prevents mood from resetting or snapping to a default on load.
- Interaction event log append-only: Append-only events avoid update conflicts, allow the tick to read events since the high-water mark, and support lifecycle delete only through account deletion cascade.
- Tick state with event_hwm and next_tick_at: `event_hwm` prevents processed events from being re-read; `next_tick_at` drives scheduler dispatch.
- Notebook entries immutable after insert: The notebook is read-only after generation, preserving field-note entries as ordered observations.
- Visit invitations and sessions with expiry, revocation, consumption, and pings: These fields make invitation access time-bounded, revocable, and able to return 410 when no longer valid.

### 4. API Surface

- REST/JSON over HTTPS, bearer auth, gateway rate limits, and session validity: NOT RECOVERABLE FROM PLAN
- `POST /auth/request-link`: It always returns 202 to "never confirm or deny email existence for enumeration safety."
- `GET /auth/confirm`: The plan uses an HttpOnly, SameSite=Lax cookie and one-time-token consumption for session safety, with matter-of-fact errors when expired or already consumed.
- Session revoke and session listing endpoints: NOT RECOVERABLE FROM PLAN
- `/aviary/snapshot`: It returns canonical state for rendering and includes `call_state` as a deterministic seed, while excluding personality vectors "ever."
- `/aviary/snapshot/delta`: It returns only changed fields and is "Used by keepalive poll."
- `/aviary/events`: It is async so the client "does not wait for tick"; batching, retries, and idempotency make event writes robust.
- `/aviary/notebook` cursor pagination and newest-first ordering: NOT RECOVERABLE FROM PLAN
- Host visit invitation endpoints: Masked emails avoid exposing full visitor addresses in lists, and revocation causes active sessions to get 410 on the next snapshot pull.
- `/visits/log` ordered by visit_started_at descending: NOT RECOVERABLE FROM PLAN
- Visitor snapshot endpoints: The invitation token is the auth, visitor access is read-only, and visitors "cannot POST events."
- `POST /account/export`: The export is queued and emailed to the verified address rather than exposed immediately in-app.
- `POST /account/delete` and `POST /account/recover`: Soft deletion is recoverable during the 30-day window.
- Visit-notification setting: The plan keeps visit notifications "off by default" and makes them an account setting.

### 5. Simulation Engine Design

- Tick scheduler and ~60 second cadence: Slippage up to ~5 minutes is tolerable because "drift is slow" and "mood transitions are gradual"; the p99 alarm catches degradation.
- Row-level idempotent ticks: If a tick crashes mid-write, re-running from the same `event_hwm` should produce the same output.
- Tick algorithm with aviary row lock, event high-water mark, and transactional write: Locking and high-water-marked event consumption keep state updates consistent for an aviary.
- Drift learning rate: The rate is chosen so instruments detect change after about a week and users feel change after about three weeks.
- Exact event-to-trait weight table values: NOT RECOVERABLE FROM PLAN
- Monotonic constraint: Deltas are always non-negative so "Traits never decrease."
- Learning rate environment config: The rate is the "primary calibration knob" and must be tunable without code deploy.
- Mood transition probability table: Mood is driven by personality, time of day, recent events, and weather so the current state reflects context rather than only direct interaction.
- Reduced probability for non-adjacent mood transitions: NOT RECOVERABLE FROM PLAN
- Server-generated weather: Weather state is generated by the simulation service and included in snapshots so the client does not independently generate simulation-affecting weather.
- Client-side ambient ornaments only: Passing leaves and midnight calls are rendering ornaments and "do not affect the simulation."
- Notebook generation criteria, rarity, and throttle: Entries appear for notable behavioral changes, unusual stability, quiet stretches, or unusual offers, but are rare by design and never numeric.

### 6. Sync Model

- Canonical state propagation: The flow ensures laptop and phone both render the same canonical record after ticks advance.
- Conflict prevention: The plan avoids conflict resolution because clients never write personality, events are append-only, and simultaneous event writes are safe.
- Snapshot caching: A 10 second edge TTL is acceptable "given the slow tick cadence," while focus changes request a fresh no-cache snapshot.
- Full snapshot on reconnect: A full pull avoids stale interpolation state after a hidden tab or suspended laptop.

### 7. Frontend Rendering Pipeline

- Canvas 2D for the aviary scene: Canvas gives "full control over compositing order, parallax, and mood-dependent color grading."
- Web Workers for rendering: OffscreenCanvas and worker rendering keep frame rendering independent from main-thread network work.
- React or equivalent for top-bar chrome and modal surfaces only: NOT RECOVERABLE FROM PLAN
- Scene composition layer order: NOT RECOVERABLE FROM PLAN
- First-frame strategy: Embedding the snapshot and starting the Render Worker early supports "birds mid-action within 500ms" with "No loading spinner"; stale render is preferred to blank state.
- Idle micro-motion system: Mood-shaped motion makes wary, content, curious, drowsy, and alert birds behaviorally distinct.
- Deterministic idle motion seed: Two clients showing the same bird at the same tick time should show "the same motion pose."
- Mood-to-visual mapping: The plan expresses plumage saturation visually, boldness through server-side perch selection, and mood through behavior instead of exposing trait values.
- Perch, mood, settle, and listen-in transitions: The plan says birds "Never teleport," mood shifts are behavioral cross-fades, settle warms and quiets the scene, and listen-in avoids distracting visual changes.
- Reduced-motion mode: Animated motion becomes cross-fades or static overlays, ambient leaf drift is disabled, wind is suppressed, and gradual day/night color transitions remain.
- Top bar fade behavior and exact top-bar contents: NOT RECOVERABLE FROM PLAN
- Loading and empty aviary state: The sky gradient appears immediately with no spinner, and after two birds the user "never sees an empty aviary again."

### 8. Audio Pipeline

- AudioContext created on first user gesture: This satisfies the "browser autoplay policy constraint."
- Render Worker owns AudioContext: The plan says this "avoids jank from main-thread activity."
- Per-bird call grammar: Calls are shaped by motif libraries, personality, pitch, mood, and seeded jitter so birds vary while remaining deterministic across devices.
- Chorus mixing with per-bird GainNodes and compressor: The compressor prevents clipping when multiple birds call at the same time.
- Listen-in mix: Gain ramps foreground the focused bird and lower others to ambient without hard cuts.
- Day/night volume modulation: Morning, midday, evening, night, and settle gains make the aviary quieter as night or settle approaches.
- Weather audio effect: Rain adds quiet pink noise and reduces individual bird gains while rain is active.
- Procedural vs recorded boundary: No audio files are downloaded; oscillator, filter, and convolver nodes keep calls programmatic, and code motifs make iteration cheap.
- Temporary per-call subgraphs: Nodes disconnect on completion so there is "No permanent per-call allocation after playback."
- WebAudio fallback: If AudioContext is unavailable or denied, visuals continue and captions turn on automatically, yielding "graceful silence with captions."

### 9. Accessibility Surfaces

- Screen-reader narration: A live region receives slow-cadence and event-specific naturalist prose, with return-greeting narrations priority-queued for session start.
- Narration constraints and non-template prose generation: The function must avoid trait values and announcement framing, stay lowercase and present-tense, and vary by current snapshot state.
- Call captions: Captions mirror the call scheduler, use naturalist descriptions instead of technical terms, and limit visibility to two captions to avoid clutter.
- WCAG contrast and focus treatment: Icons, text, captions, and focus indicators get contrast rules across bright and dark aviary states; the hidden narration uses a clip pattern so screen readers can access it.
- Keyboard navigation: Overlaid interactive elements, tab order, arrow cycling, Enter, and Escape make birds, offers, and notebook panels operable without making the canvas itself focusable.

### 10. Performance Budgets and Observability

- Bundle size under 2MB gzipped initial JS: The plan budgets subsystems and lazy-loads settings, invitations, and notebook so initial load stays small.
- Time to first bird visible under 500ms: Embedded snapshots, early worker init, and immediate sky rendering target the first visible bird pixel and avoid a blank white frame.
- 60fps idle for 30 minutes: The plan uses worker rendering, single composite draw calls, state-machine micro-motion, and pre-allocated particles to stay under frame budget.
- No memory growth over 30 minutes: Audio cleanup, notebook virtualization, holding only current and previous snapshots, and non-accumulating worker messages keep heap growth under 5MB.
- Error budget and alarms: The listed thresholds page or alert on tick latency, event write failures, snapshot latency, audio context errors, first-bird timing, and magic-link delivery.
- Measured and not-measured telemetry boundary: The product measures aggregate operational signals while forbidding per-bird vectors, per-account histories, presence totals, listened-in birds, and per-user notebook content.

### 11. Rollout

- Pre-launch internal testing: Internal daily use validates drift calibration, mood persistence, presence accounting, first-bird-visible timing, screen readers, and reduced motion before beta.
- Private beta launch: Invite-only beta monitors tick latency, snapshot latency, event volume, session histograms, audio errors, memory growth, timing regressions, and notebook voice quality.
- V1 launch: Public sign-up opens after beta; visits ship behind account-level off-default behavior, bird ramping begins, export is available day one, and unsupported browsers get a matter-of-fact surface.
- Bird-count ramping: Birds 3-7 become available by aviary age, thresholds are tunable, and the offer appears in account settings with "no pressure, no expiration countdown."
- Day-one instrumentation: Synthetic monitoring, RUM, alarms, memory CI, and endpoint error-rate alarms run from public launch day.

### 12. Risks

- Drift calibration mitigation: The plan treats wrong learning_rate as a risk and mitigates it with a tunable environment constant, internal instrumentation, and explicit pass/fail criteria.
- Sync ordering mitigation: DB-assigned sequence order avoids relying on skewed client clocks for tick consumption.
- Audio uncanniness mitigation: The plan protects the "affective spine" by using an ornithological audio consultant, beta focus testing, motif iteration time, and cutting species that miss the quality bar.
- Accessibility narration regression mitigation: Unit tests, bounded module review, and screen-reader testing protect naturalist prose from becoming awkward or generic.
- Presence accounting mitigation: Isolated code and tests for visibility, focus, activity recency, and all-three-satisfied cases prevent over-reporting that would inflate drift.
- Tick scheduling failure mitigation: `tick_states` heartbeat checks, structured logs, and re-runnable transactional ticks make frozen aviary states detectable and recoverable.
- Gamification creep mitigation: Feature review checks the non-goals list, escalates streak-adjacent metrics, and uses onboarding to explain the reason for the rule.
