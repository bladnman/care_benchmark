## System-level intent

1. **Principles behave like invariants, not aspirations.** The plan says the PRD has "five principles that behave like invariants, not aspirations," and it converts each refusal into schema grants, lint rules, contract tests, CI gates, and infra-policy checks. This shows up in the invariant register, the "refusals suite," the OpenAPI deny-list, database triggers, and the note that "we agreed not to" does not survive eighteen months of contributors.

2. **Aliveness must be continuous and non-canned.** The plan repeatedly protects "first frame already in motion," "no load state," "no canned cue," "no looped audio," "no cycle animation," and failure modes where the aviary "continues, quieter" rather than freezing. It appears in the inlined bootstrap snapshot, quiet field, vector-first birds, forward-scheduled calls, procedural motion, procedural audio, and degradation rules.

3. **Notice, never announce.** The plan treats product affect as something the aviary does, not something UI copy declares: "no toast/banner/notification/welcome copy," no "welcome back," no absence-length text, no unison greeting, and no assertive live region. Return-greeting, top-bar audio affordance, notebook prose, and social notifications are all constrained by this.

4. **No gamification, ever.** The plan prevents counting, ranking, numbering, streaks, "days visited," "birds adopted: N," stats endpoints, and the underlying aggregates that would make them possible. It appears in the data model, endpoint deny-list, telemetry omissions, age-only offer schedule, and CI lexicon checks.

5. **Personality is hidden, slow, monotonic, and server-authoritative.** The plan makes "only the simulation writes personality," says "neglect never reduces a trait," prevents "last-write-wins," and keeps personality numbers off product surfaces. The why appears in the drift function, DB grants, monotonicity trigger, snapshot shape, export exception, calibration conservatism, and sync model.

6. **The product is about noticing consequences, not reading state.** The plan says the user reads mood from motion and personality by comparing to memory; a direct mood label or trait number "collapses two layers into one and removes the noticing." This appears in the conceptual three clocks, absent `mood` and trait fields, naturalist narration, captions, and render consequences like posture, fluffed state, calls, perches, and plumage tier.

7. **Bird identity is stable forever.** The plan treats immutable `bird_id`, fixed `timbre_seed`, species retention, and no row recreation as load-bearing because changing identity is "the worst possible failure of this product." It appears in the `birds` schema, migration policy, recognizability design, export, soft delete recovery, and audio identity/variant split.

8. **Presence must be credited honestly.** The plan defines presence as "visible ∧ focused ∧ recent input" while making the server distrust client assertions. It appears in ping cadence, stale rejection, server-side gap ceilings, multi-device interval union, daily caps, and fixture tests.

9. **Privacy boundary is architectural, not policy.** The plan says per-bird interaction data never leaves the simulation boundary and telemetry is aggregate only. It enforces this with separate stores, subnet isolation, no account dimension, label deny-lists, no CDC connector, no route from simulation subnet to analytics, and the accepted cost of running without behavioral analytics.

10. **Email is PII; everything else uses synthetic identity.** The plan treats email as the "single most important boring detail," allowed only in encrypted account and invite fields. Blind indexes support lookup, and CI rejects email in metric labels, logs, cache keys, or foreign-key columns outside the two tables.

11. **Accessibility is a designed surface, not a fallback.** The plan says accessibility "ships with v1, not after" and that AA with a flat state-list feel is a fail. Reduced-motion, screen-reader narration, captions, keyboard focus, contrast tests, prose, and definition-of-done are all product surfaces with their own acceptance criteria.

12. **Voice split: naturalist product surface, matter-of-fact system surface.** The plan requires naturalist prose for the aviary, notebook, narration, and captions, and matter-of-fact prose for auth, account, sync, accessibility, and browser errors. This appears in `@aviary/prose`, banned inline strings, error codes mapped client-side, mailer constraints, and prose property tests.

13. **Continuous state belongs on the client; discrete canonical state belongs on the server.** The plan draws a "discrete versus continuous state" line: server owns personality, mood, perches, call intents, weather, settled state, offers, notebook, and greetings; client owns sub-pixel motion, easing, synthesis, ornaments, parallax, captions, focus rings, and local listen-in target. The rationale is first paint, graceful continuation, and retargeting rather than teleporting.

14. **Constraints should be made machine-checkable.** The plan repeatedly converts product choices into "schema column, API field, client string," lint rules, type partitions, grants, triggers, contract tests, fixture tests, and Terraform tests. This appears in non-goals as engineering constraints, refusals suite, definition-of-done, and risks.

15. **Launch conservatively where mistakes are irreversible.** The plan states "too-fast drift is unrecoverable and too-slow drift is a config change," so `α` ships at 0.6× fitted and is raised only after dogfood signal. Similar conservatism appears in the birds-per-aviary cap, five-species contingency, no recorded-audio fallback, and privacy boundary.

16. **The aviary should feel like a place, not an app or dashboard.** The plan prefers quiet field over loader, subtle parallax, day/night, weather, ambient bed, no chrome in the scene, sparse notebook, top bar fade, and "no notification surface." Failure states and accessibility surfaces are shaped to keep the affective core.

## Per-feature whys

### 0. How to read this plan

- **Invariant register and inline INV tags:** The rationale is traceability into tests: the plan says most engineering decisions exist because a principle would otherwise be violated, and every invariant has at least one automated check.

### 1. Scope

- **Single-user accounts, one canonical aviary per account:** NOT RECOVERABLE FROM PLAN

- **Email + magic-link sign-in, per-device revocable sessions, verified email change:** Magic links avoid password or SSO auth, and per-device sessions support revocation. The plan does not further explain choosing magic link over other browser auth.

- **Account export:** The plan frames export as a data-portability artifact and account mitigation: a user with an export "at least holds a copy of their aviary."

- **Soft delete with 30-day recovery then hard delete:** Soft delete lets sign-in recovery work and resumes birds unchanged, drift intact. Hard delete completes privacy obligations after the recovery window.

- **Five-trait hidden personality vector:** Personality changes the distribution of mood and behavior while remaining something the user notices only by comparing to memory; direct numbers would remove the noticing.

- **Monotonic slow drift:** The rationale is "no Tamagotchi": neglect means no decrease, and the relationship deepens slowly without reducing a bird.

- **Six-state mood machine:** Mood gives the hours-to-a-day layer that changes which behaviors are likely right now. The added `settled` state is justified because night and post-settle need a distinct state.

- **Six-species pool:** The plan uses species for silhouettes, palettes, motif libraries, ceilings, call rates, night activity, and perch bias. The specific reason for exactly six is partly recoverable as the PRD's "about six"; a species may be cut if recognizability fails.

- **Two starter birds at adoption:** NOT RECOVERABLE FROM PLAN

- **Age-gated offers up to hard cap of seven:** Age-only gating avoids visits, interactions, or payment as a gamified input. Seven is tied to recognizability and voice-pool/perch limits; if recognition fails, cap can drop.

- **Bird-to-bird interaction: call-and-answer, mood contagion, emergent chorus:** The rationale is to make the aviary "a small social system" and make chorus happen from compatible moods and schedules rather than from a canned event.

- **Server-side simulation tick whether or not a client is connected:** The plan wants mood, perches, weather, and call schedule to continue without the viewer, so the returning user sees an aviary that has caught up to now.

- **Single horizontal non-panning scene, three perch zones, responsive 320-2560 CSS px:** No panning, scrolling, or zoom keeps the scene as one place and prevents camera features from creeping in. Wider viewports spread perches apart rather than scaling birds up.

- **First frame already in motion:** The rationale is aliveness: any spinner, entry animation, fade-from-static, or phase-0 motion is an "app just woke up" tell.

- **Day/night cycle anchored to local time:** Lighting must follow the user's local time and "must never be stale"; the client clock is authoritative for rendering.

- **Rare ambient weather:** Weather contributes to mood, lighting, audio bed, and the aviary as a place. The plan does not explain why rarity specifically.

- **Ambient ornament drift and subtle parallax:** The rationale is depth and place without synchronized simulated state; ornaments are decoration and "not simulated."

- **Thin fading top bar:** The plan keeps account/settings, accessibility settings, notebook, and offer available while avoiding chrome inside the scene. Fade is a cursor-stillness affordance and never hides controls during keyboard navigation.

- **Return-greeting:** The rationale is that the aviary notices without announcing. Greeting uses boldness, mood, warmth, absence band, variation, and staggering so it is not a canned arrival animation.

- **Listen-in:** The rationale is "leaning in toward one bird" while other birds remain present; it rebalances and lowpasses rather than muting, avoiding a solo-track mixing desk.

- **Offer:** The rationale is partly recoverable as a small interaction that can influence boldness, warmth, and curiosity, with cooldowns and caps preventing offer-spam from becoming farmable drift.

- **Settle with 5s undo:** Settle gives a lighting state and mood-quieting nudge while contributing nothing to drift. The plan does not explain why the undo window is five seconds.

- **Field notebook:** The rationale is sparse, auto-generated naturalist prose that records noteworthiness without becoming a log, unread count, badge, or user-behavior surface.

- **Procedural client-side WebAudio call synthesis:** The rationale is no recorded playback, no looped audio, infinite variation, and per-bird recognizability while keeping variation-rich synthesis on the client.

- **Real-time chorus mixing:** The rationale is to make overlapping calls sound like they are in one place and to let choruses emerge from scheduled calls rather than a trigger.

- **Silence-plus-captions fallback:** The rationale is graceful accessibility and browser support without adding a recorded-audio path.

- **Screen-reader naturalist narration:** The rationale is to deliver the product's affective core by describing observable details, not state lists, trait numbers, or mood labels.

- **Reduced-motion mode:** The rationale is a "distinct designed render mode" with charm, not the default renderer with animations disabled.

- **Opt-in call captions generated from actual synthesized call:** The rationale is that captions match what was heard because they come from the same call spec the synthesizer consumed.

- **Full keyboard navigation and visible focus treatment:** The rationale is keyboard reachability and focus visibility against all lighting states, with no single-letter shortcuts because they collide with screen-reader quick-nav keys.

- **Social off by default: per-invite email invitations, read-only ambient visitor view, immediate revocation, expiry, log, opt-in notification toggle:** The rationale is limited, revocable viewing without creating a social network, co-presence, chat, visitor identity in scene, or visitor-driven drift.

- **Aggregate-only telemetry and RUM:** The rationale is privacy boundary and anti-gamification: product decisions come without behavioral analytics or per-bird/per-account funnels.

- **Synthetic browser fleet:** The rationale is to verify first-bird timing, greeting, call synthesis, listen-in, offer, and settle continuously without measuring private per-account behavior.

- **Perf and a11y budgets as CI gates:** The rationale is that budgets and accessibility are completion criteria, not late sweeps.

### 1.2 Explicitly out of scope for v1

- **No native iOS/Android apps or native-driven protocol concession:** The API is designed for a browser client; native token flows and offline-first mutation semantics are deliberately not added "in case."

- **No payments or tiers:** NOT RECOVERABLE FROM PLAN

- **No shared, team, or household aviaries; no multiple aviaries per account:** The plan keeps one canonical aviary and avoids cross-account read paths. The plan does not further explain each excluded account model.

- **No customizable or purchasable scenes:** NOT RECOVERABLE FROM PLAN

- **No public discovery, profiles, follows, feeds, ratings, featuring, leaderboards, rankings, achievements, badges, levels, XP, scores, streaks, visit-frequency surfaces:** The rationale is no gamification and no social network; even underlying aggregates are excluded because they would enable those surfaces.

- **No push notifications or marketing email about the aviary:** The rationale is no notification surface; mailer is transactional only.

- **No hunger, health, happiness meters, death, or distress state:** The rationale is no Tamagotchi and neglect never reduces a trait.

- **No co-presence, chat, comments, avatars, or visitor identity rendered in the scene:** The rationale is social remains read-only ambient visiting, not a social network.

- **No recorded-audio playback paths:** The rationale is procedural mandate and no recorded fallback under schedule pressure.

- **No password or SSO auth:** NOT RECOVERABLE FROM PLAN

- **No support for browsers older than last two majors:** NOT RECOVERABLE FROM PLAN

### 2. Architecture

- **Five deployable units:** The split is driven by "only the simulation writes personality," "per-bird data never reaches analytics," and edge-rendered first paint.

- **Edge worker serving HTML, static assets, inlined bootstrap snapshot, immutable assets:** The rationale is to remove a round trip from first bird, serve quiet field immediately, and keep asset serving/cache close to the user.

- **`aviary-api` stateless service:** It handles auth, snapshot read, event ingest, notebook read, settings, visits, and export while lacking write authority over personality and mood.

- **Postgres as simulation primary:** The rationale is row-level grants for INV-5, partitioned append-only event log, and `SKIP LOCKED` tick leasing without a broker.

- **Redis snapshot cache and rate limits:** NOT RECOVERABLE FROM PLAN

- **Separate `sim-worker`:** The rationale is that it holds the only database role with `UPDATE` on personality and mood; sharing that grant with request handlers would make INV-5 a convention.

- **`notebook-writer` in-process module of `sim-worker`:** It needs the same transactional view of state and events, runs slower, and a separate service would reread rows for no isolation benefit.

- **Mailer with four allow-listed templates:** The rationale is no notification surface; adding a fifth requires touching a file whose header says why.

- **Telemetry/RUM in separate store and VPC subnet:** The rationale is architectural enforcement that per-bird data never reaches analytics.

- **Server/client split by discrete versus continuous state:** The rationale is canonical persisted state on the server, derived disposable experience on the client, yielding first paint, continuation, and retargeting.

- **Ornaments seeded from `(aviary_id, wall-clock minute)` and not simulated:** The rationale is that leaves and feathers are decoration; similar across devices is enough, and frame-identical sync would put real-time protocol in the critical path for decoration.

- **Calls scheduled server-side and voiced client-side:** The rationale is first-frame audibility without round trip, server-side chorus grouping with hidden traits, and keeping expensive variation-rich synthesis on the client.

- **Snapshot as three-minute forward description:** The rationale is zero-fetch first frame, offline/stale "continuing" instead of freezing, and reconciliation by retarget rather than teleport.

- **TypeScript strict client and server:** The rationale is shared types and shared prose/engine packages.

- **Canvas 2D renderer:** The rationale is that 7 birds and subtle parallax do not need WebGL; WebGL adds bytes, context-loss handling, and old-GPU risk.

- **No UI framework on critical path:** The rationale is that every framework KB competes with the 500ms budget.

- **WebAudio graph with pre-allocated pool:** The rationale is procedural calls and no memory growth.

- **No queue at v1:** The event log is the queue; Kafka would add a second ordering authority and a tempting partition key derived from email.

- **Two email providers behind one interface:** A broken magic-link path is total account loss.

- **No WebSocket, SSE, or realtime channel:** The 60s tick and client interpolation make polling enough; omitting realtime removes connection-state bugs, and visitor lag up to 45s is invisible at the cadence.

### 3. Domain model, conceptually

- **Three nested clocks: weeks/personality, hours-to-a-day/mood, seconds/behavior:** The rationale is layered perception: personality biases mood, mood biases behavior, behavior is visible, mood is read from motion, personality is noticed only by memory.

### 4. Vocabulary rules

- **Code uses PRD nouns:** The rationale is that identifiers leak into UI copy through templating; wrong nouns like `sfx` can eventually produce wrong captions.

- **No user-facing string inline:** The rationale is voice split survival; contributors cannot add "Welcome back!" outside writer-owned prose files and tests.

### 5. Data model

- **UUID v7 IDs:** Time-ordered IDs index well and give a natural event-log sequence.

- **Encrypted email plus blind index in `accounts`:** The rationale is sign-in lookup without storing bare email; email handling is unretrofittable and must be a schema decision.

- **`tz_iana` account hint:** The server needs time-of-day mood affinities while no client is connected; client remains authoritative for lighting.

- **`settings` in account:** NOT RECOVERABLE FROM PLAN

- **`sessions.token_hash`:** Raw token is never stored. The plan does not further explain the session token design.

- **Magic links with 15-minute expiry and single-use consumption:** The rationale includes no replay and matter-of-fact error on replay/expiry. The plan does not explain why the expiry is exactly 15 minutes.

- **`aviaries.created_at` as aviary age:** New birds become available by age and nothing else, avoiding visits or interactions as gamified inputs.

- **No visit/session/presence counters on `aviaries`:** The rationale is that such counters would become "days visited" or similar surfaces.

- **Immutable `birds` identity fields and fixed `timbre_seed`:** The rationale is stable identity and recognizing a bird by ear forever.

- **Species migration policy keeping retired species assets forever:** The rationale is never reassigning or recreating birds, avoiding the "worst possible failure."

- **Species-specific ceilings frozen at adoption:** The rationale is to prevent monotonic drift from homogenizing long-tenured birds into identical maximally-bold birds.

- **`filter_state` storing low-pass accumulators:** The rationale is drift is never recomputed from event history, so retention trimming cannot affect personality.

- **Monotonic drift trigger raises rather than clamps:** The rationale is to page on a real bug and leave the vector intact; clamping would silently absorb the violation.

- **`activity_phase` in `bird_state`:** The rationale is first frame mid-action rather than every bird starting a micro-motion at phase 0.

- **Append-only `interaction_events` with server-assigned `ingest_seq`:** The rationale is canonical order, idempotent retry, and protection from skewed or backdated client timestamps.

- **45-day event log retention:** The rationale is personality is stored, not derived, and interaction history retention is minimized while leaving replay margin.

- **`presence_credit` unreachable from API:** Presence is the dominant drift input and exactly the future "days visited" number, so surfacing it requires a migration, grant change, and conversation.

- **Notebook entries immutable and read-only:** The rationale is a final naturalist record, not a user-edited or badge-tracked log.

- **Visit sessions never joined to interaction events and never read by tick:** The rationale is visitor attention does not drift host birds.

- **Species pool as code config, not rows:** Motifs are code-coupled to synthesis primitives, and DB-editable species would invite production edits that change an existing bird's voice without review.

- **Soft delete stopping ticks and treating invites as revoked:** The rationale is recovery without changing birds or drift, while making deleted accounts unavailable to visitors.

- **Hard delete order and backup retention:** The rationale is complete deletion, including aging out restorable snapshots by day 65.

- **Export signed URL emailed with 7-day expiry:** The rationale is data portability and access mitigation. The plan does not explain why the expiry is exactly seven days.

- **Export contains vectors but no in-app viewer/parser/import:** The rationale is that INV-6 governs product surfaces, while export is a data-portability artifact.

### 6. API surface

- **REST/JSON with HttpOnly Secure SameSite=Lax cookies:** Browser client focus and cookie sessions. The plan does not further explain the transport and cookie attributes.

- **Server error bodies carry codes, not English:** The rationale is the client maps to `prose/system/*`, preserving voice split.

- **`POST /auth/magic-link` always 202 with identical latency/body:** The rationale is no account enumeration.

- **Magic-link rate limits:** The rationale is abuse control. The plan does not explain why the exact limits are 5/hour and 20/hour.

- **`GET /auth/consume` 303 to `/` and matter-of-fact error on failure:** The rationale is to avoid redirect loops and keep system voice.

- **Session list and revoke endpoints:** The rationale is per-device revocable sessions.

- **Verified email change where old address works until new confirms:** The rationale is account continuity during email change.

- **Edge-served `GET /` with inline sky CSS and bootstrap snapshot:** The rationale is first bird under 500ms and no white flash; a stale snapshot is fine because client extrapolates from server time.

- **Quiet field when snapshot fetch fails or times out at edge:** The rationale is never spinner, progress indicator, or skeleton; the field is indistinguishable from the aviary's sky.

- **Snapshot ETag and 304:** The rationale is cheap polling when unchanged.

- **Snapshot omits traits and mood:** The rationale is personality and mood must be read from consequences, not dev-tools or product fields.

- **`render.plumage_tier` crosses wire:** The rationale is minimum leak required for rendering plumage, quantized and unlabeled.

- **Events endpoint inserts log rows only:** The rationale is no event mutates personality or mood; the tick is the only writer.

- **Synchronous `settle`/`settle_undo` and offer cooldown writes:** The rationale is immediate visible response for presentation state without touching personality.

- **Reject presence pings older than 10 minutes:** The rationale is suspended-laptop replay must not credit hours of dishonest presence.

- **Payload assertions advisory and server credit based on arrival spacing:** The rationale is the server does not trust client visibility/focus claims.

- **Notebook endpoint with unbounded scrollback but no unread count/view tracking:** The rationale is sparse record without badges or gamified counters.

- **Bird rename endpoint:** The plan says name is user-assigned, renameable, and has no effect on anything else. The plan does not further explain rename support.

- **Settings endpoint:** NOT RECOVERABLE FROM PLAN

- **Visit invite creation with max 10 outstanding:** Visitor access is per-invite and revocable. The plan does not explain why the maximum is exactly 10.

- **Invite listing with visitor email and used status:** NOT RECOVERABLE FROM PLAN

- **Invite deletion with no confirmation surface; absence from log is confirmation:** The rationale is immediate revocation without adding announcement UI.

- **Visit log with approximate duration:** The rationale is on-demand visit log and approximate duration by design from last pull. More specific rationale is NOT RECOVERABLE FROM PLAN.

- **Visitor route and snapshot separate from host route:** The rationale is structural read-only guarantee, no event ingest access, no host greeting, no offer/listen-in/settle/notebook/settings modules, and no visitor-triggered drift.

- **410 Gone `visit_unavailable`:** The rationale is matter-of-fact unavailability when the invite is revoked, expired, or the account is deleted. The plan does not further explain the status-code choice.

- **Endpoints that will never exist:** The rationale is "the discipline is the feature"; no stats, streak, traits, notifications, plural aviaries, or leaderboard paths can accidentally ship.

### 7. Simulation engine design

- **Tick leasing with `SKIP LOCKED`:** The rationale is an aviary is never ticked concurrently, supporting INV-5 under horizontal scaling.

- **60s active cadence and 300s hibernation:** The rationale is active aliveness with cheaper unwatched continuation; drift during absence is zero, so hibernation advances only mood, perches, weather, and calls.

- **Synchronous catch-up on snapshot read:** The rationale is hibernation must be unobservable to a returning user.

- **Deterministic tick core with seeded PRNG and injected clock:** The rationale is replayability for tests, calibration, and incident forensics.

- **Tick idempotency with transactional watermark:** The rationale is crash-and-retry produces the same result and consumes events exactly once.

- **Presence pings every 15s while visible/focused/recent input:** The rationale is watching without moving is the product, so the input window is "a few still minutes" rather than immediate timeout.

- **Credit `min(gap, 20s)` and first ping 15s:** The rationale is one ping of jitter tolerance and no credit for missing-ping gaps.

- **Merge intervals across sessions:** The rationale is one aviary-second per wall-clock second; two devices must not drift birds twice as fast.

- **Daily presence cap at 4 hours:** The rationale is above that something is wrong or the signal no longer means what the drift assumes.

- **Settle and tab-close are absence of pings, not penalties:** The rationale is neither is penalized or required; settle is only lighting and mood-quieting.

- **Drift formula `x' = x + α · u · (ceiling − x)`:** The rationale is monotonic, saturating, slow relationship deepening that cannot decay on neglect.

- **Low-pass intensity and daily saturation cap:** The rationale is a user cannot click their way to a bolder bird or farm drift with long sessions/offers.

- **Presence/listen-in/offers signal ordering:** The rationale is the PRD's ordering: presence dominates, listen-in second, offers third, settle nothing.

- **Calibration profiles and harness:** The rationale is making the drift target executable, catching over-fast drift, farmability, double credit, and drift-on-neglect.

- **Ship `α` at 0.6× fitted:** The rationale is too-fast monotonic drift is unrecoverable while too-slow drift is a config change.

- **Human dogfood validation of drift:** The rationale is the harness validates math, but whether three weeks of drift is felt is a human question.

- **Mood scoring with time, recent events, weather, contagion, personality, hysteresis:** The rationale is stored, persistent mood that changes behaviors without snapping to a default.

- **Daily-ish reset via dawn temperature/affinity shift:** The rationale is morning re-roll without assignment to neutral and without session-start input.

- **Mood contagion:** The rationale is a small social system where nearby birds influence each other by warmth and boldness.

- **Perch stable matching with stay bonus:** The rationale is avoiding tick-to-tick swaps that read as twitchy and mechanical.

- **Flight arc emitted as `perch_from` + timing:** The rationale is client animation from server decisions without user control over perches.

- **No endpoint to move a bird:** The rationale is "perch is a signal, not a layout."

- **Call forward window to `now + 180s`:** The rationale is one lost tick never produces silence.

- **Gamma-distributed inter-call intervals:** The rationale is natural clustering rather than metronomic spacing.

- **Call motif and variation seed:** The rationale is repeated snapshot pulls voice the same call consistently while no call is identical to another.

- **Call-and-answer:** The rationale is acoustic answering, with pitch lean, rather than coincidental overlap.

- **Chorus group tagging from overlapping calls:** The rationale is emergent chorus, not a trigger/event/cooldown.

- **Return-greeting trigger on presence resumption:** The rationale is greeting based on absence and hidden traits, computed by tick rather than client.

- **Absence bands:** The rationale is absence length must be wired through as a first-class signal, but not exposed as text.

- **One primary greeter and optional staggered secondary:** The rationale is no guaranteed canned cue and no unison announcement.

- **Greeting form recency penalty and variation:** The rationale is procedural variety rather than rotating a few clips.

- **Tick cost model with batching and writing only changed rows:** The rationale is DB IO dominates; sticky mood/perch keep write volume low.

### 8. Audio pipeline

- **Motif DSL authored by sound designer:** The rationale is expressive procedural calls authored by a designer, not engineers tuning test tones.

- **Normalized motif pitch inside bird pitch band:** The rationale is same phrase in individual voices.

- **Sampling ranges and large seed space:** The rationale is a user will not hear the same call twice.

- **Pre-allocated voice pool of 8:** The rationale is no WebAudio node churn and no memory growth in 30-minute sessions.

- **Persistent bird buses:** The rationale is stable max-7 bird mixing. The plan does not further explain persistence beyond that mixing boundary.

- **Algorithmic FDN reverb, not convolution:** The rationale is no downloaded audio asset, 2MB budget, and making calls sound like one place.

- **Lookahead scheduling with smoothed server/context offset:** The rationale is audible hard resyncs are avoided while small drift is inaudible.

- **Procedural ambient bed:** The rationale is wider-world sense and keeping listen-in from becoming near-silence.

- **Anti-repetition ring buffer:** The rationale is cheap insurance against audible near-repeat.

- **Per-bird recognizability from fixed `timbre_seed`:** The rationale is "you know Pip by ear" and seven-bird cap viability.

- **Identity/expression split in synthesis parameters:** The rationale is a bird can be in a different mood while obviously remaining the same bird.

- **Listen-in gain, lowpass, ambient, and reverb ramps:** The rationale is perceived closeness and no hard cuts, mutes, or soloable tracks.

- **Disengage triggers for listen-in:** The rationale is interaction completeness around leaving, switching, clicking away, pressing `Escape`, or moving focus. The plan does not further explain the exact trigger set.

- **Clock alignment EMA and outlier rejection:** The rationale is schedule calls from server time into audio context without audible hard resync.

- **Autoplay resolution with silent aviary and passive icon:** The rationale is browser policy conflicts with first-frame audio, and a "tap to begin" gate would violate the entry-state conceit.

- **Listening study:** The rationale is the seven-bird cap is empirical, so recognizability is verified rather than assumed.

- **Ship cap at 5 if 7 fails; fix authoring if 5 fails:** The rationale is recognizability binds the count, and motif distinctness is a sound design problem.

- **MFCC spectral-distance proxy in CI:** The rationale is a cheap regression guard for motif edits.

- **WebAudio unavailable fallback with captions on by default:** The rationale is silent accessibility without recorded audio decoder, asset pipeline, or `<audio>` path.

### 9. Sync model

- **One canonical record, one writer, no client-side state to reconcile:** The rationale is multi-device sync by absence of divergence rather than protocol.

- **Pull on bootstrap, visibility, long frame gap, keepalive, forward-window exhaustion, visible-state events:** The rationale is hidden tabs and suspended machines need current snapshots, scheduled calls must not run out, and visible state changes need server response.

- **Backoff while aviary keeps rendering:** The rationale is transient failures should not interrupt a rendering aviary.

- **Listen-in not mirrored across devices:** The rationale is listen-in is a person's device-local act of attention; mirroring would yank another device's mix.

- **Settle propagates and resolves last-write-wins by `ingest_seq`:** The rationale is settle is presentation state with no history, unlike personality.

- **Personality last-write-wins unreachable:** The rationale is no endpoint accepts traits, API grants cannot write vectors, writes are current-row deltas in tick transactions, and server order is canonical.

- **Forward-window exhausted degradation with local calls and no perch changes:** The rationale is quieter and stiller, never frozen.

- **Retarget, never teleport on new snapshot:** The rationale is visual continuity after divergence.

- **Matter-of-fact session expired and server failure surfaces:** The rationale is system voice and no interruption while the aviary can survive.

### 10. Frontend rendering pipeline and accessibility

- **Normalized scene coordinates and no camera transform:** The rationale is responsive scene without panning, scrolling, zoom, or cropping.

- **No-crop test matrix:** The rationale is every bird remains fully inside canvas across widths, counts, and perch permutations.

- **Four canvas layers with cached redraw cadences:** The rationale is performance while supporting sky/light, parallax foliage, bird plane, and ornaments.

- **Damped parallax maximum around 10px:** The rationale is depth without becoming a layered-illustration showpiece.

- **Single rAF loop with dt clamp:** The rationale is suspended tabs must not integrate an hour of motion into one frame.

- **Stop rendering and suspend audio when hidden:** The rationale is battery, while server simulation continues.

- **Adaptive quality degrades ornaments/parallax/sky before bird motion:** The rationale is bird motion is the product and never disabled.

- **Bird actors from layered continuous oscillators:** The rationale is no cycles; two consecutive seconds are never identical and first frame is mid-motion.

- **Sprite atlas with runtime plumage tint:** The rationale is compact shipping of art while motion stays procedural.

- **Ban `frames[]` keyframe animation type:** The rationale is making cycle animation awkward to express.

- **No attract mode or idle timeout state:** The rationale is micro-motion remains the same whether or not the user interacts.

- **Quiet field before JS:** The rationale is the load state is indistinguishable from the aviary's own sky.

- **Critical module with vector bird silhouettes before atlas decode:** The rationale is first bird visible under 500ms without depending on image decode.

- **Atlas detail cross-fade:** The rationale is plumage refinement, not a bird appearing.

- **Empty-aviary soft fly-in for first adoption:** The rationale is the only entry animation because a bird arriving is the event and happens once per bird.

- **Loading-path forbidden components and copy:** The rationale is avoiding semantics of "the app is starting."

- **Reduced-motion pose-driver substitution:** The rationale is calmer register with designed charm while retaining calls, drift, mood, and notebook.

- **Prose package shared by client and server:** The rationale is one lexicon, one grammar, one voice across narration, captions, and notebook.

- **`forbidden.ts` banned subjects and phrasings:** The rationale is prevent prose from observing the user, announcing, gamifying, or shifting voice.

- **Property tests over generated prose:** The rationale is tone violations fail tests rather than relying on reviewers.

- **Screen-reader narration client-side:** The rationale is match the current frame, avoid round trip, and respond immediately to user events.

- **Notebook server-side:** The rationale is it needs multi-day history.

- **Two polite live regions, never assertive:** The rationale is assertive interruption is the auditory form of a toast.

- **Idle and priority narration cadence with rate limiting/coalescing:** The rationale is natural observation without flooding.

- **Narration through observable details, not labels:** The rationale is same as visual surface: mood conveyed through consequences.

- **Captions near calling bird with collision avoidance and scrim:** The rationale is opt-in text accessibility that remains readable against all sky states.

- **Transparent button per bird over canvas:** The rationale is keyboard and AT access to canvas birds.

- **10Hz button position/name sync:** The rationale is avoid flooding assistive technology with 60Hz DOM mutations.

- **Aviary keyboard model:** The rationale is full keyboard operation. The plan specifically explains avoiding single-letter shortcuts because they collide with screen-reader quick-nav keys, but does not further explain every key choice.

- **Dual-tone focus ring:** The rationale is contrast against bright midday and night dim without per-state variants.

- **Top bar never fades during focus and returns on keydown:** The rationale is fade must not hide controls from keyboard users.

- **Axe, canvas contrast, reduced-motion screenshots, manual scripts, external audit:** The rationale is standard tooling cannot see canvas contrast, reduced-motion can rot, and affective accessibility matters beyond AA.

### 11. Performance budgets and observability

- **Internal 450KB target under 2MB PRD ceiling:** The rationale is 2MB is unrecoverable for affect and parse time; 500ms first bird is the real constraint.

- **500ms breakdown with audio/narration/top bar outside it:** The rationale is first-bird path must not depend on non-critical modules.

- **Frame, heap, WebAudio node, tick, snapshot, payload, availability budgets:** The rationale is no memory growth, smooth 30-minute sessions, timely ticks, small payloads, and availability of snapshot/auth. Some numeric choices are tied to the PRD; others are not further explained.

- **Client RUM aggregate buckets:** The rationale is know performance and failures without account dimension.

- **Server metrics aggregate by route/reason/provider:** The rationale is operate auth, tick, snapshot, notebook, invites, and email without per-bird telemetry.

- **Accessibility settings enabled counts:** The rationale is know whether surfaces are used without knowing who uses them.

- **No behavioral analytics, retention, DAU/MAU, cohorts, A/B framework, session recording, heatmaps, click analytics:** The rationale is those metrics create pressure for streaks and engagement features and violate the privacy boundary.

- **Typed metrics labels and allow-listed logs:** The rationale is make forbidden identifiers unassignable or dropped.

- **Simulation subnet with no analytics route/credential/replication:** The rationale is privacy boundary by architecture rather than policy.

- **Privacy policy naming aggregates and exclusions:** The rationale is making the commitment externally legible.

### 12. Testing and quality strategy

- **Engine property tests:** The rationale is injected clock and seeded PRNG make drift, determinism, idempotency, perch capacity, mood dwell, call windows, and greeting rules exhaustively testable.

- **Calibration tests:** The rationale is catching silent failures named by the PRD: over-fast drift, farmable drift, double-credited presence, drift-on-neglect.

- **Presence fixture tests:** The rationale is presence miscounting is the canonical corruption no test would catch unless explicitly tested.

- **Deterministic render tests:** The rationale is fixed inputs can catch visual regressions across viewport, lighting, weather, bird count, and reduced motion.

- **Audio graph assertions:** The rationale is enforce no hard mutes, no direct gain assignments, bounded nodes, and ramped mix changes.

- **Snapshot-stale simulation:** The rationale is client must keep rendering and calling without fresh snapshots.

- **Refusals suite:** The rationale is absolute product rules scale only if automatic: no gamification lexicon, notification components, deny-list endpoints, trait/mood schema, email leaks, infra leakage, inline strings, random/time calls in engine, frame cycles, or extra mail templates.

- **Definition of done:** The rationale is accessibility and voice are completion criteria for every visual or behavioral feature.

- **Load and failure testing:** The rationale is verify tick throughput, ingest, snapshot cache, crash replay, Redis degradation, email failover, and Postgres failover without correctness loss.

### 13. Rollout

- **Phase 0 foundations:** The rationale is establish budgets-as-gates, schema grants, auth, tick scheduler, event ingest, snapshot, edge HTML, and first bird early so refusals and first-paint constraints are present from the start.

- **Phase 1 bird engine:** The rationale is get full engine, calibration, sound motifs, synthesis, listen-in, presence, and greeting working early, with recognizability and presence exactness.

- **Phase 2 aviary as a place:** The rationale is complete species, weather, ornaments, top bar, offers, settle, adoption, responsive no-crop, and no-load-state experience as a place.

- **Phase 3 accessibility as designed surface:** The rationale is accessibility overlaps rather than follows visual work, with affective audit as exit.

- **Phase 4 notebook, account, social:** The rationale is add notebook, account, export/delete, and visits after core surfaces, with visitor read-only and deletion verified.

- **Phase 5 hardening:** The rationale is budgets, listening study, accessibility audit, privacy boundary, load/chaos, and runbooks before GA.

- **Sound design and prose start in Phase 1:** The rationale is long-lead creative work becomes engineer placeholders if compressed, producing canned calls and log-like prose.

- **Four-week internal dogfood:** The rationale is real elapsed time is the only way to validate a three-week drift claim as felt.

- **Closed beta:** The rationale is watch capacity, timings, audio-context failure, notebook sparsity, and support contacts in the wild without production-scale opening.

- **Waitlist drip:** The rationale is admit batches only when tick and snapshot budgets hold for 72 hours.

- **No launch or re-engagement email:** The rationale is no notification surface and no "your aviary misses you."

- **Offer schedule as server config:** The rationale is age-gated per PRD and adjustable cap without changing engine limits.

- **Seeded aviaries for pre-launch 3-7 bird validation:** The rationale is no GA account reaches three birds for three months.

- **Day-one instrumentation:** The rationale is operate critical budgets and failure modes from first beta while keeping §11.3 absent from day one.

- **Runbooks:** The rationale is named operational responses for backlog, storm catch-up, monotonicity trigger, email failure, `α` changes, and personality corruption without clamping or recomputing canonical vectors.

### 14. Risks

- **R1 drift calibration wrong:** The rationale is monotonic over-fast drift is permanent; detection and response are conservative `α`, ceilings, caps, dogfood, and narrow aggregate plumage-tier histogram.

- **R2 sync correctness fails silently:** The rationale is multi-device, stale, non-idempotent, or client-write failures must become structural loud errors rather than quiet data loss.

- **R3 procedural calls sound synthetic or chorus turns to mush:** The rationale is audio is the affective spine; response is designer-authored motifs, early listening, study gates, cap/species cuts, and no recorded fallback.

- **R4 500ms first-bird budget missed:** The rationale is real hardware may be worse than Lighthouse; response is vector-first birds, inlined snapshot, split modules, simpler silhouettes, never load state.

- **R5 accessibility regresses:** The rationale is narration, reduced motion, and keyboard support can drift; response is done criteria, prose package, peer render mode, screenshots, and audits.

- **R6 charm erosion by accumulation:** The rationale is many reasonable additions can cumulatively become fatal; response is invariant register, refusals suite, and PR template question.

- **R7 notebook prose reads canned or observes the user:** The rationale is the notebook can become repetitive or forbidden; response is sparsity budgets, noteworthiness, anti-repetition, forbidden tests, and writer review.

- **R8 tick cost or backlog at scale:** The rationale is tick throughput could lag; response is hibernation, batching, changed-row writes, horizontal workers, and catch-up cap.

- **R9 magic-link email delivery fails:** The rationale is failed email can lock users out of relationships; response is provider failover, domain auth, support, and export.

- **R10 privacy boundary erosion:** The rationale is a temporary join or CDC connector can pierce the boundary; response is infra-policy CI and data-boundary review.

### 15. Decisions this plan makes where the PRD is open

- **Presence cadence/input window/gap credit:** The rationale is the PRD says a few minutes, longer because watching birds without moving is the product.

- **60s/300s tick cadence:** The rationale is preserve continuation observably while cutting cost, with unobservable hibernation.

- **Six mood states including `settled`:** The rationale is night and post-settle need a distinct state.

- **Mood absent from client:** The rationale is user reads mood from motion, and a wire field enables a status dashboard.

- **Only quantized plumage crosses wire:** The rationale is minimum possible leak for rendering.

- **Client narration and server notebook from one prose package:** The rationale is current-frame narration, multi-day notebook history, and single voice.

- **Canvas 2D:** The rationale is no WebGL need against bundle and old-GPU constraints.

- **No realtime channel:** The rationale is polling is sufficient at 60s tick and removes connection bugs.

- **Listen-in not mirrored:** The rationale is device-local attention, not aviary state.

- **Settle last-write-wins:** The rationale is presentation state with no history.

- **Audio unlock by silent aviary and passive icon:** The rationale is avoiding a "tap to begin" entry gate.

- **`α` at 0.6× fitted:** The rationale is asymmetric cost of too-fast versus too-slow drift.

- **Species ceilings fixed at adoption:** The rationale is prevent monotonic homogenization.

- **45-day event retention:** The rationale is personality stored, interaction history minimized.

- **Export vectors but no in-app viewer/import:** The rationale is data portability outside product surfaces.

- **Offer schedule and adjustable config cap:** The rationale is age-only gating and recognizability contingency.

- **Narrow aggregate plumage histogram:** The rationale is avoiding blind launch of unrecoverable parameter.

- **Five species contingency:** The rationale is recognizability binds more than count.

- **35-day PITR:** The rationale is deletion completeness by architecture.

- **Client lighting authoritative; server timezone hint for mood:** The rationale is non-stale lighting and unwatched time-of-day mood evaluation.

### 16. Team shape and sequencing note

- **Engineer mapping to engine, client render, audio, and platform boundaries:** The rationale is the team shape follows the plan's service and domain boundaries.

- **Accessibility is not a person:** The rationale is accessibility is a completion criterion on every seat, independently checked by audit.

- **Do not compress dogfood or Phase 1 sound/prose:** The rationale is drift validation requires real weeks, and placeholder calls/prose are the canned failure the PRD says users detect and do not forgive.
