## System-level intent

1. Quiet, non-gamified relationship over product scoring. This shows up in the explicit non-goals: no "streaks, badges, levels, scores, calendars, XP," no "welcome back toast components," no "stats panels listing boldness," and the success definition that "neglect is quiet ambient, never guilt." The plan repeatedly protects the product from "Tamagotchi feel" and "streak-like DAU gamification funnels."

2. Server-owned, canonical personality. The plan makes "server-side simulation tick" the "sole writer of personality, mood, and bird placement state," says "no client-owned personality state," and repeats in sync that only the "sim-worker" updates personality columns. The reason is multi-device correctness: devices "GET the same snapshot" with "no merge of personality," avoiding "LWW personality bug" and "silent character loss."

3. Monotonic, slow, attention-shaped drift. The plan's drift rule is upward only: "Neglect -> Delta t = 0 (no decrease)." It calibrates for small weekly change and says single sessions should be "invisible." The philosophy is named in the presence-harmony note: "quieter, not mistrusting," so absence modulates expression without negative stored drift.

4. Alive, not loading theater. The first paint path asks for "soft sky CSS immediately," a bird drawn "mid-pose" with "no fade-from-static, no entry animation," and "quiet field only" if slow, "never spinner." The success definition restates this as a user opening a tab and seeing "somewhere mid-preen a bird notices them without a banner."

5. Naturalist voice for the aviary; matter-of-fact voice for systems. The plan separates "naturalist screen-reader narration" and "lowercase naturalist" notebook prose from "matter-of-fact error bodies," "auth timeouts," and a11y settings copy. The modules map formalizes this with "Voice lint: product copy files tagged `naturalist` vs `system`."

6. Accessibility is part of the affective product, not a later compliance pass. Accessibility appears in scope as "naturalist screen-reader narration," "reduced-motion designed surface," "captions," and "full keyboard nav." Rollout requires notebook, narration, reduced motion, captions, and keyboard to "ship same milestone as 'scene complete'," and launch criteria require reviewers to experience reduced motion and screen-reader story as "charm, not bare labels."

7. Privacy and data minimization are hard architecture boundaries. The plan uses synthetic UUIDs, encrypted email, "append-only" events tied to account lifetime, visitor read-only mode, aggregate-only telemetry, and "no bird ids, no traits, no notebook prose, no emails" in RUM. It also states a "Hard privacy line: sim DB credentials not available to BI warehouse roles."

8. Social contact must stay quiet, opt-in, and read-only. Visits are described as "Quiet visit invitations," "opt-in per invite," "read-only ambient," "revocable," and "default off." The risk table names "Visit feature creep" as "Social network pivot" and mitigates it by maintaining "RO only" with "no visitor write APIs ever."

9. Procedural, recognizable life instead of canned media. The audio design insists on "Procedural call synthesis (WebAudio)," "never looping WAV," "No MP3 pack," and "procedural only." Recognizability is protected by a "spectral fingerprint" and "motif vocabulary" that stay sticky per species and bird instance.

10. Performance is an affective requirement. The budgets target first bird visibility, "60fps idle," and "no memory growth over 30 min." The plan ties bundle bloat and memory leaks to product failure, and launch criteria include "First frame always mid-motion" and "Silence+captions path polish."

## Per-feature whys

### 1. Scope

- Browser-only SPA: NOT RECOVERABLE FROM PLAN

- Single-user accounts with email magic-link auth and synthetic account UUID primary key: Synthetic UUIDs are for privacy and identity separation: account IDs are "never email-derived," account IDs are "never in logs as email," and email is encrypted while lookup uses a keyed hash. Magic links are part of the auth surface, but a product rationale beyond web auth is NOT RECOVERABLE FROM PLAN.

- One canonical aviary per account: The plan ties this to canonical sync by making `account_id` unique on `aviaries` and saying there is "one aviary per account v1," but a separate product rationale for one aviary is NOT RECOVERABLE FROM PLAN.

- Two starter birds: NOT RECOVERABLE FROM PLAN

- Hard cap of 7 birds: The cap protects recognizability and product feel. The risk table says "Recognizability collapse at N->7" would mean "Cap fails product," with mitigation to "keep 7 hard."

- Server-side simulation tick as sole writer of personality, mood, and placement state: The rationale is canonical multi-device behavior and conflict avoidance: "no client-owned personality state," clients "never PATCH vectors," and only the worker role should update trait columns to prevent "silent character loss."

- Client pull of snapshots plus local interpolation: HTTP pull keeps realtime out of the critical path for v1: "Do not require WebSocket." Local interpolation on resume prevents visible jumps: reconcile poses over "300-800ms" with "no teleport unless delta large."

- Append-only interaction/presence event log: Events are the input to tick deltas; the worker loads "unprocessed events," applies drift, marks events processed, and keeps one transactional history. The plan also needs ordering and duplicate handling: server timestamp order and unique client event IDs.

- Multi-device sync as an architectural property: The plan says devices A and B both receive "the same snapshot" and there is "no merge of personality." The why is to avoid shareable mutable client state and last-write-wins personality bugs.

- Return-greeting interaction: The plan uses it to make a bird notice the returning user "without a banner." Selection avoids spectacle: "Never synchronized full-chorus greet."

- Listen-in interaction: Listen-in lets attention focus on one bird while preserving the ambient aviary: focused gain rises and others drop to a floor, "never 0." Listen-in seconds also contribute to drift, "stronger on warmth & vocal_frequency."

- Offer interaction: Offers provide "proximity/accept signals" for curiosity and boldness drift, and can produce an "immediate reaction hint" when UX needs acknowledgement before the next tick. A deeper product rationale for seed, song fragment, and still pool as specific offer types is NOT RECOVERABLE FROM PLAN.

- Settle interaction: Settle is a quiet session-ending/cosmetic mode: it applies a "warm evening grade + gain reduce," stops presence, and contributes "P end cleanliness only" with "no directed trait push."

- Field notebook, read-only: The notebook preserves sparse naturalist observations while avoiding user scoring and trait exposure: entries are "lowercase naturalist," "never user behavior scoring," and "never raw trait deltas."

- Presence accounting from visibility, focus, and recent activity: The rationale is honest attention. The plan says do not count "hidden tabs, unfocused windows, pure open-idle-overnight," and names lax presence as causing the population to "over-drifts silently."

- Day/night from user local timezone: Local time modulates moods, with morning -> alert/curious, dusk -> drowsy, and night -> settled/sleeping. The success definition also expects "phone at night matches laptop morning."

- Rare ambient weather: Weather affects mood and audio expression, such as rain that can "dampen vocal" into "drowsy/content quieter." A specific rationale for rarity is NOT RECOVERABLE FROM PLAN.

- Three perch zones: Perch zones support placement and scale in the scene, with back, mid, and front layers and mood x boldness placement. A specific product rationale for exactly three zones is NOT RECOVERABLE FROM PLAN.

- Procedural call synthesis: The plan avoids recorded loops and sample packs so bird calls remain unique and alive: "never looping WAV," "No MP3 pack," and "Never identical double."

- Listen-in mix rebalance: The rationale is focus without erasing the shared soundscape: the focused bird ramps up while others ramp down to a floor, "never 0."

- Captions: Captions keep calls available when audio is muted or unavailable. The fallback makes captions default "on" if WebAudio is unavailable, and mute makes "captions auto-on."

- Silence plus captions fallback: The fallback preserves the product's life without recorded audio: "permanent silent mode + captions on" and "No MP3 pack."

- Accessibility surface: The rationale is first-class affective access, with "naturalist screen-reader narration," "reduced-motion designed surface," and launch criteria requiring accessibility reviewers to experience "charm."

- Quiet visit invitations: Visits allow read-only ambient sharing while preventing a social-network pivot: opt-in, read-only, revocable, default off, "no profiles, follows, discovery, comments, chat," and "no visitor write APIs ever."

- Account sessions revoke: Session revoke is a security/account-control feature; it appears in auth and security notes as "Session revoke."

- Email change: NOT RECOVERABLE FROM PLAN

- Export JSON: Export is framed as a "user-owned dump" including birds, names, vectors, moods, notebook, and settings.

- Soft-delete for 30 days then hard delete: The rationale is privacy and account lifecycle control: "Hard delete (day 30+)" cascades birds, events, notebook, invites, sessions, and "scrub backups per policy schedule."

- Privacy policy link: The plan wants plain disclosure of allowed aggregates: "Privacy policy link plain text listing aggregates allowed."

- Visit log: The plan includes host visit durations and visit sessions, but a specific rationale beyond account transparency is NOT RECOVERABLE FROM PLAN.

- Visit-notification toggle off by default: The rationale is privacy and quiet defaults: visit email notifications are optional and "double opt-in host-side toggle off by default."

- Performance budgets: The plan treats performance as part of the experience: first bird visible quickly, idle 60fps, no memory growth, and "First frame always mid-motion when data ready."

### 2. Architecture

- API gateway: It centralizes auth, snapshot reads, event ingest, account/export/delete, and visits. A separate product rationale for this gateway shape is NOT RECOVERABLE FROM PLAN.

- Logical services that can start as one deployable: The plan allows "one deployable with internal modules" while naming auth, aviary, sim, visit, account, and mailer responsibilities. A deeper rationale for this deployment choice is NOT RECOVERABLE FROM PLAN.

- `auth-api`: NOT RECOVERABLE FROM PLAN

- `aviary-api`: NOT RECOVERABLE FROM PLAN

- `sim-worker`: Its rationale is the single-writer simulation boundary for personality, mood, and placement.

- `visit-api`: Its rationale is to isolate invite create/revoke/accept and visitor read-only snapshots, keeping visits quiet and revocable.

- `account-api`: NOT RECOVERABLE FROM PLAN

- `notify-mailer`: The rationale is to restrict email to "transactional email only," with no push/email about the aviary except magic links and user-requested export/recover flows.

- No analytics warehouse read path into per-bird tables: The rationale is privacy; ops telemetry has "no join keys to bird personality."

- Client/server split for personality vector and drift: Server ownership keeps personality canonical and avoids client-owned state.

- Client ownership of visual interpolation and procedural audio synthesis: The plan keeps snapshots as pure data and lets the client render, interpolate, and synthesize from parameters, keeping HTML frames and audio samples off the server path.

- Render pipeline boundary with pure JSON snapshots: The rationale is a clean data-to-scene mapping: "No HTML frames from server."

- Suspending rAF and audio when hidden: The rationale is performance/resource control while presence fails visibility anyway.

- Resume snapshot fetch on visibility or long frame gap: The rationale is correctness after time has passed, with a hard reconcile to current server state.

- Monorepo layout: NOT RECOVERABLE FROM PLAN

- Pure `sim-core`: The plan says it "must be pure and unit-testable without DB"; the worker wraps persistence and locking around it.

### 3. Data model

- Encrypted email and keyed email hash: The rationale is privacy and lookup separation: email is encrypted at rest, while hash is "for lookup only."

- Account timezone: It supports local day phase and mood/time behavior.

- Settings JSON: It stores a11y preferences, visit notification opt-in, and related account settings. A deeper rationale is NOT RECOVERABLE FROM PLAN.

- Sessions with token hash and device label: Sessions enable device listing and revocation. A deeper rationale for device labels is NOT RECOVERABLE FROM PLAN.

- Magic links table: It supports single-use, expiring auth. A deeper product rationale is NOT RECOVERABLE FROM PLAN.

- Aviary `settled_until`: It supports the settle cosmetic state; the server may clear it.

- Aviary `weather_state`: It stores rare ambient weather. A deeper rationale beyond weather state is NOT RECOVERABLE FROM PLAN.

- Aviary `last_ticked_at` and `sim_version`: These support tick scheduling and sim engine migration/versioning. A product rationale is NOT RECOVERABLE FROM PLAN.

- Stable bird IDs forever: The rationale is stable bird identity.

- Bird personality columns: The rationale is canonical, server-only personality state used by drift, mood, placement, calls, and greetings, while never exposing numeric vectors in UI.

- Bird mood, perch zone, animation seed, and call phase: The rationale is to make snapshots resume into living mid-action state and allow server-ticked mood/placement with client rendering.

- `last_offer_at`: It enforces per-offer cooldowns. A product rationale for the exact cooldown is NOT RECOVERABLE FROM PLAN.

- Never expose personality fields on client-facing UI DTOs: The rationale is to avoid "personality vector numeric exposure anywhere" and "stats panels listing boldness."

- Interaction events append-only: The rationale is ordered, processable sim input; tick consumes unprocessed events and marks them processed.

- Notebook entries immutable with no user edit/delete: The plan states "Immutable; no user edit/delete," but a specific rationale is NOT RECOVERABLE FROM PLAN.

- Visit invites and visit sessions: The rationale is opt-in, revocable visit access with approximate durations and no presence contribution to sim.

- Snapshot DTO omitting raw personality: The rationale is to prevent raw personality exposure while still giving the renderer derived fields.

- Visitor snapshots strip account chrome and flag `visit_ro`: The rationale is read-only ambient visits with no host/account affordances.

### 4. API surface

- Auth routes using session cookie or Bearer token: NOT RECOVERABLE FROM PLAN

- Magic-link request returns generic OK and is rate limited: The rationale is security and privacy around auth, avoiding email/account enumeration while protecting deliverability.

- Magic-link consume single-use with 15-minute expiry: The rationale is security; the security notes repeat "single-use, 15m, rate limits."

- Matter-of-fact auth error bodies: The rationale is voice separation: system failures use matter-of-fact copy, not naturalist voice.

- Snapshot GET with ETag on `sim_tick`: The plan uses HTTP pull and efficient snapshot refresh; a specific rationale for ETag beyond caching is NOT RECOVERABLE FROM PLAN.

- Batched event POST: The rationale is one append log for presence and interactions, schema-validated and rate-limited, with no personality numbers returned.

- Notebook GET with cursor pagination: Pagination supports the notebook list over time; a deeper rationale is NOT RECOVERABLE FROM PLAN.

- Timezone POST: The rationale is local day/night and mood computation.

- Events-only preferred path for settle/offers/listen-in: The rationale is "one write path" and preserving the tick as "sole vector writer."

- Optional offer endpoint with immediate reaction hint: The rationale is UX acknowledgement "before next tick" without a persisted personality write.

- Optional settle endpoint: The rationale is to append the settle event and expose the snapshot flag while keeping the event path.

- Onboarding complete assigning two species server-side: The server chooses starter species, but a specific rationale for server-side assignment is NOT RECOVERABLE FROM PLAN.

- Bird rename endpoint: It allows `{ display_name }` only; a specific rationale is NOT RECOVERABLE FROM PLAN.

- Bird adopt endpoint: The rationale is age-gated, capped growth: adoption only if age gate is open and count is below 7, with the system choosing species and no engagement gate.

- Visit invite create/list/revoke/view/snapshot routes: The rationale is host-controlled, revocable, read-only ambient access.

- Revoked visitor snapshot returning 410 with matter-of-fact copy: The rationale is immediate revocation plus system voice for revoked access.

- Account settings GET/PATCH: It supports a11y and visit notification preferences. A deeper rationale is NOT RECOVERABLE FROM PLAN.

- Account export endpoint: The rationale is a "user-owned dump."

- Account delete and cancel endpoints: The rationale is soft delete with a 30-day cancellation window before hard delete.

- HTTP pull only, no required WebSocket: The rationale is to keep SSE/WebSocket "out of critical path" for v1.

### 5. Simulation engine design

- Tick loop every about 60 seconds: The rationale is steady server-side evolution with load flexibility; exact cadence is chosen as "60s" with "45-90s under load" and "never faster than 30s."

- Tick lock or row claim: The rationale is to ensure one worker claims an aviary at a time.

- Loading unprocessed events since last tick: The rationale is event-driven drift and mood computation.

- Wall-clock and local day phase computation: The rationale is local time mood behavior.

- Ambient weather generator seeded by day and aviary: It produces consistent ambient weather. A deeper rationale is NOT RECOVERABLE FROM PLAN.

- Aggregating presence and interactions into deltas: The rationale is to translate attention into drift inputs.

- Monotonic upward drift: The rationale is no punishment on neglect; "no decrease" and "quieter, not mistrusting."

- Mood transitions: The rationale is living offline behavior: local hour, offers, alarms, rain, and personality modulate persistent mood.

- Perch preference updates from mood and boldness: This translates personality and mood into visible placement, such as front perch rate.

- Animation seed and call phase advancement: The rationale is mid-action first frames and non-synchronized rhythm.

- Notebook entry emission with sparsity gate: The rationale is sparse observational history, targeting "2-4 entries/week" and cap about one per day.

- Adoption eligibility by age only: The rationale is growth without engagement pressure: "never engagement-gated," "never meta progress bar," and "Never sell unlocks."

- Transactional state write and processed marks: The rationale is idempotence: "marking processed and state write in one transaction."

- Drift calibration harness: The rationale is to avoid drift too fast or too slow by checking 7-day and 21-day targets.

- Ambient attention response/presence-harmony term: The rationale is expression change without negative trait drift: "quieter, not mistrusting."

- Optional optimistic mood nudge on offer: The safer default avoids API mood writes; if used, it must be the same pure function as tick and versioned.

- Call-grammar shared IR: The rationale is shared motif definitions, mood mods, and caption templates across server/client.

- Server scheduling of `next_call_eta`: The server stores parameters and can simulate abstract calls for notebook without audio.

- Client synthesis of actual samples: The rationale is unique WebAudio calls from motif weights, mood mods, and sticky species/bird fingerprints.

- Return-greeting selection by boldness, warmth, and mood: The rationale is bird-specific noticing without synchronized spectacle.

- Bird-to-bird micro-rules: The rationale is visible and audible inter-bird behavior: call response, wary contagion, and chorus windows.

### 6. Sync model

- Canonical single writer: The rationale is no personality merge and no client vector patches.

- Server timestamp event ordering: The rationale is deterministic processing order.

- Duplicate event IDs with unique constraint: The rationale is safe duplicate client delivery.

- Avoiding shareable mutable client state: The rationale is conflict prevention: "No last-write-wins on birds."

- Presence lease: The rationale is to "Avoid accidental dual-device drift inflation" by allowing only the lessee's presence to count.

- Resume correctness flow: The rationale is to flush queued events, fetch current truth, and reconcile without teleporting.

- Matter-of-fact error surfaces: The rationale is copy voice separation for auth timeouts, snapshot failures, and revoked visits.

### 7. Frontend rendering pipeline

- React or Preact for chrome/settings only: Preact may reduce bytes; otherwise a deeper rationale is NOT RECOVERABLE FROM PLAN.

- Canvas 2D scene core: The rationale is "predictable perf" and "easy reduced-motion stills."

- Optional SVG parts prerendered to bitmap atlas: NOT RECOVERABLE FROM PLAN

- DOM top bar: The rationale is accessibility.

- No WebGL requirement for v1: The rationale is to avoid blocking time-to-first-paint; WebGL is "optional later for polish."

- Layered scene composition: It supports visual depth and fixed perch zones; a deeper rationale for exact layer order is NOT RECOVERABLE FROM PLAN.

- Fixed zone widths and soft anchors: The rationale is stable bird placement and responsive frame preservation.

- First paint with soft sky, critical atlas, and mid-pose bird: The rationale is "alive, not loading theater."

- Quiet field instead of spinner on slow snapshot: The rationale is to preserve the same calm place rather than loading chrome.

- Silent audio unlock on first pointer/key: The rationale is to satisfy browser gesture requirements without a modal wall "walling the scene."

- Idle micro-motion: The rationale is that the aviary remains alive: "Never freeze unless tab hidden."

- Perch transitions with reduced-motion cross-fade: The rationale is to support motion-sensitive users while keeping state changes visible.

- Settle and undo transition: Settle creates a warm quiet mode; undo within 5 seconds reverses the grade. A deeper rationale for the exact undo timing is NOT RECOVERABLE FROM PLAN.

- Reduced-motion mode: The rationale is a "designed mode" that keeps "full sim, notebook, audio/captions" while replacing loops with slow crossfades and removing leaf drift.

- Responsive single vertical composition: The rationale is to preserve all birds in frame, keep minimum hit targets, and "never crop birds."

- Top bar icons that fade after cursor idle: The rationale is quiet chrome; "No chrome inside scene."

- Empty aviary onboarding fly-in once: The plan says "once; never again," but a specific rationale is NOT RECOVERABLE FROM PLAN.

### 8. Audio pipeline

- Audio graph with per-bird gains, chorus bus, master, and optional ambient bus: It supports chorus mixing and listen-in automation. A deeper rationale for this exact graph is NOT RECOVERABLE FROM PLAN.

- Band-limited noise bursts plus FM/AM partials: This is the procedural synthesis implementation; a product rationale beyond non-recorded calls is NOT RECOVERABLE FROM PLAN.

- Per-bird sticky pitch offsets: The rationale is recognizability per bird instance.

- Mood-based rate and brightness changes: The rationale is to express mood in sound.

- Never identical double: The rationale is avoiding canned loops and keeping calls alive.

- Soft limiter and density cap: The rationale is to prevent audio overload and manage concurrent voices.

- Front birds louder and less low-pass filtered: The rationale is spatial perspective.

- Listen-in engage/disengage gain automation: The rationale is focus while preserving the chorus floor.

- Recording listen-in start/end events: The rationale is to feed drift and sim inputs for the focused bird.

- Captions from grammar templates and actual motif IDs: The rationale is synchronized captions that describe the real chosen call motif.

- Permanent silent mode if WebAudio is missing/denied: The rationale is graceful accessibility without a recorded fallback: captions stay on and copy is matter-of-fact.

- Audio memory pooling and leak test: The rationale is no memory growth over long gentle sessions.

### 9. Accessibility surfaces

- Screen-reader live region every 30-60 seconds: The rationale is observational narration without overwhelming telemetry.

- Priority queue for greeting, offer reaction, and settle: The rationale is to surface important events while keeping "observational prose."

- Avoid dumping pose telemetry: The rationale is naturalist narration rather than mechanical state output.

- Shared style rules between narration and notebook: The rationale is voice consistency.

- Keyboard navigation through controls and birds: The rationale is full keyboard access to listen-in, offers, settle, and panels.

- Focus ring with contrast on light and night skies: The rationale is usable keyboard focus across day/night visuals.

- WCAG AA chrome, settings, errors, captions, notebook text: The rationale is accessibility compliance for UI text, while scene art is exempt except overlays.

- A11y settings for reduced motion, captions, narration verbosity, and mute: The rationale is user control over sensory surfaces.

- Mute with captions auto-on: The rationale is that mute is "user audio preference," not "silence the product's life."

### 10. Presence accounting

- Presence condition requiring visible, focused, and recently active: The rationale is to count honest attention only.

- Presence pings every 30-60 seconds while true: The pings feed the append-only event log for sim deltas.

- Stop presence on settle/tab close: The rationale is session end cleanliness and no open-idle counting.

- Remote-config activity window: The rationale is calibration without redeploy.

- No scolding UI: The rationale is quiet non-guilt relationship design.

### 11. Field notebook generation

- Server-side sparse generator during tick: The rationale is canonical observation generation from sim state, gated to stay sparse.

- Noteworthy triggers like greeter swap, weather, quiet stretch, and new bird: The rationale is to capture meaningful naturalist observations rather than generic volume.

- Lowercase naturalist prose and lowercase bird names: The rationale is product voice.

- No user behavior scoring and no raw trait deltas: The rationale is anti-gamification and no numeric personality exposure.

- Store forever with infinite scroll virtualization: The rationale for virtualization is performance; the rationale for forever storage is NOT RECOVERABLE FROM PLAN.

### 12. Performance budgets and observability

- Initial JS budget: The rationale is meeting the first-bird and bundle bloat constraints; CI should fail over 2MB.

- Time to first bird budget: The rationale is affective immediacy; first frame must be mid-motion when data is ready.

- Idle FPS and memory budgets: The rationale is long gentle sessions without degradation.

- Snapshot size target: The rationale is fast pull snapshots; a deeper rationale is NOT RECOVERABLE FROM PLAN.

- Tick p99 alarm: The rationale is operational health for the simulation loop.

- Code splitting settings, visits, and export: The rationale is keeping the critical bird path small.

- Inline critical CSS sky: The rationale is immediate quiet field/sky first paint.

- Compress atlas and procedural body tints: The rationale is asset size control.

- Edge CDN static assets and ASAP snapshot: The rationale is faster first bird.

- Aggregate-only synthetic and RUM telemetry: The rationale is ops health without per-bird or personal join keys.

- Sim telemetry for duration, lag, and lock contention: The rationale is worker health.

- Deliberately not measuring boldness averages, offer rates per user, visit leaderboard inputs, or streak-like DAU funnels: The rationale is to avoid population productization and gamified measures of relationship quality.

### 13. Privacy implementation

- Email encrypted only on account row and all else UUID: The rationale is data minimization and identity separation.

- Interaction event retention tied to account lifetime and delete: The rationale is privacy lifecycle consistency.

- Visitors emit no sim events: The rationale is read-only visits that do not affect the host aviary.

- Optional visit email notifications double opt-in and off by default: The rationale is quiet privacy-preserving notification behavior.

- Hard delete cascading account data: The rationale is privacy after the 30-day soft-delete window.

### 14. Frontend modules map

- `shell`: NOT RECOVERABLE FROM PLAN

- `scene/renderer`: NOT RECOVERABLE FROM PLAN

- `scene/animator`: NOT RECOVERABLE FROM PLAN

- `audio/engine`: NOT RECOVERABLE FROM PLAN

- `presence`: NOT RECOVERABLE FROM PLAN

- `sync/client`: NOT RECOVERABLE FROM PLAN

- `a11y/narration`: NOT RECOVERABLE FROM PLAN

- `a11y/captions`: NOT RECOVERABLE FROM PLAN

- `notebook/ui`: NOT RECOVERABLE FROM PLAN

- `offers/ui`: NOT RECOVERABLE FROM PLAN

- `visits/ui`: NOT RECOVERABLE FROM PLAN

- `auth/ui`: NOT RECOVERABLE FROM PLAN

- Product copy files tagged `naturalist` vs `system`: The rationale is voice lint and keeping naturalist copy separate from matter-of-fact system copy.

### 15. Rollout

- Foundations phase: The rationale is to establish monorepo, auth, account UUID, empty snapshot, and quiet field first. A deeper why for this sequence is NOT RECOVERABLE FROM PLAN.

- Sim-core pure phase: The rationale is drift/mood tests and calibration harness before persistence.

- Tick worker plus event log phase: The rationale is proving "idle evolution offline."

- Two birds visual plus mid-action bootstrap phase: The rationale is the time-to-first-bird path and alive first frame.

- Audio grammar plus mixer plus listen-in phase: NOT RECOVERABLE FROM PLAN

- Offers plus settle plus presence phase: The rationale is to sign off "drift fingerprints."

- Notebook, a11y narration, reduced motion, captions, and keyboard phase: The rationale is that these ship with "scene complete," not as v1.1.

- Multi-device lease plus soak tests phase: The rationale is to prevent dual-device inflation and long-session failures.

- Visits read-only phase: The rationale is quiet sharing after core aviary behavior.

- Export/delete plus finds phase: NOT RECOVERABLE FROM PLAN

- Bird count ramp with launch max 2 and unlock flags disabled: The rationale is post-calibration growth and "Never sell unlocks."

- Day-one ops instrumentation: The rationale is monitoring auth, snapshot, tick, TTFP, audio, presence false positives, and JS health from launch.

- Six species packs: The plan specifies silhouettes, pose atlas, motif IR, and seeds, but a deeper rationale for exactly six is NOT RECOVERABLE FROM PLAN.

- Calm palette design tokens with AA pairs: The rationale is accessible chrome contrast and calm visual voice.

- Affective QA launch gate: The rationale is that no welcome toasts, mid-motion first frames, drift targets, accessibility charm, and silence+captions polish define release readiness.

### 16. Risks and mitigations

- Drift too fast mitigation: The rationale is avoiding "Tamagotchi feel" and spoiling the attention idea.

- Drift too slow mitigation: The rationale is avoiding a product that "Feels inert."

- Lax presence mitigation: The rationale is preventing silent over-drift.

- Dual-device inflation mitigation: The rationale is the same over-drift risk; use presence lease.

- Last-write-wins personality bug mitigation: The rationale is avoiding "Silent character loss."

- Audio uncanniness/loops mitigation: The rationale is preserving the spell with procedural calls, QA, and density limits.

- Recognizability collapse mitigation: The rationale is keeping the hard cap meaningful.

- Tick backlog mitigation: The rationale is avoiding "Time skip weirdness."

- Notebook genericity mitigation: The rationale is avoiding product-wide voice failure.

- A11y-afterthought mitigation: The rationale is affective product gating.

- Bundle bloat mitigation: The rationale is the 500ms first-bird budget.

- Memory leak mitigation: The rationale is keeping multi-hour gentle sessions alive.

- Streak/toast contributor regression mitigation: The rationale is preventing a "Principle break."

- Visit feature creep mitigation: The rationale is preventing a "Social network pivot."

- Magic-link deliverability mitigation: The rationale is avoiding "Auth death."

- Soft-delete misuse mitigation: The rationale is privacy.

### 17. Testing strategy

- Unit tests for sim-core, mood, greet selection, and grammar determinism: The rationale is validating pure simulation and deterministic seeded behavior.

- Property tests for presence: The rationale is proving presence requires all three signals.

- Contract tests for snapshot and event validation: The rationale is API/schema correctness.

- Visual screenshot tests for mid-pose load: The rationale is protecting the alive first frame.

- Performance tests: The rationale is enforcing first-bird, FPS, and budget targets.

- Accessibility tests: The rationale is validating chrome, keyboard path, and narration cadence.

- Privacy tests: The rationale is preventing email in logs and BI access to birds.

- Chaos test killing worker mid-tick: The rationale is lock release and no partial trait corruption.

### 18. Security notes

- Magic links single-use, 15-minute, rate-limited: The rationale is auth security.

- Session revoke: The rationale is account/session control.

- Unguessable and immediately revocable visit tokens: The rationale is safe read-only visit access.

- CSRF on cookie sessions: The rationale is standard web security.

- Expiring signed export URLs: The rationale is safe delivery of exported user data.

- Standard OWASP API hygiene: NOT RECOVERABLE FROM PLAN

### 19. Document collapse of "what not to build" into eng checklist

- Banned streak, achievement, badge, XP, level-up, leaderboard, feed, follow: The rationale is preventing gamification and social-network surfaces.

- Banned hunger, health bar, die/death for birds: The rationale is preventing Tamagotchi mechanics and guilt.

- Banned welcome back toast components: The rationale is preserving return without announce-on-return chrome.

- Banned stats panels listing boldness: The rationale is preventing personality vector numeric exposure.

- Banned recorded bird call assets as product path: The rationale is preserving procedural call synthesis.

### 20. Success definition

- User opens a tab and a bird notices them mid-preen without a banner: The rationale is alive quiet return.

- Presence quietly accrues and drift becomes visible over weeks: The rationale is slow relationship growth without numbers.

- Phone and laptop match across time of day: The rationale is canonical sync plus local day/night behavior.

- Muted users still get captions and prose that feel like the same place: The rationale is accessible continuity.

- Reduced-motion users get a calmer aviary that is still alive: The rationale is reduced motion as designed mode.

- Neglect is quiet ambient, never guilt: The rationale is anti-punitive relationship design.

- Engineering correctness defined as system properties, not marketing: The rationale is that these sentences must be implemented behavior.
