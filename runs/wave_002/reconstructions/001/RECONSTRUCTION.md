## System-level intent

- Continuing relationship over session mechanics. The plan repeatedly centers an aviary that "feels as though it has continued between visits": the first normal frame is "an already-running scene," the worker processes "accounts without connected clients," absence cannot make a user "lose a trait, a bird, a name, or accumulated drift," and completion means preserving "the birds the user has come to know."

- Server-authored canonical truth. The tick is "the only transition function"; the client is limited to "observation reporting and presentation" and cannot "advance mood, decide drift, run a second canonical simulation, or submit trait values." Server-authored action tracks, call intents, weather intervals, command cues, and snapshot projections all reinforce one authority.

- Identity preservation. The plan treats each bird's "immutable UUID and call identity" as core. Renaming, migrations, device changes, grammar changes, backup/restore, rollback, and recovery must not change IDs, call signatures, vectors, or "recognizable signatures."

- Positive, non-punitive growth. Persisted personality only receives "nonnegative deltas." Absence, mute, audio permission, deletion recovery, tab-close, and settle produce "no negative personality signal." The product excludes hunger, illness, death, suffering, maintenance schedules, rewards, streaks, and punitive offer states.

- Quiet naturalist product voice instead of gamification. The plan forbids "achievements, scores, streaks, counters," numeric panels, mood meters, unread badges, attendance calendars, and success toasts for routine interactions. It favors "naturalist prose," "matter-of-fact" system surfaces, "lowercase, present-tense" observations, and a restrained scene with controls in the top bar.

- Sensory life must be procedural and recognizable. Calls are "procedurally synthesized" with no recorded loops or fallback tracks. Species and individual signatures use grammars, seeds, pitch neighborhoods, timbral envelopes, motif relations, and bounded variation so variation does not erase identity.

- Accessibility ships with the primary experience. The plan says "Accessibility ships with the primary experience" and requires narration, keyboard navigation, reduced motion, captions, contrast, semantic surfaces, and human accessibility review. Reduced motion keeps "aliveness" rather than becoming an exemption from the experience.

- Privacy by structure and minimization. Operational telemetry "cannot contain bird state or owner interaction history." Email is kept out of URLs, keys, partitions, traces, and job payloads. Metrics/log schemas are allowlisted; analytics roles cannot read simulation storage; no ranking, training, recommendation, replay, or production drift dashboard exists.

- Honest ordinary-browser presence, not surveillance. Presence requires a visible document, focused window, and recent pointer/key activity, but the plan says the goal is "correct ordinary browser behavior, not surveillance or anti-cheat machinery." Presence remains private simulation input and is never displayed as a streak, duration, or visits calendar.

- Deliberate, revocable, read-only sharing. Visits are individually issued and read-only. Visitors receive the same canonical birds, mood, positions, weather, and day/night state but cannot reach any simulation-writing path. Revocation is authoritative on each pull, while the private visit log preserves "historical transparency."

- Sparse long-term memory instead of a feed. The notebook is "sparse" and "indefinitely browsable," with persisted immutable prose. It records specific bird observations, not user compliance, per-session logs, numeric traits, badges, or a calendar.

- Measured quality before expansion. Performance budgets, physical fixtures, synthetic trajectories, human audio/accessibility reviews, privacy-schema scans, and limited release gates appear throughout. Expansion depends on "worker/database headroom and unchanged user-state guarantees," not engagement data.

## Per-feature whys

### Release scope and governing invariants

- Browser-based delivery: NOT RECOVERABLE FROM PLAN

- Single-user aviary and exactly one aviary per account: the plan frames the product as a private relationship with one canonical aviary, reinforced by "multiple/shared/household aviaries" being excluded and by the account-to-aviary uniqueness constraint.

- Two system-assigned starter birds: the plan says initial adoption assigns both starters transactionally and presents them as "meeting these particular birds, not configuring avatars," so the first relationship starts with stable, particular birds rather than a catalog or reroll.

- Up to seven lifetime birds: the cap supports the invariant that "all birds [are] visible" in one horizontal view and that seven remain "visible and audible as individuals." It also bounds adoption, audio, layout, performance, and testing.

- Distinct species pool: the plan gives the pool a purpose of distinctness: "distinct silhouettes, palettes, and procedural call signatures."

- Approximately-six-species count: NOT RECOVERABLE FROM PLAN

- Nightjar-like species active at night: the plan uses it to preserve ordinary nighttime life, where "night leaves most birds settled with the nightjar-like species able to call."

- Email magic-link accounts: generic responses, single-use 15-minute links, non-consuming GET, scanner safety, and no password/SSO scope support quiet account access without identity enumeration.

- Magic-link choice over all other account models: NOT RECOVERABLE FROM PLAN

- Device-session management: sessions have hashed opaque tokens, expiry, revocation, and a coarse user-visible device label so users can manage access without turning user-agent/device identity into "an analytics dimension."

- Email change: the new address is verified before "an atomic switch," old address remains valid until completion, and concurrent changes invalidate earlier pending verification so delivery identity does not become ambiguous.

- Export: the plan calls it a "narrow portability exception" to numeric-vector secrecy, allowed only through an explicitly requested authenticated export file with protected delivery.

- Deletion and recovery: soft deletion gives a clear 30-day recovery period while preserving state; hard deletion removes account-linked data, jobs, diagnostics, and keys so deleted data cannot reappear.

- Continuous server simulation: it makes the aviary continue when no client is open. The plan says not to "use return navigation as the trigger to restart simulation" and says scaling is not a reason to stop absent aviaries.

- Return greetings: greetings express return as bird behavior, "never a toast, banner, modal, counter, or textual welcome," and support recognition through varied primary notices shaped by absence length, mood, and history.

- Idle attention: ordinary ambient behavior is needed so a newly seen bird can be "mid-preen, mid-weight-shift or already partway through a call," making the scene alive before explicit interaction.

- Listen-in: listen-in is a "local auditory focus with owner attention events," letting a user attend to one bird without changing every device's mix or causing abandoned starts to count indefinitely.

- The three offers: offers provide bounded, natural interactions, with seed, still pool, and melodic fragment outcomes that birds may approach, drink, bathe, watch, join, quiet, or ignore. Their inputs are small and capped so they do not become reward farming.

- Settle: settle is a session-end gesture that "gently reduce[s] call activity" and fades light toward evening without becoming full silence, a fourth offer, or a personality penalty.

- Sparse field notebook: sparse observations create long-term memory from "specific facts already needed by that aviary" while avoiding a busy feed, attendance log, numeric trait log, or gamified notebook.

- Age-based adoption: elapsed age, including absence, makes new birds available without a progress bar, countdown, share prompt, visit requirement, payment, or "attendance requirement."

- Renaming: rename changes "only name fields" and has "no state-reset side effect," preserving stable identity while allowing user names.

- Responsive day/night: canonical local time reconciles devices and visits to "a single canonical day/night state" and supports dawn/dusk/night mood tendencies.

- Occasional weather: seeded weather adds short-lived mood/expression and call-density variation without real-world geolocation, weather services, or persistent trait changes.

- Multi-device snapshots: snapshots and revisions let devices converge to current canonical state without vector uploads, stale overwrites, replayed audio, or duplicated presence.

- Read-only visits: visits let another person see the host's canonical birds and scene while visitor credentials cannot create greetings, presence, offers, settle, notebook changes, adoption, drift, or owner-state changes.

- Revocation: revocation must stop further access quickly because a visitor's authorization is rechecked against authoritative storage every pull and the scene/audio are cleared once access is no longer valid.

- Private visit log: the log exists for "historical transparency" about past sharing, but contains sharing metadata only and is not an owner attendance history.

- Narration, keyboard navigation, reduced motion, and call captions: these features make the primary experience available when audio, motion, pointer use, or visual inspection are limited, while avoiding raw state labels and numeric substrate.

### Decisions where the supplied specifications leave room or conflict

- Canonical local time: persisting the first owner browser's IANA timezone, with explicit account-setting changes, "reconciles local time with a single canonical day/night state" and prevents a travelling device from silently changing every device and visit.

- Four top-bar icons with settle inside the offer panel: this keeps exactly the required account/settings, accessibility, notebook, and offer icons while making settle "directly reachable from the top bar without adding a fifth permanent icon" and preserving settle's separate semantics.

- Keyboard focus and Enter listen behavior: focus starts listen-in and Enter is idempotent so a focus event does not "immediately undo" the Enter action; Escape ends listen without moving focus.

- Numeric vectors in export only: the plan chooses the "narrow portability exception" because normal disclosure prohibition and downloadable JSON export conflict literally. Normal APIs, UI, DOM/ARIA, notebook, narration, debug surfaces, and telemetry still exclude vectors.

- Optional visit email: the exception is limited to "one optional transactional email per actual visit start," default off and absent from onboarding, so it does not become push, reminder, badge, or engagement notification.

- Revoked invite removal plus historical log preservation: active/outstanding entries disappear after revocation, while the past visit remains in history to preserve "transparency about past sharing."

- Browser autoplay handling: rendering cannot wait for sound permission, so blocked audio becomes silence and automatic captions until a user gesture enables WebAudio; no autoplay modal interrupts "the first bird."

- Top-bar fading with accessibility: decorative framing may fade, but actionable icons, focused controls, open panels, touch use, errors, and text contrast remain usable because "Accessibility takes precedence."

- Additional adoption ages: the ages are internal configuration with no progress display or attendance requirement, and they allow "a year-old aviary" to have six birds.

- Exact later-adoption eligibility days: NOT RECOVERABLE FROM PLAN

- Muting and personality: muted presence counts the same as audible presence so sound preference affects rendering without creating a "disadvantaged accessible experience."

- Missing visual-design document: the plan chooses to establish palette, pose sheets, focus treatment, and contrast tokens in the first stage so implementation does not depend on "an unprovided design-system file."

### Architecture and boundaries

- TypeScript web application, small server application, and separately deployable simulation worker: this split keeps presentation, server boundaries, and background ticks explicit while allowing the worker to scale independently.

- Canonical transactional storage: the plan uses PostgreSQL as "the canonical transactional store" for ordering, constraints, locks, snapshots, and lifecycle operations.

- PostgreSQL specifically beyond canonical transactional storage: NOT RECOVERABLE FROM PLAN

- Modular monolith: identity, command intake, snapshot projection, simulation, notebook, visits, and lifecycle have explicit interfaces but no separate network services, so boundaries exist before operational complexity.

- Database-backed job/outbox: ticks, transactional mail, exports, and deletion need durable background dispatch and retry before an additional broker is introduced.

- React shell with imperative SVG scene controller: forms, navigation, and panels can live in React while the scene stays outside per-frame React state updates so first-bird rendering and animation remain small and controlled.

- Separate bounded WebAudio scheduler: audio consumes the same immutable presentation model as the renderer while scheduling against the audio clock rather than visual frames.

- Server-only simulation package: excluding personality storage, drift, and behavioral decisions from client builds enforces the client's observation/presentation boundary.

- Inline first scene and safely serialized snapshot in the HTML route: this lets the first response draw an actual current bird quickly while authenticating each request and avoiding shared caching of personal HTML/snapshots.

- Lazy shell, panels, notebook, and audio worklet: the first scene should not wait for hydration, account panels, notebook data, audio context setup, or all species assets.

- Durable short reaction cues: cues allow immediate offer, greeting, and settle responses before the next slow tick without letting the client mutate mood or personality.

- Command cue persisted under the aviary lock: the chosen outcome survives retries, reconnects, other devices, and the next tick; command intake is "not a second model."

- Server-authored 120-second action/call/weather lookahead: clients interpolate and synthesize instructions but do not choose canonical perches, roll mood changes, or originate bird-to-bird responses.

- Client-only ornaments: leaves, feathers, and gentle parallax can add life without behavioral consequences or event writes.

- Engine/grammar versions, revisions, and random-stream counters: versioning permits future tuning, rolling deployments, and rollback without changing identity or rebuilding vectors from logs.

### Data model and retention

- UTC storage with canonical local timezone as input: UTC preserves ordering while local timezone remains a presentation/simulation input for daylight, dawn, and settings.

- UUID identities and integer revisions/sequences: immutable IDs and ordered revisions support bird identity, relationship identity, idempotency, snapshot consistency, and race-free processing.

- Avoiding email in URLs, keys, partitions, traces, and job payloads: this confines email to protected identity/delivery contexts and reduces PII leakage.

- Protected account identity record: email and pending replacement address live only in this record so pending identities can exist without an aviary and identity storage stays isolated.

- Auth challenge token hashes: raw bearer tokens are never stored, limiting replay damage if storage is inspected.

- Owner device session records with coarse labels: users can revoke devices while neither user-agent text nor device identity becomes analytics.

- Aviary record with unique owner account UUID: uniqueness enforces exactly one activated aviary per account.

- Bird record with immutable UUID/signature seed and persisted traits/filter/mood/action state: identity, drift, mood, calls, and migrations stay stable; rename updates only names.

- Minimal interaction event records: sequence, bounded intervals, and typed payloads support simulation input while excluding pointer coordinates, keys, screenshots, and free text.

- Command result/cue records: acknowledged cues survive worker crashes and appear in snapshots on other devices, preserving command response idempotently.

- Presence window records: window state and bounded segments allow account-level union accounting while staying "operational simulation input, not an engagement log."

- Transient simulation inputs: persisting filter state and counters avoids reconstructing vectors from old events at startup.

- Scene timeline records: short-lived tracks provide rendering/tick coverage and are pruned once no longer needed, avoiding a second durable bird history.

- Notebook entry records: immutable naturalist prose and names-at-observation prevent later templates or renames from silently rewriting old observations.

- Adoption opportunity records: unique aviary/ordinal rows prevent duplicate adoption across devices; dismissing does not expire the opportunity or reroll its species.

- Invitation records without recipient email copy: the invitation points to protected identity rather than duplicating email.

- Visit session records: token redemption creates a scoped render-only session; refreshes reuse that session instead of the one-time email token.

- Visit log records: sharing metadata supports the private visit history without copying visitor email into the log or becoming interaction history.

- Async job/export records: jobs can retry safely, exports remain encrypted and short-lived, and mail payloads refer to identities/templates instead of addresses or interaction bodies.

- Account settings as synchronized preferences plus local capabilities: audio/captions/narration/reduced-motion/timezone/visit-mail preference sync where appropriate, while browser permissions and AudioContext readiness remain local.

- Exact email lookup digest without provider-specific alias rules: lookup and throttling can work without inventing address equivalence that might merge distinct users.

- Event, presence, export, token, log, and deletion retention: the plan retains what is necessary for continuity, recovery, and sparse notebook history while pruning raw simulation events, working presence data, exports, tokens, diagnostics, and account-linked data on defined schedules.

- Separate API and worker database write privileges: command APIs cannot write personality columns, reinforcing that only the worker transition transaction mutates personality.

### API contracts

- Same-origin HTTPS, secure cookies, CSRF/origin protection, and separate visit capability: these preserve owner authority and prevent visit sessions from being upgraded by body fields.

- Token stripping and no-referrer token pages: bearer tokens should not leak through URLs or referrers.

- Redacted infrastructure logging: request bodies, query secrets, names, email, and snapshots are excluded to support the privacy boundary.

- `POST /api/v1/auth/magic-links`: generic responses avoid existing/new identity enumeration, and rate limits reduce abuse without indefinite lockout.

- `POST /api/v1/auth/consume`: atomic token consume and non-consuming GET prevent email scanners or parallel consumes from signing in unexpectedly or exhausting tokens.

- `GET /api/v1/account`: account-management information excludes personality because personality is not a normal account API surface.

- `PATCH /api/v1/account/settings`: version checks prevent stale settings writes and return current permitted fields for reapplication.

- `GET /api/v1/account/sessions` and `DELETE /api/v1/account/sessions/{id}`: device listing/revocation lets an owner manage sessions, and revoking the current device returns to sign-in.

- Email-change endpoints: verification before switch and invalidation of older pending changes avoid racing or sending to an obsolete address.

- `GET /api/v1/aviary/snapshot`: a transactionally consistent projection, revision, clock, and coverage let clients render current canonical state and conditionally pull.

- `POST /api/v1/aviary/resume`: an idempotent visibility/window epoch creates a server-authored return cue without counting presence absent the three conditions.

- `POST /api/v1/aviary/events`: typed, bounded, sequenced events let the server validate presence/listen/settle/undo/close/re-engage inputs and return accepted/duplicate/rejected outcomes.

- `POST /api/v1/aviary/offers`: server-side receiver/outcome/cooldown selection rejects user-supplied mood, traits, arbitrary targets, and positions so offers cannot mutate simulation directly.

- Bird list and rename endpoints: identity/name management is separated from state; names are text, version checked, not uniqueness-forced, and do not reset state.

- Adoption endpoints: only the next age-eligible system-selected opportunity is returned or accepted, keeping adoption capped and idempotent.

- Notebook endpoint: keyset pagination returns persisted prose with no edit/delete/comment endpoint or historical cutoff, supporting an immutable sparse record.

- Invitation endpoints: invitation is an explicit host action to a recipient email with no bulk, discovery, or automatic sharing API.

- Visits list/revocation endpoints: private outstanding/active/history sections and idempotent revocation separate current access from historical transparency.

- Visit redemption endpoint: a one-time invite token becomes a render-only capability and does not greet birds, record presence, or create an owner aviary.

- Visit snapshot endpoint: every pull checks live status and removes owner-only management fields, preventing cached authorization after revocation.

- Export endpoints: explicit request, consistent snapshot, protected email delivery, owner session, single-use capability, no-store streaming, and expiry bound the portability exception.

- Deletion/recovery endpoints: deletion marks immediately, recovery must be explicit before the 30-day deadline, and sign-in alone does not cancel deletion.

- Owner authorization on every simulation-writing request: being authenticated is insufficient; the target account must match to protect cross-account boundaries.

- Idempotency keys and strict discriminated schemas: retries should not create duplicate drift/cues, and unknown state fields are rejected instead of silently becoming authority.

- Snapshot allowlisted projection: presentation DTOs contain birds, moods, anchors, appearance, and call instructions but exclude vectors, filter values, internal trait labels, and drift deltas.

- Error codes and plain system prose: expired sessions, wrong capabilities, stale versions, invalid events, unavailable visits, and cooldowns are explained without success toasts, countdowns, punitive language, or naturalist euphemism.

### Server simulation and exact update semantics

- Stable phase within a 60-second interval: phase distribution spreads work across accounts and supports scheduled processing regardless of connected-device count.

- Indexed due rows with bounded `SKIP LOCKED` leases: multiple workers can share work while serializing each aviary independently so one slow account does not block unrelated accounts.

- Shared aviary row lock for command admission and tick processing: this prevents sequence races where the tick might permanently skip an earlier event.

- One logical-minute transaction: vectors, filter state, mood, random counters, timeline, notebook insertion, cursor, revision, and next due time commit together so failures do not partially advance state.

- Tick retry idempotence: retries see the advanced cursor/revision and avoid applying events twice; unique IDs, notebook fact keys, and adoption ordinals reinforce boundaries.

- Outage catch-up from persisted state: ordered minute steps preserve continuity and avoid using return navigation, history reconstruction, or age-derived replacement as simulation.

- Capacity sizing against absent-account ticks: scheduled load must be measured and provisioned; load is not a reason to stop absent aviaries.

- Trait storage in `[0, 1]` with moderate varied seeds: bounded heterogeneous starts avoid identical starters and saturation before drift begins.

- Presence/listen/offer signal normalization: signals are daily-normalized, capped, and positive so regular presence dominates while listen and offers add smaller bird-specific influence.

- Low-pass filter and additive increment: the causal filter makes change gradual, residual positive input can continue briefly, and traits never decay.

- Synthetic and human calibration: numerical bands are starting configurations; tests and human perception feedback decide whether change is continuous, recognizable, and not abrupt.

- Persistent mood state: mood is distinct from trait vector and has dwell, weights, and hysteresis to avoid abrupt alternation.

- Exact five-mood vocabulary: NOT RECOVERABLE FROM PLAN

- Local dawn/dusk/night mood tendencies: time-of-day changes add ambient rhythm without resetting traits or snapping birds to content.

- Seeded rain and wind: weather adds short-lived expression and call-density variation without geolocation, weather services, or persistent vocal-frequency changes.

- Absence behavior: birds retain traits/signatures and continue ambient activity after time away so absence does not become distrust, lost plumage, or reset mood.

- Server action selection and perch reservation: mood/personality map to bounded behaviors and perches while preventing collision and excluding owner placement commands.

- Immutable individual call signature: the signature preserves identity through mood and drift; mood affects articulation/pauses/density while drift changes frequency/willingness more than recognizable sound.

- Client expansion of versioned call grammar: audio and captions share the same concrete description, preserving deterministic variation without recorded assets.

- Bird-to-bird responses and capped chorus chains: responses make the aviary social while refractory windows and chain caps prevent recursive call explosions.

- Primary and secondary greeter selection: weighted greeter choice and fresh server seed produce varied, legible notices tied to absence, mood, and history.

- Resume idempotence: coalescing visibility flapping prevents one arrival from becoming five greetings, and non-owner reads never create greetings.

### Presence, interactions and multi-device sync

- Five-minute recent-activity window: the window is "toward the long end so still watching counts," while still requiring visibility, focus, and trusted activity.

- No raw key/input capture: modern keyboard and pointer signals observe activity without recording keys or raw input data.

- Thirty-second qualified heartbeat: heartbeats report only intervals where all presence conditions held, and segmenting prevents partially invalid intervals from being fully counted.

- Server-bounded presence credit: receipt spacing, reporting windows, and rejection of impossible segments prevent client clocks from manufacturing hours.

- Union across owner windows/devices: overlapping device time counts once, duplicate listen is unioned, and split listen cannot exceed global presence duration.

- Settle and close terminate the invoking presence window: a settled display can still fetch state without earning presence, and undo/new interaction cannot backfill a terminal interval.

- Visitor exclusion from presence reporter: visitor time contributes nothing and direct capability checks enforce that exclusion.

- Listen-in end conditions: changing birds, focus leaving, suspension, or error closes listen exactly once, preventing abandoned listen-starts from accumulating.

- Offer outcomes and drowsy non-response: birds can respond naturally or ignore offers, so interaction remains ambient rather than guaranteed reward.

- Three-minute per-receiver cooldown: atomic reservation blocks parallel-device bypass and includes ignored offers so users cannot farm reactions while waiting for the next tick.

- Natural offer unavailability message: when all receivers cool down, the panel shows a brief naturalist explanation with no timer or punitive state.

- Settle directive visibility across snapshots: settle appears in canonical snapshots, including visits, because it is a durable scene directive associated with its initiating window.

- Five-second settle undo: the short accidental-click path reverses the local transition and clears the matching directive, while later deliberate re-engagement is still allowed.

- Snapshot pull triggers and 15-second polling: clients refresh on navigation, visibility, refocus, render gaps, mutations, and visible intervals so the displayed scene stays authoritative.

- Monotonic revision handling and clock offset smoothing: clients discard stale responses, reconcile cues, avoid replay, and adjust time without jerking birds or detuning calls.

- Presentation-only merge: clients interpolate drawn poses toward latest anchors but do not merge mood or trait copies.

- Connectivity degradation: the client retains only bounded presentation, disables server-dependent actions, avoids pretending unacknowledged offers succeeded, and does not invent an offline simulation.

- Name/settings/session/outage conflict prose: ordinary system prose explains retry, expiry, replay, and outages without uploading stale snapshots or using naturalist euphemism.

### Frontend scene and interaction surfaces

- Compact SVG species artwork and articulated pose components: this supports procedural variation, small payloads, and direct first-bird rendering without a large 3D/game engine, animation runtime, or video background.

- Render boundary independent of account/UI framework: the first bird should not wait for hydration, AudioContext, notebook data, or all six species assets.

- Server-rendered first SVG at absolute-time anchors: first paint can show the continuing authoritative pose directly rather than a generic entrance sequence.

- Single `requestAnimationFrame` loop with transform/opacity updates: this protects frame budget, avoids frame-loop DOM measurement, and keeps captions in a slower layout pass.

- Quiet field instead of spinner/skeleton flock: delayed snapshots or outages should not fake birds or gamify loading; healthy first load draws the real continuing pose.

- One-time starter introduction exception: after naming, two starter birds may softly enter because they are being introduced; the consumed cue prevents replay on reload.

- Logical 1000-by-480 horizontal scene: normalized safe anchors and letterboxing preserve proportions and ensure birds remain visible without camera scroll, crop, or stretched silhouettes.

- Responsive narrow layouts: staggered occupancy, collision-aware spacing, and larger invisible hit targets keep two through seven birds usable on phones and wide screens.

- Soft blue, green, brown, and ochre tokens: NOT RECOVERABLE FROM PLAN

- Subtle plumage saturation and daylight transitions: appearance changes can express drift and time without neon, alarm flashes, assertive weather, or accents competing with birds.

- Four top-bar buttons with large hit areas: all management text/actions live in panels, while controls stay discoverable, keyboard/touch usable, and quiet in the scene.

- No inline bird labels, mood badges, tooltips, offer buttons, status meters, or counters: the scene remains unlabelled and naturalist; clicking a bird means listen-in, not configuration.

- Reduced-motion preference before motion starts: users should not see a full-motion frame before settings load, and media-query changes update in place.

- Still-pose reduced-motion sequences: slow cross-fades and held poses preserve mood, greeting, settle, calls, captions, notebook, and interactions while removing micro-motion, flight translation, leaves, feathers, and parallax.

### Procedural audio and captions

- Versioned call grammar: calls are data describing motif relations, syllables, contours, rests, envelopes, and breath/noise; this preserves procedural synthesis and avoids recorded assets.

- One concrete expanded call description for audio and captions: captions truthfully match actual note count, contour, rhythm, rest, articulation, and context rather than a separately randomized or stock description.

- Small procedural song-fragment offer library: melodic offers remain synthesized motif data with no uploaded user recordings.

- AudioWorklet voice pool with native WebAudio fallback: synthesis remains procedural while controlling allocations, voice count, queue length, and AudioContext usage.

- Audio-clock scheduler with lookahead: scheduling against `AudioContext.currentTime` gives accurate note timing, reconciles snapshots by call ID, and prevents replay or clicks on partial mid-call starts.

- Hidden/focused audio behavior: hidden documents end owner listening/presence, ramp/suspend audio for battery, and refetch on return; visible-but-unfocused rendering can continue while presence stops.

- Dedicated gain bus and shallow pan per bird: each bird remains identifiable in space and loudness, with headroom and limiting to avoid peaks.

- Restrained chorus: overlapping compatible calls and varied phrasing create chorus while avoiding synchronized recordings, motif cancellation, or harsh high-frequency wash.

- Listen-in mixing: a focused bird is raised and others lowered but never solo-muted, preserving ambient life and avoiding abrupt gain steps.

- Settle audio ramps: canonical settle gently quiets calls; undo or re-engagement reverses from current values instead of jumping or restarting tracks.

- WebAudio failure fallback captions: graceful silence plus captions keeps the aviary's vocal life legible without recorded fallbacks, autoplay modals, or scene-interrupting toasts.

- Bird-proximate caption layout: captions near callers with contrast backing and collision solving keep call facts readable, connected to the scene, and not dumped into an unrelated fixed log.

### Accessible experience and naturalist writing

- Landmarks, roving-tabindex bird group, arrows, Enter, Escape, and Tab behavior: keyboard users can reach top bar, aviary, notebook, panels, birds, listen-in, and exit the scene without seven required tab stops.

- Natural bird accessible names: names identify bird, species, and perch without exposing traits, numbered perches, mood meters, or numeric values.

- Keyboard-reachable offers and settle: all core interactions remain reachable through the top bar, focus order, Escape, focus return, and a normal focused-button Enter path.

- Optional modifier shortcut: a documented remappable/disableable shortcut can help but is never the only route, avoiding assistive-technology conflicts.

- Dialog/popover focus behavior: panels can contain focus while open and return it to the opener without trapping focus inside the aviary.

- Stable focus identity: a focused bird keeps its semantic identity through perch changes, and pagination cannot recycle focused notebook content under a reader.

- Visitor accessibility controls without owner controls: visitors get meaningful scene descriptions and sensory preferences but cannot dispatch host events.

- Deterministic running narration: prose consumes the same scene/call description as rendering and writes connected observations instead of state-mutation bullet lists.

- Polite live region cadence and coalescing: ambient narration updates only when meaningful and avoids tiring queues or duplicate call-caption announcements.

- Prompt narration of greetings/offers/settle: user-significant events are available as observations while ordinary ambient announcements pause during system forms.

- Naturalist/system voice boundary: bird scene prose uses lowercase present tense, while account, error, sync, and accessibility instructions use normal capitalized matter-of-fact language.

- Contrast tokens and dynamic caption backing: measured contrast on day/night/weather backgrounds makes text, non-text controls, focus, and captions readable when scene colors shift.

- Combined accessibility testing: keyboard, zoom, text enlargement, reduced motion, muted/failed audio, high contrast, forced colors, screen readers, and human review ensure accessible modes retain charm and calmness.

### Notebook, adoption and account experience

- Server-generated notebook candidates from aviary facts: entries can reflect greeter changes, perch choices, bird exchanges, weather/posture combinations, or behavior change without production population statistics.

- Sparse notebook spacing and repetition guards: entries every two or three days for regular aviaries, with burst guards and deduplication, prevent active users from creating a busy feed by repeating offers.

- Bird observations, not user compliance: permitted entries describe birds; forbidden entries include visit streaks, session starts, and numeric trait changes.

- Read-only persisted entries with names-at-observation: historical text remains stable after rename and across pagination eviction.

- Starter selection from different species: distinguishable silhouettes/calls help the first two birds feel particular.

- No species catalog, reroll, rarity, or share prompt: adoption is about meeting birds and age-based opportunity, not avatar configuration, rarity hunting, social prompting, or activity rewards.

- Later adoption variety and seventh individual: the pool prefers variety until all six species are represented, then a distinct individual signature gives the seventh bird identity without ranking or rarity.

- Adoption lock and idempotent retry: concurrent devices cannot adopt an eighth bird or divergent versions of the same opportunity.

- Quiet account pages and explicit recovery actions: account flows use system voice, clear states, and recovery paths without translating errors into bird metaphors.

- Export worker with version-pinned read: export captures a consistent revision of birds, names, current vectors under the portability exception, moods, notebook, and settings, while excluding secrets and operational logs.

- Deletion pending state: active visits, invites, exports, and device access end immediately, simulation jobs suspend, and later recovery restores the same bird UUIDs/vectors with missed server time advanced but no manufactured presence.

- Hard-deletion worker and backup key destruction: erasure must remove live records, replicas, objects, jobs, capabilities, diagnostics, visit references, and account data keys, with drills proving deleted data cannot be restored.

### Visits, revocation and privacy boundaries

- Deliberate email-address invitation: sharing starts from an explicit host entry, with no discoverability, reciprocal invite, friend connection, or public URL directory.

- Cryptographically random bearer links with token hashes: inbox/link possession proves access while raw tokens are not stored.

- Unused invitation expiry and one-time redemption: stale invitations expire; redemption consumes once and creates a scoped session, while reloads reuse the visit cookie.

- Exact 30-day unused invitation duration: NOT RECOVERABLE FROM PLAN

- Visit session expiry and idle limit: a fresh visit after expiry requires a new deliberate invitation; multiple refreshes in the browser reuse the same session.

- Exact 24-hour absolute and 30-minute idle visit limits: NOT RECOVERABLE FROM PLAN

- Visitor common renderer/audio grammar with host canonical state: visitors see the same birds, palette, weather, active cues, and moods, not a special beautified state.

- Visitor response omissions: owner presence, host-online status, device identities, settings, notebook, vectors, and interaction logs are excluded to maintain the privacy boundary.

- Per-pull visit validation, including 304s: revocation must not leave cached successful authorization; scene and audio stop after the short lease when access is lost.

- Shorter visitor offline grace: when validation fails from network loss, visit exposure stops at lease expiry rather than using the owner's 120-second offline presentation grace because access may have been revoked.

- Approximate private visit duration: start and approximate duration give sharing transparency without inferring continuous attention or creating presence/drift input.

- Revocation without confirmation badge: active access closes and lists update, but there is "no notification or badge confirming that revocation worked," matching quiet product voice.

- Default-off visit email: opt-in mail is restrained and transactional, deduplicated by session UUID, rechecked before send, and excludes bird state, tracking pixel, reminder, or recommendation.

- Structural privacy enforcement: simulation storage, identity decryption, logs, aggregate metrics, account-specific errors, and privacy policy are all shaped so bird state and interaction history do not leak into analytics, monitoring, or exportable attendance features.

### Performance budgets and observability

- Initial JavaScript and critical first-bird transfer budgets: small eager payloads and dependency reporting ensure the first actual bird is not blocked by panels, audio preparation, unused species, or transitive imports.

- First-bird under 500 ms: the metric protects the "already-alive" conceit and must be proven with visual frame evidence on a defined healthy network/device profile.

- Snapshot size target: excluding notebook history, raw logs, full traits, and unused species assets keeps seven-bird snapshots compact and presentation-only.

- Return notice budget: visible bird behavior within 1-2 seconds verifies return greetings as observed aliveness, including audio-blocked and reduced-motion paths.

- Idle frame and memory budgets: sustained frame rate and no retained-heap/resource growth protect long quiet sessions with seven birds, weather, captions, panels, offers, notebook, settings, hide/restore, and audio.

- Tick latency and queue-age monitoring: p99 latency, due-to-commit lateness, queue age, and retries reveal worker health so a fast worker behind a long queue cannot appear healthy.

- Physical browser/device fixtures: stable release fixtures make performance comparisons meaningful and prevent support-window drift from hiding regressions.

- Cold-navigation checks and filmstrips: cold-cache regressions count even if averages look fast; initial adoption is measured separately from returning-owner loads.

- Bounded caches and resource disposal: caption/cue caches, note queues, notebook cache, ornament pools, subscriptions, observers, nodes, timers, and worklets are bounded to prevent leaks.

- Allowed aggregate RUM and server metrics: only technical timing, frame, audio failure, browser/capability, duration-bucket, request, worker, export/deletion, and rate-limit health are measured.

- Disallowed metrics and analytics: account/session/invitation/bird IDs, names, species, vectors, mood, offer type, listen target, presence history, full URL, email, and interaction payloads are excluded so technical health does not become behavior analysis.

- Technical alerts: alerts point the operating team to tick lag, failures, broken jobs, regressions, and error spikes, not to user-facing aviary notifications.

### Verification strategy, delivery, rollout, and risks

- Seeded fixtures and controllable server clock: tests target invariants and failure modes rather than restating markup or helper implementation details.

- Automated checks plus human reviews: drift, visual aliveness, audio recognition, accessibility charm, and calmness need both machine evidence and named human perceptual/accessibility review.

- Drift correctness tests: property tests prove bounded nondecreasing traits, no absence decay, no reset, calibration trajectories, and filter-state preservation.

- Presence, multi-device, and ordered-tick tests: truth tables, overlap, reordered heartbeats, crashes, races, duplicate leases, and serial references protect calibration and canonical state.

- Canonical continuity tests: server state must advance with all browsers closed, and devices must converge without vector uploads, resets, or replayed audio.

- Command, settle, notebook, adoption, auth, visit, lifecycle, visual, audio, accessibility, performance, and privacy tests: each acceptance area targets a plan-stated failure mode such as duplicate cue, eighth bird, token race, host mutation, revoked cached state, hard-deletion residue, cropped bird, recorded asset request, narration fatigue, or prohibited telemetry.

- Model migration tests: long-lived named birds with filter state and old grammar versions must upgrade/rollback without ID changes or trait decrease.

- Sequential integration stages: contracts/perceptual slice, persistence/clocks, owner session, lifecycle/visits, hardening/calibration, and limited release ensure boundaries, evidence, and full v1 scope are complete before v1.

- Private technical preview: smaller slices may be tested internally but cannot redefine missing v1 requirements as later enhancements.

- Stage-1 instrumentation: performance and privacy-schema checks begin early so they are not post-launch add-ons.

- Separate bird-count rollout clocks: engineering validates synthetic two through seven birds, while actual owners start with two and gain birds only through age-based opportunities.

- Versioned server-side configuration: drift, mood, weather, notebook, and call-density tuning applies prospectively, preserves stored state, and avoids personalized or engagement-optimized variants.

- Rollback paths: faulty UI/audio code, invitations, adoption acceptance, and new-account intake can be adjusted without clearing vectors, accepting client snapshots as truth, or pausing absent-bird simulation.

- Risk mitigations: the plan ties each risk to detection and mitigation, preserving monotonic drift, individuality, canonical ordering, honest presence, prompt interaction cues, recognizable calls, accessible relationship, fast cold load, background simulation, revoked-visit closure, PII boundaries, explicit exceptions, erasure safety, accessible chrome, and sparse notebook behavior.
