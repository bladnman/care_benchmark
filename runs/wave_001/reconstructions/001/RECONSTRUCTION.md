## System-level intent

- **Affective value before utility.** The executive summary says Pocket Aviary's "entire value is affective": birds should "feel alive" through honestly measured presence, procedural expression, and no announcing. This shows up again in the risks, where audio is the "affective spine" and miscalibrated drift risks either a "Tamagotchi feel" or "screensaver feel."

- **The server is the only writer of canonical state.** The plan's central rule is explicit: "clients emit events and render snapshots." This appears in the architecture, snapshot contract, API event schema, sync model, offer reactions, greetings, and behavior projection. The why is to make multi-device sync architectural and make "last-write-wins data loss structurally unreachable."

- **The aviary has been running.** The simulation service uses a Δt-parameterized `advance()` function, lazy as-of-now evaluation, and edge-inlined first-frame state so scheduler hiccups or loading do not corrupt "the fiction that 'the aviary has been running.'" The first painted frame is birds "mid-action"; "no spinner" and no entry animation are enforced by the charm-guard suite.

- **Two clocks, not punishment.** The plan separates "slow personality drift" from "fast mood/recency dynamics." Personality drift is monotonic, additive, and non-negative; absence changes expression through a recency envelope so birds become "quieter, not mistrustful."

- **Presence must be honestly measured.** Presence is called the "dominant drift input," so the client detector requires visibility, focus, and recent input, while the server adjudicates windows from received pings, dedupes devices, clamps daily credit, and biases beacon loss toward under-counting, "the honest direction."

- **Procedural, recognizable, never identical.** The plan repeatedly bans canned/looping outputs: audio is WebAudio and "zero recorded audio"; call realization varies every time while per-bird signatures keep a bird recognizable "by ear"; greetings have seeds, stagger offsets, and anti-repeat history.

- **Accessibility is a designed surface.** The key pillars say accessibility ships in v1 through "server-generated naturalist narration prose," reduced motion as "its own cross-fade rendering register," and runtime captions. Later sections reject cheap state lists, blanket `animation: none`, and retrofitted a11y.

- **Privacy is architecture.** The plan uses "synthetic UUIDs everywhere," encrypted email columns, no analytics access to simulation rows, aggregate-only telemetry, and synthetic staging calibration. It repeatedly says the enforcement point is schema, role, pipeline, and CI rather than "team discipline."

- **Restraint against engagement loops and gamification.** The scope and charm-guard sections ban achievements, streaks, counters, welcome surfaces, notifications, toasts, badges, leaderboards, and user-behavior notebook entries. The line is: "observations of the aviary are allowed; observations of the user's behavior are never surfaced."

- **Two product voices.** The plan maps aviary scene, notebook, narration, captions, offers, adoption, and greeting to "Naturalist" voice: lowercase, present-tense, specific, no "you," no exclamation. Identity, errors, settings, export, deletion, and visit surfaces use "Matter-of-fact" copy that states what happened and what to do.

## Per-feature whys

### Accounts

- **Email magic-link auth.** The plan frames the mechanics as auth hardening: 256-bit tokens, hashed at rest, 15-minute TTL, single-use transactions, per-email rate limits, and uniform 200 responses so there is "no account enumeration."

- **Per-device revocable sessions.** The rationale is account control and hardening: sessions are listed with matter-of-fact device labels and can be revoked; revoked sessions are rejected for events.

- **Email change with verification.** NOT RECOVERABLE FROM PLAN

- **Account export.** The rationale is data portability. Appendix A says export may include personality vectors because it is a "private data-portability artifact the user triggers on themselves," not a product surface.

- **Soft-delete, restore, hard delete, and crypto-shred.** The rationale is the privacy commitment that "interaction history is the user's, not ours." The 30-day window allows "I changed my mind"; hard delete purges keyed rows, invalidates snapshots, revokes sessions, and crypto-shreds the per-account key so backups age out unreadable.

### Aviary

- **One aviary per account.** NOT RECOVERABLE FROM PLAN

- **Two starter birds.** The starter pair uses contrasting greeting styles so the first session already demonstrates "one bird notices you."

- **Cap of seven birds.** The rationale is recognizability and performance: the audio section says stable signatures are load-bearing and "also why 7 is the cap"; rollout says chorus load, call-plan density, and projection cost grow gradually.

- **User-assigned renameable names.** NOT RECOVERABLE FROM PLAN

- **Stable bird identity forever.** The rationale is long-term recognizability: bird ids are "STABLE FOREVER," per-bird audio signatures are stable across moods and drift, and "a two-week user knows Pip by ear."

- **Age-gated adoption offers.** The rationale is pacing without gamification: new birds are keyed "only" to aviary age, "not visit-count, not interaction score, not paid," with quiet naturalist copy and no "unlocked" language.

- **Species pool.** The rationale is variety and behavioral contrast without rarity. Adoption draws uniformly from species not yet in the aviary; "rarity is not a feature"; species define silhouettes, motif libraries, behavioral priors, and greeting affordances.

### Presence and interactions

- **Sit-and-watch presence accounting.** The rationale is to reward the actual product behavior without inflating drift: the activity window leans longer because "watching birds without moving is the actual product," while background tabs, unfocused windows, and co-present devices do not over-credit.

- **Return greeting.** The rationale is that the "bird greeting is the entire welcome." It is absence-shaped, server-issued, staggered, non-unison, and biased by boldness, recency, and mood instead of showing textual welcome surfaces.

- **Listen-in.** The rationale is focused attention without muting the aviary: the selected bird rises in the mix while others fall to an ambient floor, "never to silence," because "the aviary stays a place where multiple things happen." Listen-in duration also feeds that bird's social warmth and vocal frequency.

- **Offers: seed, song fragment, still pool.** The rationale is in-world interaction with canonical server outcomes. The server decides reactions from mood/personality, appends the paired reaction event, enforces cooldowns to prevent curiosity saturation, and renders cooldown as a "quiet dimmed state."

- **Settle with 5-second undo.** The rationale is mood quieting and a clean end to presence, not punishment or drift. The plan says settle has "no directional drift," ends the presence window cleanly, and treats undo as a no-op for drift if within the window.

- **Field notebook.** The rationale is sparse, truthful, aviary-only observation. Entries may reference only server-canonical facts, are capped by a sparsity governor, deduped for specificity, and filtered so "the notebook observes the aviary, never the user."

### Sync and state

- **Append-only event log.** The rationale is strict in-order consumption and replay safety: the log gives a per-aviary cursor, additive server-authored deltas, idempotency keys, and no client-state ownership.

- **Snapshot pulling.** The rationale is to avoid a merge/consistency surface: pull on open, visibility, keepalive, and long frame gaps is sufficient for a ~60-second canonical cadence, and "there is no sync code in the client beyond snapshot pulling."

- **Behavior projection.** The rationale is to make "never exposed numerically" architectural. Clients receive bounded, quantized, non-invertible render parameters instead of personality vectors, keeping snapshots small and removing the "temptation surface."

- **Edge-inlined bootstrap state.** The rationale is the <500 ms first-bird budget and the already-running conceit: no blocking API round-trip, no spinner, no entry animation, and birds appear mid-action from inlined snapshot state.

- **Quiet-field fallback.** The rationale is graceful loading without violating the central conceit: cold cache or slow link shows soft sky and faint ambient cues, then cross-fades to the full scene; "no spinner exists in the component library."

- **Stale-then-fresh reconciliation.** The rationale is continuity: server state always wins, but stale state eases to fresh over ~1.5 s and canonical changes reconcile visually without snapping, teleporting, or correction surfaces.

### Simulation engine

- **Δt-parameterized dynamics kernel.** The rationale is cadence safety and debuggability: 60-second ticks, 15-minute dormant ticks, and 48-hour jumps yield the same distribution; seeded determinism enables golden-vector tests and drift histories.

- **Tick scheduling and lazy as-of-now evaluation.** The rationale is that snapshots are never stale and backlog does not make aviaries stop continuing; lazy reads mask scheduler lag while writes remain controlled by tick workers, offer decisions, and greeting issuance.

- **Monotonic personality drift.** The rationale is to make absence non-punitive and prevent Tamagotchi mechanics: the code path for negative deltas "does not exist," neglect yields Δp = 0, and per-tick clamps keep single-session movement invisible.

- **Recency envelope.** The rationale is fast expression without personality loss: it lowers greeting propensity, call energy, and front-perch dwell after absence while leaving personality exactly where it was.

- **Mood system.** The rationale is persistent, readable continuity: moods are stored, never reset on client open, soften over 6-10 hours, and are shaped by time of day, weather, events, coupling, persistence, and seeded noise.

- **Bird-to-bird coupling and chorus.** The rationale is emergent social life: wary moods can spread by susceptibility, social warmth schedules response chains, and overlapping high-vocal windows create chorus without a special code path.

- **Ambient weather scheduler.** The rationale is subtle environmental variation: rain and wind are deterministic, sparse, mood-affecting, and "never assertive."

- **Day/night behavior.** The rationale is local-time continuity: server dynamics use the account timezone, visuals use device-local time, and travelers get correct visuals immediately with dynamics corrected from the next session.

- **Server-planned call windows.** The rationale is to keep canonical state server-owned and sync-coherent while allowing sub-tick procedural audio; two devices hear the same "shape" differently rendered.

- **Synchronous offer reaction decisions.** The rationale is immediate felt response without client authority: tick is too slow for the reaction, so the server takes a short lock, decides, logs, and returns canonical animation parameters.

- **Greeting directives.** The rationale is absence-shaped, non-repeating return behavior that remains in-world: buckets map absence to form, greeters are weighted by boldness and recency, and greeting history prevents back-to-back identical form/greeter pairs.

- **Calibration harness.** The rationale is to tune drift and sparsity without production population analysis. Ghost-watcher profiles run against synthetic staging accounts, and constants are config-owned with the harness as regression suite.

### Frontend rendering

- **WebGL2 scene renderer.** The rationale is the support/performance envelope for "7 rigged birds + weather + parallax" on a 5-year-old laptop; Appendix A says Canvas2D is marginal and WebGL2 is universal across the support matrix.

- **Scene layout solver.** The rationale is responsive continuity: perch zones scale from 320 px to 3840 px with the invariant that "no bird is ever cropped or offscreen."

- **First frame mid-action.** The rationale is the already-running fiction: authenticated navigation inlines positions, moods, activities, and motion phase so the first paint has birds mid-preen, mid-call-posture, or mid-scan.

- **Interpolation and motion reconciliation.** The rationale is that server state wins without visual snapping. Positions tween, moods/projections ease, activities cross-fade, and returning users see changes resolve in the first seconds rather than jump.

- **Idle micro-motion.** The rationale is legibility without labels: mood is "readable from motion alone," with no tooltips, status icons, or hover chrome inside the scene.

- **Top bar and fading chrome.** The rationale is quiet scene primacy and system-only chrome: four icons expose account/settings, accessibility, notebook, and offers, then fade to low opacity after stillness.

- **Reduced-motion rendering register.** The rationale is parity, not subtraction: motion becomes slow pose cross-fades, flight becomes cross-fade between perches, weather becomes static-tinted variants, while audio, captions, drift, mood, and notebook are unchanged.

### Audio

- **Fully procedural WebAudio.** The rationale is "zero recorded audio," no loops, and no canned calls; synthesis plus variation fulfills the plan's "never identical twice" contract.

- **Per-species motif libraries and per-bird signatures.** The rationale is both variation and recognizability: species provide grammar, bird seeds provide stable timbre/motif identity, and mood/personality color realization without erasing the signature.

- **Chorus and response chains.** The rationale is legible emergent chorus without loop artifacts: responses are planned by the server and synthesized in real time, with panning and ducking preserving readability.

- **Listen-in mix ramps.** The rationale is gentle focus without hard cuts: gain changes use slow ramps, other birds remain audible at an ambient floor, and charm-guard tests ban hard cuts.

- **Autoplay strategy.** The rationale is platform reality: browser autoplay policy is "a hard platform law," so the plan attempts resume on load, resumes on first natural gesture, provides no interstitial or icon, and uses captions to cover silence.

- **Graceful silence with captions.** The rationale is the only acceptable audio fallback: if WebAudio is unavailable, captions default on and no recorded-audio fallback exists.

- **Audio memory discipline.** The rationale is the 30-minute no-growth performance budget: pooled voices, reused buffers, stopped/dereferenced nodes, and recycled caption nodes keep graph and heap bounded.

### Social visits

- **Visit invitations.** The rationale is optional, controlled ambient visiting rather than social networking: invites are per-invite opt-in, one-time emailed links, read-only visitor sessions, and host-scoped.

- **Visitor read-only scope.** The rationale is that "a visitor's attention does not drift the host's birds." Visitor tokens have no event-ingest permission and create no presence windows or interaction rows.

- **Revocation.** The rationale is host control: revoke is immediate, and active visitor sessions die at the next snapshot pull.

- **Visit log.** The rationale is account transparency without engagement pressure: logs are on-demand only, never badged, and never notified unless the off-by-default toggle is enabled.

- **Visit-notification toggle.** The rationale is explicitly "account-transparency," not an engagement loop; it is off by default and is the only notification exception.

### Accessibility surfaces

- **Screen-reader narration.** The rationale is naturalist access to the same canonical state the visuals read. It avoids state-list spam, uses one shared grammar with notebook prose, and prioritizes user-initiated events without flooding the queue.

- **Bird focusables.** The rationale is to avoid the "cheap version" of labeling every visual state: each bird's aria-label is only its name, while descriptive prose lives in narration.

- **Call captions.** The rationale is fidelity and access: captions are opt-in, auto-on when audio is unavailable, and generated from the actual synthesized realization so the caption matches what played.

- **Keyboard navigation.** The rationale is full pointerless operation: all interactive surfaces are reachable, the scene is operable one-handed, and focus returns predictably after offer and panel flows.

- **Focus indicators.** The rationale is scene legibility: a soft double-stroke outline stays readable across bright midday and dim night lighting.

- **Contrast and legibility.** The rationale is WCAG AA as a floor for user copy, captions, settings, errors, and displayed narration; automated contrast checks cover lighting keys and caption scrims.

- **Voice register map.** The rationale is continuity and clarity: naturalist copy belongs to in-world surfaces, while any place the user engages the system "as a system" is matter-of-fact.

### Security, privacy, performance, and operations

- **PII firewall.** The rationale is to prevent email from creeping into logs, keys, URLs, or metrics: email appears only in encrypted columns plus hashes, and one identity module can decrypt.

- **Telemetry boundary.** The rationale is privacy and anti-gamification: only aggregate operational metrics are allowed, there is no per-account dimension, and the pipeline cannot touch simulation rows.

- **Privacy policy link in account settings.** The rationale is plain account transparency about aggregate categories and exclusion of per-bird interaction state.

- **Browser support gate.** The rationale is keeping the critical path lean: feature detection rejects unsupported browsers with matter-of-fact copy, with "no compatibility shims, no polyfill bloat."

- **Performance budgets.** The rationale is to preserve first-bird immediacy, sustained 60 fps, no memory growth, small snapshots, and tick reliability; each budget is a CI gate.

- **Synthetic fleet and aggregate RUM.** The rationale is day-one operational visibility without per-account tracking: measure load, frame time, audio errors, snapshot delivery, greeting success, and tick health as aggregate histograms.

- **Deliberately not measuring per-account/per-bird engagement.** The rationale is that "no streak-adjacent data exists to leak into a feature" and no metric labels should reconstruct a user's relationship with the aviary.

- **Charm-guard suite.** The rationale is executable product restraint: it is the product's "immune system," asserting banned surfaces do not exist and tracing each absence test to a PRD clause in a manifest.

- **Rollout stages.** The rationale is calibration and risk control: internal alpha focuses on presence, tick, and first-frame budgets; closed beta tunes constants and narration cadence; public v1 ramps traffic while watching operational alarms.

- **Birds-per-aviary ramp.** The rationale is natural risk staging: the cap-7 engine ships, but age gates keep accounts at two birds for 90 days and let chorus load, call density, and projection cost grow gradually.

- **Operational runbooks.** The rationale is preserving the product voice and continuity during incidents: lazy evaluation masks tick backlog, KV outage degrades to quiet field plus API pull, and incident copy stays matter-of-fact with "no naturalist error pages, ever."
