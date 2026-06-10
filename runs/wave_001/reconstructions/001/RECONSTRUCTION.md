## System-level intent

1. **Structural enforcement over reminders.** The plan repeatedly turns product rules into topology, CI, schemas, permissions, or missing endpoints rather than relying on review. This appears in the scope note that non-goals are "enforced structurally where possible," in the simulation goal of "structural impossibility" for named failure modes, in the no-LWW design enforced by "the database's permission system, not convention," and in the final statement that every "load-bearing PRD rule gets a structural enforcement, not a reminder."

2. **Single canonical state with the simulation as sole writer.** The plan's central state philosophy is that the API and clients never mutate bird state. The architecture says the simulation service is "the only writer of canonical aviary state," the client is "a renderer plus an event reporter," and sync makes the "phone overwrites the laptop's morning drift" failure "unrepresentable."

3. **The boundary rule: bird-changing state lives on the server; frame-only state lives on the client.** The plan gives the rule directly: "if losing it would change the bird, it lives on the server; if losing it only changes this device's current frames, it lives on the client." This governs listen-in mix state, settle lighting transition, ambient leaves, rendering interpolation, and canonical drift/mood/personality.

4. **Privacy minimization is infrastructure, not policy.** Email is stored in one encrypted place, raw pings are deleted after coalescing, event logs expire after use, telemetry has "no account dimension, no bird state, no interaction history," and the analytics warehouse has no path to the simulation database. The plan explicitly says the "privacy line is infrastructure, not policy."

5. **Anti-gamification and restraint are load-bearing.** The plan binds non-goals including "streaks, scores, badges, levels, counters," social-network surfaces, notification outreach, negative drift, and visit calendars. It reinforces this through no per-account aggregates, no notebook access to visit-frequency aggregates, banned-word CI, no notification infrastructure, and a required "voice & restraint" reviewer.

6. **Relationship change is monotonic, quiet, and presence-shaped.** Drift is "monotonic-upward," driven primarily by presence-time, with "no decay term, no neglect penalty, anywhere." The plan says "ambient quietness on neglect is emergent, not stored," so absence can change recent greeting/call density without decreasing personality.

7. **Long-term drift must occupy a narrow band between Tamagotchi and screensaver.** The risk section names the product-defining failure: "too fast → Tamagotchi; too slow → screensaver." The calibration harness encodes one-week instrument drift, three-week visible drift, a "screensaver guard," daily caps, and typical-user simulations as executable tests.

8. **No numeric personality exposure in product surfaces.** The plan's phrase is "numeric-exposure firewall." Trait floats exist only in `birds.personality`, tick-worker memory, and the user-requested account export. Snapshots carry quantized, render-facing projections so there is no client build or devtools path to recover floats.

9. **One naturalist voice across every surface.** The shared prose engine exists to guarantee "one voice" across notebook, narration, captions, offer/adoption copy, and accessibility. The style contract is "lowercase product voice, present tense, no exclamation marks, no second person in observational text, no numerals for any bird property," with banned words such as "achievement," "streak," "level," and "score."

10. **The scene should feel alive without announcements, spinners, or chrome pressure.** The boot path paints the "quiet field" rather than a spinner, top-bar chrome fades to near-transparent, offers are "quiet naturalist" surfaces, errors are matter-of-fact, and launch includes a "no-spinner audit" and "no-toast/no-announcement audit."

11. **Procedural variation should be recognizable, not canned.** The plan uses per-bird `call_seed`, procedural grammar, motif variation, greeting form weights, and stagger jitter to make signatures stable while "no two greetings are frame-identical" and no two calls are identical. Audio loops and recorded fallback are deliberately excluded.

12. **Accessibility is part of the feature, not post-launch.** The accessibility section says "Nothing here is post-launch." Reduced-motion is "a register, not a removal," narration and captions use the same state as visuals/audio, keyboard surfaces are complete, and every workstream includes its accessibility surface in definition-of-done.

13. **Performance budgets are product constraints.** The plan treats <2MB initial bundle, <500ms time-to-first-bird, 60fps idle, and memory flatness as gates. Rendering choices, edge bootstrap snapshots, split chunks, object pools, and synthetic fleet checks are all justified by those budgets.

14. **Social is deliberately ambient and read-only.** The one social feature is "read-only ambient visits" via invite links. Visitor presence never feeds the simulation, visitor tokens have read-snapshot-only capability, notifications are default off, and social-network surfaces are named non-goals.

15. **Defensible calls are explicit where the spec left room.** The plan says it "names every defensible call" and collects decisions such as no WebSockets, mood set, tick cadence, presence window, species-offer schedule, event-log retention, and Canvas 2D renderer in "Ambiguities resolved by this plan."

## Per-feature whys

### Scope

- **One horizontal scene per account.** The plan later supports this by saying the scene is "one screen" with no panning/scrolling/zooming and that Canvas 2D is enough for "≤7 animated birds, subtle parallax, one screen."

- **Two starter birds.** NOT RECOVERABLE FROM PLAN

- **Cap of seven birds.** The cap is tied to performance and audio recognizability: the renderer and audio pipeline are designed for "≤7 animated birds" and "7 birds + 1 ambient overlap slack," while the risk section says recognizability is tested "at 2, 4, and 7 birds" and could gate a product decision.

- **Three perch zones.** NOT RECOVERABLE FROM PLAN

- **Local-time day/night cycle.** The account `timezone` "drives day/night and mood time-of-day inputs"; mood transitions use the account's IANA timezone so morning, dusk, and night behavior align with local time.

- **Rare ambient weather.** Weather is server-scheduled because it is canonical: "host and visitor must see the same rain." It also supplies mood-modifier inputs with decay.

- **Client-side ambient ornaments.** Leaves and feathers are explicitly client-owned because losing them only changes "this device's current frames"; they are stateless and kept out of canonical state.

- **Hidden five-trait personality vector per bird.** Personality is stored, never derived, because "a lost vector is a deleted relationship." Hiding it also supports the "numeric-exposure firewall" and prevents product surfaces from normalizing numeric personality.

- **Monotonic-upward drift driven primarily by presence-time.** The drift function exists to make presence shape the relationship while avoiding decay, neglect penalty, or negative drift. Daily caps keep change out of gaming range.

- **Fast-timescale mood with cross-session persistence.** Mood persists because it "only ever changes in tick-time"; daily-ish reset is a slow pull so the user never observes a snap.

- **Procedural call grammar with per-bird recognizable signatures.** The layering makes Pip remain Pip while mood and drift change when/how often/how bright the call is; per-call randomness prevents repeated identical renditions.

- **Mood-shaped idle motion.** The behavior interpreter uses mood, idle profile, and activity to pick micro-motions so posture and movement reflect current state without visible loops.

- **Bird-to-bird interaction.** Alarm calls apply wary pressure, compatible moods create `chorus_window`s, and head-tilts are wired to actual scheduled call events, making chorus and response "causal, not random."

- **Return-greeting.** The plan's why is that variation should be real and never canned: absence class, mood, boldness band, motif grammar, weighted greeter selection, recent-greeter penalty, and stagger jitter make greetings observable and truthful.

- **Listen-in.** Listen-in focuses one bird without muting the others: focused gain rises while other birds stay "audible ambient, never −∞." It remains client-local while start/end events report drift input.

- **Offer types: seed / song fragment / still pool.** NOT RECOVERABLE FROM PLAN

- **Per-bird offer cooldown.** The cooldown is an anti-gaming and no-chrome mechanism: violations are accepted with `cooldown_suppressed=true` and contribute zero drift so the client does not need an error surface inside the aviary.

- **Settle.** Settle runs the evening lighting ramp immediately, reports a canonical event, and ends presence cleanly; visitors then see canonical settled state.

- **Settle 5-second undo.** NOT RECOVERABLE FROM PLAN

- **Presence accounting with three-signal conjunction.** The sampler requires visibility, focus, and recent activity so drift reflects real presence; the plan explicitly says to "lean long because motionless watching is the product."

- **Field notebook: sparse, naturalist, read-only.** Sparsity prevents the notebook from becoming an activity feed; naturalist prose keeps the product voice; read-only construction prevents editing/deleting observations and keeps it as a generated record of aviary moments.

- **Magic-link email auth.** The plan gives operational guardrails, including no account-existence oracle, consumed tokens, and rate limits, but it does not articulate why magic links are chosen over another auth method: NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions.** The sessions list exposes device labels and revocation because sessions are per-device and must be manageable from settings.

- **Email change with verification.** The switch commits only after consuming the new-address link and the old email remains valid until then; the rationale is account safety around address changes.

- **Synthetic UUID account IDs and encrypted email stored once.** This supports the hard privacy rule that no other table, log, payload, partition key, or telemetry event may contain email.

- **Server-side simulation tick as single writer.** The tick is the single writer to make no-last-write-wins a deployment and database property rather than code discipline.

- **Snapshot-pull clients.** Snapshot pull is chosen because the stated pull triggers cover freshness at a roughly one-minute cadence and "a push channel adds connection-state machinery the product doesn't need."

- **Append-only interaction event log.** The log gives a server-assigned order for drift inputs and prevents clients from submitting absolute bird state.

- **Account export by emailed JSON link.** Export is framed as "the single named exception to numeric exposure" because it is "a file the user requested about their own data, not a product surface."

- **Soft delete for 30 days then hard delete.** The plan describes the marker, restore route, and reaper, but gives no specific rationale for 30 days: NOT RECOVERABLE FROM PLAN

- **Read-only ambient visits by per-invite email links.** Visits are kept to one restrained social feature; visitor tokens can only read snapshots and visitor presence writes nothing to the event log.

- **Revocable, 30-day visit expiry.** Revocation is checked on every pull and expiry returns a matter-of-fact unavailable surface; the 30-day duration itself is not justified: NOT RECOVERABLE FROM PLAN

- **Silent visit log.** The visit log supports settings visibility without feeding the simulation or becoming a social surface.

- **Opt-in visit notification toggle default off.** The toggle is the "one opt-in social toggle," preserving the broader no-notification and default-off social posture.

- **Visitor presence never feeds the simulation.** This protects the owner's aviary state from visitor behavior and is enforced by route capability and DB role.

- **Screen-reader running narration.** Narration gives the same living aviary to screen-reader users by consuming the same snapshot, behavior interpreter, call events, and narration context as the visual renderer.

- **Reduced-motion as designed cross-fade rendering.** Reduced motion is a second render mode over the same interpreter so behavior, drift, notebook, greetings, audio, and captions remain identical while motion maps to pose cross-fades.

- **Procedural call captions.** Captions are generated from the actual synthesized parameters so they describe "what just played, by construction"; they auto-enable when WebAudio fails.

- **Full keyboard navigation.** Keyboard navigation makes birds, listen-in, offer, settle, undo, and escape reachable without pointer use; focus cannot land on invisible chrome.

- **WCAG AA contrast.** Contrast checks ensure copy, captions, errors, and focus treatment remain readable across dawn, midday, and night scenes.

### Architecture

- **Edge/CDN serving bootstrap state snapshot.** The bootstrap snapshot is critical for the time-to-first-bird budget because first bird paint should not wait on an origin round-trip.

- **API service.** The API is stateless and owns auth, reads, event writes, account, notebook, visit, export, and deletion because API load follows client traffic.

- **Simulation service.** Simulation load follows account count and is steady, and keeping it separate makes the no-LWW rule a deployment property.

- **Auxiliary jobs.** NOT RECOVERABLE FROM PLAN

- **Client receives semantic state, not frames/audio.** Semantic state keeps snapshots in low kilobytes, makes reduced-motion a pure client mode over identical state, and lets narration, captions, and visuals derive from one state object so they "can never disagree."

- **TypeScript end-to-end with shared packages.** Shared state types, call grammar, and prose library guarantee consistent contract and "one voice" across server and client surfaces.

- **Node.js server runtime.** The plan says tick math is small and the workload is I/O-bound, not compute-bound, so Node.js is sufficient for API and simulation workers.

- **Postgres as single canonical store.** It supports one row-set per aviary as truth, transactions, SKIP LOCKED worker claiming, grants, WAL/PITR, and the single-writer model.

- **Canvas 2D with layered offscreen canvases.** Canvas 2D is chosen because Pixi/Three would consume too much of the 2MB budget for unneeded capability.

- **Procedurally-tinted compact vector parts rasterized to sprite atlases.** This lets plumage-saturation drift render without shipping per-saturation art.

- **Preact chrome UI with code-splitting.** Preact is small and settings/account/visit bundles load on demand, preserving the initial bundle budget.

- **WebAudio AudioWorklet synthesizer.** The worklet enables procedural synthesis with preallocated voices and graceful degradation while avoiding recorded audio.

- **HTTPS request/response only, no WebSockets.** No WebSockets is a v1 simplification because snapshot pull triggers cover the freshness need and avoid connection-state machinery.

### Data model

- **UUIDv7 IDs.** They are "time-ordered, index-friendly."

- **`accounts.email_encrypted` and `email_hash`.** Encryption stores email only once, while HMAC hash allows sign-in lookup without decryption.

- **Lint ban on email outside legitimate holders.** The lint rule enforces the hard rule that email cannot leak into tables, logs, payloads, partition keys, or telemetry.

- **`sessions` with token hashes, device labels, last seen, and revocation.** This supports per-device session lists and revocation.

- **Magic-link token consumption transactionally on first use.** This prevents replayed links and supports expired/consumed matter-of-fact conflict surfaces.

- **Magic-link rate limit of 5 per email-hash per hour.** The plan calls this a defensible default for abuse control, tunable later.

- **Separate `aviaries` table despite one aviary per account.** The separation exists so "multi-aviary never requires a migration of bird FKs."

- **`tick_seq` monotonic counter.** It is the per-aviary canonical tick counter used for ordering and snapshot coherence.

- **`birds.bird_id` stable identity.** It is "never reused, never regenerated," preserving continuity.

- **`birds.personality` as stored state.** It is never derived or recomputed because rebuilding from events would violate "never recomputed from event logs."

- **Continuous WAL archiving and PITR.** Backup posture follows because losing a vector is treated as deleting a relationship.

- **`interaction_events` append-only with server-assigned `seq`.** This gives the tick a total order and makes the log an input queue rather than an archive.

- **Event retention of 90 days after consumption.** The plan frames this as privacy minimization because keeping the log forever would contradict the stance that interaction history is not theirs to keep indefinitely.

- **`presence_windows`.** The tick coalesces pings into windows because windows are the drift integrand and return-greeting absence input.

- **Deleting raw pings after coalescing.** This is explicitly "privacy minimization."

- **`notebook_entries` read-only.** Read-only is by construction: no update/delete API exists.

- **`invites` holding visitor email.** The invite is the "one other legitimate email holder" because the product must email the visitor.

- **Visitor sessions scoped to read-snapshot-only.** The route layer enforces that visitor tokens cannot reach event-write, notebook, or account endpoints.

- **Six-species static pool.** Species data is versioned so adding species later never alters existing birds; existing `species_id` is fixed at adoption.

### API surface

- **Routes keyed by session account, no aviary ID in URLs.** This prevents an enumerable resource space.

- **`GET /api/aviary/snapshot`.** It is the core read because clients and visitors render snapshots and interpolate locally.

- **Snapshot projection fields instead of raw traits.** The projection layer makes "never exposed numerically" an API property.

- **`GET /api/aviary/notebook?cursor=`.** Paginated read-only reads support unlimited scroll-back without an edit surface.

- **Visitor snapshot without greeting, offer cooldown, and personalized narration context.** This keeps visits ambient and restricted rather than owner-personalized.

- **Visitor snapshot returns 410 after revocation/expiry.** Revocation is enforced on pull with a matter-of-fact unavailable surface.

- **Batched `POST /api/aviary/events`.** Batching one POST per roughly 60s of continuous presence reduces request load while preserving the event stream.

- **Presence flush on hide via `navigator.sendBeacon`.** This captures end-of-presence when visibility changes or the tab closes.

- **`client_event_id` idempotency.** Upserted client IDs make retries never double-count drift inputs.

- **Dedicated `POST /api/aviary/settle`.** The route lets the client sendBeacon settle on tab close only when the user actually settled; tab-close itself sends no settle.

- **Cooldown-violating offers accepted with zero drift.** This avoids an error surface inside the aviary while keeping the server authoritative.

- **Auth request link always returns 200.** This prevents an account-existence oracle.

- **Auth consume returns session cookie or 410.** The 410 path gives expired/consumed links matter-of-fact copy.

- **Session list and revoke endpoints.** They expose per-device revocable sessions.

- **Email change verification link.** The switch commits only on consumption so a typed new email does not immediately replace the verified one.

- **Account timezone report.** Timezone drives day/night and mood inputs, so the client reports it on session start and detected change.

- **Invite creation limit of 10 outstanding invites.** The plan calls this a defensible anti-abuse default.

- **Invite revoke takes effect on visitor next snapshot pull.** That matches the pull-based sync model without WebSockets.

- **Adoption and rename endpoints.** Species are chosen server-side for adoption/offered adoption; the plan does not articulate a rationale for rename beyond it having no effect on anything else: NOT RECOVERABLE FROM PLAN

- **API-layer voice rule.** The server centralizes matter-of-fact error copy so clients do not invent system-surface language.

### Simulation engine design

- **Canonical 60-second tick.** The tick is the unit that consumes events, applies drift, transitions mood, schedules greetings, and writes canonical state.

- **Workers claim due aviaries with `FOR UPDATE SKIP LOCKED`.** This gives horizontal scalability and at-most-one concurrent tick per aviary.

- **Single transaction per tick.** A crashed tick rolls back wholly, leaves events unconsumed, and makes tick processing idempotent over inputs.

- **Dormancy tiering.** Coarser ticks for inactive aviaries make fleet cost scale with active users while preserving observably identical canonical state because absence contributes zero drift and mood/day-night transitions are elapsed-time functions.

- **Immediate catch-up tick on dormant snapshot request.** The catch-up preserves the sense that "the aviary has been running" at return time.

- **Tick computation order.** The order ensures presence, event inputs, drift, weather, mood, perches, calls, greeting, notebook, and offers are applied consistently before state is written.

- **Drift formula with `(1 - personality_t)`.** The formula creates a low-pass and soft ceiling as traits approach 1.0.

- **`max(0, delta)` monotonicity.** It structurally prevents decreases and makes the asymmetric-drift rule unbreakable with property tests.

- **Daily drift caps.** Caps prevent a single long session from producing visible change and serve as anti-saturation/anti-gaming control.

- **Calibration targets as executable tests.** The tests define visible drift as quantized projection changes and pin the narrow band between too-fast and too-slow.

- **Ambient quietness on neglect as emergent.** Recent presence affects greeting probability and call density so absence feels quieter without moving traits down.

- **Mood state machine.** Mood scores combine time of day, interactions, ambient events, bird-to-bird pressure, and personality projections so mood is responsive without being personality itself.

- **Mood inertia.** A mood persists at least three ticks unless overridden so the user does not read flicker as twitchy.

- **Continuous pull toward time-of-day baseline.** This avoids a discrete reset or observable snap.

- **Alarm-class call spread and chorus windows.** Wary pressure and compatible chorus scheduling create bird-to-bird interaction server-side and render it client-side.

- **Server-scheduled weather.** Canonical weather makes host and visitor see the same rain and gives weather mood effects.

- **Absence classes for greeting.** Classes shape form without surfacing "gone X days" numbers.

- **Greeter weighted lottery.** Boldness, social warmth, mood, and recent-greeter penalty make which bird greets first both characterful and varied.

- **Greeting form selection.** Absence class, mood, and boldness band make return forms match context, with longer re-orientation forms after days away.

- **Secondary greeter staggers.** Randomized staggers avoid simultaneous greetings.

- **Species-offer pacing by aviary age.** Offers are never driven by visits, interactions, or drift, which keeps them out of engagement-loop territory.

- **Species offers as quiet naturalist surface with no expiry or re-prompt.** The surface avoids badge/notification logic and keeps declining non-punitive.

- **Species selected server-side from unused pool.** This prevents client-controlled species selection and preserves canonical adoption state.

- **Internal tooling renders tiers and sparklines, not floats.** This extends numeric non-exposure into tooling so internal screenshares do not normalize personality numbers.

- **Notebook noteworthiness scoring.** The scorer picks aviary observations such as greeter changes, mood streaks, weather-behavior coincidences, chorus events, and first days so entries are grounded in actual tick state.

- **Notebook token bucket.** The bucket guarantees sparsity regardless of activity.

- **Notebook candidate vocabulary excludes user behavior.** Since visit counts and streak-like facts are not available to the generator, "you visited every day this week" is unwritable.

- **Notebook prose templates.** Slotted naturalist templates with concrete tick details keep entries varied, lowercase, present-tense, and non-numeric.

### Sync model

- **Single canonical row-set per aviary.** This makes multi-device sync a "non-feature": two signed-in devices read the same record and become coherent within one pull interval.

- **Visibility, long-frame-gap, and keepalive pulls.** These triggers handle tab returns, laptop resume, and normal visible freshness without a push channel.

- **No client write path to bird state.** Because clients append events instead of writing bird values, last-write-wins state overwrites cannot happen.

- **Server-assigned per-aviary sequence.** Interleaved devices produce one ordered log and one drift outcome.

- **At-least-once delivery plus dedup.** Idempotent event ingestion makes drift input effectively exactly-once.

- **API DB role cannot update bird state.** The grant structure enforces the no-LWW rule in the database.

- **Stale-render reconciliation.** If the local render is stale, the event is still recorded against true server state and the next snapshot self-heals cosmetic divergence.

- **Visitor sync through restricted projection.** Visitors share the snapshot pipeline but only with a restricted projection and keepalive-based duration updates.

- **Visitor DB role lacks `INSERT` on `interaction_events`.** This structurally prevents visitor presence from feeding simulation.

### Frontend rendering pipeline

- **Quiet field first paint.** The quiet field is both the pre-state and empty-aviary state, avoiding spinners and preserving the mood while network/code load completes.

- **Edge-cached bootstrap snapshot.** It lets first bird draw within the <500ms contract by avoiding origin latency for returning sessions.

- **Birds render mid-action from snapshot state.** Activity kind and `since_s` seed animation phase so there is no artificial entry animation or fade-from-static.

- **Audio context initializes lazily.** Visuals never wait on browser audio permissions.

- **Three offscreen-canvas layers.** Layering lets background redraw rarely, keeps ornaments separate, and supports frame budget discipline.

- **Runtime-tinted part sprites.** This supports plumage tier changes without shipping full art for every saturation.

- **Skeletal-lite animation.** It gives continuous, mood-parameterized idle motion without a heavy engine or frame-flip loops.

- **Responsive layout solver clamps birds inside viewport.** Narrow viewports compress spacing without cropping a bird.

- **Behavior interpreter between snapshots.** It keeps birds alive between one-minute canonical snapshots while remaining driven by semantic state.

- **Smoothed noise and no per-frame `Math.random()`.** This prevents detectable loops and protects per-frame performance.

- **Interpolated snapshot deltas.** Flights/cross-fades and mood ramps prevent teleporting or snapped posture.

- **Settle immediate client lighting ramp.** The user sees the settle transition immediately while the server receives the canonical event.

- **Single `requestAnimationFrame` loop and dirty flags.** This enforces frame loop discipline and avoids redundant work.

- **No rendering while hidden.** Background tab suspension matches the PRD and presence accounting stops by definition.

- **Object pools and no hot-path allocations.** This is the frontend expression of the memory-flatness rule.

- **Top bar fades after stillness and restores on activity/focus.** Chrome stays quiet but keyboard focus never lands on an invisible control.

- **Notebook and offer as quiet edge panels.** They add surfaces without taking over the scene.

- **Presence sampler every 10 seconds.** Sampling and batching turn DOM visibility/focus/activity into drift-accounting events without constant writes.

- **Config-served activity window.** Calibration can change the "few minutes" window without a client release.

### Audio pipeline

- **AudioWorklet DSP voice chain.** Procedural synthesis creates bird calls without shipping recorded audio.

- **Preallocated pool of eight voices.** The pool covers seven birds plus one ambient overlap slack and satisfies audio memory flatness.

- **Species motif library and probabilistic grammar.** The grammar produces call structure that can vary by mood.

- **Immutable per-bird call seed.** Stable signature parameters keep a bird recognizable through mood and drift.

- **Server-provided call-plan seed.** Seeded runtime variation ensures a given call plan renders the same on two devices while still avoiding identical renditions across calls.

- **WebAudio clock-domain scheduling.** Lookahead scheduling gives sample-accurate call and chorus timing.

- **Chorus windows.** They create call/response among independent procedural voices rather than layered loops.

- **Call events feed behavior, captions, and narration.** The same audio event can trigger causal head tilts and accessible text.

- **Listen-in gain ramps.** The ramp feels like focusing attention over 2.5s while never muting the rest of the aviary.

- **Autoplay handling with slow fade-in after user gesture.** The fade reads as "you tuned in," avoids an announcement banner, and prevents a burst by dropping calls scheduled while suspended.

- **ScriptProcessor fallback.** It keeps procedural calls available on older browsers at reduced polyphony.

- **Graceful silence plus captions fallback.** When WebAudio cannot run, the product remains accessible and honest without recorded audio.

### Accessibility surfaces

- **Shared prose engine.** One library makes the voice consistent and forces new events to have templates before they can ship.

- **ARIA live region narration.** The live region gives screen-reader users a slow, prioritized stream of aviary observations without backlogging the queue.

- **Canvas `role="img"` and per-bird focusable DOM elements.** This turns a visual canvas scene into addressable accessible objects with naturalist labels.

- **Reduced-motion mode from `prefers-reduced-motion` or toggle.** It respects system preference while allowing explicit user override.

- **Pose-graph cross-fades.** Cross-fades replace continuous skeletal motion while preserving the same state and design attention.

- **Keyboard traversal and controls.** Tab, arrows, Enter, Escape, dialog focus trapping, and keyboard undo make the primary interactions fully navigable.

- **Dual-tone focus halo.** The light inner/dark outer treatment guarantees contrast across changing scene lighting.

- **Captions from synthesized parameters.** Caption text is truthful to the call that actually played.

- **Adaptive caption scrim.** Sampling local luminance supports WCAG AA contrast near the calling bird.

- **Axe-core and custom contrast CI.** Automated checks keep accessibility surfaces from regressing across light, dark, dawn, and night.

### Performance budgets and observability

- **<2MB gzipped initial bundle allocation.** The explicit budget and chunk allocation keep renderer, audio, art, prose, state, chrome, and fonts within headroom.

- **Settings/account/visit/notebook/adoption split chunks.** Non-core surfaces load outside the initial budget.

- **Snapshot written through to edge KV.** Edge hits make returning-session first bird paint fast.

- **Species art for this aviary in the edge snapshot.** First bird never waits on a separate art request.

- **Fonts non-blocking.** Text rendering cannot block the first-bird path.

- **30-minute scripted runtime test.** Sustained 60fps and memory flatness are tested under interactions, weather, and settle instead of only idle.

- **Notebook-panel virtualization.** Entries release references on scroll-out to support memory flatness.

- **Synthetic browser fleet.** Automated browsers continuously measure boot, idle, interactions, timing, audio errors, and generation errors across geographies.

- **Aggregate-only RUM.** RUM measures operational health by browser, geo, and device class without account, bird, or interaction-history dimensions.

- **Telemetry schema proxy.** The proxy drops nonconforming events so privacy boundaries are enforced at egress.

- **Tick latency and backlog alarms.** Server observability catches simulation health problems, including p99 tick latency over 5s.

- **Deliberately unmeasured engagement metrics.** The absence of per-account engagement, retention, drift distributions, and visit-frequency analytics makes leaderboards and streaks structurally hard to add.

### Rollout

- **M0 engine-on-rails.** The engine starts first because drift calibration is the longest calendar bake and must prove monotonicity, visible drift, and screensaver guard before later surfaces depend on it.

- **M1 living scene.** Internal alpha starts as early as the engine allows because multi-week aviaries need calendar time for drift calibration.

- **M2 full surface closed beta.** The beta deliberately recruits screen-reader and reduced-motion users and requires three or more weeks to cover one full visible-drift period.

- **M3 visits enabled at GA.** Visits ride the existing snapshot pipeline and are "lowest-risk-last."

- **Bird-count ramp.** Species-offer pacing means production chorus complexity ramps naturally while staged age-skewed aviaries validate 4-7 bird recognizability.

- **Production calibration canaries.** Synthetic accounts run scripted presence as a live regression check on drift's narrow band.

- **Launch-blocking audits and scans.** No-spinner, no-toast, banned words, numeric-exposure scan, monotonic drift, API role grants, and voice review tie launch directly to invariants.

### Risks

- **Drift calibration mitigation.** Constants are isolated, simulations and canaries pin bounds, and server-side config allows retuning without a client release because drift misses are product-defining.

- **Personality-vector loss mitigation.** PITR/WAL, restore drills, transactionality, invariant checks, and internal audit logs exist because a reset bird can betray the user silently weeks later.

- **Sync correctness mitigation.** DB grants, single-writer topology, idempotent ingestion, and interleaved-device tests prevent future convenience endpoints from writing bird state.

- **Audio uncanniness mitigation.** Listening panels, perceptual references, signature distance checks, and possible cap decision exist because audio is "the affective spine."

- **Accessibility regression mitigation.** Workstream definitions of done, shared prose templates, reduced-motion interpreter parity, recruited beta users, and external audit prevent accessibility from lagging features.

- **Presence-signal dishonesty mitigation.** Synthetic browser tests, server sanity bounds, and aggregate browser-family dashboards exist because presence miscounting would be silent and population-wide.

- **Time-to-first-bird erosion mitigation.** CI budgets, edge snapshot hit-rate, staleness monitoring, and cold-cache quiet field protect the boot experience from bundle creep and cache misses.

- **Magic-link friction and abuse mitigation.** Transactional provider setup, deliverability monitoring, resend affordance, and rate limits address email as a product dependency.

- **Scope gravity mitigation.** Missing aggregates, excluded notebook vocabulary, banned-word CI, absent notification infrastructure, onboarding non-goals, and voice review resist engagement features over time.

### Ambiguities resolved by this plan

- **Mood set.** The plan chooses `wary, content, curious, drowsy, alert, settled` under the license that mood is finalized in implementation.

- **Presence activity window.** Five minutes is chosen because the plan wants to lean long; motionless watching is the product.

- **Tick cadence.** Sixty seconds plus dormancy tiering is chosen because the plan fixes user-visible behavior, not fleet scheduling.

- **Offer cooldown.** Three minutes per bird is selected as the concrete "few minutes," but the plan gives no additional rationale beyond anti-gaming elsewhere.

- **No WebSockets in v1.** Pull triggers satisfy freshness and avoid connection-state machinery.

- **Species-offer schedule.** The plan translates "a few months" and "a year" into concrete age milestones.

- **Export includes personality floats.** The rationale is user-owned, on-demand data, but the plan flags it for explicit product sign-off because it strains the never-expose rule.

- **Event-log retention.** Ninety days is aligned with privacy posture, pending privacy review.

- **Mood reset as continuous pull.** The plan avoids any discrete reset a user could observe.

- **Canvas 2D renderer.** Bundle budget and scene simplicity justify avoiding an engine dependency.

- **Invite cap.** Ten outstanding invites is an anti-abuse default; the exact number is otherwise not justified: NOT RECOVERABLE FROM PLAN

- **Initial drift daily caps and weights.** The constants are starting points; the calibration tests are the contract.

### Workstreams and sequencing

- **Simulation engine workstream.** It starts first because it owns the longest calendar bake: tick, drift, mood, greetings, weather, species offers, and calibration harness.

- **Platform and accounts workstream.** It owns schema, auth, sessions, API, export/delete, DB-role enforcement, and edge snapshot cache because those are the platform surfaces that uphold canonical state and privacy.

- **Scene and rendering workstream.** It owns the canvas, behavior interpreter, boot path, responsive layout, reduced motion, and top bar because all are driven by snapshot shape.

- **Audio workstream.** It owns DSP, grammar runtime, listen-in, fallback, and panels because audio believability is a long pole.

- **Voice and prose workstream.** It owns `@aviary/prose`, notebook, narration, captions, product copy, and style CI because one voice is a cross-surface invariant.

- **Quality and observability workstream.** It owns perf CI, synthetic fleet, RUM privacy proxy, accessibility CI, and invariant checkers because the plan's central method is to encode invariants as gates and grants.
