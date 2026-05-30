## System-level intent

- Presence is relational, not a counter. This shows up in the hard non-goal of "no streaks, achievements, badges, levels, scores, counters, or "days visited" surfaces" and in the review question: "Does this teach the user that presence is for a counter rather than the birds?" It also shows up in the refusal of the "gamification trap" where "more attention earns more stuff."
- The relationship is observational, not custodial. The plan says "birds do not die, hunger, decay, or show distress" and names the relationship "observational, not custodial." It reappears in monotonic drift: "absence is fine, not penalized."
- The aviary is a continuing place, not a session-bounded app. The "server-authoritative simulation" keeps ticking; first frame has birds "mid-action" because "it has been"; mood "persists across sessions"; and the 500ms target exists so the aviary "feels like it was already running."
- Canonical server state protects coherence and history. The plan repeatedly makes the server the only writer of personality state: "clients render what the server says," "client never writes personality state," and "No last-write-wins for personality state." This is the basis for multi-device sync and for not deleting "drift history."
- Bird state should be felt through behavior, not read as numbers. Personality vectors are "never" shown; the user reads perch choice "as signal," reads mood from motion "without being told," and captions/narration use naturalist prose rather than labels like "Wren mood: content."
- Change should be slow, sparse, and meaningful. Drift is a "low-pass filter" with "no visible change in a single session"; notebook entries are "not on every session" and "sparsity is a design constraint"; new birds arrive by "aviary age" rather than effort.
- The product voice is naturalist and observational, with one explicit exception. Notebook prose is "lowercase, present-tense, specific"; screen-reader narration uses the "same voice as the field notebook"; call captions are "short naturalist prose." Sync conflicts intentionally leave that voice for "matter-of-fact tone: direct, clear, no naturalist phrasing."
- Accessibility is first-class design, not a fallback. Reduced-motion is "a designed surface, not a stripped fallback"; screen-reader narration and reduced-motion "ship with v1"; accessibility surfaces are "first-class, not a checklist."
- Procedural audio is part of the spell of liveness. The plan rejects recorded audio because looped audio is the "audible signature of dead software" and because the chorus mechanic needs "real-time per-call variation" and independent synthesis streams.
- Performance is affective, not only technical. The bundle budget, first-bird budget, 60fps idle motion, and no-growth memory rule all serve the sense that the aviary is alive and already running; the plan names the 500ms threshold as the "affective-perf bridge."

## Per-feature whys

**Scope and non-goals**

- Two starter birds per new account: The first encounter should be "meeting an animal, not configuring an avatar," so v1 starts with birds rather than a catalog choice.
- Optional adoption flow up to a cap of seven: New birds arrive on "aviary age" so the pacing matches "the rhythm of a relationship deepening." The cap of seven exists because beyond seven "the ear cannot reliably separate individual call signatures" and the "per-bird relationship collapses."
- Single canonical aviary per account: This supports the "single canonical aviary state" that all clients read, making sync a property of the architecture.
- Single-user accounts: NOT RECOVERABLE FROM PLAN
- Magic-link sign-in and no passwords: NOT RECOVERABLE FROM PLAN
- Multi-device sync via server-side canonical state: Both laptop and phone pull "the same canonical state snapshot," so "there is nothing to sync because both clients read from the same record."
- Procedural calls: Looped audio would make the product feel like "dead software," while procedural calls support variation and real chorus mixing.
- Mood system: Mood lets birds behave differently across time, interactions, ambient events, and personality; it is read through motion and must not "snap" to a default on tab open.
- Personality vector drift and presence accounting: Regular presence and interactions shape birds slowly, making the relationship matter without creating "a stat-management exercise."
- Return-greeting: NOT RECOVERABLE FROM PLAN
- Listen-in: Focusing a bird is a "strong attention signal," but the audio change must be a "re-balance, not a mute" so the aviary remains a place where "multiple things are happening at once."
- Offer: Offers create small personality and mood nudges: accepted offers move curiosity toward the offering bird, offering near a bird nudges boldness, and accepted offers nudge mood toward content.
- Settle gesture: Settle "ends presence cleanly" and carries "no directional drift," so it closes a visit without manipulating traits.
- Field notebook: The notebook should record only when "something worth noting has happened." It is naturalist observation, not a generic event log; "an entry per session dilutes the entries that matter."
- Visit invitations: A friend sees a "read-only ambient view" with no interaction events and "No co-presence mechanism," keeping the feature from becoming a social network surface.
- Screen-reader narration: Narration uses the same state and "same voice as the field notebook" so nonvisual access receives the product's naturalist experience, not a state-change list.
- Reduced-motion mode: A vestibular user gets "a Pocket Aviary that is calmer and slower, not broken." Motion is replaced with cross-fades while birds still drift, moods change, calls play, and the notebook notices.
- Call captions: Captions describe what a call sounds like in "short naturalist prose" and are generated from the procedural grammar so each caption "matches what was actually played."
- Performance budgets: The budgets preserve the feeling of liveness: fast first bird, 60fps idle motion, and no memory growth make the aviary feel already present rather than loaded on demand.
- No gamification surfaces: The hard rule prevents teaching that "presence is for a counter rather than the birds."
- No Tamagotchi mechanics: The plan rejects death, hunger, decay, and distress because the relationship is "observational, not custodial"; absence becomes quieter ambience, not punishment.
- No social network surfaces: Profiles, follows, public discovery, leaderboards, and shared exploration are excluded so visits remain read-only, ambient, and not a social graph.
- Native mobile apps, with web-only at v1: NOT RECOVERABLE FROM PLAN
- Multi-aviary accounts, customizable scenes, panning/zooming, and payment features: NOT RECOVERABLE FROM PLAN

**Architecture, data, and API surface**

- Server-authoritative simulation with stateless clients: This makes multi-device sync coherent and prevents clients from owning state that could diverge.
- Client never writes personality state: This prevents "last-write-wins corruption of drift history"; clients send interaction events and the server decides what they mean.
- Append-only event log: The event log is "the only thing clients write," and the tick consumes it "in order," preserving ordered drift history.
- Client-side rendering interpolation, WebAudio synthesis, DOM lifecycle, input capture, and presence detection: The client owns local presentation and signals while the server owns simulation state, keeping the split clear.
- Simple in-process store, Redis-backed option, or SQLite at small scale: The plan avoids committing to an external database "until scale forces it."
- Lean client and no heavy framework by default: The "<2MB initial JS bundle" demands "a lean render path"; React must be justified against that budget.
- CDN-edge state snapshot payloads: Snapshots are served from the edge for "fast first-bird delivery."
- Account synthetic ID and encrypted email field: NOT RECOVERABLE FROM PLAN
- Account soft-delete with 30-day window before hard delete: NOT RECOVERABLE FROM PLAN
- Stable bird internal ID that is never recycled: NOT RECOVERABLE FROM PLAN
- Species pool of about six options: NOT RECOVERABLE FROM PLAN
- User-assigned editable bird name: NOT RECOVERABLE FROM PLAN
- Personality vector hidden from the user: "Vector values are felt through behavior, not read as numbers," so there is no stats panel, debug view, or "show me how my bird is doing" surface.
- Call grammar version: This "allows evolution without migration."
- Idempotent-keyed interaction events: Event writes are "retry-safe."
- Rename endpoint changes name only and leaves personality untouched: The rename operation is scoped so identity text can change without altering the bird's personality state.
- Adopt endpoint where the system selects species and the user names it: This preserves adoption as meeting an animal instead of choosing from a catalog or configuring an avatar.
- Call grammar endpoint: The client needs the motif library because "client-side synthesis needs this."
- Visitor path with read-only snapshot stream: Visitors can observe the host aviary without writing events or creating co-presence.
- Account export by emailed download link: NOT RECOVERABLE FROM PLAN

**Simulation and sync**

- Server-side tick: The tick is the only writer of personality state and the place where events become drift, moods, positions, notebook entries, weather scheduling, and visit log updates.
- Drift as a low-pass filter: It protects the "no visible change in a single session" invariant.
- Drift calibration target: The named target balances two risks: too fast becomes "a stat-management exercise"; too slow becomes "a screensaver where nothing matters."
- Drift monotonicity: Traits do not move down on neglect because "absence is fine, not penalized," implementing "no Tamagotchi."
- Mood state machine: Time of day, recent interactions, ambient events, and personality make mood transitions contextual and bird-specific.
- Mood persistence across sessions: The user should never see mood "snapping" to a default on tab open.
- Bird-to-bird interaction: Calls, wary spread, and chorus events make the aviary feel like "a small social system rather than a row of independent NPCs."
- Snapshot consumption and interpolation: Clients smooth movement between snapshots so a bird moves between perches with "no teleport."
- Fresh snapshots on visibility change, sleep gaps, and visible keepalive: These refresh current state after hidden tabs, resumed laptops, or long frame gaps.
- Conflict prevention through additive server-authored deltas: A client sends "user was present for 12 minutes," never "set boldness to 0.62," so personality conflicts do not exist as client writes.
- Sync conflict surface tone: In rare errors the product uses direct, clear, matter-of-fact language because "the exception is named and intentional."

**Frontend rendering and audio**

- Single horizontal scene with three perch zones and no panning, scrolling, or zooming: Birds choose perches by mood and personality, and "the user reads this as signal, not as layout control."
- Foreground/background separation and subtle parallax: NOT RECOVERABLE FROM PLAN
- Calm naturalist palette: The palette is specified as "calm and naturalist" with soft blues, greens, warm browns, and muted ochres, matching the product's naturalist stance.
- Idle micro-motion: Birds are "never still in a way that reads as paused"; mood-shaped movement lets the user read mood "without being told."
- First frame with birds mid-action: There is no wake-up or entry sequence because the simulation "has been" in progress on the server.
- Perch transitions as cross-fades: Cross-fades avoid flight-path motion and are the primary transition type in reduced-motion mode.
- State transitions through the animation state machine: This avoids "hard cuts."
- Loading state as a quiet field rather than a spinner: "A spinner says "machine"; we are not selling a machine."
- Responsive handling with no bird cropped out of frame: The scene compresses or widens while preserving aspect ratio so "No bird is ever cropped out of frame."
- Procedural call synthesis with no recorded audio: Recorded loops break the spell, and the chorus mechanic requires "real-time per-call variation."
- Chorus mixing as independent synthesis streams: This prevents "two identical loops in a phase-canceling stack" and keeps individual birds recognizable.
- Listen-in slow ramp and ambient background birds: A hard cut would turn the aviary into "a UI of soloable tracks"; a slow ramp preserves a place where multiple things continue at once.
- WebAudio fallback to graceful silence with captions: The no-recorded-audio rule is unconditional; "silence with captions is a better fallback than canned audio."

**Accessibility, performance, observability, and rollout**

- Keyboard navigation: "All interactive surfaces are reachable by keyboard," including birds, listen-in, offers, and settle.
- Focus indicators: They must be visible against the aviary background with a soft, high-contrast outline.
- Contrast: User-copy text must pass "WCAG AA contrast minimum."
- Bundle budget under 2MB gzipped: This drives the lean render path, code-splitting, procedural visual generation, and small assets.
- Time to first bird under 500ms: Below the threshold the aviary feels "already running"; above it the user notices load.
- Runtime 60fps idle motion: The requirement applies to a "30-minute session, not just the first minute."
- No memory growth over 30 minutes: Reused buffers, bounded contexts, and released references make this "a CI-enforced test, not a guideline."
- Synthetic checks: Automated browsers from common geographies watch whether the aviary keeps working as experienced by real clients.
- RUM aggregate only: Metrics exclude per-bird state and per-account interaction history so "the privacy boundary is honored at the metric definition level."
- Simulation tick latency p99 alarm: A 5-second alarm "catches degradation early."
- Birds-per-aviary ramp by aviary age: Time-based availability refuses the "gamification trap" and makes more birds "a function of time, not effort."
- Instrumentation from day one: Drift calibration, presence signal honesty, WebAudio errors, first-bird timing, and tick latency are monitored because the plan names them as calibration needs or likely failure modes.
