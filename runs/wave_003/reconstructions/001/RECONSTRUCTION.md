## System-level intent

- Principle: The product claim is affective aliveness. The executive summary says the "entire product claim is affective" and depends on birds that "feel alive" because the "server-side simulation advances whether or not anyone is watching." This shows up again in the first-frame requirement: "the first painted frame is the aviary, mid-action - never a load state."
- Principle: Canonical truth belongs to the server. The plan repeats that "the server is the only writer of canonical state," "the client is a renderer and an event source, nothing more," and "there is one canonical aviary state, in one database, written by one code path."
- Principle: The client should feel expressive without owning truth. The canonicality table separates "Canonical simulation state," "Render-time behavior," and "Pure ornaments"; the plan says the server decides "who/what/why" while the client decides "exactly how it looks and sounds."
- Principle: Sync is designed out, not patched in. The plan's vocabulary is "there is nothing to sync," "no merge, no reconciliation, no last-write-wins," and "last-write-wins corruption structurally unreachable." This appears in the architecture, API schema shape, event log, tick writer, and failure-mode discussion.
- Principle: The product has two clocks and local ephemeral variation. The executive summary names "Two clocks, two layers": a server-only slow clock for personality drift and a fast clock for mood, calls, and motion, plus a "purely local layer" for ornaments, exact call notes, and micro-motion timing.
- Principle: Drift is visible over weeks, never as a session-by-session number. The plan says traits drift "monotonic toward expressive," "no numeric personality readouts," "visible to the user" only through behavior and render parameters, and "no trait ever decreases." The calibration lab encodes "measurable in instruments" by week 1 and "visible to the user" around week 3.
- Principle: Privacy is structural plumbing. The plan says "Privacy is an architectural boundary, not a policy" and later "the boundary as plumbing." Email exists in one encrypted place, every other reference is a synthetic UUID, raw per-bird interaction data does not enter telemetry, and no third-party path receives bird data.
- Principle: Refusals are load-bearing. The plan calls the no-toasts, no-streaks, no-badges, no-gamification, no notifications, no recorded-audio, no native-app boundaries "refusals" and says they get "engineering enforcement" through lints, closed enums, schema shape, checklists, and telemetry allowlists.
- Principle: The product voice is split into naturalist and matter-of-fact registers. The appendices define "Naturalist voice" for aviary, notebook, narration, captions, offer microcopy, and onboarding, and "Matter-of-fact voice" for sign-in, sessions, sync/error surfaces, account settings, accessibility settings, visit management, export/delete, and unsupported-browser.
- Principle: The product should not announce itself. The plan forbids textual welcome surfaces, toasts, badges, counts, default visit notifications, launch counters, and in-product visit indicators. It describes quiet pull surfaces and says a simultaneous chorus "would announce the arrival"; staggering makes it "the aviary noticing."
- Principle: Procedural variation is part of the charm contract. The plan repeatedly uses "never identical twice," "real procedural variation," "device-local variation," and the audio warning that "once the user hears the same call twice, exactly the same way, the spell breaks."
- Principle: Accessibility is a designed surface. The plan says accessibility is "a designed surface, not a parity checklist," giving screen-reader users prose that "feels alive," reduced-motion users "a calmer rendering with its own charm," and audio-off users captions in the product's voice.
- Principle: Performance budgets are product promises, not merely engineering hygiene. "Time-to-first-bird," "60fps sustained," and "no client memory growth" are tied to the central conceit; the plan says the quiet field softens slow starts because otherwise the product reads as "loading" where "the conceit matters most."
- Principle: Calibration and observability must avoid production behavior mining. The plan says "drift calibration runs exclusively on synthetic persona accounts in staging," forbids per-account and per-bird telemetry, and rejects an A/B framework because it could "optimize against the product's soul."

## Per-feature whys

### Executive summary and scope

- Feature: Browser-only, single-user virtual aviary - Rationale: The plan frames the product as "a small, quiet, browser-only aviary" whose claim is affective aliveness, and explicitly refuses native apps and social-network surfaces so the architecture does not reserve space for them.
- Feature: Server-side simulation advancing while nobody watches - Rationale: This is the mechanism behind birds that "feel alive"; the aviary "has been continuing without the viewer," so return state is not created at the moment of return.
- Feature: Client as renderer and event source - Rationale: Keeping the client from owning or merging state makes multi-device sync "a property of the architecture rather than a feature" and makes last-write-wins corruption "structurally unreachable."
- Feature: One canonical aviary per account - Rationale: A single database state avoids merge and conflict surfaces; the plan says "there is one canonical aviary state" and therefore "there is nothing to sync."
- Feature: Two starter birds - Rationale: Starter selection uses two distinct species and is weighted toward `warbler` and `wren` so first-encounter recognizability is higher and the plan's own voice samples/common reality line up.
- Feature: User-assigned renameable bird names - NOT RECOVERABLE FROM PLAN
- Feature: Name limit of 24 characters - NOT RECOVERABLE FROM PLAN
- Feature: Stable internal bird identity forever - Rationale: The plan says "losing a personality vector amounts to deleting the bird"; a UUID-forever identity preserves continuity through renames, species-pool changes, migrations, and restores.
- Feature: Seven-bird cap - Rationale: The plan ties the ceiling to recognizability, chorus density, soak tests, and the invariant that a user should know a bird's call "by ear" at seven-bird density.
- Feature: Age-gated adoption offers - Rationale: Adoption should match a slow rhythm where "a few months old offers a third bird" and a year-old aviary may have grown further, while avoiding engagement-metric-driven pacing.
- Feature: Exact adoption gate schedule values - NOT RECOVERABLE FROM PLAN
- Feature: No gamification, Tamagotchi mechanics, native app, push notifications, payments, or social network surfaces - Rationale: The plan treats these as "refusals" and says they are "not deferred features"; they would change the relationship by making absence costly, measurable, or socially optimized.
- Feature: Engineering guardrails for refusals - Rationale: The plan identifies "well-meaning contributor adding 'just one' announcement or engagement surface" as the largest threat, so refusals are enforced by copy lint, no-toast UI kit, schema shape, PR checklist, and telemetry allowlist.

### Architecture

- Feature: Four deployables plus datastores - Rationale: The plan chooses "deliberately few services" because v1 scale is small and "every service boundary is a place canonical state could leak or lag."
- Feature: Stateless API service - Rationale: It serves pulls and ingests events while keeping "no in-memory canonical state," so it can scale horizontally without becoming a second writer.
- Feature: Simulation service as the only writer of traits, moods, and canonical positions - Rationale: This preserves single-writer discipline for drift and prevents state leaks across service boundaries.
- Feature: Auth service as a separate deployable - Rationale: Credential-handling code gets "its own review and deploy bar."
- Feature: Transactional mail provider with no bird data in email - Rationale: The third-party boundary stays clean against the privacy commitment; emails contain links and system copy only.
- Feature: PostgreSQL 16 primary store - Rationale: The plan says the key invariants, including append-only event log, per-account sequence, advisory locks, and transactional snapshot writes, are "exactly what Postgres makes boring."
- Feature: Redis for hot snapshot cache, revocation list, scheduler queue, and rate limits - Rationale: Redis improves latency while correctness remains in Postgres because "all Redis data is derivable."
- Feature: No message broker in v1 - Rationale: At v1 scale, the event log "is the queue"; avoiding a broker removes ordering bugs and a dual-write class while leaving a Kafka-shaped migration path.
- Feature: TypeScript everywhere - Rationale: Shared engine/client types for mood enums and event taxonomy "kill drift-between-layers bugs."
- Feature: Node.js 22 LTS - Rationale: The plan calls it "mature, hireable" and sufficient for a tick-per-minute workload.
- Feature: Preact for chrome and hand-rolled canvas scene layer - Rationale: The scene is "a game-loop problem, not a DOM-diffing problem," while chrome is a small DOM problem; the split keeps the bundle small.
- Feature: Canvas 2D with layered offscreen canvases - Rationale: Seven birds and particles are within budget if static layers are cached; WebGL adds bundle and driver risk, and Canvas2D makes reduced-motion cross-fades straightforward.
- Feature: Procedural pose/skeleton bird visuals - Rationale: Procedural poses support mood-shaped idle motion, mid-action first frames, and generated assets without a heavy sprite sheet.
- Feature: Raw WebAudio API - Rationale: The plan needs "sample-accurate lookahead scheduling" and per-bird gain nodes; libraries would add bundle and abstraction.
- Feature: CDN edge functions for snapshot injection - Rationale: The first-paint path needs the initial snapshot delivered with HTML to support the under-500ms "time-to-first-bird" requirement.
- Feature: Staging calibration lab - Rationale: Drift, mood, notebook rarity, and greeting behavior are tuned on synthetic persona accounts because production user data is never used for calibration.

### Data model

- Feature: Synthetic UUIDv7 account identifiers - Rationale: Account IDs become the only identifier in tables, logs, cache keys, and queues, separating all non-auth systems from PII.
- Feature: Email encrypted once with HMAC blind index - Rationale: Email is PII stored in exactly one place; lookup by email is auth-service-only and cannot be resolved by other services.
- Feature: Visitor email stored encrypted and hashed on invitations - Rationale: Visitor email follows the same PII rule while the visit log can display it back to the host because "the host supplied them."
- Feature: `accounts.settings` for accessibility, audio, and notification settings - Rationale: These are "system-surface data" and persist across devices, so laptop and phone honor the same accessibility/audio choices.
- Feature: No streak, visit-count, or engagement-score table - Rationale: The plan intentionally makes gamification "schema-hostile"; the schema cannot easily answer "how many days in a row did this user visit."
- Feature: No mood history table - Rationale: Mood transitions are "ephemeral canonical state"; the field notebook is "the only long-term record of moments."
- Feature: Append-only event log - Rationale: Events are immutable observations consumed by the tick in sequence, making drift replayable, deduped, and ordered.
- Feature: Visitor sessions write nothing to `event_log` - Rationale: This structurally guarantees that "a visitor sitting and watching for an hour does not drift the host's birds."
- Feature: Snapshot payload as sync contract - Rationale: The snapshot is small, versioned, self-sufficient for first paint, and marks the boundary between deterministic persisted state and stochastic local rendering.
- Feature: No raw personality values in client-facing snapshots - Rationale: Traits should be felt as behavior and presentation, not read as numbers; the account export is the only user-initiated exception.
- Feature: Raw event-log pruning after 90 days - Rationale: Drift is already folded into vectors, the notebook is the user-facing record, and retaining raw events longer is "privacy-negative."
- Feature: Latest-only snapshot row - Rationale: The plan treats snapshots as the hot materialized current state; no historical snapshot product surface is articulated.
- Feature: Notebook entries kept for account lifetime - Rationale: The notebook is the indefinite user-facing scrollback of the aviary's life, unlike pruned raw events.
- Feature: Backup rotation bounded after deletion - Rationale: Deleted data ages out of PITR windows and the privacy policy can state that plainly.

### API surface

- Feature: JSON over HTTPS with `/v1/` prefix - NOT RECOVERABLE FROM PLAN
- Feature: HttpOnly, Secure, SameSite session cookie - Rationale: Session tokens are signed and revocable while browser-accessible script cannot read them.
- Feature: Matter-of-fact error model - Rationale: Errors are system surfaces, so messages are direct, user-displayable, and never naturalist; they also avoid email addresses and account UUIDs.
- Feature: Idempotent state-changing requests with client-generated event IDs - Rationale: Offline replay and retries are safe because duplicates return the original result.
- Feature: Conditional snapshot pulls with `If-None-Match` - Rationale: Low-frequency keepalives become cheap because unchanged snapshots return 304.
- Feature: No client-submitted state fields - Rationale: Request bodies contain observations, IDs, durations, names, settings, and timestamps, preventing clients from writing simulation state.
- Feature: Synchronous offer endpoint - Rationale: Offers need a reaction "now," while the drift consequence remains folded by the next tick.
- Feature: Notebook read-only endpoint with no edit/delete - Rationale: Entries are immutable naturalist observations, not user-authored content or a management surface.
- Feature: Visitor read-only snapshot endpoint with no visitor event endpoint - Rationale: The API surface itself cannot record visitor presence, so visits have zero simulation footprint.
- Feature: Rate limits on auth, events, offers, invitations, and visitor pulls - Rationale: The plan names invitation emails as the only abuse vector with third-party impact and caps presence/offers so a buggy or malicious client cannot inflate drift.
- Feature: Exact auth and event rate-limit numbers - NOT RECOVERABLE FROM PLAN

### Simulation engine

- Feature: Pure-function engine library - Rationale: Determinism given state, events, tick index, and seed makes lazy catch-up identical to eager ticking and makes the calibration harness trustworthy.
- Feature: Absolute wall-clock tick indices - Rationale: They are the "determinism backbone," making state at time T a function of earlier state plus events and tick indices.
- Feature: Eager ticking for recently active accounts and dormant catch-up - Rationale: This honors the "tick runs whether or not any client is connected" guarantee without spending a "worker-year ticking accounts nobody opens."
- Feature: Bounded collapsed catch-up for long dormancy - Rationale: It bounds compute while preserving the same formulas and equivalence tests.
- Feature: Per-account advisory lock and single-transaction tick persist - Rationale: They create exactly one writer and prevent torn states.
- Feature: Keyed PRNG and `Math.random` ban in engine code - Rationale: All stochastic draws must be reproducible by purpose, bird, account, and tick; device-local randomness is reserved for render/audio variation.
- Feature: Additive, saturating, non-negative drift - Rationale: Growth is asymptotic, traits cannot overshoot, and no path can punish absence.
- Feature: Presence as dominant drift input with per-tick caps - Rationale: Presence is how users "show up," but caps prevent a marathon or spoofed session from spiking drift.
- Feature: Listen-in drift signal by duration - Rationale: Focused listening affects warmth and vocal traits for the focused bird without relying on client-submitted conclusions.
- Feature: Offer acceptance and nearness as drift inputs - Rationale: The plan maps accepted offers to curiosity and nearby offers to boldness, making small interactions shape behavior while remaining monotonic.
- Feature: Settle as mood-quieting but not drift direction - Rationale: Settle acknowledges leaving and ends presence cleanly without making departure a penalty.
- Feature: Muted and unmuted sessions in vocal drift - Rationale: Muting cannot be penalized; unmuted sessions get a small positive multiplier because "listening to calls is attention to calls."
- Feature: Rapport as a decaying behavioral expressiveness term - Rationale: It lets returning birds be "quieter" without negative personality drift, implementing ambient quietness instead of mistrust.
- Feature: Trait-to-observable presentation mapper - Rationale: It is the only place traits affect visible output, keeping raw traits hidden while making plumage, perches, call rate, and greeting behavior expressive.
- Feature: Mood enum of `alert`, `curious`, `content`, `wary`, `drowsy`, `resting` - Rationale: The judgment log says the set combines the named examples with `resting` for night while keeping the transition table legible.
- Feature: Sticky mood transition model with self-loop bias - Rationale: Moods should be sticky so "changes read as changes."
- Feature: Daily dawn mood re-anchor - Rationale: It prevents multi-day mood ruts and makes "mornings feel like mornings."
- Feature: Mood persistence across sessions - Rationale: A tab opened after time away shows what interim ticks produced; there is "no reset-to-neutral path."
- Feature: Server call activity and client call grammar - Rationale: Canonical state is rate, energy, and motif weights, not waveform; two devices can hear different renderings of the same mood without a sync bug.
- Feature: Six-species v1 pool - Rationale: The plan wants a coherent set of "birds you might see in one place," with all equally available because rarity is not a feature.
- Feature: Exact inclusion of `finch`, `thrush`, `nightjar`, and `swallow` beyond described character roles - NOT RECOVERABLE FROM PLAN
- Feature: Per-bird audio signature - Rationale: Stable pitch, formants, rhythm, and ornament bias let a user who has spent weeks with Pip know Pip's call by ear.
- Feature: Shared scene manifest for perches and anchors - Rationale: Server positions and client pixels must mean the same thing on both sides.
- Feature: Perch moves with staggered departure - Rationale: Birds animate flight or hop rather than teleport and do not move in lockstep.
- Feature: Account-local day/night signal - Rationale: Lighting and canonical mood/call effects use the same `sun_t`, so behavior and palette agree.
- Feature: Nightjar-like species active at night - Rationale: "Night is not dead"; the scene always has at least one living motion source at night.
- Feature: Account-level timezone with hysteresis - Rationale: The same laptop/phone account should show "the same aviary in the same mood"; hysteresis prevents device-hopping flip-flop.
- Feature: Pure weather scheduler - Rationale: Catch-up replays weather identically and the notebook can honestly say "a passing rain."
- Feature: Exact weather probabilities and duration ranges - NOT RECOVERABLE FROM PLAN
- Feature: Greeting director at snapshot-pull time - Rationale: The greeting is the "anchor moment" and must land within the first second or two of the session, faster than the next tick.
- Feature: Arrival-window greeting dedupe - Rationale: A refresh or second device joining mid-session should not re-greet; the directive executes once per arrival.
- Feature: Absence bands for greeting forms - Rationale: Greeting form classes should vary with absence from brief glances to re-orientation after longer absence.
- Feature: Greeting staggering and anti-repeat ring - Rationale: Staggering makes it "the aviary noticing, one bird at a time," and anti-repeat preserves never-identical procedural variation.
- Feature: Offer reaction resolver - Rationale: Reactions feel immediate while the single-writer rule for traits is preserved because drift is folded by the next tick.
- Feature: Seed, song, and still-pool offer kinds - Rationale: They create different mood-shaped observation/reaction surfaces: approach or hesitation, call response or listening, drink/bathe/watch.
- Feature: Per-bird offer cooldown - Rationale: Cooldown keeps curiosity drift from saturating in a session and quiets the affordance without making cooldown an error.
- Feature: Field notebook template grammar rather than LLM - Rationale: The privacy commitment forbids sharing per-account interaction data with third parties, and templates are auditable for voice.
- Feature: Notebook content restricted away from user behavior - Rationale: Entries may say Pip greeted first but may never say the user has been there daily; the notebook observes birds, weather, light, and scene.
- Feature: Notebook salience and rarity budget - Rationale: Entries should be rare, moment-based, "sparse-but-not-dead," and not a reward for visiting.
- Feature: Notebook template cooldowns - Rationale: They prevent repeated structures from becoming noticeable over weeks.
- Feature: Pull-only adoption surface - Rationale: The user goes to the offer; it never comes to the user by toast, badge, email, or announcement.
- Feature: Three-signal presence gate - Rationale: Presence is only banked when visible, focused, and recently active, protecting the slow drift relationship from background-tab inflation.
- Feature: Presence activity window of 180 seconds - Rationale: The judgment log says this follows "a few minutes, lean longer" and starts long because too-short failures are visible while too-long failures are silent.
- Feature: Server-side presence plausibility validation - Rationale: The client sees focus/visibility, but the server caps, dedupes, gap-breaks, and rejects stale uploads so drift cannot be inflated past the calibrated ceiling.

### Sync model

- Feature: Device-local event queue mirrored to IndexedDB - Rationale: Offline or crash survival works without client ownership of canonical state.
- Feature: Server-assigned `account_seq` processing order - Rationale: The tick consumes events in server receipt order, identical regardless of which device sent which event.
- Feature: Snapshot pull triggers on load, visibility, resume gap, keepalive, and sync interactions - Rationale: The client keeps its rendered keyframes fresh without doing any canonical ticking.
- Feature: Snapshot interpolation and version-jump easing - Rationale: Nothing should pop or teleport; after suspend, the aviary has been running and the ease reads as "the viewer's eye catching up."
- Feature: Failure behavior rendering last snapshot - Rationale: The product degrades gracefully because canonical truth resumes on reconnect and the scene remains visually alive enough.
- Feature: Matter-of-fact inline status after repeated snapshot failures - Rationale: A persistent silent failure would be worse, but it stays in top-bar account chrome rather than becoming a toast over the scene.
- Feature: Duplicate-tab handling - Rationale: Presence dedupe prevents double banking, while per-tab greeting is acceptable because each tab is a viewing surface.

### Frontend rendering pipeline

- Feature: Edge-injected first paint - Rationale: It removes the origin round trip from first paint and makes the first bird appear within the affective threshold.
- Feature: First frame mid-action - Rationale: The scene must not read as starting up; action phase in the snapshot means the first frame is already alive.
- Feature: Quiet field slow path - Rationale: The loading state should be "indistinguishable from a distant view of a calm aviary," with no spinner, skeleton, progress bar, or logo animation.
- Feature: Marketing-free sign-in and onboarding - Rationale: Sign-in is a matter-of-fact system surface, while new-account naming uses naturalist voice without becoming marketing.
- Feature: One lifetime fly-in after adoption - Rationale: The plan allows one entrance animation for arrival, then treats future empty bird lists as error states so the user never sees an empty aviary again.
- Feature: Layered scene composition with cached offscreen layers - Rationale: Static layers cached plus bounded particles keeps 60fps on budget hardware.
- Feature: Procedural bird skeleton and pose system - Rationale: Pose blending supports normal motion, reduced-motion cross-fades, call posture, and mid-action first frames through one mechanism.
- Feature: Plumage saturation and feather detail as drift output - Rationale: Drift becomes visible on the bird without numeric personality readouts.
- Feature: Deterministic signature markings - Rationale: Same-species birds remain visually distinguishable at a glance.
- Feature: Idle micro-motion scheduler - Rationale: "Birds are never still in a way that reads as paused"; motion is mood-shaped and canonical enough to read mood while local enough to avoid sync.
- Feature: Mood changes with no status labels - Rationale: Mood should be read from motion; "the moment the user has to be told, the contract fails."
- Feature: Settle transition - Rationale: The aviary "acknowledges the goodbye" as soft collective settling without confirmation text or reward/punishment.
- Feature: Five-second settle undo - NOT RECOVERABLE FROM PLAN
- Feature: Weather transitions - Rationale: Weather should be eased, never abrupt or assertive, matching the quiet product tone.
- Feature: OKLCH day/night palette engine - Rationale: OKLCH interpolation makes perceptually smooth dawns and avoids muddy RGB midpoints while preserving night legibility.
- Feature: Weather rendering as particles and posture offsets - Rationale: Weather affects mood and scene, but per-particle state remains a pure local ornament.
- Feature: Ambient leaves and feathers - Rationale: The scene and birds acknowledge each other without canonical events; ornaments are shed before bird micro-motion on weak hardware.
- Feature: Reduced-motion rendering strategy - Rationale: It is a "designed surface, not a fallback," giving the same canonical scene a calmer register and shipping with v1.
- Feature: Responsive single horizontal composition - Rationale: All birds remain visible, no pan/scroll/zoom shifts attention to geography, and the aviary does not become a panorama.
- Feature: Top bar with exactly offer, notebook, accessibility, account/settings - Rationale: The closed enum blocks creeping badges, counts, and extra chrome.
- Feature: Top bar fade with keyboard focus pin - Rationale: Chrome recedes when idle, but fading must never hide focus.
- Feature: Hidden-tab lifecycle behavior - Rationale: Rendering and audio halt for battery, events flush best-effort, and on return the client catches the eye up to the continuing server state.

### Audio pipeline

- Feature: WebAudio graph with per-bird voice chains - Rationale: Synthesized oscillator/noise/filter voices honor the no-recorded-audio rule and bundle budget.
- Feature: No `AudioBuffer` sample libraries - Rationale: Both the bundle budget and unconditional no-recorded-audio rule require oscillator/noise/filter synthesis only.
- Feature: Motif DSL per species - Rationale: Calls are compact data, not audio, and can be composed into species-recognizable procedural calls.
- Feature: Stable per-bird audio signature in snapshots - Rationale: Mood and drift change delivery, not identity; the user learns "this" bird's voice.
- Feature: Emit-time jitter and motif-history fingerprinting - Rationale: Exact repeats break the spell, so repeats are rejected by a data structure rather than hoped away.
- Feature: Lookahead scheduler - Rationale: Scheduling against `AudioContext.currentTime` keeps calls sample-accurate even when main-thread frames drop.
- Feature: Chorus coupling - Rationale: Chorus should emerge when high-vocal birds overlap, but a cap keeps it a chorus rather than a wall.
- Feature: Audible call-and-response - Rationale: Responses should sound like responses, not independent loops.
- Feature: Subtle stereo placement by x-position - Rationale: The judgment log treats this as "free spatial aliveness" inside the fixed scene, not camera panning.
- Feature: Gentle bus compressor - Rationale: It glues overlapping calls without pumping.
- Feature: Listen-in mix rebalance - Rationale: The focused bird rises while others fall only to an ambient floor; listen-in is "the re-balance, never a mute."
- Feature: Audio ramp helper and direct value lint ban - Rationale: Hard cuts are review-blocking because they produce clicks and break the slow-rise feel.
- Feature: Autoplay-policy handling - Rationale: The plan accepts that browsers forbid pre-gesture audio and minimizes the affective cost without an interstitial or "tap to enable sound" modal.
- Feature: Song-fragment offer library - Rationale: Fragments are composed motif DSL data so reacting birds can grammar-compose against them and captions can be generated from descriptors.
- Feature: Exact six-fragment song library size - NOT RECOVERABLE FROM PLAN
- Feature: Graceful silence plus captions fallback - Rationale: Silence loses sound but keeps events, call postures, caption emission, and narration; no recorded-audio fallback exists.
- Feature: Fixed-size audio queues and node cleanup - Rationale: The audio side must satisfy the 30-minute no-growth rule and keep live node counts flat.

### Accessibility surfaces

- Feature: Server-generated screen-reader narration - Rationale: One server-side grammar from canonical state guarantees voice consistency, keeps the client thin, and gives SR users prose rather than state lists.
- Feature: Narration cadence and priority bumps - Rationale: The product should describe the place slowly and promptly narrate user-initiated moments without interrupting.
- Feature: Polite live region - Rationale: "The product does not interrupt people"; priority changes queue order rather than using assertive announcements.
- Feature: Show narration text setting - Rationale: Some users read better than they listen, so the narration can be made visible.
- Feature: DOM focus proxies for canvas birds - Rationale: Canvas content is invisible to assistive tech, so each bird becomes an accessible interactive proxy.
- Feature: Proxy labels composed at snapshot cadence - Rationale: Labels should not churn and should agree with narration and captions because they share presentation-mapper sources.
- Feature: Full keyboard navigation map - Rationale: Every interaction is reachable without a pointer; canvas hit-testing is only a mouse convenience.
- Feature: Focus indicators with double-ring contrast - Rationale: Focus must remain visible against dynamic day, night, weather, and settle backgrounds.
- Feature: Call captions generated from motif descriptors - Rationale: Captions always match what actually played or would have played in silent mode.
- Feature: Limit of two simultaneous captions - Rationale: Captions should not become "subtitles-for-everything."
- Feature: Contrast-token suite - Rationale: User-copy contrast must be tested against the product's own changing backgrounds.
- Feature: Accessibility settings surface - Rationale: Controls are system-voice settings, persisted server-side, and applied immediately across devices.
- Feature: Accessibility CI and manual SR matrix - Rationale: Mechanical regressions are caught in CI, and manual screen-reader runs catch what automation cannot.

### Accounts, auth, privacy, and social

- Feature: Magic-link sign-in with always-204 request response - Rationale: It avoids account enumeration and keeps the sign-in surface matter-of-fact.
- Feature: Atomic single-use magic-link consumption - Rationale: Row-level atomicity makes replay impossible.
- Feature: 15-minute magic-link expiry - NOT RECOVERABLE FROM PLAN
- Feature: Active sessions list and revocation - Rationale: Users can see device labels and revoke sessions; API access is immediate and edge-injected paint catches up within 30 seconds.
- Feature: Thirty-day sliding session expiry - NOT RECOVERABLE FROM PLAN
- Feature: Email change verified at the new address - Rationale: The old email remains the sign-in credential until the swap completes transactionally.
- Feature: Twenty-four-hour email-change token - NOT RECOVERABLE FROM PLAN
- Feature: Account export with personality vectors - Rationale: This is the one sanctioned numeric exposure because it is "the user's own data leaving through the user's own hands."
- Feature: Seventy-two-hour signed export link - NOT RECOVERABLE FROM PLAN
- Feature: Soft-delete followed by hard delete - Rationale: The 30-day window gives recovery, then hard deletion removes keyed rows, Redis keys, and exports while backup rotation bounds residue.
- Feature: Pending-deletion account bar - Rationale: It is an account-surface exception to no banners because it is system state, not an aviary announcement.
- Feature: Privacy DB grants and no ETL/warehouse connector - Rationale: Per-bird interaction events drive only that user's simulation and are never aggregated for model training or population analysis.
- Feature: First-party-only RUM beacon with fixed schema - Rationale: Product health can be measured without cookies, identifiers, or third-party analytics.
- Feature: Visit invitations - Rationale: Social is host-initiated, opt-in, read-only, quiet, and small; nothing exists until a host sends an invite.
- Feature: One-time visit link exchanged for short visit session - Rationale: The link is one-time while the session makes refreshes not fragile; revocation still kills access.
- Feature: Seven-day visit-session lifetime - Rationale: The plan uses it to bound exposure while preserving a non-fragile visit after one-time exchange.
- Feature: Visitor render-only client - Rationale: Visitors see the same host snapshot, not a prettified mode, while data omissions and disabled interactions keep them from changing anything.
- Feature: Minimal visitor chrome and accessibility icon - Rationale: Visitors are read-only, but accessibility is not host-gated.
- Feature: Visitor zero simulation footprint - Rationale: Visit-scoped cookies cannot call event endpoints, so host drift comes from host presence only.
- Feature: Visit log - Rationale: It provides host-visible transparency without badges, pushes, or a social feed.
- Feature: Visit notification toggle off by default - Rationale: A host can opt into one matter-of-fact email per new visit, but no in-product indicator or onboarding mention revives notification pressure.

### Performance, observability, testing, rollout, and team shape

- Feature: Bundle, first-bird, frame-rate, memory, snapshot, tick, edge, and ingest budgets - Rationale: The budgets measure the product's promises: quick aliveness, sustained motion, no leaks, fresh ticks, and cheap snapshots.
- Feature: Runtime frame-budget governor - Rationale: Weak hardware degrades gracefully by shedding ornaments first; bird motion is never fully shed because "a frozen aviary is worse than a slow one."
- Feature: Aggregate-only operational and RUM metrics - Rationale: Observability should track health without account, bird, email, or behavioral dimensions.
- Feature: Deliberate absence of A/B experimentation - Rationale: Experimentation infrastructure could become engagement-optimization infrastructure and conflict with privacy and the product's soul.
- Feature: Synthetic check fleet - Rationale: Product promises like first-bird, frames, memory, auth, visitor flow, listen-in, offer, and settle are checked from geographies and device profiles before users are disappointed.
- Feature: Alarms and error budgets - Rationale: The product tolerates brief degradation by rendering the last snapshot, so budgets can be modest, but tick freshness and snapshot availability still gate reliability work.
- Feature: Unit and integration tests per workstream - Rationale: The plan's invariants are testable: monotonicity, no traits in API, migration identity, presence truth table, focus paths, and governor behavior.
- Feature: Drift calibration lab personas - Rationale: Calibration is an executable harness using synthetic accounts, not production behavior mining, with week-1 instrument visibility and week-3 user visibility as explicit targets.
- Feature: Presence-honesty scenario suite - Rationale: Named failure modes such as overnight background laptop, unfocused second monitor, and spoofed ping rates become executable tests.
- Feature: Audio recognizability and uncanniness protocol - Rationale: Machines check variation and spectral invariants, while human ears check recognizability, naturalness, and repetition.
- Feature: Memory and frame soak - Rationale: The 30-minute no-growth and 60fps rules are CI gates, not guidelines.
- Feature: Determinism and equivalence tests - Rationale: Eager and lazy ticking, cross-device canonical equality, and replay safety are tested as the foundation of sync correctness.
- Feature: Week-6 vertical slice - Rationale: One bird end to end is "the product's thesis in one bird"; later work is breadth.
- Feature: Internal dogfood - Rationale: "Feels alive" is a judgment made by people living with the aviary for months, not by a dashboard.
- Feature: Closed beta with qualitative feedback - Rationale: The beta is "personal-invite style" and watches for audio uncanniness, greeting repetition, narration verbosity, and presence surprises.
- Feature: Server-side config flags and kill switches - Rationale: Tuning can happen without a client release, but every flag is reviewed and tested so config cannot invert monotonicity or break promises silently.
- Feature: Six named workstreams - Rationale: The engine is the spine, accessibility is not a phase, and observability/privacy enforcement must exist from M0.
- Feature: Design-owner-of-voice review - Rationale: Every template, email, error string, and label follows the two-register voice guide as a reviewable artifact, "not a vibe."
- Feature: Definition of done checklist - Rationale: Release requires all CI gates, calibration reports, human audio and SR protocols, synthetic fleet, failure drills, privacy audit, guardrail review, and on-call readiness to be green.
