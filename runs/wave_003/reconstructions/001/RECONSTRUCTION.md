## System-level intent

- Contemplative idle attention is the primary product posture. This shows up in the opening description of a "browser-based, contemplative virtual aviary where idle attention is the primary mode of interaction," in the client as an "observational viewport," in "continuous micro-motion," and in the "tranquil ambient sky field" that appears "never a spinner."
- The system is web-only and scene-centered. The plan repeats "Evergreen web browsers," "Web-only," "No Native Mobile Clients," TypeScript plus Canvas2D plus WebAudio, and a "single horizontal scene" that avoids zooming, panning, and "cropping birds."
- Attention must be non-gamified and non-punitive. Progressive adoption is "monotonically unlocked strictly by aviary age" while "refusing any engagement/visit-frequency gates"; the non-goals ban "XP, leveling, streaks" and counters; the drift section says traits "NEVER decay or drop on neglect" and that "absence produces ambient calm, not wariness or punishment."
- Server authority protects consistency and prevents overwrite behavior. The plan names an "authoritative server-side simulation tick," says "Server is the sole author of world state," sends client activity into an "idempotent append-only log," and rejects "No-Last-Write-Wins" because neither device sends absolute state.
- Privacy is a hard boundary around identity, telemetry, and social viewing. The plan uses "internal synthetic UUIDv4," encrypted email, "deterministic HMAC-SHA256 blind indexing," prohibitions against email in logs/cache/telemetry, and telemetry that contains no "per-bird state or interaction histories."
- The aviary should feel procedural, ambient, and alive rather than asset-driven or announcement-driven. The plan uses procedural ambient drift, procedural WebAudio motif grammars, "Zero recorded audio files," staggered chorus responses, return-greetings without "toasts, banners, or welcome-back text," and graceful silence with captions.
- The system hides explicit numerical machinery from users. "Trait vectors are strictly internal server state," direct numerical exposure is barred from API, DOM, tooltips, and debug panels, and the adoption cap is enforced by "audio call discriminibability bounds" rather than a visible counter.
- Accessibility is a designed equivalent surface, not a degraded fallback. The plan calls for "continuous naturalist screen-reader narration," real-time call captioning, keyboard navigation, WCAG AA contrast, and a reduced-motion mode that is "not an 'animations off' degradation" but uses "tranquil cross-fades."
- Product voice is split by surface. "Naturalist Voice" governs aviary, notebook, narration, and captions: "lowercase, present-tense, quiet, specific, no gamification jargon." "Matter-of-Fact Voice" governs system, auth, errors, sync conflict, and settings: "Capitalized, direct, clear, no affected warmth."
- Performance is part of the experience of aliveness. The plan budgets "< 500ms time-to-first-bird," "60fps steady-state runtime," "zero memory growth over 30 minutes," cached snapshots at the CDN edge, and "First frame aliveness."

## Per-feature whys

### Scope and Architectural Non-Negotiables

- Platform on evergreen web browsers: NOT RECOVERABLE FROM PLAN
- Responsive single horizontal scene without zooming or panning: The plan ties this to a browser "observational viewport" and later requires the scene to fit "without cropping birds or panning."
- Starting population of exactly 2 starter birds from a 6-species pool: NOT RECOVERABLE FROM PLAN
- Hard maximum of 7 birds per aviary: The plan gives "audio call discriminibability bounds" as the enforcing reason.
- Progressive adoption by aviary age: The plan says adoption is "monotonically unlocked strictly by aviary age" and explicitly refuses "engagement/visit-frequency gates," matching the no-streak/no-counter boundary.
- Return-Greeting: It exists for "procedural recognition of user arrival" within "1-2 seconds," with modulation by absence, boldness, and mood; staggered secondary responses create "natural organic flow" without announcement UI.
- Presence Accounting: The rationale is to count only validated attention: all three conditions must hold, and "Background tabs and unattended monitors accrue zero presence."
- Listen-In: The plan states the purpose directly: elevate the focused bird "to foreground" while ducking other birds "to ambient."
- Offers: Offers affect "short-term mood and long-term curiosity/boldness"; the per-bird cooldown "prevents saturating traits and preserves gestural meaning."
- Settle Gesture: It shifts the aviary to "warm evening dusk" and "quiets the chorus"; the "5-second reversal window" allows recovery from accidental clicks.
- Field Notebook: NOT RECOVERABLE FROM PLAN
- Passwordless email magic-link auth: NOT RECOVERABLE FROM PLAN
- Synthetic UUID tenant isolation: The rationale is the privacy boundary: account identity is "solely" an internal UUID, with email encrypted and barred from logs, cache keys, telemetry, and message brokers.
- Authoritative server-side simulation tick: It keeps the server as the source of world state, evaluates drift and mood centrally, and supports cross-device consistency.
- Single canonical aviary per account: The plan connects this to "real-time snapshot pull with cross-device consistency."
- Real-time snapshot pull and SSE state updates: Cached snapshots target "<500ms TTFBird," and SSE carries "live aviary state updates," tick updates, weather, and notebook entries.
- Single-guest read-only ambient view: The rationale is "Quiet Social Affordance" without social-network mechanics: one guest, read-only, revocable, and ambient.
- Revocable 30-day magic link for guests: The plan's reason is bounded quiet social access: a single-guest link that can expire or be revoked.
- Zero visitor presence drift impact: Guest viewing must not alter the owner's aviary; the plan says "zero visitor presence drift impact."
- Private visit log: The plan frames the visit record as private within the quiet social affordance.
- No Native Mobile Clients: NOT RECOVERABLE FROM PLAN
- No Gamification or Streaks: The plan rejects XP, leveling, streaks, badges, achievements, adoption counters, visit counters, and green-dot calendars to preserve the age-based, idle-attention posture.
- No Tamagotchi Dynamics: The plan explains that absence should produce "quiet, ambient behavior" and never "punishment or negative drift."
- No Social Network Features: The rationale is to keep social affordance quiet and bounded: no directories, discovery feeds, profiles, follows, chat, comments, co-presence, or leaderboards.
- No Direct Numerical Exposure: The plan says trait vectors are "strictly internal server state" and must not leak through API, DOM, tooltips, or debug inspection panels.
- No Recorded Audio Fallback: The plan requires procedural WebAudio and says unsupported environments should become "graceful silence with captions enabled."
- No Announcement UI: The rationale is that greeting happens through product behavior, with no "toasts, greeting banners, or 'welcome back' modals."

### System Architecture & Topology

- Client Application using TypeScript, Canvas2D, and WebAudio API: The plan gives its role as an "observational viewport and local synthesizer."
- Client rendering of birds at perch positions with continuous micro-motion: This supports the contemplative viewport and "First frame aliveness."
- Client procedural ambient drift: NOT RECOVERABLE FROM PLAN
- Client procedural call synthesis from motif grammars: The plan ties this to generating every call at runtime through WebAudio and "Zero recorded audio files."
- Client presence tracking and batched event sending: This feeds the server-side presence accounting and append-only interaction events.
- Edge Gateway & API Service: It manages magic-link authentication, session tokens, and route rate-limiting while serving cached snapshots for "<500ms TTFBird."
- Append-only interaction event ingestion: The rationale is "interaction provenance," idempotence, and no client overwrite of world state.
- SSE channels: They exist for "live aviary state updates."
- Authoritative Simulation Service: It runs the 60-second tick, additive drift equations, Markov mood state machine, and sparse notebook generation.
- PostgreSQL relational store: NOT RECOVERABLE FROM PLAN
- Append-only event log table: The plan says it preserves "interaction provenance."
- Email encryption and blind indexing: The rationale is private lookup without placing email into application logs, cache keys, telemetry, or message brokers.
- Aggregate-only operational telemetry: It measures health while being "explicitly barred" from per-bird state or interaction histories.

### Data Models & Schema Design

- `accounts`: NOT RECOVERABLE FROM PLAN
- `auth_identities`: The email fields support encrypted storage and blind-index lookup under the privacy boundary.
- `sessions`: NOT RECOVERABLE FROM PLAN
- `aviaries`: Its unique account link, version, settled state, and tick timestamps support the "single canonical aviary," server ticks, and snapshot versioning.
- `birds`: The species, perch, trait, and mood fields hold the internal state used by drift, rendering, mood transitions, and call grammar.
- `linteraction_eventsa`: The table supports append-only interaction provenance and server-side event ingestion.
- `notebook_entries`: NOT RECOVERABLE FROM PLAN
- `visit_invitations`: The token, expiration, and revocation fields support the "revocable 30-day magic link."
- `visit_logsa`: NOT RECOVERABLE FROM PLAN

### Simulation Engine Design

- Server-Side 60-Second Tick: The tick centralizes active aviaries, unprocessed events, validated presence minutes, drift, mood transitions, notebook generation, and atomic version increments.
- Querying active sessions or unprocessed events: NOT RECOVERABLE FROM PLAN
- Presence minutes validated by tri-condition heartbeats: This prevents background tabs and unattended monitors from accruing presence.
- Additive personality drift: The monotonic equation makes presence-time dominant while ensuring traits "NEVER decay or drop on neglect."
- Calibration target for drift visibility: Harness instruments should detect drift after about "1 week of regular visits," while users should perceive it after about "3 weeks."
- Fast-timescale mood state machine: It allows moods to respond to diurnal local time, weather, recent offers/interactions, and boldness.
- Five core moods: NOT RECOVERABLE FROM PLAN
- Diurnal Cycle modulation: Dawn/morning, evening/settled, and night push mood toward ALERT, CONTENT, DROWSY, or sleep, matching local time.
- Weather modulation: Rain dampens vocal frequency and nudges quiet moods; wind raises ALERT or WARY based on inverse boldness.
- Interaction modulation of mood: Accepted offers nudge birds toward CONTENT or CURIOUS, while high-boldness birds resist WARY transitions.
- Species motif library: It gives each species 4-6 musical primitives for procedural calls.
- Timing and cadence shaped by vocal frequency and mood: This connects call behavior to internal personality and current mood.
- Chorus assembly: Staggered responses from high-social-warmth nearby birds form "harmonic chiming without phase-cancelling artifacts."

### API Surface & Client-Server Sync Model

- `POST /api/v1/auth/magic-link`: NOT RECOVERABLE FROM PLAN
- `GET /api/v1/auth/verify`: NOT RECOVERABLE FROM PLAN
- `GET /api/v1/aviary/snapshot`: It returns current aviary state "for hydration" and supports fast first bird visibility.
- `POST /api/v1/aviary/events`: It appends interaction events and presence heartbeats so clients send events, not absolute state.
- `GET /api/v1/aviary/stream`: It streams tick updates, weather, and notebook entries through SSE.
- `POST /api/v1/aviary/settle`: It supports the settle gesture that quiets the chorus and shifts lighting to dusk.
- `GET /api/v1/notebook`: NOT RECOVERABLE FROM PLAN
- `POST /api/v1/social/invites`: It creates the bounded "30-day read-only guest invite."
- `DELETE /api/v1/social/invites/:id`: It revokes guest access, matching the revocable quiet social link.
- `GET /api/v1/social/visits/:gest_token`: It serves the read-only viewport endpoint for guest viewing.
- No-Last-Write-Wins conflict prevention: The plan explains that because the server is sole author and clients never send absolute state, neither laptop nor phone can overwrite the other's attention.
- Idempotent append-only client event log: In-flight delays are safe because they are "processed at the next tick."

### Frontend Rendering & Audio Pipelines

- Canvas2D scene composition: It keeps one horizontal scene usable across viewports "without cropping birds or panning."
- Back plane perch zone: It provides depth 0.3 with "muted colors" and ambient presence.
- Middle plane perch zone: It provides depth 0.6 as "primary resting branches."
- Front rail perch zone: It provides depth 1.0 with "foreground, crisp detail" for bold birds.
- First frame aliveness: Birds load "mid-action" at snapshot positions; during cold fetch the user sees a "tranquil ambient sky field" rather than a spinner.
- Idle micro-motion: Saccadic glances, head tilts, feather fluffing, preening, and breathing oscillation keep the aviary alive in idle attention.
- Reduced-Motion Mode: The rationale is accessibility without "animations off" degradation, replacing movement with "tranquil cross-fades."
- Removing ambient leaf/feather drift in reduced motion: It reduces motion while keeping audio, captions, mood shifts, and notebook narration active.
- Procedural Audio Pipeline: Runtime WebAudio synthesis generates every call from motif grammars with "Zero recorded audio files."
- Dynamic Chorus & Listen-In Mix: Baseline ambient levels, focused-bird ramping, ducking, and smooth decay create foreground listening without losing chorus ambience.
- WebAudio fallback: Unsupported or blocked audio becomes "graceful silence with call captions automatically enabled."

### Interaction Protocols & UR Specifications

- Return-Greeting Protocol: It recognizes tab arrival in "1-2 seconds" through behavior based on absence duration, boldness, and mood rather than announcement UI.
- Short-absence greeting behaviors: A glance or soft head-tilt reflects a short absence without overstatement.
- Long-absence greeting behaviors: A hop to the front rail and full species call reflect longer absence.
- Staggered secondary bird responses: They ensure "natural organic flow."
- Presence Accounting Protocol: The three simultaneous conditions make presence accrue only when the document is visible, the window focused, and activity is recent.
- Sixty-second presence pings: NOT RECOVERABLE FROM PLAN
- Offers from the fading top bar: NOT RECOVERABLE FROM PLAN
- Per-bird 3-minute offer cooldown: The plan says it "prevents saturating traits and preserves gestural meaning."
- Settle lighting and chorus quieting: The settle gesture creates "warm evening dusk" and a quieter chorus.
- Five-second settle reversal: It allows "recovery from accidental clicks."
- Removing streak counters, visit-frequency widgets, and green-dot calendars: This maintains the no-gamification boundary.

### Accessibility & Voice Separation

- Screen-reader narration: The continuous `aria-live` prose narration makes the aviary available through a "naturalist voice" with specific present-tense observations.
- Narration update cadence of 30-60 seconds: NOT RECOVERABLE FROM PLAN
- Priority bumps for user-initiated events: Greetings and offers receive polite priority so initiated events are noticed.
- Call Caption Surface: Real-time subtitle badges adjacent to calling birds describe the procedural motif.
- Keyboard navigation: Tab, Shift+Tab, Arrow keys, Enter, and Escape provide comprehensive keyboard traversal and activation.
- Focus rings and contrast: They must pass "WCAG AA contrast ratios under all diurnal lighting conditions."
- Naturalist Voice: It protects product surfaces from gamification jargon through lowercase, present-tense, quiet, specific prose.
- Matter-of-Fact Voice: It keeps system, auth, error, sync conflict, and settings surfaces capitalized, direct, clear, and without "affected warmth."

### Performance Budgets & Observability

- Initial JS Bundle Size under 2.0 MB gzipped: NOT RECOVERABLE FROM PLAN
- Time to First Bird Visible under 500ms: It supports the fast "First frame aliveness" and edge-cached hydration.
- Steady-State Frame Rate at 60 fps: It protects the continuous micro-motion and idle viewport experience.
- Zero heap growth across a 30-minute session: It enforces the "Memory Invariant" for long contemplative sessions.
- Simulation Tick Latency p99 under 5.0 seconds: It bounds server-side tick responsiveness for authoritative updates.
- Operational telemetry: It measures "system health" through request counts, latencies, errors, frame drops, and WebAudio errors.
- Analytics privacy boundary: The plan bars per-bird state, presence logs, and interaction events from analytics warehouses or cross-account aggregation.

### Rollout Strategy, Population Ramp & Risks

- Phase 1 rollout: NOT RECOVERABLE FROM PLAN
- Phase 2 rollout: NOT RECOVERABLE FROM PLAN
- Phase 3 rollout: NOT RECOVERABLE FROM PLAN
- Phase 4 rollout: NOT RECOVERABLE FROM PLAN
- Population ramp from Day 1 through Day 360: The ramp implements age-based adoption and the "hard maximum cap" of seven birds.
- Drift Calibration Risk mitigation: Accelerated 90-day synthetic sessions in CI verify the one-week instrument drift and three-week perceptible drift targets.
- Audio Fatigue / Uncanniness mitigation: Procedural micro-variations in pitch, cadence, and offsets "eliminate phase-cancelling and loop artifacts."
- Error/Sync Overwrites mitigation: Clients never mutate state, and additive server-side event ingestion prevents drift loss across devices.
- Voice Contamination mitigation: Architectural linters prevent naturalist prose in system/sync errors and system jargon in product surfaces.
