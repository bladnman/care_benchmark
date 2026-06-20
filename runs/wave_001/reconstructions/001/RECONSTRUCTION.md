## System-level intent

- Server-side simulation is the load-bearing way to make the product real. The plan says "the single most important property of the system is the server-side simulation tick," because it makes "feels alive, not robotic," "the aviary continues without the viewer," and "multi-device sync coherent" literally true. This shows up again in Architecture, Sync model, Simulation engine design, and the repeated rule: "The simulation is server-side."

- Canonical state belongs on the server; the client is "a renderer and an event-ingestor." This appears in the Client/server split, the Render pipeline boundary, the Sync model, and the rule that "the client never writes personality state." The plan's philosophy is to prevent client forks, local replays, and last-write-wins behavior by giving the tick sole authority.

- The aviary should "notice, never announce." The plan makes the bird greeting "the entire welcome surface," rejects a "Welcome back" toast, avoids an audio-enable modal, uses a fading top bar, and lets the first frame appear already in motion rather than as an entry animation or spinner.

- The product is deliberately non-game and non-Tamagotchi. The plan refuses "achievements, streaks, levels, scores, badges, XP," refuses hunger, distress, death, or decaying happiness, ties new birds to aviary age "not to visit count or interaction total," and says the age pacing is a "load-bearing refusal of the gamification trap."

- Drift is care without punishment. The plan repeatedly says drift is "monotonic toward expressive," "never subtracts," and that "neglect is not punished." The intended feeling is that an unattended bird is "quieter than it was, not sadder than it was."

- Personality must be felt, not inspected numerically. The plan says the user "never sees personality numerically," implements this at the "API boundary" through "derived render hints" rather than raw scalar values, and treats even a staff debug view showing scalars as a regression.

- The voice is sparse naturalist prose, except for errors. The notebook prose is "lowercase, present-tense, naturalist, specific," the screen-reader narration uses the same current-state observation voice, and API errors are explicitly "matter-of-fact English" and "never naturalist."

- The notebook observes the aviary, not the user. The plan says entries are "rare naturalist observations," "not per-session counting," with "no visit-frequency observations of the user," and repeats the rule: "Pip greeted first today" is in scope while "You visited every day this week" is out of scope.

- Privacy is structural, not cosmetic. The plan uses a synthetic account UUID "anywhere" except encrypted email storage, keeps telemetry "aggregate-only," refuses per-account visit frequency and session duration metrics, and says a log line containing an email is a "PII leak."

- Social is narrow, private, and opt-in. The plan calls visit-invitation "the only social affordance," forbids profiles, follows, comments, discovery, co-presence, and default notifications, and picks "privacy-correct" visitor re-validation over a lower-friction URL.

- Accessibility ships as part of v1 and is a designed surface. The plan says reduced motion is "not a stripped fallback," "the user still gets the actual product," and repeats that "Accessibility ships with v1, not after."

- Audio should be procedural, recognizable, and ambient rather than canned. The plan rejects "recorded audio," "audio loops," and a "Spotify-of-birds," makes per-bird recognizability a playtest target, and treats the procedural call synth as "the affective spine."

- Performance is part of the product feeling. The plan ties inline snapshots, no spinner, <500ms first bird, <2MB bundle, 60fps idle motion, and no memory growth to the promise that the aviary is present, already alive, and not mechanically loading in front of the user.

- Calibration is expected to come from careful observation, not fixed formulas alone. The drift learning rates are "the most sensitive parameter," the 1-week/3-week targets are "a calibration band," and the rollout includes opted-in drift trajectories, weekly review, and tunable settings.

## Per-feature whys

### Scope

- Single-user accounts: the plan's rationale is to avoid shared aviaries, household models, and multi-user aviaries; the account model is built around one canonical aviary per account.

- Email magic-link sign-in: the plan gives security and privacy reasons through 15-minute TTL, single-use tokens, rate limits, and 202 responses "even for unknown emails" to avoid enumeration.

- Per-device session tokens: the plan's rationale is revocability and device-specific presence accounting; sessions carry `device_id`, can be listed and revoked, and presence is "per-device, summed server-side."

- One canonical aviary per account: the plan's rationale is coherent multi-device sync; both devices read "the same canonical record on `sim`" and there is "no per-device fork."

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN

- Hard cap of seven birds: the plan's rationale is the birds-per-aviary ramp and refusal of gamification; the cap is enforced in `sim` adoption logic and tied to long-term age pacing rather than activity.

- Six-species pool at launch: NOT RECOVERABLE FROM PLAN

- New birds offered based on aviary age: the plan says this is the "load-bearing refusal of the gamification trap," because pacing is tied to `created_at`, "not to visit count or interaction total."

- Server-side simulation tick: the plan's rationale is that the aviary "continues without the viewer," sync remains coherent, and the system feels "alive, not robotic."

- Personality vector with five traits: the plan's rationale is to drive drift, render hints, mood and call behavior while keeping the raw numbers server-only and invisible to the user.

- Monotonic personality drift: the plan's rationale is that "neglect is not punished"; the bird gets "quieter than it was, not sadder than it was."

- Mood persisted across sessions: the plan's rationale is continuity; "a session-open does not reset mood to neutral."

- Procedural call grammar per species: the plan's rationale is recognizable birds without recorded loops, small bundle size, and per-bird calls that remain identifiable across mood and drift.

- Three-perch zone layout with no user placement: the plan's rationale is server-authoritative perch choice by mood and personality, so the laptop and phone show the same aviary rather than a client-local arrangement.

- Day/night cycle anchored to local timezone: the plan's rationale is that mood, sky color, and palette are driven by the account's local time; the first paint already has the account-tz-correct color.

- Rare ambient weather: the plan's rationale is to create ambient events that affect mood and notebook-worthy observations without depending on user activity.

- Ambient micro-motion: the plan's rationale is the aviary appearing alive on first paint and at idle rather than static.

- Return-greeting: the plan's rationale is "notice, never announce"; the greeting replaces a welcome toast and varies by absence length, boldness, and mood.

- Listen-in: the plan's rationale is to rebalance attention toward one bird's call while keeping the rest "audible-ambient, never silent."

- Offer of seed, song fragment, or still pool: the plan ties offers to interaction events, mood transitions, and drift inputs. Specific rationale for the three offer types themselves: NOT RECOVERABLE FROM PLAN

- Per-bird offer cooldown: NOT RECOVERABLE FROM PLAN

- Settle: the plan's rationale is to support a post-settle lighting state, quiet calls through `settled` mood, and shift the scene toward evening.

- Five-second settle undo: NOT RECOVERABLE FROM PLAN

- Field notebook: the plan's rationale is sparse naturalist observation, not a feed; notability is based on simulation deltas rather than per-session counting.

- Presence accounting by visibility, focus, and recent pointer or key activity: the plan's rationale is precision against drift inflation while "leaning long because watching birds without moving is the actual product."

- Visit-invitation social feature: the plan's rationale is a private, revocable, read-only social affordance that avoids profiles, feeds, co-presence, chat, comments, discovery, and leaderboards.

- Host visit log: NOT RECOVERABLE FROM PLAN

- Opt-in visit notifications off by default: the plan's rationale is that the aviary does not ping the user and host visit notifications are never surfaced during onboarding.

- Screen-reader running narration: the plan's rationale is to give current-state naturalist observation from the same snapshot as the visual scene, with slow cadence so it does not overwhelm the screen reader queue.

- Reduced-motion mode: the plan's rationale is accessibility parity; it is a "different aesthetic, not a degraded one."

- Call captions: the plan's rationale is to keep captions synchronized with the actual procedural calls by generating them from call-plan entries at runtime.

- WCAG AA contrast: the plan's rationale is readable user-copy across the chrome, captions, settings, account, and error surfaces.

- Full keyboard navigation: the plan includes it as a v1 accessibility surface. Specific rationale for the exact tab and arrow-key order: NOT RECOVERABLE FROM PLAN

- Visible focus indicators: the plan's rationale is that focus must read against bright, dim, night, and settled palettes by computing outline color against the current day-phase palette.

- <2MB gzipped initial JS bundle: the plan's rationale is performance at first paint; code splitting, lazy species assets, procedural calls, and compact visuals all serve this budget.

- <500ms time-to-first-bird: the plan's rationale is that the first bird renders before the API round trip from the inline bootstrap snapshot.

- 60fps idle motion: the plan's rationale is smooth idle life on older hardware, with `requestAnimationFrame`, SVG, and AudioWorklet separation.

- No memory growth over 30 minutes: the plan's rationale is stable long-running ambient use, enforced by CI and bounded buffers/contexts.

- WebAudio fallback to silence plus captions: the plan's rationale is graceful no-audio behavior while preserving the unconditional "no recorded audio" rule.

- Last-two-major-versions browser support: NOT RECOVERABLE FROM PLAN

- Account export: the plan names JSON snapshot email to the verified address. Specific product rationale: NOT RECOVERABLE FROM PLAN

- Soft-delete with 30-day recovery then hard-delete: the plan names recovery by sign-in during the window. Specific product rationale: NOT RECOVERABLE FROM PLAN

- Aggregate operational telemetry only: the plan's rationale is privacy; metrics that require per-bird or per-account state are not collected.

### Decisions

- 120-second presence activity window: the plan says it leans long because "watching birds without moving is the actual product."

- 60-second simulation tick with jitter: the plan's rationale is regular server-side state advancement while avoiding herding all accounts onto the same wall-clock second.

- Personality trait range [0.0, 1.0] with 0.95 soft cap: the plan's rationale is that traits "never pin to 1.0 and the math stays smooth."

- Mood set `{wary, content, curious, drowsy, alert, settled}`: the plan's rationale for `settled` is to support "the post-settle lighting state without conflating it with drowsy."

- Notebook generation target around one entry per three days: the plan's rationale is sparsity enforced by "noteworthy" simulation deltas, not per-session counting.

- 120-second per-bird offer cooldown per offer type: NOT RECOVERABLE FROM PLAN

- Visit link TTL of 30 days outstanding and 24 hours once opened: the plan later gives the rationale that the host invited a specific email, not a URL, and the flow picks "privacy-correct" over low-friction.

- Magic-link TTL and rate limits: the plan's rationale is security and anti-enumeration around email sign-in.

- Indefinite notebook retention: NOT RECOVERABLE FROM PLAN

- Three-second top-bar fade threshold: the plan's rationale is the "notice, never announce" surface and keeping chrome quiet until cursor movement or keydown.

### Architecture

- CDN-served edge HTML and static bundle: the plan's rationale is fast first paint with a small inline encrypted snapshot so the first bird renders before the API round-trip.

- Quiet-field color fallback: the plan's rationale is a calm missing-bootstrap state without spinner or fly-in.

- Stateless horizontally scalable API: the plan's rationale is that writes go to `sim`'s event log and reads come from `sim`'s snapshot cache, keeping the public API from owning canonical state.

- `sim` service: the plan's rationale is one owner for canonical aviary state, personality vectors, mood, call config, and tick scheduling.

- Snapshot cache as the only API read path: the plan's rationale is that clients never read live in-memory state through a different path, preserving a single authoritative snapshot stream.

- `notifier`: the plan's rationale is to send auth, visit, export, and opted-in visit emails while not pushing to the client or pinging about the aviary.

- Separate `notebook-writer`: the plan's rationale is that notebook prose generation is "heavier, slower, and more LLM-shaped" and should run on a different cadence and resource pool than the tick.

- Client-owned rendering, audio synthesis, presence detection, optimistic UI, keyboard/focus, narration cadence, reduced motion, and ornament generation: the plan's rationale is to keep presentation local while canonical personality, mood, perch, call timing, notebook, and drift remain server-owned.

- Server-emitted snapshot: the plan's rationale is a compact per-tick source of truth for rendering and interpolation.

- Client snapshot pulls on focus, suspend gaps, and visible keepalive: the plan's rationale is to refresh after tab and device gaps while skipping redundant transfers with ETags.

- First frame from bootstrap snapshot: the plan's rationale is no entry animation, no fade-from-static, no spinner, and an aviary already present.

### Data model

- Stable bird UUID never reused: NOT RECOVERABLE FROM PLAN

- Synthetic account UUID rather than email as bird `account_id`: the plan's rationale is privacy; email is stored once encrypted and never used in logs, telemetry, shard keys, kafka, or identifiers.

- Renameable bird name with no effect on identity/personality/mood/call: the plan's rationale is stable identity independent of user-assigned name.

- Append-only drift history for audit and export only: the plan's rationale is audit/export, not runtime behavior.

- Coarse render hints instead of raw personality values: the plan's rationale is that the user never sees personality numerically and the API boundary enforces that rule.

- Mood drivers struct: the plan's rationale is to make transitions depend on recent interaction, time of day, ambient event, and personality baseline.

- Presence windows rather than raw presence-time submitted by the client: the plan's rationale is that the server computes duration from server-confirmed timestamps and does not trust the client clock for drift math.

- Append-only interaction event log: the plan's rationale is that the event log is the only thing the client writes to and the tick consumes it in canonical order.

- Notebook entries with `observed_at_sky`: the plan distinguishes the in-aviary moment from write time. Specific rationale: NOT RECOVERABLE FROM PLAN

- Read-only notebook API: the plan's rationale is that the notebook is auto-generated observation, with no update/delete route.

- Account email encrypted at rest with KMS-managed key: the plan's rationale is privacy and PII containment.

- Host-only visit log visibility: the plan names it as host-only and not exposed to visitors. Specific rationale: NOT RECOVERABLE FROM PLAN

### API surface

- Matter-of-fact API errors: the plan's rationale is the named voice exception; errors are never naturalist and never stack traces.

- Magic-link request always returns 202: the plan's rationale is no email enumeration.

- Magic-link consume invalidates token and issues per-device token: the plan's rationale is single-use authentication and revocable device sessions.

- Account session list and revoke routes: the plan's rationale is revocable per-device sessions.

- `/aviary/snapshot` with ETag: the plan's rationale is the steady-state renderer route and skipped redundant transfers.

- Batched `/aviary/events`: the plan's rationale is client-as-event-ingestor; the server accepts events but does not echo personality or mood changes because the client learns via the next snapshot.

- `/aviary/presence` defensive 30-minute cap: the plan's rationale is protection against drift inflation from a stuck signal.

- Paginated notebook read with no mutation routes: the plan's rationale is sparse read-only notebook entries and indefinite retention.

- Visit invitation creation: the plan's rationale is private one-time visitor access via email rather than public discovery or social surfaces.

- Visit revocation returning 410: the plan's rationale is immediate revocation with a matter-of-fact "visit no longer available" surface.

- Visitor snapshot without event submit or notebook access: the plan's rationale is read-only ambient visits; the notebook is the host's.

- Visit notification opt-in route: the plan's rationale is default-off host notifications.

- Account settings routes: NOT RECOVERABLE FROM PLAN

### Simulation engine design

- Tick reads recent event log since last consumed event: the plan's rationale is event-sourced canonical updates in server order.

- Presence aggregation across devices: the plan's rationale is to feed account-level `presence_time_total` from per-device windows.

- Drift computation from presence, listen-in, offer, and settle quieting: the plan's rationale is care-shaped expression while settle is "mood-quiet only, no drift."

- Mood transitions per current state and drivers: the plan's rationale is birds responding to recent interactions, local time, ambient weather, and personality baseline.

- Server-side perch re-evaluation: the plan's rationale is that perch zone is server-authoritative and the client never picks a perch.

- Server-side call plan for the next 60 seconds: the plan's rationale is that the client renders calls through synth but "does not invent calls."

- Ambient scheduler for weather: the plan's rationale is low-frequency weather state that mood and notebook logic can observe.

- Notebook-worthy detection classifier: the plan's rationale is rarity, first-of-week, weather correlation, and drift threshold crossings rather than per-session counting.

- Writing canonical snapshot and bumping ETag: the plan's rationale is a single cache read path for the API and clients.

- Low-pass additive drift formula: the plan's rationale is smooth, monotonic, asymptotic change where one week is measurable and three weeks is visible.

- Fuzz test asserting drift never subtracts: the plan's rationale is guarding the "load-bearing rule" against adversarial absence or malformed inputs.

- Mood timer gates: the plan's rationale is to prevent immediate or implausible state changes, e.g. `wary` must remain at least 5 minutes absent strong positive input.

- Personality baseline as transition probability bias: the plan's rationale is modulation rather than rule replacement.

- Per-species motif library: the plan's rationale is recognizable timbre and species-specific calls with bounded variation.

- Per-bird stable variation seed: the plan's rationale is per-bird recognizability across time, mood, and drift.

- Call-plan parameter envelope audit: the plan's rationale is to avoid overlapping call identities too often within the same aviary.

### Sync model

- No client-to-client sync: the plan's rationale is that both devices read the same canonical record on `sim`; there is no client-side state to merge.

- Conditional snapshot pulls with ETags: the plan's rationale is one ETag-stream over time and no per-device fork.

- Local interpolation overwritten by next snapshot: the plan's rationale is to hide recent snaps and make a visible catch-up after suspend, proving the aviary "continued without the viewer."

- No last-write-wins personality conflicts: the plan's rationale is that clients never write personality at all.

- Soft local mood preview for settle: the plan's rationale is immediate visual feedback that never feeds back into drift and is reconciled by the tick.

- Server timestamps and monotonic sequencing for events: the plan's rationale is protection from clock skew and wrong-ordered drift math.

- Auth/session-level sync errors only: the plan's rationale is that architecture precludes per-bird reconciliation errors.

### Frontend rendering pipeline

- One horizontal scene with background, middle, and foreground planes: the plan describes composition. Specific rationale for the three-plane structure: NOT RECOVERABLE FROM PLAN

- Responsive scene preserving birds onscreen: the plan's rationale is that birds remain onscreen at all viewports, compressing rather than cropping.

- Procedural SVG-with-inlined-bitmaps bird rendering: the plan's rationale is compact visual assets and simple shapes that support performance.

- Mood-shaped idle micro-motion: the plan's rationale is to make state visible through pose and idle choice.

- Snapshot-seeded animation phase: the plan's rationale is that two devices render the same bird in the same pose at the same moment.

- Perch-to-perch tweens: the plan's rationale is smooth, mood-shaped motion between server-determined perch zones.

- Mood-to-pose cross-fades: the plan's rationale is a visual transition when snapshot mood changes.

- Day-to-evening palette tween: the plan's rationale is local-time and settle-driven lighting rather than abrupt color jumps.

- Settle lighting and call quieting: the plan's rationale is a post-settle evening state driven by server-side `settled` mood and longer call gaps.

- Reduced-motion cross-fade path: the plan's rationale is "not animations off" but a designed alternate aesthetic.

- Removal of leaf/feather drift in reduced motion: the plan's rationale is to reduce motion while preserving calls, notebook, simulation, and drift.

- Aviary appears in motion on first paint: the plan's rationale is no fade-from-static, no spinner, and first paint at snapshot-determined perches and phase.

- Post-adoption first-bird fly-in exception: the plan says this is a "deliberate exception" and "by spec." Specific rationale: NOT RECOVERABLE FROM PLAN

### Audio pipeline

- WebAudio oscillators, envelopes, filters, and shared reverb: the plan's rationale is species timbre and a chorus that sounds like birds "in the same place rather than birds in separate studios."

- No recorded loops or canned audio fallback: the plan's rationale is unconditional adherence to procedural calls and bundle-size/performance constraints.

- Chorus mixing as overlapping per-bird plans: the plan's rationale is that chorus "emerges" rather than being a separate mode.

- Per-bird gain by vocal hint, listen-in state, and mood: the plan's rationale is a mix that reflects personality and state.

- Listen-in gain ramp: the plan's rationale is a 700ms re-balance where other birds remain "audible-ambient, never silent."

- WebAudio unavailable fallback: the plan's rationale is graceful silence with captions on by default, without recorded audio.

- AudioContext created on first user gesture: the plan's rationale is browser autoplay compliance without a "click to enable audio" modal.

### Accessibility surfaces

- Polite live-region narration: the plan's rationale is screen-reader narration that coexists with event prose and avoids pre-empting or overwhelming the queue.

- Server-authored narration prose from the same generator as notebook: the plan's rationale is a unified voice across aviary and notebook surfaces.

- Client cadence throttle for narration: the plan's rationale is that high-frequency narration is explicitly avoided.

- Opt-in call captions with auto-on in no-audio fallback: the plan's rationale is captions for calls when audio is absent and user control otherwise.

- Captions near the calling perch: the plan ties placement and timing to the call. Specific rationale for near-perch placement: NOT RECOVERABLE FROM PLAN

- Keyboard route to listen-in, offer, settle, and notebook/top bar: the plan's rationale is full keyboard navigation for the v1 surface.

- Adaptive high-contrast focus outline: the plan's rationale is visibility against both bright and dim aviary states.

- AA contrast for all user copy: the plan's rationale is a minimum contrast floor across chrome and text surfaces.

### Performance budgets and observability

- Code-splitting account/settings/visit/notebook/offer assets: the plan's rationale is to keep the initial bundle under <2MB gzipped.

- Lazy-loading species silhouettes and motif libraries: the plan's rationale is first paint without waiting for every species asset.

- Inline snapshot at the edge: the plan's rationale is first bird visible before the API round-trip and <500ms on mid-tier mobile over 4G.

- Edge replication of active snapshot cache: the plan's rationale is a hot snapshot near the user.

- `requestAnimationFrame` render loop: the plan's rationale is 60fps local tweening without round-trips.

- SVG as starting render substrate: the plan's rationale is browser portability and simple-enough bird shapes.

- AudioWorklet for synth: the plan's rationale is to keep audio from contending with the main-thread render loop.

- Thirty-minute synthetic memory CI: the plan's rationale is to enforce no heap growth in a long ambient session.

- Synthetic browser fleet: the plan's rationale is scheduled performance and tick-latency checks from common geographies.

- Real User Monitoring: the plan's rationale is operational measurement of page load, first bird, frames, audio-context errors, and tick latencies without per-bird or per-account dimensions.

- Tick-latency p99 alarm at 5 seconds: the plan's rationale is catching degradation before users notice the aviary "running slow."

- Deliberately not measuring visit frequency or session duration: the plan's rationale is privacy; those would require per-account state in telemetry.

### Rollout

- 100% new sign-ups and ramped waitlist invites: the plan's rationale is to surface drift calibration issues at small N before big N.

- `sim` read-only shadow period: the plan's rationale is that `sim` is the only canonical-state writer and rollout risk.

- Birds-per-aviary age thresholds: the plan's rationale is to pace adoption by aviary age, not visit count, interaction total, or paid tier.

- Staff-only configurable age thresholds: the plan's rationale is tuning pacing from real adoption data without code changes.

- User "aviary audit": the plan's rationale is making drift visible only as the user's own naturalist-prose summary, not numbers.

- Staff calibration dashboard for opted-in alpha cohort: the plan's rationale is calibrating the drift LPF without aggregating across accounts or exposing it to users.

### Risks and mitigations

- Drift calibration instrumentation: the plan's rationale is that too-fast drift becomes Tamagotchi, too-slow drift becomes screensaver, and user reports are a late signal.

- Event ordering mitigation with a per-account sequencer: the plan's rationale is that wrong event order silently corrupts drift.

- Audio playtests: the plan's rationale is that uncanniness is "felt, not measured" and the synthetic fleet cannot catch it.

- Accessibility structural tests and code review: the plan's rationale is to prevent quiet degradations like pre-stored captions drifting out of sync.

- Server-side presence validation: the plan's rationale is preventing drift inflation across the user base.

- Visit email re-validation on each open: the plan's rationale is "privacy-correct vs. low-friction" because the host invited a specific email, not a URL.

- `sim` multi-AZ hot replication and 30-minute graceful client rendering: the plan's rationale is that `sim` failure is invisible until users notice stale moods, so operational alarms and a user-facing buffer are needed.

### Load-bearing refusals and out-of-scope confirmations

- No welcome toast: the plan's rationale is that it would violate "notice, never announce."

- No streak counter, green-dot calendar, or visit-frequency surface: the plan's rationale is the absolute refusal of gamification.

- No numerical personality debug view: the plan's rationale is that personality is never exposed numerically, even staff-only.

- No client-side tick: the plan's rationale is that it would collapse multi-device sync.

- No email in logs or identifiers: the plan's rationale is PII containment.

- No recorded audio fallback: the plan's rationale is the unconditional procedural-audio rule.

- No native apps, payments, social network surfaces, co-presence, scene customization, hunger, death, distress, achievements, profiles, follows, comments, mutual visits, or discovery: the plan treats these as "load-bearing refusals, not as missing features."
