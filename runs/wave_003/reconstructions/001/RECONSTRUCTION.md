## System-level intent

1. Server-canonical state, client as renderer, and no last-write-wins.
   Shows up in "client is a pure renderer of snapshots," "server-canonical state," "The simulation tick is the only writer of canonical state," and the sync section's "read-many/write-one architecture." The plan treats this as the concrete implementation of the "no last-write-wins" rule.

2. Interaction is additive and non-punitive.
   Shows up in "additive, never absolute," "drift never moves down on neglect," and "Deltas pass through `max(0, delta)`." The plan also says this makes additive deltas "physically true, not just conventionally true."

3. Calm relationship-deepening, not gamification or Tamagotchi mechanics.
   Shows up in the explicit non-goals: "no achievements/streaks/levels/scores/badges," "no death, hunger, decay, distress states," and later risk language about "Gamification creep via 'harmless' feature requests." The calibration risk says tuning must never become a way to make the product more "engaging" in the gamification sense; it tunes "felt-aliveness pacing only."

4. Procedural aliveness instead of recorded or looped software.
   Shows up in "Procedural, client-side WebAudio call synthesis; no recorded audio, ever," "never replays a stored buffer verbatim," and the risk that "looped audio is the audible signature of dead software." The same intent appears in probabilistic mood transitions so "the same inputs don't always produce the same output."

5. Accessibility surfaces are first-class product surfaces with the same voice.
   Shows up in "naturalist prose, not state-list," "reduced-motion mode (its own designed rendering, not animations-off)," and "Accessibility surfaces built in parallel with phase 2-3, not after." The narration text is "the same server-authored prose the visual/notebook surfaces use, so voice is identical across modalities."

6. Performance budgets are design constraints, not cleanup tasks.
   Shows up in the hard targets for "≤2MB gzipped initial JS," "<500ms time-to-first-bird," "60fps idle motion," and "no memory growth," plus CI gates for bundle size and first-bird timing as "hard failure, not a warning."

7. Privacy boundary and aggregate-only observability.
   Shows up in "email is never a key anywhere outside the account record," encrypted email fields, "aggregate-only" observability, "all anonymized, no per-account dimension," and no "day-one engagement dashboard."

8. Tunable calibration through config and CI, not buried constants.
   Shows up in the fixed-but-configurable "60s" tick cadence, "3 minutes" presence window, "global calibration constant," and the "synthetic aviary" harness that asserts "1-week-instrument / 3-week-visible targets."

9. Deliberately constrained social surface.
   Shows up in "per-invite opt-in, read-only ambient, revocable, off by default, no notifications by default" and the risk that viewer counts or "someone's watching" indicators would push visits toward co-presence and "should be treated as a different product decision."

## Per-feature whys

### 1. Scope

- Single-user accounts, magic-link auth, one canonical aviary per account, multi-device sync via server-canonical state: The why is sync correctness without merge UI: one canonical state, "no per-device copy," and "there is no conflict to resolve" because clients write events and the tick writes state.
- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN
- Server-selected species: NOT RECOVERABLE FROM PLAN
- User-named birds: NOT RECOVERABLE FROM PLAN
- Cap of seven birds: NOT RECOVERABLE FROM PLAN
- Age-gated new-bird offers: The plan says pacing is a config table so the "pacing curve" can be tuned post-launch to keep the "relationship-deepening feel right."
- Server-side simulation tick: The tick runs on a schedule regardless of client connection so mood and personality persist across sessions, and a scheduled worker separates simulation from request/response traffic.
- Procedural WebAudio calls with no recorded fallback: The rationale is that calls should vary every time, avoid "stacked recorded loops," avoid audio files in the bundle, and honor "no recorded-audio fallback, ever."
- Return-greeting: NOT RECOVERABLE FROM PLAN
- Listen-in: The rationale is focused attention without fully muting the aviary: focused bird gain ramps up, others ramp down "slow, not a hard cut," and others' gain floor is "never fully muted."
- Offer cooldown: The rationale is to prevent spammed offers from reaching the log as countable events past the first.
- Seed offer: NOT RECOVERABLE FROM PLAN
- Song fragment offer: NOT RECOVERABLE FROM PLAN
- Still pool offer: NOT RECOVERABLE FROM PLAN
- Settle: NOT RECOVERABLE FROM PLAN
- Field notebook: The rationale is sparse, naturalist, read-only prose generated from tick output and event log, with provenance for debugging/QA never exposed to client.
- Single horizontal scene: NOT RECOVERABLE FROM PLAN
- Three perch zones: Perch zones support boldness/mood rendering: boldness biases toward front, wary mood biases toward back, and weighted picks keep similar birds from always sitting in the same configuration.
- Day/night cycle on local time: The plan recomputes `time_of_day_phase` from last-known timezone offset per request "to avoid stale day/night on long-idle reconnects."
- Ambient weather: Weather is one of the independent signals in mood transitions.
- Rarity of ambient weather: NOT RECOVERABLE FROM PLAN
- Top-bar chrome fading on idle: The rationale is real focusable DOM elements for accessibility plus CSS-opacity-driven idle fade.
- Visit-invitation social feature: The rationale is a deliberately constrained read-only ambient visit surface, with opt-in, revocation, no notifications by default, and no co-presence drift or viewer-count surfaces.
- Screen-reader narration: The rationale is naturalist prose, not a state list, with the same server-authored voice as visual/notebook surfaces.
- Reduced-motion mode: The rationale is its own designed render path that still "feels alive," not animations-off or an afterthought.
- Call captioning: The rationale is caption-audio match by deriving captions from the same procedural call grammar at runtime.
- WCAG AA contrast: The rationale is that required-reading chrome/settings/account/error/caption text remains readable; the canvas scene carries no required-reading user copy.
- Full keyboard navigation: NOT RECOVERABLE FROM PLAN
- Account export: NOT RECOVERABLE FROM PLAN
- 30-day soft account deletion: NOT RECOVERABLE FROM PLAN

### 2. Architecture

- Five services behind one BFF: The rationale is failure-domain separation: simulation tick has a "fundamentally different scaling and failure profile" and a slow tick must not block sign-in or snapshot reads.
- Client writes only to event log: The rationale is that the client never writes personality or mood and the API schema has no fields for those mutations.
- Simulation tick as only writer: The rationale is the implementation of "no last-write-wins."
- Three-layer render pipeline: The rationale is to keep ambient ornament "rendering noise, not product state," keep snapshots small, and keep the "no per-leaf state" rule honest.

### 3. Data model

- Synthetic UUID identifiers: The rationale is that email is never a key anywhere outside the account record.
- Encrypted account email: The rationale is that the account record is "the only place email lives."
- `visit_notifications_enabled` default false: NOT RECOVERABLE FROM PLAN
- `reduced_motion_opt_in` distinct from media query: The rationale is explicit user control in both directions beyond OS preference.
- `captions_enabled` default false and true if WebAudio unavailable: The rationale is that silence-with-no-captions would be a worse failure than overriding a default.
- Aviary as separate entity even though 1:1 with account: The rationale is so schema does not have to change if multi-aviary is ever revisited.
- `timezone_offset_minutes`: The rationale is day/night calculation when no client is connected and avoiding stale reconnect behavior.
- Bird stable lifetime id: NOT RECOVERABLE FROM PLAN
- Personality floats normalized to `[0,1]`: NOT RECOVERABLE FROM PLAN
- New-bird trait seeds in `[0.35, 0.65]`: The rationale is to avoid a new bird reading as maximally bold or shy on day one, which would look like the engine already "knows" the bird.
- Mood enum including `settled`: The rationale is that the spec treats "settled" as a distinct rendered/narrated state from "drowsy."
- SpeciesDefinition as static reference data: The rationale is that adding a 7th species later should be a data change, not a code change.
- InteractionEvent append-only log: The rationale is that the tick consumes events in `received_at` order and additive deltas remain correct under concurrent writes.
- PresenceWindow as derived state: The rationale is presence is computed from consecutive pings, not stored as raw client state.
- NotebookEntry trigger provenance hidden from client: The rationale is debugging/QA without exposing provenance to the client.
- VisitInvitation one-time link token hashed at rest: NOT RECOVERABLE FROM PLAN
- Session per-device and revocable: The rationale is session list/revoke and multi-device hardening.

### 4. API surface

- `/aviary/snapshot` with derived render parameters, not raw personality values: The rationale is the literal implementation of "personality is never exposed numerically."
- `etag` / `snapshot_version`: The rationale is client-side change detection.
- Snapshot polling on visibility, render gap, and 20s keepalive: The rationale is that the 60s tick cadence makes push infrastructure "unjustified complexity" for the freshness gain.
- `/aviary/notebook` paginated and read-only: NOT RECOVERABLE FROM PLAN
- Account sessions list/revoke: The rationale is per-device revocation.
- Account export email link: NOT RECOVERABLE FROM PLAN
- Soft delete / restore within 30-day window: NOT RECOVERABLE FROM PLAN
- `/aviary/events` closed enum and no state mutation fields: The rationale is to reject anything resembling state mutation.
- Idempotency key on event writes: The rationale is retries on flaky connections should not double-count offers or double-pair listen-in start/end.
- Batched presence pings: The rationale is bounded write volume and matching the "few minutes" granularity drift needs.
- Magic-link rate limiting: NOT RECOVERABLE FROM PLAN
- Magic-link consume invalidates token atomically with DB-level consumed flag: The rationale is to close the replay race.
- Visit consume issues visit session token: The rationale is a narrower-scoped read-only token that cannot reach `/aviary/events`.
- Host visit log: NOT RECOVERABLE FROM PLAN
- `/aviary/narration` returned alongside snapshot: The rationale is avoiding a second connection class and feeding the accessibility layer from `snapshot.narration`.
- No call-caption endpoint: The rationale is captions stay in sync with the audio engine without a round trip.

### 5. Simulation engine design

- Tick loop fan-out per aviary: The rationale is independent processing and error isolation so one malformed aviary does not block other ticks.
- Tick runs when events exist or mood timers warrant transition: The rationale is that time-of-day always advances and mood can transition absent new events.
- Presence-time delta with diminishing returns: The rationale is an open-all-night tab should not dwarf an attentive hour, and "tab open != engagement" is enforced at magnitude level.
- Listen-in delta capped per session: The rationale is to avoid a single long listen-in producing an outsized one-day jump.
- Offer cooldown enforced at write time: The rationale is that spammed offers never reach the log as countable events past the first.
- `max(0, delta)` before applying drift: The rationale is code-level enforcement that drift never moves down on neglect.
- Global drift calibration constant: The rationale is to tune between Tamagotchi and screensaver pacing and hit 1-week-instrument / 3-week-visible targets.
- Perch recomputation weighted, not deterministic: The rationale is that two birds with similar boldness do not always sit in the same configuration.
- Wary-mood contagion: NOT RECOVERABLE FROM PLAN
- Chorus detection and transient `in_chorus`: The rationale is renderer hinting for call timing without persisting chorus as state.
- Weighted mood state machine: The rationale is four independent signals must combine into one transition probability.
- Personality resistance in mood transitions: The rationale is that high-boldness birds resist sliding into `wary`, social warmth dampens contagion, vocal frequency raises chorus probability, and curiosity raises offer-approach probability.
- Probabilistic mood transition: The rationale is that the same inputs should not always produce the same output.
- Mood persistence across sessions: The rationale is mood is read from `Bird.mood`, not reset, and ticks run regardless of client connection.
- Client-side call grammar runtime: The rationale is server supplies shape parameters while WebAudio generates calls locally, keeping audio procedural and varied.
- Lazy species motif libraries: The rationale is to load only adopted species and protect bundle budget.
- Shared graph for simultaneous calls: The rationale is true real-time phase interaction and avoiding "stacked recorded loops" phase-cancellation artifact.

### 6. Sync model

- Single source of truth in one row per bird: The rationale is no per-device copy.
- Tick worker DB role owns state updates: The rationale is enforcement at schema/permissions layer, not just code convention.
- API INSERT-only event log permissions: The rationale is a compromised or buggy client cannot send personality fields or alter interaction history.
- Conflict resolution by additive deltas: The rationale is concurrent device events interleave in an append-only log and addition is order-independent.
- Independent snapshot polling per device: The rationale is sub-20s freshness is unnecessary against a 60s tick cadence.
- Session conflicts treated as auth/session issues: The rationale is there is nothing to merge.

### 7. Frontend rendering pipeline

- Canvas2D stack: The rationale is zero additional runtime bundle weight, enough visual complexity without 3D, and direct first-frame drawing without mount/hydration delay.
- Render loop with environment, parallax, birds/perches, foreground ornament: NOT RECOVERABLE FROM PLAN
- Procedurally composed pose-sprites: The rationale is keeping asset budget down versus one bitmap per pose-per-species.
- Position interpolation as flight arc: The rationale is no teleport while preserving server ownership of target and client ownership of tween.
- Inlined first snapshot: The rationale is removing a network round trip from the critical path to achieve <500ms time-to-first-bird.
- Quiet-field loading state: The rationale is never showing a spinner and avoiding a seam between "loading" and "loaded."
- Empty-aviary fly-in after adoption: The rationale given is that this is allowed only for the one-time adoption moment, not returning sessions.
- Reduced-motion distinct renderer module: The rationale is a separate draw strategy that keeps the surface alive instead of turning animations off.
- Reduced-motion override in both directions: The rationale is explicit accessibility-settings control in addition to media query.
- Top bar as separate DOM layer: The rationale is real focusable DOM elements and idle CSS-opacity fade.
- Top bar exactly four icons: NOT RECOVERABLE FROM PLAN
- Critical path code splitting: The rationale is keeping always-loaded code under budget while the full feature surface stays under 2MB in aggregate.

### 8. Audio pipeline

- Single shared `AudioContext`: The rationale is browser autoplay-policy handling and one graph for all active calls.
- "Tap to enable sound" only if needed: The rationale is an unobtrusive top-bar affordance, not a blocking modal.
- Per-bird call scheduler with jitter: The rationale is that timing is never metronomic.
- Chorus mixing through one graph: The rationale is true chorus, not pre-mixed stems.
- Listen-in gain ramps: The rationale is slow transition, not a hard cut, and no complete muting of other birds.
- Audio node lifecycle and reuse: The rationale is no memory growth while reusing graph topology and motif data.
- Caption generation at call schedule time: The rationale is guaranteed caption-audio match without a separate content pipeline.
- WebAudio silent fallback with forced captions: The rationale is "silence-with-no-captions would be a worse failure" and recorded fallback is forbidden.

### 9. Accessibility surfaces

- ARIA live region narration: The rationale is naturalist prose, with polite cadence and assertive priority only for a single priority update.
- Server-authored narration: The rationale is identical voice across modalities, not client-generated raw-state text.
- Captions near calling bird coordinates: The rationale is to tie the phrase to the calling bird's canvas position.
- Keyboard navigation model: NOT RECOVERABLE FROM PLAN
- Two-tone focus indicator: The rationale is contrast against both brightest daytime and darkest night palettes.
- Automated contrast checking for chrome components: The rationale is WCAG AA enforcement for required-reading text.
- Reduced-motion QA as first-class surface: The rationale is the same acceptance bar as full-motion for "feels alive."

### 10. Performance budgets and observability

- ≤2MB initial JS: The rationale is Canvas2D over WebGL, code splitting, procedural audio, and lazy motif loading.
- <500ms first bird: The rationale is inlined snapshot, first-paint render loop, and edge delivery of initial HTML plus snapshot.
- 60fps idle: The rationale is minimal per-frame allocation, pose-sprite compositing, and throttled-CPU CI profiling.
- No memory growth: The rationale is node disposal, notebook virtualization, one shared context, and bounded workers.
- Synthetic checks: The rationale is measuring page-load-to-first-bird-paint from multiple geographies.
- RUM histograms and audio-context error counts: The rationale is aggregate operational measurement with no per-account dimension.
- Tick latency p99 alarm at 5s: The rationale is specified latency monitoring for the scheduled worker.
- Ops dashboard separate from product surface: The rationale is telemetry has no read access to state DB or event log and none is queryable by account or bird.

### 11. Rollout

- Foundation first: The rationale is that getting drift wrong invalidates everything downstream.
- Core client before remaining features: The rationale is performance and 60fps are infrastructural and easier to fix before more surface area is built.
- Full interaction surface after core client: NOT RECOVERABLE FROM PLAN
- Accessibility built in parallel: The rationale is that accessibility must ship with v1, not as a fast-follow.
- Accounts/sync hardening and social last: The rationale is they are additive to a working single-device single-user core and have no engine dependency.
- Launch readiness checks: The rationale is monitoring, alarms, browser support, and unsupported-browser surfaces before launch.
- Ramping birds per aviary via config table: The rationale is tuning the pacing curve post-launch without touching engine code.
- Day-one instrumentation: The rationale is operational visibility from launch, with no product-usage analytics or engagement dashboard.

### 12. Risks

- Drift calibration: The rationale for mitigation is that 1-week-instrument / 3-week-visible is a felt-experience claim requiring synthetic CI, private beta review, and a tunable config.
- Sync correctness: The rationale for DB permission enforcement is that code-review discipline can erode under deadline pressure.
- Audio uncanniness: The rationale for listening-test review is that procedural can still sound looped or drift a bird's call into unrecognizability.
- Accessibility regressions: The rationale for definition-of-done deliverables is that narration and reduced-motion are full parallel implementations with ongoing maintenance cost.
- Performance budget erosion: The rationale for hard CI gates is that many incremental changes can violate budgets without one obvious culprit.
- Gamification creep: The rationale for non-goals review at planning stage is that refusal is cheaper before implementation starts.
- Visit feature scope creep: The rationale is that small co-presence touches imply a structural redesign of simulation and should be treated as a different product decision.
