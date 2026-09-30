## System-level intent

- The plan carries a relationship-first intent: "one continuing aviary per owner" and a "believable relationship over weeks, not a collection of working controls." This shows up in the product contract, the final completion sentence ("two already-living birds quickly; watching has an honest slow effect"), and the repeated rejection of resets, staged arrivals, counters, and grind.

- Identity continuity is a central principle. The plan says "Identity and personality survive renames, device changes, migrations, and absence," then reinforces that with stable bird UUIDs, immutable species/motif seeds, version-pinned species definitions, retained IDs through recovery, and "once adopted, a bird is never hidden, removed, or reset if a feature flag is rolled back."

- Absence must be harmless and obligation-free. The product contract says absence never creates "hunger, sickness, distress, resentment, or obligation," ordinary tab closure is "a complete and valid goodbye," and later sections forbid absence blame, streak language, absence-duration labels, and negative personality deltas.

- Slow presence, not buying or grinding, is the durable input. The plan says "No action can buy or rapidly grind expressiveness," presence remains "each trait's dominant signal," targeted interaction credit is capped, adoption is by account age rather than engagement, and no engagement/retention target or daily-active-user streak report is a release objective.

- The aviary is canonical server reality with client rendering, not client simulation. The plan's vocabulary is "canonical server simulation," "one committed record," "only the tick role writes personality columns," "Never choose persistent mood, personality, weather, or bird identity," and "no client merge."

- Social access is deliberately narrow and read-only. The plan allows "explicit email visit invitations" but excludes shared aviaries, co-presence, feeds, chat, comments, follows, discovery, and global sharing. Guests "can only observe," and visitor routes cannot reach the command writer.

- Privacy is a design boundary, not just a policy. The plan repeatedly uses "telemetry firewall," encrypted email, synthetic UUIDs, no request-body logs for state routes, no analytics that could later power excluded surfaces, no raw vector exposure, and deletion/key-erasure tests.

- Accessibility is an equal way to experience the same aviary. The plan says "the same quiet aliveness is available through sound, captions, narration, or reduced motion," treats captions/focus as explicit exceptions to the no-inline-copy rule, and says "No screen-reader view is a stats interface."

- Expression must be grounded and procedural. Calls, captions, narration, and notebook entries come from "the exact render projection," "actual expanded call phrase," "admitted user-event facts," and server-side rule engines. The plan rejects canned captions, recorded call audio, and external language-model notebook requests carrying private events.

- The product voice is quiet, specific, and plain. The product contract requires "lowercase, present tense, named birds, specific observations, and restrained bird vocabulary," while settings, failures, unsupported-browser messages, and account operations use "direct ordinary English."

- Naturalism should avoid spectacle. The plan calls for quiet sky/foliage, gentle weather, no dramatic rain or flashes, no entry sequence, no staged whole-aviary arrival performance, no spinner as normal return theater, and no attention-seeking event.

- Performance and operational evidence are part of the product definition. The plan makes first-bird paint, smooth 30-minute scenes, no memory growth, small snapshots, tick latency, and privacy egress scans release gates, and says "a working checkbox alone is insufficient."

- Calibration must use synthetic histories and explicit studies, not production engagement mining. The plan repeats "synthetic accounts," "consented qualitative studies," "Do not A/B users' bird personalities and compare engagement," and "Never compute average real-user drift or mine private event logs to tune the model."

- Scope discipline is itself an intent. The plan's explicit exclusions, section 1.1 binding decisions, rollout ceilings, kill switches that preserve state, and final instruction to "implement only this defined release" all point to avoiding silent feature creep.

## Per-feature whys

### Product contract and scope

- Browser window into one continuing aviary per owner: the plan's rationale is the "believable relationship over weeks" and the idea that a new owner meets birds that are already living, not a resettable collection of controls.

- Two server-selected, individually named starter birds: the plan ties this to a first encounter with "real birds in current poses" and the completion criterion that a new owner meets "two already-living birds quickly." Server selection and persisted preview prevent rerolling from undermining identity.

- Age-based additional adoptions up to seven: the plan says "age, not engagement, remains the only criterion," so additional birds cannot become a grind, scarcity, rarity, countdown, or attention requirement.

- Six species, including a nightjar-like species: the rationale is distinct "silhouettes and procedural call signatures," with the nocturnal profile preserving gentle activity at night.

- Choice of magic-link accounts rather than passwords, SSO, or native identity providers: NOT RECOVERABLE FROM PLAN

- Canonical server simulation: the plan's why is that birds "continue moving through moods without a connected client," all clients share one committed state, and only the tick role can write durable personality.

- Cross-device snapshots: the plan uses them so another device sees the same birds, current mood, light, weather, settled state, and reaction overlays without client-side merge or last-write-wins personality paths.

- Presence accounting: the rationale is "honest accounting and bounded amplification," where one human minute is at most one minute of presence and absence/closure has no negative delta.

- Greeting: the plan frames greeting as a "gentle noticing gesture" on return that varies with canonical attention history while avoiding absence labels, whole-aviary arrival performance, and repeated fingerprints.

- Listen-in: the plan's why is owner-local attention and accessibility; it foregrounds an attended bird without silencing others to zero and contributes drift only when the full presence conjunction is met.

- Exact count of three offers: NOT RECOVERABLE FROM PLAN

- Offer interaction as a feature: the plan makes offers small semantic gestures whose reactions are server-admitted, cooldown-limited, and nonpunitive; they can slightly shape relevant traits but cannot substitute for watching.

- Settle: the rationale is a quiet, explicit goodbye equivalent to tab closure. It ends future presence for that session, previews a calm scene, and never applies a negative personality delta.

- Rename: the plan allows names to change while preserving identity; names are bounded text and "do not change seeds or simulation state."

- Adoption and account settings controls: the plan locates additional naming/adoption controls in account settings rather than scene chrome to preserve the single-scene quiet surface and keep settings/account operations in direct ordinary English.

- Sparse read-only field notebook: the rationale is grounded naturalist memory without a feed. It records specific, novel aviary observations, avoids daily quota/event logs/gamification, and keeps arbitrary history available through cursor pagination.

- Explicit email visit invitations: the plan's why is deliberate, nonpermanent, read-only access by inbox possession, without discovery, global sharing, friends relation, chat, or co-presence.

- Privacy/account lifecycle: the rationale is continuity during recovery and erasure after hard deletion; export, deletion, backups, keys, logs, visits, and identities are all scoped so a deleted account cannot be resurrected.

- Narration: the plan treats narration as the accessible bird greeting and slow naturalist access to the scene, not a welcome announcement or stats interface.

- Captions: captions preserve calls when audio cannot run, describe the actual expanded phrase, and provide an accessibility exception without becoming status labels, badges, or tooltips.

- Reduced motion: the plan's why is equivalent aliveness through still-pose cross-fades and restrained lighting while preserving mood, identity, response timing, call quality, notebook, and presence.

- Explicit exclusions such as native clients, payments, feeds, rankings, streaks, push notifications, and recorded audio: the plan uses these exclusions to protect the release from obligation, gamification, public discovery, social expansion, and analytics surfaces.

- Product prose rules: lowercase, present tense, named birds, specific observations, restrained bird vocabulary, and direct ordinary English keep the product quiet and avoid welcome/return/absence announcement language.

### Decisions where details are open or conflicting

- Sixty-second canonical simulation steps, 15-second owner snapshots, 10-second visitor snapshots, 15-second presence pings, and a 5-minute recent activity window: the plan says these are initial cadences to calibrate with synthetic histories and usability studies while keeping canonical state fresh.

- One stored IANA account timezone: the plan says this "resolves the impossibility of one shared mood being in two local evenings simultaneously" and prevents devices from silently overwriting canonical local time.

- Mood states `wary`, `content`, `curious`, `drowsy`, and `alert`: the plan makes sleeping "a drowsy pose, not a health state," and hides mood labels so birds are not turned into visible status enums.

- Three-minute per-bird offer cooldown across devices and offer kinds: the rationale is server-enforced admission that limits spam/grind while rejected or ignored reactions carry "no punishment."

- New adoption dates at days 90, 180, 270, 365, and 540: the plan uses persistence, no countdown, no unlock announcement, and no scarcity to keep age as the only criterion.

- Small fifth top-bar settle affordance: the plan adds it because interactions require settle while the layout's sparse icon list omitted it; the feature stays in the top bar with "no scene chrome."

- Captions and high-contrast focus as explicit visual exceptions: the plan says they are accessibility exceptions and "not status labels, badges, or tooltips."

- Keyboard listen-in behavior: focus, Enter, arrows, Escape, and Tab are specified so listen-in remains operable without accidental automatic re-engagement after Escape.

- Server-sealed personality capsule in export: the rationale is the "absolute no-numbers rule"; the capsule preserves current vectors without readable numerical fields in JSON, DOM, ARIA, logs, or APIs.

- Visit notification toggle, off by default: the plan treats this as a narrow exception to no-notifications, only after explicit opt-in, and never for birds, absence, weather, adoption, or notebook entries.

- Revocation retaining completed visit-history rows: the plan keeps them "for transparency" and refuses to erase who previously saw the aviary merely to manufacture no active visitor.

- Autoplay-denied sound behavior: captions immediately render the actual call if audio cannot run, because browser restrictions prevent promising audible calls on every first visit and birds must never be blocked behind a sound-enable modal.

### Architecture and ownership boundaries

- Pure versioned simulation kernel shared by workers and synthetic tests: the rationale is deterministic engine behavior and testability while ensuring the client is "never imported as a client state writer."

- Transactional outbox and durable database jobs: the plan says they "avoid infrastructure that adds ordering ambiguity," fitting a v1 without an event broker or analytics warehouse.

- Private authenticated HTML with CDN-cached immutable assets: immutable code/assets can be edge cached for performance, but "Public CDN caches must never contain account snapshots."

- Sharding by synthetic aviary UUID with one primary writer per aviary at greater scale: the plan rejects geographic multi-master personality writes to protect ordering and identity continuity.

- Server responsibilities for authentication, authorization, event admission, tick, projections, notebook, export, and deletion: the rationale is authoritative state, fresh descriptors, and no client-authored mood/personality drift.

- Client responsibilities limited to drawing projections, interpolation, local audio, captions, narration, and ephemeral presentation state: the plan says these are "not an alternative aviary."

- Initial SVG and compact snapshot embedded in authenticated HTML: the why is first bird before full client load and no hydration reset, with actual current pose/phase rather than placeholder or spinner.

- Arrival request returning a reserved greeting descriptor: the plan uses this to provide a noticing action within 1-2 seconds while preventing concurrent owner tabs from provoking synchronized greetings.

- Canonical settle override with immediate origin preview: this makes another device and visitors see the same settled aviary while still letting the origin client preview the lighting shift immediately.

- Visitor requests causing no greeting or host-scene modification: the rationale is that the guest only observes and cannot change host presence, drift, notebook, or scene.

- Synthetic UUIDs and encrypted email that is never a foreign key, queue identity, URL identifier, metric label, or log field: the plan's why is privacy and no PII leakage into telemetry or operational surfaces.

- Exactly one aviary per activated owner account: the rationale comes from "one continuing aviary per owner" and the exclusion of multiple aviaries.

- HttpOnly device sessions with revocation times: the plan keeps session tokens out of JavaScript storage and makes revocation reject later commands and snapshot pulls.

- Versioned immutable species definitions pinned for existing birds: the plan says existing birds remain compatible across releases, preserving species identity and call signature continuity.

- Interaction event IDs, aviary sequence, idempotency, and append-only retention while retained: these support ordered admission, retry safety, and no duplicated drift.

- Presence accumulator as private tick/day accumulators, not a user-facing history or telemetry stream: the rationale is simulation accounting without creating analytics or visit-frequency summaries.

- Invitation records with no permanent friends relation: the plan allows one-use visit access while preventing a quiet social feature from expanding into a friends graph.

- Private expiring export artifacts with no state payload in job logs: the why is requested export without leaking state through logs, queues, or object storage.

- Starter creation transaction that creates exactly two distinct species, birds, vectors, names/default suggestions, and an initial scene, with rollback on failure: the rationale is no partial state and no reset to repair identity.

- Bounded Unicode names that render as text and do not change seeds or simulation state: the plan supports owner naming while protecting identity and preventing name input from becoming state logic.

- Encrypted double-precision personality payload with no unencrypted shadow columns: the rationale is minute-scale increments, finite-range checks, and account-key erasure.

- Storing vectors, calibration/filter memory, and consumption cursors rather than rebuilding from event replay: the plan says service restarts must not repeat deltas or erase the slow current.

- Same-origin JSON endpoints with CSRF, origin checks, bounded input sizes, endpoint rate limits, and typed owner/visitor contexts: the plan's why is secure mutations and ensuring visitor routes cannot reach the command writer.

- Choice of TypeScript specifically for the web client and HTTP service: NOT RECOVERABLE FROM PLAN

### API and command contracts

- Generic response from `POST /auth/magic-links`: the rationale is to prevent account enumeration while still issuing a bounded one-use challenge.

- Confirmation GET plus atomic consume POST for auth links: the plan says this keeps email-link scanners from consuming links.

- `POST /auth/logout`: NOT RECOVERABLE FROM PLAN

- Settings mutations with expected revision and timezone applying on the next canonical tick: the plan uses this to avoid silent conflicts and transition lighting smoothly.

- Device session listing and revocation: revocation rejects subsequent commands and snapshot pulls, so owner control over devices is effective.

- Email change staging and verification before replacing the encrypted verified email: the rationale is preserving the existing address until atomic commit and giving direct recovery guidance on uniqueness collision.

- Starter adoption idempotency and persisted pre-adoption preview: the plan prevents repeated creation or rerolling on reload.

- Snapshot ETags with authorization before `304`: the why is cache efficiency without letting stale authorization return private state.

- Events endpoint returning per-event acceptance, corrected server time, event cursor, and authoritative descriptors while never accepting mood/personality fields: this keeps interactions prompt but server-authored.

- Name edits with expected revision and `409` on stale edit: the plan prevents quietly replacing another edit.

- Adoption endpoint with persistent eligible age-slot presentation and no catalog, rarity, progress bar, or unlock toast: the rationale is non-gamified age-based adoption.

- Notebook endpoint as read-only cursor pages with no edit/delete/annotation operations: the plan keeps it a sparse field notebook rather than a user-curated feed or log editor.

- Invitation creation by owner-specified email and no global sharing switch: the plan keeps access deliberate and narrow.

- Visit list/revocation under lock with active session invalidation and in-place settings update: the rationale is immediate access removal without toasts, badges, or confirmation mail.

- Visit redemption issuing a scoped HttpOnly visit cookie and never an owner session: the plan's why is inbox-possession proof for a read-only visitor, not immutable identity or account creation.

- Visit snapshot/end with no mutation or presence event endpoint: the rationale is a render-only visitor who cannot trigger greetings, drift, notebook, or host scene changes.

- Requested export email with a one-use private download link and no scheduled export emails: the plan keeps export user-requested and prevents state payloads in mail.

- Thirty-day deletion/recovery endpoints: the rationale is direct recovery before the exact deadline and irrecoverable hard deletion afterward.

- All list routes scoped by authenticated synthetic account UUID, never supplied email: the plan prevents email from becoming an internal identity or access key.

- Mute/unmute as device preference, not a listen mechanic or penalty signal: audio muting cannot reduce traits or block accessible drift participation.

### Canonical simulation engine

- Minute scheduling for every non-deleted active aviary with hash-based phase offsets and `FOR UPDATE SKIP LOCKED`: the plan avoids fleet-wide bursts and duplicate worker advancement.

- Tick transaction that reads events, unions presence, updates filters, computes deltas, transitions mood/perch/action/weather/settle, selects notebook candidates, and persists everything together: the rationale is "No partial consumption."

- Deterministic missed-step replay after downtime rather than viewer-triggered evolution: the plan says birds continue without a connected client and correctness comes before compression.

- Presence eligibility requiring visible, focused, recent pointer/key activity, and not settled: the rationale is honest attention accounting without counting open tabs, another app, timers, audio, blur, or offer clicks alone.

- Server bounding of presence intervals against receipt time, last session interval, delivery grace, flags, and activity recency: the plan rejects fabricated long spans and never infers unreported presence from a lease or missing close message.

- Account-level merge of overlapping intervals across tabs/devices: the plan's why is "one human minute is at most one minute of presence."

- Drift function with presence dose dominance and targeted listen/offer caps: the rationale is that clicking alone cannot substitute for watching, targeted credit is small, and no single session creates a visible jump.

- No negative trait deltas from absence or settle, with recent-attention modulation separate from durable traits: the plan ensures birds remain healthy and absence cannot reduce plumage, vocal-frequency traits, or mood into blame.

- Synthetic calibration over 1, 7, 21, 90, and 540 days plus qualitative studies: the plan uses this to tune slow, bounded, visible change without mining private real-user event logs.

- Seeded semi-Markov mood process with local-time baseline: the rationale is gradual, personality- and context-shaped moods rather than forced midnight neutral reset.

- Server-seeded rain and wind with no real weather query or location permission: the plan gets rare, gentle weather without location privacy cost or persistent penalty.

- Three perch zones and collision-free slots: the rationale is keeping birds readable, uncropped, and not stacked indistinguishably across narrow layouts.

- Greeting based on canonical owner attention/session activity, one initiating bird, bounded lease, descriptor variation, and recent fingerprints: this provides a modest natural glance without parallel-tab sync, exact repetition, absence exposure, or whole-aviary performance.

- Offer admission that reserves cooldowns before returning descriptors and allows approach, hesitation, watching, drinking/bathing, joining/calling, or quiet ignoring: the rationale is natural response variety with no failure copy or negative mood.

- Settle preview, five-second undo, and later re-engagement semantics: the plan makes settle a valid goodbye, prevents passive pointer motion from renewing attention, and keeps undo tied to the server-admitted event and deadline.

### Snapshot sync and consistency

- Serialized vector/mood writes, idempotent command retries, no absolute personality API, and no client merge: the plan protects canonical drift from last-write-wins or client-authored state.

- Client snapshot store with canonical/projection versions and server-clock offset: the rationale is discarding reordered lower versions and sampling pose/call programs on the right timeline.

- Interpolating compatible transitions over 300-800 ms and jumping to current phase after long gaps: the plan avoids teleporting and also avoids accelerated replay of unseen history.

- Pulling on visible resume, focus return, render-frame gap, network recovery, and visible keepalive while hidden tabs stop polling/rendering: the plan keeps state fresh without inventing hidden activity.

- Prolonged stale state surfaced as a matter-of-fact system issue: the plan refuses to hide outages behind endlessly invented activity.

- Visitor module using identical projection and ambient synthesis but no owner controls or emitters: the rationale is that visitors see current birds, light, weather, and calls while remaining render-only.

- Revocation rechecks after hidden/suspended time, cancels scene/audio, clears sensitive cached projection, and forbids stale authorized `304`: the why is server-side effective revocation.

### Frontend scene and rendering pipeline

- Layered SVG scene with quiet sky/foliage, semantic perch zones, foreground ornaments, and a small renderer: the plan uses this to draw two-to-seven birds without a large game engine or WebGL dependency.

- Private HTML inline SVG paths, initial pose, essential styles, and compact snapshot: the rationale is first bird before JavaScript, font, settings chunk, or audio initialization, with no hydration reset.

- Normal return showing an in-progress idle/preen/scan pose, no entry sequence, fade-from-static, greeting overlay, or spinner: the plan reinforces that birds are already living.

- Quiet soft sky field only when current snapshot is unavailable, followed by direct error/retry copy after persistent failure: the rationale is matter-of-fact failure handling without unexplained fields.

- One-time initial adoption empty-field/fly-in flag: the plan permits a first-adoption transition but prevents ordinary navigation, stale response, rename, or restore from invoking it.

- Responsive geometry using actual usable width/height, safe areas, no scene scrolling, no panning/zoom controls, and no wide-canvas crop: the plan preserves silhouette readability and seven visible separated birds.

- Collision-aware caption placement and grouping near callers: the rationale is legibility without overlap or scrolling ticker.

- Finite meaningful bird poses with continuously parameterized micro-motion: the plan avoids exposed loops while letting mood-shaped programs drive head, breathing, preen, scan, drowsy, and attentive motion.

- One `requestAnimationFrame` loop with batched transforms, precomputed paths, no hot-path allocations, bounded ornaments, and time-based parallax: the rationale is long-session smoothness and no mouse-tracking spectacle.

- Hidden-page cancellation of rAF, ornaments, sound, and transient audio jobs with fresh sampling on resume: the plan prevents stale animation clocks and resource leaks.

- Top bar fade after stillness with focus/hover/menu/error exceptions and accessible opacity floors: the plan interprets "nearly transparent" as usable, not invisible or hover-only.

- Reduced motion renderer mode before first meaningful frame: the rationale is same-state accessibility through 3-6-second still-pose cross-fades, no moving ornaments/parallax, and no burst of queued motion on runtime changes.

### Procedural audio and caption runtime

- Species call grammar and bird stable seed for sub-signature: the plan's rationale is recognizable identity across moods/drift while preventing variation from making a bird sound like another species or identity.

- Server descriptors plus deterministic client expander consumed by both audio and captions: the plan ensures captions describe the phrase actually synthesized, not a canned species line or guessed mood label.

- Shared temporal recipes for owners and visitors: the rationale is that neither client independently decides that a chorus happened.

- One lazily resumed AudioContext, reusable waves/noise, bounded voices, spatial perch placement, master bus, and transparent limiter: the plan supports procedural sound without downloaded calls, recorded fallback, clicks, or harsh mobile loudness.

- Audio scheduling with lookahead, server clock offset, no stale calls across suspension, and no burst on focus regain: the rationale is temporal consistency with snapshot state.

- Voice limits, staggered envelopes, tonal spacing, and preserved signatures: the plan keeps seven-bird choruses recognizable rather than mechanical or blurred.

- Listen-in gain ramps with a low nonzero ambient floor for other birds: the rationale is foreground attention without treating mute/listen as a command to silence other birds or expose a status mechanic.

- Graceful silence with captions enabled by default for the session when audio fails: the plan says sound loss must not erase calls or block aliveness.

- Audio resource disposal and bounded caches: the rationale is no memory growth over 30-minute sessions and no accumulating audio nodes/contexts.

- Blind listening tests for signatures, mood transitions, choruses, devices, mute/caption mode, and repeated sessions: the plan uses these quality gates before expanding the adoption rollout ceiling.

### Accessibility and notebook surfaces

- Real positioned semantic bird controls paired with SVG, roving `tabindex`, arrow movement, Enter listen-in, Escape exit, and named bird/species action descriptions: the rationale is access to the aviary without a canvas-only app role or raw trait/state labels.

- Top bar keyboard order and `Alt+O` offer shortcut with `aria-keyshortcuts`: the plan avoids unmodified letters that interfere with assistive-technology reading commands.

- Dual-tone focus ring, contrast targets, 200-400% zoom, enlarged text, and 44 CSS-pixel targets: the plan keeps controls, captions, and focus usable across dawn, night, rain, plumage, touch, and zoom.

- Narration generated from exact projection and admitted facts at a slow idle cadence with capped queue and polite live region: the rationale is living naturalist prose without frame spam, machine state lists, numeric traits, or ordinary bird-action interruptions.

- Narration enabled without detecting screen-reader software and with pacing/quieting controls: the plan keeps the feature operable as an accessibility setting rather than a hidden alternate stats view.

- Visual captions near the calling bird from actual expanded phrase contours, syllables, loudness, pauses, and source: the rationale is a concise call alternative, especially when audio cannot work, without duplicating every caption into the live narration queue.

- Notebook server-side rule engine over simulation facts, not an external language-model request carrying private events: the rationale is grounded prose and privacy.

- Notebook sparsity caps of ordinary entries every 3 days, one noteworthy extra per 24 hours, and soft 3 per week: the plan avoids manufacturing a daily quota or feed while letting busy users receive relevant entries.

- Notebook immutable entries with date headings, observed names retained, cursor pagination, and no edit/delete/annotation/hide: the plan keeps it a field notebook, not a generic event log or user-edited content surface.

### Identity, visiting, privacy, and account lifecycle

- High-entropy opaque tokens stored as digests with secure same-site HttpOnly cookies and session fixation protection: the rationale is account and link security without plaintext secrets.

- Pending email verification and atomic replacement of verified email: the plan preserves synthetic account/bird IDs while keeping pending addresses private until verified.

- Mail providers receiving only address and required transactional text/link: the rationale is no vectors, moods, names, notebook, or per-bird interactions in email infrastructure.

- One-use invitation redemption and scoped visitor sessions lasting at most 2 hours: the plan preserves one-time, nonpermanent access while acknowledging token forwarding risk.

- Visit duration approximated from authorized pulls and stored outside simulation events: the rationale is sharing transparency without owner presence or behavioral analytics.

- Default-off visit notifications with at most one plain email per redeemed invitation after opt-in: the plan prevents attention-driving language, badges, default mail, or bird-state emails.

- Telemetry firewall excluding per-bird events from analytics SDKs, crash replay, third-party collectors, training systems, recommendation pipelines, and population drift dashboards: the rationale is that bird interactions belong only to the account's simulation store.

- Raw interaction deletion after transactional consumption and a 7-day recovery window, with compacted presence accumulators and retained vectors/notebook: the plan prevents archival user behavior while preserving the current aviary.

- Operational metrics allowlist with bucketed durations/counts and no IDs, names, vectors, event kinds, precise location, or URL token: the rationale is performance/availability monitoring without account or bird state.

- Export consistent snapshot with sealed personality capsules and no secrets or raw interaction history: the plan provides account export while preserving the no-readable-numbers rule and avoiding leaked session/invite data.

- Soft deletion that revokes access and pauses simulation jobs, then hard deletion/key erasure after 30 days: the rationale is recovery before deadline and unreadable account-specific remnants afterward.

- Restore-time deletion registry/key-erasure checkpoint: the plan prevents backups from resurrecting deleted accounts, sessions, or visit grants.

### Performance, operations, build, and rollout

- Initial JS hard cap below 2 MB with lower internal first-scene targets: the plan says the 500 ms first-bird budget is more demanding than the cap alone, so the team must reserve space instead of filling it.

- First bird metric defined as first painted non-placeholder bird: the rationale is to prevent measuring quiet-field paint as success.

- Sixty fps 30-minute idle scene and no memory growth: the plan protects long-session smoothness and requires bounded audio nodes, captions, ornaments, timers, notebook pages, and listeners.

- Tick p99 latency alarms and small snapshot budgets: the rationale is healthy canonical simulation and polling without oversized scene payloads.

- Aggregate-only RUM and scheduled synthetic browsers: the plan measures navigation, frame, audio, API, freshness, memory, keyboard, and narration flows without account/bird/event/drift dimensions.

- Health checks, worker capacity expansion by UUID partition, and no dropping events or suppressing inactive-account ticks: the plan protects canonical state rather than making charts look good.

- Milestones in dependency order with executable evidence and product review: the plan says "a working checkbox alone is insufficient" and ties build order to cross-disciplinary design/a11y ownership.

- Essential tests across property simulation, transaction integration, browser end-to-end, accessibility, golden grammar/fact, soak/load/privacy/lifecycle: the rationale is proving invariants, product voice, and privacy boundaries, not merely response codes.

- Staged rollout by deterministic synthetic UUID hash: the plan avoids per-account behavioral analytics while checking operational budgets and targeted usability review.

- Feature-gated adoption ceiling: the rationale is to wait for seven-signature audio, mobile layout, accessibility, and worker-load gates before allowing more birds, while never hiding adopted birds after rollback.

- Configuration versions and changelog for presence windows, cooldowns, drift coefficients, mood, weather, notebook, grammar, and renderer mapping: the plan applies changes forward only and avoids recalculating vectors from old history.

- Supported matrix of last two major Chrome/Safari/Firefox/Edge versions: NOT RECOVERABLE FROM PLAN

- Exact internal targets below 200 KB for first-scene JS and below 500 KB including immediately needed grammar: NOT RECOVERABLE FROM PLAN

- Kill switches that stop new invitations/adoptions/offers while preserving existing scene/state: the rationale is severe-incident control without replacing birds or losing earned drift.

- Direct unsupported-browser explanatory copy instead of a heavy compatibility bundle: the plan keeps failure plain and avoids expanding delivery scope.
