## System-level intent

1. Make the aviary feel "continuously alive, specific, and private across weeks of use."
   - This appears in "Intent and planning stance" as the product goal and is repeated in the "Execution summary," where success depends on whether birds feel "continuous, specific, and gentle to return to."
   - It also drives the "first-bird presence over UI richness" priority in the execution summary.

2. The product is intentionally not about "feature breadth."
   - The opening stance says the goal is "not feature breadth."
   - The scope excludes native clients, gamification, Tamagotchi mechanics, social-network surfaces, production debug panels, recorded audio fallback paths, and heavy live-sync infrastructure.
   - The execution summary calls the plan "intentionally biased toward preserving that feeling even when it costs extra implementation discipline."

3. The aviary must feel alive before the user thinks about the app.
   - The plan names this directly: "the aviary must feel in motion before the user thinks about the app."
   - It shows up again in "Shell and first paint": hit the "already alive" feel, embed the latest bootstrap snapshot, render a "quiet field fallback" only when needed, "Never show a spinner," and hydrate so "the first bird can render before non-critical chrome loads."

4. Presence should matter but must not become obligation.
   - The plan states "presence must matter without becoming obligation."
   - This shows up in the "expressive energy" bridge between "no punishment" and "quiet after absence," and in tests for "no negative trait drift."
   - It also shows up in exclusions of "streaks, achievements, levels, counters, badges, green-dot calendars" and Tamagotchi-style "hunger, distress, decaying happiness, death, punishment for absence."

5. Canonical bird identity and drift belong to the server.
   - The opening constraints say "the server must own canonical bird identity and drift."
   - The "Client/server split" assigns stable bird identity, persistent bird state, personality vectors, mood state, expressive energy, weather, notebook generation, visit entitlement, and snapshot versions to the server.
   - "Render boundary" says the client "must never invent canonical bird behavior."
   - "Sync, consistency, and conflict handling" says the server is "the only writer of personality, mood, expressive energy, and notebook state."

6. Accessibility and privacy must be first-class product work, not follow-on fixes.
   - The opening constraints say "accessibility and privacy must ship as first-class parts of the product rather than follow-on fixes."
   - Accessibility is in v1 scope and has dedicated sections for narration, keyboard navigation, visual accessibility, audio accessibility, and motion accessibility.
   - Privacy is built into "Privacy boundaries," "Identifier policy," telemetry exclusions, export/delete flows, and the requirement that visit records "exist for host transparency only and must not flow into cross-account analytics."

7. Keep architecture conservative because "the product risk is in simulation feel, not infrastructure novelty."
   - The stack recommendation is "intentionally conservative."
   - The plan chooses polling over WebSockets, a single deployable backend plus workers, a Postgres-backed durable job queue, and no separate Kafka dependency.
   - Delivery workstreams land early in integration because "the core risks are cross-cutting."

8. Deterministic, sparse, safe generation is preferred over freeform or noisy systems.
   - The plan chooses "rule-based notebook and narration generation" using "curated templates, grammar rules, and event prioritization" rather than a freeform LLM path.
   - The why is to keep the voice "specific, sparse, safe, and consistent."
   - The same principle appears in notebook sparsity rules, dedupe keys, captions generated from the same call grammar, and narration dedupe.

9. Implementation should preserve coherence across devices and visitor mode.
   - "Render boundary" says server-authored snapshots let the client interpolate, animate, and synthesize while preserving "coherence across devices and in visitor mode."
   - "Sync, consistency, and conflict handling" rejects last-write-wins failures, client-authored absolute values, and stale clients pushing corrected canonical state.
   - Return-greeting generation is deterministic from snapshot seed and absence bucket so host and visitor render the same underlying state if they load simultaneously.

10. The product voice is naturalist, concrete, matter-of-fact on failures, and never based on hidden numbers.
   - Notebook entries must be "lowercase," "present tense," "bird-specific," with "no system jargon," "no mention of hidden numeric state," and no visit-streak observations.
   - Screen-reader narration must "stay in naturalist voice," "reference concrete birds and positions," and "avoid numeric state labels."
   - Failure surfaces use "matter-of-fact" copy, with "No naturalist copy on these surfaces."

11. User relationship should not become a KPI artifact.
   - Day-one instrumentation is "sufficient to keep the product healthy without measuring the user relationship as a KPI artifact."
   - Observability explicitly avoids account-comparable engagement metrics, population-level "most visited aviary" metrics, notebook text, and per-bird trait values in analytics.

## Per-feature whys

### 1. Intent and planning stance

- Implementation-ready v1 for a small engineering team: the plan makes explicit calls "so execution does not stall."
- Four cross-cutting constraints: they serve the goal of a browser-based aviary that feels "continuously alive, specific, and private across weeks of use."

### 2. Scope

- Modern-browser web app only with responsive desktop and mobile layouts: NOT RECOVERABLE FROM PLAN
- Email magic-link authentication and one aviary per account: supports account identity, sessions, and the one-aviary ownership model, but no more specific why is articulated.
- Two starter birds at account creation: gives every account an initial aviary; alpha keeps bird count fixed at two for calibration.
- Age-based expansion up to seven birds: private beta enables the unlock scheduler behind remote config, with rollback levers for "new bird unlocks"; no deeper product rationale is articulated.
- Canonical server-side simulation tick: advances mood, presence-derived expressivity, weather, notebook generation, and personality drift under the server-owned canonical model.
- Passive watching: makes "user is quietly watching" count without requiring frequent motion and supports the product goal of presence without obligation.
- Listen-in: contributes to expressive energy and social warmth/vocal-frequency drift; its mix should "rebalance, not solo" so the focused bird is emphasized without muting the aviary.
- Offer: drives curiosity only when approached or accepted, supports offer reactions and notebook triggers, and is constrained by server-side cooldowns to keep the client truthful.
- Settle: affects mood only, creates a reversible quieting interaction, and is treated as "mood-quieting plus presence end" rather than long-term drift.
- Persistent field notebook: preserves sparse auto-generated observations that are bird-specific and never become one entry per session.
- Multi-device sync: uses authoritative server snapshots and append-only events to avoid stale clients and last-write-wins failures.
- Quiet, opt-in visit invitations: provide read-only visitor sessions while keeping the surface "quiet" and preventing visitors from affecting host settings, notebook, listen-in, offers, or settle.
- Screen-reader narration: makes accessibility a first-class surface and is sourced from canonical snapshot state rather than ad hoc ARIA labels.
- Reduced-motion mode: respects motion accessibility while keeping mood and drift logic intact.
- Keyboard navigation: ensures top-bar items, birds, listen-in, panels, and overlays are operable without pointer input.
- Call captions: keep audio accessibility first-class; when WebAudio fails, captions turn on by default.
- Contrast-compliant chrome: supports WCAG AA text and focus treatments across bright morning and dim evening palettes.
- Operational telemetry: keeps the product healthy while limiting data to aggregate counts, latencies, error classes, duration histograms, and performance timings.
- Performance instrumentation: protects the "already alive" feel through first-bird render timing, frame metrics, bundle gates, and synthetic checks.
- Account export: provides account integrity and user-owned export packages containing current bird state, notebook entries, settings, and visit metadata.
- Account deletion: hides the aviary immediately on soft delete and erases account-bound records after 30 days.
- Session revocation: makes session revocation immediate for future requests.
- Native iOS or Android clients: NOT RECOVERABLE FROM PLAN
- Gamification exclusions: support "presence must matter without becoming obligation" and avoid measuring the relationship as a KPI artifact.
- Tamagotchi mechanics exclusions: support "no punishment," no negative trait drift, and "quiet after absence" without decaying happiness, death, or punishment.
- Social-network surface exclusions: protect privacy and avoid profiles, feeds, discovery, comments, chat, co-presence, and leaderboards.
- User-facing numerical personality stats or production debug panels: prevent hidden numeric state from becoming user-facing product language.
- Recorded audio fallback paths: preserve the plan's audio direction and avoid a "recorded fallback compromise."
- Heavy live-sync infrastructure: consistent with polling, snapshots, and the simpler v1 architecture.

### 3. Decision calls and assumptions

- Server-owned "expressive energy" layer: reconciles upward-only personality drift with birds becoming quieter after absence; it shapes greeting frequency, call density, and approach behavior without lowering permanent traits.
- Rule-based notebook and narration generation: keeps the voice "specific, sparse, safe, and consistent."
- Polling over WebSockets: is "simpler, cheaper," and matches the rule that "clients pull snapshots and interpolate."
- Single deployable backend plus workers: keeps architecture "understandable" while preserving clean module boundaries.
- Visitor surface is render-only: lets visitors see the same ambient scene and audio state without host settings, notebook access, listen-in, offers, or settle controls.
- Remote-config calibration: lets presence window length, drift gains, notebook sparsity thresholds, and age-based bird unlock timing change without code deploys.

### 4. System architecture

- Web application with edge-served shell and embedded bootstrap snapshot: supports fast initial HTML delivery and first render.
- React/TypeScript client for scene rendering, input, accessibility, and WebAudio: NOT RECOVERABLE FROM PLAN
- Responsive scene layer, top-bar UI, account/settings pages, and visitor pages: NOT RECOVERABLE FROM PLAN
- Application API: centralizes magic links, snapshot reads, event ingestion, notebook reads, settings, export/delete, invites, visit logs, validation, revocation, and access control.
- Simulation worker: owns the per-aviary tick and canonical mutation of mood, expressive energy, monotonic drift, weather, calls, notebook entries, and snapshots.
- Background jobs: handle email delivery, soft/hard deletion, synthetic checks, and alerting outside the request path.
- React + TypeScript with SSR-capable framework support: chosen for "fast initial HTML delivery."
- Node.js + TypeScript API/worker runtime: NOT RECOVERABLE FROM PLAN
- PostgreSQL primary database: NOT RECOVERABLE FROM PLAN
- Postgres-backed durable job queue: avoids a separate Kafka dependency for v1.
- Short-lived cache for rate limits, ETags, and session lookups: supports magic-link rate limiting, snapshot cache, and session lookup efficiency.
- Transactional email provider: sends tokenized magic links and invite links.
- Small SVG/compact bitmap species art plus procedural audio motif definitions: supports lightweight species rendering and procedural call generation.
- Server-owned account/session/bird/simulation/notebook/visit/snapshot state: preserves canonical identity, drift, and coherence.
- Client-owned interpolation, ornaments, WebAudio synthesis, presentation state, local accessibility preferences, and event capture: keeps client work presentation-only and avoids inventing canonical bird behavior.
- Snapshot render boundary: preserves coherence across devices and visitor mode by sending bird order, pose, mood-derived flags, weather, call seeds, notebook metadata, version, and server time.

### 5. Data model

- `accounts`: supports synthetic identity, encrypted email, lookup hash, timezone/locale, deletion lifecycle, and settings; raw email must not be used outside the auth/account boundary.
- `device_sessions`: supports device listing, last-seen display, revocation, and coarse optional region.
- `aviaries`: stores one account's canonical aviary version, tick timing, presence end, day/weather, settle state, and next unlock.
- `birds`: stores stable forever bird identity plus traits, mood, expressive energy, perch/pose/call state, offer cooldown, and greeting history.
- `interaction_events`: append-only input for presence, listen-in, offers, settle, visibility resume, idempotency, and worker processing.
- `presence_windows`: gives the worker "clean, auditable presence inputs without depending on raw DOM events."
- `aviary_snapshots`: optimized read model served to host and visitor clients.
- `notebook_entries`: stores sparse observations with type, body text, and dedupe key.
- `visit_invites`: supports one-time, expiring, revocable visitor entitlements with encrypted-plus-hash invitee email.
- `visit_sessions`: exists for host transparency only and must not flow into cross-account analytics.
- `account_export_jobs`: supports asynchronous export jobs and expiring download tokens.

### 6. API surface

- `POST /api/auth/magic-link/request`: issues one-time tokens with per-email rate limits and generic success, supporting magic-link auth and abuse resistance.
- `GET /auth/magic-link/consume`: validates one-time token, creates session, invalidates token, and redirects into the app.
- `GET /api/account/sessions`: lets users list current device sessions.
- `POST /api/account/sessions/:sessionId/revoke`: lets users revoke a device session; session revocation is immediate for future requests.
- `POST /api/account/export`: creates an export job and emails a download link for account export.
- `POST /api/account/delete`: starts the soft-delete window.
- `POST /api/account/delete/cancel`: restores the account inside the 30-day window.
- `GET /api/aviary/bootstrap`: returns the initial snapshot and settings for first render and can be embedded into SSR HTML.
- `GET /api/aviary/snapshot?sinceVersion=<n>`: returns authoritative snapshots and uses ETag/version to avoid unnecessary payloads.
- `POST /api/aviary/events/batch`: accepts ordered idempotent interaction events, validates access/cooldowns/birds, and returns acknowledgements plus canonical timing hints.
- `GET /api/aviary/notebook?cursor=<opaque>`: serves notebook entries in reverse chronological paged order.
- `POST /api/visits/invites`: creates a quiet visit invite, sends a one-time link, and defaults to 30-day expiry.
- `GET /visit/<token>/bootstrap`: validates the token and returns a read-only snapshot plus host display metadata.
- `GET /api/visits/log`: gives the host invitation states and past visits for transparency.
- `POST /api/visits/invites/:inviteId/revoke`: revokes an invite immediately.
- Opaque IDs on mutating APIs: prevents trusting client-computed state.
- Idempotent event ingestion: tolerates retries.
- Version/ETag snapshot caching: makes snapshot APIs cacheable but "never shared cross-user."
- Matter-of-fact system errors: map errors to explicit UI states without naturalist copy.

### 7. Presence model

- Dedicated presence module with exhaustive tests: presence is "the most contamination-sensitive logic in the product."
- Client-side qualification with visible document, focused window, and recent activity: satisfies the PRD's "exact conjunction rule."
- 180-second activity window in remote config: gives a starting calibration point that can be tuned after observing data.
- 60-second presence ping cadence: NOT RECOVERABLE FROM PLAN
- Immediate `visibility_resumed`: lets the system react when the tab becomes visible after hidden state or a long frame gap.
- Stop emitting when qualification fails: enforces the presence rule.
- Server-side coalescing into contiguous windows: avoids storing noisy DOM-level movement data and gives the tick compact drift inputs.
- Max 75-second ping gap, truncation on visibility/focus loss, future-skew rejection: supports clean, auditable presence windows.
- Quiet watching counts without frequent motion: supports presence without obligation.

### 8. Simulation engine design

- Minute tick cadence with global scheduler: NOT RECOVERABLE FROM PLAN
- Per-aviary advisory lock: ensures only one worker mutates an aviary at a time.
- Transactional tick writes: ensure the new state version commits in full or nothing does.
- Processing loop order: loads state/events/windows, computes time phase, updates expressive energy, mood, drift, perch/pose/calls, weather/settle, notebook entries, and snapshot projection so canonical state advances coherently.
- Expressive energy from recent qualified presence, listen-in, and successful offers: bridges "no punishment" and "quiet after absence."
- Expressive energy decay to low ambient baseline: birds become quieter after inactivity without personality loss.
- Expressive energy floor above zero: birds "never become inert."
- Expressive energy controls greeting probability, call density, front-perch approach, and prompt offer reaction: lets presence affect expressivity without changing personality values.
- Personality traits in `[0,1]`: NOT RECOVERABLE FROM PLAN
- Daily capped additive positive drift: makes drift measurable after about one week and perceptible after about three weeks without becoming gameable in one session.
- Weekly trait movement cap: keeps visible change gradual.
- Boldness drift: presence near the front and offers while near the viewer explain increased boldness.
- Social warmth drift: repeated return greetings and listen-in engagement explain increased social warmth.
- Vocal frequency drift: sustained presence and listen-in explain more calls and chorus joining.
- Plumage saturation drift: sustained presence over longer windows explains saturation changes.
- Curiosity drift: approached or accepted offers, not merely offered items, explain curiosity.
- 70/20/10 drift weight split: NOT RECOVERABLE FROM PLAN
- Settle excluded from long-term drift: settle should influence mood only.
- Mood state machine: keeps mood explicit and persistent across sessions.
- Time-of-day mood biases: early morning tends alert; dusk/night tends drowsy.
- Rain mood effects: temporarily dampens vocal behavior and nudges mood depending on personality.
- Nearby alarm-like calls spreading wary: explains same-tick bird signal influence.
- High boldness dampening wary: ties personality to mood transitions.
- Return-greeting logic separate from generic animation: greeting needs its own path.
- Absence buckets: let greetings vary for short, medium, and long absences.
- Greeter scoring by boldness, social warmth, expressive energy, mood, and cooldown: selects a primary greeter in a grounded way.
- Action tuples and staggered secondary reactions: produce specific return greetings.
- Deterministic greeting path from snapshot seed plus absence bucket: lets host and visitor render the same underlying state when loading simultaneously.
- Worker schedules call intent rather than synthesizing audio: keeps audio synthesis in the client while the server owns canonical timing and seeds.
- Per-bird call motif, timing, pitch, intensity, and chorus eligibility in snapshot horizon: gives the client enough instruction to produce coherent sound and captions.
- Seeded weather stream per aviary: makes weather feel specific while fully server-authored.
- Rare rain, soft wind, and no severe weather in v1: supports gentle weather effects without severe events; exact targets have no deeper rationale.
- Rule-engine notebook generation: uses curated templates and dedupe keys for sparse, specific observations.
- Notebook triggers for unusual greeters, long absence, quiet mornings, rest/preen, weather, and offer interest: make entries noteworthy rather than session-based.
- Notebook sparsity caps: prevent "one entry per session" and keep the field notebook sparse.
- Notebook language requirements: preserve naturalist voice and hide system jargon, numeric state, and attendance patterns.

### 9. Sync, consistency, and conflict handling

- Server-only writes for personality, mood, expressive energy, and notebook: protects canonical model.
- Clients write interaction events only: prevents stale clients from correcting state.
- Monotonically increasing snapshot versions: support consistency and cache/version checks.
- Session-scoped idempotency keys: deduplicate retries.
- Worker ordering by `server_received_at` and stable insertion order: avoids relying on raw client time alone.
- Fresh snapshot on visible client: prevents resumed clients from staying stale.
- 30-second polling when active: NOT RECOVERABLE FROM PLAN
- Poll after long frame gaps or device resume: refreshes stale clients.
- No client endpoint accepting absolute personality values: prevents last-write-wins failures.
- State-version check on snapshot write: maintains snapshot invariants.
- Separate event ingestion and snapshot reads: keeps events valid from older snapshots subject to current server validation.
- Server-side offer cooldown validation: keeps cooldown correctness canonical.
- Accepting too-early offer envelope but marking event rejected: lets the client remain truthful with canonical cooldown expiry.
- Explicit failure surfaces for expired links, timed-out sessions, snapshot load failure, expired/revoked visits, and unsupported browser: keeps failures matter-of-fact and mapped to UI states.

### 10. Frontend rendering pipeline

- Authenticated HTML shell with inline bootstrap snapshot: supports the "already alive" feel.
- Quiet field fallback: avoids a spinner when no snapshot is available in time.
- No spinner: supports first impression of life rather than app waiting.
- Progressive hydration with first bird before non-critical chrome: prioritizes first-bird presence over UI richness.
- Scene layer order: NOT RECOVERABLE FROM PLAN
- Three logical perch zones with scale/parallax keyed to depth: supports front/middle/back bird positioning.
- Lightweight species art plus pose-state transitions: keeps rendering compact and avoids frame-heavy sprite sheets.
- Idle micro-motion through interpolation: keeps motion continuous while visible.
- Snapshot-driven pose selection: keeps canonical behavior server-authored.
- Reduced-motion cross-fades and no drift overlays: reduces motion while preserving mood and drift logic.
- Top bar limited to account/settings, accessibility, notebook, offer, and settle: keeps chrome focused.
- Top bar fade and return on activity: supports an ambient scene while staying reachable and focusable.
- Bird focus toggles listen-in: direct bird interaction enters/exits focused listening.
- Offer from top bar targeting chosen bird in panel flow: avoids direct bird-click offering and keeps offer flow explicit.
- Settle top-bar trigger with 5-second reversal: makes settle visually reversible before mood quieting plus presence end.
- Visitor read-only shell reusing scene renderer: shares the scene while removing offer, settle, listen-in, and host-affecting writes.
- Local-only visitor accessibility controls: lets visitors adjust their rendering without changing host state.
- Desktop/mobile responsive spacing with all birds in frame: keeps birds visible and interaction viable across screens.
- Tap targets and focus order on small screens: preserves mobile accessibility and usability.

### 11. Audio pipeline

- WebAudio master graph with per-bird, ambient/chorus, listen-in, and master buses: supports procedural calls, listen-in rebalancing, mute, and gain control.
- Species motif library: gives each species envelope, oscillator/noise, rhythm, pitch, and ornament definitions.
- Per-bird call identity from species motif, stable seed, mood, vocal frequency, and expressive energy: keeps birds recognizable and state-responsive.
- Schedule 3 to 5 seconds ahead: NOT RECOVERABLE FROM PLAN
- Refresh schedule on authoritative snapshot: keeps audio aligned with canonical state.
- Blend schedule updates: avoids hard-resetting active calls.
- Listen-in rebalancing: focuses one bird while never muting other birds completely, preserving ambient balance.
- Captions from the same call grammar tokens as audio: ensures captions describe what actually played.
- Silent WebAudio fallback with captions enabled: preserves visual and notebook behavior while treating muted/silent audio as acceptable.
- Audio errors only through aggregate operational telemetry: avoids user-facing audio failure noise.

### 12. Accessibility surfaces

- Dedicated narration channel from canonical snapshot state: avoids ad hoc ARIA labels and keeps narration aligned with real scene state.
- Hidden live region with queued prose updates: implements screen-reader narration.
- Idle narration every 30 to 60 seconds: NOT RECOVERABLE FROM PLAN
- Prioritized narration on greeting, offer reaction, and settle: surfaces meaningful scene changes.
- Narration dedupe: avoids repeating near-identical observations.
- Naturalist narration with concrete birds and positions: matches product voice and avoids numeric/chatty phrasing.
- Keyboard flow through top bar, scene, birds, listen-in, and overlays: makes the app fully keyboard navigable.
- WCAG AA contrast and visible focus ring across palettes: preserves visual accessibility in bright and dim scene states.
- Captions avoiding bird bodies: keeps captions readable without covering the main subject.
- Optional captions easy to enable: supports audio accessibility.
- Silence fallback defaults captions on: makes no-audio mode first-class.
- Muted audio first-class mode: "not an error condition."
- Respect `prefers-reduced-motion` on first load: honors system-level motion preferences.
- Explicit reduced-motion override persisted per account: lets users choose and retain motion accessibility behavior.

### 13. Privacy, security, and account integrity

- Per-bird interaction history used only for that account's simulation: preserves privacy boundaries.
- Analytics excludes bird traits, notebook text, and detailed interaction events beyond operational necessity: prevents privacy leaks through logging or analytics convenience.
- Aggregate telemetry limited to counts, latencies, errors, histograms, and timings: keeps operational visibility without simulation detail.
- Synthetic `account_id` for joins: enforces identifier discipline.
- Raw email confined to auth/account tables: prevents raw email from becoming a general identifier.
- Invitee encrypted-plus-hash treatment: applies the same email privacy treatment to visitors.
- 15-minute magic link expiry: limits auth token lifetime.
- Immediate invalidation of used links: prevents reuse.
- Per-email and IP abuse controls: protects magic-link issuance.
- Immediate session revocation: protects account integrity after revocation.
- Soft delete hides normal use immediately: gives prompt deletion effect.
- Hard delete after 30 days: erases account-bound records after the soft-delete window.
- Export packages include bird state, notebook, settings, and user-owned visit metadata: gives users their account data.

### 14. Performance budgets and observability

- Initial JS bundle under 2MB gzipped: supports fast first render.
- First bird visible under 500ms on mid-tier mobile over 4G: protects the "already alive" feel.
- Idle motion at 60fps on older laptop hardware: keeps continuous visible motion smooth.
- No measurable memory growth over 30 minutes: supports long passive watching sessions.
- Simulation tick p99 under 5 seconds: keeps canonical state timely.
- Minimum inline bootstrap state: supports first render without excess payload.
- Code-split settings, account pages, and visit management: avoids letting non-core flows bloat initial load.
- Compact species art and reused pose assets: supports asset budget and renderer performance.
- Reused audio nodes/buffers: avoids per-call allocation churn.
- Suspend rendering when hidden but not server simulation: saves client work without stopping canonical world state.
- Aggregate observability collection: monitors health and performance without per-account simulation detail.
- Telemetry exclusions: prevent account-comparable engagement, notebook/body text, and per-bird traits from entering analytics.
- Synthetic browser probes: validate first-bird SLA, audio initialization, snapshot freshness, and degraded-state copy.

### 15. Delivery workstreams

- Platform and auth workstream: covers account schema, synthetic IDs, magic links, sessions, settings shell, delete/export primitives.
- Canonical state and simulation workstream: covers bird schema, events, presence, tick worker, mood/drift, snapshots, notebook generator.
- Scene renderer and interaction UX workstream: covers responsive scene, top bar, focus/listen-in, offers, settle, notebook UI, loading state.
- Audio and accessibility workstream: covers motif library, WebAudio, captions, narration, reduced motion, keyboard semantics.
- Social visits and operational tooling workstream: covers invite lifecycle, visitor page, logs, rate limiting, synthetic monitoring, telemetry dashboards, privacy review.
- Feature flags and early integration: needed because "the core risks are cross-cutting."

### 16. Testing strategy

- Fixed-seed drift fixtures over 1 day, 1 week, and 3 weeks: test gradual measurable/perceptible drift.
- Absence scenarios proving no negative trait drift: enforce no punishment.
- Expressive-energy decay tests: prove birds become quieter without personality loss.
- Mood transition snapshots: verify time-of-day and weather combinations.
- Notebook sparsity tests: enforce sparse field notebook behavior.
- Idempotent event ingestion tests: verify retry tolerance.
- Cooldown enforcement tests: verify server-side offer cooldown correctness.
- Invite revoke/expire tests: verify visitor entitlement lifecycle.
- Session revocation tests: verify immediate future-request revocation.
- Deletion/export tests: verify account integrity flows.
- Account creation and two-bird adoption E2E: verify first-use core.
- Return after absence bucket E2E: verify short/medium/long greeting behavior.
- Multi-device stale resume E2E: verify snapshot and stale-client behavior.
- Visitor revoked-link E2E: verify visit failure surface.
- Reduced-motion and keyboard-only E2E: verify accessibility paths.
- Screen-reader walkthroughs: verify narration in real assistive technology.
- Caption readability across palettes: verify captions stay readable over scenes.
- Focus-order audits: verify keyboard navigation.
- Contrast snapshots: verify WCAG AA across morning/evening/night.
- Bundle gate, first-bird benchmark, 30-minute soak, and 60fps benchmark: verify performance budgets.

### 17. Rollout plan

- Internal calibration: tune presence window, expressive-energy decay, and first-bird render path using synthetic accounts before exposing social visits.
- Closed alpha: focuses on drift calibration, greeting quality, notebook sparsity, audio recognizability, and accessibility breakpoints.
- Fixed two-bird alpha: keeps bird count stable for alpha calibration.
- Private beta age-based unlock scheduler: tests unlock timing behind remote config.
- Initial unlock proposal at 8, 16, 28, 40, and 52 weeks: NOT RECOVERABLE FROM PLAN
- Quiet visit invites for subset of accounts: lets visits be tested gradually.
- Public v1 with two starter birds and unlock schedule by cohort: launches core while controlling unlock rollout.
- Visits off by default at public v1: keeps visit exposure controlled.
- Rollback levers for invites, unlocks, notebook, audio experiments, and presence calibration: preserve control over high-risk cross-cutting systems.
- Day-one instrumentation: monitors product health without measuring the user relationship as a KPI artifact.

### 18. Major risks and mitigations

- Drift too fast or too slow: fixed-seed simulations, remote-config gains, weekly beta calibration, and "visible at ~3 weeks" criteria mitigate calibration risk.
- Birds feel static after absence under no-negative-drift rule: expressive energy, "ambient but alive" tests, and long-absence design review mitigate static feeling.
- Audio sounds bad or birds lose recognizability: tiny species pool, signature seeds, device listening reviews, grammar-sourced captions, and no recorded fallback mitigate audio risk.
- Snapshot/polling visible jumps: short behavior horizon, interpolation, visibility-resume refresh, and stale-state E2E mitigate jumps.
- Multi-device ordering corrupts state: append-only events, locks, server-received ordering, no client absolute values, and version invariants mitigate corruption.
- Accessibility becomes second-class: core renderer inclusion, QA gates, and public-launch blocking criteria mitigate accessibility risk.
- Performance budgets missed: week-one bundle gate, code-splitting, species asset tracking, and CI first-bird checks mitigate performance risk.
- Privacy leaks through logs or analytics convenience: identifier discipline, analytics allowlist review, account-ID-only logs, no notebook/body telemetry, and beta privacy review mitigate privacy risk.

### 19. Execution summary

- Server-authoritative web product: fastest credible v1 because it keeps a small, carefully tested simulation core canonical.
- Lightweight renderer/audio client: prioritizes first-bird presence over UI richness.
- Extra implementation discipline: protects the feeling of birds that are continuous, specific, and gentle to return to.
