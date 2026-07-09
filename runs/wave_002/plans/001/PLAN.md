# Pocket Aviary v1 implementation plan

## 1. Product contract and implementation decisions

Pocket Aviary v1 is a web-only, single-account/single-aviary experience whose primary quality bar is continuity: the first rendered frame depicts an aviary already in progress, the same birds persist across devices, and attention changes them slowly without making absence harmful. Engineering decisions must preserve these invariants before adding breadth.

The following rules are release blockers, not aspirations:

- Start every account with exactly two system-selected birds and enforce a hard maximum of seven. Additional birds become eligible by aviary age only, never by visits, actions, payment, or an engagement score.
- Keep one canonical server-authored aviary per account. Clients submit facts about interactions and render snapshots; they never submit personality values or advance simulation time.
- Treat presence as the conjunction of visible document, focused window, and recent pointer or keyboard activity. An open tab alone never counts.
- Make all personality movement nonnegative and slow. Absence produces no negative trait delta, distress, hunger, death, guilt, or loss.
- Never expose personality numbers in the live product, accessibility tree, snapshot API, logs, metrics, or support tooling. Users encounter personality through behavior.
- Use procedural call synthesis only. Do not add recorded calls, including as fallback.
- Ship screen-reader narration, call captions, keyboard access, and the designed reduced-motion renderer with v1, not after it.
- Keep visits named, revocable, read-only, and off by default. Visitor activity must not enter the host's simulation.
- Use naturalist voice only for aviary, notebook, narration, captions, and offers. Authentication, settings, accessibility controls, errors, export, and deletion use direct matter-of-fact language.
- Build no scores, counters, streaks, achievements, levels, public profiles, discovery, feeds, follows, comments, shared aviaries, native clients, push engagement loop, or user-behavior summaries disguised as notebook observations.

### Explicit resolutions of PRD ambiguities

1. **Canonical timezone:** store one IANA timezone on the aviary, initialized from the owner's browser during adoption and editable in settings. Do not let simultaneous devices silently alternate it. This gives the server one coherent day/night and mood timeline; travel is handled by an explicit timezone update. All clients render the canonical timeline returned in the snapshot.
2. **Mood “reset” versus persistence:** implement a daily-ish pull toward a time-of-day baseline, not a hard reset at midnight or navigation. The stored mood always persists and transitions through ticks, so opening a tab cannot snap it to neutral.
3. **Settle in the top bar:** interactions require settle to be initiated from the top bar, while the layout's icon list omits it. Include a fifth, visually restrained settle control and validate the treatment in design review. Hiding settle in account settings would make a core session gesture undiscoverable.
4. **Personality export:** the account spec explicitly requires vectors in the JSON export while the bird spec prohibits numeric personality surfaces. Treat the user-requested, emailed portability export as the sole narrow exception; raw values remain absent from all interactive surfaces and ordinary APIs. Mark the export schema as internal-versioned and explain that the values are not controls or scores.
5. **Audio autoplay:** attempt audio only when the browser permits it. If an `AudioContext` cannot run without a gesture, show the aviary in graceful silence and automatically enable matching call captions; begin procedural audio after a later qualifying user gesture if permitted. Do not add an interrupting modal or recorded fallback.
6. **Mute and drift:** audio preference cannot reduce drift or expressive potential, because that would penalize Deaf users, muted environments, and autoplay-blocked sessions. Listen-in remains an attention signal whether represented by sound or captions. Mute itself is a presentation preference, not a negative simulation input.
7. **Visit notifications:** implement no push infrastructure and no badges. The default is silent logging. If the owner explicitly enables visit notifications in settings, send a plain email after a completed visit; keep the setting out of onboarding.

## 2. V1 scope and acceptance boundary

### In scope

- Email magic-link authentication, per-device sessions and revocation, verified email changes, 30-day recoverable deletion, hard deletion, and on-demand account export.
- Adoption of two system-selected starter species, user naming/renaming, stable bird identity, and age-based availability up to seven birds.
- A single responsive horizontal aviary with front/middle/back perch zones, canonical local-time day/night, rare ambient weather, server-authored bird activity, and client-only leaf/feather ornamentation.
- Return-greeting, honest presence, listen-in, seed/song-fragment/still-pool offers with per-bird cooldowns, optional settle with five-second undo, and sparse read-only field notebook.
- Persisted personality, mood, render, call, weather, and notebook state advanced by a roughly one-minute server tick.
- Small versioned snapshots, append-only interaction ingestion, owner multi-device consistency, suspend/resume recovery, and idempotent conflict handling.
- A client procedural audio engine, gradual listen-in mixing, chorus management, captions derived from the exact call program, and silence-plus-captions fallback.
- Screen-reader prose narration, complete keyboard navigation, focus treatment, WCAG AA text, and a purpose-built reduced-motion path.
- Named email invitations, one-time redemption into a read-only visitor session, silent visit logging, revocation, and 30-day unused-invite expiry.
- Privacy-separated operational telemetry, synthetic performance monitoring, launch flags, and production safety controls.

### Explicitly out of scope

- Native iOS/Android clients or protocols optimized for native clients; passwords or SSO; payments; scene customization; multiple aviaries; shared or household ownership.
- Direct bird placement, bird catalogs or rarity, trait/stat panels, editable notebook entries, user journals, visit-frequency export, care schedules, hunger, health, or punitive decay.
- Chat, comments, avatars, co-presence, shared cursors, mutual-visit graphs, public links, profiles, discovery, ratings, leaderboards, “show-off” rendering, or visitor interactions.
- Recorded audio, old-browser compatibility bundles, high-impact weather, scrolling/panning/zooming the scene, notifications intended to drive return, and any gamification surface.

## 3. System architecture

Use a small modular service shape first; preserve hard boundaries in code and data so components can split only when load requires it.

### 3.1 Runtime components

1. **Edge web application/BFF**
   - Delivers a minimal HTML shell plus an authenticated, private initial snapshot projection when available.
   - Owns session validation, CSRF protection, snapshot and interaction APIs, settings/account endpoints, and visitor-token exchange.
   - Sends `Cache-Control: private, no-store` for account content. Static code and species assets are content-hashed and CDN-cached; private state is never put in a shared CDN cache.
   - Uses an edge-accessible snapshot projection keyed by synthetic account UUID to reach the first-bird target without placing email or interaction history at the edge.

2. **Browser client**
   - Contains a state adapter, interpolation clock, SVG/DOM scene renderer, reduced-motion renderer, WebAudio engine, semantic accessibility layer, and event outbox.
   - Renders only from versioned server projections plus non-semantic local ornaments. It does not calculate drift, choose canonical mood transitions, or persist authoritative bird state.
   - Stops animation frames and audio scheduling while hidden, then discards stale interpolation and requests a fresh snapshot on visibility return or a long frame gap.

3. **Simulation worker and scheduler**
   - Claims due aviaries with database leases, advances each through deterministic minute-scale steps, consumes new owner interaction events in order, and atomically writes bird state plus the next snapshot projection.
   - Runs without connected clients. Retries are idempotent; two workers cannot advance the same aviary version.

4. **Relational primary database**
   - Stores accounts, sessions, aviaries, birds, canonical simulation state, append-only events, notebook entries, invites, visit records, and lifecycle jobs.
   - Uses transactions, foreign keys, encrypted PII columns, unique idempotency constraints, and version checks. Begin with PostgreSQL rather than event sourcing the entire product; the append-only interaction log is an input ledger, while current simulation state is explicitly persisted.

5. **Email and lifecycle worker**
   - Sends magic links, invite links, optional visit emails, email-change verification, and export download links through a transactional outbox.
   - Executes export, invite expiry, session cleanup, soft-delete purge, and hard-delete jobs. Message payloads use opaque IDs; workers retrieve encrypted email only at send time.

6. **Operational telemetry pipeline**
   - Accepts allowlisted aggregate metrics only and has no credentials or network route to query simulation tables.
   - Production per-account interaction data is never copied to a warehouse, training corpus, or product-analysis stream.

### 3.2 Render/simulation boundary

The server owns semantic facts: bird identity, perch, pose/activity, mood, call event schedule, greeting directive, weather window, day phase, settle state, and transition start/end server times. The client owns presentation: frame interpolation, responsive coordinate mapping, pose tweening/cross-fading, parallax, leaf/feather particles, audio synthesis from an authorized call program, and caption placement.

Every snapshot carries `state_version`, `simulated_through`, `server_time`, `aviary_timezone`, and stable transition identifiers. The client estimates server-clock offset and evaluates transitions against server time. Re-fetching the same snapshot therefore resumes in the same phase instead of replaying an entry animation.

## 4. Persistence model

Use UUIDs for all identities and UTC timestamps for storage. Store the aviary timezone separately as an IANA zone. Encrypt all email values with envelope encryption and keep searchable normalized-email HMACs in a narrowly authorized identity schema; never use email as a foreign key, partition key, log field, or message key.

### 4.1 Core records

- `accounts`: `account_id`, encrypted verified email, email HMAC, status (`active`, `pending_delete`, `deleted`), delete/recover deadlines, created timestamps, schema version.
- `device_sessions`: opaque session ID, `account_id`, hashed refresh token, device label, created/last-used/expires/revoked timestamps. Access tokens are short-lived, audience-scoped, and do not contain email.
- `account_settings`: `account_id`, audio/caption/reduced-motion/narration preferences, explicit visit-email opt-in, optimistic `version`, timestamps. System preference takes precedence over browser default only after the user changes it.
- `aviaries`: `aviary_id`, `account_id` unique, canonical IANA timezone, creation age anchor, settle state and undo deadline, last owner-presence time, `state_version`, `last_simulated_at`, `next_tick_at`, simulation schema/version, deterministic RNG seed.
- `birds`: stable `bird_id`, `aviary_id`, species/profile version, user-visible name, adoption ordinal/time, normalized hidden traits (`boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`), persisted mood and mood-entered time, perch/pose/activity state, call-profile reference, drift accumulator/version, timestamps. Trait columns are inaccessible to the web read role.
- `species_profiles`: immutable versioned silhouette/palette tokens, pose capabilities, and call grammar/motif definitions for roughly six coherent species. A bird pins a profile version so later asset releases cannot silently replace its identity.
- `weather_events`: `weather_event_id`, aviary, type (`rain`, `wind`), start/end, intensity band, seed. Frequency constraints prevent clustering and assertive weather.
- `interaction_events`: `event_id`, owner account/aviary/bird IDs as applicable, session/device ID, client sequence, server received time, bounded observed time, type, validated payload, idempotency key, consumed tick/version. Payloads contain no email.
- `presence_intervals`: server-normalized owner intervals derived from valid presence samples, with session/device provenance, start/end, and merge/consumption status. Overlapping intervals across owner devices are unioned before drift calculation so attention is not double-counted.
- `offer_cooldowns`: bird and offer class, accepted/available timestamps. Enforced transactionally on the server; the client may predict availability only for responsiveness.
- `notebook_entries`: entry ID, aviary, observed-at/date, immutable naturalist prose, structured source-fact references, prose-template version, created time. Retain for account lifetime and paginate indefinitely.
- `snapshot_projections`: aviary ID/version, compact owner projection, compact visitor projection, generated time. Neither projection contains hidden vectors or raw interaction history.

### 4.2 Social and lifecycle records

- `visit_invitations`: invitation UUID, host account/aviary, encrypted invitee email, invitee-email HMAC, hashed one-time token, state (`outstanding`, `active`, `revoked`, `expired`), issued/expires/redeemed/revoked times. Email is data, never identity.
- `visitor_sessions`: random session UUID, invitation UUID, hashed token, issued/expires/revoked/last-pull times. It authorizes only the visitor snapshot route.
- `visit_records`: invitation and visitor-session IDs, first/last heartbeat, approximate duration, completed time. Keep immutable visit history separate from the active-invitation list; revocation removes access but does not falsify historical transparency.
- `outbox_messages`: opaque subject ID, template, encrypted destination reference, send status/retries; redact provider payloads from logs.
- `export_jobs`: account ID, requested/completed/expiry status, encrypted object key. Export objects use short-lived signed links and are deleted after expiry.

### 4.3 Data invariants

- Database roles enforce that only the simulation worker may update personality, mood, canonical perch/pose, weather, or simulation version.
- A bird ID is never reused, regenerated, or changed by rename, profile migration, device sync, or export/import work.
- `birds.aviary_id` and `aviaries.account_id` are immutable. Exactly one aviary per active account is guaranteed by a unique constraint.
- Interaction events are immutable after acceptance; uniqueness on account/session/client-sequence and event UUID makes retries safe.
- Vectors are clamped to their normalized bounds and may not decrease. Add a database check/audit assertion comparing each committed update with the prior vector.
- Visitors cannot write to interaction, presence, notebook, settings, bird, or simulation tables through either application permissions or route availability.

## 5. API and client contracts

Version JSON contracts under `/v1`; use generated schema validation on both sides. Mutations return an operation/event ID and current canonical version, not a speculative personality result. Use same-site secure HTTP-only cookies for owner and visitor sessions, CSRF tokens on owner mutations, strict origin checks, rate limits, and generic responses that do not reveal registered emails.

### 5.1 Identity and account

- `POST /v1/auth/magic-links` accepts an email and always returns the same neutral response. Hash, expire after 15 minutes, rate-limit by email HMAC and IP, and invalidate atomically on first use.
- `POST /v1/auth/magic-links/consume` atomically exchanges the token for a per-device session and rejects replay.
- `GET /v1/account/sessions` and `DELETE /v1/account/sessions/{session_id}` list labels/times and revoke one device; revoking the current device ends it cleanly.
- `POST /v1/account/email-change` and `POST /v1/account/email-change/verify` retain the old verified email until the new token succeeds.
- `PATCH /v1/account/settings` uses `If-Match`/version and returns a matter-of-fact conflict with the latest settings. Preferences, unlike personality, are normal optimistic-concurrency records.
- `POST /v1/account/export` starts an idempotent export job; `GET /v1/account/export/{job_id}` reports status. The emailed link is short-lived and single-purpose.
- `POST /v1/account/deletion` marks pending deletion and revokes non-current sessions; `POST /v1/account/recover` restores within 30 days. The purge worker hard-deletes all account-linked simulation, events, notebook, visit, outbox, and object data after the deadline.

### 5.2 Adoption and bird management

- `POST /v1/aviary/adoption` is available once for a new account, accepts two validated names and timezone, and transactionally creates one aviary plus two randomly selected distinct starter birds from the coherent pool.
- `PATCH /v1/birds/{bird_id}` permits name changes only and uses a record version. It cannot accept species, vector, mood, perch, color, call, or identity fields.
- `POST /v1/aviary/adoptions/{eligibility_id}` accepts the age-unlocked bird that “arrived” and a name; the server validates aviary age and count. Eligibility is returned as quiet product copy, never a progress bar or countdown.

### 5.3 State and interaction

- `GET /v1/aviary/snapshot` returns the compact canonical render projection, accessibility prose facts, current settings, optional one-shot greeting directive, and next suggested pull interval. Support `If-None-Match` by state version; a 304 still includes authoritative server time.
- `POST /v1/aviary/arrivals` is sent on fresh navigation and meaningful visibility return. The server calculates absence from accepted owner presence, chooses at most one primary greeter plus optional staggered response, and returns a session-scoped, expiring greeting directive. Reusing the idempotency key cannot replay a greeting.
- `POST /v1/aviary/events:batch` accepts ordered owner events: listen-in start/end, offer, settle, settle undo, presence sample/end, and audio/caption representation facts needed for exact playback diagnostics. Each event includes UUID, device session, monotonic client sequence, observed timestamp, and type-specific schema.
- The server clamps client times, rejects impossible ordering/payload size, applies per-bird offer cooldowns, and acknowledges accepted/rejected events individually. Offline queues are bounded; stale presence is dropped rather than credited, while discrete offers/listen-in boundaries may be accepted within a short replay window.
- `GET /v1/notebook?cursor=...` returns immutable entries newest-first with stable cursor pagination. There are no create/update/delete endpoints.

Do not put vectors, drift deltas, user visit counts, streak-like summaries, raw event history, exact hidden mood labels intended only for simulation, or “days until next bird” counters in public snapshot contracts. The render projection contains behavior directives and visual tokens sufficient to display current state without becoming a stats API.

### 5.4 Invitations and visits

- `POST /v1/invitations` accepts one email, creates a 30-day invitation, and emails a hashed one-time redemption token. It does not prompt contact import or suggest recipients.
- `GET /v1/invitations` returns outstanding/active invites and a separate cursor-paginated visit log; `DELETE /v1/invitations/{id}` revokes it immediately and invalidates all derived visitor sessions.
- `POST /v1/visits/redeem` consumes the invite token once and issues a scoped visitor cookie. The original link cannot be replayed; an established visitor session remains usable until its bounded expiry or host revocation.
- `GET /v1/visits/{session_id}/snapshot` returns the host's current visitor projection and records a coarse heartbeat for approximate duration. On revoke/expiry, the next pull returns `410` with matter-of-fact copy.
- Visitor HTML contains no owner controls, notebook/account endpoints, greeting request, event outbox, presence detector, or mutation client. Server authorization remains the primary protection even if a visitor modifies client code.

## 6. Simulation engine

### 6.1 Tick execution and correctness

- Default to a 60-second cadence behind configuration. Scheduler workers claim due aviaries with `FOR UPDATE SKIP LOCKED` or equivalent leases, then acquire an aviary-version compare-and-swap inside the transaction.
- Advance from `last_simulated_at` to the current bounded target through deterministic logical steps. After a long outage, aggregate quiet spans analytically and replay only event/weather boundaries; do not execute millions of literal ticks or invent client-side catch-up.
- Read accepted, unconsumed events ordered by `(effective_time, server_received_at, event_id)`. Normalize/union presence, compute deltas and transitions, create notebook facts, write state/projection, mark events consumed, increment `state_version`, and set `next_tick_at` in one transaction.
- Seed pseudorandom decisions from aviary seed + logical tick number + subsystem + bird ID. A retry against the same input must yield the same weather, greeting candidate, mood choice, and call schedule.
- Publish only after commit. The old complete projection remains readable during work; clients never observe a half-updated flock.

### 6.2 Honest presence

The client maintains a local presence state machine from `visibilitychange`, focus/blur, pointer movement, and keydown. It becomes eligible only while all three PRD conditions are true and the last eligible activity is within a configurable initial window (start calibration at three minutes). Send low-rate start/heartbeat/end samples, not every browser event.

The server:

- Accepts samples only from an authenticated owner session and caps intervals to the heartbeat cadence plus grace.
- Rejects hidden/focus-false claims, future times, excessive gaps, visitor tokens, and replayed sequences. Browser claims are not fraud-proof; the goal is accurate normal operation, not invasive surveillance.
- Unions overlapping eligible intervals across tabs/devices before generating presence minutes. Closing, hiding, losing focus, activity timeout, settle, session expiry, or heartbeat loss closes the interval.
- Stores only normalized intervals needed by the user's simulation and deletion/export obligations. Never emits pointer coordinates, keys, or presence history to analytics.

Property tests must prove that hidden tabs add zero time, open-but-idle tabs time out, overlapping devices do not multiply attention, settle and ordinary close end presence equivalently, and reordered/retried heartbeats cannot inflate credit.

### 6.3 Personality drift

Represent each trait in `[0,1]` with versioned seed distributions by species. For each tick, convert normalized owner signals into nonnegative exposure channels:

- Presence supplies the dominant low-weight input to all birds' expressiveness.
- Per-bird listen-in adds stronger targeted social-warmth and vocal-frequency exposure.
- A valid nearby offer adds a small boldness exposure; accepted/investigated offers add a small curiosity exposure.
- Settle contributes no trait direction; it closes presence and affects short-term mood only.

Apply a saturating low-pass update such as `v_next = min(cap, v + alpha_trait * exposure * (cap - v))`, with all coefficients in a versioned configuration and no negative branch. Keep fractional accumulators so tiny legitimate changes are not lost to storage precision. Plumage rendering derives from persisted saturation but the raw value never leaves the server projection builder.

Calibrate with synthetic schedules and consented internal test aviaries, not population-level production histories. Gates:

- A standard “regular attention” scenario produces statistically measurable internal deltas by day 7.
- Blinded longitudinal visual/audio reviews can distinguish baseline from week 3 without per-session jumps.
- A one-day intense session cannot produce an obvious trait change or saturate offer-driven drift.
- Zero presence for weeks leaves personality unchanged; the bird may be currently quieter through mood/recency behavior, never less trusting, less colorful, sick, or distressed.
- All vector migrations are reversible from backups, monotonic, canaried, and covered by before/after invariant checks. A vector reset is a severity-one data-loss event.

### 6.4 Mood, ambient state, and bird-to-bird behavior

Finalize a small v1 enum (`wary`, `content`, `curious`, `drowsy`, `alert`) and a versioned transition model. On every tick, compute transition weights from canonical local time, recent accepted interactions, current weather, current personality, dwell-time hysteresis, and nearby birds. Sample deterministically and persist the result.

- Use slow baseline attraction across the local day: alert is more likely in early morning, drowsy near evening, and most birds settle at night. A nightjar-like species keeps a bounded late-night activity profile.
- Accepted offers can nudge content/curious; settle nudges drowsy/settled presentation; rain temporarily lowers scheduled calls; wind biases alert/wary by species/personality.
- Wary and alarm-call influence can spread with strict caps and decay so the flock feels social without synchronized state flips.
- Enforce minimum mood dwell times and transition hysteresis to prevent flapping. Opening a client is never a mood-reset input.
- Schedule rare rain/wind a few times per canonical week with spacing, low intensity, and short after-effects. No severe weather or per-leaf server state.

### 6.5 Perch, pose, greetings, and calls

- Convert mood/personality into weighted perch choices: boldness favors front, wary favors back, and social warmth affects neighbor distance. Reserve spatial capacity and use deterministic conflict resolution so birds never overlap or teleport.
- Choose pose/activity plans with mood-keyed dwell ranges (preen, scan, head tilt, shuffle, fluff, rest). Persist plan start/end times and pose phase so clients join mid-action.
- For an owner arrival, weight greeting candidates by boldness, warmth, mood, time, and server-calculated absence. Issue one primary directive within 1–2 seconds; if another responds, stagger it by a bounded random offset. Visitors never generate greetings.
- Generate a short future call schedule in each projection. Each event references bird ID, stable call-profile version, motif choices, pitch/rhythm variation seed, mood modifier, start time, duration, and caption syntax tree. Preserve a bird's base interval/timbre signature while varying realization. Coordinate chorus windows from bird-to-bird response rules without making simultaneous canned cues.

### 6.6 Notebook generation

Build notebook prose from structured, server-observed facts and a reviewed deterministic grammar rather than a generic activity log or unconstrained model. Candidate facts include unusual greeting order, sustained quiet/preening, weather plus a specific bird response, a rare chorus, or a perch-pattern change. Score for specificity and novelty, require a long cooldown (initial target 48–72 hours), and suppress entries that merely describe session start, visit frequency, user absence, stats, or every interaction.

Persist the exact selected facts and template version beside immutable prose for testing. Run editorial snapshot tests for lowercase present-tense naturalist voice, bird names, no exclamation/achievement language, no numerical traits, and no claims unsupported by canonical state. Notebook generation failure must never block the simulation tick.

## 7. Multi-device synchronization and failure behavior

- Canonical writes are serialized per aviary. Simulation state uses compare-and-swap versions; settings/name changes use optimistic concurrency; interactions are commutative append-only facts with idempotency keys.
- The browser keeps a small IndexedDB event outbox for transient network loss. It retries with the same IDs in client-sequence order and removes only acknowledged items. Cap size/age and discard stale presence rather than synthesizing attention after reconnect.
- Pull snapshots on navigation, visible-tab keepalive, visibility return, online return, and detected suspension/large frame gap. Add randomized polling jitter; use low-frequency polling for v1 rather than a mandatory persistent socket. Poll intervals may tighten during an active transition or revocation-sensitive visitor session.
- When a newer snapshot arrives, reconcile by stable transition ID. Continue an in-progress transition at current server-time phase; cross-fade only when the old transition has no successor. Never replay a greeting, offer reaction, or fly-in because of a re-fetch.
- Simultaneous owner devices can each render and submit interactions. Their events enter one ordered ledger, presence overlaps are unioned, offer cooldown is enforced once, and the next canonical snapshot converges both devices. No client chooses a winner.
- During simulation outage, serve the last complete projection with a matter-of-fact loading/retry surface only if no projection exists. Queue discrete events safely; do not claim drift has occurred until committed. Alert on lag and catch up deterministically.
- On auth/session conflict, return clear system copy and a sign-in/reload action. Never use a naturalist error, silently reset a bird, or let a stale device overwrite state.

## 8. Frontend rendering and interaction pipeline

### 8.1 First render and scene composition

Use a small application shell and lightweight SVG/DOM scene graph; seven birds do not justify a large game engine. Split account, notebook, accessibility, invitation, and export/deletion UI into lazy chunks. Inline critical palette/perch geometry and the private initial projection when authenticated.

Rendering order:

1. Paint the quiet field immediately from critical CSS.
2. Decode the initial projection and draw at least one bird plus perch using inline/critical species assets; do not await audio, notebook, settings panels, ornaments, or remaining noncritical images.
3. Start birds at their server-time pose phases and initialize the semantic tree from the same projection.
4. Hydrate controls and remaining birds, then start compositor-friendly motion and optional audio.

There is no spinner, skeleton card, wake animation, global fade-in, or “ready” state. Only the one-time post-adoption empty aviary may introduce the first bird with a soft fly-in; established aviaries never show empty.

Compose normalized scene coordinates into background sky/foliage, three perch-depth zones, middle-plane birds/perches, and sparse foreground ornaments. Responsive layout changes spacing and scale while solving constraints that keep every bird fully in frame. Never pan, scroll, zoom, crop a bird, or let the user drag a bird. Day palette and weather come from the snapshot; subtle parallax and leaf/feather drift are client-only and seeded per session.

### 8.2 Motion

- Run a single `requestAnimationFrame` scheduler for all visible elements and mutate compositor-friendly transforms/opacity. Avoid per-bird timers and layout reads in the frame loop.
- Interpolate server transitions with easing and stable IDs. Add bounded, personality/mood-selected micro-motion without altering semantic state.
- Stop rAF and decorative work when hidden; on resume, request state before continuing. Bound and recycle particle/pose objects.
- Full-motion flight follows short restrained paths; idle motion includes preen, scan, tilt, and shuffle. Do not use strobing or high-frequency loops.
- Reduced-motion swaps each activity for reviewed still poses with slow cross-fades, turns flights into perch-to-perch cross-fades, removes leaf/feather drift and parallax, and slows color transitions. It does not stop mood, calls/captions, drift, weather facts, notebook, or greetings.

### 8.3 Interaction behavior

- **Top bar:** account/settings, accessibility, notebook, offer, and settle controls. Fade nearly transparent after a few seconds without pointer/keyboard activity; restore on input. Never fade while any control is focused, a menu/dialog is open, or keyboard-only navigation is active.
- **Listen-in:** click/tap bird or focus it and press Enter. Repeat, empty-space click, Escape, or moving focus away disengages. Visual focus remains restrained but keyboard focus is unambiguous. The event pair is submitted once; audio gain is a local immediate response and later reconciles with acknowledgment.
- **Offer:** open from the top bar, keyboard-navigate seed/song/still pool, then place the offer into the aviary without targeting a trait. Server selects receiving/reaction behavior and enforces per-bird cooldown. Show no countdown meter; unavailable gestures use quiet product copy.
- **Settle:** trigger from the top bar, begin a several-second warm lighting/call reduction and close presence. Any aviary click within five seconds submits undo and reverses from current progress. Closing without settle has identical engine consequences and no recovery prompt.
- **Notebook:** lazy-load a read-only, virtualized panel with cursor pagination. Dispose offscreen row references and restore focus to the invoking control on close.

## 9. Procedural audio pipeline

Model every call as a small versioned grammar program, not an audio asset. Species motifs define relative pitch intervals, note envelopes, rhythm cells, timbre/filter ranges, and optional response forms. A bird's pinned profile plus stable identity-derived parameters supplies recognizable base register/timbre; call event seeds and mood supply bounded variation.

Client pipeline:

1. Parse the call event into an immutable intermediate representation (notes, timing, articulation, timbre, caption tokens).
2. Schedule oscillator/noise/filter/envelope nodes slightly ahead against the server-clock offset. Use pooled/reused nodes or buffers where supported; release/disconnect every voice deterministically.
3. Route each bird through its own gain/pan bus into ambient/ducking groups, gentle compressor, and master limiter. Cap simultaneous voices and preserve headroom for seven birds.
4. On listen-in, ramp the focused bird gradually upward and other birds down to an audible ambient floor using click-free automation; reverse on disengage. Never hard cut or fully mute the flock.
5. Vary microtiming and phase to avoid stacked-loop artifacts while retaining motif identity. The same event ID cannot schedule twice after snapshot refresh.
6. Derive captions from the same intermediate representation actually scheduled, so note count, contour, repetition, quality, source bird, and perch match sound. Never select a generic stored caption unrelated to playback.

Respect browser autoplay and audio-context lifecycle. If creation/resume fails, close partial resources, switch to silence, enable captions for the session, and record only an aggregate error category. When user gesture later enables audio, start with future calls rather than replaying missed ones. Keep user mute separate from technical fallback; both preserve full visual/caption behavior.

Test grammar determinism, variation distribution, bird-recognition listening panels, caption equivalence, listen-in ramps, seven-bird peak levels, long-session node cleanup, suspend/resume, device sample rates, and Safari audio interruptions. Release is blocked if repeated calls sound identical, individual birds cannot be recognized in a seven-bird test, or 30-minute heap/audio-node counts grow.

## 10. Accessibility implementation

Build a semantic DOM alongside the visual scene from the same snapshot adapter; do not try to infer accessibility text from rendered pixels or expose raw state labels.

- Give the scene a named region and birds stable focusable elements with names/species, not personality values. Tab enters at the first bird, arrow keys use a roving tabindex among birds, Enter toggles listen-in, Escape exits, and top-bar/dialog controls follow standard focus order and trapping.
- Maintain a prose narration composer using reviewed naturalist templates and canonical facts. At idle, coalesce changes into one `aria-live="polite"` update every 30–60 seconds. Queue return-greeting, offer reaction, and settle observations with higher priority, but never interrupt repeatedly or narrate raw transitions.
- Keep visual and screen-reader captions/narration consistent, lowercase, present-tense, and specific. Provide a user control to pause running narration without disabling keyboard use or static bird descriptions.
- Show call captions near the calling bird with collision-aware placement, while mirroring them to a caption log/live region at a rate that does not flood assistive technology. Captions are optional by preference and automatic when WebAudio is unavailable.
- Honor `prefers-reduced-motion` on first paint, before animation starts, and allow a matter-of-fact settings override. Avoid a full-motion flash during hydration.
- Specify and test contrast tokens for bright, dim, rain, and settled palettes. All copy, captions, focus rings, icons with meaning, dialogs, and error states meet WCAG AA; never depend on plumage/color alone to identify focus or call source.
- Preserve 44px touch targets where possible, zoom/text scaling, logical reading order, accessible names, error associations, and focus restoration. Automated checks are necessary but release also requires manual screen-reader testing with VoiceOver/Safari, NVDA/Firefox or Chrome, keyboard-only use, 200% zoom, muted audio, and reduced motion.

Accessibility acceptance is affective as well as mechanical: a screen-reader or reduced-motion session must contain continuing, specific aviary behavior and not degrade into a state list or static placeholder.

## 11. Security, privacy, and lifecycle

- Threat-model token theft/replay, invitation forwarding, CSRF, IDOR between aviaries, visitor privilege escalation, email enumeration, event replay/inflation, stored-name injection, export leakage, and worker over-privilege before external beta.
- Store magic-link/invite/session secrets only as hashes, use at least 128 bits of randomness, single-use atomic consumption, secure same-site cookies, short access lifetime, token rotation, content-security policy, output escaping, and route-level account/aviary ownership checks.
- Encryption keys for account and invitation emails are held by a narrow identity service role. Application logs redact emails, URLs/tokens, names where unnecessary, event payloads, notebook prose, and snapshot bodies.
- Keep simulation storage and analytics physically/logically separated. Metrics SDK exposes an enum allowlist; it cannot accept arbitrary properties. Code review and CI fail metrics containing account, aviary, bird, invitation, session, email, species/name, mood, offer, or interaction dimensions.
- Operational request logs may use short-lived trace IDs, not account identifiers. Investigations requiring account access use audited break-glass tooling and never reveal vectors through routine support UI.
- Export is generated from a consistent database snapshot and includes account settings, birds/names, current vectors/moods, and notebook entries as specified. Do not include magic/session tokens, other users' data, internal email HMACs, or raw visitor credentials. Encrypt at rest, use a short-lived signed link, and audit completion without logging content.
- Soft deletion immediately disables social access, new interactions, and outgoing optional notifications while allowing authenticated recovery. At day 30, a resumable purge deletes all keyed records and export objects, then tombstones only a non-identifying purge job. Aggregate telemetry has no account key and therefore cannot reconstruct or target the deleted account.

## 12. Performance budgets and observability

### 12.1 Enforced budgets

- Initial JavaScript: hard maximum 2 MB gzipped, with a lower working target of 500 KB for shell, renderer, snapshot adapter, and audio runtime. CI reports per-chunk deltas and fails the hard cap. Lazy chunks cover settings, notebook, invitations, and lifecycle flows.
- First bird visible: p75 and synthetic target under 500 ms on the agreed mid-tier mobile/4G profile, measured from navigation start to a custom `first-bird-painted` mark after actual paint. Quiet-field paint is not counted as a bird.
- Idle rendering: sustained 60 fps on the reference five-year-old mid-range laptop for 30 minutes, with a frame-time budget near 16.7 ms, long-task tracking, and no background-tab rAF.
- Memory: no statistically meaningful upward heap, DOM-node, WebAudio-node, worker, listener, or retained-notebook-row trend across the 30-minute soak after warm-up. Define a small noise tolerance in the test harness rather than accepting unbounded growth.
- Snapshots remain kilobytes, gzip-friendly, and vector-free. Establish a contract-size test at two and seven birds.
- Simulation p99 latency alarms above five seconds; also monitor scheduler lag, failed/retried ticks, snapshot staleness, and due-aviary backlog.

### 12.2 Privacy-safe signals

Collect aggregate counts/histograms only for HTTP status/latency by route template, first field/bird paint, JS errors by release fingerprint, frame timing, long tasks, heap-soak synthetic results, audio-context failure category, snapshot size/version compatibility, tick latency/lag/retry, outbox backlog, email provider status, and hard-delete completion. Do not record which bird, mood, offer, interaction, account, or visit produced a metric.

Run synthetic journeys from common geographies for sign-in shell, initial snapshot, first bird, 30-minute render/audio soak, reduced motion, and visitor revocation. Synthetic accounts are isolated and clearly marked; their simulation data does not become a source of product conclusions.

Dashboards and alerts should answer “is the service healthy?” rather than “how do people use their birds?” Do not build engagement, retention, average drift, popular species, offer frequency, per-account session, or notebook-content dashboards.

## 13. Verification strategy

### 13.1 Automated tests

- **Simulation unit/property tests:** deterministic retry, monotonic/clamped drift, no absence delta, mood dwell/hysteresis, timezone/DST transitions, nightjar behavior, weather frequency bounds, stable identity, age-only adoption, seven-bird cap, cooldowns, chorus limits, and notebook sparsity/content rules.
- **Data/concurrency tests:** duplicate/reordered events, overlapping devices, worker double claim, crash before/after commit, version conflict, snapshot atomicity, session revocation, magic-link replay, invite expiry/redeem/revoke, and deletion/recovery races.
- **Contract tests:** owner versus visitor projections, no hidden traits/events in responses, schema backward compatibility, ETag behavior, stale-client event handling, matter-of-fact error copy, and naturalist prose linting.
- **Frontend tests:** first frame already in phase, no replay on refresh, responsive all-birds-visible constraints, top-bar fade/focus, listen-in state, settle undo, hidden-tab stop/resume, quiet-field fallback, virtualization cleanup, and reduced-motion no-full-motion flash.
- **Audio tests:** grammar/caption equivalence, no recorded assets in bundle, event deduplication, gain ramps/no hard mute, clipping/headroom, unavailable WebAudio fallback, object cleanup, and cross-browser lifecycle.
- **Accessibility tests:** axe/static rules, keyboard map, focus order/restoration, live-region cadence/coalescing, captions, theme contrast matrices, 200% zoom, and reduced-motion snapshots.
- **Performance/security tests:** bundle and snapshot caps, network/device synthetic budget, 30-minute CPU/memory/audio soak, authorization matrix, CSRF, token hashing/replay, PII log scans, metrics-schema lint, and export-link expiry.

### 13.2 Human quality gates

- Longitudinal internal test with time-compressed nonproduction clocks, followed by real-time multiweek dogfood, to judge whether drift is measurable at week 1 and felt—not announced—around week 3.
- Blind listening sessions at two through seven birds to validate stable individual call recognition and non-canned variation.
- Editorial review of greetings, notebook, narration, captions, offers, settings, and errors against the two voice registers.
- Manual assistive-technology sessions and vestibular review of the cross-fade renderer.
- Cross-device test matrix for current/previous Chrome, Safari, Firefox, and Edge on phone and desktop, including suspension, offline recovery, DST, different device timezones, simultaneous sessions, and revoked visitor access.

## 14. Delivery sequence and rollout

### Phase A — contracts and vertical skeleton

- Freeze v1 terminology, ambiguity decisions above, JSON schemas, data classification, and architectural decision records.
- Build identity/account skeleton, database roles/migrations, one aviary/two stable birds, deterministic tick transaction, private snapshot projection, and a minimal SVG scene that paints one bird under budget.
- Establish CI gates for vector leakage, PII/metrics schema, bundle size, first-bird mark, and deterministic simulation before feature expansion.

Exit: repeated/simultaneous clients read one state; a worker retry is bit-for-bit deterministic; no client can update a vector; first bird is measurable on the reference profile.

### Phase B — behavioral core

- Implement honest presence, event outbox/ingestion, drift accumulators, mood state machine, perches/poses, server-time interpolation, return-greetings, offers/cooldowns, settle/undo, weather, and bird-to-bird effects.
- Add synthetic time-compressed scenario harness and immutable golden histories. Begin real-time internal dogfood immediately because perceived drift cannot be compressed completely.

Exit: invariants and calibration scenarios pass; tab open/background contributes zero presence; no session visibly shifts a trait; opening never resets mood.

### Phase C — audio, notebook, and complete interaction surface

- Ship six versioned species profiles and grammars, WebAudio synthesis/mixing, exact captions/fallback, sparse notebook grammar/persistence, complete top bar, responsive scene, age-based adoption configuration, and field/notebook panels.
- Use an initial configurable eligibility schedule based only on aviary age (provisional launch values: birds three through six at approximately 90/180/270/365 days and bird seven later, e.g. 540 days). Product tuning may change durations before launch, but never the age-only input.

Exit: recognition and 30-minute soak gates pass at seven birds; no recorded audio exists; notebook remains rare and specific under high activity.

### Phase D — accessibility, social, and lifecycle completion

- Complete semantic/narration layer, reduced-motion renderer, keyboard/focus/contrast work, visitor invitation/session/log/revocation, account export, email change, session revocation, and deletion/recovery/purge.
- Perform security/privacy review and manual accessibility sign-off; accessibility is not allowed to trail the beta.

Exit: visitor has no mutation path; revocation ends access on next pull; all accessibility modes preserve the actual aviary; hard-delete rehearsal removes all attributable data.

### Phase E — staged production rollout

1. Staff dogfood with two birds, synthetic and real-time drift observation, privacy-safe health metrics, and daily vector-integrity backup checks.
2. Small invite-only external cohort with two birds; enable visits for a subset only after auth/revocation abuse tests. Keep visit invitations user-initiated and off by default.
3. Percentage ramp by accounts/regions while watching only operational SLOs, support reports, accessibility regressions, and qualitative research—not engagement optimization.
4. Enable age-eligible third and later birds behind a server capability flag after seven-bird rendering/audio capacity is proven. The flag controls system safety, never user reward; eligibility remains account age.
5. General availability only after three-week real-time cohort evidence, browser/performance SLOs, deletion rehearsal, and severity-one restore drill for personality state.

Use independent kill switches for new interactions, notebook generation, weather scheduling, optional visit email, invites, and additional-bird eligibility. Kill switches freeze or remove a peripheral effect without resetting birds, vectors, identity, mood, or existing notebook history. Database backups, point-in-time recovery, and vector checksums must be tested before public beta.

## 15. Primary risks and mitigations

| Risk | Consequence | Mitigation and release gate |
|---|---|---|
| Drift too fast, slow, or saturating | Product feels manipulable or inert | Versioned coefficients, fractional accumulators, synthetic canonical schedules, multiweek dogfood, week-1 instrument/week-3 perception gates, no production population mining |
| Presence falsely counted | Entire fleet drifts too quickly | Three-signal client FSM, bounded heartbeats, server interval union, stale-drop policy, overlap/background property tests |
| Personality lost or overwritten | The perceived bird identity is destroyed | Server-only writer, transactional versions, no LWW, invariant audits, PITR backups, restore drills, severity-one reset alert |
| Tick races or long-outage catch-up diverges | Devices disagree or behavior jumps | Per-aviary lease/CAS, deterministic seeds, atomic projection, analytical quiet-span catch-up, chaos tests |
| Calls sound canned or uncanny | Aliveness collapses immediately | Versioned grammars, stable signature plus bounded variation, no recordings, listening panels, repetition detector, authored motif review |
| Chorus clips or blurs at higher bird counts | Seven-bird relationship becomes ambient noise | Voice cap, gain staging/limiter, staggered motifs, recognition tests at each count, staged capacity flag |
| Browser autoplay prevents already-audible scene | First session feels broken | Nonblocking silent/caption mode, future-call start after gesture, aggregate error monitoring, no modal or canned fallback |
| Full-motion or semantic surface ships while accessible modes feel stripped | V1 excludes users from the core value | Shared state adapter, designed cross-fades/prose, manual AT/vestibular gates in the same release train |
| Live regions or captions overwhelm | Assistive users silence the product | 30–60s coalescing, priority queue with rate limits, user pause, real screen-reader testing |
| Snapshot/API leaks vectors or private history | Stats optimization and privacy breach | Dedicated projection types/DB role, denylist plus allowlist contract tests, response/log scans, sole export exception |
| Visitor link leaks or visitor mutates host | Host relationship is reshaped or exposed | Token hashing/one-time exchange, scoped cookies, read-only projection/database role, next-pull revocation, authorization matrix |
| PII enters logs/metrics/messages | Compliance and trust failure | Synthetic UUIDs, encrypted email, HMAC lookup, opaque outbox IDs, typed metric allowlist, CI log scans, pipeline isolation |
| Timezone/DST or multiple-device zones cause mood jumps | Canonical aviary appears inconsistent | One explicit aviary IANA zone, UTC storage, DST transition suite, versioned setting update, server time in every snapshot |
| First-bird or long-session budgets regress | Aviary visibly loads or degrades | Critical render path, private edge projection, chunk budgets, reference-device synthetic tests and 30-minute soak in CI/release |
| Sparse notebook becomes generic feed | Charm turns into event logging | Structured notable-fact selection, long cooldown, novelty filter, editorial grammar tests, independent kill switch |
| “Harmless” engagement features creep in | Product center shifts to obligation/comparison | Non-goals encoded in API/telemetry absence, terminology lint/review checklist, no visit/retention dashboards or notification substrate |

## 16. V1 definition of done

V1 is ready only when an authenticated owner can adopt two named birds, open the same canonical aviary on two devices, see an already-in-progress scene in under the target budget, be greeted procedurally, sit with honest presence, listen in, offer, settle or simply leave, read rare naturalist notes, and return to persisted mood/personality without any merge or reset. Calls must be procedural and individually recognizable; the experience must remain complete with screen reader, captions, keyboard, muted audio, or reduced motion. A named visitor must be able to redeem a revocable read-only invite without influencing any bird. Export, recovery, deletion, privacy separation, operational SLOs, and restore procedures must work in production-like drills.

No release candidate passes if it contains a spinner-first established aviary, visible personality number, client-authored simulation state, recorded call, negative neglect mechanic, missing accessible mode, visitor interaction, per-account behavioral analytics, streak/counter/achievement, public social surface, or unresolved personality-reset path.
