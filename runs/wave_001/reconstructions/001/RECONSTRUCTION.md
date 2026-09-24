## System-level intent

- Continuity over gamification. The plan's governing acceptance criterion is "continuity": "the same named birds persist and continue evolving on the server" without turning "the relationship into a score or a notification loop." This also appears in "No scores, levels, streaks" and in the rejection of "eligibility timers or bird-count progress."
- Server-authored canonical life. The server owns "canonical bird identity, hidden personality vector, mood, perch intent, current weather, and notebook records," while the browser receives a compact "render projection." This intent returns in "only the server tick writes vectors," "no client state merge," and "no known identity/vector loss."
- One state, many surfaces. The plan repeatedly requires visual, auditory, captioned, and narrated surfaces to reflect one state: "a visible call, caption, and synthesized call describe the same event," and call intents are consumed by "synthesizer," "beak/body cue," and "caption generator."
- Quiet watching rather than visit-frequency surfaces. Presence is "quiet watching," not "click frequency"; ordinary tab close and settle carry "no penalty." The plan excludes notification loops, public discovery, scores, streaks, and engagement prompts.
- Naturalist product voice. Product prose is "lowercase, present-tense naturalist observation, specific to a bird and moment." Identity, sync, account, and accessibility settings are "direct and matter-of-fact." The notebook, captions, and narration all inherit this observational voice.
- Privacy by design. The plan frames the visit log as "an account privacy surface," keeps visits "off until a host sends an explicit invitation," encrypts email, forbids email in logs and metrics, separates visitor projections, and limits observability to "aggregate-only counters/histograms."
- Accessibility as a full scene. The plan says to "ship narration grammar and reduced-motion renderer with the normal renderer" and to test whether alternate surfaces feel like "a continuing aviary rather than a status feed."
- Calibration through versioned configuration. Hidden traits are normalized, starter counts, ticks, moods, drift rates, and offer cooldowns are "configuration constants" or "calibration data." Changes happen through "versioned simulation configuration," with deterministic harnesses and panels, not through client logic.
- Ambient systems are core experience. "Day/night, rare mild weather, bird-to-bird behavior, and procedural calls are part of the core experience, not later decoration." Weather, local light, calls, mood, and perches all feed the same continuity.
- Bounded interaction with no resource meters. Offers, listen-in, and settle have modest or temporary effects, and "Still pool and song fragment are brief scene/audio events, not resource meters." Event bonuses are capped so "button-mashing" cannot dominate.
- Visitor separation. Visitor sessions are "read-only," sanitized, and unable to mutate host state. Visitor arrival does not trigger greeting, cannot obtain presence, and has no interaction endpoint.
- Fast first bird and quiet-field delay handling. The plan prefers edge/private projection, "first bird visible within 500 ms," no spinner, no autoplay modal, no entry animation, and a "quiet field" on delayed snapshot.

## Per-feature whys

### Product boundary and decisions

- Browser-based, single-screen aviary per email-authenticated account: The plan keeps the experience to one private aviary per account and later requires birds and perch zones to remain in view with "no pan/zoom/crop," supporting the single-screen watching surface.
- Two system-selected starter birds: NOT RECOVERABLE FROM PLAN.
- Naming and renaming birds: Renames preserve "UUID/vector/history," and shipping requires the "same named birds" to persist, so names attach to continuing identity rather than replacing a bird.
- Age-gated additional adoptions up to seven birds: NOT RECOVERABLE FROM PLAN.
- Declined adoption offers remaining available privately: NOT RECOVERABLE FROM PLAN.
- No eligibility timers or bird-count progress: This keeps age-gated additions from becoming visible progression or optimization; the rates are "not user-visible progression."
- Watching and presence: Presence "dominates" drift, "quiet watching counts," and a bird with attention can evolve without click frequency, meters, or tasks.
- Listen-in: It raises a focused bird's channel while keeping others at "a nonzero ambient floor," and its lasting effect is only a modest favoring of "social warmth/vocal frequency for that bird."
- Seed, song-fragment, and still-pool offers: They provide bounded, server-authored reactions and modestly favor curiosity/boldness, while remaining "brief scene/audio events, not resource meters."
- Settle: Settle ends presence, temporarily lowers light and calls, affects "only mood/presence ending," and has "no penalty" compared with ordinary tab close.
- Five-second settle undo: The plan specifies the behavior and prevents accidental offer/move on undo, but its product rationale is NOT RECOVERABLE FROM PLAN.
- Sparse read-only field notebook: The notebook preserves server-created observations from "true state/event facts" without attendance, streaks, or raw trait deltas, and history stays stable because prose is stored at creation.
- Optional read-only invitations: Visits remain off until an explicit invitation, and visitors receive a sanitized read-only projection so visiting cannot become public discovery or host-state influence.
- Screen-reader narration: It lets the scene be understood as naturalist observation generated from projection and event facts, without becoming "raw ARIA state dumps."
- Call captions: Captions expose the exact call event as prose from motif, contour, timing, and mood, not a static species label.
- Reduced motion: The parallel renderer preserves calls, captions, moods, drift, and notebook while removing flight paths, parallax, and leaf drift.
- Keyboard operation: The plan gives bird focus targets, Enter listen-in, Escape exit, and menu focus behavior so the scene and top bar can be operated without pointer input.
- Account export, deletion, and device-session management: These are privacy and control surfaces: export is private user data, deletion purges account-linked records, and device sessions are revocable.
- Day/night, mild weather, bird-to-bird behavior, and procedural calls as core experience: They are explicitly "part of the core experience, not later decoration," and feed mood, calls, local light, and continuity.
- Product vocabulary and user copy: The plan preserves PRD vocabulary in product concepts and user copy so code-facing and user-facing surfaces use "bird, call, presence, listen-in, offer, settle, notebook, visit."
- No textual return greeting: It prevents arrival from becoming a banner; arrival cues are local bird behavior and "never paired with a welcome banner."
- Exclusion of scores, levels, streaks, hunger, distress, deaths, notification loop, public discovery, profiles, comments, shared aviaries, native client, payment, or scene customization: The plan uses these exclusions to protect the relationship from score, social feed, engagement cadence, punishment, payment, and scene customization loops.

### System shape and ownership

- Lightweight web client plus account/auth/invitation service, canonical aviary store, and partitioned simulation workers: This separates identity, canonical state, and simulation behind one authenticated API while keeping the browser to rendering and local control.
- Relational database: The plan chooses it for "transactions, uniqueness constraints, encrypted account email, canonical bird records, append-only interaction events," and atomic tick/cursor commits.
- Queue or partitioned scheduler dispatching account IDs about once per minute: Workers update inactive accounts too, so server continuity does not depend on an open browser.
- Sharding by synthetic account UUID, not email: This keeps email out of queue keys, metrics, tracing, and identifiers.
- Bounded chronological catch-up for overdue ticks: The plan says to catch up rather than "skipping elapsed time or inventing presence," preserving canonical time.
- Server-only writes for identity, personality vector, mood, perch intent, weather, and notebook: This prevents the browser from submitting "absolute state or trait values" and keeps canonical bird life server-authored.
- Browser render projection: The projection lets the browser own interpolation, ornaments, sound synthesis, and local focus while not owning canonical events.
- Client-only leaves and feathers: Render ornaments "never become canonical events," so visual decoration cannot determine mood or notebook facts.
- Versioned projection contract: It separates "internal simulation fields from scene fields" and lets public/visitor responses omit vectors, event history, account settings, and notebook.
- Shared projection and simulation clock for visuals and audio: This ensures "a visible call, caption, and synthesized call describe the same event."
- Edge-delivered initial shell and signed projection: The plan uses it to render the first bird immediately while preserving privacy through short-lived authenticated reads and "private, no-store."
- Cold or delayed snapshot quiet field: It avoids a spinner and joins the current scene once state arrives.
- No entry animation except post-adoption empty field and first soft fly-in: The plan specifies the exception, but the rationale for the general ban is NOT RECOVERABLE FROM PLAN.

### Data model and invariants

- `Account` with encrypted verified email only here: Email is kept out of metrics, logs, queue keys, foreign keys, and tracing to preserve the privacy boundary.
- `DeviceSession` checked on every protected request: Continuous revocation checks make per-device session removal meaningful.
- One-to-one `Aviary`: It enforces one stable aviary per account and gives the tick a single version, cursor, weather state, local light, adoption state, and notebook cursor.
- Stable `Bird` UUID, immutable species/call-signature seed, and hidden vector: These fields preserve bird identity, recognition, and history through renames and migrations.
- Transactional seven-bird cap: NOT RECOVERABLE FROM PLAN.
- Append-only `InteractionEvent`: Events record inputs with idempotency and consumed tick cursor, while forbidding "direct trait update."
- `PresenceLease`: Host-only leases with visibility, focus, activity freshness, clamped intervals, and union accounting prevent duplicate tabs or stale heartbeats from double-counting.
- `NotebookEntry` immutable and indefinitely ordered: Stored prose and stable IDs keep history stable when templates change and avoid archiving.
- `Invitation`, `VisitSession`, and `VisitLog`: These keep read-only visitor access separate, single-use, revocable, approximate, and unable to grant host write permissions.
- Backup/restore drills comparing bird UUIDs and vectors: This directly protects against identity/vector loss.
- Database/service guard on vector writes: Only the tick role can update vectors, enforcing server-authored drift.
- Persisting vector values directly: This avoids rebuilding vectors from events at request time and keeps current vectors canonical.
- Export including current vectors: The plan treats the file as "private user data" with a short-lived link and explicitly blocks turning vectors into an in-product stats view.

### API and event contracts

- Versioned HTTPS JSON endpoints with secure cookies, CSRF protection, request limits, and typed errors: These provide secure, typed contracts for authenticated state and mutations.
- Magic-link auth with neutral response, rate limits, 15-minute expiry, and atomic one-use consumption: Neutral responses avoid account enumeration; expiry and one-use consumption limit replay.
- Revocable per-device sessions: Device-session management can revoke access and is checked by protected routes.
- Snapshot endpoint with `afterVersion`: It returns projection, timing, weather, light, transition anchors, and a next pull hint, while unchanged versions can be lightweight for sync efficiency.
- Host greeting inputs in snapshot but omitted for visitor: Greeting is host-only and visitor arrival does not influence the host scene.
- Events endpoint accepting bounded batches with idempotency keys: It supports retry across network failures while validating ownership, sequence, cooldown, and session.
- Events endpoint rejecting trait or mood values: This preserves server authority over canonical personality and mood.
- Offer transient reaction envelope: The client can display an accepted reaction immediately while lasting effects wait for the next tick.
- Host-only notebook pagination: Notebook is private to the host and uses stable cursors.
- Bird rename endpoint: Names are validated while ownership and identity remain server-controlled.
- Adoption endpoint: It enforces age eligibility, unclaimed offer, ownership, and seven-bird cap in a transaction; no species catalog or rarity selection keeps adoption system-selected.
- Account/settings/session endpoints: They centralize settings and session revocation, and require new-email verification before change.
- Export, deletion, and restore endpoints: They provide short-lived verified-email export, immediate soft delete, explicit 30-day restore, and hard delete after deadline.
- Invitation endpoints: Explicit creation, host list, and revocation keep visits opt-in, operational, and not an automatic share prompt.
- Visit redemption and visitor snapshot endpoints: One-use email link redemption creates scoped read-only sessions, checks revocation/expiration on every pull, logs approximate visits silently, and exposes no mutation route.
- Stale client handling: Valid additive events may be accepted, but there is "no last-write-wins vector merge"; the client must refresh current version.
- Bounded short-disconnect event queue: The server rejects expired presence and old offers, because a long-offline browser "cannot simulate or author canonical history."

### Simulation design

- Minute-scale UTC tick with account timezone: UTC gives stable tick windows while stored IANA timezone drives local day phase, including DST.
- Locked tick transaction consuming events once: The lock, cursor, and atomic state commit prevent double consumption and partial state.
- Deterministic pseudorandom seeds: Choices derived from aviary ID, bird ID, tick index, and config version make retry produce the same result.
- External email outbox outside tick transaction: This keeps external delivery outside canonical state commits.
- Worker outage backfill through bounded windows: Backfill avoids skipping canonical time, caps work, reschedules until caught up, and alerts on lag.
- Browser presence conjunction of visible, focused, and recent pointer/key activity: The conjunction reduces idle/background inflation while allowing quiet watching.
- Four-minute activity freshness: The plan gives this so "quiet watching counts" and freshness is not click frequency.
- Presence heartbeat and prompt end on blur, hide, settle, or unload: Heartbeats keep leases current, and ineligible conditions end presence.
- Server clamping and union of presence intervals: This prevents stale heartbeats and multiple host devices from double-counting.
- No inferred presence from open tab, audio playback, visits, or stale heartbeat: This keeps presence bounded to eligible host attention only.
- Exponentially smoothed bounded presence signal: It lets regular presence shape drift over days while saturating per day.
- Additive nonnegative trait deltas: A bird "never loses boldness, warmth, voice, or plumage," and absence causes no negative personality drift.
- Recency/mood expression factor after absence: Apparent quietness can show absence without changing hidden vectors negatively.
- Event bonus caps and server offer cooldown: These prevent "button-mashing from dominating."
- Calibration harness and listening/visual panels: The plan uses no-visits, short visits, long watch sessions, frequent offers, and overlapping devices to tune felt continuity, because "numeric thresholds alone cannot judge felt continuity."
- Tick model/config version storage: It allows changes to be audited without recomputing vectors.
- Mood state machine: Transition weights from offers, bird interaction, personality, local day phase, and weather create continuity; opening a tab is "never a reset."
- Bird-authored perch choices with spacing constraints and no placement API: This keeps perches chosen by birds rather than user scene customization.
- Rare short rain and wind effects: Weather changes call probability and alert/wary mood without becoming a "task or warning surface."
- Night quiet and nightjar-like exception: NOT RECOVERABLE FROM PLAN.
- Arrival cue from mood, boldness, warmth, and absence length: Return behavior expresses canonical state and absence without resetting mood or showing a welcome banner.
- One likely greeter and staggered answers: It avoids a "synchronized chorus" and uses per-arrival variation.
- Visitor snapshots omitting greeting cues: Visitor arrival does not trigger or mutate host state.
- Species motif grammar and immutable bird signature seed: These preserve recognition through contour, timbre family, and interval/rhythm.
- Mood and vector effects on phrase density, timing, inflection, response probability, and dynamics: The variation stays bounded so recognition remains intact.
- Sparse deterministic call intents and response/chorus opportunities: They avoid exact periodic loops and prevent two identical call events in a row.
- Seven-bird chorus density control and priority: This preserves individual signatures at the maximum aviary size.
- User listening panels: They test whether familiar birds are identifiable across moods and aviary sizes.
- Notebook generation from true facts: Entries require source facts and avoid false claims.
- Notebook deduplication and cadence limits: Fingerprints and ordinary cadence of roughly one every few days keep the notebook sparse.
- Notebook exclusions of attendance, streaks, and raw trait deltas: This prevents the notebook from becoming a score or visit-frequency surface.
- Storing notebook prose at creation: History remains stable when templates change.

### Scene and interaction implementation

- Retained 2D scene graph plus DOM layer: The scene graph supports efficient rendering, while DOM handles top bar, focus targets, captions, and assistive narration.
- Responsive horizontal scene with all birds and perch zones in viewport: It preserves the single-screen aviary at phone and desktop widths.
- Restrained sky, foliage, perches, foreground leaf/feather, and parallax: NOT RECOVERABLE FROM PLAN.
- Six coherent species using silhouettes and compact vectors/atlases: NOT RECOVERABLE FROM PLAN.
- Trait-based visual saturation through bounded palette mapping: It reads hidden traits visually without displaying numbers.
- Local light from account timezone: It makes sunrise, day, evening, and night part of the current aviary state.
- Settling visual/audio override: It temporarily shifts toward evening and lowers calls over seconds until re-engagement or close.
- Undo click protected from offer or bird movement: This prevents a settle undo from accidentally becoming another interaction.
- Initial projection with pose and action start times: The first frame samples ongoing action instead of playing an entry animation.
- Snapshot interpolation anchored to server timestamps: It keeps motion aligned to canonical state transitions.
- Long render gap fetch before resuming: The browser avoids animating an "hours-old path."
- Hide behavior stopping animation/audio and closing presence: It conserves local work and prevents hidden tabs from continuing presence, while the server tick keeps running.
- Ambient leaves and feathers as client-only: They never determine mood or notebook facts.
- No controls, labels, badges, or tooltips inside the scene: The plan specifies the rule, but its specific rationale is NOT RECOVERABLE FROM PLAN.
- Thin top bar containing only account/settings, accessibility settings, notebook, and gestures: It reconciles the four-icon limit while keeping controls legible/focusable.
- Gestures control with offers and separate settle action: It exposes the offer affordance and keeps settle clearly separate inside the menu.
- Listen-in engagement and disengagement: Crossfades keep a focused bird audible without muting ambient birds, and disengagement ramps slowly.
- Offer menu reaction envelope: The server chooses actual bird reaction from current mood/personality/cooldown, and duplicates do not produce a second reaction.
- Still pool and song fragment events: They are brief scene/audio events, not meters.

### Audio and captions

- WebAudio motif synthesis library: It creates procedural calls with small assets, stable signatures, envelopes, filters, and per-bird gain/pan.
- Look-ahead scheduling tied to projection timestamps: It keeps audio timing aligned with server projection and visual events.
- Bounded jitter: It prevents mechanical repetition while preserving bird signature.
- Single call event object for synthesizer, beak/body cue, and caption generator: This makes audible, visible, and captioned calls agree.
- Captions derived from exact motif, contour, timing, and mood: They describe what was selected for the call rather than showing a static species label.
- Chorus limiter, density, and ducking policy: These protect headroom and intelligibility at seven birds.
- Never muting nonfocused birds under listen-in: This preserves ambient continuity even while focusing on one bird.
- Node and buffer pools, polyphony limits, hidden-tab scheduling stop, and gain fades: These prevent leaks, reduce load, and avoid abrupt audio cuts.
- Autoplay-safe first paint: The aviary and captions appear without waiting for AudioContext permission, and sound resumes on an allowed gesture.
- No disruptive autoplay modal: It protects the first encounter from an interruption.
- WebAudio failure path: The experience remains silent, captions default on, and the setting/error surface is matter-of-fact.
- No recorded-call fallback: NOT RECOVERABLE FROM PLAN.
- Headphone, phone speaker, clipping, fatigue, phase, intelligibility, and leak tests: These quality gates protect long listening sessions and seven-bird audio.

### Accessibility as a full scene

- Semantic DOM aviary region, named bird controls, observation, notebook, and top bar controls: These make the rendered scene available to assistive technology as an aviary, not just pixels.
- Arrow-key bird focus after Tab enters scene: It creates predictable keyboard navigation among bird focus targets.
- Enter, Escape, and focus-away listen-in behavior: These give keyboard users the same listen-in lifecycle as pointer users.
- High-contrast focus treatment in morning and night palettes: Focus remains visible across light states.
- Offer menu/settings tab order, focus trapping/restoration, and mobile touch targets: These preserve operability in dialogs and menus.
- Visitor focus/read without listen-in or host action: Visitors can understand the aviary without gaining host interaction powers.
- Naturalist narration from projection and facts: It avoids raw ARIA dumps and keeps narration in the same observational voice.
- Polite coalesced live-region updates every 30-60 seconds: Coalescing prevents flooding while still narrating ongoing life.
- Prioritizing user-initiated offer/settle and return greeting: User actions and arrival cues are narrated promptly.
- Manual "describe the aviary" action: Users can request current prose without waiting.
- Screen-reader audits for queue behavior: The plan calls this out because simultaneous captions and notebook navigation can conflict.
- Reduced-motion parallel renderer: It crossfades still poses, removes parallax and leaf drift, and preserves calls, captions, moods, drift, and notebook.
- Honoring `prefers-reduced-motion` on first paint with explicit override: Motion preference is respected immediately while remaining configurable.
- Captions near calling bird with viewport and contrast constraints: Spatial captions remain legible in every light/weather state.
- Nonspatial caption list for narrow screens/screen readers if proximity cannot stay legible: It preserves caption access when spatial placement fails.
- WCAG AA contrast automation and visual inspection: This protects top bar, system surfaces, captions, and rendered composites.

### Invitations, accounts, and privacy controls

- Transactional sign-up provisioning account UUID, aviary, and two starter birds: This creates the stable account and bird identity together.
- Showing system-selected species as arrivals, then collecting names with suggestions: NOT RECOVERABLE FROM PLAN.
- Changing names without altering bird identity: Names remain editable while UUID, vector, and history persist.
- Magic-link neutral responses, expiry, and one-use behavior: These avoid enumeration and token replay.
- Device sessions listed and individually revocable: Users can remove access per device.
- Pending email change until new address verifies: This prevents replacing the verified email before proof, while the old address remains valid.
- Export snapshot with vectors, notebook, and settings, encrypted at rest and emailed short-lived: The export is private user data delivered to the verified address without logging email or payload.
- Soft deletion, explicit 30-day recovery, purge job, and auditable status: Deletion blocks ordinary use immediately, allows recovery, then proves purge of account-linked records.
- Anonymous aggregate counters with no account dimension: This preserves privacy in metrics.
- One-recipient invitation with hashed redemption token and 30-day unused expiry: Invite access is scoped, private, and not transferable through stored raw tokens.
- Atomic invite redemption into limited browser visit session: Single-use redemption prevents replay and creates scoped read-only access.
- Visitor session renewal while invite remains valid but no transfer or write: Access persists only within the invite boundary and never grants mutation.
- Revocation returning unavailable on next snapshot pull: Revocation closes active views promptly.
- Frequent visible-tab visitor pulls plus authorization on every read: The plan targets revocation within about one minute and verifies access every time.
- Visitors seeing current host scene/audio/weather without prettifying it: Visitor view reflects the host scene, not a special presentation.
- Visitor omissions of notebook, controls, host-presence effects, greeting, co-presence marker, and interaction endpoint: These prevent privacy leaks and visitor influence on host state.
- Approximate visit duration in host settings only: Visit logging remains a privacy surface, with no badge or default notification.
- Optional host visit notification toggle off by default and absent from onboarding: Notifications occur only when requested and have "no engagement cadence."
- Simulation database unavailable to analytics jobs: Credentials and network policy enforce the privacy boundary.
- Aggregate-only operational metrics with forbidden dimensions and payloads: Observability cannot include account UUID, bird UUID, email, names, event payload, per-bird state, or per-account history.
- Short-retention redacted security debugging with synthetic IDs: Debugging is separate from behavioral dashboards and constrained by redaction.
- Privacy policy in account settings: It plainly states operational categories and excludes per-bird interactions from aggregation, training, and sharing.

### Performance, observability, and quality gates

- Initial gzipped JavaScript below 2 MB and CI bundle-size gate: The plan uses this to protect the first-bird performance budget.
- Lazy-loading notebook, account, accessibility, and invitation panels: Non-first-draw panels do not block first bird.
- First bird visible within 500 ms on mid-tier mobile/4G: The metric measures actual first rendered bird pixel, not shell paint.
- Inline or edge-delivered compact projection, small assets, critical CSS, and no audio/settings dependency on first draw: These all serve the fast first encounter.
- 60 fps idle target on five-year-old mid-range laptop for 30 minutes: Performance must hold over time, not only at load.
- Heap and audio-node plateau threshold in CI: This exposes leaks instead of masking a "steady positive slope."
- Worst-case seven birds plus captions/weather/reduced-motion profiling: The maximum supported scene is part of the quality gate.
- Synthetic browsers across geographies and last two major browser versions: The plan checks supported browser and geography coverage.
- Aggregate page-load, first-bird, frame-time, audio-context error, request, tick, backlog, and snapshot-size metrics: These monitor health without per-bird or per-account dimensions.
- Alerts on tick latency, backlog, and first-bird/render regressions: These protect continuity and performance budgets.
- Unsupported older browser explanation: The surface stays "plain matter-of-fact."
- Performance release gate scenarios: First visit, returning visit, hidden/resume, slow network, and seven-bird long session must all pass.

### Delivery sequence and verification

- Contracts and foundations first: Terminology, projection/event schemas, design tokens, accessibility semantics, timezone behavior, and privacy metric schema are locked before later work.
- Early account UUID/auth/session primitives and migrations: The plan proves one-use magic links, revocation, ownership, and absence of email in internal identifiers/logs.
- Canonical engine before visual polish: Scheduler, tick/cursor, birds, mood, weather, call intent, presence, drift, cooldowns, adoptions, and notebook facts must pass deterministic and failure tests first.
- Assertions that only server tick writes vectors, no negative delta, no double consumption, and no client merge: These protect canonical state.
- First complete aviary pairing visible behavior with narration, caption, and reduced motion: Accessibility surfaces ship alongside visible behavior.
- Naturalist copy validation against generated observations: Copy must match real state and moment.
- Audio and accessibility listening and screen-reader studies: They test recognition and whether alternate surfaces feel like "a continuing aviary rather than a status feed."
- Account completeness and visits pen tests: Token replay, cross-account access, visitor mutation, stale revoked sessions, and export-link leakage are tested.
- Ramp from internal dogfood to external cohort and natural age eligibility: Access expands gradually and without engagement prompts.
- Simulation fixtures for load and seven-bird audio tests: Later bird counts are tested before naturally occurring for all users.
- Gates on recognition/listening, frame/audio memory, captions, and sync stability: Each bird-count increase waits for experiential and technical stability.
- Rollback flags retaining canonical bird IDs/vectors: Renderers or model configuration can roll back without resetting birds.
- Release acceptance scenarios: They cover concurrent devices, background overnight, four-minute no input, settle versus close, worker outage, DST/timezone change, rename/migration, visit revocation, WebAudio denial, reduced motion from first paint, screen-reader narration, and seven-bird duration.
- Final shipping requirements: Shipping requires "no known identity/vector loss, no drift from absence, no visitor influence on host state, and no missing accessibility mode."

### Principal risks and responses

- Drift too fast, too slow, or punitive: The response is time-compression simulation, blinded visual/audio comparisons, versioned rates, recency expression, no negative vector drift, and no user-facing meter.
- Presence inflated by idle/background or duplicate devices: The response is browser conjunction tests, server expiry and interval union, duration caps, and synthetic scenario inspection rather than population histories.
- Concurrent ticks or retries losing or double-counting drift: The response is row lock/compare-and-swap, atomic cursor/state commit, idempotency, deterministic seeds, failure injection, and restore drill.
- Calls canned, clipping, or blurred at seven birds: The response is motif variation, stable signature constraints, limiter/density control, repeat detector, and human listening panels.
- Autoplay or WebAudio failure breaking first encounter: The response is visual scene and captions independent of AudioContext, resume on gesture, silent captioned fallback, and aggregate error counts.
- Accessible experience becoming a state dump or static image: The response is to ship narration grammar and reduced-motion renderer with the normal renderer, test actual screen readers and vestibular settings, and hold release on regressions.
- First-frame budget forcing spinner or stale bird: The response is edge private projection, tiny critical path, synthetic 4G gate, quiet-field delay handling, and stale-snapshot refresh.
- Visitor access leaking private state or surviving revocation: The response is separate visitor credentials/projection, endpoint authorization on every pull, no visitor write capability, short pull cadence, and security tests.
- Privacy boundary eroding through observability: The response is allowlisted aggregate metric schema, restricted analytics credentials, CI checks for forbidden dimensions/payloads, redacted logs, and deletion audit.
