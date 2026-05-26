## System-level intent

1. Literal constraint-respect and restraint. The plan says "Explicit non-goals respected" and ends by saying "All decisions interpret PRD constraints literally." This shows up in the browser-only scope, the refusal of "gamification/streaks/achievements/levels/scores," "no Tamagotchi," "no social network surfaces," "no notifications," and "no personalization of scene beyond birds." It also shows up in UI language like "sparse icons," "auto-fade," "quiet-field loading state," and the risk mitigation against "Over-announcing temptation."

2. "Notice, never announce" as a product-voice invariant. The plan names "naturalist voice + 'notice, never announce'" as "load-bearing invariants." It carries through the "field notebook (naturalist voice)," screen-reader narration in the "same voice as notebook," captions in the "same voice," "quiet-field loading state," and lint rules plus "PRD references in every UI surface PR."

3. Presence honesty and non-punitive change. The plan explicitly names "presence honesty" as load-bearing, defines "presence-based monotonic drift," gives "presence-time dominant weight," calibrates drift to become "measurable after ~1 week" and "user-visible after ~3 weeks," and says "No negative drift on neglect." This also aligns with "no Tamagotchi (no hunger/death/distress)."

4. Server-owned canonical truth. The client is a "thin renderer + input capture + state subscriber" with "No ownership of canonical bird state." The plan repeats "No client-side simulation of personality," "Server is sole writer of vectors," "Server-only canonical state," and "all clients observe identical canonical aviary." The data model reinforces this with ordered event logs and "no LWW."

5. Procedural living variation over canned media. The plan uses a "procedural bird engine," "WebAudio procedural synthesis," "motif library per species," "personality-shaped timing/pitch variations," and "client-generated ornaments." It explicitly says "No recorded loops at any point" and mitigates "Audio uncanniness / canned feel" with a "strict procedural-only rule."

6. Accessibility is v1 scope, not a stripped fallback. Accessibility appears in Scope as "screen-reader narration, reduced-motion mode, call captions, WCAG AA." Later the plan says reduced-motion is a "designed alternative surface (not stripped)," narration and notebook keep "identical naturalist tone," keyboard is fully supported, and "Accessibility regression" is mitigated with automated and manual WCAG checks.

7. Read-only, opt-in boundaries instead of social surfaces. The product is "for a single user," with "no social network surfaces." Visits are "read-only," "opt-in, revocable, off-by-default," "host only," and visitor access is a "read-only snapshot + limited narration." Observability also avoids "per-bird/per-account" metrics.

8. Performance and observability are part of the experience. The plan sets budgets for "Initial bundle <2MB gzipped," "Time-to-first-bird <500ms," "60fps idle," and "30min no memory growth." It instruments "first-bird timing, render fps, tick latency, narration queue depth, event loss rate" and sets a "p99 tick latency alarm >5s."

## Per-feature whys

Scope and ambiguity resolution:

- Browser-based virtual aviary for a single user: The plan ties this to the explicit non-goals of "no native apps" and "no social network surfaces," while keeping v1 to a "single user."
- 2 starter birds: NOT RECOVERABLE FROM PLAN
- Bird cap at 7: NOT RECOVERABLE FROM PLAN
- Ramp birds/aviary from 2 -> 7 using age-based gating: The plan places this under rollout instrumentation, so the recoverable why is controlled ramping while tracking "first-bird timing, render fps, tick latency, narration queue depth, event loss rate."
- Two starter birds selected server-side from ~6-species pool (no catalog): NOT RECOVERABLE FROM PLAN
- Adoption name picking is post-adoption: NOT RECOVERABLE FROM PLAN
- Procedural bird engine with personality vectors and mood: The plan uses this so bird behavior can be shaped by "personality vectors and mood," with mood transitions affected by "time-of-day, ambient weather, recent offers/listen-ins, personality influence."
- Personality vector never exposed to client/UI: The plan grounds this in server ownership: "No client-side simulation of personality," "Server is sole writer of vectors," and "Drift deltas authored only by simulation tick from ordered event log."
- Presence-based monotonic drift: The plan articulates this as "presence honesty," with drift "monotonic ↑ only," "presence-time dominant weight," target rates of "measurable after ~1 week" and "user-visible after ~3 weeks," and "No negative drift on neglect."
- Presence defined precisely by the three-condition conjunction: The plan marks this as "Ambiguity resolution," so the why is precision around what counts as presence.
- Listen-in: The plan uses listen-in as an event that shapes mood and drift, and in the audio pipeline it creates focus where "focused bird rises, others to ambient" with a "gradual" mix.
- Offers: The plan makes offers part of the simulation input: "recent offers/listen-ins" shape mood, and "listen-in/offer" are secondary weights for personality deltas.
- Settle: NOT RECOVERABLE FROM PLAN
- Field notebook (naturalist voice): The plan grounds this in "generation from simulation events" and in a shared tone with narration and captions: "Narration and notebook keep identical naturalist tone."
- Day/night + ambient weather: The plan uses "time-of-day" and "ambient weather" in mood transitions, plus "local-time palette shifts" on the client.
- Single horizontal scene: NOT RECOVERABLE FROM PLAN
- Magic-link auth: NOT RECOVERABLE FROM PLAN
- Server-side simulation tick: The plan uses the tick as the sole writer that reads recent events, updates mood timers and personality deltas, persists vectors/moods, and emits snapshots; this protects sync correctness and personality state.
- Multi-device sync with canonical server state: The plan says "all clients observe identical canonical aviary" and narrows conflict to "auth/session expiry."
- Read-only visit invitations (opt-in, revocable, off-by-default): The plan ties this to single-user and no-social boundaries: visit endpoints are host-controlled, "host only," and visitor access is "read-only snapshot + limited narration."
- Screen-reader narration: The plan makes this an accessibility surface with "slow-cadence running naturalist narration," generated from state and sharing the notebook voice.
- Reduced-motion mode: The plan says reduced-motion is a "designed alternative surface (not stripped)," using "cross-fade poses" and removing "ambient drift."
- Call captions: The plan uses captions as an accessibility and fallback surface, defaulted when WebAudio is unavailable and written in "naturalist prose."
- WCAG AA: The plan treats this as part of v1 accessibility and mitigates regression with "automated + manual WCAG checks in CI."
- No gamification/streaks/achievements/levels/scores: The plan gives the why as respecting "Explicit non-goals" and interpreting constraints "literally."
- No Tamagotchi (no hunger/death/distress): The plan gives the why as an explicit non-goal, reinforced by "No negative drift on neglect."
- No social network surfaces: The plan gives the why as an explicit non-goal, reinforced by read-only, opt-in, revocable visits and no profiles/feeds/discovery/leaderboards/comments.
- No notifications: The plan gives the why as respecting "Explicit non-goals"; no deeper rationale is recoverable.
- No personalization of scene beyond birds: The plan gives the why as respecting "Explicit non-goals"; no deeper rationale is recoverable.

Architecture and data:

- React + TypeScript + WebGL/WebAudio canvas or SVG fallback client: The plan uses this for a "thin renderer + input capture + state subscriber" that has "No ownership of canonical bird state."
- Auth service with magic-link, synthetic UUID account IDs, and session tokens: NOT RECOVERABLE FROM PLAN
- Simulation service with a single-writer tick consuming an event log: The plan uses this to update personality vectors and moods from ordered events and to avoid client-owned simulation.
- Snapshot service with small JSON snapshots served from CDN edge: The plan connects this to a small snapshot payload and fast rendering budgets, including "Time-to-first-bird <500ms."
- Notebook service generation from simulation events: The plan uses this to keep notebook prose derived from simulation state rather than free-floating copy.
- Account/settings service: NOT RECOVERABLE FROM PLAN
- Postgres for accounts/birds/vectors/notebook: NOT RECOVERABLE FROM PLAN
- Append-only event store / interactions and presence pings: The plan uses this so the simulation tick consumes ordered events and avoids "LWW."
- Render pipeline boundary where client receives snapshot, interpolates at 60fps, and sends events asynchronously: The plan uses this to keep the client responsive while preserving canonical server state.
- Account model with encrypted email: NOT RECOVERABLE FROM PLAN
- Bird model with stable UUID and personality vector fields: The plan uses stable IDs and vectors so per-bird state persists and can shape boldness, social_warmth, vocal_frequency, plumage_saturation, and curiosity.
- Mood model with enum state, timers, and last_transition_at: The plan uses this for the "fast timescale" layer of bird state.
- PresenceEvent / InteractionEvent append-only log: The plan uses this as the source for drift deltas and mood updates in the server tick.
- NotebookEntry with prose and timestamp: The plan connects this to generated naturalist notebook prose from simulation events.
- VisitInvite with expires_at and revoked_at: The plan uses this to make visits expiring and revocable.
- Snapshot as derived/cache: The plan uses snapshots as emitted tick output and as the small canonical payload clients consume.
- No LWW for drift deltas: The plan uses this to protect ordered server-authored personality changes and mitigate "Sync correctness / personality loss."

API surface:

- POST /auth/magic-link and POST /auth/verify: NOT RECOVERABLE FROM PLAN
- GET /aviary/snapshot: The plan uses this for "current canonical" state and multi-device sync.
- POST /events for offer, listen-in-start/end, settle, and presence-ping batch: The plan uses event writes as fire-and-forget inputs to the server simulation tick.
- GET /notebook: The plan uses this to expose generated notebook entries.
- POST /invites and DELETE /invites/:id (host only): The plan uses this for opt-in, revocable read-only visits.
- Visitor GET /visit/:token: The plan limits this to "read-only snapshot + limited narration."
- Snapshot payload small (<5KB): The plan connects this to a thin client, CDN snapshots, and time-to-first-bird performance.
- Events fire-and-forget: The plan uses this because clients write events only and do not own canonical bird state.

Simulation engine design:

- Server tick sequence: The plan uses the sequence to read events, update mood, compute monotonic personality deltas, advance seeds, persist state, and emit snapshots.
- Mood set of wary/content/curious/drowsy/alert + settled: NOT RECOVERABLE FROM PLAN
- Mood transitions from time-of-day, ambient weather, recent offers/listen-ins, and personality influence: The plan uses this so mood is state-derived and interaction-aware.
- Call grammar with motif library per species and personality-shaped timing/pitch variations: The plan uses this for recognizable but varied calls generated from state.
- WebAudio synthesis on client from seed: The plan uses this to keep audio procedural while snapshots provide call timing seeds.
- Drift low-pass filter calibrated and unit-tested: The plan uses this to hit the explicit rate targets and mitigate "Drift calibration drift."
- No negative drift on neglect: The plan uses this to preserve non-punitive "presence honesty" and avoid Tamagotchi-like distress.
- Idle-motion seeds and call timing seeds: The plan uses these so the client can interpolate motion and synthesize calls while the server remains canonical.

Sync model:

- Pull snapshot on visible/focus/keepalive/suspend-resume: The plan uses this so clients observe canonical state across device visibility and lifecycle changes.
- Interpolate motion locally: The plan uses this for 60fps rendering without client-side personality simulation.
- Write events only: The plan uses this to keep canonical state and personality vectors server-owned.
- Conflict surface only on auth/session expiry: The plan uses this to keep multi-device conflicts narrow and "matter-of-fact."

Frontend rendering pipeline:

- Three perch zones (front/mid/back): NOT RECOVERABLE FROM PLAN
- Canvas/WebGL primary or high-quality SVG; responsive, aspect-preserving, no bird cropping: The plan uses this so the horizontal scene remains visible and birds are not cropped across layouts.
- Day/night local-time palette shifts: The plan uses this to reflect day/night state in the scene.
- Subtle parallax foliage and leaf/feather ambient drift: The plan calls these "client-generated ornaments," so they decorate without owning simulation state.
- Idle micro-motion mood-shaped and personality-shaped: The plan uses this so visible behavior reflects mood and personality.
- Listen-in gradual mix ramp: The plan specifies "no hard cut or full silence" and keeps other birds ambient.
- Top bar sparse icons, auto-fade on inactivity: The plan uses this as restrained UI consistent with "notice, never announce."
- First frame birds already mid-action: The plan says "no wake animation" and "quiet-field loading state," supporting the quiet observational product voice.
- Keyboard tab/arrows/enter/escape with visible focus ring: The plan uses this for full keyboard accessibility.

Audio pipeline:

- WebAudio procedural synthesis from motif library: The plan uses this to avoid "recorded loops" and "canned feel."
- Per-bird recognizable call signature preserved across mood/drift: The plan uses this for recognizability, reinforced by "listener AB tests on recognizability."
- Chorus mixing with real variation, not phase artifacts: The plan uses this so simultaneous calls remain varied instead of artifact-prone.
- Listen-in audio focus: The plan uses this to raise the focused bird while keeping others ambient, with gradual transitions.
- Fallback silence plus captions when WebAudio unavailable: The plan uses this so the surface remains accessible without audio.
- Runtime caption generation from grammar: The plan uses this to keep captions in "naturalist prose" and aligned with procedural calls.

Accessibility surfaces:

- Slow-cadence running naturalist narration: The plan uses this as the screen-reader surface, generated from state at "30-60s idle" cadence.
- Priority bump on user events: The plan uses this so narration responds to user events without abandoning slow cadence.
- Captions as per-call floating text: The plan uses this to make calls legible in the same voice.
- Reduced-motion alternative surface: The plan uses this as a "designed alternative surface (not stripped)."
- All text WCAG AA, keyboard full navigation, focus visible on aviary: The plan uses these as accessibility requirements and CI regression targets.
- Narration and notebook identical naturalist tone: The plan uses this to keep product voice consistent across text surfaces.

Performance budgets and observability:

- Initial bundle <2MB gzipped: The plan places this in performance budgets; no deeper rationale is recoverable beyond initial load control.
- Time-to-first-bird <500ms: The plan uses this as a first-visible-experience budget.
- 60fps idle on 5yo laptop: The plan uses this as an idle rendering budget.
- 30min no memory growth: The plan ties this to "buffer reuse, bounded workers" and mitigates "Performance memory."
- Synthetic RUM + aggregate telemetry only: The plan uses this for observability while avoiding "per-bird/per-account" metrics.
- p99 tick latency alarm >5s: The plan uses this to watch server tick health.
- Last 2 major versions Chrome/Safari/Firefox/Edge: NOT RECOVERABLE FROM PLAN

Rollout and risk mitigations:

- Week 0-6 staged rollout: NOT RECOVERABLE FROM PLAN
- Internal dogfood + synthetic + limited beta before 100%: The plan uses this as the gate before full ship.
- Instrument first-bird timing, render fps, tick latency, narration queue depth, event loss rate: The plan uses these metrics to watch performance, simulation, accessibility, and event reliability.
- Early instrumented A/B on drift rates: The plan uses this to mitigate "Drift calibration drift" against the "measurable after 1wk / visible 3wk" target.
- Only-server-writer + event-ordering enforced: The plan uses this to mitigate "Sync correctness / personality loss."
- Soft-delete window + export + recovery tests: The plan uses these as additional mitigation for "Sync correctness / personality loss."
- Strict procedural-only rule and listener AB tests: The plan uses these to mitigate "Audio uncanniness / canned feel."
- Automated + manual WCAG checks in CI: The plan uses these to mitigate "Accessibility regression."
- Explicit 30-min test in CI gate: The plan uses this to mitigate "Performance memory."
- Lint rules + PRD references in every UI surface PR: The plan uses these to mitigate "Over-announcing temptation."
