# Pocket Aviary v1 — implementation plan

## 1. Scope and product invariants

Build a browser-only, single-user Pocket Aviary. Each account owns one canonical aviary. A new aviary starts with two system-selected birds; the account can grow to seven as the aviary ages. Ship magic-link sign-in, persistent simulation, multi-device snapshots, offers, listen-in, settle, a sparse read-only field notebook, account export/deletion, per-invite read-only visits, screen-reader narration, reduced-motion rendering, and call captions.

Keep the product observational and quiet. Do not add scores, streaks, visit calendars, achievements, hunger or distress, public discovery, profiles, comments, shared aviaries, payments, native clients, scene customization, or user-controlled perch placement. Do not show personality values in product UI. Names and species remain attached to stable bird identities. The aviary is a single horizontal scene with no panning, zooming, or scrolling.

Use naturalist, lowercase, present-tense prose for bird, notebook, caption, and narration surfaces. Use clear matter-of-fact language for sign-in, account, settings, unsupported-browser, and error surfaces. Do not announce a return with a toast or banner.

### Decisions where the PRD leaves a seam

- Keep personality values server-only and out of the product UI. The explicit account export includes the current vectors as required by accounts_sync.md; treat that user-requested JSON download as the narrow exception to the no-stats UI rule.
- Social defaults to no notifications. Implement the social file’s explicit opt-in visit-notification setting as the sole exception, with a matter-of-fact email sent only when the host enables it. Never send a default, marketing, or general aviary notification. Confirm this narrow exception during product review because the brief also says the product does not email about the aviary.
- Redeem an invite link once into a revocable, read-only visitor credential. The credential can refresh snapshots while the grant remains active; a forwarded or reused redemption link cannot mint additional credentials. Unused grants expire after 30 days.
- Browser autoplay rules may prevent audio before a trusted user gesture. Render the aviary immediately; resume WebAudio at the earliest trusted pointer or keyboard event. Until then, show captions for scheduled calls when captions are enabled, and never substitute recorded audio.
- Put age-eligible bird adoption in a user-initiated aviary options flow, with no badge or arrival notification. The exact entry point is a small UX decision to validate against the sparse top bar; it must not become another persistent scene control.
- Measure bird age from the aviary creation timestamp. Initial configurable eligibility targets: third bird at about 90 days, fourth at 180, fifth at 270, sixth at 365, and seventh at 540. These are age gates, never visit-count rewards, and can be tuned without a schema change.

## 2. Architecture and ownership

Use a web client, a small authenticated application API, a persistent relational store, and a separate scheduled simulation worker. Keep the first release as a modular backend with clear account, aviary, simulation, notebook, and visit modules; split services only if measured load requires it. A transactional outbox handles email delivery. Static versioned assets are served from a CDN.

The server owns all canonical state: account and bird identity, personality vectors, mood, simulation time, notebook records, invitation grants, and event ordering. The browser owns only transient rendering state: interpolated poses, audio nodes, focus, local accessibility preferences not yet synced, and an unsent event retry queue. A visitor client receives a constrained render snapshot and has no event-write permission.

The page bootstrap should return a small personalized snapshot with the HTML through an authenticated edge/API path. Mark personalized responses private and non-cacheable; cache only static assets. The first visual frame must not wait on audio, notebook, settings, or noncritical art. Treat the snapshot as a presentation projection, not a copy of the server’s model.

Partition account-owned records by synthetic account UUID. A row lock or optimistic version check serializes ticks for one aviary. In one transaction, a tick applies eligible events, updates canonical bird state and notebook entries, advances its event cursor and last-tick time, and increments the state version. A retried worker job must be idempotent. Never use an email address as a primary key, partition key, log field, or telemetry dimension.

## 3. Data model

Use versioned schemas and server-generated UUIDs.

- Account: synthetic UUID, encrypted verified email, verification timestamp, IANA timezone, creation/deletion timestamps, settings, and account state. Store per-device session records separately, with creation/last-use and revocation timestamps. Store magic-link token hashes, expiry, use time, and rate-limit state; never log link secrets.
- Aviary: account UUID, creation time, current state version, last completed tick, next due tick, event cursor, and a compact current scene/weather state.
- Bird: stable bird UUID, aviary UUID, species, user name, adopted timestamp, normalized server-only vector for boldness, social warmth, vocal frequency, plumage saturation, and curiosity; persisted mood and mood timer; current perch/pose; stable call-signature identifier; and simulation schema version. Renaming changes only the name. Clients never submit or persist trait values.
- Interaction event: client-generated idempotency UUID, server sequence, aviary/bird reference where applicable, event type, server-received time, bounded event payload, and processing cursor. Types include offer, listen-in start/end, settle/undo, and eligible presence interval. Reject client-supplied personality or mood changes. Preserve ordered append semantics at ingestion, then compact/delete processed event payloads after the tick commit and retry/idempotency window; canonical vectors, not raw history, are authoritative.
- Presence: short-lived server-bounded intervals derived from visibility, focus, and recent pointer/key activity. Retain only enough event detail to process a tick and prevent duplicate submissions. Do not store raw keystrokes, pointer paths, or general browsing activity.
- Notebook entry: UUID, aviary UUID, created-at time, structured observation facts, selected prose template/variant, and rendered prose. Append-only and indefinitely browsable until account deletion.
- Invitation: UUID, host aviary UUID, encrypted invitee email, hash of one-time redemption secret, created/expiry/redeemed/revoked times, notification preference, and status. Keep visitor grants and visit sessions distinct so revocation is immediate and each session’s approximate duration can be recorded. Do not write invitee email or tokens to logs.
- Operational telemetry: aggregate counters/histograms without account, bird, or event identifiers. Keep telemetry storage and credentials separate from the simulation store.

## 4. API and event contracts

All authenticated calls use secure, same-site, revocable per-device sessions and matter-of-fact errors. Validate authorization at the account and bird boundary, apply request size/rate limits, and make mutations idempotent.

- Authentication: request a magic link, consume a single-use link, list/revoke device sessions, and verify a new address before switching it. Links expire after 15 minutes; apply per-email request limits without revealing whether an address has an account.
- Bootstrap/snapshot: fetch the current aviary projection, version, server time, tick time, birds’ names/species, current presentation/perch cues, active transitions, and scheduled call descriptors. Pull on initial load, visibility return, long frame gap, and a low-frequency visible-tab keepalive. Support a version/ETag cursor for unchanged snapshots. Return no raw personality vector.
- Event ingestion: submit a batch with client idempotency IDs and event type. The server assigns sequence and receive time, validates bird ownership, offer cooldowns, durations, settle state, and allowed event payloads, then appends it. Clients send facts such as “listen-in ended” or a bounded presence interval, never state outcomes.
- Notebook: page entries by cursor, newest first, with no edit/delete API.
- Account: update timezone/accessibility preferences, rename a bird, generate a JSON export, request/recover deletion, and manage sessions. Export download links should be short-lived and single-use; send to the verified account address.
- Invitations: create an invite for an entered email, list outstanding grants and visit history, revoke a grant, redeem a one-time link, and request a visit snapshot. On redemption, issue a visitor-only credential scoped to one aviary and the grant. Visit snapshot pulls recheck grant status; a revoked or expired grant returns the same matter-of-fact unavailable surface. Visitors have no event-ingestion route.
- Visit log: store visitor identity from the invitation and approximate session start/end. Derive duration from bounded heartbeats and last-seen time, not a client-reported arbitrary duration. Do not count visitor time as host presence.

The event API must reject old state writes by design: there is no endpoint that sets a bird vector, mood, perch, or simulation timestamp. For concurrent browser sessions, the server sequence is the total order. A lost response can be retried with the same client event ID without double-applying an offer or presence interval.

## 5. Server simulation and drift

Run a due-aviary scheduler approximately once per minute. A worker claims an aviary lease, reads events after its committed cursor, and advances from last-tick time to server-now. If the worker is delayed, integrate elapsed time deterministically rather than running unbounded catch-up loops. Use UTC for persisted times and the account’s IANA timezone for local day/night and mood inputs, including daylight-saving changes.

Per tick:

1. Read the canonical aviary and ordered, unprocessed events under a version/lease check.
2. Fold events into bounded signals: qualified owner presence time; listen-in duration by bird; accepted offers; and terminal settle state. Deduplicate by event ID and apply cooldowns server-side.
3. Update each bird’s mood from its persisted mood, recent session events, local time, short-lived weather, and nearby birds’ call/alarm events. Personality can bias transition probabilities but does not replace mood. Persist mood across sessions; opening a tab never resets it.
4. Update personality with a low-pass filter over capped, positive daily inputs. Use presence as the dominant signal; weight listen-in for social warmth/vocal frequency; use accepted offers for curiosity and proximity of an offer for boldness. Plumage responds to sustained presence. Apply small per-day deltas, clamp to the normalized range, and never decrement a trait for neglect. Calibrate with deterministic cohorts so instrumentation detects a small change after roughly a week of regular presence and visible behavioral differences emerge around week three. Do not expose these values or create a user-facing progress indicator.
5. Derive greeting likelihood, perch choice, idle pose, call timing, and bird-to-bird response from mood, species, and vector. Use days since owner presence only to make greetings appropriately less frequent or more exploratory after a long absence; do not create illness, distress, decay, or punishment. Birds continue their server-side life and calls when no client is connected.
6. Schedule short, rare weather events and ambient scene state. Keep leaf/feather drift client-side as rendering ornaments; it has no per-particle server state.
7. Create a notebook observation only for a meaningful, sufficiently rare aviary event. Use structured facts and naturalist prose templates with variation; do not log sessions, user attendance frequency, trait numbers, or generic event rows. Tune sparsity to about one entry every few days for a regularly visited aviary, with noteworthy moments allowed sooner.
8. Atomically save canonical state, notebook entries, processed-event cursor, tick timestamp, and version. Publish/cache-invalidate the next snapshot only after commit.

Define bounded transition tables, trait weights, step sizes, and mood probabilities as versioned simulation configuration with seeded replay fixtures. Roll out calibration changes behind versioned parameters, not destructive bird resets. Stable IDs and vectors survive simulation deployments and species-pool changes. Mood and behavior should be deterministic for a given tick seed so retries cannot create extra calls, drift, or notebook entries.

## 6. Interaction implementation

- Return greeting: on bootstrap, server selects one likely greeter using boldness, mood, and absence duration. Return a start cue and deterministic procedural variation. Stagger any second greeting by a small random offset. Do not emit a welcome string, elapsed-absence UI, or toast.
- Listen-in: allow pointer, touch, and keyboard focus. The client gradually raises the focused bird and lowers others to an ambient floor, never to silence. Focusing another bird transfers the mix; clicking empty scene, clicking the focused bird, or moving keyboard focus away disengages. Send start/end facts for drift and recovery after reconnect.
- Offer: expose seed, song fragment, and still pool through the top-bar offer control, not by clicking a bird. Let the user choose an intended recipient in the offer flow. Enforce a configurable per-bird cooldown of a few minutes on the server. Mood and curiosity determine approach/reaction; a song fragment also uses vocal frequency; a pool may evoke drinking, bathing, or watching. An offer is a gesture, never sustenance.
- Settle: start a soft several-second evening shift and lower calls. Keep the five-second click-anywhere undo local and send the resulting settle or settle-undo event idempotently. Either settle or loss of qualified presence ends presence; neither creates a penalty or drift decrement. A later trusted interaction may resume normal scene and presence.
- Field notebook: make it on-demand from the top bar. Entries are sparse, specific, lowercase observations and read-only; retain scroll history indefinitely while the account exists.

## 7. Rendering pipeline

Build a scene renderer with a strict boundary: consume immutable snapshot/presentation cues plus local interpolation time; produce a frame without mutating canonical state. Use a single responsive SVG/canvas scene with compact vector bird art and DOM-based semantic/focus targets. Keep all birds inside the viewport at every supported aspect ratio. Separate background, perch planes, bird sprites, subtle foreground, and ambient ornaments so palette/time/weather transitions do not require rebuilding bird behavior. Do not add scene panning, zoom, or user placement.

Embed or fetch the first snapshot alongside the authenticated page shell, preload the minimal scene assets, and draw a frame already in progress. Returning users see current motion without an entry animation. Only the initial adoption flow may briefly show the quiet empty field before the two birds arrive with a soft fly-in. A slow snapshot may show the quiet field with a faint ambient cue, never a spinner. Keep top-bar chrome thin and let it fade after stillness; restore it on pointer/key input and whenever keyboard focus enters it.

Use requestAnimationFrame interpolation between snapshots. Clamp animation deltas after suspend, stop rendering in hidden tabs, and pull a fresh snapshot on visibility restoration. Keep idle movement continuous in visible tabs and within the 60fps budget. Day/night palette follows local time; night remains alive through the nightjar-like species. Reduced motion substitutes slow cross-fades between still poses, removes leaf drift, and slows ambient color changes while preserving the same mood, calls, and state. Do not represent personality with numbers, badges, meters, or hover labels.

## 8. Procedural audio

Create a client audio module from species motif libraries and a small set of synthesis primitives, preferably in an AudioWorklet where supported. Each scheduled call has a stable bird signature and a per-event seed so its pitch contour, rhythm, and timbre vary while remaining recognizable. Mood shapes call contour and articulation; vocal frequency shapes cadence and chorus participation. Keep per-bird channels, chorus headroom, reusable buffers/nodes where possible, and a hard cap on simultaneous voices. Avoid audio loops and downloaded recordings.

Generate the call caption from the same seeded grammar event that creates the sound, so prose accurately describes the audio actually synthesized. Fade captions with the call and place them near the calling bird without introducing persistent UI chrome. On WebAudio failure or denied permission, switch to graceful silence and turn captions on by default; do not attempt a recorded-audio fallback. Resume an audio context only on a browser-permitted gesture. Listen-in uses slow gain ramps; other birds remain audible at a quiet ambient floor. Keep audio allocations bounded and close/reuse contexts appropriately across hidden-tab and teardown transitions.

## 9. Accessibility, privacy, and performance

Ship accessibility with v1. Provide a polite screen-reader narration region driven from the same snapshot, in slow naturalist prose at roughly 30–60 second idle intervals. Prioritize user-initiated greeting, offer reaction, and settle observations without flooding the speech queue. Do not announce raw state rows or personality values. Provide complete tab/arrow/Enter/Escape navigation, visible focus that works in day and night palettes, keyboard access to offers and settle, and WCAG AA contrast for all user copy. Respect both prefers-reduced-motion and the account preference. Captions are user-configurable; if audio is unavailable, enable them by default.

Performance gates for release:

- Initial JavaScript at first paint below 2 MB gzipped.
- First bird visible within 500 ms on a mid-tier mobile device over 4G for an authenticated returning visit.
- Idle motion sustains 60 fps on a five-year-old mid-range laptop.
- No client memory growth across a 30-minute session; reuse audio buffers, dispose bounded workers/contexts, and release notebook rows after they leave view.
- Support the last two major versions of Chrome, Safari, Firefox, and Edge; older browsers get a clear unsupported-browser surface.

Run synthetic browser checks from common geographies and aggregate-only RUM for page load, first bird, frame timing, audio-context failures, request errors, and tick latency. Alert when tick p99 exceeds five seconds. Do not attach account UUID, email, bird ID, event payload, or per-account session history to operational metrics. Keep analytics and simulation storage disconnected. Access logs must redact credentials and query tokens.

## 10. Delivery sequence and rollout

1. Lock API/event schemas, privacy boundaries, visual state projection, and simulation parameter versioning. Produce deterministic sample snapshots and validate the design/accessibility contracts before styling all states.
2. Ship the authenticated web shell, magic-link flow, account/session controls, initial two-bird records, snapshot bootstrap, and responsive moving scene. Meet first-frame requirements before adding secondary surfaces.
3. Add the scheduled server tick, ordered event ingestion, presence qualification, mood transitions, drift, bird-to-bird behaviors, local day/night, and weather. Tune drift against week-one and week-three fixtures using synthetic accounts.
4. Add WebAudio motif synthesis, caption generation, listen-in mixing, offers/cooldowns, settle/undo, and notebook observation generation. Validate audio identity and caption matching across species and moods.
5. Add account export/deletion, verified email change, accessibility settings/narration, then read-only invite grants, visit logs, revocation, and the explicit notification preference. Exercise deletion through mail queues, grants, events, snapshots, and notebook records.
6. Release internally, then to a small canary cohort, then expand in measured steps (for example 1%, 5%, 25%, 100%) only while privacy, sync, audio, accessibility, and performance gates hold. Use aggregate counts and latency only. Disable invitations first if authorization/revocation health degrades; core aviary remains available.
7. Ramp bird count through age-based eligibility only. Keep two at creation and enforce the seven-bird cap in both API and database invariants. Change age thresholds from configuration after observing performance and call recognizability; never use visits, offers, time spent, or payment to unlock birds.

Before launch, exercise concurrent device sessions, duplicate/reordered event retries, hidden-tab resume, delayed ticks, worker restarts, DST boundaries, revoked visitor sessions, email-link expiry/replay, export/delete recovery, reduced-motion, no-audio, and keyboard-only paths. These are release checks to build into the engineering work, not instrumentation of personal behavior.

## 11. Principal risks and controls

- Drift may be imperceptible or too fast. Keep inputs capped, use long-horizon fixtures and internal instrumentation, and adjust versioned coefficients slowly. Never solve weak perceived response by rewarding clicks or penalizing absence.
- Sync may lose or double-apply a delta. Enforce one server writer, ordered events, idempotency IDs, transactional cursor advancement, and tick replay/recovery; clients never write absolute state.
- Audio may sound uncanny or repetitive, or browser policy may block immediate playback. Preserve species identity while varying motif execution, compare synthesized calls with captions, cap chorus density, and keep the first frame independent of audio unlock. Silence plus captions is the only fallback.
- Accessibility can regress as scene motion evolves. Treat narration, reduced motion, captions, keyboard paths, and contrast as release gates and review them with each new animation or control.
- Personalized edge bootstrap can leak another account’s scene if cached incorrectly. Mark responses private/no-store, isolate account auth at the edge, and include cross-account cache-isolation checks.
- Presence signals can overcount idle tabs or be spoofed. Require visible + focused + recent pointer/key activity, cap heartbeat duration on the server, exclude visitor sessions, and never retain raw input.
- Invitation links can be forwarded or linger after revocation. Hash one-time secrets, scope visitor credentials to one grant, check revocation on every snapshot, expire unused grants after 30 days, and expose outstanding grants and access history to the host.
- Cross-file policy seams remain around optional visit emails and vector export. Keep the email exception explicit opt-in only; keep vectors out of normal UI; get product review of these two written choices before general release.
- The top bar is deliberately sparse but still must house adoption, offers, notebook, and settings. Prototype the user-initiated age-eligible adoption path early and remove any badge, streak, or unsolicited prompt.

