# Pocket Aviary - phase-2A reconstruction from PLAN

## System-level intent

1. **"Aliveness is load-bearing."**  
   Where it shows up: the binding product invariants, the render pipeline, "First frame and loading," the performance budgets, and the acceptance snapshot. The plan treats an already-moving scene as a correctness constraint: "First painted frame is mid-action," "0 spinner," "first RAF presents mid-action," and "meet two already-moving birds within 500ms."

2. **The product should notice the user without announcing or rewarding them.**  
   Where it shows up: "Notice, never announce," return-greeting, notebook writer, top bar, visits, and the acceptance snapshot. The plan repeatedly bans welcome copy, "you've been gone N days," badges, streaks, achievements, and second person, while still requiring that the user "be noticed (not announced)."

3. **Idle attention is the primary interaction.**  
   Where it shows up: the binding invariant, presence detection, drift, keepalive/resume, and testing. Presence is the "triple conjunction" of visible, focused, and recent pointer/key activity, with a 4-minute activity window and 30s pings, because "watching without moving is the product."

4. **Personality is canonical, hidden, and monotonic toward expressive.**  
   Where it shows up: the binding invariants, named decisions around personality on the wire, data model, drift, sync, telemetry, ARIA, and CI tests. The server is the only writer; clients never write traits; neglect never decrements traits; snapshots send "derived render params" only; the user never sees personality numbers except in export.

5. **Quietness is an envelope, not punishment.**  
   Where it shows up: the `expression_gain` decision, drift calibration, mute handling, settle, and risk mitigations. The plan separates "ambient quietness" from negative drift: `expression_gain` may fall, but traits do not, and a floor "keeps the place alive."

6. **Voice split is mechanical.**  
   Where it shows up: the binding invariant, API errors, top bar, notebook, narration, captions, accessibility copy, and team working agreements. Product surfaces use "lowercase naturalist present-tense"; system surfaces use "matter-of-fact sentence case."

7. **The aviary is a place, not a game or social network.**  
   Where it shows up: out-of-scope items, visits, telemetry, notebook bans, top bar bans, rollout, and acceptance. The plan excludes scores, streaks, badges, levels, XP, public discovery, profiles, follows, comments, chat, leaderboards, hunger/death/distress meters, and "collect all seven" framing.

8. **Identity continuity matters.**  
   Where it shows up: stable bird UUIDs, starter-pair determinism, age-gated arrivals, export/restore, migrations, content freeze, and risks. The plan says rename, species edits, migrations, and export/reimport must not mint a replacement bird; changing a learned call interval set is "an identity break."

9. **Procedural expression protects voice, privacy, and identity.**  
   Where it shows up: notebook/narration/caption authorship, calls, audio fallback, and risks. The plan chooses constrained grammars and procedural audio, not third-party LLMs or recorded loops, for "voice control, privacy boundary, offline-determinism," and because calls must never repeat exactly.

10. **Visits are deliberately low-gravity.**  
    Where it shows up: the binding invariant, visit flow, visit telemetry, visit a11y, and social-gravity risk. Visits are "read-only, opt-in per invite, off by default, non-co-present"; visitors cannot write events, read notebooks, trigger fly-ins, or drift host birds.

11. **Performance and accessibility are product constraints, not polish.**  
    Where it shows up: v1 scope, first-bird budgets, 60fps and memory budgets, reduced-motion, screen-reader narration, keyboard, captions, launch gates, and risks. Accessibility "ships in the same release train as the scene," and the first-bird target shapes architecture.

12. **V1 should stop at a small, coherent whole.**  
    Where it shows up: out-of-scope, rollout, launch shape, team agreements, and the final line "That is the whole v1. Stop there." The plan treats new surfaces about money, stats, recorded audio, toasts, public discovery, and native clients as product-drift risks.

## Per-feature whys

### Scope and named decisions

- **Browser-only SPA + API:** NOT RECOVERABLE FROM PLAN

- **Email magic-link auth and uniform magic-link responses:** The uniform `202` response exists "to avoid enumeration." Single-use consumption and replay returning the expired-link error prevent a replay from minting a second session.

- **Per-device revocable sessions:** NOT RECOVERABLE FROM PLAN

- **Email change with verify-before-swap:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account, with the server as only personality writer:** The plan's rationale is sync correctness and identity stability: personality "cannot diverge" because only `sim.apply` writes it under the aviary row lock, in received order.

- **Server-side simulation with 60s logical ticks, lazy catch-up, and an eager worker for recent presence:** The reason is to be semantically "continues without the viewer" while staying "operationally affordable"; lazy catch-up keeps absence affordable, and the eager worker keeps interactive sessions warm.

- **Event-triggered micro-ticks for greetings and offers:** The plan says greetings and offers "cannot wait a minute." Micro-ticks provide "aliveness of gesture" without letting clients author traits.

- **Two starter birds chosen by the system:** The starter pair is deterministic so retry/refresh "cannot re-roll identity." The birds use different registers and complementary seeds so the "first chorus is separable by ear."

- **Age-gated arrivals up to seven:** The gates are meant to match "few months -> third; year-old -> five or six." The arrival has no catalog because "First encounter is meeting, not configuring."

- **Six-species pool with one night-active nighthawk:** The plan's rationale is a "coherent temperate-edge set" and "one nocturnal call signature."

- **Single horizontal no-pan no-zoom scene:** The plan ties this to keeping the world stable and every bird visible: resize recomputes perch positions so "every bird remains in frame," and narrow phones compress spacing but "never crop."

- **Accessibility shipping on day one:** The plan says accessibility is "Not a v1.1." It is a launch gate because the product must work with keyboard, screen reader, reduced motion, and silence+captions in the same release train as the scene.

- **Visit invitations that are read-only, opt-in, off by default, and non-co-present:** The plan's why is "follow the link and see" without social gravity. Visitor attention must not drift host birds, visits get no badge, and defaults stay off.

- **Account export JSON emailed:** NOT RECOVERABLE FROM PLAN

- **Soft-delete for 30 days, then hard-delete:** The plan's restore rationale is that recovered birds are "the same birds," not "30 days of unattended drama" and not stuck in old lighting. The hard-delete rationale is privacy: wipe birds, events, notebook, visits, sessions, and account-tied telemetry with "No backup warehouse copy."

- **Performance budgets and first-bird target:** The plan says the 2MB cap is "a ceiling, not a goal" and "cannot hit 500ms on mid-tier 4G"; the first-bird target drives inlined snapshot, code-splitting, CSS sky, and no tag manager.

### Architecture, data, and API

- **Three deployables in one monorepo with shared packages:** The plan uses packages for sim, prose, species, protocol, and telemetry so the API, worker, client, grammars, render contracts, and allowlists share one set of definitions instead of inventing parallel products.

- **`packages/sim` as pure functions plus persistence adapter:** The module must run identically in API and worker with "No hidden wall-clock I/O" so catch-up, micro-ticks, and eager ticks produce deterministic results.

- **Client/server ownership split:** The split protects canonical state: the server owns account identity, birds, personality, weather, notebook, visits, age gates, and exports; the client owns rendering, interpolation, procedural audio, presence detection, and chrome.

- **Boot pipeline with inlined snapshot, scene-core first, audio and chrome later:** The plan calls the inlined snapshot the "Primary lever for time-to-first-bird." Drawing the scene before worklet and chrome preserves the mid-action first frame.

- **HTTP pull plus event POST, with no WebSocket in v1:** The plan says the product model is "pull/interpolate" and WebSocket would add "presence-sync complexity we do not need."

- **Redis limited to tokens, rate limits, and short hot snapshot cache:** NOT RECOVERABLE FROM PLAN

- **No analytics warehouse connection to the sim database:** The reason is the privacy boundary: operational metrics emit from process memory, "never by selecting bird rows," and per-bird interaction history never enters aggregate telemetry, warehouses, or training sets.

- **Aviary row lock, one ordered log, and no CRDT/client merge:** The plan deletes the last-write-wins failure mode where an old device snapshot overwrites a newer morning presence vector. Additive deltas from the log make that unrepresentable.

- **Authoritative server time and received-order application:** Client timestamps are kept only as ordering hints; the server stores both timestamps and "applies in `server_received_at` order" because client time is never trusted for drift.

- **Synthetic account UUID, encrypted email, and lookup hash:** The rationale is PII containment: email lives in one encrypted column plus a lookup hash and is not a partition key, log field, analytics dimension, Redis key, or visit URL.

- **Stable bird UUIDs, server-only traits, and derived render snapshots:** Stable UUIDs preserve identity through rename, species edits, migrations, and export/reimport. Derived render params prevent a stats panel from being "one DevTools copy away from becoming UI."

- **Append-only event log with idempotent client-generated event ids:** Idempotency lets re-posts be no-op successes, and the append-only log gives the server a single ordered source for catch-up and drift.

- **Notebook `fact_keys` hidden behind opaque observation ids:** The `fact_keys` support rarity control such as "already wrote greet_order_inverted this week," but they must not expose trait names or internal facts to the client.

- **Visit tokens sufficient for visitor access and hashed at rest:** The plan says this matches "follow the link and see." The visitor need not have an account, and token hashes protect the invite secret at rest.

- **Snapshot shape with no traits, no presence totals, no visit counts, and no days-since field:** Absence and traits affect server-side directives only. The why is to keep personality numbers and streak-like surfaces out of UI, ARIA, captions, and debug panels.

- **Export as the one user-facing place personality numbers exist, under account settings:** The plan allows raw personality numbers only in schema-versioned export and says not to link export from aviary chrome because export belongs to matter-of-fact account settings.

- **Deliberately not storing click scores, daily visit calendars, heatmaps, visitor drift, per-leaf entities, or plaintext email:** The rationale is to avoid turning attention into scores, streak dashboards, heatmaps, or PII spray.

### Simulation and sync

- **Trait drift only in the slow tick and only upward:** The plan's rationale is monotonic expressive growth without Tamagotchi decay. "Neglect" leaves traits unchanged while `expression_gain` may fall.

- **Calibration harness for regular, neglect, mash, and rate changes:** The harness prevents too-fast Tamagotchi behavior, too-slow screensaver behavior, and offer spamming. `rate_week` should change only through harness diffs reviewed as a product change.

- **Aviary-level `expression_gain`:** It implements "ambient quietness without Tamagotchi punishment." It can fall as a separate envelope, while the floor keeps "the place alive."

- **Species visual assets as vector path kits, not large bitmaps:** The plan's rationale is compact procedural variation and performance: silhouettes, palettes, and path kits can be parameterized by plumage without blowing 60fps or bundle targets.

- **Starter pair algorithm excluding nighthawk and choosing different registers:** The plan says refresh cannot re-roll identity, the first chorus should be separable by ear, and nighthawk is reserved for later age-gated arrival.

- **Mood driven by local solar phase, weather, neighbors, personality, interaction, noise, and hysteresis:** The local-time rationale is explicit: "Local morning must be aviary morning." Hysteresis and persistence avoid snapping every session to `content`.

- **Perch intent and user inability to assign perches:** NOT RECOVERABLE FROM PLAN

- **Bird-to-bird answers, chorus cap, wary spread, and no bonds/fighting:** The answer logic creates place-like chorus behavior, while the plan forbids "fighting, dominance meters, no user-visible bonds" to avoid social stats and game systems.

- **Procedural call grammar with stable interval sets, individual seeds, and mandatory jitter:** Calls are procedural because "No recorded call loops" is binding. Recognizability lives in interval set and timbre, while jitter ensures two realizations never replay exactly.

- **Return-greeting as one primary greeter with staggered secondary glances:** The plan's why is "be noticed, not announced" and "Never unison." The greeting is snapshot-scoped and procedurally varied so it does not become canned welcome copy.

- **Offers: seed, song fragment, and still pool shaped by mood and curiosity:** Offers are gestures with immediate micro-tick reactions, not trait writes. The shared cooldown "prevents curiosity saturation" and keeps the "gesture register."

- **Settle with 3s lighting shift, 5s undo, and no scold path:** Settle closes the presence window cleanly, makes the aviary quiet without guilt, and lets closing the tab be equivalent to ending presence.

- **Rare weather with no feature surface or forecast:** Weather is ambience, not a feature surface. The plan says no forecast, no thunder/snow, and no notebook entry for every weather event.

- **Age-gated arrival fly-in, host-only ack, and visitor non-triggering:** The first host snapshot makes arrival a meeting; visitors neither trigger fly-in nor acknowledge it, preserving the host's first encounter.

- **Notebook writer with sparse constrained prose grammar and voice lint:** The why is rarity, privacy, and tone. The assembler avoids LLM-authored text, second person, trait deltas, presence totals, visit counts, achievement language, and gamified words.

- **Deterministic catch-up tick:** The same `(state0, events, rng_seed, tz)` must produce the same `state1`; collapsed empty minutes make two-week catch-up affordable while preserving equivalence to minute stepping.

- **Offline/flaky event queue with no client-side sim:** The client may queue events and show limited optimistic settle/listen visuals, but it must not "upload the truth." Server simulation remains canonical.

- **Snapshot interpolation contract:** The contract avoids teleporting, freezing, T-poses, and replayed greetings. Past `t0` starts actions mid-cycle, and resume gaps cross-fade so birds do not warp.

### Frontend, audio, accessibility, performance, rollout, and testing

- **First-run flow from sign-in to quiet field to naming two suggested names to fly-in:** The plan's rationale is that first-run should be naturalist and in-motion, "not a catalog," with no marketing carousel in-app.

- **Scene composition with one viewport-fitting world, depth planes, muted palette, and no crop:** The why is a readable place where all birds stay in frame across devices; nighthawk stays readable at dusk "not neon."

- **Bird drawing with pose tables, tiny bone chains, mood-shaped idle, and breathing:** Idle "never hard-stops"; breathing makes "paused" impossible, supporting the aliveness invariant without a general skeletal engine.

- **First-frame/loading behavior with quiet field, no spinner, optional faint leaf, and audio after first bird:** The reason is the load-bearing aliveness constraint: stay quiet if data is missing, but never use a spinner or static ready-pop.

- **Top bar icons, fade, no counts, no badges, and keyboard-safe restore:** The fade creates "Stillness without trapping keyboard users," while no counts or badges blocks notebook/visit/streak gravity.

- **Offer, notebook, and settle UX as lightweight panels over the aviary:** Offer remains a small gesture, notebook is read-only naturalist type with no search/share/like, and settle is one click with no confirm modal.

- **Reduced-motion mode as a designed renderer:** The plan explicitly rejects reduced motion as "off"; it uses still poses, cross-fades, slower solar shifts, no drift particles, and shared art while leaving drift, mood, notebook, and captions unchanged.

- **Visibility and battery behavior:** When hidden, the client cancels RAF, suspends audio, and stops leaf spawns, but sends `session_close`; on return, the server has continued the simulation.

- **60fps scene budget and ornament degradation order:** The reason is to preserve birds as the product center. If frame time is high, drop particles first, then shadows, "never bird idle."

- **Memory limits for particles, audio buffers, notebook DOM, and event queue:** The soak test enforces the plan's "no client memory growth over 30 minutes" budget.

- **Listen-in mix with 1000ms equal-power ramp and other birds floored at 0.28:** The plan says it should feel "like listening, not channel-switching," and the place should never be muted.

- **Chorus audio with independent schedulers, answer hooks, stereo by perch, and high-threshold compressor:** The risk rationale is to avoid a machine-like chorus, exact repeats, and pumping when two birds call.

- **WebAudio fallback as silence plus captions, with no mp3/ogg substitute and no scene toast:** The rationale follows "Calls are procedural" and avoids broken-speaker scene noise. If audio cannot run, captions cover the would-have-been phrase.

- **Mute handling with no negative drift:** The plan rejects a vocal-frequency penalty for mute because presence is "visual attention, not audio-on," and monotonic expressive/no punishment still applies.

- **Caption generation from realized motif descriptors:** Captions come from the same schedule as audio so caption and sound match; in silent fallback they describe the would-have-been phrase.

- **Screen-reader narration through one polite live region plus a parallel list of bird buttons:** The reason is to avoid live-region flood, avoid trait leakage, and mitigate blank Canvas exposure in some screen-reader/browser combinations.

- **Keyboard path and dual-color focus ring:** The rationale is full keyboard access across top bar, birds, listen-in, panels, and settle, with focus contrast holding against day and night palettes.

- **Visit-view accessibility with narration/captions but no activating controls:** The why is read-only visiting: visitors can know "which bird am I hearing about" without implying they can listen-in, offer, or settle.

- **Telemetry allowlist, stripped RUM, and synthetic dogfood exclusions:** The plan measures health and performance without per-bird research temptation. Forbidden dimensions include account id, bird id, species, mood, traits, presence minutes, and offer kind.

- **Browser support via feature detection and `/unsupported`:** The plan's rationale is to fail closed with matter-of-fact copy and avoid a "polyfill jungle" that would blow the bundle.

- **Privacy/security implementation for tokens, sessions, CSRF, rate limits, and isolated sim DB:** The why is PII containment and account safety: only token hashes are stored, email is encrypted/hashed, URLs use random tokens, and warehouse roles cannot select bird/event rows.

- **Rollout engineering slices with deterministic tick before visits or notebook chrome:** The reason is to avoid inventing "a second product" before canonical simulation works; each increment should be shippable to dogfood.

- **Bird-count ramp, non-prod time travel, and no support-minted birds:** Age gates are the ramp. If restoration is needed, support must reinsert the same UUID and last known traits, not create a replacement identity.

- **Content freeze for species art, motif libraries, notebook templates, and name lists:** The rationale is identity continuity: changing a learned species call interval set is "an identity break" and should be treated like a data migration.

- **CI invariants, Playwright checks, and PR review checklist:** The plan says green unit tests will not catch tone and product leaks such as tab-open presence, welcome toasts, personality in ARIA, recorded audio, or disguised streaks, so every relevant PR must walk the invariants.
