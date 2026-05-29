## System-level intent

- **Aliveness is the product.** The plan states this directly in the opening: "aliveness is the product." It defines aliveness as the conjunction of "server-authoritative simulation, monotonic-toward-expressive drift, procedural (never recorded) audio, and notice-never-announce framing." Those four ideas recur in the service split, drift math, audio pipeline, first-frame behavior, and anti-feature architecture.

- **Promises become architectural properties, not contributor conventions.** The opening says hard lines should become "an architectural property a reviewer can check, not a convention a contributor can erode." This shows up wherever `[INV]` is tied to DB grants, schema validation, CI checks, property tests, architecture tests, and missing pipelines.

- **Simulation owns meaning; client owns motion.** The render boundary says exactly this: "simulation owns meaning; client owns motion." The client consumes "snapshots + interpolation targets" and never raw personality. This carries through the server-authoritative split, snapshot DTO, interpolation, and reduced-motion renderer.

- **The server is the sole writer of canonical life.** The plan repeatedly grounds continuity and coherence in the `sim` worker as "the *only* writer of personality vectors and canonical mood/scene state." It connects this to "the aviary continues without the viewer," multi-device coherence, and no last-write-wins.

- **Drift must be monotonic toward expressive, never punitive.** The drift section names the invariant "monotonic toward expressive": neglect produces `delta = 0`, "never a decrease." The plan ties this to "no Tamagotchi," "no reset, ever," and settle/tab-close equivalence.

- **Notice, never announce.** The product voice avoids welcome toasts, streaks, labels, green-dot calendars, and hard arrival cues. It appears in return-greeting stagger, no unison greeting, quiet field loading, sparse notebook, silent visit log, opt-in visit notifications, and anti-feature architecture.

- **Hard exclusions should be structurally hard to re-add.** The non-goals are "not 'later'"; the plan says protocols and data model are built so re-adding them is "harder, not a flag-flip." Examples include no cross-account aggregation pipeline, no visit-frequency datum, and a notebook input contract with no user-behavior signals.

- **Personality is felt through presentation, never exposed numerically.** The scope call says raw personality numbers "never cross the API boundary." The snapshot only carries "derived presentation hints," and the scene uses perch, motion, mood-shaped behavior, call timing, and captions rather than raw trait scalars or labels.

- **Identity must feel stable over time.** Stable `bird_id`, persisted personality, fixed motif libraries, recognizable call identity, and the 7-bird cap all protect the user's sense that a bird remains itself as it drifts. The plan calls `bird_id` stability "the foundation of drift's perceived validity."

- **Procedural variation is necessary to avoid dead software.** The audio section says "looped audio is the audible signature of dead software," and greeting must be "procedurally varied" rather than one of fixed variants. This principle also supports chorus, call captions from grammar, and first-frame motion already in progress.

- **Accessibility surfaces deliver the actual product.** The accessibility section says they deliver "the actual product, not a stripped variant" and ship "with v1, not as v1.1." Screen-reader narration, captions, keyboard interaction, and reduced motion all use canonical state and naturalist voice rather than state lists or animations-off fallbacks.

- **System voice and naturalist voice are deliberately separated.** The API section says system error bodies are "matter-of-fact" and "naturalist voice never appears in an error/system response." Naturalist voice is reserved for notebook, narration, and captions.

- **Privacy boundaries are product boundaries.** Synthetic UUIDs, email encrypted on exactly one record, no warehouse access to the simulation DB, aggregate-only telemetry, and own-account-only interaction data all enforce the idea that "the relationship is the user's."

- **Performance is part of the conceit.** The plan treats bundle size, first-bird time, 60fps idle, no memory growth, and no spinner as product-level invariants. It says visible loading "kills the central conceit" and that above 500ms "the user notices loading; below, they don't."

- **Calibration values are tracked, not guessed.** `[CAL]` values are "never a magic constant; always traceable." The drift harness and open calibration table make tick cadence, drift weights, activity window, offer cooldown, mix ramps, and age thresholds retunable by named instruments.

## Per-feature whys

**Scope and account surface**

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** The plan's rationale is coherence: there is "one canonical record per account" so both devices read the same `aviary_state` and there is no client-to-client sync or client-side personality state to reconcile.

- **Email magic-link sign-in:** The plan grounds this in matter-of-fact auth, privacy, and oracle avoidance. `POST /auth/request-link` returns `202` always, and email lookup is through HMAC rather than plaintext email.

- **15-min magic-link expiry:** NOT RECOVERABLE FROM PLAN

- **Single-use magic links:** The reason given is replay protection: consuming a link sets `consumed_at`, and replay returns a matter-of-fact error.

- **Per-email magic-link rate limit:** The open calibration table gives the rationale as "abuse vs friction."

- **Per-device revocable session tokens:** The plan connects this to account control through `GET /account/sessions` and `POST /account/sessions/:id/revoke`; each device can be revoked without collapsing the account.

- **Email change with new-address verification:** NOT RECOVERABLE FROM PLAN

- **On-demand JSON account export:** The plan's rationale is "the relationship is the user's." Export is the deliberate place where the user can obtain their own current personality vectors, moods, notebook, and settings.

- **Soft-delete then hard delete:** The plan again ties this to "the relationship is the user's": recover first, then hard delete "every record tied to the account."

- **Exact 30-day recoverable delete window:** NOT RECOVERABLE FROM PLAN

**The aviary and bird engine**

- **2 starter birds:** NOT RECOVERABLE FROM PLAN

- **7-bird cap:** The cap preserves legibility and recognizability. The audio section says the mix stays legible at the 7-bird cap, and the call-grammar section says recognizability is what "makes the 7-bird cap meaningful."

- **Exact ~6-species pool size:** NOT RECOVERABLE FROM PLAN

- **Per-species silhouette, palette, and call-motif library:** The rationale is recognizable identity. Species motif libraries give birds stable auditory character while drift changes frequency and timing, not motif identity.

- **Hidden per-bird personality vector:** The vector drives drift, mood modulation, perch behavior, call liveliness, and presentation while staying hidden so personality is felt rather than shown as a numeric meter.

- **Fast-timescale mood:** Mood gives a "fast clock" shaped by interactions, time of day, weather, and personality. The plan describes personality as "the slow current" and mood as the "fast weather."

- **Server-side simulation tick:** The tick embodies "the aviary continues without the viewer." It consumes events, computes additive deltas, advances mood and scene, and writes canonical state regardless of client connectivity.

- **Procedural call synthesis:** The plan rejects recorded calls because repeated loops are "the audible signature of dead software" and because chorus needs real-time variation.

- **Bird-to-bird interaction and emergent chorus:** The plan's rationale is that overlapping scheduled calls from high-vocal-frequency birds should become a "real chorus," not stacked loops or phase-cancel artifacts.

- **Stable internal bird IDs:** Stability protects "drift's perceived validity." Renames, sync, migrations, and species-pool changes must preserve identity.

- **User-assigned renameable names:** NOT RECOVERABLE FROM PLAN

- **Age-gated additional birds:** The rationale is relationship-deepening without gamification: additions are based on aviary age, "not visit-count, not score, not paid."

**Interactions**

- **Procedurally varied return-greeting:** The greeting makes absence and return visible without announcing arrival. Boldness and mood decide who greets; absence length changes form; staggered greeters avoid a unison cue that would "announce" arrival.

- **Listen-in:** Listen-in is an attention signal for drift and a perceptual mode. The mix ramps so it feels like "listening, not channel-switching," and other birds drop to an ambient floor, "never silent."

- **Offer interaction:** Offers give a bounded way to affect curiosity and boldness. The tick computes reaction from mood and curiosity; the user sends an event, not an absolute state write.

- **Seed / song-fragment / still-pool as the offer menu:** NOT RECOVERABLE FROM PLAN

- **Per-bird, per-type offer cooldown:** The plan gives the cooldown as a guard against saturation and nagging: the server enforces "not yet," while the UI simply disables the affordance.

- **Settle gesture with undo:** Settle is a quieting interaction: it ramps evening light, ends presence cleanly, and contributes "mood-quieting only." Undo makes the gesture reversible without producing negative drift.

- **Presence accounting:** The three-signal conjunction keeps drift honest. The plan says a laxer rule would "silently inflate everyone's drift," while "tab open" alone must produce zero presence.

- **Activity window around pointer or key input:** The window leans long because "watching without moving is the actual product," but it is still retuned by the drift-calibration harness.

- **Read-only field notebook:** The rationale is product voice and anti-gamification. It is an auto-generated naturalist record of aviary observations, not a user-authored social or achievement surface.

- **Notebook input contract excluding user behavior:** This makes a "you visited every day" entry impossible and keeps the notebook from becoming a streak or user-surveillance surface.

**Scene and rendering**

- **Single horizontal non-panning scene:** NOT RECOVERABLE FROM PLAN

- **Three perch zones as read-only signals:** Front, middle, and back perches let the simulation express boldness and mood through position without exposing raw personality numbers.

- **Local-time day/night cycle:** The plan uses `local_tz` plus server clock so day/night feels local while staying derived and consistent, never a mutable flag that can desync.

- **Rare ambient weather:** Weather supplies transient mood inputs: rain dampens vocal frequency, wind raises alert/wary probabilities, adding variation without resetting personality.

- **Continuous ambient leaf and feather drift:** Ambient ornaments keep the scene alive between bird actions while staying client-side and pooled so they do not burden canonical state or memory.

- **Thin auto-fading top bar:** NOT RECOVERABLE FROM PLAN

- **First frame with motion already in progress:** This protects the illusion that the aviary kept running. The plan forbids entry animation, fade-from-static, and spinner because the user should arrive into ongoing life.

- **Quiet-field loading state:** The quiet field should read as "the aviary catching up," not "the app loading." The no-spinner rule protects notice-never-announce and first-bird performance.

- **Empty-aviary state:** The same quiet field prevents an empty app-like screen; after the first bird enters, "the user never sees an empty aviary."

- **Canvas2D render runtime:** Canvas2D is chosen because <=7 sprites and light particles are within budget on old hardware, while WebGL adds context-loss, shader, and bundle costs, and DOM-per-bird risks layout/compositing fragility.

- **Procedural/SVG bird sprites rasterized into atlases:** The plan uses compact assets to keep per-frame draw trivial and support the <=2MB first-paint bundle.

- **Snapshot interpolation:** Interpolation prevents teleports. A bird moving from perch A to B should render a smooth path, and local clocks keep motion from stalling between network snapshots.

- **Mood-shaped idle micro-motion:** The user should read mood from motion with "no label/tooltip/status-icon." Continuous low-amplitude motion prevents a static frame from reading as paused.

- **Responsive safe frame:** The plan requires narrow viewports to compress without cropping and wide viewports to spread perches; the rationale is "never crop a bird."

- **Tab-hidden behavior:** Hidden tabs stop rendering for battery, but the server simulation continues. On return, the client pulls a fresh snapshot rather than resuming a frozen scene.

**Architecture, data model, and API**

- **`edge` request service:** `edge` is stateless so it can scale horizontally and hold no simulation authority. It handles auth, snapshots, account management, visits, and static delivery while not writing personality.

- **`sim` tick worker:** `sim` is the architectural embodiment of server authority and continuity: it is the only writer of personality, mood, and canonical scene state.

- **Web client with zero canonical state:** The client renders, synthesizes audio, and emits events, but owns only ephemeral render/interpolation state so it cannot clobber canonical life.

- **Shared typed schema package:** Shared DTOs, event enums, and snapshot shapes keep wire contracts from drifting across `edge`, `sim`, and web client.

- **Server-authoritative state boundary:** The boundary makes server-side continuity, multi-device coherence, and no last-write-wins true at once because clients cannot write absolute personality or mood.

- **Render pipeline boundary:** The client never reads raw personality; it consumes snapshot tokens and derived presentation hints, so "simulation owns meaning; client owns motion."

- **Append-only event log:** Events record what happened without becoming state. This lets the tick decide meaning, consume in order, and avoid last-write-wins.

- **Client-generated `event_id`:** The rationale is idempotent deduplication. Clients can buffer offline and replay safely.

- **Snapshot pull:** Small snapshot reads let clients refresh on open, visibility change, resume gaps, and keepalive without client-to-client sync.

- **Initial snapshot inlined into HTML:** The reason is first-bird performance: it removes a round trip so the first bird renders under the 500ms budget.

- **Personality-safe snapshot DTO:** Derived buckets like `plumage_level` and `call_liveliness` enforce that raw trait floats never cross the API boundary.

- **Matter-of-fact system errors:** The plan keeps naturalist prose out of auth, rate-limit, session, outage, and visit-expiry errors so system surfaces do not pretend to be the aviary.

- **Postgres primary store:** The stated rationale is strong ordering and transactions for no-LWW and exactly-once event consumption at v1 scale.

- **Event log in the same Postgres:** At v1 volume this is "ample," and the high-water-mark pattern is simpler than adding Kafka.

- **Envelope encryption plus HMAC email lookup:** This lets the system find accounts by email without storing plaintext email or using email as a key.

- **Persisted personality rather than read-time derivation:** The vector is "the integral and is canonical"; replaying history at read time would undermine that canonical persisted state.

- **`local_tz` stored, day/night derived:** This anchors the cycle to local time while preventing a mutable `is_night` flag from desyncing.

- **Exactly-once event consumption:** Per-account high-water marks and idempotent event IDs ensure each event is consumed once, in order.

**Simulation engine design**

- **Drift as slow low-pass filter:** The plan wants drift to be measurable over about a week and visible after about three weeks, not a click-by-click meter or screensaver.

- **Non-negative drift clamp:** This enforces no neglect penalty: absence produces quieter expression but never lower traits.

- **Presence-dominant drift weights:** Presence is dominant because regular presence is the core relationship signal; listen-in and offers matter, but less.

- **Drift accumulators:** Accumulators make drift "a true integral over time," not a per-tick recomputation.

- **Mood state machine with hysteresis:** Hysteresis prevents flicker, while time-of-day, weather, recent interactions, and personality create fast variation.

- **Mood persistence across sessions:** Mood is canonical state advanced during absence, so a bird changes through the night because the tick moved it, not because the client reset it.

- **Call-grammar runtime:** The server schedules when calls happen and with what character, while the client realizes sound. This keeps behavior server-authoritative and audio procedural.

- **Fixed motif identity per bird/species:** Drift changes frequency and timing, not motif identity, so a bird remains recognizable by ear.

- **Greeting random seeds and staggered offsets:** The plan requires real variation and avoids simultaneous greetings because unison would "announce" arrival.

- **Calibration harness:** The harness pins `[CAL]` values by replaying synthetic presence profiles and asserting one-week instrument drift, three-week visible drift, no single-session bucket jump, and no negative drift.

**Audio pipeline**

- **Client-side WebAudio synthesis:** The plan uses WebAudio recipes so calls are generated from motif seeds and biases, not shipped as recordings.

- **Independent chorus mixing:** Multiple synthesized voices make a "real chorus" and avoid stacked-loop phase cancellation.

- **Bounded voice pool:** The pool keeps simultaneous voices legible and bounded at the 7-bird cap.

- **Audio buffers and nodes pooled:** Pooling supports the no-memory-growth budget and avoids per-call allocation churn.

- **Listen-in gain ramp:** Slow gain changes make focus feel like listening rather than channel switching, while preserving ambient floor for other birds.

- **WebAudio fallback to silence plus captions:** The plan says "silence-with-captions beats canned audio" and keeps the no-recorded-audio invariant intact.

**Accessibility surfaces**

- **Screen-reader narration:** Narration uses running naturalist prose, not a state list, so screen-reader users receive the actual field-notebook-like product.

- **Slow narration cadence and bounded queue:** The reason is to avoid overwhelming the screen reader; user-initiated events get priority but still as observations.

- **Runtime-generated call captions:** Captions are generated from the same procedural grammar as the sound so the caption matches what actually played.

- **Reduced-motion renderer:** Reduced motion is "calmer and slower Pocket Aviary," not "broken Pocket Aviary"; it remains driven by the same snapshots so it cannot fall behind.

- **Keyboard navigation:** Keyboard focus maps to the same listen-in events as pointer interaction, so keyboard users interact with the same engine surface.

- **High-contrast focus indicator:** The focus state must remain legible against bright and dim aviary states.

- **WCAG AA contrast:** The plan pins contrast for all user copy, chiefly top-bar and account/error chrome, so the text surface remains readable.

- **No personality vector via ARIA:** The accessibility surface should not become a backdoor numeric personality exposure.

**Social**

- **Host-issued read-only ambient visits:** This gives one ambient social feature without creating profiles, feeds, chat, discovery, co-presence, or a social-network surface.

- **Email-addressed visit invite:** The plan uses direct per-invite addressing rather than discovery or friend-of-friend mechanics.

- **Visit revocation:** Host control is explicit: revoked or expired visits return a matter-of-fact unavailable response.

- **30-day visit invite expiry:** NOT RECOVERABLE FROM PLAN

- **Silent visit log:** The log is on-demand only, with no badge or push, preserving quiet social presence rather than announcement.

- **Visit notifications off by default and opt-in:** This keeps notifications from becoming a push surface about the aviary; even the only notification surface is silent unless enabled.

- **Visitor snapshot with no interaction endpoints:** Visitors can look but cannot emit events, so their attention does not enter the host's simulation.

- **No co-presence, chat, avatars, comments, discovery, or leaderboards:** The plan treats these as hard exclusions to avoid social-network and gamification surfaces.

**Privacy and telemetry**

- **Synthetic UUID identity:** UUIDs are used in keys, logs, telemetry, shards, and inter-service messages so email is not identity infrastructure.

- **Email encrypted on exactly one record:** The rationale is PII minimization; schema lint forbids email columns elsewhere and email-derived keys.

- **Per-bird/per-account interaction data used only for that account's simulation:** This prevents training, recommendations, population analysis, or cross-account product use from the relationship data.

- **No analytics warehouse connection to the simulation DB:** The absence of a pipeline is the enforcement against cross-account aggregation.

- **Aggregate-only operational telemetry:** Page timings, frame histograms, audio errors, tick latency, and request metrics are allowed because they exclude per-account and per-bird dimensions.

- **Plain-text privacy policy in account settings:** The policy names the aggregate categories and explicitly excludes per-bird interaction state.

**Performance, observability, rollout, and support**

- **Initial JS bundle <=2MB gzipped:** The budget protects first paint and is helped by compact procedural assets and no audio files.

- **Lazy-loaded account, accessibility settings, and visit-invitation flows:** These are kept out of first paint so the initial bundle carries only the aviary scene renderer, audio engine, and snapshot bootstrap.

- **Time-to-first-bird under 500ms:** The plan says above 500ms the user notices loading, below it they do not.

- **Render first bird before non-critical assets:** Audio and ornaments initialize after first paint so visible aliveness arrives first.

- **60fps idle over a 30-minute session:** Smooth idle motion is necessary because paused or janky motion breaks the central conceit.

- **No memory growth over 30 minutes:** The plan makes pooling and bounded contexts a CI gate so long sessions do not degrade.

- **Synthetic perf fleet and aggregate RUM:** These catch field regressions while staying aggregate-only and privacy-scoped.

- **Tick p99 latency alarm at 5s:** The alarm catches simulation degradation before users feel a "slow aviary."

- **Browser support limited to last two major versions:** The plan rejects legacy compat paths because they would cost bundle budget.

- **Build sequence starting with foundations:** Foundations come first because they "bake in the privacy invariants from line one."

- **Simulation core before client:** The headless engine gates on calibration assertions before rendering exists, so drift and mood are correct before presentation.

- **Accessibility built in parallel from client render step:** The plan explicitly says accessibility is "not after" and ships with v1.

- **Engine and audio support the full path to 7 from day one:** This keeps the cap enforced in the engine rather than patched on later.

- **New species with no rarity and no catalog pick:** The rationale is that first encounter should be "meeting a bird, not configuring an avatar."
