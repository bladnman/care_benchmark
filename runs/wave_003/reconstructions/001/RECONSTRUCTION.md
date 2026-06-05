## System-level intent

1. Felt aliveness over time is the primary product value. The plan says the "core value is felt aliveness over time" and repeats this through "server-side continuity," "procedural variation," "mood-shaped motion," and audio that "never repeats as a loop."

2. The aviary should feel like "a small place that continues without the viewer." This appears in the product interpretation, the server-side simulation tick, inactive aviaries that still tick, return greetings after absence, weather, day/night phases, and notebook entries as a "historical record."

3. The user is noticed by the aviary, not announced by the system. The plan states that "the bird greeting is the welcome surface," then forbids "Welcome back," text toasts, banners, and absence summaries. Return greeting is rendered as "ongoing behavior, not an overlay."

4. Presence is meaningful, but absence has no penalty. The plan states "Presence is real interaction, but absence is never punished," excludes decay, death, hunger, distress, and obligation mechanics, says "Neglect never decreases traits," and treats "Tab close without settle" as equally valid.

5. Canonical life belongs to the server. The plan repeatedly says the client "renders snapshots and submits events," while the simulation service is "the only writer of canonical aviary state," personality vectors, and moods. This shows up again in sync, conflict prevention, event ingestion, and tick locking.

6. Hidden state should be expressed through behavior, not exposed as product UI. Personality vectors are "never exposed to users"; mood is exposed "only through behavior, pose, call cadence, narration, and notebook prose, not labels or badges"; snapshots do not include raw personality vectors.

7. The product rejects game framing. The exclusions ban goals, scores, achievements, streaks, badges, XP, levels, rankings, counters, Tamagotchi-style obligation, and collectible rarity. Offer cooldowns avoid visible timers, adoption is not an "unlock," and copy must not say "reward."

8. Product voice is naturalist, sparse, lowercase, present-tense, and specific; system voice is matter-of-fact. The plan uses this split for notebook prose, captions, narration, settings, errors, privacy, auth, export, deletion, and visit surfaces.

9. Privacy is not only a compliance layer; it is "part of the product." The plan disallows per-bird analytics, personality vectors in telemetry, mood history tied to an account, behavior dashboards, and ML/recommendation use of per-bird records.

10. Accessibility ships as full product behavior, not a patch or fallback. The plan says "Accessibility ships with v1, not as a patch," requires narration, captions, reduced motion, keyboard access, and WCAG AA, and insists reduced-motion preserves "all simulation, audio, captions, notebook, and visit behavior."

11. Performance is part of the feeling of aliveness. The plan requires "First bird visible within 500ms," "No spinner," "first visible frame" already in motion, and a quiet field with faint motion if the snapshot is slow.

12. Social access must stay conservative and single-user. Visits are "read-only," "off by default and revocable," visitors do not create presence or interaction events, and the plan warns against visits expanding into "notifications, public profiles, comparison, or co-presence."

13. Calibration and determinism protect the product feel. The simulation should be deterministic for a given input, the simulation module should be independent from HTTP controllers, and calibration tests guard against drift that is too fast, too slow, inflated by idle tabs, or saturated by repeated offers.

14. Birds should remain distinct without becoming collectibles. Stable IDs, stable per-bird call identity, hidden personality, calm variety constraints, no user-facing rarity, and recognition across drift all support identity without score or rarity framing.

## Per-feature whys

### Product Interpretation And Scope

- **Modern browser web app only:** NOT RECOVERABLE FROM PLAN

- **Email magic-link accounts:** NOT RECOVERABLE FROM PLAN

- **One account, one canonical aviary:** Why: the plan uses this to preserve one canonical server state across devices and avoid multiple aviaries, shared aviaries, or client-owned state.

- **Two starter birds per new aviary:** NOT RECOVERABLE FROM PLAN

- **Bird growth toward a hard cap of seven:** Why: growth is based on aviary age, not visit count, score, or interaction volume; the seven-bird cap is also tied to launch checks where the "seven-bird stress profile remains performant and comprehensible."

- **Server-side simulation tick:** Why: it gives the aviary continuity, makes the simulation service the only writer of canonical state, and supports deterministic calibration.

- **Hidden per-bird personality vectors with slow monotonic drift:** Why: vectors create procedural variation and long-term expressive change while staying private, never becoming visible stats, and never decreasing from neglect.

- **Daily-ish mood state and mood-shaped idle behavior:** Why: mood provides fast-timescale, persistent context that shapes perch choice, pose, call cadence, narration, and notebook prose without labels or badges.

- **Procedural client-side calls, listen-in mixing, and call captions:** Why: procedural calls avoid recorded loops, listen-in makes presence a real interaction, and captions must match the actual generated call for accessibility.

- **Offer interactions for seed, song fragment, and still pool:** Why: offers are intended to feel like "offering a gesture," give small positive signals, and avoid action-selection or punishment framing.

- **Settle gesture:** Why: settle is an "optional session-end affordance" that quiets the aviary but does not make tab close wrong and does not create a recovery surface.

- **Field notebook:** Why: it gives sparse, naturalist, historical observations that notice visible behavior, not raw vector values or event logs.

- **Multi-device sync through one canonical server state:** Why: multiple devices can submit events, but the server aggregates them into one canonical stream so personality and mood do not split or overwrite.

- **Read-only visit invitations, off by default and revocable:** Why: visits allow limited viewing without public discovery, co-presence, visitor events, drift, or comparison surfaces.

- **Screen-reader narration, reduced-motion mode, keyboard access, captions, and WCAG AA copy contrast:** Why: accessibility is part of v1 and must preserve product behavior rather than flattening the aviary into state labels.

- **Aggregate operational telemetry:** Why: telemetry is limited to operational metrics so per-bird state and per-account interaction history do not become product analytics.

- **Account export and soft-then-hard deletion:** Why: account data can be exported and deletion can be recovered for 30 days before hard deletion removes account, birds, vectors, notebook, visits, events, telemetry join keys, and export artifacts.

- **Future requests that conflict with exclusions require explicit re-approval:** Why: this protects the no-game, no-obligation, no-public-network scope from quietly becoming backlog.

### System Architecture

- **TypeScript-first web architecture:** Why: the plan asks for "clear ownership boundaries" and later defaults to TypeScript across client and server for shared domain types.

- **React with TypeScript client baseline:** NOT RECOVERABLE FROM PLAN

- **Browser client:** Why: the client renders the aviary, interpolates, synthesizes audio, captures presence signals, submits events, and displays surfaces without computing canonical personality, mood, drift, adoption timing, notebook entries, or visit authorization.

- **API service:** Why: it owns authentication, account settings, snapshots, event ingestion, notebook reads, visit invites, exports, deletion, and accessibility settings as server-owned surfaces.

- **Simulation service:** Why: it is "the only writer" of canonical aviary state, personality vectors, mood, notebook decisions, and derived bird state.

- **Scheduler/worker system:** Why: it enqueues and runs per-aviary simulation ticks at roughly one-minute cadence and can also handle export jobs.

- **Durable relational database and PostgreSQL:** Why: durable state, transactions, row-level locks, and advisory locks are central for per-aviary tick serialization.

- **Realtime-lite delivery through pull snapshots:** Why: the sync model is "intentionally simple," keeps the server as the only source of canonical state, and allows SSE only for invalidation, "not as a second state source."

- **Aggregate observability pipeline:** Why: observability must provide operational metrics while not reading per-bird simulation records for product analytics.

- **Domain modules rather than generic layers:** Why: the plan wants ownership around auth, accounts, aviaries, birds, events, presence, simulation, notebook, visits, renderer, audio, accessibility, and observability.

- **Simulation independent from HTTP controllers:** Why: the module can accept canonical state, ordered events, and time inputs, then return deltas and observations for deterministic calibration tests.

- **Single renderer abstraction for normal and reduced-motion modes:** Why: normal and reduced-motion renderers should share state interpretation so reduced motion preserves the product rather than becoming static.

### Core Data Model

- **Opaque UUIDs or ULIDs, with account UUID as the only internal identifier:** Why: email must not be used in logs, partitions, messages, or foreign keys, keeping identity synthetic and private.

- **Encrypted email stored only on the account record:** Why: the plan states no service uses email as an identifier and email is encrypted.

- **Soft-deleted accounts recoverable for 30 days, followed by hard deletion:** Why: recovery is possible within the soft-deletion window, then hard deletion removes tied account and aviary data.

- **Per-device session tokens and revocation:** Why: sessions are per-device and revocable from account settings.

- **Magic links that expire after 15 minutes and are invalid after first use:** Why: rate-limiting, abuse controls, and token consumption protect auth without exposing account existence.

- **Aviary with one account, one aviary, local timezone, bird cap, tick timing, and settled visual state:** Why: it supports the v1 one-aviary invariant, local day/night behavior, seven-bird cap, tick scheduling, snapshot versioning, and settle as presentation rather than long-term penalty.

- **Bird species pool with no user-facing rarity:** Why: new birds draw by "calm variety constraints, not collectible rarity."

- **Stable bird identity and renaming invariants:** Why: renaming must not change species, personality, mood, or call identity, and migrations must preserve ID and personality vectors.

- **Persisted personality vector:** Why: hidden numeric traits create distinct birds and drift, but values must not appear in product UI or telemetry.

- **Persisted bird mood:** Why: mood persists across sessions and is advanced by the server tick.

- **Append-only interaction events with idempotency:** Why: clients submit events, never state deltas; duplicates are ignored; the simulation tick consumes events in deterministic server order.

- **Presence windows requiring visibility, focus, and recent pointer/key activity:** Why: the server should validate cadence and cap maximum presence to prevent runaway drift from buggy clients.

- **Offer cooldown per bird and offer class:** Why: cooldown "prevents within-session saturation without framing the user as punished."

- **Field notebook entries stored as rendered prose:** Why: the notebook is a "historical record, not a live reinterpretation."

- **Visit sessions that do not create presence windows or simulation interaction events:** Why: visits are read-only and must not affect drift, notebook, or canonical state.

- **Accessibility settings with matter-of-fact system voice:** Why: settings are system surfaces and should use matter-of-fact voice rather than naturalist product prose.

### API Surface And Sync

- **Versioned JSON endpoints with idempotency keys on mutations:** Why: mutation handling needs deterministic acceptance and safe retries.

- **Magic-link request generic success:** Why: the response is generic "regardless of account existence."

- **Aviary snapshot that excludes personality vector numbers and separates canonical state, render hints, and client-only behavior:** Why: the server owns canonical state while the client interpolates and synthesizes locally.

- **Bootstrap snapshot:** Why: it gives enough state to draw the first bird without waiting on settings, notebook, or account chrome, supporting time-to-first-bird.

- **Interaction event endpoint with ownership, visitor, bird-ID, cooldown, and idempotency validation:** Why: events are accepted as constrained input while protecting account ownership and preventing visitors from mutating the aviary.

- **Offer-state endpoint without countdowns:** Why: the UI should disable or soften controls quietly and avoid exposing cooldown as a game timer.

- **Notebook read API with no user create, update, or delete endpoints:** Why: entries are simulation-created historical prose, not user-authored or edited logs.

- **Visit invite, consume, revoke, and read-only snapshot endpoints:** Why: visits are off unless an invite exists, revocable, and have no interaction affordances.

- **Export, delete, recover, and privacy endpoints:** Why: account settings need export jobs, short-lived links, soft deletion, recovery, and aggregate telemetry categories.

- **Lifecycle snapshot polling:** Why: pulling on navigation, visibility gain, focus regain, reconnect, and wake detection keeps clients aligned with the simple server-source sync model.

- **Conflict prevention with append-only events, locks, monotonic versions, and no last-write-wins personality:** Why: concurrent devices must not overwrite drift or split canonical state.

- **Multiple-device presence cap and dedupe:** Why: overlapping devices should not create "impossible drift."

- **Visitor revocation on next snapshot pull plus server-side token invalidation:** Why: revocation must be enforced for read-only visits.

### Simulation Engine Design

- **Tick cadence and coalesced inactive ticks:** Why: active aviaries tick about once per minute, while inactive catch-up is bounded "to avoid waste."

- **Per-aviary transaction for the tick:** Why: locking, ordered reads, normalized events, drift, mood, call hints, notebook generation, persistence, and snapshot increments keep canonical updates serialized.

- **Drift as a low-pass filter:** Why: no single session should be "visibly meaningful," but regular use should be detected after about a week.

- **Positive drift inputs and monotonic trait effects:** Why: presence-time, listen-in, offers, and age can nudge traits upward, while "Neglect never decreases traits."

- **Drift calibration tests:** Why: tests prove measurable week drift, visible three-week differences, no negative absence movement, no idle-tab drift, and no offer saturation.

- **Mood weighted state machine with hysteresis:** Why: mood should be fast-timescale and contextual but should not "flicker across adjacent ticks."

- **Mood expressed through behavior:** Why: the plan exposes mood through perch preference, scanning, preening, calls, drowsy posture, alert responses, and settled quietness, not labels.

- **Return greeting:** Why: the greeting is the welcome surface; it varies by absence, boldness, social warmth, mood, local time, and greeting history without toast, banner, or absence summary.

- **Ambient weather and day/night:** Why: rare rain, soft wind, and local time phases add small, short-lived mood and call effects without assertive events.

- **Notebook generation through sparse gates and templates:** Why: entries should be rare, notable, naturalist, historical, and based on visible behavior rather than raw vectors or visit frequency.

### Frontend Rendering Pipeline

- **One horizontal scene without panning, scrolling, or zooming:** Why: all birds must stay visible, narrow viewports compress without cropping, and wide viewports add breathing room without becoming an explorable map.

- **Layered scene with a separate top bar:** Why: the aviary can compose sky, foliage, perch zones, foreground branches, and controls while keeping "No UI chrome" inside the scene.

- **First frame and loading without spinner:** Why: the first visible frame should be "already in motion"; a slow connection shows a quiet field with faint motion cues, then birds appear directly in current poses.

- **Idle motion plus reduced-motion rendering:** Why: normal mode expresses preening, scanning, tilts, shifts, and call posture changes; reduced motion replaces motion with slow cross-fades while keeping product behavior.

- **Top bar fade behavior with no badges, counters, or notification dots:** Why: controls stay reachable by keyboard but remain calm and do not introduce gamified or notification surfaces.

- **Bird focus and listen-in by pointer, tap, and keyboard:** Why: users can focus birds and listen in through accessible scene navigation.

- **Offer UI from the top bar with gentle unavailable state:** Why: offering should feel like a gesture, not a game action, and cooldown should not appear as a visible timer.

- **Settle with evening shift, quiet calls, undo, and valid tab close:** Why: settle is optional and reversible, and leaving without settle receives no recovery surface.

### Audio Pipeline

- **Species motif libraries and stable per-bird call identity parameters:** Why: each bird remains recognizable across mood and drift while still producing variation.

- **Runtime procedural call generation with no loops or recorded fallback:** Why: repeated canned audio would break aliveness, so snapshots provide hints or seeds and the client synthesizes calls.

- **Ambient mix and listen-in ramps:** Why: the focused bird comes forward gradually while other birds ramp down but "never to silence," avoiding hard cuts.

- **Captions generated from the same procedural call grammar:** Why: captions must match the actual generated call rather than a generic bird label.

- **WebAudio failure path:** Why: the visual aviary continues, captions turn on by default, and no recorded audio fallback is loaded.

### Accessibility Plan

- **Dedicated screen-reader narration stream:** Why: narration should be naturalist prose, not raw ARIA state dumps, and should avoid personality values, mood labels, and perch numbers.

- **Keyboard and focus behavior:** Why: all controls and scene birds are keyboard reachable, with focus indicators visible at WCAG AA contrast across day/night palettes.

- **Reduced-motion setting and renderer:** Why: it respects `prefers-reduced-motion`, persists user choice, removes drift/flight paths, and preserves simulation, audio, captions, notebook, and visit behavior.

- **Contrast and copy rules:** Why: user-facing text passes WCAG AA while product prose and system surfaces keep their distinct voices.

### Privacy, Performance, Delivery, Copy, Tests, Operations, Security, And Risks

- **Allowed and disallowed telemetry categories:** Why: request, latency, error, render, audio, bundle, and memory metrics are allowed, while per-bird interaction history, vectors, mood history, behavior dashboards, and drift analysis are disallowed.

- **Operational debugging with sampled/redacted logs and audited support views:** Why: debugging uses synthetic account IDs and minimal necessary state to preserve the privacy boundary.

- **Performance budgets:** Why: the product needs fast first-bird render, smooth idle motion, no memory growth, bounded simulation latency, and seven-bird performance.

- **Performance tactics such as lazy-loading and procedural assets:** Why: critical renderer work stays small, account/settings/notebook flows do not block first bird, and recorded audio assets are avoided.

- **Seven rollout milestones:** NOT RECOVERABLE FROM PLAN

- **Bird growth and adoption pacing based on aviary age:** Why: additional birds are "birds that arrived," not rewards earned through visit count, score, or interaction volume.

- **User naming without species catalog choice:** Why: species are selected by the system from the pool and not turned into configurable catalogs or collectible rarity.

- **Adoption notebook entries without celebratory achievement language:** Why: adoption can be observed, but the plan forbids "unlock," "level," "achievement," and "milestone reward" framing.

- **Copy and voice rules:** Why: naturalist product surfaces and matter-of-fact system surfaces protect the bird-led moment and avoid exclamation-heavy celebration, absence framing, trait names, and game words.

- **Static and snapshot guardrail tests:** Why: they catch predictable regressions such as welcome strings, game strings, visible trait labels, visit notification badges, and scene buttons.

- **Operational dashboards and alerts without behavior dashboards:** Why: operations need auth, snapshot, tick, render, audio, memory, export, deletion, and visit authorization health, but must not show listened birds, drift, offer rates, or popularity.

- **Security and abuse controls:** Why: encrypted email, token hashes, rate limits, CSRF where needed, ownership validation, visitor limits, short-lived export links, and audits protect the account while avoiding user-visible friction unless necessary.

- **Drift too fast and drift too slow mitigations:** Why: too-fast drift would feel like stat management, while too-slow drift would make the aviary a screensaver.

- **Presence, audio, sync, accessibility, social, privacy, and performance risk mitigations:** Why: the plan protects drift from idle devices, audio from canned repetition, personality from concurrent overwrites, accessibility from checklist fallback, visits from network behavior, analytics from per-bird leakage, and first-bird load from excess code.

- **Engineering review checklist:** Why: launch must verify no spinner, no return text, hidden personality values, all-three-signal presence, server-only mood/personality writes, render-only visitors, sparse notebook, no gamification, designed reduced motion, procedural captions, no recorded fallback, matter-of-fact system voice, telemetry exclusions, and seven-bird enforcement.

- **Defensible implementation choices for ambiguities:** Why: the defaults are meant to keep the product "conservative, privacy-preserving, and aligned" while leaving visual design specifics to the design system.
