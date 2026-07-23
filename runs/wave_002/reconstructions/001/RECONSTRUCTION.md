## System-level intent

- **Felt aliveness through technical integrity**: The plan states the "core mandate" as "felt aliveness through technical integrity." It ties this to "server-driven background simulation," "procedural call synthesis," and "seamless first-frame rendering without spinners." This intent recurs in the Simulation Tick Worker, WebAudio synthesis, idle micro-motion, day/night cycles, and hard performance budgets.
- **Feels alive, not robotic**: The plan explicitly names this product principle and grounds it in "server-driven background simulation," "procedural call synthesis," and mood-shaped motion. It shows up again in "mood-shaped idle micro-motion," "Fast-Timescale Mood Engine," "staggered procedural return-greeting," and the use of diurnal/weather/offers as mood inputs.
- **Notice, never announce**: The plan expresses this as "zero toast popups, zero textual welcome banners" and says return-greetings are "purely through subtle bird motion and staggered vocalizations." It also appears in "top-bar UI with auto-fade," "quiet top-bar naturalist notification," and "quiet mode gracefully without error popups."
- **Charm comes from specificity**: The plan names this principle and supports it through "Naturalist prose generation across the field notebook, screen-reader narration, and audio captions." It also appears in per-species motif libraries, mood/personality-modulated calls, dynamic captions such as "a soft three-note rise," and notebook prose such as "pip greeted before wren today."
- **Restraint over richness**: The plan grounds this in a "fixed single-screen horizontal viewport," "strict 7-bird cap," and "minimalist top bar with cursor auto-fade." It also appears in the explicit prohibitions on gamification, Tamagotchi mechanics, social-network surfaces, scrolling, zooming, panning, and direct personality mutation.
- **Naturalist voice for the product, matter-of-fact for the system**: The plan calls for a "strict architectural boundary between bird interaction surfaces (naturalist) and auth/settings/error surfaces (matter-of-fact)." It appears in field notebook prose, screen-reader narration, call captions, magic-link responses, and invalid/revoked visit errors.
- **Server owns canonical state; client renders and synthesizes**: The plan states that the "server holds sole ownership of canonical state and personality simulation," while the "client acts as a render-and-synthesis engine." This shows up in the server tick, append-only events, snapshot pulls, "No Client-Side Mutation," and "Race Condition Elimination."
- **Slow, non-punitive relationship over days and weeks**: The opening describes birds evolving "over days and weeks in response to idle presence and quiet interactions." The drift section says "Traits never decrease" and "Neglect reduces expressiveness by dampening mood activations, not by reducing underlying trait scalars." The non-goals reinforce that "Birds never die" and neglect never causes distress.
- **Privacy enforced by architecture and observability boundaries**: The plan isolates "PII (encrypted email)" and uses "synthetic UUID identifiers." It prohibits telemetry such as "per-bird trait values," "individual interaction event payloads," "user notebook text," and "visitor email tracking." The risk matrix repeats this under "PII Leakage in Telemetry."
- **Accessibility as part of the aviary voice**: Accessibility is not framed as raw system output. Screen-reader output is routed through the "naturalist prose generator," captions are "procedural call captions," reduced motion uses cross-fades rather than stripping all atmosphere, and focus rings guarantee "WCAG AA compliance across all day/night background palettes."

## Per-feature whys

### Included in V1

- **Web-only client**: NOT RECOVERABLE FROM PLAN
- **Single horizontal viewport**: The plan ties this to "Restraint over richness" through a "fixed single-screen horizontal viewport" and later specifies no scrolling, zooming, or panning.
- **3 perch zones**: The plan uses the zones to make state legible: "Back Perch" is for "Wary / Low Boldness Birds," "Middle Perch" for "Content / Mid Boldness Birds," and "Front Perch" for "Bold / Curious / Focused Birds."
- **Real-time day/night lighting cycles tied to local timezone**: The plan uses local time in the diurnal mood machine, including drowsy between 22:00 and 06:00 and alert from 06:00 to 09:00.
- **Ambient weather (rain, wind)**: Weather is part of "Environmental Mood Machine"; sudden rain can trigger `alert`, and weather events feed mood transitions.
- **Subtle leaf/feather drift**: The plan places this inside the living scene and later disables "leaf/feather drift particles" in reduced-motion mode, making it part of the normal-motion ambience.
- **Top-bar UI with auto-fade**: This supports "Notice, never announce" and "Restraint over richness" through a "minimalist top bar with cursor auto-fade."
- **Pool of 6 bird species**: NOT RECOVERABLE FROM PLAN
- **Starting aviary with 2 birds**: NOT RECOVERABLE FROM PLAN
- **Age-based unlocks capped at 7 birds**: The cap supports "Restraint over richness," and the unlock schedule supports birds that evolve "over days and weeks."
- **Hidden 5-trait personality vector**: The vector gives the bird engine hidden specificity while remaining server-owned; the plan also prohibits clients from writing personality vector numbers.
- **Fast-timescale daily mood engine**: The plan separates quick mood changes from slow personality drift so birds can respond to local time, weather, offers, and boldness.
- **Procedural call grammar synthesis**: This serves "Feels alive, not robotic" and "Charm comes from specificity" through species motifs, pitch/tempo modulation, and no shipped `.mp3` or `.wav` files.
- **Mood-shaped idle micro-motion**: This makes birds feel alive in the quiet viewport through preening, scanning, weight shuffle, and head tilt rather than robotic idling.
- **Staggered procedural return-greeting**: The plan says return-greetings should be expressed "purely through subtle bird motion and staggered vocalizations," avoiding banners and popups.
- **Listen-in with exponential audio mix cross-fading**: The mix focuses one bird while ensuring "No bird is completely muted," preserving the ambient chorus.
- **Offer gestures (seed, song fragment, still pool)**: Offers are quiet interactions that trigger `curious` for 3-5 minutes, scaled by the bird's `curiosity` trait.
- **Per-bird cooldowns on Offer gestures**: NOT RECOVERABLE FROM PLAN
- **Settle evening gesture**: Settle is explicitly one of the triggers for `drowsy` mood.
- **5-second undo grace period**: NOT RECOVERABLE FROM PLAN
- **Field notebook displaying naturalist observations**: This implements "Charm comes from specificity" through naturalist observations generated from canonical state.
- **Strict presence accounting**: The plan validates "true presence" only when the page is visible, focused, and active within 180 seconds, so drift reflects actual quiet presence.
- **Monotonic server-driven personality drift toward expressive**: The plan says traits move up and "NEVER down," with neglect dampening mood activations rather than reducing traits.
- **Magic-link authentication with 15-minute expiry**: NOT RECOVERABLE FROM PLAN
- **Single canonical aviary per account**: This supports one source of truth for sync and prevents conflicting aviary states.
- **Multi-device sync via server-side simulation ticks**: The plan says multi-device conflicts are "mathematically impossible" because clients ingest snapshots from the single ticking server.
- **Account JSON export**: NOT RECOVERABLE FROM PLAN
- **30-day soft deletion before hard purge**: NOT RECOVERABLE FROM PLAN
- **Encrypted email storage with synthetic UUID identifiers**: This protects PII and enforces the observability boundary against email leakage in logs or metrics.
- **One-time read-only visit invitations sent via email link**: This permits optional social access while avoiding profiles, feeds, co-presence, and state mutation.
- **Revocable host control**: Revocation keeps social visits under host control and maps to a revoked invite returning 410.
- **Silent visit logging**: The schema names this a "Visit Access Audit Log," while the product surface stays silent and avoids mandatory notifications.
- **Social off by default**: This supports "No Social Network Surfaces" and the setting default of `"visit_notifications": false`.
- **Naturalist screen-reader narration engine**: Accessibility remains in the product voice by generating narration from naturalist prose rather than raw ARIA data.
- **Reduced-motion cross-fade rendering mode**: The plan keeps the aviary usable for reduced-motion users by replacing flights with cross-fades and disabling continuous particles.
- **Procedural call captions**: Captions serve accessibility and also provide graceful output when AudioContext fails.
- **WCAG AA contrast**: The focus indicator and palette rules guarantee readable controls across day/night backgrounds.
- **Full keyboard navigation**: Keyboard users can traverse top bar controls and birds, activate Listen-in, and disengage with Escape.
- **Initial JS bundle under 2 MB gzipped**: The budget enforces the fast, restrained browser experience and is checked in CI.
- **Time-to-first-bird under 500 ms**: This directly supports "seamless first-frame rendering without spinners."
- **60fps idle motion**: The plan treats smooth motion as necessary for the quiet aliveness of the idle aviary.
- **Zero client memory growth over 30-minute sessions**: This keeps long idle sessions stable rather than degrading over time.

### Explicit Non-Goals and Out-of-Scope

- **No Native Apps**: NOT RECOVERABLE FROM PLAN
- **No Gamification**: The prohibition on streak counters, visit calendars, levels, XP, badges, counters, and rank boards supports "Restraint over richness."
- **No Tamagotchi Mechanics**: The plan rejects death, hunger, distress, and negative personality drift so the relationship stays non-punitive.
- **No Social Network Surfaces**: The plan avoids profiles, friend lists, chat, comments, feeds, leaderboards, co-presence, and mandatory notifications to keep social optional and restrained.
- **No Direct Personality Mutation**: Clients never write trait numbers or absolute state updates, preserving server-owned canonical personality.

### System Architecture and Service Topology

- **Decoupled client-server architecture**: The rationale is explicit: the server owns canonical state and personality simulation, while the client renders and synthesizes.
- **API Gateway and Auth Service**: It centralizes magic-link generation, session issuance/revocation, rate limits, SSL termination, and stripping sensitive headers.
- **Aviary API Service**: It serves compressed snapshots, validates and appends events, processes notebook queries, and verifies read-only visit tokens for the render client and optional social flow.
- **Simulation Tick Worker Engine**: It performs background 60-second loops for active aviaries, computing presence, drift, mood, and naturalist entries server-side.
- **Naturalist Prose Generator Subservice**: It uses deterministic templates and rules to produce notebook and screen-reader prose without external LLM dependencies.
- **Browser Client Application in Vanilla TypeScript/JavaScript**: NOT RECOVERABLE FROM PLAN
- **Canvas/WebGL render pipeline**: The browser is the render engine for the single-screen aviary and must hit first-frame and 60fps goals.
- **WebAudio synthesis engine**: The browser is the synthesis engine for real-time procedural calls rather than shipped recordings.

### Data Models and Schemas

- **Accounts table with encrypted email and settings**: This isolates PII and stores user-facing settings such as reduced motion, call captions, screen-reader narration, and visit notifications.
- **Aviaries table with timezone and weather state**: Timezone drives local day/night and diurnal mood; weather state feeds environmental transitions.
- **Birds table with `slot_index` between 0 and 6**: The constraint enforces the strict 7-bird cap.
- **Personality vectors table**: The server-only vector stores boldness, social warmth, vocal frequency, plumage saturation, and curiosity for drift and behavior without client mutation.
- **Bird moods table**: This persists the fast-timescale mood state-machine separately from slower personality traits.
- **Append-only interaction event log**: The server tick processes raw client events from this log, preserving the no-client-mutation model.
- **Notebook entries table**: This stores generated naturalist observations for the field notebook.
- **Visit invitations table**: This supports one-time read-only invitations with expiry and revocation.
- **Visit logs table**: This supports the silent visit/audit log described by the social feature and schema.

### API Surface and Protocol Specifications

- **Magic-link request endpoint**: The response is matter-of-fact system copy: "Check your email for sign-in link."
- **Magic-link verify endpoint**: Verification redirects with an HttpOnly session cookie or returns matter-of-fact JSON errors.
- **Aviary snapshot endpoint**: This gives the client server time, local offset, weather, birds, call signatures, and narration for rendering and synthesis.
- **Interaction events endpoint**: This accepts raw presence, offer, listen-in, and settle events for append-only server processing.
- **Notebook endpoint**: This exposes generated field notebook observations.
- **Social invite, revoke, and visit snapshot endpoints**: These keep visits read-only, revocable, and stripped of host controls and account metadata, with invalid/revoked errors in matter-of-fact tone.

### Simulation Engine and Drift Mechanics

- **Four-stage 60-second simulation tick**: The stages make event processing, presence validation, drift, mood transitions, and canonical DB commits atomic.
- **Presence validation using visible, focused, and active conditions**: This defines "true presence" before personality drift can occur.
- **Low-pass monotonic drift formula**: The formula makes drift slow, capped per tick, and additive so traits become more expressive without decreasing.
- **No drift when `presence_seconds = 0`**: The plan keeps neglect from punishing underlying traits.
- **Database calibration target after 7 days**: The target makes drift measurable in instruments after regular 15-minute daily visits.
- **User perceptual target after 21 days**: The target makes perch preference and call frequency visibly altered on a longer relationship timescale.
- **`drowsy` mood**: This mood exists for local night hours or after Settle.
- **`curious` mood**: This mood exists for the 3-5 minute response window after an Offer, scaled by curiosity.
- **`wary` mood**: This mood covers sudden ambient alarm calls or low boldness during initial return-greeting.
- **`content` mood**: This is the unbothered day-hours baseline.
- **`alert` mood**: This covers early morning and sudden rain weather events.

### Multi-Device Synchronization and Conflict Resolution

- **Client pull and interpolation**: Clients pull snapshots on initialization, tab return, and every 60 seconds, then smooth positions between snapshots with Bezier curves.
- **Race condition elimination**: The plan avoids last-write-wins vector updates because clients never mutate canonical state.

### Frontend Rendering Pipeline

- **Responsive canvas scaling from 320px mobile to 4K ultra-wide**: This preserves the fixed single-screen aviary across device sizes.
- **Back, middle, and front perch rendering details**: Distance, scale, plumage, and focus translate mood and boldness into visual placement.
- **Preening idle motion**: Content birds ruffle feathers every 40-80 seconds to add specific quiet life.
- **Scanning idle motion**: Head turns with easing keep birds from seeming static.
- **Weight shuffle idle motion**: A 2px vertical bounce adds restrained body motion on the perch.
- **Head tilt idle motion**: Birds react toward leaf drops or song offers, connecting ambient events and interaction.
- **Reduced-motion disabling of continuous animations and particles**: This respects prefers-reduced-motion while keeping the aviary usable.
- **Reduced-motion flight cross-fades**: Cross-fades replace perch flights with static pose transitions.
- **Reduced-motion slowed lighting transitions**: Ambient day/night remains but is slowed by 2x.
- **Inline initial snapshot from CDN edge**: This supports fast first render and the time-to-first-bird budget.
- **Immediate frame-1 birds in mid-action**: This avoids spinners and makes the first frame feel alive.
- **Quiet sky canvas background on slow fetch**: This avoids "fade-in from black" and loading spinners during cold network delay.

### Audio Architecture and Synthesis Pipeline

- **No recorded audio files**: The plan requires real-time WebAudio synthesis for all bird calls instead of `.mp3` or `.wav` assets.
- **Species motif library of 4-6 primitives**: Per-species oscillators and envelopes create specific bird call signatures.
- **Pitch and speed modulation by mood and vocal frequency**: Mood and personality alter pitch, tempo, responsiveness, and `wary` jitter.
- **PannerNodes based on bird X-coordinate**: Audio position matches the bird's horizontal canvas position.
- **Staggered timing algorithm**: This prevents simultaneous calls and preserves per-bird clarity up to the 7-bird cap.
- **Listen-In gain ramps**: The focused bird rises to +3 dB while others fall to -18 dB, so focus increases without muting the rest of the aviary.
- **Quiet mode on AudioContext failure**: The app enters quiet mode gracefully without error popups.
- **Naturalist call captions near the calling bird**: Captions replace failed or muted audio with specific visual descriptions.

### Accessibility Surface Implementation

- **Dedicated `aria-live="polite"` live region**: Screen-reader narration stays non-disruptive.
- **Periodic narration every 30-60 seconds**: Narration updates from canonical state in naturalist prose.
- **Priority event narration**: Return-greeting, offer reaction, and settle trigger immediate but non-disruptive narration.
- **Call captions adjacent to calling birds**: High-contrast caption overlays are unobtrusive and spatially tied to the sound source.
- **Captions derived from call motif characteristics**: The wording preserves call specificity, such as "a soft three-note rise."
- **Keyboard focus map**: Tab, ArrowKeys, Enter, and Escape provide full navigation and Listen-in control.
- **Dual high-contrast focus ring**: The white outer and black inner ring guarantee WCAG AA across day/night palettes.

### Performance Budgets, Optimization and Observability Strategy

- **Bundle analyzer or CI check for JS size**: This enforces the under-2 MB gzipped budget.
- **Lighthouse or Synthetic RUM for time-to-first-bird**: This enforces the under-500 ms first-bird goal.
- **Playwright heap inspection**: This verifies the zero-leak, 0 MB growth target over 30 minutes.
- **Datadog or Prometheus tick latency alerting**: This keeps p99 simulation tick latency under 5 seconds.
- **Allowed operational metrics**: Status codes, latency, tick durations, audio initialization failures, and WebGL context loss preserve operational insight without simulation content.
- **Prohibited telemetry**: Blocking trait values, event payloads, notebook text, visitor email tracking, and simulation table ingestion protects user privacy and PRD commitments.

### Release Strategy, Aviary Pacing and Rollout Plan

- **Unlocking progression through days 30, 90, 180, 270, and 360**: The plan frames this as "Aviary Pacing" and uses it to grow toward the 7th and final bird over time.
- **Phase 1 architecture and planning**: The current phase finalizes specs, schemas, and API contracts.
- **Phase 2 core engine and synthesis**: The next phase builds the simulation tick worker, WebAudio grammar, and Canvas rendering pipeline.
- **Phase 3 auth, sync and accounts**: This phase implements magic-link service, PostgreSQL schema, and the multi-device snapshot API.
- **Phase 4 accessibility and polish**: This phase delivers prose narration, reduced-motion cross-fader, call captions, and privacy audit.

### Risk Matrix and Mitigation Tactics

- **30-day CI simulation testing for drift**: This mitigates drift that is too fast or too slow by keeping deltas within the 0.02-0.05 range.
- **Strict prohibition of client state mutation**: This mitigates multi-device sync race conditions.
- **Pitch/rhythm micro-jitter and spectral filtering**: These mitigate procedural calls sounding synthetic or grating.
- **Automated DOM auditing for narration**: This mitigates screen readers reading raw ARIA data or announcements instead of naturalist prose.
- **Encrypted email storage and synthetic UUID enforcement**: These mitigate PII leakage in logs, metrics, and internal foreign keys.
