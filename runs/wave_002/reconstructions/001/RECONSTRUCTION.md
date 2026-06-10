## System-level intent

- **Notice-never-announce is a product law, not a vibe.** The plan names "notice-never-announce" in the opening constraints and converts it into review gates: "no surface may describe the user's behavior" and "no announcement-register UI." It also shows up in no spinner, the "quiet field" fallback, no welcome modals, no fast-forward replay after suspend, and the "muted-state icon" as the only autoplay acknowledgment.

- **The aviary must feel alive without becoming a game, chore, or Tamagotchi.** The scope excludes "gamification of any form" and "Tamagotchi mechanics"; the drift function makes neglect equal "zero input = zero drift," not decay. The rollout gate says the vertical slice must "feel alive," while the risk section warns that too-fast drift reads as "Tamagotchi" and too-slow drift reads as "screensaver."

- **Canonical life belongs on the server; clients render and report.** The repeated rule is "the server is the only writer of canonical aviary state." Clients "pull snapshots," "push interaction events," and never submit "an absolute state value." Sync correctness is designed so last-write-wins on personality is "unreachable, not just discouraged."

- **Presence precision is load-bearing.** Presence-time "dominates" drift inputs, but only when visibility, focus, and recent input agree. The plan calls presence corruption a "silent-failure class" and gives it headless honesty tests, server-side caps, and synthetic production checks.

- **Privacy is structural, not just policy.** The plan insists on "synthetic UUID account identifiers," email in "exactly one column," aggregate-only operational telemetry, no per-account dimensions, and a pipeline-level analytics boundary where the telemetry SDK cannot accept bird/account-state objects.

- **Product voice has two registers.** Naturalist voice belongs to the notebook, narration, and captions; matter-of-fact voice belongs to settings, auth, visits, and errors. The plan says "naturalist copy never appears in error responses" and repeats the distinction as "talking to the system vs. experiencing the product."

- **Birds are recognizable individuals, but their internals stay hidden.** The plan combines "stable bird identity," per-bird stable call seeds, recognizable call signatures, and hidden personality vectors. Product surfaces receive render-derived values, never trait values; the export is "the one sanctioned place raw vectors leave the server."

- **Accessibility is a first-class surface.** The plan says accessibility is "shipped with v1, not after" and treats narration, reduced motion, captions, keyboard navigation, and contrast as launch-blocking workstreams. Reduced motion is "a designed register," not an engineering fallback.

- **Performance is part of aliveness.** The plan makes "first bird visible <500ms," "60fps idle," and "zero memory growth" launch gates. The first frame is sky, perches, and birds already in motion; failures like "load spinner regressions" are called product-fatal.

- **Social access is observation, not co-presence.** Visits are per-invite, revocable, expiring, and read-only. Visitor presence and interactions "never feed the host's simulation," and the out-of-scope list rejects profiles, follows, discovery, comments, leaderboards, and co-presence.

## Per-feature whys

### Scope

- **Single-user accounts and one canonical aviary per account:** The plan keeps v1 centered on one private aviary and explicitly excludes shared or multiple aviaries and social-network surfaces. That supports a single canonical simulation and avoids data-merge or co-presence product surfaces.

- **Email + magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions:** Revocation matters because session tokens gate event writes; pings from revoked sessions are dropped, and conflict surfaces stay auth-level only.

- **Email change with verification:** NOT RECOVERABLE FROM PLAN

- **Account export:** The export is the sanctioned way account data leaves the server, including raw vectors. Neutral labels and an internal-values header satisfy the export requirement without turning the export into a stats dashboard.

- **Soft-delete for 30 days, then hard-delete:** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick:** The tick makes multi-device sync a property of the architecture: the server advances canonical state, consumes events in order, and is the sole writer of personality vectors and canonical bird state.

- **Hidden 5-trait personality vector:** The hidden vector lets the engine shape behavior while preserving the no-numeric-exposure rule. Client payloads receive render values, never trait values.

- **Monotonic-toward-expressive drift:** The plan uses monotonic drift to embody "no Tamagotchi": neglect creates no input and no decay, while regular presence slowly produces expressive change.

- **Presence-time as the primary drift driver:** Presence-time dominates because watching is the product. The plan weights presence about five times higher than interaction inputs and calibrates for week-scale visibility, not single-session movement.

- **Persisted fast-timescale mood:** Mood persists so tab-open never resets it. Aviaries with no client connected still tick because mood and time-of-day must advance.

- **Bird-to-bird interaction:** Adjacent-bird mood contagion and chorus hints make birds affect one another, giving the aviary more life than independent idle animations.

- **Stable bird identity:** Stable identity supports recognizable calls, stable seeds, named birds, and notebook observations that refer to the same bird over time.

- **Six-species pool:** NOT RECOVERABLE FROM PLAN

- **Age-gated bird offers up to a hard cap of 7:** The cap protects recognizability and performance. The age gates launch conservative, are server-configurable, and only loosen so no offered bird is retroactively revoked.

- **Return-greeting:** Return-greeting makes absence visible through the aviary rather than through user-facing announcements. Server-side selection keeps devices consistent and lets the notebook composer ground observations in real data.

- **Idle presence accounting:** The three-condition predicate prevents background tabs, stale focus, or idle sessions from inflating drift while still honoring "watching without moving."

- **Listen-in audio focus:** Listen-in re-balances the mix around one bird without muting the others. It creates focus while preserving an ambient floor and feeds duration events into the simulation.

- **Offers for seed, song fragment, or still pool with per-bird cooldown:** NOT RECOVERABLE FROM PLAN

- **Settle gesture with 5-second undo:** Settle closes the presence window cleanly, triggers an evening lighting ramp, and applies a mood-quieting nudge. Undo suppresses or cancels the settle event so accidental settling does not become canonical.

- **Read-only field notebook:** The notebook is server-authored during tick processing so entries are sparse, canonical, and grounded in actual state transitions. Read-only status prevents it from becoming a user-authored journal or task surface.

- **Non-scrolling horizontal scene:** NOT RECOVERABLE FROM PLAN

- **Three perch zones:** The zones give the simulation and UI shared spatial structure: bird positions, adjacent-zone mood contagion, keyboard ordering, and animated movement all refer to perch zones.

- **Local-time day/night cycle:** The cycle is tied to the user's local time, so the server stores the last reported IANA timezone and ticks against it. This supports both mood baselines and the visual palette.

- **Rare ambient weather:** NOT RECOVERABLE FROM PLAN

- **Client-side ambient ornaments:** Leaves and feathers are client-only, seeded, and excluded from reduced motion so they add ambient life without becoming simulation state.

- **Top bar with idle fade:** The fade keeps chrome quiet during watching, while focus and open panels force full opacity so accessibility overrides the fade.

- **Load-with-motion-in-progress and quiet field fallback:** The plan avoids spinner or entry animation because load should not announce itself. The first frame should already feel alive, and the fallback stays in the same quiet register.

- **Procedural audio with no recorded audio:** Procedural WebAudio satisfies the no-recorded-audio rule, avoids sample assets, and gives each call runtime variation while keeping signatures recognizable.

- **Per-bird recognizable call signatures:** Recognizable signatures let birds become identifiable individuals; the plan even asks whether testers can name a bird by ear after dogfood.

- **Read-only visits:** Visits give controlled observation without social-network mechanics. Visit tokens can only read snapshots and keepalive, never emit interaction events.

- **Visitor presence and interactions never feed the host simulation:** This protects the host's simulation from outside influence and keeps visits observational rather than co-present.

- **Visit log in settings:** The log records visit sessions and duration for the host, matching the plan's account/settings surface rather than adding a social feed.

- **Visit notifications off by default with opt-in:** The default supports the broader no-push/no-email-notifications posture; opt-in keeps notifications an account setting, not an aviary attention loop.

- **Accessibility shipped with v1:** The plan treats accessibility as launch-blocking so reduced motion, narration, captions, keyboard navigation, and contrast are not fast-follows.

- **Synthetic UUIDs, encrypted email, and aggregate-only telemetry:** These choices isolate PII and prevent product analytics from carrying per-account or per-bird state.

### Architecture

- **Deliberately small service shape:** The plan keeps the system to an API service, simulation tick, notebook/narration composer, and edge/CDN so the architecture stays small and explicit.

- **Stateless API service:** Statelessness supports horizontal scale for auth, reads, event writes, settings, export, deletion, and visits.

- **Simulation service as sole canonical writer:** The tick owns personality vectors and canonical bird state so concurrent clients cannot overwrite one another.

- **Notebook composer inside the simulation service:** Notebook entries are generated immediately after a tick from the same state transition, so observations are grounded in actual state changes.

- **Client-side live narration with shared grammar:** Live narration is client-side to keep latency low, while the shared grammar keeps narration, captions, and notebook prose in one voice.

- **Edge/CDN serving app shell and first snapshot:** Edge delivery exists for time-to-first-bird; the first state snapshot is critical to rendering birds immediately.

- **Clients as renderers and event reporters:** This boundary is enforced by API design, because no endpoint accepts personality, mood, perch position, or absolute state.

- **State layer, renderer, audio engine, and chrome split:** The state layer is the only network-aware module, letting reduced motion swap the renderer without touching state or audio and keeping chrome code-split from the hot path.

- **Fixed tick cadence with per-account jitter:** The 60-second tick and +/-10-second jitter flatten load while preserving the minute-scale simulation model.

- **Idle aviaries still ticking:** Even with no client connected, mood and time-of-day advance. Cheap idle ticks make cost scale with events rather than raw account count.

- **Tick idempotency with tick sequence and event-log offset:** Conditional writes prevent crash/retry from double-applying drift or mood transitions.

### Data Model

- **Synthetic UUID keys on all tables:** UUIDs keep non-PII identifiers throughout the data model, matching the privacy principle.

- **Email in exactly one encrypted column on account:** The one-column discipline limits PII spread and is enforced by lint and schema review, not only policy.

- **HMAC email hash for lookup:** The hash supports sign-in lookup while keeping the raw email non-derivable without the key.

- **Session device descriptor with no fingerprinting:** The descriptor gives the user enough session context for revocation without creating a fingerprinting surface.

- **Aviary as its own entity:** The plan models the aviary separately so age gates, tick sequence, event offset, weather, and settled flag live off the account record.

- **Per-trait drift accumulators:** Accumulators let sub-visible drift accrue continuously, keep single-tick writes small, and make drift-rate instrumentation straightforward.

- **Append-only interaction event log:** Append-only input preserves ordering, enables tick replay/debug, and keeps client input additive rather than overwriting state.

- **90-day retention for consumed events:** Retention is for replay/debug; the plan says canonical state, not the log, is the system of record.

- **Static species config:** The same species parameters ship with app and server so silhouette, palette, motif library, and night-active behavior are a shared contract.

- **Exactly one nightjar-like species:** NOT RECOVERABLE FROM PLAN

- **Export labels for raw vectors:** Neutral internal names and a header satisfy export contents without making raw vectors a product stats surface.

### API Surface

- **Auth request always returns 202:** This avoids an account-existence oracle.

- **Magic links are single-use, short-lived, and rate-limited:** The risk section frames auth-link abuse as a threat to account access and privacy commitments; atomic consume, expiry, and rate limits reduce replay and spam.

- **Snapshot endpoint as the hot path:** A compact snapshot gives the renderer everything needed for birds, weather, tick sequence, and server time in a few KB.

- **ETag and tick-sequence conditional snapshots:** Unchanged pulls return cheap 304s, making polling viable.

- **Server-derived plumage render parameters:** The client gets render parameters rather than trait values, preserving the hidden-personality rule.

- **Snapshot pull triggers on load, visibility, render gap, and visible keepalive:** These triggers keep visible clients fresh and force recovery after suspend or rendering gaps.

- **Polling over WebSockets at v1:** The plan chooses polling because a one-minute tick has nothing useful to push at sub-poll latency, and polling removes connection-state bugs.

- **Batched event append:** Batching reduces chatter while preserving the append-only input model.

- **sendBeacon terminal flushes:** Terminal flushes keep tab-close presence-end from being lost.

- **Client-generated event IDs and server dedupe:** Dedupe makes retries idempotent.

- **Presence ping interval and wall-clock caps:** Pings accrue recent presence, while caps prevent forged or inflated pings from exceeding real elapsed time.

- **Bird rename endpoint:** NOT RECOVERABLE FROM PLAN

- **Bird offer acceptance with no decline penalty:** Keeping an unaccepted offer available prevents offers from becoming pressure, penalty, or gamified urgency.

- **Visit tokens scoped to snapshot reads and keepalive:** Scope enforces read-only visits at the token level.

- **Revocation and expiry checked on every visit snapshot pull:** The next pull can return "visit no longer available," so host control remains effective without a push channel.

- **Matter-of-fact API-adjacent copy:** Errors are system surfaces, so they use matter-of-fact `user_message` copy rather than naturalist prose.

### Simulation Engine Design

- **Bounded, monotonic drift function:** Non-negative accumulation and no trait decay make the "no Tamagotchi" rule a CI invariant.

- **Diminishing returns near ceilings:** Diminishing returns prevent saturation and preserve visible week-3 movement.

- **Calibration harness built first:** The plan calls this the highest-leverage early artifact because calibration is in-scope and the stack otherwise cannot prove one-week measurable drift, three-week visible drift, and no single-session visible movement.

- **Mood enum persisted in bird record:** Persistence keeps tab-open from resetting mood and lets snapshots report current mood.

- **Slow pull toward time-of-day baseline:** This avoids a discrete midnight reset, so users never observe a snap.

- **Last-reported IANA timezone:** This is the simplest honest model for a single-user account while supporting local-time day/night.

- **Mood contagion and chorus hints:** Adjacent wary transitions and overlapping high-vocal-frequency windows let birds influence each other and produce chorus moments.

- **Server-scheduled call timing with client-side synthesis:** Server timing keeps multiple devices, narration, and captions consistent; client synthesis keeps audio procedural.

- **Call schedule extrapolation between snapshots:** Extrapolation keeps calls from starving at poll boundaries, and the next snapshot re-anchors timing.

- **Seeded per-rendition variation within signature bounds:** Variation is real, but the motif skeleton and pitch center stay stable enough for recognition.

- **Server-side greeting selection:** Server selection keeps multi-device greeting behavior consistent and lets the notebook composer observe greetings from real data.

- **Greeting variety pressure:** Variety pressure prevents the greeting bird from always being the boldest.

### Sync Model

- **One writer guarantee:** No state-write endpoint exists, so personality conflicts are unreachable by construction.

- **Append-only client input across concurrent sessions:** Events from multiple devices interleave by log offset and are additive, never conflicting.

- **Read-your-tick freshness with up to about 60 seconds of skew:** The plan accepts brief skew because no UI displays cross-device state side by side, making it unobservable in practice.

- **No data-merge UI:** There is no data to merge because clients do not write canonical state.

- **Suspend recovery by fresh snapshot:** The plan rejects fast-forward replay because replaying absence would announce the absence; the aviary simply resumes as it currently is.

### Presence Accounting

- **Visibility, focus, and recent-input predicate:** Presence requires all three so background tabs, unfocused windows, and stale idle sessions stop accruing drift.

- **Five-minute activity window:** The value leans longer because "watching without moving is the product," then gets calibrated with the drift harness.

- **Passive, throttled input listeners:** NOT RECOVERABLE FROM PLAN

- **Presence pings every 30 seconds while present:** This turns continuous client presence into bounded server-accruable intervals.

- **Presence-end markers on visibility change, blur, and pagehide:** These close the presence window cleanly, including on tab close.

- **Server-side presence caps:** Caps keep inflated or forged pings from exceeding real wall-clock time.

- **Presence honesty tests:** The tests cover background tab, focused-but-idle, watching-without-mouse, and focus flapping because this failure class is too important for code review alone.

- **Settle contributes nothing to drift:** Settle only closes presence cleanly and quiets mood; it is not a progression input.

### Frontend Rendering Pipeline

- **Canvas 2D baseline renderer:** Canvas 2D is chosen because one scene with up to 7 birds, soft parallax, and ornaments is within the reference hardware budget and keeps bundle and complexity down.

- **Lightweight reactive layer for chrome only:** Chrome can use ordinary DOM while the scene renderer avoids virtual DOM in the render loop.

- **Single frame object consumed by renderer and audio:** One in-memory current aviary frame keeps rendering, audio, captions, and narration aligned.

- **Continuous day/night tint curve:** Continuous sampling avoids a visible snap and ties the scene to local time.

- **Programmatic bird bodies and pose graphs:** Procedural silhouettes, plumage, poses, and seeded jitter keep birds varied without making them idle in phase.

- **Snapshot deltas that glide rather than teleport:** Gentle reconciliation preserves the sense of living continuity across server updates.

- **Immediate boot render with birds mid-pose:** Poses are enterable at any phase, so the product can load in motion instead of playing an entry animation.

- **Stopping rAF and audio in hidden tabs:** Hidden-tab pause saves battery and pairs with a forced fresh snapshot on resume.

- **Reduced-motion alternate renderer:** Cross-fades, removed ornaments, and slower tinting make reduced motion a designed surface using the same state layer.

- **Top bar fully opaque under focus or open panels:** Accessibility takes priority over idle fade.

### Audio Pipeline

- **One AudioContext and small synthesis graph:** The graph produces calls from oscillator/FM, noise, filters, and envelopes without samples or loops.

- **Pre-allocated synthesis voice pool:** Voice reuse serves the zero-memory-growth gate.

- **Chorus on a shared timeline:** Server hints and scheduling windows create overlapping calls while a gentle bus keeps the mix calm.

- **Listen-in ramps over about 1.5 to 2 seconds:** Smooth ramps make focus feel like re-balance rather than mute.

- **Ambient floor for non-focused birds:** Other birds remain audible so listen-in does not collapse the aviary into solo mode.

- **Settled/night audio behavior:** Settled state lowers call probability and mix level; at night only the nightjar-like species schedules calls.

- **Muted-state icon for autoplay policy:** The icon acknowledges browser audio constraints without violating the no-announcement rule.

- **Graceful silence with captions:** If WebAudio fails or is denied, captions preserve access to call events.

- **Runtime-generated call captions:** Captions describe what was actually synthesized, in the same naturalist voice, near the calling bird.

### Accessibility Surfaces

- **Polite ARIA live narration:** `polite` narration keeps updates observational and non-interruptive, never assertive.

- **Priority lane for user-initiated events:** Return-greeting, offer reaction, and settle can jump the idle narration queue while still staying polite.

- **Observational narration rather than state lists:** Naturalist prose preserves the product voice instead of exposing raw state.

- **Keyboard traversal through top bar and scene:** Keyboard support makes listen-in, bird selection, offer, and settle reachable without pointer input.

- **Focus ring specified against day and night palettes:** Focus visibility remains reliable across lighting extremes.

- **Automated contrast checks:** WCAG AA is verified against both day and night lighting, not just one palette.

- **Voice split in accessibility surfaces:** Settings use matter-of-fact copy, while narration and captions use naturalist copy.

### Performance Budgets and Observability

- **Initial JS bundle gate:** The <2MB gzipped gate and code-splitting protect load time and keep the critical renderer small.

- **Time to first bird under 500ms:** Edge-delivered snapshots, critical renderer in the entry chunk, and deferred settings/notebook/visits/auth all serve first-bird speed.

- **Sustained 60fps idle:** Frame-time assertions protect the long-running watching experience.

- **Zero memory growth over 30 minutes:** Long-session stability is enforced with heap snapshots and implementation levers like voice pools and bounded references.

- **Aggregate-only RUM:** Operational metrics are allowed only without account, bird, or interaction-history dimensions.

- **Synthetic fleet:** Continuous browser runs exercise load, greeting, listen-in, offer, settle, performance budgets, and presence honesty in production.

- **Simulation health metrics:** Tick duration, backlog, lag, and aggregate drift-rate histograms detect engine health and calibration drift.

- **No engagement funnels or retention cohorts:** The plan names these as deliberately unmeasured so no one adds them as instrumentation hygiene.

### Rollout and Risks

- **Engine-first foundations:** Tick, data model, event log, drift, calibration, presence tests, and auth land first because drift and presence are core correctness risks.

- **Vertical slice gate that must feel alive:** The internal review explicitly scores aliveness because canned greetings, audio loops, and spinners are product-fatal.

- **Full surface phase:** Species, offers, settle, notebook, greeting, day/night, weather, chrome, adoption, reduced motion, narration, captions, keyboard, export/deletion, and visits land together so v1 is complete rather than partial.

- **Hardening phase with dogfood data:** Real elapsed weeks are needed to validate the three-week visibility target, so team aviaries run from Phase 1 onward.

- **Launch with adoption capped at 2 birds:** This matches the designed launch state and gives the conservative age gates time to prove themselves.

- **Server-configured age gates that only loosen:** Gates can adjust without deploys, but never revoke an offered bird.

- **Standing review checklist:** The checklist converts load-bearing rules into process so they survive contributor turnover.

- **Drift calibration mitigation:** Slower drift is recoverable, but too-fast drift cannot be walked back without violating monotonicity, so launch should favor the conservative end of the band.

- **Audio prototyping and listening reviews:** Recognizability is tested by whether people can name birds by ear; the 7-bird cap is honored because recognizability can fail before feature ambition does.

- **Tone and scope erosion controls:** The plan guards against "the toast, the streak, the stats panel" with review gates, codified non-goals, product owner sign-off, and the intentional absence of per-account engagement data.
