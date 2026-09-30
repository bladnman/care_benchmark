## System-level intent

- Real continuity, not arrival simulation. Shows up in "the server advances the birds while nobody is watching," "a browser renders their current lives," "no entry sequence for a returning account," and the repeated insistence that login "does not reset mood" and a current flight is "a continuation, not a teleport."
- Server authority over the relationship. Shows up in "server-owned personality, mood, perch choices, weather, and behavioral schedules," "Only the simulation worker updates persisted personality," "Clients never run the domain tick," and "avoid any 'save client snapshot' route."
- Slow, positive personality drift. Shows up in "slow, positive personality drift," "nonnegative additive delta," "an absence never subtracts a trait," "Ignored/refused offers do not become negative input," and "Returning after two weeks is a quiet glance from the same bird, not visible reproach."
- Recognizable individual birds first. Shows up in "engineering priority is recognizable individual birds," "stable identities and individual call signatures," "stable per-bird acoustic fingerprint," "stable bird-specific timings," and "recognizable when the user returns."
- Honest owner presence, not engagement farming. Shows up in the three-signal presence rule, "do not count an unattended window," "no rewards, streaks, or user-visible attendance calculations," "background tabs open overnight produce zero extra credit," and "Do not accelerate adoption because someone visits frequently."
- Privacy boundaries are part of the product, not later compliance. Shows up in "Accessibility and privacy behavior ship with the first public version," "no payloads/PII/vectors in captured logs/RUM," "no pipeline scans birds," and "relationship history" being kept out of analytics.
- The product voice splits naturalist observation from matter-of-fact system surfaces. Shows up in "Naturalist narration," "All system text uses matter-of-fact voice," "Product prompts, observations, and captions use the naturalist voice," and "no 'Welcome back.'"
- Accessibility carries the affective core. Shows up in "keyboard access, visible focus, reduced-motion rendering, and WebAudio failure behavior at initial release," "Accessibility loses the affective core" as a named risk, and reduced-motion birds being "not frozen default silhouettes."
- Immediate interaction feedback without giving the browser authority. Shows up in "Active interaction presentation is a separate server-authored response plan," "subsecond interaction response from the one-minute long-term clock," and "It does not directly update the personality vector."
- Bounded systems, calibrated before claims. Shows up in "initial implementation parameters and acceptance hypotheses," "require calibration," "synthetic histories and qualitative design reviews," "bounded arrays," "bounded voices," and "strict bounded arrays."
- Canonical cross-device convergence. Shows up in "one canonical aviary per account," "all devices pulling a scene see the same active world plans," "same scene_revision," and "No merge UI, last-write-wins vector resolution, client-to-client channel."
- Recovery preserves birds rather than recreating them. Shows up in "Bird UUIDs and persisted personality vectors survive," "Recovery restores stored vectors and accumulators," "Missing vectors are rejected, never reseeded," and "Never 'reset the aviaries' as a recovery strategy."

## Per-feature whys

### 1. Purpose, scope, and delivery rules

- Modern-browser product supporting the last two major versions of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN
- Browser-based aviary with the server advancing birds while nobody is watching: this exists so continuity is real and the browser "renders their current lives rather than starting a simulation on arrival."
- One canonical aviary per account: this preserves "correct continuity across devices" and lets every browser read one canonical state.
- Email magic-link accounts and device sessions: the plan grounds the details in scanner-safe consumption, single-use challenges, opaque cookies, and revocation rather than passwords or local bearer credentials.
- Two system-chosen starter birds: NOT RECOVERABLE FROM PLAN
- Names and renaming: naming is allowed while identity remains stable, because UUIDs and personality vectors must survive naming changes and there is "no replacement-on-rename path."
- Roughly six-species pool with stable identities and individual call signatures: the reason is recognizable birds, including distinct repeated-species fingerprints and a coherent art/audio direction.
- Server-owned personality, mood, perch choices, weather, and schedules: the rationale is that the domain tick and persisted personality remain authoritative, while clients only read projections and interpolate approved trajectories.
- Age-based adoption invitations and seven-bird ceiling: eligibility is independent of attendance or interactions so adoption is not a reward, streak, or engagement mechanic, while seven keeps the scene and projection bounded.
- Honest owner presence accounting: presence requires visibility, focus, and recent activity so opening a tab, playing audio, or visiting someone else does not count as attention.
- Return-greetings: a visible, focused user "deserves a greeting" without a textual welcome, while the first bird notices within the supported loading budget.
- Listen-in: the feature foregrounds an attended bird but "never silences the other birds," preserving the aviary as a living group.
- Seed offer: NOT RECOVERABLE FROM PLAN
- Song-fragment offer: the plan makes fragments a small synthesized motif library so there is no recording upload or library editor, and a bird may respond, pause, or counter-call.
- Still-pool offer: NOT RECOVERABLE FROM PLAN
- Settle and accidental-settle undo: settle creates a quiet evening plan and only a small temporary mood-quieting impulse, while undo handles accidental settling without snapping the world back.
- Responsive horizontal scene with front, middle, and back perch zones: normalized zones let the same server choices map to different viewports while avoiding overlap, cropping, and impossible trajectories.
- Continuous local-time lighting: lighting is anchored to the host timezone so day/night continuity survives devices and visitors see the host's time.
- Rare weather: mild rain and wind add ambient life and brief mood/acoustic effects without live external weather, storms, tasks, or trait decrease.
- Restrained ambient ornaments: ornaments are browser-only, bounded visual details with no persistence or mood effects.
- Sparse, read-only, indefinitely browsable field notebook: the notebook records evidence-based prose sparsely, avoids session/attendance commentary, and keeps old observations accessible.
- Multi-device reads of one canonical state: the why is convergence, with phone and laptop reading identical domain projection bytes at the same revision.
- Revocable device sessions: revocation immediately stops ingestion and pulls for that session, keeping account access controllable.
- Verified email changes: the existing verified address remains authoritative until verification, and changing email must not reseed the account or aviary.
- Export: the authenticated raw JSON download is a data-portability exception that includes vectors while app views, captions, narration, and ordinary APIs hide them.
- Reversible deletion for 30 days followed by complete deletion: recovery preserves UUIDs, vectors, and notebook entries before the deadline, while hard deletion enforces the deletion promise across storage and backups.
- Explicit, revocable read-only visit invitations: invitations allow deliberate sharing while visitors cannot contribute to simulation, mutate state, or gain the host account session.
- Naturalist narration: narration expresses current birds, perches, calls, and light in product voice instead of raw enum labels, vector values, or achievement language.
- Runtime call captions: captions keep calls accessible and truthful to the actual expanded score, especially when audio is muted or unavailable.
- Keyboard access, visible focus, and reduced-motion rendering: these ship in the first version so the accessible path is not a post-launch patch and does not lose the scene's affective core.
- WebAudio failure behavior: missing or blocked audio leaves the aviary visually alive with captions and clear sound status, never blocking the first bird or fetching recorded samples.
- Performance checks, telemetry, privacy boundaries, backup/restore, and rollout: the plan ties release to technical health, privacy, first-bird runtime budgets, recoverability, and gradual ramps rather than visit frequency or optimization metrics.

### 2. Decisions where the specification leaves gaps

- Sixty-second logical simulation tick plus immediate response plans: long-term mood and personality stay tick-owned while greetings and offers do not wait a minute.
- Four-minute presence activity window and 15-second heartbeats: the reason is to evaluate recent trusted activity while avoiding unattended windows.
- Stored aviary timezone: one host timezone prevents a phone from silently moving the laptop's day/night cycle, and visitors use the host's stored timezone.
- Mood set of `wary`, `content`, `curious`, `drowsy`, and `alert`: drowsy covers sleeping/eyes-closed without implying death or a needs state, and night-active species can remain alert.
- Repeated species with stable per-bird fingerprints: repeated species are allowed because the fingerprint distinguishes birds of the same species and there are no rarity weights.
- Four top-bar icons with "offer and settle": this reconciles the layout's four-icon rule with the requirement that settle be reachable from the top bar.
- Keyboard listen-in behavior: focus-based listen-in, Enter, Escape, and suppressed re-engagement support both the focus interaction and the stated Enter affordance.
- Export vectors as a raw JSON exception: the plan resolves the tension between hidden traits and data portability by exposing vectors only through authenticated export.
- Optional visit notice inside account settings: this is the "least intrusive notification channel" and avoids email, push, toasts, badges, or scene interruptions.
- Revocation retaining historical visit logs: "absence" means absence of current access, not erasure of who previously saw the aviary.
- Browser autoplay restrictions: the product renders fully with captions and sound controls until permission exists, without blocking the first bird or adding a recorded fallback.

### 3. Architecture and authority boundaries

- TypeScript modular monolith with separate HTTP, simulation worker, and private job processes: the plan says this reduces distributed write coordination in v1.
- PostgreSQL as authoritative transactional store with DB outbox jobs: queues may transport job IDs but cannot become an independent state authority.
- One primary database authority per aviary: this keeps an aviary from split-brain behavior as capacity expands.
- Module boundaries using typed UUID commands and separate database roles: modules avoid passing raw email or personality into telemetry and do not rely only on code conventions.
- Small TypeScript shell with an SVG scene renderer: the renderer can write transforms and opacity into a bounded tree without rerendering the scene each frame.
- Shared semantic event stream: rendering, audio, captions, and narration use the same facts rather than separate state stores.
- Personalized inline scene from `GET /app`: SSR uses the current pose and timestamp so hydration adds motion without replacing the scene with a new arrival animation.
- No waiting for fonts, settings bundles, notebook history, or AudioWorklet loading: the first bird path stays fast and shows an actual account bird.
- Private no-store personalized HTML and API caching: authorization and canonical version are checked before delivery so stale cache cannot be labeled fresh.
- `sim_revision` and `scene_revision`: the two counters separate domain ticks from visible projection changes such as interaction response plans.
- Persisted immediate response plans: all devices see the same active world plans, and retries cannot invent new greeters or duplicate feedback.

### 4. Data model, durability, and retention

- UUIDs for accounts, aviaries, birds, sessions, events, notebook entries, invitations, visits, exports, and jobs: an account UUID is independent of email, and email is never a sharding key, URL parameter, log identifier, or telemetry dimension.
- Stable `bird` records with species, vector, seeds, call fingerprint, mood, perch, and trajectory: this preserves identity across rename, migration, devices, and asset releases.
- Fixed-point personality storage with residual accumulators and write grants: tiny tick deltas must not round to zero, and only the simulation role can write personality columns.
- Append-only accepted `interaction_event` facts and idempotency keys: corrections become subsequent events, and retries, multiple devices, crashes, and delayed responses process at most once.
- Private notebook evidence references and observation summaries: prose is grounded in facts while evidence stays private and bounded.
- Invitation and visitor capability records: sharing is explicit, digest-backed, revocable, and never feeds drift.
- Private mail outbox with UUID/template references: email addresses are resolved only while delivering so jobs and monitoring do not copy addresses.
- Thirty-day raw interaction retention before compaction: the plan allows diagnosis of private simulation correctness and retry windows without preserving interaction history in analytics.
- Backup and restore from stored vectors and accumulators: recovery never rebuilds personality from interaction logs.
- Missing vector as integrity error: the aviary mutation fails safely because reseeding a bird would break identity and continuity.

### 5. API contracts and authorization

- Same-origin HTTPS JSON APIs, HttpOnly cookies, CSRF/origin validation, and strict schemas: these protect identity credentials and keep bearer credentials out of browser local storage.
- Log exclusions for query strings, request bodies, cookies, email, names, snapshots, and tokens: privacy requires safe operational correlation instead of payload logging.
- Magic-link challenge consumption after explicit sign-in action: this avoids consuming tokens merely because an email scanner fetched the URL.
- Session listing and revocation endpoints: owners can see device labels and revoke devices immediately without location inference beyond display metadata.
- Account settings without counters or traits: the surface updates defaults and timezone without turning traits or attendance into UI.
- Export creation and redemption requiring recent/current verified owner identity: a forwarded export link must not share the account.
- `GET /aviary/state` with ETag and fresh authorization on 304: cached reads still check authorization and resumption pulls fresh canonical state.
- Page activation endpoint: no presence credit or greeting comes from creation or prefetch; only a real visible/focused return can get a server-authored greeting plan.
- Batched event endpoint with UUID, page epoch, and sequence: retries get duplicate receipts instead of duplicate offers, presence, or settle effects.
- Adoption start/complete endpoints: starter allocation and naming are idempotent so concurrent devices cannot create extra starters.
- Adoption opportunity endpoint with no progress bar or countdown: age eligibility is not presented as a reward, next-threshold timer, or visit total.
- Bird name patch endpoint: names are bounded, escaped at rendering, and modify only naming metadata rather than bird state.
- Visit state endpoint checked on every request: revocation, host deletion, expiry, and 304 handling fail closed instead of retaining a host scene indefinitely.

### 6. Precise presence, listen duration, and session lifecycle

- Browser eligibility state machine: visibility, focus, recent trusted pointer/key activity, and open page status must all hold so quiet watching is credited but unattended tabs are not.
- Fifteen-second qualifying interval heartbeats split at signal changes: bounded intervals avoid turning a suspended-clock gap into attention.
- Back/forward-cache and large frame-gap handling: stale epochs are discarded and state is fetched before any new credit.
- Server clipping and 30-second receipt window: intervals are bounded against server-anchored time because browser heartbeats cannot prove human attention.
- Multi-device union accounting: overlapping laptop and phone intervals count once, and simultaneous same-bird listens do not multiply drift.
- Dividing attention across different listened birds: total bird-specific listen credit cannot exceed qualifying owner attention.
- Listen-in end routes: focus changes, Escape, settle, blur, hidden, and page termination close attention so local audio mix is not treated as trusted duration.
- Settle: it immediately terminates the submitting page's interval, creates shared evening presentation, and is not a personality penalty or bonus.
- Five-second undo and later reengagement: accidental settle can reverse smoothly, while later activity deliberately reopens presence only when eligibility holds.

### 7. Server simulation engine

- Deterministic minute offset per aviary and row-locked logical ticks: this avoids minute-boundary spikes and prevents redelivered jobs from creating a second writer.
- Transactional tick step: birds, accumulators, notebook additions, revisions, projection, cursor, and next due time are written together so crashes cannot replay trait deltas.
- Drift function with private capped inputs: presence, listen, and offers have bounded impact and are "not rewards, streaks, or user-visible attendance calculations."
- Nonnegative low-pass trait deltas: changes should be hundredths over weeks rather than observable jumps in one visit, and absence decays input without lowering traits.
- Temporary "quieter after absence" excitation: quieter return behavior comes from a separate bounded signal, not negative drift, illness, or visible reproach.
- Weighted mood model with dwell and hysteresis: birds avoid shuffling states every minute and daily reset means decaying transient bias, not login or midnight reset.
- Perch choices with stored trajectories: clients arriving midway evaluate the correct phase and flights continue rather than teleport.
- Local-time lighting with UTC duration accounting: day/night follows the host while DST and timezone changes do not double-credit time or reset moods suddenly.
- Mild weather scheduling: private server weather adds rain/wind variation without live external weather, storms, alerts, tasks, or vocal-frequency decreases.
- Bird-to-bird coupling: warmth, species-compatible intervals, mood, refractory periods, and chain depth create response without perpetual alarms or a synchronous chorus.
- Return-greeting selection and variation: one bird is chosen by mood/personality-weighted lottery with seeded continuous variation instead of canned clips.
- Catch-up after outages: active aviaries continue through deterministic logical steps, with no client estimates, no reset, and no burst of all missed calls.

### 8. Sync and consistency across devices

- Owner read lifecycle with inline envelope and 15-second visible polling: the scene starts immediately, hidden pages stop rendering/routine polling, and the server tick continues.
- Aligned presentation clock using `performance.now()`: rendering and audio avoid wall-clock jumps, and major corrections fetch fresh state and skip expired events.
- Interpolation of active actions and calls: a current flight is evaluated at phase, completed actions are not replayed, and a seen-ID ring suppresses duplicate calls.
- Conflict and retry policy using event UUIDs and sequences: acceptance order under the aviary lock gives total order, duplicates return original receipts, and changed payloads conflict.
- No long offline queue: stale presence, offers, settle, or greetings should not be replayed later as if still fresh.
- Name/settings resource conflicts: user metadata can use explicit revision checks, but personality never lives in the same editable blob.
- Auth expiry or revocation behavior: private state is cleared and the owner is told to sign in again instead of continuing interactions with stale authority.
- Convergence checks: same `scene_revision` yields identical shared-world projection bytes, with only viewport, audio permission, and local mix differing.

### 9. Frontend scene and interaction rendering

- SVG viewBox scene with bounded rigs and ornaments: compact vector rigs, node ceilings, and no per-frame path regeneration support the seven-bird performance budget.
- Imperative renderer pipeline: the client validates a versioned envelope, maps server slots, evaluates current plans, and feeds audio/captions/narration from the same semantic facts.
- Mood-specific visible behavior: wary, content, curious, drowsy, and alert have distinct posture and motion so birds do not share a generic idle loop.
- Responsive one-screen scene: the aviary has no scrolling, panning, zooming, or bird cropping because the complete logical scene must remain visible and usable.
- Usable hit areas and deterministic overlap resolution: small visual birds at seven on phones still need accessible interaction.
- Returning-account loading without spinner or entry sequence: SSR current pose and a quiet sky field preserve continuity and avoid fake arrival.
- One-time starter arrival flights: this is the onboarding exception to the returning-scene rule, persisted so reloads or second browsers cannot replay the empty aviary.
- Additional adoption opportunities in quiet management surfaces: no badges, unlock messages, catalogs, toasts, counters, or timers keeps adoption from becoming a reward loop.
- Top-bar fade behavior: cursor stillness reduces chrome, but focus, menus, contrast, and keyboard discoverability remain protected.
- Scene bird controls without visible labels or tooltips: the visual scene stays natural while semantic controls expose names/species/action labels to assistive technology.
- Offer/actions popover guidance without countdowns or toasts: cooldown feedback stays quiet and cannot be bypassed by choosing another bird.
- Reduced-motion rendering: OS preference or account opt-in gets cross-faded expressive pose families, not frozen silhouettes or reset greetings.

### 10. Procedural audio and call captions

- Species grammar packages and per-bird fingerprints: calls remain recognizable across variation, mood, personality drift, and repeated species without claiming biological fidelity.
- Deterministic PRNG score expansion: semantic call events become bounded scores with variation, not fixed waveforms or prerecorded loops.
- Server grammar expansion without audio: the server can reserve durations and response opportunities while the client alone produces waveforms.
- Caption generation from the actual score: captions describe real rise/fall, note count, trill, softness, spacing, and context rather than a fixed species label.
- One AudioContext and bounded voice pools: the runtime controls object churn, headroom, clipping, and resource growth.
- Gain/pan buses and chorus limits: the mix preserves individual signatures and space rather than stacking seven full-volume tracks.
- Listen-in ramps with a nonzero floor for other birds: attention brings one bird forward but does not silence the aviary.
- Audio failure and permission handling: blocked or unavailable audio keeps the visual/narrated/captioned scene live, resumes on a permitted gesture, and never fetches samples.
- Caption overlays near the bird: captions are the explicit accessibility exception to the scene's no-label rule and must be collision-aware, bounded, and high contrast.

### 11. Field notebook and accessible prose

- Curated phrase engine for notebook entries: it uses deterministic private semantic facts instead of an external language model, avoiding third-party disclosure and preserving editorial control.
- Sparse notebook cadence and duplicate suppression: ordinary spacing and exception budgets avoid an entry per session and suppress generic "session began" notes.
- Lowercase, present-tense, specific observations: the notebook uses real names, perch/time/weather context, and factual predicates rather than attendance commentary.
- Historical names in notebook entries: stored names at observation time prevent renaming from rewriting the historical observer's record.
- Read-only notebook with indefinite retrieval: owners can browse old observations, visitors cannot access it, and virtualization must not hide history permanently.
- Polite live-region narrator: it turns current projection facts into useful prose without streaming every animation or exposing vector/progress/achievement language.
- Narration priority and coalescing: greetings, offer reactions, and settle can interrupt idle prose while obsolete paragraphs are discarded and forms are not interrupted.
- Narration pause/resume and repeat controls: pausing is a user choice, not a fallback that disables other accessibility.
- Roving-tabindex bird group and arrow navigation: assistive technology sees one bird control per bird rather than hundreds of SVG path nodes.
- Dialog and popover focus behavior: deliberate focus entry, Escape closure, focus return, and traps keep account and action surfaces navigable.
- Matter-of-fact system voice and naturalist product voice: identity, failures, settings, export, deletion, and access use direct language, while prompts, observations, and captions use naturalist voice.
- Contrast, forced-colors, zoom, screen-reader, and touch support: actual composited surfaces are tested so focus and captions remain legible in all lighting states.

### 12. Security, account lifecycle, and privacy enforcement

- Private simulation boundary: analytics, metrics, replay, click tracking, heatmaps, and generative services cannot receive notebook, state, event payloads, vectors, names, or interaction histories.
- Separated identity, simulation, export, and monitoring roles: narrow access prevents broad copying of private state for support or observability.
- Transactional mail exception: mail delivery receives only the necessary recipient and approved template, never bird state or interaction history.
- High-entropy digest-only tokens and ownership checks: invitations and sessions are narrow capabilities, not proof of host account ownership or UUID secrecy.
- Export repeatable-read snapshot: the JSON reflects account settings, current birds/names/vectors/moods, and notebook entries without recomputing personality or including secrets.
- Export redemption by current verified owner: forwarding a link cannot share the account, and pending email change does not redirect export delivery.
- Deletion initiation and recovery: deletion stops simulation and visitor access while preserving canonical records for recovery without treating the pause as negative drift.
- Hard-deletion purge job: all account-linked records, exports, jobs, logs, and unneeded address records are removed idempotently after revoking access.
- Sanitized backup baselines and purge ledger: the backup system must support the deletion promise and prevent restores from resurrecting accounts.

### 13. Performance budgets and operational observability

- Initial JavaScript and critical controller budgets: first paint depends on a tiny critical path while settings, notebook, invitations, and export stay lazy.
- Time to first bird budget: the first visible bird must be an actual named account bird, not a placeholder, and measured on real reference devices and launch geographies.
- Critical transfer and delivery breakdown budgets: the SVG first response, small snapshot, and no blocking fonts/audio protect the 500ms target.
- Snapshot, frame, memory, simulation, and audio budgets: bounded projections, 60fps scene work, no sustained memory growth, tick alarms, and no audio clicks keep the calm experience dependable.
- Real lab profile: actual devices, network shaping, Safari paths, cold server paths, and reproducible test settings prevent budget claims from being synthetic-only.
- Low-powered device degradation: ornament counts and pixel cost may shrink, but birds, captions, and reduced-motion experience cannot be dropped.
- Memory cleanup discipline: buffers, audio nodes, seen-call rings, notebook caches, timers, and contexts are bounded and released.
- Thirty-minute seven-bird soak: a renderer that runs cleanly for one minute does not pass; retained memory and resource counts must plateau.
- Aggregate day-one observability: metrics capture request, tick, first-bird, frame, memory, and audio health without vectors, names, payloads, or raw presence timelines.
- RUM allowlist and client-side prebucketing: coarse categories avoid uploading full URLs, identifiers, input events, invite recipients, or fine-grained personal timelines.
- Operational alerts and ramp criteria: ramps stop on budget regressions, integrity faults, privacy failures, or accessible-path failures, not on engagement metrics.

### 14. Verification and release acceptance

- Failure-mode-oriented tests: verification is organized around presence, drift, persistence, exactly-once processing, mood, interactions, sync, audio, accessibility, privacy/security, notebook, and performance failures rather than snapshots that repeat implementation details.
- Controllable clock, deterministic PRNG, and synthetic event generator: engine correctness is separable from rendering and perceptual review.
- Presence evidence: exhaustive visibility/focus/activity combinations prove only all three credit time.
- Drift evidence: 30/90-day synthetic histories show traits never decrease, one session cannot visibly change them, and week-three differences are reviewed blind.
- Persistence evidence: rename, migration, sign-in, device change, restore, and outage preserve IDs/vectors and reject missing vectors.
- Exactly-once evidence: fault injection around inserts, reservations, trait writes, notebook writes, cursor updates, and commits proves a single final effect.
- Accessibility evidence: manual AT, keyboard, reduced motion, focus, forced-colors, zoom, and contrast reviews complement automated checks.
- Privacy/security evidence: access denials, token replay, revoked sessions/invites, log captures, export ownership, and deletion restore checks enforce boundaries.
- Editorial/UX acceptance: the affective contract explicitly rejects welcome text, days-away surfaces, celebrations, hunger/distress, mood/trait tooltips, visit badges, and wrong voice.

### 15. Delivery sequence and rollout

- Milestone A foundations: schema, permissions, versions, clocks, PRNGs, grammar, prose, tokens, fixtures, and telemetry contracts come before feature growth.
- Milestone B two-bird relationship: the plan delays notebook/social complexity until two birds feel specific, canonical, accessible, and fast.
- Milestone C offers, settle, notebook, and lifecycle: all accepted outcomes must process once through failures, renamed birds retain identity, exports stay private, and deletion restore exercises pass.
- Milestone D six species, age adoption, and quiet visits: repeated-species fingerprints, seven-bird fit, age adoption, read-only visits, and revocation prove the wider product shape without visitor simulation effects.
- Milestone E launch qualification: synthetic suites, manual supported-browser/accessibility review, drift trajectories, tick capacity, rollback, and deletion/restore rehearsal must pass before public access.
- Operational ramp cohorts: account access ramps by deterministic synthetic account UUID hash, with complete accessibility and privacy features in every cohort.
- Rollback strategy: code can roll back with compatible schema readers while retaining bird seeds, IDs, vectors, and ticking on the last safe engine version.

### 16. Main risks and mitigation owners

- Drift risk mitigations: versioned low-pass parameters, caps, synthetic comparisons, and blind review prevent too-fast, too-slow, saturated, or converged birds.
- False presence mitigations: strict three-signal state, bounded monotonic intervals, account union, and suspension tests prevent background tabs and multiple tabs from amplifying drift.
- Lost or double personality delta mitigations: single writer locks, additive fixed-point deltas, idempotent receipts, grants, crash injection, and restore validation protect the relationship.
- Slow interaction acknowledgment mitigations: persisted immediate response plans give greeting and offer feedback from committed state without local fabricated state.
- Audio, accessibility, performance, privacy, invitation, deletion, and UI-drift mitigations: the launch decision requires canonical-state and privacy invariants, runtime budgets, procedural-call quality, and complete accessible surfaces to pass together.
