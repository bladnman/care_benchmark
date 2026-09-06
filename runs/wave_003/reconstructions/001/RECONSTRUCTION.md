## System-level intent

- **A private living aviary, not an app shell or landing page.** This appears first in the product contract: "Ship a private, browser-based aviary" that "appears to have been continuing before the viewer arrived." It recurs in the first-paint, simulation, and performance sections: ordinary navigation "always begins mid-action," the server keeps advancing "when no clients are connected," and first-bird performance must be tied to "a real visible persisted bird."

- **Continuity is the central promise.** The plan repeatedly protects stable identity and current state: birds' "identities and slowly evolving personalities survive sessions, devices, deployments, and migrations"; migrations must "preserve bird UUIDs and exact current vectors"; rollback "must never restore an old state snapshot over newer drift"; recovery "restores the same UUIDs/vectors/moods."

- **Watching is valid attention, but absence is never punishment.** The contract says "Watching counts as an interaction" and "Absence never subtracts personality, damages birds, or creates an obligation to return." The drift equation, settle/close rules, privacy rules, and tests all repeat this principle: vectors "may only increase," quietness after absence is not a penalty, and no hunger, distress, reminders, or care obligations may appear.

- **The product rejects game and engagement mechanics.** The release scope forbids "scores, achievements, streaks, counters, tiers, competitive rankings, or return reminders" and also forbids hidden engagement rankings. Adoption is "pacing constants," "not reward milestones"; ramp criteria must use operational gates and qualitative review, "never retention, time spent, offer frequency, streak-like metrics, or average trait growth."

- **The server owns truth; clients render and express it.** Architecture gives the simulation worker tick scheduling, personality update, mood/weather/perches, timelines, and notebook evidence, while the scene runtime must not own "tick execution, choice of canonical mood/perch, presence inference from rendering." The plan also says clients do not "optimistically display" offer acceptance, do not merge vectors, and reconnect by installing "the current server state rather than replaying a local simulation."

- **Immediate reactions are allowed only as server-authored presentation.** The plan resolves "sub-second reactions versus a minute tick" by appending "server-authored presentation decisions" promptly while "only the minute tick writes canonical bird personality/mood." This shows up again in command receipts, presentation actions, and the two-axis revision model.

- **Personality change is bounded, positive, slow, and perceptual.** The drift design uses "nonnegative, saturating daily input doses," a "persistent low-pass filter," and no "max-level surface." Presence remains "the dominant signal," "extreme command activity cannot outrun the daily input bounds," and final coefficients are determined by "perceptual targets," not exposed knobs.

- **Individuality should remain recognizable without labels or dashboards.** The product forbids a "user-facing personality dashboard" and numerical trait leaks, yet requires recognizable "individual signatures." This is carried through species silhouettes, seed/current vectors, call identity seeds, call-identification review, and the risk that "saturation erases individuality."

- **The voice is naturalist observation, with system language reserved for account and control surfaces.** The contract says product observations and prompts use "specific, lowercase naturalist prose," while identity, errors, settings, privacy, and account operations use "clear ordinary system language." Accessibility follows the same split: "Accessible observations are part of the product voice, while accessibility controls are system voice."

- **Quietness is intentional.** The plan repeatedly avoids intrusive surfaces: no welcome message, toast, banner, countdown, badge, notification loop, milestone announcement, or reward vocabulary. The top bar fades; blocked actions use a "quiet system status"; visit email is narrow, off by default, and "quiet."

- **Accessibility is the same product, not an accommodation layer.** The accessibility section is titled "Accessibility as the same product" and requires implementation "during the first two-bird scene, not as a post-launch audit." Reduced motion must still be "a living series of still poses," and screen-reader narration must be "paced connected observations, not an event list."

- **Privacy boundaries are product boundaries.** Operational observability "cannot read private simulation inputs." Private simulation input is used "only to advance its owner's aviary"; no analytics warehouse, behavioral SDK, training, recommendations, session replay, population bird analysis, or hidden engagement ranking is allowed. The visit log is "transparency about access, not an engagement surface."

- **Sharing is render-only and revocable.** Invitations are "individually issued, emailed, expiring, revocable" and grant only a "render-only visit." Visitors have no greetings, listen-in, offers, settle, adoption, notebook, or host event endpoints, and visitor activity never cancels settle or contributes presence.

- **Evidence gates matter more than claims.** The plan insists that numerical constants are "initial engineering decisions with explicit validation gates." Release requires synthetic calibration, perceptual review, physical-device checks, assistive-technology review, first-frame filmstrips, memory soaks, restore drills, and operational alarms. "A backend unit test cannot stand in for audio taste, accessibility experience, or first-frame performance."

- **Operational performance is measured on the actual aviary.** Performance is "a release property of the actual animated aviary, not a landing-page score." The first-bird mark must be tied to a persisted bird, not a placeholder or quiet field, and a warm desktop measurement is not evidence for mobile.

## Per-feature whys

### 1. Product contract and release scope

- **Private, browser-based aviary**: The plan's rationale is that the experience should feel like an aviary already in motion: it "appears to have been continuing before the viewer arrived."

- **Stable birds across sessions, devices, deployments, and migrations**: The rationale is relationship continuity; "identities and slowly evolving personalities survive" all of those boundaries.

- **Watching as an interaction**: The rationale is that quiet watching is legitimate attention. The plan later sets a five-minute activity window "because quiet watching is expected."

- **Absence never subtracts personality, damages birds, or creates an obligation**: The rationale is to avoid penalty, guilt, care debt, distress, and return pressure. The plan repeats this through "no hunger/death/distress," no reminders, and vectors that cannot fall.

- **One canonical aviary per account**: The rationale is one shared relationship state. The plan enforces one active account aviary and rejects multiple aviaries, shared ownership, and local state forks.

- **Email magic-link accounts as the login method**: NOT RECOVERABLE FROM PLAN

- **Device-session management**: The rationale is cross-device continuity plus revocability. Sessions are listed as coarse devices, can be revoked immediately, and must not use detailed fingerprinting.

- **Verified email changes**: The rationale is account continuity and safe identity replacement: the new address is verified before replacing the old encrypted email/index, while account UUIDs, aviaries, and birds remain unchanged.

- **Account export**: The rationale is private portability. The plan treats the export as the sole exception for current personality vectors and says the exception belongs in an isolated serializer, not normal UI, analytics, diagnostics, or snapshots.

- **Recoverable deletion**: The rationale is a reversible account lifecycle before a hard deadline. Deletion marks the account with an exact 30-day recovery deadline, suspends simulation writes, and restores the same UUIDs/vectors/moods if recovered.

- **Approximately six coherent species as the exact count**: NOT RECOVERABLE FROM PLAN

- **Coherent species, including a nightjar-like species**: The rationale is legible variety and day/night behavior. Species carry silhouettes and call families; the nightjar-like species "may remain active" at night.

- **Two system-chosen starters with naming and renaming**: The rationale is a stable initial relationship, not a chooser/catalog. The system chooses distinct starter species, persists seeds before showing name suggestions, and prevents refresh or abandoned naming from rerolling birds.

- **Optional age-based adoption up to seven birds**: The rationale is pacing, not reward. Eligibility "does not expire, depend on activity, or require accepting previous offers on time," and the plan says these ages are "not reward milestones."

- **Server simulation during the day and during absence**: The rationale is that the aviary is alive when nobody visits. The worker schedules every active aviary, advances state when no clients are connected, and catches up outages from persisted state.

- **Persistent personality, mood, perch choice, weather, and bird-to-bird behavior**: The rationale is continuous identity and shared truth. These states live in the canonical server state rather than being invented by rendering.

- **Procedural greetings**: The rationale is that a bird can notice arrival without turning it into a welcome banner or counter. Greetings vary by absence bucket, mood, warmth, boldness, and recent greeter history.

- **Listen-in**: The rationale is focused local attention on a bird's sound while preserving the shared simulation. It changes playback gain and bounded per-bird attention dose, but does not force canonical calls or affect another device's mix.

- **Three kinds of offer**: The rationale is gentle interaction without needing to click a bird or create success/failure. Responses are observable behavior choices such as approach, watch, ignore, drink, bathe, or call.

- **Settle and undo**: The rationale is a quiet presentation mode that stops current presence/listen intervals without penalty. Undo protects accidental activation for five seconds, and re-engagement resumes the shared view.

- **Sparse, indefinitely browsable field notebook**: The rationale is to record noteworthy aviary observations, not produce a feed or stats log. Entries require evidence, use cooldown/novelty rules, remain immutable, and can be browsed backward indefinitely.

- **Single responsive horizontal scene with three depth zones**: The rationale is one fitted aviary, not navigation or placement gameplay. Front/middle/back proximity remains legible through scale, contrast, and placement across viewports.

- **Local-time lighting**: The rationale is shared day/night continuity from the account's canonical timezone. Later devices do not silently rewrite it, preserving one shared day/night state.

- **Restrained procedural weather and ambient ornaments**: The rationale is ambient life independent of owner activity. Weather is not a location API, notification, or severe alert; ornaments remain outside canonical state and strictly capped.

- **Fading top bar**: The rationale is to keep controls available while reducing visual chrome. It restores on pointer, keyboard, proximity, focus, or touch, and never fades open menus or focused controls.

- **Procedural WebAudio calls with individual signatures**: The rationale is recognizable living birds without recorded loops. Species supply motif families, identity seeds supply contour/register/timbre, and variation stays within signature bounds.

- **Captions, naturalist screen-reader narration, and reduced motion**: The rationale is that accessible observations are part of the product voice. Captions derive from the realized call descriptor, narration uses current pose/call/light/weather facts, and reduced motion remains alive through still-pose cross-fades.

- **Individually issued, emailed, expiring, revocable render-only invitations**: The rationale is deliberate private sharing without co-presence or visitor influence. Visitors receive the same scene projection but cannot write presence, greetings, notebook, offers, settle, or adoption.

- **On-demand visit log**: The rationale is access transparency, not engagement. It shows email/date/approximate duration and outstanding invitations without badges, unsolicited display, or bird-interaction fields.

- **Operational observability, synthetic calibration, and launch gates**: The rationale is to operate and validate the system without reading private relationship data. Metrics are allowlisted, synthetic fixtures calibrate behavior, and release is gated by performance/accessibility/privacy evidence.

- **V1 exclusions: native apps, passwords/SSO, payments, multiple aviaries, social network features, game/care mechanics, and user-facing dashboards**: The rationale is scope protection for a quiet private aviary. The plan repeatedly says not to build obligations, public discovery, engagement mechanics, hidden rankings, or future-feature data capture.

- **Two registers for copy**: The rationale is voice separation. Product observations/prompts use "specific, lowercase naturalist prose," while identity, errors, settings, privacy, and account operations use "clear ordinary system language."

### 2. Explicit decisions where the PRDs leave gaps or conflict

- **Machine-readable export as the sole vector exposure exception**: The rationale is that `accounts_sync.md` and `bird_engine.md` conflict literally. The plan preserves portability while isolating the contradiction from normal snapshots, UI, ARIA, captions, diagnostics, and analytics.

- **Off-by-default opt-in visit email**: The rationale is to honor a narrow social exception without creating reminders or campaign infrastructure. It permits one quiet visit email while forbidding push, toasts, badges, onboarding prompts, reminders, and auto-invites.

- **Four top-bar icons with settle inside the offer menu**: The rationale is to retain exactly account/settings, accessibility, notebook, and offer icons while still making settle reachable from the top bar.

- **Keyboard focus, Enter, Escape, and pointer listen-in semantics**: The rationale is to avoid accidental toggling while preserving keyboard access. Focus starts listen-in, Enter ensures it, Escape disengages without losing focus, and pointer activation on the listened-to bird toggles off.

- **One canonical account timezone**: The rationale is to preserve one shared day/night state. First browser suggests it, all devices and visitors see it, and travel changes require explicit settings update.

- **Account-wide settle presentation override**: The rationale is consistent multi-device presentation. It closes owner presence/listen windows, applies a small canonical mood influence at the next tick, and can be canceled by explicit owner re-engagement, never visitor activity.

- **Immediate command-service presentation receipts with minute-tick canonical updates**: The rationale is sub-second visual response without client prediction or out-of-band personality mutation. Commands append accepted presentation decisions immediately; only the tick writes canonical personality/mood.

- **Synthetic account-identity UUID for invitation recipients**: The rationale is to store the recipient once without silently signing an unregistered recipient up for an aviary.

- **Additional-bird eligibility ages**: The rationale is pacing. Eligibility is based on account age, persists indefinitely, and is explicitly described as constants to validate, "not reward milestones."

### 3. Architecture and ownership boundaries

- **TypeScript as the web application language**: NOT RECOVERABLE FROM PLAN

- **Server-rendered HTML/SVG plus a small imperative scene runtime**: The rationale is first-bird continuity and lightweight rendering. The initial snapshot is embedded in HTML so "no client waterfall blocks the first bird," and the runtime samples server timelines without replacing the scene.

- **Code-split component islands for forms, settings, notebook, and invitation administration**: The rationale is to keep panel work from restarting bird action or audio and to defer noncritical code from the first-bird path.

- **Relational PostgreSQL primary as sole transactional source of truth**: The rationale is consistent locking, explicit foreign keys, revisions, and exactly-once state transitions for personality, events, adoption, and deletion.

- **Modular application service plus separately scalable simulation worker, not microservices**: The rationale is clear ownership and scalable ticking without fragmented canonical writers.

- **Durable job mechanism and transactional outbox**: The rationale is reliable ticks, mail, exports, and deletion with retries and lifecycle fences.

- **Static CDN assets and encrypted private export objects**: The rationale is to separate public immutable performance assets from private transient account exports.

- **Revision-fenced reads and private no-store responses**: The rationale is to prevent stale state after accepted commands and avoid unauthorized or shared caching of personalized HTML, snapshots, or visits.

- **Operations collector without simulation database access**: The rationale is observability without private inputs, identifiers, or aggregate metrics that can inspect relationships.

### 4. Durable data model

- **UUID primary keys, UTC instants, versioned schemas, foreign keys, and per-root authorization**: The rationale is stable identity, migration safety, and explicit authorization across accounts, aviaries, birds, visits, jobs, and exports.

- **Timezone as presentation/circadian logic rather than persisted event-time change**: The rationale is that event time remains UTC truth while local time affects only light, circadian behavior, and prose.

- **Simulation database role as the only updater of personality columns**: The rationale is to keep personality changes canonical and prevent clients or unrelated services from writing drift.

- **Integer millionths plus fractional remainders for vector arithmetic**: The rationale is that repeated minute-sized increments are not lost to rounding.

- **Encrypted email plus restricted blind lookup index on accounts**: The rationale is identity lookup without using email as a relational key, sharding key, log field, trace label, queue identifier, or analytics dimension.

- **Account settings separated from local device audio capability**: The rationale is to keep durable preferences distinct from per-device capability and fallback state.

- **Single-use expiring auth tokens with encrypted temporary new-email payloads**: The rationale is scanner-safe, replay-resistant authentication and a short-lived exception for email-change verification.

- **Device sessions with token digests and coarse device labels only**: The rationale is revocation without raw tokens or detailed fingerprinting.

- **Aviary records with revisions, tick cursor, weather/circadian state, RNG state, and simulation version**: The rationale is deterministic continuation, exactly-once ticks, missed-tick catch-up, and server-authored timelines.

- **Bird records with stable UUIDs, seed vector, current vector, filters, mood, perches, behavior seed, and immutable call identity seed**: The rationale is durable identity and continuity. The seed vector is "historical identity data, never a reset source."

- **Append-only interaction events with minimal validated payloads**: The rationale is ordered, idempotent command processing without raw pointer coordinates, key contents, or email.

- **Presence intervals as private simulation input**: The rationale is precise unioned presence accounting while keeping attention evidence private, bounded, and expirable.

- **Command receipts and presentation actions**: The rationale is durable retry behavior and shared authoritative greetings/offers/settle/resume cues without client invention.

- **Interaction guards under the aviary lock**: The rationale is race-free offer reservations, arrival coalescing, and settle epochs across devices.

- **Bird observation summaries and immutable notebook entries**: The rationale is evidence-backed naturalist prose without retaining raw logs indefinitely or rewriting old observations after rename/template changes.

- **Persisted adoption offers**: The rationale is that refresh "never rerolls" the predetermined species and seed.

- **Separate visit invitations, visit sessions, and visit log records**: The rationale is one-time render-only grants, revocation, and access transparency without duplicating email or adding bird-interaction fields.

- **Jobs/outbox and export_jobs with minimal private references**: The rationale is retryable lifecycle work without putting email or full simulation snapshots in queue payloads.

- **Name rules: 1-40 grapheme clusters, normalized/escaped, not keys, no uniqueness constraint**: The rationale is safe display and identity preservation. Renaming must not change species, call seed, personality, mood, or old notebook prose.

- **Version migrations preserving bird UUIDs and exact current vectors**: The rationale is that deployments must not recreate birds from history or a new seed algorithm, and fixtures must catch renamed birds, drift remainders, active mood, and old grammar versions.

### 5. API contracts and authorization

- **Same-origin HTTPS JSON APIs with Secure, HttpOnly cookies, CSRF, origin validation, and strict schemas**: The rationale is route-level authorization and prevention of caller-selected raw personality/mood/perch state.

- **Matter-of-fact error codes and safe messages**: The rationale is to avoid reflecting tokens or identities while using clear ordinary system language for system surfaces.

- **Magic-link request and consume behavior**: The rationale is account-enumeration resistance and email-scanner safety. Requests always return generic accepted responses, new requests do not extend token expiries, and landing GET does not consume the link.

- **Account/session/settings/email-change endpoints**: The rationale is owner-only account control, device revocation, conditional preference writes, timezone validation, and new-address verification before replacement.

- **Export endpoints**: The rationale is explicit owner-requested private JSON delivery, idempotent jobs, consistent snapshots, token-bound downloads, and no token-bearing referrers.

- **Delete and recover endpoints**: The rationale is exact 30-day deletion with lock-protected recovery after fresh identity verification.

- **Aviary snapshot endpoint**: The rationale is monotonic canonical and presentation state, authorization even on 304, no vectors or private inputs, and compact seven-bird delivery under 20 KB.

- **Aviary event and receipt endpoints**: The rationale is ordered, server-timed, idempotent command acceptance. Same event ID and payload returns the same result; different payload returns 409.

- **Notebook, adoption, and rename endpoints**: The rationale is stable keyset notebook browsing with no edit/delete/annotate routes; quiet age-based adoption without catalog/countdown/badge; and renaming without patching immutable simulated fields.

- **Visit endpoints**: The rationale is deliberate invitations, one-time token consume, live revocation checks on every pull, render-only projection, and an on-demand visit log without notebook or account data.

### 6. Simulation engine and continuity

- **60-second tick cadence as the exact cadence**: NOT RECOVERABLE FROM PLAN

- **Scheduling every active aviary, not only connected ones**: The rationale is continuity during absence and a server-side living aviary.

- **Leased workers, row locks, logical tick keys, and atomic transaction steps**: The rationale is exactly-once effects. A crash before commit rolls back all effects; a crash after commit cannot reapply drift.

- **Fixed event lock order and server-assigned sequences**: The rationale is no cooldown decision or event racing into the middle of a tick's committed state.

- **Missed tick catch-up from persisted checkpoints**: The rationale is recovery without lifetime event replay or personality reset. Large gaps are degradation, not the normal absence mechanism.

- **Deployment compatibility, canaries, rollback, and migration backups**: The rationale is preserving grammar/signature interpretation and newer drift while avoiding database rollback that can erase the relationship.

- **Trait initialization with species baselines and bounded per-bird offsets**: The rationale is legibly different starters without labeling either better.

- **Nonnegative, saturating daily input doses**: The rationale is bounded positive drift from presence, listen-in, and offers rather than click counts or penalties.

- **Presence-dominant trait routing**: The rationale is that watching remains the main relationship signal and command activity cannot replace attention. Settle and mute have zero negative personality effect.

- **Low-pass drift equation with filter tail**: The rationale is to integrate minutes after the user leaves without making absence-based growth unbounded or allowing vectors to fall.

- **Separate recent-attention envelope**: The rationale is quiet ambient behavior after absence without reducing warmth, boldness, saturation, or vocal frequency, and without wary/sad/hungry/distressed states caused by leaving.

- **Five v1 moods: wary, content, curious, drowsy, alert**: The rationale is observable motion/perch/call variation with dwell periods that avoid flicker and ordinary temporary wary responses that are not absence penalties.

- **Daily-ish randomized mood refresh**: The rationale is gradual release of yesterday's session influences without resetting to default at midnight or tab open.

- **Probabilistic server-assigned perch zones and slots**: The rationale is emotional proximity decided by mood/boldness/space, not client layout hacks.

- **Server timelines for bird-to-bird calls and response caps**: The rationale is staggered social behavior without all-wary feedback loops or client/browser audio playback driving canonical transitions.

- **Procedural weather rather than location/weather API**: The rationale is restrained ambiance without location permission, severe alerts, actionable notifications, or owner-activity dependence.

- **Weather frequency of two or three rains per week on average**: NOT RECOVERABLE FROM PLAN

- **Continuous lighting from stored timezone using clock-time curves**: The rationale is local-time ambience without geographic sunrise or location permission, with timezone edits smoothed rather than jumping.

- **Rolling 120-second deterministic behavior timeline**: The rationale is server-authored action/call/perch continuity while allowing leaf/feather particles and pose sampling to remain rendering, not simulation.

### 7. Presence, interactions, and session state machines

- **Owner view states and exact `present` conjunction**: The rationale is precise attention accounting: visible, focused, and trusted pointermove or keydown within five minutes. Open tabs, timers, focus alone, and media playback do not count.

- **Five-minute recent activity window**: The rationale is explicit: "quiet watching is expected."

- **Presence pings, terminal flush, segment caps, and no replayable offline queue**: The rationale is measured past spans with bounded failure and no open-ended future credit after exit or offline sessions.

- **Deduplicated union presence across devices**: The rationale is that two visible devices must not produce two seconds of presence for one wall-clock second.

- **Listen-time association only with qualified presence**: The rationale is to prevent disconnected or stale focus flags from crediting attention.

- **Visitor endpoints lacking presence permission**: The rationale is that visitors must not affect host simulation.

- **Arrival events after scene visibility**: The rationale is that a bird can notice arrival before sustained attention, while absence duration is estimated server-side and never exposed.

- **Weighted primary greeter and resampled greetings**: The rationale is individual variety: warmth, boldness, mood, and recent greeter history shape the cue, exact repeats are resampled, and one return cannot make every bird greet.

- **Greeting as the only welcome surface**: The rationale is product voice without toast, banner, text addressed to the user, or absence counter.

- **Listen-in local audio attention state**: The rationale is local focus with smooth gain transfer, bounded server leases, and no forced canonical calls.

- **Caption and muted-user listen-in credit**: The rationale is accessibility parity. Focused attention can count even when audio is muted or unavailable, and failures cannot penalize personality.

- **Offer menu from the top bar**: The rationale is deliberate gestures without clicking a bird, with the server choosing a naturally nearby eligible receiver.

- **Shared per-bird offer cooldown**: The rationale is to prevent offer spam across all types/devices while avoiding countdowns or penalty language.

- **Offer outcomes as observable behavior, not success/failure**: The rationale is emotional neutrality; narration can describe a bird watching without treating non-acceptance as user failure.

- **Settle display heartbeat, undo, resume, and close expiry**: The rationale is reversible quiet presentation that cannot generate presence, cannot be held forever by network loss, and cannot be canceled by visitor activity.

- **Visible-owner and visitor pull schedules**: The rationale is client synchronization and prompt revocation. Owner polling is not simulation ticking; visitor two-second pulls make revocation prompt.

- **Two-axis revision tracking and tombstones**: The rationale is deterministic merge of canonical state and presentation receipts without reviving expired actions or erasing newer cues.

- **Server-clock offset and sleep handling**: The rationale is aligned audio/render cue timing without replaying missed calls or greetings in a burst after laptop sleep.

- **Motion reconciliation from current pose/velocity**: The rationale is continuity without fading the whole aviary or fabricating a mood to hide stale data.

- **Network degraded mode**: The rationale is honest last-valid rendering, no unsent attention credit, no invented social events/notebook observations, and quiet system errors instead of repetitive toasts.

- **Pending command receipt retry**: The rationale is idempotency after uncertain network responses; clients query the original event ID instead of generating a new one.

- **Settings/name conflict handling**: The rationale is a scoped reload/reapply flow for the user's edit, not a personality-history sync conflict.

### 8. Pull synchronization and degraded operation

- **Out-of-order response handling**: The rationale is that older HTTP responses must not overwrite newer state or action receipts.

- **Conditional responses with live authorization**: The rationale is that revoked sessions and visitors cannot continue through cached authorization success, even on 304.

- **Expired sessions stopping sends and showing sign-in**: The rationale is clear system-language recovery rather than hidden failed commands.

### 9. Frontend scene and loading pipeline

- **Retained SVG with compact species silhouettes**: The rationale is that seven birds do not justify a large 3D or game-engine dependency.

- **One rendering tree from first paint onward**: The rationale is continuity. The server renders first valid bird poses, bootstrap sets elapsed phase, and the runtime takes over the same SVG nodes without replacing or fading the scene.

- **Mid-action ordinary navigation**: The rationale is that the aviary was already living; the plan says never show a static preview and then animate an entry sequence.

- **Quiet sky/foliage fallback**: The rationale is an honest loading contingency when the initial snapshot is delayed, not a substitute first bird or empty-new-account state.

- **New-account empty-to-bird arrival sequence**: The rationale is that new activation is the sole intentional empty-to-bird moment after adoption is committed.

- **Responsive depth composition and no scene navigation tools**: The rationale is a single fitted scene with legible front/middle/back proximity, not panning, zooming, placement, labels, or inline buttons.

- **Safe slot mapping and path validation**: The rationale is to keep bird bodies, wings, captions, and focus rings inside the safe rectangle across aspect ratios without changing canonical emotional proximity.

- **Acceptance viewport and zoom constraints**: The rationale is that the aviary remains a single fitted scene while panels may scroll, hit targets remain usable, and keyboard access survives extreme zoom.

- **Muted blue, green, brown, ochre, warm light palette, and contrast samples**: The rationale is to satisfy visual design in the absence of the separate visual design document and verify every lighting extreme for readable text and controls.

- **Idle pose generators and ornament caps**: The rationale is quiet living motion without obvious loops, twitching tick boundaries, memory/performance growth, or mouse-follow/scroll interaction.

- **Top bar fade behavior**: The rationale is discoverable controls with reduced decorative weight, while preserving accessible contrast and never hiding active/focused surfaces.

- **Panels outside the scene proper**: The rationale is that account/accessibility/notebook/offer panels must not reset bird action or pan the aviary.

### 10. Procedural audio and captions

- **Species motif libraries as compact parameter grammars**: The rationale is procedural variation by species and individual without recorded bird calls or loops.

- **Immutable call-identity seed and signature parameters**: The rationale is recognizable contour/register/timbre across mood and drift changes.

- **Shared realized call descriptor for synthesis, beak timing, and caption text**: The rationale is consistency: captions describe the sound actually emitted, and client expansion cannot decide whether canonical calls happen.

- **Symbolic offered melodic fragments synthesized through the same runtime**: The rationale is no downloaded audio recordings or phrase fallbacks masquerading as procedural variation.

- **One AudioContext with fixed voice pool or bounded fallback graphs**: The rationale is performance and memory stability: no new context on each listen-in, return, or settings change, and no allocations in real-time processing where possible.

- **Per-bird buses, chorus mix, stereo placement, gain ramps, limiter, and headroom**: The rationale is focused listen-in while keeping other birds audible, avoiding synchronized mechanical choruses, shrill transients, pumping, clicks, underruns, or unbounded voice growth.

- **Listen-in as playback gain only**: The rationale is that local focus cannot alter another user's mix or rewrite a bird's call identity.

- **Autoplay and audio failure behavior**: The rationale is honest browser permission handling: no blocking splash, fake sound, recorded fallback, or overridden mute; captions and silent scene remain available.

- **Caption generation from realized call features**: The rationale is descriptive accuracy. The caption includes note count, contour, pauses, texture, intensity, and perch, and updates if the voice limit drops notes.

- **Caption placement near the calling bird**: The rationale is readable association without suppressing important captions for a clean screenshot, including deterministic slots/lanes for seven birds.

### 11. Accessibility as the same product

- **Real DOM controls and semantic bird navigation aligned with SVG positions**: The rationale is accessible operation without double-announcing every SVG part or leaking trait values into labels/debug attributes.

- **Screen-reader initial paragraph and routine narration**: The rationale is naturalist scene awareness from current pose, call, light, and weather facts, not a system announcement or raw event log.

- **Bounded narration queue and dialog suppression**: The rationale is paced observations that prioritize user gestures without flooding, interrupting control labels, or replaying missed chatter.

- **Keyboard contract for top bar, roving bird focus, arrow movement, listen-in, menus, and settle**: The rationale is full primary flow access without custom shortcut conflicts or accidental focus identity changes as birds move.

- **Two-tone focus outlines**: The rationale is visible focus across every lighting/weather state.

- **Reduced-motion modes with still-pose cross-fades**: The rationale is to reduce continuous motion without freezing the scene or removing greeting/offer reactions, moods, calls, captions, drift, or notebook behavior.

- **Visitor-mode accessibility**: The rationale is the same naturalist narration, captions, sound controls, and motion preferences while preventing accessible controls from sending owner interaction events.

- **Assistive-technology acceptance coverage**: The rationale is that automated ARIA checks cannot establish whether the aviary feels alive.

### 12. Field notebook and adoption pacing

- **Local deterministic observation composer with authored naturalist grammar**: The rationale is no external language-model call receiving private events.

- **Evidence-backed candidate facts**: The rationale is truthful qualifiers such as "first time this week" from compact summaries, not assertions derived from whatever logs remain.

- **Candidate scoring only inside that aviary with cooldown and novelty rules**: The rationale is sparse specificity without ranking accounts or generating a heavily active owner's feed.

- **Immutable final notebook prose with template version and evidence**: The rationale is stable old entries after template improvement or rename.

- **No logs of sessions, deltas, totals, visited days, or user habits**: The rationale is observations of the aviary, including quietness, without guilt, neglect, stats, or habit surveillance.

- **Cursor pagination and bounded rendered window**: The rationale is indefinite browsing without DOM/memory growth, archiving, hiding, or truncating old entries.

- **Initial account activation creates exactly two stable bird records atomically**: The rationale is refresh-safe identity with persisted seeds before naming.

- **Later adoption in account/bird settings only**: The rationale is quiet, deliberate adoption without onboarding invitations, scarcity, countdowns, celebratory unlocks, achievements, or adopted-count display.

### 13. Privacy, account lifecycle, and visit details

- **Account UUID independent of email**: The rationale is stable identity and privacy; email is encrypted and indexed only for lookup, never for sharding, logging, tracing, queueing, or analytics.

- **Encrypted transport/storage, scoped roles, CSP, encoded snapshot JSON, token redaction, digest tokens, same-site cookies, and rate limits**: The rationale is preventing name/script injection, token leakage, account enumeration, and unauthorized data access.

- **Email provider receives only transactional recipient/link/message**: The rationale is that bird events, vectors, notebook content, and presence remain private.

- **Session timeouts and revocation**: The rationale is secure renewal and user control without raw tokens.

- **One-time invite bearer capability**: The rationale is honest security: forwarding can transfer capability, so it must be one-time, short-lived after use, and revocable rather than pretending strong recipient identity.

- **Retention periods for events, presence, notebook evidence, receipts, visit logs, and operational errors**: The rationale is bounded recovery/debugging and privacy publication, while not deleting unconsumed events before the worker covers them.

- **Metrics collector separation and no behavioral SDK/warehouse/ML**: The rationale is that private relationship data cannot flow into training, recommendations, population bird analysis, heatmaps, or session replay.

- **Aggregate operations data without stable IDs**: The rationale is monitoring that cannot later be mapped back to an account, bird, session, invitation, or stable device.

- **Export consistent snapshot**: The rationale is private account portability with the explicit vector exception, not a UI stats panel or export of presence histograms, visit streaks, raw owner history, other accounts, or secrets.

- **Deletion, final erasure, key destruction, and deletion journal replay**: The rationale is to honor the 30-day hard-deletion promise even across jobs, backups, restores, ticks, exports, mail, and recovery races.

- **Pending recipient identity garbage collection**: The rationale is to remove unactivated identity rows once no valid grants or retained visit records reference them.

- **Visit revocation, approximate durations, and visit email toggle**: The rationale is transparency and host control without visitor attention tracking, revocation toasts, implicit notification opt-in, or further snapshot disclosure after revocation.

### 14. Performance budgets and operational observability

- **Reference mobile/laptop devices and reproducible network profiles**: The rationale is that performance evidence must represent the actual release target, not a warm desktop measurement.

- **Initial JS cap and tiny scene bootstrap**: The rationale is first-paint performance with panel code and noncritical work deferred.

- **First bird under 500 ms tied to a visible persisted bird**: The rationale is that the budget applies to the actual animated aviary, not a placeholder element or quiet field.

- **First response path allocation**: The rationale is traceable latency responsibility across TTFB, compressed scene payload, parse/bootstrap, and style/paint, with architecture revision if the sum misses 500 ms.

- **Seven-bird snapshot budget**: The rationale is compact projection with no unbounded history or sampled animation frames.

- **Greeting, idle frame, audio, memory, tick, and revocation budgets**: The rationale is release-quality behavior under return latency, old-laptop frame time, audio stability, long-session heap stability, service tick latency, and live authorization.

- **Deferring noncritical assets and avoiding large media/frameworks**: The rationale is protecting the first-bird path and never substituting a random bird or loading field for success.

- **Memory soak scenario**: The rationale is detecting retained growth across calls, listen transfers, offers, settle/undo, snapshots, notebook pagination, resize, and visibility transitions.

- **Synthetic browser probes and moving supported-browser matrix**: The rationale is ongoing measurement across geographies and current browser versions without permanent compatibility bundles for old unsupported browsers.

- **Day-one aggregate metrics allowlist and label-cardinality bounds**: The rationale is useful operations data without payloads, names, vectors, offered items, presence totals, per-account sequences, stable visitor identity, or cross-linkable trace IDs.

- **Operational alerts and runbook**: The rationale is distinguishing tick duration from queue lateness and supporting pause admission, drain queues, compatible rollback, and authorization restore without rebuilding personality from logs.

### 15. Verification plan and acceptance evidence

- **Simulation and transaction property suites**: The rationale is to prove no negative drift, no double-counted devices, exact presence conjunction, settle/close equivalence, idempotent ticks, serialized commands, catch-up equivalence, migration preservation, and bounded social/weather influence.

- **Property-based generation and golden fixtures**: The rationale is covering nonnegative input sequences, event retries, random gaps, concurrent writers, engine versions, and drift algebra beyond a single example.

- **Owner and visitor end-to-end scenarios**: The rationale is validating real primary flows with pointer, touch, and keyboard, including magic links, adoption, returns, listen-in, offers, settle, notebook, invitations, visit email, rename, export, deletion, and recovery.

- **Server-scope auth tests**: The rationale is that hiding buttons is not enough; cross-account substitution, visitor owner-route attempts, revoked-session 304s, CSRF, token logs, hostile names, invite enumeration, and email-change races must fail on the server.

- **Perceptual and accessible review**: The rationale is to judge initial motion, greeting specificity, naturalist language, emotional neutrality after absence, quiet continuity, screen-reader pacing, caption legibility, and reduced-motion liveliness with humans.

- **Drift calibration and 21-day pilot**: The rationale is to check felt timescale with week-one numerical evidence and week-three observable differences without exposing numbers or using exported private histories/population statistics.

- **Call-identification review**: The rationale is ensuring individual calls remain recognizable at two, three, five, and seven birds without adding visible identification stats.

- **Traceability and evidence artifacts**: The rationale is that each feature needs user-visible demonstration plus invariants; implementation evidence must not contain private production bird data.

### 16. Delivery sequence and rollout

- **10-12 engineering week delivery sequence as an exact estimate**: NOT RECOVERABLE FROM PLAN

- **Milestones M0 through M5**: The rationale is dependency ordering from contracts/fixtures, to persistent living slice, relationship loop, private account, perceptual/scale hardening, and staged release.

- **API schemas and fixtures as interfaces between work areas**: The rationale is parallel work: frontend renders compiled fixtures while simulation is built, and audio/captioning share realized descriptors.

- **Identity/privacy work before real accounts**: The rationale is that private account correctness cannot be deferred until after users exist.

- **Persisted drift and deletion correctness blocking release**: The rationale is that relationship continuity and lifecycle promises are launch-critical.

- **Bird-count ramp from two to three to five to seven**: The rationale is quality validation for density, frame time, narration load, caption placement, and recognition, not an attention reward.

- **Public accounts start with two and follow true creation-age eligibility**: The rationale is avoiding beta acceleration, feedback rewards, or age manipulation.

- **Public traffic ramp with healthy-operation holds**: The rationale is admission bounded by measured unattended tick capacity and operational/security/performance gates, not engagement metrics.

- **Kill switches for new invitations, visit emails, and new adoption acceptance**: The rationale is fault containment without turning off the base scene, accessibility surfaces, or existing bird records.

- **Failure drills before broad availability**: The rationale is proving recovery for tick-worker outage, database failover, delayed queues, mail failure, export expiry, visitor-token revocation, and hard-deletion backup restore.

### 17. Risks, mitigations, and final readiness criteria

- **Drift speed risk controls**: The rationale is blocking release if a session feels like leveling up or three weeks feels unchanged.

- **Saturation individuality controls**: The rationale is preventing older birds from converging into one expressive behavior without inventing negative drift.

- **Presence inflation and exclusion controls**: The rationale is preventing background tabs, duplicate devices, retries, stale focus, or unsupported input modalities from distorting change without invasive tracking.

- **Lost/duplicate personality controls**: The rationale is preventing client overwrite, tick retry, stale replica, migration reseed, and partial commits from corrupting identity.

- **Stale/inconsistent reaction controls**: The rationale is keeping immediate cues and next-tick decisions consistent across devices.

- **Audio, accessibility, and performance risk controls**: The rationale is preventing uncanny loops, inaudible/faked sound, frozen reduced-motion, caption/narration failures, first-bird misses, leaks, and lag.

- **Privacy, export, notification, deletion, notebook, and feature-drift controls**: The rationale is preventing data leakage, hidden vector exposure, reminder creep, incomplete deletion, generic/false notebook entries, and scope expansion into counters, geography, shared presence, or more birds.

- **Final readiness criteria**: The rationale is the integrated v1 promise: an owner can meet two birds, return across devices without resets, observe truthful weeks-scale change, interact without care obligations, hear/read recognizable calls, use accessible surfaces, keep visitors render-only, honor deletion/export, meet performance budgets, and avoid private relationship analytics.
