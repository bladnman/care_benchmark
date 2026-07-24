## System-level intent

- Non-gamified, non-Tamagotchi relationship design. This shows up in the explicit non-goals against "streaks, badges, levels, XP", "hunger, death, distress, decay meters", the instruction to "do not plan, scaffold, or leave hooks", the risk row "Gamification PR creep | Concept poison", and the observability line "Instrument health, not exploitation."

- Server-canonical continuity instead of client-owned simulation. This appears in "Multi-device sync via server-canonical state", the invariant that "Only the tick worker mutates `personality_vector`", the hard ban on "client vector writes", and the risk that sync bugs or dual simulation cause "Lost relationship history."

- Monotonic expressive drift, with quietness as behavior rather than punishment. The plan says "positive presence/interaction may increase traits toward 1.0; neglect does not decrease traits" and later clarifies that a return after two weeks gives "quieter greets, not greyer plumage down-drift."

- "Already in motion" as the first impression. This intent is carried by the first paint protocol, "Show quiet field immediately", "Draw birds at pose.phase", "no entry animation", and the forbidden "logo splash, gamified loader, 'Welcome', confetti."

- Quiet, bounded social presence. The visit feature is "Optional quiet visits", "read-only", "no co-presence", "OFF by default", and must not become "Social network"; the plan rejects "chat, avatars, comments, discovery, leaderboards, show-off render path."

- Accessibility as designed product surface. The scope calls "Accessibility as designed surfaces", the rollout says "a11y not deferred", reduced motion is "equal product quality, not muted broken checkbox", and the risk table warns that "A11y as checklist" means "SR users get inferior product."

- Privacy-bounded data and telemetry. This appears in "privacy-bounded telemetry", "email never a FK or log key", "Synthetic UUID everywhere", "simulation DB not CDC'd into warehouse", and the telemetry rule to emit only counters/histograms.

- Performance and battery as part of the experience. The plan gives hard budgets for bundle, first-bird paint, 60fps, and memory; it says rendering must pause when hidden "to save battery"; and the risk row says bundle/TTFB slip means "Loading destroys 'already in motion'."

- Naturalist voice with matter-of-fact system boundaries. The plan uses "sparse naturalist entries", "naturalist lowercase", "never 'mood: content'", "no user streak language", and "Matter-of-fact error bodies for auth/system; no naturalist prose on errors."

## Per-feature whys

### 1. Scope

- Browser-only SPA: NOT RECOVERABLE FROM PLAN

- Single-user accounts via email magic link: The plan's articulated why is account safety and privacy: magic links are "15 min, single-use", the magic-link endpoint returns "202 always (anti-enum)", sessions can be listed/revoked, and email is encrypted or hashed rather than used as a log key.

- One canonical aviary per account: NOT RECOVERABLE FROM PLAN

- Multi-device sync via server-canonical state: The rationale is to avoid "Last-write-wins personality sync", "client-owned simulation", vector clobber, and "Lost relationship history"; clients append events and snapshots are read models.

- Personality vectors with monotonic drift: The plan's why is expressive relationship growth without neglect punishment: regular visits produce week-scale measured delta and multi-week visible changes, while pure absence ticks assert "no negative" delta.

- Mood: The why is continuity on a faster timescale; mood "persists across sessions", "never snap[s] to neutral on open", and advances with time of day, weather, interactions, personality, and neighbors.

- Procedural calls: The plan ties this to aliveness and avoiding uncanny repetition: "never bit-identical twice", species recognizable blind tests, "no sample packs", and "no recorded MP3 fallback ever."

- Idle motion and pose: The why is a living scene that starts mid-action: snapshot `pose.phase` gives the "First frame" and explicitly avoids "entry animation"; random micro-parameters are preferred to "looped GIFs."

- Bird-to-bird interaction: NOT RECOVERABLE FROM PLAN

- Two starter birds with no catalog: The plan says the first two are auto-assigned with "complementary motif families", giving "diversity without choice UI."

- User naming and renaming: The rationale is stable identity: bird UUID is "stable identity forever" and "immutable across renames."

- Aviary growth to max 7 by aviary age: The why is to keep growth out of pay or engagement systems: "Never unlock via pay/engagement score" and "age-only" unlocks avoid gamified "unlock!" language.

- Return-greeting: The plan's rationale is behavioral acknowledgement without chrome: absence buckets and bird ranking drive greeting, while "Zero text welcome surfaces" and no "Welcome" preserve the quiet scene.

- Presence accounting: The why is bounded honesty: focus, visibility, and recent activity prevent "Background tabs inflate drift", while the 180s window mitigates "Still watchers under-credited."

- Listen-in: NOT RECOVERABLE FROM PLAN

- Offer (seed / song fragment / still pool): NOT RECOVERABLE FROM PLAN

- Offer cooldown: The rationale is bounded interaction; cooldowns are server-enforced per bird and type, and the plan deliberately does not measure "conversion on offers spam."

- Settle: The why is quieting rather than decay or cross-device conflict: settle gives "mood quiet only", ends presence, and is treated as "session soft state" so two devices do not fight a hard global settle.

- Field notebook: The why is sparse observation instead of a metric log: entries are "read-only", "sparse naturalist", capped for novelty, and filtered to avoid streak language, numeric traits, and "you visited."

- Single horizontal scene: NOT RECOVERABLE FROM PLAN

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Local-time day/night: The rationale is mood and atmosphere continuity from local time: the client renders the palette from local timezone, and the server uses stored timezone for "night mood transitions in tick."

- Rare ambient weather: The why is ambience without attention steal; the plan calls weather "rare", uses "weekly expected rates", and mitigates "Weather/assertive FX" with capped frequency and low amplitude.

- Top-bar chrome that fades on idle: The rationale is a low-chrome aviary: top bar opacity fades after pointer idle, there are no in-scene buttons or badges, and system dialogs use matter-of-fact copy.

- Optional quiet visits: The plan's why is bounded sharing without social network mechanics: visits are read-only, revocable, off by default, have no co-presence, and visitor presence "must not" write host presence events.

- Screen-reader naturalist narration: The rationale is equal access in the same product voice: narration uses live regions, updates on meaningful events, and never exposes state as labels like "mood: content."

- Reduced-motion cross-fade mode: The why is accessible motion without degrading the product: it replaces frame animation with authored still pose cross-fades and is "equal product quality."

- Call captions: The why is audio access and fallback: captions appear when muted or when AudioContext fails, and the phrase must "Match actually synthesized call."

- WCAG AA chrome and full keyboard navigation: The rationale is that essential controls must be operable and legible across bright day and dim night palettes, with a focus ring and keyboard path included in day-one accessibility.

- Account export: The plan's why is user ownership and portability: live UI hides raw personality numbers, but export includes vectors because "user owns their data for portability."

- Soft-delete for 30 days then hard-delete: The rationale is account recovery followed by purge; delete marks the account, undelete works "within 30d", and purge happens at day 31 including events.

- Session list/revoke: The why is device control for account safety: sessions carry device labels, last-seen state, and revocation.

- Privacy-bounded telemetry: The rationale is "Instrument health, not exploitation"; allowed metrics are counters/histograms, while per-bird personality, joinable interaction analytics, and presence timelines into ML training are forbidden.

### 2. Architecture

- Single deployable service initially: The plan's articulated why is operational simplicity with "clear module boundaries"; even in one binary, auth, aviary, events, sim, notebook, visits, accounts, telemetry, and mail remain logical packages.

- Postgres primary store: The rationale is authority: Postgres is the "system of record" for accounts, birds, events, and snapshots.

- Tick worker as exclusive personality writer: The why is authoritative drift and sync safety: it is the "exclusive writer of personality", updates vectors/mood, and prevents client authorship.

- Snapshots as read models: The rationale is fast and canonical reads; `aviary_state` stores denormalized snapshot fields for "fast GET", and clients discard local prediction on snapshot.

- Client-owned call synthesis and idle motion: The why is responsive rendering/audio from server constraints while keeping personality and mood authoritative on the server.

- Client-only ambient ornaments: NOT RECOVERABLE FROM PLAN

- Rendering pause or 1fps throttle when hidden: The rationale is explicit: "to save battery"; the simulation "never" depends on client frames.

- Service module boundaries: The why is maintainability across one binary: the plan says "Even if one binary" and lists logical packages with separate concerns.

### 3. Data model

- Synthetic account UUIDs with encrypted email and blind index hash: The rationale is to avoid "Email as ID leakage" and "PII sprawl"; email is never a FK or log key.

- Account settings defaults: The why is quiet and private defaults: visit notifications default false, audio and accessibility settings are explicit, and visit notifications are opt-in/off by default.

- Aviary `created_at` driving unlocks: The rationale is age-only growth, separate from engagement metrics, pay, or gamified progress.

- Bird stable ID, species, name, personality, mood, perch, pose, and call seed: Stable ID preserves identity; pose seed supports mid-action render; call seed supports recognizable procedural variation.

- Append-only interaction events: The why is ordered, idempotent processing by the tick; events are client append-only and personality drift is not patched directly by clients.

- `client_event_id`: The rationale is idempotency for batched event append.

- `rename_bird` marked administrative, not drift: The why is to prevent administrative changes from affecting personality.

- Visit session/log without host presence events: The rationale is that visitor activity records duration only and does not become host drift.

- Presence windows with server-side clamps: The why is bounded correctness; the server prevents pings from claiming more than wall-clock or more than the max credit interval.

- No live API for raw personality numbers: The plan's why is that numbers "ruin relationship"; live snapshots use hints only, with vectors export-only.

### 4. API surface

- Matter-of-fact auth/system errors: The rationale is voice boundary; errors are `{ "message": "..." }` and have "no naturalist prose."

- Auth magic-link endpoint returning 202 always: The why is "anti-enum" behavior plus rate limiting.

- `/v1/bootstrap`: The rationale is first paint: a "minimal snapshot" and "edge-friendly" path help draw the aviary quickly.

- `/v1/aviary/stream`: The why is optional near-live convergence; SSE can publish version bumps and light diffs for multi-device pose convergence.

- Live snapshot excluding personality floats: The rationale is to avoid exposing raw numbers while still allowing discrete visual hints such as `plumage_hint`.

- Server-computed greeting: The why is canonical behavior from boldness, mood, and `last_presence_end`, without client-owned personality logic.

- Batch `/v1/events`: The rationale is append-only idempotent ingest; visitor tokens and foreign birds are rejected.

- Notebook API with no write/delete/annotate: NOT RECOVERABLE FROM PLAN

- Visit APIs with token snapshot and heartbeat: The rationale is read-only sharing plus duration logging; heartbeat is "duration only" with "no presence into host drift."

- Visitor revoke returning 410: The rationale is immediate revocation with matter-of-fact failure on the next pull.

- Narration helper assembled server-side: The why is "voice consistency" from the same state as the snapshot.

- `ETag` / `If-None-Match`: The rationale is "cheap keepalives."

### 5. Simulation engine design

- Minute-scale tick loop: The why is to process events, time-of-day, weather, mood, birds, notebooks, and snapshots centrally while allowing jitter under load.

- Minimum tick for time-driven work: The rationale is that even without unprocessed events, mood and time-of-day transitions still need periodic advancement.

- Ordered per-tick event processing: NOT RECOVERABLE FROM PLAN

- Clamped additive personality deltas: The why is monotonic expressive drift; traits only move upward and are capped at 1.0.

- Leaky integrator and `trait_openness`: The rationale is slow progress with diminishing returns, avoiding instant max while still producing measurable week-scale deltas.

- Calibration targets: The why is to avoid both "Tamagotchi feel" from too-fast drift and "Screensaver feel" from too-slow drift.

- Recent presence score as behavior policy: The rationale is ambient quiet after absence without trait decay: quieter greets, not down-drift.

- Mood transition drivers: The why is continuity across interaction residuals, local hour, weather, personality modifiers, and neighbor alarm.

- Call-grammar server seed plus client synthesizer: The rationale is recognizable species identity with non-identical variation and no recorded samples.

- Idle motion reduced-motion alternate: The why is accessibility parity through cross-fade keyposes rather than frame animation.

- Age-gated adoption card in naturalist voice: The rationale is adding birds without gamified "unlock!" framing.

- Notebook template generator: The why is reliability in v1; the plan says "v1 template compositional" and only defers a light LLM optionally.

### 6. Sync model

- Local UI instant feedback before server incorporation: The plan's why is "snappy UX", with local feedback discarded or reconciled on the next snapshot.

- Offline-capable event outbox: The rationale is degraded operation without local personality invention; events flush when online and no local simulation invents drift.

- No LWW on personality: The why is conflict prevention; clients cannot PATCH vectors, so personality clobber is "impossible path."

- Concurrent listen-ins from two devices: The rationale is additive event history with diminishing returns instead of conflicts.

- Settle as session soft state: The why is that "two devices don't fight hard global settle."

- Server `received_at` ordering for clock skew: The rationale is that client timestamps are advisory only.

- Pull on open, visible, resume, keepalive, and optional SSE: The why is snapshot freshness after sleep, visibility changes, and multi-device convergence.

- Quiet field loading on snapshot failure: The rationale is degraded calm: "never spinner-as-brand."

### 7. Frontend rendering pipeline

- TypeScript + Vite: NOT RECOVERABLE FROM PLAN

- Canvas 2D with SVG sprites rasterized to atlas: The rationale is a "predictable 60fps path."

- CSS only for top bar/settings/notebook panel: NOT RECOVERABLE FROM PLAN

- Route shell `/`, `/settings`, `/auth`, `/visit/:token`: NOT RECOVERABLE FROM PLAN

- Code-splitting settings, visits, and account paths: The why is budget control; non-aviary routes are code-split to protect first paint and bundle size.

- Scene layer order: NOT RECOVERABLE FROM PLAN

- No in-scene buttons, badges, tooltips, or drag handles: The why is preserving aviary behavior over selection chrome and gamified surfaces.

- Responsive all-birds-on-screen rule: The rationale is direct: always keep all birds visible, with no pan/zoom/scroll scene.

- Safe areas and constant top bar height: NOT RECOVERABLE FROM PLAN

- First paint protocol: The why is "already in motion"; it shows the quiet field immediately and forbids splash, welcome, loader, and confetti.

- Audio context resume on first gesture: The rationale is browser autoplay policy; visual aliveness must not wait.

- Continuous micro-motion FSM: The why is expressive aliveness conditioned by mood and personality, using random walks rather than looped GIFs.

- Perch change transition: The rationale is motion accessibility: hop arc for full motion, cross-fade for reduced motion.

- Listen-in gentle visual weight: The why is focus without selection chrome, except for an accessibility focus ring when keyboard.

- Top-bar icons only: The rationale is quiet chrome; tools are present but fade and avoid text-heavy announcement surfaces.

- Empty aviary after starter naming: The why is "quiet field -> soft fly-in once -> never empty again."

### 8. Audio pipeline

- Listen-in bus and chorus bus graph: NOT RECOVERABLE FROM PLAN

- Listen-in gain ramps with floor greater than zero: NOT RECOVERABLE FROM PLAN

- WebAudio oscillators, noise buffers, filters, envelopes, and motif DSL: The rationale is procedural synthesis without asset packs, recorded audio, or identical repeats.

- Buffer pooling: The why is the "no-memory-growth budget."

- Independent chorus schedulers with soft phase avoidance: The rationale is to avoid "phase-canceled drones" and forced quantization.

- Captions generated at play time: The why is fidelity; captions must match the actually synthesized call.

- AudioContext fallback to quiet visuals and captions: The rationale is graceful silence without recorded fallback, plus captions when audio fails.

### 9. Accessibility surfaces

- Live-region narration every 30-60s idle and faster on events: The rationale is screen-reader access to the living aviary without exposing mechanical labels like "mood: content."

- Accessible bird focus names: The why is focus clarity through user bird names and short species gloss.

- Notebook list semantics as observation log: The rationale is to make notebook entries navigable as observations.

- Keyboard path through controls and birds: The why is full operation without pointer for listen-in, offer, settle, and panels.

- Visible focus ring across palettes: The rationale is usability in both bright day and dim night palettes.

- Contrast rules with scrim on busy foliage: The why is WCAG AA for chrome/text/captions and avoiding essential UI on busy scene areas.

- Accessibility in same release train: The rationale is that screen-reader and reduced-motion users should not get an "inferior product."

### 10. Performance budgets and observability

- Initial JS, first-bird, FPS, memory, snapshot, and tick budgets: The why is to protect "already in motion" loading, 60fps idle, flat memory, and timely tick processing.

- Procedural audio/visuals over asset packs: The rationale is budget control: "Procedural audio/visuals beat asset packs" and no multi-MB PNGs.

- Pausing render when hidden and using object pools: The why is battery and memory stability.

- Virtualized notebook history: The rationale is to avoid retaining all notebook DOM nodes.

- Aggregate-only observability: The why is privacy isolation; per-bird personality and joinable analytics are forbidden.

- Synthetic fleet for first-bird and FPS checks: The rationale is to verify performance across multiple geos and browser profiles.

- Deliberately not measuring DAU streaks, offer conversion spam, or engagement leaderboards: The rationale is explicit: "Instrument health, not exploitation."

### 11. Interactions implementation notes

- Return-greeting bird ranking by boldness, mood, and RNG: The rationale is bird-behavior greeting without text welcome surfaces.

- Secondary greet staggering: NOT RECOVERABLE FROM PLAN

- Presence probe requiring visibility, focus, and recent pointer/key activity: The rationale is to keep drift honest and avoid background tab inflation.

- Presence ending on settle, tab close, hide, blur, or timeout: The why is to prevent residual presence while hidden or settled.

- Listen-in exit rules: NOT RECOVERABLE FROM PLAN

- Offer reactions by mood and curiosity: The rationale is that interactions express current bird state rather than fixed buttons.

- Settle lighting transition: The rationale is session quieting: 2-4s lighting shifts to evening and calls quiet.

- Settle 5s undo window: NOT RECOVERABLE FROM PLAN

- Field notebook infinite history scroll: NOT RECOVERABLE FROM PLAN

### 12. Social visits

- Default-off invites: The why is explicit quiet sharing; no invites exist until host acts.

- Per-invite email and 30-day expiry: NOT RECOVERABLE FROM PLAN

- Visitor read-only snapshot and heartbeat duration log: The rationale is to share the aviary without co-presence or host drift.

- Host visit log with email, time, and duration but no badge pings: The why is factual accountability without gamified notifications.

- Visit notification email toggle off by default: The rationale is opt-in notification behavior.

- Revocation returning 410 on next visitor pull: The why is immediate revocation.

### 13. Accounts, privacy, compliance engineering

- Export JSON including birds, names, vectors, moods, notebook, and settings: The rationale is private portability while keeping live UI free of personality numbers.

- Hard purge including events after soft delete: The why is privacy completion after the 30-day recovery window.

- Privacy policy link in settings: NOT RECOVERABLE FROM PLAN

- Per-bird events for that user's tick only: The rationale is purpose limitation; the plan says events' "sole purpose" is that user's tick.

### 14. Rollout

- M0 foundations: The rationale is to establish auth, account UUIDs, quiet field shell, snapshots/events, tick skeleton, and perf harness before feature depth.

- M1 one living bird: The why is proving the living loop: visuals, procedural call MVP, presence, drift sims, greeting, idle motion, day/night, reduced motion, focus ring, and live region.

- M2 companion pair: The rationale is to add the two-bird experience, bird-to-bird calls, listen-in mix, offers, notebook, and memory/FPS soak tests.

- M3 product completeness: The why is to finish settle, weather, account controls, captions, visits, narration cadence, and locked budgets.

- M4 hardening: The rationale is to validate drift over 3-week sims, multi-device soak, tick lag chaos, and accessibility dogfood before soft launch.

- Day-one instrumentation: The why is ops and health: tick p99, ingest errors, first-bird paint, audio failures, magic-link delivery, and visit revoke correctness.

- Content ops for six species and naturalist strings: The rationale is voice and non-gamification review.

### 15. Testing strategy

- Deterministic accelerated simulation harness: The rationale is to assert monotone personality, calibration envelopes, no hidden presence, and mood continuity across sessions.

- Visual regression on palette phases: The plan says "not every feather", so the why is regression coverage at the palette/phase level rather than exhaustive feather precision.

- Reduced-motion goldens: The rationale is to keep reduced-motion mode as a first-class path.

- Keyboard path end-to-end tests: The why is to verify full keyboard operability.

- Audio tests with mocked AudioContext and captions fallback: The rationale is to ensure captions cover unavailable audio.

- Memory soak playwright 30 min: The why is the no-memory-growth budget.

- Two-client event race tests: The rationale is to prove "no vector clobber."

- Visitor cannot POST host events: The why is authorization and visit scope containment.

- Voice lint banned patterns: The rationale is to prevent UI strings such as "Welcome back", "achievement", "streak", "level up", and "daily reward" from reintroducing gamification.

### 16. Risks, layout guide, and done criteria

- Repository layout with `/apps/web`, `/apps/api`, and packages: NOT RECOVERABLE FROM PLAN

- Definition of done requiring two living birds, path to seven, drift, interactions, procedural chorus, visits, a11y, budgets, privacy, and rejection of gamification/native/social scopes: The rationale is to preserve the "affective core" while delivering the v1 quality bar.

- Deliberately not implementing code, colors, hosting vendor, ML training, or hidden achievements: The why is scope control and avoiding dilution; the final line says the plan is an engineering blueprint "without diluting the PRD's affective core."
