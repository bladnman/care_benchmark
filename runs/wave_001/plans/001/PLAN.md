# Pocket Aviary implementation plan

## 1. Product framing and v1 scope

Pocket Aviary v1 is a web-only, single-account, single-aviary product where two starter birds live in one horizontal browser scene and gradually evolve through the user's measured presence and a small set of quiet interactions. The implementation should treat "feels alive, not robotic" as the primary acceptance criterion: the aviary must appear already in progress, birds must behave with continuous mood-shaped motion, calls must vary procedurally, and the server simulation must advance while no client is connected.

V1 includes:

- Browser-only app for current major Chrome, Safari, Firefox, and Edge.
- Magic-link email sign-in, per-device sessions, session revocation, verified email changes, export, and deletion.
- One canonical aviary per account, starting with two birds and allowing up to seven over time.
- Server-authored simulation tick, persistent bird identity, persistent personality vectors, persistent mood, event-log-driven interaction processing, and no client ownership of canonical state.
- Single-scene responsive aviary with three perch zones, local-time day/night, rare ambient weather, ambient visual motion, top-bar controls, quiet loading field, and no UI chrome inside the scene.
- Return-greeting, listen-in, offer, settle, field notebook, precise presence accounting, and adoption/naming.
- Procedural WebAudio call synthesis, listen-in mix, call captions, and graceful silence with captions when WebAudio is unavailable.
- First-class accessibility: screen-reader narration, reduced-motion rendering, captions, keyboard navigation, focus treatment, and WCAG AA user-copy contrast.
- Optional read-only visit invitations: named email invite, one-time link, 30-day expiry, revocation, visit log, and off-by-default visit notifications.
- Aggregate-only operational telemetry, performance observability, and privacy boundaries that exclude per-bird and per-account interaction data from analytics.

V1 explicitly does not include native apps, passwords, SSO, payments, shared aviaries, household accounts, multiple aviaries per account, customizable scenes, bird catalogs, drag-to-place controls, public discovery, profiles, follows, comments, chat, co-presence, leaderboards, achievements, streaks, badges, scores, levels, push notifications, recorded-audio fallback, exposed personality numbers, hunger meters, death, distress, or any mechanic that punishes absence.

The plan below assumes a frontier team with strong web, simulation, audio, and accessibility capacity. It favors a simple service boundary, a carefully constrained data model, and heavy investment in calibration, test harnesses, and design QA because the product's hardest problems are subtle correctness problems rather than breadth of feature surface.

## 2. Target architecture

Use a three-part architecture:

1. Web client: a TypeScript browser app that renders the scene, synthesizes procedural calls, collects interaction and presence events, presents account/settings flows, and interpolates canonical snapshots. It never computes or writes personality drift.
2. API service: authenticated HTTP endpoints plus server-sent events or low-frequency snapshot polling for aviary state. It handles auth, accounts, visit permissions, event ingestion, exports, settings, and state reads.
3. Simulation worker: a server-side scheduled worker that runs roughly once per minute per active or recently active aviary partition, consumes append-only events in order, applies mood and drift deltas, advances ambient state, writes canonical snapshots, and emits notebook candidates.

Keep the simulation worker and API service in the same deployable initially if that reduces operational complexity, but preserve a clean module boundary:

- API code may append interaction events and read canonical snapshots.
- Simulation code is the only writer of personality vectors, mood transitions, bird perch intent, canonical call schedule, weather state, and notebook observations.
- Client code may interpolate and render between snapshots but cannot become an alternate simulation source.

The render pipeline boundary is important. The server owns canonical state that matters for continuity: bird IDs, species, names, personality vectors, mood, current perch zone, target perch zone, motion phase, ambient weather, settled state, notebook entries, account settings, invitation state, and recent interaction-event checkpoints. The client owns ephemeral presentation details: frame interpolation, exact pose curves between server states, leaf/feather ornament generation, audio synthesis execution, top-bar fade, local focus state, and local accessibility display preferences before they are persisted.

Recommended deployment shape:

- CDN/edge-served HTML, CSS, initial JS, compact species/motif assets, and an edge-cached bootstrap state envelope where possible.
- Regional API service close to the database; avoid embedding sensitive state at CDN unless it is delivered through authenticated edge logic.
- Relational database for accounts, birds, aviaries, events, snapshots, notebook, sessions, invites, and deletion/export jobs.
- Queue or database-backed lease table for simulation ticks. The tick must be idempotent and safe to retry.
- Object storage for generated exports, with short-lived signed download links emailed to verified addresses.
- Transactional email provider for magic links, invite links, export links, and account email verification.
- Aggregate-only metrics pipeline for operational and performance telemetry.

## 3. Domain model and persistence

Use a relational schema with synthetic UUIDs throughout. Store email exactly once on the account record, encrypted at rest; every foreign key, log field, queue key, and metric dimension uses the synthetic account ID or non-PII operational IDs.

Core tables:

- `accounts`: `id`, encrypted `email`, `email_verified_at`, `status`, `deletion_requested_at`, `created_at`, `updated_at`, settings flags, privacy-policy acknowledgement version if needed.
- `sessions`: `id`, `account_id`, hashed token, device label, browser family, created time, last seen, revoked time, expiry. Revocation must invalidate the token immediately.
- `magic_links`: `id`, `account_or_pending_email_ref`, hashed token, expires at 15 minutes, consumed time, request metadata for rate limits.
- `aviaries`: `id`, `account_id`, `created_at`, `settled_state`, local timezone, current weather state, current day/night phase cache, next eligible adoption time, bird cap.
- `birds`: `id`, `aviary_id`, stable `species_id`, user name, created time, active flag, current mood, mood entered at, current perch zone, target perch zone, motion phase, call signature seed, personality vector, drift accumulator state, last offer times by offer type, and version.
- `personality_snapshots`: optional periodic audit table for server-side calibration and rollback, keyed by bird and timestamp, not exposed to users or analytics.
- `interaction_events`: append-only log with `id`, `aviary_id`, optional `bird_id`, event type, client event time, server received time, idempotency key, payload, processed tick ID, and validation status.
- `presence_windows`: server-derived windows from presence pings, with start, end, confidence, source events, and processed status. This table exists to make drift processing honest and debuggable without treating raw pings as drift directly.
- `aviary_snapshots`: latest canonical state plus optional short history. Store compact JSON with version and server time, not per-frame animation data.
- `notebook_entries`: `id`, `aviary_id`, created time, prose text, source observation type, referenced bird IDs, rarity score, visible order.
- `visit_invites`: `id`, `aviary_id`, host account ID, encrypted visitor email, hashed link token, status, expires at, used time, revoked time, notification preference at issue if needed.
- `visit_sessions`: `id`, `invite_id`, visitor session token hash, first seen, last seen, approximate duration, revoked/expired termination reason.
- `exports`: `id`, `account_id`, requested time, completed time, status, object storage key, link expiry.
- `outbox_email`: transactional email queue for magic links, invites, exports, and verification.

Personality vector shape:

```json
{
  "boldness": 0.0,
  "social_warmth": 0.0,
  "vocal_frequency": 0.0,
  "plumage_saturation": 0.0,
  "curiosity": 0.0
}
```

Use normalized internal ranges, for example 0.0 to 1.0, but keep this strictly server-side and absent from every user API response, UI, accessible label, export prose, and account setting except account export JSON where the PRD explicitly includes current personality vectors. Even there, mark it as raw export data rather than product UI, and keep it out of normal surfaces.

Mood should be an enum with stable internal values such as `wary`, `content`, `curious`, `drowsy`, `alert`, and `settled`. Avoid expanding mood too early; the first release needs enough states to support visible variation, not a sprawling emotional taxonomy. Store mood transition metadata so the tick can avoid snapping.

Interaction-event types:

- `presence_ping`: client claims visibility, focus, and recent pointer/key activity; includes local clock, activity age, visibility state, focus state, and idempotency key.
- `presence_end`: sent on settle, hidden, blur, or best-effort unload; the server also closes stale windows by timeout.
- `listen_in_start` and `listen_in_end`: bird ID, local time, reason for end.
- `offer`: offer type, intended target context, nearby bird IDs from current snapshot version, idempotency key.
- `settle_start` and `settle_undo`: local time and snapshot version.
- `adoption_name_set` and `bird_renamed`: user naming events; these mutate user-facing bird name through API validation, not simulation drift.
- `accessibility_setting_changed` and `account_setting_changed`: settings events for audit and multi-device consistency, not simulation input.

Do not record visitor presence or visitor interactions into host simulation tables. Visitor logs should live in visit-specific tables and never feed the bird engine.

## 4. API surface

Use a small versioned JSON API. Responses that expose product state should be optimized for rendering and accessibility generation while hiding internal numerical personality values.

Authentication and account:

- `POST /api/auth/magic-link`: accepts email, rate-limited per email and IP. Always returns a neutral success response. Sends a 15-minute magic link.
- `POST /api/auth/magic-link/consume`: consumes a token once, creates or resumes an account, creates the starter aviary when needed, returns a per-device session.
- `GET /api/account`: returns email, settings, sessions, deletion state, notification preferences, and export/delete availability.
- `PATCH /api/account/settings`: updates system settings, accessibility preferences, visit notification preference, and timezone if user-controlled.
- `POST /api/account/email-change`: begins verified email change.
- `POST /api/account/sessions/{id}/revoke`: revokes a device session.
- `POST /api/account/export`: starts export job and emails link when ready.
- `POST /api/account/delete`: marks account for deletion.
- `POST /api/account/delete/cancel`: restores within the 30-day soft-delete window.

Aviary state:

- `GET /api/aviary/bootstrap`: returns latest canonical snapshot, compact species/motif manifest references, server time, snapshot version, account settings needed for rendering, and whether adoption naming is needed. This is the first state request and must be fast.
- `GET /api/aviary/snapshot?since_version=...`: returns the latest snapshot or a no-change envelope. Use on visibility return, long frame gaps, and low-frequency visible keepalive.
- `POST /api/aviary/events`: appends batched interaction events. Requires idempotency keys. Returns accepted/rejected per event and current authoritative snapshot version.
- `GET /api/aviary/notebook?cursor=...`: paginates notebook entries.
- `PATCH /api/birds/{bird_id}/name`: renames a bird after validating account ownership.

Offer and adoption:

- Offers can be submitted through the generic event endpoint, but expose `GET /api/aviary/offers` if the available offer library becomes dynamic. In v1 the offer set is seed, song fragment, and still pool.
- `GET /api/aviary/adoption-availability`: returns whether an age-based new bird can arrive, without gamified framing.
- `POST /api/aviary/adoptions`: accepts names for a server-selected bird or initial two-bird adoption. Do not expose a species catalog or rarity list.

Visits:

- `POST /api/visits/invites`: host enters visitor email; creates one-time invite, expires after 30 days, emails link.
- `GET /api/visits/invites`: host account-settings view of outstanding/current invitations and visit log.
- `POST /api/visits/invites/{id}/revoke`: revokes immediately.
- `POST /api/visits/consume`: visitor consumes a one-time link and receives a visitor session scoped to the host aviary.
- `GET /api/visit/aviary/bootstrap` and `GET /api/visit/aviary/snapshot`: read-only visitor state. No event endpoint except operational heartbeat for visit duration logging; never write host interaction events.

Error tone:

- Product surfaces use naturalist prose; system endpoints return matter-of-fact messages for sign-in, sync, unsupported browser, account, visit expiration, and accessibility settings.
- Never return "welcome back," streak language, score-like counts, or personality labels in API messages intended for UI display.

## 5. Server-side simulation engine

The simulation worker is the product's behavioral core. Build it as deterministic, testable modules with explicit seeds:

- Event ingestion normalizer.
- Presence-window builder.
- Drift calculator.
- Mood transition engine.
- Perch and idle-intent planner.
- Call schedule planner.
- Weather/day-night updater.
- Notebook observation selector and prose generator.
- Snapshot writer.

Tick cadence and ownership:

- Run approximately once per minute per aviary. Use a lease to ensure only one worker processes an aviary at a time.
- Read unprocessed events in server-received order, with client timestamps used only for duration reconstruction inside trusted bounds.
- Derive presence windows server-side from valid presence pings. A presence ping counts only if the client reports visible document, focused window, and recent pointer/key activity within the calibrated window.
- Close presence windows when a presence-end arrives, when visibility/focus/activity no longer qualifies, or when pings expire.
- Apply all personality deltas as server-authored additive deltas. Store the processed event checkpoint transactionally with vector updates and snapshot write.
- Make tick idempotent: reprocessing a crashed tick must not double-apply drift.

Drift design:

- Treat presence-time as the dominant input. Aggregate qualifying presence minutes per bird exposure window; all birds in the aviary receive some baseline expressive drift from host presence, with bird-specific weighting for visible/near/front birds if desired.
- Listen-in adds stronger bird-specific drift toward social warmth and vocal frequency, scaled by listen duration and capped per day.
- Offers add small drift toward curiosity when investigated/accepted and small drift toward boldness when an offer appears near a bird, but apply per-bird cooldowns so a single session cannot saturate curiosity.
- Settle quiets mood and ends presence cleanly; it should not produce material personality drift beyond any already accrued presence.
- Neglect never applies negative drift. It may lower near-term greeting probability through lack of recent positive inputs, but it cannot reduce personality vector values or make birds visibly distressed.
- Use low-pass filters and daily caps so instrument-visible drift appears after about one week of regular use and user-visible drift appears after about three weeks.

Calibration plan:

- Create simulation fixtures for representative users: light weekly visitor, regular short daily visitor, long passive watcher, offer-heavy user, listen-in-heavy user, absent-returning user.
- Run accelerated multi-week simulations and inspect vector movement, mood distributions, greeting frequency, perch occupancy, and call frequency.
- Establish guardrails: no single session moves any trait visibly; three weeks of regular presence changes greeting/perch/call tendencies enough for qualitative recognition; two weeks of absence does not create distress or negative visual states.
- Keep calibration instruments internal. Do not build product dashboards that expose trait values to users.

Mood transitions:

- Mood resets on a daily-ish cadence, not on open. It should be modulated by local time, current weather, recent interactions, bird personality, and bird-to-bird signals.
- Persist mood across sessions. If a bird ended drowsy last night, the morning state is computed through ticks over time, not defaulted.
- Avoid abrupt changes in snapshots. Include transition metadata so the client crossfades or animates toward the new posture.
- Model bird-to-bird effects: alarm-like call can nudge nearby birds toward wary; chorus can emerge from high vocal-frequency birds; content/warm birds may perch nearer one another.

Return-greeting:

- On bootstrap after absence or visibility return, select at most one primary greeter within the first one or two seconds. Use absence length, boldness, mood, social warmth, and recent greeting history.
- If secondary greetings occur, stagger them by randomized offsets. Never trigger all birds in unison.
- Represent greeting as state in the snapshot: greeter bird ID, gesture type, call motif hint, start offset, and variation seed. The client renders it procedurally.
- No textual welcome is generated by the engine.

Call grammar:

- Each species owns motif libraries: pitch contour primitives, rhythm cells, timbral parameters, and allowed variation ranges.
- Each bird has a stable call seed and personality-shaped parameters. Vocal frequency affects how often it calls and joins chorus; mood affects tempo, amplitude envelope, and contour choice.
- Server snapshots should schedule call intents and motif parameters, not audio buffers. The browser synthesizes the actual sound via WebAudio.
- Recognizability acceptance: a bird's call remains identifiable across mood and drift. Seven birds remain separable in the mix.

Notebook generation:

- Generate rare observations, not logs. Rough target: about one entry every few days for a regularly visited aviary, with extra entries only for genuinely noteworthy moments.
- Candidate triggers: first greeter changed, unusual quiet stretch, weather affecting posture, bird investigating an offer, aging into a new bird arrival, long absence return with calm greeting, distinctive chorus.
- Filter out user-behavior observations such as visit streaks, total time, or "you returned after X days." Notebook prose should describe the aviary, not score the user.
- Store generated prose so old entries remain stable. Use naturalist voice: lowercase, present tense, specific, no exclamation, no gamified language, no exposed numbers.

## 6. Client rendering pipeline

Build the client as a state-snapshot renderer with procedural presentation layers.

Initial load:

- Serve a minimal shell capable of drawing the quiet field and first birds quickly.
- Request `bootstrap` immediately, or embed a short-lived authenticated bootstrap envelope if edge architecture permits.
- Draw a quiet field while waiting only if the snapshot is not available. Avoid spinner, progress bar, skeleton card, or machine-like loading UI.
- The first bird frame must appear as an in-progress state: mid-preen, mid-call, settled posture, or another plausible ongoing pose. No wake-up animation after normal load.

Scene:

- One horizontal responsive scene. No panning, scrolling, or zooming.
- Maintain aspect ratio and responsive constraints so no bird is cropped on narrow phone or wide desktop.
- Three perch zones map server perch intent to visual depth, scale, and parallax: front, middle, back.
- The user cannot drag birds, assign perches, or otherwise control layout.
- Day/night color is based on user's local timezone and server snapshot phase; client interpolates gradual palette shifts.
- Weather and ambient events should be rare and subdued. Use server weather state for rain/wind, but generate non-stateful leaf/feather drift client-side.

Bird animation:

- Use a pose/state-machine per species with mood-shaped idle clips or procedural pose curves: preen, scan, head tilt, body shuffle, perch shift, call posture, drowsy settle, cautious retreat, curious approach.
- Blend from current pose to server target state smoothly when snapshots update.
- Keep idle motion continuous while visible, but pause rendering when hidden. On visibility return, fetch a fresh snapshot and resume from canonical state.
- Support reduced-motion by swapping animation paths for slow cross-fades between still poses and removing ambient leaf drift.

Top bar:

- Thin top bar above the scene with only account/settings, accessibility settings, field notebook, and offer affordance.
- Fade nearly transparent after cursor stillness; return on pointer movement or keyboard activity.
- Keyboard focus should reveal the bar and show clear focus indicators.
- Do not place buttons, labels, badges, hover tooltips, or status icons inside the aviary scene.

Interactions:

- Return-greeting is rendered from snapshot greeting state. No UI copy accompanies it.
- Listen-in starts from click, tap, or keyboard focus/Enter on a bird. It changes local audio mix immediately and posts events to the server. Disengage on same bird, another bird, empty scene, or focus exit.
- Offer opens from top bar, not by clicking a bird. Submit a single offer event with idempotency key; render response from subsequent snapshot or optimistic short-lived ambient placement only if it cannot contradict server state.
- Settle starts from top bar. Apply slow evening lighting and quieting locally, post settle event, and allow five-second undo by any click in the aviary. Undo posts `settle_undo`.
- Field notebook opens from top bar as a calm reading surface outside the scene. It is read-only, paginated, and not a feed with badges.

Do not add instructional in-app prose explaining the product's charm. The UI should provide affordances, labels where needed for accessibility, and matter-of-fact system settings, but the aviary itself should carry the experience through behavior.

## 7. Audio pipeline

Use WebAudio for all bird calls. There is no recorded-loop path and no recorded fallback.

Audio architecture:

- A global audio manager owns an `AudioContext`, master gain, per-bird gain nodes, ambient bus, listen-in bus, compressor/limiter, and caption event bridge.
- Per-bird synthesizers generate calls from motif data: oscillators, noise sources, filters, envelopes, pitch curves, tremolo/trill modulation, and subtle random variation seeded by bird identity and call instance.
- Server call intents provide timing windows and motif choices; client schedules synthesis with jitter within allowed ranges so calls do not sound grid-like.
- The chorus mixer allows overlapping calls without phase artifacts. Use per-bird timbral spacing and gain limits to maintain recognizability.

Listen-in:

- On listen-in, ramp focused bird gain upward slowly and other birds downward to ambient, never silent.
- On disengage, ramp back to ambient with matching duration.
- The ramp should feel like attention, not channel switching. Avoid abrupt cuts.

Captions:

- Generate caption text from the actual call grammar event, not a static species string.
- Examples: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch".
- Captions appear near the calling bird, meet contrast rules, fade in/out with the call, and are available by setting or as fallback when WebAudio is unavailable.

Audio permissions and fallback:

- If the browser requires a user gesture before audio, the visual aviary still loads alive; enable audio on first permissible gesture without a modal. If needed, expose a quiet matter-of-fact audio setting, not a promotional prompt.
- If WebAudio is unavailable or denied, play in silence with captions on by default.
- Do not download recorded audio as a fallback.

Performance:

- Reuse buffers, envelopes, and node pools where possible. Avoid per-call allocations that accumulate.
- Instrument audio-context errors and aggregate counts only.
- Include a 30-minute automated memory test with calls and listen-ins active.

## 8. Accessibility plan

Accessibility ships in v1 as part of the core product.

Screen-reader narration:

- Provide a live region with low-frequency naturalist narration generated from the same canonical state as the visual scene.
- Idle cadence: roughly every 30 to 60 seconds. User-initiated events such as return-greeting, offer response, and settle get prompt but still restrained narration.
- Narration is prose, not a state list. Do not expose perch indices, mood labels, or trait numbers.
- Maintain a queue with debouncing so the screen reader is not flooded.

Keyboard:

- Tab order: top bar controls, then aviary bird focus group, then any open panel.
- Arrow keys move between birds inside the aviary focus group.
- Enter toggles listen-in for focused bird.
- Escape exits listen-in or closes panels in predictable order.
- Offer menus, account settings, accessibility settings, and notebook are fully keyboard-operable.
- Focus indicators are visible against all day/night/weather palettes and do not look like game targeting UI.

Reduced motion:

- Respect `prefers-reduced-motion` by default and allow override in accessibility settings.
- Replace micro-animation with slow cross-fades between still poses.
- Replace flight/perch motion with cross-fades or very slow transitions.
- Remove leaf/feather drift. Keep slow color transitions.
- Preserve calls, captions, drift, mood, notebook, and interaction semantics.

Captions and visual text:

- Call captions are optional unless audio fallback requires them.
- Captions use naturalist voice and match actual call events.
- All user-copy text passes WCAG AA contrast; account, error, sync, and settings surfaces use matter-of-fact tone.

Testing:

- Include automated accessibility checks for focus order, ARIA roles, contrast, reduced-motion activation, and keyboard interaction.
- Include manual screen-reader QA for narration cadence and prose quality.
- Include QA scenarios with audio disabled, WebAudio unavailable, reduced motion enabled, narrow phone viewport, and night palette.

## 9. Sync and conflict model

The system has one canonical aviary state per account. Clients read snapshots and append events; they never merge state.

State propagation:

- On navigation or visibility return, client requests a snapshot.
- While visible, client polls at low frequency or uses a lightweight stream if the product needs faster updates. Keep traffic modest because tick cadence is slow.
- On long render-frame gaps or resume from suspended laptop, client discards local interpolation assumptions and fetches a snapshot.
- Multi-device sessions each send events with idempotency keys. The server appends them in received order, validates them against account ownership and snapshot version, and the tick processes them in order.

Conflict prevention:

- No last-write-wins for personality, mood, or canonical bird state.
- Client updates to user-controlled settings and names can use optimistic concurrency with version checks; conflicts get matter-of-fact surfaces.
- Offer cooldowns are enforced server-side per bird. If two devices submit offers, the tick accepts/rejects according to server order and cooldown state.
- Listen-in events from multiple devices should affect only the host's simulation as attention signals, but the audio focus is local per client. Do not sync "currently listening in" as a global UI state across devices.
- Settle is account-level session intent only if the active client performs it; another device opening later reads whatever canonical settled/mood state the tick computed. Closing without settle is equivalent for presence.

Presence correctness:

- The client sends pings only when visible, focused, and recently active.
- The server validates ping cadence, closes stale windows, and caps credit for suspicious or duplicate events.
- A hidden tab, background window, or unattended open laptop must not accrue presence-time.
- Activity timeout should lean long enough to allow quiet watching, then be calibrated through internal testing.

Visit sync:

- Visitors receive read-only snapshots of the host's current canonical aviary.
- Visitor snapshot pulls check invite validity each time or at a short cache TTL. Revocation takes effect on next pull.
- Visitor sessions write visit-duration metadata only. They do not generate greetings, presence, offers, listen-in, settle, notebook, or drift events.

## 10. Privacy, telemetry, and observability

Privacy boundary:

- Per-bird interaction events, personality vectors, moods, notebook sources, and account simulation history exist only to drive that user's aviary.
- Do not send per-bird or per-account interaction data to analytics, model training, recommendation systems, or third parties.
- The simulation database is not a source for the analytics warehouse.
- Use synthetic account IDs in operational logs only where account-scoped debugging requires it; prefer request IDs and aggregate counters.
- Email is encrypted on the account record and never used as a partition key, metric dimension, log field, or service identifier.

Allowed aggregate telemetry:

- API request counts, status codes, endpoint latencies.
- Magic-link send/consume success rates without email dimensions.
- Simulation tick latency, queue depth, tick failures, retry counts.
- Snapshot payload size and cache hit/miss.
- Client page load timings, first-bird-render timing, render-frame timing, memory-growth test results, WebAudio error counts.
- Anonymous session-duration histograms with no account dimension.
- Export job success/failure counts and deletion job counts.

Explicitly disallowed telemetry:

- Average drift by trait across accounts.
- Per-species interaction rates.
- Offer acceptance by bird personality.
- User streaks, visit counts per account, leaderboards, most-visited aviaries.
- Any data product that could reconstruct a user's relationship with their birds.

Operational observability:

- Alarm if simulation tick p99 exceeds 5 seconds.
- Alarm on tick backlog, failed event processing, snapshot staleness, magic-link delivery failures, export failures, and hard-delete job failures.
- Synthetic browsers should measure first bird visible under mid-tier mobile over 4G profile, idle FPS, WebAudio startup, reduced-motion rendering, and 30-minute memory growth.

Logs:

- Keep structured logs with request ID, synthetic account ID only where necessary, endpoint, status, latency, and error category.
- Scrub emails, invite tokens, magic-link tokens, notebook prose, event payload details, and raw personality vectors from logs.

## 11. Performance plan

Budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on a mid-tier mobile device over 4G.
- 60fps idle motion on a five-year-old mid-range laptop for a 30-minute session.
- No client memory growth over 30 minutes.
- Snapshot payloads in kilobytes, not megabytes.

Implementation tactics:

- Code-split account settings, accessibility settings, notebook history, visit management, export/delete flows, and admin-only tooling out of the initial bundle.
- Keep the initial render path limited to quiet field, two bird renderers, essential state fetch, motif manifest references, and top-bar shell.
- Use compact vector or sprite assets for birds; avoid large bitmaps. Procedural visual variation should be data-driven.
- Lazy-load full notebook and account panels only on demand.
- Use CSS transforms/canvas/WebGL/SVG based on prototype profiling; choose the renderer that meets the 60fps and memory budgets with the least complexity. The product does not require 3D.
- Reuse object pools for particle-like leaf/feather drift and audio nodes.
- Pause rendering when hidden; resume from server snapshot.
- Keep captions and narration generation bounded and debounced.

Testing gates:

- Bundle budget CI check.
- Lighthouse/WebPageTest synthetic profile for time to first bird.
- Playwright performance trace for desktop and mobile viewports.
- 30-minute soak test with idle motion, calls, captions, listen-in toggles, notebook open/close, and reduced-motion mode.
- Memory snapshots before/after soak with failure threshold.

## 12. Rollout plan

Phase 0: Foundations and prototypes

- Build simulation model in isolation with fixtures and accelerated time.
- Prototype bird rendering, reduced-motion rendering, and WebAudio motif synthesis.
- Validate that two to seven birds can remain visually and audibly recognizable.
- Define species pool, motif grammar, pose vocabulary, and naturalist prose templates with design/writing input.
- Establish privacy and telemetry schema review before product data exists.

Phase 1: Account, canonical state, and initial aviary

- Implement magic-link sign-in, account creation, session tokens, synthetic IDs, starter aviary creation, two starter birds, naming, bootstrap snapshot, and basic scene rendering.
- Ship server-side tick with mood persistence, day/night, simple perch intent, and presence event ingestion.
- Add no-spinner quiet loading field and first-frame in-progress rendering.
- Gate with internal accounts only.

Phase 2: Core interactions and drift

- Add return-greeting, listen-in, offers, settle/undo, per-bird cooldowns, presence-window builder, personality drift, mood transitions, and bird-to-bird call response.
- Add field notebook candidate generation and sparse entry policy.
- Run accelerated multi-week calibration before opening outside the team.

Phase 3: Accessibility and performance hardening

- Complete screen-reader narration, captions, keyboard model, reduced-motion mode, focus treatment, contrast, WebAudio fallback, and unsupported-browser surface.
- Enforce bundle, first-bird, FPS, and memory budgets in CI.
- Conduct manual screen-reader and reduced-motion QA.

Phase 4: Sync, privacy, export/delete, and visits

- Harden multi-device event ordering, idempotency, stale snapshot handling, session revocation, email change, account export, soft/hard deletion.
- Add visit invitations, visitor read-only bootstrap/snapshot, visit log, revocation, expiration, and off-by-default visit notifications.
- Verify visitor activity does not touch host simulation events.

Phase 5: Private beta and bird-cap ramp

- Start with two birds only for all accounts.
- Enable third-bird age-based arrival for older internal/beta aviaries after drift and audio recognizability pass.
- Ramp toward five to seven birds only after performance, recognizability, and chorus-mix checks remain healthy.
- Instrument aggregate-only operational metrics from day one. Do not add engagement dashboards.

Launch criteria:

- First bird under 500ms in synthetic target profile.
- 60fps idle and no memory growth over 30 minutes in CI.
- Simulation tick p99 below 5 seconds and no stale snapshot backlog.
- Presence accounting passes hidden-tab, blurred-window, inactive-user, and quiet-watcher tests.
- Drift calibration meets one-week instrument and three-week user-visible targets in accelerated fixtures.
- Accessibility flows pass manual QA, not just automated checks.
- Privacy review confirms analytics cannot read simulation data.
- Visit revocation works within one snapshot pull.

## 13. Engineering workstreams

Simulation and data:

- Schema migrations, event log, tick lease, tick idempotency, snapshot writer, drift/mood modules, notebook generator, calibration fixtures, simulation tests.

Frontend scene:

- Responsive scene renderer, bird pose system, interpolation, top bar, quiet loading field, adoption/naming, notebook panel, account/settings panels, reduced-motion renderer.

Audio:

- Motif library format, WebAudio synthesizers, chorus mixer, listen-in ramps, call captions from grammar, audio fallback, memory/performance tests.

Accessibility:

- Narration generator, keyboard focus model, captions, contrast validation, reduced-motion QA, screen-reader QA.

Auth/account/privacy:

- Magic links, sessions, email changes, export, deletion, account settings, encrypted email handling, log scrubbing, privacy policy surface.

Visits:

- Invite issuance, one-time visitor sessions, read-only snapshot API, revocation, expiration, visit log, optional notification toggle.

Observability/release:

- Aggregate metrics, synthetic performance checks, alarms, CI budgets, beta rollout controls, runbooks.

Each workstream should maintain product-language guardrails in acceptance criteria: no announcement toasts, no gamified counters, no exposed traits, no distress states, no social-network creep.

## 14. Test strategy

Unit tests:

- Drift monotonicity, caps, low-pass timing, and no negative drift on absence.
- Mood transitions from time of day, weather, interactions, and personality.
- Presence-window construction from visibility/focus/activity combinations.
- Event idempotency and ordered processing.
- Offer cooldown enforcement.
- Magic-link expiry/consumption and session revocation.
- Visit invite expiry/revocation and visitor permission checks.
- Notebook sparsity and prohibited language filters.

Integration tests:

- Multi-device sessions append events without last-write-wins state loss.
- Laptop listen-in plus phone offer processes in server order and preserves drift.
- Hidden tab does not accrue presence; visible/focused/recently active tab does.
- Tab close and settle both end presence without penalty.
- Visitor can watch but cannot create interaction events or greetings.
- Export contains allowed account/aviary data and no unrelated telemetry.
- Soft deletion blocks normal use, restore works inside 30 days, hard deletion removes dependent records.

End-to-end tests:

- New account magic-link sign-in, starter birds, naming, first aviary render.
- Return after absence produces one varied bird greeting and no textual welcome.
- Listen-in ramps audio and disengages through all specified routes.
- Offer seed/song/pool produces mood/personality-shaped responses and respects cooldown.
- Settle lighting shifts and five-second undo works.
- Notebook remains read-only and sparse.
- Keyboard-only user can operate every v1 interaction.
- Reduced-motion user gets cross-fade aviary, not static fallback.
- WebAudio unavailable produces captions-on graceful silence.
- Visit invite, consume, watch, revoke, and expired-link flows.

Simulation calibration tests:

- Accelerated one-week regular visits produce measurable internal drift.
- Accelerated three-week regular visits produce qualitative greeting/perch/call changes.
- Two-week absence produces quieter/ambient presentation but no distress or negative trait movement.
- Seven-bird chorus remains within recognizability and performance bounds.

Content and tone checks:

- Naturalist surfaces are lowercase, present-tense, specific, and free of "you" announcement framing where prohibited.
- System surfaces are clear, capitalized, matter-of-fact, and actionable.
- No surface uses welcome-back, streak, achievement, score, level, badge, rank, hunger, death, distress, public discovery, or social comparison language.

## 15. Key risks and mitigations

Drift calibration risk:

- Risk: drift changes too fast and feels like stat manipulation, or too slowly and feels irrelevant.
- Mitigation: accelerated fixtures, daily caps, low-pass filters, internal instruments, and beta cohorts held at two birds until calibration is stable.

Presence correctness risk:

- Risk: background tabs or unattended laptops inflate presence, corrupting drift.
- Mitigation: strict visible/focused/recent-activity conjunction, server-side window derivation, stale-window closure, hidden-tab tests, and conservative caps.

Sync correctness risk:

- Risk: multi-device writes lose drift or create conflicting aviary states.
- Mitigation: append-only event log, server-only personality writes, idempotency keys, tick leases, transactional checkpoints, and no last-write-wins on canonical state.

Audio uncanniness risk:

- Risk: procedural calls sound synthetic, repetitive, or blur into noise at higher bird counts.
- Mitigation: motif libraries per species, stable bird call seeds, mood/personality variation, chorus mix QA, recognizability tests, and gradual bird-count ramp.

Accessibility regression risk:

- Risk: accessible paths become state lists or static fallbacks, losing the product.
- Mitigation: design accessibility as a first-class surface, manual screen-reader QA, reduced-motion art direction, caption grammar, and v1 launch gate.

Performance risk:

- Risk: first bird misses 500ms or long sessions leak memory.
- Mitigation: strict bundle budget, code splitting, compact assets, node/object reuse, synthetic checks, and 30-minute soak CI.

Tone/product-creep risk:

- Risk: harmless-seeming toasts, counters, notifications, or social loops slip in.
- Mitigation: explicit UI copy review, prohibited-language tests, design review checklist, and architecture that does not compute gamified/social aggregate metrics.

Privacy risk:

- Risk: emails or bird interaction data leak into logs or analytics.
- Mitigation: synthetic IDs, encrypted email single-source rule, log scrubbers, analytics schema allowlist, and no simulation database reads by telemetry pipelines.

Notebook quality risk:

- Risk: entries become generic event logs or appear too often.
- Mitigation: sparse observation policy, source-trigger scoring, prose templates with review, prohibited language checks, and stored immutable entries.

Visit boundary risk:

- Risk: visitor sessions accidentally influence host birds or become social-network footholds.
- Mitigation: separate visitor session tables, no visitor event ingestion into simulation, read-only APIs, no co-presence markers, no public discovery routes, and off-by-default notifications.

## 16. Acceptance checklist

- The first normal session shows birds already in motion, not an app waking up.
- A returning user is noticed by one bird through behavior, never by a welcome toast.
- Presence requires visible document, focused window, and recent pointer/key activity.
- Personality vectors are persistent, server-owned, additive, hidden from product UI, and monotonic toward expressive.
- Mood persists across sessions and changes through server ticks.
- Calls are procedural WebAudio, recognizable per bird, and captioned from actual grammar.
- Listen-in rebalances the mix and never silences the rest of the aviary.
- Offers are gestures with cooldowns, not repeatable stat buttons.
- Settle is optional, undoable for five seconds, and equivalent to tab close at the engine level.
- The notebook is sparse, read-only, naturalist, and never a user-behavior log.
- The scene is one horizontal view with no panning, no scrolling, no bird placement controls, and no chrome inside the aviary.
- Accessibility surfaces carry the actual aviary experience.
- The app meets bundle, first-bird, 60fps, and memory budgets.
- Visits are named, opt-in, revocable, read-only, and unable to affect host drift.
- Aggregate telemetry excludes per-bird and per-account relationship data.
- No v1 surface contains gamification, Tamagotchi mechanics, social-network mechanics, push/ping behavior, or native-app scope.
