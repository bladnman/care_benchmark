## System-level intent

- Quiet, ambient aliveness over announcements. This shows up in the "quiet sky field," "NO spinner," "no toast," top-bar fade, and the ship gate "No welcome toast/banner on any return path." The plan treats "Welcome back" patterns as a culture risk: "'Welcome back' creep" is mitigated by prose lint and design review.

- Server-authoritative simulation with the client as renderer and event emitter. The plan repeats "no client-authored personality writes," "Clients are pure event emitters," and "All personality and mood authority lives in tick worker." This also appears in the authority table: the simulation tick owns vectors, mood, perch assignments, weather, notebook generation, and canonical positions; the client never writes personality state or advances the simulation clock.

- Relationship growth without punishment. The drift principle is "monotonic toward expressive; no negative drift on neglect." Presence is calibrated longer because "watching without moving is the product." Bird unlocks are by "Age not visit count" to avoid gamification, and inactive birds become "ambient" through mood/perch rather than lower personality.

- Anti-gamification and anti-Tamagotchi boundaries. The plan explicitly says not to build "achievements, streaks, levels, badges," "death, hunger, distress meters, decay-on-neglect," or extension hooks for them. Notebook generation "never logs user visit frequency or streaks," and prose lint blocks "gamification words."

- Privacy and read-only social by default. Social is limited to "single read-only visit invitations"; visit invitations are "off by default," revocable, expiring, and visitor presence is discarded. Observability is "aggregate only" and must never collect per-bird state, personality values, notebook content, or per-account interaction sequences.

- Accessibility as a designed surface, not an afterthought. Accessibility appears in v1 scope and rollout, with "screen-reader narration (naturalist prose)," "reduced-motion designed surface," captions, keyboard navigation, and WCAG AA. The risk table says to "Ship a11y with v1, not after" because it is "legal + product ethics."

- First-bird immediacy and performance as part of the core conceit. The plan sets <500ms time-to-first-bird, <2MB gzipped initial JS, 60fps idle, memory stability, and synthetic checks that assert a bird visible under 500ms. The first frame must show birds mid-pose and "never a spinner."

- Procedural, parameterized media instead of heavy assets. Snapshots carry motif IDs and timing params, "not audio"; the client synthesizes calls through WebAudio and captions are generated from the same params. The plan uses "No audio files shipped" and "No recorded fallback - ever" to keep the audio surface procedural and bounded.

## Per-feature whys

### Scope and defensible implementation calls

- Modern-browser, responsive single-screen horizontal aviary: NOT RECOVERABLE FROM PLAN

- Single-user accounts, one aviary per account, email magic-link auth, and per-device revocable sessions: NOT RECOVERABLE FROM PLAN

- Two starter birds and an age-gated cap of seven birds: The plan says third+ bird unlocks use aviary age because "Age not visit count" avoids gamification. It also delays more birds in rollout until drift calibration is validated.

- User-assigned renameable names and stable internal bird IDs: NOT RECOVERABLE FROM PLAN

- Server-side simulation tick: The plan keeps personality, mood, perch assignments, weather, notebook generation, and canonical positions on the server so clients cannot write personality state or advance the simulation clock.

- Presence accounting: The 5-minute window is chosen because the source says "few minutes, lean longer" and the plan repeats that "watching without moving is the product." Strict presence also mitigates "silent drift corruption."

- Listen-in: Listen-in minutes feed social warmth and vocal-frequency drift, while the mix ramps a focused bird up and others down but "never 0," preserving the chorus rather than muting the aviary.

- Offers of seed, song fragment, or still pool: Offers are interaction events that can bias mood and drift. The 3-minute per-bird per-type cooldown exists to "prevent drift saturation."

- Settle: Settle creates an evening quiet state: it pushes birds toward `settled` / `drowsy`, shifts lighting immediately, and has a post-settle "evening quiet" mood.

- Multi-device canonical state through snapshots plus append-only event log: This avoids last-write-wins because clients do not send personality absolutes; the tick worker merges events in log order.

- Visit invitations: The plan narrows social to read-only ambient visiting so host drift is unaffected. Invitations are off by default, revocable, expiring, and logged for host visibility.

- Accessibility surface: Screen-reader narration, reduced motion, captions, keyboard navigation, and WCAG AA are in v1 because accessibility is treated as product ethics and a ship requirement.

- Audio surface: Procedural WebAudio avoids shipped audio files, allows captions from descriptors, and provides a silence-plus-captions fallback when WebAudio fails.

- Performance surface: The budgets protect first-bird immediacy, idle smoothness, small payloads, and stable memory; the risk table names first-frame loading as a violation of the "core conceit."

- Explicit non-goals: The no-build/no-stub/no-extension-hook rule prevents gamification, Tamagotchi decay, social-network expansion, push/digest state nudges, payments, recorded fallback, and numerical personality exposure from entering v1.

- Mood enum: The selected moods come from plan rationale: "PRD examples + settle state"; `settled` is specifically "post-settle evening quiet."

- Normalized personality trait range: `[0.0, 1.0]` floats are used as "Normalized scalars" and are "never shown to user."

- Tick cadence: The 60-second default follows the plan's rationale for the source phrase "~once per minute," with environment configurability.

- Tech stack and hosting: The plan justifies React/Vite, Node/Fastify, PostgreSQL, Redis, and CDN-backed hosting with "Team velocity," browser WebAudio, budget fit, and meeting <500ms first-bird with small snapshot payloads.

### Architecture and data model

- Service topology and authority boundaries: Separate client, API/auth, simulation, PostgreSQL, and Redis components preserve ownership: tick logic owns canonical simulation state, the client owns rendering/audio/presence/UI, and the event log stores interaction records rather than derived truth.

- Snapshot render pipeline: The client receives compact snapshots and does not replay the full event log. The plan says this keeps snapshots "kilobyte-scale" and preserves procedural audio variation client-side.

- Repository layout: NOT RECOVERABLE FROM PLAN

- Account privacy fields: Synthetic account IDs are the "only identifier in logs, telemetry, sharding." Encrypted email supports auth emails, while email hash supports lookup/rate-limit without decrypting.

- Session records: NOT RECOVERABLE FROM PLAN

- Aviary timezone and settle storage: Timezone drives the local-time day/night cycle; `settled_until` stores the evening settle state.

- Bird record and tick-only personality JSON: Stable bird records carry species, name, mood, perch, pose, and personality, but the hard rule says personality JSON is written "only by tick worker transactions."

- Append-only interaction events: The event log records presence, listen-in, offers, settle, visibility, and tick ambient events so interactions can be merged and replayed as input to server simulation without becoming the source of derived state.

- Derived presence segments: Presence segments are derived from `presence_ping` events and are not directly client-writable, supporting server-side presence integrity.

- Notebook entries: Entries store "lowercase naturalist prose" and optional trigger events, matching the notebook voice while grounding entries in sparse triggers.

- Visit invitation and session records: Token hashes, expiry, revocation, and host audit logs support revocable read-only visiting and host visibility into visits.

- Snapshot schema omitting personality vectors: The plan says personality vectors are "omitted" from client snapshots and raw personality must not be numerically exposed anywhere.

### API surface

- New account bootstrap: The bootstrap creates an aviary, selects two starter species from a deterministic seed based on account UUID "for reproducibility," prompts naming client-side, and records internal adoption events.

- Aviary snapshot, event append, and notebook endpoints: Snapshot reads provide the current `AviarySnapshot`, `?since_version=` allows 304 responses, event append batches interaction events, and notebook uses cursor-based pagination newest first.

- Bird rename endpoint: NOT RECOVERABLE FROM PLAN

- Bird availability and adoption endpoints: Availability returns the next age-gated adoption offer only when eligible, keeping the bird-unlock model tied to aviary age rather than visits.

- Account settings, export, and delete/cancel endpoints: NOT RECOVERABLE FROM PLAN

- Host visit endpoints: Invite, list, revoke, and visit-log routes support the plan's limited social surface: email invite, revocable invitation, and host-readable visit history.

- Visitor token read path: Visitor exchange and snapshot routes use a separate token and expose no event write endpoints, so visitors get the same renderer without affecting host simulation.

- Optional SSE stream: Polling ships first; SSE is only added if "snapshot latency matters in dogfood," keeping real-time complexity conditional.

- Error contract: Errors use HTTP problem+json with "matter-of-fact copy" and "never naturalist voice," preserving product voice boundaries for failures.

### Simulation engine design

- Tick pipeline and slow tick: The minute pipeline locks an aviary, loads state, processes events, updates presence, mood, drift, perches, weather, calls, notebook triggers, and snapshots. All aviaries continue ticking so mood/time advance; inactive aviaries move to slow tick "to save compute."

- Drift function: Drift is monotonic toward expressive, routing presence, listen-in, offer acceptance, and offers-near-bird into specific traits while capping at 1.0.

- Drift configuration and calibration: Constants live in `simulation/config/drift_v1.json` so they can be tuned "without code deploy via feature flag." CI calibration demands measurable vector change by 7 days and visible behavior by day 21.

- Neglect behavior: If no presence occurs for 14 days, drift deltas go to zero, but mood and time still move. Birds become "ambient" through greeting probability, mood, and perch, "not lower personality."

- Mood transition FSM: Mood uses local hour, weather, interactions, neighboring moods, and personality. It persists across sessions with "no reset on tab open," preserving continuity.

- Return-greeting directive: After `session_visible`, the server computes absence, chooses a greeting bird from social warmth and boldness modulated by mood, and sends a directive for client execution with "no toast."

- Call grammar runtime: Species motif libraries produce `CallDescriptor` params; the client synthesizes audio and captions from the same params, such as "a soft three-note rise."

- Notebook generation: Sparse triggers and a 48-hour rolling rate limit keep entries notable rather than noisy. The notebook "never logs user visit frequency or streaks."

- Bird-to-bird interaction: NOT RECOVERABLE FROM PLAN

### Sync model

- Single canonical writer: Personality and mood authority lives in the tick worker, and clients are "pure event emitters," preventing client-side state from becoming canonical.

- Event ordering and idempotency: Per-session `client_seq`, server ordering by recorded time/session/sequence, and event UUID idempotency support retries without duplicate simulation input.

- Multi-device and offline behavior: Multiple devices append events and receive the same snapshot version; offline clients discard stale interpolation state on reconnect. "No last-write-wins" is possible because clients cannot send personality absolutes.

- Conflict UX: Expired links, revoked sessions, snapshot failures, and deleted aviaries get direct recovery actions such as retry, sign in again, reload/support, or account recovery.

- Visit read path isolation: Visitor tokens map to read-only snapshots, middleware blocks writes except heartbeat, and visitor presence is discarded so host drift is unaffected.

### Frontend rendering pipeline

- Boot sequence with quiet field and no spinner: Critical CSS shows a "quiet sky field" while snapshot loads, then renders birds mid-pose. Deferred chunks keep settings, visits, and notebook off the critical path.

- Scene composition, layers, parallax, and fixed perch zones: NOT RECOVERABLE FROM PLAN

- Motion system: Standard motion uses mood-selected preen, scan, fluff, and hop animation; reduced motion uses still poses and cross-fades so the result is an aesthetic surface rather than `animation: none`.

- Idle micro-motion: NOT RECOVERABLE FROM PLAN

- Top bar chrome auto-fade: NOT RECOVERABLE FROM PLAN

- Keyboard and focusable interaction wiring: Keyboard navigation gives a non-pointer path through the bar and birds, supporting the accessibility surface and QA tab order.

- Offer and settle gesture wiring: Offers post an event and wait for server mood reaction in the next snapshot; settle posts a settle event, applies immediate lighting locally, and listens for a 5-second undo.

- Visit read-only client: The visitor client strips write affordances while reusing the same renderer, matching the read-only invite promise.

### Audio pipeline

- Procedural audio pipeline: Call descriptors flow through a grammar engine, voice nodes, master bus, listen-in mix, and limiter. The plan ships "No audio files" and uses mood and personality to shape scheduling and timbre.

- Chorus mixing: Default gains normalize if more than three birds call at once "to prevent clipping."

- Listen-in mix: Focused bird gain ramps to 1.0 and others to 0.35, "never 0," so listen-in creates focus without silencing the ambient chorus.

- WebAudio fallback: If `AudioContext` fails, calls are disabled, captions auto-enable, and a matter-of-fact banner appears. The plan forbids recorded fallback "ever."

- Resource management: Reused oscillator nodes, pre-allocated noise buffers, no per-call `AudioBuffer` allocation, and a 30-minute heap-stability CI test protect memory budgets.

### Accessibility surfaces

- Screen-reader narration: An `aria-live="polite"` region outside the canvas uses the same snapshot/template voice as notebook prose. Positional bird labels are forbidden, preserving naturalist narration rather than mechanical coordinates.

- Reduced motion setting: The plan honors `prefers-reduced-motion` plus a settings override and keeps audio, notebook, and drift unchanged, so reduced motion changes presentation rather than simulation.

- Captions: Captions are generated from `CallDescriptor` and mood adjectives, rendered near the calling bird, and default on when audio is off.

- Keyboard, focus, and WCAG AA: Full tab order, visible contrast-passing focus ring, 4.5:1 chrome text, and caption backing plates make keyboard/focus/contrast part of the ship gate.

### Performance, observability, rollout

- Budgets and measurements: Bundle, first-bird render, idle FPS, memory, snapshot payload, and tick duration all have explicit budgets and measurement mechanisms, making performance a merge and ship condition.

- Privacy-safe observability: The plan collects aggregate histograms and counts for latency, tick duration, first render, audio failures, and bundle size, while never collecting per-bird state, personality, notebook content, or per-account sequences.

- Alerting and synthetic checks: Tick p99 above 5 seconds pages on-call; hourly Playwright checks from three regions assert visible birds under 500ms and 60 idle frames in 1 second.

- CI gates: Bundle size, drift calibration, memory soak, prose lint, and axe checks block merges so budget, relationship drift, product voice, and a11y do not regress.

- Internal dogfood: Phase 0 ships simulation tick and API only with a CLI snapshot inspector so drift constants can be tuned against automated scripts before the full client.

- Private alpha feature subset: NOT RECOVERABLE FROM PLAN

- Closed beta feature subset: NOT RECOVERABLE FROM PLAN

- Public v1 ramp: Gradual traffic ramp from 5% to 100% limits release risk while visits and bird unlock pacing are monitored.

- Birds-per-aviary ramp: Public beta starts with two birds only, and age-gated third bird enables only after drift calibration is validated in production RUM.

- Day-one instrumentation: First-bird render, snapshot lag, aggregate presence minutes, audio init failures, and notebook entries per aviary per week track the plan's main performance, sync, drift, audio, and prose risks without account-dimension dashboards.
