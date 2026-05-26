## System-level intent

- **A living place, not a game.** This shows up in the scope exclusions for "Gamification of any form," "Tamagotchi-style mechanics," and "Social network surfaces," and again in the drift rule that "no trait ever decreases." The plan's product philosophy is that the aviary should feel alive without becoming a "stat-manager," a streak loop, or a score surface.

- **Notice, never announce.** The plan names this phrase in the feature-creep risk, and it appears in the bird-count ramp: new birds arrive as "a quiet moment" and "a scene moment," not a notification or email. The field notebook also follows this by recording observations rather than generic event logs.

- **Birds choose; users read the signal.** The perch-selection section states this directly: "The user never controls perch position. Birds choose; users read the signal." The same intent appears in the scope line that bird positioning is "driven by mood/personality, not any user drag-to-place affordance."

- **Positive presence without punishment.** The personality drift section says drift is "monotonic toward expressive," "up only on positive presence," and "never down on neglect." Presence and tab-close both terminate presence identically, and the plan calls the monotonic drift rule the engine-level expression of "no Tamagotchi punishment."

- **A sharp client/server boundary protects the core fantasy.** The architecture section says "the client renders snapshots, never owns state," "the client never computes drift," and "only the simulation ticker writes personality vectors." Multi-device sync, stale-write protection, and "no last-write-wins for personality" all carry this same intent.

- **Hidden inner life, observable outer life.** Personality values are "never exposed to the user numerically," "never included" in state responses, and absent from client models. The user sees derived signals like perch, mood, plumage level, calls, and notebook prose rather than raw values.

- **Naturalist voice for the aviary; matter-of-fact voice for system surfaces.** Notebook entries are "naturalist voice, lowercase, present-tense, specific." Screen-reader narration is "never diagnostic," "never numeric," and follows the "field-notebook voice." Error responses and accessibility settings use "matter-of-fact prose."

- **Procedural life rather than canned assets.** Return greetings are "never identical twice." Calls use motif libraries and WebAudio synthesis, "not stacked loops." Idle motion is procedural tweening between poses, "not as sprite-sheet animation loops." The plan repeatedly avoids repeated, recorded, or canned-feeling output.

- **Accessibility surfaces are designed surfaces.** Reduced motion is "not a CSS `animation: none` override" but "a parallel rendering path." Accessibility surfaces "ship with the product, not after," including narration, captions, reduced motion, contrast, and keyboard navigation.

- **Privacy boundaries are architectural, not just policy.** Email is "never used as identifier anywhere else." Telemetry excludes "per-account interaction history," "per-bird drift trajectories," "visit-frequency per account," and anything that could reconstruct a user's relationship with birds. The telemetry pipeline "never reads the simulation database."

- **The aviary is already running.** The boot sequence avoids a spinner or skeleton screen; the "quiet field" says "the aviary is here, catching up." The first bird appears "mid-action," with "no entry animation," so opening the product reveals a continuous place rather than launching a staged scene.

- **Structural enforcement over good intentions.** The plan repeatedly asks for enforcement in code and infrastructure: no personality routes, database permissions for ticker-only writes, CI bundle checks, a gamification linter, no audio file extensions, and architectural rules that "must survive into implementation."

## Per-feature whys

### Scope

- **Single horizontal scene per account** — NOT RECOVERABLE FROM PLAN

- **Two starter birds** — NOT RECOVERABLE FROM PLAN

- **Seven-bird cap** — The plan treats seven as the maximum chorus to validate: test sessions should check whether listeners can distinguish "7 birds of different species" in a chorus, and "if not, the cap must be lowered or the species pool redesigned."

- **Day/night cycle locked to user's local timezone** — The account timezone "drives day/night cycle," time-of-day shapes mood weights, and clients receive `time_of_day` for rendering. The why is a scene that advances with the user's local day rather than a generic clock.

- **Ambient weather** — Weather creates "weather_moment" notebook entries and mood effects such as rain pushing birds toward drowsy. The appendix says frequency must be tuned because too frequent becomes "noise" and too rare "might as well not exist."

- **Three perch zones with mood/personality positioning** — The plan's rationale is explicit: no drag-to-place affordance, because "Birds choose; users read the signal." Perches make mood and boldness observable without a stats panel.

- **Species pool with distinct silhouettes, plumage palettes, and call motifs** — The why is recognizability. Species must be "visually and audibly distinct," and chorus tests check whether individual birds remain distinguishable.

- **Stable bird UUID identity** — The bird ID is "stable, invariant across renames/syncs/migrations," meaning it is "the same bird forever" across devices, renames, and implementation changes.

- **User-assigned bird name and renameability** — NOT RECOVERABLE FROM PLAN

- **Hidden personality vector** — The plan's rationale is to avoid numeric/stat surfaces and protect the simulation. Personality is never exposed numerically, never returned by APIs, stored as one JSONB blob, and not decomposed into columns that invite direct reads by non-simulation paths.

- **Drift history** — NOT RECOVERABLE FROM PLAN

- **Aviary-age bird adoption** — Birds arrive by "aviary age," "not visit count, not score." The ramp says users never purchase or earn birds through engagement; arrival is a "scene moment," preserving the refusal of gamification.

- **System-selected starter birds rather than catalog choice** — The why is tied to the same no-catalog, no-purchase philosophy: the first two are "selected by the system from the pool," and species purchasing/custom selection is out of scope.

- **Server-side simulation** — Canonical aviary state advances "regardless of client connectivity." This makes the aviary a living place and enables multi-device consistency because clients only write events and the ticker owns drift/mood.

- **Personality drift** — Drift is calibrated so regular presence produces measurable change at about one week and visible change at about three weeks. The risk section frames the why as avoiding both extremes: not so fast that it becomes a Tamagotchi-like stat manager, and not so slow that "users feel that nothing they do matters."

- **Mood state** — Mood gives a fast-timescale observable state that is modulated by recent interactions, time of day, ambient events, and personality. Persistence across sessions avoids "reset to neutral on tab open."

- **Return greeting** — The greeting makes return feel noticed without becoming canned: one bird notices within 1-2s, it varies by absence length, boldness, and mood, and is "never identical twice."

- **Listen-in** — The rationale is focused attention without killing the ambient world. The selected bird rises in the mix, others quiet "to ambient, never silent," and ramps are gradual "not a hard cut."

- **Offer interaction** — Offers are a secondary input to drift and mood/curiosity, letting the user affect birds without scores or needs meters. The offer affordance stays in the top bar rather than becoming a feeding-game loop.

- **Specific offer types: seed, song fragment, still pool** — NOT RECOVERABLE FROM PLAN

- **Settle gesture** — Settle is an "opt-in evening lighting gesture" that ends presence identically to tab-close, so it carries no penalty. The 5s undo window makes the quiet transition reversible.

- **Field notebook** — The notebook exists to produce "naturalist-prose observation" instead of "generic event logs." It is read-only, sparse, and written in a field-notebook voice rather than a diagnostic history.

- **Presence accounting conjunction** — Presence requires visibility, focus, and recent pointer/key activity. The why is to count actual presence rather than background tabs, focus alone, or idle visibility, while making tab-close and settle non-punitive.

- **Magic-link email sign-in** — NOT RECOVERABLE FROM PLAN

- **Synthetic account UUID and encrypted email** — Email is "never used as identifier anywhere else." The rationale is privacy and service-boundary hygiene: inter-service messages, foreign keys, event partitions, and telemetry use the synthetic UUID.

- **Per-device revocable session tokens** — NOT RECOVERABLE FROM PLAN

- **Email change requiring new-address verification** — NOT RECOVERABLE FROM PLAN

- **Account export** — NOT RECOVERABLE FROM PLAN

- **Account deletion with soft 30d then hard deletion** — NOT RECOVERABLE FROM PLAN

- **Multi-device sync** — The plan says sync is "not a separate feature": both clients pull the same canonical snapshot. This avoids client-to-client sync and merge reconciliation.

- **Visit invitations** — Invitations are read-only ambient links, with no co-presence, chat, avatars, comments, discovery, or social graph. The why is to allow visiting without turning the product into a social network.

- **Visit expiry, revocation, log, and opt-in notifications** — These give the host control and keep visits quiet. Visitor presence "does not drift host's birds," links expire, revocation works anytime, and visit notifications are off by default.

- **Accessibility package** — The plan treats narration, reduced motion, captions, contrast, and keyboard navigation as launch scope. The appendix says these "ship with the product, not after."

- **Audio package** — Procedural client-side call synthesis provides recognizable per-bird signatures, real-time chorus mixing, and no recorded audio bundle. The fallback is silence with captions, preserving the aviary without adding recorded-audio fallback.

- **Performance package** — The budgets protect the "aviary already running" feeling: first bird under 500ms, 60fps idle motion, no 30-minute memory growth, and tick p99 alarms.

- **Browser support for last two majors and unsupported-browser notice** — NOT RECOVERABLE FROM PLAN

### Architecture, data, and API

- **Independent services** — Services are "deployable independently," separating auth, aviary API, ticker, notebook generation, and visits so each concern can evolve without mixing simulation, prose generation, and account flows.

- **Auth service** — The service keeps email/magic-link sign-in and session issuance isolated, with synthetic UUID assignment and encrypted email so email does not leak into other identifiers.

- **Aviary API** — The API is the client read/write surface: it reads canonical state and writes interaction events. This keeps clients from mutating simulation state directly.

- **Simulation ticker** — The ticker is the sole writer of personality vectors and the worker that consumes events, transitions moods, advances time, and writes canonical state atomically. The why is authoritative, ordered simulation.

- **Notebook generator service** — Entry generation is "creative-text work at a sparse cadence," so it is separated from the per-minute simulation tick.

- **Visit manager** — The manager handles one-time links, read-only snapshots, revocation, and logs, keeping visits bounded to ambient viewing rather than account or social access.

- **Canonical state store** — The relational store holds account metadata, bird records, notebook entries, visit log, and settings as the canonical source of truth for snapshots.

- **Personality vector as one JSONB column** — The plan states the why: avoid decomposition "that would invite direct SQL reads by non-simulation code paths."

- **Append-only event log** — Events are time-ordered inputs for the ticker and notebook generator. The Aviary API does not read it for snapshots, preserving a clean distinction between canonical state and historical events.

- **Auth store with token hashes** — The store separates account, session token, and magic-link records, using token hashes and encrypted email to keep authentication material out of other tables.

- **Client/server boundary** — The client pulls snapshots, renders, synthesizes calls, interpolates, and posts events; it never owns state or computes drift. The rationale is to keep personality and mood authoritative server-side.

- **Pure render pipeline** — Rendering is a pure function of snapshot, local time, and audio state. All state writes go through `POST /events`, preventing render-loop side effects from changing server state.

- **Computed time-of-day instead of stored column** — NOT RECOVERABLE FROM PLAN

- **Weather state in canonical snapshot** — Weather is short-lived and written into the snapshot with type, intensity, start, and remaining ticks so clients can render current ambient state without treating weather as permanent account data.

- **State endpoint excluding personality values** — Raw personality "never crosses the wire." Derived visual values such as `plumage_level` may be sent, preserving expressiveness while hiding raw traits.

- **Small full snapshot and incremental pull** — The snapshot targets under 20KB, and `since_version` returns diffs for keepalive/visibility-change pulls. The why is lightweight state synchronization.

- **Asynchronous event writes** — Events get `202 Accepted`, server timestamps, and append-only storage; the ticker processes them on its next pass. This keeps client interactions decoupled from immediate simulation writes.

- **Stale-write protection** — `snapshot_version` rejects too-old client writes with 409 so the client re-pulls instead of writing against an obsolete view.

- **Specialized presence endpoint** — Presence pings happen every about 30s, so a lighter endpoint records `last_activity_at` and accumulated presence without forcing the full event-write path.

- **Notebook API** — The client reads paginated entries, but "never generates notebook prose." The why is server-side control of the naturalist prose surface.

- **Visitor state endpoint** — Visitors receive only birds, mood, time of day, and weather, with no account metadata or settings. This preserves a read-only ambient visit.

- **JSON-only API with static frontend bundle** — The plan says no HTML templating from the API because the frontend bundle is static and served from CDN.

- **Matter-of-fact error responses** — Error messages use plain system prose like "Your session timed out," matching the plan's matter-of-fact voice for system surfaces.

- **Endpoint rate limiting** — Rate limits apply to auth, event writes, and invite creation, while state pulls and notebook reads remain un-rate-limited beyond DoS protection. The why is protecting abuse-prone write/auth surfaces without making reading the aviary feel metered.

### Simulation and sync

- **Active-account scheduled ticks and lazy ticks** — Accounts with recent activity tick regularly; inactive accounts lazy-tick on next pull. The plan's rationale is to limit infrastructure cost "while preserving correctness" for users returning after months.

- **Drift coefficients and simulation harness** — Coefficients are starting values and must be tested against the week/three-week calibration target. The harness turns calibration targets into assertions instead of comments.

- **Mood transition weights** — Mood transitions are probabilistic and shaped by time of day, personality, events, weather, nearby moods, and inertia, making mood responsive without exposing numeric traits.

- **Mood minimum dwell time** — Dwell time prevents "mood flicker."

- **Perch selection by mood and boldness** — Deterministic perch choice turns mood and boldness into visible behavior while preserving "Birds choose; users read the signal."

- **Call motif library** — Motifs encode species-specific sound descriptors rather than raw audio, giving each species a recognizable grammar without recorded files.

- **Server-side call timing, client-side synthesis** — The plan states the why directly: timing is canonical state, while client synthesis satisfies the bundle budget and enables real-time chorus mixing.

- **Return-greeting greeter selection and stagger** — Social warmth, boldness, mood, and position order select a plausible greeter; randomized stagger prevents multiple birds from firing "in unison."

- **Snapshot-version conflict handling** — Conflicts are resolved by re-pulling fresh state. The event log preserves ordering with server timestamps, avoiding last-write-wins behavior.

- **No personality write route** — Personality cannot be submitted by clients because there is no route that accepts it. The constraint is enforced by absence of capability, not validation policy.

- **Serving pre-tick state during a tick** — `GET /state` can serve the previous version rather than block. The plan says this is fine because the tick is slow enough that clients do not notice the boundary.

### Frontend rendering

- **Quiet field boot sequence** — The quiet sky and perch silhouettes replace spinner/skeleton loading. The plan says the visual should communicate "the aviary is here, catching up."

- **First bird visible under 500ms** — The budget is measured to the first rendered bird, not just a background, because the product depends on the aviary feeling immediately present.

- **No entry animation** — Birds appear in current poses, "mid-action," because the aviary should feel already running rather than staged on entry.

- **Idle micro-motion state machines** — Preening, scanning, head-tilt, weight-shift, and feather-fluff map visible behavior to mood, making state legible without numeric readouts.

- **Procedural tweening between key poses** — Tweening avoids sprite-sheet loops and supports smooth morphing from compact vector or JSON descriptors.

- **Scene layer order** — NOT RECOVERABLE FROM PLAN

- **Perch-to-perch transition arcs** — Soft arcs with wing-flaps make movement feel like bird movement, not a straight-line UI tween.

- **Day/night gradient lerp** — The plan says there should be "no abrupt color change"; transitions are distributed over the full phase.

- **Settle visual and audio transition** — Settle shifts the sky toward evening, quiets calls, and moves birds toward drowsy postures, matching the opt-in evening gesture.

- **Reduced-motion transitions** — Perch moves become cross-fades, particles are removed, and color shifts remain because they are "non-vestibular."

- **WebGL preferred, Canvas 2D fallback** — WebGL is preferred for smooth 60fps morphing on mobile; Canvas 2D is the fallback.

- **HTML/CSS for top bar and chrome** — Account settings, notebook, offer panel, and accessibility settings are standard component surfaces rather than part of the canvas scene.

- **SVG bird and perch assets** — SVG is small, scalable, and tintable by plumage palette, supporting the bundle budget and visual variation.

- **CSS custom properties for palettes** — Palette variables let reduced-motion mode and high-contrast overrides swap colors without re-rendering.

### Audio

- **AudioWorklet with per-bird gain nodes** — Dedicated gain nodes allow listen-in and mixing control per bird; ambient reverb gives "spatial depth."

- **Procedural call synthesis** — Motifs, harmonics, modulation, personality variation, and node pools make calls varied, recognizable, and memory-stable. The same motif "never sounds identical twice."

- **Chorus mixing** — The rationale is explicit: procedural calls create a true chorus with harmonic interplay; stacked recorded loops can produce phase cancellation artifacts.

- **Stereo panning** — Bird position subtly maps to left/right pan, adding spatial separation to the chorus.

- **Listen-in mix decay** — Linear ramps bring the focused bird to full volume and others to ambient, creating smooth focus without silence.

- **AudioContext lifecycle** — Creating audio on first gesture satisfies autoplay policies; suspending on hidden tabs saves CPU and battery.

- **Skipping missed hidden-tab calls** — The client does not retroactively play missed calls; it resumes from current state so the aviary remains current rather than replaying a backlog.

- **WebAudio-unavailable fallback** — Silent mode with captions on by default keeps the aviary visual and interactive without adding recorded-audio fallback or polyfills.

### Accessibility

- **Screen-reader narration queue** — The queue diffs snapshots into observable changes and announces through `aria-live="polite"` so updates do not interrupt the user's current context.

- **Narration cadence and priority bump** — Idle narration drains at most once per 30 seconds, while user-initiated events are narrated within 5 seconds. The why is calm presence with prompt feedback for direct actions.

- **Naturalist narration prose** — Narration avoids diagnostics, numbers, and gamified phrasing so screen-reader users receive "a living place being observed," not a state machine report.

- **Reduced-motion detection and account setting** — `prefers-reduced-motion` auto-enables the mode, and the account setting persists it across devices while still allowing user override.

- **Reduced-motion as parallel rendering path** — The plan says the scene "still has visual life; it just moves differently," so reduced motion is designed rather than disabled.

- **Call captions from procedural grammar** — Captions are generated from the same motif parameters as the audio, preserving the call's affect when sound is unavailable or captions are enabled.

- **Caption display and semantics** — High-contrast captions near the calling bird and `role="status"` let captions serve both visual and optional screen-reader needs.

- **Keyboard navigation** — Tab, arrows, Enter/Space, and Escape provide full access to top bar, birds, listen-in, and offer panel without a pointer.

- **Dual-color bird focus indicator** — The plan specifies a contrast technique that works against any time-of-day palette behind the bird.

- **Accessibility settings panel** — The panel exposes narration, captions, reduced motion, shortcuts, and last narration text, with labels in matter-of-fact system voice.

### Performance and observability

- **Initial JS bundle budget** — The <2MB gzipped limit is enforced in CI so performance remains a shipping constraint rather than an aspiration.

- **Code splitting** — Settings, accessibility, visits, and full notebook are lazy-loaded so the initial bundle contains only the aviary surface, top bar, and offer panel.

- **Procedural asset strategy** — Vector bird assets and motif JSON avoid large PNG spritesheets and recorded audio files, keeping the bundle small.

- **First-bird timing measurement** — The metric starts at navigation and ends at the first frame with a bird. This preserves the product's "first-bird" promise rather than measuring a lower-value placeholder.

- **Inline critical CSS and parallel state/bundle fetch** — These are used to hit the first-bird target and avoid flash-of-white or load-complete blocking.

- **Runtime frame budget** — 16ms frame targets and CI profiling protect 60fps idle motion on a 5-year-old laptop.

- **Memory stability budget** — The 30-minute heap test and audio node pools protect the "no memory growth" constraint and prevent call synthesis from causing GC pressure.

- **Synthetic, RUM, and server-side telemetry** — These measure page load, first-bird timing, frame histograms, audio errors, tick latency, and event throughput so the team can see whether the aviary remains fast and alive in production.

- **Aggregate-only privacy boundary** — RUM is anonymized and excludes per-account dimensions, drift trajectories, visit frequency, and any metric that could reconstruct a user's relationship with birds.

- **Error budget alarms** — Tick p99 >5s and page-load p95 >3s are explicit alarms, making simulation lag and slow launch visible operational failures.

### Rollout, risks, and invariants

- **Phased ship sequence** — The phases separate static rendering/audio, live integration, accessibility/polish, beta validation, and public launch. Each phase has a goal such as "complete rendering + audio pipeline" or "feature-complete v1."

- **Bird count ramp** — New birds arrive over aviary age milestones with no purchase, no engagement earning, and no email/push. The rationale is anti-gamification and quiet scene continuity.

- **Day-1 instrumentation** — Production launch includes synthetic checks, RUM, server histograms, and alerting so performance and tick health are live from day one.

- **Drift calibration mitigation** — The harness, CI thresholds, and beta tuning exist because drift too fast turns into a stat manager, while drift too slow makes birds feel static.

- **Sync correctness mitigation** — Additive-only drift, personality vector versions, concurrent tick tests, and audit records protect against invisible personality data loss.

- **Audio naturalness mitigation** — Internal tester feedback, audio-designer motif work, pitch drift, breath/noise, and recognizability tests protect the "affective core" from synthetic-sounding calls.

- **Narration quality mitigation** — Template variety, blind comparison against field-notebook prose, screen-reader beta feedback, and CI checks against generic patterns protect felt-aliveness for screen-reader users.

- **Gamification linter and review gate** — The plan says non-goals need "teeth"; CI fails UI strings tied to user-behavior counts, and review includes a "gamification check."

- **Event-log partitioning and archive** — Partitioning by account and time, indexed reads since last tick, and cold archive after 90 days prevent ticker lag from growing with history.

- **Browser audio policy mitigation** — First-gesture AudioContext creation and captions bridge policy changes that would otherwise silence page-open calls.

- **Implementation unknowns** — Presence window, tick cadence, mood set, drift coefficient, notebook sparsity, weather frequency, species composition, and motif count are explicit calibration decisions to resolve before Phase D.

- **Architectural rules that must survive** — The appendix marks personality boundary, ticker-only writes, UUID identity, presence conjunction, no gamification language, no recorded audio, and launch accessibility as invariant constraints derived from the plan.
