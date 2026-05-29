## System-level intent

1. **Server-authoritative personality, event-authored by clients.** The plan treats "The server is the only writer of personality state" as an "architectural invariant." This shows up in the service split (`api` validates but "never computes drift"; `sim` is "the only writer of personality vectors"), the data model ("Personality values are server-internal"), the API ("No endpoint accepts a personality value"), and sync ("api has read-only grants" while the `sim` service role writes personality columns).

2. **Presence must be honest because it is the dominant drift input.** The plan says "Presence is the conjunction of three independently-checkable signals" and "must be measured honestly." This recurs in the drift function ("presence-time dominant"), the three-signal detector (`visible`, focus, recent `pointermove` or `keypress`), server dwell caps, and the explicit risk around "tab open all night."

3. **The aviary is ongoing life, not a session that starts when the tab opens.** The render boundary says the server emits "intent + current phase, not frames" and "never sends 'start' events for ambient life." Frontend boot places birds "mid-motion"; mood persists across sessions; the tick runs "whether or not a client is connected"; background tabs stop rendering while "the server keeps ticking."

4. **Notice, never announce.** The plan repeatedly blocks UI that reports the user back to them: "No textual 'welcome back'," "No surface that reports the user's own visit behavior," notebook entries are "observations of the aviary, never observations of the user," and user-event narration is "still written as observations."

5. **Naturalist voice for the aviary; matter-of-fact voice for system copy.** Narrative surfaces use "lowercase, present-tense, bird-named, specific" prose, while auth/settings/sync/export/errors use "matter-of-fact copy." The API encodes this as a `voice` discriminator so the split is "mechanical, not per-developer judgment."

6. **Slow, monotonic movement toward expressive, with no punishment for neglect.** Drift uses a low-pass filter, `max(0, signal)`, additive deltas, and a non-decreasing plumage guard. The plan says "Neglect -> ambient, never wary/sad/faded" and frames the risk as too fast becoming "Tamagotchi-by-clicking" and too slow becoming "screensaver."

7. **Calibration is config plus tests, not hard-coded vibes.** Defaults marked **[CALIBRATION]** are routed through versioned config. The drift harness has "1-week-measurable / 3-week-visible / no-single-session-visible" assertions; open items are "routed through versioned config" so re-tuning is "a config change rather than a code change."

8. **Privacy is architectural, not policy.** The plan uses "Synthetic UUID everywhere," keeps email "exactly once," separates simulation from analytics, forbids analytics from reading simulation tables, and calls the boundary "architectural, not policy." Telemetry is "aggregate-only" and must not reconstruct "a user's relationship with their aviary."

9. **Sync is server-canonical, not a merge feature.** "Sync = server-canonical, not a feature." Two devices read the same record; clients submit events, not vectors; ordered additive deltas make the laptop/phone overwrite failure "unreachable."

10. **Accessibility is part of the product's charm, not a stripped fallback.** The plan says accessibility "ships with v1" and that reduced-motion, screen-reader, and audio-off users get an aviary that "feels alive." Reduced motion is "a calmer register of the same aviary"; narration is naturalist prose, "not ARIA-label automation."

11. **Audio must be procedural and recognizable, never canned.** The audio rule is "non-negotiable, no recorded audio." Per-bird seeds and motif libraries preserve "recognizable-across-drift," chorus is independent procedural voices, and silence-with-captions is preferred because "Silence-with-captions beats canned audio."

12. **Performance budgets are product gates.** Bundle size, time-to-first-bird, 60fps idle, memory growth, synthetic checks, and RUM are all treated as "gates" or CI checks. The plan ties these budgets to product feel: no spinner, first bird before non-critical assets, sustained runtime, and tick-latency alarms before users feel the aviary "running slow."

## Per-feature whys

### 1. Scope

- **Single horizontal scene:** The rationale is recovered from the rendering section: "no pan/scroll/zoom," three depth planes, and a render-time invariant that "all birds always visible." The scene is meant to stay calm, continuous, and inspectable rather than navigational.

- **2 starter birds:** NOT RECOVERABLE FROM PLAN

- **Cap of 7 birds:** NOT RECOVERABLE FROM PLAN

- **Three perch zones:** The plan gives them a rendering and simulation role: front/middle/back map to depth planes, and same/adjacent perch proximity feeds bird-to-bird influence.

- **Day/night on user-local time:** The plan says this honors "the user's morning is the aviary's morning" and avoids "a stored per-account clock that could drift."

- **Rare ambient weather:** The plan connects weather to mood inputs, notebook "weather moment" noteworthiness, and ambient state. The specific rarity level is NOT RECOVERABLE FROM PLAN.

- **Ambient leaf/feather drift:** The plan treats this as "pure client ornament" with "no server/per-leaf state," useful for first-frame life and idle atmosphere without adding canonical state.

- **Hidden personality vector:** The rationale is to keep personality server-internal, prevent numeric exposure, and make the server the only writer of drift-bearing state.

- **Fast-timescale mood enum:** Mood lets recent interactions, time-of-day, weather, and personality shape behavior quickly. It persists across sessions and is read through motion "without a label."

- **Slow-timescale drift via low-pass filter:** The plan uses this to make change slow, additive, monotonic-up, and testable, avoiding both "Tamagotchi-by-clicking" and "screensaver."

- **Bird-to-bird interaction:** The plan says this makes the aviary "a small social system, not independent NPCs."

- **Species pool (~6 species):** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick:** The tick advances canonical state from the append-only event log, lets mood/day-night continue while absent, and keeps clients from computing drift.

- **Return-greeting:** The rollout describes it as based on absence length, boldness, mood, and procedural variation, "explicitly not a canned arrival clip." It supports return as aviary behavior without textual "welcome back."

- **Listen-in:** Listen-in creates attention input for drift and audio focus through a slow mix re-balance. The plan explicitly rejects hard solo/mute because that would make it "a soloable-tracks UI - wrong product."

- **Offer (seed / song-fragment / still-pool):** Offers provide small signals for curiosity and boldness. The specific three offer types are NOT RECOVERABLE FROM PLAN.

- **Per-bird offer cooldown:** The plan makes the server authoritative over cooldown and ties calibration to "no single session moves a trait into a visibly different state," preventing interaction spam from dominating drift.

- **Settle:** Settle ends the presence window cleanly, quiets mood/calls, carries "~0 drift direction," and is equivalent to tab-close at the engine level so there is no penalty surface.

- **5s undo for settle:** NOT RECOVERABLE FROM PLAN

- **Field notebook:** The rationale is sparse, read-only "observations of the aviary," not a feed/log or a record of user behavior. The plan says "absence is the design" for no write/delete/annotate endpoints.

- **Presence accounting:** Presence is the "dominant drift input," so the feature exists to produce an honest, validated dwell signal rather than counting open tabs or visits.

- **Reduced-motion mode as in-scope v1:** The rationale is that accessibility ships "with v1" and reduced motion is a designed surface, "not a kill switch."

### 2. Architecture

- **Edge/web tier with inlined initial state snapshot:** The reason is the "<500ms budget"; the SPA shell includes a small state snapshot so first bird can render immediately.

- **API service:** `api` handles auth, reads, event writes, settings, export, and invites while validating and authorizing but "never computes drift." This preserves the server split and keeps drift in `sim`.

- **Simulation service:** `sim` owns the tick and is "The only writer of personality vectors," which enforces server-authored personality state and drift.

- **Auth service:** Auth owns magic links, sessions, email change/verify, and "the single encrypted email field," supporting the one-place-email privacy rule.

- **Notebook generator as a sub-component of `sim`:** It runs on the tick so notebook entries come sparsely from canonical state, not from client narration or user-maintained notes.

- **Narration generator from canonical state:** The plan prefers server-side narration so screen-reader prose is produced from "the same canonical state the visual reads."

- **Render pipeline boundary:** The server sends "intent + current phase," not frames or ambient "start" events, so the client can render 60fps and load already "mid-action" from a slow tick.

### 3. Data model

- **Synthetic UUIDs and one encrypted email field:** The rationale is privacy and boundary control: account IDs are used everywhere, and email appears "exactly once."

- **Reduced motion, captions, visit notifications, audio settings:** The plan uses settings to make accessibility/audio/social behavior explicit and account-level. The reason for `visit_notifications` defaulting false is recovered from the non-goal against push/email engagement notifications; the reason for the exact settings shape is otherwise NOT RECOVERABLE FROM PLAN.

- **Per-device sessions with revocation:** The plan specifies revocable sessions and "best-effort UA summary, no PII." The specific product reason for per-device session management is NOT RECOVERABLE FROM PLAN.

- **Stable `bird_id`:** The rationale is identity continuity: "STABLE FOREVER; never reassigned on rename/sync/migration," later tied to avoiding "deleting the bird the user knows."

- **Personality vector fields:** Boldness, social warmth, vocal frequency, plumage saturation, and curiosity are server-internal traits that shape drift, rendering, audio, mood, and offers without exposing raw numbers.

- **Plumage saturation non-decreasing:** The plan calls this "load-bearing" for "monotonic toward expressive" and enforces it at the write layer as a guard.

- **MoodState with `entered_at`:** This supports mood persistence and time-driven transitions, so mood does not "reset to neutral on tab open."

- **WeatherState:** Weather supports rare ambient state and mood/notebook inputs. More specific rationale is NOT RECOVERABLE FROM PLAN.

- **Append-only InteractionEvent with server timestamp:** Events are client-written inputs, while `server_ts` is the authoritative ordering key for additive drift and idempotent cursor processing.

- **NotebookEntry with local date label and prose:** The entry shape supports user-local, "naturalist" observations in lowercase present tense, newest-first, without exposing user behavior.

- **NarrationSnapshot as ephemeral / short-retention:** It supports live prose for accessibility while avoiding long-lived narrative state. The exact retention policy rationale is NOT RECOVERABLE FROM PLAN.

- **VisitInvite token and state:** The invite model supports "per-invite, opt-in, revocable" visits with one-time links and explicit lifecycle.

- **VisitLogEntry:** NOT RECOVERABLE FROM PLAN

- **Event log as source of truth for inputs; canonical state as source of truth for outputs:** This lets old events be trimmed once consumed without affecting state and avoids recomputing personality from full session history.

### 4. API surface

- **Magic-link auth:** The plan specifies magic links and cookie simplicity, but the product rationale for choosing magic-link auth over another auth method is NOT RECOVERABLE FROM PLAN.

- **`POST /auth/request-link` returns 202 always:** The reason is "no account enumeration."

- **One-time auth token with 15-min expiry:** The plan states the behavior; the specific rationale for 15 minutes is NOT RECOVERABLE FROM PLAN.

- **Email change request/verify with old email valid until new verifies:** The behavior preserves account continuity during email change.

- **Session listing and revocation:** The API supports the revocable session model. More specific rationale is NOT RECOVERABLE FROM PLAN.

- **Snapshot endpoint with render-facing derived fields:** The rationale is enforcement of "never exposed numerically"; raw personality floats are excluded and visual buckets are quantized.

- **Inlined initial snapshot and visibility/suspend recovery pulls:** The inlined snapshot supports the <500ms target; visibility and long-frame-gap pulls recover from suspend without pretending the aviary stopped.

- **Event write endpoint:** Events are appended to the log and stamped with `server_ts` so clients submit interactions while the server orders and later applies drift.

- **Batched event writes:** Batching presence pings with other events exists "to limit requests."

- **Offer cooldown enforcement in API:** Server-side enforcement makes cooldown authoritative even if the client greys the affordance.

- **Notebook endpoint read-only:** "No write/delete/annotate endpoints exist" because "absence is the design"; the notebook is generated, sparse observation, not user-authored content.

- **Narration stream endpoint:** It emits prose from the same canonical state, with idle cadence and priority bumps on user events, preserving visual/narrative continuity.

- **Account export:** Export exists because "the relationship is theirs"; it is the only place vector values leave the system, and they go to the owner.

- **Soft delete and restore:** The 30-day restore path supports "I changed my mind"; hard deletion later removes birds, vectors, notebook, and account-tied telemetry.

- **Invite creation:** Invites are opt-in per invite; "no invite -> no sharing."

- **Invite revocation:** Revocation cuts off the next visitor snapshot with matter-of-fact "visit no longer available," giving the host revocable control.

- **Unauthenticated visitor path:** It is read-only and has no `/events` access so visitor events are "never written" and visitor attention cannot affect host birds.

- **Default-off visit notifications:** This aligns with the non-goal against push/email engagement notifications and keeps social surfaces default-off.

- **API error/voice contract:** The voice discriminator keeps matter-of-fact system copy and naturalist narrative prose separated mechanically.

### 5. Simulation engine design

- **Idempotent, resumable tick with log cursor:** The tick can process new events in order, advance canonical state once, and recover safely after interruption.

- **Tick runs without connected clients:** The rationale is that mood/day-night keep evolving; "A long-idle account still ticks," but no presence means no drift.

- **Catch-up after downtime:** Bounded sub-steps make mood/day-night "land correctly" after outages without unbounded per-minute replay.

- **Drift function:** The low-pass "expressive signal" makes personality movement slow and additive, with `max(0, signal)` enforcing that neglect contributes nothing.

- **Presence-time/listen/offers/settle weights:** The plan gives their rough order: "presence-time dominant; listen-in strong; offers small; settle ~0 directional."

- **Calibration harness:** Tests assert measurable change after about one week, visible change after about three weeks, and no visible single-session jump, so tuning is test-driven.

- **Server-side dwell cap:** The cap exists so "a malformed client can't inflate drift."

- **Mood FSM:** A small enum with propensities, not hard edges, lets interactions, time-of-day, weather, and personality combine into fast-timescale behavior.

- **Mood persistence and soft daily-ish reset:** Mood persists across sessions, while long idle is handled by time-of-day pull instead of a hard "reset to content."

- **Bird-to-bird propagation:** Wary mood and chorus flags let birds influence neighbors, supporting the "small social system" intent.

- **Call grammar seed and call intent:** Stable per-bird seed/motif libraries keep calls recognizable, while timing/pitch modifiers allow mood and vocal frequency to modulate.

- **Caption text from call intent:** Captions match what was actually played because they are derived from the same call-intent, "not a stored string."

- **Notebook sparsity gate:** Cooldown plus noteworthiness keeps entries around "1 entry per few days" and prevents drift toward a feed/log even for active users.

- **Notebook naturalist grammar and content rule:** Entries stay bird/weather/moment-specific and never report "you visited," frequency, or streaks.

### 6. Sync model & presence

- **Server-canonical sync:** Multi-device sync is emergent because clients read one canonical record and write events, with no client-to-client sync, merge, or eventual consistency.

- **No last-write-wins on personality:** The plan prevents morning-laptop/lunch-phone overwrites by never accepting vector writes and by applying additive deltas in event-log order.

- **DB permission guard:** Personality columns are writable only by the `sim` role so the invariant is enforced "at the DB permission layer, not just in code review."

- **Conflict surfaces with matter-of-fact copy:** The plan avoids asking the user to arbitrate personality state because "there is nothing to arbitrate."

- **Three-signal presence detector:** It counts presence only when the tab is visible, focused, and recently active, with a long window because "watching birds without moving is the actual product."

- **Presence pings with validated dwell:** The detector only counts intervals where the conjunction held and stops when tab hides/blurs or activity lapses, producing honest drift input.

- **Settle and tab-close equivalence:** Both are terminal, neither penalized, and there is no "'you didn't settle' surface."

- **Background tab behavior:** Rendering stops for battery, presence stops because visibility fails, and the server continues ticking so return state is honest.

### 7. Frontend rendering pipeline

- **Canvas/WebGL scene with DOM chrome/accessibility:** Canvas/WebGL serves 60fps birds and ambient motion; DOM keeps top bar and accessibility surfaces as "real elements."

- **Bird sprites and quantized plumage render tier:** Compact SVG/procedural sprites support performance, and render tiers avoid raw personality float exposure.

- **Responsive no-crop scene:** The invariant "all birds always visible" is asserted in dev builds so responsive compression never hides a bird.

- **Loads-already-in-motion boot:** The plan rejects spinner, fade-from-static, wake-up animation, and entry sequence so the first frame is already alive.

- **Quiet field fallback:** For slow snapshot/cold cache, the fallback is a "quiet field" rather than a spinner, preserving the product register even before full state arrives.

- **Mood-shaped idle micro-motion:** Preening, scanning, head-tilt, and weight-shuffle let mood be read from movement, with "No status icon, tooltip, or label."

- **Procedural motion seeded per bird:** Seeds create variation so motion "never loops identically."

- **Snapshot interpolation:** Interpolation prevents teleports and lets slow snapshots become continuous motion.

- **Reduced-motion rendering register:** Cross-fades, slowed color shifts, retained calls/captions, and ongoing drift make it "a calmer register of the same aviary."

- **Thin top bar above the scene:** The plan says "Nothing else" and "No UI chrome inside the scene," keeping interaction chrome out of the aviary itself.

- **Top bar fade:** The fade reduces visual chrome during stillness while keyboard reachability remains intact because focus brings it back.

- **Ambient leaf/feather drift as client ornament:** It adds idle life at client cadence without server state or per-leaf canonical data.

### 8. Audio pipeline

- **Procedural synthesis:** The plan bans samples/loops so bird sound remains variable, lightweight, and not canned.

- **Per-bird motif library and seed:** Stable motifs and seeds keep each bird recognizable while vocal frequency and mood modulate timing/pitch.

- **Real-time chorus mixing:** Independent procedural voices create "emergent chorus" and avoid stacked loops that "phase-cancel audibly."

- **Listen-in slow ramps:** Slow re-balancing focuses one bird while others drop to ambient "never to silence," avoiding hard cuts and the wrong solo-tracks feel.

- **Settle and night audio:** Settle quiets calls through an evening shift; night dims calls but is "not a dead state" because a nightjar-like species remains active.

- **Bounded WebAudio graph:** Reused buffers/nodes and bounded contexts support the "no memory growth over 30 min" gate.

- **WebAudio fallback:** With old browsers, permission denial, or hardware issues, the product uses "silence + captions on by default" because no recorded fallback path exists.

### 9. Accessibility surfaces

- **Screen-reader narration:** The rationale is charm and continuity: naturalist prose from canonical state, "not ARIA-label automation, not a state list."

- **Narration cadence:** Slow idle updates and priority bumps prevent chatter while still reflecting user events as observations.

- **Call captions:** Runtime-generated captions match the actual call and provide the audio-off/audio-unavailable surface in naturalist voice.

- **Keyboard navigation:** Top-bar, scene, bird focus, listen-in, offers, and settle are keyboard reachable so the scene is operable without pointer input.

- **Visible focus indicator:** The high-contrast outline is needed to remain legible across bright and dim aviary states.

- **WCAG AA contrast:** The plan limits copy to chrome/accessibility surfaces, then requires those labels, settings, errors, captions, and displayed narration to pass AA.

### 10. Performance budgets & observability

- **Initial JS bundle <2MB gzipped:** This drives procedural audio, compact bird assets, and code-splitting of less-frequent surfaces, with a bundle-size CI check.

- **Time-to-first-bird <500ms:** Inlined snapshot, CDN delivery, first-bird-before-non-critical-assets, and no spinner serve the immediate alive-state goal.

- **60fps idle:** This is a "runtime budget, not just launch," sustaining the living scene over a 30-minute session.

- **No memory growth over 30 min:** Long-session CI catches leaked audio nodes, workers, and references so the aviary stays stable.

- **Aggregate-only telemetry:** The reason is the privacy boundary; metrics can measure health without per-account dimensions or bird state.

- **Do-not-collect list:** The plan refuses per-bird state and per-account interaction history to avoid reconstructing "a user's relationship with their aviary."

- **Simulation-tick p99 alarm >5s:** This is "early-warning before users feel the aviary 'running slow.'"

- **Synthetic perf fleet:** Automated browsers from common geographies measure first-bird, frame timing, and audio errors continuously.

### 11. Privacy & data boundary

- **No email-derived identifiers:** Synthetic UUIDs and one encrypted email field prevent identifiers from being derived from email.

- **Separate simulation and analytics datastores/roles:** The hard boundary is enforced by storage and role grants, not just policy.

- **Visitor isolation:** The visit path has no event-write capability, so visitor attention "can never drift the host's birds."

- **Soft-then-hard deletion:** Soft deletion allows sign-in restore for 30 days; hard deletion removes birds, vectors, notebook, and account-tied telemetry.

- **Export JSON:** Export sends birds, names, current vectors, moods, notebook, and settings to the verified owner, preserving "the relationship is theirs" while not exposing vectors in product UI.

### 12. Rollout

- **Foundation first:** Accounts/auth, UUIDs, write grants, event ingestion, and tick skeleton "validates the architecture before the engine."

- **Engine behind calibration tests:** Personality drift, mood FSM, and bird-to-bird work are gated by the 1-week/3-week calibration assertions.

- **Render + audio after engine:** The plan groups scene, motion, interpolation, procedural calls, chorus, listen-in, and fallback as the sensory layer that reads engine outputs. A more specific sequencing rationale is NOT RECOVERABLE FROM PLAN.

- **Interactions after render/audio:** Return-greeting, offers, settle, and notebook depend on procedural variation, cooldowns, and sparsity gates. A more specific sequencing rationale is NOT RECOVERABLE FROM PLAN.

- **Accessibility shipped with the above:** The rationale is explicit: accessibility surfaces ship "with the above, not after."

- **Social after accessibility:** NOT RECOVERABLE FROM PLAN

- **Hardening last:** Perf budgets, synthetic fleet, RUM, error budgets, and privacy-boundary audit are grouped as release gates after core surfaces exist.

- **Birds-per-aviary age ramp:** New birds are driven by aviary age "never visit count/score/paid tier," preserving the non-gamified relationship model.

- **Instrument from day one:** The plan names tick latency, first-bird-render, frame timing, audio errors, and bundle size as aggregate-only signals to watch from the start.

### 13. Risks

- **Drift miscalibration mitigations:** The calibration harness and versioned config exist because drift is "the central risk"; too fast is "Tamagotchi-by-clicking," too slow is "screensaver."

- **Presence dishonesty mitigations:** The three-signal detector, server dwell cap, and explicit unfocused-tab test exist because "tab open all night" would silently inflate drift.

- **Sync correctness mitigations:** Server-only writes, ordered deltas, replay safety, backups, and stable bird identity defend against "personality loss" and "deleting the bird the user knows."

- **Audio uncanniness mitigations:** Procedural-only build checks, independent chorus voices, recognizability tests, and slow listen-in ramps protect against looped/canned audio that "breaks the spell."

- **Accessibility fallback mitigations:** Canonical-state narration, state-list review gates, reduced-motion visual QA, and same-release a11y keep the product from becoming a stripped fallback.

- **"Notice never announce" mitigations:** Scope guards, voice discriminator, and surface audits prevent welcome toasts, streaks, notifications, or user-behavior-reporting UI from slipping in.

- **Perf erosion mitigations:** Bundle and memory CI, synthetic fleet, tick alarms, and RUM first-bird watch keep budgets from becoming advisory.

- **Notebook drift mitigations:** Sparsity tests, content rules, and naturalist-prose review prevent the notebook from becoming frequent or user-behavior reporting.

### 14. Open calibration items

- **Tick cadence, presence window, offer cooldown, drift weights, narration cadence, notebook sparsity, personality seed values, and add-a-bird curve:** The rationale is that each has "a defensible default" and a test/harness path, so re-tuning remains a config change rather than a code change.
