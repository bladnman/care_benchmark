## System-level intent

- **Aliveness is the product.** The plan states this directly in the planning stance: "Aliveness is the product." It recurs in the first-frame requirements ("mid-motion or a quiet field"), the boot sequence ("No spinner. No logo splash."), simulation continuity ("the aviary continues without the viewer"), and the success story where "a bird notices them -- not a banner."

- **The aviary should feel already alive, not started by the user.** The plan repeatedly protects the feeling that "the aviary was already going when you opened the tab." This shows up in the quiet-field boot, the first bird drawn "mid-cycle," warm inlined snapshots, no fade-from-static, calls that can be "mid-call" on tab open, and catch-up that makes returning mid-rain possible.

- **Relationship is not engagement.** The plan rejects "gamification of any flavor," "Tamagotchi mechanics," streaks, XP, visit counts, and "your birds miss you." The success story says the user leaves and "nothing scolds them"; two weeks later the aviary is "quieter, not angry"; three weeks later nobody congratulates them.

- **Change should be slow, positive, and non-punitive.** The locked invariants say drift deltas are non-negative and neglect never decrements a trait. The drift section uses a low-pass of positive signals, a daily cap, and no "decay to mean." Absence produces lower EMA quietness, not duller birds.

- **The server owns the living state; the client performs the place.** The plan makes personality, mood, perch, weather, greeting selection, notebook prose, adoption eligibility, and authorization server-owned. The client owns scene graph, interpolation, WebAudio mix, presence monitoring, and optimistic UI, but "never source of truth." This is tied to multi-device sync and avoiding a client-side stats console.

- **Privacy is architecture, not a policy add-on.** The plan has a section named "Privacy architecture (not a policy add-on)." It separates Simulation and Operations planes, with no shared readers, no telemetry pipeline reading the simulation database, no analytics SDK that fingerprints users, no marketing pixel, and no dashboard that would let someone "reconstruct how a person is with their birds."

- **The product voice splits naturalist surfaces from matter-of-fact system surfaces.** The plan says errors are "matter-of-fact English, not naturalist," settings and account copy are matter-of-fact, while aviary, notebook, offer prompts, captions, and narration are naturalist. It forbids lines like "the aviary could not find its way back to you" and "your birds miss you."

- **Individuality comes from procedural, seeded variation, not canned assets.** The plan requires procedural calls, no recorded-call asset path, `signatureSeed` identity, motif variation, WebAudio synthesis, procedural captioning from the actual motif, and CI failure for `public/calls/*.mp3`. Rendering similarly uses pose seeds, local phase, and derived `renderProfile`.

- **Accessibility is a designed surface, not a state dump.** The plan says accessibility ships "as the same product in another register, not a bolt-on state dump." This appears in screen-reader narration composed as prose, reduced-motion as "a second rig," captions from real calls, keyboard navigation, visitor a11y, and AA requirements for chrome, captions, and focus.

- **Refusal is part of the implementation.** The plan's non-goals and risk response make refusal structural: no toast component, no unused score columns, no recorded fallback, no public discovery, no engagement experiment layer, no native wrapper, no stats drawer "for power users." The final line says, "Everything else is refusal."

## Per-feature whys

### Scope

- **Web-only client:** NOT RECOVERABLE FROM PLAN

- **Native apps out of v1:** The plan says not to design protocols or the data model around native-client constraints.

- **Single-user accounts and one canonical aviary per account:** NOT RECOVERABLE FROM PLAN

- **Generic magic-link acknowledgement:** The auth route always returns a generic acknowledgement so it does not reveal account existence.

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Hard cap of seven birds and new birds by aviary age:** The plan says "a year-old aviary has had time to reach 5-6; 7 is a long relationship," and "not visit-count, not paid." In rollout it adds that recognizability is "not a growth lever."

- **Server-authoritative simulation:** The rationale is to keep personality, mood, perch, greeting selection, notebook facts, weather, and day/night canonical, and to make multi-device sync an architectural property rather than a merge protocol.

- **Client rendering, procedural WebAudio, presence sensing, and interaction events:** The plan gives the client ephemeral performance work while the server owns truth, so the browser can render aliveness without inventing mood, personality, or drift.

- **Field notebook:** The notebook exists as sparse, immutable naturalist prose from a closed fact set; it must not become a visit-frequency log, a trait display, or a per-session engagement surface.

- **Listen-in:** The plan frames listen-in as "leaning in, not like soloing a track"; other birds recede but never vanish, and listen-in produces targeted drift through events rather than local state.

- **Offers for seed, song fragment, and still pool:** The offer cooldown is "long enough to stop session-saturation of curiosity" and cooldown on attempt prevents "mashers" from farming drift. The plan also keeps offer objects in the scene, not as DOM overlays or toasts.

- **Settle:** Settle quiets lighting and call gain for the session and is treated as `presence_end` only; "settle: no delta" keeps it from being a trait-changing action.

- **Visit invitations:** Visits are off by default, opt-in, read-only, and revocable to preserve a single visit-invite affordance without profiles, follows, discovery, comments, co-presence, or social-network defaults.

- **Accessibility surfaces in v1:** The plan says accessibility is "the same product in another register," which is why screen-reader narration, reduced motion, captions, keyboard navigation, and AA chrome ship with v1.

- **Account export:** Export is the one place personality numbers leave the server "because the user asked for their data"; the live product still never renders those numbers.

- **Session revoke, email change, soft delete, recover, and hard delete:** NOT RECOVERABLE FROM PLAN

- **No toast, banner, modal, or gone-N-days welcome surface:** The plan treats these as testable invariants because conventional product ceremony would break aliveness and turn return into scolding.

- **No streak, visit-count, calendar, XP, badge, or birds-adopted widget:** These are excluded to avoid "gamification of any flavor" and streak-adjacent telemetry.

- **First painted aviary frame as mid-motion or quiet field:** The plan says a spinner, static frame, or fade-from-static is wrong because the first impression must be the aviary already alive.

- **Procedural calls with no recorded fallback:** The plan says calls are procedural and "there is no recorded-call asset path" because recorded loops would be canned and fail the variation and chorus goals.

- **Telemetry separated from simulation rows:** The plan forbids telemetry pipelines from reading the simulation database so operations cannot become relationship analytics.

### Architecture

- **No client-to-client channel and no WebSockets in v1:** The plan says 20-30s snapshot pulls fit the slow simulation cadence, keep the battery story simple, and avoid a second consistency path.

- **Server/client split:** The plan assigns canonical persisted state to the server and ephemeral interpolation, audio, presence monitoring, and chrome to the client so the client never owns personality, mood, perch, greeting selection, notebook prose, weather, adoption eligibility, or authorization.

- **Renderer consumes `AviarySnapshot` only:** This stops the renderer from calling simulation, moving birds because the user is watching, inventing greetings, or advancing mood/personality.

- **Local cache of last snapshot for quiet-field continuity:** The cache is only a hint; the server snapshot always wins, preserving continuity without turning local state into authority.

- **Shared schema package:** The plan makes `packages/schema` the contract so API, tick worker, and client share event, snapshot, account, and visit DTOs; CI fails if the client snapshot gains personality-trait field names.

- **Derived `renderProfile` instead of named trait fields in snapshots:** The plan says this "stops the client from becoming a stats console."

- **Two-plane privacy architecture:** Simulation and Operations have separate readers so metrics can see request counts, latency, errors, fps, and anonymous histograms without bird vectors, notebook, or presence intervals.

- **Synthetic UUID identifiers and encrypted email stored once:** The plan keeps email on the account row and transactional mail provider only, avoiding logs, Kafka keys, and error messages.

- **Service modules:** NOT RECOVERABLE FROM PLAN

- **`packages/prose` pure function library:** The plan uses facts-in, lowercase-present-tense strings-out across notebook, narration, and captions, with no I/O, so prose surfaces share one voice without sending events to another system.

### Data model and defensible calls

- **Trait range as `numeric(6,5)` in `[0, 1]`:** The plan says this is "small, comparable, easy to clamp," with five decimals enough for week-1 instrument sensitivity.

- **Starter seeds with species prior plus noise:** The plan says this leaves "room to drift up" while making starters feel distinct without being maxed.

- **Mood enum of `wary`, `content`, `curious`, `drowsy`, and `alert`:** The plan says it matches the working set and should not be expanded in v1.

- **60s logical tick cadence:** The plan ties this to the "~once per minute" cadence and even wall-clock catch-up math.

- **Presence activity window and ping behavior:** The plan says to lean long because "watching without moving is the product"; pings every 15s and a 45s missed-ping close survive brief focus blips without counting a forgotten tab.

- **Offer cooldown per bird and offer kind:** The plan says four minutes is long enough to stop session-saturation of curiosity and short enough to try a second gesture.

- **Notebook sparsity gate:** The plan maps it to "every few days," with a valve for rain, first-greeter-this-week, and new arrival events.

- **Species pool with one nocturnal bird:** The plan says this creates a coherent place-set and ensures night is not a dead state.

- **Deterministic composition engine with no third-party LLM:** The plan says per-bird events must not leave the system.

- **Lazy simulation catch-up plus live sweeper:** The plan says this is observationally identical to "always ticking" while remaining scalable.

- **TypeScript, Vite, Preact chrome, and custom Canvas 2D:** The plan cites bundle budget, 60fps idle, and avoiding "game-engine tax."

- **TypeScript/Node/Hono/Postgres/Redis/email-provider backend:** The plan calls this "shared schema package" and "boring ops."

- **IANA timezone on account:** The plan's rationale is that "Local morning = aviary morning."

- **Account model with no display name, public handle, or avatar:** This follows the plan's rejection of profiles, follows, discovery, public handles, and social-network surfaces.

- **Stable bird identity row:** The identity rule says renames, species-pool edits, and migrations update columns and never insert a replacement row, so stable seeds and identity survive motif-library changes.

- **Append-only event log with idempotency:** The plan uses receipt order as authority, rejects personality values, and keeps clients to inserts so old snapshots cannot clobber canonical bird state.

- **Collapsed presence intervals:** The plan calls this so the tick does not re-parse ping storms.

- **Reusable visitor session after a one-time email link:** The plan says this matches "revoke outstanding or active" without forcing the friend to click email every visit.

- **Magic-link rate limits:** NOT RECOVERABLE FROM PLAN

- **Export JSON including personality values:** The plan says export is the one place personality numbers leave the server because the user asked, while in-app UI still never renders them.

- **Scores, streaks, hunger, health, death clocks, friend graphs, per-leaf simulation, and client-authored timelines deliberately not modeled:** The plan excludes these to protect against gamification, Tamagotchi mechanics, social network surfaces, and client-authoritative animation state.

### API surface

- **Account creation side effects with no notebook entry and no welcome event:** The plan keeps first account creation transactional but avoids a welcome surface so the birds are not introduced through product ceremony.

- **Owner event submission validation:** Unknown event types, personality fields, and visitor sessions are rejected to preserve tick authority and visitor isolation.

- **Offer endpoint resolves on the server:** The server catches up, enforces cooldown, chooses the receiver, appends a server-authored `offer_resolved`, and leaves drift to the tick so the client cannot farm or author drift.

- **Visitor snapshot scope:** Visitor snapshots omit notebook, pending-arrival controls, and settled-as-host because visitor clients are render-only.

- **Visitor listen-in disallowed:** The plan says otherwise a visitor would have a private interactive surface that the PRD forbids, and the team would be tempted to log it.

- **Snapshot pull triggers:** The plan pulls on session start, visibility, sleep/resume, keepalive, settle undo, offer return, and accept-arrival so state is refreshed at meaningful lifecycle points; it explicitly says not to pull on every pointermove.

- **Matter-of-fact auth and load failure copy:** The plan keeps these system routes plain and forbids naturalist error language like "the aviary could not find its way back to you."

### Simulation engine

- **Pure tick contract with replay:** The tick is "pure enough to replay" with same inputs producing same outputs, enabling golden replays, catch-up, and consistent API/worker behavior.

- **Fast-forward empty stretches:** The plan collapses no-event clear-weather spans and applies at least hourly circadian plus EMA decay to make 24h and 36h catch-up fast without losing continuity.

- **Live sweeper for open presence intervals:** Connected tabs see mood and weather change without waiting for their next action, while idle accounts wait for next snapshot or session-start.

- **Positive-only trait drift:** Presence, listen-in, and offers add small deltas; absence adds zero. This keeps "no negative terms" and no punishment for neglect.

- **Daily per-trait drift cap:** The plan says a four-hour sitting cannot burn a week of drift, preserving "no single session shifts a trait visibly."

- **Ambient quietness as EMA, not a trait:** The plan uses EMA so a two-week absence makes greetings and unobserved calls quieter while birds still call, live, and keep their color.

- **Persisted mood:** The plan says tab open does not reset mood and never sets all birds to content on session start, so a wary dusk bird is softer by morning rather than snap-reset.

- **Perch assignment by mood, boldness, and warmth:** The plan makes perch movement a simulation decision rather than user placement, so front/middle/back position becomes part of gradual personality expression.

- **Server call scheduler and client synth:** The server schedules signature, rate, and hints while the client synthesizes PCM, preserving identity and aliveness without shipping samples.

- **Return-greeting on session-start only:** The plan keeps greetings tied to real absence and prevents repeated keepalive greetings; absence class is used for form selection, not copy about days since visit.

- **Primary greeter and staggered answer:** The plan chooses a greeter from boldness, warmth, mood, and EMA, and says a second bird may answer but "Never unison," preserving noticed-by-bird aliveness rather than a banner or chorus.

- **Bird-to-bird call-answer, wary contagion, chorus, and adjacent perching:** The plan says these are tick rules, not client easter eggs, so relationships among birds are canonical.

- **Weather:** The plan uses rain and wind a few times a week so a user returning mid-rain sees rain already happening; weather affects calls, moods, and ornaments without adding thunder or snow.

- **Circadian lighting:** Account timezone and local phases make local morning be aviary morning; night remains alive through a nocturnal species.

- **Settle lighting override:** Settle moves lighting toward evening and quiets audio for the current session, while tab close clears it and the engine treats it as presence ending.

- **Notebook composer:** Closed fact extractors, sparsity gates, lowercase present-tense templates, and no "you," numbers, trait names, or visit frequency keep the notebook naturalist and non-gamified.

- **Arrival and age gates:** The plan makes new birds real through pending arrival after age gates; the bird is already real, the name field is not a modal or toast, and species are preferred unused with no rarity.

### Sync model

- **Single writer:** The tick worker or catch-up inside the API process writes personality, mood, perch, weather, and notebook under row locks so clients only append events.

- **No last-write-wins for bird state:** The plan makes `PUT /birds/:id { boldness }` unreachable; two devices emit events applied in sequence, so an old phone snapshot cannot clobber a laptop morning.

- **Name and settings last-write-wins only where acceptable:** Rename is acceptable as last-write-wins because it is a user-facing string, not drift; settings are last-write-wins per key.

- **Multi-device felt consistency:** Devices get the same moods, perch zones, and notebook while leaf positions and exact phase can differ because "the place is the same; the camera isn't locked."

- **Greeting suppression across near-simultaneous devices:** If another device session-starts within 10 minutes, the plan returns no greeting because "the aviary already noticed someone."

- **Counting simultaneous open tabs as presence:** The plan calls this "honest attention," while the daily cap keeps drift bounded.

- **Hidden tab and long frame gap handling:** The plan pulls a snapshot on resume and hard-syncs pose rather than interpolating across a gap, because that would smear a bird across the scene.

- **No merge UI:** The plan says true state conflicts should not occur and there is no "pick a version of Pip."

### Frontend rendering pipeline

- **Quiet-field boot sequence:** The plan calls this the product's first impression and forbids spinner or logo splash, keeping the first visible state within the aviary world.

- **Warm snapshot inline when possible:** The edge may inline a snapshot for warm sessions so first bird draw can happen quickly; otherwise the quiet field holds.

- **Critical JS chunk target under 80KB gz:** The plan keeps scene, snapshot apply, and sprites on the first-bird path while deferring audio and chrome.

- **First bird drawn mid-cycle:** The plan explicitly avoids pose zero so the bird appears already in motion.

- **Chrome fade after scene:** Preact chrome hydrates after the first scene path, starts hidden, then fades in so UI does not lead the experience.

- **Starter soft fly-in once in the aviary's life:** The plan allows a first-ever fly-in, then says birds are always already there after that.

- **Single horizontal world with no camera pan, zoom, or scroll:** The plan keeps the aviary as one place rather than a navigable game map.

- **Responsive fit that never crops a bird:** The plan compresses horizontally on narrow phones so every bird remains in frame.

- **Idle micro-motion families:** The plan says birds are never paused-still and that personality/mood choose motion family, not a canned full-body clip library.

- **Motion phase resumes from snapshot and elapsed time:** The plan avoids returning from hidden state at phase zero, preserving continuity.

- **Perch transitions and offer objects as scene motion:** The plan keeps transitions, seeds, pools, and songs in the aviary scene instead of UI overlays.

- **Top bar with icons only and fading opacity:** The plan keeps "zero chrome inside the scene," no names over birds, no mood icons, and no hover tooltips in the aviary.

- **Reduced-motion as a second rig:** The plan says it is not `animation: none`, keeps calls and captions unchanged, and ships in v1 alongside full motion.

- **Listen-in visual without spotlight/nameplate/dimming to zero:** The plan says over-marking the selected bird would make listen-in become "select."

- **Calm naturalist palette:** The plan uses tokenized colors, WCAG AA chrome and captions, and forbids saturated accent reds and electric blues.

- **Frame loop budget and hidden cancellation:** The plan targets 60fps idle on a five-year-old laptop, preallocates ornament pools, and cancels rAF when hidden.

### Audio pipeline

- **Procedural audio:** The plan gives three operational reasons: bundle size, chorus behavior, and recognizability through identity seed rather than sample.

- **No recorded-call assets:** CI fails if recorded calls are added because fallback recordings would become more canned under stress.

- **Motif library per species:** Motifs as data plus timbre patch allow species character while keeping synthesis procedural.

- **`signatureSeed` identity:** A bird's seed fixes f0 range, motif weights, vibrato, and formant offset for life; vocal frequency retimes the scheduler rather than retuning identity.

- **Pooled synth graph:** The plan pools voices and uses AudioParam ramps instead of creating and destroying nodes per call.

- **Chorus mixing without sidechain ducking:** Overlapping calls remain separate voices; no global quantized grid or ducking except listen-in keeps chorus natural.

- **Listen-in gain ramps:** The 1400ms ramp and nonzero other-bird gains make the action feel like leaning in, not isolating a track.

- **Captions from the motif instance actually scheduled:** The classifier uses note count, interval direction, and zone words so two plays of a motif can caption differently if they sound different.

- **Captions default off but forced when WebAudio is unavailable:** The plan stays silent and turns captions on rather than fetching recordings.

- **Audio unlock and background behavior:** The plan unlocks on first pointer/key, suspends when hidden, and avoids auto-enabling captions after every brief suspend so tab switches do not suddenly cover the scene in words.

- **Uncanniness controls:** Minimum variation, no identical motif plus tempo twice in a row, humanized chorus, and a five-minute listening panel prevent identifiable loops and ringtone-like chorus.

### Accessibility surfaces

- **Screen-reader narration as composed prose:** The plan rejects "DOM soup of seven birds" and uses the same prose voice as notebook so narration is observational, sparse, and charming without mood stats.

- **Narration cadence and queue:** One idle paragraph every 30-60s, max two pending, and dropped idle updates prevent live-region spam.

- **Birds as a flat tab-order list:** The plan gives each bird an accessible name and static species clause, while live changes go through narration to avoid spamming screen readers on pose frames.

- **Keyboard navigation:** Tab order, arrow navigation, Enter/Space listen-in, Escape, and keyboarded sheets make listen-in, offer, settle, and naming usable without pointer input.

- **Focus ring:** The plan requires a contrast-passing soft outline on dawn and night and keeps it visible in reduced motion.

- **Caption/live-region de-duplication:** The plan avoids duplicating a caption into the live region if a call was already described in the last narration sentence.

- **Settings voice split:** Accessibility and account settings stay matter-of-fact, while aviary, notebook, offers, captions, and narration stay naturalist.

- **Visitor accessibility:** Visitors get the same narration, reduced motion, and captions even though they do not get interactive listen-in.

### Performance budgets and observability

- **Initial JS, time-to-first-bird, fps, memory, tick, snapshot, and email budgets:** The plan defines these as CI and synthetic gates so aliveness is not lost to a slow first frame or a degrading 30-minute session.

- **Edge shell, critical renderer split, inline warm snapshot, sprite strategy, and no webfont on scene:** The plan groups these under hitting TTFB and first bird.

- **No analytics SDK:** The plan lists this as part of hitting the first-bird path and preserving privacy boundaries.

- **Synthetic fleet metrics:** Hourly synthetic checks measure load, first-bird mark, fps, audio errors, and snapshot latency so product performance is observed without bird-level relationship data.

- **Aggregate-only RUM:** The plan permits navigation timing, first-bird, long tasks, fps histogram, audio-context errors, API latency, and session duration histogram without account id.

- **Deliberately unmeasured product behavior:** Per-bird interaction counts, per-account presence, drift distributions, most-listened species, visit funnels, rage-clicks, heatmaps, and session replay are excluded because replay would capture the relationship.

- **Client marks without bird ids:** The plan's marks can go to RUM because they avoid bird identifiers.

- **Last-two-major browser support only:** The plan avoids "polyfill soup" that would blow the bundle.

### Rollout

- **Milestone order:** The plan says teams can parallelize after the contract, so schema, prose stubs, migrations, auth, empty quiet field, and frozen snapshot DTO come first.

- **Reduced-motion in the visible-aviary milestone:** The plan says reduced motion ships in the same milestone, not later.

- **Staging clocks accelerated for birds 3-7:** The plan allows this so QA can see the ramp while production age gates stay unchanged.

- **No v1 cap increase or A/B higher cap:** The plan says recognizability is not a growth lever.

- **Quiet beta and GA with the same binary:** The plan rejects public discovery, press promises about notifications, and an engagement experiment layer.

- **Instrumentation from day one:** Synthetic first-bird, fps, tick latency, auth rates, event counts by type, privacy CI, and product CI make performance and refusal testable from the start.

- **Drift calibration harness:** The plan replays regular, heavy, and absent scenarios and says if staff see day-2 change alpha is too high, while no day-21 change means alpha is too low.

- **No in-product vector debug panel:** The plan keeps vector inspection staging-only, behind VPN, matter-of-fact, and never shipped to production bundles.

### Risks and implementation notes

- **Versioned drift constants, daily cap, and presence conjunction tests:** The plan uses these to mitigate drift that is too fast, too slow, negative, or driven by loose presence.

- **Redis invalidation, visitor route constraints, and catch-up tests:** These mitigate stale snapshots after offers, visitor leakage into drift, and mood snap-to-default.

- **Audio listening panel and motif variation rules:** These mitigate loops, ringtone-like chorus, seven-bird mush, and nightjar sounding like a soundboard joke.

- **Reduced-motion rig, narration cadence tests, caption review, contrast CI, and keyboard e2e:** These mitigate live-region spam, fake reduced motion, bad caption copy, invisible focus, and late a11y.

- **Quiet-field boot, server `phase01`, greeting stagger, welcome-string lint, and first-paint snapshots:** These mitigate spinner, logo, fade-from-grey, pose zero, unison greeting, and "Welcome back."

- **Email HMAC, Sentry scrubbers, no replay vendors, and warehouse deny-list:** These mitigate email partition keys, per-bird events in error extras, logged export links, and replay leakage.

- **Closed notebook facts, sparsity gate, copy review, and forbidden-phrase tests:** These mitigate timestamp-like entries, LLM drift, one entry per session, and entries about user habits.

- **No toast component:** The plan says engineering should make predictable scope creep expensive to sneak in.

- **No game loop library:** The plan says custom canvas keeps the budget and avoids stock loaders.

- **Preact only for chrome:** The plan says the scene stays framework-free so first-bird JS stays small.

- **Bird names limited to 1-24 graphemes:** NOT RECOVERABLE FROM PLAN

- **Song-fragment library:** The plan keeps song fragments distinct from bird grammars and avoids v1 pitch-matching beyond join, quiet, or against.

- **Still pool:** The plan makes it a temporary translucent ellipse, "not a permanent furniture unlock."

- **Seed object:** The plan removes the seed after reaction or 20s so it remains an offer object, not a durable scene change.

- **Single support contact, no chatbot:** The plan ties this to matter-of-fact error copy.

- **Privacy policy in account settings:** The plan says it should name operational telemetry categories and explicitly exclude per-bird interaction state.

- **English-only v1 prose:** The plan says the prose engine is English and warns against generic i18n interpolation that title-cases naturalist strings.
