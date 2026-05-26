# Comprehensive Implementation Plan for Pocket Aviary (v1)

## Scope
V1 implements a browser-based virtual aviary for a single user: 2 starter birds (cap at 7), procedural bird engine with personality vectors and mood, presence-based monotonic drift, listen-in, offers, settle, field notebook (naturalist voice), day/night + ambient weather, single horizontal scene, magic-link auth, server-side simulation tick, multi-device sync with canonical server state, read-only visit invitations (opt-in, revocable, off-by-default), screen-reader narration, reduced-motion mode, call captions, WCAG AA.

Explicit non-goals respected: no native apps, no gamification/streaks/achievements/levels/scores, no Tamagotchi (no hunger/death/distress), no social network surfaces (profiles/feeds/discovery/leaderboards/comments), no notifications, no personalization of scene beyond birds.

Ambiguity resolution: Two starter birds selected server-side from ~6-species pool (no catalog). Adoption name picking is post-adoption. Bird cap enforced in engine and UI. Presence defined precisely by the three-condition conjunction. Drift rate calibrated to produce measurable instrument change after ~1 week regular use, user-visible after ~3 weeks.

## Architecture
Client (React + TypeScript + WebGL/WebAudio canvas or SVG fallback): thin renderer + input capture + state subscriber. No ownership of canonical bird state.

Server (Node/TS or Go): 
- Auth service (magic-link via SES/SendGrid, synthetic UUID account IDs, session tokens).
- Simulation service: single-writer tick (~1/min) consuming event log → updating personality vectors/moods → writing snapshots. Event log append-only.
- Snapshot service: small JSON snapshots served from CDN edge.
- Notebook service: generation from simulation events.
- Account/settings service.

Data store: Postgres for accounts/birds/vectors/notebook; append-only event store (or Postgres table) for interactions/presence pings.

Render pipeline boundary: client receives snapshot (positions, moods, call timing seeds, day state); interpolates locally at 60fps; sends interaction events asynchronously.

No client-side simulation of personality. Server is sole writer of vectors.

## Data Model
- Account: {id: UUID, email_encrypted, created_at, settings}
- Bird: {id: UUID (stable), account_id, species_id, name, personality_vector: {boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}, created_at}
- Mood: per-bird enum state + timers + last_transition_at (fast timescale)
- PresenceEvent / InteractionEvent: append-only log {account_id, bird_id?, type, duration?, metadata, timestamp}
- NotebookEntry: {id, account_id, prose, timestamp}
- VisitInvite: {id, host_account_id, visitor_email, expires_at, revoked_at?}
- Snapshot (derived/cache): current per-bird state emitted by tick.

Personality vector never exposed to client/UI. Drift deltas authored only by simulation tick from ordered event log (no LWW).

## API Surface
REST/GraphQL (or tRPC):
- POST /auth/magic-link, POST /auth/verify
- GET /aviary/snapshot (current canonical)
- POST /events (offer, listen-in-start/end, settle, presence-ping batch)
- GET /notebook
- POST /invites, DELETE /invites/:id (host only)
- Visitor GET /visit/:token (read-only snapshot + limited narration)

Snapshot payload small (<5KB). Events fire-and-forget.

## Simulation Engine Design
Server tick:
1. Read recent events since last tick.
2. Update mood timers + transitions (time-of-day, ambient weather, recent offers/listen-ins, personality influence).
3. Compute personality deltas (presence-time dominant weight, listen-in/offer secondary; monotonic ↑ only).
4. Advance idle-motion seeds, call timing seeds.
5. Persist new vectors + moods.
6. Emit new snapshot.

Mood set: wary/content/curious/drowsy/alert + settled.
Call grammar: motif library per species + personality-shaped timing/pitch variations. WebAudio synthesis on client from seed.
Drift low-pass filter calibrated + unit-tested against target rates. No negative drift on neglect.

## Sync Model
Server-only canonical state. Clients:
- Pull snapshot on visible/focus/keepalive/suspend-resume.
- Interpolate motion locally.
- Write events only.
Multi-device: all clients observe identical canonical aviary. Conflict surface only on auth/session expiry (matter-of-fact error messages).

## Frontend Rendering Pipeline
- Single horizontal scene, three perch zones (front/mid/back).
- Canvas/WebGL primary (or high-quality SVG); responsive, aspect-preserving, no bird cropping.
- Day/night + local-time palette shifts, subtle parallax foliage, leaf/feather ambient drift (client-generated ornaments).
- Idle micro-motion mood-shaped + personality-shaped.
- Listen-in: gradual mix ramp (no hard cut or full silence).
- Top bar: sparse icons, auto-fade on inactivity.
- First frame: birds already mid-action (no wake animation; quiet-field loading state).
- Reduced-motion: cross-fade poses instead of frame animation; ambient drift removed.
- Keyboard: tab/arrows/enter/escape fully supported with visible focus ring.

## Audio Pipeline
- WebAudio procedural synthesis from motif library.
- Per-bird recognizable call signature preserved across mood/drift.
- Chorus mixing: simultaneous calls produce real variation, not phase artifacts.
- Listen-in: focused bird rises, others to ambient (gradual).
- Fallback: silence + captions defaulted when WebAudio unavailable.
- No recorded loops at any point.

Caption generation: runtime from grammar, naturalist prose.

## Accessibility Surfaces
- Screen-reader: slow-cadence running naturalist narration (same voice as notebook) generated from state; 30-60s idle, priority bump on user events.
- Captions: per-call floating text, same voice.
- Reduced-motion: designed alternative surface (not stripped).
- All text WCAG AA; keyboard full navigation; focus visible on aviary.
- Narration and notebook keep identical naturalist tone.

## Performance Budgets & Observability
- Initial bundle <2MB gzipped.
- Time-to-first-bird <500ms (mid-tier mobile 4G).
- 60fps idle on 5yo laptop; 30min no memory growth (buffer reuse, bounded workers).
- Synthetic RUM + aggregate telemetry only (no per-bird/per-account in metrics). p99 tick latency alarm >5s.
- Last 2 major versions Chrome/Safari/Firefox/Edge.

## Rollout
- Week 0-2: core engine + auth + snapshot + render.
- Week 3-4: audio + notebook + presence + drift tests.
- Week 5: accessibility (narration, reduced-motion, captions).
- Week 6: visits + sync edge cases + perf hardening.
- Ship to 100% after internal dogfood + synthetic + limited beta.
- Instrument: first-bird timing, render fps, tick latency, narration queue depth, event loss rate. Ramp birds/aviary from 2 -> 7 using age-based gating.

## Risks & Mitigations
- Drift calibration drift: early instrumented A/B on rates; explicit target "measurable after 1wk / visible 3wk".
- Sync correctness / personality loss: only-server-writer + event-ordering enforced; soft-delete window + export + recovery tests.
- Audio uncanniness / canned feel: strict procedural-only rule, listener AB tests on recognizability.
- Accessibility regression: narration & reduced-motion in v1 scope, automated + manual WCAG checks in CI.
- Performance memory: explicit 30-min test in CI gate.
- Over-announcing temptation: lint rules + PRD references in every UI surface PR.

All decisions interpret PRD constraints literally; naturalist voice + "notice, never announce" + presence honesty are load-bearing invariants across every layer.
