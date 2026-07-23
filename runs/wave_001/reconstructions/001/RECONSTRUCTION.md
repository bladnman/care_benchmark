## System-level intent

- **Quiet, ambient, non-gamified presence.** This shows up in Scope and Risks: the plan makes "gamification of any kind" absolute out of scope, says neglect produces "ambient quietness only," bans "achievements, streaks, levels, scores, badges, counters," and warns that a "small toast saying hi" or a "harmless streak counter" would rotate the product into "a different product." New-bird offers are "an adoption moment, not a reward screen."
- **Server-owned continuity.** This shows up in Architecture, Sync model, and Risks: "the server is the only place time passes," "one canonical aviary per account, server-owned," and "clients never tick, never write personality state." The intended feeling is that "the aviary continues without the viewer" and the returning user meets "continuity, not a reset."
- **Conflict prevention rather than conflict resolution.** This shows up in the server-side tick, single-writer simulation service, and Sync model: "no last-write-wins, ever," "only the tick writes personality vectors," and conflicts are made "structurally unreachable rather than handled."
- **Small declarative state, rich local rendering.** This shows up in the render-pipeline boundary and Frontend rendering pipeline: the snapshot is "declarative scene-state + parameterized-intent," "not a frame stream," while the client owns "pixels and sound," "interpolation," "idle micro-motion jitter," "ambient ornaments," and "call synthesis."
- **Affective life through procedural variation, not canned media.** This shows up in Calls, chorus, Audio pipeline, and Risks: calls use a "call grammar," "per-bird recognizable signature," "variation seeds," and "genuinely independent synthesized voices." The plan treats "procedural calls that read as synthesized beeps" as a risk to the "affective spine," but still holds the "strict no-recorded-audio rule."
- **Slow, monotonic, presence-dominant personality drift.** This shows up in the Bird engine, Drift function, and Drift calibration risk: drift is a "slow low-pass filter," "presence-time" is dominant, the clamp floor means "there is no negative path," and calibration aims for "1-week-measurable / 3-weeks-visible" drift without becoming "clickable Tamagotchi" or "screensaver."
- **One product voice with a hard register boundary.** This shows up in the Voice contract, Accessibility surfaces, Notebook generation, copy lint, and error surfaces: "Naturalist" is lowercase, present-tense, bird-named, specific; "Matter-of-fact" is direct system copy. The plan says there is a "hard line between them" and enforces it through "copy-voice lint in CI" plus human review.
- **Designed accessibility, not fallback parity.** This shows up in Scope, Reduced-motion mode, Accessibility surfaces, and Risks: accessibility is "planned in from day one," "ships with v1," and is treated as "designed surfaces with their own charm, not parity checklists." Reduced motion is "a designed surface, not a fallback."
- **Privacy by architecture, not policy text.** This shows up in Data model, Observability, Privacy mechanics, and Privacy boundary erosion: email appears only on the account record and invite/email path; telemetry "never reads the simulation database"; the plan deliberately does not measure "anything reconstructable into a user's relationship with their aviary."
- **Visitors are ambient viewers, not participants.** This shows up in Scope, Visits API, visit events, and Sync model: visit links are opt-in, read-only, revocable, and "Visitors generate zero presence-time and zero interaction events." The visitor sees the aviary "as ambient" and receives "no greeting plan."
- **Performance budgets are product requirements.** This shows up in Scope, Frontend first paint, Performance budgets, and Rollout: the plan calls budgets "CI-enforced gates, not guidelines," requires "<500ms time-to-first-bird," "60fps idle motion," and "zero memory growth," and puts perf soak and memory CI in Hardening.
- **Operational simplicity where cadence allows it.** This shows up in Service topology, Explicit defensible calls, and Tick reliability: SSE is chosen over WebSocket "for simplicity" because cadence is "~1 update/min"; adaptive cadence changes "only compute scheduling" while the "user-visible state is identical."

## Per-feature whys

### Scope

- **Single-user accounts**: The plan connects this to "one canonical aviary per account" and a sync model where laptop and phone "both pull the same snapshots from the same record," with "nothing to merge."
- **Email magic-link sign-in**: The plan requires 15-minute, single-use links and an identical response "whether or not the email exists" so sign-in is "enumeration-safe."
- **Per-device revocable sessions**: The plan lets a device session be revoked from settings; revocation "invalidates its token at the auth layer," while already logged events remain because "they were the user's real attention."
- **Verified email change**: NOT RECOVERABLE FROM PLAN.
- **Account export**: Export is the only surface where the personality vector leaves the server because it is "the user's own data"; delivery is to the verified address via a "short-lived signed download link."
- **Soft-delete with 30-day recovery and hard-delete**: The plan preserves a recovery path during the soft window and then purges "birds, vectors, notebook, events, visit logs -- everything keyed to the account UUID."
- **One canonical aviary per account**: The rationale is architectural: multi-device sync becomes "a property of the architecture," with "no client-to-client sync, no client-side canonical state, nothing to merge."
- **Two starter birds at adoption**: NOT RECOVERABLE FROM PLAN.
- **System-selected starter species from a pool of about six**: NOT RECOVERABLE FROM PLAN.
- **User-assigned bird names and renameability**: The plan states names are renameable and have "no engine effect," keeping naming out of personality, drift, and simulation state.
- **Bird count cap of seven**: The cap is "empirical, not a plan tier"; future reconsideration requires "audio-mix work proving recognizability above seven."
- **New birds offered at aviary-age intervals**: The plan ties offers to "aviary age, not engagement" so arrival is not a reward screen and has no "you've earned a new bird!" framing.
- **Hidden five-trait personality vector**: The plan hides it from product APIs and rendering so the aviary is not numeric, while the server uses it for behavior and the export path treats it as user-owned data.
- **Monotonic-toward-expressive drift**: The plan's rationale is that absence creates "no negative drift" and reads as "ambient quietness," avoiding death, hunger, distress, decaying meters, and clickable Tamagotchi mechanics.
- **Presence-time as the dominant drift input**: The plan calibrates drift around real attention, not mere open tabs; the risk section says lax presence would cause "population-wide drift inflation."
- **Listen-in as a drift input**: The plan makes it "strong, targeted" for social warmth and vocal frequency on the listened bird, while the UI mix remains a "re-balance, never a mute."
- **Offers as a drift input**: Offers nudge curiosity or boldness, but per-bird cooldowns exist so "mashing offers cannot saturate curiosity drift."
- **Settle interaction**: The plan makes settle drift-neutral and says it "cleanly closes the presence window"; the 5-second undo window itself has no further articulated rationale beyond the specified behavior.
- **Field notebook**: The notebook exists for "auto-generated, read-only, sparse naturalist entries" that observe the aviary, not the user. The generator avoids "one-per-session" entries and user-behavior copy such as "you visited every day."
- **Presence accounting as visibility AND focus AND recent input activity**: The rationale is anti-inflation: the server rejects pings whose active-window exceeds wall-clock since the previous ping so buggy or hostile clients cannot corrupt drift.
- **Single horizontal scene with no pan, scroll, or zoom**: The plan says "all birds always in frame" and narrow viewports compress spacing "without ever cropping a bird."
- **Three perch zones**: NOT RECOVERABLE FROM PLAN.
- **Local-time day/night cycle**: The plan uses account timezone so lighting and moods follow local day phases; night is "not a dead state."
- **One night-active species**: The plan keeps night from becoming "a dead state"; most birds settle, while the nightjar-signature species remains active into late hours.
- **Rare ambient weather**: Weather gives small mood and vocal modulations, and visitors see "the same weather as the host"; the plan frames events as "rare, gentle."
- **Continuous ambient micro-motion**: The plan uses leaves, feathers, subtle parallax, idle poses, and first-frame motion so the scene feels continuous rather than restarted.
- **Thin fading top bar and no chrome inside the scene**: The plan restricts top-bar items to account/settings, accessibility settings, notebook, and offers, and bans in-scene tooltips, badges, overlays, or labels to keep the scene itself free of UI chrome.
- **Quiet field as load state**: The plan rejects "spinner," "fade-from-static," and "entry animation"; the first frame renders mid-action so motion looks "continuous, not restarted."
- **Visit invitations only**: The rationale is to keep social read-only and opt-in: invites are by email, one-time, revocable, and unused invites expire after 30 days.
- **No chat, avatars, comments, discovery, leaderboards, or show-off rendering**: The plan's rationale is to keep out "social-network surfaces" and gamification; visitors are ambient, not a social graph.
- **Visit notifications default OFF**: The plan pairs this with "No badge, no push" and no announcement surfaces, preserving quiet settings-level awareness.
- **Visitors generating zero presence-time and zero interaction events**: The plan says visitors never write interaction events, the ingestion endpoint rejects visit-scoped event writes, and visitor rendering has "no presence/drift side effects."
- **Screen-reader naturalist prose narration**: The plan grounds narration in the same canonical state as visuals, delivered as observations rather than state transitions; "Pip mood: content" is banned.
- **Reduced-motion mode**: It is a "designed surface, not a fallback"; motion becomes slow cross-fades while "Audio, captions, narration, drift, mood, notebook" remain intact.
- **Runtime-generated call captions**: Captions are derived from the same grammar parameters as sound, so they "always match what was actually played -- or would have played."
- **WCAG AA contrast**: The rationale is accessibility for "all user copy" across top bar, settings, errors, captions, and visual narration.
- **Full keyboard navigation**: The plan treats keyboard access as part of the accessibility surface: top bar, scene bird focus, listen-in, exit, offers, and settle are all reachable without a pointer.
- **Procedural WebAudio only and graceful-silence fallback**: The plan bans recorded audio "anywhere in the product" and says fallback is "silence+captions, never canned loops."

### Architecture and sync

- **Three deployable units plus edge**: The plan separates client rendering/audio, stateless API/realtime service, and single-writer simulation so each owns a distinct responsibility and the API can be "horizontally scaled."
- **Snapshot as the render-pipeline boundary**: The server sends "declarative scene-state + parameterized-intent," not frames, keeping snapshots "small (kilobytes)" and high-frequency motion client-side for 60fps at a roughly one-minute tick cadence.
- **Server-side tick**: The plan calls this "non-negotiable" because it makes "the aviary continues without the viewer" true and makes concurrent-client personality corruption "structurally impossible."
- **No client-side simulation fallback**: Offline clients show the last snapshot and a "quiet reconnecting state"; on reconnect they re-pull and cross-fade so no client advances canonical state locally.
- **SSE instead of WebSocket for live updates**: The plan chooses SSE "for simplicity" because the live cadence is about one update per minute.
- **HTTP event ingestion with idempotency keys**: The rationale is safe retries: events are deduped on client event UUID and appended in receipt order.
- **Log receipt order as authoritative**: Client timestamps are advisory because drift "doesn't care about minute-level skew."
- **Per-aviary advisory lock and single worker**: The plan enforces one tick writer per aviary so only the simulation service writes canonical state.
- **Canonical Postgres store**: It holds accounts, birds, vectors, moods, notebook, invites, sessions, and offsets as canonical state keyed by synthetic account UUID.
- **Append-only event log**: The log captures presence and interactions in order, with a short replay window for disaster recovery, but the plan says it is "an input, not a source of truth for recomputation."
- **Snapshot cache**: It exists for "fast first paint" of the latest rendered-state snapshot.
- **Object store for account-export artifacts**: Export artifacts are "short-lived signed URLs," matching the account export flow.
- **Email templates in the object store**: NOT RECOVERABLE FROM PLAN.
- **Single-writer sync model**: By making the tick the only personality writer, the stale phone overwriting the laptop case is "structurally unreachable rather than handled."
- **Matter-of-fact operational errors**: Expired magic links, timed-out sessions, and outages are system surfaces, so their copy is matter-of-fact and "never naturalist."

### Data model and API surface

- **Synthetic account UUID everywhere except encrypted email**: The plan's rationale is privacy: no log key, telemetry dimension, shard key, or internal reference may contain email.
- **Notification preferences schema review**: The account model keeps only the visit-notification toggle there and is "designed so nothing else can land here without a schema review."
- **Stable bird internal identifier**: The plan says renames, syncs, migrations, and species-pool changes "never touch it," protecting continuity.
- **Personality vector server-written only**: It is "never exposed in any API, never recomputed from logs," preventing client mutation and numeric product surfaces.
- **Seed profile**: Adoption-time seed values keep each bird's call signature recognizable, constant over drift.
- **Notebook provenance hidden from client**: `source_context_json` exists for tuning generation sparsity and is "never served to the client."
- **Invite token stored hashed**: The plan ties visit links to one-time tokens and stores only a token hash, keeping the link credential out of stored cleartext.
- **Visit log rows with no presence or drift side effects**: The plan explicitly says visit rows are displayed in settings and visitors never write interaction events.
- **Private cached owner snapshot**: Owner snapshots are personalized per account, so caching is private and authorization-keyed.
- **Read-only notebook API with no write endpoint**: The plan makes notebook entries immutable and says "no write endpoint exists at all."
- **Event batch submission with optimistic local reactions**: The client can render immediate cosmetic affordances, but "state is not" cosmetic: authoritative reaction arrives in the next snapshot.
- **Visit snapshot excluding private owner data**: Visitor snapshots omit settings, offer cooldowns, and notebook access; the visitor client is "render-only."
- **No return-greeting for visitors**: The plan says greetings are for the owner, and visitors see the aviary "as ambient."
- **Invite rate limits and outstanding-invite cap**: NOT RECOVERABLE FROM PLAN.

### Simulation engine

- **Tick cadence of about once per minute**: The plan makes exact cadence configurable and pairs it with small snapshots and low-frequency live updates.
- **Opportunistic tick before snapshot if overdue**: The rationale is that clients never see a "stale-by-hours state as current."
- **Lazy deterministic catch-up**: After worker gaps or long offline periods, catch-up is one pass over aggregated inputs rather than 1440 sequential ticks, preserving current state without excessive compute.
- **Drift as a slow low-pass filter**: The plan aims for measurable drift after about a week and user-visible drift after about three weeks, balancing "too fast" and "too slow."
- **Clamp floor of zero in drift**: The rationale is no negative path: absence is only absence of positive drift.
- **Offer cooldown enforced at ingestion and tick aggregation**: The plan blocks mashing offers from saturating curiosity drift.
- **Persisted personality vectors**: Backup/restore restores vectors "full stop"; the event log is not used to recreate personality.
- **Mood state machine**: Mood responds to offers, time of day, ambient events, other birds, and personality, making state continuous and bird-specific rather than session-only.
- **Daily-ish reset as decay, not midnight snap**: The plan says mood persists across sessions and "never visibly snaps on tab open."
- **Return-greeting computed from canonical state at snapshot request**: The rationale is that greetings react to the "true absence length."
- **One greeter with day-level stickiness**: One bird greets first, weighted by boldness and mood, so the same bird can tend to greet within a day "but not forever."
- **Parameterized greeting variation and staggered responses**: The plan says greetings are "never identical twice" and multiple birds never arrive as "a unison chorus."
- **Species call grammar**: Motif libraries plus variation rules let each species and bird remain recognizable without shipping audio.
- **Per-bird signature parameters**: Signature parameters are constant across mood and drift so the bird stays recognizable.
- **Server call schedule with client synthesis**: The schedule sends motif IDs, seeds, and timestamps; this lets captions match the played call exactly without shipping audio.
- **Bird-to-bird coupling and emergent chorus**: One bird's call can raise nearby response probability, and overlapping high-vocal windows produce chorus events.
- **Notebook sparsity governor**: The plan targets about one entry per few days for regular use, "never one-per-session," and uses sparsity as a sameness guard.
- **Notebook prose templates over real state**: Entries are naturalist voice, never numeric, and never about the user's behavior.
- **Weather scheduler**: Weather is server-side so visitors and host share the same weather; effects are "small, short-lived mood/vocal modulations."
- **Lighting from account timezone**: The server computes lighting phase so clients render a consistent local day/night state.

### Frontend rendering and audio

- **TypeScript SPA with WebGL and Canvas2D fallback**: The plan chooses a batched renderer to meet the rendering budget, with fallback only if WebGL context creation fails.
- **Procedurally assembled SVG/vector bird sprites**: Compact vector parts support personality-parameterized plumage saturation inside the bundle budget.
- **Aggressive code-splitting**: The aviary scene is the "critical path"; account settings, accessibility settings, invites, and notebook are lazy chunks.
- **First-frame mid-motion boot**: Snapshot seeds and server timestamps phase-align the first render so motion looks continuous.
- **Empty-aviary quiet field before first bird**: The plan says the first bird enters with a soft fly-in and the user never sees an empty aviary again; the deeper rationale for the exact empty-state treatment is not further articulated.
- **Responsive compressed scene spacing**: Narrow viewports compress horizontal spacing without cropping birds; wide viewports add inter-perch space.
- **Layering and subtle parallax**: The plan specifies background, middle plane, foreground branch/leaf, and subtle parallax, but a specific rationale beyond scene composition is NOT RECOVERABLE FROM PLAN.
- **Idle micro-motion parameterized by mood**: Wary, content, curious, and drowsy birds move differently, using seeds and local jitter so motion "never loops identically."
- **Snapshot-to-snapshot interpolation**: Birds glide and lighting blends; the plan says a bird "never teleports."
- **Pause discipline for background tabs**: Hidden tabs stop rendering "battery" while the server keeps ticking, then re-pull and cross-fade on return.
- **WebAudio procedural synthesis**: Motifs are oscillator/envelope/filter recipes shaped by mood and signature parameters; the plan holds this as the only audio path.
- **Chorus as independent synthesized voices**: The rationale is that stacked loops "phase-cancel and read as dead."
- **Pre-allocated reusable audio buffers**: The plan ties this directly to the zero memory-growth CI test.
- **Ambient chorus mix by perch zone and mood**: The plan specifies front birds louder and levels by mood, but a separate rationale is NOT RECOVERABLE FROM PLAN.
- **Listen-in slow ramps**: Focused bird rises and others drop slowly to ambient, "never a mute"; risk mitigation says ramp curves avoid a "switching channels" feel.
- **Settle audio quieting**: Calls quiet over the same few-second window as the lighting shift, aligning audio with the soft session-end.
- **Mute setting drift-neutral**: The plan says muting is not a drift input because the listed drift inputs are presence, listen-in, offers, and settle.
- **Graceful WebAudio fallback**: If WebAudio is unavailable or denied, captions are on by default and silence replaces sound; no recorded-audio fallback exists.
- **Runtime captions from motif shape**: Caption text comes from note count, contour, register, and tempo so it matches the call that played or would have played.

### Accessibility, observability, rollout, and risks

- **Screen-reader live region cadence**: Narration updates every 30-60 seconds at idle, with priority bumps for user-initiated events, so narration stays observational and not a state-list.
- **Keyboard focus indicators**: The plan requires a soft high-contrast outline visible against bright and dim scenes; rationale is recoverable as part of keyboard accessibility.
- **Accessibility settings in matter-of-fact voice**: The settings surface belongs to the system register, preserving the Naturalist/Matter-of-fact boundary.
- **Bundle, first-bird, frame-rate, memory, and tick-latency gates**: The plan treats these as "hard fail" release gates, not guidelines.
- **Synthetic browser checks**: They measure load timings, first-bird timings, and frame timings from common geographies to enforce product performance.
- **Aggregate RUM only**: The plan measures page-load, first-bird, frames, audio-context errors, tick latency, requests/errors, and anonymized duration histograms with "no per-account dimension."
- **No telemetry over per-bird or per-account relationship state**: The plan says no per-bird state, per-account interaction history, real-user drift curves, or anything reconstructable into "a user's relationship with their aviary."
- **Telemetry separated from simulation database**: The rationale is architectural privacy: telemetry "never reads the simulation database" and analytics never read the simulation store.
- **Synthetic-only calibration harness**: It verifies the 1-week measurable and 3-week visible drift band without inspecting real-user drift curves.
- **Build order starting with foundations**: The plan says account model, canonical schema, event log, ingestion, and snapshot delivery are "the spine -- everything hangs off it."
- **Private alpha with full v1 surface and operational kill-switches only**: The plan instruments from day one and rejects feature flags that "change the product's shape."
- **Limited beta lasting 4-6 weeks**: The rationale is that visible drift is a "3-week phenomenon," so beta must be long enough to validate drift calibration.
- **Audio uncanniness as beta exit criterion**: The plan makes laptop, phone, and headphones review explicit because beepy synthetic calls would damage the "affective spine."
- **GA without waitlist or launch-day marketing surface in-product**: The rationale is that "the product has no announcement surfaces and launch changes nothing about that."
- **Adaptive tick cadence for idle aviaries**: The plan uses deterministic catch-up so only compute scheduling changes while "the user-visible state is identical."
- **No client personality write path in the API service**: Only the sim service has write credentials to personality tables, enforcing the single-writer invariant.
- **Log-scrubbing and metric denylist**: These prevent email-as-identifier and per-account/per-bird fields from creeping into logs and metrics.
- **Copy/UI banned-pattern checklist and lint**: The plan uses this to prevent spec-creep such as welcome toasts, visit-frequency surfaces, user-behavior notebook entries, announcement-style UI, and gamification vocabulary.
- **Tick p99 alarm and worker rebalancing**: These mitigate stale aviaries, hot accounts, and tick backlog.
- **Greeting and notebook dedupe/variation**: Continuous timing/pitch/motion spaces, template pools, recent-entry dedupe, and sparsity prevent "three variants in rotation" and visible template repetition.
