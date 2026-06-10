## System-level intent

- Design principles are meant to bind engineering, not sit above it as tone guidance. The plan says the five principles are "enforceable engineering constraints, not tone guidance" in section 0, then turns them into lints, schema constraints, CI gates, library absences, and review checklist items.

- "Feels alive, not robotic" is implemented through continuous canonical state and procedural presentation. It appears in section 0 as "Server-side tick is canonical," "first frame renders mid-action," "procedural calls only," and "no spinner anywhere." The same intent recurs in the tick, first-frame, rendering, greeting, and audio sections.

- "Notice, never announce" is a product-voice rule enforced by removing announcement surfaces. Section 0 says no toast/banner/modal components exist; section 5.6 says "The greeting is the entire welcome surface"; section 5.8 says the visiting bird has "No modal, no announcement, no badge"; section 16 calls announcement-surface creep a risk to "the product's identity."

- "Charm from specificity" depends on shared, concrete, non-repeating prose. Section 0 points to `@aviary/voice`, "template review process," and "no-repeat memory"; section 5.9 requires "lowercase, present tense, bird-named, concrete slot fillers"; section 14 tests repetition and voice lint.

- "Restraint over richness" is treated as an engine and schema property. Section 0 names the bird cap and scene constraints as "engine constants and schema CHECK constraints"; section 1.2 says out-of-scope items are "enforced, not just omitted"; section 15.2 says changing the cap is "a calibration project, not a config flip."

- The plan preserves two voice registers: "naturalist" and "matter-of-fact." Section 0 assigns surfaces to two copy registries and forbids cross-register imports; Appendix B says new surfaces that engage "the system as a system" use matter-of-fact, while "everything else" uses naturalist.

- Vocabulary is part of the product contract. Section 0 requires code, schemas, APIs, and metrics to use PRD vocabulary exactly, because "vocabulary drift in code is how vocabulary drift reaches the UI."

- The core architecture is "server decides what, client decides how." Section 2.3 calls this the "central architectural contract": future-changing semantic state is server-canonical, while frame-by-frame cosmetic state is client-presentation.

- Correctness is preferred over distributed cleverness. Section 2.1 chooses a modular monolith because tick correctness depends on "transactional access to events + bird state"; section 6.1 makes "single writer" an invariant; Decision 22 chooses "sync correctness over geo-latency."

- Determinism is a recurring design philosophy. Section 5.1 seeds all tick randomness from `(aviary.rng_seed, tick_no)` so re-running a tick is identical; section 5.7 uses a shared pure function for offers; section 14.2 requires fast-forward equivalence to be "bit-identical."

- Privacy is implemented as architecture and absence, not policy. Section 3.4 says absences are "the cheap way to keep section 1.2's refusals refused"; section 10.4 says retention "converts the privacy stance into physics"; section 10.5 calls the telemetry boundary "architecture, not policy."

- Attention is an "honest signal," not a metric to farm. Section 5.3 defines presence as a three-condition conjunction, section 5.4 caps drift and says drift-farming is "structurally bounded," and section 10.6 says an attack "harms only the attacker's own aviary" because no leaderboards exist.

- Absence must never read as punishment. Section 5.4 says "Traits never decrease, ever"; neglect contributes zero; quieting lives in a separate "expression rhythm" that has "zero trait penalty." Section 10.2 says restore resumes exactly and "absence was never punished anyway."

- Accessibility is product integrity, not compliance afterthought. Section 1.1 says accessibility ships "with v1, not after"; section 9 says these are "designed surfaces, not retrofits"; section 15.1 puts narration and keyboard focus into the alive vertical slice.

- Performance budgets are part of preserving the experience. Section 12 gives bundle, first-bird, frame-time, and memory-growth budgets; section 14 turns them into CI gates; section 16 names perf erosion as a risk because mid-tier users would lose 60fps and budget.

- Observability is allowed only where it protects operations without becoming engagement analytics. Section 13 uses server metrics, synthetic fleet checks, and aggregate-only RUM; section 10.5 deliberately omits per-feature engagement, retention cohorts, offer/listen-in usage analytics, and notebook read rates.

## Per-feature whys

### Scope

- Web client targeting the last two major versions of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: Why: section 10.1 makes the session list the "audit surface," especially because there are "No new-device security emails at v1."

- Email change with verification: Why: section 10.1 says "verify-new-then-commit" and "old address works until confirmation," grounding the feature in account continuity and avoiding premature lockout.

- Account export: Why: section 10.3 frames export as "a data-portability artifact engaged with as a system surface."

- Soft-then-hard deletion: Why: section 10.2 freezes the account so restoration "resumes exactly where things stood" and then hard-deletes every account-keyed record; the plan also documents the physical erasure bound.

- One aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN

- Bird cap of seven: Why: section 0 ties it to "Restraint over richness"; section 15.2 says future change is "a calibration project, not a config flip."

- New-bird offers gated purely on aviary age: Why: section 5.8 says "No other input - not visits, not interactions, not anything - reaches this check," making growth auditable and avoiding interaction-driven gamification.

- Server-authoritative simulation: Why: section 2.3 says state that would change a bird's future is server state; section 5.1 says the row lock serializes ticks and creates the "single-writer invariant."

- Single-screen horizontal scene with three perch zones: NOT RECOVERABLE FROM PLAN

- Day/night by user local time: Why: section 5.5 uses account timezone for time-of-day priors, while section 6.3 says `tod_phase` comes from the server so a wrong device clock still shows the right aviary.

- Ambient weather and micro-motion: Why: weather and motion are part of aliveness; section 5.5.1 makes weather canonical so host and visitor see the same rain, and section 7.3 says users read mood from motion rather than labels.

- Top bar with account/settings, accessibility, notebook, and offer: Why: section 7.5 keeps the bar to "four items only," fading it so controls do not dominate the scene while still returning on input.

- Presence accounting: Why: section 5.3 calls presence "the honest signal," requiring visibility, focus, and recent input so a tab being open never counts as attention.

- Return-greeting: Why: section 5.6 calls it "the anchor moment" and says the greeting is the entire welcome surface, with no toast, banner, modal, or text.

- Listen-in: Why: section 8.4 focuses the chosen bird by bringing it forward in the mix while keeping others in an ambient bed, so focus does not become muting.

- Offers of seed, song fragment, and still pool: Why: section 5.7 uses deterministic shared resolution so the client gives "an immediate reaction" and the server records the same outcome for drift.

- Settle with 5s undo: Why: section 5.3 says settle ends presence cleanly with "No penalty, recovery surface, or notification"; section 8.5 makes it a quieting and lighting shift rather than a hard stop.

- Field notebook: Why: section 5.9 gives sparse observations that "observe the aviary, never the user," preserving naturalist specificity without turning behavior into a report card.

- Client-side procedural WebAudio synthesis: Why: section 8.1 and 8.2 avoid audio assets and recorded fallback; section 16 says synthetic calls are the "affective spine."

- Per-bird stable call signatures: Why: section 8.2 makes signature parameters independent of mood and traits so "Pip sounds like Pip across drift and mood."

- Chorus mixing: Why: section 8.3 creates call-and-response and chorus windows while anti-unison and pitch-collision rules keep birds distinct.

- Graceful-silence fallback with captions defaulted on: Why: section 8.6 says this is the fallback when WebAudio is unavailable and that no recorded-audio fallback path exists.

- Screen-reader narration: Why: section 9.1 gives screen-reader users the same product through naturalist prose, not a state list, and uses the same grammar library as the notebook.

- Reduced-motion mode: Why: section 9.4 says it is "a second renderer mode, not a kill-switch," so "The aviary is the aviary" with motion swapped for calm cross-fades.

- Runtime-generated call captions: Why: section 9.2 renders from the motif trace "from what was actually synthesized," so captions mirror the real call rather than a generic label.

- Full keyboard navigation: Why: section 9.3 gives a full session path without pointer input, and section 14.8 tests "full session start-to-settle without a pointer."

- WCAG AA copy contrast: Why: section 9.5 requires all user copy to meet AA across palette states; captions use plates so scene brightness does not break readability.

- Visits as per-invite email-link read-only ambient view: Why: section 11 says visitors get the same snapshot pipeline but render-only, no "visitor flattering" mode, and their watching "drifts nothing."

- Visit expiry, revocation, log, and notifications off by default: Why: section 11 keeps visits pull-only and quiet: revocation takes effect on next pull, notifications are off by default, and there is "no badge anywhere."

- Synthetic account UUIDs everywhere: Why: section 3 says the synthetic-ID rule is absolute; email exists only in encrypted fields, keeping account identity out of logs, payloads, and tooling.

- Per-bird interaction data never leaves the simulation boundary: Why: section 10.5 separates the simulation database from telemetry and denies bird, trait, mood, offer, listen, and presence fields in metric/log schemas.

- Aggregate-only operational telemetry: Why: section 13 and section 10.5 allow metrics that protect budgets and operations while deliberately not measuring "how are birds typically interacted with."

- Time-to-first-bird, 60fps idle, and zero memory growth budgets: Why: section 12 treats these as product experience constraints; section 16 says erosion would make mid-tier users lose 60fps and budget.

### System architecture

- API service and tick worker pool deployed from one image: Why: section 2.1 separates HTTP latency from simulation work while keeping one codebase and one canonical Postgres state.

- PostgreSQL as canonical state: Why: section 2.1 says tick correctness depends on transactional access to events and bird state, "exactly what a single Postgres gives us for free."

- No Redis at v1: Why: Decision 19 says Postgres-only infrastructure has "fewer moving parts" and "the tick's transactionality is the point."

- Object storage and transactional email provider: NOT RECOVERABLE FROM PLAN

- CDN for app shell and species packs: Why: section 2.4 and section 12.2 place the shell and packs at the edge to support first-frame performance, while the authenticated API is not edge-cached.

- Modular monolith: Why: section 2.1 says the tick is the only computationally interesting workload and module boundaries make a later split "mechanical, not archaeological."

- TypeScript end-to-end: Why: section 2.1 says call grammar, deterministic reaction functions, and voice library must run identically on server and client; otherwise they would be implemented twice and kept bit-identical.

- `@aviary/engine-shared`: Why: section 2.2 lets the server decide canonical outcomes and the client render the same outcome without a round-trip.

- `@aviary/voice`: Why: section 2.2 makes the notebook, narration, captions, and system copy share voice so "a screen-reader user hears the same product the notebook writes."

- `@aviary/schemas`: Why: section 2.2 makes schemas "the single source of truth for the render-pipeline boundary contract."

- Render-pipeline boundary: Why: section 2.3 encodes the rule that future-changing state is server state, while frame-only state is client state.

### Data model

- UUIDv7 primary keys: Why: section 3 calls them "time-ordered, index-friendly."

- Encrypted email plus HMAC lookup: Why: section 3.1 stores email in encrypted form and uses HMAC for lookup because it is "not reversible."

- Magic-link tokens and session tokens stored as hashes: Why: section 3.1 and section 10.1 store hashes so the raw tokens are not persisted and consumption can be replay-safe.

- Stable bird identity with renameable names: Why: section 3.2 says `species_id` and bird id are stable identity and never reused, while the name is renameable and "identity unaffected."

- Server-only personality vector columns: Why: section 3.2 says they are "NEVER serialized into snapshot_json," which supports the never-expose rule.

- Non-prod monotonic drift trigger: Why: section 3.2 calls it "a tripwire for bugs," while removing it in prod for write cost.

- Append-only `interaction_events`: Why: section 3.3 calls it "the ONLY upward data path," supporting additive deltas, ordering, idempotency, and retention.

- Monthly event partitions dropped after 30 days: Why: section 10.4 says raw history older than a month should not exist, so it cannot leak or tempt aggregate analysis.

- Read-only `notebook_entries`: Why: section 5.9 says no mutation endpoint exists; entries are sparse observations, not an editable journal or engagement surface.

- `visit_sessions` with approximate duration: Why: section 3.3 says duration is for the host's log and "not telemetry."

- `bird_trait_log`: Why: section 3.3 limits it to calibration/debug audit, same privacy class as simulation DB, and section 10.5 limits human querying to synthetic or consented internal accounts.

- Static code-shipped species catalog with six species: NOT RECOVERABLE FROM PLAN

- No visit counts, stats tables, engagement rollups, or per-account interaction aggregates: Why: section 3.4 says not modeling them keeps refusals refused because features needing those tables require a reviewed migration.

### API surface

- JSON over HTTPS with secure cookies, CSRF, rate limits, ETags, and brotli: Why: the conventions combine security, mutation safety, and efficient polling; ETag appears in snapshot and notebook to return 304 when unchanged.

- Account-scoped endpoints deriving aviary from session: Why: section 4 says aviary ids never appear in editable URLs, reducing object-access risk.

- Magic-link request returns 202 always: Why: section 4.1 and 10.1 say this avoids an "account-existence oracle."

- Bird rename endpoint only: Why: section 4.1 says "rename only; nothing else is client-writable," preserving server-canonical state and keeping click-on-bird reserved for listen-in.

- Snapshot polling with ETag and visibility/long-gap triggers: Why: section 4.2 keeps clients current after visibility changes, suspend, and bfcache while Decision 3 says the tick cadence makes push latency pointless.

- Snapshot target under 8KB brotli: NOT RECOVERABLE FROM PLAN

- Trait quantization to levels 1-7: Why: section 4.3 says raw traits must never ship, "not to the client, not to logs, not to telemetry"; levels survive devtools and create the user-visible drift threshold.

- Batched idempotent event POSTs: Why: section 4.4 allows retry with backoff and `sendBeacon` while server idempotency makes retries safe.

- Server-side event ordering by `seq` and `received_at`: Why: section 4.4 and 6.1 say client timestamps are advisory only, preventing client-clock ordering errors.

- Per-session sanity clamps on events: Why: section 4.4 and 10.6 reject implausible volumes, keeping presence inflation bounded.

- Notebook cursor pagination and indefinite scrollback: Why: section 5.9 says notebook entries are retained for account lifetime and in export; the plan gives no separate rationale for infinite scrollback beyond that persistence.

- Visitor router with exactly three routes and separate cookie scope: Why: section 4.5 and section 11 enforce read-only by scope, not by UI.

### Simulation engine

- Per-minute tick cadence: Why: section 5.1 makes the server-side tick canonical and supports the "alive" contract; latency alarms protect the PRD tick requirement.

- Row-lock claim with `FOR UPDATE SKIP LOCKED`: Why: section 5.1 says the row lock serializes ticks per aviary, making single-writer a property of the claim.

- Tick transaction ordering: Why: section 5.1 applies events, drift, mood, weather, notebook, bird offers, and snapshot rebuild atomically so state and processed events move together.

- PCG32 deterministic tick randomness: Why: section 5.1 says re-running from the same inputs yields identical output, supporting fast-forward equivalence and replayable bug reports.

- Dormancy fast-forward: Why: section 5.1 keeps inactive aviaries scalable while making behavior "indistinguishable from a literal per-minute tick."

- Synchronous catch-up before snapshot: Why: section 5.1 says the user always sees "the aviary that kept running."

- Mood set and roosting as behavior, not mood: Why: section 5.2 says night is not dead and the nightjar activity profile keeps a species calling late.

- Presence sampler with visibility, focus, and recent input: Why: section 5.3 uses the conjunction as the honest signal and says there is no "tab open" path to presence.

- Touch-device input adaptation: Why: section 5.3 and Decision 4 translate the desktop pointermove/keypress baseline "honestly to coarse pointers" rather than making touch stricter than intended.

- Multi-device interval union for presence: Why: section 5.3 and Decision 15 say watching on two screens is one attention, not double drift.

- Low-pass drift application: Why: section 5.4 says a burst session surfaces over following days and "no single session moves a trait visibly."

- Daily drift cap: Why: section 5.4 bounds marathon usage and drift-farming so it is structurally limited.

- Drift calibration targets: Why: section 5.4 uses instrument thresholds and quantization so regular use is measurable within week 1 but visibly changes over roughly three weeks.

- Monotonic personality drift: Why: section 5.4 says neglect contributes zero and traits never decrease, satisfying the no-punishment stance.

- Expression rhythm outside personality vector: Why: section 5.4 reconciles quieter returns after absence with fully intact personality, so it can never read as punishment or decay.

- Mood machine with hysteresis and dwell: Why: section 5.5 says mood should read as weather, not noise, with no flapping.

- Personality affinity in mood scores: Why: section 5.5 implements examples like a bold bird being resistant to wary as a multiplier, not a special case.

- Bird-to-bird mood propagation: Why: section 5.5 creates spatially local neighbor effects such as alarm calls spreading to same or adjacent perch zones.

- Daily-ish mood reset: Why: section 5.5 gives mood a daily cadence without a visible snap.

- Server-side weather process: Why: section 5.5.1 makes weather canonical so a host and simultaneous visitor see the same rain; it is "Never assertive."

- Return-greeting decided at snapshot-serve time: Why: section 5.6 says it must reflect the actual return instant rather than the last tick.

- Return-greeter selection by traits, mood, and rhythm: Why: section 5.6 lets stable and day-specific behavior fall out of the math instead of special-casing it.

- Greeting intensity tiers by absence: Why: section 5.6 shapes the anchor moment according to how long the user has been gone without adding an announcement surface.

- Cached greeting directive per presence gap: Why: section 5.6 makes laptop and phone openings within the same return see the same bird greet.

- Offer resolution shared by client and server: Why: section 5.7 gives immediate local reaction and identical server drift resolution with zero round-trip latency.

- Per-bird offer cooldown without countdown timer: Why: section 5.7 keeps cooldown from becoming a meter; the affordance "just isn't ready yet."

- Adoption as "the birds that arrived" with no catalog: Why: section 5.8 gives the rationale implicitly in the quoted phrase and in the no-announcement posture; it avoids a catalog surface. 

- Visiting-bird new-offer pattern: Why: section 5.8 says it is "the one design that grows the aviary without announcing anything" and lets the user discover by watching.

- Notebook detectors and salience: Why: section 5.9 captures specific aviary observations such as first-greeter changes, weather moments, and new birds.

- Notebook sparsity governor: Why: section 5.9 preserves sparse charm by construction; active users do not get more entries.

- Notebook voice lint denylist: Why: section 5.9 enforces the hard line that entries observe the aviary, never the user.

### Sync model and consistency

- Single writer invariant: Why: section 6.1 prevents API paths from writing personality, mood, behavior, weather, or rhythm and makes import-boundary lint enforce it.

- Additive deltas only: Why: section 6.1 says last-write-wins is unrepresentable because clients submit events, never trait values.

- Total server order: Why: section 6.1 prevents client clocks from determining state and ensures interleaved laptop/phone events both land.

- Event idempotence: Why: section 6.1 makes retries safe and prevents tick reprocessing in the same transaction that advances `tick_no`.

- No client-to-client merge path: Why: section 6.2 keeps multi-device behavior bounded by snapshot polling and cosmetic-only divergence.

- Client retry queue and sendBeacon: Why: section 6.3 says lost events lose at most marginal drift and never corrupt state.

- Rendering from last snapshot on fetch failure: Why: section 6.3 lets presentation remain self-sufficient for minutes, with a matter-of-fact reload surface only after 90s.

- Snapshot reconciliation after suspend: Why: section 6.3 says birds cross-fade or move to canonical positions and never teleport.

- Server time as only state clock: Why: section 6.3 says a wrong device clock still sees the right aviary.

### Frontend architecture and rendering pipeline

- Custom Canvas 2D renderer: Why: section 7.1 and Decision 2 cite bundle and control; the scene is small enough for Canvas 2D on the target laptop.

- Preact for chrome only: Why: section 7.1 keeps the scene out of the UI framework and code-splits panels.

- Service worker precache and last-snapshot cache: Why: section 7.1 and 7.4 support instant warm starts; clearing on sign-out/revocation is shared-computer hygiene.

- Layered scene composition: Why: section 7.2 provides cached compositing and accessibility-controlled captions/top bar while keeping the scene as one visual field.

- Gentle parallax with no camera controls: Why: section 7.2 keeps motion autonomous and gentle; no panning, scrolling, or zooming exists in the input model.

- Responsive perch anchors and viewport assertions: Why: section 7.2 prevents narrow phones from cropping a bird.

- Palette interpolation by local time: Why: section 7.2 carries day/night mood through scene color without user-visible clocks.

- Behavior layer scheduling local cosmetic bouts: Why: section 7.3 allows visible aliveness between snapshots while preserving that the client cannot originate semantic change.

- Motion layer as mood carrier: Why: section 7.3 says the user reads mood from motion, so there is no label, tooltip, or status icon.

- Parts-based pose solver and cached sprites: Why: section 7.3 links plumage levels to visual richness while limiting rerasterization for performance.

- Snapshot interpolator: Why: section 7.3 prevents teleports by turning canonical changes into planned transitions or reduced-motion cross-fades.

- Hidden-tab render stop: Why: section 7.3 says rendering is wasted when hidden, presence is false, and server simulation continues.

- Quiet field loading state: Why: section 7.4 makes waiting a designed aviary state with no spinner, progress bar, skeleton, or fade-from-static.

- Birds drawn mid-action on first frame: Why: section 7.4 supports "Feels alive, not robotic" and avoids an entry animation except the adoption moment.

- Top bar fade behavior: Why: section 7.5 reduces chrome intrusion but suspends fade for focus, screen reader, or keyboard modality so focused controls remain usable.

- Pointer/touch input model: Why: section 7.5 keeps bird click/tap mapped to listen-in and avoids hidden gestures.

### Audio pipeline

- One AudioContext with pooled per-bird voices: Why: section 8.1 says voices are reused forever with "zero per-call allocation," matching the memory rule.

- Procedural synth graph and generated reverb: Why: section 8.1 and 8.6 avoid audio assets and recorded fallback paths.

- Scheduler lookahead without AudioWorklet: Why: section 8.1 says plain nodes suffice for the voice count and reduce Safari risk.

- Species motif libraries as code: Why: section 8.2 keeps calls parametric and synthesizable rather than recorded clips.

- Immutable bird signature seed: Why: section 8.2 guarantees identity continuity in sound across drift and mood.

- Mood and trait modulation of calls: Why: section 8.2 lets calls express state while keeping the signature stable.

- Weighted stochastic grammar with jitter: Why: section 8.2 ensures no two calls are bit-identical and supports repetition detection.

- Structured motif trace: Why: section 8.2 makes synth, captions, and narration share one source of truth.

- Bird-to-bird chorus scheduler: Why: section 8.3 produces call-and-response and emergent chorus windows.

- Anti-unison and pitch-collision avoidance: Why: section 8.3 keeps simultaneous birds distinct and avoids phase-cancellation artifacts.

- Listen-in ramp floor: Why: section 8.4 ensures other birds are never muted and a "channel-switcher feel" cannot be introduced casually.

- Settle and night audio ramps: Why: section 8.5 makes quieting gradual and aligned with lighting and time-of-day curves.

- Hidden-tab audio fade and suspend: Why: section 8.5 says presence has ended, the aviary lives where the user visits it, and reliable hidden scheduling is expensive.

- Autoplay audio-waiting glyph: Why: section 8.6 accepts browser limits while avoiding banners, modals, or "click to enable sound" prompts.

- WebAudio-unavailable silence with captions: Why: section 8.6 keeps the experience accessible and enforces that recorded-audio fallback does not exist.

### Accessibility surfaces

- Dedicated accessibility-focused engineer from M1: Why: section 9 says the surfaces are "designed surfaces, not retrofits."

- Client-side narration composer: Why: section 9.1 says narration must describe the actually rendered scene, including local cosmetic bouts.

- Polite ARIA live region and idle cadence: Why: section 9.1 avoids assertive interruption, queue flooding, and state-list narration.

- Priority narration for greeting, offers, settle, visiting bird: Why: section 9.1 gives important scene moments promptly while replacing pending idle updates.

- Caption rendering near calling bird: Why: section 9.2 connects captions to what was actually synthesized and keeps them spatially tied to the caller.

- Caption collision avoidance and contrast plate: Why: section 9.2 preserves readability during simultaneous calls and palette changes.

- Keyboard roving focus over birds: Why: section 9.3 creates in-scene keyboard access while keeping birds ordered front-to-back then left-to-right.

- Offer panel focus trap and return focus: Why: section 9.3 makes the panel fully keyboard-operable.

- Keyboard-accessible settle undo: Why: section 9.3 makes the 5s undo work without pointer use.

- Focus indicator across palette states: Why: section 9.3 ensures focus is visible under dawn, midday, dusk, night, and settled palettes.

- Reduced-motion held poses and cross-fades: Why: section 9.4 preserves the same calls, captions, narration, drift, mood, and notebook while reducing motion.

- Automated and manual accessibility test matrix: Why: section 14.8 catches narration drift, reduced-motion regressions, and screen-reader/browser issues before release.

### Accounts, privacy, security

- Magic-link token length, hash storage, expiry, and row-lock consumption: Why: section 10.1 makes login replay-safe and time-bounded.

- Rate limits per email-HMAC and IP: Why: section 10.1 protects magic-link requests without exposing raw email.

- No passwords and no SSO at v1: NOT RECOVERABLE FROM PLAN

- No new-device security emails at v1: Why: section 10.1 says the session list is the audit surface and the no-email posture should stay clean.

- Soft-deleted accounts pause simulation: Why: section 10.2 says restore resumes exactly where things stood and absence was never punished.

- Hard deletion of account-keyed records and backup aging: Why: section 10.2 gives a documented physical erasure bound.

- Export includes raw personality vectors: Why: section 10.3 says the source explicitly enumerates them, treats them as data portability, and keeps them out of product UI.

- Raw event retention of 30 days: Why: section 10.4 says drift is recursive and mood short-window, so older raw interaction history is unnecessary and privacy is enforced physically.

- Separate ops pipeline and simulation database: Why: section 10.5 prevents simulation data from entering analytics, dashboards, or ML systems.

- Telemetry schema registry denylist: Why: section 10.5 makes privacy build-enforced by rejecting metrics/logs with bird, trait, mood, offer, listen, presence, or email fields.

- No engagement analytics or A/B tests on engagement: Why: section 10.5 says production users' simulation data drives their simulation, "Full stop."

- Calibration via synthetic cohorts and consented internal accounts: Why: section 10.5 allows tuning targets without population-level interaction analysis.

- Abuse integrity for presence inflation: Why: section 10.6 clamps wall-clock presence and caps drift so the attack buys little and has no social payoff.

- Invite spam controls: Why: section 10.6 rate-limits and dedups invites, with standard email-abuse reporting headers.

- Strict CSP and self-hosted assets: Why: section 10.6 says self-hosted everything is also a privacy stance.

- External security review in M4: Why: section 10.6 targets auth, visit-token scope isolation, and IDOR sweeps.

### Visits

- One-time visit link with persistent visitor cookie: Why: Decision 12 reads "one-time link" as anti-sharing, not single-viewing, while preserving instant revocation.

- Visitor snapshot minus greeting, cooldowns, and absence data: Why: section 11 gives visitors the ambient view without host-specific return or interaction affordances.

- No visitor flattering mode: Why: section 11 says same renderer is the implementation of "no-show-off-mode."

- Render-only visitor client: Why: section 11 prevents visitor presence, offers, settle, notebook, or listen-in from writing events or drifting birds.

- Revocation returns a matter-of-fact surface: Why: section 11 cuts visitor access within the poll/heartbeat window without adding notifications.

- Visit log in settings: Why: section 11 makes visits visible to the host only as a pull-only settings surface, with no badge or onboarding notification.

- Not built: chat, visitor avatars, comments, co-presence, discovery, visit counts outside host log: Why: section 11 keeps social surfaces beyond the quiet visit affordance structurally absent.

### Performance engineering

- Critical path under 250KB and full initial bundle fail under 1.5MB: Why: section 12.1 says headroom under the 2MB contract is "the defense against erosion."

- Lazy chunks for settings, notebook, offers, invite, export, deletion: Why: section 12.1 keeps the first-bird critical path small.

- No font payload and no audio assets: Why: section 12.1 allocates zero font bytes and no audio assets by definition, supporting bundle budget and procedural audio.

- Warm-path first-bird under 500ms: Why: section 12.2 calls this "the product's actual life" because the service worker can serve shell and cached snapshot locally.

- Cold-load quiet field with 2.0s budget: Why: section 12.2 says 4G plus authenticated state cannot beat physics, so quiet field covers the gap honestly.

- RUM segmented warm/cold: Why: section 12.2 measures both paths separately from day one.

- Offscreen caching, dirty compositing, pooled objects, and particle cap: Why: section 12.3 preserves 60fps on the target laptop.

- Ambient degradation before bird motion: Why: section 12.3 protects the birds as the most important motion when frames get long.

- 30-minute memory soak: Why: section 12.4 enforces zero memory growth and catches leaks under idle, listen-in, offers, and notebook scrolling.

- Browser feature detection with unsupported surface: Why: section 12.5 gates only required features; WebAudio absence alone leads to silence fallback, not unsupported browser.

### Observability, testing, rollout, calibration

- Server SLO metrics: Why: section 13 protects snapshot, event, tick, email, and DB health, with tick p99 over 5s as the PRD alarm.

- Synthetic fleet every 15 minutes: Why: section 13 alerts on budget breach "before users feel it."

- Aggregate-only RUM: Why: section 13 measures first-bird, long tasks, FPS, audio init, cache hits, and JS errors without per-account dimensions.

- Product, simulation, and delivery dashboards: Why: section 13 separates budget health, tick health, and email health while omitting the never-measured list.

- Engine property tests and golden simulation personas: Why: section 14 asserts monotonic drift, caps, presence union, offer determinism, mood stability, and calibration windows.

- Voice lint: Why: section 14.4 build-fails copy that breaks lowercase, present tense, no second person, no exclamation, denylist, system registry, or identifier vocabulary.

- Audio tests and listening panels: Why: section 14.5 combines automated signature/repetition/ramp checks with human gates because "uncanniness is not unit-testable."

- Privacy CI: Why: section 14.6 asserts no per-account-keyed analytics records and enforces import boundaries and retention drops.

- Release checklist questions: Why: section 14.10 keeps announcement surfaces, copy bypasses, metric creep, and wait states under human review.

- M0 gates from first commit: Why: section 15.1 says gates are cheapest before there is anything to grandfather.

- M1 alive vertical slice: Why: section 15.1 makes aliveness, narration, keyboard focus, and first listening session part of the first real experience.

- M2 interactions and voice surfaces: Why: section 15.1 completes offers, settle, notebook, captions, narration, reduced motion, species, and motifs before internal cohort expansion.

- M3 sync hardening and visits: Why: section 15.1 makes two-device behavior indistinguishable and calibration within target before broader release.

- M4 hardening: Why: section 15.1 requires perf, memory, chorus recognizability, a11y, security, privacy audit, and failure drills before beta.

- M5 private beta to GA: Why: section 15.1 waits for SLOs to hold for two weeks at beta scale and synthetic fleet to be green.

- Bird-count ramping: Why: section 15.2 uses the 90-day third-bird runway to test 3-7 bird choruses before organic users reach them.

- Silent pause flag for new-bird offers: Why: section 15.2 lets mix work take more time without announcing a pause.

- Instrumented from day one: Why: section 15.3 monitors budgets and aggregate sanity signals while preserving the never-instrumented boundary.

- Prospective-only drift retunes: Why: section 15.4 says identity continuity includes drift history, so existing trait values are never rescaled or migrated.

- Mood and grammar tuning freer than personality-path changes: Why: section 15.4 distinguishes fast-timescale, non-accumulated tuning from accumulated personality state that deserves migration gravity.
