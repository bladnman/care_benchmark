## System-level intent

- **Treat v1 scope and non-goals as architecture, not absence.** This shows up in the scope section's claim that exclusions are "standing constraints" and "architectural exclusions," with the example that "a field that exists is a field someone eventually surfaces." The product intent is to keep gamification, Tamagotchi mechanics, and social-network surfaces out of the system shape, not merely off the UI.

- **Ship accessibility as part of the product's first-class surface.** The plan repeatedly treats narration, reduced-motion, and captioning as things that "cannot ship late," with beta requiring "visits, narration, reduced-motion, and captions live from day one" and risks mitigated by "accessibility sign-off as a release gate equal to the visual surface."

- **Keep the bird world canonical on the server, but make liveliness local.** The plan says "the server is the sole writer of personality and mood" and "there is no 'latest write wins'"; at the same time, "moment-to-moment liveliness" comes from "client-local procedural simulation" driven by "slow-changing server parameters." This shows up in sync, tick design, interpolation, flinch animation, frontend pose scheduling, and audio scheduling.

- **Expose personality as expression, never numbers.** The plan resolves the export conflict by saying "never the raw `[0,1]` scalars" and routes outward data through one "presentation projection" function. The same intent appears in snapshot responses, plumage tokens, call-rate descriptors, exports, logs, and internal tooling.

- **Prefer simple v1 systems that fit the product's actual scale.** Architecture is "five deployables, deliberately not more for v1"; Postgres is chosen over an "event-sourced/Kafka pipeline"; polling is chosen over WebSocket because a WebSocket "would be solving a problem this product doesn't have." The plan favors forward scaling levers over premature infrastructure.

- **Make performance and memory budgets structural gates.** The plan treats the "2MB gzip bundle cap," "<500ms time-to-first-bird," "60fps idle motion," and "No memory growth over 30 minutes" as CI or perf-lab gates that "fail the build" or are "a real gate, not a guideline." The edge snapshot, code-splitting, DOM/SVG choice, and audio memory discipline all support this.

- **Use a naturalist voice that observes only true facts.** Notebook and narration are grounded in phrase banks, "ObservedFact" rows, and "specific-but-never-invented" prose. The plan warns that a notebook stating something false "breaks trust in the whole product's voice in one stroke."

- **Preserve privacy by construction.** The plan uses "synthetic `Account.id` UUID" everywhere cross-service, stores email in "exactly one encrypted column," forbids per-bird/per-account telemetry dimensions, rejects geolocation weather, and states that calibration "cannot tune drift constants by aggregating real users' personality data."

- **Keep calibration tunable without making the product gameable.** Drift constants and species-unlock thresholds live in config, synthetic harnesses validate bands, and the drift risk is framed as avoiding both "a Tamagotchi the user can game" and "a screensaver."

## Per-feature whys

### Scope

- **Two starter birds**: NOT RECOVERABLE FROM PLAN

- **Cap of seven birds**: The plan ties seven to rendering and audio constraints: "birds (max 7...)" are "well within the 60fps budget," and audio voice count is "capped at the bird cap (7)."

- **Magic-link single-user accounts**: NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account**: The sync rationale is that both devices read "the same canonical record"; "No client-to-client reconciliation exists or is needed."

- **Multi-device sync**: The plan's why is to make the two-device race "unreachable": clients send events, the server applies ordered increments, and "there is no 'latest write wins.'"

- **Field notebook**: The plan's why is to render rare, trustworthy naturalist prose from `ObservedFact`; it must not invent comparisons because a false notebook entry "breaks trust."

- **Presence accounting**: The plan uses presence as the dominant drift input and caps gaps so "a missed ping or a stale background tab can't be credited as continuous presence."

- **Visit-invitation opt-in, off by default**: The plan treats visits as the feature to slip if capacity forces sequencing because it is "opt-in anyway" and can be "behind a feature flag, OFF for everyone."

- **Screen-reader narration**: The plan's why is that a screen-reader user and a sighted user should experience "the same product, voiced two ways, not two products."

- **Reduced-motion mode**: The plan's why is accessibility parity: it is a "full design, not a fallback" and "a fully designed alternate rendering, not a strip-down."

- **Call captioning**: The plan's why is to keep captions aligned with actual generated calls: caption text is looked up from the same motif/mood parameters that drove synthesis, "so the caption always matches what actually played."

- **Web only**: NOT RECOVERABLE FROM PLAN

- **Native apps exclusion**: NOT RECOVERABLE FROM PLAN

- **Gamification surface exclusion**: The plan treats gamification as a standing constraint because even "a harmless engagement feature" can reintroduce "exactly what the product refuses."

- **Tamagotchi mechanics exclusion**: The drift risk says too-fast drift would read as "a Tamagotchi the user can game"; the plan wants slow, non-gameable change.

- **Social-network surfaces beyond the single visit affordance exclusion**: The plan frames social-network surfaces as part of the architectural exclusions and limits v1 to the single visit affordance.

### Architecture

- **Edge layer with embedded state snapshot**: The why is "<500ms time-to-first-bird" and avoiding "client round trip to origin before the first bird renders."

- **API service (BFF)**: The why is to keep auth, reads, ingestion, settings, visits, notebook, and exports in a "stateless HTTP service" that is "Horizontally scalable" with "no in-process state."

- **Simulation tick service**: The why is workload separation: batch, CPU-light, latency-sensitive work scales "proportional to account count, not request volume."

- **Scheduler / sweep jobs**: The plan gives this component recurring housekeeping work: enqueue due accounts, sweep expired invites, sweep hard deletes, evaluate unlocks, expire stale links.

- **Transactional email service integration**: The why is to use "A managed provider... not a bespoke mailer."

- **Postgres primary datastore**: The why is relational transactional behavior: the no-LWW rule depends on atomically reading the event log and writing vectors "inside one transaction with a per-account lock."

- **Snapshot cache**: The why is first paint and cheap polling: it feeds the edge embedded snapshot and lets the API answer polls "without hitting Postgres on every request."

- **Object storage for exports**: The plan's why is serving "account export JSON files" through "short-lived signed URLs emailed to the user."

- **Polling instead of push/WebSocket**: The plan says polling matches the spec and product physics: liveliness is local, parameters are slow-changing, and WebSocket would add "real complexity."

- **No event-sourced/Kafka pipeline at v1**: The why is that per-account event volume is low and a streaming platform is for "a scale this product doesn't have yet."

### Data model

- **Synthetic `Account.id`**: The why is privacy: cross-service references use the synthetic UUID, "never the email."

- **Encrypted account email**: The why is identifier hygiene: "Email exists in exactly one encrypted column."

- **Account timezone**: The why is narration/day-night fallback "when no client is present to supply 'now' in local time during a tick."

- **Accessibility preferences**: The why is account-level support for reduced motion, captions, and narration across sessions.

- **Visit notifications enabled default false**: The why is opt-in visits and notification behavior; the plan keeps visit-invitation "opt-in, off by default."

- **`last_tick_at` and `last_tick_sequence`**: The why is tick correctness: workers process events since the last sequence and advance the sequence to avoid double-processing.

- **Stable bird identity, never reissued**: NOT RECOVERABLE FROM PLAN

- **Bird name, renameable**: The plan identifies rename as the only client-writable bird field; no separate user-facing why is articulated.

- **Server-side `personality_vector` floats**: The why is "to do additive drift math," while never serializing the raw scalars outward.

- **Bird mood and mood start time**: The plan uses these for mood transitions, snapshot rendering, and current-state narration.

- **`mood_bias`**: The why is neighbor contagion: it carries "small transient contagion weights consumed by next tick's transition" and decays each tick.

- **Offer cooldown timestamps**: The why is "per-bird cooldown enforcement."

- **`last_greeted_session_at`**: The why is an "absence-length input + anti-repeat-within-session guard" so greetings do not refire on keepalive polls.

- **`call_seed`**: The why is deterministic client-side call and animation scheduling; it rotates on adoption and "never on rename."

- **Species seed data, not user-writable**: NOT RECOVERABLE FROM PLAN

- **Append-only `InteractionEvent` log**: The why is ordered, deduped, server-authoritative inputs for drift and sync; events are "never updated, never deleted except by hard account deletion."

- **Server-assigned globally monotonic event id**: The why is sequencing authority: client time is "never for ordering."

- **Client event id**: The why is idempotency: it is a "unique per account" key.

- **Client occurred timestamp**: The why is duration math only, "never for ordering."

- **ObservedFact ledger**: The why is ground truth for notebook prose and narration; rendered phrases must bind to queried facts.

- **Read-only NotebookEntry**: NOT RECOVERABLE FROM PLAN

- **DeviceSession**: The why is per-device sessions with list and revoke.

- **VisitInvite token, status, expiry, viewed fields**: The why is tokenized, revocable, expiring read-only visits, checked on every call so revocation takes effect on next poll.

### API surface

- **Session cookie tokens**: The why is authenticated endpoints with per-device `httpOnly`, `secure`, `sameSite=lax` session tokens.

- **Request magic link**: NOT RECOVERABLE FROM PLAN

- **Consume magic link**: NOT RECOVERABLE FROM PLAN

- **Logout**: NOT RECOVERABLE FROM PLAN

- **Device list and revoke**: The why is session visibility and revocation for devices.

- **Aviary snapshot endpoint**: The why is to be "the read path," returning deterministic rendering data and derived presentation data without raw vectors.

- **Client-side day/night calculation**: The why is that day/night is "a pure function of 'now'" and sending it would be "a stale snapshot."

- **Greeting directive in snapshot**: The why is first-session return greeting, computed only for the first snapshot fetch of a session.

- **Event ingestion endpoint**: The why is append-only sync with idempotency and server-side cooldown validation.

- **Benign no-op offer cooldown response**: The why is product voice: it is "not an HTTP error" because this is "a product surface, not a system error."

- **Notebook read endpoint**: The why is reverse-chronological, cursor-paginated access to unbounded notebook scrollback.

- **Account settings endpoint**: NOT RECOVERABLE FROM PLAN

- **Email change endpoint**: NOT RECOVERABLE FROM PLAN

- **Account export endpoint**: The why is account export via async job and signed download link, constrained by the no-raw-vector rule.

- **Account delete and restore endpoints**: NOT RECOVERABLE FROM PLAN

- **Bird rename endpoint**: The why is that name is "the only client-writable bird field."

- **No client endpoint for personality or mood writes**: The why is server authority: no endpoint ever accepts personality or mood from a client.

- **Visit invite endpoint**: The why is the single visit affordance, opt-in.

- **Visit log and outstanding invites endpoint**: The why is host visibility into visit history and outstanding invites.

- **Visit revoke endpoint**: The why is revocability; status/expiry are checked every call so revoke applies on the next poll.

- **Visitor snapshot endpoint**: The why is read-only, tokenized access with the same snapshot shape and no event submission path.

### Personality vector exposure

- **Personality expression snapshot in export**: The why is resolving the conflict in favor of the "never" numeric-exposure rule; export gets qualitative presentation data, "never the raw `[0,1]` scalars."

- **Shared presentation projection function**: The why is enforcing a single serialization path; it is the "only code path allowed to read `personality_vector`" and the "only thing ever serialized outward."

- **No raw vector in logs, debug tooling, export, API, or admin panels**: The why is that even internal "boldness: 0.62" would violate the rule "just as much as a user-facing one."

### Simulation engine design

- **Due-account tick scheduling**: The why is that ticks must run for new events and for time/weather/notebook reevaluation even with no clients, because the tick should "run whether or not any client is connected."

- **Relaxed cadence for accounts absent 24h**: The why is that no spec behavior needs "sub-5-minute precision for an absent account."

- **Per-account tick lock**: The why is preventing double-processing on retries.

- **Presence-time reconstruction with capped gaps**: The why is to avoid crediting missed pings or stale background tabs as continuous presence.

- **Listen-in duration pairing and capped orphan starts**: The why is bounded duration math; orphaned starts are capped rather than left open-ended.

- **Offer events outside cooldown as drift signal**: The why is curiosity/boldness nudge while enforcing cooldown.

- **Settle as mood-quieting only**: The why is it has "no drift direction."

- **Additive, monotonic-only drift**: The why is no absolute state writes, no negative deltas, and slow approach-to-ceiling change.

- **Mood transitions from interaction, time, weather, personality, and bias**: The why is to compute mood from recent valence, time-of-day, active weather, personality, and neighbor contagion.

- **Neighbor contagion as next-tick bias**: The why is a slow "statistical nudge, not an instant flip," consistent with once-a-minute granularity.

- **Client-local flinch animation**: The why is to deliver "felt bird-to-bird reactivity" while keeping canonical mood "single-sourced on the server."

- **Simulated ambient weather**: The why is avoiding geolocation: real-world weather would cut against "minimal-data privacy posture" for no stated benefit.

- **Perch-zone recomputation with minimum dwell**: The why is preventing visual flicker on rapid mood swings.

- **ObservedFact updates from ticks**: The why is to ground notebook facts like first greeter, quiet stretches, weather, and drift milestones.

- **Age-gated species-unlock eligibility**: The why is controlled long-term unlocking up to cap, with tunable thresholds because calibration is expected to be revisited.

- **Config-table unlock thresholds**: The why is that these values are "exactly the kind of calibration the spec says to revisit."

- **Snapshot cache write-through on every tick**: The why is to keep the read path fast and edge/API snapshots current.

- **Drift calibration bands**: The why is instrument-detectable change after about a week, user-visible after about three weeks, and "never visible within a single session."

- **Presence as dominant drift input**: The why is the PRD's stated weight ordering: "presence > listen-in > offers."

- **Phrase-bank/template notebook generation instead of per-entry LLM**: The why is deterministic latency and cost, enough diversity for sparse entries, and safer voice consistency.

- **Notebook trigger scoring**: The why is preserving sparsity with novelty and days-since-last-entry rather than a flat per-session check.

- **Return-greeting computed at read time**: The why is it must fire "within the first second or two" and cannot wait for the next scheduled tick.

- **Greeting selection and stagger**: The why is procedural variation and avoiding simultaneous greetings.

### Sync model

- **Server as sole writer of personality and mood**: The why is sync correctness and avoiding the LWW race; clients write only events.

- **Client event log instead of absolute state writes**: The why is that clients send "listened in for 3 minutes," never "set boldness to 0.62."

- **Strict server-assigned event ordering**: The why is deterministic application of additive deltas inside a locked transaction.

- **No separate sync protocol**: The why is that both devices read the same canonical snapshot.

- **Visible-tab polling aligned to tick cadence**: The why is freshness without push complexity.

- **Refetch on visibility change and render-frame gaps**: The why is suspend/resume recovery.

- **First paint from embedded edge-cached snapshot**: The why is that the first bird should not wait on polling.

- **Snapshot interpolation and smooth retargeting**: The why is making the aviary appear already in motion and avoiding snaps on new snapshots.

### Frontend rendering pipeline

- **DOM/SVG birds**: The why is that at max seven birds DOM is within 60fps, uses GPU transforms/opacity, and keeps focus/ARIA native.

- **Canvas ambient ornaments**: The why is that unbounded decoration benefits from "a single draw call" and has no accessibility semantics.

- **Client-local bird pose state machine**: The why is deterministic reconstruction of current pose and progress on fresh load, with "no spinner, no wake-up animation."

- **Procedurally parameterized layered SVG assets**: The why is small bundle size and runtime recoloring as plumage saturation drifts.

- **Quiet-field placeholder**: The why is to avoid a spinner and preserve the quiet aviary surface before a cached snapshot or first bird.

- **First-bird entrance animation only once**: The why is avoiding a recurring wake-up/entrance pattern after adoption; the bird flies in once and "never recurs after the first session."

- **Critical bundle limited to scene, audio, top bar**: The why is preserving the 2MB/500ms "bird visible" budget.

- **Lazy settings, accessibility settings, and visit flow chunks**: The why is that these chunks should not compete with the critical budget.

- **Reduced-motion cross-faded still poses**: The why is full alternate rendering while preserving calls, captions, mood, and slowed day/night shift.

### Audio pipeline

- **One WebAudio `AudioContext` per session**: The why is session audio management, lazy creation on gesture if required, battery saving on hidden tabs, and resume on visible.

- **Per-bird call-generator subgraphs into shared listen-in mix**: The why is independent bird calls with focused listening and a common mix bus.

- **Listen-in gain ramps**: The why is it must feel "like listening, not switching channels"; ramps avoid hard cuts.

- **Other birds quiet but never zero during listen-in**: The why is retaining ambient presence rather than muting the aviary.

- **Poisson-ish independent call scheduling**: The why is natural concurrency driven by vocal-frequency and mood.

- **Real-time mixed chorus**: The why is to avoid "the phase-cancellation artifact of layered recordings."

- **Caption lookup from synthesis parameters**: The why is captions match what actually played.

- **Graceful silence on WebAudio unavailability**: The why is the app can run without audio while captions default on.

- **No recorded-audio fallback**: The why is that it would feel canned and "blow the bundle budget."

- **Audio node disposal and voice cap**: The why is the 30-minute no-memory-growth test.

### Accessibility surfaces

- **Client-side screen-reader narration generator**: The why is the same snapshot feeds visual and narration surfaces, so users get the "same product, voiced two ways."

- **Polite live-region narration cadence**: The why is periodic current-state description without interrupting constantly.

- **Immediate prioritized narration for user-initiated events**: The why is timely feedback for return-greeting, offers, and settle, still as observation.

- **Roving-tabindex bird keyboard navigation**: The why is avoiding N separate tab stops while allowing arrows among birds.

- **Enter for listen-in and Escape to exit**: NOT RECOVERABLE FROM PLAN

- **Keyboard-operable offer affordance**: The why is full keyboard operation of the offer flow.

- **High-contrast focus-ring treatment**: The why is visible contrast against both daytime and night/settled backgrounds.

- **WCAG AA token checks in CI**: The why is avoiding a manual audit that drifts as surfaces ship.

### Performance budgets and observability

- **CI-enforced 2MB gzip cap**: The why is preventing bundle budget creep by failing builds rather than warning.

- **Sub-500ms time-to-first-bird**: The why is immediate bird visibility via edge snapshot and rendering first bird before non-critical assets.

- **60fps idle motion perf check**: The why is sustained smoothness on a 5-year-old laptop using transform/opacity-only animation and pooling.

- **30-minute heap soak test**: The why is proving bounded memory growth.

- **Notebook scroll virtualization**: The why is no memory growth during unbounded scrollback.

- **Synthetic multi-geography browser checks**: The why is observing time-to-first-bird and page load from multiple geographies.

- **Aggregate-only RUM**: The why is privacy: no per-bird or per-account dimension on telemetry.

- **Schema-validated telemetry allowlist**: The why is to enforce privacy structurally at ingestion, not by policy alone.

- **Simulation tick p99 alarms at 5s**: The why is latency observability for the tick service.

- **Last two major browser versions support**: NOT RECOVERABLE FROM PLAN

- **Unsupported-browser surface for older browsers**: The why is matter-of-fact fallback with "no compatibility shimming."

### Privacy and identifier hygiene

- **Synthetic UUID across logs, queues, cache, telemetry**: The why is never using email as a cross-service reference.

- **No per-bird interaction data in analytics warehouse**: The why is privacy and preventing cross-account aggregation.

- **No average drift dashboard**: The why is the account-sync privacy section names that exact example as "the thing not to build."

- **Synthetic calibration instead of real-user personality aggregation**: The why is real user per-bird data cannot be used even "for a good reason."

### Rollout

- **Internal dogfood**: NOT RECOVERABLE FROM PLAN

- **Synthetic drift calibration harness before real users**: The why is tuning constants before exposure while respecting privacy constraints.

- **Invite-only beta with full feature set from day one**: The why is avoiding accessibility and visit surfaces shipping later than the visual surface.

- **Synthetic perf checks and aggregate RUM during beta**: The why is measurement beginning with real users while staying aggregate-safe.

- **Public web launch**: NOT RECOVERABLE FROM PLAN

- **Post-launch retuning via config table**: The why is thresholds and drift constants are likely to need adjustment without deploy.

### Risks and mitigations

- **Drift calibration mitigation**: The why is avoiding both a gameable Tamagotchi feel and a static screensaver feel.

- **Sync correctness mitigation**: The why is avoiding double-applied deltas on tick-worker retry.

- **Audio-design prototyping**: The why is avoiding procedural calls that sound synthetic rather than charming and undermine the "affective spine."

- **Accessibility release gate**: The why is avoiding narration, reduced-motion, and captions "bit-rotting" after the visual surface.

- **Notebook content review and fact binding**: The why is avoiding generic/announcement-style phrasing and false fact claims.

- **CI hard gates for performance**: The why is preventing budgets from eroding silently.

- **Privacy validation failures**: The why is future contributors cannot use email or per-bird state as convenient identifiers or analytics dimensions.

- **Non-goals as standing design-review gate**: The why is preventing "gamification-creep" indefinitely, not just in v1.

- **Explicit product confirmation for export conflict**: The why is not silently resolving a genuine spec tension in code.
