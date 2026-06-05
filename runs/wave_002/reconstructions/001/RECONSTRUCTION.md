## System-level intent

- Protect the "central affective contract": the aviary should feel like "a small living place that continues without the viewer," not "an app that starts performing when opened." This shows up in Product Reading and Implementation Posture, first frame/loading, server-side simulation, performance budgets, and the instruction that the scene should appear "already in motion."
- Keep continuity canonical on the server. The plan says "the server as the canonical owner of aviary continuity" and "the browser as a renderer, interaction collector, and procedural audio/runtime animation surface." This appears in architecture, simulation tick ownership, sync, conflict prevention, and offline handling: clients "never tick the simulation" and "never own personality state."
- Preserve hidden interiority without exposing numeric machinery. The plan repeatedly says personality vectors are hidden, raw values are not sent to ordinary surfaces, and clients receive "derived render/audio parameters." It allows raw vectors only in the private JSON export because that export is "private account data," not an in-product statistics surface.
- Use two different voices deliberately. Product-facing aviary, notebook, captions, and narration surfaces use "naturalist, lowercase, present-tense prose." System, account, error, and accessibility settings surfaces use "matter-of-fact prose."
- Avoid gamification, obligation, punishment, and re-engagement loops. The plan excludes "achievements, streaks, scores," "Tamagotchi mechanics," visit-frequency displays, notifications, and default visit nudges. It also says absence can quiet current mood but "cannot make personality regress."
- Make the product "quietly social" without turning it into a social network. Visit invitations are read-only, off by default, revocable, expiring, and create "no simulation inputs." The plan excludes profiles, feeds, comments, chat, co-presence, visitor avatars, and leaderboards.
- Treat privacy as a product boundary, not only a security detail. The plan uses encrypted email, synthetic UUIDs, aggregate-only operational telemetry, forbidden per-bird analytics, and tests that block simulation schemas from analytics emitters.
- Treat accessibility as part of the aviary, not a fallback. The plan says narration, captions, and reduced-motion rendering are "primary product surfaces" and must preserve naturalist voice, call grammar, mood, notebook, and drift.
- Make performance part of felt aliveness. The plan ties "first bird visible under 500ms," minimal bootstrap snapshots, no spinner, no memory growth, and compact procedural assets to avoiding loading patterns that undermine the "continuing-place conceit."
- Prefer recognizable procedural variation over canned media. The audio plan uses per-species motif libraries, per-bird stable seeds, mood/personality modifiers, and runtime variation, while excluding recorded-audio fallback paths.
- Keep calibration controlled and testable. Drift constants are versioned, notebook prose is templated, synthetic seeded accounts are used for monitoring, and beta rollout keeps additional birds disabled until drift, audio, readability, chorus recognizability, and performance budgets hold.

## Per-feature whys

**V1 scope**

- Modern browser web app only, responsive from phone to desktop: NOT RECOVERABLE FROM PLAN
- Email magic-link authentication: the plan frames the flow as a protected account surface with one-time links that expire, are rate limited, are invalidated immediately on use, and "always respond generically."
- One account per user and one aviary per account: NOT RECOVERABLE FROM PLAN
- Two starter birds selected by the system from a coherent species pool: the plan says species selection should "feel like arrival, not user configuration."
- User naming and renaming of birds without changing bird identity: bird ID is "permanent and survives renaming"; names are direct metadata, not personality or drift.
- Hard V1 cap of seven birds per aviary: the rollout says to move toward seven only after "chorus recognizability and performance budgets hold."
- Age-based availability of additional birds: it is explicitly "not tied to visit count, streaks, score, or payments," supporting the anti-gamification and no-obligation posture.
- Server-side canonical aviary state advanced by a slow simulation tick: this preserves continuity and prevents clients from owning personality, mood, or simulation time.
- Hidden per-bird personality vectors with monotonic drift: the plan wants measurable change after regular visits and visible change after weeks, while ensuring "no single session creates visible personality change" and neglect does not reduce traits.
- Mood state persisted across sessions: opening the tab "must never reset mood to neutral"; the snapshot should show the server-advanced mood.
- Procedural client-side calls with per-bird recognizable signatures: stable seeds make a bird remain recognizable, while runtime variation prevents fixed loops and repeated identical calls.
- Listen-in interaction: it lets focus on one bird through a "gradual audio mix ramp" while other birds remain ambient; listen-in also provides a stronger per-bird signal for social warmth and vocal frequency.
- Offer interaction: offers are "gestures, not inventory or feeding mechanics"; outcomes shape mood and drift without "success/failure scoring" or obligation.
- Settle interaction: settle is a "mood-quieting and presence-ending signal," shifting the scene quieter without being a personality reward.
- Field notebook: entries are "sparse observations" in a naturalist voice, "not an event log," not every session, and never a summary of user engagement.
- Presence accounting: presence requires visibility, focus, and recent pointer/key activity so idle tabs and background windows do not corrupt drift.
- Return greeting: the plan selects "one primary bird, not all birds," with varied but stable behavior and "no text welcome, no absence banner, no toast."
- Single horizontal aviary scene with three perch zones and no UI chrome inside the scene: the scene should keep all birds in frame, avoid panning/zooming, and not become "an explorable geography."
- Multi-device sync: every device reads the same canonical state, with "no client-to-client sync" and no merge of client-owned personality state.
- Read-only visit invitations: invitations support quiet social viewing while protecting the host aviary; visitors see read-only ambient state and "create no simulation inputs."
- Screen-reader narration, call captions, reduced motion, keyboard navigation, focus treatment, and WCAG AA contrast: the plan treats these as V1 product surfaces so accessibility does not feel like a flattened fallback.
- Account export, account deletion, session management, and email-change verification: these provide private account control through verified links, revocable sessions, and 30-day soft deletion.
- Aggregate-only operational telemetry and performance observability: the plan forbids per-bird and per-account interaction analytics to protect privacy and avoid making "future leaderboards or engagement ranking easy."

**Architecture overview and storage**

- Modular monolith service shape: the plan says services can start together and scale into separate deployments "only when load requires it."
- Simulation worker as the only writer of canonical aviary state: this concentrates ownership of personality deltas, mood, weather, notebook entries, availability, and state versions.
- Snapshot cache and edge delivery: the cache supports the "first state snapshot quickly enough" for first bird under 500ms, but it is never authoritative.
- Relational primary store: PostgreSQL is justified by transactional event ingestion, row-level locking for per-aviary ticks, compact JSON fragments, and durable privacy boundaries.
- Short-lived cache/queue: Redis or equivalent supports scheduling, idempotency locks, rate limiting, and fanout, while cache loss degrades to database reads "rather than state loss."
- Canonical versus ephemeral state boundary: canonical state lives on the server, while client-only animation, audio ramps, focus state, captions timing, and decorative drift can be reconciled to the next authoritative snapshot.

**Core data model**

- Account encrypted email field: email appears only on the account record and email delivery jobs, and is "never used as a partition key or telemetry identifier."
- Account timezone: timezone drives day/night and mood signals, and is updated conservatively with confirmation when it changes materially.
- Session records and revocation: tokens are per-device and revocable; revoked or expired sessions cannot request snapshots or write events.
- Aviary state version: `current_state_version` is returned in snapshots and referenced by writes for diagnostics, but clients do not submit authoritative state.
- Bird stable UUID and server-owned personality: the stable ID preserves identity across renames and future migrations; personality values are updated only by the simulation worker.
- Species model: rarity is not a V1 feature, and starter selection should feel like "arrival," not catalog configuration.
- Append-only interaction event log: it is "the only client-to-server path for simulation inputs" and supports idempotency, receive-order processing, and monotonic safeguards.
- Visitor event boundary: visitor events are render/session telemetry only and "never simulation inputs."
- Presence windows: derived windows split long gaps and refuse visible-but-idle or focused-but-hidden cases so attention is not inflated.
- Notebook entries: entries are rare, read-only, and describe "the aviary, not the user's engagement pattern."
- Visit invitation and visit log: the host can revoke outstanding or active invites immediately, invite links expire after 30 days, and default notifications are not promoted.

**API surface**

- Matter-of-fact API copy: system, account, and error flows avoid product prose; aviary, notebook, and narration content carry the naturalist voice.
- Authentication endpoints: magic links are short-lived, rate limited by email and IP, generic in response, and invalidated on use.
- Account settings endpoint: it updates matter-of-fact controls such as captions, audio default, reduced motion, visit notifications, and timezone confirmation.
- Account export endpoint: the export is queued and delivered by a short-lived verified email link, keeping raw private data outside ordinary product surfaces.
- Account delete and restore endpoints: soft deletion starts a 30-day recovery window before hard deletion.
- Bootstrap endpoint: it returns "the minimum renderable state" to support time-to-first-bird.
- Snapshot endpoint: it returns compact canonical state with derived render and audio parameters while excluding raw personality vector values.
- Events endpoint: it validates ownership, session, presence evidence, idempotency, and allowed event types, then appends events without mutating personality or mood directly.
- Rename endpoint: bird rename is handled as a typed event plus metadata update while preserving identity.
- Adoption endpoint: new-bird arrival accepts naming when age-based availability exists, without exposing a catalog or rarity mechanics.
- Notebook endpoint: it paginates read-only entries and excludes generic event logs and user-behavior summaries.
- Visit invitation endpoints: they create, list, and revoke revocable email invitations that expire after 30 days.
- Visitor snapshot endpoint: it returns a read-only render snapshot without notebook mutation or host controls, and revoked/expired links get a matter-of-fact unavailable surface.

**Simulation engine design**

- Slow per-aviary tick: a roughly 60-second tick gives the server sole ownership of personality, mood, weather, settled state, notebook generation, availability, and state-version increments.
- Per-aviary lock or lease: only one worker ticks an aviary at a time, and idempotency prevents retries from applying the same event twice.
- Tick inputs limited to aviary-local state: the tick reads current state, unconsumed owner events, presence, local time, weather, cooldowns, bird influence, and age; it does not read analytics or cross-account population data.
- Drift function: it is a low-pass filter so drift is instrument-measurable after about one week, user-visible after about three weeks, and invisible after a single session.
- Trait effects: presence broadly nudges expressiveness, listen-in affects warmth and vocal frequency, offers nudge curiosity and boldness modestly, and settle quiets mood rather than boosting personality.
- Monotonic expressive rule: positive signals can move traits upward within bounds, but neglect does not reduce boldness, warmth, vocal frequency, plumage saturation, or curiosity.
- Bounded render/audio mappings: raw normalized traits become derived parameters so small changes do not produce sudden visual or audio jumps.
- Mood system: time of day, recent interactions, weather, personality, and bird-to-bird influence create persistent bird mood instead of tab-open resets.
- Return-greeting selection: one primary bird responds based on absence length, personality, mood, perch zone, and recent greeting history, with seeded randomness that is "varied but stable for the moment."
- Offer handling and cooldowns: cooldowns prevent trait saturation, while outcomes shape mood and future drift without reward, feeding, or obligation language.
- Field notebook generation: templated prose with controlled variation keeps privacy, tone, latency, and testability under control.

**Sync model and conflict prevention**

- Snapshot pulling and interpolation: clients pull on load, visibility changes, long gaps, keepalive, and event submission, then interpolate locally from canonical snapshots.
- Event ingestion with idempotency keys: clients can retry safely, and rejected reasons are stored for debugging while visible errors remain matter-of-fact.
- Personality and mood conflict rules: personality and mood have no client writes, no last-write-wins, and no merge UI.
- Bird rename conflict rule: the last accepted owner rename can win because names are direct metadata, not drift.
- Settle conflict rule: recent accepted settle or undo determines visual settled state, while presence accounting still follows accepted events.
- Visit revocation conflict rule: revocation wins immediately and the next visitor snapshot returns unavailable.
- Offline and poor connectivity: V1 supports short graceful disruptions, not an offline product; stale clients cannot simulate personality or advance canonical mood.

**Frontend rendering pipeline**

- Deterministic rendering boundary: the scene runtime consumes snapshots and emits render commands, captions, and audio cues; UI components consume derived view models, not raw server records.
- First frame and loading: the first frame is "the aviary or a quiet field, never a machine-like spinner," and birds appear in current mid-action pose rather than at an entry default.
- Quiet-field slow connection path: subtle non-bird motion cues are allowed, but no spinner, progress bar, welcome copy, or app-like skeleton.
- Scene composition: a single horizontal scene with background, perches, optional foreground, and three logical perch zones keeps the product contained and readable.
- Responsive scene rules: the plan preserves all birds in frame and avoids cropping, panning, scrolling, or zooming.
- Bird rendering: mood, perch zone, motion intent, species, and derived personality parameters make behavior visible without exposing raw traits.
- Idle micro-motion: preening, scanning, head tilts, shuffles, and rests keep birds from being "perfectly still in a paused way."
- Reduced-motion rendering: slow cross-fades replace micro-animation and flight, leaf drift is removed, and calls, captions, notebook, mood, and drift remain intact.
- Top bar controls: the sparse set keeps account/settings, accessibility, notebook, offer, and settle available without putting UI chrome inside the scene.
- Top bar fade and focus behavior: controls fade nearly transparent during cursor stillness but return on pointer or keyboard activity; keyboard focus visibly brings them back.
- No top-bar badges or scene tooltips: the plan forbids badges for visit logs, notebook updates, achievements, or visits, and avoids hover tooltips inside the aviary scene.
- Return flow: the client renders the snapshot immediately, one bird notices within one to two seconds, and there is no textual welcome or absence copy.
- Listen-in flow: activation focuses a bird, ramps audio gradually, lowers others to ambient instead of silence, and submits start/end events.
- Offer flow: offers open from the top bar, use a "small, calm affordance," enforce cooldown quietly, and avoid punitive language.
- Settle flow: settle shifts lighting toward evening, quiets calls, allows a five-second undo, and treats closing without settle as ordinary.
- Notebook flow: notebook is read-only, virtualized for safe long scroll, and excludes edit, delete, annotate, and visit-frequency summaries.

**Audio pipeline**

- Procedural call runtime: per-species motif libraries and per-bird stable seeds give each bird a recognizable call identity.
- Mood and personality audio modifiers: tempo, brightness, pause length, sharpness, call likelihood, and response probability are derived from state rather than invented by the client.
- Runtime call variation: timing, pitch microvariation, envelope, and spacing change each call so the result does not feel canned.
- Default mix: the aviary remains a soft ambient chorus, with subtle spatialization that should not feel like "a technical demo."
- Listen-in mix: focused birds ramp up slowly, other birds ramp down to ambient rather than zero, and hard cuts or "track-solo metaphors" are avoided.
- Chorus rendering: overlapping calls are actual overlapping procedural calls, not stacked identical loops or fixed samples.
- Captions from call grammar: captions use the same procedural grammar as the audible call, making them useful for audio-off and hearing differences.
- WebAudio unavailable fallback: the product plays in graceful silence, enables captions by default for that session when appropriate, and does not load recorded audio.
- Audio performance: bounded voices, reused resources, and aggregate error tracking prevent leaks and preserve privacy.

**Accessibility surfaces**

- Screen-reader narration: narration consumes the same derived state as visual rendering and emits slow-cadence naturalist prose instead of state-list labels.
- Narration queue: priority levels and coalescing prevent high-frequency spam while preserving prompt updates for user-initiated events.
- Keyboard navigation: tab reaches top bar controls and the bird focus group, arrows move between birds, enter toggles listen-in, escape exits, and offer/settle remain keyboard reachable.
- Focus indicators: indicators must remain visible across morning, evening, and night palettes.
- Reduced motion preferences: OS preference and explicit settings are honored, with the in-app setting able to override default behavior in a matter-of-fact settings surface.
- Reduced motion testing: the plan calls reduced motion a "designed rendering mode" that must be tested alongside default rendering.
- Captions and contrast: captions must not obscure controls or become scene labels, and all user-copy text passes WCAG AA across day, night, weather, and focus states.

**Performance budgets, observability, privacy, rollout, and testing**

- Initial JS budget and code splitting: keeping first paint under 2MB gzipped and splitting non-aviary surfaces lets the first bird render without waiting for settings, notebook, visit, export, or delete bundles.
- Time to first bird: the 500ms budget supports the plan's felt-aliveness goal and avoids spinner/loading patterns.
- Runtime performance: 60fps idle motion, no 30-minute memory growth, hidden-tab pause, and fresh snapshots on visibility return maintain the living-place illusion.
- Simulation performance: p99 tick latency alarms and retry/backpressure safeguards prevent slow or double-applied simulation.
- Allowed observability metrics: request counts, latencies, payload size, first-bird timings, frame timings, audio errors, tick health, auth success, job success, and anonymized duration histograms are operational only.
- Forbidden telemetry: per-bird state, raw personality vectors, per-account histories, cross-account drift dashboards, visit-frequency surfaces, and leaderboard-ready data paths are disallowed.
- Synthetic monitoring: seeded test accounts use artificial data, stay isolated from production analytics, and are clearly marked.
- Privacy and security controls: synthetic UUIDs, encrypted emails, hashed tokens, expiring links, verified email changes, secure exports, soft deletion, hard deletion, and telemetry anonymization protect account and simulation data.
- Analytics import tests: automated checks fail builds if event payload schemas containing bird or personality fields enter analytics emitters.
- Phase A foundations: the exit criteria prove sign-in, one aviary, two starter birds, server snapshot rendering, and no client personality writes.
- Phase B simulation and presence: the exit criteria prove canonical multi-device state, calibrated drift, no single-session jump, and no negative personality drift from absence.
- Phase C rendering and audio: the exit criteria prove first-bird budget, 60fps, no memory growth, and no repeated identical procedural calls.
- Phase D product surfaces: the exit criteria prove naturalist narration, visitor read-only boundaries, next-snapshot revocation, and secure account export.
- Phase E private beta and calibration: additional bird availability stays disabled until drift, audio, readability, recognizability, and performance calibration pass.
- Testing strategy: unit, integration, end-to-end, performance, and privacy tests guard drift monotonicity, presence, mood persistence, event idempotency, auth, visits, voice, captions, sync, deletion, rendering, and telemetry boundaries.
- Voice and UI copy linting: banned phrases such as welcome back, streak, achievement, level, score, and days visited prevent product surfaces from drifting toward gamification.
- Risk mitigation for drift calibration: version constants, time-travel tests, hidden QA comparisons, and bounded low-pass drift prevent changes from feeling too fast or never arriving.
- Risk mitigation for presence inflation: visibility, focus, recent activity, long-gap splitting, no visitor presence, and tests for hidden/unfocused/idle/suspended states protect drift input quality.
- Risk mitigation for sync correctness: simulation-worker ownership, service-role restrictions where possible, ownership tests, contract tests, and endpoint review guard against client personality writes.
- Risk mitigation for audio: stable identity plus runtime variation, golden audio snapshots, human QA, and ramp testing keep calls recognizable rather than canned or uncanny.
- Risk mitigation for accessibility: design review, voice lint tests, screen-reader QA, visual regression for reduced motion, and shipping accessibility with V1 prevent flattened fallback experiences.
- Risk mitigation for privacy boundary erosion: separated simulation database access, early allowed metrics schemas, forbidden-field checks, and a ban on aggregate drift analytics protect private bird interactions.
- Risk mitigation for felt-aliveness performance: CI gates, minimal snapshots, code splitting, and compact procedural assets reduce the chance that slow loading forces spinner-like patterns.
