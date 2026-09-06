## System-level intent

- **The bird remains the same bird.** The plan makes continuity a hard invariant: “Stable bird UUID, persistent vector and call identity, migration continuity, tested restores; no regeneration from interaction history.” This recurs in the persistent model, export, deletion recovery, rollback, migration, and verification sections, all of which preserve UUIDs, vectors, signature seeds, grammar compatibility, and prior notebook prose.

- **Absence is acceptable.** The plan says there is “No negative personality delta, no absence-induced distress, no goodbye requirement, and no copy about missed visits.” Mood, settle, deletion, notebook, and risk sections repeat that birds do not become ill, plead, lose color, lose stored traits, or require a recovery interaction because the user was away.

- **Presence is the principal input.** The plan states “Credit only visible AND window-focused AND recently active owner intervals” and “union simultaneous device intervals before computing drift.” Presence is made dominant in the drift formulas, while offers, listen-in, and acceptance are capped supplementary inputs.

- **The server owns the simulation.** The plan says “Clients submit events and render projections; only the simulation service creates or changes personality vectors.” This shows up in the service boundaries, event contracts, tick transaction, synchronization rules, and no-client-state-merge risk response.

- **The place was already moving.** Initial and resumed frames must “sample existing action phases,” with “no ordinary entry sequence, spinner, wake-up, or fade from a static scene.” The first-frame, client synchronization, and rendering sections all preserve the sense that the aviary continued while no one was watching.

- **Noticing is the welcome.** The plan says a bird’s “varied response is the entire arrival surface” and forbids welcome text, toast, badge, banner, or absence counter. Return-greeting is therefore bird behavior, timing, gaze, pose, and phrase variation rather than interface announcement.

- **The relationship is private.** The plan says simulation data has “no analytics, training, recommendation, or third-party export pipeline.” Privacy controls shape storage credentials, telemetry allowlists, exports, visits, notebooks, and the refusal to collect population-average drift or engagement charts.

- **Accessible modes carry the same experience.** The plan requires shared state and call descriptors to drive “visuals, prose, reduced-motion poses, and captions from the first public release.” Reduced motion, captions, narration, keyboard focus, and assistive technology are treated as complete product surfaces, not later fallbacks.

- **The scene remains comprehensible.** The plan emphasizes “All birds visible in one scene, three perch zones, seven-bird hard cap, restrained calls and visual density.” This principle appears in layout, call overlap limits, caption collision handling, bird-count ramp, and performance soak tests.

- **Restraint is part of the product voice.** The plan excludes achievements, counters, scores, streaks, levels, badges, rarity, catalogs, public rankings, custody mechanics, and absence guilt. Adoption, notebook, sharing, telemetry, and rollout all avoid “earned” language, visible timers, engagement rankings, and comparison surfaces.

- **System language is matter-of-fact.** The plan repeatedly calls for direct system surfaces: expired links, unavailable visits, loading failure, recovery, audio unavailability, and browser requirements use plain system language. It also says birds must not report errors, plead during deletion, or become a vehicle for guilt.

- **Contradictions are resolved as narrow explicit exceptions.** The plan records tension decisions for raw vectors in export, optional visit notifications, captions/focus outlines, audio autoplay, and settle placement. It says to “Keep one chosen behavior per surface” and not quietly add both competing interpretations.

## Per-feature whys

**Basis, scope, and acceptance invariants**

- **Browser-based aviary:** The plan scopes v1 to a browser experience and excludes native apps; the browser surface supports authenticated HTML, multi-device rendering, WebAudio/caption fallback, keyboard access, and the first-bird performance budget.

- **One account and one canonical scene:** The plan ties this to one persisted aviary, one canonical projection, and “no divergent persistent device simulations to merge.”

- **Two starter birds:** The plan says a verified new account initializes an aviary and “two different starter species atomically,” so repeat sign-in or retry cannot create another aviary or starter pair.

- **Eventual maximum of seven birds:** The plan grounds this in comprehensibility: “All birds visible in one scene,” bounded visual density, restrained calls, responsive layout, and a seven-bird hard cap.

- **Continuous bird behavior:** The plan’s why is that the aviary is already alive: actions continue through sessions, devices, restarts, and unattended ticks rather than beginning from a static scene.

- **Procedural calls:** The plan uses procedural grammar so calls can vary while preserving “immutable individual signature seed” identity; it also avoids recordings and third-party/sample-loop fallback.

- **Varied return-greeting:** The plan makes “Noticing is the welcome”; variation prevents a “small rotating library of greeting clips” from becoming repetitive and keeps arrival as bird behavior rather than UI chrome.

- **Honest presence accounting:** The plan says browser signals are a “cooperative attention approximation” and chooses undercounting over “manufacturing unattended hours.”

- **Listen-in:** The plan gives focused attention to one bird while keeping “other birds remain audible”; attention through captions or narration receives the same credit as audible attention.

- **Offer:** The plan makes offers a small, capped, server-interpreted input; exposure can shape boldness, acceptance can shape curiosity, and a request never manufactures an accepted gift.

- **Settle:** The plan uses settle as a canonical quiet evening state that ends owner presence windows and avoids goodbye requirements, penalties, or recovery interactions.

- **Sparse, permanent field notebook:** The plan says entries should describe “birds and place” from factual evidence, be read-only and immutable, and remain sparse so they do not become session reports, streaks, or generic filler.

- **Email magic-link accounts:** The plan uses high-entropy single-use links, no passwords, non-enumerating wording, scanner-safe confirmation, encrypted addresses, and synthetic account identities to protect identity and privacy.

- **Multi-device access:** The plan’s rationale is canonical continuity: owner devices read the same projection, events are ordered, presence is unioned, and device simulations never merge client bird state.

- **Private, revocable visits:** The plan frames visits as deliberate, render-only sharing with one-time tokens, no discovery, no co-presence, no host-online status, no owner actions, and revocation checked on every pull.

- **Complete accessibility surfaces:** The plan treats accessibility, privacy controls, and performance as “release requirements, not follow-up enhancements,” because accessible modes must carry the same experience.

- **Personality changes after regular attention:** The plan calibrates personality to be “instrumentally measurable” after about a week and perceptible after about three weeks, while one isolated session should not visibly alter the bird.

- **Six species:** NOT RECOVERABLE FROM PLAN

- **Coherent species silhouettes and motif families:** The plan says species differences must be “recognizable without a catalog or rarity label.”

- **Editable bird names:** The plan preserves user naming while keeping identity stable: rename changes only name/revision and not personality, mood, adoption age, grammar identity, or old notebook prose.

- **Age-based adoption:** The plan makes adoption unrelated to visits, gifts, payment, or drift, preserving absence acceptance and avoiding reward/counter mechanics.

- **Local-time lighting:** The plan uses the account timezone so all devices and visitors render the same clock, while timezone changes do not create drift or adoption-age effects.

- **Occasional weather:** The plan keeps weather synthetic, brief, canonical, and mild so it affects mood/call rate without adding tasks, real-world services, or dramatic events.

- **Per-device session revocation:** The plan uses revocation to invalidate queued/pending authorization, wake long-polls, and take effect on protected requests.

- **Verified email changes:** The plan stages pending address verification before switching the canonical address so account and bird UUIDs are preserved and old sign-in challenges can be invalidated.

- **Account export:** The plan makes export an explicit private archive with a consistent canonical version; vectors are included only as a narrow exception and not exposed in ordinary surfaces.

- **Recoverable deletion followed by hard deletion:** The plan gives a 30-day “I changed my mind” recovery path that restores the same birds, followed by tested erasure and key destruction so hard deletion cannot be undone by backup restore.

**Explicit implementation decisions and PRD tensions**

- **Raw numerical vectors in export only:** The plan says this is a “narrow exception” to the non-exposure rule because a downloadable JSON archive explicitly requires current vectors; ordinary snapshots, settings, narration, captions, notebook entries, errors, and debug surfaces still never expose them.

- **Optional visit notifications:** The plan implements only an off-by-default quiet transactional email when an invite is first used, resolving the notification tension without push, reminders, badges, return invitations, or other aviary emails.

- **Four top-bar icons with settle in the offer popover:** The plan keeps exactly account/settings, accessibility, notebook, and offer as permanent icons, while settle remains reachable without adding a fifth icon or scene button.

- **Call captions and visible focus as scene-chrome exceptions:** The plan permits them as accessibility requirements while still forbidding persistent bird labels, mood icons, hover tooltips, and numerical overlays.

- **One account-level IANA timezone:** The plan prevents devices and visitors from silently disagreeing on the aviary clock and makes travel changes deliberate in settings.

- **Sixty-second scheduled simulation tick:** The plan uses it so server work continues whether anyone is connected or not and so interaction wakeups cannot accelerate drift.

- **Immediate interaction wakeups:** The plan lets arrival, offer, settle, and adoption wake the serialized simulation worker for responsiveness while preserving the same elapsed-time integration.

- **Audio permission fallback:** The plan acknowledges browser autoplay rules; when sound is blocked, the already-moving aviary and captions remain truthful without a welcome modal or fake audible-call claim.

- **Keyboard listen-in rules:** The plan makes Enter idempotent so keyboard focus does not accidentally undo listen-in, arrows transfer attention, and Escape releases.

- **Canonical settle scope:** The plan makes settle visible to owner devices and visitors so everyone sees the same evening state, but only deliberate owner action can resume it.

- **Sound and accessibility settings not penalizing birds:** The plan says mute, captions, reduced motion, screen-reader use, and silence are not negative inputs, keeping access preferences from harming the relationship.

- **Eight-hour non-renewable consumed visit session:** The plan chooses this so another visit requires a new deliberate invitation and approximate duration covers only the visitor’s visible render session.

**Service shape and module boundaries**

- **Small TypeScript application:** NOT RECOVERABLE FROM PLAN

- **Server-rendered HTML and small browser runtime:** The plan uses SSR so the first response can contain current bird geometry, elapsed action phases, and a minimal controller for fast first bird paint.

- **PostgreSQL canonical store:** The plan relies on row locks, durable due-work records, committed cursors, constraints, and transactions for one-writer simulation correctness.

- **Separate worker sharing a pure simulation package:** The plan separates due ticks, ordered event consumption, vectors, moods, action schedules, and notebook observations from HTTP request handling.

- **Browser shell and SVG scene renderer boundary:** The plan lets the browser own focus, lifecycle, input capture, interpolation, local audio, captions, and preferences, but not canonical mood transitions, drift, adoption eligibility, or writable vectors.

- **HTTP/auth service boundary:** The plan uses the service for authentication, authorization, validation, event append, reads, account commands, and visit grants, while rejecting client-proposed personality writes.

- **Trusted edge delivery and private no-store state:** The plan prevents shared caching of private HTML, snapshots, invites, or exports and checks authorization on every request.

- **Operational collector without simulation credentials:** The plan lets aggregate health be measured without giving metrics a read credential to simulation data.

- **Normal data path with snapshot long-poll:** The plan uses authenticated snapshot reads, durable events, worker commits, version-change hints, and interpolation so visible clients update from canonical versions.

**Persistent model and integrity rules**

- **Synthetic UUIDs:** The plan uses them for accounts, birds, sessions, invitations, and internal references to avoid email as identifier and preserve stable identity.

- **Encrypted email envelope:** The plan says email is authentication/contact data, never an account identifier, partition key, log dimension, or routing value.

- **Invitation recipient encryption:** The plan stores recipient addresses encrypted once per invitation and exposes them only in the host’s settings/log.

- **Server-side finite normalized personality values:** The plan gives the simulation database role sole authority to insert or change vectors and keeps names, mood, and call identity from being reconstructed from vectors.

- **Atomic vector/filter/cursor/projection transaction:** The plan preserves all state needed for continuity and says backups must preserve them together.

- **No vector rebuild from retained events:** The plan says event retention is intentionally shorter than the bird’s life, so replay cannot become the source of identity.

- **Bounded retention defaults:** The plan keeps only what serves live engine, notebook truth, host transparency, operational diagnosis, or idempotency, while minimizing raw events, interval-union data, errors, auth secrets, and export objects.

**HTTP contracts, commands, and error handling**

- **Versioned schema-validated JSON contracts:** The plan requires explicit contracts, unknown-field rejection, and authorization before reads and long-poll returns so private state and event ordering stay controlled.

- **Magic-link request endpoint with same accepted response:** The plan avoids address enumeration whether an address exists or not.

- **Atomic link consume:** The plan prevents simultaneous consumes from duplicating identity or creating extra starter birds.

- **Account settings revision:** The plan uses revisioned updates so timezone, preferences, and notification changes do not silently overwrite conflicts.

- **Email-change staging and verification endpoints:** The plan preserves the old address until verification commits.

- **Account deletion and recovery endpoints:** The plan gives lifecycle actions during the 30-day recovery window while restricting ordinary aviary interaction.

- **Export job and download endpoints:** The plan queues a version-consistent archive and redeems only a short-lived authorized download.

- **Snapshot endpoint with known_version and bounded wait:** The plan lets clients long-poll canonical versions without holding stale private state or excessive connections.

- **Owner event API:** The plan accepts only arrival, presence_interval, listen_start, listen_end, offer, settle, undo_settle, resume, and view_end so no event can supply a trait, absolute mood, perch assignment, or arbitrary call parameters.

- **Prior event receipt lookup:** The plan lets uncertain outcomes resolve by event ID without resubmitting a new gesture.

- **Bird rename with expected metadata revision:** The plan prevents silent overwrites and preserves identity.

- **Adoption read and idempotent accept:** The plan lets the current age-based offer be accepted once under the aviary lock.

- **Notebook keyset pagination:** The plan preserves immutable entries newest first, with stable pagination and no archive boundary.

- **Invitation and visit APIs:** The plan separates owner sharing commands from visitor render-only snapshot and liveness calls.

- **Snapshot envelope excluding hidden/private state:** The plan keeps vectors, filter values, presence totals, visit frequency, raw events, trait names, email, and notebook history out of ordinary snapshots.

- **Visitor projection:** The plan gives visitors the same bird/environment projection but omits owner-only capabilities, private account fields, and notebook data.

- **Durable receipts and idempotency:** The plan distinguishes accepted, duplicate, processing, applied, and rejected; identical retries return the original result and different payloads with the same ID are rejected.

- **Bounded long-polls:** The plan limits waits, reauthorizes before returning, wakes on revocation, and avoids holding a database transaction or connection while waiting.

- **Direct error language:** The plan keeps expired links, timed-out sessions, unavailable visits, loading failures, and conflicts in the system layer rather than making birds explain failures.

**Canonical simulation and multi-device correctness**

- **Independent scheduled ticks for every live aviary:** The plan keeps unattended accounts advancing and spreads load with stable phase offsets.

- **Locked tick transaction:** The plan prevents last-write-wins personality and applies each committed event prefix once against locked vector/filter/action state.

- **Server-authored nonnegative personality deltas:** The plan says no client can submit competing absolute values and crashes before or after commit do not double-apply deltas.

- **Deterministic random streams:** The plan makes retries produce the same result and preserves behavior identity across deployment.

- **UTC simulation boundaries with local display anchors:** The plan avoids browser clock manipulation and DST double execution while preserving day/night and daily mood anchors.

- **Backlog processing:** The plan refuses to drop missed days or defer all inactive-account simulation until someone opens a tab.

- **Analytical batching only after equivalence tests:** The plan allows future optimization only if it preserves minute-by-minute behavior.

- **Stale snapshot handling:** The plan prefers retaining the last valid scene with matter-of-fact status over inventing replacement simulation or resetting birds.

- **Client highest canonical_version tracking:** The plan discards out-of-order responses and uses server version, not client timestamps, for freshness.

- **Hidden-client shutdown and fresh resume:** The plan saves CPU/battery, avoids replaying missed calls, and samples the current phase on return.

- **Clock-offset mapping and slew:** The plan aligns action timing to server_now and local monotonic clocks while preventing phone wall-clock changes from advancing the engine.

- **Render interpolation boundaries:** The plan lets clients smooth authoritative poses but not choose new mood, perch destination, call, or personality.

- **Short in-memory offline retry queue:** The plan discards stale presence and expired offers/arrivals so offline sessions do not replay as fresh attention.

- **Concurrent command ordering:** The plan orders rename, adoption, offer, settle, resume, and revocation through committed events and locks rather than merging device state.

**Presence and attention accounting**

- **Visible/focused/recent-active owner predicate:** The plan uses this conjunction because presence is the principal input and must not be created by open tabs, visitors, audio, animation, or read receipts.

- **Five-minute recent-activity window:** The plan calls it “deliberately generous to still watching,” so continual mouse movement is not required.

- **Trusted input evidence without key values or pointer paths:** The plan records enough to validate activity while not collecting typed content or invasive tracking.

- **Boundary listeners and fifteen-second heartbeat:** The plan ends intervals exactly on blur, hidden state, activity expiry, settle, or close.

- **Lost unreported time on disappearing browsers:** The plan favors a “small undercount” over guessing or manufacturing unattended hours.

- **Server validation of bounded presence intervals:** The plan clips or rejects impossible, stale, visitor, revoked, deleted, future, or terminal-crossing reports.

- **Ninety-second delayed-delivery grace:** The plan accepts explicitly reported coverage briefly, never extrapolated heartbeat gaps.

- **Union across owner devices/tabs:** The plan ensures laptop plus phone overlap counts as real elapsed attention, not multiplied attention.

- **Listen-in bounded by live qualifying presence:** The plan prevents bird-specific attention from exceeding total owner presence, even across simultaneous devices.

- **Private presence measurements:** The plan says presence is never displayed as visit frequency, exported as a visit log, or sent to telemetry.

**Bird engine implementation**

- **Normalized rolling 24-hour drift inputs:** The plan uses validated coverage rather than click totals to reflect attention without becoming an optimization game.

- **Dominant presence contribution:** The plan keeps P at 0.85 in all trait inputs so offers and listening cannot replace quiet, qualifying presence.

- **Offer exposure and acceptance effects:** The plan says exposure can influence boldness, while acceptance specifically influences curiosity.

- **Settle, mute, captions, reduced motion, screen-reader use, and absence as nonnegative:** The plan prevents access choices, evening closure, and time away from harming birds.

- **Seven-day low-pass and continuous-time integration:** The plan makes change slow, persistent, and independent of API call count or worker wakeups.

- **Calibration gates for seven days, twenty-one days, isolated session, high offers, and absence:** The plan uses these to make drift measurable/perceptible over time without visible stats or negative absence effects.

- **Five initial moods:** NOT RECOVERABLE FROM PLAN

- **Mood dwell, daily baseline, and decaying impulses:** The plan preserves continuity across midnight, sessions, devices, and restarts without forced neutral resets.

- **Wary as transient environmental response:** The plan says wary is never a punishment for not visiting.

- **Rolling 120-second action window:** The plan gives clients a bounded future timeline and prevents abrupt reversal of in-progress motion.

- **Semantic perch zones with separation and visibility:** The plan lets final pixels adapt responsively while keeping all birds visible and not client-chosen into different zones.

- **Bird-to-bird responses and capped reaction chains:** The plan creates living social behavior while preventing alarm cascades and an undifferentiated chorus.

- **Return-greeting leader selection:** The plan varies the greeting through identity preferences, boldness, warmth, mood, and absence length without announcing absence.

- **Prompt but staggered greeting timing:** The plan keeps arrival noticed within two seconds while avoiding the whole aviary greeting on cue.

- **Greeting descriptor variation guard:** The plan prevents repeated complete descriptors and rejects a fixed clip library.

- **Offer popover with seed, song fragments, and still pool:** The plan keeps offerings limited and concrete, with fragments synthesized procedurally rather than recorded or uploaded.

- **Server-chosen receiving bird:** The plan prevents current listen-in from forcing acceptance and keeps acceptance as bird behavior.

- **Per-bird offer cooldowns and daily caps:** The plan prevents repeated ignoring from bypassing the throttle and avoids timers, progress rings, scarcity, or punitive copy.

- **Offer response uncertainty:** The plan says curious/content birds may approach, wary birds may wait, and drowsy birds may stay; a successful request does not imply accepted gift.

- **Settle transition, undo, and resume:** The plan records settlement epoch, clips late presence, offers a five-second undo, and resumes ordinary local-time lighting only after deliberate owner action.

- **Nightjar-like full-night activity:** The plan keeps the place audibly alive at low density when most birds are resting.

- **Suggested editable names and skippable naming:** The plan allows names without making first setup a catalog or identity reroll.

- **Stable adoption offer identity:** The plan prevents refreshing or dismissing from rerolling the proposed species/identity.

- **Quiet adoption availability:** The plan avoids badge, countdown, completion meter, “earned” language, and automatic multi-bird spawning for old aviaries.

- **Unused species preference and seventh duplicate allowance:** The plan balances recognizable species variety with distinct stable individual call signatures.

**Frontend rendering and session surface**

- **Compact SVG scene and DOM controls:** The plan avoids a large game engine or GPU texture pipeline for seven birds.

- **Shared bird rig with species-specific shapes:** The plan preserves individual plumage and silhouette across moods.

- **Responsive safe rectangle:** The plan prevents clipping, scene scrolling, app zoom controls, and inaccessible hit areas while keeping seven birds keyboard reachable.

- **Fewer than 600 live scene nodes and one animation driver:** The plan keeps runtime bounded and avoids per-bird per-frame layout reads.

- **Initial authenticated HTML with current action phases:** The plan makes the first visible frame already mid-action and avoids waiting for settings, notebook, fonts, audio permission, or unused species data.

- **Quiet sky/field fallback:** The plan uses it honestly only when delivery is slow; it is not counted as first-bird success and does not fake a bird.

- **First-adoption fly-in exception:** The plan allows a soft arrival only for the real initial adoption moment and persists a marker so reloads never repeat it.

- **Hidden-tab return phase reset:** The plan avoids animating yesterday’s last rendered perch as though the intervening day happened in a second.

- **Day/night color curves:** The plan makes light follow stored account timezone with dawn, midday, evening, and night states.

- **Synthetic rain and wind:** The plan avoids real-world weather services and keeps weather mild, brief, canonical, and not task-like.

- **Bounded client-only ornaments:** The plan allows ambient leaves/feathers but shuts them down when hidden and prevents pointer-tracking spectacle.

- **Soft blues, greens, browns, and muted ochres:** NOT RECOVERABLE FROM PLAN

- **Four-icon top bar fading behavior:** The plan keeps chrome quiet but immediately discoverable on movement, keyboard activity, focus, open popover, persistent-controls preference, and coarse-pointer devices.

- **Bird activation only for listen-in:** The plan keeps offers, notebook, settings, settle, naming, and adoption out of floating scene buttons.

- **Reduced-motion preference before first paint:** The plan ensures system or explicit preferences are respected before motion appears.

- **Reduced-motion cross-fade surface:** The plan replaces continuous motion and parallax while retaining calls, captions, drift, mood, notebook observations, and greeting identity.

**Audio and procedural caption pipeline**

- **Shared call descriptor grammar:** The plan lets one descriptor drive synthesis and captions, tying audible and silent experiences to the same canonical event.

- **Immutable individual signature seed:** The plan preserves recognizability across weeks of drift, moods, reloads, renames, and deployments.

- **Descriptor variation tests plus human listening:** The plan says numeric variation alone does not prove calls feel alive.

- **One AudioContext per visible app instance:** The plan prevents per-bird/call contexts and resource growth.

- **AudioWorklet voice pool and no render-callback allocation:** The plan keeps procedural synthesis efficient and leak-resistant.

- **Scheduled call IDs and timing:** The plan maps canonical call time to AudioContext time, deduplicates refreshes, cancels future unsounded actions, and avoids animation-frame sound timing.

- **Per-bird gain, mild pan, headroom, and limiter:** The plan preserves legibility and avoids harsh energy, hard stereo jumps, and abrupt gating.

- **Overlapping synthesized chorus:** The plan creates chorus from independent phrases and neighbor responses, not recorded loops, while capping simultaneous calls.

- **Listen-in gain ramps:** The plan makes the target clearer without soloing or muting the other birds.

- **Settle audio quieting:** The plan gently reduces master level and lets phrases finish softly.

- **Night low-density audio:** The plan keeps the nightjar-like signature active so the aviary is still alive at night.

- **Silence with captions fallback:** The plan handles missing/blocked WebAudio truthfully, without MP3/OGG/sample-loop fallback or downloaded recordings.

- **Matter-of-fact sound controls:** The plan puts sound availability and mute controls in accessibility settings, not as aviary toasts.

- **Runtime call captions:** The plan generates prose from notes, contour, trills, pauses, intensity, and perch context, avoiding stock species/mood sentences.

- **Caption timing tied to audible onset:** The plan keeps captions synchronized when audio runs and avoids claiming sound was heard when muted/unavailable.

- **Collision-free nearby captions:** The plan keeps caller identity legible even on narrow layouts and avoids omitting callers to fit a budget.

- **Assistive current-caption region:** The plan supports readable current captions without forcing every caption into narration unless detailed call narration is chosen.

**Notebook, narration, keyboard, and visual access**

- **Deterministic curated notebook templates:** The plan avoids an external language model to keep voice controllable, factual, predictable in cost, and without third-party transfer of bird interactions.

- **Evidence-backed notebook predicates:** The plan requires factual evidence for relational wording and simpler truthful wording when evidence windows are incomplete.

- **Sparse notebook publication:** The plan uses 72-hour ordinary intervals, 24-hour noteworthy exceptions, and three entries per rolling seven days so entries appear every few days rather than every session.

- **Notebook content boundaries:** The plan describes birds and place, not visit frequency, session start, presence minutes, drift, streaks, achievement, or absence.

- **Persisted prose and names-at-observation:** The plan keeps renaming from rewriting the observer’s past.

- **Read-only notebook:** The plan provides no edit, delete, annotation, or visitor-writing endpoint, preserving notebook observations as generated field notes.

- **Running narration from projected state:** The plan uses the same scene state so prose matches visible actions, light, weather, and calls.

- **Idle narration every 30–60 seconds:** The plan gives screen-reader users a living scene without enumerating raw fields or animation frames.

- **Priority narration for greeting, offer, and settle:** The plan gives user-initiated or arrival behavior prompt observations while avoiding “welcome back” phrasing.

- **Bounded live-region queues:** The plan prevents stale idle prose and repetitive narration backlogs.

- **Narration controls and visible transcript:** The plan lets screen-reader users and caption-only users manage prose without competing redundant streams.

- **Roving bird focus group:** The plan makes all birds keyboard reachable in stable scene order and ties user-directed focus to listen-in.

- **Accessible bird names and naturalist descriptions:** The plan exposes identity and current behavior without vector numbers, trait debug names, tooltips, or debug accessibility surfaces.

- **Keyboard-operable offer/settings/notebook dialogs:** The plan preserves focus management, close controls, focus restoration, and shortcuts without intercepting typing or assistive modifiers.

- **Visitor accessibility without owner actions:** The plan lets visitors focus/read scene descriptions and adjust local preferences but not create hidden listen-in, offers, settle, or drift paths.

- **Contrast and access verification:** The plan makes captions, focus, touch targets, text scaling, keyboard traps, reduced motion, and muted/caption-only journeys release-blocking across day/night/weather states.

**Accounts, visits, exports, and erasure**

- **High-entropy magic-link tokens stored as digests:** The plan protects secrets, makes links single-use, and prevents token substitution between auth, invite, and export purposes.

- **Link landing page plus confirmation POST:** The plan prevents email scanners from consuming tokens or creating visits.

- **URL fragment and no third-party assets on token pages:** The plan keeps secrets out of logs/referrers and avoids leaking token pages.

- **Secure HttpOnly SameSite device cookies:** The plan avoids localStorage session credentials and supports per-device expiry/revocation.

- **Rate limits with non-enumerating wording:** The plan limits abuse without exposing whether an address exists or turning limits into aviary behavior rules.

- **Per-object authorization and CSRF protection:** The plan protects notebook pages, renames, invitations, exports, long-polls, and cookie-authenticated mutations.

- **Read-only visit invitations:** The plan requires deliberate owner-entered email, one-time token consumption, no persistent friend relationship, and no automatic repeat invitation.

- **Visitor render-only session:** The plan blocks event endpoints, notebook, renaming, adoption, offers, listen-in, settle, host presence, and UUID-based cross-account changes.

- **Same projection for visitors:** The plan avoids special plumage, celebration, greeting, host-online status, cursor, or co-presence indicators.

- **Coarse visit duration:** The plan tracks visible-session liveness for host transparency only and keeps it out of simulation inputs and behavioral analytics.

- **Visit revocation:** The plan invalidates sessions, wakes reads, reauthorizes every snapshot, stops rendering/audio, and clears in-memory snapshots.

- **Opt-in consumed-invite email:** The plan checks the setting again before delivery, sends one deduplicated transactional email, and disables mail-open/click tracking.

- **Repeatable-read JSON export:** The plan captures one canonical version with stable identities, names, species, vectors, moods, notebook entries, and settings.

- **Export exclusions:** The plan excludes secrets, raw interaction logs, presence totals, visit-frequency history, operational records, and other people’s invitation contact data.

- **Encrypted temporary export storage and short-lived download:** The plan requires the signed-in account plus random link, no-store responses, and no rendered traits table.

- **Mail provider minimal export knowledge:** The plan sends only delivery address and transactional link text, never the JSON archive or bird interaction data.

- **Recoverable deletion state:** The plan stops new interactions, invalidates invitations and export links, but continues unattended simulation so recovery restores the same birds rather than reconstructing them.

- **Hard deletion workflow:** The plan purges every account-owned store and UUID-linked operational record, rechecking deletion generation to avoid races.

- **Account-scoped key erasure:** The plan makes backup restore unable to resurrect hard-deleted accounts once keys no longer exist.

**Privacy boundary and operational observability**

- **Credential separation for metrics:** The plan prevents the analytics/metrics collector from querying simulation state.

- **No browser replay, click tracking, heatmaps, third-party analytics SDKs, or payload-capturing errors:** The plan blocks tooling that could capture private relationship data.

- **Telemetry allowlist:** The plan strips URLs, auth, bodies, email, bird names/IDs, vectors, mood, calls/actions, invite recipients, offer types, and presence/listen timing.

- **Aggregate operational metrics:** The plan collects request health, scheduler lag, first-bird render time, audio failures, coarse page-view histograms, and job queues without account or bird dimensions.

- **Minimum aggregation groups and bounded cardinality:** The plan prevents sparse operational combinations from identifying accounts.

- **No production relationship analytics:** The plan forbids population-average drift, per-species interaction rates, most-offered birds, active relationship charts, visit streaks, rankings, recommendations, and training examples.

- **Synthetic monitoring with fictional aviaries:** The plan verifies first render, interactions, browsers, and error surfaces without using real-user data.

- **Plain-text privacy policy:** The plan names allowed aggregate categories and explicitly excludes per-bird interaction state.

**Performance budgets and measurement**

- **Initial JavaScript ceiling and smaller engineering target:** The plan protects the first-bird load path by lazy-loading settings, accessibility settings, notebook history, invitations, and unused species detail.

- **Compact snapshots:** The plan uses compact action/grammar descriptors instead of waveform data, full notebook history, event history, or vector models in HTML.

- **Time-to-first-bird metric:** The plan measures the first painted real bird, not quiet field, hydration, or placeholders, because fake birds and unsafe shared caches would violate the experience and privacy boundary.

- **Mid-tier mobile 4G test profile:** The plan validates the supported envelope on physical hardware and representative network conditions rather than warm-cache assumptions.

- **Seven-bird runtime frame budget:** The plan keeps sustained 60 fps on older hardware through pooled ornaments, rigs, curves, bounded queues, and one animation driver.

- **Thirty-minute soak:** The plan catches retained heap, node, context, timer, and page-count growth across audio, interactions, notebook, resize, visibility, and reduced motion.

- **Hidden-document zero rendering:** The plan saves CPU/battery while server ticks continue and reopen reconstructs current state.

- **Tick computation and scheduler alerts:** The plan treats a fast tick run an hour late as unhealthy and alerts on lag, backlog, failed projections, and stopped unattended advancement.

- **Interaction latency target:** The plan keeps event-to-authoritative-reaction-start responsive while measuring receipt, tick, snapshot, and interpolation separately.

- **Last two major browser versions:** The plan uses feature detection and silent-caption fallback rather than legacy bundles that would defeat the size budget.

**Implementation sequence, release gates, verification, and risks**

- **M0 contracts and sensory prototype:** The plan front-loads tokens, silhouettes, grammar, voice lexicon, reduced motion, schemas, and exception decisions so divergent surfaces do not get built.

- **M1 durable two-bird vertical slice:** The plan proves one persisted aviary across two clients, restarts, hidden return, first-bird measurement, and accessibility smoke before broad buildout.

- **M2 full interaction and drift engine:** The plan waits for synthetic 7/21/absence fixtures and duplicate/crash/concurrent checks before treating drift and interaction as complete.

- **M3 full audiovisual and accessible product:** The plan requires human signature/variation review, VoiceOver/NVDA journeys, viewport tests, and reduced-motion scenarios before completion.

- **M4 account and visit lifecycle:** The plan gates lifecycle completion on authorization, one-use token races, read-only visits, consistent export, and deletion/restore tests.

- **M5 operational hardening:** The plan gates on telemetry privacy, worker failover, load, soak CI, browser matrix, and synthetic monitors.

- **M6 controlled release:** The plan requires lived pacing, copy review, rollout cohorts, rollback behavior, support surfaces, and complete v1 capabilities.

- **Early audio, accessibility, and engine work:** The plan says these constraints should start in the first vertical slice because they can change architecture while it is still inexpensive.

- **Bird-count ramp:** The plan stages two, then three/four, then seven birds so mixing, layout, soak, signature, and accessibility review can pass before public v1.

- **Feature flags for adoption and rollout:** The plan uses flags only to gate new availability/code, never to remove or hide already adopted birds.

- **Rollback preserving existing birds:** The plan forbids restoring old database snapshots for visual/engine rollback because UUIDs, vectors, and seven-bird accounts must survive.

- **Versioned migrations and repair path:** The plan keeps old fixtures, expand/contract compatibility, future-only engine changes, and recognizability checks instead of reseeding birds.

- **Deterministic rollout cohorts:** The plan uses synthetic IDs and operational configuration, not relationship behavior or engagement scores.

- **Release-blocking evidence:** The plan requires invariants, event-order survival, week/three-week drift intent, keyboard/assistive routes, performance budgets, privacy payload checks, and irreversible hard deletion.

- **Verification matrix:** The plan defines required scenarios so implementation can prove presence, events, ticks, mood, identity, greeting, offers, settle, adoption, notebook, rendering, reduced motion, audio, captions, auth, visits, export/deletion, telemetry, and performance.

- **Property, integration, audio, and human review:** The plan uses property-based tests for invariants, database tests for concurrency/auth, audio tests for clipping/timing/caption agreement, and human review for recognition, uncanniness, and intended experience.

- **Risk stop conditions:** The plan says to block release, hold bird-count ramp, stop rollout, revoke faulty sharing, or stop faulty pipelines when drift, presence, identity, audio, accessibility, privacy, telemetry, or performance violate the intended constraints.
