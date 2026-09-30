# Pocket Aviary — v1 implementation plan

## 1. Purpose, scope, and delivery rules

Build a browser-based aviary whose continuity is real: the server advances the birds while nobody is watching, and a browser renders their current lives rather than starting a simulation on arrival. The relationship develops through slow, positive personality drift, with moment-to-moment variation expressed through mood, posture, calls, and small interactions. The engineering priority is recognizable individual birds, followed by correct continuity across devices and equally considered visual, audio, and accessible experiences.

This plan is grounded in `prd/product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, and `non_goals.md`. Numerical choices below are initial implementation parameters and acceptance hypotheses, not measurements already established by the PRD. Keep those parameters versioned and tune them with synthetic histories and qualitative design reviews. There are no product implementation changes in this deliverable.

### V1 deliverables

- Modern-browser product supporting the last two major versions of Chrome, Safari, Firefox, and Edge, including their relevant mobile browser environments.
- Email magic-link accounts, one canonical aviary per account, two system-chosen starter birds, names and renaming, and a roughly six-species pool with stable identities and individual call signatures.
- Server-owned personality, mood, perch choices, weather, and behavioral schedules. Age-based invitations to adopt further birds, with an absolute ceiling of seven.
- Honest owner presence accounting; return-greetings; listen-in; seed, song-fragment, and still-pool offers; settle and accidental-settle undo.
- One responsive horizontal scene with front, middle, and back perch zones; continuous local-time lighting; rare weather; and restrained ambient ornaments.
- Sparse, read-only, indefinitely browsable field notebook.
- Multi-device reads of one canonical state; revocable device sessions; verified email changes; export; reversible deletion for 30 days followed by complete deletion.
- Explicit, revocable invitations to a read-only visit. No visitor contribution to the host's simulation.
- Naturalist narration, runtime call captions, keyboard access, visible focus, reduced-motion rendering, and WebAudio failure behavior at initial release.
- Performance checks, aggregate operational telemetry, privacy boundaries, backup and restore procedures, and gradual operational rollout.

### Exclusions and invariants

There are no native clients, payments, SSO, passwords, shared aviaries, multiple aviaries per account, scene customization, bird placement controls, quests, points, badges, levels, rarity, streaks, attendance displays, adoption counters, hunger, death, or neglect distress. Do not create any storage or analytical machinery for public ranking, discovery, recommendation, or relationship-based model training. Visits have no profiles, follows, comments, chat, avatars, co-presence, shared cursors, mutual-visit system, or special appearance for visitors. No recorded bird-call fallback exists.

The following are release-blocking invariants:

1. Bird UUIDs and persisted personality vectors survive naming changes, new devices, server upgrades, and asset migrations.
2. Only the simulation worker updates persisted personality. Every update is a nonnegative additive delta applied to the existing record; an absence never subtracts a trait.
3. The normal UI, its accessible tree, and ordinary state APIs contain no personality vector values or numerical trait scores.
4. Presence requires visibility, window focus, and recent qualifying pointer/key activity at the same time. Opening a tab, playing audio, or visiting somebody else's aviary is insufficient.
5. An interaction is processed at most once despite retries, multiple devices, crashes, and delayed responses. Nobody merges client personality snapshots.
6. Returning does not reset mood, start the scene from a default pose, or produce a textual welcome. One bird notices first within one to two seconds under the supported loading budget.
7. Calls are synthesized procedurally; birds remain individually recognizable across variation and drift. Listen-in never silences the other birds.
8. Accessibility and privacy behavior ship with the first public version.

## 2. Decisions where the specification leaves gaps

These decisions let the team execute without waiting for clarification. The product owner and engineering reviewers should preserve the stated intent when refining parameters.

| Topic | V1 decision and reason |
| --- | --- |
| Simulation cadence | A 60-second logical tick, staggered per aviary. Active interaction presentation is a separate server-authored response plan, so greeting and offer feedback do not wait a minute. Persistent mood and personality updates remain tick-owned. |
| Presence activity window | Start at four minutes since a trusted pointer movement or keyboard press. Heartbeats report at 15-second intervals. Evaluate three-to-five-minute alternatives using synthetic cases and accessible usability sessions; do not count an unattended window. |
| Timezone on multiple devices | Store one IANA timezone on the aviary. Initialize from the browser at account creation; offer an explicit change in settings when a different timezone is detected. A phone cannot silently move the laptop's day/night cycle. Visitors use the host's stored timezone. |
| Mood set | `wary`, `content`, `curious`, `drowsy`, `alert`. Sleeping/eyes-closed is a drowsy posture, not death or a needs state. Night-active species retain an alert/calling baseline. |
| Additional adoption | Age thresholds of 90, 180, 270, 365, and 540 days offer the third through seventh birds. Eligibility is independent of attendance or interactions. Adoption is optional and never automatic. |
| Seven birds from six species | Repeated species are allowed. A stable per-bird acoustic fingerprint distinguishes birds of the same species. Starter species are chosen without replacement. There are no rarity weights. |
| Top bar and settle | Preserve four icons: account/settings, accessibility, notebook, and the offer affordance. The last opens an actions popover containing the three offer types and a separated settle action. Its accessible name is “offer and settle.” This reconciles the layout's four-icon rule with the interaction requirement that settle be reachable from the top bar. |
| Keyboard listen-in | Keyboard focus on a bird engages listen-in; Enter explicitly engages it and is idempotent if already engaged. Escape disengages while leaving focus in place; automatic re-engagement is suppressed until focus moves or Enter is pressed. This supports both the focus-based interaction and the stated Enter affordance. |
| Export versus hidden vector values | The brief contains a real tension: numerical traits are never displayed, while account export explicitly includes current vectors. Honor the specific export requirement as a data-portability exception: include vectors only in the authenticated raw JSON download, never in app views, narration, captions, debug controls, or ordinary snapshot APIs. Do not silently omit export fields. |
| Optional visit notifications versus no notification surface | Provide the explicit off-by-default setting, implemented as a quiet visit notice inside account settings while that surface is open. Turning it on does not enable email, push, toasts, badges, or interruptions in the scene. The settings screen can show the most recent visit; the complete visit log remains available regardless. This chooses the least intrusive notification channel left unspecified by the social PRD. |
| Revocation versus historical visit log | Remove a revoked invitation from outstanding/active access immediately, while retaining historical visits with a revoked annotation. “Absence” in the revocation description means absence of current access, not erasure of who previously saw the aviary. |
| Missing visual design-system document | Do not seek additional repository documents. Establish palette, typography, focus, and contrast tokens within the implementation workstream, with designer approval and explicit contrast measurements before release. |
| Browser autoplay restrictions | Try an already-authorized audio context on return. Until browser permission is available, render fully with captions enabled and provide a matter-of-fact sound control in accessibility settings. Never block the first bird or introduce a recorded fallback. |

## 3. Architecture and authority boundaries

### Service shape

Use a TypeScript modular monolith with separate deployment processes for HTTP rendering/API, scheduled simulation workers, and private background jobs. This reduces distributed write coordination in v1. PostgreSQL is the authoritative transactional store. Durable jobs and an outbox use database tables initially; a queue may later transport job IDs, but cannot become an independent state authority. Deploy static assets through a CDN and route authenticated dynamic requests to the account's home region. Use one primary database authority for an aviary, even if application capacity expands into multiple regions.

Recommended modules are identity, account lifecycle, simulation, interaction ingestion, snapshot projection, notebook, invitations/visits, private mail/export jobs, and operational instrumentation. Modules exchange typed UUID-based commands and versioned schemas; they do not pass raw email or personality into telemetry. Separate database roles enforce the module boundaries rather than relying only on code conventions.

The browser uses a small TypeScript shell and an SVG scene renderer. A lightweight component layer can handle forms and lazy-loaded panels; it must not rerender the scene tree each frame. A dedicated imperative renderer writes transforms and opacity into an existing, bounded SVG tree. Audio is a separate WebAudio controller. A shared semantic event stream drives rendering, audio, captions, and narration.

### Ownership table

| Concern | Authoritative owner | Browser responsibility |
| --- | --- | --- |
| Bird ID, personality, mood, perch destination, ambient weather | Primary DB; simulation worker for behavioral mutation | Read projections; interpolate approved trajectories |
| Interaction receipt, cooldown reservation, immediate response plan | HTTP server transaction using committed state | Submit intent; render returned plan; retry by ID |
| Presence eligibility observations | Browser observes the three signals; server validates bounded intervals | Report qualifying intervals without raw input content |
| Credited presence and listen duration | Server unions accepted intervals and caps their contribution | Never send absolute trait updates or trusted total duration |
| Calls | Server schedules semantic call events; versioned grammar defines expansion | Expand event seed into note/envelope descriptors and synthesize |
| Day/night | Stored host timezone plus server time | Evaluate the provided lighting curve at aligned render time |
| Leaf/feather ornaments | Browser only | Generate bounded visual ornaments; no persistence or mood effects |
| Notebook | Server writes evidence-based prose | Read, paginate, and render |
| Listen-in mix, local sound permission, reduced-motion preference | Device presentation state, with account defaults | Ramp mix and choose rendering register |
| Visitor access | Invitation capability checked by server on every pull | Render read-only projection; stop on denial |

Clients never run the domain tick, extrapolate personality, choose new moods, or reconstruct birds from history. Sub-frame animation math, procedural waveform generation, lighting interpolation, and ornaments are presentation work, not simulation.

### Personalized first response

`GET /app` authenticates the device, reads a transactionally stored scene envelope, and streams HTML containing the bootstrap state and an already-posed lightweight SVG bird scene. The SVG uses the same coordinate system, pose evaluation, and timestamp as the interactive renderer, so hydration adds motion without replacing the scene with a new arrival animation. A tiny render controller starts independently of lazy panels and audio initialization. Do not wait for web fonts, settings bundles, notebook history, or AudioWorklet loading.

The CDN serves static immutable assets and delivers personalized HTML from an authenticated edge handler. Private scene envelopes are stored alongside canonical state and updated in the same transaction. They are not publicly cacheable. If an edge cache is used for immutable envelope bytes, it must validate the canonical version and current authorization before delivery; a stale cache cannot be labeled a fresh successful snapshot. State GETs read the primary authority, not a lagging replica. Use `Cache-Control: private, no-store` for personalized HTML and APIs, and never put tokens or account data in shared CDN cache keys.

Do not introduce an eventually consistent edge simulation to meet the loading target. Measure origin routing and snapshot latency in the intended launch geographies; if distance makes the 500ms target unattainable, improve region placement and critical payload delivery before expanding that geography.

### Persistent simulation and immediate interaction presentation

There are two version counters: `sim_revision` advances when the tick commits domain state, and `scene_revision` advances whenever the visible projection changes, including a committed response plan. A state response includes both. Every mutation affecting an aviary takes its transaction lock in the same order.

An owner activation or offer request can reserve a server-authored plan immediately from the last committed bird state. The plan records its event ID, base simulation revision, server timestamp, random seed, applicable birds, timed actions, and semantic outcome. It is persisted with the accepted event. It may depict a head turn, approach, brief call, or evening transition; it does not directly update the personality vector or persistent mood. At the next tick, the worker consumes the accepted outcome exactly once and applies its mood/drift implications. The tick preserves any still-running plan rather than abruptly replacing it.

This separates a subsecond interaction response from the one-minute long-term clock without giving the browser authority or silently running the tick on request. All devices pulling a scene see the same active world plans. A device's listen-in audio emphasis remains local; it is not a command to change what another device hears.

## 4. Data model, durability, and retention

Use UUIDs for account, aviary, bird, device session, page instance, interaction event, notebook entry, invitation, visit, export, and job identifiers. An account UUID is generated independently of its email. Email is never a sharding key, trace attribute, queue address, log identifier, URL parameter, or telemetry dimension.

### Core records

| Record | Principal fields and constraints |
| --- | --- |
| `account` | UUID PK; encrypted verified email; identity-only blind lookup index; creation time; active/deleting status; deletion due time; verified-email-change state; settings revision. The account is the single durable location of its verified email. |
| `device_session` | UUID; account FK; digest of opaque session token; issued/expiry/revoked timestamps; bounded device display label; last authentication time. Separate token per device; no email copied here. |
| `aviary` | UUID; unique account FK; created time; IANA timezone; `sim_revision`, `scene_revision`; logical tick index; last simulated time; deterministic simulation seed; engine version; last consumed event sequence; next tick due time; active weather; canonical projection bytes. |
| `bird` | Stable UUID; aviary FK; immutable species key; adopted time; name and name revision; persistent five-trait vector in `[0,1]`; seed values; stable call fingerprint and grammar version; mood and dwell timestamps; perch zone/slot; timed pose/trajectory; per-bird offer eligibility time. No replacement-on-rename path. |
| `drift_accumulator` | Bird FK; persisted low-pass signal components; last update; bounded rolling presence/listen/offer bins; daily delta totals; coefficient version. This is simulation data, never analytics. |
| `owner_page` | UUID; account/session FK; activation epoch; last client sequence; monotonic-clock anchor; open/settled/ended status; last acknowledged presence interval; return/greeting receipt. Expiring leases are transport bookkeeping, not presence credit. |
| `interaction_event` | Event UUID; aviary/account/session/page UUIDs; server-assigned per-aviary sequence; event type; receive time; bounded timing metadata; validated payload; accepted response plan/outcome; consumption status. Append-only accepted facts; corrections become subsequent events. Unique `(aviary_id, event_id)` and `(page_id, epoch, client_sequence)`. |
| `scene_plan` | Stable plan ID/event FK; start/end times; participating bird UUIDs; paths/poses/call IDs; semantic facts; deterministic seed; source and schema versions. Normally embedded into the projection; old plans can be compacted after consumption. |
| `notebook_entry` | UUID; aviary FK; observation time; prose; template/version; private evidence references; names at observation time; uniqueness key for observation. No user write/delete endpoint. |
| `aviary_observation_summary` | Per-aviary recent first-greeter history and noteworthy facts needed for sparse prose. Bounded, private, and independent of user attendance rewards. |
| `account_settings` | Narration/caption defaults, reduced-motion opt-in, audio default, visit-notice opt-in default false, privacy-policy version, revision. Browser audio permission is not synchronized as if it were an account preference. |

Use fixed-point integers for personality persistence, for example millionths of the normalized range, and higher-precision residual accumulators to prevent tiny tick deltas from rounding to zero. Add constraints for bounds and nonnegative updates, and restrict write grants on personality columns to the simulation role. Initial bird creation goes through a dedicated, audited creation procedure, not a generic mutable JSON endpoint.

### Sharing and lifecycle records

`invitation` stores host UUID, recipient UUID, token digest, creation/unused-expiry times, redeemed/revoked timestamps, and permission state. `visitor_session` stores a digest-only capability, invitation FK, expiry, and revocation status. `visit_log` stores the invitation/recipient references, start/end times, and approximate duration. It never feeds drift. Unknown recipients have a private, UUID-keyed encrypted address record for invitation delivery; known recipients can reference their account identity instead. The host's email remains stored only on the account record. Address lookup and per-email rate limiting stay within the identity module; lookup digests are not general-purpose identifiers.

`magic_link_challenge` stores token digest, identity draft/account reference, issued/expiry/used state. Unverified new addresses and pending email changes require short-lived encrypted candidate addresses; discard them after expiry or successful verification. Private mail outbox rows refer to identity/recipient UUIDs and templates, not copied addresses. The sending adapter resolves addresses only while delivering; disable payload logging, recipient analytics, and vendor retention of message content where configurable.

`export_job` records account UUID, coherent snapshot revision, status, private object reference, token digest, and expiry. `deletion_job` records lifecycle deadlines and purge progress. Transactional outbox entries carry object UUIDs and operation types; they never contain private vectors or email for an observability consumer.

### Retention and recovery

- Keep birds, vectors, account settings, and all notebook entries until account deletion. Old notebook pages remain accessible indefinitely, with indexed cursor pagination.
- Keep consumed raw interaction events for 30 days to diagnose private simulation correctness and retry windows, then compact/delete them after ensuring all required accumulators and notebook evidence have been persisted. Do not preserve interaction history in an analytics warehouse. Unconsumed events must not expire.
- Retain a small idempotency receipt/tombstone for the event retry window after event compaction; reject events outside the freshness/sequence policy instead of applying a very old intent again.
- Keep visit history for the account lifetime unless deleted with the account. Delete unused expired invitation secrets and temporary recipient records when no retained host visit/invite needs them.
- Export objects expire within 24 hours. Failed partial objects are cleaned up by an idempotent sweeper.
- Use encrypted point-in-time backups and regularly exercised restore procedures. Recovery restores stored vectors and accumulators; it never rebuilds personality from interaction logs. Retain recovery media for at most 30 days, and retire any affected backup/WAL chain at an account hard-deletion deadline after creating a sanitized baseline, as specified in section 12. Reapply a minimal UUID-only purge ledger before a restored database accepts traffic, then remove obsolete tombstones when all affected older media have been retired. Ordinary whole-volume encryption alone does not provide per-account erasure.

A missing vector is a data-integrity error, not permission to reseed a bird. Fail that aviary's state mutation safely, restore its persisted record from recovery material, and raise an operational integrity alarm without uploading the vector. Migrations preserve UUIDs and vectors byte-for-byte unless the migration explicitly applies a documented, nonnegative engine update.

## 5. API contracts and authorization

Use same-origin HTTPS JSON APIs under `/api/v1`, secure HttpOnly session cookies, CSRF/origin validation on mutations, and strict request schemas. Cookie sessions are opaque and stored as token digests. Use `SameSite=Lax` where required for email-link navigation; never store bearer identity credentials in browser local storage. All resource lookups verify owner UUID or invitation capability, including nested bird IDs. Start owner device sessions with a 30-day idle expiry and 90-day absolute expiry; revocation takes precedence over either. Renew activity expiry through authenticated use without persisting an engagement timeline. Exclude query strings, request bodies, cookies, email, names, snapshots, and tokens from access logs.

Successful state responses include `schema_version`, `server_time`, `scene_revision`, `sim_revision`, `simulated_through`, and a projection. Errors include a stable code, safe matter-of-fact message, retryability, and an opaque request ID. Ordinary state responses do not include raw personality or drift accumulators. Settings/name updates use resource revisions and return `409 revision_conflict` with the current safe settings/name view when necessary. There is no generic “update bird state” API.

### Identity and account endpoints

| Method and path | Request / behavior |
| --- | --- |
| `POST /auth/magic-links` | Email input; generic accepted response independent of account existence. Issue a random, single-use, 15-minute challenge. Initial rate limit: five deliveries per address per hour plus an IP-based abuse limit, adjustable without a user penalty mechanism. |
| `POST /auth/magic-links/consume` | Exchange challenge token after an explicit sign-in action on a landing page. Atomically mark it consumed and create that browser's device session. Replay/expiry yields direct retry guidance. Avoid consuming tokens merely because an email scanner fetched the URL. |
| `POST /auth/sign-out` | Revoke current session and clear cookie. Best-effort terminal owner presence interval is flushed first. |
| `GET /account/sessions` | Current and other device labels, last use, and revocation controls; no location inference beyond necessary display metadata. |
| `DELETE /account/sessions/{session_id}` | Revoke an owned device immediately. Ingestion and pulls reject that session thereafter. |
| `POST /account/email-change` | Require recent verified authentication; issue a 15-minute challenge to the new encrypted candidate address. Existing verified address remains authoritative until consumption. |
| `POST /account/email-change/verify` | Atomically verify uniqueness and commit address change. Invalidate outstanding change challenges; do not reseed the account or aviary. |
| `GET`, `PATCH /account/settings` | Read/update defaults and explicit timezone with settings revision. No counters or traits in this surface. |
| `POST /account/exports` | Require recent authentication. Queue a coherent raw JSON export and return job ID/status, with email delivery of a private download link when ready. |
| `POST /account/exports/redeem` | Verify link, expiry, and matching authenticated owner; exchange for a short-lived download ticket. Forwarding a link must not share the account. |
| `POST /account/deletion`, `POST /account/recovery` | Mark for deletion and restore within 30 days, respectively. Deleting accounts render a clear recovery option on every signed-in page. |

Put magic-link and invitation tokens in URL fragments where practical; the landing page exchanges them by POST and immediately removes them from browser history. Set `Referrer-Policy: no-referrer`, no third-party scripts on these routes, and no request-body logging. The account email is resolved within the private sending path, never copied into general-purpose jobs or monitoring events.

### Owner scene and interaction endpoints

| Method and path | Request / response |
| --- | --- |
| `GET /aviary/state` | Pull current canonical projection; supports an ETag based on scene revision. A 304 still returns fresh time/freshness headers and checks current authorization. Full unconditional pull on resumption. |
| `POST /aviary/pages` | Allocate a page UUID/epoch and monotonic-clock anchor. No presence credit and no greeting merely from creation/prefetch. |
| `POST /aviary/pages/{page_id}/activate` | Owner reports a real visible/focused return, with activation ID and last rendered scene revision. Atomically deduplicate and return a server-authored greeting plan plus fresh state. Visitor capabilities cannot use this endpoint. |
| `POST /aviary/events` | Bounded batch of typed events: qualifying presence interval, listen-in start/end, offer, settle, reengage, page end. Each has event UUID, page epoch, client sequence, and small allowlisted payload. Respond per event with accepted/duplicate/rejected status, assigned server sequence, relevant plan/cooldown result, and current scene revision. |
| `POST /aviary/adoption/start` | Idempotently allocate two starter records for a new account and return species/name suggestions. No catalog. Existing starters are returned on retry. |
| `POST /aviary/adoption/complete` | Commit initial names; transition the one-time onboarding state and return arrival plans. Concurrent devices cannot create an additional starter pair. |
| `GET /aviary/adoption-opportunity` | Return one age-eligible proposed bird, or none; no progress bar, next-threshold countdown, rewards, or visit totals. |
| `POST /aviary/adoptions` | Accept current age-eligible opportunity, name optional. Serialize under the aviary lock; enforce seven even if two devices accept simultaneously. Return the new stable bird and soft arrival plan. |
| `PATCH /aviary/birds/{bird_id}/name` | Name of 1–32 Unicode grapheme clusters after trimming, with controls rejected, plus name revision. Escape at rendering rather than treating names as markup. Modify only naming metadata. Bird settings are reached through account/settings. |
| `GET /aviary/notebook?before=...&limit=...` | Owner-only cursor pagination, newest first. Max page size 50; include opaque next cursor. No write/annotation/delete methods. |

A presence event reports only monotonic interval endpoints, last qualifying-activity age, eligibility flags, and sequence/clock-anchor information. It contains no pointer coordinates, key identities, DOM paths, camera data, or user-entered text. The server never accepts a client-provided trait delta, mood override, absolute presence total, call frequency score, or arbitrary scene plan.

Listen-in events reference bird UUIDs and bounded start/end timing, not “three minutes credited” supplied as truth. An offer payload names one of `seed`, `song_fragment`, or `still_pool`, a small-library motif ID if relevant, and an optional target bird UUID. The top-bar flow may offer near the currently attended bird; without a target the server picks an eligible bird from the committed state. Bird clicks only engage listen-in. Cooldown is per receiving bird across all devices. Start at three minutes, reserve it transactionally on acceptance, and let a drowsy refusal count as a real gesture without fabricating curiosity gain.

Settle and reengage carry a settled-session epoch/plan ID. The server acknowledges whether that settle epoch is still current before accepting an undo. Duplicate settle, repeated clicks, or a late undo cannot resurrect an older state.

### Projection shape

The scene envelope carries aviary UUID, revisions/time, host timezone and lighting phase, current weather timing, and up to seven birds. For each bird it includes ID, name, species/asset version, mood, rendered plumage colors, perch zone/slot, current pose/action start/duration, planned transition control points, semantic call schedule, stable signature/grammar reference, and any active response plan. Include a short, versioned narration summary and relevant semantic observations. Do not send seed personality, normalized traits, low-pass signals, user attendance history, or analytical “expressiveness” scores.

Represent call events as event ID, bird UUID, start time, grammar version, motif tokens, deterministic variation seed, and bounded acoustic parameters. The client expands the score consistently; exported raw personality is accessible only through the distinct account-export pipeline. Keep a two-minute semantic schedule horizon, replenished by minute ticks and visible polling. Projection target: below 12KB compressed at seven birds, including active plans, with strict bounded arrays. Notebook history is not embedded.

### Invitation and visit endpoints

| Method and path | Behavior |
| --- | --- |
| `POST /account/invitations` | Host explicitly enters recipient email. Persist a new invitation and queue one email with a one-use link. Unused invitation expires in 30 days. No global sharing flag or automatic re-invite. |
| `GET /account/invitations`, `GET /account/visits` | Owned outstanding/active invitations and historical visits with recipient email, date, approximate duration, revocation status. Pagination; no settings badge. |
| `DELETE /account/invitations/{invitation_id}` | Transactionally revoke token and any derived visitor session. Update current-access list inline; no success toast or host notification. |
| `POST /visits/redeem` | Atomic single-use token exchange into a read-only visitor cookie. Recipient address remains server-side. Visiting does not require creating an owner aviary. A bearer email link is not proof of a different person's identity; do not expose additional private account resources. |
| `GET /visits/state` | Validate derived cookie, invite status, host status, and expiry on every request, including ETag/304 handling. Return the host's rendering projection, excluding private notebook, settings, export, and interaction data. |
| `POST /visits/end` | Optional best-effort visit-log termination; does not append a simulation event. Server request timestamps provide approximate duration if the browser disappears. |

A redeemed visitor capability lasts at most 24 hours and can also end on revocation, host deletion, or explicit close. The one-time link cannot create another session. The expiry of an unused link and the expiry of its derived session are distinct. Returning after the session expires requires a deliberately issued new invitation, not automatic permanent access.

Visitors can use their own sound permission, captions, reduced motion, and narration controls, but cannot focus a bird into listen-in, activate greetings, offer, settle, alter host settings, or browse the host's private notebook. The server enforces permissions even if a modified browser calls an owner endpoint. The normal visitor state poll is every 15 seconds; revocation terminates at the next pull. When a visitor cannot refresh because of connectivity, stop display/audio on the 30-second authorization lease expiry and show the matter-of-fact unavailable surface. Do not retain a host scene indefinitely after losing authorization checks.

## 6. Precise presence, listen duration, and session lifecycle

### Browser eligibility state machine

Track visibility, actual window focus, `performance.now()` of the last trusted pointer movement or keyboard press, and the page's open/settled/ended state. Normalize modern keyboard press events to the PRD's keypress concept without recording their contents. Ignore synthetic programmatic events. A pointer click alone, an audio callback, a network heartbeat, or animation frame is not qualifying recent activity. Touch pointer movement is eligible; verify supported mobile visibility/focus behavior instead of replacing focus with visibility.

Eligibility is the conjunction:

`visible AND document.hasFocus() AND activity_age <= 240 seconds AND page_is_open`

When a signal changes, close the current qualifying interval immediately. Start another only when all conditions hold. On visibility/blur changes, flush the short completed interval using a best-effort same-origin request/beacon; on hidden, stop frame rendering and audio scheduling. When activity becomes stale, terminate the interval at the timeout boundary without a warning or reminder. Mouse stillness hides the top bar sooner than presence ends: they are separate timers.

Send acknowledged qualifying intervals approximately every 15 seconds, split at changes in eligibility. Use monotonic endpoints tied to a server clock anchor and page sequence. An interval's duration cannot exceed the heartbeat window plus a small scheduling tolerance; never turn a large suspended-clock gap into attention. On pageshow from back/forward cache, a long frame gap over five seconds, or an unexpectedly large heartbeat gap, discard the unacknowledged gap, close the old epoch, fetch state, and establish a new anchor before crediting time. Re-entry can greet without awarding presence until a qualifying activity exists.

The server clips intervals to plausible server-anchored time, rejects future/old intervals, and accepts only bounded reports received within 30 seconds of interval end. An ambiguous unload can lose at most the unacknowledged short interval; it cannot gain unattended time. Do not issue credit because an owner-page lease remains alive. Authenticity of human attention cannot be proved by a browser heartbeat, so enforce honest-client behavior plus bounded abuse protection rather than claiming this mechanism is fraud-proof.

### Multi-device accounting

The worker credits the union of qualifying owner intervals across page instances. Overlapping laptop and phone intervals yield one second of aviary presence, not two. A duplicated heartbeat yields zero additional credit. For a specific bird, union same-bird listen-in intervals; if multiple owner devices attend different birds at once, divide the available account attention across simultaneous targets so their total does not exceed qualifying owner attention. This keeps multiple tabs from becoming a drift multiplier.

Listen-in duration used for drift is gated by valid presence and active page state, even if the local mix remains audible while eligibility briefly lapses. End it on focus leaving a bird, bird toggle, empty-space click, another bird engagement, Escape, blur/hidden, settle, or page termination. The server closes uncertain intervals at their last acknowledged boundary, not at a future lease deadline.

### Settle and ending

On settle, immediately terminate the submitting page's qualifying interval, flush listen-end, and create a server-authored evening lighting/quiet-call plan. Settle contributes only a small temporary mood-quieting impulse; it is never a personality penalty or bonus. The shared world presentation becomes settled, so a second owner device sees the same evening plan. Each page must stop crediting once it observes that settle epoch; any still-visible, already-active second page can explicitly reengage the shared aviary.

Any click in the scene within five seconds submits an undo tied to the plan ID and reverses the transition from its current phase. After five seconds, a deliberate scene click or keyboard action reengages; pointer movement that merely reveals the top bar does not undo settling. Reengagement opens a new presence window only when the eligibility conjunction holds. A reversal ramps lighting and audio smoothly; it does not snap back to the pre-settle frame.

Closing without settle sends only terminal accounting when possible. It has the same drift consequence as ending by settle: no further presence is counted and no absence penalty exists. There is no recovery message, missing-goodbye flag, or visit-frequency UI. A settle from one page does not fabricate an end for valid attention on another page before that page observes it; union accounting and server epoch ordering establish the actual credited boundary.

## 7. Server simulation engine

### Scheduling and transactional tick

Give each aviary a deterministic offset within the 60-second cadence to avoid minute-boundary spikes. Due workers select bounded batches using database leases/row locking; logical tick identity is `(aviary_id, tick_index)`. Database time establishes due times. A stale worker lease may be retried safely because commit is guarded by tick index and canonical revision.

Each logical step executes as follows:

1. Lock the aviary and its behavioral records; confirm account is active, expected tick index matches, and existing vectors are present.
2. Determine the next UTC tick boundary and local circadian phase from the stored timezone. Snapshot the eligible event-log sequence boundary in this transaction.
3. Read a contiguous prefix of accepted events after the consumed cursor in server sequence order, stopping at the first event whose server acceptance time is beyond this logical step. Never skip a future event and advance the cursor past it. Validate effect timing, union qualifying presence/listen intervals into the appropriate UTC bins, and apply already-reserved offer/settle outcomes once. Receipt time determines the step that consumes an event; its bounded validated interval determines the time credited. Remaining events wait for the appropriate next step.
4. Update persisted low-pass signals, compute nonnegative trait deltas, cap/round with residual preservation, and add deltas to stored vectors. No user-input absolute vector enters this path.
5. Advance mood dwell/recovery timers, weather impulses, bird-to-bird reactions, perch choices, and semantic call/action schedules with deterministic versioned randomness. Retain compatible active interaction plans.
6. Evaluate sparse notebook candidates against current facts and private observation summaries, reserving unique entry keys if an observation qualifies.
7. Write all birds, accumulators, summaries, notebook additions, simulation revision, scene projection, cursor, and next due time in one transaction. Mark the tick committed. Publish only UUID/version outbox references if needed.

A crash before commit has no effect; a crash after commit cannot replay trait deltas because the tick identity and event cursor already advanced atomically. API ingestion, adoption, and simulation acquire the aviary lock consistently. A transport queue can redeliver a job but cannot create a second writer. The API role cannot UPDATE personality columns. Avoid any “save client snapshot” route.

### Drift function and calibration

Persist each trait `p_j` in `[0,1]`. Per bird, compute private rolling 24-hour inputs: `P` is union presence minutes normalized by 15 minutes and capped at 1; `L` is credited listen minutes normalized and capped at 1; `O_b` and `O_c` are capped qualifying offered-near and accepted-offer signals, normalized by at most three gestures per bird per day. These caps bound impact; they are not rewards, streaks, or user-visible attendance calculations.

Use a nonnegative signal mix:

- All five traits receive the presence component `P`.
- Social warmth and vocal frequency additionally receive up to `0.15 * L`.
- Boldness receives up to `0.05 * O_b`; curiosity receives up to `0.05 * O_c`.
- Plumage has no click-driven component. Settle has no slow-trait term. Ignored/refused offers do not become negative input.

Low-pass each signal with a seven-day time constant: `E_j_next = E_j * exp(-dt/tau) + U_j * (1 - exp(-dt/tau))`. Compute `delta_j = eta_j * (1 - p_j) * max(E_j_next, 0) * dt_days`, initially with `eta_j` around `0.008/day`, bounded by a per-trait daily increase ceiling of `0.006`. Apply `p_j_next = min(1, p_j + max(delta_j, 0))`. Inputs and coefficients are versioned; initial trait values should occupy roughly `0.25–0.55`, with enough variation for two starters to feel distinct and enough headroom for months of change.

The exact values require calibration. For an initial trait near 0.4 under regular 15-minute daily presence, this family of parameters should yield changes on the order of hundredths by the first week and several hundredths by three weeks, rather than an observable jump in one visit. The team must establish actual output using the complete rolling-window implementation, fixed-point rounding, and visual mappings; do not declare perceptibility from this estimate alone.

The filter retains a short positive tail from attention before the user left, consistent with absence-time drift from prior inputs. It decays to zero input and never lowers `p_j`. Clamp negligible residuals to zero so years of absence do not create material gains. Daily gain ceilings and diminishing headroom prevent clicking from saturating traits. Do not renormalize traits down when changing coefficient versions.

For the “quieter after absence” behavior, maintain a separate, bounded temporary interaction excitation that decays over approximately 36 hours. It modulates greeting elaboration and call joining above a stable species/personality ambient floor. It never changes stored social warmth, vocal frequency, or plumage downwards, never induces illness/wary from absence alone, and never removes the required first-bird notice. Returning after two weeks is a quiet glance from the same bird, not visible reproach.

### Mood, perches, and circadian behavior

Use a weighted transition model with persistent dwell times and explicit reasons. Initial typical dwell periods are tens of minutes to a few hours, with at least five minutes of hysteresis except a significant immediate ambient/offer impulse. Daily-ish reset means decay of yesterday's transient biases and movement toward the time-of-day baseline, not setting every bird to content at midnight or login.

Start ordinary mood-change opportunities with a mean interval near 90 minutes and a five-minute minimum dwell: at each tick, opportunity probability is `1 - exp(-dt / 90 minutes)`. On an opportunity, sample the next state from normalized nonnegative weights for circadian baseline, personality bias, and recent impulses. Recent offer/alarm biases decay on a roughly six-hour clock, while brief weather modifiers expire with the weather/recovery window. Persist both the sampled mood and its timers. These initial parameters require the same synthetic validation as drift. At early morning weight alert/curious; daytime favor content/curious; dusk and night favor drowsy. Species configuration modifies those baselines, including the nightjar-like species remaining active. Current personality biases probabilities, not eligibility for survival: high boldness reduces the wind/alarm chance of wary; curiosity increases investigation; warmth increases nearby perching and responses. Accepted offers nudge toward content; song fragments can prompt a response, a pause, or a counter-call. Wary is an ordinary short-lived posture, never an absence punishment.

Perch choice samples front/middle/back utility from boldness, mood, nearby birds, and available safe slots. Add dwell/hysteresis so birds do not shuffle every minute. Use normalized scene slots rather than physical pixels; the responsive renderer maps the same choices to its viewport. Prevent impossible overlapping destinations and concurrent conflicting flights. Store action start/end and trajectory so a client arriving midway evaluates the correct phase. A flight at a tick boundary is a continuation, not a teleport.

Lighting is a continuous curve anchored to the host's local time, evaluated from server time. DST changes do not double-credit elapsed time: duration accounting remains monotonic/UTC. Blend a timezone change over several minutes instead of resetting moods suddenly. Day/night affects call cadence and posture; night always retains some living motion, and the night-active species may call.

### Weather and bird-to-bird coupling

Schedule mild rain roughly two or three times per week, initially five to ten minutes per occurrence, plus occasional soft wind. Weather is generated by private server randomness and persists in the canonical scene even without clients. It is not a live external weather integration. Rain briefly suppresses calls through mood/acoustic scheduling, never by decreasing the vocal-frequency trait. Wind can make a bird alert or briefly wary; decay these effects after the event. There are no storms, thunder, snow, alerts, or user weather tasks.

Call-response candidates use social warmth, species-compatible intervals, and current mood. A soft alarm can briefly influence nearby birds' mood at the next tick or create a bounded response plan. Restrict a response chain to a small depth, with refractory periods, to avoid an escalating perpetual alarm or chorus. Choruses emerge from overlapping schedules and probabilistic responses, rather than firing all birds as an arrival effect. Cap planned concurrency and reserve gaps so individual signatures remain audible.

### Return-greeting

Activation is independent of qualifying presence: a visible, focused user deserves a greeting even before moving a pointer. An inactive/preloaded/background page does not. Record each genuine activation idempotently and compute absence from the account's most recent active owner-page boundary, not merely the latest navigation timestamp. Quick returns and multi-device opens while another page is active use the short-absence branch.

Select one initial bird using a weighted lottery favoring boldness/social warmth and accounting for mood. Use continuous seeded variation in head-turn angle, latency, step distance, motif form, spacing, and envelope; do not rotate a finite set of canned clips. A short absence is often a glance; a longer one may produce reorientation, a step toward the front, or a longer call. Wary/drowsy birds can greet by small posture changes. Any secondary response is staggered by a randomized offset and follows bird-to-bird rules, not a synchronous welcome chorus.

Plan the initial notice around 0.6–1.5 seconds after visible activation, with an acceptance ceiling of two seconds after an eligible first scene is available. The end-to-end navigation test must also verify the first-bird load target, not hide a slow load behind this relative clock. Persist the plan so retries cannot replay it and another device does not independently choose a different greeter for the same activation. Treat an audio-blocked greeting as a full visual/narrated/captioned greeting.

### Catch-up and outages

Every active aviary is scheduled even without viewers. On worker outage, retain last committed vectors and pending events; never reset or advance from a client estimate. Catch up through deterministic logical minute steps in bounded transactions and fairly scheduled batches. Use fixed event boundaries and random seeds so chunking does not change outcomes. Capacity-plan for recovery of at least a one-day outage without dropping intervals or starving recently active aviaries. Tick latency and backlog age are separate alarms.

For very long downtime, an optimized quiet-span integrator may handle spans with no events, but only after proving equivalence for mood timers, drift integrals, weather, and notebook sparsity against the minute-step reference. Do not ship an unverified analytical shortcut initially. During stale state, browsers keep safe idle presentation and request state recovery, stop expired semantic calls, and expose a calm matter-of-fact loading/retry status through the shell when necessary. They cannot invent missed history or audio-burst all elapsed calls.

## 8. Sync and consistency across devices

### Read lifecycle

On owner navigation, deliver the inline canonical envelope and begin rendering immediately; register/activate the real visible page independently of the render-critical path. Poll state every 15 seconds while visible, with slight per-page jitter. Poll on becoming visible, regained focus when stale, back/forward-cache return, a render gap greater than five seconds, and successful interactions whose response indicates a newer revision. Hidden pages stop rendering and routine owner polling; the server tick continues. Every fresh GET and mutation reads the primary authority.

Estimate server-to-monotonic offset from request/response midpoint and track uncertainty. Use `performance.now()` for local elapsed time; do not trust wall-clock jumps. Schedule rendering/audio against an aligned presentation clock. Ease small offset corrections over time; a major correction triggers fresh state and skips expired events. State revision prevents out-of-order responses from replacing a newer projection. An old response can acknowledge its own event receipt without rewinding the scene.

Interpolate positions, lighting, pose phase, and active response plans. A current scheduled flight is evaluated at its actual phase on return. For a just-learned new transition, use its remaining trajectory or a short position reconciliation; never replay a completed action. Calls have unique IDs and an absolute event time; a bounded seen-ID ring suppresses replay after polling or page resumption.

### Conflict and retry policy

Each page serializes its local event sequence and submits small batches. The server assigns a total order under the aviary lock; cross-device order is acceptance order, not client clock order. A repeated event with the same ID and payload returns its original receipt. Reusing an ID with different payload returns a validation conflict and does not mutate state. For missing per-page sequence, return the expected next sequence and allow a bounded replay of receipts; terminal events explicitly close an epoch rather than silently skipping gaps.

A transient network failure retries only still-fresh intents with the same event UUID and sequence. Do not persist a long offline queue of presence, offers, settle, or greetings. An offer whose delivery is uncertain is retried by the same UUID to discover whether it was accepted; do not create a second offer on timeout. Once too old, acknowledge its outcome via the receipt or end the retry with direct guidance. Audio mix can respond locally to listen-in, but trait/mood state is never optimistically written.

Name/settings conflicts can use explicit resource-version checks because they are user metadata, not personality. Reject stale writes with current metadata and allow a deliberate resubmit. Never put personality in the same editable blob as names. Simultaneous adoption checks age eligibility and count in one transaction. Simultaneous offers to a bird reserve one cooldown and return the existing/current eligibility for the other request.

If authentication expires or a device is revoked, terminate interaction submission, clear private page state, and show “Your session timed out. Sign in again to keep watching.” An offline owner can retain the already-loaded scene only as an explicitly stale local presentation for a short grace period; after the two-minute schedule horizon, no new semantic behavior is invented. Private snapshots are kept in memory, not a persistent cross-device offline store.

### Convergence checks

A phone and laptop at the same `scene_revision` must receive identical domain projection bytes other than transport/time headers; their viewport, audio permission, and local mix may differ. Same-bird personality updates are additive and ordered once on the primary. No merge UI, last-write-wins vector resolution, client-to-client channel, or reconstructed event-history vector is necessary. A browser must be able to discard all its memory, sign in, and render the same bird identities and current moods from the server.

## 9. Frontend scene and interaction rendering

### Scene composition

Use an SVG viewBox for the horizontal scene with layered sky/foliage, back/middle/front perch geometry, seven bounded bird rigs, and a small foreground ornament pool. Bird rigs use compact vector paths and a few articulated transforms for head, body, wings, tail, and feather detail. Pre-render reusable shapes where beneficial; avoid per-frame path regeneration. Keep scene DOM below approximately 800 nodes at seven birds and ornaments below 12 concurrent elements. These are internal performance ceilings, not visible counters.

The renderer pipeline is:

1. Validate a versioned scene envelope and resolve local assets by immutable species/grammar version.
2. Map server perch slots into responsive safe coordinates.
3. Evaluate current actions and trajectories at aligned scene time, including already-progressing phases.
4. Apply posture-specific micro-motion, lighting, layer order, and ornaments.
5. Feed current semantic facts to audio, captions, and narration without re-reading a second state store.

One `requestAnimationFrame` loop updates imperative transforms and only the needed lighting values. Reactivity is limited to actual scene revisions and panel state. CSS/layout reads do not occur inside the hot loop. At 60Hz, rendering is interpolation of existing plans rather than a stochastic domain update. Server-specified micro-motion programs can contain small breathing/scanning oscillators and seeded nonrepeating variations; these do not generate a new mood or interaction history.

Mood has visible specificity: wary scans from deeper perches; content preens and shifts weight; curious tilts toward a call or item; drowsy lowers and fluffs; alert makes small quicker orienting motions. Feather detail and final color change slowly with persistent plumage expression. Avoid a generic identical idle loop on every bird. Stable bird-specific timings and posture programs must remain recognizable even when two birds share species.

### Responsive layout

Size the scene to available viewport height below the thin bar, accounting for mobile browser chrome, safe areas, and orientation changes. Fit the complete logical scene in one screen with no page scrolling, panning, zooming, or bird cropping. In narrow portrait view, compress gaps and use staggered safe slots within the three depth zones; preserve bird aspect ratios and keep every bird inside a padded safe rectangle. A bird's entire trajectory must also stay within that rectangle, not merely its destination.

Target 320 CSS pixels as the narrow supported scene width, and test landscape phones and browser zoom to 200%. Settings and notebook panels can scroll internally; the aviary itself cannot. At extreme text zoom, prioritize readable system panels and a fitted scene over clipping controls. Reflow layout after `ResizeObserver` changes outside the hot loop, preserve normalized server positions, and remap focus/caption anchors without resetting actions.

Visual birds may be small at seven on a phone, but invisible hit areas must remain usable, generally at least 44 by 44 CSS pixels where space allows. Resolve overlapping hit regions deterministically by depth and nearest center; arrows always provide unambiguous access. Do not place all seven directly atop one another or let resized trajectories leave the scene.

### Loading and first adoption

SSR evaluates the current pose for the inline scene; the interactive renderer continues its phase. There is no entry sequence for a returning account, no fade from static, and no spinner. On unusually delayed state, show the quiet sky field with faint bounded ambient cues. Present a matter-of-fact retry surface only if the state cannot be recovered, outside the scene.

The newly created account has an explicit one-time adoption state: two system-selected birds are introduced and named with defaults available. While initial adoption commits, render the quiet field. Then the starter birds have soft server-authored arrival flights; this is the specified onboarding exception to the returning-scene rule. In reduced motion, cross-fade into their perched poses. Persist onboarding completion so reload or a second browser cannot replay the empty-aviary sequence. Later accepted adoptions arrive once and keep their new UUID indefinitely.

Additional opportunities are discovered only through a quiet account/bird-management surface when the user opens it, or the relevant deliberate adoption flow. No badge, earned-unlock message, species catalog, reward toast, adoption-count label, or “come back in 90 days” timer exists. Declining/deferring does not affect moods or traits.

### Chrome and interaction behavior

The thin top bar returns on cursor movement, touch interaction, or keyboard activity. Begin its fade after about four seconds of cursor stillness; never fade a keyboard-focused control or an open menu. Use low opacity only for unfocused decorative icons; every visible text label and active focus state still meets contrast. Focus reveals the bar before the user needs to identify a control.

Birds have semantic controls without visible labels/icons/tooltips in the scene. Clicking/tapping a bird toggles listen-in. Clicking empty space disengages. A different bird hands attention over with a smooth audio ramp. Entering the bird group by Tab focuses the first bird; arrow keys use stable bird order rather than a constantly reordering moving position. Leaving the group ends listen-in. Focus outlines are a necessary interaction indicator, not a permanent scene overlay. The naming/settings interface lives in the account panel.

The offer/actions popover contains seed, song fragment, still pool, and settle. Song fragments come from a small finite motif library and are themselves synthesized softly; no recording upload/library editor exists. A selected offer appears at the appropriate server-planned point in the scene, with a curious approach, hesitant approach, bath, watch, or refusal that follows its accepted outcome. A cooldown does not display a countdown or error toast; the action can show quiet availability guidance in its popover. Choosing another bird never bypasses the receiving bird's own cooldown.

### Reduced-motion rendering

Effective reduced motion is OS preference OR the account's explicit opt-in. Do not require an OS user to discover an app toggle. Replace micro-motion with slowly cross-fading still pose families, initially three-to-six-second blends with multi-second holds; replace flights with cross-fades between perch poses. Remove leaves, falling feathers, parallax, and animated rain streaks. Keep slow ambient color/weather changes, with lighting transitions lengthened to roughly eight seconds. Avoid flashing or rapidly overlapping fades.

Use the same server actions, events, mood, calls, drift, offers, and notebook. Reduced-motion birds are expressive through chosen pose families and prose/calls; they are not frozen default silhouettes. A runtime preference change switches representation from the current action phase without resetting mood or replaying a greeting.

## 10. Procedural audio and call captions

### Grammar and recognizable identity

Provide about six species-specific grammar packages. Each contains motif rules, syllable contour families, spacing constraints, timbral shaping, and permissible phrase transformations. A bird's stable fingerprint selects an anchor pitch neighborhood, interval tendencies, envelope character, and timbral subset within the species. Mood can change pace, softness, phrase length, and spacing within recognizable limits; personality changes call opportunity frequency and response likelihood over weeks. Neither renaming nor an asset release changes the fingerprint.

An initial six-species working set can use warbler-like, wren-like, finch-like, robin-like, tit-like, and nightjar-like silhouettes/signatures, with the last supplying the night-active role. These are a coherent art/audio direction, not claims of biological sound fidelity. Give each package at least three composable motif families; favor contrasting rises, separated notes, low repeated pulses, and trills rather than six differently pitched versions of the same phrase.

A semantic call event expands with a deterministic PRNG into a bounded score: note onsets, lengths, pitch trajectories, harmonics/noise texture, amplitude envelopes, and gaps. Start with phrase lengths around 0.15–1.5 seconds, smoothly bounded syllable envelopes, per-event timing variation around 15%, and pitch-center variation around 7% of the fingerprint anchor. Tune timbre/contours by listening, not by assuming a sine-wave beep sounds like a bird. Daytime ambient opportunity rates can initially span roughly 0.2–1.5 calls per bird per minute, with lower non-night-active night rates and explicit refractory gaps; chorus-density limits take precedence over independent opportunities. These are synthesis/calibration starting points, not exposed settings or fixed repeated recordings. Continuous bounded variation changes timing and pitch ornamentation; no repeated fixed waveform or prerecorded loop is selected. The server uses the same pure grammar expansion library without producing audio to reserve durations and response opportunities. The client produces waveforms only. Pin grammar versions on existing birds, with tested compatible migrations; do not regenerate signatures to get newer content.

The score is the common source for audio and caption descriptions. Caption generation inspects the actual expanded rise/fall, note count, trill repetitions, softness, spacing, and perch context. It produces short naturalist descriptions, not a fixed caption per species. A silent call in muted/unavailable audio mode still has an intended score and truthful caption. Do not claim a loud call when the actual rendered envelope is soft.

### Runtime graph

Use one AudioContext per visible page, created/resumed only in a permitted browser interaction. Feature-detect WebAudio and AudioWorklet. Where AudioWorklet is available, synthesize into a fixed voice pool without per-call object churn. Where only standard WebAudio is available, use procedural oscillator/noise nodes with explicit bounded lifetimes, disconnecting and releasing each completed node. This remains procedural synthesis, not a recorded fallback.

Each bird has a gain/pan bus feeding a chorus/master bus with conservative limiting and headroom. Bound voices initially to 24 and at most three simultaneously prominent birds; larger choruses can have quiet secondary syllables rather than seven full-volume phrases at once. Preallocate note descriptors, pooled scratch buffers, and caption objects. Keep roughly 100–200ms of scheduling lookahead with a ~25ms control interval while visible. Translate aligned server event times to `AudioContext.currentTime`, skip expired calls after a gap, and never emit all overdue calls on resume.

Avoid phase-coherent identical oscillators across birds by distinct fingerprints/variation and envelopes. Chorus mixing should preserve space and headroom instead of treating calls as stacked tracks. Bird position subtly affects pan, but mono output remains intelligible. Song-fragment offers feed their own quiet motif bus and may elicit bird responses; they do not replace call signatures.

### Listen-in mix and decay

Normal per-bird gain is an audible ambient baseline. On listen-in, ramp the attended bird toward a modest foreground level over roughly 1.5 seconds and ramp others to about 0.25–0.4 of their ambient gain, with a nonzero floor. On disengagement, ramp all buses back over about two seconds. Switching bird starts smooth opposing ramps from current values, never a hard cut. Holding attention maintains the local mix; its departure decays to ambient. Focus loss, empty-space click, toggle, Escape, settle, or leaving the page invokes the same safe return/quieting path.

Settle lowers the overall scene mix smoothly and biases future response presentation toward quietness. Explicit user mute and hidden-page suspension can silence the whole output; these are separate from listen-in's no-silencing rule. Muting contributes no negative personality drift. Preserve recent user audio preference, but browser permission remains local and must be reacquired where required.

### Failure and permission behavior

Attempt sound on return only if already allowed. A browser-blocked resume, missing WebAudio, context suspension/error, or hardware failure leaves the aviary visually alive, enables captions by default, and offers a clear sound status in accessibility settings. First permitted pointer/key gesture can resume the context and join the current schedule without restarting the scene or replaying old calls. If AudioWorklet fails but basic WebAudio works, use bounded native synthesis. If synthesis is unavailable, use silence plus captions; do not fetch samples.

Caption overlays are the deliberate accessibility exception to the scene's no-label rule. They are short, positioned near their calling bird, collision-aware, and set on a high-contrast translucent backing chosen for day/night/weather. Fade with the call; in reduced motion use long gentle fades. Bound displayed captions and combine overlapping chorus descriptions when necessary without losing which bird called. Captions must never encode trait values.

## 11. Field notebook and accessible prose

### Sparse notebook generation

Use a deterministic, curated naturalist phrase engine driven by private semantic facts, not an external language model reading interaction histories. That avoids third-party disclosure and gives editorial control. Each candidate has a fact predicate, salience, recurrence key, and time window. Examples: a genuinely unusual first-greeter ordering relative to this aviary's recent greetings; a sustained quiet preening interval; a bird's first observed use of the front perch under its existing history; or a distinctive weather/call response.

Start with a minimum 48-hour ordinary-entry spacing, a target near one entry every two-to-four days for regular visits, and a noteworthy exception budget capped at one per day. Suppress repeated facts and generic “session began” notes. Highly active users do not receive an entry per session. Quiet periods can be noted when supported by scheduled actions, not invented because the user was away. A comparison such as first this week requires the private greeting-order summary; it never becomes “the user visited every day this week.”

Compose lowercase, present-tense, specific observations with real bird names and appropriate perch/time/weather context. Validate templates and factual predicates in the same transaction as emission; entry IDs/uniqueness keys prevent duplicates after retries. Store names as used at observation time so renaming does not rewrite the historical observer's record. Escape user names, support Unicode, and keep supplied spelling in settings; narration can apply the product's lowercase styling without mutating stored identity.

The notebook is read-only for owners and unavailable to visitors in v1. Internally virtualize long lists with bounded page caches, preserve focus/reading position when paging, and provide an accessible “older observations” action as well as scrolling. Do not archive entries, let virtualization hide old history permanently, or retain DOM/audio references when pages leave the cache. Account deletion is the only user-driven removal of the complete notebook.

### Narration

Maintain one polite live-region narrator, generated from the same projection and semantic events the renderer uses. Idle cadence starts at 45 seconds, within the 30–60-second requirement. A paragraph relates the current birds, perches, calls, and light; it is not a stream of every animation or raw enum label. Include enough specificity to distinguish birds, while limiting a typical update to roughly 30–60 words. No vector values, progress claims, attendance counts, or achievement language are allowed.

Return-greeting, successful offer reaction, and settle have priority over idle prose. Coalesce simultaneous observations, discard obsolete pending paragraphs, and avoid interrupting the user's form/menu reading. Keep at most one idle paragraph and one priority observation pending. Use polite announcements rather than blanket assertive ones; prioritize by replacing stale queued prose and promptly publishing current observations. Narration stays useful when audio is muted or motion is reduced.

Provide explicit narration pause/resume and repeat-current-observation controls in accessibility settings. A pause is a user choice, not a fallback that disables other accessibility. Do not auto-announce every snapshot or caption into the same live region. Call captions are readable near the bird and in an optional recent-call text surface, with narrator coalescing to avoid double-speaking each call.

### Semantic and keyboard surfaces

Expose the scene as a named region with a short description and a roving-tabindex bird group. Each semantic bird control has its name/species and a concise descriptive action label; no mood statistics. Hide purely decorative SVG structure from assistive technology, and expose one bird control per bird rather than hundreds of path nodes. Tab order goes through the four top-bar controls, then the scene group. Arrows navigate birds; Enter engages listen-in; Escape disengages. Shift-Tab exits predictably and ends attention. Do not put an application role on the entire document.

Offer popovers, settings, notebook, adoption naming, and account dialogs have deliberate focus entry, Escape closure, focus return, and modal focus trapping where appropriate. A visible high-contrast focus ring follows the bird and remains legible in all lighting states. Provide an optional configurable modified-key shortcut to open offers, scoped away from text fields and screen-reader browse commands; ordinary keyboard traversal always works without shortcuts.

All system text uses matter-of-fact voice: identity, session failures, unavailable visits, settings, export/deletion, and access instructions. Product prompts, observations, and captions use the naturalist voice. Contrast tokens target at least 4.5:1 for normal text, 3:1 for qualifying large text, and 3:1 for nontext controls/focus against adjacent surfaces. Test actual composited caption backgrounds and faded chrome, not only nominal palette colors. Support forced-colors mode for chrome and semantic focus, text zoom, screen-reader reading, and touch without hover.

## 12. Security, account lifecycle, and privacy enforcement

### Private simulation boundary

Keep the simulation database inaccessible to analytics accounts and export/metrics collectors. The worker can read private events only to evolve that aviary and create its notebook. Operational metrics are emitted from timing/error boundaries with fixed categorical dimensions; no pipeline scans birds to derive “average boldness,” “popular offers,” “most listened bird,” or attendance segments. Do not install session replay, click tracking, behavior heatmaps, third-party product analytics, or generative services that receive notebook/state/event payloads.

Separate identity-address access, simulation access, export access, and aggregate monitoring with database roles and network access rules. Error logs can carry an account UUID only when necessary for a private support/security failure record; keep those records separate from aggregate metrics, short-lived, owner-scoped, and deletable. An opaque request ID is the default log correlation key. No logger receives request bodies or state objects by default. Names and email are PII and must not appear in exceptions or trace attributes.

Mail delivery is a narrowly scoped transactional exception to the no-third-party-interaction-sharing rule: the delivery adapter receives only the necessary recipient and approved authentication/invitation/export-link template, never bird state or interaction history. Prefer an isolated mail relay with no marketing analytics, disable open/click tracking, and document the email processor plainly. Invitation emails are host-requested sharing messages, not a return-engagement campaign.

Use high-entropy link/cookie tokens, digest-only token storage, TLS, encrypted storage, strict CSP, output escaping, origin checks, and per-route rate limits. Verify ownership rather than trusting a UUID's unguessability. The private scene projection must not be accidentally cached into another account's HTML. Invitations grant a narrow capability, not the host's account session. Visitor email and host email are never serialized into a visit's scene response.

### Export consistency

Generate the requested JSON from a repeatable-read snapshot of account settings, current birds/names/vectors/moods, and all notebook entries. Include a format version and export time; document that raw interaction history and authentication secrets are excluded. Stream large notebooks to private storage without holding the whole file in memory. The export does not recompute personality. The completion email has only a download link, not attachment state or prose about the birds.

Require the currently verified owner identity to redeem an export link. A pending email change does not silently redirect export delivery. Use a 24-hour link expiry and a short-lived redeemed download ticket; revoke access on account deletion or session compromise. Export files and pending delivery jobs are within deletion scope.

### Deletion and recovery

On deletion initiation, transactionally mark the account deleting and record `hard_delete_at = requested_at + 30 days`. Stop simulation and new interaction/adoption writes; revoke outstanding and active visitor access immediately. Preserve the last canonical bird records for recovery, and show a plain recovery option whenever the owner signs in. Recovery before the deadline clears deleting status, restores scheduling, and keeps every UUID/vector/notebook entry. Resume the fast clock from current time with preserved slow state; do not treat a deletion pause as negative drift or invent presence during it.

At hard deletion, an idempotent purge job removes device sessions and challenges, invitations and visit history owned by the account, raw events and accumulators, bird records, notebook/summary records, aviary, settings, exports, account-linked support/security logs, mail/jobs, address records no longer referenced, and account keys. Revoke access first so a purge retry cannot expose half-deleted data. If the account was also a visitor, remove or anonymize that account identity in another host's visit history; preserve only the other host's non-identifying observation of access where necessary.

Account-linked records in every recovery path must be included in hard-deletion work. Choose backup infrastructure that permits retirement of affected backup/WAL chains and creation of sanitized baselines; do not claim that deleting live rows erases an immutable backup. Prepare replacement recovery baselines before the deadline, exclude due accounts, and retire recovery material containing their records at the hard-delete boundary. This is an operational cost of the stated deletion promise. A backup product that cannot support the policy is unsuitable for v1. Keep a short, UUID-only purge control ledger outside restored application data until older media have been retired; prevent any restore from resurrecting an account. Aggregate metrics carry no account dimension and need no per-account historical deletion.

Expose job failure only as a clear account-system status and an operational alarm. Retry until every storage class is cleared; validate deletion by enumerating owned FK/resource references, not just deleting the top-level row. No deletion success email or notification is needed.

## 13. Performance budgets and operational observability

### Budgets and measurement definitions

| Area | Budget / measurement |
| --- | --- |
| Initial JavaScript | Absolute release gate below 2MB gzip across all scripts required at first paint; engineering target at or below 250KB gzip. The critical pre-first-bird controller should be roughly 10KB gzip or less. Lazy settings/notebook/invitation/export code must remain outside this path. |
| Time to first bird | `first_visible_bird_paint - navigation_start < 500ms` on a calibrated mid-tier mobile reference device over the defined 4G profile. Test both cold and warm navigation in launch geographies; the first bird must be an actual named account bird, not a placeholder. Target p95 below 500ms in repeated lab runs and track production timing distribution without changing the product requirement. |
| Critical transfer | Aim below 45KB compressed before the first bird can paint, including HTML, bootstrap projection, bird geometry, and critical controller. No fonts or audio downloads block it. |
| Delivery breakdown | Aim navigation-to-first-byte below 300ms, transfer of the first-bird chunk below 80ms, parse/pose below 35ms, and first composited paint below 35ms. Keep remaining headroom for scheduling variance. Profile the full path; these are allocation targets, not claimed measurements. |
| Snapshot | At seven birds, below 12KB compressed for ordinary state. Visible pull every ~15s; hidden owner pages do not routine-poll. API target p95 below 150ms inside the selected home region. |
| Frames | 60fps for idle motion on a five-year-old mid-range laptop for 30 minutes, at seven birds. Aim scene work below 8ms per frame and p95 total frame time within 16.7ms under the reference idle workload. Record missed-frame distribution as well as averages. |
| Memory | No sustained growth over 30 minutes. Bounded rigs/particles/voices/listeners/seen-event rings/notebook cache; post-GC retained heap returns to the warmed baseline within measurement noise. |
| Simulation | Tick computation/commit p99 alarms above 5 seconds. Target ordinary per-aviary steps well below that, initially below 250ms p99 at seven birds. Separately monitor scheduler lateness/backlog; warn at two missed cadences. |
| Audio | No audible clicks on ramps, no clipping in the seven-bird workload, bounded voices, no stale-call burst after suspension, and no growth in AudioContext/node/worklet resources. |

Define the lab profile before implementation gates: a real mid-tier mobile device with approximately 4GB memory, 4G shaping around 10Mbps down/80ms RTT, and a five-year-old mid-range laptop class with integrated graphics. Keep exact device/OS/browser and network settings in future test configuration so comparisons are reproducible. Automated CPU throttling is a useful companion, not a substitute for the actual supported devices and Safari path. Test server cold paths as well as asset caches.

The 2MB limit is a ceiling, not enough by itself to guarantee 500ms. The SVG first response and small snapshot are essential. If measured first-bird paint is late, eliminate critical network dependencies before cutting accessible behavior or substituting a fake bird. Low-powered devices can reduce ornament counts and rendering pixel cost; they cannot drop birds, call captions, or the reduced-motion experience.

### Memory and cleanup discipline

Reuse pose/score/particle buffers and stop per-frame closures or growing arrays. Every completed native audio node is disconnected and released. Seen-call IDs use a ring with a fixed horizon. Keep at most a few notebook pages resident; remove old node/listener references while preserving server pagination access. Opening/closing panels, switching birds, and hiding/showing the tab must not create additional persistent timers or audio contexts.

CI runs a 30-minute seven-bird soak after a short warm-up, with calls, repeated offers, listen switches, panel opening, notebook paging, and hidden/resume cycles. Compare forced-GC retained heaps and resource counts at multiple checkpoints; fail on retained-object accumulation or a positive retained-memory trend. Allow only measured tool/runtime noise, not an annualized leak allowance. Inspect browser-process/audio/GPU resource plateaus as well as JS heap. A renderer that runs cleanly for one minute does not pass.

### Day-one observability

Instrument only aggregate operational categories:

- Request counts, latency buckets, safe error-code counts, and database contention/connection health.
- Tick compute/commit latency, scheduler lag, retries, backlog depth, integrity-error counts, and private job queue latency. Never include a vector, bird name, species-dependent drift outcome, or interaction payload.
- Navigation/first-bird timing buckets, frame timing/missed-frame buckets, memory/resource error counts where supported, and audio-context/worklet error counts.
- Anonymous, coarse session-duration histograms as permitted by the PRD, with no account/device/page ID dimension or raw presence timeline.
- Synthetic browser probes from common launch geographies, testing load, visible motion, caption/audio-permission paths, and state availability using dedicated synthetic accounts only.

RUM uses an allowlisted schema with coarse build, browser-family, region, timing bucket, and safe error category. Drop IP and unique identifiers before durable metric storage; do not emit full URLs, referrers, request bodies, input events, invite recipients, or trace breadcrumbs from scene interaction. Prebucket frame data client-side instead of uploading a fine-grained personal timeline. Maintain cardinality caps and schema rejection tests. The metrics collector has no credential granting access to the simulation database.

Alert on p99 tick latency above five seconds, scheduler backlog, elevated authentication/state errors, persistent first-bird regression, unusual audio initialization errors, and increased memory-test failures. Support operational debugging through safe request IDs and narrowly scoped account error records, not broad copying of interaction histories. Drift calibration uses synthetic scripted lives, not population averages of private birds. Feature ramps and success are judged by technical health and qualitative fit, not streak maintenance, visit frequency, click-through, or bird “optimization.”

## 14. Verification and release acceptance

Tests are organized around the product's failure modes rather than snapshots that merely repeat implementation details. Use a controllable clock, deterministic PRNG, and synthetic event generator for engine tests. Separate reference simulation correctness from rendering/perceptual evaluation.

| Workstream | Required evidence before release |
| --- | --- |
| Presence | Exhaust all eight visibility/focus/recent-activity combinations; only all-three credits time. Verify expiry at four minutes, blur/hidden boundaries, pointer-only/key-only paths, quiet watching, sleep/clock jumps, synthetic input rejection, mobile focus, duplicate pings, missed unload, settle, and union across two devices. Background tabs open overnight produce zero extra credit. |
| Drift | Thirty/90-day synthetic histories: regular 15-minute visits, irregular visits, one binge, rapid offers, muted use, no activity, simultaneous pages, and two-week absence. Traits never decrease; a session cannot visibly change them; week-one change is measurable; week-three visual differences are reviewed blind. Tests preserve residual fixed-point deltas and stored vectors on upgrades. |
| Persistence | Rename, schema migration, fresh sign-in, changed device, asset/grammar release, restore, and worker outage all preserve bird IDs and vectors. Missing vectors are rejected, never reseeded. Recovery uses stored records, not history reconstruction. |
| Exactly-once processing | Fault injection before/after event insert, cooldown reservation, trait write, notebook write, cursor update, and commit; tick lease expiry and job redelivery. End state matches a single execution. Concurrent worker/API transactions cannot lose deltas or reorder accepted facts. |
| Mood and continuity | Login does not reset mood. Circadian/DST/timezone changes, rain/wind decay, bird alarm limits, and quiet absence preserve the intended baseline. Open midway through flight/preen and confirm the initial pose is mid-action. No synchronous greeting chorus. |
| Interactions | Greeting selection/variation/absence signal and one-to-two-second timing; listen-in all exit routes and nonzero other-bird gain; three-minute per-bird cooldown across devices; refusals; song/pool responses; settle five-second undo and later reengagement. No timeout retry duplicates an offer. |
| Sync | Phone/laptop concurrent presence/listens/offers/renames/adoption; out-of-order snapshot responses; frame gaps; authentication expiry; network retries. Same revision produces identical shared world, with independent local mix. No endpoint accepts an absolute vector. |
| Audio | Generated-score tests verify bounded pitch/envelope, runtime variation, no identical repeated full scores in a long seeded sample, stable per-bird anchor, distinct repeated-species fingerprints, headroom, smooth gain ramps, and caption/score agreement. Synthetic listening sessions at two/four/seven birds review recognizable identity and uncanniness. |
| Accessibility | Manual screen-reader testing with VoiceOver/Safari, NVDA with a supported desktop browser, and mobile screen-reader coverage; keyboard-only entire journey; narration queue/cadence, call captions, OS/manual reduced motion, focus during chrome fade, forced colors, 200% text zoom, and contrast through all scene palettes. Automated checks complement, not replace, these reviews. |
| Privacy/security | Cross-account UUID access and nested bird access denied; visitor mutations denied; token expiry/replay and scanner-safe consume; revoked sessions/invites checked even on 304; no payloads/PII/vectors in captured logs/RUM; analytics role denied simulation reads; export ownership; deletion and sanitized-backup restore cannot resurrect records. |
| Notebook | Fact predicates support every sentence; duplicate suppression and scarcity under intense synthetic use; rename historical continuity; no attendance commentary/stat prose; indefinite retrieval with bounded client memory. |
| Performance | Gzip-size gate, cold/warm first-bird tests, actual-device 60fps/30-minute soak, no-memory-growth CI gate, seven-bird resize/orientation tests, WebAudio denied/unavailable/worklet-failure paths, supported real browsers, and unsupported-browser matter-of-fact surface. |

For drift perceptibility and call recognition, use known synthetic birds and scripted histories in blind design/listening comparisons. Gather qualitative feedback without collecting or combining a real user's per-bird interaction history. If the three-week difference is imperceptible, tune projection/coefficient families together, then rerun the complete calibration suite. Do not accelerate the whole population as an undocumented live experiment.

Add editorial/UX acceptance to functional review: no “Welcome back,” no days-away/attendance surface, no celebration on adoption, no hunger/distress implication, no mood/trait tooltip, no settings badge for a visit, and correct naturalist/system voice by surface. A technically correct implementation can still fail the affective contract, so these checks are explicit release gates.

## 15. Delivery sequence and rollout

Use workstreams with clear handoff artifacts. Accessibility, procedural audio, and canonical persistence begin alongside the scene; they are not post-launch patches. Do not promise a calendar date before the performance and three-week calibration work has evidence.

### Milestone A — foundations and executable contracts

Backend/simulation leads define schema migrations, module permissions, event/projection versions, fixed-point vector storage, time/PRNG abstractions, and tick transactions. Frontend/audio/accessibility leads define the shared semantic event contract, species rigs, grammar format, prose engine, and visual/accessibility tokens. Produce a synthetic fixture aviary at several times of day and at different personality ages.

Exit: one synthetic bird can be restored with the same identity/vector; clients cannot mutate personality; the early scene is already mid-action; the projection is within size budget; telemetry accepts only safe fields. Establish CI device/network profiles, fault-injection harness, and first-bird measurement before feature growth.

### Milestone B — a complete two-bird relationship

Implement two starters, magic-link/device sessions, presence conjunction/union, tick-owned drift/mood, continuous rendering, procedural calls, listen-in ramps, and greeting activation. In parallel, ship keyboard controls, designed reduced motion, captions, and slow narration against the same two birds. Do not add notebook/social complexity until the birds feel specific and the 500ms path is credible.

Exit: two browsers observe one canonical aviary; background/suspension gives no false presence; no greeting toast; procedural identity survives variation; full accessible journey works; first-bird and 30-minute rendering budgets pass.

### Milestone C — offers, settle, notebook, and account lifecycle

Complete all three offer types, cross-device cooldowns, refusal outcomes, settle/undo, sparse notebook generation/history, naming, settings/timezone, session revocation, verified email changes, coherent export, and 30-day deletion/recovery/hard purge. Start simulated one-week/three-week/months-long drift evaluation and backup-restore exercises.

Exit: all accepted outcomes process once through worker failures; a renamed bird retains its identity; notebook factuality/sparsity passes; export contains the explicit required fields only through the private download; restored deleted accounts remain unavailable.

### Milestone D — six species, age adoption, and quiet visits

Expand the coherent species pool, including the night-active signature, and validate repeated-species individual fingerprints. Implement age-based additional adoption and enforce seven under races. Build email invitations, one-time redemption, read-only visitor projections, private visit log, off-by-default in-settings notice preference, and revocation/lease expiry.

Exit: visitor attendance never enters simulation events; visitor endpoints cannot interact; hosts and visitors see the same world without co-presence; revoked access ends at the next pull; unused links expire at 30 days; seven birds fit and remain recognizable.

### Milestone E — launch qualification and operational ramp

Run full synthetic fault/load/privacy suites and manual supported-browser/accessibility reviews. Complete at least one full three-week reference drift trajectory (accelerated synthetic time plus qualitative review, with a real-time design observation period where feasible). Load-test tick capacity at projected account counts, including inactive accounts and a one-day catch-up backlog. Rehearse rollback and deletion/restore before public access.

Launch with two birds for all new accounts. Ramping birds per aviary is a quality/capacity ramp, not a reward mechanism: test two, four, then seven in synthetic and consenting internal fixtures; existing real accounts remain subject to the age thresholds, never engagement metrics. Do not accelerate adoption because someone visits frequently. Higher-count qualification must be complete before any actual account becomes age-eligible for that count. If a higher-count deployment gate is delayed, the user's existing birds remain intact; do not lower count or remove birds.

Operationally ramp account access through internal, limited invite-only, and public cohorts, for example 1%, 5%, 25%, then full capacity after stable health windows. Cohort assignment uses a deterministic synthetic account UUID hash, not bird state or behavior. Each cohort receives complete accessibility and privacy features. Invitations are available by deliberate host request, not promoted at onboarding.

Watch day-one load/frame/audio/tick health and aggregate error rates. Stop a ramp on budget regressions, integrity faults, privacy failures, or accessible-path failures. Roll back application/grammar code with backward-compatible schema readers, retain existing bird seeds/IDs/vectors, and continue ticking with the last safe engine version. Never “reset the aviaries” as a recovery strategy. Keep a last-known-good private projection for safe presentation while mutation is repaired; do not label it fresh.

Support runbooks cover tick backlog, partial outages, invalid vector detection, repeated mail delivery failures, invitation revocation, export leakage, missing accessibility behavior, audio errors, and hard-deletion failure. Owner support must use UUID/request-ID references and minimum authorized private access, not ask users to expose personality numbers in a debugging panel.

## 16. Main risks and mitigation owners

| Risk | Likely failure / consequence | Mitigation, owner, and release signal |
| --- | --- | --- |
| Drift too fast or too slow | Clicks visibly change a bird, or three weeks feels static | Simulation + design: versioned low-pass parameters, caps, synthetic week/three-week comparisons, and blind pose/call review. Require both instrumented and perceptual targets; never use private population drift analytics. |
| Drift saturation/identity convergence | Months of positive drift makes all birds feel alike | Simulation + audio/design: retain stable fingerprints, species/seed-specific motion character, diminishing headroom, and 90/365-day synthetic review. Tune future gains without subtracting existing traits. |
| False presence | Background/suspended/multiple tabs amplify drift | Client + backend: strict three-signal state machine, bounded monotonic intervals, account union, no lease-derived credit, and exhaustive truth-table/suspension tests. |
| Lost or double personality deltas | Retried tick, stale client, or concurrent device corrupts the relationship | Backend: single writer/lock, additive fixed-point deltas, atomic cursor/state commit, idempotent event receipts, grants, crash injection, and restore validation. |
| Slow interaction acknowledgment | Greeting/offer waits until minute tick | API + simulation: persisted immediate response plans from committed state, with tick-owned persistent effects. Measure full navigation greeting and avoid local fabricated state. |
| Uncanny or indistinguishable calls | Mechanical repeat, harsh chorus, same-species confusion | Audio: constrained procedural variation, stable anchors, bounded chorus density, headroom, blind recognition at seven birds, and signature-preserving grammar releases. Silence/captions on audio failure. |
| Autoplay/runtime audio denial | New visitors hear nothing or the product blocks entry | Frontend + audio: gesture-safe resume, visible/captioned/narrated greeting, silent full scene, and clear sound controls; test Safari/mobile and denied contexts. |
| Accessibility loses the affective core | Narration floods, reduced motion freezes, focus disappears | Accessibility + design: first-release prose/cross-fade surfaces, queue coalescing, fade-safe focus, manual AT reviews each major scene change, and shared-event contract tests. |
| First-bird target fails on real 4G | Loading becomes the experience | Frontend + infrastructure: small SSR SVG/inline state, tiny controller, private edge delivery near account home, no blocking audio/fonts/panels, and cold real-device geography gates. |
| Tick cost grows with dormant accounts | Continuous simulation backlog degrades continuity | Infrastructure + simulation: minute staggering, bounded jobs, private indexed events, worker scaling, inactive-account load tests, and validated quiet-span optimization only if necessary. No client-side or return-only replacement tick. |
| Memory/audio leaks | A calm half-hour tab gets slower and battery-heavy | Frontend + audio: bounded pools, cleanup, hidden suspend, resource-count assertions, and mandatory 30-minute soak CI. |
| Private data escapes observability/cache/mail | Relationship history becomes analytics or another user's state | Security + infrastructure: separate roles, allowlist schemas, no payload logs or shared personalized cache, processor boundaries, secret-safe URLs, and adversarial leakage tests. |
| Invitation races or slow revocation | Unintended continued access or visitor mutation | Backend + frontend: atomic redeem/revoke, capability scopes, authorization on every GET/304, 15-second polls and short lease fail-closed behavior. |
| Deletion restores from backup | A supposedly erased account returns | Account/infrastructure: sanitized baseline/retired recovery chains, purge ledger on restores, deadline-aware jobs, export/mail cleanup, and deletion restoration exercises. |
| UI gradually adds announcement/gamification | The birds become decorative around product counters | Product + design review: explicit negative acceptance tests, no underlying leaderboard/engagement pipeline, no general-purpose toast system on the scene, and editorial voice ownership. |

The launch decision requires the canonical-state and privacy invariants, first-bird/runtime budgets, procedural-call quality, and complete accessible surfaces to pass together. The product should be small enough to watch at a glance, dependable enough to leave for weeks, and specific enough that the same birds are recognizable when the user returns.
