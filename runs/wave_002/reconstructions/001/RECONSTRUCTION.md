## System-level intent

- **The aviary should feel alive before the product explains itself.** This shows up in the opening scope: the scene is "already alive when opened," birds are "mid-motion," and the user may "quietly watch." The frontend repeats this through "No spinner, no fade-from-static, no wake-up animation, no 'ready' transition."

- **The emotional contract is notice, not notification.** The plan states this directly: "the user is noticed by birds, not announced to by software." Greeting behavior follows the same rule: "Never show a textual welcome, absence count, or 'you've been gone' message."

- **Absence must not be punished or converted into engagement pressure.** The scope says "absence is never punished." The drift section grounds that mechanically: "no negative trait movement," and absence may only reduce "current expressiveness through mood and lower greeting frequency."

- **System surfaces are clear; product surfaces are naturalist and restrained.** The scope says "system surfaces are clear and matter-of-fact" while "product surfaces are naturalist and restrained." This reappears in account/error responses using "matter-of-fact voice," notebook entries using "naturalist lowercase prose," and screen-reader narration using "naturalist lowercase present-tense voice."

- **The server, not the browser, owns canonical life.** The architecture says the "simulation worker is the only writer of personality vectors and mood progression," the client "never writes personality state," and there is "no client-to-client sync and no last-write-wins path for personality or mood."

- **Behavior should emerge slowly from valid presence and host interaction, not from scores, payments, or mechanical optimization.** Growth is "based on aviary age, not engagement or payment." Drift has "Dominant weight: presence time," "per-day or per-week maximum deltas," and targets "measurable internal drift" after a week but "visible behavioral difference" after about three weeks.

- **Hidden internal state should shape expression without becoming a visible stats game.** Personality vectors are "never returned numerically to user-facing clients," mood labels must not appear as "status chips," exports label vectors as "raw JSON," and guardrail tests ban "numeric trait exposure."

- **Privacy boundaries are product architecture, not an analytics afterthought.** The scope establishes "Aggregate-only operational telemetry" and a "hard boundary" against per-account/per-bird interaction state in analytics, ML, recommendations, or dashboards. Observability repeats that telemetry must not reconstruct "a user's relationship with their aviary."

- **Visitors are allowed only as quiet, read-only viewing, not social presence.** Visits are "optional read-only," "off by default," "revocable," "expiring after 30 days," and "explicitly not co-presence." The visit risk mitigation is "API absence" for comments, chat, listings, co-presence, profiles, and visitor interaction events.

- **Accessibility is part of the core aviary, not a secondary fallback.** The plan says "Accessibility is part of the core product, not a later fallback" and includes screen-reader narration, reduced-motion rendering, captions, keyboard navigation, focus, contrast, and unsupported-browser surfaces in V1.

- **Audio should be procedural, recognizable, and bounded.** The call grammar uses species motifs plus stable bird seeds, and the audio pipeline says "Use WebAudio for all calls" and "Never ship recorded calls." Recognizability is preserved by anchoring "timbre/motif identity" while allowing variation.

- **Performance constraints protect the quiet richness of the aviary.** The plan sets budgets for "Initial JS bundle under 2MB gzipped," "First bird visible within 500ms," "60fps idle motion," no memory growth, and tick p99 alarms, then ties implementation choices to hitting those budgets.

- **Open decisions are calibration details, not invitations to expand scope.** The final section says decisions are "implementation-calibration items, not product-scope openings" and explicitly cannot add gamification, native clients, public social surfaces, punishment for absence, numeric personality displays, or recorded audio.

## Per-feature whys

### Product Boundary and V1 Scope

- **Web-only, single-user virtual aviary with one canonical aviary per account**: NOT RECOVERABLE FROM PLAN

- **Two starter birds, future birds unlocked by aviary age, and a hard cap of seven**: The plan ties growth to aviary age "not engagement or payment," preserving the non-gamified relationship. The rollout also says to "Ramp birds-per-aviary carefully" and monitor support confusion, audio recognizability, CPU, and tick cost, while keeping the "hard cap" at seven.

- **Single horizontal browser scene already alive on open**: The rationale is the quiet, immediate experience: birds are "mid-motion," ambient sound may be present, one bird notices the returning user "without any textual welcome," and the user may "quietly watch."

- **Account lifecycle controls: session revocation, verified email change, export, soft deletion, hard deletion**: These support the data lifecycle and privacy model. Soft deletion "immediately disables normal use," recovery is allowed during the 30-day window, and hard deletion removes birds, vectors, notebook entries, events, sessions, invites, telemetry linkages where applicable, and export artifacts.

- **Email magic-link sign-in**: NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick**: Its why is canonical ownership. The tick "owns canonical aviary state" and is the only path for personality drift, mood transitions, notebook generation, weather, bird positions, and event consumption.

- **Browser client renders snapshots, interpolates motion, synthesizes procedural calls, and collects events**: The client exists to render and collect input while avoiding canonical writes. The plan says the browser "never writes personality state" and client ornaments "must never affect state or telemetry."

- **System-chosen starter birds from a small coherent species pool**: NOT RECOVERABLE FROM PLAN

- **User-assigned bird names**: NOT RECOVERABLE FROM PLAN

- **Stable bird identities through renames or species changes**: The plan gives the reason directly in data modeling: a bird `id` is "stable" and "never replaced by renames or species changes."

- **Hidden per-bird personality vectors, fast moods, cooldowns, interactions, and monotonic drift**: These let birds become more expressive without exposing numeric traits or creating a stats surface. The plan repeatedly says personality values are hidden, drift is slow and monotonic, and mood drives behavior rather than UI labels.

- **Field notebook entries**: The notebook is meant to be "sparse, read-only, naturalist, specific," generated from "noteworthy aviary moments rather than user-engagement facts." It must not become a streak, score, visit-frequency, or event-log surface.

- **Optional read-only visit invitations**: Visits are designed for controlled viewing without social mechanics: they are off by default, revocable, expire after 30 days, and are "explicitly not co-presence."

- **First-class accessibility surfaces**: The plan names accessibility as a V1 feature and later says it is "part of the core product, not a later fallback," so screen-reader narration, reduced motion, captions, keyboard support, focus, contrast, and unsupported-browser guidance are not optional polish.

- **Aggregate-only operational telemetry**: The rationale is the privacy boundary: per-account and per-bird interaction state is not used for analytics, ML, recommendations, or population dashboards.

### System Architecture

- **Three-part browser/API/simulation-worker architecture**: The plan uses this split to keep rendering and UI on the client, identity and authorization in the API, and canonical derived state in the worker. It states that the "simulation worker owns all derived state that must be canonical across devices."

- **Relational database as authoritative canonical state**: Row-level transactional guarantees are called for because accounts, aviaries, birds, vectors, moods, notebook entries, invites, sessions, and event logs need canonical consistency. Snapshot documents may be cached, but "database state" remains authoritative.

- **Snapshot-based render pipeline**: Snapshots keep the boundary compact and controlled: the server sends identity, mood, perch targets, call hints, weather, offer reactions, and deltas, while the client interpolates and adds only non-canonical ornaments.

### Data Model

- **Synthetic account UUIDs and encrypted email**: The plan says email must never be a partition key, log key, analytics dimension, event stream identifier, or URL identifier. Logs and traces use UUIDs only where operationally necessary.

- **Per-device sessions with limited metadata**: Sessions support device labeling, revocation, and expiration while limiting client metadata to "operational device/browser info, not bird state."

- **Magic link expiration, one-time consumption, and rate limiting**: These are grounded in security and account-protection mechanics: 15-minute expiration, hashed token storage, one-time consumption, and rate-limited lookup.

- **Account timezone**: Timezone is used for "local day/night and mood signals," and local day phase is derived from timezone plus current time.

- **Bird stable UUIDs, pose state, cooldown state, and call signature seed**: Stable UUIDs preserve identity through rename or species changes. Pose state is compact canonical motion rather than a full animation frame, and `call_signature_seed` gives each bird stable procedural call identity.

- **Personality vector storage with internal calibrated ranges**: Constraints and migrations protect vectors from null/reset, while hidden normalized decimals allow calibrated drift. User-facing clients do not receive numeric values because they should not become UI stats.

- **Interaction event logs with idempotency and consumption state**: Events preserve ordered, validated host interaction input for the simulation. `client_event_id` deduplicates retries, and `consumed_at_tick` marks what the worker has applied.

- **Presence qualification from visibility, focus, and recent activity**: The plan requires all three to prevent background or idle tabs from inflating presence. Server-side rate rejection and duration clamping protect against clock skew and replay.

- **Visitor events limited to operational view events**: Visitor events "must not feed presence/drift," preserving the read-only visit contract.

- **Notebook source moments and private source references**: Source moment types let the system generate entries from noteworthy aviary moments, while private refs avoid exposing internal state/events by default.

- **Visit logs reachable on demand without default notification surfaces**: Visit logs provide host transparency, but the plan forbids badge, toast, push, email, or default notification unless the host opts in.

### API Surface

- **Auth/account APIs with matter-of-fact responses and non-enumerating magic-link request**: The plan says responses should not reveal whether an email exists, and account/error responses use matter-of-fact voice rather than naturalist product voice.

- **Account export by verified-address download link**: Export is queued and sent to a verified email address, keeping account data access inside explicit privacy/account settings rather than turning raw state into product display.

- **Host snapshot endpoint with versions and deltas**: Versioned snapshots let clients detect stale views and refresh while keeping payloads compact and authorized.

- **Visitor snapshot endpoint**: Visitor snapshots are read-only and must not trigger greeting, presence, offer eligibility, settle controls, or extra notebook interactions.

- **Polling-first snapshot freshness model with optional SSE/WebSocket**: Polling is acceptable because "cadence is slow and payloads are small"; SSE/WebSocket is only justified if it simplifies revocation or invalidation.

- **Host event batch endpoint with server validation**: The server validates cooldowns, ownership, bird identity, and event shape so invalid events "do not mutate simulation state."

- **Offers reached through top bar, not direct bird-click offer mechanics**: NOT RECOVERABLE FROM PLAN

- **Quiet disabled offer choices and no gamified cooldown timers**: The plan says cooldowns protect "drift calibration" and that disabled offer choices should be restrained; it explicitly says to "avoid gamified cooldown timers."

- **Visit APIs with no chat, comments, visitor interactions, co-presence, profiles, discovery, follows, public listings, or visitor-originated drift**: The reason is to keep visits from becoming social-network patterns; the risk section says mitigation is "API absence."

### Simulation Engine Design

- **Transactional simulation tick**: The worker locks or version-checks the aviary, consumes events, applies drift, transitions moods, writes snapshots, and marks events consumed. If latency exceeds budget, the user sees a "stale but coherent aviary," not divergent client-owned state.

- **Drift function weighted primarily by validated presence**: The plan names presence as the "Dominant weight" and treats listen-in, offers, song fragments, still pool, and settle as secondary signals, making quiet host presence the main relationship input.

- **No negative persisted drift from absence**: This directly supports "absence is never punished." Absence can affect current expressiveness through mood and greeting frequency, but persisted traits do not drift downward.

- **Bounded, asymptotic, capped drift with a decaying accumulator**: These mechanisms "prevent saturation" and stop "a single long session" from producing visible change.

- **One-week measurable and three-week visible drift calibration**: The plan uses these as acceptance timing: internal drift after about one week of regular visits, visible behavioral difference after about three weeks.

- **Internal drift audit records**: Audit records are for "debugging/calibration" and must not be exposed to users or aggregated into analytics.

- **Simulated drift cohorts**: Cohorts test regular visits, invalid background tabs, heavy offers, absence, overlap, and visitor sessions so that "only valid host presence and host interactions produce slow positive drift."

- **Small hidden mood enum that persists across sessions**: Mood is "fast-timescale" but persists and resets through "daily-ish server progression, not on page open." It drives behavior while the UI avoids mood labels as status chips.

- **Mood transition inputs from time, interactions, weather, personality, and bird-to-bird signals**: These inputs make mood drive perch zone, idle motion, call rate, greeting probability, offer reaction, and prose without exposing status labels.

- **Greeting opportunity on session start or return**: Greeting exists so one bird notices the user within one to two seconds. The plan forbids textual welcomes and absence counts, preserving the notice-not-notification contract.

- **Procedural greeting forms**: Glances, head tilts, steps, calls, and response calls are "procedural combinations, not fixed canned animations," so greetings can vary naturally.

- **Call grammar with species motifs, stable bird seed, and personality-shaped variation**: This preserves per-bird recognizability across drift by anchoring motif/timbre identity while allowing timing, frequency, and mood variation.

### Sync and Conflict Model

- **Event idempotency, server-received ordering, and snapshot versions**: These prevent repeated submissions from duplicating effects, keep overlapping devices append-only, and let clients detect stale views.

- **No client-submitted absolute personality, mood, perch, or drift values**: This enforces the server-canonical model and prevents client-to-client or last-write-wins mutation of relationship state.

- **Server-side offer cooldowns and settle state**: These prevent client-side bypass and preserve drift calibration.

- **Rename conflicts using last-write-wins with audit history**: Names can use last-write-wins because they are "user-authored labels, not drift state." Audit history is for support/export.

- **Settings last-write-wins by field**: Settings can use this simpler model because they are "explicit account preferences."

- **Matter-of-fact sync/account error surfaces**: Session timeouts, replayed links, and snapshot failures should not use naturalist voice because the plan says not to wrap sync/account errors in naturalist voice.

### Frontend Rendering Pipeline

- **Routes for aviary, sign-in, account settings, accessibility settings, notebook, and visitor view**: The app shell exists to keep the aviary as the first authenticated screen while still exposing account, accessibility, notebook, and visit surfaces.

- **Aviary scene as first screen with no dashboard/landing framing**: The plan explicitly says "The aviary scene is the first screen after auth" and to "Avoid dashboard/landing framing."

- **Top bar containing account/settings, accessibility, field notebook, and offer affordance, with near-transparent fade**: NOT RECOVERABLE FROM PLAN

- **One horizontal viewport-fitting scene with no panning, scrolling, or zooming**: The rationale is visibility and framing: "Keep every bird visible" and prevent birds from drifting offscreen or being cropped on phone or desktop layouts.

- **Quiet field fallback instead of spinner or ready transition**: If snapshot delivery is delayed, the fallback remains a quiet field with faint motion cues, preserving the alive/quiet product feel instead of exposing software loading.

- **Mood/personality-shaped idle motion**: Animation expresses `wary`, `content`, `curious`, `drowsy/settled`, and `alert` states behaviorally through perch choice, posture, motion, calls, and responsiveness instead of UI labels.

- **Deterministic seeded variation per bird**: The plan gives the reason: behavior should feel "consistent without looping" and avoid obvious cycle repetition.

- **Smooth interpolation between snapshot changes**: This prevents "teleporting" and keeps state changes masked with natural motion.

- **Listen-in by mouse/touch/keyboard with gradual mix ramp**: Listen-in is attention, not isolation. The plan says ramp the focused bird up and others down but "never silence others."

- **Subtle accessible visual focus for listen-in**: The reason is to support focus without turning the bird into a "selected-state badge."

- **Offer menu with seed, song fragment, and still pool choices**: Server validation and cooldowns protect drift calibration, and reactions are rendered according to bird mood/personality.

- **Settle**: Settle creates a "slow evening lighting shift and quieted calls," while "Closing tab without settle is normal and not surfaced as failure."

- **Five-second settle undo window**: NOT RECOVERABLE FROM PLAN

- **Notebook UI as read-only indefinite scrolling without event-log formatting**: This preserves sparse naturalist entries and prevents the notebook from becoming a log or engagement record.

### Audio Pipeline

- **WebAudio procedural call engine**: The plan uses WebAudio for species motif libraries, stable seeds, mood/personality modifiers, scheduling, mixer behavior, and bounded reusable nodes/buffers. It also supports the "Never ship recorded calls" boundary.

- **Listen-in as mix rebalance, not solo/mute**: The rationale is explicitly sensory: ramp times should feel like "attention settling," and other birds remain audible as ambient.

- **Song-fragment offers as procedural motifs**: These avoid recorded tracks and let bird responses join, quiet, or call against depending on mood and vocal frequency.

- **Graceful silence with captions when WebAudio is unavailable or denied**: The plan preserves the visual/caption experience before audio permission and forbids recorded audio fallback.

### Accessibility Surfaces

- **Screen-reader narration generated from the same snapshot state**: This keeps narration aligned with the visual renderer and uses naturalist present-tense prose without state lists, hidden numeric values, mood labels, or event spam.

- **Reduced-motion rendering**: Reduced motion replaces micro-motion and flights with cross-fades, removes leaf/feather drift, and still preserves day/night shifts, calls/captions, mood, notebook, and drift.

- **Call captions generated from actual procedural grammar**: Captions match synthesized call parameters and appear near the calling bird with contrast across day/night states.

- **Keyboard navigation across controls and birds**: Keyboard support lets users tab through top-bar controls, focus birds, move with arrows, toggle listen-in, and exit transient panels.

- **Visible but restrained focus indicators and WCAG AA contrast**: Focus needs to remain visible on bright/dim backgrounds without turning the scene into a "UI grid," and copy surfaces must meet contrast requirements.

- **Unsupported-browser surfaces**: Unsupported browsers receive "matter-of-fact guidance," matching the system-surface voice rule.

### Performance and Observability

- **Bundle, first-bird, frame-rate, memory, and tick-latency budgets**: These keep the first bird visible quickly, preserve idle smoothness, prevent long-session memory growth, and alert when simulation ticks exceed budget.

- **Server-rendered or edge-embedded initial snapshot**: This is an implementation choice to hit the first-bird and snapshot-latency budgets.

- **Code splitting for settings, notebook, visits, export, and similar flows**: Code splitting protects the initial aviary bundle budget by moving non-initial surfaces out of the first load.

- **Compact procedural assets and no recorded audio**: These choices support the 2MB bundle and visual/audio performance budgets.

- **Suspending rendering when hidden and refreshing on visibility return**: This preserves client resources while returning to canonical server state after visibility changes.

- **Synthetic checks, aggregate RUM, and server metrics**: Observability is for operational health: first-bird render, latency, frame timing, audio initialization, reduced-motion path, request counts, queue depth, ticks, email delivery, exports, and deletions.

- **Telemetry exclusions for per-bird state, personality values, notebook content, visitor identity dimensions, and relationship reconstruction**: These exclusions preserve the privacy boundary while still allowing aggregate operational monitoring.

### Privacy, Security, and Data Lifecycle

- **Per-bird interactions stored only for that account's simulation**: The rationale is privacy: analytics pipelines and ML/model-training datasets must not receive per-bird or per-account interaction fields.

- **Privacy policy naming aggregate operational categories and excluding per-bird interaction state**: This makes the telemetry boundary explicit in account settings.

- **Encrypted email fields, hashed tokens, secure browser-safe sessions, CSRF protection, rate limits, and schema validation**: These protect identity, tokens, and mutation endpoints.

- **Unguessable visit tokens hashed at rest**: This protects visit access while allowing revocation and expiration.

- **Export as raw JSON including birds, names, vectors, moods, notebook entries, and settings**: The export satisfies data access while avoiding an in-product stats surface; vectors are raw JSON in account settings, not general display.

- **Soft deletion, recovery window, and hard deletion**: The lifecycle allows recovery during the 30-day window and then removes birds, vectors, notebook entries, events, sessions, invites, telemetry linkages where applicable, and export artifacts.

### Rollout Plan

- **Phase A through Phase F rollout sequence**: NOT RECOVERABLE FROM PLAN

- **Launching closed beta with capped two-bird aviaries**: The plan delays additional bird availability until drift, audio, and notebook telemetry are stable and support reports do not indicate confusion.

- **Age-based third-bird availability after calibration**: Additional birds are enabled by aviary age cohorts while monitoring recognizability research, CPU, and tick cost, keeping the hard cap at seven.

### Testing Strategy

- **Drift, mood, presence, cooldown, caption, notebook, token, sync, visitor, deletion, export, accessibility, performance, and guardrail tests**: The tests exist to enforce the plan's core constraints: slow monotonic drift, no background/visitor drift, server-only personality mutation, read-only visits, account lifecycle correctness, accessible alternatives, performance budgets, and product-language exclusions.

- **Product-guardrail static/content/API/analytics tests**: These ban welcome toasts, streak/achievement/badge/score/level language, hunger/death/distress states, numeric trait exposure, public discovery, comments/chat, recorded audio assets, personality fields in snapshots, and analytics exports of per-bird/per-account interaction fields.

### Key Risks and Mitigations

- **Drift calibration mitigations**: Simulation harnesses, capped deltas, cohort replay tests, and staged rollout address the risk that drift feels too fast, too slow, or too controllable.

- **Sync correctness mitigations**: Server-only personality writes, append-only idempotent events, transactional tick consumption, and versioned snapshots address duplicate or lost interaction effects across overlapping devices.

- **Audio uncanniness mitigations**: Early audio prototyping, motif reviews, recognizability tests, chorus stress tests, and no recorded fallback address artificial or repetitive procedural calls.

- **Accessibility regression mitigations**: Narration/reduced-motion review, screen-reader QA, caption grammar tests, and launch-blocking accessibility criteria address the risk that accessible surfaces become static fallbacks or state lists.

- **Performance mitigations**: Compact assets, code splitting, edge snapshots, procedural audio, and CI gates address the tension between richness and the 500ms/2MB/60fps budgets.

- **Tone leakage mitigations**: Copy linting, design review checklists, and route/component banned-pattern tests address naturalist voice leaking into system UI or gamified announcements entering product UI.

- **Privacy boundary mitigations**: Separate storage permissions, pipeline schema tests, access reviews, and aggregate metrics definitions address interaction data leaking into analytics.

- **Notebook quality mitigations**: Source-moment selection, sparsity rules, forbidden event-log phrasing, and editorial review address generic or overly frequent notebook entries.

- **Visit scope mitigations**: The plan mitigates social expansion through API absence: no comments, chat, public listing, co-presence, profiles, or visitor interaction event types.

### Open Implementation Decisions

- **Personality ranges, presence activity window, final mood enum, rendering technology, snapshot transport, and notebook generation approach**: These remain open only as calibration or implementation choices. The plan says they cannot be used to add gamification, native clients, public social surfaces, punishment for absence, numeric personality displays, or recorded audio.
