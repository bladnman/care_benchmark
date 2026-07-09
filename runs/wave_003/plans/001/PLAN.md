# Pocket Aviary v1 implementation plan

## 1. Delivery goal and product invariants

Build a web-only, single-account/single-aviary product in which two initially adopted birds continue to change on the server whether a client is open or not. The implementation is successful only if the first visible frame reads as a continuing place, presence can be trusted as an input, each bird remains recognizable over weeks, and accessible modes preserve that same character.

Treat the following as release-blocking invariants rather than preferences:

1. The server is the sole authority for birds, personality, mood, offers, notebook observations, and simulation time. Clients submit facts and render derived state; they never submit personality or mood values.
2. A bird's stable UUID and personality vector survive renames, device changes, deployments, migrations, and species-library revisions. Destructive replacement or implicit regeneration is forbidden.
3. Personality drift is slow and non-negative. Absence may reduce current expressiveness through mood and recency, but it never subtracts from an accumulated personality trait.
4. Presence requires all three signals at the same time: visible document, focused window, and recent pointer or keyboard activity. Concurrent devices cannot double-count the same wall-clock interval.
5. Normal product APIs, logs, analytics, ARIA data, and UI never expose numeric personality values. The user-requested account export is the one explicit product exception.
6. Calls are generated from procedural grammars in the browser. There are no recorded call assets or loop fallbacks.
7. A visitor can only render a revocable, read-only view. Visitor activity never enters the host's simulation.
8. Product prose uses the naturalist voice; identity, settings, accessibility configuration, unsupported-browser, and error surfaces use direct system language.
9. V1 ships only when the ordinary, screen-reader, captioned/silent, and reduced-motion experiences are complete and tested.
10. Production telemetry is operational and aggregate-only. It cannot be joined to a bird, aviary, account, email, invitation, or interaction stream.

Document these invariants as architecture tests and code-review checks. Reject changes that add a toast, streak, score, status meter, public discovery, caretaking penalty, client-side simulation, or per-account analytics even if the change appears locally harmless.

## 2. V1 scope

### Included

- Responsive browser aviary for the last two major versions of Chrome, Safari, Firefox, and Edge.
- Email magic-link authentication, revocable device sessions, verified email changes, account export, and 30-day recoverable deletion.
- One canonical aviary per account, two starter birds, user naming and renaming, an approximately six-species coherent pool, and an age-gated path to no more than seven birds.
- Persistent personality and mood; server simulation at approximately one-minute cadence; local-day cycle; rare ambient weather; bird-to-bird responses; procedural greetings, calls, and chorus behavior.
- Honest presence accounting, listen-in, the three offer types, settle with five-second local undo, and a sparse read-only field notebook.
- Multi-device snapshot consumption with no client state merge and no last-write-wins personality path.
- Per-email, one-time visit invitations; visitor rendering; revocation; expiration; host visit log; optional visit notifications off by default.
- Naturalist screen-reader narration, runtime call captions, complete keyboard interaction, WCAG AA text contrast, and designed reduced-motion rendering.
- The specified launch/runtime performance budgets and privacy-preserving operational observability.

### Explicitly excluded

- Native apps or native-oriented protocol work, passwords, SSO, payments, subscriptions, multiple aviaries, shared or household accounts, and customizable scenes.
- Hunger, illness, death, distress, decaying happiness, required visits, required settling, or any other punishment for absence.
- Scores, levels, achievements, badges, streaks, visit calendars, counters, rarity, attention-based unlocks, or numerical bird/personality dashboards.
- Profiles, follows, chat, comments, co-presence, public links, feeds, discovery, friend-of-friend access, comparisons, leaderboards, or visitor interaction.
- Push notifications and default notifications of any kind. The only visit notification considered in v1 is an explicit account-level email opt-in, off by default and absent from onboarding.
- Recorded calls, a second audio implementation, per-leaf server state, or an old-browser compatibility bundle.
- Production ML training, recommendation systems, or population analysis using interaction, bird, mood, personality, notebook, presence, or visit data.

## 3. Proposed system shape

Use a small TypeScript monorepo with separately deployable packages and strict schema boundaries:

- **Web client:** Preact plus TypeScript for controls and account surfaces; a custom Canvas 2D scene renderer; DOM-based semantic controls, focus proxy elements, captions, and live narration; WebAudio for calls. Build with route-level code splitting so account, notebook, accessibility, and invitation code are not in the first-aviary chunk.
- **Edge web layer:** serves the HTML shell, hashed static assets, compatibility checks, and an authenticated bootstrap snapshot. Inline only the minimum current scene state and starter visual primitives needed for the first bird; never put private HTML in a shared cache.
- **API service:** versioned JSON APIs for auth, snapshots, commands/events, notebook, account operations, and invitations. It authenticates the actor, validates event schemas and idempotency, and exposes no direct simulation mutation endpoint.
- **Simulation workers:** claim due aviaries, fold unconsumed events transactionally, advance mood/weather/call plans, apply drift, select notebook observations, and publish a new immutable render snapshot revision.
- **PostgreSQL:** canonical accounts, aviaries, birds, simulation state, append-only input events, notebook entries, invitations, sessions, visit logs, and tick/outbox bookkeeping. Use row transactions and constraints rather than distributed client reconciliation.
- **Scheduler/queue:** enqueue only an aviary UUID and due logical time. At-least-once delivery is acceptable because the worker is idempotent by tick boundary and event watermark.
- **Private snapshot cache:** a small, encrypted, revisioned read-through cache populated only after the canonical database transaction commits. It improves snapshot latency but is disposable and never authoritative.
- **Transactional outbox:** carries snapshot-cache invalidations and authorized email jobs. It never carries personality values or raw interaction payloads into general infrastructure.
- **Email provider:** sends magic links, invite links, export links, email-verification links, and explicitly opted-in visit notices. Provider metadata uses delivery IDs and account UUIDs, not plaintext email in application logs.
- **Operational telemetry pipeline:** accepts pre-aggregated counters, histograms, and errors with bounded dimensions. It has no credentials or network route to read the simulation database.

Keep the API and worker in the same codebase and data model for v1, while deploying them as separate processes. This avoids premature service boundaries without letting request latency or worker load interfere with one another.

### Canonical-state and render boundary

The canonical database stores semantic state: bird identity, personality, mood, perch, ongoing behavior, weather, local-day phase, and server schedules. The published client snapshot contains only what is required to reproduce the current experience:

- stable bird ID, name, species asset version, perch/pose and timed transition descriptors;
- mood-derived presentation tags, never raw mood probabilities or personality values;
- a stable voice identity plus upcoming procedural call instructions and seeds;
- current lighting/weather, server time, snapshot revision, and a bounded future action window;
- a one-session greeting cue when applicable;
- capability and accessibility-relevant presentation data.

The client interpolates timed transitions and synthesizes scheduled calls; it does not choose the next mood, perch, weather event, call frequency, offer outcome, or personality delta. Purely ornamental leaves/feathers and subtle parallax use deterministic client-local randomness and never feed back into state.

## 4. Data model and retention

Use UUIDv7/UUID values for internal identity and UTC timestamps everywhere. Store the account's chosen IANA time zone separately for simulation. Seed it from the browser during onboarding, show it in account settings, and require confirmation before changing it; never let two devices silently fight by repeatedly overwriting it. A visitor renders in the host aviary's time zone.

### Core records

**accounts**

- `id` synthetic UUID primary key.
- `email_ciphertext`, `email_key_version`, and a keyed blind index on the same record for equality lookup. The blind index is auth-service-only and must never be used as an identifier, log field, partition key, or telemetry dimension.
- `email_verified_at`, `iana_timezone`, `created_at`, `deletion_requested_at`, `hard_delete_after`.
- `visit_notifications_enabled` default false and accessibility/settings defaults.

**device_sessions**

- `id`, `account_id`, hashed bearer-token identifier, coarse user-agent label for the user's session list, `created_at`, `last_used_at`, `expires_at`, `revoked_at`.
- Do not store IP history. If short-lived abuse controls require an IP, keep it in the auth gateway with a brief TTL and no account join key.

**magic_links / email_change_tokens**

- Hashed random token, purpose, `account_id` when known, encrypted target email for email change, 15-minute expiry for sign-in, consumed timestamp, and attempt metadata.
- Consume with a single conditional update so concurrent clicks cannot issue two sessions.

**aviaries**

- `id`, unique `account_id`, `created_at`, `species_pool_version`, current `revision`, `last_tick_at`, `next_tick_at`, and event-consumption cursor.
- `current_weather_state`, weather start/end, local-day phase, and deterministic simulation seed.
- Do not store a user-facing visit count, streak, score, or engagement total.

**birds**

- Stable `id`, `aviary_id`, user name, species ID/version, adoption timestamp/order, and immutable voice/behavior seed.
- Five normalized traits: boldness, social warmth, vocal frequency, plumage saturation, curiosity. Use typed numeric columns with database bounds, not an unvalidated JSON blob.
- Current mood enum (`wary`, `content`, `curious`, `drowsy`, `alert` for the initial implementation), mood-entered time, transition/dwell metadata, perch zone, pose/action, and action start/end.
- Per-trait low-pass accumulator and calibration-version fields so engine migrations are explicit and replay-free.
- No foreign key from analytics or email systems.

**interaction_events**

- `id`, `aviary_id`, authenticated actor account ID, device session ID, client-generated idempotency key, per-session sequence, server receipt time, client monotonic offset, event type, validated payload, and `consumed_tick`.
- Types: session/open-or-return, presence interval, listen-in start/end, offer request/result reference, settle, settle undo/reengage, rename, and adoption acceptance where applicable.
- Partition by receipt time. Raw events remain inside the simulation store, are compacted after successful consumption, and expire after a short operational window (target seven days). Persist only the engine's bounded accumulators and durable product outcomes. No raw event export to a warehouse.

**presence_sessions**

- Server session ID, account/aviary/device, opened/last-seen/ended times, last accepted sequence, and terminal reason.
- Accepted presence intervals are clamped, merged by wall-clock time across devices, and folded into the aviary accumulator. Raw pings expire with interaction events.

**offer_commands / offer_cooldowns**

- Command ID, aviary, offer type (`seed`, `song_fragment`, `still_pool`), receiving bird selected by the engine, submitted/decided times, reaction descriptor, and simulation event reference.
- A unique per-bird cooldown row/constraint prevents two devices from accepting offers inside the same few-minute window. Rejected cooldown attempts do not create drift.

**simulation_ticks**

- Aviary, logical tick boundary, engine/calibration version, input watermark, output revision, start/completion time, and status. Unique `(aviary_id, logical_tick_boundary)` makes retries idempotent.
- Tick rows contain no prose rationale and are retained only as needed for operational correctness.

**notebook_entries**

- Stable ID, aviary, observation time/local date, naturalist prose, source observation category, referenced bird IDs, and prose-template version.
- Entries are immutable/read-only and retained indefinitely until account deletion. The structured source is internal, contains no personality number, and supports safe future wording migrations without rewriting history.

**invitations**

- ID, host account/aviary, encrypted invitee email, hashed token, created/expires/consumed/revoked timestamps, and active visitor-session generation.
- Expire unused tokens after 30 days. Consuming once creates a scoped visitor session; it does not create a profile, friend edge, or permanent public URL.

**visitor_sessions / visit_log_entries**

- Visitor session is scoped to one invitation and aviary, has expiry/revocation generation, and has no mutation permissions.
- Visit log stores invitation ID, encrypted/displayable invitee email reference, started/ended time, and rounded approximate duration. It is visible only to the host in settings and deleted with the account.

### Export and deletion

Generate exports in an isolated job from the canonical store. The JSON includes birds, names, numeric personality vectors, current moods, notebook entries, and account settings as required, with a schema/version header and explanatory warning that the trait values are normally intentionally hidden. Encrypt the generated object, email a short-lived signed URL to the verified address, and delete the object after 24 hours. Never place exports in analytics or ordinary CDN logs.

Deletion immediately revokes sessions, links, invites, and visitor access and marks the account recoverable. A recovery clears the pending deletion and issues a fresh session. At 30 days, a cascading, retryable erasure job removes canonical rows, cache entries, queued emails, exports, and any account-keyed operational records; an erasure audit stores only an unlinked completion count and time. Backups follow a documented expiry schedule and cannot be restored at single-account granularity into production.

## 5. API contracts

Use `/v1` JSON endpoints, secure same-site cookies for owner sessions, CSRF protection on mutations, and opaque bearer cookies for invitation sessions. Every mutation accepts an idempotency key; every successful state-changing response returns the resulting canonical revision or command ID. Use matter-of-fact RFC 9457-style problem responses for errors.

### Auth and account

- `POST /v1/auth/magic-links` with email; always return the same neutral response. Rate-limit by a short-lived keyed email digest and network abuse bucket. Link expires after 15 minutes.
- `POST /v1/auth/magic-links/consume` atomically consumes the token and establishes a device session.
- `GET /v1/account` returns settings and the revocable device list, never personality or interaction history.
- `PATCH /v1/account` updates time zone, accessibility preferences, and the off-by-default visit notification preference with optimistic versioning.
- `POST /v1/account/email-change` and `POST /v1/account/email-change/verify` retain the old address until verification commits.
- `DELETE /v1/account/sessions/{id}` revokes a device session.
- `POST /v1/account/exports` queues an export and returns only job status; delivery is to the verified email.
- `POST /v1/account/deletion`, `POST /v1/account/deletion/recover` implement the 30-day flow.

### Owner aviary

- `GET /v1/aviary/bootstrap` returns the smallest authenticated snapshot plus asset manifest/version. Support `ETag`/`If-None-Match`, but bypass shared caches.
- `GET /v1/aviary/snapshot?after_revision=N` returns `304`, a full small snapshot, or a bounded delta plus authoritative server time. A full snapshot remains only kilobytes.
- `POST /v1/aviary/session` opens/resumes a rendering session and provides visibility-return/absence context; the server may attach exactly one primary greeting plan and an optional staggered response. A visitor cannot call it.
- `POST /v1/aviary/events:batch` accepts sequenced presence and listen-in/settle facts. It returns accepted sequence and server-clamped intervals; it never returns drift deltas.
- `POST /v1/aviary/offers` asks the server to evaluate one of the three offer types. The response is an accepted reaction cue or a quiet cooldown/unavailable result, not a trait explanation.
- `POST /v1/aviary/settle` ends this device's presence window and returns a timed visual/audio settle cue. Settling is a per-session presentation state plus a small server mood input, so one device cannot force another active device into an evening overlay. Re-engagement within five seconds sends an idempotent undo/reengage fact.
- `PATCH /v1/birds/{bird_id}` permits name changes only; validate length, Unicode, and product-safe rendering.
- `GET /v1/notebook?before=cursor&limit=N` returns immutable entries in reverse chronological pages.
- `POST /v1/adoptions/{offer_id}/accept` accepts only a server-created age-gated offer and cannot exceed the current rollout cap or seven-bird hard cap.

Do not create generic PATCH endpoints for birds, aviaries, moods, perches, or vectors. Generated API clients should make forbidden state writes impossible by construction.

### Invitations and visits

- `POST /v1/invitations` takes an invitee email, creates a 30-day one-use token, and sends it. There is no public/shareable URL endpoint.
- `GET /v1/invitations` lists outstanding/active invitations and the visit log for the owner on demand; it does not return an unread count.
- `DELETE /v1/invitations/{id}` increments the revocation generation and invalidates unconsumed tokens/active sessions immediately.
- `POST /v1/visits/consume` exchanges the one-time token for a scoped visitor session.
- `GET /v1/visits/snapshot?after_revision=N` rechecks invitation status on every pull and returns scene/audio state only. It excludes host account controls, notebook, offer/listen-in/settle controls, greeting cues, presence endpoints, and all mutation affordances.
- `POST /v1/visits/end` is a best-effort duration close; server timeout provides the fallback.

Revocation must be visible no later than the next low-frequency snapshot pull. Return the same direct “visit no longer available” response for expired, used-without-session, and revoked links so token state is not disclosed.

## 6. Simulation engine

### Tick transaction

Run nominal ticks once per minute using logical UTC boundaries. For each due aviary, the worker:

1. Claims the row with a lease and opens a database transaction using a row lock.
2. Computes every missed logical boundary since `last_tick_at`, up to a safety cap. For a long outage, integrate slow accumulators over the elapsed span and materialize only the final/intermediate states required for correctness rather than replaying millions of render moments.
3. Reads validated, unconsumed events in server-receipt order plus idempotency/sequence rules; merges overlapping presence intervals across devices.
4. Updates presence and interaction low-pass accumulators, then applies non-negative trait deltas.
5. Evaluates local-day phase, scheduled ambient weather, current mood dwell, bird-to-bird effects, offer outcomes, and perch/action transitions.
6. Produces a deterministic near-future call/action plan and evaluates noteworthy notebook candidates.
7. Writes birds, simulation state, event watermark, a new immutable render revision, and the unique tick-completion row in one transaction.
8. Adds a snapshot invalidation/publication event to the transactional outbox and commits. A retry sees the unique tick and watermark and cannot apply drift twice.

Keep the engine as a pure, versioned function around explicit state, elapsed time, folded inputs, and deterministic random seed. Database and queue adapters wrap it. This enables weeks-long accelerated tests without clocks, browsers, or services.

### Presence model

The client maintains a presence state machine from four browser inputs: `visibilitychange`, window focus/blur, pointer movement, and keyboard input. A qualifying interval begins only while visible + focused + activity age below a configurable threshold. Start with a three-minute activity window and 30-second heartbeats, then calibrate using synthetic/user-research sessions rather than production per-account analysis.

Each heartbeat reports the locally observed qualifying interval since the previous accepted sequence, not an unbounded total. The server:

- rejects intervals during hidden/blurred periods reported by the same state machine transition stream;
- clamps future times, long gaps, heartbeat duration, and out-of-order sequence;
- merges the union of simultaneous intervals across devices so two open screens do not double drift;
- closes a window on settle, page lifecycle termination when delivered, heartbeat timeout, session revoke, or auth expiry;
- treats missing close events as timeout, never as continuing presence.

Pointer/keyboard content is never captured—only a recent-activity timestamp. Pointer movement should be rate-limited locally. Do not count touch scroll or media playback unless it maps to an allowed recent activity signal; settle and tab close both end presence without penalty.

### Drift function

Represent each trait and its latent signal in `[0,1]`. For trait `j`, update a slow input accumulator at each tick:

`signal_j(t+dt) = signal_j(t) * exp(-dt / tau_j) + normalized_weighted_inputs_j`

Then apply:

`delta_j = max(0, rate_j * response(signal_j) * (1 - trait_j) * dt)`

and `trait_j = clamp(trait_j + delta_j, 0, 1)`.

Use presence as the dominant input to every expressive dimension, listen-in as a stronger targeted input for that bird's warmth/vocal dimensions, and accepted offers as small curiosity/boldness inputs. Settle affects current mood/presence only. Saturating response and `(1-trait)` prevent runaway growth. Zero input yields zero delta; it never yields a negative delta.

Calibrate with deterministic cohorts representing no visits, quiet regular visits, long sessions, many short sessions, multi-device overlap, offers at cooldown, and listen-in. Required targets:

- no trait changes in the absence-only cohort;
- instrument-detectable but visually imperceptible movement around seven days of regular presence;
- blinded visual/audio review detects a coherent difference around three weeks, not after one session;
- repeated clicking cannot outperform honest presence enough to turn interactions into an optimization strategy;
- clamping and saturation keep every vector valid over multi-year accelerated runs.

Store calibration version with each bird. Engine upgrades operate forward on current values; never rebuild a vector from event history. Roll out new calibration in shadow simulation on synthetic fixtures, then a small production cohort selected without inspecting individual state.

### Mood and expression

Use the five initial mood states with a transition matrix modified by time of day, weather, recent accepted interactions, nearby bird signals, and personality. Enforce minimum dwell/hysteresis so a bird cannot flicker between states. Around local dawn, draw a new daily baseline from personality and environmental context, but transition from the persisted prior mood rather than resetting on tab open.

Separate accumulated personality from recent-expression modifiers. After a long absence, a bird can greet less and call more quietly because current recency/mood weights are low; its boldness, warmth, color, and other learned traits remain intact. This is how “ambient after absence” coexists with monotonic drift.

Perch selection is a server decision: bold/curious/content weights favor front/middle, wary favors back, drowsy favors stable sheltered poses. Apply occupancy and spacing constraints so up to seven birds remain legible and never collide. Other birds' wary cues can temporarily raise wary transition probability; compatible call windows can create chorus plans.

### Local day and weather

Derive sun phase from the account's IANA zone with timezone-database rules, including daylight-saving changes. Lighting is continuous, while mood weights change gradually. At night most species settle, with the nightjar-like species retaining a bounded late-call probability.

Generate weather from a per-aviary seeded calendar, targeting a short rain a few times per local week and occasional soft wind. Weather must be rare, non-destructive, and free of thunder/snow. Its state is canonical so owner devices and visitors see the same event. Rain temporarily damps call scheduling; wind modestly shifts alert/wary probabilities. Do not generate weather as a response to user behavior.

### Return greeting

On owner session open or a real return-to-visible transition, compute absence from the last accepted owner-presence endpoint, not “last HTTP request.” Select one primary bird using mood, boldness, warmth, recent greeter history, and a seeded variation term. Generate a short/medium/long-absence cue from current action context so it begins from a plausible mid-action pose. If another bird responds, schedule a randomized stagger rather than a simultaneous chorus. Never provide greetings to visit sessions and never pair the cue with text UI.

### Notebook generation

The tick emits structured observation candidates only for specific moments: unusual greeter order, extended quiet/preening, weather-linked behavior, first use of a perch in a recent window, a chorus, or a visible slow change. A sparsity gate considers time since last entry, novelty category, and recent wording; target roughly one entry every few days for a regularly visited aviary.

Render prose from reviewed deterministic grammar/templates rather than sending private state to a third-party model. Templates are lowercase, present tense, named/specific, and cannot mention visits, streaks, user behavior, numeric traits, generic achievements, or system timestamps. Snapshot tests and editorial review cover every template combination. Persist the final prose so old entries never change with a template deployment.

## 7. Client rendering and interaction pipeline

### First-frame path

1. Return a server-rendered quiet field immediately with critical sky/perch styles and no spinner.
2. Embed the authenticated minimal bootstrap snapshot in the HTML when available and preload the tiny initial species atlas/grammar manifest.
3. Draw at least one bird synchronously in its current phase before hydrating nonessential controls. Derive animation phase from server action start/end and current server time, so it is already mid-preen/call rather than starting an entry sequence.
4. Hydrate the top bar and full scene, fetch any later assets, then begin snapshot keepalive. Account, notebook, invite, and advanced settings chunks load only on demand.
5. On a cold/slow dynamic snapshot, leave the quiet field with faint ambient color motion; never display a progress spinner or fake bird. The empty post-adoption state uses this same field once, then a soft first fly-in.

Instrument `navigationStart` to the first successfully painted bird with a dedicated performance mark. Do not count the quiet field as a bird.

### Scene renderer

- Use one Canvas 2D scene with background, middle bird/perch plane, and restrained foreground. Keep all actionable UI and semantic equivalents in DOM layers outside the scene.
- Represent bird behavior as interruptible pose/state graphs driven by server transition descriptors. Blend poses locally but do not invent semantic behaviors.
- Fit the three perch zones into the viewport via scene coordinates and safe margins. On narrow phones compress horizontal gaps and bird scale within tested bounds; never crop a bird or add pan/zoom/scroll.
- Use `requestAnimationFrame`, pooled objects, cached paths/atlases, and dirty-region or layer caching where profiling supports it. Pause rendering and ornamental generators while hidden; on return discard stale interpolation, pull a snapshot, and resume from authoritative time.
- Generate leaf/feather ornaments locally at slow bounded cadence. Remove leaf drift entirely in reduced-motion mode.
- Drive the local light palette from canonical day phase and settle overlay. Top bar is a separate sparse DOM strip containing account/settings, accessibility, notebook, offer, and settle access; fade it nearly transparent after inactivity but restore it on any keyboard activity/focus, not only pointer movement.

### Snapshot interpolation and recovery

Keep two revisions at most. Synchronize to server time using a smoothed offset. Interpolate perch/action transitions with specified easing and duration; never extrapolate semantic state beyond the bounded plan. On revision gap, long animation frame gap, visibility return, device resume, or cache mismatch, request a full snapshot and cross-fade/reconcile to it without resetting the whole scene.

If the network is temporarily unavailable, finish only already scheduled micro-actions, show no alarm toast, and retry with capped jitter. Do not advance mood, weather, drift, offers, or notebook locally. If staleness exceeds the bounded plan, hold a plausible idle pose until a snapshot arrives. A sustained/auth failure moves to a direct system surface outside the scene.

### Interaction behavior

- **Listen-in:** click/tap or Enter on a focused bird toggles focus; selecting another transfers focus; empty scene, second activation, Escape, or moving keyboard focus away disengages. Send start/end facts, but change the local mix immediately with an idempotent server record following. Other birds ramp to ambient, never zero.
- **Offer:** open from the top bar/shortcut, keyboard-select one of three gifts, and send a command. Animate only the authoritative receiving-bird reaction. Present cooldown as the bird not taking another offer yet in restrained naturalist copy, without a countdown meter.
- **Settle:** trigger from the top bar, ramp scene toward evening and calls down over several seconds, and end local presence. For five seconds, any aviary click/tap/keyboard re-engagement reverses the local ramp and reports undo. Closing without settling performs no special action or recovery message.
- **Notebook:** open a lazy-loaded, read-only, virtualized overlay/panel with stable focus management and indefinite cursor pagination. It never exposes event-log rows or allows editing/deleting.
- **Adoption:** server age policy creates an offer independent of attention. Present the arriving bird, ask for a name, and preserve the “bird that arrived” framing; never display a species catalog, rarity, counter, or reward language.

## 8. Procedural audio

### Grammar and synthesis

Create a reviewed motif library per species containing relative pitch contours, note envelopes, gaps, timbre/noise parameters, and legal combination rules. Each bird's immutable voice seed derives a stable base register, timbre, and microtiming signature within species-safe bounds. Mood and the server call plan alter tempo, intensity, spacing, and response likelihood without erasing identity.

For each scheduled call, the snapshot supplies bird ID, grammar/asset version, event seed, start time, motif intent, and bounded mood-derived parameters. The client grammar expansion is deterministic. It creates oscillator/noise/envelope nodes or reuses pooled buffers and connects them through a per-bird gain/pan strip into ambient/listen-in/settle masters. It never downloads or caches recorded calls.

Use look-ahead scheduling against `AudioContext.currentTime` so calls remain stable across render-frame variance. Limit simultaneous voices, compressors, reverb, and total gain to prevent clipping when up to seven birds chorus. Bird-to-bird chorus timing comes from the canonical call plan; client scheduling adds only inaudible sub-frame precision.

### Listen-in and lifecycle

Ambient mix keeps every bird audible at a low, spatially coherent level. On listen-in, ramp the selected bird up and others down over a calm, tested interval (initial target 600–1000 ms); apply the inverse ramp on exit. Never hard-cut or mute nonfocused birds. Settle uses a slower master/call-density ramp, while reengage reverses it smoothly.

Respect browser autoplay rules: draw the moving aviary immediately, then start/resume audio on the first permitted user gesture. Explain blocked audio only in accessibility/settings, not a scene toast. Suspend and release scheduling when hidden, reuse nodes/buffers, cap every queue, and close abandoned contexts.

### Captions and fallback

Generate a structured caption from the exact expanded call event—contour, note count, rhythm, intensity, source perch—then render reviewed naturalist phrasing near the bird. Captions therefore cannot disagree with audio. They are opt-in normally; if WebAudio construction/resume fails, switch to graceful silence and enable captions by default for that session with a direct explanation in settings. Do not load a recorded fallback.

Test call distinctiveness with blind listening across species, birds of the same species, moods, and three-week drift fixtures. Establish a recognizability threshold before enabling bird counts above two; seven remains a hard cap even if lab results are stronger.

## 9. Accessibility implementation

### Semantic scene and narration

Provide a DOM scene model parallel to Canvas: one named aviary region, an ordered list of birds as keyboard focus targets, and concise action instructions. Do not expose mood labels or trait numbers as routine ARIA properties. The rendered scene and semantic layer consume the same presentation snapshot.

Build narration from structured scene facts with an editorially reviewed naturalist grammar. An idle queue emits one cohesive prose observation every 30–60 seconds. Return greetings, authoritative offer reactions, and settle observations enter a higher-priority queue but coalesce rather than interrupt repeatedly. Use a dedicated polite live region, suppress duplicate text, clear stale queued observations, and offer pause/repeat controls in accessibility settings.

Validate behavior with VoiceOver/Safari, VoiceOver/iOS browser, NVDA/Firefox and Chrome, and JAWS/Chrome where available. Acceptance is comprehension and calm queue behavior, not merely the presence of labels.

### Keyboard and focus

- Tab order covers sparse top-bar controls, then the aviary group. Arrow keys move among birds in spatial/order-consistent sequence; Enter toggles listen-in; Escape exits it.
- Offer, settle, notebook, account, invitation, and accessibility dialogs are fully operable with predictable focus entry/return and no traps.
- Focus rings remain visible at WCAG AA-compatible contrast in morning, midday, evening, night, rain, and settle palettes. Top-bar fade cannot hide the currently focused control.
- Touch targets and pointer alternatives meet platform sizing guidance; no behavior depends on hover alone.

### Reduced motion and visual access

Resolve reduced motion as the OR of `prefers-reduced-motion` and the account setting, with an explicit local preview. Use curated still poses with slow cross-fades; convert flights/perch moves to source/destination cross-fades; remove parallax and leaf/feather drift; retain slow color/day/settle changes. Do not replace the scene with a static illustration.

All top-bar labels, captions, displayed narration, settings, and system/error copy meet WCAG 2.2 AA contrast. Test browser zoom, 200% text, narrow viewports, forced colors for controls, and color-vision simulations. No essential bird state is conveyed only by plumage color.

### Accessibility release gate

Maintain automated axe/semantic/keyboard tests plus manual assistive-technology scripts for: sign-in, adoption/naming, return greeting, bird navigation/listen-in, each offer, settle/undo, notebook, invitation management, visit view/revocation, export, and deletion. A regression in narration cadence, reduced-motion completeness, captions, keyboard reachability, or contrast blocks the same release as a broken visual scene.

## 10. Sync, consistency, and failure handling

- Serialize each aviary's tick under a database row lock and unique logical boundary. Only that transaction writes personality/mood.
- Deduplicate every command/event by actor scope plus idempotency key. Preserve server receipt order, using client sequence only to detect gaps/duplicates—not to trust client clocks.
- Apply rename/settings changes with record versions and return a direct conflict response containing current editable values. Never surface or ask the user to choose between two personality states.
- Publish a snapshot revision only after its canonical transaction commits. Cache reads include revision; stale cache entries are replaced or bypassed, never written back.
- A phone and laptop may render different interpolation frames but must converge on the same semantic revision. Settle/listen-in are session presentation states; offers, mood, birds, notebook, weather, and drift are canonical.
- On worker outage, queue due ticks and catch up from elapsed time/input watermark. Clients keep bounded idle rendering but do not simulate. Alert before tick latency p99 exceeds five seconds.
- On expired auth, failed magic link, revoked visit, unsupported browser, or unrecoverable snapshot error, leave the naturalist register and provide a concise action. Do not mask system failures with themed prose.

Run concurrency tests that open multiple owner devices, duplicate/reorder all event types, submit simultaneous offers/renames, revoke a visit mid-pull, retry a tick after commit uncertainty, and resume after daylight-saving and multi-hour suspension. Assert no lost or double drift and no client-authored canonical fields.

## 11. Performance budgets and observability

### Enforced budgets

- Initial JavaScript at first paint: less than 2 MB gzipped, with a tighter team target (for example 350 KB) for the aviary route so asset growth remains visible. CI reports each entry chunk and fails the hard cap.
- First bird painted: under 500 ms on the agreed mid-tier mobile profile over shaped 4G, tested from cold cache and representative geography. Track p50/p75/p95; the release gate is the specified test condition, not a fast developer machine.
- Idle rendering: 60 fps on a named five-year-old mid-range laptop for a continuous 30-minute trace, including day lighting, two-bird calls, and periodic interactions. Define acceptable frame-time percentile and no long-task bursts.
- Memory: no positive retained-heap trend across a scripted 30-minute session after warm-up and garbage collection checkpoints. Explicitly inspect audio nodes/buffers, animation objects, event listeners, snapshot retention, virtualized notebook rows, workers, and contexts.
- Snapshot payload: establish a kilobyte budget at two and seven birds and fail schema growth in CI. The bootstrap must not wait on notebook, visits, settings panels, or noncritical species assets.
- Tick latency: p99 alarm at five seconds, plus queue-age and failed/retried tick alerts.

### Permitted production telemetry

- Request count, status class, route template, latency histograms, queue age, tick duration, tick retry/dead-letter count, cache hit/staleness, email delivery category, and deletion/export job health.
- Browser family/major version, coarse device performance class, first-bird timing, frame-time histogram, long-task count, memory-warning signal where available, audio-context error category, and caption fallback count.
- An anonymized, nonlinkable session-duration histogram with aggressive bucketing and no persistent client ID.

### Forbidden telemetry

- Account, aviary, bird, invitation, session-token, email/blind-index, IP, bird name, species, mood, personality, notebook, presence, listen-in, offer, settle, visit-recipient, or per-user history as metric dimensions or event payloads.
- Product dashboards for visit frequency, retention streaks, average drift, popular birds/species, offer conversion, notebook engagement, or cross-account behavior.
- Analytics database access to canonical tables or raw event topics.

Enforce the boundary with typed metric wrappers that allow only enumerated low-cardinality fields, CI scans for forbidden identifiers, telemetry payload tests, separate credentials/accounts, short logs, and privacy review for every new metric. Use synthetic aviaries and explicit research studies—not silent production aggregation—to tune bird behavior.

## 12. Verification strategy

### Engine and data

- Property tests: drift never decreases, traits remain bounded, no-input drift is zero, event retry is idempotent, a tick replay produces identical output, overlapping presence is unioned, visitor events cannot enter the owner log, and seven is a hard cap.
- Accelerated simulations: minutes to years across local times/DST, absence, high interaction, weather, multi-device, outage catch-up, and every calibration version. Review time-series and rendered weekly checkpoints without exposing those instruments in product code.
- Migration tests: copy realistic versioned fixtures, migrate, and prove stable bird IDs/vector values/notebook prose. Back up and restore canonical data in staging; treat vector loss as a severity-one failure.
- Contract/security tests: owner/visitor privilege matrix, snapshot personality-field denylist, token single-use/expiry/revocation, session revoke, CSRF, object authorization, export isolation, and deletion completeness.

### Experience

- Golden visual sequences for ordinary and reduced-motion modes at all day phases, weather states, 2–7 birds, narrow/wide viewports, settle/undo, return-from-hidden, and slow bootstrap.
- Audio deterministic tests for grammar expansion/caption agreement, node cleanup, mix ramps, clipping limits, chorus timing, unavailable WebAudio, and stable per-bird voice. Add human recognizability/uncanniness sessions before each bird-count ramp.
- Editorial tests lint forbidden announcement/gamification phrases and snapshot every notebook/narration/caption template. Product/design review generated samples for specificity and repetition.
- End-to-end multi-browser flows cover auth through adoption, two-device convergence, presence state transitions, offers/cooldowns, notebook sparsity via accelerated time, invite consume/revoke/expire, account export, and deletion recovery/hard deletion.
- Accessibility and performance gates run on every release candidate, with the manual matrix described above.

### Acceptance scenarios

Before public v1, demonstrate these scenarios to product, design, accessibility, privacy, and engineering owners:

1. A returning owner sees one personality-consistent, procedurally varied greeting within two seconds and no textual welcome.
2. Leaving a background tab open overnight adds no presence or drift; quietly watching with occasional activity does.
3. Two devices generate overlapping presence and simultaneous commands without lost/double drift; both converge after a snapshot.
4. Three accelerated weeks produce a noticeable but not session-by-session change; two weeks away produces no negative vector movement or distress.
5. A screen-reader-only user can perceive the aviary, identify/focus birds, listen in, offer, settle, and read the notebook without queue overload.
6. Reduced motion remains alive through still-pose cross-fades and color/audio, with flight/leaf/parallax motion absent.
7. Two birds chorus procedurally, listen-in ramps rather than cuts, captions match, and WebAudio failure yields captioned silence.
8. A visitor sees the exact current ambient state, cannot affect it, and loses access on the next pull after revocation.
9. The first bird and 30-minute frame/memory budgets pass on named target hardware/network.
10. A telemetry inspection cannot reconstruct a user's birds, interactions, or visits, and hard deletion removes all account-linked product data.

## 13. Work breakdown and sequencing

### Milestone 0 — contracts, prototypes, and calibration harness

- Ratify product invariants, owner/visitor authorization matrix, snapshot/event schemas, time-zone decision, retention schedule, and telemetry allowlist.
- Build pure engine interfaces, seeded simulation harness, presence state-machine prototype, Canvas first-bird spike, procedural call/caption spike, and reduced-motion/narration prototypes.
- Establish target hardware, shaped-network profiles, performance traces, accessibility manual-test environments, and audio recognizability research method.
- Exit when the team proves a mid-action first frame, two distinct procedural bird voices, valid caption derivation, honest presence intervals, and one-week/three-week calibration feasibility on synthetic runs.

### Milestone 1 — identity and canonical aviary

- Implement account UUID/email encryption and blind lookup, magic links, sessions/revocation, account time zone, PostgreSQL schema/migrations, two-bird adoption/naming, tick scheduler, event watermarking, canonical snapshots, and cache publication.
- Add vector backup/migration checks, tick idempotency/concurrency tests, and matter-of-fact auth/error surfaces.
- Exit when a new account can adopt/name two stable birds and see the same canonical snapshot on two devices after worker restarts and duplicate deliveries.

### Milestone 2 — behavioral core and ordinary scene

- Implement presence folding, drift, mood, perches, local-day/weather, greetings, bird-to-bird effects, call plans, Canvas state graph/interpolation, responsive layout, top-bar fade, listen-in, offers/cooldowns, settle/undo, and visibility recovery.
- Integrate WebAudio grammars, per-bird mix, chorus limits, and scene performance instrumentation.
- Exit when accelerated calibration, interaction invariants, audio identity, first-bird, frame, and memory gates pass with two birds.

### Milestone 3 — notebook, accessibility, and account lifecycle

- Implement sparse observation candidates and prose grammar, indefinite read-only notebook, semantic scene, narration scheduler, keyboard model, captions/fallback, reduced motion, contrast/focus treatments, email change, export, deletion/recovery, and unsupported-browser path.
- Complete cross-browser and assistive-technology matrices. Accessibility is not deferred behind a beta flag.
- Exit only when ordinary and accessible acceptance scenarios pass at parity.

### Milestone 4 — quiet visits and scale readiness

- Implement invite email/token, scoped visitor snapshot, visit session/log, immediate revocation/expiration, optional off-by-default email notification, and absence of visitor simulation inputs.
- Load-test ticks, snapshots, cache invalidation, email jobs, and deletion. Perform privacy threat model and telemetry separation audit.
- Exit when read-only authorization is mechanically enforced and revocation works during an active view.

### Milestone 5 — release hardening

- Run long soak, outage/catch-up, migration/restore, daylight-saving, seven-bird synthetic, browser, editorial repetition, security, accessibility, performance, and privacy tests.
- Freeze snapshot/grammar/calibration versions; create dashboards and alerts from the allowlist only; write rollback/runbooks for API, worker, schema, audio asset, and calibration releases.
- Product/design sign off that no announcement, gamification, caretaking, or social-network surface has leaked in.

## 14. Rollout and bird-count ramp

1. **Team dogfood:** synthetic and staff accounts, fixed two-bird cap. Validate continuity, privacy boundary, daily clock, call repetition, and system recovery. Staff data is still subject to production privacy rules.
2. **Invited accessibility/performance preview:** two birds, all accessible modes and account lifecycle enabled, visits initially disabled. Do not collect per-bird behavior analytics; use direct consented research feedback and operational metrics.
3. **Limited beta:** two birds, visits enabled for a small random account cohort, engine/calibration version pinned. Watch operational SLOs, support reports, audio fallback, accessibility regressions, and deletion correctness.
4. **General v1:** two starter birds for all new accounts. Raise rollout caps only after audio recognizability, layout, memory, snapshot, and accessibility gates pass at the next count.
5. **Age-gated growth:** configure initial eligibility around aviary age, not behavior: third near 90 days, fourth near 180 days, fifth near 300 days, sixth near 365 days, and seventh no earlier than 540 days. Treat these as calibratable product configuration, never a reward schedule. The worker creates a quiet adoption offer once both age eligibility and rollout cap allow it; waiting accounts lose nothing.

Use server kill switches for new offer creation, visit creation, optional notification email, a calibration version, and maximum enabled bird count. Kill switches must preserve existing birds and vectors; never solve an incident by rolling a bird back or regenerating it. Roll back client/render/audio assets independently from canonical data migrations.

Day-one dashboards cover first-bird latency, frame/memory lab results, snapshot/API/tick health, queue age, cache staleness, auth/email delivery, audio error categories, visit authorization failures, and erasure/export jobs. They deliberately do not answer how often a named or pseudonymous user visits, what they do, or how their birds change.

## 15. Principal risks and mitigations

| Risk | Failure signal | Prevention and response |
|---|---|---|
| Drift is too fast, too slow, or exploitable | One session visibly changes a bird; three weeks feel static; clicking dominates presence | Versioned pure engine, multi-week synthetic cohorts, saturation/cooldowns, instrument vs blinded-perception targets, staged calibration flag; never rewrite current vectors during rollback. |
| “Ambient after absence” accidentally becomes negative drift | Trait decreases, distressed visuals, guilt copy | Property-test non-negative deltas, keep recency in expression/mood only, editorial/non-goal review of every state and template. |
| Presence overcounts background/concurrent devices | Open tab or two devices accelerate change | Client conjunction state machine, short sequenced intervals, server clamping/timeout, cross-device interval union, adversarial lifecycle tests. |
| Tick retry or concurrent command loses/doubles personality | Vector discontinuity or divergent devices | Single server writer, row transaction, logical-tick uniqueness, event watermark/idempotency, outbox publication, restore/migration drills. |
| Numeric personality leaks | Values appear in snapshots, logs, ARIA, support or analytics | Separate internal/render schemas, response allowlists, serialization tests, log redaction, no simulation DB analytics access; isolated explicit export only. |
| Calls feel canned, uncanny, or indistinguishable | Users recognize repeats or cannot identify birds | Stable voice seeds, combinatorial reviewed grammars, deterministic variation, mix limits, blind recognizability/uncanniness research, count ramp gates. |
| Audio lifecycle degrades performance | Leaked nodes, clipping, context failures | Pools and bounded schedulers, 30-minute audio/memory soak, gain ceiling/compressor tests, captioned-silence fallback. |
| First load tells a loading story | Spinner/blank delay over 500 ms or all motion starts at zero | Minimal inline snapshot/assets, quiet field, server-timed mid-action phase, chunk budgets, shaped-network release gate. |
| Accessible modes become reduced substitutes | Static reduced-motion scene, robotic ARIA changes, caption mismatch | Shared presentation state, designed pose cross-fades, prose narration grammar/queue, caption from exact call event, manual AT sign-off in every release. |
| Visitor access expands or reshapes birds | Visitor can mutate, trigger greeting/presence, or retain revoked access | Separate scoped endpoint/session and response schema, no owner routes, status check each pull, authorization matrix tests, immediate generation revocation. |
| Email/PII spreads into infrastructure | Email appears in logs, keys, queue, metrics | Synthetic UUID everywhere, encrypted single account field, auth-only blind lookup, structured-log allowlist, provider review and automated PII scans. |
| Telemetry quietly becomes engagement analytics | Per-account dimensions or bird behavior in dashboards | Separate operational pipeline and credentials, typed metric allowlist, privacy review/CI payload tests, no warehouse access to simulation data. |
| Notebook/narration becomes repetitive or announces state | Generic event-log prose, trait/user-behavior wording | Sparse candidate gate, reviewed deterministic grammar, wording dedupe, exhaustive snapshot/editorial tests, stored immutable prose. |
| Time zone/device ambiguity creates inconsistent day | Devices fight over zone or visitor sees their own day | Account-scoped IANA zone seeded once and changed explicitly; host zone drives canonical day and visitor rendering; DST tests. |
| Bird-count growth breaks layout, audio, or access | Cropping, chorus blur, focus confusion, performance drop | Hard seven cap, per-count visual/audio/AT/perf qualification, server rollout cap independent of age, preserve existing birds on rollback. |
| Product accretes conventional engagement UI | Toasts, counters, notifications, public sharing appear | Architecture decision record, forbidden-copy lints, scope checklist in design/code review, product-owner sign-off before release. |

## 16. Open implementation decisions to resolve in Milestone 0

These decisions do not require changing product scope, but they must be recorded before schema/API freeze:

- Exact presence activity window and heartbeat cadence after quiet-watching tests; the plan starts at three minutes/30 seconds.
- Final mood enum and transition/dwell constants while retaining the specified examples and daily-ish behavior.
- Exact trait ranges/seed distributions and the visual/audio perceptual mapping that meets one-week/three-week targets.
- Species silhouettes, motif grammars, and the empirical same-species/per-bird recognizability threshold.
- Snapshot keepalive interval balancing revocation latency, freshness, and battery; visibility return always forces a pull.
- Detailed palette/focus tokens from the visual design system, with AA verified across every canonical scene state.
- Visit opt-in notification email wording/frequency. It remains account-level, off by default, absent from onboarding, with no push or badge.
- Final age-gate dates after long-horizon product review; they may move, but can never depend on visits, interaction volume, payment, rarity, or score.

Resolve each through a small prototype or explicit product/design/accessibility/privacy decision, add it to versioned configuration or contracts, and cover it with acceptance tests. None of these are permission to add a new feature or relax the invariants above.
