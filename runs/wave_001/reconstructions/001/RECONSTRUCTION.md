## System-level intent

- **A small living place, not an app or game.** The executive direction says Pocket Aviary should "feel like a small living place rather than an app or game." This shows up again in the rejection of mechanics that would turn it into "a game, a pet-care chore, a notification loop, or a canned animation surface."

- **Continuity as the central implementation promise.** The plan names "continuity" as the central promise: the aviary advances on the server whether the user is "present or absent," and clients render "the current canonical state without owning it." This principle drives the server-owned simulation tick, canonical state, snapshots, sync model, offline behavior, and return-greeting design.

- **Narrow but deep V1.** The plan says the team should build "a narrow but deep product." It limits V1 to one account, one canonical aviary, two starter birds, web-only support, opt-in visits, and core accessibility, while excluding broad social, native, and game-like systems.

- **Quiet presence shapes the aviary without making absence failure.** The summary says "the user's quiet presence shapes a small living-feeling aviary over time." Presence must be "honest," requires visible/focused/recent activity, and absence "never" creates negative drift, distress, chore debt, or a "you have been gone" surface.

- **Slow, monotonic expressiveness over weeks.** The plan repeatedly frames growth as slow: birds become "more expressive over weeks of honest presence"; drift is measurable after "about one week" and visible after "about three weeks"; "no single session creates a visible trait jump"; deltas are "never negative due to absence."

- **Reject attention-as-score and care-chore mechanics.** The plan says anything that "turns attention into a score," exposes hidden numeric traits, makes absence failure, or adds achievements, streaks, scores, hunger, death, sickness, distress, or happiness meters must be rejected at design and code review.

- **Hidden mechanics, stable identity, and relationship privacy.** Hidden vectors drive behavior, but "the user never sees hidden numerical personality or drift values." Bird identity must remain stable across renames, sync, migrations, and species asset updates. Per-bird interaction and personality data are only for that user's simulation and export, not analytics, training, recommendations, or dashboards.

- **Naturalist product voice separated from matter-of-fact system voice.** Product surfaces use "naturalist voice," with lowercase, present-tense, specific prose. Account, sync, settings, error, unsupported-browser, visit revocation, and accessibility configuration surfaces use "matter-of-fact system voice."

- **Accessibility is the same aviary, not a fallback.** The plan says accessibility ships with V1 and that accessibility modes are "alternate presentations of the same aviary, not stripped fallbacks." Screen-reader narration, reduced motion, captions, keyboard navigation, and contrast compliance carry the same product care as visuals and audio.

- **Procedural liveness instead of canned loops.** Calls are procedural WebAudio from grammar and stable bird seeds; captions are generated from "the exact motif execution"; recorded loops and recorded fallback audio are excluded. Animation likewise avoids "visible looping" through stochastic timing, pose variation, desynchronization, and slow motion.

- **Determinism, auditability, and strict ownership.** Simulation uses deterministic seeded randomness so retries do not change canonical results. Interaction events are append-only, ordered by server sequence, and consumed by ticks. Presence windows, drift ledgers, tick records, and schema/versioned static definitions make calibration, review, and ownership auditable.

- **Sharing stays quiet, opt-in, and read-only.** Visits are off by default, invitation-based, render-only, revocable, and do not create presence, interactions, greetings, drift, co-presence, comments, profiles, feeds, or discovery. Visitor identity is not represented inside the aviary.

## Per-feature whys

**Executive Direction and Product Scope**

- **One account, one canonical aviary:** The plan uses this to support continuity and remove multi-aviary or shared-account complexity. Later it says one canonical aviary means devices "do not merge state"; they submit events and read snapshots.

- **Web-only modern browser support:** The plan scopes V1 to browser-only and later says not to ship old-browser compatibility paths that "bloat the critical bundle," tying browser support to first-bird performance and a focused V1.

- **Email magic-link sign-in/account creation:** NOT RECOVERABLE FROM PLAN.

- **Per-device sessions and session revocation:** The plan treats sessions as account/security surfaces: tokens are hashed, revocation is "immediate by token invalidation," and settings expose device/session control.

- **Account export and deletion:** Export supports account/data rights while keeping personality vector numbers "outside normal product UI." Deletion is part of privacy/data governance, with soft deletion, recovery, and hard deletion of account, aviary, birds, vectors, events, notebook, visits, exports, sessions, and telemetry linkages.

- **Two starter birds and growth up to seven:** The plan calls for a "narrow but deep product" and later says additional birds become available by "aviary age only," not visits, offers, streaks, payment, or achievements. The seven-bird cap is also tied to chorus recognizability and cap validation.

- **System-selected starter birds from a small species pool:** NOT RECOVERABLE FROM PLAN.

- **User naming and renaming of birds:** NOT RECOVERABLE FROM PLAN.

- **Hidden per-bird personality vectors:** The rationale is expressiveness without stats. Vectors shape boldness, social warmth, vocal frequency, plumage saturation, and curiosity, but raw values are hidden from ordinary UI so the product does not become a score or stats panel.

- **Persisted per-bird mood state:** Mood persistence supports continuity across sessions. Mood drives perch preference, pose, greeting likelihood, call rate, narration, and notebook facts, making the birds feel like ongoing beings rather than fresh animations each load.

- **Slow personality drift dominated by valid presence-time:** The plan says quiet presence should shape the aviary over time. Drift is slow, low-pass, monotonic toward expressive traits, and never downward from absence, preserving calm growth without neglect punishment.

- **Return-greeting behavior:** The plan makes greeting a canonical, state-driven session entry behavior, not a "client-only canned animation." It should happen without a toast, banner, modal, or text announcing return.

- **Listen-in interaction:** Listen-in creates focused attention toward one bird while keeping the whole aviary present. The audio section says this must feel like "leaning attention toward one bird, not soloing a track."

- **Offer interactions for seed, song fragment, and still pool:** Offers give calm interaction hooks and modest drift inputs while avoiding gameable control. The server resolves which bird notices first, cooldowns prevent farming, and the UI must not feel like "an inventory or action bar."

- **Settle gesture with five-second undo:** Settle ends the presence window and mood-quiets the aviary without creating "a required goodbye obligation." Re-engagement within five seconds undoes it, and closing the tab without settling remains "equally valid."

- **Field notebook:** The notebook gives "rare, specific, naturalist observations" rather than event logs. It avoids user visit frequency, streaks, and numerical traits, and it is read-only so it remains observation rather than social or editing surface.

- **Thin top bar:** The top bar keeps account/settings, accessibility, notebook, and offers available without turning the scene into UI chrome. It uses sparse icons, fades after stillness, and remains discoverable for keyboard and screen-reader users.

- **Day/night cycle tied to local timezone:** Day/night supports continuity and mood. The plan ties phase to account timezone and server time, quiets calls in evening, nudges drowsiness, and keeps night "dim but not dead."

- **Rare ambient weather and ornaments:** Weather adds mild ambient variation and affects mood/call rate modestly, but it must never become a managed feature or "notification-worthy event."

- **No panning, scrolling, zooming, or offscreen birds:** The scene design keeps every bird visible across desktop and mobile. Responsive perch mapping compresses spacing without cropping birds and avoids user control of layout.

- **Screen-reader narration, reduced motion, captions, keyboard navigation, and contrast compliance:** These ship in V1 because accessibility is product-critical and not follow-up work. They are alternate presentations of the same aviary, with complete mood, calls, notebook, and drift behavior.

- **Quiet opt-in read-only visits:** Visits let someone observe without changing the aviary. They are off by default, deliberate, revocable, and do not contribute presence, interactions, greetings, call scheduling changes, or drift.

- **Host visit log:** NOT RECOVERABLE FROM PLAN.

- **Aggregate operational telemetry:** The rationale is operational health without relationship analytics. The plan allows request counts, latencies, error rates, frame timing, audio errors, and tick latency, while excluding per-bird/per-account interaction content, drift averages, names, and payloads.

- **Forbidden game, chore, distress, social, and notification features:** These are excluded because they would violate tone and mechanics, turning the product into a game, pet-care chore, notification loop, public social surface, or score system.

**System Architecture, Data Model, and API Surface**

- **Modular monolith plus workers:** The plan says to start here because the simulation stays "transactionally close to the database" while worker processes can run separately. It prioritizes code ownership boundaries over "premature microservices."

- **API edge with HTTP JSON endpoints:** NOT RECOVERABLE FROM PLAN.

- **Relational primary database:** NOT RECOVERABLE FROM PLAN.

- **Job system queues:** NOT RECOVERABLE FROM PLAN.

- **CDN/edge cache and signed initial state envelope:** The rationale is fast first render: the shell is served efficiently and an optional signed initial state envelope can support "fast first render."

- **Observability pipeline with redaction and schema guardrails:** Observability is allowed for aggregate operations, but guardrails prevent leaks of raw emails, tokens, event payloads, per-bird state, and relationship-derived metrics.

- **Server-owned account identity and canonical aviary state:** Server ownership protects correctness, determinism, privacy isolation, and low-latency snapshots. The server owns identity, sessions, birds, personality, moods, drift, positions, notebook entries, exports, visit permissions, and snapshots.

- **Client rendering, interpolation, and ephemeral state:** The client makes the scene continuous and responsive while not owning canonical state. It may keep animation phase, audio envelopes, focus state, and pending UI, but canonical changes happen only through accepted events.

- **Semantic render instructions in snapshots:** The plan says snapshots should contain semantic instructions, not final animation frames, to keep bandwidth small and avoid pretending server ticks operate at animation-frame rate.

- **UUIDs, encrypted email, and keyed email hashes:** The plan uses synthetic UUIDs internally, encrypted email only on account/invitation records, and keyed hashes for lookup/rate limiting so email is not a key, log identifier, telemetry dimension, or exposed value.

- **Separate `bird_personality_vectors` table:** The plan says storing vectors separately makes "write ownership and audit checks strict," with only simulation code having write access.

- **`bird_drift_ledgers`:** The ledger supports debugging, calibration, and export while remaining internal only and not feeding aggregate analytics.

- **`interaction_events` with idempotency and server sequence:** Append-only events let clients submit intent without canonical writes. Idempotency keys prevent duplicates, server sequence numbers give deterministic processing order, and ticks record consumed ranges.

- **`presence_windows`:** Materializing presence windows makes simulation calibration auditable and avoids repeatedly reconstructing credited presence from raw pings.

- **`state_snapshots` read model:** The JSON read model is safe because it is generated from canonical tables, but it is not the sole source for personality vectors.

- **`notebook_entries`:** Notebook entries persist read-only naturalist observations with source ticks/birds and rarity score. Their purpose is sparse observation, not editing or event logging.

- **`visit_invitations` and `visit_sessions`:** These authorize read snapshots and support the host's on-demand visit log while keeping visitor identity out of the aviary scene.

- **Versioned static content and config:** Species, mood, offer, call grammar, notebook, narration, and calibration definitions are versioned so migrations can preserve bird identity and call recognizability when assets change.

- **Account export JSON shape:** Export includes account, aviary, birds, notebook, visits, and summaries. Personality vector numbers may appear here because the plan frames export as a data-rights/account surface, not ordinary product UI.

- **API write and snapshot principles:** Idempotency, `state_version`, `server_time`, cache-control, event submission instead of canonical state, scoped visitor auth, and matter-of-fact errors all support predictable sync and system-surface clarity.

- **Magic-link request generic response and rate limits:** Generic success avoids revealing account existence. Keyed email hash and coarse IP bucket rate limiting protect the auth surface without raw email logging.

- **Aviary bootstrap omits raw personality vectors:** The bootstrap contains mood, pose, visual bands, call seeds, and static versions for immediate rendering, but "do not include raw personality vector values" preserves hidden mechanics.

- **Bird rename endpoint as account action:** Rename writes a name/account event, not a personality event, so user-owned text does not become drift input.

- **Event validation and optimistic reconciliation:** Server validation enforces schemas, size limits, ownership, cooldowns, time bounds, and settle undo windows. The client should only optimistically animate low-risk reactions if it can reconcile with the next snapshot to avoid divergent behavior.

- **Offer flow reaction descriptors:** Reaction descriptors drive rendering and narration without becoming announcements. They let the server confirm mood/drift-relevant outcomes before the client commits to visible behavior.

**Simulation, Sync, Frontend, Audio, Accessibility, Privacy, and Delivery**

- **Listen-in flow events and server-computed duration:** The client can begin an audio ramp immediately for responsiveness, but the server computes final drift duration from consumed event sequence and timestamps.

- **Settle flow events:** `settle_start`, `settle_undo`, and `settle_confirmed` let settle end presence and quiet mood while preserving the five-second undo and avoiding a goodbye obligation.

- **Visit APIs:** Visit APIs create deliberate, revocable read-only sessions. Visitor snapshots exclude host-only settings, interaction affordances, and unnecessary account details; revoked or expired visits return matter-of-fact errors.

- **Error surface contract:** Standard errors use matter-of-fact copy because auth, sync, settings, visit, unsupported-browser, and accessibility configuration are system surfaces, not naturalist prose.

- **Adaptive tick scheduling:** Active aviaries tick around once per minute, inactive aviaries still tick with batching/coalescing, and deletion stops nonessential simulation. The rationale is preserving day/night, mood, drift, and continuity without unnecessary work.

- **Transactional tick pipeline:** The tick loads state/events, computes deterministic transitions, applies deltas, updates positions, creates notebook entries, consumes event ranges, increments state version, and writes the snapshot so canonical changes stay coherent.

- **Deterministic seeded randomness:** Seeds from aviary, tick, bird, and time bucket ensure retries do not produce different canonical results.

- **Honest presence pings:** Presence requires visible document, focused window, and recent pointer or keyboard activity. The plan uses these conditions to distinguish real quiet watching from tab-open inflation.

- **Server presence processing and clamping:** The server builds windows from consecutive valid pings, caps gaps, clamps credited time to server receipt windows, and ends presence on settle, hidden, blur, expiry, or missing ping to prevent fabricated long sessions.

- **Drift low-pass function:** Drift should be measurable after about seven days, visible after about twenty-one days, never negative from absence, and resistant to offer spamming through cooldown, low-pass smoothing, and headroom.

- **Mood weighted transitions:** Mood is fast-timescale and shaped by current mood, time of day, weather, recent events, personality, neighboring birds, and small seeded noise. It drives perch zone, pose, greeting likelihood, call rate, narration, and notebook facts.

- **Return-greeting selection:** The server ranks birds by boldness, social warmth, mood readiness, recent greeting history, absence bucket, time, and current pose so one bird does not always greet and the scene avoids simultaneous all-bird greetings.

- **Perch and pose selection:** Perch zones communicate mood and personality: bold/content birds favor front or middle, wary birds favor back, drowsy birds favor stable lower poses, and curious birds shift toward sounds/offers. The user cannot control perches.

- **Weather and day/night scheduling:** Server-scheduled rare rain/wind and timezone-derived day phase add mood and call variation while staying mild, capped, unmanaged, and non-notification-worthy.

- **Offer cooldowns and server targeting:** Per-bird and per-offer cooldowns plus low-pass filtering prevent single-session farming. Server targeting by mood, curiosity, boldness, proximity, and history keeps response behavior canonical rather than click-controlled.

- **Notebook generation pipeline:** Simulation emits candidates for noteworthy conditions, sparsity rules keep entries rare, templates keep lowercase present-tense naturalist voice, and constrained generation is safer, cheaper, reviewable, and easier to keep on voice than free-form LLM generation in V1.

- **Call grammar runtime and captioning:** Stable species motifs and per-bird seeds keep calls recognizable. Drift changes rate and expressive variation, not identity. Captions come from actual motif execution so they are not static species labels.

- **Canonical state sync model:** One canonical aviary record per account removes client-to-client sync as a problem. Devices submit events and read snapshots instead of merging state.

- **Server event ordering:** Monotonic per-aviary sequence numbers, transaction assignment, consumed ranges, idempotency constraints, and late-event rules prevent retries or old events from rewriting prior tick state.

- **Conflict prevention:** The plan avoids conflict surfaces by design: no client writes personality or mood, bird names use ordinary account mutation, settings use per-setting updates, and revocation invalidates tokens immediately.

- **Multi-device union presence:** Multiple host devices can be open, but concurrent valid presence is credited by wall-clock union, not summed, so two devices open for ten minutes count at most ten minutes.

- **Offline and suspended clients:** V1 does not promise offline operation because clients must not advance canonical simulation locally or accumulate long offline presence. On reconnect or long frame gap, the client fetches a fresh snapshot and interpolates.

- **First render and critical renderer:** The goal is "first bird visible within 500ms" on mid-tier mobile over 4G. The plan avoids blocking first bird on WebAudio, notebook, settings, full atlas, or non-current species assets, and avoids spinners.

- **Scene composition and layering:** One horizontal scene with three perch zones, responsive mapping, and no in-scene buttons/badges/labels keeps the aviary visual rather than a control surface and keeps birds visible without panning or cropping.

- **Animation model and desynchronized motion:** Local bird state machines turn server pose/mood/transition/call inputs into continuous motion. Stochastic timing, pose variation, slow movement, and desynchronization avoid visible loops and all-bird beat changes.

- **Reduced-motion rendering:** Reduced motion is "not static." It uses cross-fades and removes drifting ornaments and flight paths while keeping mood, calls, captions, notebook, and drift unchanged, preserving the same aviary.

- **Top bar behavior:** The bar fades nearly transparent after stillness to keep the scene quiet, but appears on pointer, keyboard, or focus and remains in the accessibility tree so controls remain discoverable.

- **Listen-in UI:** Click/tap or keyboard focus plus Enter toggles listen-in; disengagement paths are explicit. The visible focus should be subtle but accessible, supporting attention without breaking the scene.

- **Offer UI:** A small calm top-bar menu with seed, song fragment, and still pool keeps offers deliberate and keyboard navigable while avoiding inventory/action-bar feeling.

- **Settle UI:** Lighting shifts and calls quiet over a few seconds; any click/key re-engagement undoes within five seconds. The plan explicitly says there is no recovery surface because closing the tab without settling is valid.

- **Notebook UI:** The notebook is a read-only panel with naturalist entries and no edit, delete, annotate, react, share, or export-from-notebook affordance, keeping it observation rather than social content.

- **Screen-reader semantics and narration manager:** A named aviary region, roving bird focus, polite live region, coalescing, and 30-60 second idle cadence make narration useful without high-frequency mechanical status updates.

- **Browser and device support:** Supporting latest two major browser versions and detecting WebAudio, renderer capability, motion preference, and visibility/focus APIs lets the product stay modern and avoids old-browser paths that bloat the critical bundle.

- **WebAudio architecture:** Audio modules separate context lifecycle, scheduling, synthesis, voice registry, mixing, captions, and aggregate telemetry. Node pools and cleanup prevent leaks during long sessions.

- **Listen-in audio mix:** Focused bird gain ramps up slowly, non-focused birds ramp down only to an ambient floor, and the ambient scene remains present so listen-in feels like attention, not soloing.

- **WebAudio fallback:** If WebAudio cannot run, the product uses captions by default and a matter-of-fact settings note, keeps visuals/narration/drift/notebook, and avoids recorded fallback audio that would introduce canned loops.

- **Accessibility V1-critical implementation:** The plan says accessibility features ship with V1 and are product surfaces with the same design QA as visuals and audio.

- **Captions:** Captions are off by default when audio works, on by default when audio fails, accessible visually and programmatically, placed near the calling bird, and generated from actual call grammar execution.

- **Keyboard navigation:** Tab, arrow keys, Enter/Space, Escape, and possible top-bar shortcuts let a keyboard-only user operate core flows with stable focus order and no hidden scene controls.

- **Contrast and settings language:** WCAG AA contrast applies to text surfaces, captions, forms, errors, and settings. Accessibility settings use matter-of-fact language because they are system surfaces.

- **PII boundary:** Email is encrypted where necessary, never used as a primary key, shard key, log identifier, telemetry dimension, or queue partition key, and raw tokens/links/session tokens are rejected from structured logs.

- **Simulation data boundary:** Per-bird events and personality state are only for the user's simulation and export. The plan forbids analytics warehouse reads, aggregate drift dashboards, ML training, recommendations, and relationship-derived metrics.

- **Auth security:** Short-lived magic links, immediate invalidation, hashed session tokens, secure HTTP-only cookies where possible, CSRF protection, rate limits, session revocation, and verified email change protect account access.

- **Account deletion flow:** Soft deletion allows a 30-day cancellation window; hard deletion is idempotent and auditable and removes account-dependent data.

- **Performance budgets, observability, synthetic monitoring, and instrumentation:** The plan tracks bundle size, first-bird timing, frame timing, memory, audio init, tick latency, queue lag, exports, deletion, and visit latency to protect the living-feeling experience and operational health without per-bird analytics.

- **Rollout, testing, review gates, and launch readiness:** Milestones, cohort simulations, browser and accessibility tests, tone/invariant QA, workstream ownership, schema contracts, review gates, and launch checklist enforce the few strict rules that keep V1 precise, calm, privacy-preserving, accessible, and technically coherent.
