## System-level intent

- **v1 is shaped by explicit refusals.** The plan says "The shape of v1 is what's left after those subtractions" and that adjacent feature pitches should be "rejected at the design step, not the implementation step." This shows up in the "Out of scope (v1) — explicit refusals" list, in "What we don't ship with a toggle," and in the risk called "Just one streak counter."

- **The aviary is not a game loop or a Tamagotchi.** The plan repeatedly refuses "achievements, streaks, levels, scores, badges, counters," "Tamagotchi-style decay," "negative drift on neglect," and visible distress. Drift is "monotonic toward expressive," slow, bounded, and positive-presence-driven.

- **Server-canonical state prevents merge problems and last-write-wins.** The plan's architecture says the server is "the only writer" of personality, mood, notebook entries, and visit tokens, while the client writes only interaction events. The sync section says "There is no merge problem" because personality, mood, perch, and notebook entries are server-only.

- **The product voice is naturalist, quiet, and not user-addressing.** Notebook and narration are "lowercase, present-tense," with "no second person," "no exclamation," and no visit-frequency observations. Error, sync, session, and unsupported-browser surfaces are the named exception: they use "matter-of-fact copy" and are "never naturalist."

- **Presence should feel alive without feeling manipulative.** The plan asks for a first frame that is "the aviary, mid-motion," no spinners, no "Welcome back!" surfaces, and notebook entries that are "never per session." The user should not see trait movement "in a single session" but should see it "after weeks."

- **Recognition comes from procedural stability, not canned assets.** Bird calls use a stable `call_signature_seed`, procedural motif variation, and client-side synthesis so the user recognizes a bird "by ear after weeks." The plan refuses "recorded audio assets," "canned greetings," and "recorded-audio fallback."

- **Accessibility is a designed surface, not a fallback.** The plan says accessibility is "designed in, not retrofitted," owned by the same engineers as visual/audio, and ships with v1. Reduced-motion is "a first-class rendering mode, not a fallback"; screen-reader narration reads from the same state cache as the visual renderer.

- **Privacy boundaries are architectural, not policy-only.** The plan keeps analytics on a "separate ingestion path," says telemetry is "strictly aggregate," and rejects metrics that require reading a bird, notebook entry, or interaction event "at the metric definition step, not the policy step."

- **Performance budgets drive product and implementation choices.** The 2MB bundle cap, time-to-first-bird target, 60fps idle, flat memory growth, snapshot payload size, and listen-in ramp are described as budgets with failure modes. Bundle composition choices are explicitly downstream of the 2MB cap.

- **Calibration and review surfaces are part of the design.** Drift, mood dwell time, notebook voice, call grammar, bundle size, first-bird-render time, narration, captions, and keyboard behavior all get fixture tests, synthetic checks, style-spec tests, or human review during alpha and beta.

## Per-feature whys

### Scope

- **Web-only single-page application served from a CDN edge**: The plan treats web-only as part of v1's subtraction-based scope; native apps are explicitly refused for v1, with a possible native app only later "if/when the team has the time and the case is clear." CDN/static delivery also supports the web bundle and performance budgets.

- **Single-user accounts**: The plan pairs this with "one account, one aviary, one user," refuses household/shared aviaries, and separates account/auth from simulation state.

- **Email magic link authentication**: NOT RECOVERABLE FROM PLAN

- **Two starter birds, species-pool ramp, and hard cap of seven**: The cap aligns with recognizability and audio-mix limits. The post-launch section names only a possible future audio-mix enhancement that raises the "recognizability ceiling above seven birds." New-bird pacing is tied to aviary age and calibrated against drift rhythm so the aviary grows over weeks to months, not as a score loop.

- **Server-authoritative slow simulation tick**: The tick makes state server-authored, avoids client merge paths, and sets latency expectations at "at most one tick." The slow cadence supports the plan's slow, presence-driven drift and mood persistence.

- **Personality vectors drifting monotonically toward expressive**: The plan's why is explicit: there is "no negative branch," no "negative drift on neglect," and no Tamagotchi-style decay. Drift must be measurable after about a week and visible after about three weeks so the user does not see single-session trait movement.

- **Per-bird mood with time-of-day and ambient-event modulation**: Mood keeps the aviary from snapping to neutral on tab open. Time, weather, recent events, and personality create soft pulls, dwell times, and temporary mood-level effects without changing personality negatively.

- **Procedural call grammar and stable per-bird call signatures**: Client-side synthesis satisfies the audio constraint and bundle budget. Stable seeds make calls recognizable "across devices and mood states"; vocal drift changes how often a bird calls, "not what Pip sounds like."

- **Sit/watch passive presence**: Presence is a qualified input to the drift function. It gives all five traits a small upward push while preserving the no-decay rule.

- **Listen-in**: Listen-in gives a specific bird stronger social_warmth and vocal_frequency drift, while the audio mix makes the interaction feel like listening to "the same aviary, rebalanced," not switching channels.

- **Offer made / accepted / ignored effects**: The plan explains offer effects generally: offer accepted nudges curiosity and content; offer made nudges boldness; ignored offers can nudge mood toward wary but "only at the mood level."

- **Seed / song fragment / still pool offer types**: NOT RECOVERABLE FROM PLAN

- **Settle**: Settle has "no direct drift effect" and "ends the presence window cleanly." It is also an aviary-level event and a sync/poll trigger.

- **Return-greeting procedural variation**: The rollout metrics include return-greeting variation to ensure the system produces "real variation, not three rotated variants."

- **Field notebook**: The notebook is read-only naturalist prose because the generator writes about "the aviary's behavior" rather than observing the user. It is roughly one entry every few days, "never per session," with hard voice rules to avoid visit-frequency observations.

- **Multi-device sync**: Sync is "a property of the architecture, not a feature." The why is conflict prevention: devices read the same server record, client events are append-only, and personality/mood/perch/notebook have no client writer.

- **Optional visit invitations**: The feature is off by default, per-invite opt-in, revocable, and read-only so it does not become a social-network surface. Revocation takes effect on the next snapshot pull.

- **Accessibility: screen-reader narration, reduced-motion, captioning, keyboard, contrast**: These ship with v1 because accessibility is "designed in, not retrofitted." Each surface preserves the same aviary and voice rather than becoming a stripped fallback.

- **Aggregate-only RUM and synthetic perf checks**: The reason is privacy-clean observability. The plan measures timings, histograms, success rates, and aggregate distributions while deliberately not measuring per-bird state, notebook contents, visit log contents, or relationship-reconstructing data.

- **Soft-then-hard account deletion**: The soft-delete window lets the user sign in during 30 days to recover.

- **Per-account JSON export**: NOT RECOVERABLE FROM PLAN

### Architecture

- **Three services: web client, simulation service, account/auth service**: The split keeps rendering/input/audio in the client, simulation state in one writer, and email/session/account work isolated. The account/auth service reads `account_id` only and "never queries the simulation database."

- **Simulation service as only writer to per-account simulation tables**: This is the core guard against last-write-wins and client mutation of personality, mood, notebook, and visit tokens.

- **Account/auth service isolation**: The rationale is privacy and separation: email is encrypted and account/auth never queries simulation data.

- **Client writes only interaction events**: The event log captures user actions without letting the client write absolute personality values, mood, perch position, or notebook entries.

- **No client-to-client sync**: There is no merge problem because two devices read the same server record and never reconcile local personality state.

- **No realtime push in v1**: NOT RECOVERABLE FROM PLAN

- **Thin state cache rebuilt on load**: The client cache is ephemeral because the server is canonical; rebuilding on every page load prevents persisted client state from becoming a second source of truth.

- **Narration generator reading from the same state cache**: This keeps screen-reader output "voice-continuous with the visual surface."

- **Audio graph created lazily on first user gesture**: The stated reason is browser autoplay policies; before that calls are silent or caption-only.

### Data model

- **Synthetic `account_id` and encrypted email**: `account_id` is "the only identifier used outside this row"; email is stored once, encrypted, and never used as an identifier. This supports the privacy boundary.

- **Aviary timezone and weather fields**: Timezone drives day/night; current weather is short-lived server state used by mood and rendering.

- **Stable `bird_id` never reused or replaced**: The plan says the identity rule is enforced by never having a code path that updates `bird_id` or replaces a row.

- **`personality` as server-only raw numbers**: Raw personality numbers are never returned to the client, because client paths must not write or expose personality values and the product should not become a system showing states.

- **`call_signature_seed`**: The seed makes the procedural call grammar deterministic enough that the same bird has the same recognizable call across devices and mood states.

- **Append-only interaction event log**: Append-only events give the tick ordered input and keep clients from writing derived state. Archival keeps recent events online and preserves older rows for "drift debugging, not for runtime."

- **Presence window rollup**: The drift function reads this aggregate rather than raw session observations. The raw log remains available for re-derivation if the rollup is wrong.

- **Notebook entries not user-editable**: The generator decides when an entry is warranted, roughly one per few days and never per session, to preserve the product voice.

- **Visit / invite revocation fields**: Revocation is explained as taking effect on the next snapshot pull.

- **One-time-use bcrypt-hashed visit tokens**: NOT RECOVERABLE FROM PLAN

- **Privacy boundary across tables and analytics**: Simulation state, notebook, visit, bird, and interaction data are never read by analytics or included in telemetry so aggregates cannot be re-per-account'd.

### API surface

- **Snapshot read**: The snapshot is small and excludes personality numbers. Polling on visibility, wake gaps, keepalive, and interactions lets the client pick up state transitions without realtime push.

- **Interaction event write**: Idempotency on `client_event_id` prevents duplicate event effects; server ordering by receipt time lets the tick consume a single ordered stream.

- **Magic-link auth endpoints**: NOT RECOVERABLE FROM PLAN

- **Account settings for timezone and preferences**: Settings hold timezone, reduced-motion, captions, visit notification toggle, and other preferences so accessibility and local day/night behavior persist.

- **Display name in account settings**: NOT RECOVERABLE FROM PLAN

- **Account delete endpoint**: Delete sets `soft_delete_at` and can be recovered for 30 days.

- **Account export endpoint**: NOT RECOVERABLE FROM PLAN

- **Visit / invite endpoints**: The endpoints implement opt-in, revocable, short-lived read-only visit access and a host visit log, keeping visits controlled rather than social.

- **Single predictable error envelope**: The why is copy consistency. Errors are "matter-of-fact," HTTP-standard, and written for the user in the surface where they appear.

### Simulation engine design

- **Per-account tick transaction and locks**: Locking accounts due for update and committing all updates in one transaction keeps tick results consistent. Per-account locking lets the global tick run in parallel.

- **Separate slower day/night and weather loop**: NOT RECOVERABLE FROM PLAN

- **Low-pass drift function**: It keeps personality movement slow, bounded, additive, server-authored, and visible only over weeks.

- **Personality-shaped drift**: High traits near their cap move less under the same input, so a bird already at the ceiling does not visibly over-move.

- **Mood transition function**: Time, weather, recent events, and personality produce mood-level changes while keeping personality protected from negative effects.

- **Nearby bird alarm call effect**: NOT RECOVERABLE FROM PLAN

- **Call grammar runtime**: It uses stable seed, mood modulation, vocal frequency gating, and timing seeds so calls stay recognizable while still feeling natural.

- **Notebook generation gates and templates**: The time-since-last-entry and noteworthy-event gates prevent per-session writing; templates keep prose in the same product voice and about aviary behavior.

- **Style-spec tests for notebook prose**: Generic phrasing and voice regressions fail the build to protect the naturalist voice.

### Sync model

- **Canonical state split between simulation and account services**: Each service owns only its authoritative domain; the client owns only cache plus outbox.

- **Conflict prevention rules**: Server-only personality, mood, perch, and notebook entries eliminate two-writer scenarios. Append-only event ordering makes simultaneous device interactions additive rather than corrupting.

- **Offline event replay**: IndexedDB queueing and idempotent replay preserve actions after reconnect. If the queue is older than one hour, the client gives a matter-of-fact notice that some actions were not recorded.

- **Session-expired mid-action handling**: The action is rejected with "sign in again" and no partial state is written.

- **Visit revocation**: Returning 410 Gone with "visit no longer available" makes revocation clear and matter-of-fact.

### Frontend rendering pipeline

- **Single full-viewport canvas plus SVG top bar**: NOT RECOVERABLE FROM PLAN

- **Background, middle plane, foreground layers**: These layers create soft sky/foliage, three perch zones, birds, and occasional ambient foreground without per-leaf server tracking.

- **Procedural bird composition instead of sprite sheets**: The plan says this keeps the bundle small and lets the renderer compose variation from species, personality, mood, and seed.

- **Interpolated perch, pose, micro-motion, and color**: Interpolation keeps birds continuous between server snapshots and avoids frame-stepped movement.

- **No entry animation on reload**: The first frame is "the aviary, mid-motion," so the product does not feel like an app starting up.

- **Reduced-motion rendering mode**: It replaces micro-motion with cross-faded poses and removes leaf drift while preserving calls, captions, notebook, and day/night color. The why is that "the aviary is still the aviary — just rendered in a different visual register."

- **Quiet field loading state and no spinner**: The user should perceive "the aviary catching up, not the app loading."

- **First bird fly-in exactly once per bird**: NOT RECOVERABLE FROM PLAN

- **Thin top bar with four controls and fade behavior**: It keeps only account/settings, accessibility, notebook, and offer affordance visible.

- **Dirty-region rendering, cached SVG composition, and micro-update dropping**: These exist to sustain 60fps and keep perch-position updates more important than pose micro-updates under load.

### Audio pipeline

- **Single WebAudio graph**: It mixes per-bird calls, listen-in rebalance, master gain, destination, and a procedural ambient bed from one context.

- **Listen-in mix ramp**: The ramp avoids hard cuts that "feel like a UI of soloable tracks" and preserves "the same aviary, rebalanced."

- **Procedural call synthesis**: It avoids sample libraries and recorded assets, stays within bundle budget, and lets species, seed, and mood shape living variation.

- **AudioWorklet with reused node pool**: The plan's why is avoiding main-thread blocking and minimizing per-call allocation.

- **Caption generation in parallel with synthesis**: Captions come from the same call parameters at schedule time, so caption text matches the actual procedurally generated call.

- **Graceful silence with captions when WebAudio fails**: The aviary remains readable without recorded fallback. The plan says a canned fallback would "feel canned," and a recorded-quality procedural fallback would "blow the bundle budget."

- **Audio observability without recording or transcription**: Aggregate counters and state changes catch pipeline issues while preserving that there is no recording, transcription, or mic input.

### Accessibility surfaces

- **Screen-reader narration**: It reads from the same state cache as visuals and updates a polite live region so screen-reader output stays in the same naturalist voice and timing.

- **Narration fixture tests**: The tests protect lowercase, no second person, no announcement framing, and no visit-frequency observations.

- **Reduced-motion preference persistence**: System and user preference both reach the same first-class mode; account preferences persist the choice.

- **Call captioning**: Runtime captions pair each call with prose generated from mood, motif, and variation, giving an audio-off path that still reads the aviary.

- **Keyboard navigation and shortcuts**: All interactive surfaces are keyboard-reachable; the keymap supports top bar, bird focus, listen-in, offer, notebook, settle, and help.

- **Focus indicators and contrast**: High-contrast focus and WCAG AA copy keep controls and captions readable across bright and dim aviary states.

- **Accessibility settings copy**: Matter-of-fact labels and help text avoid framing one choice as a naturalist-voice recommendation.

### Performance budgets and observability

- **Initial JS bundle cap**: Above 2MB, "time-to-first-bird becomes unrecoverable," so procedural audio, procedural SVG birds, code-splitting, inline icons, and runtime seed variation follow from the cap.

- **Time-to-first-bird target**: Above 500ms, "the aviary feels like it's loading."

- **Idle 60fps and flat memory**: A 30-minute session must hold performance; procedural audio buffers are reused and CI verifies no client memory growth.

- **Simulation tick latency p99**: The p99 alarm fires before users notice the aviary "running slow."

- **Snapshot payload targets**: Small payloads drive pull frequency and battery cost.

- **Aggregate RUM and synthetic checks**: These verify budgets across real and automated browsers without reading per-bird or per-account relationship data.

- **Deliberately unmeasured data**: The plan refuses measurement of per-bird state, interaction history, personality values, notebook contents, and visit log contents to prevent relationship reconstruction.

### Rollout

- **Internal alpha**: It isolates bird engine and tick, drift calibration, mood transitions, notebook style, accessibility, and audio before wider exposure.

- **Closed beta**: It tests full end-to-end behavior with a small cohort and calibrates drift function, mood dwell times, and notebook entry rate.

- **Limited public launch**: New-bird pacing stays slow while the team confirms calibration across a few hundred accounts.

- **General availability**: The two-starter-bird flow, magic-link auth, and visit invitations are live after calibration.

- **Day-one instrumentation**: Drift distribution, mood dwell time, notebook rate, return-greeting variation, accessibility engagement, audio success, fallback rate, snapshot latency, payload size, and first-bird-render time are measured to validate calibration and budgets.

- **Features not shipped with a toggle**: Streaks, notifications, public surfaces, visitor show-off mode, and personality debug views are not toggles because "a toggle implies the feature can come back."

- **Post-launch possible native app, pacing refinement, audio-mix enhancement**: These are possible future levers only; they are "not assumed, not planned."

### Risks

- **Drift calibration mitigation**: Tests and beta measurements prevent the aviary from feeling like a longer-timescale Tamagotchi or like a screensaver.

- **Sync correctness mitigation**: CI and code review protect the single-writer rule so no client personality write path appears.

- **Audio uncanniness mitigation**: Human review and a "sounds alive" corpus protect against technically varied but emotionally flat synthesis.

- **Accessibility regression mitigation**: Shared engineering ownership, fixture tests, and preview affordances protect reduced-motion, caption voice, keyboard navigation, and screen-reader prose.

- **Performance regression mitigation**: CI bundle gates and synthetic throttled-network checks block changes that break the 2MB and 500ms budgets.

- **Voice drift mitigation**: Style-spec tests and UI-string review prevent "welcome back," "you've been here X days," and status-chip language that makes the aviary read as a system showing states.

- **Privacy boundary erosion mitigation**: Synthetic UUID rules, encrypted email isolation, warehouse access limits, metric-definition rejection, and quarterly audit keep PII and simulation state out of analytics.

- **"Just one streak counter" mitigation**: The refusal is named, untoggleable, un-A/B-testable, and enforced by review against behavior counts.

- **Browser fragmentation mitigation**: Supported browsers are the last two major versions of Chrome, Safari, Firefox, and Edge; older browsers get a matter-of-fact unsupported-browser surface rather than compatibility paths.

### Defensible calls where the PRD is silent

- **Tick cadence: 60 seconds per account**: The plan says the PRD says "slow, ~once per minute," and 60 seconds is the starting value.

- **Mood set: wary, content, curious, drowsy, alert**: The examples are used as the working set because the exact set is finalized in implementation.

- **Species pool size: about six**: The working number follows the plan's "about six" statement.

- **New-bird pacing**: Weeks-to-months intervals tied to aviary age follow the rhythm that "an aviary a few months old offers a third bird"; exact intervals are tuned in build.

- **Field notebook entry gap**: A minimum of about 36 hours for active aviaries is the working floor because the target is roughly one entry every few days.

- **Presence activity window**: Three minutes leans toward the longer side of "a few minutes."

- **Magic-link expiration: 15 minutes**: NOT RECOVERABLE FROM PLAN

- **Visit-link expiration: 30 days**: NOT RECOVERABLE FROM PLAN

- **Account soft-delete window: 30 days**: The plan says this is explicit and elsewhere says the user can sign in during the window to recover.

- **Reduced-motion cross-fade duration: about 600ms**: This is a working value because the plan says the PRD does not specify.

- **Listen-in mix ramp duration: about 1.5s**: The plan ties this to "gradual" and the requirement that listen-in feel like listening, not switching channels.

- **Snapshot keepalive schedule**: About 15 seconds idle and 60 seconds during active interaction follows "low-frequency keepalive," visibility change, and long render-frame gap requirements.

### What this plan is not

- **Not a UI design**: The reason is that palette, perch-zone positions, bird proportions, and top-bar treatment belong in a separate design system spec.

- **Not a content policy**: Notebook voice is specified here, but per-template prose is built and reviewed during build.

- **Not a security model**: Auth flow is described, while threat model, key management, and rate-limiting values are separate.

- **Not a data-retention policy**: The 30-day online / cold-storage event-log archival is only a starting position; retention and right-to-be-forgotten mechanics are separate.
