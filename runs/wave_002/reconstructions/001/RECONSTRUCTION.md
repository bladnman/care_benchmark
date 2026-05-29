## System-level intent

- **"felt-aliveness is the product"**: The plan states this directly in the opening and treats "server-side simulation tick, procedural audio, monotonic-toward-expressive drift, and the 'notice, never announce' discipline" as "the architecture," not "features layered on a CRUD app." It shows up again in the first-frame requirement, the continuous micro-motion, procedural calls, the server tick that runs "whether or not a client is connected," and the final summary: "This plan builds the aviary that has been there all along."

- **"notice, never announce"**: The plan repeatedly avoids surfaces that would declare the system's mechanics. It rejects "welcome back" toasts, spinners, streak counters, "your friend visited!" pings, status icons, mood labels, and generic notebook events. It appears in the return-greeting design, the notebook rule that it observes "the aviary, never the user," the slow-load "quiet field," and the copy lint that flags "announcement vocabulary."

- **"two registers, one hard line"**: The plan splits product voice into naturalist field-notebook voice for the aviary, notebook, narration, captions, and offer prompts, and matter-of-fact voice for "identity, money, errors, settings." The intent is that any surface where the user engages the system "as a system" drops out of the naturalist register. The plan makes this "a lint-able copy convention," not a per-PR judgment call.

- **Structural enforcement over good intentions**: The plan repeatedly says the desired behavior should be unreachable to violate, not merely discouraged. Examples include clients having "no personality-write code path at all," no edit/delete columns for notebook entries, "no recorded-audio path exists in the codebase," metrics that are "not computed," separate simulation and operational planes, and copy lint that protects voice. The privacy commitment is enforced "by network policy and separate credentials, not by reviewer vigilance."

- **Privacy boundary as architecture**: The plan draws a "hard, named boundary" between the simulation plane and the operational plane, with "no data store and no ETL path" shared. Email appears in "exactly one column," synthetic UUIDs are the universal identifier, telemetry has "no per-account dimension," and per-bird events drive "only that user's own simulation." The plan treats this as a product principle, not just compliance plumbing.

- **Server-authoritative continuity**: The aviary should continue without the viewer. The server is "the only writer of personality, mood, and authoritative position," the tick runs connected or not, clients render snapshots and interpolate, and multi-device sync becomes "free" because clients read one canonical record. This is the reason the first frame can be "already in motion" rather than a resumed freeze.

- **Slow, monotonic, non-punitive change**: Personality drift is "monotonic-toward-expressive" and uses a "MONOTONIC-UP" clamp. Neglect produces "ambient quietness," never negative drift, distress, guilt, or a decaying happiness meter. The plan's drift calibration tests exist to prevent both "Tamagotchi-by-clicking" and "screensaver."

- **The product is defined by subtractions**: The plan says "The product is what's left after the subtractions." It excludes gamification, Tamagotchi mechanics, social-network surfaces, notification surfaces, personality numbers, and hypothetical native-client constraints. Those refusals are tied to concrete system choices: no visit-frequency metrics, no leaderboard/discovery pipeline, no public profiles, no streaks, and no "show-off rendering."

- **Stable individuality and recognizability**: Birds have stable `bird_uuid` identity, hidden slow traits, user names, species motifs, and call fingerprints. The plan says "Pip stays recognizably Pip by ear" and that a bird is never "reset," "regenerated," or "swapped." The 7-bird cap protects the "recognizability ceiling."

- **Accessibility is the actual product, not a fallback**: The plan says alternate paths must deliver "the actual product that feels alive," not "a semantic-markup transcription of visual states." Screen-reader narration, reduced-motion, captions, keyboard navigation, and contrast ship with v1 and "gate launch." Reduced-motion is "a different rendering of the same aviary," not "animations off."

- **Performance protects the conceit**: The plan treats budgets as "gates, not guidelines." The <500ms time-to-first-bird target matters because above it "the central conceit collapses into a visible load." The bundle, frame-rate, memory, inlined snapshot, WebGL2, and procedural audio decisions all defend the feeling that the aviary was already there.

- **Deterministic, testable, bounded systems over opaque generation**: The plan prefers calibration tests, copy lint, CI bundle and memory gates, a deterministic notebook/narration grammar, shared schemas, additive deltas, and bounded numeric inputs. The LLM is deferred because the grammar is "testable, cheap, offline-safe, privacy-clean" and "incapable of drifting into announcement voice."

## Per-feature whys

### Scope

- **One canonical aviary per account**: The plan ties this to server-authoritative architecture: both clients read one canonical record, so multi-device sync is a property of the system rather than a separate reconciliation feature.

- **Two starter birds**: NOT RECOVERABLE FROM PLAN

- **Hard cap of 7 birds**: The plan says the cap exists because per-bird call recognizability has a ceiling. It protects distinct motif and timbre fingerprints in the chorus, so birds remain individually recognizable.

- **One horizontal single-screen scene with no pan/scroll/zoom**: NOT RECOVERABLE FROM PLAN

- **Day/night anchored to user local time**: The plan uses local time to make the scene feel like "a window on a real morning" and to drive mood transitions such as morning alertness and dusk/night `settled` states.

- **Rare ambient weather**: Weather is a gentle overlay that also feeds mood inputs: rain dampens vocal frequency and nudges calmer moods; wind can push birds toward `alert` or `wary`.

- **Continuous ambient micro-motion**: The plan uses micro-motion to keep the scene alive between explicit actions and to make mood legible without labels.

- **Hidden 5-trait personality vector**: The vector is hidden and "never exposed numerically" because exposing it would turn felt change into a system/debug surface. The plan protects the affective contract by keeping personality expressive, not quantified.

- **Monotonic-toward-expressive drift**: The rationale is to allow slow cumulative change without punishment. The "max(0, ...)" clamp means low activity yields zero delta, never regression; neglect creates ambient quietness, not guilt or negative trait movement.

- **Fast-timescale mood**: Mood handles short-lived state separately from slow personality. It persists across sessions and is modulated by ticks, so the aviary never "snap[s] to neutral" on tab open.

- **Procedural call grammar**: Procedural grammar provides runtime variation and a real chorus. It avoids the spell-breaking repetition and phase artifacts of recorded loops while staying inside the bundle budget.

- **Mood-shaped idle motion**: Motion is how the user reads mood "without a label." The plan says a status icon or tooltip would break the affective contract because the bird's feeling should be visible in posture and motion.

- **Bird-to-bird interaction**: NOT RECOVERABLE FROM PLAN

- **Stable internal bird identity**: Stable `bird_uuid` prevents a bird from being "reset," "regenerated," or "swapped" across rename, sync, species-pool change, or migration. It protects personality continuity and reconstructibility.

- **User-assigned renameable names**: NOT RECOVERABLE FROM PLAN

- **Species pool size of about 6**: NOT RECOVERABLE FROM PLAN

- **One nightjar-like nocturnal signature**: The plan's reason is that it "remains active at night," supporting the day/night aviary without making night feel empty.

- **Age-gated new-bird offers**: New birds are paced by aviary age, "months, not visit count," so the product never teaches "more attention earns more stuff."

- **Notebook sparsity target of <=1 entry/day**: The plan wants the notebook to stay sparse and salient. The cooldown rises with recent activity so more visiting "must not mean more entries."

### Interactions

- **Return-greeting**: The greeting is the only welcome surface and must feel like birds noticing, not the system announcing. It varies by absence length, boldness, and mood, and multiple birds are staggered so they do not become a "unison chorus-on-cue."

- **Listen-in**: The gradual mix re-balance is meant to feel like listening, "not channel-switching." Other birds are lowered but "never a mute" because muting would teach the wrong audio surface.

- **Listen-in event logging**: Listen-in duration is a "strong attention signal" for that bird's warmth and vocal_frequency, so start/end events go into the append-only log.

- **Offer interaction**: Offers give bounded inputs to curiosity and boldness while reactions are mood-shaped. The per-bird cooldown is "functional, not punitive" and prevents curiosity-trait saturation in one session.

- **Offer kinds (`seed`, `song`, `pool`)**: NOT RECOVERABLE FROM PLAN

- **Settle**: Settle is a soft session end that cleanly closes the presence window. It is equivalent to tab close at the engine level and has no penalty or "you didn't settle" recovery surface.

- **Settle 5s undo**: NOT RECOVERABLE FROM PLAN

- **Field notebook**: The notebook exists to record sparse, specific aviary observations, not user behavior or generic logs. It observes the aviary, "never the user," and its read-only structure prevents it from becoming an editable journal or task surface.

- **Presence accounting by 3-signal conjunction**: Presence is "honest by construction." The plan rejects the "tab is open" shortcut because it would count a laptop left open overnight as watching and corrupt population-wide drift.

- **4-minute activity window**: The window leans long because "watching birds without moving is the actual product."

- **Stopping pings and rendering on hidden/blurred tab**: Pings stop to avoid false presence, rendering stops for battery and memory, and the server tick continues so return pulls the aviary that "was running," not a freeze.

### Accounts, Sync, And Social

- **Email magic-link auth**: The plan uses 15-minute, single-use magic links with hashed nonces and rate limiting. The request endpoint always returns 200 to avoid an account-existence oracle.

- **Per-device revocable session tokens**: Sessions have a token id, coarse device label, and `revoked_at` so users can revoke specific devices.

- **Verified email change**: NOT RECOVERABLE FROM PLAN

- **Synthetic UUID as universal identifier**: This prevents email from appearing in paths, keys, logs, events, telemetry, or downstream systems. Email is stored encrypted in one column only.

- **Account export**: Export gives the user an on-demand JSON snapshot of their own account. The plan notes this is the one place vectors leave the system, and only "to the user, about the user's own birds," preserving the UI rule against showing numbers.

- **Soft-delete 30 days then hard-delete**: NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick**: The tick makes "the aviary continues without the viewer" true and makes clients readers of canonical state rather than writers of personality.

- **Multi-device sync through additive server-authored deltas**: The rationale is to make "no last-write-wins" structurally unreachable. Personality changes are additive and ordered by server time rather than arbitrated between clients.

- **Per-invite, opt-in, read-only ambient visits**: The visit is the only social affordance because the plan refuses social-network surfaces. It is opt-in, per-invite, and read-only to avoid profiles, follows, feed, co-presence, comments, leaderboards, or show-off rendering.

- **Visitor events never written to the host event log**: "Visitor attention must not drift the host's birds." The host's birds remain the host's simulation, not a product of someone else's watching.

- **Revocable visit links**: Revocation makes the visitor's next snapshot pull return a matter-of-fact "visit no longer available" surface, so access control is immediate and system-voiced.

- **Silent visit log**: The visit log is on-demand and has no badge or push, preserving "notice, never announce" and avoiding social notification pressure.

- **Off-by-default visit notification toggle**: The toggle exists only as an opt-in quiet per-friend notification, never onboarding default or general notification surface.

### Architecture And Domain Model

- **Simulation plane and operational plane split**: The split enforces the privacy commitment. The simulation database is never connected to analytics, and the two planes share no data store or ETL path.

- **Preact for frontend chrome**: Preact is chosen for the small runtime, defending the <2MB bundle. The chrome is also code-split out of the first-paint path.

- **Custom WebGL2 canvas renderer**: WebGL2 is chosen to hit 60fps with birds, parallax, leaf drift, and reduced-motion cross-fades. The plan says DOM/SVG animation risks layout thrash and fails the runtime budget.

- **Canvas2D fallback for day-one render**: The fallback is a cheap path for the first render when needed.

- **Raw WebAudio API**: The plan avoids a heavy audio library and keeps calls procedural, which supports variation and the bundle budget.

- **Node services with shared `zod`/OpenAPI schema**: Shared type definitions make the wire format "one source of truth" across client and API services.

- **Go tick worker**: Go is recommended because the tick is CPU-bound numeric work on a tight cadence; goroutines and predictable GC help hold the p99 tick-latency budget.

- **Postgres primary datastore and partitioned event log**: Postgres fits v1 scale for accounts, birds, notebook, invites, and the append-only event log. The ingest interface is written so a later move to Kafka/Redpanda would not change client contracts.

- **Redis for sessions, nonces, rate limiting, and snapshot cache**: The plan assigns Redis to short-TTL and fast lookup work: session-token cache, magic-link nonces, rate limiting, and snapshot cache.

- **First state snapshot inlined at the edge**: This is called "the single biggest lever" on the <500ms time-to-first-bird budget.

- **Canonical terminology exported in shared glossary enums**: The plan wants "bird," "call," "mood," "personality vector," and the rest enforced so "the wrong word can't be typed."

- **Encrypted email plus keyed `email_hash`**: The hash allows login lookup and per-email rate limiting "without storing plaintext as a key."

- **`birds.personality` as server-written JSON with schema**: The current vector equals seed plus deltas, and the schema field makes recalibration auditable.

- **`perch_zone` as a signal, never user-set**: NOT RECOVERABLE FROM PLAN

- **`last_greeter_rank`**: The plan uses it to support notebook salience such as "pip greeted before wren today" without creating a user-behavior metric.

- **Append-only `event_log` as the only client write target for simulation**: This keeps clients from mutating simulation state directly and makes event folding, replay, and idempotency possible.

- **Ordering by `server_ts` plus `event_uuid`**: The server clock is authoritative because client clocks are untrusted and only useful as ordering hints.

- **`tick_consumed_at`**: This supports idempotency and replay-safety when ticks fold events into deltas.

- **`personality_deltas` table**: The table is the safety net for "losing a personality vector is the worst failure." It makes drift reconstructible as seed plus summed deltas.

- **`inputs_summary` with bounded numeric inputs and no qualitative text**: The plan keeps audit/correctness data bounded and avoids storing qualitative per-user text in delta records.

- **Bounded `mood_history`**: The plan keeps recent transitions for narration continuity and debugging while making it not user-visible.

- **Read-only `notebook_entries` with no edit/delete/annotate column**: "Un-editability is structural," so the notebook remains an observed field record, not an editable user journal.

- **`presence_windows`**: These derived rows feed drift and anonymized session-duration histograms, while the operational plane receives only anonymized aggregates.

- **`invites` and `visits` separated from host `event_log`**: The separation prevents visitor presence from changing the host's birds and keeps visits read-only.

### Simulation Engine

- **Per-account tick lease**: Advisory locking or `SELECT ... FOR UPDATE SKIP LOCKED` ensures exactly one worker ticks an account, preserving the tick as the sole writer under a worker fleet.

- **Folding events into bounded numeric inputs**: The tick turns interactions into bounded numbers per bird so drift can be calibrated and audited without qualitative logs.

- **Low-pass filter for drift**: The plan chooses a slow filter "so no single session moves a trait visibly." It prevents intense sessions from producing sudden personality change.

- **Monotonic-up clamp**: The clamp is called "the load-bearing asymmetry." It ensures neglect cannot reduce traits; low-activity ticks produce zero delta.

- **Drift calibration tests**: Tests guard the most important constants: detectable change around 7 days, visible change around 21 days, no visible change from one binge, zero negative movement after absence, and no one-session curiosity saturation.

- **Presence as the dominant drift input**: The weights are ordered "presence >> listen-in > offers" so ordinary watching matters more than clicking.

- **Mood transition FSM**: Mood probabilities combine recent interactions, local time, ambient events, personality, and bird-to-bird spread to make mood responsive without exposing numbers.

- **Mood persistence across sessions**: The plan rejects "snap to neutral" so a bird's last and intervening states matter when the user returns.

- **Server call intent with client synthesis**: The server decides motif, count, and timing shape while the client renders procedurally. This keeps canonical state server-side and lets the client schedule smooth audio.

- **Near-future call intents in snapshots**: The snapshot carries planned onsets and parameters so the client can schedule synthesis and interpolate smoothly between snapshots.

- **Recognizability invariant**: A bird's motif identity stays stable across mood and drift so the bird remains itself by ear.

- **Presence sanity cap**: Implausibly continuous presence is capped to defend the population drift signal from stuck or spoofed clients.

- **Notebook salience score**: Salience prevents the notebook from becoming a generic event log. It selects moments like first-greeter changes, quiet stretches, chorus events, or a wary bird coming forward.

- **Template-with-slots notebook grammar**: The grammar is deterministic, naturalist, testable, privacy-clean, and voice-safe. The plan rejects an LLM for v1 because a model call could drift into announcement voice or leak simulation-plane data.

### API Surface

- **`GET /v1/aviary/snapshot`**: The endpoint returns kilobytes, not megabytes, so the client can render canonical state quickly and edge-cache briefly per session.

- **Client pulls snapshot on visibility change, long render-frame gap, and low-frequency keepalive**: These pulls make suspend/resume and return behavior reflect the server-running aviary.

- **Client interpolation using `snapshot_seq` and motion intents**: Interpolation makes perch changes smooth and "never teleport."

- **Single guarded `POST /v1/events`**: A single ingest path ensures clients write only interaction events and have no personality-write path.

- **Server-derived listen-in duration**: Duration is derived from start/end pairs server-side so the attention signal is not trusted directly from the client payload.

- **Auth request endpoint always returns 200**: This prevents an account-existence oracle.

- **Auth consume invalidates link immediately**: Immediate invalidation enforces single-use magic links before issuing a per-device session token.

- **Bird rename endpoint changes name only**: Rename never touches personality, mood, or call, preserving stable bird identity.

- **Server-selected new-bird offer with no catalog**: NOT RECOVERABLE FROM PLAN

- **Hard cap 7 enforced server-side**: Server enforcement makes the recognizability cap a product invariant, not a client preference.

- **Visit-by-token read-only session**: The visitor sees the same birds, moods, and drift as the host, with "no show-off rendering" and no write capability.

- **Unused invite links silently stop working**: This keeps expired/revoked access matter-of-fact and non-notification-like.

- **`GET /v1/account/visits` with no badge or push**: The host can inspect visits on demand without turning visits into a social attention loop.

- **Settings endpoint in matter-of-fact voice**: Settings and accessibility controls are system surfaces, so they intentionally use the matter-of-fact register.

### Frontend Rendering Pipeline

- **Single full-viewport canvas with background, middle, and foreground planes**: The plan wants a layered "window on a real morning," with gentle parallax rather than a showpiece.

- **Responsive layout solver that never crops a bird out of frame**: Keeping birds in frame preserves the single-screen aviary and prevents the scene from losing its subjects on narrow or wide viewports.

- **SVG-authored birds baked into a texture atlas**: The atlas keeps species poses compact and efficient for the WebGL2 renderer.

- **Plumage saturation as a shader uniform**: This makes richer plumage drift a cheap per-frame uniform change rather than re-rasterization.

- **Procedurally varied idle controller**: Phase randomization and parameter jitter prevent idle motion from reading as a looped cycle.

- **No mood status icon, tooltip, or label**: The plan says if the user must be told what a bird feels, "the affective contract is broken."

- **First frame already in motion**: The client starts with birds mid-action so the aviary feels as if it had been rendering all along.

- **No spinner, fade-from-static, wake-up animation, or entry sequence**: The plan says "a spinner says machine; we are not selling a machine."

- **Slow-load quiet field**: The quiet field lets a slow path degrade as the aviary "catching up" rather than as machine loading.

- **Empty-aviary quiet field before first bird**: After first adoption, "the user never sees an empty aviary again," preserving continuity.

- **Return-greeting intent from the server**: Server-provided greeting intent encodes absence length, bird selection, and procedural variation so the greeting is grounded in canonical state.

- **Client-side leaves and feathers**: These ornaments keep the scene alive between bird actions without adding per-leaf simulation state.

- **Day/night palette shifts and weather overlays**: These make the scene respond continuously to local time and ambient events that also shape mood.

- **Top bar with exactly four icons**: The plan keeps chrome out of the aviary scene and limits the bar to account/settings, accessibility settings, field notebook, and offer.

- **Top bar fading nearly transparent**: A constantly opaque bar "reads as an app frame"; fading makes it an "ignorable thin layer."

- **Render loop halts when hidden**: Halting rendering defends battery and the no-memory-growth budget while the server-side simulation continues.

### Audio Pipeline

- **Procedural synthesis with no recorded audio ever**: The plan says identical recorded calls break the spell, stacked loops create audible artifacts, and recorded variation would violate the bundle budget.

- **Species motif library with personality- and mood-shaped variation**: This preserves species signature and individual call character while letting calls change with mood and drift.

- **Pooled WebAudio nodes and buffers**: Pooling avoids per-call allocation leaks and feeds the no-memory-growth CI test.

- **Spatial pan by perch zone**: Panning reinforces proximity: front birds feel closer, back birds more ambient.

- **Runtime chorus mixing**: Independent procedural calls create an emergent chorus, while limiting prevents clipping and recognizability remains preserved.

- **Listen-in slow gain ramp**: Slow ramps make focus feel like attention, not a hard audio cut.

- **Graceful silence with captions on WebAudio failure**: Silence-with-captions is preferred over canned audio because the no-recorded-audio rule is unconditional.

- **Audio context created/resumed on first gesture without announcement chrome**: The plan avoids "click to enable sound" UI because it would announce the machine rather than preserve the aviary surface.

### Accessibility Surfaces

- **Naturalist screen-reader narration**: The narration is generated from the same canonical state and grammar as the notebook so the user hears "one product," not a state list.

- **Slow narration cadence with priority bump**: Slow updates avoid flooding the screen-reader queue; user-initiated events can be prioritized but remain observations rather than state transitions.

- **Reduced-motion mode**: It is "a different rendering of the same aviary" with cross-fades, slowed color shifts, continuing calls, continuing drift, and notebook notices. The plan rejects a broken-looking static fallback.

- **Call captions generated from actual procedural call grammar**: Runtime captions match what was played and support audio-off, hearing differences, noisy environments, and WebAudio fallback.

- **Keyboard navigation**: Keyboard paths make birds, listen-in, offer, settle, and top-bar items operable without pointer input.

- **Soft high-contrast focus indicators**: The focus treatment must read against both bright and dim aviary states.

- **WCAG AA contrast on all user copy**: Contrast is a floor for top-bar labels, settings, account/error surfaces, captions, and visually displayed narration.

### Performance And Observability

- **Initial JS bundle <2MB gzipped**: The plan uses Preact, a small custom renderer, procedural audio, atlas assets, and code splitting so first paint stays light.

- **Time-to-first-bird <500ms**: This budget protects the central conceit. The inlined snapshot, CDN edge path, and first-bird-before-non-critical-assets path all serve this target.

- **60fps idle motion on a 5-year-old laptop**: The plan treats a 30-minute session, not minute one, as the runtime budget, which is why WebGL2, pooled buffers, and hidden-tab halting matter.

- **No memory growth over 30 minutes**: Pooled audio resources, virtualized notebook rows, and bounded workers/contexts make memory a tested invariant.

- **Operational metrics with aggregate-only privacy boundary**: Request metrics, tick latency, RUM timings, and audio errors catch regressions without per-bird state or per-account interaction history.

- **p99 tick latency alarm above 5s**: Tick latency is the leading indicator for server-side simulation health.

- **Synthetic browser fleet**: Automated browsers catch first-bird and frame-timing regressions before users do.

- **No per-account engagement metric or retention dashboard backed by per-bird data**: Such dashboards would require the aggregation the plan forbids.

- **Copy-voice lint**: The lint preserves "notice, never announce" by flagging announcement words, user-behavior observations, and register violations.

### Rollout And Risks

- **Simulation core first**: The plan says risk concentrates in drift, mood, event log, and tick correctness, and that drift is cheap to get wrong but expensive to discover late.

- **Auth and accounts second**: The plan calls this the "boring-with-teeth layer," reflecting the importance of identity, sessions, UUID discipline, and deletion before richer surfaces.

- **Snapshot read path, event write path, and interpolation before renderer polish**: This proves server-authoritative sync and "no client personality write" end-to-end on two devices.

- **Accessibility in lockstep with rendering, audio, and interactions**: Accessibility is gated with launch rather than added after, because a late reduced-motion mode would tell users "the product wasn't for them."

- **Social after core interaction and accessibility work**: The plan sequences invite/visit after the core aviary, audio, interactions, and accessibility surfaces; a separate rationale for this sequencing is NOT RECOVERABLE FROM PLAN

- **Birds-per-aviary ramp by aviary age**: Age gating avoids teaching that "more attention earns more stuff."

- **Tick fleet scales horizontally by account leases**: Per-account leased work lets the fleet scale while p99 tick latency indicates when capacity is strained.

- **Instrument from day one, aggregate-only**: The plan wants first-bird timing, frame timing, tick latency, audio errors, bundle size, and memory in place without creating per-account engagement or retention metrics.

- **Drift calibration as highest-leverage risk**: If too fast, it becomes "Tamagotchi-by-clicking"; if too slow, "screensaver." Tests and versioned config guard the number hardest.

- **Sync correctness and personality loss as worst-case risk**: Additive deltas, single-writer tick leases, idempotent replay, stable bird IDs, and no last-write-wins make silent overwrite recoverable or structurally unreachable.

- **Audio uncanniness as spell-breaking risk**: Procedural-only calls, listening tests, independent synthesis, and the 7-bird cap protect the chorus from becoming fake or repetitive.

- **Accessibility regression as product-quality risk**: Designed narration and reduced-motion surfaces, shared canonical state, voice lint, and launch gates keep accessibility from becoming a stripped fallback.

- **"Notice, never announce" erosion as a likely future risk**: Copy lint, code review checks, restated non-goals, off-by-default visit notifications, and the bird greeting as the only welcome surface defend against reasonable-looking violations.

- **First-bird performance miss as conceit risk**: The inlined snapshot, bundle gate, synthetic perf checks, and quiet-field fallback make loading either fast or at least non-machine-like.

- **Presence-signal corruption as silent drift risk**: Strict conjunction, ping stopping, server caps, and calibration tests protect drift from inflated or spoofed presence.

- **Privacy-boundary erosion as architectural risk**: Synthetic UUIDs, grep/lint gates, separate credentials, no analytics path from simulation DB, and never-computed leaderboard/discovery metrics prevent later exposure.
