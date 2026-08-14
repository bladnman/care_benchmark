## System-level intent

- **Observational attention over maintenance or retention.** This is stated directly in the headline: the product "centers on observational attention rather than custodial maintenance or gamified retention loops." It repeats through the strict refusals: "No Gamification," "No Tamagotchi Mechanics," no "streaks, scores, levels," and neglect becoming "ambient quietness, never penalty or negative trait drift."

- **Aliveness should be quiet, procedural, and slowly earned by honest presence.** The plan says "Aliveness is expressed through procedural calls, subtle mood-based idle animations, and server-side personality drift driven by honest presence accounting." The same intent appears in the 60-second tick, the monotonic drift formula, the "instrument-detectable" and "visibly distinct" calibration, mood transitions from time/weather/social contagion, and the procedural call grammar where "no two calls are identical."

- **Canonical inner life belongs on the server; the client observes and renders.** The client/server boundary says the "Server" is the "sole author and writer of canonical state, personality vectors, mood transitions, and notebook entries," while the "Client" is "pure rendering, synthesis, and observation." This also shows up in the hidden vectors, append-only event log, and multi-device mitigation where clients "never submit absolute trait values."

- **Keep hidden traits hidden.** The plan repeatedly refuses direct trait exposure: "Personality vector values are strictly hidden server-side," with "no stats screens, debug toggles, or telemetry leakage." Even the state API note says boldness, curiosity, and social warmth "remain strictly hidden on the server."

- **Social and telemetry surfaces must stay quiet and privacy-isolated.** Optional social is "quiet": one-time tokenized email invitations, "read-only ambient observer," "no co-presence," "no visitor drift impact," and "silent visit logging." The privacy architecture similarly says telemetry "never" contains Account UUIDs, Bird IDs, trait vectors, interaction counts, or notebook contents, and data warehouse pipelines have "zero read permissions" on the simulation database.

- **Accessibility and performance are part of the aviary, not an afterthought.** Scope includes WCAG AA, ARIA running prose, reduced motion, captions, <2MB bundle, <500ms first bird, 60fps, and zero memory growth. The risk matrix makes this explicit: accessibility degradation is mitigated by "Screen-reader running narration and call captioning built and tested in parallel from Phase 1."

- **Product voice splits between naturalist calm and plain account utility.** The Voice & Tone matrix reserves "Naturalist" for scene, narration, captions, notebook, offers, and settle: "Lowercase, present-tense, specific, calm." Auth, errors, sync conflicts, account/session management, and accessibility settings are "Matter-of-Fact," with "direct English, no charm pretense."

## Per-feature whys

### Executive Summary & Product Scope

- **Quiet browser-based virtual aviary and small group of procedural birds:** The plan frames this as a place for "observational attention" and a "quiet" virtual aviary, not custodial maintenance or retention loops.

- **Two starters, scaling up to 7 based on aviary age:** NOT RECOVERABLE FROM PLAN

- **Single-page evergreen-browser client with responsive horizontal viewport:** NOT RECOVERABLE FROM PLAN

- **Passwordless magic-link sign-in with 15-minute expiry:** NOT RECOVERABLE FROM PLAN

- **Revocable device sessions:** NOT RECOVERABLE FROM PLAN

- **Synthetic internal UUIDs, encrypted PII, data export, and 30-day soft deletion:** The plan ties these to "PII isolation," "never logs PII," encrypted `account_auth`, account export, soft deletion, and restore within the 30-day window.

- **Hidden 5-dimensional personality vectors per bird:** The rationale is to preserve hidden inner state: "No Direct Trait Exposure," no stats screens, no debug toggles, no telemetry leakage, and selected scalars remaining "strictly hidden on the server."

- **Monotonic-expressive personality drift via low-pass server ticks:** The plan connects this to "honest presence accounting," strictly nonnegative deltas, slow calibration, and the refusal that neglect never causes penalty or negative trait drift.

- **Fast-timescale mood state machine:** The plan uses moods to express aliveness through time of day, weather, bird-to-bird social contagion, and perch placement, giving birds visible short-timescale state without exposing trait vectors.

- **Procedural WebAudio calls and chorus management:** The plan refuses "canned MP3/OGG audio loops" and says procedural variation, motif primitives, jitter, pitch bend, and note count variations ensure "no two calls are identical."

- **Return-greeting engine:** NOT RECOVERABLE FROM PLAN

- **6-species pool:** NOT RECOVERABLE FROM PLAN

- **Idle presence detection with visibility, window focus, and activity:** The why is "honest presence accounting." The simulation validates all three criteria and rejects "fraudulent/bloated spans"; the risk matrix says strict presence signals prevent traits from maxing out in days.

- **Listen-in mix focus:** The feature gives attention to one bird while keeping the aviary ambient: the focused bird ramps up, others ramp down but "never zero/mute," and all return smoothly to equal ambient bus levels.

- **Offer gestures:** The plan treats offers as "quiet gesture prompts" and gives accepted offers a behavioral role: curiosity increases, and boldness can increase when the offer is near.

- **The specific offer kinds seed, song fragment, still pool, and per-bird cooldowns:** NOT RECOVERABLE FROM PLAN

- **Top-bar settle gesture:** The plan frames settle as a quiet naturalist prompt, "settle for the evening," and accessibility narration bumps immediately on settle.

- **5-second undo on settle:** NOT RECOVERABLE FROM PLAN

- **Field notebook generator:** The rationale is sparse naturalist observation rather than game logging. It emits only when a distinct threshold is met, rate limits to at most 1 entry per 2-3 days, uses lowercase naturalist voice, and is "never game logs or streak notes."

- **Optional quiet social visit invitations:** The plan's why is "Optional & Quiet": read-only ambient observation, no co-presence, no visitor drift impact, revocation, 30-day expiry, and silent visit logging, while refusing profiles, followers, comments, chat, leaderboards, and avatars.

- **WCAG AA, ARIA narration, reduced motion, and procedural captions:** The plan makes accessibility a core surface: running prose narration, call captioning, reduced-motion cross-fades, and parallel Phase 1 testing mitigate accessibility becoming a "secondary checklist."

- **Initial bundle, time-to-first-bird, frame rate, and memory budgets:** The plan ties these to instant visible aliveness: <500ms first bird, "no wakeup/loading spinner," 60fps steady rendering, and zero memory growth over 30 minutes.

- **No native mobile apps:** NOT RECOVERABLE FROM PLAN

- **No gamification:** The plan refuses streaks, scores, levels, badges, achievements, counters, calendars, and push notifications because the core product centers on observational attention rather than gamified retention loops.

- **No Tamagotchi mechanics:** The plan says birds never die, starve, or show distress, and neglect produces "ambient quietness, never penalty or negative trait drift."

- **No social network surfaces:** The why is to keep social optional and quiet: no profiles, followers, public discovery, comments, chat overlays, leaderboards, or visitor avatars.

- **No direct trait exposure:** The plan's rationale is hidden server-side personality and privacy: no stats screens, debug toggles, or telemetry leakage.

- **No recorded audio fallback:** The plan keeps sound procedural; if WebAudio is unavailable, fallback is "graceful silence" with procedural captions rather than canned audio loops.

### System Architecture & Boundaries

- **Edge API Gateway:** The plan gives it the boundary work of TLS, session-token validation, magic-link verification, synthetic Account ID substitution, append-only event ingestion, and snapshot caching so auth, PII isolation, event ingress, and fast reads are kept at the edge.

- **Simulation Worker Fleet:** The why is server-authored aliveness: it processes active and background aviaries, ingests events, applies drift formulas, transitions moods, evaluates chorus, emits notebook observations, and writes atomic snapshots.

- **PostgreSQL database with strict isolation:** The plan uses it for accounts, aviaries, bird state, event journals, notebook entries, and encrypted PII, with separate key management for `account_auth`.

- **Static CDN / Edge Delivery:** The rationale is performance: static assets under the <2MB gzipped bundle and edge-injected snapshot bootstrap for <500ms first bird render.

- **Server/client boundary:** The plan states the server is the sole canonical writer and the client is a rendering, synthesis, and observation engine. This protects hidden vectors and keeps drift, moods, and notebook entries server-authored.

- **Visitor boundary:** Visitors pull read-only snapshots and are blocked from presence or interaction events so they remain ambient observers with no visitor drift impact.

### Data Models & Database Schemas

- **`accounts` and `account_auth`:** These implement PII isolation: synthetic account UUIDs, encrypted email, and blind-index email lookup.

- **`sessions` and `magic_links` table details:** NOT RECOVERABLE FROM PLAN

- **`aviaries` and `birds`:** These store the aviary age, settled/weather state, hidden personality vector, mood, perch zone, greeting timing, and offer timing needed for server-authored drift, mood, perch, greeting, and offer behavior.

- **Append-only `interaction_events`:** The rationale is batched, processable event input for presence spans, listen-in spans, offers, and settle gestures. The risk matrix also uses append-only additive deltas to avoid multi-device drift overwrite.

- **`field_notebook_entries`:** The table preserves sparse prose records generated by the simulation worker as naturalist observations.

- **`visit_invitations` and `visit_logs`:** These support tokenized email visit invitations, revocation/expiry, and silent visit logging for quiet read-only social access.

### API Surface & Protocols

- **`GET /api/v1/aviary/state`:** The endpoint fetches the current canonical snapshot while preserving hidden traits; the note says boldness, curiosity, and social warmth remain strictly hidden on the server.

- **`POST /api/v1/aviary/events`:** The endpoint batches raw interaction events so the server can validate presence, process listen-in spans and offers, and apply canonical drift.

- **Authentication and account management endpoints as individual endpoint choices:** NOT RECOVERABLE FROM PLAN

- **Notebook and social APIs:** The rationale is access to the naturalist observation log and quiet visitor-invite management, including listing silent visit logs and revoking invitations.

### Simulation Engine Design

- **Atomic 60-second tick cycle:** The plan uses the tick as the server-side cadence for consuming events, computing drift, transitioning moods, generating notebook entries, evaluating adoption, and writing canonical state.

- **Aviary lease via `FOR UPDATE SKIP LOCKED`:** The plan presents this as a distributed lock for the atomic per-aviary cycle.

- **Interaction buffer consumption:** The feature exists so unprocessed presence, listen-in, offer, and settle events can be ingested before drift, mood, notebook, and snapshot work.

- **Presence validation:** The rationale is to require document visible, window focused, and recent activity, then reject "fraudulent/bloated spans."

- **Monotonic drift formula and calibration:** The plan explains drift as strictly nonnegative, saturating toward 1.0, slow enough that 1 week of daily 15-minute presence is "instrument-detectable" and 3 weeks is "visibly distinct."

- **Mood transitions and perch placement:** The plan uses local time, weather, social contagion, and boldness/mood-based perch probability to make bird state responsive and visible.

- **Sparse notebook generation thresholds and rate limits:** The why is to record distinct observations over 48-72 hours without turning them into frequent game logs: at most 1 entry per 2-3 days.

- **Age-based adoption milestones such as 60 days and 120 days:** NOT RECOVERABLE FROM PLAN

- **Canonical snapshot write and lease release:** The feature completes the server-authoritative tick by writing the state other clients read.

### Client Rendering & Motion Engine

- **Scene composition layers, three depth zones, procedural SVG birds, and deterministic particles:** NOT RECOVERABLE FROM PLAN

- **Instant-on mid-action initialization:** The rationale is the <500ms time-to-first-bird and "no wakeup/loading spinner" requirement. Server-time offsets place wings, preen angles, and particles mid-cycle on Frame 0.

- **Calm ambient sky when network snapshot is delayed:** The plan uses this to avoid spinners while still showing the aviary shell immediately.

- **Reduced-motion cross-fades:** The why is accessibility without disabling the experience: continuous skeletal animation is replaced by pose cross-fades, slow perch cross-dissolves, and suppressed leaves/feathers.

### Audio Synthesis & Chorus Pipeline

- **Dedicated WebAudio voice per bird:** The plan uses motif generation, oscillators, formant filters, ADSR, panning, and a master bus so each bird can have a parametric synthesis voice mapped to perch position.

- **Species acoustic primitives and runtime variation:** The rationale is procedural liveliness: 6 species, acoustic primitives, timing jitter, pitch bend, and note variation so no two calls are identical.

- **Vocal frequency modulation:** The plan ties high vocal frequency to more frequent unobserved calls and shorter response latencies to neighboring calls.

- **Listen-in mix dynamics:** The why is focused listening without muting the aviary: focused bird to +3dB, others to -14dB "never zero/mute," then smooth ambient return.

- **WebAudio fallback:** The plan says failure or permission block should produce a silent state with 0 CPU cycles and automatic procedural captioning.

### Accessibility Surfaces

- **ARIA live screen-reader narration:** The rationale is running naturalist prose for screen readers, paced every 30-60 seconds during steady observation and updated immediately on offer or settle.

- **Call captioning:** The plan uses captions to turn current procedural motif parameters into high-contrast visual call text, default-on when audio fallback is active.

- **Keyboard navigation:** The feature makes top-bar actions, scene bird focus, listen-in, and modal dismissal reachable via Tab, arrows, Enter/Space, and Escape.

- **High-contrast focus outlines:** The rationale is legibility across dawn, midday, and night palettes.

### Performance Budgets & Observability

- **Bundle, first-bird, FPS, heap, and tick-latency budgets:** The plan attaches each budget to a verification strategy: CI bundle analysis, Lighthouse, rAF audit, Puppeteer soak test, and Prometheus histogram alarms.

- **Telemetry boundary:** The rationale is "Strict Privacy Isolation": only system metrics are recorded, never Account UUIDs, Bird IDs, trait vectors, interaction counts, or notebook contents.

- **Warehouse isolation from simulation database:** The plan requires zero read permissions so telemetry pipelines cannot inspect simulation state.

### Voice & Tone Enforcement Matrix

- **Naturalist voice for scene, narration, captions, notebook, offers, and settle:** The plan's why is calm observation: lowercase, present-tense, specific, calm, and never game logs or streak notes.

- **Matter-of-fact voice for auth, errors, sync conflicts, account/session management, and accessibility settings:** The rationale is direct utility: "Clear standard capitalization, direct English, no charm pretense," and functional labels.

### Engineering Work Breakdown & Rollout

- **Implementation phase ordering across five phases:** NOT RECOVERABLE FROM PLAN

- **Strict three-signal presence mitigation:** The plan ties this to the risk that lenient presence measurement would make birds max out traits in days.

- **Per-species formant filters and micro-timing jitter mitigation:** The plan ties these to avoiding audio uncanniness, harshness, and phase cancellation from layered procedural calls.

- **Append-only event log and additive server-authored deltas mitigation:** The plan ties these to preventing multi-device drift overwrite when two active tabs submit conflicting state.

- **Accessibility built and tested in parallel from Phase 1:** The plan ties this to the risk of treating accessibility as a secondary checklist.
