## System-level intent

- **Server-canonical aliveness.** The plan's load-bearing architectural fact is that "the server is the only writer of canonical aviary state," the simulation advances on a "slow server-side tick whether or not a client is connected," and clients are "render-only consumers of snapshots plus append-only event producers." This shows up in Architecture, Simulation engine design, Sync model, and Acceptance criteria as the basis for "sync correctness, drift integrity, multi-device coherence" and the aviary that "feels alive without the viewer."

- **Notice, never announce.** The affective rule is that "the product earns attention by being noticed, never by announcing." The plan turns that into architecture: no `Toast`, `Banner`, `WelcomeBack`, `LevelUp`, or notification-on-scene primitive; the return-greeting is "the entire welcome surface"; no streak, badge, level, score, or "you've been gone X days" surface exists.

- **Relationship over weeks, not rewards in a session.** The personality drift target is "instrument-measurable at ~1 week" and "user-visible at ~3 weeks," with "no single session" moving a trait visibly. This shows up in the slow low-pass drift function, the calibration harness, age-paced bird offers, and the refusal to let visits/interactions/payment accelerate new birds.

- **Hidden personality, expressive behavior.** Personality vectors are "server-persisted, never exposed numerically anywhere" in the product, and users read mood/personality from "motion," "greeting frequency," "comes-to-front frequency," calls, and notebook observations. The plan reinforces this in the two DTOs, the serialization test, the absence of debug/admin/tier exceptions, and the warning that vector loss is "deleting the bird the user knows."

- **No Tamagotchi.** The plan refuses death, hunger, distress, decaying happiness, negative drift, and guilt surfaces. Engine-level monotonic-up drift means neglect leads to "ambient quietness" and "quieter birds to ease back into," never suffering.

- **No gamification by computation or by disguise.** The plan does not merely hide streaks, levels, scores, badges, XP, ranks, "birds adopted: N," visit counters, or green-dot calendars; it says the system "doesn't compute" them. The notebook generator cannot see user-behavior signals, so it cannot write "a streak in disguise."

- **Naturalist voice for aviary surfaces, matter-of-fact voice for system surfaces.** The plan preserves "naturalist, lowercase, present-tense, specific" prose for notebook/narration/captions, while sign-in, account settings, sync errors, accessibility settings, and privacy policy use "matter-of-fact" copy through a distinct `SystemSurface` family.

- **Privacy as an infra rule.** Synthetic UUIDs are the only identifier outside the single encrypted email field; telemetry is "aggregate-only" and "physically separate" from the simulation database; per-bird and per-account interaction state never enters telemetry, analytics, or training. The plan repeatedly implements privacy by removing network, credential, schema, and data-flow paths.

- **Procedural variation as the spell.** The plan treats recorded loops as "the audible signature of dead software." Procedural calls, per-call variation, recognizable signatures, real-time chorus mixing, and "loads with motion already in progress" all serve the same affective intent: the aviary is already alive, not playing canned assets.

- **Accessibility is one of the product renderings.** The plan calls accessibility a "designed surface, not a checklist fallback." Visual, reduced-motion, and screen-reader narration are "three renderings of one state," with captions from the live grammar and reduced-motion as "a calmer Pocket Aviary, not less of one."

## Per-feature whys

### Core relationship engine

- **Hidden per-bird personality vectors** — The plan keeps boldness, social warmth, vocal frequency, plumage saturation, and curiosity hidden because the relationship should surface through expression, not numbers. It warns that "losing a vector = deleting the bird the user knows."

- **Monotonic-toward-expressive drift** — Drift is dominated by presence-time and secondarily interactions so the product changes through quiet attention. It is monotonic-up to implement "no Tamagotchi": a bird that is ignored "doesn't get warier"; it simply has not banked presence.

- **Fast-timescale mood persisted across sessions** — Mood persists so the next session renders "whatever the snapshot says," modulated by ticks in between, rather than snapping birds to a default on tab open.

- **Server-side simulation tick** — The tick runs for every account "whether or not a client is connected" so day/night, mood, weather, drift, perch, calls, and notebook entries belong to an aviary that has been running.

- **Procedural call grammar** — Per-species motif libraries and personality-shaped timing/pitch make each bird recognizable while avoiding recorded loops. The plan says "looped audio is the audible signature of dead software."

- **Bird-to-bird interaction** — Call/response, mood contagion, and chorus make the aviary "a small social system, not independent NPCs."

- **Stable internal bird identity** — Stable `bird.id` survives rename, sync, and future species-pool migration because "no code path regenerates or swaps a bird"; the bird's continuity is part of the relationship.

### Account & sync

- **Single-user accounts, one canonical aviary per account** — NOT RECOVERABLE FROM PLAN

- **Magic-link email auth** — The plan uses single-use, 15-minute links, per-email rate limits, and an always-202 request route to prevent account enumeration and keep sign-in failure surfaces matter-of-fact.

- **Per-device revocable session tokens** — NOT RECOVERABLE FROM PLAN

- **Email-change with new-address verification** — NOT RECOVERABLE FROM PLAN

- **Synthetic UUID as the only identifier** — The rationale is privacy and PII containment: email appears "exactly once, encrypted," and a lint/schema check prevents email from becoming a partition key, log field, telemetry dimension, or inter-service message.

- **Multi-device sync as server-canonical state** — Laptop and phone both render the same canonical record, so there is "no client-to-client sync, no client state to merge, no eventual-consistency reconciliation."

- **Additive, server-authored personality deltas** — Events are processed in `server_ts` order so the morning laptop session and lunchtime phone session both contribute; neither can overwrite the other with "last-write-wins."

- **Account export** — Export is a private data-portability artifact emailed to the account owner. It is the only user-reachable place vector numbers appear, and the plan flags that as off-surface and for privacy review.

- **Soft-delete then hard-delete** — `pending_deletion` accounts can still sign in and recover during the 30-day window; the nightly hard-delete job purges account, aviary, bird, event, notebook, and visit rows after the window.

### Session surface

- **Single horizontal aviary scene with three perch zones** — The plan keeps one place with no pan, scroll, or zoom so birds and perches remain in frame; the user reads a stable aviary, not a navigable map.

- **Loads with motion already in progress** — This is called "the central conceit": the first frame shows birds mid-action, with no entry animation, fade-from-static, spinner, or wake-up sequence.

- **Quiet-field loading state** — A spinner says "machine"; the quiet field reads as "the aviary catching up."

- **Empty-aviary state** — The empty state uses the same quiet field between adoption and first bird; after the first bird "flies in softly," the user never sees an empty aviary again.

- **Local-time day/night cycle** — The account's stored IANA timezone lets the server tick advance day/night without a client connected, so palette, calls, and mood biases continue through absence.

- **Rare ambient weather** — Weather is "rare and non-assertive" so it reads as "the aviary has its own moments," never as a weather feature.

- **Ambient leaf/feather drift** — Leaves and feathers are client-only ornaments with no server state, preserving the render boundary and keeping snapshots in kilobytes.

- **Return-greeting** — Greeting is the whole welcome surface. It is selected server-side because it depends on boldness, mood, and absence length, while the client never sees trait numbers or a "you've been gone X days" value.

- **Listen-in** — Listen-in gives attention to one bird without turning the aviary into "soloable tracks": the focused bus rises, others fall only to ambient, and events become the durable attention signal for drift.

- **Offer** — Seed, song, and still-pool offers create a small interaction whose reaction is shaped by mood and curiosity. Cooldown prevents curiosity from saturating in one session and keeps the affordance quiet.

- **Settle** — Settle is a "soft evening session-end gesture" equivalent to tab-close at the engine level, with small mood-quieting and "no drift direction."

- **Settle 5-second undo** — NOT RECOVERABLE FROM PLAN

- **Field notebook** — The notebook is sparse, read-only, and naturalist. It records "noteworthy aviary transition" moments and explicitly avoids observations of the user, preventing a streak counter in prose.

- **Two system-selected starter birds** — First encounter is "meeting an animal, not configuring an avatar"; the user names the birds after the system selects them.

- **Renameable birds** — Renaming changes `display_name` only; it never touches id, personality, mood, or call, preserving the bird's identity.

- **Third-bird-and-beyond offers paced by aviary age** — The mechanic "deliberately refuses to teach 'more attention earns more stuff.'" The cap of 7 is tied to the recognizability ceiling for call signatures.

### Accessibility

- **Screen-reader running naturalist narration** — Narration is prose in the same voice as the notebook, not "Pip at perch 2, mood content." Idle cadence is slow to avoid flooding the screen-reader queue, with priority for user-initiated events.

- **Reduced-motion mode** — Reduced motion is not "animations off"; it uses cross-fades and slowed shifts so vestibular users get "a calmer Pocket Aviary, not less of one."

- **Call captioning** — Captions come from the same call parameters actually played, so "a soft three-note rise" or similar text matches the live grammar, and captions turn on by default when audio is unavailable.

- **WCAG AA contrast floor** — All user copy must pass AA because chrome, settings, captions, and displayed narration remain readable across bright and dim aviary states.

- **Full keyboard navigation with visible focus** — Keyboard paths cover top-bar items, bird focus, listen-in, offer, settle, and Escape, so the session surface is operable without pointer-only interaction.

- **`SystemSurface` versus naturalist surfaces** — The split prevents matter-of-fact account/error copy and naturalist aviary copy from drifting into each other's register.

### Social

- **Per-invite, opt-in, read-only ambient visits** — Social is "one quiet affordance": sharing is enabled only by an invite, is off by default, and has no global discoverable flag.

- **One-time visit links with expiry and revocation** — The plan uses 30-day unused expiry and immediate revocation at the next visitor pull to preserve host control.

- **Silent visit logging and opt-in notifications** — The visit log lives in settings, with "no badge, no push" unless the host opted in; this preserves notice-never-announce.

- **Visitor sessions never record presence or events** — Visitors cannot drift the host's birds because the visit door has no event-append capability and emits no presence pings.

### Performance & observability

- **Initial JS bundle under 2MB gzipped** — The budget protects first paint on mobile and pushes the design toward code-splitting, compact assets, and synthesized audio instead of recorded files.

- **Time-to-first-bird under 500ms** — The plan treats this as part of aliveness: inline snapshots from the edge and a non-blocking render path get a bird on screen before non-critical assets load.

- **60fps idle motion over a 30-minute session** — Continuous mood-shaped idle motion only works if the render loop remains smooth on a 5-year-old mid-range laptop.

- **No client memory growth over 30 minutes** — Long sessions are normal watching behavior, so pooled sprites, bounded audio graphs, and heap checks are CI-enforced.

- **Aggregate-only telemetry** — Observability may collect counts, latencies, histograms, frame timings, and audio-context errors, but never per-bird or per-account relationship state.

### Architecture

- **Edge / BFF with inline snapshot** — The edge serves HTML plus inline snapshot because that is "critical for time-to-first-bird."

- **Auth service owns the encrypted email field** — This isolates the one PII location and issues the synthetic account UUID.

- **Aviary API service as the only externally reachable door** — It reads snapshots and appends events, but "never writes personality directly," making the API boundary part of the write-ownership rule.

- **Simulation service as sole canonical writer** — It owns personality, mood, perch, and notebook entries because those writes need ordered tick semantics and must never be client-reachable.

- **Social/visit service** — Visits are split out so visitor sessions route through a read-only snapshot path that is gated to "never append events."

- **Notebook generator inside the simulation service** — Entries are a function of canonical state transitions and must share the tick's ordering guarantees.

- **2D canvas with sprite/SVG-pose atlas** — The plan recommends 2D canvas because the scene is "one plane with subtle parallax, not a 3D world," and 2D more predictably hits the 60fps/no-growth budget.

- **Lightweight framework for UI chrome** — Preact or Svelte is recommended to protect the bundle budget, while the aviary scene remains an imperative render loop.

- **Partitioned Postgres event log at v1 scale** — The plan recommends it to keep ordering and retention simple until tick throughput demands a dedicated log.

- **Leased-shard tick scheduler** — Each worker holds a lease on account UUID ranges so ticks are exactly-once per account per tick window.

- **Telemetry pipeline physically separate from simulation DB** — No shared connection or reader means privacy is enforced by missing data-flow paths, not policy language alone.

### Data model

- **`pending_deletion` account state** — Pending accounts can recover during the 30-day window before hard delete cascades.

- **`aviary.created_at` drives bird-offer pacing** — The third bird and later offers are driven by AGE, "never by visit/interaction counts."

- **Stored user IANA timezone** — The tick can advance day/night while no client is connected.

- **Bird `display_name` separate from `id`** — Renaming never touches id, personality, mood, or call, preserving continuity.

- **Traits read-modify-written only by the tick** — Traits are never recomputed from event history or rebuilt by the client; losing the vector is treated as losing the bird the user knows.

- **Separate `BirdCanonical` and `BirdSnapshot` types** — Distinct types enforce that trait floats live only in the simulation service and DB, while the wire snapshot has no trait fields.

- **Serialization test for `BirdSnapshot`** — CI asserts no trait keys appear in snapshot JSON so the "never numeric" rule cannot regress quietly.

- **Append-only event log with `server_ts` ordering** — The event log is the defense against last-write-wins because the tick consumes unprocessed events in authoritative server order.

- **Presence pings under the visible/focused/recent-activity conjunction** — Presence means watching, not a tab left open; this prevents laxer presence from silently inflating drift.

- **Notebook `source_signal`** — Source signals support sparsity dedup and are limited to observations of the aviary, never observations of user behavior.

- **Visitor email encrypted in visit log for host display** — The plan keeps visitor email encrypted while still allowing the host-facing visit log to show who visited.

- **Species pool of about six content-defined entries** — NOT RECOVERABLE FROM PLAN

- **Species rarity not modeled** — NOT RECOVERABLE FROM PLAN

- **Nightjar-like species active at night** — Night is "not dead": most birds settle, but a nightjar-like signature can stay active and calling.

### API surface

- **Snapshot pull triggers** — Pulling on visible, long render-frame gap, and 30-second visible keepalive keeps the client current after resume while avoiding a required realtime socket.

- **Batched event write with idempotency key** — Batching keeps presence pings cheap; idempotency dedupes replay or double-send without returning personality data.

- **Visitor snapshot same as host sees** — There is "no special/prettified rendering," preventing visits from becoming show-off rendering.

### Simulation engine design

- **Idempotent-safe tick retry** — State writes and event-mark-processed commit in one transaction, so a crashed retry re-reads the same unprocessed events and avoids double application.

- **Drift `max(0, delta)` in the function** — Monotonic-up is enforced at the engine level, not as a downstream guard, so neglect cannot decrease traits.

- **Drift calibration harness** — The harness is "a real test, not a guideline"; it pins gain, tick interval, and presence-window constants to measurable and visible drift behavior.

### Sync model

- **Ignoring stale snapshots** — Monotonic `version` lets the client keep the highest version and avoid visual rubber-banding.

- **Stopping client rendering while hidden** — The client saves battery when hidden while the server keeps ticking; on return, the user sees the aviary that continued running.

### Audio pipeline

- **WebAudio fallback to graceful silence plus captions** — Silence-with-captions is preferred because "there is no recorded-audio fallback" and canned audio would violate the procedural rule.

- **Audio resource management** — Pooled buffers/nodes, a bounded `AudioContext`, and reused worklets enforce the no-memory-growth rule.

### Frontend rendering pipeline

- **No UI chrome inside the scene** — The scene remains "birds and place only"; chrome lives in a thin top bar that fades near-transparent during stillness.

- **Mood-shaped idle motion with no labels** — The user reads mood from motion; "no label, tooltip, or status icon" tells them what a bird feels.

- **Unsupported-browser surface instead of old compatibility paths** — Very old browsers get a matter-of-fact surface because compatibility paths would add unjustified bundle bloat.

### Affective-constraint enforcement

- **No announcement UI primitive** — The primitive to build toasts, banners, welcome overlays, level-ups, or scene notifications is absent, and lint flags new always-on scene overlays.

- **No gamification data exists to surface** — The system does not compute visit counters, streaks, levels, XP, badges, ranks, or similar fields, so there is nothing to expose later.

- **No social-network or leaderboard substrate** — Cross-account metrics are not computed, and surfaces like co-presence, chat, avatars, comments, discovery, leaderboards, and show-off rendering are excluded.

### Observability & privacy boundary

- **Simulation-tick latency p99 alarm** — The alarm fires before users feel "running slow"; tick work should be far below the 5-second p99 budget.

- **Synthetic performance fleet** — Automated browsers from common geographies catch first-bird and frame-timing regressions in the field.

- **Plain-text privacy policy in account settings** — The policy names aggregate categories and explicitly excludes per-bird interaction state in matter-of-fact voice.
