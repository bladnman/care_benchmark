## System-level intent

1. Presence is the primary product signal, not activity volume. This is stated in the product contract as "Presence-not click volume or a reward loop-is the primary signal," and it reappears in the drift model, the ban on "streaks, scores" and "visit frequency" copy, age-only adoption, and rollout using "aggregate health/performance signals" rather than engagement scores.

2. The aviary should feel continuing and alive between visits. The plan says it should "feel as though it continues between visits," runs a tick "for every aviary, including those without connected clients," starts the first meaningful frame with "ambient motion already in progress," and avoids an audio gate, welcome modal, spinner, and wake-up animation.

3. The server owns canonical reality; clients render and submit bounded events. This shows up in "The server owns the one canonical aviary," "clients render snapshots and submit events," "server is authoritative," monotonically increasing snapshot versions, event-log watermarks, idempotency keys, and tick workers committing state with the consumed watermark.

4. Change is slow, one-way, calibrated, and non-punitive. The plan uses "gradual, one-way personality drift," "bounded non-negative deltas," "No no-show or inactivity path subtracts from a trait," "absence never decreasing a trait," and a release risk about drift becoming "guilt-inducing."

5. The product voice is quiet, naturalist, and not gamified. Product-facing narration, reactions, captions, and notebook text use "specific lowercase naturalist prose"; failures use "plain system language"; the notebook avoids "generic event rows," "trait deltas," attendance, and streak phrasing; there is no "welcome toast or banner."

6. Privacy and data minimization are cross-cutting boundaries, not analytics afterthoughts. Email is encrypted and not a partition key, trace attribute, or log identifier; telemetry is "physically and logically apart" from simulation state; operational data is aggregate; logs use synthetic UUIDs; interaction records are for "the owner's simulation" and not analytics.

7. Accessibility is part of v1 and comes from the same world model as rendering. The plan says "Ship accessibility as part of v1," narration and captions are "sourced from the same snapshot," captions come from the same procedural grammar as audio, and accessibility/audio acceptance happens "before release, not in a later patch."

8. Social access is optional, narrow, read-only, and must not alter the host. "Visiters never affect host state," visitor sessions are "read-only," visitor data is not presence or drift input, notifications are default off, and there are no public profiles or discovery indexes.

9. Calibration, replay, and release gates protect the intended feel. The plan requires a deterministic simulation harness, versioned drift parameters, week-one numerical calibration, week-three visual review, invariants for monotonic traits and replay consistency, staged age cohorts, and rollback flags.

10. Compact performance shapes the implementation. The plan prefers a "compact 2D Canvas/SVG scene," avoids "a large 3D runtime," keeps snapshots in kilobytes, budgets the JavaScript bundle below 2 MB gzipped, and requires the first bird visible under 500 ms.

## Per-feature whys

### Product contract and scope

- **Browser-only delivery**: NOT RECOVERABLE FROM PLAN
- **Single-user account with one aviary**: Why in plan: it keeps "one canonical aviary" owned by the server, with clients rendering snapshots and submitting events rather than supporting multiple or shared aviaries.
- **Starter adoption flow**: Why in plan: the user "adopts two starter birds" and "learns them through quiet observation," which fits the presence-first contract.
- **Exactly two starter birds at creation**: NOT RECOVERABLE FROM PLAN
- **Age-paced additions up to seven**: Why in plan: adoption is a "quiet optional flow" whose thresholds are "independent of sessions or attention"; the cap also supports recognizability and performance, with a risk note to "not expand beyond seven" in v1.
- **Bird renaming**: NOT RECOVERABLE FROM PLAN
- **Moods as a product feature**: Why in plan: mood is "fast state" that drives visible action and perch choice, while personality remains "slow state" and hidden.
- **The exact five initial moods, wary, content, curious, drowsy, alert**: NOT RECOVERABLE FROM PLAN
- **Daily mood boundary**: Why in plan: a daily boundary triggers "reevaluation, not a reset to neutral," preserving continuity rather than wiping state.
- **IANA timezone on the account and aviary**: Why in plan: timezone is used for "day/night and mood timing" and for local scene time.
- **Visit notifications default off**: Why in plan: they are a "narrow exception" to the broad no-notifications principle, never prompted during onboarding, and never create a visit badge or unsolicited default message.
- **Offer affordance in the top bar**: Why in plan: the top-bar panel resolves "per-bird cooldown/recipient" needs "without adding an in-scene control."
- **Audio autoplay handling without an audio gate**: Why in plan: browser restrictions can prevent sound, but the scene should keep moving; captions cover unavailable audio and there is no welcome modal.
- **Presence lease union across overlapping devices**: Why in plan: counting same-account leases as a time union "prevents two open devices from accelerating drift."
- **No bird-count badge, visit calendar, activity dashboard, or visit-frequency copy**: Why in plan: these exclusions protect the "no reward loop" and prevent the product from reporting attendance.

### Architecture and boundaries

- **Small web client plus authenticated APIs, simulation service, storage, and tick worker**: Why in plan: the exact stack can stay open while the boundaries are preserved "regardless of stack."
- **Server-authoritative bird identity, mood, offers, ordering, notebook, weather, and snapshots**: Why in plan: the client may animate between states but "cannot mutate canonical bird state."
- **Idempotency keys on writes**: Why in plan: a retry must not apply an event or drift delta twice.
- **Monotonic snapshot version and event-log watermark**: Why in plan: workers can consume ordered events, commit the consumed watermark, and avoid last-write-wins replacement.
- **Per-aviary tick lease or compare-and-swap**: Why in plan: concurrent or restarted workers must not double-apply events or lose canonical state.
- **Operational telemetry separated from simulation state**: Why in plan: the collector has "no read path to bird records or interaction logs."
- **Aggregate-only telemetry**: Why in plan: allowed data is request counts, latencies, errors, render timings, audio errors, and anonymous session-duration histograms, not bird records or event logs.
- **Synthetic account UUIDs in logs and service keys**: Why in plan: logs and inter-service keys must use synthetic UUIDs, "never email."

### Data model

- **Encrypted verified email on Account**: Why in plan: email is "never a partition key, trace attribute, or log identifier."
- **Device sessions with opaque hashed revocable tokens**: Why in plan: sessions can be listed and revoked while raw tokens are never logged.
- **Aviary creation time driving age pacing**: Why in plan: later adoption opportunities depend only on aviary age.
- **Deterministic scene seed**: Why in plan: seeded state supports replay, stable rendering, and deterministic procedural variation.
- **Stable bird UUIDs and adoption time**: Why in plan: identity persists through drift, rename, export, and replay invariants.
- **Hidden normalized personality vector**: Why in plan: personality affects expression but raw numbers stay out of product UI and telemetry.
- **Snapshot exposing render outputs rather than raw traits**: Why in plan: clients render appearance, pose, mood, and calls without receiving numeric personality vectors.
- **Personality vectors in explicit account export only**: Why in plan: traits stay invisible except in an explicitly requested export.
- **Interaction events as owner simulation records**: Why in plan: they are stored "only for the owner's simulation" and are not copied to analytics.
- **Visitor inability to write interaction events**: Why in plan: visitors never affect host state, presence, offers, listen-in, or drift.
- **Notebook entries as stable, append-only naturalist records**: Why in plan: entries are readable indefinitely and generated from approved observations/facts rather than editable feed rows.
- **No notebook edit/delete UI**: NOT RECOVERABLE FROM PLAN
- **Visit invitation/log with one-time token hash and minimal duration records**: Why in plan: invites support delivery and visit transparency without public profiles or discovery indexes.
- **Small species pool without rarity or catalog selection**: Why in plan: the adoption flow should not become a rarity or catalog mechanic.
- **New adoption opportunities independent of attention, offers, or paid tier**: Why in plan: additions must not become a reward loop or monetized collection path.

### API surface and user flows

- **Email magic-link accounts as the authentication method**: NOT RECOVERABLE FROM PLAN
- **Magic-link rate limits, 15-minute expiry, and consume-once behavior**: Why in plan: failures must not expose account existence, tokens are short lived, and successful use invalidates the link.
- **Email change verification before switching**: NOT RECOVERABLE FROM PLAN
- **Snapshot API returning public-to-render state, server time, version, and watermark**: Why in plan: clients can sync and render without receiving trait values or event history.
- **Conditional snapshot pulls after boot, visibility return, long render gap, and low-frequency keepalive**: Why in plan: the client stays aligned with the canonical server snapshot while visible polling stays low frequency.
- **Events API with known types and bounded payloads**: Why in plan: the client never sends trait values or proposed full state.
- **Presence heartbeats credited only from fresh authenticated host sessions**: Why in plan: there is "no arbitrary client backfill," and server elapsed time bounds presence.
- **Listen-in start and end events with bird ID**: Why in plan: listen-in chiefly nudges the selected bird's social warmth and vocal frequency.
- **Offer event with kind and optional recipient**: Why in plan: the server checks eligibility/cooldown and returns an immediate deterministic reaction cue.
- **Settle with five-second undo**: Why in plan: settle quiets mood and closes the presence lease, while undo restarts a presence lease through an in-scene click during the window.
- **Paginated immutable notebook API**: Why in plan: notebook entries stay sparse, readable indefinitely, and not a visit-frequency record.
- **Export request with short-lived verified-email download link**: Why in plan: export is explicit and includes otherwise hidden vectors, moods, notebook, and settings.
- **Recoverable deletion for 30 days followed by hard delete**: Why in plan: account-linked records, birds, vectors, events, notebook, invites, sessions, backups, and artifacts must match the deletion promise.
- **Named visit invitations with redeemable read-only visitor sessions**: Why in plan: optional visits are narrow, revocable, expiring, and not a social discovery surface.
- **Visitor snapshot revocation checks on every pull**: Why in plan: revocation or expiry should produce a matter-of-fact unavailable surface.
- **Plain system language for failures and lowercase naturalist prose for product surfaces**: Why in plan: system failures are distinct from narration, offer reactions, captions, and notebook voice.

### Simulation engine and calibration

- **Approximately one-minute tick for every aviary**: Why in plan: aviaries continue even without connected clients, and delayed workers recover through elapsed-time calculations.
- **Ordered event consumption before state updates**: Why in plan: ticks apply unconsumed events exactly once and commit one canonical snapshot/version.
- **Elapsed-time recovery after missed ticks**: Why in plan: a missed tick is recovered "without inventing presence during the gap."
- **Presence active only while visible, focused, and recently moved or typed**: Why in plan: this limits overstated or spoofed presence while leaning long enough so "quiet watching counts."
- **No raw pointer or key stream**: Why in plan: only the active/inactive interval needed by the account's simulation is stored.
- **Low-pass drift over validated presence and secondary events**: Why in plan: it makes personality change gradual; regular use changes are measurable around one week and visible around three weeks.
- **Presence as the dominant drift input**: Why in plan: the product contract says presence, not click volume, is the primary signal.
- **Listen-in nudge to social warmth and vocal frequency**: Why in plan: listen-in affects the selected bird but remains secondary to presence.
- **Offer nudge to curiosity and boldness**: Why in plan: accepted offers and offering near a bird create small bounded positive deltas.
- **Settle adds no trait direction**: Why in plan: settle quiets mood and closes presence rather than becoming another trait lever.
- **Bounded non-negative trait deltas and clamping**: Why in plan: traits move only toward expressive ends and absence never decreases a trait.
- **Versioned server configuration for constants and weights**: Why in plan: calibration is reversible without resetting bird identity.
- **Deterministic replay harness**: Why in plan: seeded synthetic traces prove monotonic traits, no-decrement absence, stable IDs, caps, idempotency, and replay consistency.
- **Mood transition persistence across sessions**: Why in plan: mood is reevaluated from recent owner interactions, local time, ambient events, and hidden personality rather than reset.
- **Weather and alarm-call effects on mood and calls**: Why in plan: rain, wind, alarm calls, and boldness shape behavior through mild deterministic-weighted transitions.
- **Absence causing quieter or ambient behavior only**: Why in plan: birds never become hungry, distressed, angry, dead, or less colorful.
- **Return greeting with one likely bird and staggered others**: Why in plan: the scene avoids a "unison welcome" and varies greeting by boldness, mood, and absence duration.
- **Visitor rendering without host-arrival greeting or presence credit**: Why in plan: visitor viewing must not affect host state.
- **Notebook prose from approved templates and actual events**: Why in plan: entries remain sparse, useful, naturalist, tied to actual bird/aviary facts, and never praise or count attendance.
- **No free-form LLM generation for notebook v1**: Why in plan: approved templates and voice linting are enough for v1.

### Frontend scene and interaction pipeline

- **Shell split into top bar, renderer, narration/caption layer, and event client**: Why in plan: domain state and simulation math stay out of the renderer.
- **Compact 2D Canvas/SVG renderer with generated/compact bird assets**: Why in plan: it avoids a large 3D runtime and supports the performance gates.
- **Single horizontal, non-scrollable scene with perch zones and parallax**: NOT RECOVERABLE FROM PLAN
- **No buttons, labels, hover icons, or controls inside the scene**: Why in plan: interaction stays in the top bar and the scene remains free of in-scene chrome.
- **First meaningful frame using current pose and animation phase**: Why in plan: the aviary should look continuing, with ambient motion already in progress.
- **No fade from static, spinner, or wake-up animation**: Why in plan: these would work against the "already alive" feel.
- **First signup empty-aviary interval before two starter fly-ins**: NOT RECOVERABLE FROM PLAN
- **Quiet sky field during unavailable or slow states**: Why in plan: the fallback preserves faint motion instead of showing a spinner.
- **Fast private boot snapshot and shared-asset caching only**: Why in plan: first bird appears quickly while personalized account state is never shared-cache data.
- **Local interpolation between server snapshots**: Why in plan: it smooths motion while the server remains authoritative.
- **Stop scene rendering when hidden and refetch on return**: Why in plan: hidden pages do not keep rendering and resume from the canonical snapshot.
- **Responsive phone and desktop layouts preserving all three zones**: Why in plan: narrow layouts must not crop birds, and wider spacing still preserves zones.
- **Top bar limited to account/settings, accessibility, notebook, and offer controls**: Why in plan: controls stay out of the scene and avoid extra chrome.
- **Top bar fading nearly transparent after stillness**: Why in plan: the control surface recedes during quiet watching and returns for pointer or keyboard activity.
- **Settle in the top bar with evening shift and undo click**: Why in plan: settle makes calls quieter and allows a five-second in-scene undo.
- **Listen-in gradual mix ramp**: Why in plan: the focused bird becomes louder while others are reduced but "always audible," and disengagement is slow.
- **Offer choices and receiver selection in the panel**: Why in plan: seed, song-fragment, still-pool, and recipient choice happen without direct bird-click offer interaction.
- **Quiet cooldown state for offers**: Why in plan: the plan explains cooldown matter-of-factly and says not to turn it into a reward timer.

### Procedural audio

- **Small WebAudio synthesizer with per-species motif grammars**: Why in plan: calls stay fresh while species and bird recognizability persist.
- **Seeded stable bird call signatures**: Why in plan: each bird remains recognizable through mood and drift.
- **Call sequence counter with constrained timing and pitch variation**: Why in plan: calls vary without losing the bird's core signature.
- **Mood changes articulation and timing**: Why in plan: mood affects expression without replacing identity.
- **Vocal-frequency drift changes call probability and chorus participation**: Why in plan: drift changes how often the bird joins in, not the recognizable signature.
- **Server-published call timing, grammar version, and seeded cue**: Why in plan: snapshots carry the procedural inputs the client synthesizes.
- **No recorded-call download or recorded fallback**: NOT RECOVERABLE FROM PLAN
- **Bounded voice pool and reusable buffers/nodes**: Why in plan: calls are scheduled with low allocation and disposal discipline.
- **Chorus mixer**: Why in plan: simultaneous motifs are balanced spatially and temporally rather than becoming identical loops.
- **Rain and evening lowering call activity**: Why in plan: ambient weather and local time affect bird behavior and calls.
- **Song-fragment offers using the motif library**: Why in plan: song-fragment offers stay compact and do not use recorded music.
- **Captions generated from the same grammar parameters as audio**: Why in plan: caption and rendered call match.
- **Silent default with captions if WebAudio is missing, denied, or fails**: Why in plan: unavailable audio should not break the experience.
- **Mute preference and stopping audio while hidden**: Why in plan: audio respects user preference and page visibility.

### Accessibility and performance

- **Screen-reader naturalist narration every 30-60 seconds at idle and promptly after actions**: Why in plan: narration is part of v1 accessibility and should respond without chattering.
- **Queued narration updates**: Why in plan: updates should not overlap or chatter.
- **No coordinate, mood-code, or personality-number announcements**: Why in plan: narration stays naturalist and does not expose meters or numbers.
- **Captions as opt-in when audio works and default-on when audio is unavailable**: Why in plan: captions describe actual procedural calls and cover audio fallback.
- **Reduced-motion rendering**: Why in plan: it honors `prefers-reduced-motion` and account settings while calls and simulation continue.
- **Keyboard access through top bar, scene birds, listen-in, offer, settle, settings, and notebook**: Why in plan: every interactive surface must work by keyboard.
- **Visible focus rings over every palette**: Why in plan: keyboard focus remains usable through day, evening, weather, and scene palette changes.
- **WCAG AA text contrast**: Why in plan: all user-copy text must meet the accessibility gate.
- **Unsupported-browser page for older browsers**: Why in plan: older than the last two major releases receive a clear unsupported-browser page.
- **JavaScript, first-bird, frame-rate, memory, and payload performance gates**: Why in plan: the scene must load quickly, stay smooth, avoid memory growth, and paint the first bird without waiting for non-critical assets.

### Security, privacy, and observability

- **Magic-link account-existence protection**: Why in plan: rate limits apply per email "without exposing account existence."
- **Hashed single-use tokens with short expiry and invalidation**: Why in plan: authentication, invites, and exports should not expose reusable raw tokens.
- **Per-device revocation**: Why in plan: device sessions are independently revocable from account settings.
- **Visitor credentials restricted to read-only snapshot access**: Why in plan: visitors cannot write events or affect host behavior.
- **Account email access limited to account service and mail delivery**: Why in plan: encrypted email access is tightly scoped.
- **Interaction records excluded from training, recommendations, population bird analysis, third-party sharing, and analytics warehouse**: Why in plan: owner interactions are retained only for the owner's simulation.
- **Aggregate operations data excluding account IDs and bird fields**: Why in plan: observability covers health and performance without logging bird behavior, call content, or individual session timelines.
- **Export links, backups, and artifacts expiring on schedule**: Why in plan: expiration must be compatible with the 30-day hard-deletion promise.

### Delivery sequence and release gates

- **Contracts and calibration foundations first**: Why in plan: schemas, API contracts, idempotency rules, prose voice rules, privacy flows, and the replay harness must exist before product tuning.
- **Account and canonical engine before scene interactions**: Why in plan: starter birds, event log, tick, snapshots, ordering, weather, mood, and monotonic drift are the canonical base.
- **Engine gates on replay, idempotency, presence conjunction, and two-device checks**: Why in plan: the system must prove canonical state, exact-once behavior, and non-accelerated presence.
- **Scene gate on no announce and no in-scene chrome**: Why in plan: the interaction layer must preserve the quiet scene and avoid badges or embedded controls.
- **Accessibility and audio acceptance before release**: Why in plan: those surfaces are v1 requirements, "not in a later patch."
- **Security review of invite isolation and deletion coverage**: Why in plan: visits must not leak or affect host behavior, and deletion must cover linked records.
- **Limited launch with age-only later-bird opportunities and rollback flags**: Why in plan: recognizability and performance are validated before each additional age cohort, with rollout governed only by aggregate health/performance signals.
