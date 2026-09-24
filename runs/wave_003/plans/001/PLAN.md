# Pocket Aviary v1 implementation plan

## 1. Product contract and scope

Build a web-only, single-user aviary that feels as if it has continued between visits. Each account owns exactly one canonical aviary. It begins with two starter birds and can grow to at most seven; additional birds become available by aviary age, never by visit count, interaction volume, or payment. Birds keep stable identities and user-assigned, renameable names.

Treat presence as the main relationship signal. Presence counts only while the document is visible, its window has focus, and a pointer move or keypress occurred within a calibrated “few minutes” activity window. A quiet user can watch without clicking. Absence is never a penalty: personality traits only drift upward in the expressive direction, and birds never die, become hungry, show distress, or lose accumulated personality.

The product surface uses specific, lowercase naturalist prose. Sign-in, account, accessibility settings, and error surfaces use direct matter-of-fact copy. The bird greeting is the return welcome; do not add a welcome toast, visit badge, streak, score, achievement, leaderboard, public feed, push campaign, or other engagement surface.

V1 includes magic-link accounts, one canonical aviary, multi-device sync, two to seven birds, the field notebook, presence accounting, read-only visits by individually invited friends, procedural calls, reduced-motion rendering, screen-reader narration, call captions, export, account/session settings, and account deletion. Exclude native clients, payments, shared aviaries, multiple aviaries per account, editable/custom scenes, bird placement controls, public discovery, chat, visitor interaction, and gamification.

### Product decisions where the PRD conflicts

The bird-engine rules say personality values are never shown to the user, while the account-export description says the JSON export contains current personality vectors. Implement the stronger never-expose invariant: the downloadable export omits numeric vectors and includes observable bird state instead. Keep the internal server record canonical. Treat this as a release checklist item because it is a genuine PRD conflict.

The general brief forbids notifications, while the social spec permits a host to opt into visit notifications. Keep all visit notices off by default, do not prompt during onboarding, and provide the explicit settings toggle described by the social spec. The only permitted delivery is a visit notice after that toggle is enabled; ship no push notifications. Confirm the exact opt-in delivery channel with product before enabling it. Until then, the visit log is the only notification surface.

## 2. Service shape and trust boundaries

Use a small web client, an authenticated application API, a simulation worker, a relational canonical store, and an email provider for magic links and invitations.

- **Web client:** serves the scene, captures keyboard/pointer/focus/visibility signals, synthesizes calls locally, renders snapshots, and submits domain events. It is a renderer and event producer, never a simulation authority.
- **Application API:** handles sign-in, account settings, owner snapshot reads, event validation and append, notebook reads, export/deletion requests, and invitation lifecycle. Enforce owner versus visitor capabilities at every endpoint.
- **Simulation worker:** runs the slow tick for each due aviary, consumes ordered events, updates moods and server-only personality vectors, writes notebook observations, and publishes a new state version transactionally.
- **Canonical store:** use a relational database for accounts, sessions, aviaries, birds, append-only interaction events, notebook entries, invitations, and visit records. Maintain a per-aviary serialization point and event cursor. A transactional outbox or equivalent durable queue may wake workers, but the database remains authoritative.
- **Email:** deliver time-limited sign-in links, invitation links, export download links, and verified-address changes. Do not include bird state in email.
- **Operational telemetry:** send aggregate health and performance data to a separate sink. The analytics/monitoring path must not query the simulation database or receive bird or account identifiers.

Serve a small shell and the initial state quickly. Personalized snapshots must be private and must not enter a shared cache. Use CDN edge delivery for the shell and a nearby authenticated snapshot path, with private/no-store semantics. The first bird must render from the initial snapshot without waiting for notebook, settings, or invitation code.

## 3. Data model

Use synthetic UUIDs for accounts and all internal references. Email is encrypted at rest on the account or invitation record, is never an identifier, and must never appear in logs, telemetry dimensions, or service partition keys.

| Record | Canonical fields and invariants |
|---|---|
| Account | Synthetic UUID, encrypted verified email, email-change pending state, creation/deletion timestamps, IANA timezone, accessibility preferences, visit-notice preference. One account maps to one aviary. |
| Device session | Session ID, account UUID, creation/last-used/expiry/revocation timestamps, device label that avoids unnecessary fingerprinting. Tokens are revocable in account settings. |
| Aviary | Stable UUID, account UUID unique constraint, creation time, current state version, simulation tick time, event cursor, local timezone, current lighting/weather/settled state. |
| Bird | Stable UUID, aviary UUID, species, name, hidden server-side personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood and mood timer, current perch/pose/call state, creation/adoption time. Vector values never appear in user-facing APIs or downloads. |
| Interaction event | Server sequence, account/aviary UUID, client-generated idempotency key, type, optional bird ID, minimal payload, server receive time, processing cursor. Types include eligible presence heartbeat, listen-in start/end, offer, settle/undo, and owner state-setting actions such as rename. Visitor activity is not a simulation event. |
| Notebook entry | Stable ID, aviary UUID, generated-at time, observation category and naturalist text. Read-only, sparse, and ordered newest first. |
| Invitation | Stable ID, host aviary UUID, encrypted invitee email, hashed one-time token, created/expiry/redeemed/revoked timestamps, and state. Expire unused links after 30 days. |
| Visit record | Invitation ID, started/ended times, approximate duration, and minimum access metadata needed for the host log. No visitor presence is fed to simulation. |
| Export/deletion job | Account UUID, request/status timestamps, expiring download token or deletion deadline. Hard deletion removes all account-linked records after the 30-day recovery window. |

Do not store raw pointer coordinates, typed keys, or general browsing activity. Presence payloads carry only eligibility and timing information needed to calculate an interval. The owner may have multiple open devices; merge eligible intervals as a union so two simultaneous tabs do not double-count presence-time.

The account export is generated on demand, sent to the verified address as an expiring link, and includes names, species, observable current moods/state, notebook entries, and settings. It omits internal vectors under the product decision above. Hard deletion after 30 days removes birds, vectors, events, notebook, sessions, invitations, visit history, telemetry tied to the account if any, and pending export artifacts.

## 4. API surface and event flow

Use versioned JSON endpoints over HTTPS. Every mutation accepts an idempotency key, validates the current account/visitor capability, and returns the resulting version or a matter-of-fact error.

| Surface | Contract |
|---|---|
| Sign-in | Request a magic link by email; rate-limit per email. Links expire in 15 minutes and are invalidated on first use. Exchange a valid token for a revocable per-device session. Email change remains pending until the new address verifies. |
| Owner snapshot | GET the canonical snapshot after sign-in. Return state version, server time, local day phase, weather/settled state, birds’ observable render state, active transitions, and call plans. Never return personality values. Snapshot payload is kilobytes. |
| Event batch | POST owner events in order: eligible presence pings, listen-in start/end, offer type/target, settle, and settle undo. The server validates bird membership, per-bird offer cooldown, event age/order, and duplicate IDs, then appends events; it does not apply client-computed trait deltas. |
| Notebook | GET paginated read-only observations. Notebook pagination has no mutation effect and old entries remain available. |
| Bird/account settings | PATCH bird name, timezone, accessibility preferences, notification preference, and session revocations through narrow field-specific APIs. Clients cannot edit species, mood, perch, or vectors. |
| Export/deletion | Request an account JSON export and receive the download link through the verified address. Start soft deletion; allow sign-in and recovery for 30 days, then hard-delete. |
| Invitation management | POST an invite with a named email; list outstanding invites and visit history; revoke an invite immediately. The email link has an opaque, one-time redemption token and expires after 30 days if unused. |
| Visit redemption/read | Redeem the invitation token for a short-lived read-only visitor session, then GET the host’s current ambient snapshot. Check invite status on every snapshot pull so revocation appears on the next pull. No visitor event endpoint is authorized. |

The owner client pulls on initial navigation, visibility restoration, recovery from a long frame gap or suspended device, and a low-frequency visible-tab keepalive. Interpolate between snapshots; do not teleport birds between positions. Use state versions and conditional requests to avoid redundant payloads. If two devices submit events concurrently, append both with server ordering and idempotency; clients never upload absolute simulation state.

An invitee sees the aviary scene only: birds, day/night, and weather as currently visible to the host. Do not expose account settings or the host’s notebook through the visit snapshot. Record approximate visit start/end for the host’s on-demand visit log. Revocation and expired/revoked links return the same clear matter-of-fact access-unavailable surface.

## 5. Simulation engine

### Tick and consistency

Run a server-side tick at roughly one-minute cadence, with the precise schedule calibrated in build. The tick runs with no connected client. For each aviary, acquire a lease or row lock, read events after the durable cursor in server sequence, calculate changes, update canonical rows and state version, advance the cursor, and commit atomically. Retrying a tick must not apply its deltas twice. If a worker is delayed, process elapsed time safely without inventing presence or replaying already-consumed interactions.

The tick maintains mood and scene state across sessions. It uses the account’s stored IANA timezone for local time-of-day and daylight-saving changes. Choose and document a stable account timezone from the user’s setting; do not silently let a traveling device oscillate the aviary’s clock. Ambient rain and wind are rare, short-lived seeded events with small mood effects. The client may pause rendering while hidden, but the server tick continues.

### Presence and drift

The client emits a throttled heartbeat only while all three conditions hold: document visible, window focused, and recent pointermove or keypress within the calibrated activity window. Stop heartbeats when any condition fails. Server time bounds each interval; cap gaps between pings so a sleeping laptop cannot create hours of inferred presence. Keep enough server-side session state to merge owner-device intervals as a union. Do not persist pointer/keyboard content. At tab close or settle, end the active interval; neither is required for a valid session.

Implement a bounded low-pass drift update from accumulated presence and interaction signals. Presence-time is dominant. Listen-in weights attention toward that bird’s social warmth and vocal frequency. An offer near a bird modestly informs boldness; an offer the bird accepts, as decided by the server reaction outcome, modestly informs curiosity. Settle quiets mood and closes presence but does not drive personality drift. Apply only nonnegative expressive deltas, clamp to configured trait bounds, and never reduce a trait for inactivity. Values and exact coefficients stay server-side.

Calibrate with repeatable synthetic histories: no visible single-session shift; measurable instrument-level movement after about a week of regular eligible presence; perceptible changes in greeting, perch, plumage, and calls only after about three weeks. Quiet or absent birds remain alive and calm. Lack of recent presence may make greetings and calls less frequent through current behavior, but must not reduce personality or create distress.

### Mood, birds, and notebook

Use a small mood enum such as wary, content, curious, drowsy, and alert. Mood transitions respond to recent owner interactions, local time, short ambient events, and personality, and persist through sessions. Use a transition table or weighted state machine with hysteresis to avoid jitter; a daily-ish baseline transition is not a reset-to-neutral on page open. Bird-to-bird calls can prompt responses, wary mood can spread gently, and chorus windows emerge from overlapping call schedules.

Keep a versioned pool of about six coherent species. Each species defines silhouette, base palette, perch/pose library, and procedural call motifs. Two starter species are assigned by the system; users name the arriving birds from suggestions or custom names. Rename preserves stable bird ID and all history. Offer later birds based on aviary age only; configure age intervals and ramp from three toward seven only after behavior and audio remain recognizable.

Generate notebook observations on the server from meaningful aviary moments and sparse periodic conditions, not from every session or from user visit-frequency. Target roughly one entry every few days for regular use, somewhat more often for notable events. Use authored templates/rules keyed to actual state transitions; do not generate generic event logs or trait numbers. Ensure a notebook observation describes the aviary rather than praising or tracking the user.

## 6. Client rendering pipeline

Use a responsive one-screen horizontal scene with three depth/perch zones and a calm natural palette. A lightweight canvas scene renderer can draw foliage, perches, and bird poses; maintain a synchronized DOM interaction layer for keyboard focus, labels, and assistive technology. Preserve the full scene aspect ratio and keep all birds in view on narrow and wide screens. Birds choose perches from mood/personality; the user cannot arrange them.

Initialize from snapshot timestamps so the first rendered frame starts mid-motion. Use no spinner, wake-up animation, or fade from static. While the snapshot is unavailable, show the quiet field. After first adoption, keep that same field briefly and introduce the two birds with a soft fly-in. The scene has no in-scene buttons, badges, hover tooltips, or labels. Put only account/settings, accessibility, notebook, and offer controls in the thin top bar. Fade the bar nearly transparent after a few seconds of cursor stillness and restore it on pointer or keyboard activity.

The render loop interpolates authoritative states and computes low-cost idle pose variations, parallax, and occasional leaf/feather drift locally; ambient leaves are rendering ornaments, not simulation events. Seed or bound ornament randomness to prevent layout shifts and keep the render loop stable. Day/night colors follow the account timezone; rain/wind shift mood and scene subtly. Settle is a slow lighting/call ramp. Any click during the five-second undo window reverses it and returns to normal state.

### Accessibility surfaces

- **Screen reader:** expose one polite live region with naturalist prose built from the same snapshot as the scene. Update about every 30–60 seconds at idle and promptly for a return greeting, successful offer reaction, or settle. Avoid high-frequency pose narration, raw state lists, and personality values.
- **Keyboard:** Tab through top-bar controls; Tab into the scene focuses the first bird; arrows move between birds; Enter starts listen-in; Escape ends it. The offer flow and settle action are fully keyboard-operable. Moving focus away ends listen-in. Focus indicators remain visible in both bright and dim scenes.
- **Reduced motion:** honor prefers-reduced-motion and the account setting. Render still poses with slow cross-fades, replace flight paths with perch-to-perch fades, remove leaf drift, and slow ambient color changes. Mood, calls, notebook, and canonical simulation continue.
- **Call captions:** generate short naturalist descriptions from the same runtime call grammar actually played, place them near the calling bird, and fade them with the call. If WebAudio is unavailable, enable captions by default and retain graceful silence.
- **Contrast and copy:** all copy surfaces meet WCAG AA. Product copy stays lowercase, present-tense, and specific; account, accessibility settings, and errors use matter-of-fact language. Include captions/narration in contrast review.

## 7. Procedural audio pipeline

Maintain a small motif grammar per species and stable per-bird signature. The server snapshot supplies call timing and mood/scene cues; the client synthesizes each call from motif, stable bird identity seed, mood variation, and per-call variation. Shape timing and pitch by server-derived call plans without sending or exposing the raw personality vector. Keep the signature recognizable as mood and personality change.

Build reusable WebAudio oscillators/envelopes and bounded buffers. Mix simultaneous calls as a chorus with slight natural timing variation. A listen-in focus gradually raises one bird and lowers the others to ambient; other birds never go silent. Clicking the focused bird again, focusing another bird, clicking empty aviary space, or moving keyboard focus away reverses the ramp. Offering a song fragment plays a soft motif; settle lowers call activity gradually. Do not use downloaded recordings.

Create or resume AudioContext on the first user gesture to respect browser autoplay restrictions; audio must not block initial visual rendering. If WebAudio is unsupported or denied, stay silent and turn on captions by default. Do not substitute recorded audio. Test recognizability, listen-in ramp timing, simultaneous chorus clarity, caption alignment, and bounded allocation over a 30-minute session.

## 8. Privacy, security, and sync rules

Only the server writes personality and canonical mood. Process event deltas under one per-aviary ordering boundary. A client cannot overwrite a vector with a stale absolute value. Include an idempotency key for every client mutation, durable event sequence/cursor, state version, and transactional tick commit. Use row-level authorization so owner APIs require the account session and visit APIs require an active invite capability with read-only scope. Recheck revocation on every visitor snapshot.

Magic links are single-use, expire in 15 minutes, and are rate-limited per email. Store token hashes, revoke device sessions on request, and verify a replacement email before switching. Encrypt account and invite emails at rest. Use matter-of-fact errors that say what happened and the next step without exposing internals.

Keep event data only for the aviary’s own simulation and account experience; do not use it for model training, recommendation, population behavior analysis, or third parties. Keep operational telemetry aggregate-only and dimensionless with respect to account and bird. Maintain data separation so the RUM and synthetic-check pipelines cannot access the simulation database. The on-demand visit log is available in settings, sorted newest first, and shows visitor email, approximate time/duration, and outstanding invites without a badge.

## 9. Performance, observability, and rollout

### Required budgets

- Initial JavaScript: under 2 MB gzipped. Split account, accessibility settings, notebook, and invite flows out of the initial path.
- First bird visible: under 500 ms after navigation on a mid-tier mobile device over 4G. Measure the bird’s first painted frame, not merely response completion.
- Idle motion: 60 fps on a five-year-old mid-range laptop.
- Memory: no growth over a 30-minute session. Reuse audio buffers, bound workers/audio contexts, and release notebook render references on scroll-out.
- Tick latency: alert when p99 exceeds five seconds.
- Browser support: last two major versions of Chrome, Safari, Firefox, and Edge. Older clients receive a clear unsupported-browser surface.

Use scheduled synthetic browsers in common geographies for first-bird, load, and tick checks. Collect aggregate RUM for page load, first-bird render, frame timing, audio-context errors, and tick latency; session-duration histograms may be aggregate and anonymized. Do not attach account IDs, email, bird IDs, state snapshots, or per-account interaction histories.

### Delivery sequence

1. **Foundation:** implement account/session security, canonical records, event append and idempotency, tick worker, snapshot versioning, and an internal two-bird scene. Validate multi-device reads against the same canonical state before polishing.
2. **Affective core:** tune presence windows, monotonic drift, mood transitions, greetings, bird-to-bird behavior, notebook sparsity, and call grammars against scripted timelines. Keep tuning parameters server-versioned and reversible.
3. **Accessibility and performance:** complete keyboard, narration, captions, reduced-motion, contrast, low-end rendering, mobile first paint, and 30-minute memory profiling before launch. These are release requirements for the same v1, not follow-up scope.
4. **Private beta:** start with two birds per aviary. Use a small operational cohort and synthetic scenarios to validate tick recovery, sync, and call identity. Collect only approved aggregate operational telemetry; do not collect qualitative per-account behavior analytics.
5. **General availability and bird ramp:** launch with two birds; enable age-based third-bird availability in a controlled server configuration after the two-bird scene and chorus meet budgets. Expand age gates gradually toward five to seven birds only after each increased chorus density retains recognizable signatures and performance. The configured age gate must be based only on aviary age. Never gate on presence, offer count, a streak, or payment.
6. **Visit rollout:** keep visits off until the host deliberately sends an invite. Launch read-only access, revocation checks, expiry, host log, and explicit off-by-default visit-notice preference together. Verify that visitor activity cannot affect drift.

## 10. Release checks and principal risks

Before release, verify these observable outcomes:

- Opening or returning shows an already-moving scene and a single varied bird greeting, without text welcome.
- One account opened on two devices sees the same state version and drift; overlapping sessions do not double-count presence.
- Hidden, unfocused, or inactive tabs contribute no eligible presence; long heartbeat gaps and sleep do not inflate it.
- Offers and listen-in are idempotent; the server owns reactions, mood, drift, and cooldowns.
- Personality values never appear in UI, owner/visitor snapshots, logs, RUM, or downloaded exports.
- A visitor can only view an invited aviary; no visitor presence or events reach the host simulation; revocation takes effect on the next pull.
- Reduced-motion, narration, captions, keyboard paths, contrast, 2 MB bundle, 500 ms first bird, 60 fps, and 30-minute memory gates all pass.
- Account recovery, session revocation, verified email changes, export delivery, 30-day recovery, and eventual hard deletion work end to end.

| Risk | Early signal | Mitigation |
|---|---|---|
| Drift is too fast, too slow, or presence is inflated | Scripted week/three-week curves miss targets; hidden-tab or overlapping-device histories add time | Calibrate low-pass coefficients and heartbeat gap caps with fixtures; use union intervals and monotonic bounds. |
| Sync loses or duplicates state | Divergent state versions, repeated event effects, cursor gaps, or stale-client writes | Server-only writes, idempotency, per-aviary transaction/lease, atomic cursor updates, and replay-safe ticks. |
| Calls sound canned or lose bird identity | Listeners cannot distinguish species/birds after mood variation or at six/seven birds | Keep stable motif signatures, measure recognizability at each age-gated count, limit v1 to seven, and preserve captions from the same grammar. |
| Accessibility becomes a stripped fallback | Narration turns into state labels, reduced motion freezes the scene, captions mismatch audio | Build accessible render modes alongside the primary renderer; make prose and call-plan output share state/grammar; gate launch on keyboard and assistive-technology review. |
| First paint/performance misses the affective target | First-bird p95 over budget, frame drops, memory climb, audio allocations accumulate | Keep first bundle small, defer secondary surfaces, draw from snapshot immediately, reuse audio resources, profile low-end devices and long sessions. |
| Privacy boundary erodes through analytics or exports | Any account/bird dimension appears in dashboards, logs, or downloadable numeric fields | Separate telemetry sink and access policy; schema review export/API fields; delete account-linked records after the grace window. |
| PRD contradictions cause accidental exposure or unwanted notices | Export contains trait numbers, or visit notices appear without affirmative opt-in | Apply the decisions in section 1, keep notices off by default, and resolve delivery channel before enabling the opt-in. |

The implementation should preserve the core distinction throughout: personality is slow and private; mood is fast and observable; presence is honest, quiet attention; the client renders a single server-owned aviary.

