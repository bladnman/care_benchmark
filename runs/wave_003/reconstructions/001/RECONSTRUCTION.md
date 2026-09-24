## System-level intent

- **Vocabulary is part of the product architecture.** The plan says vocabulary follows the glossary exactly so it stays "intact in the codebase, in review comments, and in the heads of new contributors." This shows up again in terminology enforcement, voice lints, the distinction between *call* and *song fragment*, and the use of code identifiers like `listenIn`, `settle`, `offer`, and `Call`.

- **The product's value is affective, not managerial.** In the summary, Pocket Aviary's value is named as "affective." The aviary should feel like "a place that was already going before the tab opened," should "notice the user and never announce anything," and should change "over weeks, not within a session." This intent drives the slow tick, no spinners, no toasts, no counters, procedural greetings, quiet field loading, and refusal tests.

- **Product feelings are treated as hard system properties.** The summary says the plan treats those feelings as "a hard system property," and the invariants matrix explains that important product rules are enforced "through architecture rather than code review." This appears throughout the plan as database grants, absent API routes, absent component primitives, CI gates, lints, and boundary tests.

- **The server decides what is true; the client decides how it looks and sounds.** This phrase is a key bet and is repeated in the render pipeline boundary: the snapshot says "what is true now and what is scheduled to happen soon," while the client creates a realization. The purpose is to keep canonical state safe while allowing procedural, freshly seeded motion, calls, captions, and narration.

- **One canonical aviary, single writer, no last-write-wins for bird state.** The plan uses a "server-authoritative simulation on a slow tick" so the aviary continues without the viewer, syncs across devices, and avoids write conflicts. In the invariants, last-write-wins is rejected because a stale device could make "a morning's drift" disappear.

- **Drift is honest, monotonic, lagged, and non-punitive.** The summary calls drift a "monotonic, lagged low-pass filter over honest presence." I1 protects honest presence from tab-open inflation; I2 frames monotonic drift as the engine-level form of "not a Tamagotchi." Recent absence may make birds quieter, but neglect never lowers traits and never makes a bird wary "of the user."

- **Privacy wins over curiosity.** The summary says production analytics cannot verify drift because "the privacy commitment forbids that." Privacy is an "architectural boundary": synthetic UUIDs, encrypted email, no network path from per-bird interaction data to analytics, no third-party analytics, and no production measurement of drift or behavior.

- **Refusals are enforced by absence.** The plan explicitly says "features that would break the product are hard to add by construction." This appears in the absence of toast, badge, spinner, counter, streak, trait-setting, placement, notebook-write, and notification primitives, plus a refusals register and refusal tests.

- **The relationship is observational, not custodial.** The out-of-scope table rejects Tamagotchi mechanics because "the relationship is observational, not custodial." This intent also grounds monotonic drift, no hunger or health meters, no visible distress, no penalty for absence, no "you didn't settle" state, and seeds as gestures rather than food.

- **Bird identity and recognizability are sacred.** The plan says weeks of drift matter only if "the bird is still the same bird." Stable `bird_id`, immutable ceilings and call signatures, signature-stability regression, and per-bird ceilings all protect identity over long spans.

- **Aliveness comes from procedural variation, not canned media.** I8 and the audio principles reject entry animations, spinners, recorded audio, loops, and identical greetings. The plan calls a looped call "the audible signature of dead software" and requires warm-started controllers, procedural calls, anti-repetition memory, and freshly sampled behavior.

- **Accessibility is a designed surface and a launch gate.** The summary says accessibility ships in v1 "as designed surfaces." Section 13 says it should "carry the charm," rejects the cheap version, and states that it ships in v1 "or v1 doesn't ship." Narration, captions, reduced motion, keyboard model, contrast, focus, and assistive-technology fairness are therefore core product intent, not follow-up work.

- **Voice is split by what the user is doing.** I11 and the voice section divide surfaces into naturalist and matter-of-fact registers. If the user engages with birds and the aviary, the prose is naturalist; if the user engages with the system as a system, it is matter-of-fact. This protects charm without letting bird metaphors replace usefulness.

- **Performance is part of aliveness.** The first bird under 500 ms, mid-action, without a spinner is a key bet. Budgets are CI gates because a slow or memory-leaking scene would break the sense that the aviary was already there. The performance section says budgets drive procedural audio, compact vector parts, Canvas 2D, code splitting, edge-inlined snapshots, and warm-start rendering.

- **Social access is deliberately inert.** Visits are scoped as read-only and inert. Visitor watching must not drift birds, trigger greetings, create co-presence, or produce a show-off mode. This intent appears in I14, the visitor build, visitor token scope, and the rejection of social-network surfaces.

## Per-feature whys

### Scope and refusals

- **Browser-only aviary:** the plan's rationale is one team and one client; "engine quality beats a second client," and protocols should not be shaped around native constraints.

- **One scene per account:** the plan ties this to one canonical aviary, one user, and one designed scene. It supports the single-record sync model and avoids shared or multi-aviary account complexity.

- **Two starter birds:** the plan says the starters are chosen to be complementary, with different silhouettes and separated call registers, so the user meets distinct birds rather than choosing from a catalog.

- **Age-gated newcomers up to a hard cap of 7:** the rationale is that new birds should arrive only with aviary age, not visit counts, interaction data, or payment. The cap of 7 is empirical and depends on recognizability.

- **Three perch zones:** NOT RECOVERABLE FROM PLAN

- **Local-time day/night:** the aviary follows the host's local time so its light and waking rhythms belong to the host's aviary. The plan uses the IANA timezone and `zone.tab` approximate latitude to avoid collecting location.

- **Rare ambient weather:** weather is per-aviary so all host devices and visitors see the same rain. It is not linked to real-world weather because that would require location and "turn weather into a feature."

- **Ambient leaf and feather drift:** the plan uses these as quiet motion cues, especially in the quiet-field loading state, to preserve aliveness without a spinner.

- **Responsive layout that never crops a bird:** the rationale is that "the aviary always fits at a glance" and every bird must remain visible, targetable, and readable as part of a one-screen scene.

- **Five-trait hidden personality:** the rationale is affective change without exposing stats. I5 says if the user sees "boldness: 0.62," the bird becomes "a stat to manage."

- **Monotonic lagged drift:** the plan uses this so drift is visible over weeks, no single session is visible, and absence never punishes the user. It is the "not a Tamagotchi" guard.

- **Mood FSM with cross-session persistence:** mood is persisted so opening a tab never resets the birds. The user starts in whatever mood the tick has carried forward.

- **Procedural call grammars:** the rationale is recognizability without repetition. Calls are generated from grammars so no call is ever played the same way twice.

- **Bird-to-bird interaction:** the plan uses mood contagion, call-response, and chorus windows to make the aviary social and alive while the server models the social layer coarsely and the client realizes it finely.

- **About six-species pool:** NOT RECOVERABLE FROM PLAN

- **Naming and renaming:** renames touch only `name`; identity continuity outweighs strict historical records, and old notebook entries render the current name so they do not look like they are about another bird.

- **Stable identity:** the plan says "weeks of drift only mean something if the bird is still the same bird." Immutable IDs, ceilings, signatures, and signature-stability regressions protect that.

- **Return-greeting:** the birds are the welcome. The plan rejects welcome text and uses procedural, staggered greetings so the aviary notices the user without announcing arrival.

- **Listen-in:** the rationale is focused attention without turning birds into "soloable tracks." The mix re-balances, other birds stay ambient, and drift credit only applies within honest presence.

- **Offer:** offers are gestures made to the aviary, not commands aimed by clicking a bird. The server chooses the recipient, and cooldowns keep curiosity drift from collapsing into a one-session grind.

- **Settle:** the plan treats settle as a quiet goodbye. It closes presence, quiets mood, and shifts local lighting without creating a system surface, penalty, or shared dimming across devices.

- **Field notebook:** the rationale is to let the aviary "notice" specific moments and long-horizon changes as a watcher would, without quantifying them or writing about the user's behavior.

- **Presence accounting:** presence-time is the dominant drift input, so it must be honest: visible, focused, and recently active. A laxer definition would silently speed drift for every account.

- **Magic-link sign-in:** the plan gives magic link as the v1 auth model and defers SSO/passwords. It also avoids enumeration by returning identical responses and consumes links by POST to survive mail scanners.

- **Per-device revocable sessions:** the session list is a security list, not a visit history. Revocation is immediate and uses day-granular activity to avoid creating frequency surfaces.

- **Verified email change:** the old address keeps working until verification so the account is not stranded by an unverified change; the swap happens atomically.

- **JSON export:** the plan treats export as a quiet quality-of-life feature. It includes personality only as a sealed blob to satisfy inclusion while preserving "the user never sees these numbers."

- **Thirty-day soft delete to hard delete:** the soft window protects against accidental regret; hard delete honors that interaction history belongs to the user.

- **Server tick:** the tick lets the aviary continue whether or not anyone is watching, gives multi-device sync, and makes write conflicts unreachable.

- **Snapshot pull and interpolation:** the snapshot is the boundary between truth and realization. Overlapping timelines and natural reconciliation prevent snaps and teleports.

- **Append-only event log:** events are additive and ordered by the server, so stale devices cannot overwrite drift and retries remain idempotent.

- **Multi-device by construction:** the plan's rationale is that every client reads the same canonical record and appends events; there is no client-to-client merge or client-owned state to reconcile.

- **Visit invitations:** visits are opt-in, per-invite, email-addressed, revocable, expiring, and read-only so they do not become profiles, public aviaries, permanent visitor lists, or a social network.

- **Visit log:** the host can see who visited with approximate duration, but there is no badge or push. Duration comes from snapshot pulls and requires no visitor telemetry.

- **Opt-in visit email:** it is the only exception to no notifications because the host explicitly asks for it; email is the channel because push notifications are out.

- **Screen-reader narration:** narration carries the charm by turning the same scene state the renderer uses into naturalist prose, instead of exposing state lists or traits.

- **Call captions:** captions are generated from the same descriptor that drives synthesis so they cannot describe a different call than the one heard.

- **Reduced-motion mode:** it is a designed surface, not "animations off"; it preserves mood, calls, drift, and notebook while replacing motion with slow still-pose cross-fades.

- **Full keyboard model:** the rationale is parity for listen-in, offers, notebook, settings, and focus without losing orientation when birds move.

- **WCAG AA contrast and visible focus:** the plan makes these launch gates so accessibility is not deferred and focus remains readable across midday, night, and weather.

- **Matter-of-fact unsupported-browser page:** unsupported browsers are handled by direct system copy because this is a system surface. Missing WebAudio is not unsupported because silence with captions is preferred to blocking the aviary.

- **No native iOS/Android apps:** the explicit reason is "one team, one client"; engine quality matters more than a second client.

- **No gamification:** the plan rejects achievements, streaks, levels, scores, badges, XP, counters, and calendars because they would turn the relationship into management and break the product.

- **No Tamagotchi mechanics:** the reason is explicit: "the relationship is observational, not custodial."

- **No social-network surfaces:** the plan rejects profiles, follows, feeds, discovery, friend-of-friend, comments, chat, avatars, leaderboards, and show-off mode because they would make visits a larger social product.

- **No payments or tiers:** birds and features are never gated by money; newcomer pacing uses aviary age, not payment.

- **No shared or customizable scenes:** the plan protects "one user, one aviary, one designed scene."

- **No push notifications or re-engagement email:** the reason is that Pocket Aviary "never reaches for the user about the aviary."

- **No SSO or passwords in v1:** NOT RECOVERABLE FROM PLAN

- **English-only v1:** the naturalist voice is hand-authored, and every language would need its own writer.

- **Refusals register:** the rationale is that well-meaning contributors will propose product-breaking features, and recording rejections means "the argument never has to be re-run."

- **Terminology enforcement:** the rationale is to keep the plan's vocabulary intact across code, strings, specs, reviews, and contributor thinking.

### Architecture, data, and API

- **Edge-streamed HTML with inline snapshot:** the rationale is first-bird speed and the sense that the aviary was already there. The sky can paint immediately in local time and the renderer receives state early.

- **Modular monolith API:** the plan says this is "the right shape for one team": the bird engine gets attention, not service plumbing.

- **Simulation workers as only holders of `sim_tick`:** this enforces that only the tick writes personality, mood, and perch state, preventing stale overwrites and double application.

- **Jobs worker:** NOT RECOVERABLE FROM PLAN

- **Separate identity, aviary, and telemetry zones:** the rationale is that per-bird interaction data should drive only that user's simulation and have no route or credential into analytics.

- **Synthetic UUID identifier policy:** random UUIDv4s leak no creation time and keep email-derived identifiers out of foreign keys, logs, queues, shard keys, and telemetry.

- **Email stored once, encrypted, with blind index:** the rationale is privacy: email exists only in identity, encrypted, with lookup possible without spreading email-derived data.

- **Different HMAC key for rate limiting:** the plan uses a different key so rate-limit keys cannot be joined against the accounts table.

- **Aviary database stores personality as canonical state:** the reason is that losing or regenerating a vector deletes "the bird the user has been getting to know."

- **Personality journal of applied outputs:** it exists for audit and disaster recovery, not re-derivation, because personality must not be recomputed from history.

- **Presence ledger retained but never displayed or exported:** the rationale is that presence is only an input to the user's own birds and must not become a streak, calendar, count, or exportable history.

- **Aviary facts as notebook input:** the writer reads only aviary observations so entries cannot mention user behavior.

- **Static catalogs for species, motifs, and name suggestions:** NOT RECOVERABLE FROM PLAN

- **No endpoint for traits, mood, perch, notebook writes, or aviary lists:** those absences enforce no stat management, no user placement, no client-owned bird state, and no editable notebook.

- **Auth link GET landing page and POST consume:** the rationale is scanner safety: corporate mail scanners may prefetch GET links, so GET does not consume the token.

- **Snapshot ETag keyed by tick and overlay revision:** NOT RECOVERABLE FROM PLAN

- **Display overlays for unconsumed offers:** a second device can see the same seed before the tick makes it canonical, preserving cross-device continuity.

- **Interaction batching and beacon endpoint:** the plan's rationale is reliable append of events, including final presence intervals on hide or pagehide.

- **Offer endpoint decides reactions synchronously:** this lets the client animate immediately while the server still decides what happened. A timeout produces no drift because the server never decided the offer.

- **Bird rename API as the only client-writable bird field:** the rationale is that names can change without changing personality, mood, or call.

- **Notebook read-only API:** notebook entries are observations, not user-authored or editable content; no write endpoints exist.

- **Settings API with account-level preferences:** account-level accessibility preferences follow the user across devices; device-local mute and volume are not sent because muting a phone should not mute a laptop.

- **Visitor snapshot omits host-only actions:** visits show the host's aviary as read-only; visitor tokens cannot mutate or influence birds.

### Simulation engine

- **Pure `step(state, events, Δt, rng)`:** purity gives deterministic replay, idempotent retries, and a calibration harness that runs production code on synthetic traces.

- **Shard leases and fencing tokens:** the rationale is safe tick ownership; stale workers fail compare-and-set and deltas cannot be applied twice.

- **Counter-based deterministic RNG:** rerunning a tick yields identical output, so retries are safe and replay tests are exact.

- **Continuous-time catch-up:** outages recover correctly without a visible burst of flights, and presence logged during the outage is still credited.

- **Dormant cadence cost lever not enabled at launch:** the plan follows "whether or not any client is connected" literally, keeping a coarser dormant cadence only as a signed-off cost lever.

- **Per-bird trait ceilings:** with monotonic drift, universal ceilings would make long-lived aviaries converge on identical birds; per-bird ceilings preserve individuality and recognizability.

- **Daily saturation of presence input:** the first minutes count most, and a marathon visit only contributes modestly more, guarding against Tamagotchi-fast drift and mouse-jiggler inflation.

- **Reservoir release lag:** spreading a session's effect over roughly a week makes no single session visible and lets drift continue during absence from prior inputs.

- **Headroom falloff:** growth slows near each bird's ceiling so long-run change does not flatten into every bird maxing out.

- **Listen-in drift input:** it is the strong per-bird attention signal, credited only where it intersects presence.

- **Offer drift input:** accepted or nearby offers create small curiosity and boldness steps so offers read as gestures rather than progression mechanics.

- **Muted and caption-only drift treatment:** captions count as perceiving calls so deaf and caption-only users are not disadvantaged; muting slows vocal drift but never subtracts.

- **Per-bird offer cooldown and daily cap:** without them curiosity drift would saturate within one session; with them, an offer remains a gesture.

- **Mood states:** the plan chooses alert, curious, content, wary, drowsy, and settled so time of day, weather, interactions, and personality can shape visible expression. The night-rest mood `settled` is separate from settle lighting to avoid a code collision.

- **Soft dawn re-anchor:** the rationale is a daily-ish reset that still avoids snaps; birds wake one by one if watched at dawn.

- **First-hour starter exception:** starter birds are held awake and curious/alert so a first session at night is not a dark scene.

- **Absence excluded from mood valence:** wary never comes from the user having been away, preserving the non-punitive relationship.

- **Recent attention scalar:** it lets birds become "ambient" after absence by lowering expression amplitude, while traits remain monotonic and mood never becomes mistrustful.

- **Perch choice and intents:** perch position is a signal the user reads, shaped by boldness, mood, sociality, and recent attention; the user cannot control layout.

- **Dwell times of minutes:** the scene stays calm rather than twitchy.

- **Overlapping intent windows:** a client that misses one pull still has a continuous timeline.

- **Mood contagion:** a wary mood can spread to nearby birds, making the group feel socially connected.

- **Call-response:** warm, vocal neighbors answer, creating social sound without the client inventing canonical events.

- **Chorus windows:** outside chorus windows birds bias toward turn-taking; inside them overlap rises, so chorus is special and avoids constant audio masking.

- **Weather planning:** planning a day ahead lets host devices and visitors share the same weather.

- **Weather effects small and short-lived:** rain and wind add atmosphere without becoming mood destiny or a feature state.

- **Host timezone confirmed twice when traveling:** the rationale is to avoid timezone and DST oddities by confirming reports and easing light transitions.

- **No location collection for daylight:** approximate solar position comes from `zone.tab`, giving seasonal light without collecting location.

- **Greeting propensity and style:** propensity lets bolder or more attentive birds greet first without certainty; stable style lets each bird greet in its own way while procedural variation prevents repetition.

- **Greeting history:** it prevents identical greetings across devices and sessions.

- **Newcomer appearance on the back perch:** this reconciles a species offer with "notice, never announce"; the newcomer appears in-world with no notification, badge, or modal.

- **Newcomer adoption through offer menu:** adoption is an offer of a perch, not acquisition or unlocking, and declining or doing nothing carries no penalty.

- **Notebook detectors:** they choose aviary facts that are novel, specific, or rare, so entries read like observations instead of metrics.

- **Notebook sparsity governor:** it prevents fatigue and keeps very active users from getting more entries simply because they used the product more.

- **Notebook template exclusion:** excluding templates used in the last 60 days keeps the prose from becoming recognizable machinery.

- **Calibration harness:** because production analytics are forbidden, the harness and consented dogfood are the instruments for drift, absence, multi-device, muting, and long-run behavior.

### Sync, presence, and frontend scene

- **Single canonical record sync model:** multi-device sync is "a property of this shape," not a separate feature, because every device reads the same record and appends events.

- **Append-only conflict semantics for events:** additive events consumed in ingest order make the lost-drift scenario unreachable.

- **Offer race semantics:** cooldown decides races so simultaneous offers do not create duplicate reactions.

- **Local-only settled lighting, listen-in focus, panels, mute, volume, and anti-repetition memory:** these are properties of "this viewing," not canonical aviary state.

- **BroadcastChannel audio leader:** it prevents two tabs from doubling the chorus.

- **Offline local idle continuation:** the plan avoids an error surface while the scene is still fine; after two minutes it uses a quiet matter-of-fact line.

- **Offline offer disabling:** offers need a connection because the server decides reactions and undecided offers produce no drift.

- **Presence monitor three signals:** visible, focused, and recently active together protect "honest presence" and prevent hidden tabs, unfocused windows, and missing data from inflating drift.

- **Four-minute activity window biased long:** watching without moving is the product itself, so presence should not drop instantly for quiet watchers.

- **Touch `pointerdown` and keyboard `focusin` as activity:** both are signs of a person present, and `focusin` protects screen-reader users whose raw key events may not reach the page.

- **Presence requires scene at least partly on screen:** a full-screen settings sheet pauses it because attention is not on the aviary scene.

- **Fail-closed server crediting:** malformed, overlapping, late, or visitor intervals are dropped because under-counting is better than inflating drift.

- **Presence is never shown, exported, summarized, or aggregated:** the rationale is to keep attention from becoming engagement analytics or a visit-frequency surface.

- **First-bird boot path:** first frame must be the aviary, already mid-action, because an entry animation or spinner would announce loading instead of aliveness.

- **Quiet field loading state:** if the snapshot is late, the user sees local-time sky and subtle motion rather than a spinner.

- **Service Worker cache for returns:** the shell and assets return quickly, but the snapshot is fresh so old timelines do not cause visible correction.

- **Canvas 2D renderer:** the plan says it is sufficient for 60 fps at 7 birds, avoids WebGL shader-compile stalls, and costs less bundle.

- **Layered scene composition:** NOT RECOVERABLE FROM PLAN

- **No pointer-driven parallax or panning:** the scene is not a "layered illustration showing off"; it stays calm and one-screen.

- **Palette lint:** the scene uses calm naturalist tokens and rejects saturated accent colors.

- **Layout solver never scrolls, pans, or zooms:** the aviary should always fit at a glance.

- **Idle micro-motion system:** breathing, preening, scanning, fluffed feathers, and other small variations keep birds from reading as paused.

- **Beak and body motion driven by actual call envelopes:** birds visibly produce what the user hears.

- **Rendering stops when hidden:** simulation continues on the server; on return the client rebuilds from truth instead of running hidden work.

- **No hover affordances or glow for listen-in:** the scene carries no chrome, and focused birds remain natural rather than labeled or highlighted.

- **Greeting stagger and no unison:** synchronized greetings would announce the user's arrival, so birds greet with random offsets.

- **Offer presentation as in-world objects:** seed, song fragment, and still pool are placed in the scene, and natural limits replace visible cooldowns, timers, and counters.

- **Seeds as gestures, not food:** the plan explicitly says there is no feeding state, preserving the non-custodial relationship.

- **Top bar contents limited to account/settings, accessibility, Field Notebook, offer, and settle:** the rationale is to keep no badges, dots, counts, or status indicators inside the scene.

- **Settle in top bar:** the plan resolves a tension by making settle a fifth, equally quiet control; hiding it would bury the goodbye, and account settings would put a naturalist gesture in a system surface.

- **Top bar fade:** the top bar recedes so the aviary remains primary, while staying keyboard reachable and opaque on focus.

- **Adoption step:** the user meets "two birds have arrived" with descriptions and editable names; there is no catalog because the birds are being met, not configured as avatars.

- **Adoption step unlocks audio:** first-ever session audio can start because adoption supplies the unlocking gesture.

- **Empty aviary only during adoption:** after the starters arrive, no state removes all birds, preserving the living aviary.

- **Day/night rendering:** light follows solar elevation, with night still alive through breathing, murmurs, ambient motion, and possible nightjar activity.

- **Weather rendering without flashes:** it provides rain and wind atmosphere while respecting no flashing more than three times per second.

- **Hidden tab audio fade and suspend:** hearing calls from a background tab would disconnect hearing birds from visiting them and make the aviary feel notification-like; it also saves battery.

- **Visitor mode build:** compiling out presence, event writing, greetings, offers, settle, listen-in, notebook, and account chrome guarantees visits are inert and read-only.

- **Notebook panel memory discipline:** virtualization and releasing scrolled-out rows satisfy the "no retained references after scroll-out" requirement and protect memory budgets.

### Audio, voice, and accessibility

- **All sound synthesized in browser:** recorded loops would sound like dead software, phase against each other, and exceed the bundle budget.

- **AudioWorklet voice pool:** preallocated voices prevent per-call allocation and keep memory flat.

- **Native oscillator fallback:** if the worklet fails, the product still synthesizes; there is no recorded-audio fallback.

- **Graceful silence with captions:** when WebAudio is unavailable, silence with captions is preferred to canned audio.

- **Distance and zone mixing:** back-perch birds sound farther away and front-perch birds sound near, supporting scene depth.

- **Safari ambient audio category:** NOT RECOVERABLE FROM PLAN

- **Call signatures fixed at adoption:** the bird remains recognizable across mood and drift.

- **Signature distinctness within aviary:** pitch, motif, and vibrato separation matter especially when two birds share a species at seven birds.

- **Recognizability rollout gates:** the cap of 7 depends on human panels and classifier tests so product does not exceed what listeners can distinguish.

- **Signature-stability regression:** audio changes must not effectively swap a bird's voice.

- **Two-clock call scheduling:** timestamps go to the renderer so beaks align with the sound.

- **Anti-repetition in audio:** audible repetition would break the spell, so recent descriptors are rejected and resampled.

- **Turn-taking outside chorus:** this prevents chorus mud and mirrors how real birds avoid masking each other.

- **Song fragment timbre distinct from bird calls:** the offered fragment should never sound like a bird; bird responses borrow its contour into their own signature voice.

- **Listen-in floor:** other birds never go silent, so listen-in remains re-balancing rather than muting or soloing.

- **Autoplay handling without overlay:** audio runs from the first frame when allowed; otherwise it starts after first real input, with captions on. A "click for sound" prompt would be an announcement and a ready pop.

- **Captions on by default while audio is suspended:** the plan preserves access to calls when browser policy or errors prevent sound.

- **Caption scrims per light phase:** the rationale is reliable contrast in every lighting state.

- **Two content registers:** naturalist prose preserves the observer voice; matter-of-fact copy handles identity, settings, errors, privacy, and future money without bird metaphors.

- **Surface registry and string catalogs:** every surface declares its register so voice can be enforced structurally.

- **Naturalist lints:** they block announcement framing, second person, gamification lexicon, digits-as-stats, and generic lines.

- **Specificity check:** prose must reference concrete aviary facts so it does not become "your bird is happier."

- **Shared prose realizer:** notebook, narration, and captions sound like the same observer, especially for screen-reader users moving between surfaces.

- **Semantic bird overlay:** the canvas is hidden from assistive tech and birds are exposed as focusable prose descriptions, preserving access without exposing trait numbers.

- **Bird descriptions not live-updated:** they are re-authored when focus arrives so screen-reader focus does not become spammy or state-list-like.

- **Narration idle cadence:** sparse, randomized updates avoid a mechanical rhythm and live-region spam.

- **Priority narration for greetings, offers, settle, and listen-in:** user-initiated moments are observed promptly without interrupting the user's reading because the live region stays polite.

- **Captions as accessibility setting and fallback:** they are opt-in normally, default-on when audio cannot play, and can also feed the visible narration strip.

- **Single-key shortcuts can be disabled:** this addresses WCAG 2.1.4 and avoids shortcut conflicts while keeping efficient keyboard access.

- **Settle has no single-key shortcut:** the plan avoids accidental goodbyes.

- **Focus order frozen in the scene:** moving birds do not reshuffle keyboard orientation.

- **Focus target moves with the bird:** focus is never lost when a bird flies.

- **Fair presence for assistive-technology users:** the plan says under-counting AT users would ration product quality by ability, so `focusin`, long W, harness personas, and moderated sessions mitigate it.

### Accounts, privacy, social, performance, testing, and rollout

- **Magic-link account creation on consumption:** requesting links cannot create accounts for arbitrary emails.

- **No confirmation after sign-in:** the user lands directly in the aviary, preserving a quiet flow rather than a system announcement.

- **Refresh token rotation with reuse detection:** NOT RECOVERABLE FROM PLAN

- **Account device list day-granular activity:** this is a security list, not a visit history.

- **Unverified email change expiry:** NOT RECOVERABLE FROM PLAN

- **Export delivery by signed email link:** the verified address receives the export, the link expires, and downloads are limited, keeping export quiet and bounded.

- **Export excludes presence and session history:** visit logs and presence would be disguised streak surfaces.

- **Soft deletion strip:** deletion is a system surface and must be visible on every signed-in page during the recovery window.

- **Aviary keeps ticking during soft deletion:** restoring loses nothing and birds are never frozen.

- **Visits suspended during soft deletion:** visitor access should not continue while the host account is pending deletion.

- **Hard-delete saga with DEK destruction and tombstone:** this crypto-shreds encrypted PII in backups, removes UUID-keyed traces, and prevents restored backups from resurrecting deleted accounts.

- **No third-party scripts:** no analytics SDKs, replay, heatmaps, ads, or tag managers can leak or repurpose per-bird interaction state.

- **Support break-glass only:** individual aviary access requires an explicit user request, is read-only, time-boxed, and audited; no support dashboard exists.

- **Plain-text privacy policy:** it tells users aggregate telemetry categories and states that per-bird interaction state is not used for analytics, training, recommendations, or third parties.

- **Invitation lifecycle:** one-time acceptance, expiry, revocation, and 30-day active access avoid permanent visitor lists.

- **Host email shown in invitations:** the host is told this in the form; it makes the visitor email matter-of-fact and accountable.

- **Visitor's own accessibility settings stored locally:** visitors can adapt captions, reduced motion, narration, and volume without changing the host's aviary.

- **No prettified visitor rendering:** there is no show-off mode; visitors see the host's aviary as it is.

- **No co-presence:** co-presence would require a multi-user simulation with several people contributing presence and drift, a different and larger product.

- **Visitor revocation surface:** revoked or expired links show the same matter-of-fact unavailable page, avoiding extra social signaling.

- **Visit email send limit:** at most once per invitation per day, preventing notification-like spam even when opted in.

- **Visit abuse controls:** send caps, suppression, bounce handling, and fixed templates prevent invitations from becoming a spam vector.

- **Initial bundle and first-bird budgets:** these support aliveness; delays, spinners, and heavy bundles would make the aviary feel like software loading.

- **Idle motion and memory budgets:** the living scene must stay smooth and flat over a long session, not degrade as a user watches.

- **Snapshot payload budget:** the render boundary stays small enough for the first-bird and sync model.

- **Audio CPU budget:** chorus and synthesis must stay reliable on reference devices without underruns.

- **Synthetic monitoring:** scripted browsers validate first bird, audio unlock, snapshot freshness, event round-trip, visitor revocation, and magic-link delivery without behavior analytics.

- **Aggregate-only RUM:** the plan measures performance and failures without account dimensions, cookies, raw IPs, or behavior fields.

- **Session-duration histogram:** allowed only in anonymized buckets and not joined to anything.

- **Invariant-violation counters:** they are treated as error rates, not behavior metrics.

- **Deliberately unmeasured behavior:** no drift distributions, mood occupancy, offers/listen-ins by type, presence totals, greeting stats, notebook counts, retention cohorts, A/B tests, replay, or event-type metrics, because those would violate the privacy commitment and enable future product-breaking surfaces.

- **Engine property tests and golden replays:** they protect monotonicity, single-writer behavior, union crediting, cooldowns, ceilings, dwell, and deterministic state.

- **Perception and dogfood studies:** because "visible after ~3 weeks" is a feeling and wall-clock bound, it must be validated with consented real-time use, not production telemetry.

- **Chaos tests:** lease races, killed workers, replayed events, clock skew, partitions, and laptop-phone concurrency prove no event is lost or double-applied and drift matches the single-writer oracle.

- **Refusal tests:** automated tests make sure breaking features like trait write routes, notification components, spinners, permission requests, bird digits in ARIA, and return text cannot slip in.

- **Internal accelerated clock:** it is test-only and lets non-production environments exercise weeks of drift, newcomers, and seven-bird aviaries in hours.

- **Consented dogfood:** it is the only way to validate slow visible change as a feeling, so it starts as early as the vertical slice allows.

- **Closed beta by invite codes instead of email allowlist:** this avoids putting emails somewhere else.

- **Global `max_birds_enabled` safety valve:** it is operational, never per-user, paid, or behavior-based, and lets recognizability and chorus gates control the ramp.

- **If bird-ramp gates slip:** nothing is announced, so users do not experience lateness; the newcomer simply has not arrived yet.

- **Launch gates:** performance, accessibility, calibration, aliveness, voice, privacy/security, and sync must all pass because each is load-bearing for the plan's product intent.

- **Change management for drift, mood, and presence:** any change must answer whether it makes a session visible, punishes absence, or counts anything other than honest presence.

- **Flags and kill switches:** notebook, newcomers, visits, visit emails, AudioWorklet, and edge inlining can be disabled globally for safety; accessibility features cannot be killed, and flags are not used for per-user bird-mechanics experiments.
